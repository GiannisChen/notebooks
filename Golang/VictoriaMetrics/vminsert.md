## vminsert



### RequestHandler

`vminsert` 在单机版里对应着一系列的路由，这类路由唯一的工作就是把各种不同格式的数据进行解析，然后狠狠地插入数据库中。由于**单机版**的原因，所有有关插入的操作都在一个 `RequestHandler` 里：

```go
// RequestHandler is a handler for Prometheus remote storage write API
func RequestHandler(w http.ResponseWriter, r *http.Request) bool {
	startTime := time.Now()
	defer requestDuration.UpdateDuration(startTime)

	path := strings.Replace(r.URL.Path, "//", "/", -1)
	switch path {
	case "/prometheus/api/v1/import", "/api/v1/import":
		vmimportRequests.Inc()
		if err := vmimport.InsertHandler(r); err != nil {
			vmimportErrors.Inc()
			httpserver.Errorf(w, r, "%s", err)
			return true
		}
		w.WriteHeader(http.StatusNoContent)
		return true
	...
	case "/ready":
		if rdy := atomic.LoadInt32(&promscrape.PendingScrapeConfigs); rdy > 0 {
			errMsg := fmt.Sprintf("waiting for scrape config to init targets, configs left: %d", rdy)
			http.Error(w, errMsg, http.StatusTooEarly)
		} else {
			w.Header().Set("Content-Type", "text/plain; charset=utf-8")
			w.WriteHeader(http.StatusOK)
			w.Write([]byte("OK"))
		}
		return true
	default:
		// This is not our link
		return false
	}
}
```

这部分的逻辑比较简单，通过 `switch-case` 来处理多种不同的写入，原理是一致的：

1. 将统计量 `xxRequests` `+1` 以便统计分析
2. 调用对应的 `xx.InsertHandler` 处理具体的业务（**⭐重点**）
3. 错误处理
4. `Response` 头部的写入



### InsertHandler

我们以 `vmimport.InsertHandler` 为例，讲这之前，我们先插入一段写入的数据格式，虽然试图稍稍泛化地讲一下，但是还是得考虑到为了更好理解，得知道数据到底长啥样：

```json
{"metric":{"__name__":"up","job":"node_exporter","instance":"localhost:9100"},"values":[0,0,0],"timestamps":[1549891472010,1549891487724,1549891503438]}
{"metric":{"__name__":"up","job":"prometheus","instance":"localhost:9090"},"values":[1,1,1],"timestamps":[1549891461511,1549891476511,1549891491511]}
```

这是通过 `/api/v1/export` 导出的数据，可以使用 `/api/v1/import` 导入，一个非常明显的 `json` 格式的数据。

```bash
# Import the data to <destination-victoriametrics>:
curl -X POST http://destination-victoriametrics:8428/api/v1/import/native -T exported_data.bin
```

数据格式扯完了，我们来看一下这行 `POST` 请求是怎么生效的。



#### InsertHandler

`vminsert\vmimport\request_handler.go`

```go
// InsertHandler processes `/api/v1/import` request.
//
// See https://github.com/VictoriaMetrics/VictoriaMetrics/issues/6
func InsertHandler(req *http.Request) error {
	extraLabels, err := parserCommon.GetExtraLabels(req)
	if err != nil {
		return err
	}
	return writeconcurrencylimiter.Do(func() error {
		isGzipped := req.Header.Get("Content-Encoding") == "gzip"
		return parser.ParseStream(req.Body, isGzipped, func(rows []parser.Row) error {
			return insertRows(rows, extraLabels)
		})
	})
}
```

首先我们会从 `http.Request` 里获得 `extraLabels`，这部分主要是做转储时候可以直接利用原有导出的数据，仅需要在 `POST` URL 里加点私货就行。接着在协程控制器的作用下，开启一个协程来处理。

处理的过程是通过 `callback` 的思路，可以说是**函数闭包**，也就意味着在 `ParseStream` 的同时处理 `insertRows` 。



#### ParseStream

接下来，我们要处理 `POST` 过来的具体信息，需要转移战场到 `protoparser\vmimport\streamparser.go`

```go
// ParseStream parses /api/v1/import lines from req and calls callback for the parsed rows.
//
// The callback can be called concurrently multiple times for streamed data from reader.
//
// callback shouldn't hold rows after returning.
func ParseStream(r io.Reader, isGzipped bool, callback func(rows []Row) error) error {
	// 读管道是采用压缩读还是正常读
    if isGzipped {
        // 池化操作，节省资源
		zr, err := common.GetGzipReader(r)
		if err != nil {
			return fmt.Errorf("cannot read gzipped vmimport data: %w", err)
		}
		defer common.PutGzipReader(zr)
		r = zr
	}
    // 还是池化操作，获得字节流读取的上下文
	ctx := getStreamContext(r)
	defer putStreamContext(ctx)
	for ctx.Read() {
        // 创建UnmarshalWork工作的worker
		uw := getUnmarshalWork()
        // 填入callback函数
		uw.callback = func(rows []Row) {
			if err := callback(rows); err != nil {
				ctx.callbackErrLock.Lock()
				if ctx.callbackErr == nil {
					ctx.callbackErr = fmt.Errorf("error when processing imported data: %w", err)
				}
				ctx.callbackErrLock.Unlock()
			}
			ctx.wg.Done()
		}
		uw.reqBuf, ctx.reqBuf = ctx.reqBuf, uw.reqBuf
		ctx.wg.Add(1)
        // 将worker加入工作流以便调度
		common.ScheduleUnmarshalWork(uw)
	}
	ctx.wg.Wait()
	if err := ctx.Error(); err != nil {
		return err
	}
	return ctx.callbackErr
}
```

