# Go-learning

- Golang~~复习~~（学习）笔记，只有部分内容~

### todo

- [ ] rate limiting
- [ ] staterful goroutines
- [ ] text/template

## 目录

1. [切片](#切片（slices）)
2. [范围遍历](#范围遍历（range）)
3. [字符串（string），字节数组（bytes），符文（runes）和字符（characters）](#字符串（string），字节数组（bytes），符文（runes）和字符（characters）)



### 切片（slices）

- Golang的切片类型类似于数组类型，但是相较于数组类型，更加灵活，golang的切片类型加入了类似于列表（list）的功能，使切片类型更加灵活可用；

- 切片类型是在golang数组类型之上构建的抽象类型；



#### 参考

- https://tip.golang.org/ref/spec
- https://go.dev/blog/slices-intro
- https://go.dev/doc/effective_go#slices



#### 数组（arrays）

- 数组类型需要指定数据类型和数据长度，因此`[4]int`和`[5]int`是完全不同的两个类型，之间无法闲事比较；

- 数组从下标0开始索引，就像其他语言的一样；

- 数组不需要显式地初始化，当指明类型时，会生成元素全为0的数组（自动初始化）；

  ```go
  var a [4]int
  // a == [0,0,0,0]
  ```

  ![img](slice-array.png)

- 数组在作为参数传递时，不是将数组变量看作为一个指向数组的指针，这是C语言中的做法，而是看作一个整体，这就意味着当传递参数时，会引起复制机制。如果需要规避这种复制，通用的做法是传递指向数组的指针；

  ```go
  var a [4]int
  for i := 0; i < len(a); i++ {
  a[i] = i
  }
  // [0,1,2,3]
  reverse(&a)
  // [3,2,1,0]
  ```

- 数组拥有不同的初始化方法

  ```go
  a := [4]int{1, 2, 3, 4}
  a := [...]int{1, 2, 3, 4}
  ```



#### 切片（slices）

- 在golang中最常见的“数组”数据结构，切片的格式为`[]T`，其中`T`是切片元素的类型，同样，切片不指定长度；

  ```go
  nums := []int{1, 2, 3, 4}
  ```

- 切片可以用内置函数`make`来初始化，有以下几个参数，[]`T`代表了切片数据类型，`len`代表了初始化长度，`cap`是数组初始化容量，返回的是分配了一个数组的切片；

  ```go
  func make([]T, len, cap) []T
  ```

- 当然，`cap`可以通过`len`推测出，所以可以省略；

- 切片支持”切片“类型的访问，采用左闭右开区间的方法，这和python中数组的处理方式类似；

  ```go
  b := []byte{'a','b','c','d','e','f'}
  // b[1:4] == []byte{'b','c','d'}
  ```



#### 类型转换

- **数组转切片**

  ``` go
  a := [3]int{1, 2, 3}
  s := a[:] // slice
  ```

- **切片转数组**（v1.71+）

  ```go
  s := make([]byte, 2, 4)
  s0 := (*[0]byte)(s)      // s0 != nil
  s1 := (*[1]byte)(s[1:])  // &s1[0] == &s[1]
  s2 := (*[2]byte)(s)      // &s2[0] == &s[0]
  s4 := (*[4]byte)(s)      // panics: len([4]byte) > len(s)
  ```



#### 切片的内部构造

- 切片是数组段的描述符。它由指向数组的指针、段的长度和它的容量（段的最大长度）组成：

  ![img](slice-struct.png)

- 假设变量`s`，使用`make([]byte, 5)`初始化，那么切片就是这样的：

  ![img](slice-1.png)

- `len`是切片所引用的元素的数量，`cap`是底层数组中的元素数量（从切片指针引用的元素开始）：

  ```go
  s = s[2:4]
  ```

  ![img](slice-2.png)

- 切片并不复制切片的数据，它创建一个指向原始数组的新片值，这使得切片操作与操作数组索引一样有效。因此，修改重切片的元素（而不是切片本身）会修改原始切片的元素：

  ```go
  a := []int{1, 2, 3, 4}
  b := a[2:] // b == []int{3, 4}
  b[1] := 1  // b == []int{1, 4}
  // a == []int{1, 2, 1, 4}
  ```

- 之前我们切片了`s` 的一部分，但是完全没有到达它底层的最大长度，这就意味着，切片能恢复到它以前的结尾，即长度增长：

  ```go
  s := s[:cap(s)]
  ```

  ![img](slice-3.png)

- 切片不能超出其容量，因为尝试超过容量的切片行为将导致运行时产生`panic`，就像在数组越界一样。类似的，切片不能在0以下重新切片，以访问数组中更早的元素。



#### 安全的切片扩容（`copy` & `append`）

- 为了增加切片的容量，必须创建一个更大的新切片，并将原始切片的内容复制到其中，这项技术在大多数语言中由动态数组在幕后实现：

  ```go
  t := make([]byte, len(s), (cap(s)+1)*2)
  copy(t, s)
  s = t
  ```

- 当不需要关注性能时，可以使用内置的函数`append`进行扩容：

  ```go
  func append(s []T, x ...T) []T
  ```

  ```go
  a := make([]int, 1)
  // a == []int{0}
  a = append(a, 1, 2, 3)
  // a == []int{0, 1, 2, 3}
  b := []int{4, 5, 6}
  a = append(a, b...)
  // a == []int{0, 1, 2, 3, 4, 5, 6}
  ```

  

#### 要点

- 如前所述，重新切片一个切片并不会复制底层数组，整个数组将保存在内存中，直到不再被引用。有时候，这可能会导致程序在只需要一小部分数据时将所有数据保存在内存中：

  ```go
  var digitRegexp = regexp.MustCompile("[0-9]+")
  
  func FindDigits(filename string) []byte {
      b, _ := ioutil.ReadFile(filename)
      return digitRegexp.Find(b)
  }
  ```

  这段代码的返回的`[]byte`指向包含整个文件的数组。由于切片引用的是原始数组，所以只要切片被没有超出适用范围，哪怕是只需要使用几个字节，垃圾收集器就不能释放数组。为了解决这个问题，我们需要将一部分数据复制到其他的切片中，而不是对其进行引用：

  ```go
  func CopyDigits(filename string) []byte {
      b, _ := ioutil.ReadFile(filename)
      b = digitRegexp.Find(b)
      c := make([]byte, len(b))
      copy(c, b)
      return c
  }
  ```

---



### 范围遍历（range）

- `for-range`是golang里常见的范围便利方法，可以遍历`array`，`slice`，`map`，`string`；

  ```go
  nums := []int{2, 3, 4}
  // idx & num
  for idx, num := range nums {
  	...
  }
  // num
  for _, num := range nums {
  	...
  }
  ```

  ```go
  kvs := map[int]int{1: 2, 2: 3, 3: 4}
  // key-value
  for k, v := range kvs {
  	...
  }
  // key
  for k := range kvs {
  	...
  }
  ```

  ```go
  for i, c := range "go" {
  	...
  }
  ```

---



### 字符串（string），字节数组（bytes），符文（runes）和字符（characters）

- 以下是不同的”字符“表达，各有不同：
  - byte, character, rune
  - Unicode, UTF-8
  - string, string literal



#### 参考

- https://go.dev/blog/strings



#### 字符串（string）

- 字符串是一个只读的字节切片，只不过该字节切片并不关心其存储的内容是什么，只要是一些字节即可，也可以看作是一些`uint8`，即`0x00~0xFF`：

  ```go
  const s = "\xbd\xb2\x3d\xbc\x20\xe2\x8c\x98"
  // print s -> "��=� ⌘"
  ```

  如果需要逐字节读取：

  ```go
  for i := 0; i < len(s); i++ {
  	fmt.Printf("%x ", s[i])
  }
  // print s -> "bd b2 3d bc 20 e2 8c 98"
  
  fmt.Printf("%x\n", sample) // 同样是十六进制
  // print s -> "bdb23dbc20e28c98"
  
  fmt.Printf("% x\n", sample) // 多加了个空格，不错的格式化方法
  // print s -> "bd b2 3d bc 20 e2 8c 98"
  
  fmt.Printf("%q\n", sample) // 只显示有效字符，其余用十六进制表示
  // print s -> "\xbd\xb2=\xbc ⌘"
  
  fmt.Printf("%+q\n", sample) // 可以看到上一种显示中的一些端倪
  // print s -> "\xbd\xb2=\xbc \u2318"
  ```



#### UTF-8和string literals（字符串字面值）

- golang的源代码是以UTF-8来编码的，那么UTF-8的字符串是如何转换为字节字符串呢，看一个例子，在之前的内容里出现了一个特殊字符`⌘`，这代表了一种名胜古迹，这种特殊字符并没有出现在ASCII码中：

  ```go
  const placeOfInterest = `⌘`
  fmt.Printf("plain string: %s\n", placeOfInterest)
  fmt.Printf("quoted string: %+q\n", placeOfInterest)
  fmt.Printf("hex bytes: % x\n", placeOfInterest)
  
  // print placeOfInterest -> 
  plain string: 	⌘
  quoted string: 	"\u2318"
  hex bytes: 		e2 8c 98  
  ```

  所以，golang将特殊字符UTF-8转换成通用的字节字符串存储，但是golang不仅仅只有一种编码方式，只不过显示的时候都是使用UTF-8而已，所以这种显示就被称之为字符串字面值（string literals）。



#### 符文（rune）

- 码点（code point）在Unicode中的含义与符文（rune）在golang中是一致的，它代表了一个复杂的字符（chararcter），这里的字符不再是简单的英文字符，也有可能是中文等，一个符文在golang里处理成一个`int32`类型：

  ```go
  const s = `安倍晋三挂了` // 6个rune，长度为18而非6
  
  for index, runeValue := range s {
  	fmt.Printf("%#U starts at byte position %d\n", runeValue, index)
  }
  
  for i, w := 0, 0; i < len(s); i += w {
      runeValue, width := utf8.DecodeRuneInString(s[i:])
      fmt.Printf("%#U starts at byte position %d\n", runeValue, i)
      w = width
  }
  
  // same print ->
  U+5B89 '安' starts at byte position 0
  U+500D '倍' starts at byte position 3 
  U+664B '晋' starts at byte position 6 
  U+4E09 '三' starts at byte position 9 
  U+6302 '挂' starts at byte position 12
  U+4E86 '了' starts at byte position 15
  ```


---



### 可变参数函数

- 参数个数可变化，从无参数到数个参数

```go
func sum(args ...int) {
	ans := 0
	for _, arg := range args {
		ans += arg
	}
	fmt.Println(args, ans)
}

sum()
// [] 0
sum(1, 2, 3)
// [1 2 3] 6
sum([]int{1, 2, 3, 4}...)
// [1 2 3 4] 10
```

---



### 函数闭包

- 匿名函数，每个函数闭包参数之间相互独立，但是一个函数闭包之间共享参数

```go
func initFunc() func() int {
	i := 0
	return func() int {
		i++
		return i
	}
}

f1 := initFunc()	// init
fmt.Println(f1())	// 1
fmt.Println(f1())	// 2
fmt.Println(f1())	// 3
f2 := initFunc()	// init
fmt.Println(f2())	// 1
```

---



### 结构体封装（继承）

- 在go中没有显式的继承这一说法，但是有个东西可以看作是继承，而且还是多继承，同时也可以重写函数方法

```go
type Animal interface {
	Speak() string
}

type Dog struct {
	Age int
}

func (d *Dog) Speak() string {
	return "汪"
}

func (d *Dog) Jump() string {
	return "0"
}

type Husky struct {
	Dog
	Name string
    Age	 int
}

func (h *Husky) Speak() string {
	return "嗷呜"
}

func (h *Husky) Jump() string {
	return "2"
}
```

- `Husky`这个数据结构继承了`Dog`，同时也实现了`Animal`这个接口，还实现了`Animal`的`Speak`方法，重写了`Dog`的`Jump`方法，跑一下：

```go
james := &Husky{Dog{Age: 3}, "james", 2}
println(james.Speak(), james.Dog.Speak())	// 嗷呜 汪
println(james.Age, james.Dog.Age)			// 2 3
println(james.Jump(), james.Dog.Jump())		// 2 0
```

- 显然都得到了重写，但是还能调用基类的属性和方法
- 如果我修改了基类的属性字段，变成了事实上的私有属性`age`，那么，在包外是无法访问的，无论是

```go
type Dog struct {
	age int
}

println(james.Age, james.Dog.age)
```

---



### Go泛型

- 类似于C++和Java的泛型，Go在1.18以后也支持起了泛型，泛型是很优雅的编程方式，虽然会带来一定的额外开销
- 两个关键词`comparable`和`any`

```go
func MapKeys[K comparable, T any](m map[K]T) []K {
	r := make([]K, 0, len(m))
	for k := range m {
		r = append(r, k)
	}
	return r
}

type List[T any] struct {
	head, tail *element[T]
}

type element[T any] struct {
	prev, next *element[T]
	val        T
}

func (l *List[T]) Push(v T) {
	if l.tail == nil {
		l.head = &element[T]{val: v}
		l.tail = l.head
	} else {
		l.tail.next = &element[T]{val: v, prev: l.tail}
		l.tail = l.tail.next
	}
}

func (l *List[T]) GetAll(reverse bool) []T {
	var elems []T
	if reverse {
		for e := l.tail; e != nil; e = e.prev {
			elems = append(elems, e.val)
		}
	} else {
		for e := l.head; e != nil; e = e.next {
			elems = append(elems, e.val)
		}
	}
	return elems
}
```

---



### 带`error`处理的`errgroup`

- 文档路径：https://pkg.go.dev/golang.org/x/sync/errgroup

- `errgroup`为处理公共任务的子任务的`goroutine`组提供了同步、错误传播和上下文取消

---



### `recover`现场恢复

- `recover()`函数用于在`panic()`之后的`defer`申明中

```go
package main

import "fmt"

func mayPanic() {
    panic("a problem")
}

func main() {

    defer func() {
        if r := recover(); r != nil {

            fmt.Println("Recovered. Error:\n", r)
        }
    }()

    mayPanic()

    fmt.Println("After mayPanic()")
}
```

---



### 标准化格式JSON和XML

- `json`: https://go.dev/blog/json
- `XML`: https://gobyexample.com/xml

```go
package main

import (
    "encoding/xml"
    "fmt"
)

type Plant struct {
    XMLName xml.Name `xml:"plant"`
    Id      int      `xml:"id,attr"`
    Name    string   `xml:"name"`
    Origin  []string `xml:"origin"`
}

func (p Plant) String() string {
    return fmt.Sprintf("Plant id=%v, name=%v, origin=%v",
        p.Id, p.Name, p.Origin)
}

func main() {
    coffee := &Plant{Id: 27, Name: "Coffee"}
    coffee.Origin = []string{"Ethiopia", "Brazil"}

    out, _ := xml.MarshalIndent(coffee, " ", "  ")
    fmt.Println(string(out))

    fmt.Println(xml.Header + string(out))

    var p Plant
    if err := xml.Unmarshal(out, &p); err != nil {
        panic(err)
    }
    fmt.Println(p)

    tomato := &Plant{Id: 81, Name: "Tomato"}
    tomato.Origin = []string{"Mexico", "California"}

    type Nesting struct {
        XMLName xml.Name `xml:"nesting"`
        Plants  []*Plant `xml:"parent>child>plant"`
    }

    nesting := &Nesting{}
    nesting.Plants = []*Plant{coffee, tomato}

    out, _ = xml.MarshalIndent(nesting, " ", "  ")
    fmt.Println(string(out))
}
```

---



### URL的自动解析

- https://adam.herokuapp.com/past/2010/3/30/urls_are_the_uniform_way_to_locate_resources/
- https://gobyexample.com/url-parsing





---

# Go-tricks

- 一些有用的Golang的小技巧，灵感来源于：https://github.com/rachitiitr/modern-cpp-tricks



### 带空格的数组输出

```go
func formatWithSpace(strs any) string {
	return strings.Trim(fmt.Sprint(strs), "[]")
}
// print 
1 2 3 4
```



### Padding

- 常规的

```go
func paddingRight(strs []string) {
	for _, str := range strs {
		fmt.Printf("| %10v |\n", str)
	}
}
// print
|       test |
|          a |
|       what |
```

```go
func PaddingLeft(strs []string) {
	for _, str := range strs {
		fmt.Printf("| %-10v |\n", str)
	}
}
// print
| test       |
| a          |
| what       |
```

- 浮动格式化输出

```go
func VariablePadding(tableLikeStrings [][]string) {
	// get number of columns from the first table row
	columnLengths := make([]int, len(tableLikeStrings[0]))
	for _, line := range tableLikeStrings {
		for i, val := range line {
			if len(val) > columnLengths[i] {
				columnLengths[i] = len(val)
			}
		}
	}

	var lineLength int
	for _, c := range columnLengths {
		lineLength += c + 3 // +3 for 3 additional characters before and after each field: "| %s "
	}
	lineLength += 1 // +1 for the last "|" in the line

	for i, line := range tableLikeStrings {
		if i == 0 { // table header
			fmt.Printf("+%s+\n", strings.Repeat("-", lineLength-2)) // lineLength-2 because of "+" as first and last character
		}
		for j, val := range line {
			fmt.Printf("| %-*s ", columnLengths[j], val)
			if j == len(line)-1 {
				fmt.Printf("|\n")
			}
		}
		if i == 0 || i == len(tableLikeStrings)-1 { // table header or last line
			fmt.Printf("+%s+\n", strings.Repeat("-", lineLength-2)) // lineLength-2 because of "+" as first and last character
		}
	}
}
```

- 总结

| format         | description                                                  |
| -------------- | ------------------------------------------------------------ |
| `"%s"`         | 标准格式输出                                                 |
| `"%*s"`        | 左对齐，浮动字符串长度，需要字符串长度和字符串两个参数（字符串长度在前） |
| `"%-*s"`       | 右对齐，浮动字符串长度，需要字符串长度和字符串两个参数（字符串长度在前） |
| `"%-[2]*[1]s"` | 右对齐，浮动字符串长度，需要字符串长度和字符串两个参数（字符串长度在后） |

- 使用标准库完成上述操作

```go
// NewWriter allocates and initializes a new tabwriter.Writer.
// The parameters are the same as for the Init function.
//
func NewWriter(output io.Writer, minwidth, tabwidth, padding int, padchar byte, flags uint) *Writer {
	return new(Writer).Init(output, minwidth, tabwidth, padding, padchar, flags)
}

// flags
const (
	// Ignore html tags and treat entities (starting with '&'
	// and ending in ';') as single characters (width = 1).
	FilterHTML uint = 1 << iota

	// Strip Escape characters bracketing escaped text segments
	// instead of passing them through unchanged with the text.
	StripEscape

	// Force right-alignment of cell content.
	// Default is left-alignment.
	AlignRight

	// Handle empty columns as if they were not present in
	// the input in the first place.
	DiscardEmptyColumns

	// Always use tabs for indentation columns (i.e., padding of
	// leading empty cells on the left) independent of padchar.
	TabIndent

	// Print a vertical bar ('|') between columns (after formatting).
	// Discarded columns appear as zero-width columns ("||").
	Debug
)

func VariablePaddingStandard(tableLikeStrings [][]string) {
	w := tabwriter.NewWriter(os.Stdout, 0, 4, 0, '\t', tabwriter.TabIndent)
	for _, line := range tableLikeStrings {
		fmt.Fprintln(w, strings.Join(line, "\t")+"\t")
	}
	w.Flush()
}
```

