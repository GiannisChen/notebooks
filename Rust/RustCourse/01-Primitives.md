## 基础（Primitives）

- **注意**，**Rust** 提倡给变量和函数用下划线命名法（**under_score_case**），而**驼峰命名法** 留给了结构体等的类oo的数据结构，从根目录就开始贯彻这一原则，为了不出现讨厌的 **warning** ，我得改变习惯了。😀



### 包管理工具 cargo

- `cargo` 提供了一系列的工具，从项目的建立、构建到测试、运行直至部署，为 **Rust** 项目的管理提供尽可能完整的手段，同时，与 **Rust** 语言及其编译器 `rustc` 紧密结合。



#### 基本操作

##### 新建项目

- 该新建一个项目了：

  ```shell
  $ cargo new hello_world
  ```

- 如果在**已有文件夹**上创建项目，用：

  ```shell
  $ cargo init
  ```

- 文件架构：

  ```shell
  $ tree
  .
  ├── .git
  ├── .gitignore
  ├── Cargo.toml
  └── src
      └── main.rs
  ```

##### 运行

1. “一键”运行：

   ```shell
   $ cargo run
   ```

   ```shell
      Compiling rust_course v0.1.0 (F:\My Github Repositories\RustByExample\rust_course)
       Finished dev [unoptimized + debuginfo] target(s) in 0.51s
        Running `target\debug\rust_course.exe`
   Hello, world!
   ```

2. 手动编译：

   ```shell
   $ cargo build
   $ .\target\debug\rust_course.exe
   ```

   ```shell
   # cargo build
       Finished dev [unoptimized + debuginfo] target(s) in 0.01s
   # run
   Hello, world!
   ```

- 在调用的时候，路径 `./target/debug/rust_course.exe` 中有一个明晃晃的 `debug` 字段，没错我们运行的是 `debug` 模式，在这种模式下，**代码的编译速度会非常快**，可是福兮祸所依，**运行速度就慢了**. 原因是，在 `debug` 模式下，**Rust** 编译器不会做任何的优化，只为了尽快的编译完成，让你的开发流程更加顺畅。

- 如果要用更加高效的代码，即 `release` 模式，则要加上 `--release` 来编译：

  - `cargo run --release`
  - `cargo build --release`

- 运行高性能的 `release` 程序：

  ```shell
  $ cargo run --release
  # output:
     Compiling rust_course v0.1.0 (F:\My Github Repositories\RustByExample\rust_course)
      Finished release [optimized] target(s) in 0.41s
       Running `target\release\rust_course.exe`      
  Hello, world!
  ```

##### 检查

- 由于 **Rust** 要做编译优化和特性解析，其速度显然会很慢，为了能够高效开发，需要快速检查编译，而不是用 `cargo run` / `cargo build`，所以就有了 **检查** `cargo check`：

  ```shell
  $ cargo check
  # output:
      Finished dev [unoptimized + debuginfo] target(s) in 0.01s
  ```



#### 结构文件

- `Cargo.toml` 和 `Cargo.lock` 是 `cargo` 的核心文件，它的所有活动均基于此二者。

##### Cargo.lock

- `Cargo.lock` 文件是 `cargo` 工具根据同一项目的 `toml` 文件生成的**项目依赖详细清单**，因此我们一般不用修改它，只需要对着 `Cargo.toml` 文件撸就行了。

> **什么情况下该把 `Cargo.lock` 上传到 git 仓库里？**很简单，当你的项目是一个可运行的程序时，就上传 `Cargo.lock`，如果是一个依赖库项目，那么请把它添加到 `.gitignore` 中。

##### Cargo.toml

- `Cargo.toml` 是 `cargo` 特有的**项目数据描述文件**。它存储了项目的所有元配置信息，如果 Rust 开发者希望 Rust 项目能够按照期望的方式进行构建、测试和运行，那么，必须按照合理的方式构建 `Cargo.toml`。

- **`package`** 中记录了项目的描述信息，典型的如下：

  ```toml
  [package]
  name = "world_hello"
  version = "0.1.0"
  edition = "2021"
  ```

  - `name` 字段定义了项目名称；
  - `version` 字段定义当前版本，新项目默认是 `0.1.0`；
  - `edition` 字段定义了我们使用的 **Rust** 大版本；

- **`dependencies`** 定义了项目依赖，在 `Cargo.toml` 中，主要通过各种依赖段落来描述该项目的各种依赖项：

  - 基于 **Rust** 官方仓库 `crates.io`，通过版本说明来描述；
  - 基于项目源代码的 **git** 仓库地址，通过 **URL** 来描述；
  - 基于本地项目的绝对路径或者相对路径，通过类 **Unix** 模式的路径来描述；

  ```toml
  [dependencies]
  rand = "0.3"
  hammer = { version = "0.5.0"}
  color = { git = "https://github.com/bjz/color-rs" }
  geometry = { path = "crates/geometry" }
  ```



### 变量绑定与解构（Boundings and Destructuring）

- **Rust** 支持声明可变变量和不可变变量，兼顾了灵活性和安全性，同时带来了运行性能上的提升，因为将本身无需改变的变量声明为不可变在运行期会避免一些多余的 `runtime` 检查。不过这样会给编程者带来一些小麻烦。



#### 命名规范

- https://course.rs/practice/naming.html



#### 变量绑定