这一下又不得不转移战场了，来到 `protoparser\common\unmarshal_work.go` ，看一下 `UnmarshalWork` 是怎么工作的：

```go
// ScheduleUnmarshalWork schedules uw to run in the worker pool.
// It is expected that StartUnmarshalWorkers is already called.
func ScheduleUnmarshalWork(uw UnmarshalWork) {
    // 加入调度channel
	unmarshalWorkCh <- uw
}
```

```go
// UnmarshalWork is a unit of unmarshal work.
type UnmarshalWork interface {
	// Unmarshal must implement CPU-bound unmarshal work.
    // 任务调度接口
	Unmarshal()
}

var (
	unmarshalWorkCh    chan UnmarshalWork
	unmarshalWorkersWG sync.WaitGroup
)
```

```go
// StartUnmarshalWorkers starts unmarshal workers.
func StartUnmarshalWorkers() {
	if unmarshalWorkCh != nil {
		logger.Panicf("BUG: it looks like startUnmarshalWorkers() has been alread called without stopUnmarshalWorkers()")
	}
	gomaxprocs := cgroup.AvailableCPUs()
	unmarshalWorkCh = make(chan UnmarshalWork, gomaxprocs)
	unmarshalWorkersWG.Add(gomaxprocs)
	for i := 0; i < gomaxprocs; i++ {
		// 开gomaxprocs个协程用来处理channel里的任务
        go func() {
			defer unmarshalWorkersWG.Done()
			for uw := range unmarshalWorkCh {
				uw.Unmarshal()
			}
		}()
	}
}

// StopUnmarshalWorkers stops unmarshal workers.
// No more calles to ScheduleUnmarshalWork are allowed after calling stopUnmarshalWorkers
func StopUnmarshalWorkers() {
	close(unmarshalWorkCh)
	unmarshalWorkersWG.Wait()
	unmarshalWorkCh = nil
}
```

`protoparser\vmimport\streamparser.go` 的接口具体实现：

```go
// Unmarshal implements common.UnmarshalWork
func (uw *unmarshalWork) Unmarshal() {
    // 处理行数据，返回Rows用于插入
	uw.rows.Unmarshal(bytesutil.ToUnsafeString(uw.reqBuf))
	rows := uw.rows.Rows
	for i := range rows {
		row := &rows[i]
		rowsRead.Add(len(row.Timestamps))
	}
    // 利用callback写rows
	uw.callback(rows)
	putUnmarshalWork(uw)
}
```

```go
// Unmarshal unmarshals influx line protocol rows from s.
//
// See https://docs.influxdata.com/influxdb/v1.7/write_protocols/line_protocol_tutorial/
//
// s shouldn't be modified when rs is in use.
func (rs *Rows) Unmarshal(s string) {
	rs.tu.reset()
    // 继续调用
	rs.Rows = unmarshalRows(rs.Rows[:0], s, &rs.tu)
}
```

```go
func unmarshalRows(dst []Row, s string, tu *tagsUnmarshaler) []Row {
	for len(s) > 0 {
		n := strings.IndexByte(s, '\n')
		if n < 0 {
			// The last line.
			return unmarshalRow(dst, s, tu)
		}
		dst = unmarshalRow(dst, s[:n], tu)
		s = s[n+1:]
	}
	return dst
}
```

```go
func unmarshalRow(dst []Row, s string, tu *tagsUnmarshaler) []Row {
	if len(s) > 0 && s[len(s)-1] == '\r' {
		s = s[:len(s)-1]
	}
	if len(s) == 0 {
		return dst
	}
	if cap(dst) > len(dst) {
		dst = dst[:len(dst)+1]
	} else {
		dst = append(dst, Row{})
	}
	r := &dst[len(dst)-1]
	if err := r.unmarshal(s, tu); err != nil {
		dst = dst[:len(dst)-1]
		logger.Errorf("cannot unmarshal json line %q: %s; skipping it", s, err)
		invalidLines.Inc()
	}
	return dst
}
```

