## 语法基础（Primitives）

### 默认数据类型

- **整数类型**：
  - **带符号整数**（默认类型）：`1`，`1i32`，`1i64`
  - **无符号整数**：`1u32`，`1u64`
  - **不同进制**整数：`0x`，`0o`，`0b`
- **浮点数类型**：`1.2`
- **字符类型**：`'a'`
- **字符串类型**：`"abc"`
- **布尔值类型**：`true`，`false`
- **分隔符**：`00_001`，`0.00_001`



### 符号

- 和 `C` 以及类似语言有相同的操作符号和使用方法，**参考**：https://en.wikipedia.org/wiki/Operator_precedence#Programming_languages

```rust
fn main() {
	// Integer addition
    println!("1 + 2 = {}", 1u64 + 2);

    // Integer subtraction
    println!("1 - 2 = {}", 1i64 - 2);
    // println!("1 - 2 = {}", 1u32 - 2u32); 
    // will raise error, cause raise overflow when check :(

    // Short-circuiting boolean logic
    println!("true AND false is {}", true && false);
    println!("true OR false is {}", true || false);
    println!("NOT true is {}", !true);

    // Bitwise operations
    println!("0011 AND 0101 is {:04b}", 0b0011u32 & 0b0101);
    println!("0011 OR 0101 is {:04b}", 0b0011u32 | 0b0101);
    println!("0011 XOR 0101 is {:04b}", 0b0011u32 ^ 0b0101);
    println!("1 << 5 is {}", 1u32 << 5);
    println!("0x80 >> 2 is 0x{:x}", 0x80u32 >> 2);

    // Use underscores to improve readability!
    println!("One million is written as {}", 1_000_000u32);
}
```



### 元组（Tuple）

- **元组**可以包含不同的数据类型，用 `()` 来构造，也就意味着每个元组本身就是一个值：$Tuple = (T_1,T_2, \cdots)$ ，其中 $T_i$ 是原组成员的类型。
- **元组**可以用来让函数能够返回**多个返回值**。

```rust
// Tuples can be used as function arguments and as return values
fn reverse(pair: (i32, bool)) -> (bool, i32) {
    // `let` can be used to bind the members of a tuple to variables
    (pair.1, pair.0)
}

// The following struct is for the activity.
#[derive(Debug)]
struct Matrix(f32, f32, f64, f64);

fn main() {
    let long_tuple = (
        1u8, 2u16, 3u32, 4u64, -1i8, -2i16, -3i32, -4i64, 0.1f32, 0.2f64, 'a', true,
    );

    // Values can be extracted from the tuple using tuple indexing
    println!("long tuple first value: {}", long_tuple.0);
    println!("long tuple second value: {}", long_tuple.1);

    // Tuples can be tuple members
    let tuple_of_tuples = ((1u8, 2u16, 2u32), (4u64, -1i8), -2i16);

    // Tuples are printable
    println!("tuple of tuples: {:?}", tuple_of_tuples);
    // But long Tuples (more than 12 elements) cannot be printed
    // let too_long_tuple = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13);
    // println!("too long tuple: {:?}", too_long_tuple);
    // TODO ^ Uncomment the above 2 lines to see the compiler error
    let pair = (1, true);
    println!("pair is {:?}", pair);

    println!("the reversed pair is {:?}", reverse(pair));

    // To create one element tuples, the comma is required to tell them apart
    // from a literal surrounded by parentheses
    println!("one element tuple: {:?}", (5u32,));
    println!("just an integer: {:?}", (5u32));

    //tuples can be destructured to create bindings
    let tuple = (1, "hello", 4.5, true);

    let (a, b, c, d) = tuple;
    println!("{:?}, {:?}, {:?}, {:?}", a, b, c, d);

    let matrix = Matrix(1.1, 1.2, 2.1, 2.2);
    println!("{:?}", matrix);
}
```



### 数组和切片

