## 编译原理



### 编译过程

Go 语言是一门需要编译才能运行的编程语言，也就是说代码在运行之前需要通过编译器生成二进制机器码，包含二进制机器码的文件才能在目标机器上运行，如果我们想要了解 Go 语言的实现原理，理解它的编译过程就是一个没有办法绕过的事情。

这一节会先对 Go 语言编译的过程进行概述，从顶层介绍编译器执行的几个步骤，随后的几节会分别剖析各个步骤完成的工作和实现原理，同时也会对一些需要预先掌握的知识进行介绍，确保后面的章节能够被更好的理解。



#### 预备知识

想要深入了解 Go 语言的编译过程，需要提前了解一下编译过程中涉及的一些术语和专业知识。这些知识其实在我们的日常工作和学习中比较难用到，但是对于理解编译的过程和原理还是非常重要的。这一小节会简单挑选几个重要的概念提前进行介绍，减少后面章节的理解压力。

##### 抽象语法树

[抽象语法树](https://en.wikipedia.org/wiki/Abstract_syntax_tree)（Abstract Syntax Tree、AST），是源代码语法的结构的一种抽象表示，它用树状的方式表示编程语言的语法结构[1](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/#fn:1)。抽象语法树中的每一个节点都表示源代码中的一个元素，每一棵子树都表示一个语法元素，以表达式 `2 * 3 + 7` 为例，编译器的语法分析阶段会生成如下图所示的抽象语法树。

![abstract-syntax-tree](Images/2019-12-20-15768548776645-abstract-syntax-tree.png)

作为编译器常用的数据结构，抽象语法树抹去了源代码中不重要的一些字符 - 空格、分号或者括号等等。编译器在执行完语法分析之后会输出一个抽象语法树，这个抽象语法树会辅助编译器进行语义分析，我们可以用它来确定语法正确的程序是否存在一些类型不匹配的问题。

##### 静态单赋值

[静态单赋值](https://en.wikipedia.org/wiki/Static_single_assignment_form)（Static Single Assignment、SSA）是中间代码的特性，如果中间代码具有静态单赋值的特性，那么每个变量就只会被赋值一次[2](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/#fn:2)。在实践中，我们通常会用下标实现静态单赋值，这里以下面的代码举个例子：

```go
x := 1
x := 2
y := x
```

经过简单的分析，我们就能够发现上述的代码第一行的赋值语句 `x := 1` 不会起到任何作用。下面是具有 SSA 特性的中间代码，我们可以清晰地发现变量 `y_1` 和 `x_1` 是没有任何关系的，所以在机器码生成时就可以省去 `x := 1` 的赋值，通过减少需要执行的指令优化这段代码。

```go
x_1 := 1
x_2 := 2
y_1 := x_2
```

因为 SSA 的主要作用是对代码进行优化，所以它是编译器后端[3](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/#fn:3)的一部分；当然代码编译领域除了 SSA 还有很多中间代码的优化方法，编译器生成代码的优化也是一个古老并且复杂的领域，这里就不会展开介绍了。

##### 指令集

最后要介绍的一个预备知识就是[指令集](https://en.wikipedia.org/wiki/Instruction_set_architecture)[4](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/#fn:4)了，很多开发者在都会遇到在本地开发环境编译和运行正常的代码，在生产环境却无法正常工作，这种问题背后会有多种原因，而不同机器使用的不同指令集可能是原因之一。

我们大多数开发者都会使用 x86_64 的 Macbook 作为工作上主要使用的设备，在命令行中输入 `uname -m` 就能获得当前机器的硬件信息：

```bash
$ uname -m
x86_64
```

x86 是目前比较常见的指令集，除了 x86 之外，还有 arm 等指令集，苹果最新 Macbook 的自研芯片就使用了 arm 指令集，不同的处理器使用了不同的架构和机器语言，所以很多编程语言为了在不同的机器上运行需要将源代码根据架构翻译成不同的机器代码。

复杂指令集计算机（CISC）和精简指令集计算机（RISC）是两种遵循不同设计理念的指令集，从名字我们就可以推测出这两种指令集的区别：

- **复杂指令集**：通过增加指令的类型减少需要执行的指令数；
- **精简指令集**：使用更少的指令类型完成目标的计算任务；

早期的 CPU 为了减少机器语言指令的数量一般使用复杂指令集完成计算任务，这两者并没有绝对的优劣，它们只是在一些设计上的选择不同以达到不同的目的，我们会在后面的[机器码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-machinecode/)一节中详细介绍指令集架构，不过各位读者也可以主动了解相关的内容。



#### 编译

Go 语言编译器的源代码在 [`src/cmd/compile`](https://github.com/golang/go/tree/master/src/cmd/compile) 目录中，目录下的文件共同组成了 Go 语言的编译器，学过编译原理的人可能听说过编译器的前端和后端，编译器的前端一般承担着词法分析、语法分析、类型检查和中间代码生成几部分工作，而编译器后端主要负责目标代码的生成和优化，也就是将中间代码翻译成目标机器能够运行的二进制机器码。

![complication-process](https://img.draveness.me/2019-12-20-15768548776662-complication-process.png)

Go 的编译器在逻辑上可以被分成四个阶段：**词法与语法分析**、**类型检查和 AST 转换**、**通用 SSA 生成**和**最后的机器代码生成**，在这一节我们会使用比较少的篇幅分别介绍这四个阶段做的工作，后面的章节会具体介绍每一个阶段的具体内容。

##### 词法与语法分析

所有的编译过程其实都是从解析代码的源文件开始的，词法分析的作用就是解析源代码文件，它将文件中的字符串序列转换成 Token 序列，方便后面的处理和解析，我们一般会把执行词法分析的程序称为词法解析器（lexer）。

而语法分析的输入是词法分析器输出的 Token 序列，语法分析器会按照顺序解析 Token 序列，该过程会将词法分析生成的 Token 按照编程语言定义好的文法（Grammar）自下而上或者自上而下的规约，每一个 Go 的源代码文件最终会被归纳成一个 [SourceFile](https://golang.org/ref/spec#Source_file_organization) 结构[5](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/#fn:5)：

```go
SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
```

词法分析会返回一个不包含空格、换行等字符的 Token 序列，例如：`package`, `json`, `import`, `(`, `io`, `)`, …，而语法分析会把 Token 序列转换成有意义的结构体，即语法树：

```go
"json.go": SourceFile {
    PackageName: "json",
    ImportDecl: []Import{
        "io",
    },
    TopLevelDecl: ...
}
```

Token 到上述抽象语法树（AST）的转换过程会用到语法解析器，每一个 AST 都对应着一个单独的 Go 语言文件，这个抽象语法树中包括当前文件属于的包名、定义的常量、结构体和函数等。

> Go 语言的语法解析器使用的是 LALR(1)[6](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/#fn:6) 的文法，对解析器文法感兴趣的读者可以在推荐阅读中找到编译器文法的相关资料。

![golang-files-and-ast](Images/2019-12-20-15768548776670-golang-files-and-ast.png)

语法解析的过程中发生的任何语法错误都会被语法解析器发现并将消息打印到标准输出上，整个编译过程也会随着错误的出现而被中止。[词法与语法分析](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/)一节会详细介绍 Go 语言的文法、词法解析和语法解析过程。

##### 类型检查

当拿到一组文件的抽象语法树之后，Go 语言的编译器会对语法树中定义和使用的类型进行检查，类型检查会按照以下的顺序分别验证和处理不同类型的节点：

1. 常量、类型和函数名及类型；
2. 变量的赋值和初始化；
3. 函数和闭包的主体；
4. 哈希键值对的类型；
5. 导入函数体；
6. 外部的声明；

通过对整棵抽象语法树的遍历，我们在每个节点上都会对当前子树的类型进行验证，以保证节点不存在类型错误，所有的类型错误和不匹配都会在这一个阶段被暴露出来，其中包括：结构体对接口的实现。

类型检查阶段不止会对节点的类型进行验证，还会展开和改写一些内建的函数，例如 make 关键字在这个阶段会根据子树的结构被替换成 [`runtime.makeslice`](https://draveness.me/golang/tree/runtime.makeslice) 或者 [`runtime.makechan`](https://draveness.me/golang/tree/runtime.makechan) 等函数。

![golang-keyword-make](Images/2019-12-20-15768548776677-golang-keyword-make.png)

类型检查这一过程在整个编译流程中还是非常重要的，Go 语言的很多关键字都依赖类型检查期间的展开和改写，我们在[类型检查](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/)中会详细介绍这一步骤。

##### 中间代码生成

当我们将源文件转换成了抽象语法树、对整棵树的语法进行解析并进行类型检查之后，就可以认为当前文件中的代码不存在语法错误和类型错误的问题了，Go 语言的编译器就会将输入的抽象语法树转换成中间代码。

在类型检查之后，编译器会通过 [`cmd/compile/internal/gc.compileFunctions`](https://draveness.me/golang/tree/cmd/compile/internal/gc.compileFunctions) 编译整个 Go 语言项目中的全部函数，这些函数会在一个编译队列中等待几个 Goroutine 的消费，并发执行的 Goroutine 会将所有函数对应的抽象语法树转换成中间代码。

![concurrency-compiling](Images/2019-12-20-15768548776685-concurrency-compiling.png)

由于 Go 语言编译器的中间代码使用了 SSA 的特性，所以在这一阶段我们能够分析出代码中的无用变量和片段并对代码进行优化，[中间代码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)一节会详细介绍中间代码的生成过程并简单介绍 Go 语言中间代码的 SSA 特性。

##### 机器码生成

Go 语言源代码的 [`src/cmd/compile/internal`](https://github.com/golang/go/tree/master/src/cmd/compile/internal) 目录中包含了很多机器码生成相关的包，不同类型的 CPU 分别使用了不同的包生成机器码，其中包括 amd64、arm、arm64、mips、mips64、ppc64、s390x、x86 和 wasm，其中比较有趣的就是 WebAssembly（Wasm）[7](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/#fn:7)了。

作为一种在栈虚拟机上使用的二进制指令格式，它的设计的主要目标就是在 Web 浏览器上提供一种具有高可移植性的目标语言。Go 语言的编译器既然能够生成 Wasm 格式的指令，那么就能够运行在常见的主流浏览器中。

```bash
$ GOARCH=wasm GOOS=js go build -o lib.wasm main.go
```

我们可以使用上述的命令将 Go 的源代码编译成能够在浏览器上运行 WebAssembly 文件，当然除了这种新兴的二进制指令格式之外，Go 语言经过编译还可以运行在几乎全部的主流机器上，不过它的兼容性在除 Linux 和 Darwin 之外的机器上可能还有一些问题，例如：Go Plugin 至今仍然不支持 Windows[8](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/#fn:8)。

![supported-hardware](Images/2019-12-20-15768548776695-supported-hardware.png)

[机器码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-machinecode/)一节会详细介绍将中间代码翻译到不同目标机器的过程，其中也会简单介绍不同指令集架构的区别。



#### 编译器入口

Go 语言的编译器入口在 [`src/cmd/compile/internal/gc/main.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/main.go) 文件中，其中 600 多行的 [`cmd/compile/internal/gc.Main`](https://draveness.me/golang/tree/cmd/compile/internal/gc.Main) 就是 Go 语言编译器的主程序，该函数会先获取命令行传入的参数并更新编译选项和配置，随后会调用 [`cmd/compile/internal/gc.parseFiles`](https://draveness.me/golang/tree/cmd/compile/internal/gc.parseFiles) 对输入的文件进行词法与语法分析得到对应的抽象语法树：

```go
func Main(archInit func(*Arch)) {
	...

	lines := parseFiles(flag.Args())
```

得到抽象语法树后会分九个阶段对抽象语法树进行更新和编译，就像我们在上面介绍的，抽象语法树会经历类型检查、SSA 中间代码生成以及机器码生成三个阶段：

1. 检查常量、类型和函数的类型；
2. 处理变量的赋值；
3. 对函数的主体进行类型检查；
4. 决定如何捕获变量；
5. 检查内联函数的类型；
6. 进行逃逸分析；
7. 将闭包的主体转换成引用的捕获变量；
8. 编译顶层函数；
9. 检查外部依赖的声明；

对整个编译过程有一个顶层的认识之后，我们重新回到词法和语法分析后的具体流程，在这里编译器会对生成语法树中的节点执行类型检查，除了常量、类型和函数这些顶层声明之外，它还会检查变量的赋值语句、函数主体等结构：

```go
for i := 0; i < len(xtop); i++ {
    n := xtop[i]
    if op := n.Op; op != ODCL && op != OAS && op != OAS2 && (op != ODCLTYPE || !n.Left.Name.Param.Alias) {
        xtop[i] = typecheck(n, ctxStmt)
    }
}

for i := 0; i < len(xtop); i++ {
    n := xtop[i]
    if op := n.Op; op == ODCL || op == OAS || op == OAS2 || op == ODCLTYPE && n.Left.Name.Param.Alias {
        xtop[i] = typecheck(n, ctxStmt)
    }
}
...
```

类型检查会遍历传入节点的全部子节点，这个过程会展开和重写 `make` 等关键字，在类型检查会改变语法树中的一些节点，不会生成新的变量或者语法树，这个过程的结束也意味着源代码中已经不存在语法和类型错误，中间代码和机器码都可以根据抽象语法树正常生成。

```go
	initssaconfig()

	peekitabs()

	for i := 0; i < len(xtop); i++ {
		n := xtop[i]
		if n.Op == ODCLFUNC {
			funccompile(n)
		}
	}

	compileFunctions()

	for i, n := range externdcl {
		if n.Op == ONAME {
			externdcl[i] = typecheck(externdcl[i], ctxExpr)
		}
	}

	checkMapKeys()
}
```

在主程序运行的最后，编译器会将顶层的函数编译成中间代码并根据目标的 CPU 架构生成机器码，不过在这一阶段也有可能会再次对外部依赖进行类型检查以验证其正确性。



#### 小结

Go 语言的编译过程是非常有趣并且值得学习的，通过对 Go 语言四个编译阶段的分析和对编译器主函数的梳理，我们能够对 Go 语言的实现有一些基本的理解，掌握编译的过程之后，Go 语言对于我们来讲也不再是一个黑盒，所以学习其编译原理的过程还是非常让人着迷的。

---



### 词法分析和语法分析

当使用[通用编程语言](https://en.wikipedia.org/wiki/General-purpose_programming_language)[1](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:1)进行编写代码时，我们一定要认识到代码首先是写给人看的，只是恰好可以被机器编译和执行，而很难被人理解和维护的代码是非常糟糕。代码其实是按照约定格式编写的字符串，经过训练的软件工程师能对本来无意义的字符串进行分组和分析，按照约定的语法来理解源代码，并在脑内编译并运行程序。

既然工程师能够按照一定的方式理解和编译 Go 语言的源代码，那么我们如何模拟人理解源代码的方式构建一个能够分析编程语言代码的程序呢。我们在这一节中将介绍词法分析和语法分析这两个重要的编译过程，这两个过程能将原本机器看来无序意义的源文件转换成更容易理解、分析并且结构化的抽象语法树，接下来我们就看一看解析器眼中的 Go 语言是什么样的。



#### 词法分析

源代码在计算机『眼中』其实是一团乱麻，一个由字符组成的、无法被理解的字符串，所有的字符在计算器看来并没有什么区别，为了理解这些字符我们需要做的第一件事情就是**将字符串分组**，这能够降低理解字符串的成本，简化源代码的分析过程。

```go
make(chan int)
```

哪怕是不懂编程的人看到上述文本的第一反应也应该会将上述字符串分成几个部分 - `make`、`chan`、`int` 和括号，这个凭直觉分解文本的过程就是[词法分析](https://en.wikipedia.org/wiki/Lexical_analysis)，词法分析是将字符序列转换为标记（token）序列的过程[2](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:2)。

##### lex

[lex](http://dinosaur.compilertools.net/lex/index.html)[3](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:3) 是用于生成词法分析器的工具，lex 生成的代码能够将一个文件中的字符分解成 Token 序列，很多语言在设计早期都会使用它快速设计出原型。词法分析作为具有固定模式的任务，出现这种更抽象的工具必然的，lex 作为一个代码生成器，使用了类似 C 语言的语法，我们将 lex 理解为正则匹配的生成器，它会使用正则匹配扫描输入的字符流，下面是一个 lex 文件的示例：

```C
%{
#include <stdio.h>
%}

%%
package      printf("PACKAGE ");
import       printf("IMPORT ");
\.           printf("DOT ");
\{           printf("LBRACE ");
\}           printf("RBRACE ");
\(           printf("LPAREN ");
\)           printf("RPAREN ");
\"           printf("QUOTE ");
\n           printf("\n");
[0-9]+       printf("NUMBER ");
[a-zA-Z_]+   printf("IDENT ");
%%
```

这个定义好的文件能够解析 `package` 和 `import` 关键字、常见的特殊字符、数字以及标识符，虽然这里的规则可能有一些简陋和不完善，但是用来解析下面的这一段代码还是比较轻松的：

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello")
}
```

`.l` 结尾的 lex 代码并不能直接运行，我们首先需要通过 `lex` 命令将上面的 `simplego.l` 展开成 C 语言代码，这里可以直接执行如下所示的命令编译并打印文件中的内容：

```C
$ lex simplego.l
$ cat lex.yy.c
...
int yylex (void) {
	...
	while ( 1 ) {
		...
yy_match:
		do {
			register YY_CHAR yy_c = yy_ec[YY_SC_TO_UI(*yy_cp)];
			if ( yy_accept[yy_current_state] ) {
				(yy_last_accepting_state) = yy_current_state;
				(yy_last_accepting_cpos) = yy_cp;
			}
			while ( yy_chk[yy_base[yy_current_state] + yy_c] != yy_current_state ) {
				yy_current_state = (int) yy_def[yy_current_state];
				if ( yy_current_state >= 30 )
					yy_c = yy_meta[(unsigned int) yy_c];
				}
			yy_current_state = yy_nxt[yy_base[yy_current_state] + (unsigned int) yy_c];
			++yy_cp;
		} while ( yy_base[yy_current_state] != 37 );
		...

do_action:
		switch ( yy_act )
			case 0:
    			...

			case 1:
    			YY_RULE_SETUP
    			printf("PACKAGE ");
    			YY_BREAK
			...
}
```

[lex.yy.c](https://gist.github.com/draveness/85db6ec4a4088b63ccccf7f09424f474)[4](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:4) 的前 600 行基本都是宏和函数的声明和定义，后面生成的代码大都是为 `yylex` 这个函数服务的，这个函数使用[有限自动机（Deterministic Finite Automaton、DFA）](https://en.wikipedia.org/wiki/Deterministic_finite_automaton)[5](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:5)的程序结构来分析输入的字符流，上述代码中 `while` 循环就是这个有限自动机的主体，你如果仔细看这个文件生成的代码会发现当前的文件中并不存在 `main` 函数，`main` 函数是在 liblex 库中定义的，所以在编译时其实需要添加额外的 `-ll` 选项：

```bash
$ cc lex.yy.c -o simplego -ll
$ cat main.go | ./simplego
```

当我们将 C 语言代码通过 gcc 编译成二进制代码之后，就可以使用管道将上面提到的 Go 语言代码作为输入传递到生成的词法分析器中，这个词法分析器会打印出如下的内容：

```go
PACKAGE  IDENT

IMPORT  LPAREN
	QUOTE IDENT QUOTE
RPAREN

IDENT  IDENT LPAREN RPAREN  LBRACE
	IDENT DOT IDENT LPAREN QUOTE IDENT QUOTE RPAREN
RBRACE
```

从上面的输出我们能够看到 Go 源代码的影子，lex 生成的词法分析器 lexer 通过正则匹配的方式将机器原本很难理解的字符串进行分解成很多的 Token，有利于后面的处理。

![simplego-lex](Images/2019-12-21-15769078788724-simplego-lex.png)

到这里我们已经为各位读者展示了从定义 `.l` 文件、使用 lex 将 `.l` 文件编译成 C 语言代码以及二进制的全过程，而最后生成的词法分析器也能够将简单的 Go 语言代码进行转换成 Token 序列。lex 的使用还是比较简单的，我们可以使用它快速实现词法分析器，相信各位读者对它也有了一定的了解。

##### Go

Go 语言的词法解析是通过 [`src/cmd/compile/internal/syntax/scanner.go`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/syntax/scanner.go)[6](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:6) 文件中的 [`cmd/compile/internal/syntax.scanner`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner) 结构体实现的，这个结构体会持有当前扫描的数据源文件、启用的模式和当前被扫描到的 Token：

```go
type scanner struct {
	source
	mode   uint
	nlsemi bool

	// current token, valid after calling next()
	line, col uint
	blank     bool // line is blank up to col
	tok       token
	lit       string   // valid if tok is _Name, _Literal, or _Semi ("semicolon", "newline", or "EOF"); may be malformed if bad is true
	bad       bool     // valid if tok is _Literal, true if a syntax error occurred, lit may be malformed
	kind      LitKind  // valid if tok is _Literal
	op        Operator // valid if tok is _Operator, _AssignOp, or _IncOp
	prec      int      // valid if tok is _Operator, _AssignOp, or _IncOp
}
```

[`src/cmd/compile/internal/syntax/tokens.go`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/syntax/tokens.go)[7](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:7) 文件中定义了 Go 语言中支持的全部 Token 类型，所有的 `token` 类型都是正整数，你可以在这个文件中找到一些常见 Token 的定义，例如：操作符、括号和关键字等：

```go
const (
	_    token = iota
	_EOF       // EOF

	// operators and operations
	_Operator // op
	...

	// delimiters
	_Lparen    // (
	_Lbrack    // [
	...

	// keywords
	_Break       // break
	...
	_Type        // type
	_Var         // var

	tokenCount //
)
```

从 Go 语言中定义的 Token 类型，我们可以将语言中的元素分成几个不同的类别，分别是名称和字面量、操作符、分隔符和关键字。词法分析主要是由 [`cmd/compile/internal/syntax.scanner`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner) 这个结构体中的 [`cmd/compile/internal/syntax.scanner.next`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner.next) 方法驱动，这个 250 行函数的主体是一个 switch/case 结构：

```go
func (s *scanner) next() {
	...
	s.stop()
	startLine, startCol := s.pos()
	for s.ch == ' ' || s.ch == '\t' || s.ch == '\n' && !nlsemi || s.ch == '\r' {
		s.nextch()
	}

	s.line, s.col = s.pos()
	s.blank = s.line > startLine || startCol == colbase
	s.start()
	if isLetter(s.ch) || s.ch >= utf8.RuneSelf && s.atIdentChar(true) {
		s.nextch()
		s.ident()
		return
	}

	switch s.ch {
	case -1:
		s.tok = _EOF

	case '0', '1', '2', '3', '4', '5', '6', '7', '8', '9':
		s.number(false)
	...
	}
}
```

[`cmd/compile/internal/syntax.scanner`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner) 每次都会通过 [`cmd/compile/internal/syntax.source.nextch`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.source.nextch) 函数获取文件中最近的未被解析的字符，然后根据当前字符的不同执行不同的 case，如果遇到了空格和换行符这些空白字符会直接跳过，如果当前字符是 0 就会执行 [`cmd/compile/internal/syntax.scanner.number`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner.number) 方法尝试匹配一个数字。

```go
func (s *scanner) number(seenPoint bool) {
	kind := IntLit
	base := 10        // number base
	digsep := 0
	invalid := -1     // index of invalid digit in literal, or < 0

	s.kind = IntLit
	if !seenPoint {
		digsep |= s.digits(base, &invalid)
	}

	s.setLit(kind, ok)
}

func (s *scanner) digits(base int, invalid *int) (digsep int) {
	max := rune('0' + base)
	for isDecimal(s.ch) || s.ch == '_' {
		ds := 1
		if s.ch == '_' {
			ds = 2
		} else if s.ch >= max && *invalid < 0 {
			_, col := s.pos()
			*invalid = int(col - s.col) // record invalid rune index
		}
		digsep |= ds
		s.nextch()
	}
	return
}
```

上述的 [`cmd/compile/internal/syntax.scanner.number`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner.number) 方法省略了很多的代码，包括如何匹配浮点数、指数和复数，我们只是简单看一下词法分析匹配整数的逻辑：在 for 循环中不断获取最新的字符，将字符通过 [`cmd/compile/internal/syntax.source.nextch`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.source.nextch) 方法追加到 [`cmd/compile/internal/syntax.scanner`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner) 持有的缓冲区中；

当前包中的词法分析器 [`cmd/compile/internal/syntax.scanner`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner) 也只是为上层提供了 [`cmd/compile/internal/syntax.scanner.next`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner.next) 方法，词法解析的过程都是惰性的，只有在上层的解析器需要时才会调用 [`cmd/compile/internal/syntax.scanner.next`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner.next) 获取最新的 Token。

Go 语言的词法元素相对来说还是比较简单，使用这种巨大的 switch/case 进行词法解析也比较方便和顺手，早期的 Go 语言虽然使用 lex 这种工具来生成词法解析器，但是最后还是使用 Go 来实现词法分析器，用自己写的词法分析器来解析自己[8](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:8)。



#### 语法分析

[语法分析](https://en.wikipedia.org/wiki/Parsing)是根据某种特定的形式文法（Grammar）对 Token 序列构成的输入文本进行分析并确定其语法结构的过程[9](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:9)。从上面的定义来看，词法分析器输出的结果 — Token 序列是语法分析器的输入。

语法分析的过程会使用自顶向下或者自底向上的方式进行推导，在介绍 Go 语言语法分析之前，我们会先来介绍语法分析中的文法和分析方法。

##### 文法

[上下文无关文法](https://en.wikipedia.org/wiki/Context-free_grammar)是用来形式化、精确描述某种编程语言的工具，我们能够通过文法定义一种语言的语法，它主要包含一系列用于转换字符串的生产规则（Production rule）[10](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:10)。上下文无关文法中的每一个生产规则都会将规则左侧的非终结符转换成右侧的字符串，文法都由以下的四个部分组成：

> 终结符是文法中无法再被展开的符号，而非终结符与之相反，还可以通过生产规则进行展开，例如 “id”、“123” 等标识或者字面量[11](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:11)。

- $N$ 有限个非终结符的集合；
- $Σ$ 有限个终结符的集合；
- $P$ 有限个生产规则[12](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:12)的集合；
- $S$ 非终结符集合中唯一的开始符号；

文法被定义成一个四元组 $(N,Σ,P,S)$ ，这个元组中的几部分是上面提到的四个符号，其中最为重要的就是生产规则，每个生产规则都会包含非终结符、终结符或者开始符号，我们在这里可以举个简单的例子：

1. $S→aSb$
2. $S→ab$
3. $S→ϵ$

上述规则构成的文法就能够表示 ab、aabb 以及 aaa..bbb 等字符串，编程语言的文法就是由这一系列的生产规则表示的，在这里我们可以从 [`src/cmd/compile/internal/syntax/parser.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/parser.go)[13](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:13) 文件中摘抄一些 Go 语言文法的生产规则：

```go
SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
PackageClause  = "package" PackageName .
PackageName    = identifier .

ImportDecl       = "import" ( ImportSpec | "(" { ImportSpec ";" } ")" ) .
ImportSpec       = [ "." | PackageName ] ImportPath .
ImportPath       = string_lit .

TopLevelDecl  = Declaration | FunctionDecl | MethodDecl .
Declaration   = ConstDecl | TypeDecl | VarDecl .
```

> Go 语言更详细的文法可以从 [Language Specification](https://golang.org/ref/spec)[14](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:14) 中找到，这里不仅包含语言的文法，还包含词法元素、内置函数等信息。

因为每个 Go 源代码文件最终都会被解析成一个独立的抽象语法树，所以语法树最顶层的结构或者开始符号都是 SourceFile：

```go
SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
```

从 SourceFile 相关的生产规则我们可以看出，每一个文件都包含一个 `package` 的定义以及可选的 `import` 声明和其他的顶层声明（TopLevelDecl），每一个 SourceFile 在编译器中都对应一个 [`cmd/compile/internal/syntax.File`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.File) 结构体，你能从它们的定义中轻松找到两者的联系：

```go
type File struct {
	Pragma   Pragma
	PkgName  *Name
	DeclList []Decl
	Lines    uint
	node
}
```

顶层声明有五大类型，分别是常量、类型、变量、函数和方法，你可以在文件 [`src/cmd/compile/internal/syntax/parser.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/parser.go) 中找到这五大类型的定义。

```go
ConstDecl = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
ConstSpec = IdentifierList [ [ Type ] "=" ExpressionList ] .

TypeDecl  = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
TypeSpec  = AliasDecl | TypeDef .
AliasDecl = identifier "=" Type .
TypeDef   = identifier Type .

VarDecl = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
VarSpec = IdentifierList ( Type [ "=" ExpressionList ] | "=" ExpressionList ) .
```

上述的文法分别定义了 Go 语言中常量、类型和变量三种常见的结构，从文法中可以看到语言中的很多关键字 `const`、`type` 和 `var`，稍微回想一下我们日常接触的 Go 语言代码就能验证这里文法的正确性。

除了三种简单的语法结构之外，函数和方法的定义就更加复杂，从下面的文法我们可以看到 Statement 总共可以转换成 15 种不同的语法结构，这些语法结构就包括我们经常使用的 switch/case、if/else、for 循环以及 select 等语句：

```go
FunctionDecl = "func" FunctionName Signature [ FunctionBody ] .
FunctionName = identifier .
FunctionBody = Block .

MethodDecl = "func" Receiver MethodName Signature [ FunctionBody ] .
Receiver   = Parameters .

Block = "{" StatementList "}" .
StatementList = { Statement ";" } .

Statement =
	Declaration | LabeledStmt | SimpleStmt |
	GoStmt | ReturnStmt | BreakStmt | ContinueStmt | GotoStmt |
	FallthroughStmt | Block | IfStmt | SwitchStmt | SelectStmt | ForStmt |
	DeferStmt .

SimpleStmt = EmptyStmt | ExpressionStmt | SendStmt | IncDecStmt | Assignment | ShortVarDecl .
```

这些不同的语法结构共同定义了 Go 语言中能够使用的语法结构和表达式，对于 Statement 展开的更多内容这篇文章就不会详细介绍了，感兴趣的读者可以直接查看 [Go 语言说明书](https://golang.org/ref/spec#Statement)或者直接从 [`src/cmd/compile/internal/syntax/parser.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/parser.go) 文件中找到想要的答案。

##### 分析方法

语法分析的分析方法一般分为自顶向下和自底向上两种，这两种方式会使用不同的方式对输入的 Token 序列进行推导：

- [自顶向下分析](https://en.wikipedia.org/wiki/Top-down_parsing)：可以被看作找到当前输入流最左推导的过程，对于任意一个输入流，根据当前的输入符号，确定一个生产规则，使用生产规则右侧的符号替代相应的非终结符向下推导[15](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:15)；
- [自底向上分析](https://en.wikipedia.org/wiki/Bottom-up_parsing)：语法分析器从输入流开始，每次都尝试重写最右侧的多个符号，这其实是说解析器会从最简单的符号进行推导，在解析的最后合并成开始符号[16](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:16)；

如果读者无法理解上述的定义也没有关系，我们会在这一节的剩余部分介绍两种不同的分析方法以及它们的具体分析过程。

- **自顶向下**

  [LL 文法](https://en.wikipedia.org/wiki/LL_grammar)[17](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:17)是一种使用自顶向下分析方法的文法，下面给出了一个常见的 LL 文法：

  1. $S→aS1$
  2. $S1→bS1$
  3. $S1→ϵ$

  假设我们存在以上的生产规则和输入流 $abb$，如果这里使用**自顶向下**的方式进行语法分析，我们可以理解为每次解析器会通过新加入的字符判断应该使用什么方式展开当前的输入流：

  1. $S$ （开始符号）
  2. $aS1$（规则 1)
  3. $abS1$（规则 2)
  4. $abbS1$（规则 2)
  5. $abb$（规则 3)

  这种分析方法一定会从开始符号分析，通过下一个即将入栈的符号判断应该如何对当前堆栈中最右侧的非终结符（$S$ 或 $S1$）进行展开，直到整个字符串中不存在任何的非终结符，整个解析过程才会结束。

- **自底向上**

  但是如果我们使用自底向上的方式对输入流进行分析时，处理过程就会完全不同了，常见的四种文法 LR(0)、SLR、LR(1) 和 LALR(1) 使用了自底向上的处理方式[18](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:18)，我们可以简单写一个与上一节中效果相同的 LR(0) 文法：

  1. $S→S1$
  2. $S1→S1b$
  3. $S1→a$

  使用上述等效的文法处理同样地输入流 $abb$ 会使用完全不同的过程对输入流进行展开：

  1. $a$（入栈）
  2. $S1$（规则 3）
  3. $S1b$（入栈）
  4. $S1$（规则 2）
  5. $S1b$（入栈）
  6. $S1$（规则 2）
  7. $S$（规则 1）

  自底向上的分析过程会维护一个栈用于存储未被归约的符号，在整个过程中会执行两种不同的操作，一种叫做入栈（Shift），也就是将下一个符号入栈，另一种叫做归约（Reduce），也就是对最右侧的字符串按照生产规则进行合并。

  上述的分析过程和自顶向下的分析方法完全不同，这两种不同的分析方法其实也代表了计算机科学中两种不同的思想 — 从抽象到具体和从具体到抽象。

- **Lookahead**

  在语法分析中除了 LL 和 LR 这两种不同类型的语法分析方法之外，还存在另一个非常重要的概念，就是[向前查看（Lookahead）](https://en.wikipedia.org/wiki/Lookahead)，在不同生产规则发生冲突时，当前解析器需要通过预读一些 Token 判断当前应该用什么生产规则对输入流进行展开或者归约[19](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:19)，例如在 LALR(1) 文法中，需要预读一个 Token 保证出现冲突的生产规则能够被正确处理。

##### Go

Go 语言的解析器使用了 LALR(1) 的文法来解析词法分析过程中输出的 Token 序列[20](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/#fn:20)，最右推导加向前查看构成了 Go 语言解析器的最基本原理，也是大多数编程语言的选择。

我们在[概述](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/)中已经介绍了编译器的主函数，该函数调用的 [`cmd/compile/internal/gc.parseFiles`](https://draveness.me/golang/tree/cmd/compile/internal/gc.parseFiles) 会使用多个 Goroutine 来解析源文件，解析的过程会调用 [`cmd/compile/internal/syntax.Parse`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.Parse)，该函数初始化了一个新的 [`cmd/compile/internal/syntax.parser`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser) 结构体并通过 [`cmd/compile/internal/syntax.parser.fileOrNil`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.fileOrNil) 方法开启对当前文件的词法和语法解析：

```go
func Parse(base *PosBase, src io.Reader, errh ErrorHandler, pragh PragmaHandler, mode Mode) (_ *File, first error) {
	var p parser
	p.init(base, src, errh, pragh, mode)
	p.next()
	return p.fileOrNil(), p.first
}
```

[`cmd/compile/internal/syntax.parser.fileOrNil`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.fileOrNil) 方法其实是对上面介绍的 Go 语言文法的实现，该方法首先会解析文件开头的 `package` 定义：

```go
// SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
func (p *parser) fileOrNil() *File {
	f := new(File)
	f.pos = p.pos()

	if !p.got(_Package) {
		p.syntaxError("package statement must be first")
		return nil
	}
	f.PkgName = p.name()
	p.want(_Semi)
```

从上面的这一段方法中我们可以看出，当前方法会通过 [`cmd/compile/internal/syntax.parser.got`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.got) 来判断下一个 Token 是不是 `package` 关键字，如果是 `package` 关键字，就会执行 [`cmd/compile/internal/syntax.parser.name`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.name) 来匹配一个包名并将结果保存到返回的文件结构体中。

```go
for p.got(_Import) {
    f.DeclList = p.appendGroup(f.DeclList, p.importDecl)
    p.want(_Semi)
}
```

确定了当前文件的包名之后，就开始解析可选的 `import` 声明，每一个 `import` 在解析器看来都是一个声明语句，这些声明语句都会被加入到文件的 `DeclList` 中。

在这之后会根据编译器获取的关键字进入 switch 的不同分支，这些分支调用 [`cmd/compile/internal/syntax.parser.appendGroup`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.appendGroup) 方法并在方法中传入用于处理对应类型语句的 [`cmd/compile/internal/syntax.parser.constDecl`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.constDecl)、[`cmd/compile/internal/syntax.parser.typeDecl`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.typeDecl) 函数。

```go
	for p.tok != _EOF {
		switch p.tok {
		case _Const:
			p.next()
			f.DeclList = p.appendGroup(f.DeclList, p.constDecl)

		case _Type:
			p.next()
			f.DeclList = p.appendGroup(f.DeclList, p.typeDecl)

		case _Var:
			p.next()
			f.DeclList = p.appendGroup(f.DeclList, p.varDecl)

		case _Func:
			p.next()
			if d := p.funcDeclOrNil(); d != nil {
				f.DeclList = append(f.DeclList, d)
			}
		default:
			...
		}
	}

	f.Lines = p.source.line

	return f
}
```

[`cmd/compile/internal/syntax.parser.fileOrNil`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.fileOrNil) 使用了非常多的子方法对输入的文件进行语法分析，并在最后会返回文件开始创建的 [`cmd/compile/internal/syntax.File`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.File) 结构体。

读到这里的人可能会有一些疑惑，为什么没有看到词法分析的代码，这是因为词法分析器 [`cmd/compile/internal/syntax.scanner`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner) 作为结构体被嵌入到了 [`cmd/compile/internal/syntax.parser`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser) 中，所以这个方法中的 `p.next()` 实际上调用的是 [`cmd/compile/internal/syntax.scanner.next`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner.next) 方法，它会直接获取文件中的下一个 Token，所以词法和语法分析一起进行的。

[`cmd/compile/internal/syntax.parser.fileOrNil`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.fileOrNil) 与在这个方法中执行的其他子方法共同构成了一棵树，这棵树根节点是 [`cmd/compile/internal/syntax.parser.fileOrNil`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.fileOrNil)，子节点是 [`cmd/compile/internal/syntax.parser.importDecl`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.importDecl)、[`cmd/compile/internal/syntax.parser.constDecl`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.constDecl) 等方法，它们与 Go 语言文法中的生产规则一一对应。

![golang-parse](Images/2019-12-21-15769078928092-golang-parser.png)

[`cmd/compile/internal/syntax.parser.fileOrNil`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.fileOrNil)、[`cmd/compile/internal/syntax.parser.constDecl`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.constDecl) 等方法对应了 Go 语言中的生产规则，例如 [`cmd/compile/internal/syntax.parser.fileOrNil`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.fileOrNil) 实现的是：

```go
SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
```

我们根据这个规则能很好地理解语法分析器的实现原理 - 将编程语言的所有生产规则映射到对应的方法上，这些方法构成的树形结构最终会返回一个抽象语法树。

因为大多数方法的实现都非常相似，所以这里就仅介绍 [`cmd/compile/internal/syntax.parser.fileOrNil`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.fileOrNil) 方法的实现了，想要了解其他方法的实现原理，读者可以自行查看 [`src/cmd/compile/internal/syntax/parser.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/parser.go) 文件，该文件包含了语法分析阶段的全部方法。

###### 辅助方法

虽然这里不会展开介绍其他类似方法的实现，但是解析器运行过程中有几个辅助方法我们还是要简单说明一下，首先就是 [`cmd/compile/internal/syntax.parser.got`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.got) 和 [`cmd/compile/internal/syntax.parser.want`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.want) 这两个常见的方法：

```go
func (p *parser) got(tok token) bool {
	if p.tok == tok {
		p.next()
		return true
	}
	return false
}

func (p *parser) want(tok token) {
	if !p.got(tok) {
		p.syntaxError("expecting " + tokstring(tok))
		p.advance()
	}
}
```

[`cmd/compile/internal/syntax.parser.got`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.got) 只是用于快速判断一些语句中的关键字，如果当前解析器中的 Token 是传入的 Token 就会直接跳过该 Token 并返回 `true`；而 [`cmd/compile/internal/syntax.parser.want`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.want) 就是对 [`cmd/compile/internal/syntax.parser.got`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.got) 的简单封装了，如果当前 Token 不是我们期望的，就会立刻返回语法错误并结束这次编译。

这两个方法的引入能够帮助工程师在上层减少判断关键字的大量重复逻辑，让上层语法分析过程的实现更加清晰。

另一个方法 [`cmd/compile/internal/synctax.parser.appendGroup`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.appendGroup) 的实现就稍微复杂了一点，它的主要作用就是找出批量的定义，我们可以简单举一个例子：

```go
var (
   a int
   b int
)
```

这两个变量其实属于同一个组（Group），各种顶层定义的结构体 [`cmd/compile/internal/syntax.parser.constDecl`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.constDecl)、[`cmd/compile/internal/syntax.parser.varDecl`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.varDecl) 在进行语法分析时有一个额外的参数 [`cmd/compile/internal/syntax.Group`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.Group)，这个参数是通过 [`cmd/compile/internal/syntax.parser.appendGroup`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.appendGroup) 方法传递进去的：

```go
func (p *parser) appendGroup(list []Decl, f func(*Group) Decl) []Decl {
	if p.tok == _Lparen {
		g := new(Group)
		p.list(_Lparen, _Semi, _Rparen, func() bool {
			list = append(list, f(g))
			return false
		})
	} else {
		list = append(list, f(nil))
	}

	return list
}
```

[`cmd/compile/internal/syntax.parser.appendGroup`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.appendGroup) 方法会调用传入的 `f` 方法对输入流进行匹配并将匹配的结果追加到另一个参数 [`cmd/compile/internal/syntax.File`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.File) 结构体中的 `DeclList` 数组中，`import`、`const`、`var`、`type` 和 `func` 声明语句都是调用 [`cmd/compile/internal/syntax.parser.appendGroup`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser.appendGroup) 方法解析的。

###### 节点

语法分析器最终会使用不同的结构体来构建抽象语法树中的节点，其中根节点 [`cmd/compile/internal/syntax.File`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.File) 我们已经在上面介绍过了，其中包含了当前文件的包名、所有声明结构的列表和文件的行数：

```go
type File struct {
	Pragma   Pragma
	PkgName  *Name
	DeclList []Decl
	Lines    uint
	node
}
```

[`src/cmd/compile/internal/syntax/nodes.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/nodes.go) 文件中也定义了其他节点的结构体，其中包含全部声明类型的，这里简单看一下函数声明的结构：

```go
type (
	Decl interface {
		Node
		aDecl()
	}

	FuncDecl struct {
		Attr   map[string]bool
		Recv   *Field
		Name   *Name
		Type   *FuncType
		Body   *BlockStmt
		Pragma Pragma
		decl
	}
}
```

从函数定义中我们可以看出，函数在语法结构上主要由接受者、函数名、函数类型和函数体几个部分组成，函数体 [`cmd/compile/internal/syntax.BlockStmt`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.BlockStmt) 是由一系列的表达式组成的，这些表达式共同组成了函数的主体：

![golang-funcdecl-struct](Images/2019-12-21-15769078928100-golang-funcdecl-struct.png)

函数的主体其实是一个 [`cmd/compile/internal/syntax.Stmt`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.Stmt) 数组，[`cmd/compile/internal/syntax.Stmt`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.Stmt) 是一个接口，实现该接口的类型其实也非常多，总共有 14 种不同类型的 [`cmd/compile/internal/syntax.Stmt`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.Stmt) 实现：

![golang-statement](Images/2019-12-21-15769078928107-golang-statement.png)

这些不同类型的 [`cmd/compile/internal/syntax.Stmt`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.Stmt) 构成了全部命令式的 Go 语言代码，从中我们可以看到很多熟悉的控制结构，例如 if、for、switch 和 select，这些命令式的结构在其他的编程语言中也非常常见。



#### 小结

这一节介绍了 Go 语言的词法分析和语法分析过程，我们不仅从理论的层面介绍了词法和语法分析的原理，还从源代码出发详细分析 Go 语言的编译器是如何在底层实现词法和语法解析功能的。

了解 Go 语言的词法分析器 [`cmd/compile/internal/syntax.scanner`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner) 和语法分析器 [`cmd/compile/internal/syntax.parser`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.parser) 让我们对解析器处理源代码的过程有着比较清楚的认识，同时我们也在 Go 语言的文法和语法分析器中找到了熟悉的关键字和语法结构，加深了对 Go 语言的理解。

---



### 类型检查

我们在上一节中介绍了 Go 语言编译的第一个阶段 — 通过[词法和语法分析器](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/)的解析得到了抽象语法树，本节会继续介绍编译器执行的下一个阶段 — 类型检查。

提到类型检查和编程语言的类型系统，很多朋友可能会想到几个有些模糊并且不好理解的术语：强类型、弱类型、静态类型和动态类型。但是我们既然要谈到 Go 语言编译器的类型检查过程，我们接下来就彻底搞清楚这几个『类型』的含义与异同。



#### 强弱类型

[强类型和弱类型](https://en.wikipedia.org/wiki/Strong_and_weak_typing)[1](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/#fn:1)经常会被放在一起讨论，然而这两者并没有一个学术上的严格定义，多查阅些资料理解起来反而更加困难，很多资料甚至相互矛盾。

![strong-and-weak-typing](Images/2019-12-22-15770067978360-strong-and-weak-typing.png)

由于权威的定义的缺失，对于强弱类型，我们很多时候也只能根据现象和特性从直觉上进行判断，一般会有如下结论[2](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/#fn:2)：

- 强类型的编程语言在编译期间会有更严格的类型限制，也就是编译器会在编译期间发现变量赋值、返回值和函数调用时的类型错误；
- 弱类型的编程语言在出现类型错误时可能会在运行时进行隐式的类型转换，在类型转换时可能会造成运行错误。

依据上面的结论，我们就可以认为 Java、C# 等在编译期间进行类型检查的编程语言是强类型的。同样地，因为 Go 语言会在编译期间发现类型错误，也应该是强类型的编程语言。

如果强类型与弱类型这一对概念定义不严格且有歧义，那么在概念上较真本身是没有太多太多实际价值的，起码对于我们真正使用和理解编程语言帮助不大。问题来了，作为一种抽象的定义，我们使用它是为了什么呢？答案是，更多时候是为了方便沟通和分类。让我们忽略强弱类型，把更多注意力放到下面的问题上：

- 类型的转换是显式的还是隐式的？
- 编译器会帮助我们推断变量的类型么？

这些具体的问题在这种语境下其实更有价值，也希望各位读者能够减少对强弱类型的争执。



#### 静态类型与动态类型

静态类型和动态类型的编程语言其实也是两个不精确的表述，正确的表达应该是使用[静态类型检查](https://en.wikipedia.org/wiki/Type_system#Static_type_checking)和[动态类型检查](https://en.wikipedia.org/wiki/Type_system#Dynamic_type_checking_and_runtime_type_information)的编程语言，这一小节会分别介绍两种类型检查的特点以及它们的区别。

##### 静态类型检查

静态类型检查是基于对源代码的分析来确定运行程序类型安全的过程[3](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/#fn:3)，如果我们的代码能够通过静态类型检查，那么当前程序在一定程度上可以满足类型安全的要求，它能够减少程序在运行时的类型检查，也可以被看作是一种代码优化的方式。

作为一个开发者来说，静态类型检查能够帮助我们在编译期间发现程序中出现的类型错误，一些动态类型的编程语言都会有社区提供的工具为这些编程语言加入静态类型检查，例如 JavaScript 的 [Flow](https://flow.org/)[4](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/#fn:4)，这些工具能够在编译期间发现代码中的类型错误。

相信很多读者也都听过『动态类型一时爽，代码重构火葬场』[5](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/#fn:5)，使用 Python、Ruby 等编程语言的开发者一定对这句话深有体会，静态类型为代码在编译期间提供了约束，编译器能够在编译期间约束变量的类型。

静态类型检查在重构时能够帮助我们节省大量时间并避免遗漏，但是如果编程语言仅支持动态类型检查，那么就需要写大量的单元测试保证重构不会出现类型错误。当然这里并不是说测试不重要，我们写的**任何代码都应该有良好的测试**，这与语言没有太多的关系。

##### 动态类型检查

动态类型检查是在运行时确定程序类型安全的过程，它需要编程语言在编译时为所有的对象加入类型标签等信息，运行时可以使用这些存储的类型信息来实现动态派发、向下转型、反射以及其他特性[6](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/#fn:6)。动态类型检查能为工程师提供更多的操作空间，让我们能在运行时获取一些类型相关的上下文并根据对象的类型完成一些动态操作。

只使用动态类型检查的编程语言叫做动态类型编程语言，常见的动态类型编程语言就包括 JavaScript、Ruby 和 PHP，虽然这些编程语言在使用上非常灵活也不需要经过编译，但是有问题的代码该不会因为更加灵活就会减少错误，该出错时仍然会出错，它们在提高灵活性的同时，也提高了对工程师的要求。

##### 小结

静态类型检查和动态类型检查不是完全冲突和对立的，很多编程语言都会同时使用两种类型检查，例如：Java 不仅在编译期间提前检查类型发现类型错误，还为对象添加了类型信息，在运行时使用反射根据对象的类型动态地执行方法增强灵活性并减少冗余代码。



#### 执行过程

Go 语言的编译器不仅使用静态类型检查来保证程序运行的类型安全，还会在编程期间引入类型信息，让工程师能够使用反射来判断参数和变量的类型。当我们想要将 `interface{}` 转换成具体类型时会进行动态类型检查，如果无法发生转换就会发生程序崩溃。

这里会重点介绍编译期间的静态类型检查，在 [2.1 概述](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/)中，我们曾经介绍过 Go 语言编译器主程序中的 [`cmd/compile/internal/gc.Main`](https://draveness.me/golang/tree/cmd/compile/internal/gc.Main) 函数，其中有一段是这样的：

```go
for i := 0; i < len(xtop); i++ {
    n := xtop[i]
    if op := n.Op; op != ODCL && op != OAS && op != OAS2 && (op != ODCLTYPE || !n.Left.Name.Param.Alias) {
        xtop[i] = typecheck(n, ctxStmt)
    }
}

for i := 0; i < len(xtop); i++ {
    n := xtop[i]
    if op := n.Op; op == ODCL || op == OAS || op == OAS2 || op == ODCLTYPE && n.Left.Name.Param.Alias {
        xtop[i] = typecheck(n, ctxStmt)
    }
}

...

checkMapKeys()
```

这段代码的执行过程可以分成两个部分，首先通过 [`src/cmd/compile/internal/gc/typecheck.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/typecheck.go) 文件中的 [`cmd/compile/internal/gc.typecheck`](https://draveness.me/golang/tree/cmd/compile/internal/gc.typecheck) 函数检查常量、类型、函数声明以及变量赋值语句的类型，然后使用 [`cmd/compile/internal/gc.checkMapKeys`](https://draveness.me/golang/tree/cmd/compile/internal/gc.checkMapKeys) 检查哈希中键的类型，我们会分几个部分对上述代码的实现原理进行分析。

编译器类型检查的主要逻辑都在 [`cmd/compile/internal/gc.typecheck`](https://draveness.me/golang/tree/cmd/compile/internal/gc.typecheck) 和 [`cmd/compile/internal/gc.typecheck1`](https://draveness.me/golang/tree/cmd/compile/internal/gc.typecheck1) 这中，其中 [`cmd/compile/internal/gc.typecheck`](https://draveness.me/golang/tree/cmd/compile/internal/gc.typecheck) 中逻辑不是特别多，它会做一些类型检查之前的准备工作。而核心的逻辑都在 [`cmd/compile/internal/gc.typecheck1`](https://draveness.me/golang/tree/cmd/compile/internal/gc.typecheck1) 中，这是由 switch 语句构成的 2000 行函数：

```go
func typecheck1(n *Node, top int) (res *Node) {
	switch n.Op {
	case OTARRAY:
		...

	case OTMAP:
		...

	case OTCHAN:
		...
	}

	...

	return n
}
```

[`cmd/compile/internal/gc.typecheck1`](https://draveness.me/golang/tree/cmd/compile/internal/gc.typecheck1) 根据传入节点 Op 的类型进入不同的分支，其中包括加减乘数等操作符、函数调用、方法调用等 150 多种，因为节点的种类很多，所以这里只节选几个典型案例深入分析。



#### 切片 `OTARRAY`

如果当前节点的操作类型是 `OTARRAY`，那么这个分支首先会对右节点，也就是切片或者数组中元素的类型进行类型检查：

```go
	case OTARRAY:
		r := typecheck(n.Right, Etype)
		if r.Type == nil {
			n.Type = nil
			return n
		}
```

然后会根据当前节点的左节点不同，分三种情况更新 [`cmd/compile/internal/gc.Node`](https://draveness.me/golang/tree/cmd/compile/internal/gc.Node) 的类型，即三种不同的声明方式 `[]int`、`[...]int` 和 `[3]int`，第一种相对来说比较简单，会直接调用 [`cmd/compile/internal/types.NewSlice`](https://draveness.me/golang/tree/cmd/compile/internal/types.NewSlice)：

```go
		if n.Left == nil {
			t = types.NewSlice(r.Type)
```

[`cmd/compile/internal/types.NewSlice`](https://draveness.me/golang/tree/cmd/compile/internal/types.NewSlice) 直接返回了一个 `TSLICE` 类型的结构体，元素的类型信息也会存储在结构体中。当遇到 `[...]int` 这种形式的数组类型时，会由 [`cmd/compile/internal/gc.typecheckcomplit`](https://draveness.me/golang/tree/cmd/compile/internal/gc.typecheckcomplit) 处理：

```go
func typecheckcomplit(n *Node) (res *Node) {
	...
	if n.Right.Op == OTARRAY && n.Right.Left != nil && n.Right.Left.Op == ODDD {
		n.Right.Right = typecheck(n.Right.Right, ctxType)
		if n.Right.Right.Type == nil {
			n.Type = nil
			return n
		}
		elemType := n.Right.Right.Type

		length := typecheckarraylit(elemType, -1, n.List.Slice(), "array literal")

		n.Op = OARRAYLIT
		n.Type = types.NewArray(elemType, length)
		n.Right = nil
		return n
	}
	...
}
```

在最后，如果源代码中包含了数组的大小，那么会调用 [`cmd/compile/internal/types.NewArray`](https://draveness.me/golang/tree/cmd/compile/internal/types.NewArray) 初始化一个存储着数组中元素类型和数组大小的结构体：

```go
		} else {
			n.Left = indexlit(typecheck(n.Left, ctxExpr))
			l := n.Left
			v := l.Val()
			bound := v.U.(*Mpint).Int64()
			t = types.NewArray(r.Type, bound)		}

		n.Op = OTYPE
		n.Type = t
		n.Left = nil
		n.Right = nil
```

三个不同的分支会分别处理数组和切片声明的不同形式，每一个分支都会更新 [`cmd/compile/internal/gc.Node`](https://draveness.me/golang/tree/cmd/compile/internal/gc.Node) 结构体中存储的类型并修改抽象语法树中的内容。通过对这个片段的分析，我们发现数组的长度是类型检查期间确定的，而 `[...]int` 这种声明形式也只是 Go 语言为我们提供的语法糖。



#### 哈希 `OTMAP`

如果处理的节点是哈希，那么编译器会分别检查哈希的键值类型以验证它们类型的合法性：

```go
	case OTMAP:
		n.Left = typecheck(n.Left, Etype)
		n.Right = typecheck(n.Right, Etype)
		l := n.Left
		r := n.Right
		n.Op = OTYPE
		n.Type = types.NewMap(l.Type, r.Type)
		mapqueue = append(mapqueue, n)
		n.Left = nil
		n.Right = nil
```

与处理切片时几乎完全相同，这里会通过 [`cmd/compile/internal/types.NewMap`](https://draveness.me/golang/tree/cmd/compile/internal/types.NewMap) 创建一个新的 `TMAP` 结构并将哈希的键值类型都存储到该结构体中：

```go
func NewMap(k, v *Type) *Type {
	t := New(TMAP)
	mt := t.MapType()
	mt.Key = k
	mt.Elem = v
	return t
}
```

代表当前哈希的节点最终也会被加入 `mapqueue` 队列，编译器会在后面的阶段对哈希键的类型进行再次检查，而检查键类型调用的其实是上面提到的 [`cmd/compile/internal/gc.checkMapKeys`](https://draveness.me/golang/tree/cmd/compile/internal/gc.checkMapKeys) 函数：

```go
func checkMapKeys() {
	for _, n := range mapqueue {
		k := n.Type.MapType().Key
		if !k.Broke() && !IsComparable(k) {
			yyerrorl(n.Pos, "invalid map key type %v", k)
		}
	}
	mapqueue = nil
}
```

该函数会遍历 `mapqueue` 队列中等待检查的节点，判断这些类型能否作为哈希的键，如果当前类型不合法会在类型检查的阶段直接报错中止整个检查的过程。



#### 关键字 `OMAKE`

最后要介绍的是 Go 语言中很常见的内置函数 `make`，在类型检查阶段之前，无论是创建切片、哈希还是 Channel 用的都是 `make` 关键字，不过在类型检查阶段会根据创建的类型将 `make` 替换成特定的函数，后面[生成中间代码](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)的过程就不再会处理 `OMAKE` 类型的节点了，而是会依据生成的细分类型处理：

![golang-keyword-make](Images/2019-12-20-15768548776677-golang-keyword-make.png)

编译器会先检查关键字 `make` 的第一个类型参数，根据类型的不同进入不同分支，切片分支 `TSLICE`、哈希分支 `TMAP` 和 Channel 分支 `TCHAN`：

```go
	case OMAKE:
		args := n.List.Slice()

		n.List.Set(nil)
		l := args[0]
		l = typecheck(l, Etype)
		t := l.Type

		i := 1
		switch t.Etype {
		case TSLICE:
			...

		case TMAP:
			...

		case TCHAN:
			...
		}

		n.Type = t
```

如果 `make` 的第一个参数是切片类型，那么就会从参数中获取切片的长度 `len` 和容量 `cap` 并对这两个参数进行校验，其中包括：

1. 切片的长度参数是否被传入；
2. 切片的长度必须要小于或者等于切片的容量；

```go
		case TSLICE:
			if i >= len(args) {
				yyerror("missing len argument to make(%v)", t)
				n.Type = nil
				return n
			}

			l = args[i]
			i++
			l = typecheck(l, ctxExpr)
			var r *Node
			if i < len(args) {
				r = args[i]
				i++
				r = typecheck(r, ctxExpr)
			}

			if Isconst(l, CTINT) && r != nil && Isconst(r, CTINT) && l.Val().U.(*Mpint).Cmp(r.Val().U.(*Mpint)) > 0 {
				yyerror("len larger than cap in make(%v)", t)
				n.Type = nil
				return n
			}

			n.Left = l
			n.Right = r
			n.Op = OMAKESLICE
```

除了对参数的数量和合法性进行校验，这段代码最后会将当前节点的操作 Op 改成 `OMAKESLICE`，方便后面编译阶段的处理。

第二种情况就是 `make` 的第一个参数是 `map` 类型，在这种情况下，第二个可选的参数就是哈希的初始大小，在默认情况下它的大小是 0，当前分支最后也会改变当前节点的 Op 属性：

```go
		case TMAP:
			if i < len(args) {
				l = args[i]
				i++
				l = typecheck(l, ctxExpr)
				l = defaultlit(l, types.Types[TINT])
				if !checkmake(t, "size", l) {
					n.Type = nil
					return n
				}
				n.Left = l
			} else {
				n.Left = nodintconst(0)
			}
			n.Op = OMAKEMAP
```

`make` 内置函数能够初始化的最后一种结构就是 [Channel](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/) 了，从下面的代码我们可以发现第二个参数表示的就是 Channel 的缓冲区大小，如果不存在第二个参数，那么会创建缓冲区大小为 0 的 Channel：

```go
		case TCHAN:
			l = nil
			if i < len(args) {
				l = args[i]
				i++
				l = typecheck(l, ctxExpr)
				l = defaultlit(l, types.Types[TINT])
				if !checkmake(t, "buffer", l) {
					n.Type = nil
					return n
				}
				n.Left = l
			} else {
				n.Left = nodintconst(0)
			}
			n.Op = OMAKECHAN
```

在类型检查的过程中，无论 `make` 的第一个参数是什么类型，都会对当前节点的 Op 类型进行修改并且对传入参数的合法性进行一定的验证。



#### 小结

类型检查是 Go 语言编译的第二个阶段，在词法和语法分析之后我们得到了每个文件对应的抽象语法树，随后的类型检查会遍历抽象语法树中的节点，对每个节点的类型进行检验，找出其中存在的语法错误，在这个过程中也可能会对抽象语法树进行改写，这不仅能够去除一些不会被执行的代码、对代码进行优化以提高执行效率，而且也会修改 `make`、`new` 等关键字对应节点的操作类型。

`make` 和 `new` 这些内置函数其实并不会直接对应某些函数的实现，它们会在编译期间被转换成真正存在的其他函数，我们在下一节[中间代码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)中会介绍编译器对它们做了什么。

---



### 中间代码生成

前两节介绍的[词法与语法分析](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/)以及[类型检查](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/)两个部分都属于编译器前端，它们负责对源代码进行分析并检查其中存在的词法和语法错误，经过这两个阶段生成的抽象语法树已经不存在语法错误了，本节将继续介绍编译器的后端工作 —— 中间代码生成。



#### 概述

[中间代码](https://en.wikipedia.org/wiki/Intermediate_representation)是编译器或者虚拟机使用的语言，它可以来帮助我们分析计算机程序。在编译过程中，编译器会在将源代码转换到机器码的过程中，先把源代码转换成一种中间的表示形式，即中间代码[1](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/#fn:1)。

![intermediate-representation](Images/2019-12-23-15771129929822-intermediate-representation.png)

很多读者可能认为中间代码没有太多价值，我们可以直接将源代码翻译成目标语言，这种看起来可行的办法实际上有很多问题，其中最主要的是：它忽略了编译器面对的复杂场景，很多编译器需要将源代码翻译成多种机器码，直接翻译高级编程语言相对比较困难。

将编程语言到机器码的过程拆成中间代码生成和机器码生成两个简单步骤可以简化该问题，中间代码是一种更接近机器语言的表示形式，对中间代码的优化和分析相比直接分析高级编程语言更容易。

Go 语言编译器的中间代码具有静态单赋值（SSA）的特性，我们在 [Go 语言编译过程](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/)一节曾经介绍过静态单赋值，对这个特性不了解的读者可以回到上面的章节阅读相关的内容。

我们再来回忆一下编译阶段入口的主函数 [`cmd/compile/internal/gc.Main`](https://draveness.me/golang/tree/cmd/compile/internal/gc.Main) 中关于中间代码生成的部分，这一段代码会初始化 SSA 生成的配置，在配置初始化结束后会调用 [`cmd/compile/internal/gc.funccompile`](https://draveness.me/golang/tree/cmd/compile/internal/gc.funccompile) 编译函数：

```go
func Main(archInit func(*Arch)) {
	...

	initssaconfig()

	for i := 0; i < len(xtop); i++ {
		n := xtop[i]
		if n.Op == ODCLFUNC {
			funccompile(n)
		}
	}

	compileFunctions()
}
```

这一节将分别介绍配置的初始化以及函数编译两部分内容，我们会以 [`cmd/compile/internal/gc.initssaconfig`](https://draveness.me/golang/tree/cmd/compile/internal/gc.initssaconfig) 和 [`cmd/compile/internal/gc.funccompile`](https://draveness.me/golang/tree/cmd/compile/internal/gc.funccompile) 这两个函数作为入口来分析中间代码生成的具体过程和实现原理。



#### 配置初始化

SSA 配置的初始化过程是中间代码生成之前的准备工作，在该过程中，我们会缓存可能用到的类型指针、初始化 SSA 配置和一些之后会调用的运行时函数，例如：用于处理 `defer` 关键字的 [`runtime.deferproc`](https://draveness.me/golang/tree/runtime.deferproc)、用于创建 Goroutine 的 [`runtime.newproc`](https://draveness.me/golang/tree/runtime.newproc) 和扩容切片的 [`runtime.growslice`](https://draveness.me/golang/tree/runtime.growslice) 等，除此之外还会根据当前的目标设备初始化特定的 ABI[2](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/#fn:2)。我们以 [`cmd/compile/internal/gc.initssaconfig`](https://draveness.me/golang/tree/cmd/compile/internal/gc.initssaconfig) 作为入口开始分析配置初始化的过程。

```go
func initssaconfig() {
	types_ := ssa.NewTypes()

	_ = types.NewPtr(types.Types[TINTER])                             // *interface{}
	_ = types.NewPtr(types.NewPtr(types.Types[TSTRING]))              // **string
	_ = types.NewPtr(types.NewPtr(types.Idealstring))                 // **string
	_ = types.NewPtr(types.NewSlice(types.Types[TINTER]))             // *[]interface{}
	..
	_ = types.NewPtr(types.Errortype)                                 // *error
```

这个函数的执行过程总共可以分成三个部分，首先就是调用 [`cmd/compile/internal/ssa.NewTypes`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.NewTypes) 初始化 [`cmd/compile/internal/ssa.Types`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.Types) 结构体并调用 [`cmd/compile/internal/types.NewPtr`](https://draveness.me/golang/tree/cmd/compile/internal/types.NewPtr) 函数缓存类型的信息，[`cmd/compile/internal/ssa.Types`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.Types) 中存储了所有 Go 语言中基本类型对应的指针，比如 `Bool`、`Int8`、以及 `String` 等。

![golang-type-and-pointer](Images/2019-02-05-golang-type-and-pointer.png)

[`cmd/compile/internal/types.NewPtr`](https://draveness.me/golang/tree/cmd/compile/internal/types.NewPtr) 函数的主要作用是根据类型生成指向这些类型的指针，同时它会根据编译器的配置将生成的指针类型缓存在当前类型中，优化类型指针的获取效率：

```go
func NewPtr(elem *Type) *Type {
	if t := elem.Cache.ptr; t != nil {
		if t.Elem() != elem {
			Fatalf("NewPtr: elem mismatch")
		}
		return t
	}

	t := New(TPTR)
	t.Extra = Ptr{Elem: elem}
	t.Width = int64(Widthptr)
	t.Align = uint8(Widthptr)
	if NewPtrCacheEnabled {
		elem.Cache.ptr = t
	}
	return t
}
```

配置初始化的第二步是根据当前的 CPU 架构初始化 SSA 配置，我们会向 [`cmd/compile/internal/ssa.NewConfig`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.NewConfig) 函数传入目标机器的 CPU 架构、上述代码初始化的 [`cmd/compile/internal/ssa.Types`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.Types) 结构体、上下文信息和 Debug 配置：

```go
ssaConfig = ssa.NewConfig(thearch.LinkArch.Name, *types_, Ctxt, Debug['N'] == 0)
```

[`cmd/compile/internal/ssa.NewConfig`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.NewConfig) 会根据传入的 CPU 架构设置用于生成中间代码和机器码的函数，当前编译器使用的指针、寄存器大小、可用寄存器列表、掩码等编译选项：

```go
func NewConfig(arch string, types Types, ctxt *obj.Link, optimize bool) *Config {
	c := &Config{arch: arch, Types: types}
	c.useAvg = true
	c.useHmul = true
	switch arch {
	case "amd64":
		c.PtrSize = 8
		c.RegSize = 8
		c.lowerBlock = rewriteBlockAMD64
		c.lowerValue = rewriteValueAMD64
		c.registers = registersAMD64[:]
		...
	case "arm64":
	...
	case "wasm":
	default:
		ctxt.Diag("arch %s not implemented", arch)
	}
	c.ctxt = ctxt
	c.optimize = optimize

	...
	return c
}
```

所有的配置项一旦被创建，在整个编译期间都是只读的并且被全部编译阶段共享，也就是中间代码生成和机器码生成这两部分都会使用这一份配置完成自己的工作。在 [`cmd/compile/internal/gc.initssaconfig`](https://draveness.me/golang/tree/cmd/compile/internal/gc.initssaconfig) 方法调用的最后，会初始化一些编译器可能用到的 Go 语言运行时的函数：

```go
	assertE2I = sysfunc("assertE2I")
	assertE2I2 = sysfunc("assertE2I2")
	assertI2I = sysfunc("assertI2I")
	assertI2I2 = sysfunc("assertI2I2")
	deferproc = sysfunc("deferproc")
	Deferreturn = sysfunc("deferreturn")
	...
```

[`cmd/compile/internal/ssa.sysfunc`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.sysfunc) 函数会在对应的运行时包结构体 [`cmd/compile/internal/types.Pkg`](https://draveness.me/golang/tree/cmd/compile/internal/types.Pkg) 中创建一个新的符号 [`cmd/compile/internal/obj.LSym`](https://draveness.me/golang/tree/cmd/compile/internal/obj.LSym)，表示该方法已经注册到运行时包中。后面的中间代码生成阶段中直接使用这些方法，例如：上述代码片段中的 [`runtime.deferproc`](https://draveness.me/golang/tree/runtime.deferproc) 和 [`runtime.deferreturn`](https://draveness.me/golang/tree/runtime.deferreturn) 就是 Go 语言用于实现 defer 关键字的运行时函数，你能从后面的章节中了解更多内容。



#### 遍历和替换

在生成中间代码之前，编译器还需要替换抽象语法树中节点的一些元素，这个替换的过程是通过 [`cmd/compile/internal/gc.walk`](https://draveness.me/golang/tree/cmd/compile/internal/gc.walk) 和以相关函数实现的，这里简单展示几个函数的签名：

```go
func walk(fn *Node)
func walkappend(n *Node, init *Nodes, dst *Node) *Node
...
func walkrange(n *Node) *Node
func walkselect(sel *Node)
func walkselectcases(cases *Nodes) []*Node
func walkstmt(n *Node) *Node
func walkstmtlist(s []*Node)
func walkswitch(sw *Node)
```

这些用于遍历抽象语法树的函数会将一些关键字和内建函数转换成函数调用，例如： 上述函数会将 `panic`、`recover` 两个内建函数转换成 [`runtime.gopanic`](https://draveness.me/golang/tree/runtime.gopanic) 和 [`runtime.gorecover`](https://draveness.me/golang/tree/runtime.gorecover) 两个真正运行时函数，而关键字 `new` 也会被转换成调用 [`runtime.newobject`](https://draveness.me/golang/tree/runtime.newobject) 函数。

![golang-keyword-and-builtin-mapping](Images/2019-02-05-golang-keyword-and-builtin-mapping.png)

上图是从关键字或内建函数到运行时函数的映射，其中涉及 Channel、哈希、`make`、`new` 关键字以及控制流中的关键字 `select` 等。转换后的全部函数都属于运行时包，我们能在 [`src/cmd/compile/internal/gc/builtin/runtime.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/builtin/runtime.go) 文件中找到函数对应的签名和定义。

```go
func makemap64(mapType *byte, hint int64, mapbuf *any) (hmap map[any]any)
func makemap(mapType *byte, hint int, mapbuf *any) (hmap map[any]any)
func makemap_small() (hmap map[any]any)
func mapaccess1(mapType *byte, hmap map[any]any, key *any) (val *any)
...
func makechan64(chanType *byte, size int64) (hchan chan any)
func makechan(chanType *byte, size int) (hchan chan any)
...
```

这里的定义只是让 Go 语言完成编译，它们的实现都在另一个 [`runtime`](https://github.com/golang/go/tree/master/src/runtime) 包中。简单总结一下，编译器会将 Go 语言关键字转换成运行时包中的函数，也就是说关键字和内置函数的功能是由编译器和运行时共同完成的。

我们简单了解一下遍历节点时几个 Channel 操作是如何转换成运行时对应方法的，首先介绍向 Channel 发送消息或者从 Channel 接收消息两个操作，编译器会分别使用 `OSEND` 和 `ORECV` 表示发送和接收消息两个操作，在 [`cmd/compile/internal/gc.walkexpr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.walkexpr) 函数中会根据节点类型的不同进入不同的分支：

```go
func walkexpr(n *Node, init *Nodes) *Node {
	...
	case OSEND:
		n1 := n.Right
		n1 = assignconv(n1, n.Left.Type.Elem(), "chan send")
		n1 = walkexpr(n1, init)
		n1 = nod(OADDR, n1, nil)
		n = mkcall1(chanfn("chansend1", 2, n.Left.Type), nil, init, n.Left, n1)
	...
}
```

当遇到 `OSEND` 操作时，会使用 [`cmd/compile/internal/gc.mkcall1`](https://draveness.me/golang/tree/cmd/compile/internal/gc.mkcall1) 创建一个操作为 `OCALL` 的节点，这个节点包含当前调用的函数 [`runtime.chansend1`](https://draveness.me/golang/tree/runtime.chansend1) 和参数，新的 `OCALL` 节点会替换当前的 `OSEND` 节点，这就完成了对 `OSEND` 子树的改写。

![golang-ocall-node](Images/2019-12-23-15771129929846-golang-ocall-node.png)

在中间代码生成的阶段遇到 `ORECV` 操作时，编译器的处理与遇到 `OSEND` 时相差无几，我们只是将 [`runtime.chansend1`](https://draveness.me/golang/tree/runtime.chansend1) 换成了 [`runtime.chanrecv1`](https://draveness.me/golang/tree/runtime.chanrecv1)，其他的参数没有发生太大的变化：

```go
n = mkcall1(chanfn("chanrecv1", 2, n.Left.Type), nil, &init, n.Left, nodnil())
```

使用 `close` 关键字的 `OCLOSE` 操作也会在 [`cmd/compile/internal/gc.walkexpr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.walkexpr) 函数中被转换成调用 [`runtime.closechan`](https://draveness.me/golang/tree/runtime.closechan) 的 `OCALL` 节点：

```go
func walkexpr(n *Node, init *Nodes) *Node {
	...
	case OCLOSE:
		fn := syslook("closechan")

		fn = substArgTypes(fn, n.Left.Type)
		n = mkcall1(fn, nil, init, n.Left)
	...
}
```

编译器会在编译期间将 Channel 的这些内置操作转换成几个运行时函数，很多人都想要了解 Channel 底层的实现，但是并不知道函数的入口，通过本节的分析我们就知道 [`runtime.chanrecv1`](https://draveness.me/golang/tree/runtime.chanrecv1)、[`runtime.chansend1`](https://draveness.me/golang/tree/runtime.chansend1) 和 [`runtime.closechan`](https://draveness.me/golang/tree/runtime.closechan) 几个函数分别实现了 Channel 的接收、发送和关闭操作。



#### SSA 生成

经过 `walk` 系列函数的处理之后，抽象语法树就不会改变了，Go 语言的编译器会使用 [`cmd/compile/internal/gc.compileSSA`](https://draveness.me/golang/tree/cmd/compile/internal/gc.compileSSA) 函数将抽象语法树转换成中间代码，我们可以先看一下该函数的简要实现：

```go
func compileSSA(fn *Node, worker int) {
	f := buildssa(fn, worker)
	pp := newProgs(fn, worker)
	genssa(f, pp)

	pp.Flush()
}
```

[`cmd/compile/internal/gc.buildssa`](https://draveness.me/golang/tree/cmd/compile/internal/gc.buildssa) 负责生成具有 SSA 特性的中间代码，我们可以使用命令行工具来观察中间代码的生成过程，假设我们有以下的 Go 语言源代码，其中只包含一个简单的 `hello` 函数：

```go
package hello

func hello(a int) int {
	c := a + 2
	return c
}
```

我们可以使用 `GOSSAFUNC` 环境变量构建上述代码并获取从源代码到最终的中间代码经历的几十次迭代，其中所有的数据都存储到了 `ssa.html` 文件中：

```go
$ GOSSAFUNC=hello go build hello.go
# command-line-arguments
dumped SSA to ./ssa.html
```

上述文件中包含源代码对应的抽象语法树、几十个版本的中间代码以及最终生成的 SSA，在这里截取文件的一部分让各位读者简单了解该文件的内容：

![ssa-htm](Images/2019-12-23-15771129929852-ssa-html.png)

如上图所示，其中最左侧就是源代码，中间是源代码生成的抽象语法树，最右侧是生成的第一轮中间代码，后面还有几十轮，感兴趣的读者可以自己尝试编译一下。`hello` 函数对应的抽象语法树会包含当前函数的 `Enter`、`NBody` 和 `Exit` 三个属性，[`cmd/compile/internal/gc.buildssa`](https://draveness.me/golang/tree/cmd/compile/internal/gc.buildssa) 函数会输出这些属性，你能从这个简化的逻辑中看到上述输出的影子：

```go
func buildssa(fn *Node, worker int) *ssa.Func {
	name := fn.funcname()
	var astBuf *bytes.Buffer
	var s state

	fe := ssafn{
		curfn: fn,
		log:   printssa && ssaDumpStdout,
	}
	s.curfn = fn

	s.f = ssa.NewFunc(&fe)
	s.config = ssaConfig
	s.f.Type = fn.Type
	s.f.Config = ssaConfig

	...

	s.stmtList(fn.Func.Enter)
	s.stmtList(fn.Nbody)

	ssa.Compile(s.f)
	return s.f
}
```

`ssaConfig` 是我们在这里的第一小节初始化的结构体，其中包含了与 CPU 架构相关的函数和配置，随后的中间代码生成其实也分成两个阶段，第一阶段使用 [`cmd/compile/internal/gc.state.stmtList`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.stmtList) 以及相关函数将抽象语法树转换成中间代码，第二阶段调用 [`cmd/compile/internal/ssa`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/ssa) 包的 [`cmd/compile/internal/ssa.Compile`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.Compile) 通过多轮迭代更新 SSA 中间代码。

##### AST 到 SSA

[`cmd/compile/internal/gc.state.stmtList`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.stmtList) 会为传入数组中的每个节点调用 [`cmd/compile/internal/gc.state.stmt`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.stmt) 方法，编译器会根据节点操作符的不同将当前 AST 节点转换成对应的中间代码：

```go
func (s *state) stmt(n *Node) {
	...
	switch n.Op {
	case OCALLMETH, OCALLINTER:
		s.call(n, callNormal)
		if n.Op == OCALLFUNC && n.Left.Op == ONAME && n.Left.Class() == PFUNC {
			if fn := n.Left.Sym.Name; compiling_runtime && fn == "throw" ||
				n.Left.Sym.Pkg == Runtimepkg && (fn == "throwinit" || fn == "gopanic" || fn == "panicwrap" || fn == "block" || fn == "panicmakeslicelen" || fn == "panicmakeslicecap") {
				m := s.mem()
				b := s.endBlock()
				b.Kind = ssa.BlockExit
				b.SetControl(m)
			}
		}
		s.call(n.Left, callDefer)
	case OGO:
		s.call(n.Left, callGo)
	...
	}
}
```

从上面节选的代码中我们会发现，在遇到函数调用、方法调用、使用 defer 或者 go 关键字时都会执行 [`cmd/compile/internal/gc.state.callResult`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.callResult) 和 [`cmd/compile/internal/gc.state.call`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.call) 生成调用函数的 SSA 节点，这些在开发者看来不同的概念在编译器中都会被实现成静态的函数调用，上层的关键字和方法只是语言为我们提供的语法糖：

```go
func (s *state) callResult(n *Node, k callKind) *ssa.Value {
	return s.call(n, k, false)
}

func (s *state) call(n *Node, k callKind) *ssa.Value {
	...
	var call *ssa.Value
	switch {
	case k == callDefer:
		call = s.newValue1A(ssa.OpStaticCall, types.TypeMem, deferproc, s.mem())
	case k == callGo:
		call = s.newValue1A(ssa.OpStaticCall, types.TypeMem, newproc, s.mem())
	case sym != nil:
		call = s.newValue1A(ssa.OpStaticCall, types.TypeMem, sym.Linksym(), s.mem())
	..
	}
	...
}
```

首先，从 AST 到 SSA 的转化过程中，编译器会生成将函数调用的参数放到栈上的中间代码，处理参数之后才会生成一条运行函数的命令 `ssa.OpStaticCall`：

1. 当使用 defer 关键字时，插入 [`runtime.deferproc`](https://draveness.me/golang/tree/runtime.deferproc) 函数；
2. 当使用 go 关键字时，插入 [`runtime.newproc`](https://draveness.me/golang/tree/runtime.newproc) 函数符号；
3. 在遇到其他情况时会插入表示普通函数对应的符号；

[`cmd/compile/internal/gc/ssa.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/ssa.go) 这个拥有将近 7000 行代码的文件包含用于处理不同节点的各种方法，编译器会根据节点类型的不同在一个巨型 switch 语句处理不同的情况，这也是我们在编译器这种独特的场景下才能看到的现象。

```go
compiling hello
hello func(int) int
  b1:
    v1 = InitMem <mem>
    v2 = SP <uintptr>
    v3 = SB <uintptr> DEAD
    v4 = LocalAddr <*int> {a} v2 v1 DEAD
    v5 = LocalAddr <*int> {~r1} v2 v1
    v6 = Arg <int> {a}
    v7 = Const64 <int> [0] DEAD
    v8 = Const64 <int> [2]
    v9 = Add64 <int> v6 v8 (c[int])
    v10 = VarDef <mem> {~r1} v1
    v11 = Store <mem> {int} v5 v9 v10
    Ret v11
```

上述代码就是在这个过程生成的，你可以看到中间代码主体中的每一行都定义了一个新的变量，这是我们在前面提到的具有静态单赋值（SSA）特性的中间代码，如果你使用 `GOSSAFUNC=hello go build hello.go` 命令亲自编译一下会对这种中间代码有更深的印象。

##### 多轮转换

虽然我们在 [`cmd/compile/internal/gc.state.stmt`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.stmt) 以及相关方法中生成了 SSA 中间代码，但是这些中间代码仍然需要编译器优化以去掉无用代码并精简操作数，编译器优化中间代码的过程都是由 [`cmd/compile/internal/ssa.Compile`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.Compile) 函数执行的：

```go
func Compile(f *Func) {
	if f.Log() {
		f.Logf("compiling %s\n", f.Name)
	}

	phaseName := "init"

	for _, p := range passes {
		f.pass = &p
		p.fn(f)
	}

	phaseName = ""
}
```

上述函数删除了很多打印日志和性能分析的代码，SSA 需要经历的多轮处理也都保存在了 `passes` 变量中，这个变量中存储了每一轮处理的名字、使用的函数以及表示是否必要的 `required` 字段：

```go
var passes = [...]pass{
	{name: "number lines", fn: numberLines, required: true},
	{name: "early phielim", fn: phielim},
	{name: "early copyelim", fn: copyelim},
	...
	{name: "loop rotate", fn: loopRotate},
	{name: "stackframe", fn: stackframe, required: true},
	{name: "trim", fn: trim},
}
```

目前的编译器总共引入了将近 50 个需要执行的过程，我们能在 `GOSSAFUNC=hello go build hello.go` 命令生成的文件中看到每一轮处理后的中间代码，例如最后一个 `trim` 阶段就生成了如下的 SSA 代码：

```go
  pass trim begin
  pass trim end [738 ns]
hello func(int) int
  b1:
    v1 = InitMem <mem>
    v10 = VarDef <mem> {~r1} v1
    v2 = SP <uintptr> : SP
    v6 = Arg <int> {a} : a[int]
    v8 = LoadReg <int> v6 : AX
    v9 = ADDQconst <int> [2] v8 : AX (c[int])
    v11 = MOVQstore <mem> {~r1} v2 v9 v10
    Ret v11
```

经过将近 50 轮处理的中间代码相比处理之前有了非常大的改变，执行效率会有比较大的提升，多轮的处理已经包含了一些机器特定的修改，包括根据目标架构对代码进行改写，不过这里就不会展开介绍每一轮处理的内容了。



#### 小结

中间代码的生成过程是从 AST 抽象语法树到 SSA 中间代码的转换过程，在这期间会对语法树中的关键字再进行改写，改写后的语法树会经过多轮处理转变成最后的 SSA 中间代码，相关代码中包括了大量 switch 语句、复杂的函数和调用栈，阅读和分析起来也非常困难。

很多 Go 语言中的关键字和内置函数都是在这个阶段被转换成运行时包中方法的，作者在后面的章节会从具体的语言关键字和内置函数的角度介绍一些数据结构和内置函数的实现。

---



### 机器码生成

Go 语言编译的最后一个阶段是根据 SSA 中间代码生成机器码，这里谈的机器码是在目标 CPU 架构上能够运行的二进制代码，[中间代码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)一节简单介绍的从抽象语法树到 SSA 中间代码的生成过程，将近 50 个生成中间代码的步骤中有一些过程严格上说是属于机器码生成阶段的。

机器码的生成过程其实是对 SSA 中间代码的降级（lower）过程，在 SSA 中间代码降级的过程中，编译器将一些值重写成了目标 CPU 架构的特定值，降级的过程处理了所有机器特定的重写规则并对代码进行了一定程度的优化；在 SSA 中间代码生成阶段的最后，Go 函数体的代码会被转换成 [`cmd/compile/internal/obj.Prog`](https://draveness.me/golang/tree/cmd/compile/internal/obj.Prog) 结构。



#### 指令集架构

首先需要介绍的就是指令集架构，虽然我们在第一节[编译过程概述](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/)中曾经讲解过指令集架构，但是在这里还是需要引入更多的指令集架构知识。

![instruction-set-architecture](Images/2019-02-08-instruction-set-architecture.png)

[指令集架构](https://en.wikipedia.org/wiki/Instruction_set_architecture)是计算机的抽象模型，在很多时候也被称作架构或者计算机架构，它是计算机软件和硬件之间的接口和桥梁[1](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-machinecode/#fn:1)；一个为特定指令集架构编写的应用程序能够运行在所有支持这种指令集架构的机器上，也就是说如果当前应用程序支持 x86 的指令集，那么就可以运行在所有使用 x86 指令集的机器上，这其实就是抽象层的作用，每一个指令集架构都定义了支持的数据结构、寄存器、管理主内存的硬件支持（例如内存一致、地址模型和虚拟内存）、支持的指令集和 IO 模型，它的引入其实就在软件和硬件之间引入了一个抽象层，让同一个二进制文件能够在不同版本的硬件上运行。

如果一个编程语言想要在所有的机器上运行，它就可以将中间代码转换成使用不同指令集架构的机器码，这可比为不同硬件单独移植要简单的太多了。

![cisc-and-ris](Images/2019-12-26-15773759175895-cisc-and-risc.png)

最常见的指令集架构分类方法是根据指令的复杂度将其分为复杂指令集（CISC）和精简指令集（RISC），复杂指令集架构包含了很多特定的指令，但是其中的一些指令很少会被程序使用，而精简指令集只实现了经常被使用的指令，不常用的操作都会通过组合简单指令来实现。

[复杂指令集](https://en.wikipedia.org/wiki/Complex_instruction_set_computer)的特点就是指令数目多并且复杂，每条指令的字节长度并不相等，x86 就是常见的复杂指令集处理器，它的指令长度大小范围非常广，从 1 到 15 字节不等，对于长度不固定的指令，计算机必须额外对指令进行判断，这需要付出额外的性能损失[2](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-machinecode/#fn:2)。

而[精简指令集](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer)对指令的数目和寻址方式做了精简，大大减少指令数量的同时更容易实现，指令集中的每一个指令都使用标准的字节长度、执行时间相比复杂指令集会少很多，处理器在处理指令时也可以流水执行，提高了对并行的支持。作为一种常见的精简指令集处理器，arm 使用 4 个字节作为指令的固定长度，省略了判断指令的性能损失[3](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-machinecode/#fn:3)，精简指令集其实就是利用了我们耳熟能详的 20/80 原则，用 20% 的基础指令和它们的组合来解决问题。

最开始的计算机使用复杂指令集是因为当时计算机的性能和内存比较有限，业界需要尽可能地减少机器需要执行的指令，所以更倾向于高度编码、长度不等以及多操作数的指令。不过随着计算机性能的提升，出现了精简指令集这种牺牲代码密度换取简单实现的设计；除此之外，硬件的飞速提升还带来了更多的寄存器和更高的时钟频率，软件开发人员也不再直接接触汇编代码，而是通过编译器和汇编器生成指令，复杂的机器指令对于编译器来说很难利用，所以精简指令在这种场景下更适合。

复杂指令集和精简指令集的使用是设计上的权衡，经过这么多年的发展，两种指令集也相互借鉴和学习，与最开始刚被设计出来时已经有了较大的差别，对于软件工程师来讲，复杂的硬件设备对于我们来说已经是领域下三层的知识了，其实不太需要掌握太多，但是对指令集架构感兴趣的读者可以找一些资料开拓眼界。



#### 机器码生成

机器码的生成在 Go 的编译器中主要由两部分协同工作，其中一部分是负责 SSA 中间代码降级和根据目标架构进行特定处理的 [`cmd/compile/internal/ssa`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/ssa) 包，另一部分是负责生成机器码的 [`cmd/internal/obj`](https://github.com/golang/go/tree/master/src/cmd/internal/obj)[4](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-machinecode/#fn:4)：

- [`cmd/compile/internal/ssa`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/ssa) 主要负责对 SSA 中间代码进行降级、执行架构特定的优化和重写并生成 [`cmd/compile/internal/obj.Prog`](https://draveness.me/golang/tree/cmd/compile/internal/obj.Prog) 指令；
- [`cmd/internal/obj`](https://github.com/golang/go/tree/master/src/cmd/internal/obj) 作为汇编器会将这些指令转换成机器码完成这次编译；

##### SSA 降级

SSA 降级是在中间代码生成的过程中完成的，其中将近 50 轮处理的过程中，`lower` 以及后面的阶段都属于 SSA 降级这一过程，这么多轮的处理会将 SSA 转换成机器特定的操作：

```go
var passes = [...]pass{
	...
	{name: "lower", fn: lower, required: true},
	{name: "lowered deadcode for cse", fn: deadcode}, // deadcode immediately before CSE avoids CSE making dead values live again
	{name: "lowered cse", fn: cse},
	...
	{name: "trim", fn: trim}, // remove empty blocks
}
```

SSA 降级执行的第一个阶段就是 `lower`，该阶段的入口方法是 [`cmd/compile/internal/ssa.lower`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.lower) 函数，它会将 SSA 的中间代码转换成机器特定的指令：

```go
func lower(f *Func) {
	applyRewrite(f, f.Config.lowerBlock, f.Config.lowerValue)
}
```

向 [`cmd/compile/internal/ssa.applyRewrite`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.applyRewrite) 传入的两个函数 `lowerBlock` 和 `lowerValue` 是在[中间代码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)阶段初始化 SSA 配置时确定的，这两个函数会分别转换函数中的代码块和代码块中的值。

假设目标机器使用 x86 的架构，最终会调用 [`cmd/compile/internal/ssa.rewriteBlock386`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.rewriteBlock386) 和 [`cmd/compile/internal/ssa.rewriteValue386`](https://draveness.me/golang/tree/cmd/compile/internal/ssa.rewriteValue386) 两个函数，这两个函数是两个巨大的 switch 语句，前者总共有 2000 多行，后者将近 700 行，用于处理 x86 架构重写的函数总共有将近 30000 行代码，你能在 [`cmd/compile/internal/ssa/rewrite386.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/ssa/rewrite386.go) 这里找到文件的全部内容，我们只节选其中的一段展示一下：

```go
func rewriteValue386(v *Value) bool {
	switch v.Op {
	case Op386ADCL:
		return rewriteValue386_Op386ADCL_0(v)
	case Op386ADDL:
		return rewriteValue386_Op386ADDL_0(v) || rewriteValue386_Op386ADDL_10(v) || rewriteValue386_Op386ADDL_20(v)
	...
	}
}

func rewriteValue386_Op386ADCL_0(v *Value) bool {
	// match: (ADCL x (MOVLconst [c]) f)
	// cond:
	// result: (ADCLconst [c] x f)
	for {
		_ = v.Args[2]
		x := v.Args[0]
		v_1 := v.Args[1]
		if v_1.Op != Op386MOVLconst {
			break
		}
		c := v_1.AuxInt
		f := v.Args[2]
		v.reset(Op386ADCLconst)
		v.AuxInt = c
		v.AddArg(x)
		v.AddArg(f)
		return true
	}
	...
}
```

重写的过程会将通用的 SSA 中间代码转换成目标架构特定的指令，上述的 `rewriteValue386_Op386ADCL_0` 函数会使用 `ADCLconst` 替换 `ADCL` 和 `MOVLconst` 两条指令，它能通过对指令的压缩和优化减少在目标硬件上执行所需要的时间和资源。

我们在上一节中间代码生成中已经介绍过 [`cmd/compile/internal/gc.compileSSA`](https://draveness.me/golang/tree/cmd/compile/internal/gc.compileSSA) 中调用 [`cmd/compile/internal/gc.buildssa`](https://draveness.me/golang/tree/cmd/compile/internal/gc.buildssa) 的执行过程，我们在这里继续介绍 [`cmd/compile/internal/gc.buildssa`](https://draveness.me/golang/tree/cmd/compile/internal/gc.buildssa) 函数返回后的逻辑：

```go
func compileSSA(fn *Node, worker int) {
	f := buildssa(fn, worker)
	pp := newProgs(fn, worker)
	defer pp.Free()
	genssa(f, pp)

	pp.Flush()
}
```

[`cmd/compile/internal/gc.genssa`](https://draveness.me/golang/tree/cmd/compile/internal/gc.genssa) 函数会创建一个新的 [`cmd/compile/internal/gc.Progs`](https://draveness.me/golang/tree/cmd/compile/internal/gc.Progs) 结构并将生成的 SSA 中间代码都存入新建的结构体中，我们在上一节得到的 ssa.html 文件就包含最后生成的中间代码：

![genssa](Images/2019-12-26-15773759175909-genssa.png)

上述输出结果跟最后生成的汇编代码已经非常相似了，随后调用的 [`cmd/compile/internal/gc.Progs.Flush`](https://draveness.me/golang/tree/cmd/compile/internal/gc.Progs.Flush) 会使用 [`cmd/internal/obj`](https://github.com/golang/go/tree/master/src/cmd/internal/obj) 包中的汇编器将 SSA 转换成汇编代码：

```go
func (pp *Progs) Flush() {
	plist := &obj.Plist{Firstpc: pp.Text, Curfn: pp.curfn}
	obj.Flushplist(Ctxt, plist, pp.NewProg, myimportpath)
}
```

[`cmd/compile/internal/gc.buildssa`](https://draveness.me/golang/tree/cmd/compile/internal/gc.buildssa) 中的 `lower` 和随后的多个阶段会对 SSA 进行转换、检查和优化，生成机器特定的中间代码，接下来通过 [`cmd/compile/internal/gc.genssa`](https://draveness.me/golang/tree/cmd/compile/internal/gc.genssa) 将代码输出到 [`cmd/compile/internal/gc.Progs`](https://draveness.me/golang/tree/cmd/compile/internal/gc.Progs) 对象中，这也是代码进入汇编器前的最后一个步骤。

##### 汇编器

汇编器是将汇编语言翻译为机器语言的程序，Go 语言的汇编器是基于 Plan 9 汇编器的输入类型设计的，Go 语言对于汇编语言 Plan 9 和汇编器的资料十分缺乏，网上能够找到的资料也大多都含糊不清，官方对汇编器在不同处理器架构上的实现细节也没有明确定义：

> The details vary with architecture, and we apologize for the imprecision; the situation is **not well-defined**.[5](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-machinecode/#fn:5)

我们在研究汇编器和汇编语言时不应该陷入细节，只需要理解汇编语言的执行逻辑就能够帮助我们快速读懂汇编代码。当我们将如下的代码编译成汇编指令时，会得到如下的内容：

```go
$ cat hello.go
package hello

func hello(a int) int {
	c := a + 2
	return c
}
$ GOOS=linux GOARCH=amd64 go tool compile -S hello.go
"".hello STEXT nosplit size=15 args=0x10 locals=0x0
	0x0000 00000 (main.go:3)	TEXT	"".hello(SB), NOSPLIT, $0-16
	0x0000 00000 (main.go:3)	FUNCDATA	$0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (main.go:3)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (main.go:3)	FUNCDATA	$3, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (main.go:4)	PCDATA	$2, $0
	0x0000 00000 (main.go:4)	PCDATA	$0, $0
	0x0000 00000 (main.go:4)	MOVQ	"".a+8(SP), AX
	0x0005 00005 (main.go:4)	ADDQ	$2, AX
	0x0009 00009 (main.go:5)	MOVQ	AX, "".~r1+16(SP)
	0x000e 00014 (main.go:5)	RET
	0x0000 48 8b 44 24 08 48 83 c0 02 48 89 44 24 10 c3     H.D$.H...H.D$..
...
```

上述汇编代码都是由 [`cmd/internal/obj.Flushplist`](https://draveness.me/golang/tree/cmd/internal/obj.Flushplist) 这个函数生成的，该函数会调用架构特定的 `Preprocess` 和 `Assemble` 方法：

```go
func Flushplist(ctxt *Link, plist *Plist, newprog ProgAlloc, myimportpath string) {
	...

	for _, s := range text {
		mkfwd(s)
		linkpatch(ctxt, s, newprog)
		ctxt.Arch.Preprocess(ctxt, s, newprog)
		ctxt.Arch.Assemble(ctxt, s, newprog)
		linkpcln(ctxt, s)
		ctxt.populateDWARF(plist.Curfn, s, myimportpath)
	}
}
```

Go 编译器会在最外层的主函数确定调用的 `Preprocess` 和 `Assemble` 方法，编译器在 2.1.4 中提到的 [`cmd/compile.archInits`](https://draveness.me/golang/tree/cmd/compile.archInits) 中根据目标硬件初始化当前架构使用的配置。

如果目标机器的架构是 x86，那么这两个函数最终会使用 [`cmd/internal/obj/x86.preprocess`](https://draveness.me/golang/tree/cmd/internal/obj/x86.preprocess) 和 [`cmd/internal/obj/x86.span6`](https://draveness.me/golang/tree/cmd/internal/obj/x86.span6)，作者在这里就不展开介绍这两个特别复杂的底层函数了，有兴趣的读者可以通过链接找到目标函数的位置了解预处理和汇编的处理过程，机器码的生成也都是由这两个函数组合完成的。



#### 小结

机器码生成作为 Go 语言编译的最后一步，其实已经到了硬件和机器指令这一层，其中对于内存、寄存器的处理非常复杂并且难以阅读，想要真正掌握这里的处理的步骤和原理还是需要耗费很多精力。

作为软件工程师，如果不是 Go 语言编译器的开发者或者需要经常处理汇编语言和机器指令，掌握这些知识的投资回报率实在太低，我们只需要对这个过程有所了解，补全知识上的盲点，在遇到问题时能够快速定位即可。