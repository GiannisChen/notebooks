## 属性（Attributes）

### 介绍

- 属性是应用于某些模块、 `crate` 或物品的元数据。此元数据可用于：
  - 代码的条件编译；
  - 设置 `crate` 名称、版本和类型（ `binary` 或 `library` ）
  - 禁用的限制（ `warnings` ）
  - 启用编译器特性（宏 `macros` 、 `glob` 导入等）
  - 连接到一个外部 `library`
  - 将函数标记为单元测试
  - 将函数标记为基准测试
  - 像宏的属性
- 当希望属性应用到所有 `crate` 内容时，语法是 `#![crate_attribute]` ，如果仅用于一个模块，语法是 `#[item_attribute]` （ `!` 不见了）。
- 当然这里也可以使用不同的语法：
  - `#[attribute = "value"]`
  - `#[attribute(key = "value")]`
  - `#[attribute(value)]`

- 属性可以有多个值，也可以在多行中分隔：

  ```rust
  #[attribute(value, value2)]
  
  #[attribute(value, value2, value3,
              value4, value5)]
  ```



### dead_code

- 编译器提供一个 `dead_code` 限制（ `lint` ）来告警**未使用的函数**：

  ```rust
  fn used_function() {
      println!("used_function!")
  }
  
  // `#[allow(dead_code)]` is an attribute that disables the `dead_code` lint
  #[allow(dead_code)]
  fn unused_function() {
      println!("unused_function is never used!")
  }
  
  fn noisy_unused_function() {
      println!("noisy_unused_function will raise warning!")
  }
  // FIXME ^ Add an attribute to suppress the warning
  
  fn main() {
      used_function();
  }
  ```

> **注意☠**，在实际的程序中，应该消除死代码。在这些示例中，由于示例的交互性，我们将允许在某些地方出现死代码。



### Crates

- `crate_type` 属性可用于告诉编译器一个 `crate` 是 `binary` 还是 `library`（甚至是哪种类型的 `library` ），而 `crate_name` 属性可用于设置 `crate` 的名称：

  ```rust
  // This crate is a library
  #![crate_type = "lib"]
  // The library is named `rary`
  #![crate_name = "rary"]
  
  pub fn public_function() {
      println!("called rary's `public_function()`");
  }
  
  fn private_function() {
      println!("called rary's `private_function()`");
  }
  
  pub fn indirect_function() {
      println!("called rary's `indirect_function()`, that \n>");
      private_function();
  }
  ```

  ```shell
  $ rustc lib.rs
  $ ls lib*
  library.rlib
  ```

> 然而，重要的是要注意， `crate_type` 和 `crate_name` 属性在使用 `Cargo` （ `Rust` 包管理器）时没有任何影响。由于 `Cargo` 用于大多数 `Rust` 项目，这意味着现实生产环境中对 `crate_type` 和 `crate_name` 的使用相对有限。



### cfg

- 配置条件检查可以通过两个不同的操作符进行：
  - `cfg` 属性：`#[cfg(...)]` （在属性位置）；
  - `cfg!` 宏：`cfg!(...)` （布尔表达式）

- 前者支持条件编译，后者则有条件地计算为 `true` 或 `false` ，允许在运行时进行检查，两者都使用相同的参数语法。

- `cfg !` ，与 `#[cfg]` 不同，它不删除任何代码，只计算为 `true` 或 `false` 。例如，**if/else** 表达式中的所有块在 `cfg!` 用于条件，而不管 `cfg!` 正在评估。

  ```rust
  // This function only gets compiled if the target OS is linux
  #[cfg(target_os = "linux")]
  fn are_you_on_linux() {
      println!("You are running linux!");
  }
  
  // And this function only gets compiled if the target OS is *not* linux
  #[cfg(not(target_os = "linux"))]
  fn are_you_on_linux() {
      println!("You are *not* running linux!");
  }
  
  fn main() {
      are_you_on_linux();
  
      println!("Are you sure?");
      if cfg!(target_os = "linux") {
          println!("Yes. It's definitely linux!");
      } else {
          println!("Yes. It's definitely *not* linux!");
      }
  }
  ```

##### 自定义配置

- 一些条件句如 `target_os` 是由 `rustc` 隐式提供的，但是自定义条件句必须使用 `--cfg` 标志传递给 `rustc` ：

  ```rust
  #[cfg(some_condition)]
  fn conditional_function() {
      println!("condition met!");
  }
  
  fn main() {
      conditional_function();
  }
  ```

  ```shell
  $ rustc --cfg some_condition custom.rs && ./custom
  condition met!
  ```

