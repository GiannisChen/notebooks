## 泛型（Generics）

### 介绍

- **泛型**将类型和功能泛化到更广泛的情况。这对于在许多方面减少代码重复非常有用，但可能需要相当复杂的语法。也就是说，泛型类型需要**非常小心地指定**泛型类型在哪些类型上是有效的。泛型最简单、最常见的用法是用于类型参数。

- 类型参数被指定为泛型，使用尖括号和上方驼峰形大小写：`<Aaa,Bbb,...>` 。**泛型类型参数** 通常表示为 `<T>` 。在 `Rust` 中，**泛型**还描述接受一个或多个泛型类型参数 `<T>` 的任何东西。作为泛型类型参数指定的任何类型都是泛型的，其他的都是具体的（非泛型）。

- 例如，定义一个名为 `foo` 的泛型函数，它接受任意类型的参数 `T` ：

  ```rust
  fn foo<T>(arg: T) { ... }
  ```

- 因为 `T` 是使用 `<T>` 指定为泛型类型参数的，所以在这里作为 `(arg: T)` 使用时，函数被认为是泛型的。即使 `T` 之前被定义为结构体也是如此。下面的例子展示了一些实际的语法：

  ```rust
  // A concrete type `A`.
  struct A;
  
  // In defining the type `Single`, the first use of `A` is not preceded by `<A>`.
  // Therefore, `Single` is a concrete type, and `A` is defined as above.
  struct Single(A);
  //            ^ Here is `Single`s first use of the type `A`.
  
  // Here, `<T>` precedes the first use of `T`, so `SingleGen` is a generic type.
  // Because the type parameter `T` is generic, it could be anything, including
  // the concrete type `A` defined at the top.
  struct SingleGen<T>(T);
  
  fn main() {
      // `Single` is concrete and explicitly takes `A`.
      let _s = Single(A);
      
      // Create a variable `_char` of type `SingleGen<char>`
      // and give it the value `SingleGen('a')`.
      // Here, `SingleGen` has a type parameter explicitly specified.
      let _char: SingleGen<char> = SingleGen('a');
  
      // `SingleGen` can also have a type parameter implicitly specified:
      let _t    = SingleGen(A); // Uses `A` defined at the top.
      let _i32  = SingleGen(6); // Uses `i32`.
      let _char = SingleGen('a'); // Uses `char`.
  }
  ```



### 函数（Functions）

- 同样的规则也可以应用于函数：当类型 `T` 前面有 `<T>` 时，类型 `T` 就变成泛型。

- 如果函数在返回类型为泛型的情况下调用，或者编译器没有足够的信息来推断必要的类型参数，就可能出现这种情况：使用泛型函数**需要显式指定类型参数**。

- 带有显式指定类型参数的函数调用类似于：`fun::<A, B, ...> ()`：

  ```rust
  struct A;          // Concrete type `A`.
  struct S(A);       // Concrete type `S`.
  struct SGen<T>(T); // Generic type `SGen`.
  
  // The following functions all take ownership of the variable passed into
  // them and immediately go out of scope, freeing the variable.
  
  // Define a function `reg_fn` that takes an argument `_s` of type `S`.
  // This has no `<T>` so this is not a generic function.
  fn reg_fn(_s: S) {}
  
  // Define a function `gen_spec_t` that takes an argument `_s` of type `SGen<T>`.
  // It has been explicitly given the type parameter `A`, but because `A` has not 
  // been specified as a generic type parameter for `gen_spec_t`, it is not generic.
  fn gen_spec_t(_s: SGen<A>) {}
  
  // Define a function `gen_spec_i32` that takes an argument `_s` of type `SGen<i32>`.
  // It has been explicitly given the type parameter `i32`, which is a specific type.
  // Because `i32` is not a generic type, this function is also not generic.
  fn gen_spec_i32(_s: SGen<i32>) {}
  
  // Define a function `generic` that takes an argument `_s` of type `SGen<T>`.
  // Because `SGen<T>` is preceded by `<T>`, this function is generic over `T`.
  fn generic<T>(_s: SGen<T>) {}
  
  fn main() {
      // Using the non-generic functions
      reg_fn(S(A));          // Concrete type.
      gen_spec_t(SGen(A));   // Implicitly specified type parameter `A`.
      gen_spec_i32(SGen(6)); // Implicitly specified type parameter `i32`.
  
      // Explicitly specified type parameter `char` to `generic()`.
      generic::<char>(SGen('a'));
  
      // Implicitly specified type parameter `char` to `generic()`.
      generic(SGen('c'));
  }
  ```



### 实现（Implementation）