- **数组**（`Array`）包含了一组类型同为 `T` ，长度为 `length` 的元素，存放在一段连续的内存中。数组的长度在编译是已知，使用如下方式初始化：（`[T; length]`）

  ```rust
  let a: [i32; 5] = [1, 2, 3, 4, 5]
  ```

- **切片**（`Slice`）类似于数组，但是他们的长度在编译时未知。切片由两部分组成，一部分是**切片的指针**，另一部分是**切片的长度**，切片依靠两个字来存储这些信息，字的大小与 `usize` 相同，由处理器架构决定，例如 `x86-64` 上是**64**位。切片可用于指向数组的某一部分，并具有类型名 `&[T]` 。

  ```rust
  let s: &[i32] = a[1..4]
  ```

  显然，**空数组** `[f64, 0]` 和**空切片** `&[]` 是不会相等的，如下：

  ```rust
  // Example of empty slice `&[]`
  let empty_array: [f64; 0] = [];
  assert_eq!(&empty_array, &[]);
  assert_eq!(&empty_array, &[][..]); // same but more verbose
  ```

**全部代码**：

```rust
use std::mem;

// This function borrows a slice
fn analyze_slice(slice: &[f64]) {
    println!("first element of the slice: {}", slice[0]);
    println!("the slice has {} elements", slice.len());
}

fn main() {
    let as1: [f64; 4] = [1.0, 2.0, 3.0, 4.0];
    let as2: [f64; 10] = [1.1; 10];
    // Indexing starts at 0
    println!("first element of the array: {}", as1[0]);
    println!("second element of the array: {}", as1[1]);
    println!("full array is: {:?}", as1);
    // `len` returns the count of elements in the array
    println!("number of elements in array: {}", as1.len());

    // Indexing starts at 0
    println!("first element of the array: {}", as2[0]);
    println!("second element of the array: {}", as2[1]);
    println!("full array is: {:?}", as2);

    // Arrays are stack allocated
    println!("array occupies {} bytes", mem::size_of_val(&as1));

    // Arrays can be automatically borrowed as slices
    println!("borrow the whole array as a slice");
    analyze_slice(&as1);

    // Slices can point to a section of an array
    // They are of the form [starting_index..ending_index]
    // starting_index is the first position in the slice
    // ending_index is one more than the last position in the slice
    println!("borrow a section of the array as a slice");
    analyze_slice(&as2[1..4]);
    
    let tmp_slice: &[f64] = &as2[1..5];
    println!("slice of as2: {:?}", tmp_slice);

    // Example of empty slice `&[]`
    let empty_array: [u32; 0] = [];
    assert_eq!(&empty_array, &[]);
    assert_eq!(&empty_array, &[][..]); // same but more verbose
    
    // Arrays can be safely accessed using `.get`, which returns an
    // `Option`. This can be matched as shown below, or used with
    // `.expect()` if you would like the program to exit with a nice
    // message instead of happily continue.
    for i in 0..as1.len() + 1 { // OOPS, one element too far
        match as1.get(i) {
            Some(xval) => println!("{}: {}", i, xval),
            None => println!("Slow down! {} is too far!", i),
        }
    }
}
```



### 注释（comments）

- **常规注释**，`//` 和 `/* */` ：

  ```rust
  // Line comments which go to the end of the line.
  /* Block comments which go to the closing delimiter. */
  ```

- 将会被解析成为 `HTML` 文档的注释：

  ```rust
  /// Generate library docs for the following item.
  
  //! Generate library docs for the enclosing item.
  ```

  

### 格式化输出（Formatred print）

##### 输出

