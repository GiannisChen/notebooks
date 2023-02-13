## FastHTTP 源码阅读

- Github地址：https://github.com/valyala/fasthttp



### 前言

`fasthttp` 是一个在很多场景中都出现过的项目，首先出场于 **valyala** 的时序数据库系统 [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics) 中，作为高性能数据库的接口使用；后来，在异军突起的后端架构 [fiber](https://github.com/gofiber/fiber) 中，也发现了 `fasthttp` 的身影；同时，在前段时间看协程池的时候，第三次看到了 `fasthttp` ，作为较火的 [ants](https://github.com/panjf2000/ants) 的思路来源而被提及。

`fasthttp` 和 `VictoriaMetrics` 中一脉相承了池化操作，池化让资源利用变得更加合理，同时也能避免过多的 GC 的介入。作为号称比原生 `net/http` 快上数倍乃至数十倍的包（当然，作者也强调了在一定的使用场景下，包括高负载等），自然需要一探究竟。作为笔者写下的第一篇原创源码解读，必然存在不足之处乃至谬误，如有机会定当返工订正。



### 初见

标准库 `net/http` 通过函数 `http.ListenAndServe` 提供服务，`fasthttp` 则提供了长得很像的函数 `fasthttp.ListenAndServe` 作为替代。我们以该函数为切入：

```go
package main

import (
	"fmt"
	"github.com/valyala/fasthttp"
)

func fastHTTPHandler(ctx *fasthttp.RequestCtx) {
	fmt.Fprintf(ctx, "Hi there! RequestURI is %q", ctx.RequestURI())
}

func main() {
	fasthttp.ListenAndServe(":8080", fastHTTPHandler)
}
```

运行一下：

![fasthttp-running-result](Images/fasthttp-running-result.png)

是一个简单的结果，那么其中有啥玄妙呢？

```go
// ListenAndServe serves HTTP requests from the given TCP addr
// using the given handler.
func ListenAndServe(addr string, handler RequestHandler) error {
	s := &Server{
		Handler: handler,
	}
	return s.ListenAndServe(addr)
}
```

首先会生成一个默认的 `Server` ，用来承载整个服务器的信息，`Server` 部分长这样：

```go
// Server implements HTTP server.
//
// Default Server settings should satisfy the majority of Server users.
// Adjust Server settings only if you really understand the consequences.
//
// It is forbidden copying Server instances. Create new Server instances
// instead.
//
// It is safe to call Server methods from concurrently running goroutines.
type Server struct {
	noCopy noCopy //nolint:unused,structcheck

	// Handler for processing incoming requests.
	//
	// Take into account that no `panic` recovery is done by `fasthttp` (thus any `panic` will take down the entire server).
	// Instead the user should use `recover` to handle these situations.
	Handler RequestHandler

	// ErrorHandler for returning a response in case of an error while receiving or parsing the request.
	//
	// The following is a non-exhaustive list of errors that can be expected as argument:
	//   * io.EOF
	//   * io.ErrUnexpectedEOF
	//   * ErrGetOnly
	//   * ErrSmallBuffer
	//   * ErrBodyTooLarge
	//   * ErrBrokenChunks
	ErrorHandler func(ctx *RequestCtx, err error)

	// HeaderReceived is called after receiving the header
	//
	// non zero RequestConfig field values will overwrite the default configs
	HeaderReceived func(header *RequestHeader) RequestConfig

	// ContinueHandler is called after receiving the Expect 100 Continue Header
	//
	// https://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html#sec8.2.3
	// https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.1.1
	// Using ContinueHandler a server can make decisioning on whether or not
	// to read a potentially large request body based on the headers
	//
	// The default is to automatically read request bodies of Expect 100 Continue requests
	// like they are normal requests
	ContinueHandler func(header *RequestHeader) bool

	// Server name for sending in response headers.
	//
	// Default server name is used if left blank.
	Name string

	// The maximum number of concurrent connections the server may serve.
	//
	// DefaultConcurrency is used if not set.
	//
	// Concurrency only works if you either call Serve once, or only ServeConn multiple times.
	// It works with ListenAndServe as well.
	Concurrency int

	// Per-connection buffer size for requests' reading.
	// This also limits the maximum header size.
	//
	// Increase this buffer if your clients send multi-KB RequestURIs
	// and/or multi-KB headers (for example, BIG cookies).
	//
	// Default buffer size is used if not set.
	ReadBufferSize int

	// Per-connection buffer size for responses' writing.
	//
	// Default buffer size is used if not set.
	WriteBufferSize int

	// ReadTimeout is the amount of time allowed to read
	// the full request including body. The connection's read
	// deadline is reset when the connection opens, or for
	// keep-alive connections after the first byte has been read.
	//
	// By default request read timeout is unlimited.
	ReadTimeout time.Duration

	// WriteTimeout is the maximum duration before timing out
	// writes of the response. It is reset after the request handler
	// has returned.
	//
	// By default response write timeout is unlimited.
	WriteTimeout time.Duration

	// IdleTimeout is the maximum amount of time to wait for the
	// next request when keep-alive is enabled. If IdleTimeout
	// is zero, the value of ReadTimeout is used.
	IdleTimeout time.Duration

	// Maximum number of concurrent client connections allowed per IP.
	//
	// By default unlimited number of concurrent connections
	// may be established to the server from a single IP address.
	MaxConnsPerIP int

	// Maximum number of requests served per connection.
	//
	// The server closes connection after the last request.
	// 'Connection: close' header is added to the last response.
	//
	// By default unlimited number of requests may be served per connection.
	MaxRequestsPerConn int

	// MaxKeepaliveDuration is a no-op and only left here for backwards compatibility.
	// Deprecated: Use IdleTimeout instead.
	MaxKeepaliveDuration time.Duration

	// MaxIdleWorkerDuration is the maximum idle time of a single worker in the underlying
	// worker pool of the Server. Idle workers beyond this time will be cleared.
	MaxIdleWorkerDuration time.Duration

	// Period between tcp keep-alive messages.
	//
	// TCP keep-alive period is determined by operation system by default.
	TCPKeepalivePeriod time.Duration

	// Maximum request body size.
	//
	// The server rejects requests with bodies exceeding this limit.
	//
	// Request body size is limited by DefaultMaxRequestBodySize by default.
	MaxRequestBodySize int

	// Whether to disable keep-alive connections.
	//
	// The server will close all the incoming connections after sending
	// the first response to client if this option is set to true.
	//
	// By default keep-alive connections are enabled.
	DisableKeepalive bool

	// Whether to enable tcp keep-alive connections.
	//
	// Whether the operating system should send tcp keep-alive messages on the tcp connection.
	//
	// By default tcp keep-alive connections are disabled.
	TCPKeepalive bool

	// Aggressively reduces memory usage at the cost of higher CPU usage
	// if set to true.
	//
	// Try enabling this option only if the server consumes too much memory
	// serving mostly idle keep-alive connections. This may reduce memory
	// usage by more than 50%.
	//
	// Aggressive memory usage reduction is disabled by default.
	ReduceMemoryUsage bool

	// Rejects all non-GET requests if set to true.
	//
	// This option is useful as anti-DoS protection for servers
	// accepting only GET requests and HEAD requests. The request size is limited
	// by ReadBufferSize if GetOnly is set.
	//
	// Server accepts all the requests by default.
	GetOnly bool

	// Will not pre parse Multipart Form data if set to true.
	//
	// This option is useful for servers that desire to treat
	// multipart form data as a binary blob, or choose when to parse the data.
	//
	// Server pre parses multipart form data by default.
	DisablePreParseMultipartForm bool

	// Logs all errors, including the most frequent
	// 'connection reset by peer', 'broken pipe' and 'connection timeout'
	// errors. Such errors are common in production serving real-world
	// clients.
	//
	// By default the most frequent errors such as
	// 'connection reset by peer', 'broken pipe' and 'connection timeout'
	// are suppressed in order to limit output log traffic.
	LogAllErrors bool

	// Will not log potentially sensitive content in error logs
	//
	// This option is useful for servers that handle sensitive data
	// in the request/response.
	//
	// Server logs all full errors by default.
	SecureErrorLogMessage bool

	// Header names are passed as-is without normalization
	// if this option is set.
	//
	// Disabled header names' normalization may be useful only for proxying
	// incoming requests to other servers expecting case-sensitive
	// header names. See https://github.com/valyala/fasthttp/issues/57
	// for details.
	//
	// By default request and response header names are normalized, i.e.
	// The first letter and the first letters following dashes
	// are uppercased, while all the other letters are lowercased.
	// Examples:
	//
	//     * HOST -> Host
	//     * content-type -> Content-Type
	//     * cONTENT-lenGTH -> Content-Length
	DisableHeaderNamesNormalizing bool

	// SleepWhenConcurrencyLimitsExceeded is a duration to be slept of if
	// the concurrency limit in exceeded (default [when is 0]: don't sleep
	// and accept new connections immediately).
	SleepWhenConcurrencyLimitsExceeded time.Duration

	// NoDefaultServerHeader, when set to true, causes the default Server header
	// to be excluded from the Response.
	//
	// The default Server header value is the value of the Name field or an
	// internal default value in its absence. With this option set to true,
	// the only time a Server header will be sent is if a non-zero length
	// value is explicitly provided during a request.
	NoDefaultServerHeader bool

	// NoDefaultDate, when set to true, causes the default Date
	// header to be excluded from the Response.
	//
	// The default Date header value is the current date value. When
	// set to true, the Date will not be present.
	NoDefaultDate bool

	// NoDefaultContentType, when set to true, causes the default Content-Type
	// header to be excluded from the Response.
	//
	// The default Content-Type header value is the internal default value. When
	// set to true, the Content-Type will not be present.
	NoDefaultContentType bool

	// KeepHijackedConns is an opt-in disable of connection
	// close by fasthttp after connections' HijackHandler returns.
	// This allows to save goroutines, e.g. when fasthttp used to upgrade
	// http connections to WS and connection goes to another handler,
	// which will close it when needed.
	KeepHijackedConns bool

	// CloseOnShutdown when true adds a `Connection: close` header when when the server is shutting down.
	CloseOnShutdown bool

	// StreamRequestBody enables request body streaming,
	// and calls the handler sooner when given body is
	// larger then the current limit.
	StreamRequestBody bool

	// ConnState specifies an optional callback function that is
	// called when a client connection changes state. See the
	// ConnState type and associated constants for details.
	ConnState func(net.Conn, ConnState)

	// Logger, which is used by RequestCtx.Logger().
	//
	// By default standard logger from log package is used.
	Logger Logger

	// TLSConfig optionally provides a TLS configuration for use
	// by ServeTLS, ServeTLSEmbed, ListenAndServeTLS, ListenAndServeTLSEmbed,
	// AppendCert, AppendCertEmbed and NextProto.
	//
	// Note that this value is cloned by ServeTLS, ServeTLSEmbed, ListenAndServeTLS
	// and ListenAndServeTLSEmbed, so it's not possible to modify the configuration
	// with methods like tls.Config.SetSessionTicketKeys.
	// To use SetSessionTicketKeys, use Server.Serve with a TLS Listener
	// instead.
	TLSConfig *tls.Config

	// FormValueFunc, which is used by RequestCtx.FormValue and support for customising
	// the behaviour of the RequestCtx.FormValue function.
	//
	// NetHttpFormValueFunc gives a FormValueFunc func implementation that is consistent with net/http.
	FormValueFunc FormValueFunc

	nextProtos map[string]ServeHandler

	concurrency      uint32
	concurrencyCh    chan struct{}
	perIPConnCounter perIPConnCounter

	ctxPool        sync.Pool
	readerPool     sync.Pool
	writerPool     sync.Pool
	hijackConnPool sync.Pool

	// We need to know our listeners and idle connections so we can close them in Shutdown().
	ln []net.Listener

	idleConns   map[net.Conn]time.Time
	idleConnsMu sync.Mutex

	mu   sync.Mutex
	open int32
	stop int32
	done chan struct{}
}
```

`Server` 中的属性大致可以分为以下几个部分：

- **服务器配置**

  ```go
  type Server struct {
      Name string
      Concurrency int
      ...
      StreamRequestBody bool
  }
  ```

  与如何使用该服务器有关，并不是这里的重点。

- **日志**

  ```go
  type Server struct {
      Logger Logger
  }
  ```

  日志，在本片文章中更加不重要了😀

- **池化**

  ```go
  type Server struct {
      ctxPool        sync.Pool
  	readerPool     sync.Pool
  	writerPool     sync.Pool
  	hijackConnPool sync.Pool
      ...
  }
  ```

  重要部分，是节约内存开支的重要渠道，一般用 `sync.Pool` 。

- **同步控制**

  ```go
  type Server struct {
      open int32
  	stop int32
  	done chan struct{}
      ...
  }
  ```

  同步控制用的，防止协程之间无序运行。



### 池化部分

让我们回到**调用栈**上：

```go
// ListenAndServe serves HTTP requests from the given TCP4 addr.
// Pass custom listener to Serve if you need listening on non-TCP4 media
// such as IPv6.
// Accepted connections are configured to enable TCP keep-alives.
func (s *Server) ListenAndServe(addr string) error {
	ln, err := net.Listen("tcp4", addr)
	if err != nil {
		return err
	}
	return s.Serve(ln)
}

// Serve serves incoming connections from the given listener.
// Serve blocks until the given listener returns permanent error.
func (s *Server) Serve(ln net.Listener) error {
	...
	s.mu.Lock()
	{
		// s.done和s.concurrencyCh的初始化
	}
	s.mu.Unlock()

	wp := &workerPool{
		WorkerFunc:            s.serveConn,
		MaxWorkersCount:       maxWorkersCount,
		LogAllErrors:          s.LogAllErrors,
		MaxIdleWorkerDuration: s.MaxIdleWorkerDuration,
		Logger:                s.logger(),
		connState:             s.setState,
	}
	wp.Start()
	...
}
```

首先来看上半部分，开启一个新工作池 `workerPool` ：

```go
// workerPool serves incoming connections via a pool of workers
// in FILO order, i.e. the most recently stopped worker will serve the next
// incoming connection.
// Such a scheme keeps CPU caches hot (in theory).
type workerPool struct {
	// Function for serving server connections.
	// It must leave c unclosed.
	WorkerFunc ServeHandler
	MaxWorkersCount int
	LogAllErrors bool
	MaxIdleWorkerDuration time.Duration
	Logger Logger

	lock         sync.Mutex
	workersCount int
	mustStop     bool

	ready []*workerChan
	stopCh chan struct{}
	workerChanPool sync.Pool
	connState func(net.Conn, ConnState)
}

type workerChan struct {
	lastUseTime time.Time
	ch          chan net.Conn
}
```



#### workerPool.Start() 

```go
func (wp *workerPool) Start() {
	if wp.stopCh != nil {
		panic("BUG: workerPool already started")
	}
	wp.stopCh = make(chan struct{})
	stopCh := wp.stopCh
	wp.workerChanPool.New = func() interface{} {
		return &workerChan{
			ch: make(chan net.Conn, workerChanCap),
		}
	}
	go func() {
		var scratch []*workerChan
		for {
			wp.clean(&scratch)
			select {
			case <-stopCh:
				return
			default:
				time.Sleep(wp.getMaxIdleWorkerDuration())
			}
		}
	}()
}
```

一些简单的初始化，主要是 `wp.workerChanPool.New` 设定初始化函数。开启一个协程（类似于守护协程或者 GC 协程），用于监控“过期”的 `workers` ，每次都会休眠 `maxIdleWorkerDuration` 的时间。直到有关闭整个 `workerPool` 的通知（经典 `stopCh` ）发送了关闭消息。



#### workerPool.Stop()

```go
func (wp *workerPool) Stop() {
	if wp.stopCh == nil {
		panic("BUG: workerPool wasn't started")
	}
	close(wp.stopCh)
	wp.stopCh = nil

	// Stop all the workers waiting for incoming connections.
	// Do not wait for busy workers - they will stop after
	// serving the connection and noticing wp.mustStop = true.
	wp.lock.Lock()
	ready := wp.ready
	for i := range ready {
		ready[i].ch <- nil
		ready[i] = nil
	}
	wp.ready = ready[:0]
	wp.mustStop = true
	wp.lock.Unlock()
}
```

对所有准备就绪的 `workerChan` 逐一清除，当然，运行中的 `workers` 并不会因此受到影响，他们的关闭是在读到 `wp.mustStop` 为 `true` 的时候自己关闭。



#### workerPool.clean()

```go
func (wp *workerPool) clean(scratch *[]*workerChan) {
	maxIdleWorkerDuration := wp.getMaxIdleWorkerDuration()

	// Clean least recently used workers if they didn't serve connections
	// for more than maxIdleWorkerDuration.
	criticalTime := time.Now().Add(-maxIdleWorkerDuration)

	wp.lock.Lock()
	ready := wp.ready
	n := len(ready)

	// Use binary-search algorithm to find out the index of the least recently worker which can be cleaned up.
	l, r, mid := 0, n-1, 0
	for l <= r {
		mid = (l + r) / 2
		if criticalTime.After(wp.ready[mid].lastUseTime) {
			l = mid + 1
		} else {
			r = mid - 1
		}
	}
	i := r
	if i == -1 {
		wp.lock.Unlock()
		return
	}

	*scratch = append((*scratch)[:0], ready[:i+1]...)
	m := copy(ready, ready[i+1:])
	for i = m; i < n; i++ {
		ready[i] = nil
	}
	wp.ready = ready[:m]
	wp.lock.Unlock()

	// Notify obsolete workers to stop.
	// This notification must be outside the wp.lock, since ch.ch
	// may be blocking and may consume a lot of time if many workers
	// are located on non-local CPUs.
	tmp := *scratch
	for i := range tmp {
		tmp[i].ch <- nil
		tmp[i] = nil
	}
}
```

- 首先上半部分是通过一个**二分算法**找到第一个最少最近使用（Least Recently）的连接，换句话说，就是上次使用时间比较靠前的那些，划分标准就是比 `criticalTime` 要早，剩余的就扔到 `scratch` 里面。
- 下半部分就涉及 `scratch` 里的元素一一通知需要关闭了，这个操作在 `Stop()` 中已经见过了。



#### workerPool.getCh()

```go
func (wp *workerPool) getCh() *workerChan {
	var ch *workerChan
	createWorker := false

	wp.lock.Lock()
	ready := wp.ready
	n := len(ready) - 1
	if n < 0 {
		if wp.workersCount < wp.MaxWorkersCount {
			createWorker = true
			wp.workersCount++
		}
	} else {
		ch = ready[n]
		ready[n] = nil
		wp.ready = ready[:n]
	}
	wp.lock.Unlock()

	if ch == nil {
		if !createWorker {
			return nil
		}
		vch := wp.workerChanPool.Get()
		ch = vch.(*workerChan)
		go func() {
			wp.workerFunc(ch)
			wp.workerChanPool.Put(vch)
		}()
	}
	return ch
}
```

这是我认为比较重要的函数之一，`getCh` 会返回一个 `workerChan` 的引用，但是我并不需要知道这个 `workerChan` 具体是在哪儿的：是曾经用过（`wp.ready`）还是船新版本（`n < 0`）。所以这里会有个判断，如果 `ready` 就绪队列里已经有了，就让暖玩胎的先跑，然后再去访问内存池，看看内存池里有没有我们需要的。当然，与 `VictoriaMetrics` 中略带差异的是，由于设置了 `sync.Pool.New` ，Go会自动返回非 `nil` 的 `any` ，也许是个进步（大概？）。

如果 `workerChan` 达到了上限呢，也就是 `MaxWorkersCount` ，那么将返回 `nil` 表示找不到。



#### workerPool.release(ch *workerChan)

```go
func (wp *workerPool) release(ch *workerChan) bool {
	ch.lastUseTime = time.Now()
	wp.lock.Lock()
	if wp.mustStop {
		wp.lock.Unlock()
		return false
	}
	wp.ready = append(wp.ready, ch)
	wp.lock.Unlock()
	return true
}
```

这里首先更新了一下使用时间，然后扔给了 `ready` 队列做复用，当然，如果 `wp` 已经在此之前就被通知关闭，那这个 `workerChan` 就直接被无视交给GC处理了。

这一部分也就是 [ants](https://github.com/panjf2000/ants) 借鉴的部分，很简单但是很巧妙，和数据库的连接池有相似的美感。



### worker工作

```go
func (wp *workerPool) workerFunc(ch *workerChan) {
	var c net.Conn

	var err error
	for c = range ch.ch {
		if c == nil {
			break
		}

		if err = wp.WorkerFunc(c); err != nil && err != errHijacked {
			errStr := err.Error()
			if wp.LogAllErrors || !(strings.Contains(errStr, "broken pipe") ||
				strings.Contains(errStr, "reset by peer") ||
				strings.Contains(errStr, "request headers: small read buffer") ||
				strings.Contains(errStr, "unexpected EOF") ||
				strings.Contains(errStr, "i/o timeout") ||
				errors.Is(err, ErrBadTrailer)) {
				wp.Logger.Printf("error when serving connection %q<->%q: %v", c.LocalAddr(), c.RemoteAddr(), err)
			}
		}
		if err == errHijacked {
			wp.connState(c, StateHijacked)
		} else {
			_ = c.Close()
			wp.connState(c, StateClosed)
		}
		c = nil

		if !wp.release(ch) {
			break
		}
	}

	wp.lock.Lock()
	wp.workersCount--
	wp.lock.Unlock()
}
```

调用先前传入的 `ServeHandler` 类型的 `WorkerFunc` ，用于处理传入的 `net.Conn` ，剩下的就是一些边界检查和错误处理。



### 剩下的

```go
func (s *Server) Serve(ln net.Listener) error {
	...
	// Count our waiting to accept a connection as an open connection.
	// This way we can't get into any weird state where just after accepting
	// a connection Shutdown is called which reads open as 0 because it isn't
	// incremented yet.
	atomic.AddInt32(&s.open, 1)
	defer atomic.AddInt32(&s.open, -1)

	for {
		if c, err = acceptConn(s, ln, &lastPerIPErrorTime); err != nil {
			wp.Stop()
			if err == io.EOF {
				return nil
			}
			return err
		}
		s.setState(c, StateNew)
		atomic.AddInt32(&s.open, 1)
		if !wp.Serve(c) {
			atomic.AddInt32(&s.open, -1)
			s.writeFastError(c, StatusServiceUnavailable,
				"The connection cannot be served because Server.Concurrency limit exceeded")
			c.Close()
			s.setState(c, StateClosed)
			if time.Since(lastOverflowErrorTime) > time.Minute {
				s.logger().Printf("The incoming connection cannot be served, because %d concurrent connections are served. "+
					"Try increasing Server.Concurrency", maxWorkersCount)
				lastOverflowErrorTime = time.Now()
			}

			// The current server reached concurrency limit,
			// so give other concurrently running servers a chance
			// accepting incoming connections on the same address.
			//
			// There is a hope other servers didn't reach their
			// concurrency limits yet :)
			//
			// See also: https://github.com/valyala/fasthttp/pull/485#discussion_r239994990
			if s.SleepWhenConcurrencyLimitsExceeded > 0 {
				time.Sleep(s.SleepWhenConcurrencyLimitsExceeded)
			}
		}
		c = nil
	}
}
```

```go
func acceptConn(s *Server, ln net.Listener, lastPerIPErrorTime *time.Time) (net.Conn, error) {
	for {
		c, err := ln.Accept()
		if err != nil {
			if c != nil {
				panic("BUG: net.Listener returned non-nil conn and non-nil error")
			}
			if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
				s.logger().Printf("Timeout error when accepting new connections: %v", netErr)
				time.Sleep(time.Second)
				continue
			}
			if err != io.EOF && !strings.Contains(err.Error(), "use of closed network connection") {
				s.logger().Printf("Permanent error when accepting new connections: %v", err)
				return nil, err
			}
			return nil, io.EOF
		}
		if c == nil {
			panic("BUG: net.Listener returned (nil, nil)")
		}

		if tc, ok := c.(*net.TCPConn); ok && s.TCPKeepalive {
			if err := tc.SetKeepAlive(s.TCPKeepalive); err != nil {
				tc.Close() //nolint:errcheck
				return nil, err
			}
			if s.TCPKeepalivePeriod > 0 {
				if err := tc.SetKeepAlivePeriod(s.TCPKeepalivePeriod); err != nil {
					tc.Close() //nolint:errcheck
					return nil, err
				}
			}
		}

		if s.MaxConnsPerIP > 0 {
			pic := wrapPerIPConn(s, c)
			if pic == nil {
				if time.Since(*lastPerIPErrorTime) > time.Minute {
					s.logger().Printf("The number of connections from %s exceeds MaxConnsPerIP=%d",
						getConnIP4(c), s.MaxConnsPerIP)
					*lastPerIPErrorTime = time.Now()
				}
				continue
			}
			c = pic
		}
		return c, nil
	}
}
```

剩下的一些也没其他的了，**堂堂休刊**！