## Rust by Example

- [Rust](https://www.rust-lang.org/) 是一种现代系统编程语言，关注安全性、速度和并发性。它通过在**不使用垃圾回收**的情况下**实现内存安全**来实现这些目标.
- **参考内容和可能用到的**：
  - **安装**：https://www.rust-lang.org/tools/install
  - **官方文档**：https://doc.rust-lang.org/std/
  - **RustByExample网站源码**：https://github.com/rust-lang/rust-by-example

- **目录**：

  - [Hello World](https://doc.rust-lang.org/stable/rust-by-example/hello.html) - Hello World 经典开局！

  - [Primitives](https://doc.rust-lang.org/stable/rust-by-example/primitives.html) - Learn about signed integers, unsigned integers and other primitives.

  - [Custom Types](https://doc.rust-lang.org/stable/rust-by-example/custom_types.html) - `struct` and `enum`.

  - [Variable Bindings](https://doc.rust-lang.org/stable/rust-by-example/variable_bindings.html) - mutable bindings, scope, shadowing.

  - [Types](https://doc.rust-lang.org/stable/rust-by-example/types.html) - Learn about changing and defining types.

  - [Conversion](https://doc.rust-lang.org/stable/rust-by-example/conversion.html)

  - [Expressions](https://doc.rust-lang.org/stable/rust-by-example/expression.html)

  - [Flow of Control](https://doc.rust-lang.org/stable/rust-by-example/flow_control.html) - `if`/`else`, `for`, and others.

  - [Functions](https://doc.rust-lang.org/stable/rust-by-example/fn.html) - Learn about Methods, Closures and High Order Functions.

  - [Modules](https://doc.rust-lang.org/stable/rust-by-example/mod.html) - Organize code using modules

  - [Crates](https://doc.rust-lang.org/stable/rust-by-example/crates.html) - A crate is a compilation unit in Rust. Learn to create a library.

  - [Cargo](https://doc.rust-lang.org/stable/rust-by-example/cargo.html) - Go through some basic features of the official Rust package management tool.

  - [Attributes](https://doc.rust-lang.org/stable/rust-by-example/attribute.html) - An attribute is metadata applied to some module, crate or item.

  - [Generics](https://doc.rust-lang.org/stable/rust-by-example/generics.html) - Learn about writing a function or data type which can work for multiple types of arguments.

  - [Scoping rules](https://doc.rust-lang.org/stable/rust-by-example/scope.html) - Scopes play an important part in ownership, borrowing, and lifetimes.

  - [Traits](https://doc.rust-lang.org/stable/rust-by-example/trait.html) - A trait is a collection of methods defined for an unknown type: `Self`

  - [Macros](https://doc.rust-lang.org/stable/rust-by-example/macros.html)

  - [Error handling](https://doc.rust-lang.org/stable/rust-by-example/error.html) - Learn Rust way of handling failures.

  - [Std library types](https://doc.rust-lang.org/stable/rust-by-example/std.html) - Learn about some custom types provided by `std` library.

  - [Std misc](https://doc.rust-lang.org/stable/rust-by-example/std_misc.html) - More custom types for file handling, threads.

  - [Testing](https://doc.rust-lang.org/stable/rust-by-example/testing.html) - All sorts of testing in Rust.

  - [Unsafe Operations](https://doc.rust-lang.org/stable/rust-by-example/unsafe.html)

  - [Compatibility](https://doc.rust-lang.org/stable/rust-by-example/compatibility.html)

  - [Meta](https://doc.rust-lang.org/stable/rust-by-example/meta.html) - Documentation, Benchmarking.



### Hello World!

```rust
fn main() {
    println!("hello world!");
}
```

