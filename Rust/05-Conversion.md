## 转换（Conversion）

### 介绍

- **基本类型**可以通过类型转换（`casting`）相互转换。

- `Rust` 通过使用特性（`traits`）来处理自定义类型（例如， `struct` 和 `enum` ）之间的转换。泛型转换将使用 `From` 和 `Into` 特性。然而，对于更常见的情况，特别是在转换为字符串（`String`）和从字符串转换时，有更具体的规则。



### `From` & `Into`

-  `From` 和 `Into` 特性是内在相连的，这实际上是其实现的一部分。如果你能够将**A类型**转换成**B类型**，那么我们应该很容易相信我们能够将**B类型**转换为**A类型**。

##### `From`

- `From` 特性允许类型定义如何从另一个类型创建自己，因此提供了一种非常简单的机制，用于在多个类型之间进行转换，在基本类型和公共类型的转换的标准库中有许多这种特性的实现。

- 例如我们能够简单从 `str` 转换 `String`：

  ```rust
  let my_str = "hello";
  let my_string = String::from(my_str);
  ```

- 在为自己的类型定义转换时，也可以做类似的操作：

  ```rust
  use std::convert::From;
  
  #[derive(Debug)]
  struct Int64 {
      value: i64,
  }
  
  impl From<i64> for Int64 {
      fn from(item: i64) -> Self {
          return Int64 { value: item };
      }
  }
  
  fn main() {
      let num = Int64::from(30_i64);
      println!("My Int64 {:?} {}", num, num.value);
  }
  ```

##### `Into`

- `Into` 特性只是 `From` 特性的逆函数，也就是说，如果您已经为您的类型实现了 `From` 特性，那么 `Into` 将在必要时调用它。

- 使用 `Into` 特性通常需要说明要转换为的类型，因为编译器在大多数情况下无法确定这一点。然而，考虑到我们免费获得功能，这是一个小的权衡。

  ```rust
  use std::convert::From;
  
  #[derive(Debug)]
  struct Int64 {
      value: i64,
  }
  
  impl From<i64> for Int64 {
      fn from(item: i64) -> Self {
          return Int64 { value: item };
      }
  }
  
  fn main() {
      let num_i64 = 5;
      let num_2: Int64 = num_i64.into();
      println!("My Int64 {:?} {}", num_2, num_2.value);
  }
  ```

  

### `TryFrom` & `TryInto`

- 与 `From` 和 `Into` 类似，`TryFrom` 和 `TryInto` 是用于类型之间转换的泛型特性。与 `From` / `Into` 不同， `TryFrom` / `TryInto` 特性用于易出错的转换，因此，返回 `Result`：

  ```rust
  use std::convert::{TryFrom, TryInto};
  
  #[derive(Debug, PartialEq)]
  struct EvenNumber(i32);
  
  impl TryFrom<i32> for EvenNumber {
      type Error = ();
      fn try_from(value: i32) -> Result<Self, Self::Error> {
          if value % 2 == 0 {
              return Ok(EvenNumber(value));
          } else {
              return Err(());
          }
      }
  }
  
  fn main() {
      // TryFrom
  
      assert_eq!(EvenNumber::try_from(8), Ok(EvenNumber(8)));
      assert_eq!(EvenNumber::try_from(5), Err(()));
  
      // TryInto
  
      let result: Result<EvenNumber, ()> = 8i32.try_into();
      assert_eq!(result, Ok(EvenNumber(8)));
      let result: Result<EvenNumber, ()> = 5i32.try_into();
      assert_eq!(result, Err(()));
  }
  ```



### `String` 的转换

- 转换成 `String` —— 将任何类型转换为 `String` 就像实现该类型的 `ToString` 特性一样简单。您应该实现 `fmt::Display` 特性，而非直接使用，它自动提供 `ToString` ，还允许打印类型，如 `print!` 一节中所讨论的那样。

- 从 `String` 转化 —— 要将字符串转换为的比较常见的类型之一是数字。对此的惯用方法是使用 `parse` 函数，或者安排类型推断，或者使用 ***turbofish*** 语法指定要解析的类型，下面的示例中显示了这两种备选方案。

  这将把字符串转换为指定的类型，只要为该类型实现 `FromStr` 特性。这是为标准库中的许多类型实现的。要在用户定义的类型上获得此功能，只需为该类型实现 `FromStr` 特性。

  ```rust
  use std::fmt;
  use std::num::ParseIntError;
  use std::str::FromStr;
  
  struct Circle {
      radius: i32,
  }
  
  impl fmt::Display for Circle {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          return write!(f, "Circle of radius {}", self.radius);
      }
  }
  
  impl FromStr for Circle {
      type Err = ParseIntError;
      fn from_str(s: &str) -> Result<Self, Self::Err> {
          return Ok(Circle {
              radius: s.parse().unwrap(),
          });
      }
  }
  
  fn main() {
      let circle = Circle { radius: 6 };
      let new_circle = Circle::from_str("10").unwrap();
      println!("{:?}", circle.to_string());
      println!("{:?}", new_circle.to_string());
  }
  // "Circle of radius 6"
  // "Circle of radius 10"
  ```

  