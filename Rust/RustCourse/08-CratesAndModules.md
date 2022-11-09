## 包（Crates）和模块（Modules）

- 当工程规模变大时，把代码写到一个甚至几个文件中，都是不太聪明的做法，可能存在以下问题：
  1. 单个文件过大，导致打开、翻页速度大幅变慢；
  2. 查询和定位效率大幅降低，类比下，你会把所有知识内容放在一个几十万字的文档中吗？；
  3. 只有一个代码层次：函数，难以维护和协作，想象一下你的操作系统只有一个根目录，剩下的都是单层子目录会如何：`disaster`；
  4. 容易滋生 Bug；

- 同时，将大的代码文件拆分成**包**（**Crates**）和**模块**（**Modules**），还允许我们实现**代码抽象**和**复用**：将你的代码封装好后提供给用户，那么用户只需要调用公共接口即可，无需知道内部该如何实现。

- 因此，跟其它语言一样，**Rust** 也提供了相应概念用于代码的组织管理：

  - **项目**（**Packages**）：一个 `Cargo` 提供的 `feature`，可以用来构建、测试和分享包；

  - **包**（**Crate**）：一个由多个模块组成的树形结构，可以作为三方库进行分发，也可以生成可执行文件进行运行；

  - **模块**（**Module**）：可以一个文件多个模块，也可以一个文件一个模块，模块可以被认为是真实项目中的代码组织单元；



### 包（Crates）与包（Packages）的区别

其实项目 `Package` 和包 `Crate` 很容易被搞混，官方对此做了明确的区分，但是一旦翻译过来，又在译名上混为一谈了。我们首先对这两概念稍做区分。

> 鉴于 Rust 团队标新立异的起名传统，以及包的名称被 `crate` 占用，库的名称被 `library` 占用，经过斟 **Rust Course** 将 `Package` 翻译成项目，你也可以理解为工程、软件包。

#### 包（Crates）

- 对于 **Rust** 而言，包是一个独立的可编译单元，它编译后会生成一个可执行文件或者一个库。

- 一个包会将相关联的功能打包在一起，使得该功能可以很方便的在多个项目中分享。例如标准库中没有提供但是在三方库中提供的 `rand` 包，它提供了随机数生成的功能，我们只需要将该包通过 `use rand;` 引入到当前项目的作用域中，就可以在项目中使用 `rand` 的功能：`rand::XXX`。

- 同一个包中不能有同名的类型，但是在不同包中就可以。例如，虽然 `rand` 包中，有一个 `Rng` 特征，可是我们依然可以在自己的项目中定义一个 `Rng`，前者通过 `rand::Rng` 访问，后者通过 `Rng` 访问，对于编译器而言，这两者的边界非常清晰，不会存在引用歧义。



#### 项目包（Packages）

由于 `Package` 就是一个项目，因此它包含有独立的 `Cargo.toml` 文件，以及因为功能性被组织在一起的一个或多个包。一个 `Package` 只能包含**一个**库(library)类型的包，但是可以包含**多个**二进制可执行类型的包。

##### 二进制 Package

- 让我们来创建一个二进制 `Package`：

  ```shell
  $ cargo new my-project
       Created binary (application) `my-project` package
  $ ls my-project
  Cargo.toml
  src
  $ ls my-project/src
  main.rs
  ```

  这里，Cargo 为我们创建了一个名称是 `my-project` 的 `Package`，同时在其中创建了 `Cargo.toml` 文件，可以看一下该文件，里面并没有提到 `src/main.rs` 作为程序的入口，原因是 Cargo 有一个惯例：**`src/main.rs` 是二进制包的根文件，该二进制包的包名跟所属 `Package` 相同，在这里都是 `my-project`**，所有的代码执行都从该文件中的 `fn main()` 函数开始。

- 使用 `cargo run` 可以运行该项目，输出：`Hello, world!`。

##### 库 Package

- 再来创建一个库类型的 `Package`：

  ```shell
  $ cargo new my-lib --lib
       Created library `my-lib` package
  $ ls my-lib
  Cargo.toml
  src
  $ ls my-lib/src
  lib.rs
  ```

  首先，如果你试图运行 `my-lib`，会报错：

  ```shell
  $ cargo run
  error: a bin target must be available for `cargo run`
  ```

  原因是库类型的 `Package` 只能作为三方库被其它项目引用，而不能独立运行，只有之前的二进制 `Package` 才可以运行。

