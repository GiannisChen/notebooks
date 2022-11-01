## 控制流（Flow of Control）

### if-else

- 使用 `if-else` 分支与其他语言类似。与许多条件判断不同的是，布尔条件**不需要**用括号包围，每个条件分支后面都有一个块。

  ```rust
  let n = 5;
  
  if n < 0 {
      print!("{} is negative", n);
  } else if n > 0 {
      print!("{} is positive", n);
  } else {
      print!("{} is zero", n);
  }
  ```

- `if-else` 条件判断也可以是一种表达式，并且，所有分支必须返回相同的类型。

  ```rust
  let big_n =
      if n < 10 && n > -10 {
          println!(", and is a small number, increase ten-fold");
  
          // This expression returns an `i32`.
          10 * n
      } else {
          println!(", and is a big number, halve the number");
  
          // This expression must return an `i32` as well.
          n / 2
          // TODO ^ Try suppressing this expression with a semicolon.
      };
      //   ^ Don't forget to put a semicolon here! All `let` bindings need it.
  
  println!("{} -> {}", n, big_n);
  ```



### loop

- `Rust` 提供了一个 `loop` 关键字来进行无限循环。

- `break` 语句可用于随时退出循环，而 `continue` 语句可用于跳过迭代的其余部分并开始新的迭代。

  ```rust
  fn main() {
      let mut count = 0_u32;
  
      // Infinite loop
      loop {
          count += 1;
          if count == 3 {
              println!("Three");
              // Skip the rest of this iteration
              continue;
          }
          println!("{}", count);
          if count == 5 {
              println!("OK. that's ENOUGH!");
              // Exit this loop
              break;
          }
      }
  }
  ```



#### 嵌套（nesting）和标签（labels）

- 在处理嵌套循环时，可以用 `break` 或 `continue` 应对**外部循环**，而不是局限于**最近的一层循环**。在这些情况下，循环**必须**用 `'<label>` 标签进行注释，并且标签**必须**传递给 `break` / `continue` 语句。

  ```rust
  #![allow(unreachable_code)]
  
  fn main() {
      'outer: loop {
          println!("Entered the outer loop");
  
          loop {
              println!("Entered the inner loop");
              // This would break only the inner loop
              //break;
  
              // This breaks the outer loop
              break 'outer;
          }
          // Unreachable code:
          println!("This point will never be reached");
      }
      println!("Exited the outer loop");
  }
  ```



#### 循环中的返回值

- 循环的用途之一是重试操作，直到操作成功。如果该操作返回一个值，那么您可能需要将它传递给代码的其余部分：将它放在 `break` 之后，它将由循环表达式返回：

  ```rust
  fn main() {
      let mut counter = 0;
      let result = loop {
          counter += 1;
          if counter == 10 {
              break counter * 2;
          }
      };
      println!("{}", counter);
      assert_eq!(result, 20);
  }
  ```



### while

- `while` 关键字可用于在**条件为真**时运行循环。让我们使用 `while` 循环来编写臭名昭著的 **FizzBuzz** ：

  ```rust
  fn main() {
      // A counter variable
      let mut n = 1;
  
      // Loop while `n` is less than 101
      while n < 101 {
          if n % 15 == 0 {
              println!("fizzbuzz");
          } else if n % 3 == 0 {
              println!("fizz");
          } else if n % 5 == 0 {
              println!("buzz");
          } else {
              println!("{}", n);
          }
          n += 1;
      }
  }
  ```



### for

#### for range

- `for in` 构造可用于遍历迭代器 `Iterator`。创建迭代器最简单的方法之一是使用范围表示法 `a..b` 。这将在一个步骤中生成从 $[a,b)$ 的值。让我们用 `for` 代替 `while` 来写 **FizzBuzz** ：

  ```rust
  fn main() {
      // `n` will take the values: 1, 2, ..., 100 in each iteration
      for n in 1..101 {
          if n % 15 == 0 {
              println!("fizzbuzz");
          } else if n % 3 == 0 {
              println!("fizz");
          } else if n % 5 == 0 {
              println!("buzz");
          } else {
              println!("{}", n);
          }
      }
  }
  ```

