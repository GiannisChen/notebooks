## 函数式编程（Functional Programming）

- 函数式语言的优秀特性：
  1. 使用函数作为参数进行传递；
  2. 使用函数作为函数返回值；
  3. 将函数赋值给变量；
- 函数式特性：
  1. **闭包（Closure）**
  2. **迭代器（Iterator）**
  3. 模式匹配
  4. 枚举

> **这些函数式特性可以让代码的可读性和易写性大幅提升**。



### 闭包（Closure）

闭包这个词语由来已久，自上世纪 60 年代就由 `Scheme` 语言引进之后，被广泛用于函数式编程语言中，进入 21 世纪后，各种现代化的编程语言也都不约而同地把闭包作为核心特性纳入到语言设计中来。那么到底何为闭包？

**闭包**是**一种匿名函数，它可以赋值给变量也可以作为参数传递给其它函数，不同于函数的是，它允许捕获调用者作用域中的值**，例如：

```rust
fn main() {
   let x = 1;
   let sum = |y| x + y;

    assert_eq!(3, sum(2));
}
```

上面的代码展示了非常简单的闭包 `sum`，它拥有一个入参 `y`，同时捕获了作用域中的 `x` 的值，因此调用 `sum(2)` 意味着将 `2`（参数 `y`）跟 `1`（`x`）进行相加，最终返回它们的和：`3`。

可以看到 `sum` 非常符合闭包的定义：**可以赋值给变量，允许捕获调用者作用域中的值**。



#### 使用闭包简化代码

##### 传统函数实现

想象一下，我们要进行健身，用代码怎么实现（写代码什么鬼，健身难道不应该去健身房嘛？答曰：健身太累了，还是虚拟健身好，点到为止）？这里是我的想法：

```rust
use std::thread;
use std::time::Duration;

// 开始健身，好累，我得发出声音：muuuu...
fn muuuuu(intensity: u32) -> u32 {
    println!("muuuu.....");
    thread::sleep(Duration::from_secs(2));
    intensity
}

fn workout(intensity: u32, random_number: u32) {
    if intensity < 25 {
        println!(
            "今天活力满满，先做 {} 个俯卧撑!",
            muuuuu(intensity)
        );
        println!(
            "旁边有妹子在看，俯卧撑太low，再来 {} 组卧推!",
            muuuuu(intensity)
        );
    } else if random_number == 3 {
        println!("昨天练过度了，今天还是休息下吧！");
    } else {
        println!(
            "昨天练过度了，今天干干有氧，跑步 {} 分钟!",
            muuuuu(intensity)
        );
    }
}

fn main() {
    // 强度
    let intensity = 10;
    // 随机值用来决定某个选择
    let random_number = 7;

    // 开始健身
    workout(intensity, random_number);
}
```

可以看到，在健身时我们根据想要的强度来调整具体的动作，然后调用 `muuuuu` 函数来开始健身。这个程序本身很简单，没啥好说的，但是假如未来不用 `muuuuu` 函数了，是不是得把所有 `muuuuu` 都替换成，比如说 `woooo` ？如果 `muuuuu` 出现了几十次，那意味着我们要**修改几十处地方**。

##### 函数变量实现

一个可行的办法是，把函数赋值给一个变量，然后通过变量调用：

```rust
use std::thread;
use std::time::Duration;

// 开始健身，好累，我得发出声音：muuuu...
fn muuuuu(intensity: u32) -> u32 {
    println!("muuuu.....");
    thread::sleep(Duration::from_secs(2));
    intensity
}

fn workout(intensity: u32, random_number: u32) {
    let action = muuuuu;
    if intensity < 25 {
        println!(
            "今天活力满满, 先做 {} 个俯卧撑!",
            action(intensity)
        );
        println!(
            "旁边有妹子在看，俯卧撑太low, 再来 {} 组卧推!",
            action(intensity)
        );
    } else if random_number == 3 {
        println!("昨天练过度了，今天还是休息下吧！");
    } else {
        println!(
            "昨天练过度了，今天干干有氧, 跑步 {} 分钟!",
            action(intensity)
        );
    }
}

fn main() {
    // 强度
    let intensity = 10;
    // 随机值用来决定某个选择
    let random_number = 7;

    // 开始健身
    workout(intensity, random_number);
}
```

经过上面修改后，所有的调用都通过 `action` 来完成，若未来声(动)音(作)变了，只要修改为 `let action = woooo` 即可。

但是问题又来了，若 `intensity` 也变了怎么办？例如变成 `action(intensity + 1)`，那你又得哐哐哐修改几十处调用。该怎么办？没太好的办法了，只能祭出大杀器：**闭包**。

##### 闭包实现

上面提到 `intensity` 要是变化怎么办，简单，使用闭包来捕获它，这是我们的拿手好戏：

```rust
use std::thread;
use std::time::Duration;

fn workout(intensity: u32, random_number: u32) {
    let action = || {
        println!("muuuu.....");
        thread::sleep(Duration::from_secs(2));
        return intensity;
    };

    if intensity < 25 {
        println!("今天活力满满，先做 {} 个俯卧撑!", action());
        println!("旁边有妹子在看，俯卧撑太low，再来 {} 组卧推!", action());
    } else if random_number == 3 {
        println!("昨天练过度了，今天还是休息下吧！");
    } else {
        println!("昨天练过度了，今天干干有氧，跑步 {} 分钟!", action());
    }
}

fn main() {
    // 动作次数
    let intensity = 10;
    // 随机值用来决定某个选择
    let random_number = 7;

    // 开始健身
    workout(intensity, random_number);
}
```

在上面代码中，无论你要修改什么，只要修改闭包 `action` 的实现即可，其它地方只负责调用，完美解决了我们的问题！

Rust 闭包在形式上借鉴了 `Smalltalk` 和 `Ruby` 语言，与函数最大的不同就是它的参数是通过 `|parm1|` 的形式进行声明，如果是多个参数就 `|param1, param2,...|`， 下面给出闭包的形式定义：

```rust
|param1, param2,...| {
    语句1;
    语句2;
    返回表达式
}
```

