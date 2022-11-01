## 变量（Variable）

### 介绍

- `Rust` 通过**静态类型声明**提供类型安全，变量绑定可以在声明时进行类型注释。同样，在大多数情况下，编译器将能够从上下文推断变量的类型，这大大减少了注释负担。

- 值（如字面量）可以使用 `let` 绑定绑定到变量。

  ```rust
  fn main() {
      let an_integer = 1u32;
      let a_boolean = true;
      let unit = ();
  
      // copy `an_integer` into `copied_integer`
      let copied_integer = an_integer;
      println!("An integer: {:?}", copied_integer);
      println!("A boolean: {:?}", a_boolean);
      println!("Meet the unit value: {:?}", unit);
      // The compiler warns about unused variable bindings; these warnings can
      // be silenced by prefixing the variable name with an underscore
      let _unused_variable = 3u32;
  
      let noisy_unused_variable = 2u32;
      // FIXME ^ Prefix with an underscore to suppress the warning
      // Please note that warnings may not be shown in a browser
  }
  // An integer: 1
  // A boolean: true
  // Meet the unit value: ()
  ```

  

### 可变变量（Mutability）

- 默认情况下，变量绑定是不可变的，但是可以使用 `mut` 修饰符重写它。

  ```rust
  fn main() {
      let _immutable_binding = 1;
      let mut mutable_binding = 1;
  
      println!("Before mutation: {}", mutable_binding);
  
      // Ok
      mutable_binding += 1;
  
      println!("After mutation: {}", mutable_binding);
  
      // Error!
      // _immutable_binding += 1;
      // FIXME ^ Comment out this line
  }
  ```

  

#### 作用域（Scope）

- 变量绑定有一个作用域，其作用域和周期局限在一个**块**（Block）里，块是用大括号 `{}` 括起来的语句集合。

  ```rust
  fn main() {
      // This binding lives in the main function
      let long_lived_binding = 1;
  
      // This is a block, and has a smaller scope than the main function
      {
          // This binding only exists in this block
          let long_lived_binding = 3;
          let short_lived_binding = 2;
  
          println!("inner short: {}", short_lived_binding);
          println!("inner long : {}", long_lived_binding);
      }
      // End of the block
  
      // Error! `short_lived_binding` doesn't exist in this scope
      // println!("outer short: {}", short_lived_binding);
      // FIXME ^ Comment out this line
  
      println!("outer long : {}", long_lived_binding);
  }
  // inner short: 2
  // inner long : 3
  // outer long : 1
  ```



#### 先声明后初始化

- 可以先声明变量绑定，然后再初始化它们。然而，这种形式**很少**被使用，因为它可能导致使用未初始化的变量。

- **编译器禁止使用未初始化的变量**，因为这会导致未定义的行为。

  ```rust
  fn main() {
      // Declare a variable binding
      let a_binding;
  
      {
          let x = 2;
  
          // Initialize the binding
          a_binding = x * x;
      }
  
      println!("a binding: {}", a_binding);
  
      let another_binding;
      // Error! Use of uninitialized binding
      // println!("another binding: {}", another_binding);
      // FIXME ^ Comment out this line
      another_binding = 1;
  
      println!("another binding: {}", another_binding);
  }
  // a binding: 4
  // another binding: 1
  ```

  

#### 冻结（Freezing）

- 当数据被相同名称的不可变量绑定时，它也会被冻结。冻结的数据不能被修改，直到不可变绑定超出作用域：

  ```rust
  fn main() {
      let mut _mutable_integer = 7i32;
  
      {
          // Shadowing by immutable `_mutable_integer`
          let _mutable_integer = _mutable_integer;
  
          // Error! `_mutable_integer` is frozen in this scope
          _mutable_integer = 50;
          // FIXME ^ Comment out this line
  
          // `_mutable_integer` goes out of scope
      }
  
      // Ok! `_mutable_integer` is not frozen in this scope
      _mutable_integer = 3;
  }
  ```