- 另外，可以使用 `a..=b` 来表示 $[a,b]$ 。以上的 **FizzBuzz** 可以写成：

  ```rust
  fn main() {
      // `n` will take the values: 1, 2, ..., 100 in each iteration
      for n in 1..=100 {
          if n % 15 == 0 {
              println!("fizzbuzz");
          } else if n % 3 == 0 {
              println!("fizz");
          } else if n % 5 == 0 {
              println!("buzz");
          } else {
              println!("{}", n);
          }
      }
  }
  ```



#### for iterators

- `for in` 结构能够以多种方式与迭代器（`Iterator`）交互。默认情况下，`for` 循环将对集合应用 `into_iter` 函数（详见 `Iterator` 一章）。然而，这并不是将集合转换为迭代器的唯一方法。

- `Into_iter` 、 `iter` 和 `iter_mut` 都通过提供不同的数据视角，以不同的方式处理集合到迭代器的转换：

  - `iter` —— 这将通过每次迭代借用集合中的每个元素，这样，集合本身就不会受到影响，可以在循环之后重用：

    ```rust
    fn main() {
        let names = vec!["Bob", "Frank", "Ferris"];
        for name in names.iter() {
            match name {
                &"Ferris" => println!("There is a rustacean among us!"),
                // TODO ^ Try deleting the & and matching just "Ferris"
                _ => println!("Hello {}", name),
            }
        }
        println!("names: {:?}", names);
    }
    // Hello Bob
    // Hello Frank
    // There is a rustacean among us!
    // names: ["Bob", "Frank", "Ferris"]
    ```

  - `into_iter` —— 这将消耗集合，以便在每次迭代中提供准确的数据。一旦集合被使用，它就不再可重用，因为它已经在循环中“移动”了：

    ```rust
    fn main() {
        let names = vec!["Bob", "Frank", "Ferris"];
    
        for name in names.into_iter() {
            match name {
                "Ferris" => println!("There is a rustacean among us!"),
                _ => println!("Hello {}", name),
            }
        }
        
        // println!("names: {:?}", names);
        // FIXME ^ Comment out this line, 
        // else raise `borrow of moved value: `names`` error
    }
    ```

  - `iter_mut` —— 这以可变的方式借用集合的每个元素，允许在适当的位置修改集合：

    ```rust
    fn main() {
        let mut names = vec!["Bob", "Frank", "Ferris"];
    
        for name in names.iter_mut() {
            *name = match name {
                &mut "Ferris" => "There is a rustacean among us!",
                _ => "Hello",
            }
        }
    
        println!("names: {:?}", names);
    }
    // names: ["Hello", "Hello", "There is a rustacean among us!"]
    ```

- 在上面的代码片段中注意到匹配分支的类型，这是迭代类型的关键区别。当然，类型的不同意味着能够执行的行为的不同。



### match

- `Rust` 通过 `match` 关键字提供模式匹配，它可以像C语言 `switch` 一样使用，并且必须覆盖所有可能的值：

  ```rust
  fn main() {
      for number in 1..=20 {
          match number {
              1 => println!("one"),
              2 | 3 | 5 | 7 | 11 => println!("prime"),
              13..=19 => println!("teen"),
              _ => println!("overflow"),
          }
      }
  
      let boolean = true;
      let binary = match boolean {
          true => 0,
          false => 1,
      };
      println!("{} -> {}", boolean, binary);
  }
  ```



#### 分解（Destructuring）

- `match` 块能通过各种方法分解。

##### Destructuring tuples