- 与函数类似，实现需要注意保持通用性。（？不太明白）

  ```rust
  #![allow(unused)]
  fn main() {
  struct S; // Concrete type `S`
  struct GenericVal<T>(T); // Generic type `GenericVal`
  
  // impl of GenericVal where we explicitly specify type parameters:
  impl GenericVal<f32> {} // Specify `f32`
  impl GenericVal<S> {} // Specify `S` as defined above
  
  // `<T>` Must precede the type to remain generic
  impl<T> GenericVal<T> {}
  }
  ```

  ```rust
  struct Val {
      val: f64,
  }
  
  struct GenVal<T> {
      gen_val: T,
  }
  
  // impl of Val
  impl Val {
      fn value(&self) -> &f64 {
          &self.val
      }
  }
  
  // impl of GenVal for a generic type `T`
  impl<T> GenVal<T> {
      fn value(&self) -> &T {
          &self.gen_val
      }
  }
  
  fn main() {
      let x = Val { val: 3.0 };
      let y = GenVal { gen_val: 3i32 };
  
      println!("{}, {}", x.value(), y.value());
  }
  ```



### 特性（Traits）

- 当然，特征也可以是**泛型**。这里我们定义了一个重新实现 `Drop` 特性的泛型方法来删除（`drop`）自身和输入。

  ```rust
  // Non-copyable types.
  struct Empty;
  struct Null;
  
  // A trait generic over `T`.
  trait DoubleDrop<T> {
      // Define a method on the caller type which takes an
      // additional single parameter `T` and does nothing with it.
      fn double_drop(self, _: T);
  }
  
  // Implement `DoubleDrop<T>` for any generic parameter `T` and
  // caller `U`.
  impl<T, U> DoubleDrop<T> for U {
      // This method takes ownership of both passed arguments,
      // deallocating both.
      fn double_drop(self, _: T) {}
  }
  
  fn main() {
      let empty = Empty;
      let null  = Null;
  
      // Deallocate `empty` and `null`.
      empty.double_drop(null);
  
      //empty;
      //null;
      // ^ TODO: Try uncommenting these lines.
  }
  ```



### 绑定（Bounds）

- 在使用泛型时，类型参数通常必须使用 `trait` 作为“边界”，以规定类型实现什么功能。例如，下面的例子使用特性 `Display` 来打印，所以它需要 `T` 被 `Display` 绑定；也就是说，`T` 必须实现 `Display` ：

  ```rust
  // Define a function `printer` that takes a generic type `T` which
  // must implement trait `Display`.
  fn printer<T: Display>(t: T) {
      println!("{}", t);
  }
  ```

- 绑定将泛型限制为符合绑定的类型。那就是：

  ```rust
  struct S<T: Display>(T);
  
  // Error! `Vec<T>` does not implement `Display`. This
  // specialization will fail.
  let s = S(vec![1]);
  ```

- 绑定的另一个影响是，泛型实例被允许访问绑定中指定的 `trait` 的方法。例如：

  ```rust
  // A trait which implements the print marker: `{:?}`.
  use std::fmt::Debug;
  
  trait HasArea {
      fn area(&self) -> f64;
  }
  
  impl HasArea for Rectangle {
      fn area(&self) -> f64 {
          return self.length * self.height;
      }
  }
  
  impl HasArea for Triangle {
      fn area(&self) -> f64 {
          return self.length * self.height / 2.0;
      }
  }
  
  #[derive(Debug)]
  struct Rectangle {
      length: f64,
      height: f64,
  }
  #[derive(Debug)]
  struct Triangle {
      length: f64,
      height: f64,
  }
  
  // The generic `T` must implement `Debug`. Regardless
  // of the type, this will work properly.
  fn print_debug<T: Debug>(t: &T) {
      println!("{:?}", t);
  }
  
  // `T` must implement `HasArea`. Any type which meets
  // the bound can access `HasArea`'s function `area`.
  fn area<T: HasArea>(t: &T) -> f64 {
      return t.area();
  }
  
  fn main() {
      let rectangle = Rectangle {
          length: 3.0,
          height: 4.0,
      };
      let triangle = Triangle {
          length: 3.0,
          height: 4.0,
      };
  
      print_debug(&rectangle);
      println!(
          "Area: {} == {}",
          rectangle.area(),
          area::<Rectangle>(&rectangle)
      );
  
      print_debug(&triangle);
      println!(
          "Area: {} == {}",
          triangle.area(),
          area::<Triangle>(&triangle)
      );
      // ^ TODO: Try uncommenting these.
      // | Error: Does not implement either `Debug` or `HasArea`.
  }
  ```

- 另外需要注意的是，在某些情况下，`where` 子句也可以用于应用绑定，以更直白。

