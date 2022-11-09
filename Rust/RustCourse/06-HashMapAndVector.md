## 集合类型



### 动态数组（Vector）

- 动态数组类型用 `Vec<T>` 表示，事实上，在之前的章节，它的身影多次出现，我们一直没有细讲，只是简单的把它当作数组处理。
- 动态数组允许你存储多个值，这些值在内存中一个紧挨着另一个排列，因此访问其中某个元素的成本非常低。动态数组只能存储相同类型的元素，如果你想存储不同类型的元素，可以使用之前讲过的枚举类型或者特征对象。
- 总之，当我们想拥有一个列表，里面都是相同类型的数据时，动态数组将会非常有用。



#### 创建动态数组

- 在 Rust 中，有多种方式可以创建动态数组。

##### `Vec::new`

- 使用 `Vec::new` 创建动态数组是最 rusty 的方式，它调用了 `Vec` 中的 `new` 关联函数：

  ```rust
  let v: Vec<i32> = Vec::new();
  ```

  这里，`v` 被显式地声明了类型 `Vec<i32>`，这是因为 Rust 编译器无法从 `Vec::new()` 中得到任何关于类型的暗示信息，因此也无法推导出 `v` 的具体类型，但是当你向里面增加一个元素后，一切又不同了：

  ```rust
  let mut v = Vec::new();
  v.push(1);
  ```

  此时，`v` 就无需手动声明类型，因为编译器通过 `v.push(1)`，推测出 `v` 中的元素类型是 `i32`，因此推导出 `v` 的类型是 `Vec<i32>`。

> 如果预先知道要存储的元素个数，可以使用 `Vec::with_capacity(capacity)` 创建动态数组，这样**可以避免因为插入大量新数据导致频繁的内存分配和拷贝，提升性能**。

##### `vec![]`

