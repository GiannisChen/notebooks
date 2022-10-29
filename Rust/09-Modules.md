## 模块（Modules）

### 介绍

- `Rust` 提供了一个功能强大的模块系统，可用于在逻辑单元（模块）中分层地分割代码，并管理之间的可见性（公共/私有）。
- 模块是集合，包含函数、结构体、特性、 `impl` 块，甚至其他模块。



### 可见性（Visibility）

- 默认情况下，模块中的项具有私有可见性，但这可以用 `pub` 修饰符覆盖。只有模块的公共项可以从模块范围外访问，也可以指定能够访问的范围：

  - `pub(in <path>)` —— 仅对 `<path>` 路径下可见，但 `<path>` 必须是当前模块的父亲或者祖先；
  - `pub(self)` —— 仅对当前模块可见；
  - `pub(super)` —— 仅对父模块可见；

  ```rust
  // A module named `my_mod`
  mod my_mod {
      // Items in modules default to private visibility.
      fn private_function() {
          println!("called `my_mod::private_function()`");
      }
  
      // Use the `pub` modifier to override default visibility.
      pub fn function() {
          println!("called `my_mod::function()`");
      }
  
      // Items can access other items in the same module,
      // even when private.
      pub fn indirect_access() {
          print!("called `my_mod::indirect_access()`, that\n> ");
          private_function();
      }
  
      // Modules can also be nested
      pub mod nested {
          pub fn function() {
              println!("called `my_mod::nested::function()`");
          }
  
          #[allow(dead_code)]
          fn private_function() {
              println!("called `my_mod::nested::private_function()`");
          }
  
          // Functions declared using `pub(in path)` syntax are only visible
          // within the given path. `path` must be a parent or ancestor module
          pub(in crate::my_mod) fn public_function_in_my_mod() {
              print!("called `my_mod::nested::public_function_in_my_mod()`, that\n> ");
              public_function_in_nested();
          }
  
          // Functions declared using `pub(self)` syntax are only visible within
          // the current module, which is the same as leaving them private
          pub(self) fn public_function_in_nested() {
              println!("called `my_mod::nested::public_function_in_nested()`");
          }
  
          // Functions declared using `pub(super)` syntax are only visible within
          // the parent module
          pub(super) fn public_function_in_super_mod() {
              println!("called `my_mod::nested::public_function_in_super_mod()`");
          }
      }
  
      pub fn call_public_function_in_my_mod() {
          print!("called `my_mod::call_public_function_in_my_mod()`, that\n> ");
          nested::public_function_in_my_mod();
          print!("> ");
          nested::public_function_in_super_mod();
      }
  
      // pub(crate) makes functions visible only within the current crate
      pub(crate) fn public_function_in_crate() {
          println!("called `my_mod::public_function_in_crate()`");
      }
  
      // Nested modules follow the same rules for visibility
      mod private_nested {
          #[allow(dead_code)]
          pub fn function() {
              println!("called `my_mod::private_nested::function()`");
          }
  
          // Private parent items will still restrict the visibility of a child item,
          // even if it is declared as visible within a bigger scope.
          #[allow(dead_code)]
          pub(crate) fn restricted_function() {
              println!("called `my_mod::private_nested::restricted_function()`");
          }
      }
  }
  
  fn function() {
      println!("called `function()`");
  }
  
  fn main() {
      // Modules allow disambiguation between items that have the same name.
      function();
      my_mod::function();
  
      // Public items, including those inside nested modules, can be
      // accessed from outside the parent module.
      my_mod::indirect_access();
      my_mod::nested::function();
      my_mod::call_public_function_in_my_mod();
  
      // pub(crate) items can be called from anywhere in the same crate
      my_mod::public_function_in_crate();
  
      // pub(in path) items can only be called from within the module specified
      // Error! function `public_function_in_my_mod` is private
      //my_mod::nested::public_function_in_my_mod();
      // TODO ^ Try uncommenting this line
  
      // Private items of a module cannot be directly accessed, even if
      // nested in a public module:
  
      // Error! `private_function` is private
      //my_mod::private_function();
      // TODO ^ Try uncommenting this line
  
      // Error! `private_function` is private
      //my_mod::nested::private_function();
      // TODO ^ Try uncommenting this line
  
      // Error! `private_nested` is a private module
      //my_mod::private_nested::function();
      // TODO ^ Try uncommenting this line
  
      // Error! `private_nested` is a private module
      //my_mod::private_nested::restricted_function();
      // TODO ^ Try uncommenting this line
  }
  ```

  

### 结构体的可见性（Struct visibility）