- **空绑定（Empty Bounds）**：绑定工作方式的一个结果是，即使一个特性不包含任何功能，你仍然可以使用它作为边界。 `Eq` 和 `Copy` 是 `std` 库中这类特性的例子。

  ```rust
  struct Cardinal;
  struct BlueJay;
  struct Turkey;
  
  trait Red {}
  trait Blue {}
  
  impl Red for Cardinal {}
  impl Blue for BlueJay {}
  
  // These functions are only valid for types which implement these
  // traits. The fact that the traits are empty is irrelevant.
  fn red<T: Red>(_: &T)   -> &'static str { "red" }
  fn blue<T: Blue>(_: &T) -> &'static str { "blue" }
  
  fn main() {
      let cardinal = Cardinal;
      let blue_jay = BlueJay;
      let _turkey   = Turkey;
  
      // `red()` won't work on a blue jay nor vice versa
      // because of the bounds.
      println!("A cardinal is {}", red(&cardinal));
      println!("A blue jay is {}", blue(&blue_jay));
      //println!("A turkey is {}", red(&_turkey));
      // ^ TODO: Try uncommenting this line.
  }
  ```



### 多重绑定

- 单个类型的多个绑定可以用 `+` 来应用。和普通绑定一样，不同的类型用，来 `,` 分隔：

  ```rust
  use std::fmt::{Debug, Display};
  
  fn compare_prints<T: Debug + Display>(t: &T) {
      println!("Debug   : `{:?}`", t);
      println!("Display : `{}`", t);
  }
  
  fn compare_types<T: Debug, U: Display + Debug>(t: &T, u: &U) {
      println!("t : `{:?}`", t);
      println!("u : `{:?}`", u);
  }
  
  fn main() {
      let string = "words";
      let array = [1, 2, 3];
      let vec = vec![1, 2, 3];
  
      compare_prints(&string);
      // compare_prints(&array);
      // compare_prints(&vec);
  
      // compare_types(&array, &vec);
      // compare_types(&vec, &array);
      // compare_types(&string, &vec);
      compare_types(&vec, &string);
      compare_types(&array, &string);
      compare_types(&string, &string);
      // compare_types(&array, &array);
      // compare_types(&vec, &vec);
      // compare_types(&string, &array);
  }
  ```



### where 子句

- 绑定也可以在 `{` 开头之前就用 `where` 子句来表示，而不是在第一次提到该类型时。此外，`where` 子句可以将绑定应用于任意类型，而不仅仅是类型参数。

- 在这些情况下使用 `where` 子句可以更加方便：

  - 当聚合指定泛型类型和绑定**不那么清晰时**：

    ```rust
    impl <A: TraitB + TraitC, D: TraitE + TraitF> MyTrait<A, D> for YourType {}
    
    // Expressing bounds with a `where` clause
    impl <A, D> MyTrait<A, D> for YourType where
        A: TraitB + TraitC,
        D: TraitE + TraitF {}
    ```

  - 当使用 `where` 子句比使用普通语法更有表现力时。本例中的 `impl` 不能没有 `where` 子句直接表示：

    ```rust
    use std::fmt::Debug;
    
    trait PrintInOption {
        fn print_in_option(self);
    }
    
    // Because we would otherwise have to express this as `T: Debug` or
    // use another method of indirect approach, this requires a `where` clause:
    impl<T> PrintInOption for T
    where
        Option<T>: Debug,
    {
        // We want `Option<T>: Debug` as our bound because that is what's
        // being printed. Doing otherwise would be using the wrong bound.
        fn print_in_option(self) {
            println!("{:?}", Some(self));
        }
    }
    
    fn main() {
        let vec = vec![1, 2, 3];
        vec.print_in_option();
    }
    ```



### 习惯（New Type Idiom）

- `newType` 的习惯用法提供了**编译时的保证** —— 总是提供正确的类型给程序。例如，以年为单位检查年龄的年龄验证函数，必须被赋予类型为 `Years` 的值，但是我暂时只有 `Days` 的类型：

  ```rust
  struct Years(i64);
  struct Days(i64);
  
  impl Years {
      pub fn to_days(&self) -> Days {
          return Days(self.0 * 365);
      }
  }
  
  impl Days {
      pub fn to_years(&self) -> Years {
          return Years(self.0 / 365);
      }
  }
  
  fn old_enough(age: &Years) -> bool {
      return age.0 >= 18;
  }
  
  fn main() {
      let years = Years(5);
      let days = years.to_days();
      println!("Old enought {}", old_enough(&years));
      println!("Old enought {}", old_enough(&days.to_years()));
  
      let Years(years_i64_1) = years;
      let years_i64_2 = years.0;
      println!("Old enought {}", old_enough(&Years(years_i64_1)));
      println!("Old enought {}", old_enough(&Years(years_i64_2)));
  }
  ```

