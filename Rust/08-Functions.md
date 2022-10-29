## 方程（Functions）

### 介绍

- 函数使用 `fn` 关键字声明，它的实参是有类型注释的，就像变量一样，并且，如果函数返回一个值，返回类型必须在箭头后面指定 `->` 。

- 函数中的最后一个表达式将被用作返回值。或者，可以使用 `return` 语句从函数内部返回较早的值，甚至 `loop` 或 `if` 语句中返回。

  ```rust
  // Unlike C/C++, there's no restriction on the order of function definitions
  fn main() {
      // We can use this function here, and define it somewhere later
      fizzbuzz_to(100);
  }
  
  // Function that returns a boolean value
  fn is_divisible_by(lhs: u32, rhs: u32) -> bool {
      // Corner case, early return
      if rhs == 0 {
          return false;
      }
      // This is an expression, the `return` keyword is not necessary here
      return lhs % rhs == 0;
  }
  
  // Functions that "don't" return a value, actually return the unit type `()`
  fn fizzbuzz(n: u32) -> () {
      if is_divisible_by(n, 15) {
          println!("fizzbuzz");
      } else if is_divisible_by(n, 3) {
          println!("fizz");
      } else if is_divisible_by(n, 5) {
          println!("buzz");
      } else {
          println!("{}", n);
      }
  }
  // When a function returns `()`, the return type can be omitted from the
  // signature
  fn fizzbuzz_to(n: u32) {
      for i in 1..=n {
          fizzbuzz(i);
      }
  }
  ```



### 关联方程（Associated functions）和方法（Methods）

- 有些函数连接到特定的类型。它们有两种形式：

  - **关联函数** —— 关联函数通常是在类型上定义的函数（例如：`new()` 函数）；
  - **方法** —— 方法是在类型的特定实例上调用的关联函数（形如类方法）；

  ```rust
  struct Point {
      x: f64,
      y: f64,
  }
  
  // Implementation block, all `Point` associated functions & methods go in here
  impl Point {
      // This is an "associated function" because this function is associated with
      // a particular type, that is, Point.
      //
      // Associated functions don't need to be called with an instance.
      // These functions are generally used like constructors.
      fn origin() -> Point {
          return Point { x: 0.0, y: 0.0 };
      }
  
      // Another associated function, taking two arguments:
      fn new(x: f64, y: f64) -> Point {
          return Point { x: x, y: y };
      }
  }
  
  struct Rectangle {
      p1: Point,
      p2: Point,
  }
  
  impl Rectangle {
      // This is a method
      // `&self` is sugar for `self: &Self`, where `Self` is the type of the
      // caller object. In this case `Self` = `Rectangle`
      fn area(&self) -> f64 {
          // `abs` is a `f64` method that returns the absolute value of the
          // caller
          return ((self.p1.x - self.p2.x) * (self.p1.y - self.p2.y)).abs();
      }
  
      fn perimeter(&self) -> f64 {
          return 2.0 * ((self.p1.x - self.p2.x).abs() + (self.p1.y - self.p2.y).abs());
      }
  
      // This method requires the caller object to be mutable
      // `&mut self` desugars to `self: &mut Self`
      fn translate(&mut self, x: f64, y: f64) {
          self.p1.x += x;
          self.p2.x += x;
          self.p1.y += y;
          self.p2.y += y;
      }
  }
  
  // `Pair` owns resources: two heap allocated integers
  struct Pair(Box<i32>, Box<i32>);
  
  impl Pair {
      // This method "consumes" the resources of the caller object
      // `self` desugars to `self: Self`
      fn destroy(self) {
          // Destructure `self`
          let Pair(first, second) = self;
  
          println!("Destroying Pair({}, {})", first, second);
  
          // `first` and `second` go out of scope and get freed
      }
  }
  
  fn main() {
      let rectangle = Rectangle {
          // Associated functions are called using double colons
          p1: Point::origin(),
          p2: Point::new(3.0, 4.0),
      };
  
      // Methods are called using the dot operator
      // Note that the first argument `&self` is implicitly passed, i.e.
      // `rectangle.perimeter()` === `Rectangle::perimeter(&rectangle)`
      println!("Rectangle perimeter: {}", rectangle.perimeter());
      println!("Rectangle area: {}", rectangle.area());
  
      let mut square = Rectangle {
          p1: Point::origin(),
          p2: Point::new(1.0, 1.0),
      };
  
      // Error! `rectangle` is immutable, but this method requires a mutable
      // object
      //rectangle.translate(1.0, 0.0);
      // TODO ^ Try uncommenting this line
  
      // Okay! Mutable objects can call mutable methods
      square.translate(1.0, 1.0);
  
      let pair = Pair(Box::new(1), Box::new(2));
  
      pair.destroy();
  
      // Error! Previous `destroy` call "consumed" `pair`
      //pair.destroy();
      // TODO ^ Try uncommenting this line
  }
  // Rectangle perimeter: 14
  // Rectangle area: 12
  // Destroying Pair(1, 2)
  ```



