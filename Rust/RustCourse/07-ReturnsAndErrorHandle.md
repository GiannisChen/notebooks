## 返回值和错误处理

- Go 语言为人诟病的其中一点就是 ***if err != nil {}\*** 的大量使用，缺乏一些程序设计的美感，不过我倒是觉得这种简单的方式也有其好处，就是阅读代码时的流畅感很强，你不需要过多的思考各种语法是什么意思。与 Go 语言不同，**Rust** 博采众家之长，实现了颇具自身色彩的返回值和错误处理体系，本章我们就高屋建瓴地来学习，更加深入的讲解见[错误处理](https://course.rs/advance/errors.html)。

##### Rust 的错误哲学

- 错误对于软件来说是不可避免的，因此一门优秀的编程语言必须有其完整的错误处理哲学。在很多情况下，**Rust** 需要你承认自己的代码可能会出错，并提前采取行动，来处理这些错误。**Rust** 中的错误主要分为两类：
  1. **可恢复错误**，通常用于从系统全局角度来看可以接受的错误，例如处理用户的访问、操作等错误，这些错误只会影响某个用户自身的操作进程，而不会对系统的全局稳定性产生影响；
  2. **不可恢复错误**，刚好相反，该错误通常是全局性或者系统性的错误，例如数组越界访问，系统启动时发生了影响启动流程的错误等等，这些错误的影响往往对于系统来说是致命的；

- 很多编程语言，并不会区分这些错误，而是直接采用异常的方式去处理。**Rust** 没有异常，但是 **Rust** 也有自己的卧龙凤雏：`Result<T, E>` 用于可恢复错误，`panic!` 用于不可恢复错误。



### panic 深入剖析

- 在正式开始之前，先来思考一个问题：假设我们想要从文件读取数据，如果失败，你有没有好的办法通知调用者为何失败？如果成功，你有没有好的办法把读取的结果返还给调用者？



#### panic! 与不可恢复错误

上面的问题在真实场景会经常遇到，其实处理起来挺复杂的，让我们先做一个假设：文件读取操作发生在系统启动阶段。那么可以轻易得出一个结论，一旦文件读取失败，那么系统启动也将失败，这意味着该失败是不可恢复的错误，无论是因为文件不存在还是操作系统硬盘的问题，这些只是错误的原因不同，但是归根到底都是不可恢复的错误(梳理清楚当前场景的错误类型非常重要)。

既然是不可恢复错误，那么一旦发生，只需让程序崩溃即可。对此，Rust 为我们提供了 `panic!` 宏，当调用执行该宏时，**程序会打印出一个错误信息，展开报错点往前的函数调用堆栈，最后退出程序**。

> 切记，一定是不可恢复的错误，才调用 `panic!` 处理，你总不想系统仅仅因为用户随便传入一个非法参数就崩溃吧？所以，**只有当你不知道该如何处理时，再去调用 panic!**.



#### 调用 panic!

首先，来调用一下 `panic!`，这里使用了最简单的代码实现，实际上你在程序的任何地方都可以这样调用：

```rust
fn main() {
    panic!("crash and burn");
}
```

运行后输出：

```shell
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

以上信息包含了两条重要信息：

1. `main` 函数所在的线程崩溃了，发生的代码位置是 `src/main.rs` 中的第 2 行第 5 个字符（去除该行前面的空字符）；
2. 在使用时加上一个环境变量可以获取更详细的栈展开信息：
   - Linux/macOS 等 UNIX 系统： `RUST_BACKTRACE=1 cargo run` ；
   - Windows 系统（PowerShell）： `$env:RUST_BACKTRACE=1 ; cargo run` ；

下面让我们针对第二点进行详细展开讲解。



#### backtrace 栈展开

在真实场景中，错误往往涉及到很长的调用链甚至会深入第三方库，如果没有栈展开技术，错误将难以跟踪处理，下面我们来看一个真实的崩溃例子：

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

上面的代码很简单，数组只有 `3` 个元素，我们却尝试去访问它的第 `100` 号元素（数组索引从 `0` 开始），那自然会崩溃。

如果有过 C 语言的经验，即使你越界了，问题不大，我依然尝试去访问，至于这个值是不是你想要的（`100` 号内存地址也有可能有值，只不过是其它变量或者程序的！），抱歉，不归我管，我只负责取，你要负责管理好自己的索引访问范围。上面这种情况被称为**缓冲区溢出**，并可能会导致安全漏洞，例如攻击者可以通过索引来访问到数组后面不被允许的数据。

说实话，我宁愿程序崩溃，为什么？当你取到了一个不属于你的值，这在很多时候会导致程序上的逻辑 BUG！ 有编程经验的人都知道这种逻辑上的 BUG 是多么难被发现和修复！因此程序直接崩溃，然后告诉我们问题发生的位置，最后我们对此进行修复，这才是最合理的软件开发流程，而不是把问题藏着掖着：

```shell
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

好的，现在成功知道问题发生的位置，但是如果我们想知道该问题之前经过了哪些调用环节，该怎么办？那就按照提示使用 `RUST_BACKTRACE=1 cargo run` 或 `$env:RUST_BACKTRACE=1 ; cargo run` 来再一次运行程序：

```shell
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/std/src/panicking.rs:517:5
   1: core::panicking::panic_fmt
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/panicking.rs:101:14
   2: core::panicking::panic_bounds_check
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/panicking.rs:77:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/slice/index.rs:184:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/slice/index.rs:15:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/alloc/src/vec/mod.rs:2465:9
   6: world_hello::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/ops/function.rs:227:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

上面的代码就是一次**栈展开**（也称**栈回溯**），它包含了函数调用的顺序，当然按照逆序排列：最近调用的函数排在列表的最上方。因为咱们的 `main` 函数基本是最先调用的函数了，所以排在了倒数第二位，还有一个关注点，排在最顶部最后一个调用的函数是 `rust_begin_unwind`，该函数的目的就是进行栈展开，呈现这些列表信息给我们。

要获取到栈回溯信息，你还需要开启 `debug` 标志，该标志在使用 `cargo run` 或者 `cargo build` 时自动开启（这两个操作默认是 `Debug` 运行方式）。同时，栈展开信息在不同操作系统或者 **Rust** 版本上也有所不同。



#### panic 时得两终止方式

当出现 `panic!` 时，程序提供了两种方式来处理终止流程：**栈展开**和**直接终止**：

1. **栈展开**是**默认方式**，这意味着 Rust 会回溯栈上数据和函数调用，因此也意味着更多的善后工作，好处是可以给出充分的报错信息和栈调用信息，便于事后的问题复盘。
2. **直接终止**，不清理数据就直接退出程序，善后工作交与操作系统来负责。

对于绝大多数用户，使用默认选择是最好的，但是当你关心最终编译出的二进制可执行文件大小时，那么可以尝试去使用直接终止的方式，例如下面的配置修改 `Cargo.toml` 文件，实现在 [release](https://course.rs/first-try/cargo.html#手动编译和运行项目) 模式下遇到 `panic` 直接终止：

```toml
[profile.release]
panic = 'abort'
```



#### 线程 `panic` 后，程序是否会终止？

长话短说，如果是 `main` 线程，则程序会终止，如果是其它子线程，该线程会终止，但是不会影响 `main` 线程。因此，尽量不要在 `main` 线程中做太多任务，将这些任务交由子线程去做，就算子线程 `panic` 也不会导致整个程序的结束。具体解析见 [panic 原理剖析](https://course.rs/basic/result-error/panic.html#panic-原理剖析)。



#### 何时该使用 panic!

下面让我们大概罗列下何时适合使用 `panic`，也许经过之前的学习，你已经能够对 `panic` 的使用有了自己的看法，但是我们还是会罗列一些常见的用法来加深你的理解。

- 先来一点背景知识，在前面章节我们粗略讲过 `Result<T, E>` 这个枚举类型，它是用来表示函数的返回结果：

  ```rust
  enum Result<T, E> {
      Ok(T),
      Err(E),
  }
  ```

  1. 当没有错误发生时，函数返回一个用 `Result` 类型包裹的值 `Ok(T)`；
  2. 当错误时，返回一个 `Err(E)`。

  对于 `Result` 返回我们有很多处理方法，最简单粗暴的就是 `unwrap` 和 `expect`，这两个函数非常类似，我们以 `unwrap` 举例：

  ```rust
  use std::net::IpAddr;
  let home: IpAddr = "127.0.0.1".parse().unwrap();
  ```

  上面的 `parse` 方法试图将字符串 `"127.0.0.1" `解析为一个 IP 地址类型 `IpAddr`，它返回一个 `Result<IpAddr, E>` 类型，如果解析成功，则把 `Ok(IpAddr)` 中的值赋给 `home`，如果失败，则不处理 `Err(E)`，而是直接 `panic`。

  因此 `unwrap` 简而言之：成功则返回值，失败则 `panic`，总之不进行任何错误处理。

##### 示例、原型、测试

这几个场景下，需要快速地搭建代码，错误处理会拖慢编码的速度，也不是特别有必要，因此通过 `unwrap`、`expect` 等方法来处理是最快的。

同时，当我们回头准备做错误处理时，可以全局搜索这些方法，不遗漏地进行替换。

##### 你确切的知道你的程序是正确时，可以使用 panic

因为 `panic` 的触发方式比错误处理要简单，因此可以让代码更清晰，可读性也更加好，当我们的代码注定是正确时，你可以用 `unwrap` 等方法直接进行处理，反正也不可能 `panic` ：

```rust
use std::net::IpAddr;
let home: IpAddr = "127.0.0.1".parse().unwrap();
```

例如上面的例子，`"127.0.0.1"` 就是 `ip` 地址，因此我们知道 `parse` 方法一定会成功，那么就可以直接用 `unwrap` 方法进行处理。

当然，如果该字符串是来自于用户输入，那在实际项目中，就必须用错误处理的方式，而不是 `unwrap`，否则你的程序一天要崩溃几十万次吧！

##### 可能倒置全局有害状态时

**有害状态**大概分为几类：

1. 非预期的错误；
2. 后续代码的运行会受到显著影响；
3. 内存安全的问题；

当错误预期会出现时，返回一个错误较为合适，例如解析器接收到格式错误的数据，HTTP 请求接收到错误的参数甚至该请求内的任何错误（不会导致整个程序有问题，只影响该次请求）。**因为错误是可预期的，因此也是可以处理的**。

当启动时某个流程发生了错误，对后续代码的运行造成了影响，那么就应该使用 `panic`，而不是处理错误后继续运行，当然你可以通过重试的方式来继续。

上面提到过，数组访问越界，就要 `panic` 的原因，这个就是属于内存安全的范畴，一旦内存访问不安全，那么我们就无法保证自己的程序会怎么运行下去，也无法保证逻辑和数据的正确性。



#### panic 原理剖析

当调用 `panic!` 宏时，它会：

1. 格式化 `panic` 信息，然后使用该信息作为参数，调用 `std::panic::panic_any()` 函数；
2. `panic_any` 会检查应用是否使用了 [panic hook](https://doc.rust-lang.org/std/panic/fn.set_hook.html)，如果使用了，该 `hook` 函数就会被调用（`hook` 是一个钩子函数，是外部代码设置的，用于在 `panic` 触发时，执行外部代码所需的功能）；
3. 当 `hook` 函数返回后，当前的线程就开始进行栈展开：从 `panic_any` 开始，如果寄存器或者栈因为某些原因信息错乱了，那很可能该展开会发生异常，最终线程会直接停止，展开也无法继续进行；
4. 展开的过程是一帧一帧的去回溯整个栈，每个帧的数据都会随之被丢弃，但是在展开过程中，你可能会遇到被用户标记为 `catching` 的帧（通过 `std::panic::catch_unwind()` 函数标记），此时用户提供的 `catch` 函数会被调用，展开也随之停止：当然，如果 `catch` 选择在内部调用 `std::panic::resume_unwind()` 函数，则展开还会继续；

还有一种情况，在展开过程中，如果展开本身 `panic` 了，那展开线程会终止，展开也随之停止。

一旦线程展开被终止或者完成，最终的输出结果是取决于哪个线程 `panic`：对于 `main` 线程，操作系统提供的终止功能 `core::intrinsics::abort()` 会被调用，最终结束当前的 `panic` 进程；如果是其它子线程，那么子线程就会简单的终止，同时信息会在稍后通过 `std::thread::join()` 进行收集。



### 可恢复得错误 Result

还记得上一节中，提到的关于文件读取的思考题吧？当时我们解决了读取文件时遇到不可恢复错误该怎么处理的问题，现在来看看，读取过程中，正常返回和遇到可以恢复的错误时该如何处理。

假设，我们有一台消息服务器，每个用户都通过 websocket 连接到该服务器来接收和发送消息，该过程就涉及到 socket 文件的读写，那么此时，如果一个用户的读写发生了错误，显然不能直接 `panic`，否则服务器会直接崩溃，所有用户都会断开连接，因此我们需要一种更温和的错误处理方式：`Result<T, E>`。

之前章节有提到过，`Result<T, E>` 是一个枚举类型，定义如下：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

泛型参数 `T` 代表成功时存入的正确值的类型，存放方式是 `Ok(T)`，`E` 代表错误是存入的错误值，存放方式是 `Err(E)`，枯燥的讲解永远不及代码生动准确，因此先来看下打开文件的例子：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

以上 `File::open` 返回一个 `Result` 类型，那么问题来了：如何获得变量类型或者函数得返回类型？

- 有几种常用的方式，此处更推荐**第二种方法**：

  1. 第一种是查询标准库或者三方库文档，搜索 `File`，然后找到它的 `open` 方法；

  2. 在 [Rust IDE](https://course.rs/first-try/editor.html) 章节，我们推荐了 `VSCode` IDE 和 `rust-analyzer` 插件，如果你成功安装的话，那么就可以在 `VSCode` 中很方便的通过代码跳转的方式查看代码，同时 `rust-analyzer` 插件还会对代码中的类型进行标注，非常方便好用！

  3. 你还可以尝试故意标记一个错误的类型，然后让编译器告诉你：

     ```rust
     let f: u32 = File::open("hello.txt");
     ```

     错误提示如下：

     ```shell
     error[E0308]: mismatched types
      --> src/main.rs:4:18
       |
     4 |     let f: u32 = File::open("hello.txt");
       |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
     `std::result::Result`
       |
       = note: expected type `u32`
                  found type `std::result::Result<std::fs::File, std::io::Error>`
     ```

     上面代码，故意将 `f` 类型标记成整形，编译器立刻不乐意了，你是在忽悠我吗？打开文件操作返回一个整形？来，大哥来告诉你返回什么：`std::result::Result<std::fs::File, std::io::Error>`。

别慌，其实很简单：

首先 `Result` 本身是定义在 `std::result` 中的，但是因为 `Result` 很常用，所以就被包含在了 [`prelude`](https://course.rs/appendix/prelude.html) 中（将常用的东东提前引入到当前作用域内），因此无需手动引入 `std::result::Result`，那么返回类型可以简化为 `Result<std::fs::File,std::io::Error>`，你看看是不是很像标准的 `Result<T, E>` 枚举定义？只不过 `T` 被替换成了具体的类型 `std::fs::File`，是一个文件句柄类型，`E` 被替换成 `std::io::Error`，是一个 IO 错误类型.

这个返回值类型说明 `File::open` 调用如果成功则返回一个可以进行读写的文件句柄，如果失败，则返回一个 IO 错误：文件不存在或者没有访问文件的权限等。总之 `File::open` 需要一个方式告知调用者是成功还是失败，并同时返回具体的文件句柄(成功)或错误信息(失败)，万幸的是，这些信息可以通过 `Result` 枚举提供：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

代码很清晰，对打开文件后的 `Result<T, E>` 类型进行匹配取值，如果是成功，则将 `Ok(file)` 中存放的的文件句柄 `file` 赋值给 `f`，如果失败，则将 `Err(error)` 中存放的错误信息 `error` 使用 `panic` 抛出来，进而结束程序，这非常符合上文提到过的 `panic` 使用场景。



#### 对返回得错误进行处理

直接 `panic` 还是过于粗暴，因为实际上 IO 的错误有很多种，我们需要对部分错误进行特殊处理，而不是所有错误都直接崩溃：

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

上面代码在匹配出 `error` 后，又对 `error` 进行了详细的匹配解析，最终结果：

- 如果是文件不存在错误 `ErrorKind::NotFound`，就创建文件，这里创建文件`File::create` 也是返回 `Result`，因此继续用 `match` 对其结果进行处理：创建成功，将新的文件句柄赋值给 `f`，如果失败，则 `panic`；
- 剩下的错误，一律 `panic`；

虽然很清晰，但是代码还是有些啰嗦，我们会在[简化错误处理](https://course.rs/advance/errors.html)一章重点讲述如何写出更优雅的错误。



#### unwrap 和 expect : 失败就 panic

在不需要处理错误的场景，例如写原型、示例时，我们不想使用 `match` 去匹配 `Result<T, E> `以获取其中的 `T` 值，因为 `match` 的穷尽匹配特性，你总要去处理下 `Err` 分支。那么有没有办法简化这个过程？有，答案就是 `unwrap` 和 `expect`。

它们的作用就是，如果返回成功，就将 `Ok(T)` 中的值取出来，如果失败，就直接 `panic`，真的勇士绝不多 BB，直接崩溃。

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

如果调用这段代码时 *hello.txt* 文件不存在，那么 `unwrap` 就将直接 `panic`：

```shell
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }', src/main.rs:4:37
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

