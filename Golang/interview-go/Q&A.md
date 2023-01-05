## Questions and Answers



### `new` 和 `make` 的区别？

- https://go.dev/doc/effective_go#allocation_new
- https://go.dev/doc/effective_go#allocation_make

Go有两个分配原语，内置函数 `new` 和 `make` 。它们做不同的事情，适用于不同的类型，这可能会令人困惑，但规则很简单。

**`new`**

- 它是一个分配内存的内置函数，但与其他一些语言中的同名函数不同，**它不初始化内存**，**只将其归零**。也就是说，`new(T)` 为类型为 `T` 的新项分配零存储，并返回其地址，即类型为 `*T` 的值。在Go术语中，它**返回一个指针**，指向新分配的类型为 `T` 的零值。

- 由于 `new` 返回的内存是归零的，因此在设计数据结构时，最好安排每个类型的零值，使得 `new` 初始化的变量可以在不进行进一步初始化的情况下使用。这意味着数据结构的用户可以使用 `new` 创建一个数据结构并开始工作。例如，在 `bytes.Buffer` 的文档中提到：

  > `Buffer` 的零值是一个可以使用的空缓冲区。

  同样，`sync.Mutex`没有显式的构造函数或 `Init` 方法。相反，`sync.Mutex` 的零值被定义为未锁定的互斥锁。

- 零值是可以传递的，也就是说结构体所有属性含有零值初始化，那么该结构体也是可以被 `new` 初始化为立即可用的零值，而不需要额外生成。考虑以下例子：

  ```go
  type SyncedBuffer struct {
      lock    sync.Mutex
      buffer  bytes.Buffer
  }
  ```

  `SyncedBuffer` 类型的值也可以在分配或声明时立即使用。在下一个代码片段中，`p` 和 `v` 都将正确工作，无需进一步安排。

  ```go
  p := new(SyncedBuffer)  // type *SyncedBuffer
  var v SyncedBuffer      // type  SyncedBuffer
  ```

- 简单概括就是如下：（不完全正确但是很贴切）

  ```go
  func newInt() *int {
    var i int
    return &i
  }
  someVar := newInt()
  ```

  

**`make`**

- 内置函数 `make(T, args...)` 的用途与 `new(T)` 不同。它只创建切片（`slice`）、映射（`map`）和通道（`channel`），并返回一个初始化的（未归零的）`T` 类型值（而不是`*T`）。

  区别的原因是，**这三种类型表示在使用前必须初始化的数据结构的引用**。例如，`slice` 是一个包含三项的描述符，其中包含指向数据（在数组中）的指针、长度和容量，在这些项初始化之前，`slice` 为 `nil`。对于切片、映射和通道，`make` 初始化内部数据结构并准备值以供使用。例如：

  ```go
  make([]int, 10, 100)
  ```

  分配一个包含100个 `int` 的数组，然后创建一个长度为10、容量为100的切片结构，指向数组的前10个元素。相反，`new([]int)` 返回一个指向新分配的零切片结构的指针，也就是说，一个指向 `nil` 切片值的指针。



1. `new` 和 `make` 的区别？用一个 `[]int` 就能很好地展现：

   ```go
   new([]int)		// &[]int(nil)
   make([]int)		// []int{}
   // fmt.Printf("%#+v", ...)
   ```

   ```go
   var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
   var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints
   
   // Unnecessarily complex:
   var p *[]int = new([]int)
   *p = make([]int, 100, 100)
   
   // Idiomatic:
   v := make([]int, 100)
   ```

2. 记住 `make` 只应用于映射、切片和通道，不返回指针。若要获得显式指针，可使用 `new` 进行分配或显式获取变量的地址。



### `array` 和 `slice` 的区别？

- https://go.dev/doc/effective_go#arrays
- https://go.dev/doc/effective_go#slices

`array` 在规划内存的详细布局时非常有用，有时可以帮助避免分配，但它们主要是 `slice` 的底层实现。Go 的数组有一些不同于经典 C 语言中的特性：

- `array` 是个值，而非 C 中的数组的概念，所以 `array` 的赋值是一种拷贝操作。
- 特别是，如果将 `array` 传递给函数，函数将接收 `array` 的拷贝，而不是指向该 `array` 的指针。
- `array` 的大小是其类型的一部分。类型 `[10]int` 和 `[20]int` 是不同的。

> 值属性可能是**昂贵的操作**，同时可以使用类似于 C 的方式高效访问 `array` ：
>
> ```go
> func Sum(a *[3]float64) (sum float64) {
>     for _, v := range *a {
>         sum += v
>     }
>     return
> }
> 
> array := [...]float64{7.0, 8.5, 9.1}
> x := Sum(&array)  // Note the explicit address-of operator
> ```