- 在其它语言中，我们用 `var a = "hello world"` 的方式给 `a` 赋值，也就是把等式右边的 `"hello world"` 字符串赋值给变量 `a` ，而在 Rust 中，我们这样写： `let a = "hello world"` ，同时给这个过程起了另一个名字：**变量绑定**。（这个名字和 **Rust** 的**所有权**概念形成了对应，强调了对内存的**分配**而不是对变量的**赋值**）



#### 变量可变性

- Rust 的变量在默认情况下是**不可变的**，可以通过 `mut` 关键字让变量变为**可变的**。

  ```rust
  fn main() {
      let x = 5;
      println!("The value of x is: {}", x);
      x = 6;
      println!("The value of x is: {}", x);
  }
  // will throw error in compile time
  ```

  具体的错误原因是 `cannot assign twice to immutable variable x`（无法对不可变的变量进行重复赋值），因为我们想为不可变的 `x` 变量再次赋值。

  这种错误是为了避免无法预期的错误发生在我们的变量上：一个变量往往被多处代码所使用，其中一部分代码假定该变量的值永远不会改变，而另外一部分代码却无情的改变了这个值，在实际开发过程中，这个错误是很难被发现的，特别是在多线程编程中。

  这种规则让我们的代码变得非常清晰，只有你想让你的变量改变时，它才能改变，这样就不会造成心智上的负担，也给别人阅读代码带来便利。

- 在 **Rust** 中，可变性很简单，只要在变量名前加一个 `mut` 即可：

  ```rust
  fn main() {
      let mut x = 5;
      println!("The value of x is: {}", x);
      x = 6;
      println!("The value of x is: {}", x);
  }
  // output:
  // The value of x is: 5
  // The value of x is: 6
  ```