`expect` 跟 `unwrap` 很像，也是遇到错误直接 `panic`, 但是会带上自定义的错误提示信息，**相当于重载了错误打印的函数**：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

报错如下：

```shell
thread 'main' panicked at 'Failed to open hello.txt: Os { code: 2, kind: NotFound, message: "No such file or directory" }', src/main.rs:4:37
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

可以看出，`expect` 相比 `unwrap` 能提供更精确的错误信息，在有些场景也会更加实用。



#### 传播错误

咱们的程序几乎不太可能只有 `A->B` 形式的函数调用，一个设计良好的程序，一个功能涉及十几层的函数调用都有可能。而错误处理也往往不是哪里调用出错，就在哪里处理，实际应用中，**大概率会把错误层层上传然后交给调用链的上游函数进行处理**，**错误传播将极为常见**。

例如以下函数从文件中读取用户名，然后将结果进行返回：

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file(file_name: &str) -> Result<String, io::Error> {
    // 打开文件，f是`Result<文件句柄,io::Error>`
    let f = File::open(file_name);

    let mut f = match f {
        // 打开文件成功，将file句柄赋值给f
        Ok(file) => file,
        // 打开文件失败，将错误返回(向上传播)
        Err(e) => return Err(e),
    };
    // 创建动态字符串s
    let mut s = String::new();
    // 从f文件句柄读取数据并写入s中
    match f.read_to_string(&mut s) {
        // 读取成功，返回Ok封装的字符串
        Ok(_) => Ok(s),
        // 将错误向上传播
        Err(e) => Err(e),
    }
}

fn main() {
    match read_username_from_file("hello.txt") {
        Ok(s) => println!("read success, user_name: {}, closed...", s),
        Err(e) => println!("read file failed, Err: {}", e),
    }
}
```

