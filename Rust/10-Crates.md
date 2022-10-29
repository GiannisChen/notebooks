## Crates

- `crate` 是 `Rust` 里的编译单元，当执行 `rustc some_file.rs` 的时候， `some_file.rs` 将被认为是 ***crate file***。如果 `some_file.rs` 里有 `mod` 的声明，那么模块的内容将插入到 `mod` 声明的地方，并且这个操作将在编译器遍历之前完成。换句话说，模块不会被单独编译，只有 `crates` 被编译。
- `crate` 不仅能被编译成二进制文件，也能被编译成 `library` 。默认情况下， `rustc` 会生成二进制文件，当然，设定 `--crate-type` 为 `lib` 时，将会生成 `library` 文件。

##### 创建 library

- `lib`：

  ```rust
  pub fn public_function() {
      println!("called rary's `public_function()`");
  }
  
  fn private_function() {
      println!("called rary's `private_function()`");
  }
  
  pub fn indirect_access() {
      print!("called rary's `indirect_access()`, that\n> ");
  
      private_function();
  }
  ```

  ```shell
  $ rustc --crate-type=lib rary.rs
  $ ls lib*
  # library.rlib
  ```

- 库的前缀是 **lib**，默认情况下，它们以它们的 `crate file` 命名，但可以通过将 `--crate-name` 选项传递给 `rustc` 或使用 `crate_name` 属性来覆盖这个默认名称。

##### 使用 library

- 要将一个板条箱链接到这个新库，你可以使用 `rustc` 的 `--extern` 标志。它的所有项将被导入到一个命名与库相同的模块下，这个模块的行为通常与任何其他模块相同：

  ```rust
  // extern crate rary; // May be required for Rust 2015 edition or earlier
  
  fn main() {
      rary::public_function();
  
      // Error! `private_function` is private
      //rary::private_function();
  
      rary::indirect_access();
  }
  ```

  ```shell
  # Where library.rlib is the path to the compiled library, assumed that it's
  # in the same directory here:
  $ rustc executable.rs --extern rary=library.rlib --edition=2018 && ./executable 
  # called rary's `public_function()`
  # called rary's `indirect_access()`, that
  # > called rary's `private_function()`
  ```

  