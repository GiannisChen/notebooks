## 自定义数据类型（Custom Types）

- **自定义的数据**类型通过两种方式实现：结构体（`struct`）和枚举类型（`enum`）；
- **常量**类型通过 `const` 和 `static` 关键词指定；



### 结构体（structure）

1. 给自定义的元组类型**命名**，即：

   ```rust
   struct PairA(i32, f32);
   ```

2. 经典的 `C` 的结构体：

   ```rust
   struct Point {
   	x: f32,
   	y: f32,
   }
   ```

3. **单元结构**，单元结构是无字段的，对于泛型非常有用：

   ```rust
   struct Unit;
   ```

- **结构体构造**：

  ```rust
  let point: Point = Point { x: 1.0, y: 2.0 };
  ```

- **结构体“反构造”**：（虽然用的是 `Destructure` ，我觉得反构造比析构更好一点？🤔）

  ```rust
  let Point { x: x, y: y } = point;
  // x == 1.0, y == 2.0
  ```

- **结构体自动解析**：（语法糖）

  ```rust
  let point2: Point = Point { x: 3.0, ..point }
  // x == 3.0, y == 2.0
  ```

  

**全部代码**：

```rust
// An attribute to hide warnings for unused code.
#![allow(dead_code)]

#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

// A unit struct
struct Unit;

// A tuple struct
struct Pair(i32, f32);

// A struct with two fields
struct Point {
    x: f32,
    y: f32,
}

// Structs can be reused as fields of another struct
struct Rectangle {
    // A rectangle can be specified by where the top left and bottom right
    // corners are in space.
    top_left: Point,
    bottom_right: Point,
}

fn main() {
    // Create struct with field init shorthand
    let name = String::from("Peter");
    let age = 27;
    let peter = Person { name, age };

    // Print debug struct
    println!("{:?}", peter);

    // Instantiate a `Point`
    let point: Point = Point { x: 3.2, y: 3.4 };

    // Access the fields of the point
    println!("point coordinates: ({}, {})", point.x, point.y);

    // Make a new point by using struct update syntax to use the fields of our
    // other one
    let bottom_right = Point { x: 5.2, ..point };

    // `bottom_right.y` will be the same as `point.y` because we used that field
    // from `point`
    println!("second point: ({}, {})", bottom_right.x, bottom_right.y);

    // Destructure the point using a `let` binding
    let Point {
        x: left_edge,
        y: top_edge,
    } = point;

    println!("left {}, top {}", left_edge, top_edge);

    let _rectangle = Rectangle {
        // struct instantiation is an expression too
        top_left: Point {
            x: left_edge,
            y: top_edge,
        },
        bottom_right: bottom_right,
    };

    // Instantiate a unit struct
    let _unit = Unit;

    // Instantiate a tuple struct
    let pair = Pair(1, 0.1);

    // Access the fields of a tuple struct
    println!("pair contains {:?} and {:?}", pair.0, pair.1);

    // Destructure a tuple struct
    let Pair(integer, decimal) = pair;

    println!("pair contains {:?} and {:?}", integer, decimal);
}
```



### 枚举类型（enum）

- `enum` 关键字允许创建一个类型，该类型可能是几个不同的变体之一。任何可以作为有效的结构体（`struct`）也可以作为有效的枚举类型（`enum`）。

```rust
// Create an `enum` to classify a web event. Note how both
// names and type information together specify the variant:
// `PageLoad != PageUnload` and `KeyPress(char) != Paste(String)`.
// Each is different and independent.
enum WebEvent {
    // An `enum` may either be `unit-like`,
    PageLoad,
    PageUnload,
    // like tuple structs,
    KeyPress(char),
    Paste(String),
    // or c-like structures.
    Click { x: i64, y: i64 },
}

// A function which takes a `WebEvent` enum as an argument and
// returns nothing.
fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("page loaded"),
        WebEvent::PageUnload => println!("page unloaded"),
        // Destructure `c` from inside the `enum`.
        WebEvent::KeyPress(c) => println!("pressed '{}'.", c),
        WebEvent::Paste(s) => println!("pasted \"{}\".", s),
        // Destructure `Click` into `x` and `y`.
        WebEvent::Click { x, y } => {
            println!("clicked at x={}, y={}.", x, y);
        }
    }
}

fn main() {
    let pressed = WebEvent::KeyPress('x');
    let pasted  = WebEvent::Paste("my text".to_owned());
    let click   = WebEvent::Click { x: 20, y: 80 };
    let load    = WebEvent::PageLoad;
    let unload  = WebEvent::PageUnload;

    inspect(pressed);
    inspect(pasted);
    inspect(click);
    inspect(load);
    inspect(unload);
}
```



#### 类型别名（Type aliases）

- 如果使用**类型别名**，则可以通过别名引用每个枚举变量。如果枚举的名称太长或太通用，而您想要重命名它，那么这可能很有用。

  ```rust
  enum VeryVerboseEnumOfThingsToDoWithNumbers {
      Add,
      Subtract,
  }
  
  impl VeryVerboseEnumOfThingsToDoWithNumbers {
      fn run(&self, x: i32, y: i32) -> i32 {
          match self {
              Self::Add => x + y,
              Self::Subtract => x - y,
          }
      }
  }
  
  // Creates a type alias
  type Operations = VeryVerboseEnumOfThingsToDoWithNumbers;
  
  fn main() {
      // We can refer to each variant via its alias, not its long and inconvenient
      // name.
      let add = Operations::Add;
      println!("{}", add.run(1, 2));
      let sub = Operations::Subtract;
      println!("{}", sub.run(1, 2));
  }
  ```