有几点值得注意：

1. 该函数返回一个 `Result<String, io::Error>` 类型，当读取用户名成功时，返回 `Ok(String)`，失败时，返回 `Err(io:Error)`；
2. `File::open` 和 `f.read_to_string` 返回的 `Result<T, E>` 中的 `E` 就是 `io::Error`；

由此可见，该函数将 `io::Error` 的错误往上进行传播，该函数的调用者最终会对 `Result<String,io::Error>` 进行再处理，至于怎么处理就是调用者的事，如果是错误，它可以选择继续向上传播错误，也可以直接 `panic`，亦或将具体的错误原因包装后写入 socket 中呈现给终端用户。

但是上面的代码也有自己的问题，那就是太长了（优秀的程序员身上的优点极多，其中最大的优点就是 **懒** ），因此见不得这么啰嗦的代码，下面咱们来讲讲如何简化它。



#### ? 在传播中的作用

先看一段有关 `?` 的代码：

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

看到没，这就是排面，相比前面的 `match` 处理错误的函数，代码直接减少了一半不止，但是，一山更比一山难，看不懂啊！

其实 `?` 就是一个**宏**（**macros**），它的作用跟上面的 `match` 几乎一模一样：

```rust
let mut f = match f {
    // 打开文件成功，将file句柄赋值给f
    Ok(file) => file,
    // 打开文件失败，将错误返回(向上传播)
    Err(e) => return Err(e),
};
```