- `old_enough` 函数接受的参数必须是 `Years` 类型，哪怕两个自定义类型的基础类型都是一致的，也是不能混用，要用到其基础类型，则需要 **解析** 或者 **tuple** 的方法。



### 关联项目（Associated Items）

- **关联项目** 指的是一组与各种类型的 `item` 相关的规则。它是 `trait` 泛型的扩展，允许 `trait` 在内部定义新项。一个这样的项被称为**关联类型**，当 `trait` 是其容器类型的泛型时，它提供了更简单的使用模式。

- 在其容器类型上泛型的 `trait` 具有类型规范要求 —— `trait` 的用户必须指定其所有泛型类型。在下面的示例中，`Contains` 特性允许使用泛型类型 `A` 和 `B` 。然后，为 `Container` 类型实现该特征，为 `A` 和 `B` 指定 `i32`，以便它可以与 `fn difference()` 一起使用。

  ```rust
  struct Container(i32, i32);
  
  // A trait which checks if 2 items are stored inside of container.
  // Also retrieves first or last value.
  trait Contains<A, B> {
      fn contains(&self, _: &A, _: &B) -> bool; // Explicitly requires `A` and `B`.
      fn first(&self) -> i32; // Doesn't explicitly require `A` or `B`.
      fn last(&self) -> i32;  // Doesn't explicitly require `A` or `B`.
  }
  
  impl Contains<i32, i32> for Container {
      // True if the numbers stored are equal.
      fn contains(&self, number_1: &i32, number_2: &i32) -> bool {
          (&self.0 == number_1) && (&self.1 == number_2)
      }
  
      // Grab the first number.
      fn first(&self) -> i32 { self.0 }
  
      // Grab the last number.
      fn last(&self) -> i32 { self.1 }
  }
  
  // `C` contains `A` and `B`. In light of that, having to express `A` and
  // `B` again is a nuisance.
  fn difference<A, B, C>(container: &C) -> i32 where
      C: Contains<A, B> {
      container.last() - container.first()
  }
  
  fn main() {
      let number_1 = 3;
      let number_2 = 10;
  
      let container = Container(number_1, number_2);
  
      println!("Does container contain {} and {}: {}",
          &number_1, &number_2,
          container.contains(&number_1, &number_2));
      println!("First number: {}", container.first());
      println!("Last number: {}", container.last());
  
      println!("The difference is: {}", difference(&container));
  }
  ```

- 因为 `Contains` 是泛型的，所以我们必须显式地声明 `fn difference()` 的所有泛型类型。在实践中，我们需要一种方式来表示 `A` 和 `B` 是由输入 `C` 决定的。您将在下一节中看到，**关联类型恰恰提供了这种功能**。

- **关联类型** 的使用通过将内部类型作为输出类型局部移动到 `trait` 中，提高了代码的整体可读性。`trait` 定义的语法如下：

  ```rust
  // `A` and `B` are defined in the trait via the `type` keyword.
  // (Note: `type` in this context is different from `type` when used for
  // aliases).
  trait Contains {
      type A;
      type B;
  
      // Updated syntax to refer to these new types generically.
      fn contains(&self, _: &Self::A, _: &Self::B) -> bool;
  }
  ```

- 注意，使用 `Contains` 特性的函数不再需要表达 `A` 或 `B` ：

  ```rust
  // Without using associated types
  fn difference<A, B, C>(container: &C) -> i32 where
      C: Contains<A, B> { ... }
  
  // Using associated types
  fn difference<C: Contains>(container: &C) -> i32 { ... }
  ```

- 尝试重写一遍上文代码：

  ```rust
  trait Contains {
      type A;
      type B;
  
      fn contains(&self, _: &Self::A, _: &Self::B) -> bool;
      fn first(&self) -> i32;
      fn last(&self) -> i32;
  }
  
  fn difference<C: Contains>(container: &C) -> i32 {
      return container.last() - container.first();
  }
  
  struct Container(i32, i32);
  
  impl Contains for Container {
      type A = i32;
      type B = i32;
  
      fn contains(&self, num_1: &i32, num_2: &i32) -> bool {
          return (&self.0 == num_1) && (&self.1 == num_2);
      }
  
      fn first(&self) -> i32 {
          return self.0;
      }
  
      fn last(&self) -> i32 {
          return self.1;
      }
  }
  
  fn main() {
      let number_1 = 3;
      let number_2 = 10;
  
      let container = Container(number_1, number_2);
  
      println!(
          "Does container contain {} and {}: {}",
          &number_1,
          &number_2,
          container.contains(&number_1, &number_2)
      );
      println!("First number: {}", container.first());
      println!("Last number: {}", container.last());
  
      println!("The difference is: {}", difference(&container));
  }
  ```