```rust
fn main() {
    let triples = vec![(0, -2, 3), (1, -2, 3), (9, -2, 2), (3, -2, 4), (9, -2, 3)];
    println!("Tell me about {:?}", triples);
    // Match can be used to destructure a tuple

    for triple in triples {
        match triple {
            // Destructure the second and third elements
            (0, y, z) => println!("First is `0`, `y` is {:?}, and `z` is {:?}", y, z),
            (1, ..) => println!("First is `1` and the rest doesn't matter"),
            (.., 2) => println!("last is `2` and the rest doesn't matter"),
            (3, .., 4) => println!("First is `3`, last is `4`, and the rest doesn't matter"),
            // `..` can be used to ignore the rest of the tuple
            _ => println!("It doesn't matter what they are"),
            // `_` means don't bind the value to a variable
        }
    }
}
// Tell me about [(0, -2, 3), (1, -2, 3), (9, -2, 2), (3, -2, 4), (9, -2, 3)]
// First is `0`, `y` is -2, and `z` is 3
// First is `1` and the rest doesn't matter
// last is `2` and the rest doesn't matter
// First is `3`, last is `4`, and the rest doesn't matter
// It doesn't matter what they are
```

##### Destructuring arrays/slices

```rust
fn main() {
    let arrays = [[0, -2, 6], [1, -2, 6], [-1, -2, 6], [3, -2, 6], [7, -2, 6]];
    for array in arrays {
        match array {
            // Binds the second and the third elements to the respective variables
            [0, second, third] => {
                println!("array[0] = 0, array[1] = {}, array[2] = {}", second, third)
            }

            // Single values can be ignored with _
            [1, _, third] => println!(
                "array[0] = 1, array[2] = {} and array[1] was ignored",
                third
            ),

            // You can also bind some and ignore the rest
            [-1, second, ..] => println!(
                "array[0] = -1, array[1] = {} and all the other ones were ignored",
                second
            ),
            // The code below would not compile
            // [-1, second] => ...

            // Or store them in another array/slice (the type depends on
            // that of the value that is being matched against)
            [3, second, tail @ ..] => println!(
                "array[0] = 3, array[1] = {} and the other elements were {:?}",
                second, tail
            ),

            // Combining these patterns, we can, for example, bind the first and
            // last values, and store the rest of them in a single array
            [first, middle @ .., last] => println!(
                "array[0] = {}, middle = {:?}, array[2] = {}",
                first, middle, last
            ),
        }
    }
}
// array[0] = 0, array[1] = -2, array[2] = 6
// array[0] = 1, array[2] = 6 and array[1] was ignored
// array[0] = -1, array[1] = -2 and all the other ones were ignored
// array[0] = 3, array[1] = -2 and the other elements were [6]
// array[0] = 7, middle = [-2], array[2] = 6
```

##### Destructuring enums

```rust
// `allow` required to silence warnings because only
// one variant is used.
#[allow(dead_code)]
enum Color {
    // These 3 are specified solely by their name.
    Red,
    Blue,
    Green,
    // These likewise tie `u32` tuples to different names: color models.
    RGB(u32, u32, u32),
    HSV(u32, u32, u32),
    HSL(u32, u32, u32),
    CMY(u32, u32, u32),
    CMYK(u32, u32, u32, u32),
}

fn main() {
    let colors = [
        Color::Red,
        Color::Blue,
        Color::Green,
        Color::RGB(122, 17, 40),
        Color::HSV(1, 2, 3),
        Color::HSL(2, 3, 4),
        Color::CMY(3, 4, 5),
        Color::CMYK(4, 5, 6, 7),
    ];

    println!("What color is it?");
    for color in colors {
        // An `enum` can be destructured using a `match`.
        match color {
            Color::Red => println!("The color is Red!"),
            Color::Blue => println!("The color is Blue!"),
            Color::Green => println!("The color is Green!"),
            Color::RGB(r, g, b) => println!("Red: {}, green: {}, and blue: {}!", r, g, b),
            Color::HSV(h, s, v) => println!("Hue: {}, saturation: {}, value: {}!", h, s, v),
            Color::HSL(h, s, l) => println!("Hue: {}, saturation: {}, lightness: {}!", h, s, l),
            Color::CMY(c, m, y) => println!("Cyan: {}, magenta: {}, yellow: {}!", c, m, y),
            Color::CMYK(c, m, y, k) => println!(
                "Cyan: {}, magenta: {}, yellow: {}, key (black): {}!",
                c, m, y, k
            ),
            // Don't need another arm because all variants have been examined
        }
    }
}
/*
	What color is it?
    The color is Red!
    The color is Blue!
    The color is Green!
    Red: 122, green: 17, and blue: 40!
    Hue: 1, saturation: 2, value: 3!
    Hue: 2, saturation: 3, lightness: 4!
    Cyan: 3, magenta: 4, yellow: 5!
    Cyan: 4, magenta: 5, yellow: 6, key (black): 7!
*/
```