如果结果是 `Ok(T)`，则把 `T` 赋值给 `f`，如果结果是 `Err(E)`，则返回该错误，所以 `?` 特别适合用来传播错误。虽然 `?` 和 `match` 功能一致，但是事实上 `?` 会更胜一筹。何解？

想象一下，一个设计良好的系统中，肯定有自定义的错误特征，错误之间很可能会存在上下级关系，例如标准库中的 `std::io::Error `和 `std::error::Error`，前者是 IO 相关的错误结构体，后者是一个最最通用的标准错误特征，同时前者实现了后者，因此 `std::io::Error` 可以转换为 `std:error::Error`。

明白了以上的错误转换，`?` 的更胜一筹就很好理解了，它可以自动进行类型提升（转换）：

```rust
fn open_file() -> Result<File, Box<dyn std::error::Error>> {
    let mut f = File::open("hello.txt")?;
    Ok(f)
}
```

上面代码中 `File::open` 报错时返回的错误是 `std::io::Error` 类型，但是 `open_file` 函数返回的错误类型是 `std::error::Error` 的特征对象，可以看到一个错误类型通过 `?` 返回后，变成了另一个错误类型，这就是 `?` 的神奇之处。

根本原因是在于标准库中定义的 `From` 特征，该特征有一个方法 `from`，用于把一个类型转成另外一个类型，`?` 可以自动调用该方法，然后进行隐式类型转换。因此只要函数返回的错误 `ReturnError` 实现了 `From<OtherError>` 特征，那么 `?` 就会自动把 `OtherError` 转换为 `ReturnError`。

