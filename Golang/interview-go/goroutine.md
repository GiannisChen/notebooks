## Goroutine

本章主要关注 Go 最具特色的协程（`goroutine`）相关的内容。



#### Q001-交替打印数字和字母

**问题**

使用两个 `goroutine` 交替打印序列，一个 `goroutine` 打印数字， 另外一个 `goroutine` 打印字母， 最终效果如下：

```
12AB34CD56EF78GH910IJ1112KL1314MN1516OP1718QR1920ST2122UV2324WX2526YZ2728
```

**思路**

简单的 `channel` 控制两个独立的 `goroutine` 进行相互通知做到同步，这是原博客的答案：

```go
letter,number := make(chan bool),make(chan bool)
wait := sync.WaitGroup{}

go func() {
    i := 1
    for {
        select {
            case <-number:
            fmt.Print(i)
            i++
            fmt.Print(i)
            i++
            letter <- true
        }
    }
}()
wait.Add(1)
go func(wait *sync.WaitGroup) {
    i := 'A'
    for{
        select {
            case <-letter:
            if i >= 'Z' {
                wait.Done()
                return
            }

            fmt.Print(string(i))
            i++
            fmt.Print(string(i))
            i++
            number <- true
        }

    }
}(&wait)
number<-true
wait.Wait()
```

但是似乎出现了一个逻辑问题：我希望在 `wait.Wait()` 执行完成时，打印已经全部完成，但是事实上，**最后一次数字的打印可能在** `wait.Wait()` **之后**。这在简单的手搓中似乎没什么，可以利用主线程（比如 `main` 等待全部 `goroutine` 执行完后才结束的特性），但是我认为这么改才合理（虽然丑了点）：

```go
number, letter := make(chan int), make(chan bool)
wait := sync.WaitGroup{}

go func() {
    i := 1
    for {
        select {
            case n := <-number:
            fmt.Print(i, i+1)
            i += 2
            if n == 1 {
                letter <- true
            } else {
                wait.Done()
                return
            }
        }
    }
}()
wait.Add(1)

go func() {
    l := 'A'
    for {
        select {
            case <-letter:
            fmt.Print(string(l), string(l+1))
            l += 2
            if l > 'Z' {
                number <- 2
                return
            } else {
                number <- 1
            }
        }
    }
}()

number <- 1
wait.Wait()
```