### 闭包（Closures）

- 闭包是可以捕获封闭环境的函数，例如，一个捕获 `x` 变量的闭包：

  ```rust
  |val| val + x
  ```

- 闭包的语法和功能使它们非常便于动态使用。调用闭包就像调用函数一样。不同的是，输入和返回类型都可以推断，并且必须指定输入变量名：

  ```rust
  fn main() {
      // Increment via closures and functions.
      fn function(i: i32) -> i32 {
          return i + 1;
      }
  
      // Closures are anonymous, here we are binding them to references
      // Annotation is identical to function annotation but is optional
      // as are the `{}` wrapping the body. These nameless functions
      // are assigned to appropriately named variables.
      let closure_annotated = |i: i32| -> i32 { i + 1 };
      let closure_inferred = |i| i + 1;
  
      let i = 1;
      // Call the function and closures.
      println!("function: {}", function(i));
      println!("closure_annotated: {}", closure_annotated(i));
      println!("closure_inferred: {}", closure_inferred(i));
      // Once closure's type has been inferred, it cannot be inferred again with another type.
      //println!("cannot reuse closure_inferred with another type: {}", closure_inferred(42i64));
      // TODO: uncomment the line above and see the compiler error.
  
      // A closure taking no arguments which returns an `i32`.
      // The return type is inferred.
      let one = || 1;
      println!("closure returning one: {}", one());
  }
  ```

#### 捕获（Capturing）

- 闭包本质上是灵活的，它可以满足功能的需要，使闭包在没有标注的情况下工作。这允许捕获灵活地适应用例，有时是移动，有时是借用。闭包可以通过以下方式**捕获**变量：

  - 通过引用：`&T`；
  - 通过可变引用：`&mut T`；
  - 通过数值：`T`；

- 它们**优先**通过引用捕获变量，只在使用接下来的其他方法：

  ```rust
  fn main() {
      use std::mem;
      
      let color = String::from("green");
  
      // A closure to print `color` which immediately borrows (`&`) `color` and
      // stores the borrow and closure in the `print` variable. It will remain
      // borrowed until `print` is used the last time. 
      //
      // `println!` only requires arguments by immutable reference so it doesn't
      // impose anything more restrictive.
      let print = || println!("`color`: {}", color);
  
      // Call the closure using the borrow.
      print();
  
      // `color` can be borrowed immutably again, because the closure only holds
      // an immutable reference to `color`. 
      let _reborrow = &color;
      print();
  
      // A move or reborrow is allowed after the final use of `print`
      let _color_moved = color;
  
  
      let mut count = 0;
      // A closure to increment `count` could take either `&mut count` or `count`
      // but `&mut count` is less restrictive so it takes that. Immediately
      // borrows `count`.
      //
      // A `mut` is required on `inc` because a `&mut` is stored inside. Thus,
      // calling the closure mutates the closure which requires a `mut`.
      let mut inc = || {
          count += 1;
          println!("`count`: {}", count);
      };
  
      // Call the closure using a mutable borrow.
      inc();
  
      // The closure still mutably borrows `count` because it is called later.
      // An attempt to reborrow will lead to an error.
      // let _reborrow = &count; 
      // ^ TODO: try uncommenting this line.
      inc();
  
      // The closure no longer needs to borrow `&mut count`. Therefore, it is
      // possible to reborrow without an error
      let _count_reborrowed = &mut count; 
  
      
      // A non-copy type.
      let movable = Box::new(3);
  
      // `mem::drop` requires `T` so this must take by value. A copy type
      // would copy into the closure leaving the original untouched.
      // A non-copy must move and so `movable` immediately moves into
      // the closure.
      let consume = || {
          println!("`movable`: {:?}", movable);
          mem::drop(movable);
      };
  
      // `consume` consumes the variable so this can only be called once.
      consume();
      // consume();
      // ^ TODO: Try uncommenting this line.
  }
  ```