这种转换非常好用，意味着你可以用一个大而全的 `ReturnError` 来覆盖所有错误类型，只需要为各种子错误类型实现这种转换即可。

强中自有强中手，一码更比一码短：

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

瞧见没？ `?` 还能实现链式调用，`File::open` 遇到错误就返回，没有错误就将 `Ok` 中的值取出来用于下一个方法调用，简直太精妙了，从 Go 语言过来的我，内心狂喜（其实学 **Rust** 的苦和痛我才不会告诉你们）。

> 更短的：
>
> ```rust
> use std::fs;
> use std::io;
> 
> fn read_username_from_file() -> Result<String, io::Error> {
>     // read_to_string是定义在std::io中的方法，因此需要在上面进行引用
>     fs::read_to_string("hello.txt")
> }
> ```
>
> 从文件读取数据到字符串中，是比较常见的操作，因此 **Rust** 标准库为我们提供了 `fs::read_to_string` 函数，该函数内部会打开一个文件、创建 `String`、读取文件内容最后写入字符串并返回，**因为该函数其实与本章讲的内容关系不大**。

##### ? 用于 Option 的返回

`?` 不仅仅可以用于 `Result` 的传播，还能用于 `Option` 的传播，再来回忆下 `Option` 的定义：

```rust
pub enum Option<T> {
    Some(T),
    None
}
```

