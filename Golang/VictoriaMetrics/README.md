## README



### 前言

[VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics)是[valyala](https://github.com/valyala)团队力作，展示了原始野性而又功能强劲的一款时序数据库，其中有很多模块都做了重写，有名的就是[fasthttp](https://github.com/valyala/fasthttp)，以及其他⭐超过1k的如：[quicktemplate](https://github.com/valyala/quicktemplate)，[fastcache](https://github.com/VictoriaMetrics/fastcache)，[fastjson](https://github.com/valyala/fastjson)，显得非常牛逼。

经过使用和源码阅读，`VictoriaMetrics` 的确能当得起作者的豪言壮语，当然代价就是简陋。首先他要求使用者有一定的 `Prometheus` 的使用背景，因为这一款时序数据库是其有持久化存储的替代品，虽然当下 `Prometheus` 已经有了持久化的解决方案，而 `VictoriaMetrics` 内部更像是一个简易版的 `ClickHouse` ，俄乌真是不分家啊。

说 `VictoriaMetrics` 简陋是因为其只提供了专门供给 `devops` 使用的查询语言接口 `promql` `graphql` 以及 `influxdb` 采用的部分协议，而非我们更加熟悉的 `SQL` 。同样，他并不支持部分数据的删除，这一点会让很多用 `VictoriaMetrics` 的用户头疼，因为这可能无法契合他们的使用方式，但这在 `devops` 中是可以理解的。同样， `VictoriaMetrics` 采用的是类似于 K-V 的存储策略，使用 `label` 作为其标识，这方便分散的传感器直接与数据库通信，但是对于集中发送某个时间戳下一系列的采集数据，就会出现时间戳写放大的问题。同样，确保了高性能的写入，其查询，特别是近一段时间的统计查询会出现一定的“错误”，同时其查询逻辑并没有 `SQL` 那么明析，有一定的门槛。

但是，`VitoriaMetrics` 仍是一款优秀的时序数据库产品，同样其简洁明了的逻辑让笔者选择他作为数据库源码分析的对象，希望能从中收获一些实质性的内容。



### 概览

文章除了 `README` 部分讲一些碎碎念，将会分成三个部分，主要讲解 `VictoriaMetrics` 起查询、存储、落盘功能的几个部分，分别是 `vmselect` ， `vminsert` ， `vmstorage` 。有了这几个功能，数据库的雏形基本完善，当然 `VictoriaMetrics` 还有很多很酷的功能模块，但是限于篇幅和笔者认知，暂且按下不表。

![../Images/victoriametrics-overview](../Images/victoriametrics-overview.svg)



> 单机版的 `VictoriaMetrics` 和集群版的架构上略有出入，单机版 `VictoriaMetrics` 将所有服务集成在了一个 `http.Server` 中，而集群版则是各自独立的服务。当然，好奇的是，`VictoriaMetrics` 竟然没用 `fasthttp` ，神奇。

我们暂时只关注**单机版**，首先我们来看一下 `victoria-metrics\main.go` 的主函数（省略一大部分 `logger` ）：

```go
func main() {
	// Write flags and help message to stdout, since it is easier to grep or pipe.
	flag.CommandLine.SetOutput(os.Stdout)
	flag.Usage = usage
	envflag.Parse()
	buildinfo.Init()
	logger.Init()

	startTime := time.Now()
	storage.SetDedupInterval(*minScrapeInterval)
	vmstorage.Init(promql.ResetRollupResultCacheIfNeeded)
	vmselect.Init()
	vminsert.Init()
	startSelfScraper()

	go httpserver.Serve(*httpListenAddr, requestHandler)

	stopSelfScraper()
	if err := httpserver.Stop(*httpListenAddr); err != nil {
		logger.Fatalf("cannot stop the webservice: %s", err)
	}
	vminsert.Stop()
	vmstorage.Stop()
	vmselect.Stop()
	fs.MustStopDirRemover()
}
```

```go
func requestHandler(w http.ResponseWriter, r *http.Request) bool {
	if r.URL.Path == "/" {
		if r.Method != "GET" {
			return false
		}
		...
		return true
	}
	if vminsert.RequestHandler(w, r) { return true }
	if vmselect.RequestHandler(w, r) { return true }
	if vmstorage.RequestHandler(w, r) { return true }
	return false
}
```

主协程将三个模块 `vminsert`， `vmselect` ， `vmstorage` 进行初始化，然后用 `sync.WaitGroup` 等待通知，而真正的服务，则在另一个协程上监听着，这个才是重点。`Request` 的处理非常的暴力，简单来说就是一串 `if-else` 来判断，当处理路由不是那么多时，貌似是个更好的选择。剩下的，就到各个专栏去探寻内部奥秘吧。