- 在垂直管道之前使用 `move` 命令强制闭包获取捕获变量的所有权：

  ```rust
  fn main() {
      // `Vec` has non-copy semantics.
      let haystack = vec![1, 2, 3];
  
      let contains = move |needle| haystack.contains(needle);
  
      println!("{}", contains(&1));
      println!("{}", contains(&4));
  
      // println!("There're {} elements in vec", haystack.len());
      // ^ Uncommenting above line will result in compile-time error
      // because borrow checker doesn't allow re-using variable after it
      // has been moved.
      
      // Removing `move` from closure's signature will cause closure
      // to borrow _haystack_ variable immutably, hence _haystack_ is still
      // available and uncommenting above line will not cause an error.
  }
  ```



#### 作为输入参数

- 虽然 `Rust` 选择如何动态捕获变量而不需要类型注释，但在编写函数时不允许这种模糊性。当使用闭包作为输入参数时，闭包的完整类型必须使用几个特征之一进行注释，它们由闭包对捕获值的处理方式决定。按照限制的递减顺序，它们是：

  - `Fn` —— 闭包通过引用使用捕获的值（`&T`）；
  - `FnMut` —— 闭包通过可变引用使用捕获的值（`&mut T`）；
  - FnOnce —— 闭包逐个使用捕获的值（`T`）；

- 在一个变量一个变量的基础上，编译器将以尽可能少的限制方式捕获变量。例如，考虑一个注释为 `FnOnce` 的参数。这指定闭包可以通过 `&T` 、 `&mut T` 或 `T` 捕获，但编译器最终将根据捕获的变量在闭包中使用的方式进行选择。

  这是因为如果移动是可能的，那么任何类型的借位也应该是可能的。注意，反过来就不是这样了。如果参数被注释为 `Fn` ，则不允许使用 `&mut T` 或 `T` 捕获变量。

  在下面的例子中，尝试交换 `Fn` ，`FnMut` 和 `FnOnce` 的用法，看看会发生什么：

  ```rust
  // A function which takes a closure as an argument and calls it.
  // <F> denotes that F is a "Generic type parameter"
  
  fn apply<F>(f: F)
  where
      // The closure takes no input and returns nothing.
      F: FnOnce(),
  {
      // ^ TODO: Try changing this to `Fn` or `FnMut`.
      f();
  }
  
  // A function which takes a closure and returns an `i32`.
  fn apply_to_3<F>(f: F) -> i32
  where
      // The closure takes an `i32` and returns an `i32`.
      F: Fn(i32) -> i32,
  {
      return f(3);
  }
  
  fn main() {
      use std::mem;
      let greeting = "hello";
      // A non-copy type.
      // `to_owned` creates owned data from borrowed one
      let mut farewell = "goodbye".to_owned();
      // Capture 2 variables: `greeting` by reference and
      // `farewell` by value.
      let diary = || {
          // `greeting` is by reference: requires `Fn`.
          println!("I said {}.", greeting);
          // Mutation forces `farewell` to be captured by
          // mutable reference. Now requires `FnMut`.
          farewell.push_str("!!!");
          println!("Then I sceamed {}", farewell);
          println!("Now I can sleep. zzzzz");
          mem::drop(farewell);
      };
  
      // Call the function which applies the closure.
      apply(diary);
  
      // `double` satisfies `apply_to_3`'s trait bound
      let double = |x| 2 * x;
  
      println!("3 doubled: {}", apply_to_3(double));
  }
  ```

  