`Result` 通过 `?` 返回错误，那么 `Option` 就通过 `?` 返回 `None`：

```rust
fn first(arr: &[i32]) -> Option<&i32> {
   let v = arr.get(0)?;
   Some(v)
}
```

上面的函数中，`arr.get` 返回一个 `Option<&i32>` 类型，因为 `?` 的使用，如果 `get` 的结果是 `None`，则直接返回 `None`，如果是 `Some(&i32)`，则把里面的值赋给 `v`。

其实这个函数有些画蛇添足，我们完全可以写出更简单的版本：

```rust
fn first(arr: &[i32]) -> Option<&i32> {
   arr.get(0)
}
```

有一句话怎么说？没有需求，制造需求也要上……大家别跟我学习，这是软件开发大忌。只能用代码洗洗眼了：

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

上面代码展示了在链式调用中使用 `?` 提前返回 `None` 的用法， `.next` 方法返回的是 `Option` 类型：如果返回 `Some(&str)`，那么继续调用 `chars` 方法,如果返回 `None`，则直接从整个函数中返回 `None`，不再继续进行链式调用。

> **新手用 ? 常会犯的错误**
>
> 初学者在用 `?` 时，老是会犯错，例如写出这样的代码：
>
> ```rust
> fn first(arr: &[i32]) -> Option<&i32> {
>    arr.get(0)?
> }
> ```
>
> 这段代码无法通过编译，切记：`?` 操作符需要一个变量来承载正确的值，这个函数只会返回 `Some(&i32)` 或者 `None`，只有错误值能直接返回，正确的值不行，所以如果数组中存在 0 号元素，那么函数第二行使用 `?` 后的返回类型为 `&i32` 而不是 `Some(&i32)`。因此 `?` 只能用于以下形式：
>
> 1. `let v = xxx()?;`
> 2. `xxx()?.yyy()?;`