### 虚类型参数（Phantom Type Parameters）

- **虚类型参数**是指在运行时不显示，当且仅当在编译时静态检查的参数。数据类型可以使用额外的泛型类型参数作为标记或在编译时执行类型检查。这些额外的参数没有存储值，也没有运行时行为。在下面的示例中，我们将 `std::marker::PhantomData` 与**虚类型参数**概念结合起来创建包含不同数据类型的元组。

  ```rust
  use std::marker::PhantomData;
  
  // A phantom tuple struct which is generic over `A` with hidden parameter `B`.
  #[derive(PartialEq)] // Allow equality test for this type.
  struct PhantomTuple<A, B>(A, PhantomData<B>);
  
  // A phantom type struct which is generic over `A` with hidden parameter `B`.
  #[derive(PartialEq)] // Allow equality test for this type.
  struct PhantomStruct<A, B> { first: A, phantom: PhantomData<B> }
  
  // Note: Storage is allocated for generic type `A`, but not for `B`.
  //       Therefore, `B` cannot be used in computations.
  
  fn main() {
      // Here, `f32` and `f64` are the hidden parameters.
      // PhantomTuple type specified as `<char, f32>`.
      let _tuple1: PhantomTuple<char, f32> = PhantomTuple('Q', PhantomData);
      // PhantomTuple type specified as `<char, f64>`.
      let _tuple2: PhantomTuple<char, f64> = PhantomTuple('Q', PhantomData);
  
      // Type specified as `<char, f32>`.
      let _struct1: PhantomStruct<char, f32> = PhantomStruct {
          first: 'Q',
          phantom: PhantomData,
      };
      // Type specified as `<char, f64>`.
      let _struct2: PhantomStruct<char, f64> = PhantomStruct {
          first: 'Q',
          phantom: PhantomData,
      };
  
      // Compile-time Error! Type mismatch so these cannot be compared:
      // println!("_tuple1 == _tuple2 yields: {}",
      //           _tuple1 == _tuple2);
  
      // Compile-time Error! Type mismatch so these cannot be compared:
      // println!("_struct1 == _struct2 yields: {}",
      //           _struct1 == _struct2);
  }
  ```

- 一个有用的单元转换方法可以通过使用**虚类型参数**实现 `Add` 来检查。Add 特性将在下面进行检查：

  ```rust
  // This construction would impose: `Self + RHS = Output`
  // where RHS defaults to Self if not specified in the implementation.
  pub trait Add<RHS = Self> {
      type Output;
  
      fn add(self, rhs: RHS) -> Self::Output;
  }
  
  // `Output` must be `T<U>` so that `T<U> + T<U> = T<U>`.
  impl<U> Add for T<U> {
      type Output = T<U>;
      ...
  }
  ```

  ```rust
  use std::marker::PhantomData;
  use std::ops::Add;
  
  #[derive(Debug, Clone, Copy)]
  enum Inch {}
  #[derive(Debug, Clone, Copy)]
  enum Mm {}
  
  #[derive(Debug, Clone, Copy)]
  struct Length<Unit>(f64, PhantomData<Unit>);
  
  impl<Unit> Add for Length<Unit> {
      type Output = Length<Unit>;
  
      fn add(self, rhs: Length<Unit>) -> Length<Unit> {
          return Length(self.0 + rhs.0, PhantomData);
      }
  }
  
  fn main() {
      // Specifies `one_foot` to have phantom type parameter `Inch`.
      let one_foot: Length<Inch> = Length(12.0, PhantomData);
      // `one_meter` has phantom type parameter `Mm`.
      let one_meter: Length<Mm> = Length(1000.0, PhantomData);
  
      // `+` calls the `add()` method we implemented for `Length<Unit>`.
      //
      // Since `Length` implements `Copy`, `add()` does not consume
      // `one_foot` and `one_meter` but copies them into `self` and `rhs`.
      let two_feet = one_foot + one_foot;
      let two_meters = one_meter + one_meter;
  
      // Addition works.
      println!("one foot + one_foot = {:?} in", two_feet.0);
      println!("one meter + one_meter = {:?} mm", two_meters.0);
  
      // Nonsensical operations fail as they should:
      // Compile-time Error: type mismatch.
      //let one_feter = one_foot + one_meter;
  }
  ```

  