##### Destructuring pointers/ref

- 对于指针，需要区分**分解**和**解开引用**，因为它们是不同的概念，使用方式与 C/C++ 等语言**不同**。
  - **解引用**：`*`；
  - **分解**： `&` 、 `ref` 、 `ref mut`；

```rust
fn main() {
    // Assign a reference of type `i32`. The `&` signifies there
    // is a reference being assigned.
    let reference = &4;

    match reference {
        // If `reference` is pattern matched against `&val`, it results
        // in a comparison like:
        // `&i32`
        // `&val`
        // ^ We see that if the matching `&`s are dropped, then the `i32`
        // should be assigned to `val`.
        &val => println!("Got a value via destructuring: {:?}", val),
    }

    // To avoid the `&`, you dereference before matching.
    match *reference {
        val => println!("Got a value via dereferencing: {:?}", val),
    }

    // What if you don't start with a reference? `reference` was a `&`
    // because the right side was already a reference. This is not
    // a reference because the right side is not one.
    let _not_a_reference = 3;

    // Rust provides `ref` for exactly this purpose. It modifies the
    // assignment so that a reference is created for the element; this
    // reference is assigned.
    let ref _is_a_reference = 3;

    // Accordingly, by defining 2 values without references, references
    // can be retrieved via `ref` and `ref mut`.
    let value = 5;
    let mut mut_value = 6;

    // Use `ref` keyword to create a reference.
    match value {
        ref r => println!("Got a reference to a value: {:?}", r),
    }

    // Use `ref mut` similarly.
    match mut_value {
        ref mut m => {
            // Got a reference. Gotta dereference it before we can
            // add anything to it.
            *m += 10;
            println!("We added 10. `mut_value`: {:?}", m);
        },
    }
}
// Got a value via destructuring: 4
// Got a value via dereferencing: 4
// Got a reference to a value: 5
// We added 10. `mut_value`: 16
```

##### Destructuring structs

```rust
fn main() {
    struct Foo {
        x: (u32, u32),
        y: u32,
    }

    // Try changing the values in the struct to see what happens
    let foos = [
        Foo { x: (1, 2), y: 3 },
        Foo { x: (2, 2), y: 2 },
        Foo { x: (3, 2), y: 4 },
    ];

    for foo in foos {
        match foo {
            Foo { x: (1, b), y } => println!("First of x is 1, b = {},  y = {} ", b, y),

            // you can destructure structs and rename the variables,
            // the order is not important
            Foo { y: 2, x: i } => println!("y is 2, i = {:?}", i),

            // and you can also ignore some variables:
            Foo { y, .. } => println!("y = {}, we don't care about x", y),
            // this will give an error: pattern does not mention field `x`
            //Foo { y } => println!("y = {}", y),
        }
    }
}
// First of x is 1, b = 2,  y = 3 
// y is 2, i = (2, 2)
// y = 4, we don't care about x
```



#### 哨兵（Guards）