##### 带返回值的 main 函数

在了解了 `?` 的使用限制后，这段代码你很容易看出它无法编译：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

因为 `?` 要求 `Result<T, E>` 形式的返回值，而 `main` 函数的返回是 `()`，因此无法满足，那是不是就无解了呢？

**实际上 Rust 还支持另外一种形式的 `main` 函数**：

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())
}
```

这样就能使用 `?` 提前返回了，同时我们又一次看到了`Box<dyn Error>` 特征对象，因为 `std::error:Error` 是 Rust 中抽象层次最高的错误，其它标准库中的错误都实现了该特征，因此我们可以用该特征对象代表一切错误，就算 `main` 函数中调用任何标准库函数发生错误，都可以通过 `Box<dyn Error>` 这个特征对象进行返回.

至于 `main` 函数可以有多种返回值，那是因为实现了 [std::process::Termination](https://doc.rust-lang.org/std/process/trait.Termination.html) 特征，目前为止该特征还没进入稳定版 Rust 中，也许未来你可以为自己的类型实现该特征！

##### try!

在 `?` 横空出世之前（**Rust 1.13**），**Rust** 开发者还可以使用 `try!` 来处理错误，该宏的大致定义如下：

```rust
macro_rules! try {
    ($e:expr) => (match $e {
        Ok(val) => val,
        Err(err) => return Err(::std::convert::From::from(err)),
    });
}
```

简单看一下与 `?` 的对比:

```rust
//  `?`
let x = function_with_error()?; // 若返回 Err, 则立刻返回；若返回 Ok(255)，则将 x 的值设置为 255

// `try!()`
let x = try!(function_with_error());
```

可以看出 `?` 的优势非常明显，何况 `?` 还能做链式调用。

> 总之，`try!` 作为前浪已经死在了沙滩上，**在当前版本中，我们要尽量避免使用 try!**。