`slice` 包装数组，为数据序列提供更通用、更强大、更方便的接口。除了具有显式维数的项（如转换矩阵），Go 中的大多数**数组编程**都是用 `slice` 而不是简单的 `array` 完成的。

`slice` 包含对底层 `array` 的引用，如果将一个 `slice` 分配给另一个 `slice` ，两个 `slice` 都会引用同一个 `array`。如果一个函数接受一个 `slice` 参数，它对 `slice` 元素所做的更改将对调用者可见，类似于传递一个指向底层 `array` 的指针。因此，`Read` 函数可以接受 `slice` 参数，而不是指针和计数； `slice` 中的长度设置读取数据量的上限。下面是 `os` 包中 `File` 类型的 `Read` 方法的签名：

```go
func (f *File) Read(buf []byte) (n int, err error)
```

该方法返回读取的字节数和错误值。要读取较大缓冲区 `buf` 的前32个字节，请对缓冲区进行切片：

```go
n, err := f.Read(buf[0:32])
```

这样的切片是常见和有效的。事实上，暂且不考虑效率，下面的代码段还将读取缓冲区的前32个字节：

```go
var n int
var err error
for i := 0; i < 32; i++ {
    nbytes, e := f.Read(buf[i:i+1])  // Read one byte.
    n += nbytes
    if nbytes == 0 || e != nil {
        err = e
        break
    }
}
```

`slice` 的长度可以改变，只要它仍然符合底层数组的限制；只需要给它自己分配一个切片。一个片的容量，由内置函数 `cap` 访问，报告了片可以假定的最大长度。下面是一个将数据追加到 `slice` 的函数。如果数据超过容量，则重新分配该片。返回结果片。该函数使用 `len` 和 `cap` 在应用于 `nil` 片时也是合法的，并返回0。

```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}
```

之后必须返回片，因为尽管 `append` 可以修改 `slice` 的元素，但 `slice` 本身（包含指针、长度和容量的运行时数据结构）是通过值传递的。

追加到 `slice` 的想法非常有用，它被 `append` 内置函数捕获。不过，为了理解这个函数的设计，我们需要更多的信息，所以我们将在后面讨论它。



### `slice` 扩容策略

`slice` 的主要实现是扩容。对于 `append` 向 `slice` 添加元素时：

1. 假如 `slice` 容量够用，则追加新元素进去，`slice.len++` ，返回原来的 `slice` 。当
2. 原容量不够，则 `slice` 先扩容，扩容之后 `slice` 得到新的 `slice`，将元素追加进新的 `slice`，`slice.len++`，返回新的 `slice`。对于切片的扩容规则：
   1. 当切片比较小时（容量小于 1024），则采用较大的扩容倍速进行扩容（新的扩容会是原来的 2 倍），避免频繁扩容，从而减少**内存分配**的次数和**数据拷贝**的代价。
   2. 当切片较大的时（原来的 `slice` 的容量大于或者等于 1024），采用较小的扩容倍速（新的扩容将扩大大于或者等于原来 1.25 倍），主要避免空间浪费，网上其实很多总结的是 1.25 倍，那是在不考虑内存对齐的情况下，实际上还要考虑**内存对齐**，扩容是大于或者等于 1.25 倍。



### `defer` 的作用和用法？

- https://go.dev/doc/effective_go#defer

Go 的 `defer` 语句在函数结束之前安排了立即运行一个函数调用（被延迟的函数）。这是一种不常见但有效的方法，用于处理一些情况，比如不管函数采用哪条路径返回，都必须释放资源。典型的例子是解锁互斥锁或关闭文件。因此，他的执行顺序被设计为：

1. `return`（最先）
2. `return val`
3. `defer`（最后）

```go
func a() int {
	var i int
    defer func() { i *= 2 }()
	defer fmt.Println("1. i = ", i)
	defer func() { fmt.Println("2. i = ", i) }()
	defer func(i int) { fmt.Println("3. i = ", i) }(i)
	i = 7
	return i
}

func main() {
	println(a())
}
// output:
// 3. i =  0
// 2. i =  7
// 1. i =  0
// 7  
```

在这个代码块中，存在以下几个 `defer` 的用法：

1. **函数方法**：直接请求参数并直接带入，故输出是0；
2. **无参闭包**：会使用函数中的变量，由于是执行时请求，此时 `i` 已经改变；
3. **有参闭包**：直接请求参数并直接带入，故输出是0；
4. **值方式修改返回值**：第一个 `defer` 试图修改返回值，但是返回值已被赋值，故无法修改；但是返回值是指针时，地址不变，是可以修改对应值的，比如：

