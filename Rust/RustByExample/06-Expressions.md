## 表达式（Expressions）

### 介绍

- `Rust` 程序（主要）由一系列语句组成：

  ```rust
  fn main() {
      // statement
      // statement
      // statement
  }
  ```

- `Rust` 中有几种类型的语句。最常见的两种方法是**声明变量绑定**和**在表达式中使用 `;`** ：

  ```rust
  fn main() {
      // variable binding
      let x = 5;
  
      // expression;
      x;
      x + 1;
      15;
  }
  ```

- 块（Block）也是表达式，所以它们可以用作赋值的值。块中的最后一个表达式将被赋值给位置表达式，例如局部变量。但是，如果块的最后一个表达式以 `;` 结尾，返回值将是 `()` ：

  ```rust
  fn main() {
      let x = 5u32;
  
      let y = {
          let x_squared = x * x;
          let x_cube = x_squared * x;
  
          // This expression will be assigned to `y`
          x_cube + x_squared + x
      };
  
      let z = {
          // The semicolon suppresses this expression and `()` is assigned to `z`
          2 * x;
      };
  
      println!("x is {:?}", x);
      println!("y is {:?}", y);
      println!("z is {:?}", z);
  }
  // x is 5
  // y is 155
  // z is ()
  ```

