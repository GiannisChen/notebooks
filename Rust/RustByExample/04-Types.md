## 类型（Types）

### 介绍

- `Rust` 提供了几种机制来更改或定义**原语类型**和**用户定义类型**：
  1. 原语类型之间的类型转换（casting）；
  2. 指定所需的字面量类型（literals）；
  3. 使用类型推断（inference）；
  4. 别名类型（aliasing）；



### 类型转换（Casting）

- `Rust` 不提供原始类型之间的隐式类型转换,但是，可以使用 `as` 关键字执行**显式类型转换**（强制转换）。

- 整型之间的转换规则通常遵循C语言的约定，除非C语言有未定义的行为。`Rust` 中很好地定义了整型之间所有类型转换的行为：

  ```rust
  // Suppress all warnings from casts which overflow.
  #![allow(overflowing_literals)]
  
  fn main() {
      let decimal = 65.4321_f32;
  
      // Error! No implicit conversion
      let integer: u8 = decimal;
      // FIXME ^ Comment out this line
  
      // Explicit conversion
      let integer = decimal as u8;
      let character = integer as char;
  
      // Error! There are limitations in conversion rules. 
      // A float cannot be directly converted to a char.
      let character = decimal as char;
      // FIXME ^ Comment out this line
  
      println!("Casting: {} -> {} -> {}", decimal, integer, character);
  
      // when casting any value to an unsigned type, T,
      // T::MAX + 1 is added or subtracted until the value
      // fits into the new type
  
      // 1000 already fits in a u16
      println!("1000 as a u16 is: {}", 1000 as u16);
  
      // 1000 - 256 - 256 - 256 = 232
      // Under the hood, the first 8 least significant bits (LSB) are kept,
      // while the rest towards the most significant bit (MSB) get truncated.
      println!("1000 as a u8 is : {}", 1000 as u8);
      // -1 + 256 = 255
      println!("  -1 as a u8 is : {}", (-1i8) as u8);
  
      // For positive numbers, this is the same as the modulus
      println!("1000 mod 256 is : {}", 1000 % 256);
  
      // When casting to a signed type, the (bitwise) result is the same as
      // first casting to the corresponding unsigned type. If the most significant
      // bit of that value is 1, then the value is negative.
  
      // Unless it already fits, of course.
      println!(" 128 as a i16 is: {}", 128 as i16);
      
      // 128 as i8 -> -128, whose two's complement in eight bits is:
      println!(" 128 as a i8 is : {}", 128 as i8);
  
      // repeating the example above
      // 1000 as u8 -> 232
      println!("1000 as a u8 is : {}", 1000 as u8);
      // and the two's complement of 232 is -24
      println!(" 232 as a i8 is : {}", 232 as i8);
      
      // Since Rust 1.45, the `as` keyword performs a *saturating cast* 
      // when casting from float to int. If the floating point value exceeds 
      // the upper bound or is less than the lower bound, the returned value 
      // will be equal to the bound crossed.
      
      // 300.0 is 255
      println!("300.0 is {}", 300.0_f32 as u8);
      // -100.0 as u8 is 0
      println!("-100.0 as u8 is {}", -100.0_f32 as u8);
      // nan as u8 is 0
      println!("nan as u8 is {}", f32::NAN as u8);
      
      // This behavior incurs a small runtime cost and can be avoided 
      // with unsafe methods, however the results might overflow and 
      // return **unsound values**. Use these methods wisely:
      unsafe {
          // 300.0 is 44
          println!("300.0 is {}", 300.0_f32.to_int_unchecked::<u8>());
          // -100.0 as u8 is 156
          println!("-100.0 as u8 is {}", (-100.0_f32).to_int_unchecked::<u8>());
          // nan as u8 is 0
          println!("nan as u8 is {}", f32::NAN.to_int_unchecked::<u8>());
      }
  }
  ```



### 字面量（Literals）

- 通过添加**类型后缀**，可以对**数值字面量**进行类型注释。例如，要指定字面量 `42` 的类型应为 `i32` ，则需要写入 `42_i32` 。

- **不加后缀的数值字面量**的类型取决于它们的使用方式。如果不存在约束，编译器将对整数使用 `i32` ，对浮点数使用 `f64` 。

  ```rust
  fn main() {
      // Suffixed literals, their types are known at initialization
      let x = 1u8;
      let y = 2u32;
      let z = 3f32;
  
      // Unsuffixed literals, their types depend on how they are used
      let i = 1;
      let f = 1.0;
  
      // `size_of_val` returns the size of a variable in bytes
      println!("size of `x` in bytes: {}", std::mem::size_of_val(&x)); // 1
      println!("size of `y` in bytes: {}", std::mem::size_of_val(&y)); // 4
      println!("size of `z` in bytes: {}", std::mem::size_of_val(&z)); // 4
      println!("size of `i` in bytes: {}", std::mem::size_of_val(&i)); // 4
      println!("size of `f` in bytes: {}", std::mem::size_of_val(&f)); // 8
  }
  ```

  

### 类型推导（Inference）

- 类型推理引擎非常聪明，它不仅仅是在初始化期间查看值表达式的类型，还将研究之后如何使用该变量来推断其类型。下面是一个类型推断的高级示例：

  ```rust
  fn main() {
      // Because of the annotation, the compiler knows that `elem` has type u8.
      let elem = 5u8;
  
      // Create an empty vector (a growable array).
      let mut vec = Vec::new();
      // At this point the compiler doesn't know the exact type of `vec`, it
      // just knows that it's a vector of something (`Vec<_>`).
  
      // Insert `elem` in the vector.
      vec.push(elem);
      // Aha! Now the compiler knows that `vec` is a vector of `u8`s (`Vec<u8>`)
      // TODO ^ Try commenting out the `vec.push(elem)` line
  
      println!("{:?}", vec);
  }
  ```

  

### 别名（Aliasing）

- `type` 语句可用于为现有类型指定新名称，新的类型必须有**大写驼峰型**名称，否则编译器将发出警告。此规则的例外是基本类型：`usize` 、 `f32` 等：

  ```rust
  // `NanoSecond`, `Inch`, and `U64` are new names for `u64`.
  type NanoSecond = u64;
  type Inch = u64;
  type U64 = u64;
  
  fn main() {
      // `NanoSecond` = `Inch` = `U64` = `u64`.
      let nanoseconds: NanoSecond = 5 as U64;
      let inches: Inch = 2 as U64;
  
      // Note that type aliases *don't* provide any extra type safety, because
      // aliases are *not* new types
      println!("{} nanoseconds + {} inches = {} unit?",
               nanoseconds,
               inches,
               nanoseconds + inches);
  }
  ```

- 别名的主要用途是减少样板： `IoResult<T>` 其实是 `Result<T,IoError>` 的别名。