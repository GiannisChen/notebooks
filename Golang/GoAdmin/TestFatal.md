## TestFatal

原谅我的傻逼，竟然要在Golang里面测试日志结构的`Fatal`功能，但是，为什么不呢？都是函数，都能测试（不是）。然后我就犯了难，因为`Fatal`里面有一个这个函数：

```go
func Fatal() {
    ...
    os.Exit(1)
}
```

这下不得不另辟蹊径了，`Panic`同样也会引起程序的中断，但是Golang提供了`recover()`方法可以在`defer`申明中捕获异常并恢复，看起来像这样：

```go
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered in f", r)
    }
}()
```

但`Fatal`并没有这么直白的处理，我们问问万能的`StackOverflow`：

- https://stackoverflow.com/questions/30688554/how-to-test-go-function-containing-log-fatal

看起来还是云里雾里，再问问：

- https://stackoverflow.com/questions/26225513/how-to-test-os-exit-scenarios-in-go/26226382#26226382

似乎找到了好东西：

> There's a [presentation](https://talks.golang.org/2014/testing.slide) by Andrew Gerrand (one of the core members of the Go team) where he shows how to do it.
>
> Given a function (in `main.go`)
>
> ```golang
> package main
> 
> import (
>     "fmt"
>     "os"
> )
> 
> func Crasher() {
>     fmt.Println("Going down in flames!")
>     os.Exit(1)
> }
> ```
>
> here's how you would test it (through `main_test.go`):
>
> ```go
> package main
> 
> import (
>     "os"
>     "os/exec"
>     "testing"
> )
> 
> func TestCrasher(t *testing.T) {
>     if os.Getenv("BE_CRASHER") == "1" {
>         Crasher()
>         return
>     }
>     cmd := exec.Command(os.Args[0], "-test.run=TestCrasher")
>     cmd.Env = append(os.Environ(), "BE_CRASHER=1")
>     err := cmd.Run()
>     if e, ok := err.(*exec.ExitError); ok && !e.Success() {
>         return
>     }
>     t.Fatalf("process ran with err %v, want exit status 1", err)
> }
> ```
>
> What the code does is invoke `go test` again in a separate process through `exec.Command`, limiting execution to the `TestCrasher` test (via the `-test.run=TestCrasher` switch). It also passes in a flag via an environment variable (`BE_CRASHER=1`) which the second invocation checks for and, if set, calls the system-under-test, returning immediately afterwards to prevent running into an infinite loop. Thus, we are being dropped back into our original call site and may now validate the actual exit code.
>
> Source: [Slide 23](https://talks.golang.org/2014/testing.slide#23) of Andrew's presentation. The second slide contains a link to the [presentation's video](http://www.youtube.com/watch?v=ndmB0bj7eyw) as well. He talks about subprocess tests at [47:09](https://youtu.be/ndmB0bj7eyw?t=2834)

这是Golang团队中核心骨干流出的代码👆

跑一下，诶嘿，过了！