- 还可以使用宏 `vec!` 来创建数组，与 `Vec::new` 有所不同，前者能在创建同时给予初始化值：

  ```rust
  let v = vec![1, 2, 3];
  ```

  同样，此处的 `v` 也无需标注类型，编译器只需检查它内部的元素即可自动推导出 `v` 的类型是 `Vec<i32>` （**Rust** 中，整数默认类型是 `i32`，在[数值类型](https://course.rs/basic/base-type/numbers.html#整数类型)中有详细介绍）。



#### 更新 Vector

- 向数组尾部添加元素，可以使用 `push` 方法：

  ```rust
  let mut v = Vec::new();
  v.push(1);
  ```

  与其它类型一样，**必须将 `v` 声明为 `mut` 后，才能进行修改**。



#### Vector 作用域

- 跟结构体一样，`Vector` 类型在超出作用域范围后，会被自动删除：

  ```rust
  {
      let v = vec![1, 2, 3];
  
      // ...
  } // <- v超出作用域并在此处被删除
  ```

  当 `Vector` 被删除后，它内部存储的所有内容也会随之被删除。目前来看，这种解决方案简单直白，但是当 `Vector` 中的元素被引用后，事情可能会没那么简单。



#### 从 Vector 中读取元素

- 读取指定位置的元素有两种方式可选：

  1. 通过下标索引访问；
  2. 使用 `get` 方法；

  ```rust
  let v = vec![1, 2, 3, 4, 5];
  
  let third: &i32 = &v[2];
  println!("第三个元素是 {}", third);
  
  match v.get(2) {
      Some(third) => println!("第三个元素是 {}", third),
      None => println!("去你的第三个元素，根本没有！"),
  }
  ```

  和其它语言一样，集合类型的索引下标都是从 `0` 开始，`&v[2]` 表示借用 `v` 中的第三个元素，最终会获得该元素的引用。而 `v.get(2)` 也是访问第三个元素，但是有所不同的是，它返回了 `Option<&T>`，因此还需要额外的 `match` 来匹配解构出具体的值。

- 由于 `i32` 底层就实现了 `Copy` 的特性，所以，以下这种方式也能运行，但是发生了复制，而非移动：

  ```rust
  let third: i32 = v[2];
  ```

  但是我们换一个没有实现 `Copy` 的自定义结构体：

  ```rust
  struct A(i32);
  
  impl Display for A {
      fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
          write!(f, "{}", self.0)
      }
  }
  
  fn main() {
      let v = vec![A(1), A(2), A(3), A(4), A(5)];
  
      //let third = v[2];	// raise error: cannot move out of index of `Vec<A>`
      let third = &v[2];
      println!("第三个元素是 {}", third);
      println!("第三个元素是 {}", v[2]);
  
      match v.get(2) {
          Some(third) => println!("第三个元素是 {}", third),
          None => println!("去你的第三个元素，根本没有！"),
      }
  }
  ```

  显然仅仅引用是可以的，但是移动就会报错，嗯，就很安全。

##### 下标索引与 .get 的区别

- 这两种方式都能成功的读取到指定的数组元素，既然如此为什么会存在两种方法？何况 `.get` 还会增加使用复杂度，这就涉及到**数组越界**的问题了，让我们通过示例说明：

  ```rust
  let v = vec![1, 2, 3, 4, 5];
  let does_not_exist = &v[100];
  let does_not_exist = v.get(100);
  ```

  运行以上代码，`&v[100]` 的访问方式会导致程序无情报错退出，因为发生了数组越界访问。 但是 `v.get` 就不会，它在内部做了处理，有值的时候返回 `Some(T)`，无值的时候返回 `None`，因此 `v.get` 的使用方式非常安全。

  既然如此，为何不统一使用 `v.get` 的形式？因为实在是有些啰嗦，**Rust** 语言的设计者和使用者在审美这方面还是相当统一的：简洁即正义，何况性能上也会有轻微的损耗。

  既然有两个选择，肯定就有如何选择的问题，答案很简单，当你确保索引不会越界的时候，就用索引访问，否则用 `.get`。例如，访问第几个数组元素并不取决于我们，而是取决于用户的输入时，用 `.get` 会非常适合，天知道那些可爱的用户会输入一个什么样的数字进来！



#### 同时借用多个数组元素

- 既然涉及到借用数组元素，那么很可能会遇到同时借用多个数组元素的情况，还记得在[所有权和借用](https://course.rs/basic/ownership/borrowing.html#借用规则总结)章节咱们讲过的借用规则嘛？如果记得，就来看看下面的代码 😀

  ```rust
  let mut v = vec![1, 2, 3, 4, 5];
  let first = &v[0];
  v.push(6);
  println!("The first element is: {}", first);
  ```

  首先 `first = &v[0]` 进行了不可变借用，`v.push` 进行了可变借用，如果 `first` 在 `v.push` 之后不再使用，那么该段代码可以成功编译（原因见[引用的作用域](https://course.rs/basic/ownership/borrowing.html#可变引用与不可变引用不能同时存在)）。可是上面的代码中，`first` 这个不可变借用在可变借用 `v.push` 后被使用了，那么妥妥的，编译器就会报错：

  ```shell
  $ cargo run
  Compiling collections v0.1.0 (file:///projects/collections)
  error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable 无法对v进行可变借用，因此之前已经进行了不可变借用
  --> src/main.rs:6:5
  |
  4 |     let first = &v[0];
  |                  - immutable borrow occurs here // 不可变借用发生在此处
  5 |
  6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here // 可变借用发生在此处
  7 |
  8 |     println!("The first element is: {}", first);
  |                                          ----- immutable borrow later used here // 不可变借用在这里被使用
  
  For more information about this error, try `rustc --explain E0502`.
  error: could not compile `collections` due to previous error
  ```

> - 其实，按理来说，这两个引用不应该互相影响的：一个是查询元素，一个是在数组尾部插入元素，完全不相干的操作，为何编译器要这么严格呢？
>
>   **原因在于**：**数组的大小是可变的，当旧数组的大小不够用时，Rust 会重新分配一块更大的内存空间，然后把旧数组拷贝过来。这种情况下，之前的引用显然会指向一块无效的内存**，这非常 rusty —— 对用户进行严格的教育。



#### 迭代遍历 Vector 中的元素

- 如果想要依次访问数组中的元素，可以使用迭代的方式去遍历数组，这种方式比用下标的方式去遍历数组更安全也更高效（**每次下标访问都会触发数组边界检查**）：

  ```rust
  let v = vec![1, 2, 3];
  for i in &v {
      println!("{}", i);
  }
  ```

  这里也有个坑，如果用 `v` 替代 `&v` 也是能偶编译的，但就变成了值的复制，内存安全真是无处不在啊（恼

- 也可以在迭代过程中，修改 `Vector` 中的元素：

  ```rust
  let mut v = vec![1, 2, 3];
  for i in &mut v {
      *i += 10
  }
  ```



#### 存储不同类型的元素

- 在本节开头，有讲到数组的元素必须类型相同，但是也提到了解决方案：那就是通过**使用枚举类型**和**特征对象**来实现不同类型元素的存储。

- 先来看看通过**枚举**如何实现：

  ```rust
  #[derive(Debug)]
  enum IpAddr { V4(String), V6(String) }
  fn main() {
      let v = vec![IpAddr::V4("127.0.0.1".to_string()), IpAddr::V6("::1".to_string())];
      for ip in v {
          show_addr(ip)
      }
  }
  
  fn show_addr(ip: IpAddr) {
      println!("{:?}",ip);
  }
  ```

  数组 `v` 中存储了两种不同的 `ip` 地址，但是这两种都属于 `IpAddr` 枚举类型的成员，因此可以存储在数组中。

- 再来看看**特征对象**的实现：

  ```rust
  trait IpAddr {
      fn display(&self);
  }
  
  struct V4(String);
  impl IpAddr for V4 {
      fn display(&self) {
          println!("ipv4: {:?}",self.0)
      }
  }
  struct V6(String);
  impl IpAddr for V6 {
      fn display(&self) {
          println!("ipv6: {:?}",self.0)
      }
  }
  
  fn main() {
      let v: Vec<Box<dyn IpAddr>> = vec![
          Box::new(V4("127.0.0.1".to_string())),
          Box::new(V6("::1".to_string())),
      ];
  
      for ip in v {
          ip.display();
      }
  }
  ```

  比枚举实现要稍微复杂一些，我们为 `V4` 和 `V6` 都实现了特征 `IpAddr`，然后将它俩的实例用 `Box::new` 包裹后，存在了数组 `v` 中，需要注意的是，这里必须**手动地指定类型**`Vec<Box<dyn IpAddr>>`，表示数组 `v` 存储的是特征 `IpAddr` 的对象，这样就实现了在数组中存储不同的类型。

> 在实际使用场景中，**特征对象数组要比枚举数组常见很多**，主要原因在于[特征对象](https://course.rs/basic/trait/trait-object.html)非常灵活，而编译器对枚举的限制较多，且无法动态增加类型。



### 哈希表（HashMap）

- 和动态数组一样，`HashMap` 也是 Rust 标准库中提供的集合类型，但是又与动态数组不同，`HashMap` 中存储的是一一映射的 `KV` 键值对，并提供了平均复杂度为 `O(1)` 的查询方法，当我们希望通过一个 `Key` 去查询值时，该类型非常有用，以致于 Go 语言将该类型设置成了语言级别的内置特性。



#### 创建 HashMap

- 跟创建动态数组 `Vec` 的方法类似，可以使用 `new` 方法来创建 `HashMap`，然后通过 `insert` 方法插入键值对。

##### 使用 new 方法创建

- 代码：

  ```rust
  use std::collections::HashMap;
  // 创建一个HashMap，用于存储宝石种类和对应的数量
  let mut my_gems = HashMap::new();
  // 将宝石类型和对应的数量写入表中
  my_gems.insert("红宝石", 1);
  my_gems.insert("蓝宝石", 2);
  my_gems.insert("河边捡的误以为是宝石的破石头", 18);
  ```

  和 `Vector` 一样的是， `HashMap::new` 并不需要指定其类型，而是在编译时通过第一个 `insert()` 来推导。

- 还有一点，你可能没有注意，那就是使用 `HashMap` 需要手动通过 `use ...` 从标准库中引入到我们当前的作用域中来，仔细回忆下，之前使用另外两个集合类型 `String` 和 `Vec` 时，我们是否有手动引用过？答案是 **No**，因为 `HashMap` 并没有包含在 Rust 的 [`prelude`](https://course.rs/appendix/prelude.html) 中（Rust 为了简化用户使用，提前将最常用的类型自动引入到作用域中）。

- 所有的集合类型都是动态的，意味着它们没有固定的内存大小，因此它们底层的数据都存储在内存堆上，然后通过一个存储在栈中的引用类型来访问。同时，跟其它集合类型一致，`HashMap` 也是内聚性的，即所有的 `K` 必须拥有同样的类型，`V` 也是如此。

> 跟 `Vec` 一样，如果预先知道要存储的 `KV` 对个数，可以使用 `HashMap::with_capacity(capacity)` 创建指定大小的 `HashMap`，避免频繁的内存分配和拷贝，提升性能。

##### 使用迭代器和 collect 方法创建

- 在实际使用中，不是所有的场景都能 `new` 一个哈希表后，然后悠哉悠哉的依次插入对应的键值对，而是可能会从另外一个数据结构中，获取到对应的数据，最终生成 `HashMap`。

- 例如考虑一个场景，有一张表格中记录了足球联赛中各队伍名称和积分的信息，这张表如果被导入到 Rust 项目中，一个合理的数据结构是 `Vec<(String, u32)>` 类型，该数组中的元素是一个个元组，该数据结构跟表格数据非常契合：表格中的数据都是逐行存储，每一个行都存有一个 `(team_name, scores)` 的信息。

  但是在很多时候，又需要通过队伍名称来查询对应的积分，此时动态数组就不适用了，因此可以用 `HashMap` 来保存相关的**队伍名称 -> 积分**映射关系。 理想很丰满，现实很骨感，如何将 `Vec<(String, u32)>` 中的数据快速写入到 `HashMap<String, u32>` 中？朴素的想法：

  ```rust
  fn main() {
      use std::collections::HashMap;
      let teams_list = vec![
          ("中国队".to_string(), 100),
          ("美国队".to_string(), 10),
          ("日本队".to_string(), 50),
      ];
      let mut teams_map = HashMap::new();
      for team in &teams_list {
          teams_map.insert(&team.0, team.1);
      }
      println!("{:?}",teams_map)
  }
  ```

  遍历列表，将每一个元组作为一对 `KV `插入到 `HashMap` 中，很简单，但是……也不太聪明的样子，换个词说就是 —— 不够 rusty。好在，**Rust** 为我们提供了一个非常精妙的解决办法：先将 `Vec` 转为迭代器，接着通过 `collect` 方法，将迭代器中的元素收集后，转成 `HashMap`：

  ```rust
  fn main() {
      use std::collections::HashMap;
      let teams_list = vec![
          ("中国队".to_string(), 100),
          ("美国队".to_string(), 10),
          ("日本队".to_string(), 50),
      ];
      let teams_map: HashMap<_,_> = teams_list.into_iter().collect();
      println!("{:?}",teams_map)
  }
  ```

  代码很简单，`into_iter` 方法将列表转为迭代器，接着通过 `collect` 进行收集，不过需要注意的是，`collect` 方法在内部实际上支持生成多种类型的目标集合，因此我们需要通过类型标注 `HashMap<_,_>` 来告诉编译器：请帮我们收集为 `HashMap` 集合类型，具体的 `KV` 类型，麻烦编译器您老人家帮我们推导。

  由此可见，**Rust** 中的编译器时而小聪明，时而大聪明，不过好在，它大聪明的时候，会自家人知道自己事，总归会通知你一声：

  ```shell
  error[E0282]: type annotations needed // 需要类型标注
    --> src/main.rs:10:9
     |
  10 |     let teams_map = teams_list.into_iter().collect();
     |         ^^^^^^^^^ consider giving `teams_map` a type // 给予 `teams_map` 一个具体的类型
  ```



#### 所有权的转移

- `HashMap` 的所有权规则与其它 **Rust** 类型没有区别：

  - 若类型实现 `Copy` 特征，该类型会被**复制**进 `HashMap`，因此无所谓所有权；
  - 若没实现 `Copy` 特征，所有权将被**转移**给 `HashMap` 中；

- 例如我参选帅气男孩时的场景再现：

  ```rust
  fn main() {
      use std::collections::HashMap;
  
      let name = String::from("Sunface");
      let age = 18;
  
      let mut handsome_boys = HashMap::new();
      handsome_boys.insert(name, age);
  
      println!("因为过于无耻，{}已经被从帅气男孩名单中除名", name);
      println!("还有，他的真实年龄远远不止{}岁", age);
  }
  ```

  运行代码，报错如下：

  ```console
  error[E0382]: borrow of moved value: `name`
    --> src/main.rs:10:32
     |
  4  |     let name = String::from("Sunface");
     |         ---- move occurs because `name` has type `String`, which does not implement the `Copy` trait
  ...
  8  |     handsome_boys.insert(name, age);
     |                          ---- value moved here
  9  |
  10 |     println!("因为过于无耻，{}已经被除名", name);
     |                                            ^^^^ value borrowed here after move
  ```

  提示很清晰，`name` 是 `String` 类型，因此它受到所有权的限制，在 `insert` 时，它的所有权被转移给 `handsome_boys`，所以最后在使用时，会遇到这个无情但是意料之中的报错。

- **如果你使用引用类型放入 HashMap 中**，请确保该引用的生命周期至少跟 `HashMap` 活得一样久：

  ```rust
  fn main() {
      use std::collections::HashMap;
  
      let name = String::from("Sunface");
      let age = 18;
  
      let mut handsome_boys = HashMap::new();
      handsome_boys.insert(&name, age);
  
      std::mem::drop(name);
      println!("因为过于无耻，{:?}已经被除名", handsome_boys);
      println!("还有，他的真实年龄远远不止{}岁", age);
  }
  ```

  上面代码，我们借用 `name` 获取了它的引用，然后插入到 `handsome_boys` 中，至此一切都很完美。但是紧接着，就通过 `drop` 函数手动将 `name` 字符串从内存中移除，再然后就报错了：

  ```shell
   handsome_boys.insert(&name, age);
     |                          ----- borrow of `name` occurs here // name借用发生在此处
  9  |
  10 |     std::mem::drop(name);
     |                    ^^^^ move out of `name` occurs here // name的所有权被转移走
  11 |     println!("因为过于无耻，{:?}已经被除名", handsome_boys);
     |                                              ------------- borrow later used here // 所有权转移后，还试图使用name
  ```

  最终，某人因为过于无耻，真正的被除名了 🙂



#### 查询 HashMap

- 通过 `get` 方法可以获取元素：

  ```rust
  use std::collections::HashMap;
  
  let mut scores = HashMap::new();
  
  scores.insert(String::from("Blue"), 10);
  scores.insert(String::from("Yellow"), 50);
  
  let team_name = String::from("Blue");
  let score: Option<&i32> = scores.get(&team_name);
  ```

  **上面有几点需要注意：**

  1. `get` 方法返回一个 `Option<&i32>` 类型：当查询不到时，会返回一个 `None`，查询到时返回 `Some(&i32)`；
  2. `&i32` 是对 `HashMap` 中**值的借用**，如果不使用借用，**可能会发生所有权的转移**；

- 还可以通过循环的方式依次遍历 `KV` 对：

  ```rust
  use std::collections::HashMap;
  
  let mut scores = HashMap::new();
  
  scores.insert(String::from("Blue"), 10);
  scores.insert(String::from("Yellow"), 50);
  
  for (key, value) in &scores {
      println!("{}: {}", key, value);
  }
  ```

  最终输出：

  ```shell
  Yellow: 50
  Blue: 10
  ```



#### 更新 HashMap 中的值

- 更新值的时候，涉及多种情况，咱们在代码中一一进行说明：

  ```rust
  use std::collections::HashMap;
  let mut scores = HashMap::new();
  scores.insert("Blue", 10);
  // 覆盖已有的值
  let old = scores.insert("Blue", 20);
  assert_eq!(old, Some(10));
  // 查询新插入的值
  let new = scores.get("Blue");
  assert_eq!(new, Some(&20));
  // 查询Yellow对应的值，若不存在则插入新值
  let v = scores.entry("Yellow").or_insert(5);
  assert_eq!(*v, 5); // 不存在，插入5
  // 查询Yellow对应的值，若不存在则插入新值
  let v = scores.entry("Yellow").or_insert(50);
  assert_eq!(*v, 5); // 已经存在，因此50没有插入
  ```

  具体的解释在代码注释中已有，这里不再进行赘述。

##### 在已有值得基础上更新

- 另一个常用场景如下：查询某个 `key` 对应的值，若不存在则插入新值，若存在则对已有的值进行更新，例如在文本中统计词语出现的次数：

  ```rust
  use std::collections::HashMap;
  let text = "hello world wonderful world";
  let mut map = HashMap::new();
  // 根据空格来切分字符串(英文单词都是通过空格切分)
  for word in text.split_whitespace() {
      let count = map.entry(word).or_insert(0);
      *count += 1;
  }
  println!("{:?}", map);
  ```

  上面代码中，新建一个 `map` 用于保存词语出现的次数，插入一个词语时会进行判断：若之前没有插入过，则使用该词语作 `Key`，插入次数 0 作为 `Value`，若之前插入过则取出之前统计的该词语出现的次数，对其加一。

  **有两点值得注意**：

  1. `or_insert` 返回了 `&mut v` 引用，因此可以通过该可变引用直接修改 `map` 中对应的值；
  2. 使用 `count` 引用时，需要先进行解引用 `*count`，否则会出现类型不匹配；



### 哈希函数

- 你肯定比较好奇，为何叫哈希表，到底什么是哈希。

  先来设想下，如果要实现 `Key` 与 `Value` 的一一对应，是不是意味着我们要能比较两个 `Key` 的相等性？例如 "a" 和 "b"，1 和 2，当这些类型做 `Key` 且能比较时，可以很容易知道 `1` 对应的值不会错误的映射到 `2` 上，因为 `1` 不等于 `2`。因此，一个类型能否作为 `Key` 的关键就是是否能进行相等比较，或者说该类型是否实现了 `std::cmp::Eq` 特征。

  > `f32` 和 `f64` 浮点数，没有实现 `std::cmp::Eq` 特征，因此不可以用作 `HashMap` 的 `Key`。

- 好了，理解完这个，再来设想一点，若一个复杂点的类型作为 `Key`，那怎么在底层对它进行存储，怎么使用它进行查询和比较？ 是不是很棘手？好在我们有哈希函数：通过它把 `Key` 计算后映射为哈希值，然后使用该哈希值来进行存储、查询、比较等操作。

  但是问题又来了，如何保证不同 `Key` 通过哈希后的两个值不会相同？如果相同，那意味着我们使用不同的 `Key`，却查到了同一个结果，这种明显是错误的行为。 此时，就涉及到安全性跟性能的取舍了。

  若要追求安全，尽可能减少冲突，同时防止拒绝服务（Denial of Service, DoS）攻击，就要使用密码学安全的哈希函数，`HashMap` 就是使用了这样的哈希函数。反之若要追求性能，就需要使用没有那么安全的算法。

##### 高性能三方库

- 因此若性能测试显示当前标准库默认的哈希函数不能满足你的性能需求，就需要去 [crates.io](https://crates.io/) 上寻找其它的哈希函数实现，使用方法很简单：

  ```rust
  use std::hash::BuildHasherDefault;
  use std::collections::HashMap;
  // 引入第三方的哈希函数
  use twox_hash::XxHash64;
  
  // 指定HashMap使用第三方的哈希函数XxHash64
  let mut hash: HashMap<_, _, BuildHasherDefault<XxHash64>> = Default::default();
  hash.insert(42, "the answer");
  assert_eq!(hash.get(&42), Some(&"the answer"));
  ```

> - 目前，`HashMap` 使用的哈希函数是 `SipHash`，它的性能不是很高，但是安全性很高。`SipHash` 在中等大小的 `Key` 上，性能相当不错，但是对于小型的 `Key` （例如整数）或者大型 `Key` （例如字符串）来说，性能还是不够好。若你需要极致性能，例如实现算法，可以考虑这个库：[ahash](https://github.com/tkaitchuck/ahash)
>
>   ```rust
>   fn main() {
>       use ahash::AHashMap;
>   
>       let mut map: AHashMap<i32, i32> = AHashMap::new();
>       map.insert(12, 34);
>       map.insert(56, 78);
>       let v12 = map.entry(12).or_insert(0);
>       *v12 += 23;
>   
>       match map.get(&12) {
>           Some(i) => println!("12: {}", i),
>           None => println!("12: None"),
>       }
>   }
>   ```