## Golang 协程池设计



Golang 自从出生就身带**高并发**的标签，其并发编程就是由 `goroutine` 实现的，因其消耗资源低，性能高效，开发成本低的特性而被广泛应用到各种场景，例如服务端开发中使用的HTTP服务，在 Golang [net/http](https://github.com/golang/go/blob/master/src/net/http/server.go) 包中，每一个被监听到的tcp链接都是由一个 `goroutine` 去完成处理其上下文的，由此使得其拥有极其优秀的并发量吞吐量。

```go
for {
        // 监听tcp
        rw, e := l.Accept()
        if e != nil {
            .......
        }
        tempDelay = 0
        c := srv.newConn(rw)
        c.setState(c.rwc, StateNew) // before Serve can return
        // 启动协程处理上下文
        go c.serve(ctx)
}
```

虽然创建一个 `goroutine` 占用的内存极小（大约2KB左右，线程通常2M左右），但是在实际生产环境无限制的开启协程显然是不科学的，比如上图的逻辑，如果来几千万个请求就会开启几千万个 `goroutine` ，当没有更多内存可用时，go的调度器就会阻塞 `goroutine` 最终导致内存溢出乃至严重的崩溃，所以本文将通过实现一个简单的协程池，以及剖析几个开源的协程池源码来探讨一下对 `goroutine` 的并发控制以及多路复用的设计和实现。



### 一个简单的协程池

```go
package main
import (
  "fmt"
  "sync"
  "time"
)


type SimplePool struct {
    wg   sync.WaitGroup
    work chan func() //任务队列
}

func NewSimplePoll(workers int) *SimplePool {
    p := &SimplePool{
        wg:   sync.WaitGroup{},
        work: make(chan func()),
    }
    p.wg.Add(workers)
    //初始化协程池（添加任务前）时就根据指定的并发量去读取管道并执行，以免添加任务时管道阻塞
    for i := 0; i < workers; i++ {
        go func() {
            defer func() {
                // 捕获异常 防止waitGroup阻塞
                if err := recover(); err != nil {
                    fmt.Println(err)
                    p.wg.Done()
                }
            }()
            // 从workChannel中取出任务执行
            for fn := range p.work {
                fn()
            }
            p.wg.Done()
        }()
    }
    return p
}
// 添加任务
func (p *SimplePool) Add(fn func()) {
    p.work <- fn
}

// 执行
func (p *SimplePool) Run() {
    close(p.work)
    p.wg.Wait()
}
// 测试使用
func main() {
    p := NewSimplePoll(20)
    for i := 0; i < 100; i++ {
        p.Add(parseTask(i))
    }
    p.Run()
}

func parseTask(i int) func() {
    return func() {
        // 模拟抓取数据的过程
        time.Sleep(time.Second * 1)
        fmt.Println("finish parse ", i)
    }
}
```



### go-playground/pool

上面的 `goroutine` 池虽然简单，但是对于每一个并发任务的状态，pool的状态缺少控制，所以又去看了一下 [go-playground/pool](https://github.com/go-playground/pool) 的源码实现，先从每一个需要执行的任务入手，该库中对并发单元做了如下的结构体，可以看到除工作单元的值，错误，执行函数等，还用了三个分别表示，**取消**，**取消中**，**写** 的三个并发安全的原子操作值来标识其运行状态。

```go
// 需要加入pool 中执行的任务
type WorkFunc func(wu WorkUnit) (interface{}, error)

// 工作单元
type workUnit struct {
    value      interface{}    // 任务结果 
    err        error          // 任务的报错
    done       chan struct{}  // 通知任务完成
    fn         WorkFunc    
    cancelled  atomic.Value   // 任务是否被取消
    cancelling atomic.Value   // 是否正在取消任务
    writing    atomic.Value   // 任务是否正在执行
}
```

接下来看Pool的结构：

```go
// limitedPool contains all information for a limited pool instance.
type limitedPool struct {
    workers uint            // 并发量 
    work    chan *workUnit  // 任务channel
    cancel  chan struct{}   // 用于通知结束的channel
    closed  bool            // 是否关闭
    m       sync.RWMutex    // 读写锁，主要用来保证 closed值的并发安全
}
```

初始化 `goroutine` 池, 以及启动设定好数量的 `goroutine`：

```go
// 初始化pool,设定并发量
// NewLimited returns a new limited pool instance
func NewLimited(workers uint) *limitedPool {

	if workers == 0 {
		panic("invalid workers '0'")
	}

	p := &limitedPool{
		workers: workers,
	}

	p.initialize()

	return p
}

func (p *limitedPool) initialize() {

	p.work = make(chan *workUnit, p.workers*2)
	p.cancel = make(chan struct{})
	p.closed = false

	// fire up workers here
	for i := 0; i < int(p.workers); i++ {
		p.newWorker(p.work, p.cancel)
	}
}

// passing work and cancel channels to newWorker() to avoid any potential race condition
// betweeen p.work read & write
func (p *limitedPool) newWorker(work chan *workUnit, cancel chan struct{}) {
	go func(p *limitedPool) {

		var wu *workUnit

		defer func(p *limitedPool) {
			if err := recover(); err != nil {

				trace := make([]byte, 1<<16)
				n := runtime.Stack(trace, true)

				s := fmt.Sprintf(errRecovery, err, string(trace[:int(math.Min(float64(n), float64(7000)))]))

				iwu := wu
				iwu.err = &ErrRecovery{s: s}
				close(iwu.done)

				// need to fire up new worker to replace this one as this one is exiting
				p.newWorker(p.work, p.cancel)
			}
		}(p)

		var value interface{}
		var err error

		for {
			select {
			case wu = <-work:

				// possible for one more nilled out value to make it
				// through when channel closed, don't quite understad the why
				if wu == nil {
					continue
				}

				// support for individual WorkUnit cancellation
				// and batch job cancellation
				if wu.cancelled.Load() == nil {
					value, err = wu.fn(wu)

					wu.writing.Store(struct{}{})

					// need to check again in case the WorkFunc cancelled this unit of work
					// otherwise we'll have a race condition
					if wu.cancelled.Load() == nil && wu.cancelling.Load() == nil {
						wu.value, wu.err = value, err

						// who knows where the Done channel is being listened to on the other end
						// don't want this to block just because caller is waiting on another unit
						// of work to be done first so we use close
						close(wu.done)
					}
				}

			case <-cancel:
				return
			}
		}

	}(p)
}
```

```go
// Queue queues the work to be run, and starts processing immediately
func (p *limitedPool) Queue(fn WorkFunc) *workUnit {

	w := &workUnit{
		done: make(chan struct{}),
		fn:   fn,
	}

	go func() {
		p.m.RLock()
		if p.closed {
			w.err = &ErrPoolClosed{s: errClosed}
			if w.cancelled.Load() == nil {
				close(w.done)
			}
			p.m.RUnlock()
			return
		}

		p.work <- w

		p.m.RUnlock()
	}()

	return w
}
```

在 `go-playground/pool` 包中， `limitedPool` 的批量并发执行还需要借助 `batch.go` 来完成：

```go
// batch contains all information for a batch run of WorkUnits
type batch struct {
    pool    Pool          // 上面的limitedPool实现了Pool interface
    m       sync.Mutex    // 互斥锁，用来判断closed
    units   []WorkUnit    // 工作单元的slice， 这个主要用在不设并发限制的场景，这里忽略
    results chan WorkUnit // 结果集,执行完后的workUnit会更新其value,error,可以从结果集channel中读取
    done    chan struct{} // 通知batch是否完成
    closed  bool
    wg      *sync.WaitGroup
}
```

```go
//  go-playground/pool 中有设置并发量和不设并发量的批量任务，都实现Pool interface，初始化batch批量任务时会将之前创建好的Pool传入newBatch
func newBatch(p Pool) Batch {
    return &batch{
        pool:    p,
        units:   make([]WorkUnit, 0, 4), // capacity it to 4 so it doesn't grow and allocate too many times.
        results: make(chan WorkUnit),
        done:    make(chan struct{}),
        wg:      new(sync.WaitGroup),
    }
}

// 往批量任务中添加workFunc任务
func (b *batch) Queue(fn WorkFunc) {

    b.m.Lock()
    if b.closed {
        b.m.Unlock()
        return
    }
    //往上述的limitPool中添加workFunc
    wu := b.pool.Queue(fn)

    b.units = append(b.units, wu) // keeping a reference for cancellation purposes
    b.wg.Add(1)
    b.m.Unlock()
    
    // 执行完后将workUnit写入结果集channel
    go func(b *batch, wu WorkUnit) {
        wu.Wait()
        b.results <- wu
        b.wg.Done()
    }(b, wu)
}

// 通知批量任务不再接受新的workFunc, 如果添加完workFunc不执行改方法的话将导致取结果集时done channel一直阻塞
func (b *batch) QueueComplete() {
    b.m.Lock()
    b.closed = true
    close(b.done)
    b.m.Unlock()
}

// 获取批量任务结果集
func (b *batch) Results() <-chan WorkUnit {
    go func(b *batch) {
        <-b.done
        b.m.Lock()
        b.wg.Wait()
        b.m.Unlock()
        close(b.results)
    }(b)
    return b.results
}
```

测试：

```go
func SendMail(int int) pool.WorkFunc {
    fn := func(wu pool.WorkUnit) (interface{}, error) {
        // sleep 1s 模拟发邮件过程
        time.Sleep(time.Second * 1)
        // 模拟异常任务需要取消
        if int == 17 {
            wu.Cancel()
        }
        if wu.IsCancelled() {
            return false, nil
        }
        fmt.Println("send to", int)
        return true, nil
    }
    return fn
}

func TestBatchWork(t *testing.T) {
    // 初始化groutine数量为20的pool
    p := pool.NewLimited(20)
    defer p.Close()
    batch := p.Batch()
    // 设置一个批量任务的过期超时时间
    t := time.After(10 * time.Second)
    go func() {
        for i := 0; i < 100; i++ {
            batch.Queue(SendMail(i))
        }
        batch.QueueComplete()
    }()
    // 因为 batch.Results 中要close results channel 所以不能将其放在LOOP中执行
    r := batch.Results()
LOOP:
    for {
        select {
        case <-t:
        // 登台超时通知
            fmt.Println("recived timeout")
            break LOOP
     
        case email, ok := <-r:
        // 读取结果集
            if ok {
                if err := email.Error(); err != nil {
                    fmt.Println("err", err.Error())
                }
                fmt.Println(email.Value())
            } else {
                fmt.Println("finish")
                break LOOP
            }
        }
    }
}
```

`go-playground/pool` 在比起之前简单的协程池的基础上， 对pool，worker的状态有了很好的管理。但是，但是问题来了，在第一个实现的简单 `goroutine` 池和 `go-playground/pool` 中，都是先启动预定好的 `goroutine` 来完成任务执行，在并发量远小于任务量的情况下确实能够做到 `goroutine` 的复用，如果任务量不多则会导致任务分配到每个 `goroutine` 不均匀，甚至可能出现启动的 `goroutine` 根本不会执行任务从而导致浪费，而且对于协程池也没有动态的扩容和缩小。所以我又去看了一下[ants](https://github.com/panjf2000/ants)的设计和实现。



### panjf2000/ants

[ants](https://github.com/panjf2000/ants) 是一个受 [fasthttp](https://github.com/valyala/fasthttp) 启发的高性能协程池，`fasthttp` 号称是比go原生的 `net/http` 快10倍，其快速高性能的原因之一就是采用了各种池化技术。`ants` 相比之前两种协程池，其模型更像是之前接触到的数据库连接池，需要从空余的 `worker` 中取出一个来执行任务, 当无可用空余 `worker` 的时候再去创建，而当 `pool` 的容量达到上线之后，剩余的任务阻塞等待当前进行中的 `worker` 执行完毕将 `worker` 放回 `pool` ，直至 `pool` 中有空闲 `worker` 。 `ants` 在内存的管理上做得很好，除了定期清除过期 `worker` （一定时间内没有分配到任务的 `worker` ），`ants` 还实现了一种适用于大批量相同任务的 `pool` ，这种 `pool` 与一个需要大批量重复执行的函数锁绑定，避免了调用方不停的创建，更加节省内存。

先看一下 `ants` 的 `pool` 结构体（`pool.go`）

```go
type Pool struct {
    // 协程池的容量 (groutine数量的上限)
    capacity int32
    // 正在执行中的groutine
    running int32
    // 过期清理间隔时间
    expiryDuration time.Duration
    // 当前可用空闲的groutine
    workers []*Worker
    // 表示pool是否关闭
    release int32
    // lock for synchronous operation.
    lock sync.Mutex
    // 用于控制pool等待获取可用的groutine
    cond *sync.Cond
    // 确保pool只被关闭一次
    once sync.Once
    // worker临时对象池，在复用worker时减少新对象的创建并加速worker从pool中的获取速度
    workerCache sync.Pool
    // pool引发panic时的执行函数
    PanicHandler func(interface{})
}
```

工作单元 `worker` （`worker.go`）

```go
type Worker struct {
    // worker 所属的pool;
    pool *Pool
    // 任务队列
    task chan func()
    // 回收时间，即该worker的最后一次结束运行的时间
    recycleTime time.Time
}
```

执行 `worker` 的代码（`worker.go`）

```go
func (w *Worker) run() {
    // pool中正在执行的worker数+1
    w.pool.incRunning()
    go func() {
        defer func() {
            if p := recover(); p != nil {
                //若worker因各种问题引发panic, 
                //pool中正在执行的worker数 -1，         
                //如果设置了Pool中的PanicHandler，此时会被调用
                w.pool.decRunning()
                if w.pool.PanicHandler != nil {
                    w.pool.PanicHandler(p)
                } else {
                    log.Printf("worker exits from a panic: %v", p)
                }
            }
        }()
        
        // worker 执行任务队列
        for f := range w.task {
            //任务队列中的函数全部被执行完后，
            //pool中正在执行的worker数 -1， 
            //将worker 放回对象池
            if f == nil {
                w.pool.decRunning()
                w.pool.workerCache.Put(w)
                return
            }
            f()
            //worker 执行完任务后放回Pool 
            //使得其余正在阻塞的任务可以获取worker
            w.pool.revertWorker(w)
        }
    }()
}
```

了解了工作单元 `worker` 如何执行任务以及与 `pool` 交互后，回到 `pool` 中查看其实现， `pool` 的核心就是取出可用 `worker` 提供给任务执行（`pool.go`）

```go
// 向pool提交任务
func (p *Pool) Submit(task func()) error {
    if 1 == atomic.LoadInt32(&p.release) {
        return ErrPoolClosed
    }
    // 获取pool中的可用worker并向其任务队列中写入任务
    p.retrieveWorker().task <- task
    return nil
}


// **核心代码** 获取可用worker
func (p *Pool) retrieveWorker() *Worker {
    var w *Worker

    p.lock.Lock()
    idleWorkers := p.workers
    n := len(idleWorkers) - 1
  // 当前pool中有可用worker, 取出(队尾)worker并执行
    if n >= 0 {
        w = idleWorkers[n]
        idleWorkers[n] = nil
        p.workers = idleWorkers[:n]
        p.lock.Unlock()
    } else if p.Running() < p.Cap() {
        p.lock.Unlock()
        // 当前pool中无空闲worker,且pool数量未达到上线
        // pool会先从临时对象池中寻找是否有已完成任务的worker,
        // 若临时对象池中不存在，则重新创建一个worker并将其启动
        if cacheWorker := p.workerCache.Get(); cacheWorker != nil {
            w = cacheWorker.(*Worker)
        } else {
            w = &Worker{
                pool: p,
                task: make(chan func(), workerChanCap),
            }
        }
        w.run()
    } else {
        // pool中没有空余worker且达到并发上限
        // 任务会阻塞等待当前运行的worker完成任务释放会pool
        for {
            p.cond.Wait() // 等待通知， 暂时阻塞
            l := len(p.workers) - 1
            if l < 0 {
                continue
            }
            // 当有可用worker释放回pool之后， 取出
            w = p.workers[l]
            p.workers[l] = nil
            p.workers = p.workers[:l]
            break
        }
        p.lock.Unlock()
    }
    return w
}

// 释放worker回pool
func (p *Pool) revertWorker(worker *Worker) {
    worker.recycleTime = time.Now()
    p.lock.Lock()
    p.workers = append(p.workers, worker)
    // 通知pool中已经获取锁的groutine, 有一个worker已完成任务
    p.cond.Signal()
    p.lock.Unlock()
}
```

在批量并发任务的执行过程中， 如果有超过5纳秒（`ants` 中默认 `worker` 过期时间为5ns）的 `worker` 未被分配新的任务，则将其作为过期 `worker` 清理掉，从而保证 `pool` 中可用的 `worker` 都能发挥出最大的作用以及将任务分配得更均匀（`pool.go`）

```go
// 该函数会在pool初始化后在协程中启动
func (p *Pool) periodicallyPurge() {
    // 创建一个5ns定时的心跳
    heartbeat := time.NewTicker(p.expiryDuration)
    defer heartbeat.Stop()

    for range heartbeat.C {
        currentTime := time.Now()
        p.lock.Lock()
        idleWorkers := p.workers
        if len(idleWorkers) == 0 && p.Running() == 0 && atomic.LoadInt32(&p.release) == 1 {
            p.lock.Unlock()
            return
        }
        n := -1
        for i, w := range idleWorkers {
            // 因为pool 的worker队列是先进后出的，所以正序遍历可用worker时前面的往往里当前时间越久
            if currentTime.Sub(w.recycleTime) <= p.expiryDuration {
                break
            }    
            // 如果worker最后一次运行时间距现在超过5纳秒，视为过期，worker收到nil, 执行上述worker.go中 if n == nil 的操作
            n = i
            w.task <- nil
            idleWorkers[i] = nil
        }
        if n > -1 {
            // 全部过期
            if n >= len(idleWorkers)-1 {
                p.workers = idleWorkers[:0]
            } else {
            // 部分过期
                p.workers = idleWorkers[n+1:]
            }
        }
        p.lock.Unlock()
    }
}
```

测试：

```go
func TestAnts(t *testing.T) {
    wg := sync.WaitGroup{}
    pool, _ := ants.NewPool(20)
    defer pool.Release()
    for i := 0; i < 100; i++ {
        wg.Add(1)
        pool.Submit(sendMail(i, &wg))
    }
    wg.Wait()
}

func sendMail(i int, wg *sync.WaitGroup) func() {
    return func() {
        time.Sleep(time.Second * 1)
        fmt.Println("send mail to ", i)
        wg.Done()
    }
}
```

这里虽只简单的测试批量并发任务的场景， 如果大家有兴趣可以去看看 `ants` 的压力测试， `ants` 的吞吐量能够比原生 `goroutine` 高出N倍，内存节省10到20倍， 可谓是协程池中的神器。



### valyala/fasthttp

- https://github.com/valyala/fasthttp