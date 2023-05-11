## 一些先验知识（新特性）



### Java Record

不可继承（类似于`final`），不可修改（属性`final`）的类（更加类似于不能修改的**结构体**），直接通过`.`访问属性，默认创建全参数的构造函数，可以创建缺少部分或者全部参数的构造函数，以及其他功能函数，包括静态函数。

```java
public record Student(String name, Integer age) {}
```



### Text Block 文本块

**"""..."""**类似于go中的**``**来表示整段的文字，但是仍可以使用转义字符和占位字符。

```java
String colors= """
red
green
blue
""";
```



### var 推断式声明变量

向解释型编程语言借鉴的方法，类似于go中的`:=`，C++中的`auto`或者JS中的`var`，但是类型还是强约束的类型，只不过做了自动推断。一般用于以下场景：

1. 简单的临时变量
2. 复杂，多步骤逻辑，嵌套的表达式等，简短的变量有助理解代码
3. 能够确定变量初始值
4. 变量类型比较长时



### sealed 密封类

主要思想是有限继承（`final`不能继承，`private`限制为私有），`sealed`关键词结合`permits`关键词定义限制继承的密封类。

```java
public sealed class Shape permits Circle, Square, Rectangle {
    private Integer width;
    private Integer height;
}
```

子类声明有三种：

- `final`终结，依然是密封的
- `sealed`子类是密封类，需要子类实现
- `non-sealed`非密封类，扩展使用，不受限

密封接口同密封类。