如果只有一个返回表达式的话，定义可以简化为：

```rust
|param1| 返回表达式
```

上例中还有两点值得注意：

1. **闭包中最后一行表达式返回的值，就是闭包执行后的返回值**，因此 `action()` 调用返回了 `intensity` 的值 `10`；
2. `let action = ||...` 只是把闭包赋值给变量 `action`，并不是把闭包执行后的结果赋值给 `action`，因此这里 `action` 就相当于闭包函数，可以跟函数一样进行调用：`action()`；



#### 闭包的类型推导

**Rust** 是**静态语言**，因此所有的变量都具有类型，但是得益于编译器的强大类型推导能力，在很多时候我们并不需要显式地去声明类型，但是显然函数并不在此列，**必须手动为函数的所有参数和返回值指定类型**，原因在于函数往往会作为 API 提供给你的用户，因此你的用户必须在使用时知道传入参数的类型和返回值类型。

与函数相反，闭包并不会作为 API 对外提供，因此它**可以享受编译器的类型推导能力**，无需标注参数和返回值的类型。

为了增加代码可读性，有时候我们会显式地给类型进行标注，出于同样的目的，也可以给闭包标注类型：

```rust
let sum = |x: i32, y: i32| -> i32 { x + y }
```

与之相比，不标注类型的闭包声明会更简洁些：`let sum = |x, y| x + y`，需要注意的是，针对 `sum` 闭包，如果你只进行了声明，但是没有使用，编译器会提示你为 `x, y` 添加类型标注，因为它缺乏必要的上下文：

```rust
let sum  = |x, y| x + y;
let v = sum(1, 2);
```

这里我们使用了 `sum`，同时把 `1` 传给了 `x`，`2` 传给了 `y`，因此编译器才可以推导出 `x,y` 的类型为 `i32`。

下面展示了同一个功能的函数和闭包实现形式：

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

可以看出第一行的函数和后面的闭包其实在形式上是非常接近的，同时三种不同的闭包也展示了三种不同的使用方式：省略参数、返回值类型和花括号对。

虽然类型推导很好用，但是它**不是泛型**，**当编译器推导出一种类型后，它就会一直使用该类型**：

```rust
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

首先，在 `s` 中，编译器为 `x` 推导出类型 `String`，但是紧接着 `n` 试图用 `5` 这个整型去调用闭包，跟编译器之前推导的 `String` 类型不符，因此报错：

```shell
error[E0308]: mismatched types
 --> src/main.rs:5:29
  |
5 |     let n = example_closure(5);
  |                             ^
  |                             |
  |                             expected struct `String`, found integer // 期待String类型，却发现一个整数
  |                             help: try using a conversion method: `5.to_string()`
```



#### 结构体中的闭包

假设我们要实现一个简易缓存，功能是获取一个值，然后将其缓存起来，那么可以这样设计：

- 一个闭包用于获取值
- 一个变量，用于存储该值

可以使用结构体来代表缓存对象，最终设计如下：

```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    query: T,
    value: Option<u32>,
}
```

等等，我都跟着这本教程学完 Rust 基础了，为何还有我不认识的东东？`Fn(u32) -> u32` 是什么鬼？别急，先回答你第一个问题：骚年，too young too naive，你以为 Rust 的语法特性就基础入门那一些吗？太年轻了！如果是长征，你才刚到赤水河。

其实，可以看得出这一长串是 `T` 的特征约束，再结合之前的已知信息：`query` 是一个闭包，大概可以推测出，`Fn(u32) -> u32` 是一个特征，用来表示 `T` 是一个闭包类型？Bingo，恭喜你，答对了！

那为什么不用具体的类型来标注 `query` 呢？原因很简单，每一个闭包实例都有独属于自己的类型，即使于两个签名一模一样的闭包，它们的类型也是不同的，因此你无法用一个统一的类型来标注 `query` 闭包。

而标准库提供的 `Fn` 系列特征，再结合特征约束，就能很好的解决了这个问题. `T: Fn(u32) -> u32` 意味着 `query` 的类型是 `T`，该类型必须实现了相应的闭包特征 `Fn(u32) -> u32`。从特征的角度来看它长得非常反直觉，但是如果从闭包的角度来看又极其符合直觉，不得不佩服 **Rust** 团队的鬼才设计。。。

特征 `Fn(u32) -> u32` 从表面来看，就对闭包形式进行了显而易见的限制：**该闭包拥有一个`u32`类型的参数，同时返回一个`u32`类型的值**。

> 需要注意的是，其实 `Fn` 特征不仅仅适用于闭包，还适用于函数，因此上面的 `query` 字段除了使用闭包作为值外，还能使用一个具名的函数来作为它的值

接着，为缓存实现方法：

```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    query: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(query: T) -> Cacher<T> {
        return Cacher {
            query: query,
            value: None,
        };
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => return v,
            None => {
                let v = (self.query)(arg);
                self.value = Some(v);
                return v;
            }
        }
    }
}

fn main() {
    let mut cache = Cacher::new(|x| x * 2);
    println!("{}", cache.value(1));
    println!("{}", cache.value(2));
}
```

上面的缓存有一个很大的问题：只支持 `u32` 类型的值，若我们想要缓存 `&str` 类型，显然就行不通了，因此需要将 `u32` 替换成泛型 `E`：

```rust
use std::fmt::Display;

struct Cacher<T, E>
where
    T: Fn(E) -> E,
    E: Display + Clone,
{
    query: T,
    value: Option<E>,
}

impl<T, E> Cacher<T, E>
where
    T: Fn(E) -> E,
    E: Display + Clone,
{
    fn new(query: T) -> Cacher<T, E> {
        return Cacher {
            query: query,
            value: None,
        };
    }

    fn value(&mut self, arg: E) -> E {
        match self.value.clone() {
            Some(v) => return v,
            None => {
                let v = (self.query)(arg);
                self.value = Some(v.clone());
                return v;
            }
        }
    }
}