- 与 `src/main.rs` 一样，Cargo 知道，如果一个 `Package` 包含有 `src/lib.rs`，意味它包含有一个库类型的同名包 `my-lib`，该包的根文件是 `src/lib.rs`。

##### 易混淆的 Package 和 crate

- 看完上面，相信大家看出来为何 `Package` 和包容易被混淆了吧？因为你用 `cargo new` 创建的 `Package` 和它其中包含的包是同名的！

- 不过，只要你牢记 `Package` 是一个项目工程，而包只是一个编译单元，基本上也就不会混淆这个两个概念了：`src/main.rs` 和 `src/lib.rs` 都是编译单元，因此它们都是包。

##### 典型的 Package 解构

上面创建的 `Package` 中仅包含 `src/main.rs` 文件，意味着它仅包含一个二进制同名包 `my-project`。如果一个 `Package` 同时拥有 `src/main.rs` 和 `src/lib.rs`，那就意味着它包含两个包（`Crates`）：库包和二进制包，这两个包名也都是 `my-project` —— 都与 `Package` 同名。

一个真实项目中典型的 `Package`，会包含多个二进制包，这些包文件被放在 `src/bin` 目录下，每一个文件都是独立的二进制包，同时也会包含一个库包，该包只能存在一个 `src/lib.rs`：

```css
.
├── Cargo.toml
├── Cargo.lock
├── src
│   ├── main.rs
│   ├── lib.rs
│   └── bin
│       └── main1.rs
│       └── main2.rs
├── tests
│   └── some_integration_tests.rs
├── benches
│   └── simple_bench.rs
└── examples
    └── simple_example.rs
```

- **唯一库包**：`src/lib.rs` ；
- **默认二进制包**：`src/main.rs`，编译后生成的可执行文件与 `Package` 同名；
- **其余二进制包**：`src/bin/main1.rs` 和 `src/bin/main2.rs`，它们会分别生成一个文件同名的二进制可执行文件；
- **集成测试文件**：`tests` 目录下；
- **基准性能测试** `benchmark` **文件**：`benches` 目录下；
- **项目示例**：`examples` 目录下；

> 这种目录结构基本上是 **Rust** 的标准目录结构，在 `GitHub` 的大多数项目上，你都将看到它的身影。



### 模块（Modules）

在本章节，我们将深入讲讲 **Rust** 的代码构成单元：**模块**。使用模块可以将包中的代码按照功能性进行重组，最终实现更好的可读性及易用性。同时，我们还能非常灵活地去控制代码的可见性，进一步强化 **Rust** 的安全性。



#### 创建嵌套模块

用小餐馆为例，来看看 Rust 的模块该如何使用。使用 `cargo new --lib restaurant` 创建一个小餐馆，注意，这里创建的是一个库类型的 `Package`，然后将以下代码放入 `src/lib.rs` 中：

```rust
// 餐厅前厅，用于吃饭
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

以上的代码创建了三个模块，有几点需要注意的：

1. 使用 `mod` 关键字来创建新模块，后面紧跟着模块名称；
2. 模块可以嵌套，这里嵌套的原因是招待客人和服务都发生在前厅，因此我们的代码模拟了真实场景；
3. 模块中可以定义各种 **Rust** 类型，例如**函数**、**结构体**、**枚举**、**特征**等；
4. **所有模块均定义在同一个文件中**；

类似上述代码中所做的，使用模块，我们就能将功能相关的代码组织到一起，然后通过一个模块名称来说明这些代码为何被组织在一起。这样其它程序员在使用你的模块时，就可以更快地理解和上手。



#### 模块树

我们提到过 `src/main.rs` 和 `src/lib.rs` 被称为~~包根~~（**crate root**），是由于这两个文件的内容形成了一个模块 `crate`，该模块位于包的树形结构（由模块组成的树形结构）的根部：

```shell
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

这颗树展示了模块之间**彼此的嵌套**关系，因此被称为**模块树**。其中 `crate` 包根是 `src/lib.rs` 文件，包根文件中的三个模块分别形成了模块树的剩余部分。

##### 父子模块