#### 匿名类型（Type anonymity）

- 闭包简洁地从封闭作用域捕获变量，这带来了使用闭包作为函数参数需要用到**泛型**的结果。这是必要的，因为它们的定义方式：

  ```rust
  #![allow(unused)]
  fn main() {
      // `F` must be generic.
      fn apply<F>(f: F) where
          F: FnOnce() {
          f();
      }
  }
  ```

- 当定义一个闭包时，编译器隐式地创建一个新的匿名结构来将捕获的变量存储在里面，同时通过特性之一实现该功能：针对这种未知类型的 `Fn` 、 `FnMut` 或 `FnOnce` 。此类型被赋值给变量，该变量在调用之前一直被存储。

  因为这个新类型是未知类型，所以函数中的任何用法都需要泛型。然而，一个无界类型参数 `<T>` 仍然是模棱两可的，是不允许的。因此，通过其中一个特征： `Fn` 、 `FnMut` 或 `FnOnce` （它实现的）来限定它的类型就足够了：

  ```rust
  // `F` must implement `Fn` for a closure which takes no
  // inputs and returns nothing - exactly what is required
  // for `print`.
  fn apply<F>(f: F) where
      F: Fn() {
      f();
  }
  
  fn main() {
      let x = 7;
  
      // Capture `x` into an anonymous type and implement
      // `Fn` for it. Store it in `print`.
      let print = || println!("{}", x);
  
      apply(print);
  }
  ```



#### 函数和闭包同时作为输入参数

- 如果声明一个函数，它接受一个闭包作为参数，那么任何满足闭包的特征界的函数都可以作为参数传递：

  ```rust
  // Define a function which takes a generic `F` argument
  // bounded by `Fn`, and calls it
  fn call_me<F: FnOnce()>(f: F) {
      f();
  }
  
  // Define a wrapper function satisfying the `Fn` bound
  fn function() {
      println!("I'm a function!");
  }
  
  fn main() {
      use std::mem;
      let mut msg = "test".to_owned();
      // Define a closure satisfying the `Fn` bound
      let closure = || {
          msg.push_str("!!!");
          println!("{}", msg);
          mem::drop(msg);
      };
  
      call_me(closure);
      call_me(function);
  }
  ```

- 另外需要注意的是， `Fn` 、 `FnMut` 和 `FnOnce` 特性指示闭包如何从封闭作用域捕获变量。



#### 作为返回值（输出值）

- 可以将闭包作为输入参数，因此也应该可以将闭包作为**输出参数返回**。然而，根据定义，匿名闭包类型是未知的，因此我们必须使用 `impl` 来返回它们。有效的返回值是：
  - `Fn`
  - `FnMut`
  - `FnOnce`

- 必须使用 `move` 关键字，这表明所有捕获都是按值发生的。这是必需的，因为任何引用捕获都会在函数退出时立即被删除，从而在闭包中留下无效的引用。

  ```rust
  fn create_fn() -> impl Fn() {
      let text = "Fn".to_owned();
      move || {
          println!("This is a: {}", text);
      }
  }
  
  fn create_fnmut() -> impl FnMut() {
      let text = "FnMut".to_owned();
      move || {
          println!("This is a: {}", text);
      }
  }
  
  fn create_fnonce() -> impl FnOnce() {
      let text = "FnOnce".to_owned();
      move || {
          println!("This is a: {}", text);
      }
  }
  
  fn main() {
      let fn_plain = create_fn();
      let mut fn_mut = create_fnmut();
      let fn_once = create_fnonce();
  
      fn_plain();
      fn_mut();
      fn_once();
  }
  ```