fn main() {
    let mut cache = Cacher::new(|x| x);
    println!("{}", cache.value("1"));
    println!("{}", cache.value("2"));
}
```

> （也许这么干？）但是很丑哎~ 😭😭



#### 捕获作用域中的值

在之前代码中，我们一直在用闭包的匿名函数特性（赋值给变量），然而闭包还拥有一项函数所不具备的特性：捕获作用域中的值。

```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
```

上面代码中，`x` 并不是闭包 `equal_to_x` 的参数，但是它依然可以去使用 `x`，因为 `equal_to_x` 在 `x` 的作用域范围内。

对于函数来说，就算你把函数定义在 `main` 函数体中，它也不能访问 `x`：

```rust
fn main() {
    let x = 4;

    fn equal_to_x(z: i32) -> bool {
        z == x
    }

    let y = 4;

    assert!(equal_to_x(y));
}
```

报错如下：

```shell
error[E0434]: can't capture dynamic environment in a fn item // 在函数中无法捕获动态的环境
 --> src/main.rs:5:14
  |
5 |         z == x
  |              ^
  |
  = help: use the `|| { ... }` closure form instead // 使用闭包替代
```

如上所示，编译器准确地告诉了我们错误，甚至同时给出了**提示**：

> 使用闭包来替代函数，这种聪明令我有些无所适从，总感觉会显得**我很笨**。

> **闭包对内存的影响**
>
> - 闭包不总是万能的：当闭包从环境中捕获一个值时，会**分配内存去存储这些值**。对于有些场景来说，这种额外的内存分配会成为一种负担。与之相比，函数就不会去捕获这些环境值，因此定义和使用函数不会拥有这种内存负担。



##### 三种 Fn 特征

闭包捕获变量有三种途径，恰好对应函数参数的三种传入方式：转移所有权、可变借用、不可变借用，因此相应的 `Fn` 特征也有三种：（ `FnOnce` 、 `FnMut` 、 `Fn` ）

###### FnOnce

该类型的闭包会拿走（`move`）被捕获变量的所有权。`Once` 顾名思义，说明**该闭包只能运行一次**：

```rust
fn fn_once<F>(func: F)
where
    F: FnOnce(usize) -> bool,
{
    println!("{}", func(3));
    println!("{}", func(4));
}

fn main() {
    let x = vec![1, 2, 3];
    fn_once(|z|{z == x.len()})
}
```

**仅**实现 `FnOnce` 特征的闭包在调用时会转移所有权，所以显然不能对已失去所有权的闭包变量进行二次调用：

```shell
error[E0382]: use of moved value: `func`
 --> src\main.rs:6:20
  |
1 | fn fn_once<F>(func: F)
  |               ---- move occurs because `func` has type `F`, which does not implement the `Copy` trait
                  // 因为`func`的类型是没有实现`Copy`特性的 `F`，所以发生了所有权的转移
...
5 |     println!("{}", func(3));
  |                    ------- `func` moved due to this call // 转移在这
6 |     println!("{}", func(4));
  |                    ^^^^ value used here after move // 转移后再次用
  |
```

这里面有一个很重要的提示，因为 `F` 没有实现 `Copy` 特征，所以会报错，那么我们添加一个约束，试试实现了 `Copy` 的闭包：

```rust
fn fn_once<F>(func: F)
where
    F: FnOnce(usize) -> bool + Copy,// 改动在这里
{
    println!("{}", func(3));
    println!("{}", func(4));
}

fn main() {
    let x = vec![1, 2, 3];
    fn_once(|z|{z == x.len()})
}
```

上面代码中，`func` 的类型 `F` 实现了 `Copy` 特征，调用时使用的将是它的拷贝，所以并没有发生所有权的转移。

```console
true
false
```

**如果你想强制闭包取得捕获变量的所有权**，可以在参数列表前添加 `move` 关键字，这种用法通常用于闭包的生命周期大于捕获变量的生命周期时，例如将闭包返回或移入其他线程。

```rust
use std::thread;
let v = vec![1, 2, 3];
let handle = thread::spawn(move || {
    println!("Here's a vector: {:?}", v);
});
handle.join().unwrap();
```

```rust
fn fn_once<F>(func: F)
where
    F: FnOnce(usize) -> bool + Copy,
{
    println!("{}", func(1));
    println!("{}", func(4));
}

fn main() {
    let x = 1;
    fn_once(move |z| z == x)
}
```



###### FnMut

它**以可变借用的方式捕获了环境中的值**，因此可以修改该值：

```rust
fn main() {
    let mut s = String::new();

    let update_string =  |str| s.push_str(str);
    update_string("hello");

    println!("{:?}",s);
}
```

在闭包中，我们调用 `s.push_str` 去改变外部 `s` 的字符串值，因此这里捕获了它的可变借用，运行下试试：

```shell
error[E0596]: cannot borrow `update_string` as mutable, as it is not declared as mutable
 --> src/main.rs:5:5
  |
4 |     let update_string =  |str| s.push_str(str);
  |         -------------          - calling `update_string` requires mutable binding due to mutable borrow of `s`
  |         |
  |         help: consider changing this to be mutable: `mut update_string`
5 |     update_string("hello");
  |     ^^^^^^^^^^^^^ cannot borrow as mutable
```

虽然报错了，但是编译器给出了非常清晰的提示，想要在闭包内部捕获可变借用，需要把该闭包声明为可变类型，也就是 `update_string` 要修改为 `mut update_string`：

```rust
fn main() {
    let mut s = String::new();

    let mut update_string =  |str| s.push_str(str);
    update_string("hello");

    println!("{:?}",s);
}
```

这种写法有点反直觉，相比起来前面的 `move` 更符合使用和阅读习惯。但是如果你忽略 `update_string` 的类型，仅仅把它当成一个普通变量，那么这种声明就比较合理了。

```rust
fn fn_once<F>(mut func: F)
where
    F: FnMut(i32, i32) -> bool,
{
    println!("{}", func(1, 2));
    println!("{}", func(2, 2));
}