#### `use` 关键词

- 可以使用 `use` 声明类型，不需要手动指定域。

  ```rust
  // An attribute to hide warnings for unused code.
  #![allow(dead_code)]
  
  enum Status {
      Rich,
      Poor,
  }
  
  enum Work {
      Civilian,
      Soldier,
  }
  
  fn main() {
      // Explicitly `use` each name so they are available without
      // manual scoping.
      use crate::Status::{Poor, Rich};
      // Automatically `use` each name inside `Work`.
      use crate::Work::*;
  
      // Equivalent to `Status::Poor`.
      let status = Poor;
      // Equivalent to `Work::Civilian`.
      let work = Civilian;
  
      match status {
          // Note the lack of scoping because of the explicit `use` above.
          Rich => println!("The rich have lots of money!"),
          Poor => println!("The poor have no money..."),
      }
  
      match work {
          // Note again the lack of scoping.
          Civilian => println!("Civilians work!"),
          Soldier => println!("Soldiers fight!"),
      }
  }
  // The poor have no money...
  // Civilians work!
  ```

  

#### C语言一样的枚举类型

- `enum` 也可以像 C 语言的枚举类型一样使用。

  ```rust
  // An attribute to hide warnings for unused code.
  #![allow(dead_code)]
  
  // enum with implicit discriminator (starts at 0)
  enum Number {
      Zero,
      One,
      Two,
  }
  
  // enum with explicit discriminator
  enum Color {
      Red = 0xFF0000,
      Green = 0x00FF00,
      Blue = 0x0000FF,
  }
  
  fn main() {
      // `enums` can be cast as integers.
      println!("zero is {}", Number::Zero as i32);
      println!("one is {}", Number::One as i32);
  
      println!("roses are #{:06x}", Color::Red as i32);
      println!("violets are #{:06x}", Color::Blue as i32);
  }
  ```



#### `enum` 创建链表

- 使用 `enum` 创建链表（**递归**）神操作，真滴🐂 :-)

  ```rust
  use crate::List::*;
  
  enum List {
      // Cons: Tuple struct that wraps an element and a pointer to the next node
      Cons(u32, Box<List>),
      // Nil: A node that signifies the end of the linked list
      Nil,
  }
  
  // Methods can be attached to an enum
  impl List {
      // Create an empty list
      fn new() -> List {
          // `Nil` has type `List`
          return Nil;
      }
  
      // Consume a list, and return the same list with a new element at its front
      fn prepend(self, elem: u32) -> List {
          // `Cons` also has type List
          return Cons(elem, Box::new(self));
      }
  
      // Return the length of the list
      fn len(&self) -> u32 {
          // `self` has to be matched, because the behavior of this method
          // depends on the variant of `self`
          // `self` has type `&List`, and `*self` has type `List`, matching on a
          // concrete type `T` is preferred over a match on a reference `&T`
          // after Rust 2018 you can use self here and tail (with no ref) below as well,
          // rust will infer &s and ref tail.
          // See https://doc.rust-lang.org/edition-guide/rust-2018/ownership-and-lifetimes/default-match-bindings.html
          match *self {
              // Can't take ownership of the tail, because `self` is borrowed;
              // instead take a reference to the tail
              Cons(_, ref tail) => return 1 + tail.len(),
              // Base Case: An empty list has zero length
              Nil => return 0,
          }
      }
  
      // Return representation of the list as a (heap allocated) string
      fn stringify(&self) -> String {
          match *self {
              Cons(head, ref tail) => {
                  // `format!` is similar to `print!`, but returns a heap
                  // allocated string instead of printing to the console
                  return format!("{}, {}", head, tail.stringify());
              }
              Nil => return format!("Nil"),
          }
      }
  }
  
  fn main() {
      // Create an empty linked list
      let mut list = List::new();
      // Prepend some elements
      list = list.prepend(1);
      list = list.prepend(2);
      list = list.prepend(3);
      // Show the final state of the list
      println!("linked list has length: {}", list.len());
      println!("{}", list.stringify());
  }
  ```



### 常量（constants）

- `Rust` 提供了两种不同的常量，他们都能在任意地方被声明，包括全局变量：

  1. `const`：不可更改的值（常规情况）；
  2. `static`：一个可能具有静态生存期（`'static`）的可变变量（`mut`）。静态寿命是推断出来的，不需要指定。访问或修改可变静态变量是不安全（`unsafe`）的。

  ```rust
  // Globals are declared outside all other scopes.
  static LANGUAGE: &str = "Rust";
  const THRESHOLD: i32 = 10;
  fn is_big(n: i32) -> bool {
      // Access constant in some function
      return n > THRESHOLD;
  }
  
  fn main() {
      let n = 18;
      // Access constant in the main thread
      println!("This is {}", LANGUAGE);
      println!("The threshold is {}", THRESHOLD);
      println!("{} is {}", n, if is_big(n) { "big" } else { "small" });
      // Error! Cannot modify a `const`.
      // THRESHOLD = 5;
      // FIXME ^ Comment out this line
  }
  // This is Rust
  // The threshold is 10
  // 18 is big
  ```

  
