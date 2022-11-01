## Cargo

### 介绍

- `cargo` 是 `Rust` 的包官方管理工具。它有许多真正有用的特性来提高代码质量和开发速度，包括：
  - 依赖管理和集成（官方 `Rust` 包注册表， [crates.io](https://crates.io/) ）；
  - 单元测试和基准测试；
- 具体见：[The Cargo Book](https://doc.rust-lang.org/cargo/) 。



### Dependencies

- 大多数程序都依赖于某些库。如果您曾经手工管理过依赖项，您就会知道这有多痛苦。幸运的是，`Rust` 生态系统自带标准—— `cargo` 可以管理项目的依赖关系。

- 创建一个新的 `Rust` 项目：（在本章的其余部分，让我们假设我们正在制作一个 `binary` ，而不是一个 `library` ，但所有的概念都是一样的）

  ```shell
  # A binary
  cargo new foo
  
  # OR A library
  cargo new --lib foo
  ```

- 然后就能看到如下的文件结构：

  ```shell
  foo
  ├── Cargo.toml
  └── src
      └── main.rs
  ```

- `main.rs` 将作为新项目的根目录源文件（**source file**），`Cargo.toml` 则是 `cargo` 的配置文件。他长这样：

  ```toml
  [package]
  name = "foo"
  version = "0.1.0"
  edition = "2021"
  
  # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
  
  [dependencies]
  ```

  - `name` —— 指定项目名称，这将在**发布时候**被 `crates.io` 使用，同样也作为编译的默认输出名字；
  - `version` —— 版本号，规范是 [Semantic Versioning](http://semver.org/)；
  - `authors` —— 发布时使用的作者列表名字；
  - `[dependencies]` —— 这部分可以在项目里添加依赖；

- 例如，假设我们希望程序拥有一个强大的 `CLI` 。你可以在 `crates.io` 上找到许多很好的包。一个流行的选择是 `clap` ，版本是 `2.27.1` 。要向程序添加依赖项，只需向 `Cargo.toml` 添加以下内容：

  ```toml
  [dependencies]
  clap = "2.27.1"
  ```

  就可以开始使用 `clap` 了。

- `cargo` 也支持其他类型的依赖（https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html）：

  ```toml
  [package]
  name = "foo"
  version = "0.1.0"
  authors = ["mark"]
  
  [dependencies]
  clap = "2.27.1" # from crates.io
  rand = { git = "https://github.com/rust-lang-nursery/rand" } # from online repo
  bar = { path = "../bar" } # from a path in the local filesystem
  ```

- 同样，`cargo` 不止于依赖管理，`Cargo.toml` 的其他字段如下连接中所示：https://doc.rust-lang.org/cargo/reference/manifest.html；

- 为了构建项目，可以执行 `cargo build` ，命令可以在任意地方执行并生效（包括子目录下）。同样，我们也能通过 `cargo run` 构建并运行。**注意**，这些命令将解析所有依赖项，如果需要下载 `crates`，并构建一切，包括您的 `crates` 。



### 惯例（Conventions）

- 在上一章中，我们看到了如下的目录层次结构：

  ```shell
  foo
  ├── Cargo.toml
  └── src
      └── main.rs
  ```

- 事实证明 `cargo` 支持这一点。默认的二进制文件名称是 `main` ，就像我们之前看到的，但是你可以通过将它们放在 `bin/` 目录中来添加额外的二进制文件：

  ```shell
  foo
  ├── Cargo.toml
  └── src
      ├── main.rs
      └── bin
          └── my_other_bin.rs
  ```

- 我们要告知 `cargo` 编译这个额外的二进制，而非默认的 `main.rs` ，则要在 `cargo` 中指定 `--bin my_other_bin` （显然这个是个文件名）；
- 其他功能：https://doc.rust-lang.org/cargo/guide/project-layout.html；



### 测试（Testing）

- 正如我们所知，测试对任何软件都是不可或缺的，`Rust` 对单元测试和集成测试有一流的支持。
- `todo`



### 构建脚本（Build Scripts）

- 有时，从 `cargo` 的正常构建是不够的。也许你的 `crates` 在 `cargo` 成功编译之前需要一些先决条件，比如代码生成，或者一些需要编译的本地代码。为了解决这个问题，我们有 `cargo` 可以运行的构建脚本。

- 要向包中添加构建脚本，可以在 `Cargo.toml` 中指定，如下：

  ```toml
  [package]
  ...
  build = "build.rs"
  ```

- 如果不指定，`cargo` 则会在项目目录下找 `build.rs` 。



### How to use a build script

The build script is simply another Rust file that will be compiled and invoked prior to compiling anything else in the package. Hence it can be used to fulfill pre-requisites of your crate.

Cargo provides the script with inputs via environment variables [specified here](https://doc.rust-lang.org/cargo/reference/environment-variables.html#environment-variables-cargo-sets-for-build-scripts) that can be used.

The script provides output via stdout. All lines printed are written to `target/debug/build/<pkg>/output`. Further, lines prefixed with `cargo:` will be interpreted by Cargo directly and hence can be used to define parameters for the package's compilation.

For further specification and examples have a read of the [Cargo specification](https://doc.rust-lang.org/cargo/reference/build-scripts.html).