fn main() {
    let mut x = 1;
    println!("{}", x);
    fn_once(|a, b| { x = a; b == x });
    println!("{}", x);
}
```

这种写法有点反直觉，相比起来前面的 `move` 更符合使用和阅读习惯。但是如果你忽略 `update_string` 的类型，仅仅把它当成一个普通变量，那么这种声明就比较合理了。再来看一个复杂点的：

```rust
fn main() {
    let mut s = String::new();

    let update_string =  |str| s.push_str(str);

    exec(update_string);

    println!("{:?}",s);
}

fn exec<'a, F: FnMut(&'a str)>(mut f: F)  {
    f("hello")
}
```

这段代码非常清晰的说明了 `update_string` 实现了 `FnMut` 特征。

###### Fn

它以不可变借用的方式捕获环境中的值 让我们把上面的代码中 `exec` 的 `F` 泛型参数类型修改为 `Fn(&'a str)`：

```rust
fn main() {
    let mut s = String::new();

    let update_string =  |str| s.push_str(str);

    exec(update_string);

    println!("{:?}",s);
}

fn exec<'a, F: Fn(&'a str)>(mut f: F)  {
    f("hello")
}
```

然后运行看看结果：

```shell
error[E0525]: expected a closure that implements the `Fn` trait, but this closure only implements `FnMut`
 --> src/main.rs:4:26  // 期望闭包实现的是`Fn`特征，但是它只实现了`FnMut`特征
  |
4 |     let update_string =  |str| s.push_str(str);
  |                          ^^^^^^-^^^^^^^^^^^^^^
  |                          |     |
  |                          |     closure is `FnMut` because it mutates the variable `s` here
  |                          this closure implements `FnMut`, not `Fn` //闭包实现的是FnMut，而不是Fn
5 |
6 |     exec(update_string);
  |     ---- the requirement to implement `Fn` derives from here
```

从报错中很清晰的看出，我们的闭包实现的是 `FnMut` 特征，需要的是可变借用，但是在 `exec` 中却给它标注了 `Fn` 特征，因此产生了不匹配，再来看看正确的不可变借用方式：

```rust
fn main() {
    let s = "hello, ".to_string();

    let update_string =  |str| println!("{},{}",s,str);

    exec(update_string);

    println!("{:?}",s);
}

fn exec<'a, F: Fn(String) -> ()>(f: F)  {
    f("world".to_string())
}
```

在这里，因为无需改变 `s`，因此闭包中只对 `s` 进行了不可变借用，那么在 `exec` 中，将其标记为 `Fn` 特征就完全正确。

###### move 和 Fn

在上面，我们讲到了 `move` 关键字对于 `FnOnce` 特征的重要性，但是实际上使用了 `move` 的闭包依然可能实现了 `Fn` 或 `FnMut` 特征。

因为，**一个闭包实现了哪种 Fn 特征取决于该闭包如何使用被捕获的变量，而不是取决于闭包如何捕获它们**。`move` 本身强调的就是后者，闭包如何捕获变量：

我们在上面的闭包中使用了 `move` 关键字，所以我们的闭包捕获了它，但是由于闭包对 `s` 的使用仅仅是不可变借用，因此该闭包实际上**还**实现了 `Fn` 特征。

细心的读者肯定发现我在上段中使用了一个 `还` 字，这是什么意思呢？因为该闭包不仅仅实现了 `FnOnce` 特征，还实现了 `Fn` 特征，将代码修改成下面这样，依然可以编译：

```rust
fn main() {
    let s = String::new();

    let update_string =  move || println!("{}",s);

    exec(update_string);
}

fn exec<F: Fn()>(f: F)  {
    f()
}
```

###### 三种 Fn 的关系

实际上，一个闭包并不仅仅实现某一种 `Fn` 特征，规则如下：

1. 所有的闭包都自动实现了 `FnOnce` 特征，因此任何一个闭包都至少可以被调用一次；
2. 没有移出所捕获变量的所有权的闭包自动实现了 `FnMut` 特征；
3. 不需要对捕获变量进行改变的闭包自动实现了 `Fn` 特征；

用一段代码来简单诠释上述规则：

```rust
fn main() {
    let s = String::new();

    let update_string =  || println!("{}",s);

    exec(update_string);
    exec1(update_string);
    exec2(update_string);
}

fn exec<F: FnOnce()>(f: F)  {
    f()
}

fn exec1<F: FnMut()>(mut f: F)  {
    f()
}

fn exec2<F: Fn()>(f: F)  {
    f()
}
```

虽然，闭包只是对 `s` 进行了不可变借用，实际上，它可以适用于任何一种 `Fn` 特征：三个 `exec` 函数说明了一切。强烈建议读者亲自动手试试各种情况下使用的 `Fn` 特征，更有助于加深这方面的理解。

关于第二条规则，有如下示例：

```rust
fn main() {
    let mut s = String::new();

    let update_string = |str| -> String {s.push_str(str); s };

    exec(update_string);
}

fn exec<'a, F: FnMut(&'a str) -> String>(mut f: F) {
    f("hello");
}
```

```shell
5 |     let update_string = |str| -> String {s.push_str(str); s };
  |                         ^^^^^^^^^^^^^^^                   - closure is `FnOnce` because it moves the variable `s` out of its environment
  |                                                           // 闭包实现了`FnOnce`，因为它从捕获环境中移出了变量`s`
  |                         |
  |                         this closure implements `FnOnce`, not `FnMut`
```

此例中，闭包从捕获环境中移出了变量 `s` 的所有权，因此这个闭包仅自动实现了 `FnOnce`，未实现 `FnMut` 和 `Fn`。再次印证之前讲的**一个闭包实现了哪种 Fn 特征取决于该闭包如何使用被捕获的变量，而不是取决于闭包如何捕获它们**，跟是否使用 `move` 没有必然联系。如果还是有疑惑？没关系，我们来看看这三个特征的简化版源码：

```rust
pub trait Fn<Args> : FnMut<Args> {
    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
}

pub trait FnMut<Args> : FnOnce<Args> {
    extern "rust-call" fn call_mut(&mut self, args: Args) -> Self::Output;
}

pub trait FnOnce<Args> {
    type Output;

    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
}
```

看到没？从特征约束能看出来 `Fn` 的前提是实现 `FnMut`，`FnMut` 的前提是实现 `FnOnce`，因此要实现 `Fn` 就要同时实现 `FnMut` 和 `FnOnce`，这段源码从侧面印证了之前规则的正确性。

从源码中还能看出一点：`Fn` 获取 `&self`，`FnMut` 获取 `&mut self`，而 `FnOnce` 获取 `self`。 在实际项目中，**建议先使用 `Fn` 特征**，然后编译器会告诉你正误以及该如何选择。



#### 闭包作为函数值返回

看到这里，相信大家对于如何使用闭包作为函数参数，已经很熟悉了，但是如果要使用闭包作为函数返回值，该如何做？先来看一段代码：

```rust
fn factory() -> Fn(i32) -> i32 {
    let num = 5;

    |x| x + num
}

let f = factory();

let answer = f(1);
assert_eq!(6, answer);
```

上面这段代码看起来还是蛮正常的，用 `Fn(i32) -> i32` 特征来代表 `|x| x + num`，非常合理嘛，肯定可以编译通过, 可惜理想总是难以照进现实，编译器给我们报了一大堆错误，先挑几个重点来看看：

```shell
fn factory<T>() -> Fn(i32) -> i32 {
  |                    ^^^^^^^^^^^^^^ doesn't have a size known at compile-time // 该类型在编译器没有固定的大小
```

**Rust** 要求函数的参数和返回类型，必须有固定的内存大小，例如 `i32` 就是 4 个字节，引用类型是 8 个字节，总之，绝大部分类型都有固定的大小，但是不包括特征，因为特征类似接口，对于编译器来说，无法知道它后面藏的真实类型是什么，因为也无法得知具体的大小。

同样，我们也无法知道闭包的具体类型，该怎么办呢？再看看报错提示：

```shell
help: use `impl Fn(i32) -> i32` as the return type, as all return paths are of type `[closure@src/main.rs:11:5: 11:21]`, which implements `Fn(i32) -> i32`
  |
8 | fn factory<T>() -> impl Fn(i32) -> i32 {
```

嗯，编译器提示我们加一个 `impl` 关键字，哦，这样一说，读者可能就想起来了，`impl Trait` 可以用来返回一个实现了指定特征的类型，那么这里 `impl Fn(i32) -> i32` 的返回值形式，说明我们要返回一个闭包类型，它实现了 `Fn(i32) -> i32` 特征。

完美解决，但是，在[特征](https://course.rs/basic/trait/trait.html)那一章，我们提到过，`impl Trait` 的返回方式有一个非常大的局限，就是你只能返回同样的类型，例如：

```rust
fn factory(x:i32) -> impl Fn(i32) -> i32 {

    let num = 5;

    if x > 1{
        move |x| x + num
    } else {
        move |x| x - num
    }
}
```

运行后，编译器报错：

```console
error[E0308]: `if` and `else` have incompatible types
  --> src/main.rs:15:9
   |
12 | /     if x > 1{
13 | |         move |x| x + num
   | |         ---------------- expected because of this
14 | |     } else {
15 | |         move |x| x - num
   | |         ^^^^^^^^^^^^^^^^ expected closure, found a different closure
16 | |     }
   | |_____- `if` and `else` have incompatible types
   |
```

嗯，提示很清晰：`if` 和 `else` 分支中返回了不同的闭包类型，这就很奇怪了，明明这两个闭包长的一样的，好在细心的读者应该回想起来，本章节前面咱们有提到：就算签名一样的闭包，类型也是不同的，因此在这种情况下，就无法再使用 `impl Trait` 的方式去返回闭包。

怎么办？再看看编译器提示，里面有这样一行小字：

```shell
= help: consider boxing your closure and/or using it as a trait object
```

哦，相信你已经恍然大悟，可以用特征对象！只需要用 `Box` 的方式即可实现：

```rust
fn factory(x:i32) -> Box<dyn Fn(i32) -> i32> {
    let num = 5;

    if x > 1{
        Box::new(move |x| x + num)
    } else {
        Box::new(move |x| x - num)
    }
}
```

至此，闭包作为函数返回值就已完美解决，若以后你再遇到报错时，一定要仔细阅读编译器的提示，很多时候，转角都能遇到爱。



### 迭代器（Iterator）

如果你询问一个 **Rust** 资深开发：写 Rust 项目最需要掌握什么？相信迭代器往往就是答案之一。无论你是编程新手亦或是高手，实际上大概率都用过迭代器，虽然自己可能并没有意识到这一点:)

迭代器允许我们迭代一个连续的集合，例如数组、动态数组 `Vec`、`HashMap` 等，在此过程中，只需关心集合中的元素如何处理，而无需关心如何开始、如何结束、按照什么样的索引去访问等问题。

**Rust** 中的 `for`：

```rust
let arr = [1, 2, 3];
for v in arr {
    println!("{}",v);
}
```

**Rust** 中没有使用索引，它把 `arr` 数组当成一个迭代器，直接去遍历其中的元素，从哪里开始，从哪里结束，都无需操心。因此严格来说，**Rust** 中的 `for` 循环是编译器提供的语法糖，最终还是对迭代器中的元素进行遍历。

那又有同学要发问了，在 **Rust** 中数组是迭代器吗？因为在之前的代码中直接对数组 `arr` 进行了迭代，答案是 `No`。那既然数组不是迭代器，为啥咱可以对它的元素进行迭代呢？

简而言之就是数组实现了 `IntoIterator` 特征，Rust 通过 `for` 语法糖，自动把实现了该特征的数组类型转换为迭代器（你也可以为自己的集合类型实现此特征），最终让我们可以直接对一个数组进行迭代，类似的还有：

```rust
for i in 1..10 {
    println!("{}", i);
}
```

直接对数值序列进行迭代，也是很常见的使用方式。

`IntoIterator` 特征拥有一个 `into_iter` 方法，因此我们还可以显式的把数组转换成迭代器：

```rust
let arr = [1, 2, 3];
for v in arr.into_iter() {
    println!("{}", v);
}
```

迭代器是函数语言的核心特性，它赋予了 **Rust** 远超于循环的强大表达能力，我们将在本章中一一为大家进行展现。



#### 惰性初始化

在 Rust 中，迭代器是惰性的，意味着如果你不使用它，那么它将不会发生任何事：

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("{}", val);
}
```

在 `for` 循环之前，我们只是简单的创建了一个迭代器 `v1_iter`，此时不会发生任何迭代行为，只有在 `for` 循环开始后，迭代器才会开始迭代其中的元素，最后打印出来。

这种惰性初始化的方式确保了创建迭代器不会有任何额外的性能损耗，其中的元素也不会被消耗，只有使用到该迭代器的时候，一切才开始。



#### next 方法

对于 `for` 如何遍历迭代器，还有一个问题，它如何取出迭代器中的元素？

先来看一个特征：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 省略其余有默认实现的方法
}
```