- 结构的字段具有额外的可见性。可见性默认为 `private`，可以用 `pub` 修饰符重写。这种可见性只在从定义它的模块外部访问结构并具有隐藏信息（封装）的目标时才有意义：

  ```rust
  mod my {
      // A public struct with a public field of generic type `T`
      pub struct OpenBox<T> {
          pub contents: T,
      }
  
      // A public struct with a private field of generic type `T`
      #[allow(dead_code)]
      pub struct ClosedBox<T> {
          contents: T,
      }
  
      impl<T> ClosedBox<T> {
          // A public constructor method
          pub fn new(contents: T) -> ClosedBox<T> {
              ClosedBox {
                  contents: contents,
              }
          }
          
          pub fn contents(self) -> T {
              return self.contents;
          }
      }
  }
  
  fn main() {
      // Public structs with public fields can be constructed as usual
      let open_box = my::OpenBox { contents: "public information" };
  
      // and their fields can be normally accessed.
      println!("The open box contains: {}", open_box.contents);
  
      // Public structs with private fields cannot be constructed using field names.
      // Error! `ClosedBox` has private fields
      //let closed_box = my::ClosedBox { contents: "classified information" };
      // TODO ^ Try uncommenting this line
  
      // However, structs with private fields can be created using
      // public constructors
      let close_box = my::ClosedBox::new("classified info");
  
      println!("The closed box contains: {}", close_box.contents());
  
      // let _closed_box = my::ClosedBox::new("classified information");
      // and the private fields of a public struct cannot be accessed.
      // Error! The `contents` field is private
      // println!("The closed box contains: {}", _closed_box.contents);
      // TODO ^ Try uncommenting this line
  }
  ```

  

### use 声明（use declaration）

- `use` 声明可用于将完整路径绑定到新名称，以便更容易地访问（像是 `Python` 的 `import ... as ...` ）。它经常这样使用：

  ```rust
  use crate::deeply::nested::{
      my_first_function,
      my_second_function,
      AndATraitType
  };
  
  fn main() {
      my_first_function();
  }
  ```

  ```rust
  mod deeply {
      pub mod nested {
          pub fn function() {
              println!("called `deeply::nested::function()`");
          }
      }
  }
  
  use deeply::nested::function as func;
  
  fn main() {
      // Easier access to `deeply::nested::function`
      func();
      println!("Entering block");
      {
          // This is equivalent to `use deeply::nested::function as function`.
          // This `function()` will shadow the outer one.
          use crate::deeply::nested::function;
  
          // `use` bindings have a local scope. In this case, the
          // shadowing of `function()` is only in this block.
          function();
  
          println!("Leaving block");
      }
      // function();
      // ^ will raise error: cannot find function `function` in this scope
  }
  ```

  

### super 和 self

- 可以在路径中使用 `super` 和 `self` 关键字，以消除访问项时的模糊性，并防止不必要的路径硬编码：

  ```rust
  fn function() {
      println!("called `function()`");
  }
  
  mod cool {
      pub fn function() {
          println!("called `cool::function()`");
      }
  }
  
  mod my {
      fn function() {
          println!("called `my::function()`");
      }
      
      mod cool {
          pub fn function() {
              println!("called `my::cool::function()`");
          }
      }
      
      pub fn indirect_call() {
          // Let's access all the functions named `function` from this scope!
          print!("called `my::indirect_call()`, that\n> ");
          
          // The `self` keyword refers to the current module scope - in this case `my`.
          // Calling `self::function()` and calling `function()` directly both give
          // the same result, because they refer to the same function.
          self::function();
          function();
          
          // We can also use `self` to access another module inside `my`:
          self::cool::function();
          
          // The `super` keyword refers to the parent scope (outside the `my` module).
          super::function();
          
          // This will bind to the `cool::function` in the *crate* scope.
          // In this case the crate scope is the outermost scope.
          {
              use crate::cool::function as root_function;
              root_function();
          }
      }
  }
  
  fn main() {
      my::indirect_call();
  }
  ```



### 文件结构层次（File hierarchy）

- 模块可以映射到一个文件/目录层次结构。让我们在文件中分解可见性的例子：

  ```shell
  $ tree .
  .
  ├── my
  │   ├── inaccessible.rs
  │   └── nested.rs
  ├── my.rs
  └── split.rs
  ```

- `split.rs` ：

  ```rust
  // This declaration will look for a file named `my.rs` and will
  // insert its contents inside a module named `my` under this scope
  mod my;
  
  fn function() {
      println!("called `function()`");
  }
  
  fn main() {
      my::function();
  
      function();
  
      my::indirect_access();
  
      my::nested::function();
  }
  ```

- `my.rs` ：

  ```rust
  // Similarly `mod inaccessible` and `mod nested` will locate the `nested.rs`
  // and `inaccessible.rs` files and insert them here under their respective
  // modules
  mod inaccessible;
  pub mod nested;
  
  pub fn function() {
      println!("called `my::function()`");
  }
  
  fn private_function() {
      println!("called `my::private_function()`");
  }
  
  pub fn indirect_access() {
      print!("called `my::indirect_access()`, that\n> ");
  
      private_function();
  }
  ```

- `my/nested.rs` ：

  ```rust
  pub fn function() {
      println!("called `my::nested::function()`");
  }
  
  #[allow(dead_code)]
  fn private_function() {
      println!("called `my::nested::private_function()`");
  }
  ```

- `my/inaccessible.rs` ：

  ```rust
  #[allow(dead_code)]
  pub fn public_function() {
      println!("called `my::inaccessible::public_function()`");
  }
  ```

- 结果：

  ```shell
  $ rustc split.rs && ./split
  
  # called `my::function()`
  # called `function()`
  # called `my::indirect_access()`, that
  # > called `my::private_function()`
  # called `my::nested::function()`
  ```

  