如果模块 `A` 包含模块 `B`，那么 `A` 是 `B` 的父模块，`B` 是 `A` 的子模块。在上例中，`front_of_house` 是 `hosting` 和 `serving` 的父模块，反之，后两者是前者的子模块。

聪明的读者，应该能联想到，模块树跟计算机上文件系统目录树的相似之处。不仅仅是组织结构上的相似，就连使用方式都很相似：每个文件都有自己的路径，用户可以通过这些路径使用它们，在 Rust 中，我们也通过路径的方式来引用模块。



#### 用路径引用模块

想要调用一个函数，就需要知道它的路径，在 Rust 中，这种路径有两种形式：

- **绝对路径**，从包根开始，路径名以包名或者 `crate` 作为开头；
- **相对路径**，从当前模块开始，以 `self`，`super` 或当前模块的标识符作为开头；

让我们继续经营那个惨淡的小餐馆，这次为它实现一个小功能： 文件名：src/lib.rs

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

上面的代码为了简化实现，省去了其余模块和函数，这样可以把关注点放在函数调用上。`eat_at_restaurant` 是一个定义在包根中的函数，在该函数中使用了两种方式对 `add_to_waitlist` 进行调用。

##### 绝对路径引用

因为 `eat_at_restaurant` 和 `add_to_waitlist` 都定义在一个包中，因此在绝对路径引用时，可以直接以 `crate` 开头，然后逐层引用，每一层之间使用 `::` 分隔：

```rust
crate::front_of_house::hosting::add_to_waitlist();
```

对比下之前的模块树：

```shell
crate
 └── eat_at_restaurant
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

可以看出，绝对路径的调用，完全符合了模块树的层级递进，非常符合直觉，如果类比文件系统，就跟使用绝对路径调用可执行程序差不多：`/front_of_house/hosting/add_to_waitlist`，使用 `crate` 作为开始就和使用 `/` 作为开始一样。

##### 相对路径引用

再回到模块树中，因为 `eat_at_restaurant` 和 `front_of_house` 都处于包根 `crate` 中，因此相对路径可以使用 `front_of_house` 作为开头：

```rust
front_of_house::hosting::add_to_waitlist();
```

如果类比文件系统，那么它类似于调用同一个目录下的程序，你可以这么做：`front_of_house/hosting/add_to_waitlist`，嗯也很符合直觉。

> **绝对还是相对？**
>
> - 如果只是为了引用到指定模块中的对象，那么两种都可以，但是在实际使用时，需要遵循一个原则：**当代码被挪动位置时，尽量减少引用路径的修改**，相信大家都遇到过，修改了某处代码，导致所有路径都要挨个替换，这显然不是好的路径选择。
>
> - 回到之前的例子，如果我们把 `front_of_house` 模块和 `eat_at_restaurant` 移动到一个模块中 `customer_experience`，那么绝对路径的引用方式就必须进行修改：`crate::customer_experience::front_of_house ...`，但是假设我们使用的相对路径，那么该路径就无需修改，因为它们两个的相对位置其实没有变：
>
>   ```shell
>   crate
>    └── customer_experience
>       └── eat_at_restaurant
>       └── front_of_house
>           ├── hosting
>           │   ├── add_to_waitlist
>           │   └── seat_at_table
>   ```
>
>   从新的模块树中可以很清晰的看出这一点。
>
> - 再比如，其它的都不动，把 `eat_at_restaurant` 移动到模块 `dining` 中，如果使用相对路径，你需要修改该路径，但如果使用的是绝对路径，就无需修改：
>
>   ```shell
>   crate
>    └── dining
>        └── eat_at_restaurant
>    └── front_of_house
>        ├── hosting
>        │   ├── add_to_waitlist
>   ```
>
> - 不过，如果不确定哪个好，你可以考虑优先使用绝对路径，因为调用的地方和定义的地方往往是分离的，而定义的地方较少会变动。



### 代码可见性

让我们运行下面的代码：

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

意料之外的报错了，毕竟看上去确实很简单且没有任何问题：

```shell
error[E0603]: module `hosting` is private
 --> src/lib.rs:9:28
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                            ^^^^^^^ private module
```

错误信息很清晰：`hosting` 模块是私有的，无法在包根进行访问，那么为何 `front_of_house` 模块就可以访问？因为它和 `eat_at_restaurant` 同属于一个包根作用域内，同一个模块内的代码自然不存在私有化问题。

- 模块不仅仅对于组织代码很有用，它还能定义代码的私有化边界：在这个边界内，什么内容能让外界看到，什么内容不能，都有很明确的定义。因此，如果希望让函数或者结构体等类型变成私有化的，可以使用模块。

- **Rust** 出于安全的考虑，默认情况下，所有的类型都是**私有化的**，包括函数、方法、结构体、枚举、常量，是的，就连模块本身也是私有化的。在中国，父亲往往不希望孩子拥有小秘密，但是在 **Rust** 中，**父模块完全无法访问子模块中的私有项，但是子模块却可以访问父模块、父父..模块的私有项**。

##### pub 关键字

类似其它语言的 `public` 或者 Go 语言中的首字母大写，Rust 提供了 `pub` 关键字，通过它你可以控制模块和模块中指定项的可见性。

针对之前的代码，我们知道了只需要将 `hosting` 模块标记为对外可见即可：

```rust
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}
```

但是不幸的是，又报错了：

```console
error[E0603]: function `add_to_waitlist` is private
  --> src/lib.rs:12:30
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                              ^^^^^^^^^^^^^^^ private function
```

哦？难道模块可见还不够，还需要将函数 `add_to_waitlist` 标记为可见的吗？ 是的，没错，模块可见性不代表模块内部项的可见性，模块的可见性仅仅是允许其它模块去引用它，但是想要引用它内部的项，还得继续将对应的项标记为 `pub`。

> 在实际项目中，一个模块需要对外暴露的数据和 API 往往就寥寥数个，如果将模块标记为可见代表着内部项也全部对外可见，那你是不是还得把那些不可见的，一个一个标记为 `private`？反而是更麻烦的多。

既然知道了如何解决，那么我们为函数也标记上 `pub`：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}
```