呦，该特征竟然和迭代器 `iterator` 同名，难不成。。。没错，它们就是有一腿。**迭代器之所以成为迭代器，就是因为实现了 `Iterator` 特征**，要实现该特征，最主要的就是实现其中的 `next` 方法，该方法控制如何从集合中取值，最终返回值的类型是[关联类型](https://course.rs/basic/trait/advance-trait#关联类型) `Item`。

因此，之前问题的答案已经很明显：`for` 循环通过不停调用迭代器上的 `next` 方法，来获取迭代器中的元素。

既然 `for` 可以调用 `next` 方法，是不是意味着我们也可以？来试试：

```rust
fn main() {
    let arr = [1, 2, 3];
    let mut arr_iter = arr.into_iter();

    assert_eq!(arr_iter.next(), Some(1));
    assert_eq!(arr_iter.next(), Some(2));
    assert_eq!(arr_iter.next(), Some(3));
    assert_eq!(arr_iter.next(), None);
}
```

果不其然，将 `arr` 转换成迭代器后，通过调用其上的 `next` 方法，我们获取了 `arr` 中的元素，有三点需要注意：

1. `next` 方法返回的是 `Option` 类型，当有值时返回 `Some(i32)`，无值时返回 `None` ；
2. 遍历是按照迭代器中元素的排列顺序依次进行的，因此我们严格按照数组中元素的顺序取出了 `Some(1)`，`Some(2)`，`Some(3)` ；
3. 手动迭代必须将迭代器声明为 `mut` 可变，因为调用 `next` 会改变迭代器其中的状态数据（当前遍历的位置等），而 `for` 循环去迭代则无需标注 `mut`，因为它会帮我们自动完成；

总之，`next` 方法对**迭代器的遍历是消耗性的**，每次消耗它一个元素，最终迭代器中将没有任何元素，只能返回 `None`。

##### 例子：模拟实现 for 循环

因为 `for` 循环是迭代器的语法糖，因此我们完全可以通过迭代器来模拟实现它：

```rust
let values = vec![1, 2, 3];

{
    let result = match IntoIterator::into_iter(values) {
        mut iter => loop {
            match iter.next() {
                Some(x) => { println!("{}", x); },
                None => break,
            }
        },
    };
    result
}
```

`IntoIterator::into_iter` 是使用[完全限定](https://course.rs/basic/trait/advance-trait.html#完全限定语法)的方式去调用 `into_iter` 方法，这种调用方式跟 `values.into_iter()` 是等价的。

同时我们使用了 `loop` 循环配合 `next` 方法来遍历迭代器中的元素，当迭代器返回 `None` 时，跳出循环。



#### IntoIterator 特征

其实有一个细节，由于 `Vec` 动态数组实现了 `IntoIterator` 特征，因此可以通过 `into_iter` 将其转换为迭代器，那如果本身就是一个迭代器，该怎么办？实际上，迭代器自身也实现了 `IntoIterator`，标准库早就帮我们考虑好了：

```rust
impl<I: Iterator> IntoIterator for I {
    type Item = I::Item;
    type IntoIter = I;

    #[inline]
    fn into_iter(self) -> I {
        self
    }
}
```

最终你完全可以写出这样的奇怪代码：

```rust
fn main() {
    let values = vec![1, 2, 3];

    for v in values.into_iter().into_iter().into_iter() {
        println!("{}",v)
    }
}
```



##### into_iter，iter，iter_mut

在之前的代码中，我们统一使用了 `into_iter` 的方式将数组转化为迭代器，除此之外，还有 `iter` 和 `iter_mut`，聪明的读者应该大概能猜到这三者的区别：

1. `into_iter` 会夺走所有权
2. `iter` 是借用
3. `iter_mut` 是可变借用

其实如果以后见多识广了，你会发现这种问题一眼就能看穿，`into_` 之类的，都是拿走所有权，`_mut` 之类的都是可变借用，剩下的就是不可变借用。

使用一段代码来解释下：

```rust
fn main() {
    let values = vec![1, 2, 3];

    for v in values.into_iter() {
        println!("{}", v)
    }

    // 下面的代码将报错，因为 values 的所有权在上面 `for` 循环中已经被转移走
    // println!("{:?}",values);

    let values = vec![1, 2, 3];
    let _values_iter = values.iter();

    // 不会报错，因为 values_iter 只是借用了 values 中的元素
    println!("{:?}", values);

    let mut values = vec![1, 2, 3];
    // 对 values 中的元素进行可变借用
    let mut values_iter_mut = values.iter_mut();

    // 取出第一个元素，并修改为0
    if let Some(v) = values_iter_mut.next() {
        *v = 0;
    }

    // 输出[0, 2, 3]
    println!("{:?}", values);
}
```

具体解释在代码注释中，就不再赘述，不过有两点需要注意的是：

1. `.iter()` 方法实现的迭代器，调用 `next` 方法返回的类型是 `Some(&T)` ；
2. `.iter_mut()` 方法实现的迭代器，调用 `next` 方法返回的类型是 `Some(&mut T)`，因此在 `if let Some(v) = values_iter_mut.next()` 中，`v` 的类型是 `&mut i32`，最终我们可以通过 `*v = 0` 的方式修改其值；

> **Iterator 和 IntoIterator 的区别**
>
> 这两个其实还蛮容易搞混的，但我们只需要记住：
>
> 1. `Iterator` 就是迭代器特征，只有实现了它才能称为迭代器，才能调用 `next`；
> 2. 而 `IntoIterator` 强调的是某一个类型如果实现了该特征，它可以通过 `into_iter`，`iter` 等方法变成一个迭代器。



#### 消费者与适配器

**消费者**是迭代器上的方法，它会**消费掉迭代器中的元素**，然后返回其类型的值，这些消费者都有一个共同的特点：在它们的定义中，都依赖 `next` 方法来消费元素，因此这也是为什么迭代器要实现 `Iterator` 特征，而该特征必须要实现 `next` 方法的原因。

##### 消费者适配器

只要迭代器上的某个方法 `A` 在其内部调用了 `next` 方法，那么 `A` 就被称为**消费性适配器**：因为 `next` 方法会消耗掉迭代器上的元素，所以方法 `A` 的调用也会消耗掉迭代器上的元素。

其中一个例子是 `sum` 方法，它会拿走迭代器的所有权，然后通过不断调用 `next` 方法对里面的元素进行求和：

```rust
fn main() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);

    // v1_iter 是借用了 v1，因此 v1 可以照常使用
    println!("{:?}",v1);

    // 以下代码会报错，因为 `sum` 拿到了迭代器 `v1_iter` 的所有权
    // println!("{:?}",v1_iter);
}
```

如代码注释中所说明的：在使用 `sum` 方法后，我们将无法再使用 `v1_iter`，因为 `sum` 拿走了该迭代器的所有权：

```rust
fn sum<S>(self) -> S
where 
Self: Sized,
S: Sum<Self::Item>,
{ Sum::sum(self) }
```

从 `sum` 源码中也可以清晰看出，`self` 类型的方法参数拿走了所有权。

#### 迭代器适配器

既然消费者适配器是消费掉迭代器，然后返回一个值。那么迭代器适配器，顾名思义，会**返回一个新的迭代器**，这是实现**链式方法调用的关键**：`v.iter().map().filter()...` 。

与消费者适配器不同，迭代器适配器是惰性的，意味着你**需要一个消费者适配器来收尾，最终将迭代器转换成一个具体的值**：

```rust
let v1: Vec<i32> = vec![1, 2, 3];

v1.iter().map(|x| x + 1);
```

运行后输出：

```console
warning: unused `Map` that must be used
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_must_use)]` on by default
  = note: iterators are lazy and do nothing unless consumed // 迭代器 map 是惰性的，这里不产生任何效果
```

如上述中文注释所说，这里的 `map` 方法是一个迭代者适配器，它是惰性的，不产生任何行为，因此我们还需要一个消费者适配器进行收尾：

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

##### collect

上面代码中，使用了 `collect` 方法，该方法就是一个**消费者适配器**，使用它可以将一个迭代器中的元素收集到指定类型中，这里我们为 `v2` 标注了 `Vec<_>` 类型，就是为了告诉 `collect`：请把迭代器中的元素消费掉，然后把值收集成 `Vec<_>` 类型，至于为何使用 `_`，因为编译器会帮我们自动推导。

为何 `collect` 在消费时要指定类型？是因为该方法其实很强大，可以收集成多种不同的集合类型，`Vec<T>` 仅仅是其中之一，因此我们必须显式的告诉编译器我们想要收集成的集合类型。

还有一点值得注意，`map` 会对迭代器中的每一个值进行一系列操作，然后把该值转换成另外一个新值，该操作是通过闭包 `|x| x + 1` 来完成：最终迭代器中的每个值都增加了 `1`，从 `[1, 2, 3]` 变为 `[2, 3, 4]`。

再来看看如何使用 `collect` 收集成 `HashMap` 集合：

```rust
use std::collections::HashMap;
fn main() {
    let names = ["sunface", "sunfei"];
    let ages = [18, 18];
    let folks: HashMap<_, _> = names.into_iter().zip(ages.into_iter()).collect();

    println!("{:?}",folks);
}
```

`zip` 是一个**迭代器适配器**，它的作用就是将两个迭代器的内容压缩到一起，形成 `Iterator<Item=(ValueFromA, ValueFromB)>` 这样的新的迭代器，在此处就是形如 `[(name1, age1), (name2, age2)]` 的迭代器。

然后再通过 `collect` 将新迭代器中`(K, V)` 形式的值收集成 `HashMap<K, V>`，同样的，这里必须显式声明类型，然后 `HashMap` 内部的 `KV` 类型可以交给编译器去推导，最终编译器会推导出 `HashMap<&str, i32>`，完全正确！

##### 闭包作为适配器参数

之前的 `map` 方法中，我们使用**闭包**来作为迭代器适配器的参数，它最大的好处不仅在于可以就地实现迭代器中元素的处理，还在于可以捕获环境值：

```rust
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}
```

`filter` 是迭代器适配器，用于对迭代器中的每个值进行过滤。 它使用闭包作为参数，该闭包的参数 `s` 是来自迭代器中的值，然后使用 `s` 跟外部环境中的 `shoe_size` 进行比较，若相等，则在迭代器中保留 `s` 值，若不相等，则从迭代器中剔除 `s` 值，最终通过 `collect` 收集为 `Vec<Shoe>` 类型。



#### 实现 Iterator 特征

之前的内容我们一直基于数组来创建迭代器，实际上，不仅仅是数组，基于其它集合类型一样可以创建迭代器，例如 `HashMap`。 你也可以创建自己的迭代器 —— 只要为自定义类型实现 `Iterator` 特征即可。

首先，创建一个计数器：

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}
```

我们为计数器 `Counter` 实现了一个关联函数 `new`，用于创建新的计数器实例。下面我们继续为计数器实现 `Iterator` 特征：

```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        if self.count < 5 {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}
```

首先，将该特征的关联类型设置为 `u32`，由于我们的计数器保存的 `count` 字段就是 `u32` 类型， 因此在 `next` 方法中，最后返回的是实际上是 `Option<u32>` 类型。

每次调用 `next` 方法，都会让计数器的值加一，然后返回最新的计数值，一旦计数大于 5，就返回 `None`。

最后，使用我们新建的 `Counter` 进行迭代：

```rust
fn main() {
    let mut counter = Counter::new();

    assert_eq!(counter.next(), Some(1));
    assert_eq!(counter.next(), Some(2));
    assert_eq!(counter.next(), Some(3));
    assert_eq!(counter.next(), Some(4));
    assert_eq!(counter.next(), Some(5));
    assert_eq!(counter.next(), None);
}
```



#### 实现 Iterator 特征的其他方法

可以看出，实现自己的迭代器非常简单，但是 `Iterator` 特征中，不仅仅是只有 `next` 一个方法，那为什么我们只需要实现它呢？因为其它方法都具有[默认实现](https://course.rs/basic/trait/trait.html#默认实现)，所以无需像 `next` 这样手动去实现，而且这些默认实现的方法其实都是基于 `next` 方法实现的。

下面的代码演示了部分方法的使用：

```rust
let sum: u32 = Counter::new()
    .zip(Counter::new().skip(1))
    .map(|(a, b)| a * b)
    .filter(|x| x % 3 == 0)
    .sum();
assert_eq!(18, sum);
```

其中 `zip`，`map`，`filter` 是迭代器适配器：

- `zip` 把两个迭代器合并成一个迭代器，新迭代器中，每个元素都是一个元组，由之前两个迭代器的元素组成。例如将**形如** `[1, 2, 3, 4, 5]` 和 `[2, 3, 4, 5]` 的迭代器合并后，新的迭代器形如 `[(1, 2),(2, 3),(3, 4),(4, 5)]`
- `map` 是将迭代器中的值经过映射后，转换成新的值[2, 6, 12, 20]
- `filter` 对迭代器中的元素进行过滤，若闭包返回 `true` 则保留元素[6, 12]，反之剔除

而 `sum` 是消费者适配器，对迭代器中的所有元素求和，最终返回一个 `u32` 值 `18`。

##### enumerate

在之前的流程控制章节，针对 `for` 循环，我们提供了一种方法可以获取迭代时的索引：

```rust
let v = vec![1u64, 2, 3, 4, 5, 6];
for (i,v) in v.iter().enumerate() {
    println!("第{}个值是{}",i,v)
}
```

相信当时，很多读者还是很迷茫的，不知道为什么要这么复杂才能获取到索引，学习本章节后，相信你有了全新的理解，首先 `v.iter()` 创建迭代器，其次 调用 `Iterator` 特征上的方法 `enumerate`，该方法产生一个新的迭代器，其中每个元素均是元组 `(索引，值)`。

因为 `enumerate` 是迭代器适配器，因此我们可以对它返回的迭代器调用其它 `Iterator` 特征方法：

```rust
let v = vec![1u64, 2, 3, 4, 5, 6];
let val = v.iter()
    .enumerate()
    // 每两个元素剔除一个
    // [1, 3, 5]
    .filter(|&(idx, _)| idx % 2 == 0)
    .map(|(idx, val)| val)
    // 累加 1+3+5 = 9
    .fold(0u64, |sum, acm| sum + acm);

println!("{}", val);
```



#### 迭代器性能

前面提到，要完成集合遍历，既可以使用 `for` 循环也可以使用迭代器，那么二者之间该怎么选择呢，性能有多大差距呢？

理论分析不会有结果，直接测试最为靠谱：

```rust
#![feature(test)]

extern crate rand;
extern crate test;

fn sum_for(x: &[f64]) -> f64 {
    let mut result: f64 = 0.0;
    for i in 0..x.len() {
        result += x[i];
    }
    result
}

fn sum_iter(x: &[f64]) -> f64 {
    x.iter().sum::<f64>()
}

#[cfg(test)]
mod bench {
    use test::Bencher;
    use rand::{Rng,thread_rng};
    use super::*;

    const LEN: usize = 1024*1024;

    fn rand_array(cnt: u32) -> Vec<f64> {
        let mut rng = thread_rng();
        (0..cnt).map(|_| rng.gen::<f64>()).collect()
    }

    #[bench]
    fn bench_for(b: &mut Bencher) {
        let samples = rand_array(LEN as u32);
        b.iter(|| {
            sum_for(&samples)
        })
    }

    #[bench]
    fn bench_iter(b: &mut Bencher) {
        let samples = rand_array(LEN as u32);
        b.iter(|| {
            sum_iter(&samples)
        })
    }
}
```

上面的代码对比了 `for` 循环和迭代器 `iterator` 完成同样的求和任务的性能对比，可以看到迭代器还要更快一点。

```console
test bench::bench_for  ... bench:     998,331 ns/iter (+/- 36,250)
test bench::bench_iter ... bench:     983,858 ns/iter (+/- 44,673)
```

迭代器是 Rust 的 **零成本抽象**（**zero-cost abstractions**）之一，意味着抽象并不会引入运行时开销，这与 `Bjarne Stroustrup`（C++ 的设计和实现者）在 `Foundations of C++（2012）` 中所定义的 **零开销**（**zero-overhead**）如出一辙：

> **In general, C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.**
>
> 一般来说，C++的实现遵循零开销原则：没有使用时，你不必为其买单。 更进一步说，需要使用时，你也无法写出更优的代码了。 （翻译一下：用就完事了）

总之，迭代器是 **Rust** 受函数式语言启发而提供的高级语言特性，可以写出更加简洁、逻辑清晰的代码。编译器还可以通过循环展开（Unrolling）、向量化、消除边界检查等优化手段，使得迭代器和 `for` 循环都有极为高效的执行效率。

所以请放心大胆的使用迭代器，在获得更高的表达力的同时，也不会导致运行时的损失，何乐而不为呢！
