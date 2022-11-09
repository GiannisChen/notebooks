## RustCourse

-  参考：https://course.rs/
- 习题：https://github.com/sunface/rust-by-practice
- **RustCourse** 项目 **Github** 地址：https://github.com/sunface/rust-course



## 目录

| 序号 | 标题              | 连接                                     | 描述                                                         |
| ---- | ----------------- | ---------------------------------------- | ------------------------------------------------------------ |
| 1    | 基础内容          | [01-Primitives.md](./01-Primitives.md)   | 一些 **Rust** 的基础内容，包括 `Cargo` ，基本类型            |
| 2    | **Rust** 特殊之处 | [02-Specialness.md](./02-Specialness.md) | 一些 **Rust** 区别于其他语言的基础内容，主要是 **所有权系统** |
| 3    |                   |                                          |                                                              |



## TODO

- [ ] 给下列的代码找变量的地址：

  ```rust
  fn main() {
      let x = 5;
      let y = x;
      println!("x:{}, y:{}", x, y);
  }
  ```

- [ ] [调用方法需要引入特征](https://course.rs/basic/trait/trait.html#调用方法需要引入特征)

  在一些场景中，使用 `as` 关键字做类型转换会有比较大的限制，因为你想要在类型转换上拥有完全的控制，例如处理转换错误，那么你将需要 `TryInto`：

  ```rust
  use std::convert::TryInto;
  
  fn main() {
    let a: i32 = 10;
    let b: u16 = 100;
  
    let b_ = b.try_into()
              .unwrap();
  
    if a < b_ {
      println!("Ten is less than one hundred.");
    }
  }
  ```

  上面代码中引入了 `std::convert::TryInto` 特征，但是却没有使用它，可能有些同学会为此困惑，主要原因在于**如果你要使用一个特征的方法，那么你需要将该特征引入当前的作用域中**，我们在上面用到了 `try_into` 方法，因此需要引入对应的特征。

  但是 Rust 又提供了一个非常便利的办法，即把最常用的标准库中的特征通过 [`std::prelude`](https://course.rs/appendix/prelude.html) 模块提前引入到当前作用域中，其中包括了 `std::convert::TryInto`，你可以尝试删除第一行的代码 `use ...`，看看是否会报错。

- [ ] https://nomicon.purewhite.io/vec/vec.html Rust秘典（死灵书）

- [ ] 最后，如果你想要了解 `Vector` 更多的用法，请参见本书的标准库解析章节：[`Vector`常用方法](https://course.rs/std/vector.html)

- [ ] 最后，如果你想要了解 `HashMap` 更多的用法，请参见本书的标准库解析章节：[HashMap 常用方法](https://course.rs/std/hashmap.html)

- [ ] 