#### 标准库中的真实示例（Examples in std）

- `std` 中出现的一些闭包示例。

##### Iterator::any

- `Iterator::any` 是一个函数，当传递给迭代器时，如果任何元素满足谓词，则返回 `true` ，否则 `false` ：

  ```rust
  fn main() {
      let vec1 = vec![1, 2, 3];
      let vec2 = vec![4, 5, 6];
  
      // `iter()` for vecs yields `&i32`. Destructure to `i32`.
      println!("2 in vec1: {}", vec1.iter()     .any(|&x| x == 2));
      // `into_iter()` for vecs yields `i32`. No destructuring required.
      println!("2 in vec2: {}", vec2.into_iter().any(| x| x == 2));
  
      // `iter()` only borrows `vec1` and its elements, so they can be used again
      println!("vec1 len: {}", vec1.len());
      println!("First element of vec1 is: {}", vec1[0]);
      // `into_iter()` does move `vec2` and its elements, so they cannot be used again
      // println!("First element of vec2 is: {}", vec2[0]);
      // println!("vec2 len: {}", vec2.len());
      // TODO: uncomment two lines above and see compiler errors.
  
      let array1 = [1, 2, 3];
      let array2 = [4, 5, 6];
  
      // `iter()` for arrays yields `&i32`.
      println!("2 in array1: {}", array1.iter()     .any(|&x| x == 2));
      // `into_iter()` for arrays yields `i32`.
      println!("2 in array2: {}", array2.into_iter().any(|x| x == 2));
  }
  ```

  其中 `any` 函数如下：

  ```rust
  pub trait Iterator {
      // The type being iterated over.
      type Item;
  
      // `any` takes `&mut self` meaning the caller may be borrowed
      // and modified, but not consumed.
      fn any<F>(&mut self, f: F) -> bool where
          // `FnMut` meaning any captured variable may at most be
          // modified, not consumed. `Self::Item` states it takes
          // arguments to the closure by value.
          F: FnMut(Self::Item) -> bool;
  }
  ```



##### Iterator::find

- `Iterator::find` 是一个遍历迭代器并搜索满足某个条件的第一个值的函数。如果没有一个值满足条件，则返回`none` ：

  ```rust
  fn main() {
      let vec1 = vec![1, 2, 3];
      let vec2 = vec![4, 5, 6];
  
      // `iter()` for vecs yields `&i32`.
      let mut iter = vec1.iter();
      // `into_iter()` for vecs yields `i32`.
      let mut into_iter = vec2.into_iter();
  
      // `iter()` for vecs yields `&i32`, and we want to reference one of its
      // items, so we have to destructure `&&i32` to `i32`
      println!("Find 2 in vec1: {:?}", iter     .find(|&&x| x == 2));
      // `into_iter()` for vecs yields `i32`, and we want to reference one of
      // its items, so we have to destructure `&i32` to `i32`
      println!("Find 2 in vec2: {:?}", into_iter.find(| &x| x == 2));
  
      let array1 = [1, 2, 3];
      let array2 = [4, 5, 6];
  
      // `iter()` for arrays yields `&i32`
      println!("Find 2 in array1: {:?}", array1.iter()     .find(|&&x| x == 2));
      // `into_iter()` for arrays yields `i32`
      println!("Find 2 in array2: {:?}", array2.into_iter().find(|&&x| x == 2));
  }
  ```

  ```rust
  pub trait Iterator {
      // The type being iterated over.
      type Item;
  
      // `find` takes `&mut self` meaning the caller may be borrowed
      // and modified, but not consumed.
      fn find<P>(&mut self, predicate: P) -> Option<Self::Item> where
          // `FnMut` meaning any captured variable may at most be
          // modified, not consumed. `&Self::Item` states it takes
          // arguments to the closure by reference.
          P: FnMut(&Self::Item) -> bool;
  }
  ```

