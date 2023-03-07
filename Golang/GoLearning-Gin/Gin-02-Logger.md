## 日志

- https://github.com/uber-go/zap
- https://pkg.go.dev/go.uber.org/zap
- https://github.com/natefinch/lumberjack

日志使用了`uber`的`zap`，其详细解析如文章所示：https://darjun.github.io/2020/04/23/godailylib/zap/，同时，也做了日志的分割，日志分割使用的组件是`lumberjack`。

我们使用了全局变量来表示我们自定义的日志组件：

`lib/log/logger.go`

```go
var Logger *zap.Logger
```

显然我们需要初始化她：

```go
func MustInitLogger(logPath string, errorLogPath string, logLevel string) {
	// set normal log level
	var level zapcore.Level
	switch strings.ToLower(logLevel) {
	case "debug":
		level = zap.DebugLevel
	case "info":
		level = zapcore.InfoLevel
	case "warning":
		level = zapcore.WarnLevel
	case "error":
		level = zapcore.ErrorLevel
	case "panic":
		level = zapcore.PanicLevel
	case "fatal":
		level = zapcore.FatalLevel
	default:
		level = zapcore.InfoLevel
	}
	normalLogPriority := zap.LevelEnablerFunc(func(lvl zapcore.Level) bool {
		return lvl >= level
	})

	logHook := lumberjack.Logger{
		Filename:   logPath,
		MaxSize:    100,
		MaxAge:     30,
		MaxBackups: 30,
		LocalTime:  false,
	}
	// above logLevel, multi-output: 1. File 2. STDOUT
	normalWs := zapcore.NewMultiWriteSyncer(zapcore.AddSync(&logHook), os.Stdout)
	encoderConfig := zapcore.EncoderConfig{
		TimeKey:        "time",
		LevelKey:       "level",
		NameKey:        "logger",
		CallerKey:      "lineNum",
		MessageKey:     "msg",
		StacktraceKey:  "stacktrace",
		LineEnding:     zapcore.DefaultLineEnding,
		EncodeLevel:    zapcore.LowercaseLevelEncoder,  // 小写编码器
		EncodeTime:     zapcore.ISO8601TimeEncoder,     // ISO8601 UTC 时间格式
		EncodeDuration: zapcore.SecondsDurationEncoder, //
		EncodeCaller:   zapcore.FullCallerEncoder,      // 全路径编码器
		EncodeName:     zapcore.FullNameEncoder,
	}

	// ErrorLevel and above, another path to store
	errLogHook := lumberjack.Logger{
		Filename:   errorLogPath,
		MaxSize:    100,
		MaxAge:     30,
		MaxBackups: 30,
		LocalTime:  false,
	}
	errLogPriority := zap.LevelEnablerFunc(func(lvl zapcore.Level) bool {
		return lvl >= zap.ErrorLevel
	})

	core := zapcore.NewTee(
		zapcore.NewCore(
			zapcore.NewConsoleEncoder(encoderConfig),
			normalWs,
			normalLogPriority,
		),
		zapcore.NewCore(
			zapcore.NewConsoleEncoder(encoderConfig),
			zapcore.AddSync(&errLogHook),
			errLogPriority,
		),
	)
	// add dev-mod, trace the stack
	caller := zap.AddCaller()
	// file and row number
	development := zap.Development()
	// init field, such as add a server name
	filed := zap.Fields(zap.String("application", "chat-room"))
	// build log
	Logger = zap.New(core, caller, development, filed)
	Logger.Info("Logger init success")
}
```

- `logHook`和`errLogHook`表示了`lumberjack`的钩子，用于日志分割文件，我们准备了两个钩子，因为我们想模仿`clickhouse`的模式：
  - `logHook`作为常规的日志输出，有其最低的预设等级`logLevel`，同时会从命令行输出；因此，我需要调用`zapcore.NewMultiWriteSyncer`来创建多输出的日志。
  - `errLogHook`是记录错误和崩溃的日志输出，不向命令行输出。

接下来我们做个小小的测试：

`lib/log/logger_test.go`

```go
MustInitLogger(
    "./testdata/log.log",
    "./testdata/error.log",
    "debug",
)
Logger.Debug("Debug Message")
Logger.Info("Info Message")
Logger.Warn("Warning Message")
Logger.Error("Error Message")
Logger.Panic("Panic Message")
Logger.Fatal("Fatal Message")
```

结果不错：

`error.log`

```log
2023-03-01T19:58:18.812+0800	error	F:/My Github Repositories/goChat/lib/log/logger_test.go:16	Error Message	{"application": "chat-room"}
2023-03-01T19:58:18.813+0800	panic	F:/My Github Repositories/goChat/lib/log/logger_test.go:17	Panic Message	{"application": "chat-room"}
```

`log.log`

```log
2023-03-01T19:58:18.797+0800	info	F:/My Github Repositories/goChat/lib/log/logger.go:100	Logger init success	{"application": "chat-room"}
2023-03-01T19:58:18.812+0800	debug	F:/My Github Repositories/goChat/lib/log/logger_test.go:13	Debug Message	{"application": "chat-room"}
2023-03-01T19:58:18.812+0800	info	F:/My Github Repositories/goChat/lib/log/logger_test.go:14	Info Message	{"application": "chat-room"}
2023-03-01T19:58:18.812+0800	warn	F:/My Github Repositories/goChat/lib/log/logger_test.go:15	Warning Message	{"application": "chat-room"}
2023-03-01T19:58:18.812+0800	error	F:/My Github Repositories/goChat/lib/log/logger_test.go:16	Error Message	{"application": "chat-room"}
2023-03-01T19:58:18.813+0800	panic	F:/My Github Repositories/goChat/lib/log/logger_test.go:17	Panic Message	{"application": "chat-room"}
2023-03-01T19:59:26.120+0800	fatal	F:/My Github Repositories/goChat/lib/log/logger_test.go:18	Fatal Message	{"application": "chat-room"}
```