####  使用 super 引用模块

在[用路径引用模块](https://course.rs/basic/crate-module/module.html#用路径引用模块)中，我们提到了相对路径有三种方式开始：`self`、`super`和 `crate` 或者模块名，其中第三种在前面已经讲到过，现在来看看通过 `super` 的方式引用模块项。

`super` 代表的是父模块为开始的引用方式，非常类似于文件系统中的 `..` 语法：`../a/b` 文件名：（`src/lib.rs`）

```rust
fn serve_order() {}

// 厨房模块
mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }
    fn cook_order() {}
}
```

嗯，我们的小餐馆又完善了，终于有厨房了！看来第一个客人也快可以有了。。。在厨房模块中，使用 `super::serve_order` 语法，调用了父模块（`crate root`）中的 `serve_order` 函数。

那么你可能会问，为何不使用 `crate::serve_order` 的方式？额，其实也可以，不过如果你确定未来这种层级关系不会改变，那么 `super::serve_order` 的方式会更稳定，未来就算它们都不在包根了，依然无需修改引用路径。所以路径的选用，往往还是取决于场景，以及未来代码的可能走向。



#### 使用 self 引用模块

`self` 其实就是引用自身模块中的项，也就是说和我们之前章节的代码类似，都调用同一模块中的内容，区别在于之前章节中直接通过名称调用即可，而 `self`，你得多此一举：

```rust
fn serve_order() {
    self::back_of_house::cook_order()
}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        crate::serve_order();
    }

    pub fn cook_order() {}
}
```

是的，多此一举，因为完全可以直接调用 `back_of_house`，但是 `self` 还有一个大用处，在下一节中我们会讲。



#### 结构体和枚举的可见性

为何要把结构体和枚举的可见性单独拎出来讲呢？因为这两个家伙的成员字段拥有完全不同的可见性：

- 将结构体设置为 `pub`，但它的所有字段依然是私有的；
- 将枚举设置为 `pub`，它的所有字段也将对外可见；

原因在于，枚举和结构体的使用方式不一样：

- **结构体**的应用场景比较复杂，其中的字段也往往部分在 A 处被使用，部分在 B 处被使用，因此无法确定成员的可见性，那索性就设置为全部不可见，将选择权交给程序员。

- **如果枚举的成员对外不可见，那该枚举将一点用都没有，因此枚举成员的可见性自动跟枚举可见性保持一致**，这样可以简化用户的使用。



#### 模块与文件分离

在之前的例子中，我们所有的模块都定义在 `src/lib.rs` 中，但是当模块变多或者变大时，需要将模块放入一个单独的文件中，让代码更好维护。

现在，把 `front_of_house` 前厅分离出来，放入一个单独的文件中 `src/front_of_house.rs`：

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

然后，将以下代码留在 `src/lib.rs` 中：

```rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

so easy！其实跟之前在同一个文件中也没有太大的不同，但是有几点值得注意：

- `mod front_of_house;` 告诉 **Rust** 从另一个和模块 `front_of_house` 同名的文件中加载该模块的内容
- 使用绝对路径的方式来引用 `hosting` 模块：`crate::front_of_house::hosting;`

需要注意的是，和之前代码中 `mod front_of_house{..}` 的完整模块不同，现在的代码中，模块的声明和实现是分离的，实现是在单独的 `front_of_house.rs` 文件中，然后通过 `mod front_of_house;` 这条声明语句从该文件中把模块内容加载进来。因此我们可以认为，模块 `front_of_house` 的定义还是在 `src/lib.rs` 中，只不过模块的具体内容被移动到了 `src/front_of_house.rs` 文件中。

> 在这里出现了一个新的关键字 `use`，联想到其它章节我们见过的标准库引入 `use std::fmt;`，可以大致猜测，该关键字用来将外部模块中的项引入到当前作用域中来，这样无需冗长的父模块前缀即可调用：`hosting::add_to_waitlist();`，在下节中，我们将对 `use` 进行详细的讲解。



### 模块引入

如果代码中，通篇都是 `crate::front_of_house::hosting::add_to_waitlist` 这样的函数调用形式，我不知道有谁会喜欢，也许靠代码行数赚工资的人会很喜欢，但是强迫症肯定受不了，悲伤的是程序员大多都有强迫症。因此我们需要一个办法来简化这种使用方式，在 Rust 中，可以使用 `use` 关键字把路径提前引入到当前作用域中，随后的调用就可以省略该路径，极大地简化了代码。



#### 基本引入方式

在 Rust 中，引入模块中的项有两种方式：**绝对路径**和**相对路径**，这两者在前面章节都有讲过，就不再赘述，先来看看使用绝对路径的引入方式。

##### 绝对路径引入模块

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

这里，我们使用 `use` 和绝对路径的方式，将 `hosting` 模块引入到当前作用域中，然后只需通过 `hosting::add_to_waitlist` 的方式，即可调用目标模块中的函数，相比 `crate::front_of_house::hosting::add_to_waitlist()` 的方式要简单的多，那么还能更简单吗？

##### 相对路径引入模块中的函数

在下面代码中，我们不仅要使用相对路径进行引入，而且与上面引入 `hosting` 模块不同，直接引入该模块中的 `add_to_waitlist` 函数：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
    add_to_waitlist();
    add_to_waitlist();
}
```

##### 引入模块还是函数

从使用简洁性来说，**引入函数自然是更甚一筹**，但是在某些时候，引入模块会更好：

- 需要引入同一个模块的多个函数；
- 作用域中存在同名函数；

在以上两种情况中，使用 `use front_of_house::hosting` 引入模块要比 `use front_of_house::hosting::add_to_waitlist;` 引入函数更好。

例如，如果想使用 `HashMap`，那么直接引入该结构体是比引入模块更好的选择，因为在 `collections` 模块中，我们只需要使用一个 `HashMap` 结构体：

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

> 其实严格来说，对于引用方式并没有需要遵守的惯例，主要还是取决于你的喜好，不过我们建议：**优先使用最细粒度（引入函数、结构体等）的引用方式，如果引起了某种麻烦（例如前面两种情况），再使用引入模块的方式**。



##### 避免同名引用

根据上一章节的内容，我们只要保证同一个模块中不存在同名项就行，模块之间、包之间的同名，谁管得着谁啊，话虽如此，一起看看，如果遇到同名的情况该如何处理。

1. **\<module\>::\<function\>**

   ```rust
   use std::fmt;
   use std::io;
   
   fn function1() -> fmt::Result {
       // --snip--
   }
   
   fn function2() -> io::Result<()> {
       // --snip--
   }
   ```

   上面的例子给出了很好的解决方案，使用模块引入的方式，具体的 `Result` 通过 `<module>::Result` 的方式进行调用。可以看出，避免同名冲突的关键，就是使用**父模块的方式来调用**。

2. **as 别名（as <other_fn_name>）**

   对于同名冲突问题，还可以使用 `as` 关键字来解决，它可以赋予引入项一个全新的名称：

   ```rust
   use std::fmt::Result;
   use std::io::Result as IoResult;
   
   fn function1() -> Result {
       // --snip--
   }
   
   fn function2() -> IoResult<()> {
       // --snip--
   }
   ```

   如上所示，首先通过 `use std::io::Result` 将 `Result` 引入到作用域，然后使用 `as` 给予它一个全新的名称 `IoResult`，这样就不会再产生冲突：

   - `Result` 代表 `std::fmt::Result`；
   - `IoResult` 代表 `std:io::Result`；

##### 引入再导出

当外部的模块项 `A` 被引入到当前模块中时，它的可见性自动被设置为私有的，如果你希望允许其它外部代码引用我们的模块项 `A`，那么可以对它进行再导出：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

如上，使用 `pub use` 即可实现。这里 `use` 代表引入 `hosting` 模块到当前作用域，`pub` 表示将该引入的内容再度设置为可见。

当你希望将内部的实现细节隐藏起来或者按照某个目的组织代码时，可以使用 `pub use` 再导出，例如统一使用一个模块来提供对外的 API，那该模块就可以引入其它模块中的 API，然后进行再导出，最终对于用户来说，所有的 API 都是由一个模块统一提供的。

##### 引入简化

- **使用 `{}` 简化**

  对于以下一行一行的引入方式：

  ```rust
  use std::collections::HashMap;
  use std::collections::BTreeMap;
  use std::collections::HashSet;
  
  use std::cmp::Ordering;
  use std::io;
  ```

  可以使用 `{}` 来一起引入进来，在大型项目中，使用这种方式来引入，可以减少大量 `use` 的使用：

  ```rust
  use std::collections::{HashMap,BTreeMap,HashSet};
  use std::{cmp::Ordering, io};
  ```

  对于下面的同时引入模块和模块中的项：

  ```rust
  use std::io;
  use std::io::Write;
  ```

  可以使用 `{}` 的方式进行简化:

  ```rust
  use std::io::{self, Write};
  ```

  > **self**
  >
  > - 上面使用到了模块章节提到的 `self` 关键字，用来替代模块自身，结合上一节中的 `self`，可以得出它在模块中的两个用途：
  >   1. `use self::xxx`，表示加载当前模块中的 `xxx`。此时 `self` 可省略；
  >   2. `use xxx::{self, yyy}`，表示，加载当前路径下模块 `xxx` 本身，以及模块 `xxx` 下的 `yyy` ；

- **使用 `*` 简化**

  对于之前一行一行引入 `std::collections` 的方式，我们还可以使用

  ```rust
  use std::collections::*;
  ```

  以上这种方式来引入 `std::collections` 模块下的所有公共项，这些公共项自然包含了 `HashMap`，`HashSet` 等想手动引入的集合类型。

  当使用 `*` 来引入的时候**要格外小心，因为你很难知道到底哪些被引入到了当前作用域中，有哪些会和你自己程序中的名称相冲突**：

  ```rust
  use std::collections::*;
  
  struct HashMap;
  fn main() {
     let mut v =  HashMap::new();
     v.insert("a", 1);
  }
  ```

  以上代码中，`std::collection::HashMap` 被 `*` 引入到当前作用域，但是由于存在另一个同名的结构体，因此 `HashMap::new` 根本不存在，因为对于编译器来说，本地同名类型的优先级更高。

  > 在实际项目中，这种引用方式往往用于快速写测试代码，它可以把所有东西一次性引入到 `tests` 模块中。



#### 再看可见性

在上一节中，我们学习了[可见性](https://course.rs/basic/crate-module/module.html#代码可见性)这个概念，这也是模块体系中最为核心的概念，控制了模块中哪些内容可以被外部看见，但是在实际使用时，光被外面看到还不行，我们还想控制哪些人能看，这就是 Rust 提供的受限可见性。

例如，在 Rust 中，包是一个模块树，我们可以通过 `pub(crate) item;` 这种方式来实现：`item` 虽然是对外可见的，但是只在当前包内可见，外部包无法引用到该 `item`。

所以，如果我们想要让某一项可以在整个包中都可以被使用，那么有两种办法：

- 在包根中定义一个非 `pub` 类型的 `X`（父模块的项对子模块都是可见的，因此包根中的项对模块树上的所有模块都可见）；
- 在子模块中定义一个 `pub` 类型的 `Y`，同时通过 `use` 将其引入到包根；

```rust
mod a {
    pub mod b {
        pub fn c() {
            println!("{:?}",crate::X);
        }

        #[derive(Debug)]
        pub struct Y;
    }
}

#[derive(Debug)]
struct X;
use a::b::Y;
fn d() {
    println!("{:?}",Y);
}
```

以上代码充分说明了之前两种办法的使用方式，但是有时我们会遇到这两种方法都不太好用的时候。例如希望对于某些特定的模块可见，但是对于其他模块又不可见：

```rust
// 目标：`a` 导出 `I`、`bar` and `foo`，其他的不导出
pub mod a {
    pub const I: i32 = 3;

    fn semisecret(x: i32) -> i32 {
        use self::b::c::J;
        x + J
    }

    pub fn bar(z: i32) -> i32 {
        semisecret(I) * z
    }
    pub fn foo(y: i32) -> i32 {
        semisecret(I) + y
    }

    mod b {
        mod c {
            const J: i32 = 4;
        }
    }
}
```

这段代码会报错，因为与父模块中的项对子模块可见相反，子模块中的项对父模块是不可见的。这里 `semisecret` 方法中，`a` -> `b` -> `c` 形成了父子模块链，那 `c` 中的 `J` 自然对 `a` 模块不可见。

如果使用之前的可见性方式，那么想保持 `J` 私有，同时让 `a` 继续使用 `semisecret` 函数的办法是将该函数移动到 `c` 模块中，然后用 `pub use` 将 `semisecret` 函数进行再导出：

```rust
pub mod a {
    pub const I: i32 = 3;

    use self::b::semisecret;

    pub fn bar(z: i32) -> i32 {
        semisecret(I) * z
    }
    pub fn foo(y: i32) -> i32 {
        semisecret(I) + y
    }

    mod b {
        pub use self::c::semisecret;
        mod c {
            const J: i32 = 4;
            pub fn semisecret(x: i32) -> i32 {
                x + J
            }
        }
    }
}
```

这段代码说实话问题不大，但是有些破坏了我们之前的逻辑，如果想保持代码逻辑，同时又只让 `J` 在 `a` 内可见该怎么办？

```rust
pub mod a {
    pub const I: i32 = 3;

    fn semisecret(x: i32) -> i32 {
        use self::b::c::J;
        x + J
    }

    pub fn bar(z: i32) -> i32 {
        semisecret(I) * z
    }
    pub fn foo(y: i32) -> i32 {
        semisecret(I) + y
    }

    mod b {
        pub(in crate::a) mod c {
            pub(in crate::a) const J: i32 = 4;
        }
    }
}
```

通过 `pub(in crate::a)` 的方式，我们指定了模块 `c` 和常量 `J` 的可见范围都只是 `a` 模块中，`a` 之外的模块是完全访问不到它们的。

##### 限制可见性语法

`pub(crate)` 或 `pub(in crate::a)` 就是限制可见性语法，前者是限制在整个包内可见，后者是通过绝对路径，限制在包内的某个模块内可见，总结一下：

- `pub` 意味着可见性无任何限制；
- `pub(crate)` 表示在当前包可见；
- `pub(self)` 在当前模块可见；
- `pub(super)` 在父模块可见；
- `pub(in <path>)` 表示在某个路径代表的模块中可见，其中 `path` 必须是父模块或者祖先模块；



### 使用第三方包

之前我们一直在引入标准库模块或者自定义模块，现在来引入下第三方包中的模块，关于如何引入外部依赖，我们在 [Cargo 入门](https://course.rs/first-try/cargo.html#package-配置段落)中就有讲，这里直接给出操作步骤：

1. 修改 `Cargo.toml` 文件，在 `[dependencies]` 区域添加一行：`rand = "0.8.3"` ；
2. 此时，如果你用的是 `VSCode` 和 `rust-analyzer` 插件，该插件会自动拉取该库，你可能需要等它完成后，再进行下一步（VSCode 左下角有提示）；

好了，此时，`rand` 包已经被我们添加到依赖中，下一步就是在代码中使用：

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..101);
}
```

这里使用 `use` 引入了第三方包 `rand` 中的 `Rng` 特征，因为我们需要调用的 `gen_range` 方法定义在该特征中。

##### crates.io & lib.rs

**Rust** 社区已经为我们贡献了大量高质量的第三方包，你可以在 `crates.io` 或者 `lib.rs` 中检索和使用，从目前来说查找包更推荐 `lib.rs`，搜索功能更强大，内容展示也更加合理，但是下载依赖包还是得用`crates.io`。

你可以在网站上搜索 `rand` 包，看看它的文档使用方式是否和我们之前引入方式相一致：在网上找到想要的包，然后将你想要的包和版本信息写入到 `Cargo.toml` 中。



### 一个综合例子

```rust
// 一个名为 `my_mod` 的模块
mod my_mod {
    // 模块中的项默认具有私有的可见性
    fn private_function() {
        println!("called `my_mod::private_function()`");
    }

    // 使用 `pub` 修饰语来改变默认可见性。
    pub fn function() {
        println!("called `my_mod::function()`");
    }

    // 在同一模块中，项可以访问其它项，即使它是私有的。
    pub fn indirect_access() {
        print!("called `my_mod::indirect_access()`, that\n> ");
        private_function();
    }

    // 模块也可以嵌套
    pub mod nested {
        pub fn function() {
            println!("called `my_mod::nested::function()`");
        }

        #[allow(dead_code)]
        fn private_function() {
            println!("called `my_mod::nested::private_function()`");
        }

        // 使用 `pub(in path)` 语法定义的函数只在给定的路径中可见。
        // `path` 必须是父模块（parent module）或祖先模块（ancestor module）
        pub(in crate::my_mod) fn public_function_in_my_mod() {
            print!("called `my_mod::nested::public_function_in_my_mod()`, that\n > ");
            public_function_in_nested()
        }

        // 使用 `pub(self)` 语法定义的函数则只在当前模块中可见。
        pub(self) fn public_function_in_nested() {
            println!("called `my_mod::nested::public_function_in_nested");
        }

        // 使用 `pub(super)` 语法定义的函数只在父模块中可见。
        pub(super) fn public_function_in_super_mod() {
            println!("called my_mod::nested::public_function_in_super_mod");
        }
    }

    pub fn call_public_function_in_my_mod() {
        print!("called `my_mod::call_public_funcion_in_my_mod()`, that\n> ");
        nested::public_function_in_my_mod();
        print!("> ");
        nested::public_function_in_super_mod();
    }

    // `pub(crate)` 使得函数只在当前包中可见
    pub(crate) fn public_function_in_crate() {
        println!("called `my_mod::public_function_in_crate()");
    }

    // 嵌套模块的可见性遵循相同的规则
    mod private_nested {
        #[allow(dead_code)]
        pub fn function() {
            println!("called `my_mod::private_nested::function()`");
        }
    }
}

fn function() {
    println!("called `function()`");
}

fn main() {
    // 模块机制消除了相同名字的项之间的歧义。
    function();
    my_mod::function();

    // 公有项，包括嵌套模块内的，都可以在父模块外部访问。
    my_mod::indirect_access();
    my_mod::nested::function();
    my_mod::call_public_function_in_my_mod();

    // pub(crate) 项可以在同一个 crate 中的任何地方访问
    my_mod::public_function_in_crate();

    // pub(in path) 项只能在指定的模块中访问
    // 报错！函数 `public_function_in_my_mod` 是私有的
    //my_mod::nested::public_function_in_my_mod();
    // 试一试 ^ 取消该行的注释

    // 模块的私有项不能直接访问，即便它是嵌套在公有模块内部的

    // 报错！`private_function` 是私有的
    //my_mod::private_function();
    // 试一试 ^ 取消此行注释

    // 报错！`private_function` 是私有的
    //my_mod::nested::private_function();
    // 试一试 ^ 取消此行的注释

    // 报错！ `private_nested` 是私有的
    //my_mod::private_nested::function();
    // 试一试 ^ 取消此行的注释
}
```