- `match` 可以添加**哨兵**，作为 `default` ：

  ```rust
  enum Temperature {
      Celsius(i32),
      Farenheit(i32),
  }
  
  fn main() {
      let temperatures = [
          Temperature::Celsius(35),
          Temperature::Celsius(29),
          Temperature::Farenheit(87),
          Temperature::Farenheit(85),
      ];
      // ^ TODO try different values for `temperature`
      for temperature in temperatures {
          match temperature {
              Temperature::Celsius(t) if t > 30 => println!("{}°C is above 30 Celsius", t),
              // The `if condition` part ^ is a guard
              Temperature::Celsius(t) => println!("{}°C is below 30 Celsius", t),
  
              Temperature::Farenheit(t) if t > 86 => println!("{}°F is above 86 Farenheit", t),
              Temperature::Farenheit(t) => println!("{}°F is below 86 Farenheit", t),
          }
      }
  }
  // 35°C is above 30 Celsius
  // 29°C is below 30 Celsius
  // 87°F is above 86 Farenheit
  // 85°F is below 86 Farenheit
  ```

- 注意，在检查匹配表达式是否覆盖了所有模式时，编译器**不会**考虑哨兵：

  ```rust
  fn main() {
      let number: u8 = 4;
  
      match number {
          i if i == 0 => println!("Zero"),
          i if i > 0 => println!("Greater than zero"),
          _ => unreachable!("Should never happen."),
          // TODO ^ uncomment to fix compilation
      }
  }
  ```



#### 绑定（Binding）

- **间接访问变量**使得不重新绑定就不可能对该变量进行分支和使用。`match` 提供 `@` 符号将值绑定到名称：

  ```rust
  // A function `age` which returns a `u32`.
  fn age() -> u32 {
      15
  }
  
  fn main() {
      println!("Tell me what type of person you are");
  
      match age() {
          0             => println!("I haven't celebrated my first birthday yet"),
          // Could `match` 1 ..= 12 directly but then what age
          // would the child be? Instead, bind to `n` for the
          // sequence of 1 ..= 12. Now the age can be reported.
          n @ 1  ..= 12 => println!("I'm a child of age {:?}", n),
          n @ 13 ..= 19 => println!("I'm a teen of age {:?}", n),
          // Nothing bound. Return the result.
          n             => println!("I'm an old person of age {:?}", n),
      }
  }
  ```

- 你也可以使用绑定来“解构” `enum`，例如 `Option` ：

  ```rust
  fn some_number() -> Option<u32> {
      Some(42)
  }
  
  fn main() {
      match some_number() {
          // Got `Some` variant, match if its value, bound to `n`,
          // is equal to 42.
          Some(n @ 42) => println!("The Answer: {}!", n),
          // Match any other number.
          Some(n)      => println!("Not interesting... {}", n),
          // Match anything else (`None` variant).
          _            => (),
      }
  }
  ```



### if let

- 某些情况下，匹配枚举类型时， `match` 会显得不那么方便，例如：

  ```rust
  // Make `optional` of type `Option<i32>`
  let optional = Some(7);
  
  match optional {
      Some(i) => {
          println!("This is a really long string and `{:?}`", i);
          // ^ Needed 2 indentations just so we could destructure
          // `i` from the option.
      },
      _ => {},
      // ^ Required because `match` is exhaustive. Doesn't it seem
      // like wasted space?
  };
  ```