- `Iterator::find` 返回对元素的引用，如果需要获得其索引号，则使用 `Iterator::position` ：

  ```rust
  fn main() {
      let vec = vec![1, 9, 3, 3, 13, 2];
  
      // `iter()` for vecs yields `&i32` and `position()` does not take a reference, so
      // we have to destructure `&i32` to `i32`
      let index_of_first_even_number = vec.iter().position(|&x| x % 2 == 0);
      assert_eq!(index_of_first_even_number, Some(5));
      
      // `into_iter()` for vecs yields `i32` and `position()` does not take a reference, so
      // we do not have to destructure    
      let index_of_first_negative_number = vec.into_iter().position(|x| x < 0);
      assert_eq!(index_of_first_negative_number, None);
  }
  ```



### 高阶函数

- Rust提供了高阶函数（**Higher Order Functions** —— **HOF**）。这些函数接受一个或多个函数，并/或产生一个更有用的函数。**HOF**和惰性迭代器赋予 `Rust` 独特魅力：

  ```rust
  fn is_odd(n: u32) -> bool {
      n % 2 == 1
  }
  
  fn main() {
      println!("Find the sum of all the squared odd numbers under 1000");
      let upper = 1000;
  
      // Imperative approach
      // Declare accumulator variable
      let mut acc = 0;
      // Iterate: 0, 1, 2, ... to infinity
      for n in 0.. {
          // Square the number
          let n_squared = n * n;
  
          if n_squared >= upper {
              // Break loop if exceeded the upper limit
              break;
          } else if is_odd(n_squared) {
              // Accumulate value, if it's odd
              acc += n_squared;
          }
      }
      println!("imperative style: {}", acc);
  
      // Functional approach
      let sum_of_squared_odd_numbers: u32 =
          (0..).map(|n| n * n)                             // All natural numbers squared
               .take_while(|&n_squared| n_squared < upper) // Below upper limit
               .filter(|&n_squared| is_odd(n_squared))     // That are odd
               .sum();                                     // Sum them
      println!("functional style: {}", sum_of_squared_odd_numbers);
  }
  ```

  

### Diverging functions

- （不知道咋翻译了）

- **Diverging fucntions** 永远不会返回。它们使用标记 `!`，这是意味着一个空类型。

  ```rust
  fn foo() -> ! {
      panic!("This call never returns.");
  }
  ```

- 与所有其他类型相反，这个类型不能被实例化，因为该类型的所有可能值的集合都是空的。注意，它不同于 `()` 类型，后者只有一个可能的值。例如，这个函数像往常一样返回，尽管返回值中没有任何信息：

  ```rust
  fn some_fn() {
      ()
  }
  
  fn main() {
      let a: () = some_fn();
      println!("This function returns and you can see this line.")
  }
  ```

- 与此函数相反，该函数永远不会将控件返回给调用者：

  ```rust
  #![feature(never_type)]
  
  fn main() {
      let x: ! = panic!("This call never returns.");
      println!("You will never see this line!");
  }
  ```

- 虽然这看起来像是一个抽象的概念，但实际上它是非常有用的，而且通常很方便。这种类型的主要优点是它可以转换为任何其他类型，因此可以在需要精确类型的地方使用，例如在匹配分支中。这允许我们写这样的代码：

  ```rust
  fn main() {
      fn sum_odd_numbers(up_to: u32) -> u32 {
          let mut acc = 0;
          for i in 0..up_to {
              // Notice that the return type of this match expression must be u32
              // because of the type of the "addition" variable.
              let addition: u32 = match i%2 == 1 {
                  // The "i" variable is of type u32, which is perfectly fine.
                  true => i,
                  // On the other hand, the "continue" expression does not return
                  // u32, but it is still fine, because it never returns and therefore
                  // does not violate the type requirements of the match expression.
                  false => continue,
              };
              acc += addition;
          }
          acc
      }
      println!("Sum of odd numbers up to 9 (excluding): {}", sum_odd_numbers(9));
  }
  ```

- 它也是永远循环的函数（例如： `loop {}` ）的返回类型，如网络服务器或者终止进程的函数（例如： `exit()` ）。