```go
func (r *Row) unmarshal(s string, tu *tagsUnmarshaler) error {
	r.reset()
	v, err := tu.p.Parse(s)
	if err != nil {
		return fmt.Errorf("cannot parse json line: %w", err)
	}

	// Unmarshal tags
	metric := v.GetObject("metric")
	if metric == nil {
		return fmt.Errorf("missing `metric` object")
	}
	tagsStart := len(tu.tagsPool)
	if err := tu.unmarshalTags(metric); err != nil {
		return fmt.Errorf("cannot unmarshal `metric`: %w", err)
	}
	tags := tu.tagsPool[tagsStart:]
	r.Tags = tags[:len(tags):len(tags)]
	if len(r.Tags) == 0 {
		return fmt.Errorf("missing tags")
	}

	// Unmarshal values
	values := v.GetArray("values")
	if len(values) == 0 {
		return fmt.Errorf("missing `values` array")
	}
	for i, v := range values {
		f, err := v.Float64()
		if err != nil {
			return fmt.Errorf("cannot unmarshal value at position %d: %w", i, err)
		}
		r.Values = append(r.Values, f)
	}

	// Unmarshal timestamps
	timestamps := v.GetArray("timestamps")
	if len(timestamps) == 0 {
		return fmt.Errorf("missing `timestamps` array")
	}
	for i, v := range timestamps {
		ts, err := v.Int64()
		if err != nil {
			return fmt.Errorf("cannot unmarshal timestamp at position %d: %w", i, err)
		}
		r.Timestamps = append(r.Timestamps, ts)
	}

	if len(r.Timestamps) != len(r.Values) {
		return fmt.Errorf("`timestamps` array size must match `values` array size; got %d; want %d", len(r.Timestamps), len(r.Values))
	}
	return nil
}
```

显然调用到最底层就和开始提到的 `json` 格式相关咯~



#### insertRows（callback）

好了，我们要开启下一次地狱般的调用了。从这里开始，数据已经齐整，也就意味着 `vminsert` 的工作接近尾声，需要传给 `vmstorage` 模块来做短时间的存储和落盘持久化了。

这里，我们先回到 `vminsert\vmimport\request_handler.go` 上，看一下 `insertRows` ：

```go
func insertRows(rows []parser.Row, extraLabels []prompbmarshal.Label) error {
	// 经典资源池化，获取需要的上下文
    ctx := getPushCtx()
	defer putPushCtx(ctx)

	rowsLen := 0
	for i := range rows {
		rowsLen += len(rows[i].Values)
	}
	ic := &ctx.Common
    // 重置insertCtx（详情见下）
	ic.Reset(rowsLen)
	rowsTotal := 0
	hasRelabeling := relabel.HasRelabeling()
	for i := range rows {
		r := &rows[i]
		rowsTotal += len(r.Values)
		ic.Labels = ic.Labels[:0]
        // 添加label
		for j := range r.Tags {
			tag := &r.Tags[j]
			ic.AddLabelBytes(tag.Key, tag.Value)
		}
        // 添加额外的label
		for j := range extraLabels {
			label := &extraLabels[j]
			ic.AddLabel(label.Name, label.Value)
		}
		if hasRelabeling {
			ic.ApplyRelabeling()
		}
		if len(ic.Labels) == 0 {
			// Skip metric without labels.
			continue
		}
        // 排序，转化metrics的格式
		ic.SortLabelsIfNeeded()
		ctx.metricNameBuf = storage.MarshalMetricNameRaw(ctx.metricNameBuf[:0], ic.Labels)
		values := r.Values
		timestamps := r.Timestamps
		if len(timestamps) != len(values) {
			logger.Panicf("BUG: len(timestamps)=%d must match len(values)=%d", len(timestamps), len(values))
		}
		for j, value := range values {
			timestamp := timestamps[j]
            // 向insertCtx添加数据点
			if err := ic.WriteDataPoint(ctx.metricNameBuf, nil, timestamp, value); err != nil {
				return err
			}
		}
	}
	rowsInserted.Add(rowsTotal)
	rowsPerInsert.Update(float64(rowsTotal))
	return ic.FlushBufs()
}
```

```go
// Reset resets ctx for future fill with rowsLen rows.
func (ctx *InsertCtx) Reset(rowsLen int) {
	for i := range ctx.Labels {
		label := &ctx.Labels[i]
		label.Name = nil
		label.Value = nil
	}
	ctx.Labels = ctx.Labels[:0]

	for i := range ctx.mrs {
		mr := &ctx.mrs[i]
		mr.MetricNameRaw = nil
	}
	ctx.mrs = ctx.mrs[:0]
	if n := rowsLen - cap(ctx.mrs); n > 0 {
        // 一次扩容，避免append频繁扩容
		ctx.mrs = append(ctx.mrs[:cap(ctx.mrs)], make([]storage.MetricRow, n)...)
	}
    // 重新变成nil
	ctx.mrs = ctx.mrs[:0]
	ctx.metricNamesBuf = ctx.metricNamesBuf[:0]
	ctx.relabelCtx.Reset()
}
```

经过上述操作，我们成功把数据导入到了 `InsertCtx` 中，这个上下文，将是我们与 `vmstorage` 通信的关键。┏ (゜ω゜)=☞



#### InsertCtx

// todo