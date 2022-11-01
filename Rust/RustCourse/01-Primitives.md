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