```go
func a() *int {
	var i int
	p := &i
	defer func() { *p *= 2 }()
	i = 7
	return &i
}

func main() {
	println(*a())
}
// output 
// 14
```

更常见的用法：

```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

`defer` 关键字调用**闭包**等函数的调用有两个优点：

1. 它保证您永远不会忘记关闭文件，如果稍后编辑该函数以添加新的返回路径，则很容易犯这个错误。
2. 它意味着闭值位于开值附近，这比将它放在函数的末尾要清楚得多。

延迟函数的参数（如果函数是方法，则包括接收器）在延迟执行时计算，而不是在调用执行时计算。除了避免变量在函数执行时改变值的问题外，这意味着单个延迟调用站点可以延迟多个函数的执行。这里有一个愚蠢的例子：

```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

延迟函数是按**后进先出**（**LIFO**）顺序执行的，所以这段代码将导致函数返回时打印`4 3 2 1 0`。一个更合理的例子是通过程序跟踪函数执行的简单方法，我们可以写几个简单的跟踪例程，像这样：

```go
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

每个 `defer` 语句都对应一个 `defer` 实例，多个实例使用指针连接起来形成一个单链表，保存在 `gotoutine` 数据结构中，每次插入 `defer` 实例，均插入到链表的头部，函数结束再一次从头部取出，从而形成**后进先出**的效果。

**规则**：

1. 延迟函数的参数是 `defer` 语句出现的时候就已经确定了的。
2. 延迟函数执行按照后进先出的顺序执行，即先出现的 `defer` 最后执行。
3. 延迟函数可能操作主函数的返回值。
4. 申请资源后立即使用 `defer` 关闭资源是个好习惯。



### `for-range` 的时候，内部发生了什么变化？

```go
for a,b ：= range sillyList {
	...
}
```

`a` 和 `b` 在内存中只会定义和分配一次内存，随后的循环都是对 `a` 和 `b` 进行值覆盖，而其指向的地址始终不变。由于这个特性， `for` 循环中的 `goroutine` 不能直接将 `a` 或 `b` 的地址直接传给协程，而是通过值拷贝生成一个临时变量。



### 介绍一下 `rune`

```go
type rune = int32
```

是 Go 特殊字符集的处理方案（Unicode，UTF-8），`rune` 等同于 `int32` ，常用来处理**Unicode** 或 **UTF-8** 字符。



### 结构体 `struct` 的值方法和指针方法的区别？

```go
type test struct {
	a, b int
	c    *int
}
```

**值方法（`value`）**

```go
func (t test) Update(a, b, c int) {
	t.a = a
	t.b = b
	*(t.c) = c
}
```

**值方法**将对结构体的一个副本进行操作，他不会改变原有的结构体（除非结构体中有引用变量，这样可以通过修改指向内存的元素来修改内容）。

```go
c := 3
t1 := test{ a: 1, b: 2, c: &c }
t1.Update(2, 3, 4)
```

看似修改了 `t1` ，但是我们输出时发现，结果为：`{ a: 1, b: 2, c: 4 }`。

为了修改该结构体（内存的复用，不必要的复制），我们有**指针方法**：

**指针方法（`pointer`）**

```go
func (t *test) Update(a, b, c int) {
	t.a = a
	t.b = b
	*(t.c) = c
}
```

```go
c := 3
t1 := test{ a: 1, b: 2, c: &c }
t1.Update(2, 3, 4)
```

结果为：`{ a: 2, b: 3, c: 4 }`。



### 单引号、双引号、反引号的区别？

**单引号**，表示 `byte` 类型或 `rune` 类型，对应 `uint8` 和 `int32` 类型，默认是 `rune` 类型。 `byte` 用来强调数据是**raw data**，而不是数字；而 `rune` 用来表示 **Unicode** 的code point。

```go
a := 'a'
// var a int32 = 'a'
```

**双引号**，才是字符串，实际上是字符数组。可以用索引号访问某字节，也可以用 `len()` 函数来获取字符串所占的字节长度。

```go
b := "我"
// var b string = "我"
```

**反引号**，表示字符串字面量，但**不支持任何转义序列**。字面量 raw literal string 的意思是，你定义时写的啥样，它就啥样，你有换行，它就换行。你写转义字符，它也就展示转义字符。

```go
c := `我 wywnb \n`
// var c string = `我 wywnb \n`
// var c string = "我 wywnb \\n"
```