- `if let` 在这种场景下更简洁，同时也能增加一些故障处理：

  ```rust
  fn main() {
      // All have type `Option<i32>`
      let number = Some(7);
      let letter: Option<i32> = None;
      let emoticon: Option<i32> = None;
  
      // The `if let` construct reads: "if `let` destructures `number` into
      // `Some(i)`, evaluate the block (`{}`).
      if let Some(i) = number {
          println!("Matched {:?}!", i);
      }
  
      // If you need to specify a failure, use an else:
      if let Some(i) = letter {
          println!("Matched {:?}!", i);
      } else {
          // Destructure failed. Change to the failure case.
          println!("Didn't match a number. Let's go with a letter!");
      }
  
      // Provide an altered failing condition.
      let i_like_letters = false;
  
      if let Some(i) = emoticon {
          println!("Matched {:?}!", i);
      // Destructure failed. Evaluate an `else if` condition to see if the
      // alternate failure branch should be taken:
      } else if i_like_letters {
          println!("Didn't match a number. Let's go with a letter!");
      } else {
          // The condition evaluated false. This branch is the default:
          println!("I don't like letters. Let's go with an emoticon :)!");
      }
  }
  ```

- 同样， `if let` 也可以用来匹配枚举类型：

  ```rust
  // Our example enum
  enum Foo {
      Bar,
      Baz,
      Qux(u32),
  }
  
  fn main() {
      // Create example variables
      let a = Foo::Bar;
      let b = Foo::Baz;
      let c = Foo::Qux(100);
  
      // Variable a matches Foo::Bar
      if let Foo::Bar = a {
          println!("a is foobar");
      }
  
      // Variable b does not match Foo::Bar
      // So this will print nothing
      if let Foo::Bar = b {
          println!("b is foobar");
      }
  
      // Variable c matches Foo::Qux which has a value
      // Similar to Some() in the previous example
      if let Foo::Qux(value) = c {
          println!("c is {}", value);
      }
  
      // Binding also works with `if let`
      if let Foo::Qux(_value @ 100) = c {
          println!("c is one hundred");
      }
  }
  /*
  	a is foobar
  	c is 100
  	c is one hundred	
  */
  ```

> `if let` 好处是允许我们匹配非参数化的 `enum` 变量。即使在枚举没有实现或派生 `PartialEq` 的情况下也是如此。在这种情况下，`if Foo::Bar == a` 将编译失败，因为枚举的实例不能相等，但是 `if let Foo::Bar = a` 是能工作的。



### while let

- 类似于 `if let` ，而 `let` 可以使尴尬的 `match` 序列更容易忍受。考虑下面 `i+1` 的序列：

  ```rust
  #![allow(unused)]
  fn main() {
      // Make `optional` of type `Option<i32>`
      let mut optional = Some(0);
  
      // Repeatedly try this test.
      loop {
          match optional {
              // If `optional` destructures, evaluate the block.
              Some(i) => {
                  if i > 9 {
                      println!("Greater than 9, quit!");
                      optional = None;
                  } else {
                      println!("`i` is `{:?}`. Try again.", i);
                      optional = Some(i + 1);
                  }
                  // ^ Requires 3 indentations!
              },
              // Quit the loop when the destructure fails:
              _ => { break; }
              // ^ Why should this be required? There must be a better way!
          }
  	}
  }
  
  /*
  	`i` is `0`. Try again.
      `i` is `1`. Try again.
      `i` is `2`. Try again.
      `i` is `3`. Try again.
      `i` is `4`. Try again.
      `i` is `5`. Try again.
      `i` is `6`. Try again.
      `i` is `7`. Try again.
      `i` is `8`. Try again.
      `i` is `9`. Try again.
      Greater than 9, quit!
  */
  ```

- 用 `while let` 重写一下：

  ```rust
  fn main() {
      // Make `optional` of type `Option<i32>`
      let mut optional = Some(0);
  
      // This reads: "while `let` destructures `optional` into
      // `Some(i)`, evaluate the block (`{}`). Else `break`.
      while let Some(i) = optional {
          if i > 9 {
              println!("Greater than 9, quit!");
              optional = None;
          } else {
              println!("`i` is `{:?}`. Try again.", i);
              optional = Some(i + 1);
          }
          // ^ Less rightward drift and doesn't require
          // explicitly handling the failing case.
      }
      // ^ `if let` had additional optional `else`/`else if`
      // clauses. `while let` does not have these.
  }
  ```

  