> **使用下划线开头忽略未使用的变量**：如果你创建了一个变量却不在任何地方使用它，Rust 通常会给你一个警告，因为这可能会是个 BUG。但是有时创建一个不会被使用的变量是有用的，比如你正在设计原型或刚刚开始一个项目。这时**你希望告诉 Rust 不要警告未使用的变量，为此可以用下划线作为变量名的开头**：
>
> ```rust
> fn main() { let _x = 5; let y = 10; }
> // output
> /*
> 	warning: unused variable: `y`
>      --> src/main.rs:3:9
>       |
>     3 |     let y = 10;
>       |         ^ help: 如果 y 故意不被使用，请添加一个下划线前缀: `_y`
>       |
>       = note: `#[warn(unused_variables)]` on by default
> */
> ```



#### 变量解构（Destructuring）

- `let` 表达式不仅仅用于变量的绑定，还能进行复杂变量的解构，即从一个相对复杂的变量中，匹配出该变量的一部分内容：

  > 在 [Rust 1.59](https://course.rs/appendix/rust-versions/1.59.html) 版本后，我们可以在赋值语句的左式中使用**元组**、切片和**结构体**模式了。

  ```rust
  #[derive(Debug)]
  struct Enum(i32, f32);
  
  struct Struct {
      e: i32,
  }
  
  fn main() {
      {
          let x = Enum(1, 2.0);
          println!("The value of x is: {:?}", x);
          let Enum(x_1, x_2) = x;
          println!("The value of (x_1,x_2) is: ({},{})", x_1, x_2);
      }
      
      {
          let (a, b, c, d, e);
  
          (a, b) = (1, 2);
          // _ 代表匹配一个值，但是我们不关心具体的值是什么，因此没有使用一个变量名而是使用了 _
          [c, .., d, _] = [1, 2, 3, 4, 5];
          Struct { e, .. } = Struct { e: 5 };
  
          assert_eq!([1, 2, 1, 4, 5], [a, b, c, d, e]);
      }
  
      {
          let (a, mut b): (bool, bool) = (true, false);
          // a = true,不可变; b = false，可变
          println!("a = {:?}, b = {:?}", a, b);
  
          b = true;
          assert_eq!(a, b);
      }
  }
  ```

- **函数中也能使用**：

  ```rust
  fn print_coordinates(&(x, y): &(i32, i32)) {
      println!("Current location: ({}, {})", x, y);
  }
  
  fn main() {
      let point = (3, 5);
      print_coordinates(&point);
  }
  ```



#### 常量（constant）

- 变量的值不能更改可能让你想起其他另一个很多语言都有的编程概念：**常量**（**constant**）。与不可变变量一样，常量也是绑定到一个常量名且不允许更改的值，但是常量和变量之间存在一些差异：

  - 常量不允许使用 `mut`。**常量不仅仅默认不可变，而且自始至终不可变**，因为常量在编译完成后，已经确定它的值。
  - 常量使用 `const` 关键字而不是 `let` 关键字来声明，并且值的类型**必须**标注。

- 下面是一个常量声明的例子，其常量名为 `MAX_POINTS`，值设置为 `100,000`。（**Rust** 常量的命名约定是全部字母都使用大写，并使用下划线分隔单词，另外对数字字面量可插入下划线以提高可读性）：

  ```rust
  const MAX_POINTS: u32 = 100_000;
  ```

> 在实际使用中，最好将程序中用到的**硬编码值都声明为常量**，对于代码后续的维护有莫大的帮助。如果将来需要更改硬编码的值，你也只需要在代码中更改一处即可。



#### 变量遮蔽（Shadowing）

- **Rust** 允许声明相同的变量名，在后面声明的变量会遮蔽掉前面声明的，如下所示：

  ```rust
  fn main() {
      let x = 5;								// x==5
      // 在main函数的作用域内对之前的x进行遮蔽
      let x = x + 1;							// x==6
      {
          // 在当前的花括号作用域内，对之前的x进行遮蔽
          let x = x * 2;						// x==12
          println!("The value of x in the inner scope is: {}", x);
      }
      println!("The value of x is: {}", x);	// x==6
  }
  ```

- 变量遮蔽的用处在于，如果你在某个作用域内无需再使用之前的变量（在被遮蔽后，无法再访问到之前的同名变量），就可以重复的使用变量名字，而不用绞尽脑汁去想更多的名字：

  ```rust
  // 字符串类型
  let spaces = "   ";
  // usize数值类型
  let spaces = spaces.len();
  ```



### 基本类型

- **Rust** 的基本类型：
  - 数值类型：有符号整数（ `i8`， `i16` ， `i32` ， `i64` ， `isize` ）、 无符号整数（ `u8`， `u16` ， `u32` ， `u64` ， `usize` ）、浮点数（ `f32` ， `f64` ）、以及有理数、复数；
  - 字符串：字符串字面量和字符串切片 `&str` ；
  - 布尔类型： `true` 和 `false` ；
  - 字符类型：表示单个 Unicode 字符，存储为 4 个字节；
  - 单元类型：即 `()` ，其唯一的值也是 `()` ；

> `isize` 和 `usize` 类型取决于程序运行的计算机 CPU 类型： 若 CPU 是 **32** 位的，则这两个类型是 **32** 位的，同理，若 CPU 是 **64** 位，那么它们则是 **64** 位。

- **Rust 编译器可以根据变量的值和上下文中的使用方式来自动推导出变量的类型**，同时编译器也不够聪明，在某些情况下，它无法推导出变量类型，需要手动去给予一个类型标注：

  ```rust
  let guess = "42".parse().expect("Not a number!");
  ```

  那么我们就需要指定 `guess` 的类型（**显式的类型标注**）：

  ```rust
  let guess: i32 = "42".parse().expect("Not a number!");
  ```

  或者像这样（这里涉及了**泛型**）：

  ```rust
  let guess = "42".parse::<i32>().expect("Not a number!");
  ```



#### 数值类型

- 下表显示了 **Rust** 中的内置的整数类型：

  | 长度       | 有符号类型 | 无符号类型 |
  | ---------- | ---------- | ---------- |
  | 8 位       | `i8`       | `u8`       |
  | 16 位      | `i16`      | `u16`      |
  | 32 位      | `i32`      | `u32`      |
  | 64 位      | `i64`      | `u64`      |
  | 128 位     | `i128`     | `u128`     |
  | 视架构而定 | `isize`    | `usize`    |

> **类型转换必须是显式的**. **Rust** 永远也不会偷偷把你的 16bit 整数转换成 32bit 整数。

- 整形字面量可以用下表的形式书写：

  | 数字字面量         | 示例          |
  | ------------------ | ------------- |
  | 十进制             | `98_222`      |
  | 十六进制           | `0xff`        |
  | 八进制             | `0o77`        |
  | 二进制             | `0b1111_0000` |
  | 字节 (仅限于 `u8`) | `b'A'`        |

> **Rust** 整型默认使用 `i32`，例如 `let i = 1`，那 `i` 就是 `i32` 类型，因此你可以首选它，同时该类型也往往是性能最好的。`isize` 和 `usize` 的主要应用场景是用作集合的索引。

##### 整型溢出

- 假设有一个 `u8` ，它可以存放从 0 到 255 的值。那么当你将其修改为范围之外的值，比如 256，则会发生**整型溢出**。关于这一行为 **Rust** 有一些有趣的规则：当在 `debug` 模式编译时，**Rust** 会检查整型溢出，若存在这些问题，则使程序在编译时 `panic` （崩溃，**Rust** 使用这个术语来表明程序因错误而退出）。

- 在当使用 `--release` 参数进行 `release` 模式构建时，**Rust** **不**检测溢出。相反，当检测到整型溢出时，**Rust** 会按照补码循环溢出（ **two’s complement wrapping** ）的规则处理。简而言之，大于该类型最大值的数值会被补码转换成该类型能够支持的对应数字的最小值。比如在 `u8` 的情况下，256 变成 0，257 变成 1，依此类推。程序不会 `panic` ，但是该变量的值可能不是你期望的值。**依赖这种默认行为的代码都应该被认为是错误的代码**。

- 要显式处理可能的溢出，可以使用标准库针对原始数字类型提供的这些方法：

  - 使用 `wrapping_*` 方法在所有模式下都按照补码循环溢出规则处理，例如 `wrapping_add` ；

  - 如果使用 `checked_*` 方法时发生溢出，则返回 `None` 值；

  - 使用 `overflowing_*` 方法返回该值和一个指示是否存在溢出的布尔值；

  - 使用 `saturating_*` 方法使值达到最小值或最大值；

##### 浮点数陷阱

- 浮点数由于底层格式的特殊性，导致了如果在使用浮点数时不够谨慎，就可能造成危险，有两个原因：

  1. **浮点数往往是你想要数字的近似表达** 浮点数类型是基于二进制实现的，但是我们想要计算的数字往往是基于十进制，例如 `0.1` 在二进制上并不存在精确的表达形式，但是在十进制上就存在。这种不匹配性导致一定的歧义性，更多的，虽然浮点数能代表真实的数值，但是由于底层格式问题，它往往受限于定长的浮点数精度，如果你想要表达完全精准的真实数字，只有使用无限精度的浮点数才行

  2. **浮点数在某些特性上是反直觉的** 例如大家都会觉得浮点数可以进行比较，对吧？是的，它们确实可以使用 `>`，`>=` 等进行比较，但是在某些场景下，这种直觉上的比较特性反而会害了你。因为 `f32` ， `f64` 上的比较运算实现的是 `std::cmp::PartialEq` 特征（类似其他语言的接口），但是并没有实现 `std::cmp::Eq` 特征，但是后者在其它数值类型上都有定义，举个栗子：

     > **Rust** 的 `HashMap` 数据结构，是一个 KV 类型的 Hash Map 实现，它对于 `K` 没有特定类型的限制，但是要求能用作 `K` 的类型必须实现了 `std::cmp::Eq` 特征，因此这意味着你**无法使用浮点数**作为 `HashMap` 的 `Key`，来存储键值对，但是作为对比，**Rust** 的整数类型、字符串类型、布尔类型都实现了该特征，因此可以作为 `HashMap` 的 `Key`。

- 为了避免上面说的两个陷阱，你需要遵守以下准则：

  - 避免在浮点数上测试相等性；

  - 当结果在数学上可能存在未定义时，需要格外的小心；

##### NaN

- 对于数学上未定义的结果，例如对负数取平方根 `-42.1.sqrt()` ，会产生一个特殊的结果：**Rust** 的浮点数类型使用 `NaN` （not a number）来处理这些情况，**所有跟 `NaN` 交互的操作，都会返回一个 `NaN`**，而且 `NaN` 不能用来比较，下面注释掉的代码会崩溃：

  ```rust
  fn main() {
      let x = (-42.0_f32).sqrt();
      println!("{:?}", x);
      // assert_eq!(x, x);
  }
  // output
  // NaN
  ```

  出于防御性编程的考虑，可以使用 `is_nan()` 等方法，可以用来判断一个数值是否是 `NaN` ：

  ```rust
  fn main() {
      let x = (-42.0_f32).sqrt();
      if x.is_nan() {
          println!("未定义的数学行为")
      }
  }
  ```



##### 序列（Range）

- **Rust** 提供了一个非常简洁的方式，用来生成连续的数值，例如 `1..5`，生成从 1 到 4 的连续数字，不包含 5 ；`1..=5`，生成从 1 到 5 的连续数字，包含 5；
- 序列只允许用于数字或字符类型，原因是：它们可以连续，同时编译器在编译期可以检查该序列是否为空，字符和数字值是 **Rust** 中仅有的可以用于判断是否为空的类型；



##### 有理数和复数

- 标准库没有**有理数**和**复数**的表达，需要用到社区 `Rust` 数值库：[num](https://crates.io/crates/num)，下面引入 `num` 库：

  1. 在 `Cargo.toml` 中的 `[dependencies]` 下添加一行 `num = "0.4.0"` ；

  2. 将 `src/main.rs` 文件中的 `main` 函数替换为下面的代码：

     ```rust
     use num::complex::Complex;
     
     fn main() {
         let a = Complex { re: 2.1, im: -1.2 };
         let b = Complex::new(11.1, 22.2);
         let result = a + b;
     
         println!("{} + {}i", result.re, result.im);
         println!("{}", result);
     }
     
     // output
     // 13.2 + 21i
     // 13.2+21i  
     ```

  3. 运行 `cargo run` ；



#### 字符类型（char）

- **Rust** 里字符类型比较“宽泛”，不仅是 `ASCII` ，也可以是 `Unicode`，而不是引入了其他的表达方法，（没错，就是你 Go 和 C++ 🤬）。因此，`Unicode` 的范围 `U+0000 ~ U+D7FF` 和 `U+E000 ~ U+10FFFF`都是字符，包括单个的中文、日文、韩文、emoji 表情符号等等：

  ```rust
  fn main() {
      let c = 'z';
      let z = 'ℤ';
      let g = '国';
      let heart_eyed_cat = '😻';
  }
  ```

  为此 **Rust** 也付出了一定的代价，由于 `Unicode` 都是 4 个字节编码，因此字符类型也是占用 4 个字节：

  ```rust
  fn main() {
      let c = 'z';
      let z = 'ℤ';
      let g = '国';
      let heart_eyed_cat = '😻';
      println!("{} {} {} {}", c, z, g, heart_eyed_cat);
      println!(
          "{} {} {} {}",
          std::mem::size_of_val(&c),
          std::mem::size_of_val(&z),
          std::mem::size_of_val(&g),
          std::mem::size_of_val(&heart_eyed_cat)
      );
  }
  // output
  // z ℤ 国 😻
  // 4 4 4 4
  ```

  

#### 布尔类型（bool）

- Rust 中的布尔类型有两个可能的值：`true` 和 `false`，**布尔值占用内存的大小为 `1` 个字节**。



#### 单元类型

- 单元类型就是 `()` ， `main` 函数就返回这个单元类型 `()`，你不能说 `main` 函数无返回值，因为没有返回值的函数在 **Rust** 中是有单独的定义的 —— **发散函数**（ **diverge function** ），顾名思义，无法收敛的函数。例如常见的 `println!()` 的返回值也是单元类型 `()`。再比如，你可以用 `()` 作为 `map` 的值，表示我们不关注具体的值，只关注 `key`。 这种用法和 Go 语言的 `struct{}` 类似，可以作为一个值用来占位，但是完全**不占用**任何内存。



### 语句和表达式（Statement and Expression）

- **Rust** 的函数体是由一系列语句组成，最后由一个表达式来返回值，例如：

  ```rust
  fn add_with_extra(x: i32, y: i32) -> i32 {
      let x = x + 1; // 语句
      let y = y + 5; // 语句
      x + y // 表达式
  }
  ```

- 对于 **Rust** 语言而言，**这种基于语句（statement）和表达式（expression）的方式是非常重要的，你需要能明确的区分这两个概念**, 但是对于很多其它语言而言，这两个往往无需区分。基于表达式是函数式语言的重要特征，**表达式总要返回值**。

> 由于 `let` 是语句，因此不能将 `let` 语句赋值给其它值，如下形式是错误的：
>
> ```rust
> let b = (let a = 8);
> ```
>
> 错误如下:
>
> ```shell
> error: expected expression, found statement (`let`) // 期望表达式，却发现`let`语句
>  --> src/main.rs:2:13
>   |
> 2 |     let b = let a = 8;
>   |             ^^^^^^^^^
>   |
>   = note: variable declaration using `let` is a statement `let`是一条语句
> 
> error[E0658]: `let` expressions in this position are experimental
>           // 下面的 `let` 用法目前是试验性的，在稳定版中尚不能使用
>  --> src/main.rs:2:13
>   |
> 2 |     let b = let a = 8;
>   |             ^^^^^^^^^
>   |
>   = note: see issue #53667 <https://github.com/rust-lang/rust/issues/53667> for more information
>   = help: you can write `matches!(<expr>, <pattern>)` instead of `let <pattern> = <expr>`
> ```
>
> 以上的错误告诉我们 `let` 是语句，不是表达式，因此它不返回值，也就不能给其它变量赋值。但是该错误还透漏了一个重要的信息， `let` 作为表达式已经是试验功能了，也许不久的将来，我们在 [stable rust](https://course.rs/appendix/rust-version.html) 下可以这样使用。

- 由于 **Rust** 表达式**能够返回值的特点**，我们甚至可以这么用：

  ```rust
  fn main() {
      let y = {
          let x = 3;
          x + 1
      };
  
      println!("The value of y is: {}", y);
  }
  ```

  它的最后一行是表达式，返回了 `x + 1` 的值，注意 `x + 1` 不能以分号结尾，否则就会从表达式变成语句， **表达式不能包含分号**。这一点非常重要，一旦你在表达式后加上分号，它就会变成一条语句，再也**不会**返回一个值，请牢记！



### 函数（Functions）

- 函数名和变量名使用**下划线命名**（也就是：[蛇形命名法(snake case)](https://course.rs/practice/naming.html)），例如 `fn add_two() -> {}`；
- 函数的位置可以随便放，**Rust** 不关心我们在哪里定义了函数，只要有定义即可；
- 每个函数参数都需要标注类型；

##### Rust 中的特殊返回类型

- **无返回值**：对于 **Rust** 来说，其它语言中的 `return;` 或者 `void function()` ，在 Rust 的概念中返回的是 `()`：

  - 函数没有返回值，那么返回一个 `()`；
  - 通过 `;` 结尾的表达式返回一个 `()`；

  例如以下两个函数：

  ```rust
  use std::fmt::Debug;
  
  fn report<T: Debug>(item: T) { println!("{:?}", item); }
  
  fn clear(text: &mut String) -> () { *text = String::from(""); }
  ```

- **永不发散的函数**（ **diverge function** ）：真正的不返回任何值，其书写格式是： `fn func_name(...) -> ! {...}` 。特别的，这种语法往往用做**会导致程序崩溃的函数**：

  ```rust
  fn dead_end() -> ! { panic!("你已经到了穷途末路，崩溃吧！"); }
  ```

  或者，下面的函数创建了一个无限循环，该循环永不跳出，因此函数也永不返回：

  ```rust
  fn forever() -> ! {
    loop {
      //...
    };
  }
  ```



### 流程控制

#### 循环（ for / loop / while ）

##### for

- `for` 循环：

  ```rust
  for 元素 in 集合 {
    // 使用元素干一些你懂我不懂的事情
  }
  ```

- 注意，使用 `for` 时我们往往使用集合的引用形式，除非你不想在后面的代码中继续使用该集合（比如我们这里使用了 `container` 的引用）。如果不使用引用的话，所有权会被转移（move）到 `for` 语句块中，后面就无法再使用这个集合了：

  ```rust
  for item in &container {
    // ...
  }
  ```

  > 对于实现了 `copy` 特征的数组（例如 `[i32; 10]` ）而言， `for item in arr` 并不会把 `arr` 的所有权转移，而是直接对其进行了拷贝，因此循环之后仍然可以使用 `arr` 。

- 如果想在循环中，**修改该元素**，可以使用 `mut` 关键字：

  ```rust
  for item in &mut collection {
    // ...
  }
  ```

- 总结如下：

  | 使用方法                      | 等价使用方式                                      | 所有权     |
  | ----------------------------- | ------------------------------------------------- | ---------- |
  | `for item in collection`      | `for item in IntoIterator::into_iter(collection)` | 转移所有权 |
  | `for item in &collection`     | `for item in collection.iter()`                   | 不可变借用 |
  | `for item in &mut collection` | `for item in collection.iter_mut()`               | 可变借用   |

- 如果想在循环中**获取元素的索引**：

  ```rust
  fn main() {
      let a = [4, 3, 2, 1];
      // `.iter()` 方法把 `a` 数组变成一个迭代器
      for (i, v) in a.iter().enumerate() {
          println!("第{}个元素是{}", i + 1, v);
      }
  }
  ```

- 如果想用 `for` 循环控制某个过程执行 10 次，但是又不想单独声明一个变量来控制这个流程，可以用 `_` 来替代 `i` 用于 `for` 循环中，在 **Rust** 中 `_` 的含义是忽略该值或者类型的意思，如果不使用 `_`，那么编译器会给你一个 **变量未使用的** 的警告：

  ```rust
  for _ in 0..10 {
    // ...
  }
  ```

##### 两种 for 循环方式对比

- 以下代码，使用了两种循环方式：

  ```rust
  // 第一种
  let collection = [1, 2, 3, 4, 5];
  for i in 0..collection.len() {
    let item = collection[i];
    // ...
  }
  
  // 第二种
  for item in collection {
  
  }
  ```

  第一种方式是循环索引，然后通过索引下标去访问集合，第二种方式是直接循环集合中的元素，优劣如下：

  - **性能**：第一种使用方式中 `collection[index]` 的索引访问，会因为**边界检查**（**Bounds Checking**）导致运行时的性能损耗 —— **Rust** 会检查并确认 `index` 是否落在集合内，但是第二种直接迭代的方式就不会触发这种检查，因为编译器会在编译时就完成分析并证明这种访问是合法的；
  - **安全**：第一种方式里对 `collection` 的索引访问是非连续的，存在一定可能性在两次访问之间，`collection` 发生了变化，导致脏数据产生。而第二种直接迭代的方式是连续访问，因此不存在这种风险（这里是因为所有权吗？是的话可能要强调一下）；

  由于 `for` 循环无需任何条件限制，也不需要通过索引来访问，因此是最安全也是最常用的，通过与下面的 `while` 的对比，我们能看到为什么 `for` 会更加安全。

##### while 和 for

- 我们也能用 `while` 来实现 `for` 的功能：

  ```rust
  fn main() {
      let a = [10, 20, 30, 40, 50];
      let mut index = 0;
  
      while index < a.len() {
          println!("the value is: {}", a[index]);
          index = index + 1;
      }
  }
  ```

  但这个过程很容易出错；如果索引长度不正确会导致程序 ***panic\***。这也使程序更慢，因为编译器增加了运行时代码来对每次循环的每个元素进行条件检查。

- 对应的 `for`循环代码如下：

  ```rust
  fn main() {
      let a = [10, 20, 30, 40, 50];
  
      for element in a.iter() {
          println!("the value is: {}", element);
      }
  }
  ```

  可以看出，`for` 并不会使用索引去访问数组，因此更安全也更简洁，同时避免 **运行时的边界检查**，性能更高。

##### loop

- `loop` 创建一个死循环，往往搭配 `break` 来使用。区别于 `for` 和 `while` ，`loop` 可以是个 **表达式** ，于是：

  ```rust
  loop {
      println!("again!");
  }
  ```

  ```rust
  let mut c = 0;
  let a = loop {
      if c == 10 {
          break c * 2;
      }
      c += 1;
  };
  println!("{}", a);
  ```

  - **break 可以单独使用，也可以带一个返回值**，有些类似 `return` ；
  - **loop 是一个表达式**，因此可以返回一个值；



### “模式匹配”

#### match 匹配

- `match` 是 **Rust** 中的 `switch` ，同时他能做的也远比 `switch` 更多。先来看一个关于 `match` 的简单例子：

  ```rust
  enum Direction {
      East,
      West,
      North,
      South,
  }
  
  fn main() {
      let dire = Direction::South;
      match dire {
          Direction::East => println!("East"),
          Direction::North | Direction::South => {
              println!("South or North");
          },
          _ => println!("West"),
      };
  }
  ```

  这里我们想去匹配 `dire` 对应的枚举类型，因此在 `match` 中用三个匹配分支来完全覆盖枚举变量 `Direction` 的所有成员类型，有以下几点值得注意：

  - `match` 的匹配**必须要穷举出所有可能**，因此这里用 `_` 来代表未列出的所有可能性；
  - `match` 的每一个分支都**必须是一个表达式**，且所有分支的表达式最终返回值的类型必须相同；
  - **X | Y**，类似逻辑运算符 `或`，代表该分支可以匹配 `X` 也可以匹配 `Y`，只要满足一个即可；

- 其实 `match` 跟其他语言中的 `switch` 非常像，`_` 类似于 `switch` 中的 `default`。

##### 使用 match 表达式赋值

- 还有一点很重要，`match` 本身也是**一个表达式**，因此可以用它来赋值：

  ```rust
  enum IpAddr {
     Ipv4,
     Ipv6
  }
  
  fn main() {
      let ip1 = IpAddr::Ipv6;
      let ip_str = match ip1 {
          IpAddr::Ipv4 => "127.0.0.1",
          _ => "::1",
      };
  
      println!("{}", ip_str);
  }
  ```

  因为这里匹配到 `_` 分支，所以将 `"::1"` 赋值给了 `ip_str`。

##### 模式绑定

- 模式匹配的另外一个重要功能是从模式中取出绑定的值，例如：

  ```rust
  #[derive(Debug)]
  enum UsState {
      Alabama,
      Alaska,
      // --snip--
  }
  
  enum Coin {
      Penny,
      Nickel,
      Dime,
      Quarter(UsState), // 25美分硬币
  }
  ```

  其中 `Coin::Quarter` 成员还存放了一个值：美国的某个州（因为在 1999 年到 2008 年间，美国在 25 美分(Quarter)硬币的背后为 50 个州印刷了不同的标记，其它硬币都没有这样的设计）。

  接下来，我们希望在模式匹配中，获取到 25 美分硬币上刻印的州的名称：

  ```rust
  fn value_in_cents(coin: Coin) -> u8 {
      match coin {
          Coin::Penny => 1,
          Coin::Nickel => 5,
          Coin::Dime => 10,
          Coin::Quarter(state) => {
              println!("State quarter from {:?}!", state);
              25
          },
      }
  }
  ```

  上面代码中，在匹配 `Coin::Quarter(state)` 模式时，我们把它内部存储的值绑定到了 `state` 变量上，因此 `state` 变量就是对应的 `UsState` 枚举类型。**例如**有一个印了阿拉斯加州标记的 25 分硬币：`Coin::Quarter(UsState::Alaska)`, 它在匹配时，`state` 变量将被绑定 `UsState::Alaska` 的枚举值。

- 再来看一个更复杂的例子：

  ```rust
  enum Action {
      Say(String),
      MoveTo(i32, i32),
      ChangeColorRGB(u16, u16, u16),
  }
  
  fn main() {
      let actions = [
          Action::Say("Hello Rust".to_string()),
          Action::MoveTo(1,2),
          Action::ChangeColorRGB(255,255,0),
      ];
      for action in actions {
          match action {
              Action::Say(s) => {
                  println!("{}", s);
              },
              Action::MoveTo(x, y) => {
                  println!("point from (0, 0) move to ({}, {})", x, y);
              },
              Action::ChangeColorRGB(r, g, _) => {
                  println!("change color into '(r:{}, g:{}, b:0)', 'b' has been ignored",
                      r, g,
                  );
              }
          }
      }
  }
  ```

  ```shell
  $ cargo run
     Compiling world_hello v0.1.0 (/Users/sunfei/development/rust/world_hello)
      Finished dev [unoptimized + debuginfo] target(s) in 0.16s
       Running `target/debug/world_hello`
  Hello Rust
  point from (0, 0) move to (1, 2)
  change color into '(r:255, g:255, b:0)', 'b' has been ignored
  ```

##### 穷尽匹配

- `match` 的匹配必须穷尽所有情况，下面来举例说明，例如：

  ```rust
  enum Direction {
      East,
      West,
      North,
      South,
  }
  
  fn main() {
      let dire = Direction::South;
      match dire {
          Direction::East => println!("East"),
          Direction::North | Direction::South => {
              println!("South or North");
          },
      };
  }
  ```

  我们没有处理 `Direction::West` 的情况，因此会报错：

  ```console
  error[E0004]: non-exhaustive patterns: `West` not covered // 非穷尽匹配，`West` 没有被覆盖
    --> src/main.rs:10:11
     |
  1  | / enum Direction {
  2  | |     East,
  3  | |     West,
     | |     ---- not covered
  4  | |     North,
  5  | |     South,
  6  | | }
     | |_- `Direction` defined here
  ...
  10 |       match dire {
     |             ^^^^ pattern `West` not covered // 模式 `West` 没有被覆盖
     |
     = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
     = note: the matched value is of type `Direction`
  ```

##### _ 通配符

- 当我们不想在匹配时列出所有值的时候，可以使用 Rust 提供的一个特殊**模式**，例如，`u8` 可以拥有 0 到 255 的有效的值，但是我们只关心 `1、3、5 和 7` 这几个值，不想列出其它的 `0、2、4、6、8、9 一直到 255` 的值。那么, 我们不必一个一个列出所有值, 因为可以使用特殊的模式 `_` 替代：

  ```rust
  let some_u8_value = 0u8;
  match some_u8_value {
      1 => println!("one"),
      3 => println!("three"),
      5 => println!("five"),
      7 => println!("seven"),
      _ => (),
  }
  ```

  通过将 `_` 其放置于其他分支后，`_` 将会匹配所有遗漏的值。`()` 表示返回**单元类型**与所有分支返回值的类型相同，所以当匹配到 `_` 后，什么也不会发生。



#### if let 匹配

- 然而，在某些场景下，我们其实只关心**某一个值是否存在**，此时 `match` 就显得过于啰嗦。

  有时会遇到只有一个模式的值需要被处理，其它值直接忽略的场景，如果用 `match` 来处理就要写成下面这样：

  ```rust
  let v = Some(3u8);
  match v {
      Some(3) => println!("three"),
      _ => (),
  }
  ```

  我们只想要对 `Some(3)` 模式进行匹配, 不想处理任何其他 `Some<u8>` 值或 `None` 值。但是为了满足 `match` 表达式（穷尽性）的要求，写代码时必须在处理完这唯一的成员后加上 `_ => ()`，这样会增加不少无用的代码。我们完全可以用 `if let` 的方式来实现：

  ```rust
  if let Some(3) = v {
      println!("three");
  }
  ```

> 这两种匹配对于新手来说，可能有些难以抉择，但是只要记住一点就好：**当你只要匹配一个条件，且忽略其他条件时就用 `if let` ，否则都用 `match`**。



#### while let 条件循环

- 一个与 `if let` 类似的结构是 `while let` 条件循环，它允许只要模式匹配就一直进行 `while` 循环。下面展示了一个使用 `while let` 的例子：

  ```rust
  // Vec是动态数组
  let mut stack = Vec::new();
  
  // 向数组尾部插入元素
  stack.push(1);
  stack.push(2);
  stack.push(3);
  
  // stack.pop从数组尾部弹出元素
  while let Some(top) = stack.pop() {
      println!("{}", top);
  }
  ```

  这个例子会打印出 `3`、`2` 接着是 `1`。`pop` 方法取出动态数组的最后一个元素并返回 `Some(value)`，如果动态数组是空的，将返回 `None`，对于 `while` 来说，只要 `pop` 返回 `Some` 就会一直不停的循环。一旦其返回 `None`，`while` 循环停止。我们可以使用 `while let` 来弹出栈中的每一个元素。

- 你也可以用 `loop` + `if let` 或者 `match` 来实现这个功能，但是会更加啰嗦。



#### matches!宏

- Rust 标准库中提供了一个非常实用的宏：`matches!`，它可以将一个表达式跟模式进行匹配，然后返回匹配的结果 `true` or `false`。

  例如，有一个动态数组，里面存有以下枚举：

  ```rust
  enum Enum {
      Foo,
      Bar,
  }
  
  fn main() {
      let v = &vec![Enum::Foo, Enum::Bar, Enum::Foo];
      for x in v.iter() {
          println!("{}", x == Enum::Bar);
      }
  }
  ```

  但是，实际上这行代码会报错，因为你无法将 `x` 直接跟一个枚举成员进行比较。好在，你可以使用 `match` 来完成，但是会导致代码更为啰嗦，是否有更简洁的方式？答案是使用 `matches!`：

  ```rust
  println!("{}", matches!(i, Enum::Bar));
  ```

- 很简单也很简洁，再来看看更多的例子：

  ```rust
  let foo = 'f';
  assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));
  
  let bar = Some(4);
  assert!(matches!(bar, Some(x) if x > 2));
  ```



#### 变量覆盖

- 无论是 `match` 还是 `if let`，他们都可以在模式匹配时覆盖掉老的值，绑定新的值：

  ```rust
  fn main() {
     let age = Some(30);
     println!("在匹配前，age是{:?}",age);		// 在匹配前，age是Some(30)
     if let Some(age) = age {
         println!("匹配出来的age是{}",age);		// 匹配出来的age是30
     }
     println!("在匹配后，age是{:?}",age);		// 在匹配后，age是Some(30)
  }
  ```

- 可以看出在 `if let` 中，`=` 右边 `Some(i32)` 类型的 `age` 被左边 `i32` 类型的新 `age` 覆盖了，该覆盖一直持续到 `if let` 语句块的结束。因此第三个 `println!` 输出的 `age` 依然是 `Some(i32)` 类型。

  对于 `match` 类型也是如此:

  ```rust
  fn main() {
     let age = Some(30);
     println!("在匹配前，age是{:?}",age);
     match age {
         Some(age) =>  println!("匹配出来的age是{}",age),
         _ => ()
     }
     println!("在匹配后，age是{:?}",age);
  }
  ```

> 需要注意的是，**`match` 中的变量覆盖其实不是那么的容易看出**，因此要小心！



#### 解构 Option

- **Rust** 里用 `Option<T>` 来代替传统的 `null` 处理过程。其定义如下：

  ```rust
  pub enum Option<T> {
      /// No value.
      #[lang = "None"]
      #[stable(feature = "rust1", since = "1.0.0")]
      None,
      /// Some value of type `T`.
      #[lang = "Some"]
      #[stable(feature = "rust1", since = "1.0.0")]
      Some(#[stable(feature = "rust1", since = "1.0.0")] T),
  }
  ```

  简单解释就是：**一个变量要么有值：`Some(T)`, 要么为空：`None`**。

  那么现在的问题就是该如何去使用这个 `Option` 枚举类型，根据我们上一节的经验，可以通过 `match` 来实现。

> 因为 `Option`，`Some`，`None` 都包含在 `prelude` 中，因此你可以直接通过名称来使用它们，而无需以 `Option::Some` 这种形式去使用，总之，千万不要因为调用路径变短了，就忘记 `Some` 和 `None` 也是 `Option` 底下的枚举成员！

##### 匹配 Option\<T>

- 使用 `Option<T>`，是为了从 `Some` 中取出其内部的 `T` 值以及处理没有值的情况，为了演示这一点，下面一起来编写一个函数，它获取一个 `Option<i32>`，如果其中含有一个值，将其加一；如果其中没有值，则函数返回 `None` 值：

  ```rust
  fn plus_one(x: Option<i32>) -> Option<i32> {
      match x {
          None => None,
          Some(i) => Some(i + 1),
      }
  }
  let five = Some(5);
  let six = plus_one(five);
  let none = plus_one(None);
  ```

  `plus_one` 接受一个 `Option<i32>` 类型的参数，同时返回一个 `Option<i32>` 类型的值（这种形式的函数在标准库内随处所见），在该函数的内部处理中，如果传入的是一个 `None` ，则返回一个 `None` 且不做任何处理；如果传入的是一个 `Some(i32)`，则通过模式绑定，把其中的值绑定到变量 `i` 上，然后返回 `i+1` 的值，同时用 `Some` 进行包裹。



#### 所有的匹配用法和语法糖

- https://course.rs/basic/match-pattern/all-patterns.html