- 输出交由 [`macros`](https://doc.rust-lang.org/stable/rust-by-example/macros.html) （宏）处理，这些宏在 `std::fmt` 中定义，有如下一些内容：

  - `format!`：将格式化字符串输出到 `String`；

  - `print!`：与 `format!` 类似，但是内容将被输出到控制台（`io::stdout`）。

  - `println!`：与 `print!` ，但是会多一个换行符。

  - `eprint!`：与 `print!` ，但是内容将输出到标准错误输出（`io::stderr`）。

  - `eprintln!`：与 `eprint!` ，但是会多一个换行符。

    > 它们都以相同的方式解析文本，作为一个优点，`Rust` 在**编译时**检查格式正确性。

##### 占位符

- `{}` —— 普通占位符，将会自动用参数替换掉占位符，参数必须是可以字符串化的。

  ```rust
  // In general, the `{}` will be automatically replaced with any
  // arguments. These will be stringified.
  println!("{} days", 31);
  // 31 days
  ```

- `{1}` —— 带索引的占位符，将会判断哪个参数用于替换该占位符，从 `0` 开始迭代。

  ```rust
  // Positional arguments can be used. Specifying an integer inside `{}`
  // determines which additional argument will be replaced. Arguments start
  // at 0 immediately after the format string
  println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob");
  // Alice, this is Bob. Bob, this is Alice
  ```

- `{subject}`、`{named}` —— 带命名的占位符，同上，不过现在是用名称来指定替换的参数。

  ```rust
  // As can named arguments.
  println!("{subject} {verb} {object}",
      object="the lazy dog",
      subject="the quick brown fox",
      verb="jumps over");
  // the quick brown fox jumps over the lazy dog
  ```



##### 格式化

- `{:b}`、`{:o}`、`{:x}`、`{:X}` —— 不同进制。

  ```rust
  // Different formatting can be invoked by specifying the format character after a
  // `:`.
  println!("Base 10 repr:               {}",   69420);
  println!("Base 2 (binary) repr:       {:b}", 69420);
  println!("Base 8 (octal) repr:        {:o}", 69420);
  println!("Base 16 (hexadecimal) repr: {:x}", 69420);
  println!("Base 16 (hexadecimal) repr: {:X}", 69420);
  // Base 10 repr:               69420
  // Base 2 (binary) repr:       10000111100101100
  // Base 8 (octal) repr:        207454
  // Base 16 (hexadecimal) repr: 10f2c
  // Base 16 (hexadecimal) repr: 10F2C
  ```



##### 排列（Alignment）

- `{:[fill]>5}` —— 右对齐；

- `{:[fill]^5}` —— 中间对齐；

- `{:[fill]<5}` —— 左对齐；

- `{:[fill]>width$}` —— 浮动大小的右对齐；

  ```rust
  println!("number: {:->5}", 1);
  println!("number: {num:->5}", num = 1);
  println!("number: {num:-<5}", num = 1);
  println!("number: {num:-^5}", num = 1);
  println!("number: {num:->width$}", num = 1, width = 5);
  /*
  number: ----1
  number: ----1
  number: 1----
  number: --1--
  number: ----1
  */
  ```



> 在 `Rust 1.58` 及以上的版本中，可以直接**使用局部变量作为参数**。
>
> ```rust
> // For Rust 1.58 and above, you can directly capture the argument from a
> // surrounding variable. Just like the above, this will output
> // "     1". 5 white spaces and a "1".
> let number: f64 = 1.0;
> let width: usize = 5;
> println!("{number:>width$}");
> ```

```rust
use std::fmt::{self, Display, Formatter};

struct City {
    name: &'static str,
    // Latitude
    lat: f32,
    // Longitude
    lon: f32,
}

impl Display for City {
    // `f` is a buffer, and this method must write the formatted string into it
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        let lat_c = if self.lat >= 0.0 { 'N' } else { 'S' };
        let lon_c = if self.lon >= 0.0 { 'E' } else { 'W' };
        // `write!` is like `format!`, but it will write the formatted string
        // into a buffer (the first argument)
        write!(
            f,
            "{}: {:.3}°{} {:.3}°{}",
            self.name,
            self.lat.abs(),
            lat_c,
            self.lon.abs(),
            lon_c
        )
    }
}

#[derive(Debug)]
struct Color {
    r: u8,
    g: u8,
    b: u8,
}

impl Display for Color {
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        write!(
            f,
            "0x{:0>2}{:0>2}{:0>2}",
            format!("{:X}", self.r),
            format!("{:X}", self.g),
            format!("{:X}", self.b)
        )
    }
}

fn main() {
    for city in [
        City { name: "Dublin", lat: 53.347778, lon: -6.259722 },
        City { name: "Oslo", lat: 59.95, lon: 10.75 },
        City { name: "Vancouver", lat: 49.25, lon: -123.1 },
    ]
    .iter()
    { println!("{}", *city); }
    for color in [
        Color { r: 128, g: 255, b: 90 },
        Color { r: 0, g: 3, b: 254 },
        Color { r: 0, g: 0, b: 0 },
    ]
    .iter()
    {
        // Switch this to use {} once you've added an implementation
        // for fmt::Display.
        println!("{}", *color);
    }
}
```



##### Debug

- 所有想要使用 `std::fmt` ***格式化特性***（`traits`）的类型都要是**可打印的**。自动实现只提供给诸如 `std` 库中的类型。所有其他的都必须以**某种方式手动实现**。

- `fmt::Debug` 的**特性**（`trait`）可以让格式化打印变得简单。**所有**的类型可以通过**获得**（`derive`）`fmt::Debug` 实现格式化打印。但是这在 `fmt::Display` 下不是这样的，还是需要手动实现 `fmt::Display` 来控制显示。
- 更加显示友好的打印可以使用 `{:#?}`。

```rust
// Derive the `fmt::Debug` implementation for `Structure`. `Structure`
// is a structure which contains a single `i32`.
#[derive(Debug)]
struct Structure(i32);

// Put a `Structure` inside of the structure `Deep`. Make it printable
// also.
#[derive(Debug)]
struct Deep(Structure);

#[derive(Debug)]
struct Person<'a> {
    name: &'a str,
    age: u8,
}

fn main() {
    // Printing with `{:?}` is similar to with `{}`.
    println!("{:?} months in a year.", 12);
    println!(
        "{1:?} {0:?} is the {actor:?} name.",
        "Slater",
        "Christian",
        actor = "actor's"
    );

    // `Structure` is printable!
    println!("Now {:?} will print!", Structure(3));

    // The problem with `derive` is there is no control over how
    // the results look. What if I want this to just show a `7`?
    println!("Now {:?} will print!", Deep(Structure(7)));

    let name = "Peter";
    let age = 27;
    let peter = Person { name, age };

    // Pretty print
    println!("{:#?}", peter);
}
```



##### 显示（Display）

- `fmt::Debug` 在某些情况下不够清晰和优雅，我们还是需要使用自定义的**格式化格式**来指定自定义类型的格式化输出，通过手动实现 `fmt::Display` ，在输出中使用 `{}` 即可：

  ```rust
  // Import (via `use`) the `fmt` module to make it available.
  use std::fmt;
  
  // Define a structure for which `fmt::Display` will be implemented. This is
  // a tuple struct named `Structure` that contains an `i32`.
  struct Structure(i32);
  
  // To use the `{}` marker, the trait `fmt::Display` must be implemented
  // manually for the type.
  impl fmt::Display for Structure {
      // This trait requires `fmt` with this exact signature.
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          // Write strictly the first element into the supplied output
          // stream: `f`. Returns `fmt::Result` which indicates whether the
          // operation succeeded or failed. Note that `write!` uses syntax which
          // is very similar to `println!`.
          write!(f, "{}", self.0)
      }
  }
  
  // A structure holding two numbers. `Debug` will be derived so the results can
  // be contrasted with `Display`.
  #[derive(Debug)]
  struct MinMax(i64, i64);
  
  // Implement `Display` for `MinMax`.
  impl fmt::Display for MinMax {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          // Use `self.number` to refer to each positional data point.
          write!(f, "({}, {})", self.0, self.1)
      }
  }
  
  // Define a structure where the fields are nameable for comparison.
  #[derive(Debug)]
  struct Point2D {
      x: f64,
      y: f64,
  }
  
  // Similarly, implement `Display` for `Point2D`
  impl fmt::Display for Point2D {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          // Customize so only `x` and `y` are denoted.
          write!(f, "x: {}, y: {}", self.x, self.y)
      }
  }
  
  fn main() {
      let new_struct = Structure(11);
      println!("{}", new_struct);
      
  
      let minmax = MinMax(0, 14);
  
      println!("Compare structures:");
      println!("Display: {}", minmax);
      println!("Debug: {:?}", minmax);
  
      let big_range = MinMax(-300, 300);
      let small_range = MinMax(-3, 3);
  
      println!(
          "The big range is {big} and the small is {small}",
          small = small_range,
          big = big_range
      );
  
      let point = Point2D { x: 3.3, y: 7.2 };
  
      println!("Compare points:");
      println!("Display: {}", point);
      println!("Debug: {:?}", point);
  
      // Error. Both `Debug` and `Display` were implemented, but `{:b}`
      // requires `fmt::Binary` to be implemented. This will not work.
      // println!("What does Point2D look like in binary: {:b}?", point);
  }
  ```

  他的输出看起来是这样的：

  ```rust
  /*
      11
      Compare structures:
      Display: (0, 14)
      Debug: MinMax(0, 14)
      The big range is (-300, 300) and the small is (-3, 3)
      Compare points:
      Display: x: 3.3, y: 7.2
      Debug: Point2D { x: 3.3, y: 7.2 }
  */
  ```

- 那么，这样的语句会发生什么：

  ```rust
  println!("What does Point2D look like in binary: {:b}?", point);
  // Error. Both `Debug` and `Display` were implemented, but `{:b}`
  // requires `fmt::Binary` to be implemented. This will not work.
  ```

  会在**编译阶段**发现错误，因为 `fmt::Display` 已经实现了，但 `fmt::Binary` 还没有实现，因此不能使用。`fmt` 有许多这样的**特性**（`traits`），每个特性都需要自己的实现。这在 `std::fmt` 中有进一步的详细描述。



##### `write!()?`

- 对于必须按顺序处理每个元素的结构，实现 `fmt::Display` 是很棘手的，每一次 `write!()` 都会生成一个`fmt::Result` ，但是正确处理这个问题需要处理所有的结果。`Rust` 提供了 `?` 运算符就是为了这个目的。（不得不说，`stream` + `buffer` 的思路，好用，但是好丑...）

  ```rust
  // Try `write!` to see if it errors. If it errors, return
  // the error. Otherwise continue.
  write!(f, "{}", value)?;
  ```

- 那么有序序列的格式化输出看起来是这样的：

  ```rust
  use std::fmt; // Import the `fmt` module.
  
  // Define a structure named `List` containing a `Vec`.
  struct List(Vec<i32>);
  
  impl fmt::Display for List {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          // Extract the value using tuple indexing,
          // and create a reference to `vec`.
          let vec = &self.0;
          write!(f, "[")?;
          // Iterate over `v` in `vec` while enumerating the iteration
          // count in `count`.
          for (count, v) in vec.iter().enumerate() {
              if count != 0 {
                  // For every element except the first, add a comma.
                  // Use the ? operator to return on errors.
                  write!(f, ", ")?;
              }
              write!(f, "{}: {}", v)?;
          }
          // Close the opened bracket and return a fmt::Result value.
          write!(f, "]")
      }
  }
  
  fn main() {
      let v = List(vec![1, 2, 3]);
      println!("Display: {}", v);
      // raise error: the trait `std::fmt::Display` 
      // is not implemented for `Vec<i32>`
      // println!("fmt:     {}", v.0);
  }
  ```

  
