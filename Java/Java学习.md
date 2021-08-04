# Java学习

## 基础

### finalize

finalize() 是在 java.lang.Object 里定义的，Object 的 finalize() 方法什么都不做，对象被回收时 finalize() 方法会被调用。

Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要清理工作，在垃圾收集器删除对象之前被调用的。一般情况下，此方法由JVM调用。特殊情况下，可重写 finalize() 方法，当对象被回收的时候释放一些资源，须调用 super.finalize() 。

---

### final

final 语义是不可改变的。

- 被 final 修饰的类，不能够被继承。
- 被 final 修饰的成员变量必须要初始化，赋初值后不能再重新赋值(可以调用对象方法修改属性值)。对基本类型来说是其值不可变；对引用变量来说其引用不可变，即不能再指向其他的对象。
- 被 final 修饰的方法代表不能重写。
- final 修饰基本类型变量，值不能改变
- final 修饰引用类型变量，栈内存中的引用不能改变，所指向的堆内存中的对象的属性值可能可以改变

---

### finally

- 代码提前直接返回；
- 前面代码异常throw抛出异常了；
- 系统退出（System.exit）；

---

### final & static

- 都可以修饰类、方法、成员变量。
- 都不能用于修饰构造方法。
- static 可以修饰类的代码块，final 不可以。
- static 不可以修饰方法内的局部变量，final 可以。

 

**static：**

- static 修饰表示静态或全局，被修饰的属性和方法属于类，可以用类名.静态属性 / 方法名 访问
- static 修饰的**代码块**表示**静态代码块**，当 Java 虚拟机（JVM）加载类时，就会执行该代码块,只会被执行一次
- static 修饰的属性，也就是类变量，是在类加载时被创建并进行初始化，只会被创建一次
- static 修饰的**变量**可以重新赋值
- static **方法**中不能用 this 和 super 关键字
- static **方法**必须被实现，而不能是抽象的abstract
- static **方法**不能被重写
- static 可以修饰变量、方法、代码块和内部类
- static 变量是这个类所有，由该类创建的所有对象共享同一个 static 属性
- 可以通过创建的对象名.属性名 和 类名.属性名两种方式访问
- static 变量在内存中只有一份
- static 修饰的变量只能是类的成员变量
- static 方法可以通过对象名.方法名和类名.方法名两种方式来访问
- static 代码块在类被第一次加载时执行静态代码块，且只被执行一次，主要作用是实现 static 属性的初始化
- static 内部类属于整个外部类，而不属于外部类的每个对象，只可以访问外部类的静态变量和方法

 

**static & not static:**

- 生命周期不同：非静态成员变量随着对象的创建而存在；静态成员变量随着类的加载而存在
- 调用方式不同：非静态成员变量用 对象名.变量名 调用；静态成员变量用 类名.变量名，JDK1.7以后也能用对象名.变量名调用
- 别名不同：非静态成员变量也称为实例变量；静态变量称为类变量
- 数据存储位置不同：成员变量数据存储在堆内存的对象中，对象的特有数据；静态变量数据存储在方法区(共享数据区)的静态区，对象的共享数据



**final：**

- final 修饰表示常量、一旦创建不可改变
- final 标记的成员**变量**必须在声明的同时赋值，或在该类的构造方法中赋值，不可以重新赋值
- final **方法**不能被子类重写
- final **类**不能被继承，没有子类，final 类中的方法默认是 final 的

---

### try和finally的return顺序

- 无论try里代码是否执行，finally都会执行；
- try里有return而finally没有，finally对return对象的修改无效；
- try里有return，finally里也有，以finally内的return为准；

---

### 字符串类

- **String**
  - final 修饰，String 类的方法都是返回 new String，即对 String 对象的任何改变都不影响到原对象，对字符串的修改操作都会生成新的对象；
- **StringBuffer**
  -  对字符串的操作的方法都加了synchronized，保证线程安全；
- **StringBuilder**
  - 不保证线程安全，在方法体内需要进行字符串的修改操作，可以 new StringBuilder 对象，调用 StringBuilder 对象的 append()、replace()、delete() 等方法修改字符串。
- **reverse**方法：
  - 使用 StringBuilder 或 StringBuffer 的 reverse 方法，本质都调用了它们的父类 AbstractStringBuilder 的 reverse 方法实现。（JDK1.8）

---

### 接口（interface）和抽象（abstract）的区别

- 抽象类可以有构造方法；接口中不能有构造方法。
- 抽象类中可以有普通成员变量；接口中没有普通成员变量。
- 抽象类中可以包含非抽象普通方法；接口中的所有方法必须都是抽象的。
- 抽象类中的抽象方法的访问权限可以是 public、protected 和 default；接口中的抽象方法只能是 public 类型的，并且默认即为 public abstract 类型。
- 抽象类中可以包含静态方法；JDK1.8 前接口中不能包含静态方法，JDK1.8 及以后可以包含已实现的静态方法。

- 抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量可以是任意访问权限；接口中变量默认且只能是 public static final 类型。
- 一个类可以实现多个接口，用逗号隔开，但只能继承一个抽象类。
- 接口不可以实现接口，但可以继承接口，并且可以继承多个接口，用逗号隔开。

---

### Java访问修饰符有哪些？权限的区别？

Java 语言中有四种权限访问控制符，能够控制类中成员变量和方法的可见性。

- public

  被 public 修饰的成员变量和方法可以在任何类中都能被访问到。
  被 public 修饰的类，在一个 java 源文件中只能有一个类被声明为 public ，而且一旦有一个类为 public ，那这个 java 源文件的文件名就必须要和这个被 public 所修饰的类的类名相同，否则编译不能通过。

- protected

  被 protected 修饰的成员会被位于同一 package 中的所有类访问到，也能被该类的所有子类继承下来。

- friendly

  默认，缺省的。在成员的前面不写访问修饰符的时候，默认就是友好的。
  同一package中的所有类都能访问。
  被 friendly 所修饰的成员只能被该类所在同一个 package 中的子类所继承下来。

- private

  私有的。只能在当前类中被访问到。

  ![img](https://www.javanav.com/aimgs/WechatIMG520__20191013115001.png)

---

### 内部类

- **成员内部类**

  类似于成员变量的申明方式，编译后产生两个class文件：

  - OuterClass.class && OuterClass$InnerClass.class

- **方法内部类**

  定义在方法内，编译后产生：

  - OuterClass.class && OuterClass$1Class.class

  - 只能在定义该内部类的方法内实例化；
  - 方法内部类对象不能使用该内部类所在方法的非final局部变量；
  - 当一个方法结束，其栈结构被删除，局部变量成为历史。但该方法结束后，在方法内创建的内部类对象可能仍然存在于堆中；

- **继承式匿名内部类**

  编译后：

  - Class.class && Class$1.class

- **接口式匿名内部类**

  接口式的匿名内部类是实现了一个接口的匿名类，感觉上实例化了一个接口：

  - Class.class && ClassImpl.class && ClassImpl$1.class

- **参数式匿名内部类**

  类自身没有传统的定义，而是通过匿名加上Override（or super）去实现继承；

- **静态嵌套类**

  - 静态嵌套类，并没有对实例的共享关系，仅仅是代码块在外部类内部
  - 静态的含义是该内部类可以像其他静态成员一样，没有外部类对象时，也能够访问它
  - 静态嵌套类仅能访问外部类的静态成员和方法
  - 在静态方法中定义的内部类也是静态嵌套类，这时候不能在类前面加static关键字

- **作用**

  - 内部类提供了某种进入其继承的类或实现的接口的窗口
  - 与外部类无关，独立继承其他类或实现接口
  - 内部类提供了Java的"多重继承"的解决方案，弥补了Java类是单继承的不足

- **特点**

  - 内部类仍然是一个独立的类，在编译之后内部类会被编译成独立的.class文件，但是前面冠以外部类的类名和$符号
  - 内部类不能用普通的方式访问。内部类是外部类的一个成员，因此内部类可以自由地访问外部类的成员变量，无论是否是private的
  - 内部类声明成静态的，就不能随便的访问外部类的成员变量了，此时内部类只能访问外部类的静态成员变量
  - 匿名内部类本质上是对父类方法的重写 或 接口的方法的实现；从语法角度看，匿名内部类创建处是无法使用关键字继承类 或 实现接口：
    - 匿名内部类没有名字，所以它没有构造函数。因为没有构造函数，所以它必须通过父类的构造函数来实例化。即匿名内部类完全把创建对象的任务交给了父类去完成。
    - 匿名内部类里创建新的方法没有太大意义，新方法无法被调用。
    - 匿名内部类一般是用来覆盖父类的方法。
    - 匿名内部类没有名字，所以无法进行向下的强制类型转换，只能持有匿名内部类对象引用的变量类型的直接或间接父类。

---

### Java反射

1. Class类：代表一个类
2. Field类：代表类的成员变量（类的属性）
   - getDeclaredFields(): 获取所有本类自己声明的属性, 不能获取父类和实现的接口中的属性
   - getFields(): 只能获取所有 public 声明的属性, 包括获取父类和实现的接口中的属性
3. Method类：代表类的方法
4. Constructor类：代表类的构造方法
5. Array类：提供了动态创建数组，以及访问数组的元素的静态方法

**使用场景**

- 在编译时无法知道该对象或类可能属于哪些类，程序在运行时获取对象和类的信息；

**作用**

- 通过反射可以使程序代码访问装载到 JVM 中的类的内部信息，获取已装载类的属性信息、方法信息；

**优点**

- 提高了 Java 程序的灵活性和扩展性，降低耦合性，提高自适应能力；
- 允许程序创建和控制任何类的对象，无需提前硬编码目标类
- 应用很广，测试工具、框架都用到了反射；

**缺点**

- 性能问题：反射是一种解释操作，远慢于直接代码。因此反射机制主要用在对灵活性和扩展性要求很高的系统框架上,普通程序不建议使用；
- 模糊程序内部逻辑：反射绕过了源代码，无法再源代码中看到程序的逻辑，会带来维护问题；
- 增大了复杂性：反射代码比同等功能的直接代码更复杂；

**被问实际应用：**

反射主要用于底层的框架中，Spring 中就大量使用了反射，比如：

- 用 IoC 来注入和组装 bean
- 动态代理、面向切面、bean 对象中的方法替换与增强，也使用了反射
- 定义的注解，也是通过反射查找

---

### 动态代理

**在运行时，创建目标类，可以调用和扩展目标类的方法。**

**Java 中实现动态的方式：**

- JDK 中的动态代理 
- Java类库 CGLib

**应用场景：**

- 统计每个 api 的请求耗时
- 统一的日志输出
- 校验被调用的 api 是否已经登录和权限鉴定
- Spring的 AOP 功能模块就是采用动态代理的机制来实现切面编程

---

### Java序列化

**序列化：将 Java 对象转换成字节流的过程。**

**反序列化：将字节流转换成 Java 对象的过程。**

- 当 Java 对象需要在网络上传输 或者 持久化存储到文件中时，就需要对 Java 对象进行序列化处理。

- 序列化的实现：类实现 Serializable 接口，这个接口没有需要实现的方法。实现 Serializable 接口是为了告诉 jvm 这个类的对象可以被序列化。

**注意事项：**

- 某个类可以被序列化，则其子类也可以被序列化
- 对象中的某个属性是对象类型，需要序列化也必须实现 Serializable 接口
- 声明为 static 和 transient 的成员变量，不能被序列化。static 成员变量是描述类级别的属性，transient 表示临时数据
- 反序列化读取序列化对象的顺序要保持一致
- **序列化id**：
  - SerialVersionUid 是为了序列化对象版本控制，告诉 JVM 各版本反序列化时是否兼容；
  - 如果在新版本中这个值修改了，新版本就不兼容旧版本，反序列化时会抛出InvalidClassException异常；
  - 仅增加了一个属性，希望向下兼容，老版本的数据都保留，就不用修改；
  - 删除了一个属性，或更改了类的继承关系，就不能不兼容旧数据，这时应该手动更新 SerialVersionUid；

---

### 对象克隆

- 实现Clonenable接口（和Serializable一样为无方法接口）
- 深克隆与浅克隆（对应C++里的深拷贝和浅拷贝）
- 嵌套深克隆可以用序列化解决，ByteArrayOutputStream做转存，InputStream读取oos && ois）；

---

### Java安全性

- 使用引用取代了指针，指针的功能强大，但是也容易造成错误，如数组越界问题。

- 拥有一套异常处理机制，使用关键字 throw、throws、try、catch、finally

- 强制类型转换需要符合一定规则

- 字节码传输使用了加密机制

- 运行环境提供保障机制：字节码校验器->类装载器->运行时内存布局->文件访问限制

- 不用程序员显示控制内存释放，JVM 有垃圾回收机制

---

### Java版本

- **J2SE**：

  **S**tandard **E**dition(标准版) ，包含 Java 语言的核心类。如IO、JDBC、工具类、网络编程相关类等。从JDK 5.0开始，改名为Java SE。

- **J2EE**：

  **E**nterprise **E**dition(企业版)，包含 J2SE 中的类和企业级应用开发的类。如**web**相关的servlet类、**JSP**、xml生成与解析的类等。从JDK 5.0开始，改名为Java EE。

- **J2ME**：

  **M**icro **E**dition(微型版)，包含 J2SE 中的部分类，新添加了一些专有类。一般用设备的嵌入式开发，如手机、机顶盒等。从JDK 5.0开始，改名为Java ME。

---

### JVM & JDK & JRE

- **JVM**
  - Java Virtual Machine（Java虚拟机）的缩写
  - 实现跨平台的最核心的部分
  - .class 文件会在 JVM 上执行，JVM 会解释给操作系统执行
  - 有自己的指令集，解释自己的指令集到 CPU 指令集和系统资源的调用
  - JVM 只关注被编译的 .class 文件，不关心 .java 源文件
- **JDK**
  - Java Development Kit（Java 开发工具包）的缩写。用于 java 程序的开发，提供给程序员使用
  - 使用 Java 语言编程都需要在计算机上安装一个 JDK
  - JDK 的安装目录 5 个文件夹、一个 src 类库源码压缩包和一些说明文件
    - bin：各种命令工具， java 源码的编译器 javac、监控工具 jconsole、分析工具 jvisualvm 等
    - include：与 JVM 交互C语言用的头文件
    - lib：类库    
    - jre：Java 运行环境 
    - db：安装 Java DB 的路径
    - src.zip：Java 所有核心类库的源代码
    - jdk1.8 新加了 javafx-src.zip 文件，存放 JavaFX 脚本，JavaFX 是一种声明式、静态类型编程语言
- **JRE**
  - Java Runtime Environment（Java运行环境）的缩写
  - 包含 JVM 标准实现及 Java 核心类库，这些是运行 Java 程序的必要组件
  - 是 Java 程序的运行环境，并不是一个开发环境，没有包含任何开发工具（如编译器和调试器）
  - 是运行基于 Java 语言编写的程序所不可缺少的运行环境，通过它，Java 程序才能正常运行
- **关系**
  - JDK 是 JAVA 程序开发时用的开发工具包，包含 Java 运行环境 JRE
  - JDk、JRE 内部都包含 JAVA虚拟机 JVM
  - JVM 包含 Java 应用程序的类的解释器和类加载器等

---

### 面向对象和面向过程

- 软件开发思想，先有面向过程，后有面向对象
- 在大型软件系统中，面向过程的做法不足，从而推出了面向对象
- 都是解决实际问题的思维方式
- 两者相辅相成，宏观上面向对象把握复杂事物的关系；微观上面向过程去处理
-  面向过程以实现功能的函数开发为主；面向对象要首先抽象出类、属性及其方法，然后通过实例化类、执行方法来完成功能
- 面向过程是封装的是功能；面向对象封装的是数据和功能
- 面向对象具有继承性和多态性；面向过程则没有

---

### 重载和重写

- **重写：**在子类中将父类的成员方法的名称保留，重新编写成员方法的实现内容，更改方法的访问权限，修改返回类型的为父类返回类型的子类。
- **一些规则：**
  - 重写发生在子类继承父类
  - 参数列表必须完全与被重写方法的相同
  - 重写父类方法时，修改方法的权限只能从小范围到大范围
  - 返回类型与被重写方法的返回类型可以不相同，但是必须是父类返回值的子类(JDK1.5 及更早版本返回类型要一样，JDK1.7 及更高版本可以不同)
  - 访问权限不能比父类中被重写的方法的访问权限更低。如：父类的方法被声明为 public，那么子类中重写该方法不能声明为 protected
  - 重写方法不能抛出新的检查异常和比被重写方法申明更宽泛的异常(即只能抛出父类方法抛出异常的子类)
  - 声明为 final 的方法不能被重写
  - 声明为 static 的方法不能被重写
  - 声明为 private 的方法不能被重写

- **重载：**一个类中允许同时存在一个以上的同名方法，这些方法的参数个数或者类型不同

- **重载条件：**
  - 方法名相同
  - 参数类型不同 或 参数个数不同 或 参数顺序不同

- **规则：**
  - 被重载的方法**参数列表(个数或类型)不一样
    **
  - **被重载的方法可以修改返回类型
    **
  - **被重载的方法可以修改访问修饰符
    **
  - **被重载的方法可以修改异常抛出
    **
  - **方法能够在同一个类中或者在一个子类中被重载
    **
  - **无法以返回值类型作为重载函数的区分标准**

 

- **重载和重写的区别：**
  - 作用范围：重写的作用范围是父类和子类之间；重载是发生在一个类里面
  - 参数列表：重载必须不同；重写不能修改
  - 返回类型：重载可修改；重写方法返回相同类型或子类
  - 抛出异常：重载可修改；重写可减少或删除，一定不能抛出新的或者更广的异常
  - 访问权限：重载可修改；重写一定不能做更严格的限制

---

### this & super

- **this：**
  - 对象内部指代自身的引用
  - 解决成员变量和局部变量同名问题
  - 可以调用成员变量
  - **不能**调用局部变量
  - 可以调用成员方法
  - 在普通方法中可以省略 this
  - 在**静态方法**当中不允许出现 this 关键字

- **super：**
  - 代表对当前对象的直接父类对象的引用
  - 可以调用父类的非 private 成员变量和方法
  - super(); 可以调用父类的构造方法，只限构造方法中使用，且必须是第一条语句

---

### abstract

- 可以修饰类和方法
- 不能修饰属性和构造方法
- abstract 修饰的类是抽象类，需要被继承
- abstract 修饰的方法是抽象方法，需要子类被重写

---

### 子类构造方法的调用规则：

- 如果子类的构造方法中没有通过 super 显式调用父类的有参构造方法，也没有通过 this 显式调用自身的其他构造方法，则系统会默认先调用父类的无参构造方法。这种情况下，写不写 super(); 语句，效果是一样的
- 如果子类的构造方法中通过 super 显式调用父类的有参构造方法，将执行父类相应的构造方法，不执行父类无参构造方法
- 如果子类的构造方法中通过 this 显式调用自身的其他构造方法，将执行类中相应的构造方法
- 如果存在多级继承关系，在创建一个子类对象时，以上规则会多次向更高一级父类应用，一直到执行顶级父类 Object 类的无参构造方法为止

---

### Java垃圾回收

![图摘自《码出高效》](https://upload-images.jianshu.io/upload_images/14923529-c0cbbccaa6858ca1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一个接口中的多个实现类需要的内存可能不一样，一个方法中的多个分支需要的内存也可能不一样，我们只有在程序处于运行期间时才能知道会创建哪些对象，这部分内存的分配和回收都是动态的，垃圾收集器所关注的是这部分内存。

- **回收判断**（2）

  - 引用计数法（缺点，循环引用问题，和auto_ptr一样的问题）；

  - 可达性分析算法（从GC Roots对象开始搜索，reference chain，当无相连时表示不可引用）；

  - 可作为GC Roots的潜在对象：

    - 虚拟机栈（栈帧中的本地变量表）中引用的对象；
    - 方法区中类静态属性引用的对象；
    - 方法区中常量引用的对象；
    - 本地方法栈中JNI（Native方法）引用的对象；

    作为 GC Roots 的节点主要在全局性的引用与执行上下文中。要明确的是，tracing gc必须以当前存活的对象集为 Roots，因此必须选取确定存活的引用类型对象。

    GC 管理的区域是 Java 堆，虚拟机栈、方法区和本地方法栈不被 GC 所管理，因此选用这些区域内引用的对象作为 GC Roots，是不会被 GC 所回收的。

    其中虚拟机栈和本地方法栈都是线程私有的内存区域，只要线程没有终止，就能确保它们中引用的对象的存活。而方法区中类静态属性引用的对象是显然存活的。常量引用的对象在当前可能存活，因此，也可能是 GC roots 的一部分。

- **强、弱、软、虚引用**

  ![图摘自《码出高效》](https://upload-images.jianshu.io/upload_images/14923529-3da4bc96c1ee8734.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 强引用就是指在程序代码之中普遍存在的，类似"Object obj=new Object()"这类的引用，垃圾收集器永远不会回收存活的强引用对象。
2. 软引用：还有用但并非必需的对象。在系统 **将要发生内存溢出异常之前** ，将会把这些对象列进回收范围之中进行第二次回收。
3. 弱引用也是用来描述非必需对象的，被弱引用关联的对象 **只能生存到下一次垃圾收集发生之前** 。当垃圾收集器工作时，无论内存是否足够，都会回收掉只被弱引用关联的对象。
4. 虚引用是最弱的一种引用关系。 **无法通过虚引用来取得一个对象实例** 。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。

- **Java堆永久代的回收**

  回收对象：

  - 废弃常量
  - 无用类：
    - 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。
    - 加载该类的 ClassLoader 已经被回收。
    - 该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

- **Java垃圾收集算法**

  - **标记-清除算法**

    Mark-Sweep，先标出要回收的，再一次性清除；（碎片过多）

  - **复制算法**（新生代中）

    容量大小一分为二，每次只使用其中一块，回收时将存活的复制到另一块上；（容量减半）

  - **标记-整理算法**（老年代）

    Mark-Compact，标记后移动存活对象至齐整，一次性清除边界外的内存；

  - **分代收集算法**

    Generational Collection，根据不同代（新生代，老年代）的特点采用不同的算法；

---

### 基本类和拆装箱

- 提供便捷方法，如类型转换，复杂处理；
- 基本类型方便、简单、高效，但泛型不支持、集合元素不支持（Map内键值都不能识别基本类型）；
- 不符合面向对象思维（？？？老人，地铁，手机）
- autoboxing和autounboxing，xxxValue()或是new构造方法

---

### Date类

- java.sql.Date是java.util.Date的子类；
- util包含时分秒毫秒；
- sql只有对应的日期部分，故读全要用java.sql.Timestamp；

---

### Java面向对象

**特性**：

- 抽象性：抽象是将一类对象的共同特征总结出来构造类的过程，包括数据抽象和行为抽象两方面。
- 继承性：指子类拥有父类的全部特征和行为，这是类之间的一种关系。Java 只支持单继承。
- 封装性：封装是将代码及其处理的数据绑定在一起的一种编程机制，该机制保证了程序和数据都不受外部干扰且不被误用。封装的目的在于保护信息。
- 多态性：多态性体现在父类的属性和方法被子类继承后或接口被实现类实现后，可以具有不同的属性或表现方式。

---

### 内存溢出和内存泄漏

- out of memory：没有足够的内存空间；
- memory leak：申请内存后无法释放已申请的内存空间；
- memory leak leads to out of memory；

---

### Java 创建对象的方式

1. 用 new 语句创建对象
2. 运用反射，调用 java.lang.Class 或 java.lang.reflect.Constructor 类的 newInstance() 方法
3. 调用对象的 clone() 方法
4. 运用反序列化手段，调用 java.io.ObjectInputStream 对象的 readObject() 方法

1、2 会调用构造函数
3、4 不会调用构造函数

---

### Java多态

**条件：**

- 继承
- 子类重写父类的方法
- 父类引用变量指向子类对象

**好处：**

- 消除类型之间的耦合关系
- 可替换性(substitutability)
- 可扩充性(extensibility)
- 接口性(interface-ability)
- 灵活性(flexibility)
- 简化性(simplicity)

---

### 同步代码块和同步方法区别

- 同步方法就是在方法前加关键字 synchronized；同步代码块则是在方法内部使用 synchronized
- 加锁对象相同的话，同步方法锁的范围大于等于同步方法块。一般加锁范围越大，性能越差
- 同步方法如果是 static 方法，等同于同步方法块加锁在该 Class 对象上

---

### 内部类和静态嵌套类

**Inner Class：内部类**

- 内部类就是在一个类的内部定义的类
- 内部类中不能定义静态成员
- 内部类可以直接访问外部类中的成员变量
- 内部类可以定义在外部类的方法外面，也可以定义在外部类的方法体中
- 在方法体外面定义的内部类的访问类型可以是 public, protected , 默认的，private 等 4 种类型
- 方法内部定义的内部类前面不能有访问类型修饰符，可以使用 final 或 abstract 修饰符
- 创建内部类的实例对象时，一定要先创建外部类的实例对象，然后用这个外部类的实例对象去创建内部类的实例对象
- 内部类里还包含匿名内部类，即直接对类或接口的方法进行实现，不用单独去定义内部类

```java
//内部类的创建语法
Outer outer = new Outer();
Outer.Inner inner = outer.new Innner();
```

 

**Static Nested Class：静态嵌套类**

- 不依赖于外部类的实例对象
- 不能直接访问外部类的非 static 成员变量
- 可以直接引用外部类的static的成员变量，不需要加上外部类的名字
- 在静态方法中定义的内部类也是Static Nested Class

```java
//静态内部类创建语法
Outter.Inner inner = new Outter.Inner();
```

---

### GB2312编码的字符串如何转换为ISO-8859-1编码？

```java
String strIso = new String(str.getBytes("GB2312"), "ISO-8859-1");
```

---

### Java 面向对象的设计原则

- **单一职责原则-Single Responsibility Principle（SRP）**

  一个对象一个只包含单一的职责，并且该职责被完整地封装在一个类中。

- **开闭原则-Open-closed Principle（OCP）**

  软件实体应该对拓展开放，对内修改关闭，分为抽象层（接口、抽象类）和实现层。

- **里氏代换原则-Liskov Substitution Principle（LSP）**

  所有引用基类的地方必须能透明地使用其子类的对象，即可以将一个基类对象替换成其子类的对象，程序不产生错误和异常。

- **依赖倒转原则-Dependence Inversion Principle（DIP）**
  高层模板不应该依赖低层模板，它们都应该依赖抽象。抽象不应该依赖于具体细节，细节应该依赖于抽象。

- **接口隔离原则-Interface Segregation Principle（ISP）**
  客户端不应该依赖那些它不需要的接口。

- **合成复用原则-Composite Reuse Principle（CRP）**
  优先使用对象组合，而不是通过继承来达到复用的目的。

- **迪米特法则-Law of Demeter（Lod）**
  每一个软件单位对其他单位都只有最少的知识，而且局限于那些与本单位密切的软件单位。

---

### Javascript对比Java

**JavaScript 与 Java 是两个公司开发的不同的两个产品。**

- Java 是 Sun 公司推出的面向对象的编程语言，现在多用于于互联网服务端开发，前身是 Oak
- JavaScript 是 Netscape 公司推出的，为了扩展 Netscape 浏览器功能而开发的一种可以嵌入Web 页面中运行的基于对象和事件驱动的解释性语言，前身是 LiveScript

**区别：**

- 面向对象和基于对象：Java是一种面向对象的语言；JavaScript 是一种基于对象（Object-Based）和事件驱动（Event-Driven）的编程语言，提供了丰富的内部对象供开发者使用
- 编译和解释：Java 的源代码在执行之前，必须经过编译；JavaScript 是一种解释型编程语言，其源代码不需经过编译，由浏览器直接解释执行
- 静态与动态语言：Java 是静态语言(编译时变量的数据类型即可确定的语言)；JavaScript 是动态语言(运行时确定数据类型的语言)
- 强类型变量和类型弱变量：Java 采用强类型变量检查，所有变量在编译之前必须声明类型；JavaScript 中变量声明，采用弱类型，即变量在使用前不需作声明类型，解释器在运行时检查其数据类型

---

### assert（断言）

- 一种常用的调试方式，很多开发语言中都支持这种机制
- 通常在开发和测试时开启
- 可以用来保证程序最基本、关键的正确性
- 为了提高性能，发布版的程序通常关闭断言
- 断言是一个包含布尔表达式的语句，在执行这个语句时假定该表达式为 true；如果表达式计算为 false ，会报告一个 AssertionError
- 断言在默认情况下是禁用的，要在编译时启用断言，需使用source 1.4 标记，如 javac -source 1.4 TestAssert.java
- 要在运行时启用断言，需加参数 -ea 或 -enableassertions
- 要在运行时选择禁用断言，需加参数 -da 或 -disableassertions
- 要在系统类中启用或禁用断言，需加参数 -esa 或 -dsa

**Java 中断言有两种语法形式：**

- assert 表达式1;

- assert 表达式1 : 错误表达式 ;

  表达式1 是一个布尔值

  错误表达式可以得出一个值，用于生成显示调试信息的字符串消息

---

### Stream接口

**中间操作：**

- filter：过滤元素
- map：映射，将元素转换成其他形式或提取信息
- flatMap：扁平化流映射
- limit：截断流，使其元素不超过给定数量
- skip：跳过指定数量的元素
- sorted：排序
- distinct：去重


**终端操作：**

- anyMatch：检查流中是否有一个元素能匹配给定的谓词
- allMatch：检查谓词是否匹配所有元素
- noneMatch：检查是否没有任何元素与给定的谓词匹配
- findAny：返回当前流中的任意元素（用于并行的场景）
- findFirst：查找第一个元素
- collect：把流转换成其他形式，如集合 List、Map、Integer
- forEach：消费流中的每个元素并对其应用 Lambda，返回 void
- reduce：归约，如：求和、最大值、最小值
- count：返回流中元素的个数

---

### 泛型

- "参数化类型"，将类型由具体的类型参数化，把类型也定义成参数形式（称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。
- 是 JDK 5 中引入的一个新特性，提供了编译时类型安全监测机制，该机制允许程序员在编译时监测非法的类型。
- 泛型的本质是把参数的类型参数化，也就是所操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中。


**为什么要用泛型？**

- 使用泛型编写的程序代码，要比使用 Object 变量再进行强制类型转换的代码，具有更好的安全性和可读性。
- 多种数据类型执行相同的代码使用泛型可以复用代码。

比如集合类使用泛型，取出和操作元素时无需进行类型转换，避免出现 java.lang.ClassCastException 异常；

---

## Java容器

### 容器概念

![img](https://www.runoob.com/wp-content/uploads/2014/01/2243690-9cd9c896e0d512ed.gif)

![img](https://www.javanav.com/aimgs/20191002072124515__20191013143527.jpg)

---

### HashMap和HashTable区别

- 线程安全性不同。HashMap线程不安全；Hashtable 中的方法是Synchronize的。
- key、value是否允许null。HashMap的key和value都是可以是null，key只允许一个null；Hashtable的key和value都不可为null。
- 迭代器不同。HashMap的Iterator是fail-fast迭代器；Hashtable还使用了enumerator迭代器。
- hash的计算方式不同。HashMap计算了hash值；Hashtable使用了key的hashCode方法。
- 默认初始大小和扩容方式不同。HashMap默认初始大小16，容量必须是2的整数次幂，扩容时将容量变为原来的2倍；Hashtable默认初始大小11，扩容时将容量变为原来的2倍加1。
- 是否有contains方法。HashMap没有contains方法；Hashtable包含contains方法，类似于containsValue。
- 父类不同。HashMap继承自AbstractMap；Hashtable继承自Dictionary。

---

### HashMap还是TreeMap

- HashMap基于散列桶（数组和链表）实现；TreeMap基于红黑树实现。
- HashMap不支持排序；TreeMap默认是按照Key值升序排序的，可指定排序的比较器，主要用于存入元素时对元素进行自动排序。
- HashMap大多数情况下有更好的性能，尤其是读数据。在没有排序要求的情况下，使用HashMap。

---

### Queue

- 接口（interface）；

  | 方法 | 抛出异常 | 返回数值 |
  | :--: | :------: | :------: |
  | push |   add    |  offer   |
  | pop  |   poll   |  remove  |
  | top  |   peek   | element  |

---

### 集合线程安全

**线程安全的集合类**

- Vector
- Stack
- Hashtable
- java.util.concurrent 包下所有的集合类 ArrayBlockingQueue、ConcurrentHashMap、ConcurrentLinkedQueue、ConcurrentLinkedDeque...

绝大多数集合类不是线程安全的——**效率**



**迭代器的并发策略（快速失败迭代器 Fail-Fast Iterators）**

- 当一个线程遍历不安全的集合，另一个线程加入进来对该集合进行修改时，遍历线程立刻抛出异常 **ConcurrentModificationException**；



**解决办法**

- **同步封装器**

  Collections.synchronizedXXX(collection)；

  - 但是iterator还是线程不安全，并发修改遍历还是要会fail-fast；

- **并发集合（java.util.Concurrent）**

  - 写时复制集合（读远远多于写）：

    - CopyOnWriteArrayList
    - CopyOnWriteArraySet

  - 对比交换集合（CAS-Compare And Swap）：

    - ConcurrentLinkedQueue
    - ConcurrentSkipListMap

    iterator不连贯，所以并不会把所有更新都会同步；

  - 特殊对象锁（java.util.concurrent.lock.Lock）：

    - ConcurrentHashMap
    - BlockingQueue的实现类

---

### 迭代器

- 首先说一下迭代器模式，它是 Java 中常用的设计模式之一。用于顺序访问集合对象的元素，无需知道集合对象的底层实现。
- Iterator 是可以遍历集合的对象，为各种容器提供了公共的操作接口，隔离对容器的遍历操作和底层实现，从而解耦。
- 缺点是增加新的集合类需要对应增加新的迭代器类，迭代器类与集合类成对增加。



**使用**

- java.lang.Iterable 接口被 java.util.Collection 接口继承，java.util.Collection 接口的 iterator() 方法返回一个 Iterator 对象
- next() 方法获得集合中的下一个元素
- hasNext() 检查集合中是否还有元素
- remove() 方法将迭代器新返回的元素删除
- forEachRemaining(Consumer<? super E> action) 方法，遍历所有元素



**拓展（ListIterator——List及其子类）**

- add(E e)  将指定的元素插入列表，插入位置为迭代器当前位置之前
- set(E e)  迭代器返回的最后一个元素替换参数e
- hasPrevious()  迭代器当前位置，反向遍历集合是否含有元素
- previous()  迭代器当前位置，反向遍历集合，下一个元素
- previousIndex()  迭代器当前位置，反向遍历集合，返回下一个元素的下标
- nextIndex()  迭代器当前位置，返回下一个元素的下标
  - 使用范围不同，Iterator可以迭代所有集合；ListIterator 只能用于List及其子类
  - ListIterator 有 add 方法，可以向 List 中添加对象；Iterator 不能
  - ListIterator 有 hasPrevious() 和 previous() 方法，可以实现逆向遍历；Iterator不可以
  - ListIterator 有 nextIndex() 和previousIndex() 方法，可定位当前索引的位置；Iterator不可以
  - ListIterator 有 set()方法，可以实现对 List 的修改；Iterator 仅能遍历，不能修改

---

### 保证集合不能被修改

- java.util.Collections类，unmodifiableXXX方法。
- google guava工具类（待看）；

---

### HashMap不能用基本类型做键值

- Java中是使用泛型来约束 HashMap 中的key和value的类型的，HashMap<K, V>
- 泛型在Java的规定中必须是对象Object类型的，基本数据类型不是Object类型，不能作为键值
- map.put(0, "ConstXiong")中编译器已将 key 值 0 进行了自动装箱，变为了 Integer 类型

---

### 数组p.k.集合

**数组的优点：**

- 数组的效率高于集合类
- 数组能存放基本数据类型和对象；集合中只能放对象

 

**数组的缺点：**

- 不是面向对象的，存在明显的缺陷
- 数组长度固定且无法动态改变；集合类容量动态改变
- 数组无法判断其中实际存了多少元素，只能通过length属性获取数组的申明的长度
- 数组存储的特点是顺序的连续内存；集合的数据结构更丰富

 

**JDK 提供集合的意义：**

- 集合以类的形式存在，符合面向对象，通过简单的方法和属性调用可实现各种复杂操作
- 集合有多种数据结构，不同类型的集合可适用于不同场合
- 弥补了数组的一些缺点，比数组更灵活、实用，可提高开发效率

---

### TreeSet

- TreeSet 基于 TreeMap 实现，TreeMap 基于红黑树实现

- **特点：**
  - 有序
  - 无重复
  - 添加、删除元素、判断元素是否存在，效率比较高，时间复杂度为 O(log(N))

- **使用方式：**
  - TreeSet 默认构造方法，调用 add() 方法时会调用对象类实现的 Comparable 接口的 compareTo() 方法和集合中的对象比较，根据方法返回的结果有序存储
  - TreeSet 默认构造方法，存入对象的类未实现 Comparable 接口，抛出 ClassCastException
  - TreeSet 支持构造方法指定 Comparator 接口，按照 Comparator 实现类的比较逻辑进行有序存储；
  - 需自定义实现Comparator接口并重写compare方法；
- **排序**
  - TreeMap 会对 key 进行比较，有两种比较方式，第一种是构造方法指定 Comparator，使用 Comparator的compare() 方法进行比较；第二种是构造方法未指定 Comparator 接口，需要 key 对象的类实现 Comparable 接口，用 Comparable #compareTo() 方法进行比较

---

### HashSet

- HashSet 是基于 HashMap 实现的，查询速度特别快

- HashMap 是支持 key 为 null 值的，所以 HashSet 支持添加 null 值

- HashSet 存放自定义类时，自定义类需要重写 hashCode() 和 equals() 方法，确保集合对自定义类的对象的唯一性判断(具体判断逻辑，见 HashMap put() 方法，简单概括就是 key 进行hash。

  1. 判断元素 hash 值是否相等

  2. key 是否为同个对象

  3. key 是否 equals

  第 1 个条件为 true，2、3 有一个为 true，HashMap 即认为 key 相同)

- 无序、不可重复



**与HashMap区别**

- **HashMap**
  - 实现 Map 接口
  - 键值对的方式存储
  - 新增元素使用 put(K key, V value) 方法
  - 底层通过对 key 进行 hash，使用数组 + 链表或红黑树对 key、value 存储

- **HashSet**
  - 实现 Set 接口
  - 存储元素对象
  - 新增元素使用 add(E e) 方法
  - 底层是采用 HashMap 实现，大部分方法都是通过调用 HashMap 的方法来实现

---

### Map的顺序性

- Map 的实现类有 HashMap、LinkedHashMap、TreeMap
- HashMap是有无序的

- LinkedHashMap 底层存储结构是哈希表+链表，链表记录了**添加数据的顺序**
- TreeMap 底层存储结构是二叉树，二叉树的中序遍历保证了数据的有序性，**默认升序**

---

### Collections 工具类的 sort() 方法

- 第一种要求传入的待排序容器中存放的对象比较实现 Comparable 接口以实现元素的比较
- 第二种不强制性的要求容器中的元素必须可比较，但要求传入参数 Comparator 接口的子类，需要重写 compare() 方法实现元素的比较规则，其实就是通过接口注入比较元素大小的算法，这就是回调模式的应用

---

### List去重

- List --> HashSet

---

### Vector、ArrayList、LinkedList的存储性能和特性？

- ArrayList 和 Vector 都是使用数组存储数据
- 允许直接按序号索引元素
- 插入元素涉及数组扩容、元素移动等内存操作
- 根据下标找元素快，存在扩容的情况下插入慢
- Vector 对元素的操作，使用了 synchronized 方法，性能比 ArrayList 差
- Vector 属于遗留容器，早期的 JDK 中使用的容器
- LinkedList 使用双向链表存储元素
- LinkedList 按序号查找元素，需要进行前向或后向遍历，所以按下标查找元素，效率较低
- LinkedList 非线程安全
- LinkedList 使用的链式存储方式与数组的连续存储方式相比，对内存的利用率更高
- LinkedList 插入数据时只需要移动指针即可，所以插入速度较快

---

### ConcurrentHashMap实现原理

**JDK 1.7：**

- ConcurrentHashMap 是通过**数组** + **链表**实现，由 Segment 数组和 Segment 元素里对应多个 HashEntry 组成
- value 和链表都是 volatile 修饰，保证可见性
- ConcurrentHashMap 采用了分段锁技术，分段指的就是 Segment 数组，其中 Segment 继承于 ReentrantLock
- 理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发，每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment 

- put 方法的逻辑较复杂：
  - 尝试加锁，加锁失败 scanAndLockForPut 方法自旋，超过 MAX_SCAN_RETRIES 次数，改为阻塞锁获取
  - 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry
  - 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value
  - 不为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容
  - 最后释放所获取当前 Segment 的锁 

- get 方法较简单：
  - 将 key 通过 hash 之后定位到具体的 Segment，再通过一次 hash 定位到具体的元素上
  - 由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了其内存可见性 

**JDK 1.8：**

- 抛弃了原有的 Segment 分段锁，采用了 CAS + synchronized 来保证并发安全性
- HashEntry 改为 Node，作用相同
- val next 都用了 volatile 修饰

- put 方法逻辑：
  - 根据 key 计算出 hash 值
  - 判断是否需要进行初始化
  - 根据 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋
  - 如果当前位置的 hashcode == MOVED == -1，则需要扩容
  - 如果都不满足，则利用 synchronized 锁写入数据
  - 如果数量大于 TREEIFY_THRESHOLD 则转换为红黑树

- get 方法逻辑：
  - 根据计算出来的 hash 值寻址，如果在桶上直接返回值
  - 如果是红黑树，按照树的方式获取值
  - 如果是链表，按链表的方式遍历获取值



**JDK 1.7 到 JDK 1.8 中的 ConcurrentHashMap 最大的改动：**

- 链表上的 Node 超过 8 个改为红黑树，查询复杂度 O(logn)
- ReentrantLock 显示锁改为 synchronized，说明 JDK 1.8 中 synchronized 锁性能赶上或超过 ReentrantLock

---

## Java I/O

### Java IO流

- **按数据流向:输入流和输出流**

  输入和输出都是从程序的角度来说的。

  输入流（InputStream）：数据流向程序

  输出流（OutputStream）：数据从程序流出。

- **按处理单位:字节流和字符流**

  字节流（Reader）：一次读入或读出是8位二进制

  字符流（Writer）：一次读入或读出是16位二进制

  JDK 中后缀是 Stream 是字节流；后缀是 Reader，Writer 是字符流

- **按功能功能:节点流和处理流**

  节点流：可以从某节点读数据或向某节点写数据的流。如 FileInputStream

  处理流：对已存在的流的连接和封装，实现更为丰富的流数据处理，处理流的构造方法必需其他的流对象参数。如 BufferedReader

- **常用的流**

  - 文件（File 4）
  - 管道（Piped 4）
  - 字节/字符（ByteArray IO/CharArray RW）
  - 缓冲（Buffer 4）
  - 转化（InputStreamReader、OutputStreamWriter）
  - 数据（Data I/O）
  - 打印（PrintStream PrintWriter）
  - 对象（Object IO）
  - 序列化（SequenceInputStream）

- **输入输出流区别**
  - 输入输出的方向是针对程序而言，向程序中读入数据，就是输入流；从程序中向外写出数据，就是输出流
  - 从磁盘、网络、键盘读到内存，就是输入流，用 InputStream 或 Reader
  - 写到磁盘、网络、屏幕，都是输出流，用 OuputStream 或 Writer

---

### BIO、NIO、AIO区别

**流程：**

- BIO：线程发起 IO 请求，不管内核是否准备好 IO 操作，从发起请求起，线程一直阻塞，直到操作完成。
- NIO：线程发起 IO 请求，立即返回；内核在做好 IO 操作的准备之后，通过调用注册的回调函数通知线程做 IO 操作，线程开始阻塞，直到操作完成。
- AIO：线程发起 IO 请求，立即返回；内存做好 IO 操作的准备之后，做 IO 操作，直到操作完成或者失败，通过调用注册的回调函数通知线程做 IO 操作完成或者失败。

 **线程：**

- BIO 是一个连接一个线程。
- NIO 是一个请求一个线程。
- AIO 是一个有效请求一个线程。

 **阻塞：**

- BIO：同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。
- NIO：同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。
- AIO：异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的 IO 请求都是由 OS 先完成了再通知服务器应用去启动线程进行处理。

**适用场景：**

- BIO 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4 以前的唯一选择，但程序直观简单易理解。
- NIO 方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4 开始支持。
- AIO 方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

---

### Socket

- Socket 也称作"套接字"，用于描述 IP 地址和端口，是一个通信链的句柄，是应用层与传输层之间的桥梁
- 应用程序可以通过 Socket 向网络发出请求或应答网络请求
- 网络应用程序位于应用层，TCP 和 UDP 属于传输层协议，在应用层和传输层之间，使用 Socket 来进行连接
- Socket 是传输层供给应用层的编程接口
- Socket 编程可以开发客户端和服务器应用程序，可以在本地网络上进行通信，也可通过公网 Internet 在通信

---

### Java内的Socket

- JDK 在 java.net 包中为 TCP 和 UDP 两种通信协议提供了相应的 Socket 编程类
- TCP 协议，服务端对应 ServerSocket，客户端对应 Socket
- UDP 协议对应 DatagramSocket
- 基于 TCP 协议创建的套接字可以叫做流套接字，服务器端相当于一个监听器，用来监听端口，服务器与客服端之间的通讯都是输入输出流来实现的
- 基于 UDP 协议的套接字就是数据报套接字，客户端和服务端都要先构造好相应的数据包

 

**基于 TCP 协议的 Socket 编程的主要步骤**

- **服务端：**
  - 指定本地的端口创建 ServerSocket 实例， 用来监听指定端口的连接请求
  - 通过 accept() 方法返回的 Socket 实例，建立了一个和客户端的新连接
  - 通过 Sockect 实例获取 InputStream 和 OutputStream 读写数据
  - 数据传输结束，调用 socket 实例的 close() 方法关闭连接

- **客户端：**
  - 指定的远程服务器 IP 地址和端口创建 Socket 实例
  - 通过 Socket 实例获取 InputStream 和 OutputStream 来进行数据的读写
  - 数据传输结束，调用 socket 实例的 close() 方法关闭连接

 

**基于 UDP 协议的 Socket 编程的主要步骤**

- **服务端：**
  - 指定本地端口创建 DatagramSocket 实例
  - 通过字节数组，创建 DatagramPacket 实例，调用 DatagramSocket 实例的 receive() 方法，用 DatagramPacket 实例来接收数据
  - 设置 DatagramPacket 实例返回的数据，调用 DatagramSocket 实例的 send() 方法来发送数据
  - 数据传输完成，调用 DatagramSocket 实例的 close() 方法

- **客户端：**
  - 创建 DatagramSocket 实例
  - 通过 IP 地址端口和数据创建 DatagramSocket 实例，调用 DatagramSocket 实例 send() 方法发送数据包
  - 通过字节数组创建 DatagramSocket 实例，调用 DatagramSocket 实例 receive() 方法接受数据包
  - 数据传输完成，调用 DatagramSocket 实例的 close() 方法

---

## Java多线程

### Java 中的多线程

- 通过 JDK 中的 java.lang.Thread 可以实现多线程。
- Java 中多线程运行的程序可能是并发也可能是并行，取决于操作系统对线程的调度和计算机硬件资源（ CPU 的个数和 CPU 的核数）。
- CPU 资源比较充足时，多线程被分配到不同的 CPU 资源上，即并行；CPU 资源比较紧缺时，多线程可能被分配到同个 CPU 的某个核上去执行，即并发。
- 不管多线程是并行还是并发，都是为了提高程序的性能。

---

### Java守护线程

- 守护线程（Daemon Thread）是程序运行的时候在后台提供一种通用服务的线程。所有用户线程（User Thread）停止，进程会停掉所有守护线程，退出程序。
- Java中把线程设置为守护线程的方法：在 start 线程之前调用线程的 setDaemon(true) 方法。
- setDaemon(true) 必须在 start() 之前设置，否则会抛出IllegalThreadStateException异常，该线程仍默认为用户线程，继续执行
- 守护线程创建的线程也是守护线程
- 守护线程不应该访问、写入持久化资源，如文件、数据库，因为它会在任何时间被停止，导致资源未释放、数据写入中断等问题
- **最经典的守护线程应该是GC垃圾回收器**

---

### Java创建线程的方式（4种）

**创建Thread**

```java
new Thread("thread_name") {
    @Override
    public void run() {
        ...
    }
}.start;
```

**继承Thread**

```java
new MyThread().start();
...
public class MyThread extends Thread {
    @Override
    public void run() {
        ...
    }
}
```

**匿名实现Runnable接口**

```java
//normal:
new Thread(new Runnable() {
    @Override
    public void run() {
        ...
    }
}, "thread_name").start();

//lambda:
new Thread(() -> {
    ...
}, "thread_name").start();
```

**显式实现Runnable接口**

```java
new Thread(new RunnableImpl()).start();
...
class RunnableImpl implements Runnable {
    @Override
    public void run() {
        ...
    }
}
```

**实现Callable接口，用FutureTask类创建线程**

```java
FutureTask<T> ft = new FutureTask<T>(new Callable<T>() {
    @Override
    public T call() throws Exception {
        return T;
    }
});
new Thread(ft).start();
```

**使用线程池创建、启动线程**

```java
ExcutorService singleThreadExecutor = Executors.newSingleThreadExcutor();
singleThreadExecutor.submit(() -> {
    ...
});
singleTHreadExecutor.shutdown();
```

---

### Java多线程的运行安全

**线程的安全性问题体现在：**

- 原子性：一个或者多个操作在 CPU 执行的过程中不被中断的特性
- 可见性：一个线程对共享变量的修改，另外一个线程能够立刻看到
- 有序性：程序执行的顺序按照代码的先后顺序执行
   

**导致原因：**

- 缓存导致的可见性问题
- 线程切换带来的原子性问题
- 编译优化带来的有序性问题
   

**解决办法：**

- JDK Atomic开头的原子类、synchronized、LOCK，可以解决原子性问题
- synchronized、volatile、LOCK，可以解决可见性问题
- Happens-Before 规则可以解决有序性问题

---

### Java中的线程状态

1. NEW（初始化状态）
2. RUNNABLE（可运行 / 运行状态）
3. BLOCKED（阻塞状态）
4. WAITING（无限时等待）
5. TIMED_WAITING（有限时等待）
6. TERMINATED（终止状态） 在操作系统层面，Java 线程中的 BLOCKED、WAITING、TIMED_WAITING 是一种状态（休眠状态）。即只要 Java 线程处于这三种状态之一，就永远没有 CPU 的使用权。

![img](https://www.javanav.com/aimgs/5__20190904221205.jpg)

**1. NEW 到 RUNNABLE 状态**

Java 刚创建出来的 Thread 对象就是 NEW 状态，不会被操作系统调度执行。从 NEW 状态转变到 RUNNABLE 状态调用线程对象的 start() 方法就可以了。

 

**2. RUNNABLE 与 BLOCKED 的状态转变**

- synchronized 修饰的方法、代码块同一时刻只允许一个线程执行，其他线程只能等待，等待的线程会从 RUNNABLE 转变到 BLOCKED 状态。
- 当等待的线程获得 synchronized 隐式锁时，就又会从 BLOCKED 转变到 RUNNABLE 状态。
- 在操作系统层面，线程是会转变到休眠状态的，但是在 JVM 层面，Java 线程的状态不会发生变化，即 Java 线程的状态会保持 RUNNABLE 状态。JVM 层面并不关心操作系统调度相关的状态，因为在 JVM 看来，等待 CPU 使用权（操作系统层面处于可执行状态）与等待 I/O（操作系统层面处于休眠状态）没有区别，都是在等待某个资源，都归入了 RUNNABLE 状态。
- Java 在调用阻塞式 API 时，线程会阻塞，指的是操作系统线程的状态，并不是 Java 线程的状态。

 

**3. RUNNABLE 与 WAITING 的状态转变**

- 获得 synchronized 隐式锁的线程，调用无参数的 Object.wait() 方法，状态会从 RUNNABLE 转变到 WAITING；调用 Object.notify()、Object.notifyAll() 方法，线程可能从 WAITING 转变到 RUNNABLE 状态。
- 调用无参数的 Thread.join() 方法。join() 是一种线程同步方法，如有一线程对象 Thread t，当调用 t.join() 的时候，执行代码的线程的状态会从 RUNNABLE 转变到 WAITING，等待 thread t 执行完。当线程 t 执行完，等待它的线程会从 WAITING 状态转变到 RUNNABLE 状态。
- 调用 LockSupport.park() 方法，线程的状态会从 RUNNABLE 转变到 WAITING；调用 LockSupport.unpark(Thread thread) 可唤醒目标线程，目标线程的状态又会从 WAITING 转变为 RUNNABLE 状态。

 

**4. RUNNABLE 与 TIMED_WAITING 的状态转变**

- Thread.sleep(long millis)
- Object.wait(long timeout)
- Thread.join(long millis)
- LockSupport.parkNanos(Object blocker, long deadline)
- LockSupport.parkUntil(long deadline)

TIMED_WAITING 和 WAITING 状态的区别，仅仅是调用的是超时参数的方法。

 

**5. RUNNABLE 到 TERMINATED 状态**

- 线程执行完 run() 方法后，会自动转变到 TERMINATED 状态
- 执行 run() 方法时异常抛出，也会导致线程终止
- ~~Thread类的 stop() 方法已经不建议使用~~

---

### Java线程池

- **线程池**就是创建若干个可执行的线程放入一个池（容器）中，有任务需要处理时，会提交到线程池中的任务队列，处理完之后线程并**不会被销毁**，而是仍然在线程池中等待下一个任务。
- Java 中创建一个线程，需要调用操作系统内核的 API，操作系统要为线程分配一系列的资源，成本很高，所以线程是一个重量级的对象，应该避免频繁创建和销毁。
- 线程池是一种**生产者——消费者**模式。

**JDK中线程池的使用**

```java
ThreadPoolExecutor(
    int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler)
```

- corePoolSize：线程池保有的最小线程数。
- maximumPoolSize：线程池创建的最大线程数。
- keepAliveTime：上面提到项目根据忙闲来增减人员，那在编程世界里，如何定义忙和闲呢？很简单，一个线程如果在一段时间内，都没有执行任务，说明很闲，keepAliveTime 和 unit 就是用来定义这个“一段时间”的参数。也就是说，如果一个线程空闲了keepAliveTime & unit这么久，而且线程池的线程数大于 corePoolSize ，那么这个空闲的线程就要被回收了。
- unit：keepAliveTime 的时间单位
- workQueue：任务队列
- threadFactory：线程工厂对象，可以自定义如何创建线程，如给线程指定name。
- handler：自定义任务的拒绝策略。线程池中所有线程都在忙碌，且任务队列已满，线程池就会拒绝接收再提交的任务。handler 就是拒绝策略，包括 4 种(即RejectedExecutionHandler 接口的 4个实现类)。
  - AbortPolicy：默认的拒绝策略，throws RejectedExecutionException
  - CallerRunsPolicy：提交任务的线程自己去执行该任务
  - DiscardPolicy：直接丢弃任务，不抛出任何异常
  - DiscardOldestPolicy：丢弃最老的任务，加入新的任务

**Executors**

- newFixedThreadPool：创建定长线程池，每当提交一个任务就创建一个线程，直到达到线程池的最大数量，这时线程数量不再变化，当线程发生错误结束时，线程池会补充一个新的线程
- newCachedThreadPool：创建可缓存的线程池，如果线程池的容量超过了任务数，自动回收空闲线程，任务增加时可以自动添加新线程，线程池的容量不限制
- newScheduledThreadPool：创建定长线程池，可执行周期性的任务
- newSingleThreadExecutor：创建单线程的线程池，线程异常结束，会创建一个新的线程，能确保任务按提交顺序执行
- newSingleThreadScheduledExecutor：创建单线程可执行周期性任务的线程池
- newWorkStealingPool：任务可窃取线程池，不保证执行顺序，当有空闲线程时会从其他任务队列窃取任务执行，适合任务耗时差异较大。

---

### Java线程池包含状态

**状态**

- **RUNNING**：线程池一旦被创建，就处于 RUNNING 状态，任务数为 0，能够接收新任务，对已排队的任务进行处理。

- **SHUTDOWN**：不接收新任务，但能处理已排队的任务。调用线程池的 shutdown() 方法，线程池由 RUNNING 转变为 SHUTDOWN 状态。

- **STOP**：不接收新任务，不处理已排队的任务，并且会中断正在处理的任务。调用线程池的 shutdownNow() 方法，线程池由(RUNNING 或 SHUTDOWN ) 转变为 STOP 状态。

-  **TIDYING**：
  - SHUTDOWN 状态下，任务数为 0， 其他所有任务已终止，线程池会变为 TIDYING 状态，会执行 terminated() 方法。线程池中的 terminated() 方法是空实现，可以重写该方法进行相应的处理。
  - 线程池在 SHUTDOWN 状态，任务队列为空且执行中任务为空，线程池就会由 SHUTDOWN 转变为 TIDYING 状态。
  - 线程池在 STOP 状态，线程池中执行中任务为空时，就会由 STOP 转变为 TIDYING 状态。

-  **TERMINATED**：线程池彻底终止。线程池在 TIDYING 状态执行完 terminated() 方法就会由 TIDYING 转变为 TERMINATED 状态。

![img](https://www.javanav.com/aimgs/a__20190910203037.jpg)

```java
//JDK源码
The runState provides the main lifecyle control, taking on values:

  RUNNING:  Accept new tasks and process queued tasks
  SHUTDOWN: Don't accept new tasks, but process queued tasks
  STOP:     Don't accept new tasks, don't process queued tasks,
            and interrupt in-progress tasks
  TIDYING:  All tasks have terminated, workerCount is zero,
            the thread transitioning to state TIDYING
            will run the terminated() hook method
  TERMINATED: terminated() has completed
      
//状态间的变化
RUNNING -> SHUTDOWN
   On invocation of shutdown(), perhaps implicitly in finalize()
(RUNNING or SHUTDOWN) -> STOP
   On invocation of shutdownNow()
SHUTDOWN -> TIDYING
   When both queue and pool are empty
STOP -> TIDYING
   When pool is empty
TIDYING -> TERMINATED
   When the terminated() hook method has completed

Threads waiting in awaitTermination() will return when the
state reaches TERMINATED.
```

---

### synchronized关键字

Java 中关键字 synchronized 表示只有一个线程可以获取作用对象的锁，执行代码，阻塞其他线程。

**作用**

- 确保线程互斥地访问同步代码
- 保证共享变量的修改能够及时可见
- 有效解决重排序问题

**用法**

- 修饰普通方法
- 修饰静态方法
- 指定对象，修饰代码块

**特点**

- 阻塞未获取到锁、竞争同一个对象锁的线程
- 获取锁无法设置超时
- 无法实现公平锁
- 控制等待和唤醒需要结合加锁对象的 wait() 和 notify()、notifyAll()
- 锁的功能是 JVM 层面实现的
- 在加锁代码块执行完或者出现异常，自动释放锁

**原理**

- 同步代码块是通过 monitorenter 和 monitorexit 指令获取线程的执行权（JVM的字节码，反编译）
- 同步方法通过加 ACC_SYNCHRONIZED 标识实现线程的执行权的控制

---

### volatile关键字的作用

- 保证了不同线程对共享变量进行操作时的可见性，即一个线程修改了共享变量的值，共享变量修改后的值对其他线程立即可见

- 通过禁止编译器、CPU 指令重排序和部分 happens-before 规则，解决有序性问题

- Java 中 volatile 关键字是一个类型修饰符。JDK 1.5 之后，对其语义进行了增强。

**volatile可见性的实现**

- 在生成汇编代码指令时会在 volatile 修饰的共享变量进行写操作的时候会多出 Lock 前缀的指令
- Lock 前缀的指令会引起 CPU 缓存写回内存
- 一个 CPU 的缓存回写到内存会导致其他 CPU 缓存了该内存地址的数据无效
- volatile 变量通过**缓存一致性协议**保证每个线程获得最新值
- 缓存一致性协议保证每个 CPU 通过**嗅探在总线上传播的数据**来检查自己缓存的值是不是修改
- 当 CPU 发现自己缓存行对应的内存地址被修改，会将当前 CPU 的缓存行设置成**无效状态**，重新从内存中把数据读到 CPU 缓存

**volatile有序性的实现**

**Happens-before 规则：**一个 volatile 变量的写操作发生在这个 volatile 变量随后的读操作之前。

- 3 个 happens-before 规则实现：
  1. 对一个 volatile 变量的写 happens-before 任意后续对这个 volatile 变量的读
  2. 在一个线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作
  3. happens-before 传递性，A happens-before B，B happens-before C，则 A happens-before C
- 内存屏障(Memory Barrier 又称内存栅栏，是一个 CPU 指令)禁止重排序：
  1. 在程序运行时，为了提高执行性能，在不改变正确语义的前提下，编译器和 CPU 会对指令序列进行重排序。
  2. Java 编译器会在生成指令时，为了保证在不同的编译器和 CPU 上有相同的结果，通过插入特定类型的内存屏障来禁止特定类型的指令重排序。
  3. 编译器根据具体的底层体系架构，将这些内存屏障替换成具体的 CPU 指令。
  4. 内存屏障会告诉编译器和 CPU：不管什么指令都不能和这条 Memory Barrier 指令重排序。

**内存屏障**

- 为了实现 volatile 内存语义时，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的 CPU 重排序。
- 对于编译器，内存屏障将限制它所能做的重排序优化；对于 CPU，内存屏障将会导致缓存的刷新操作
- volatile 变量的写操作，在变量的前面和后面分别插入内存屏障；volatile 变量的读操作是在后面插入两个内存屏障
  - 在每个 volatile 写操作的前面插入一个 StoreStore 屏障
  - 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障
  - 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障
  - 在每个 volatile 读操作的后面插入一个 LoadStore 屏障
- 屏障说明
  - StoreStore：禁止之前的普通写和之后的 volatile 写重排序；
  - StoreLoad：禁止之前的 volatile 写与之后的 volatile 读/写重排序
  - LoadLoad：禁止之后所有的普通读操作和之前的 volatile 读重排序
    LoadStore：禁止之后所有的普通写操作和之前的 volatile 读重排序
  - LoadStore：禁止之后所有的普通写操作和之前的 volatile 读重排序

---

### Java锁

![img](https://img-blog.csdnimg.cn/20181122101753671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2F4aWFvYm9nZQ==,size_16,color_FFFFFF,t_70)

**java常见的锁**

- synchronized 关键字锁定代码库
- 可重入锁 java.util.concurrent.lock.ReentrantLock
- 可重复读写锁 java.util.concurrent.lock.ReentrantReadWriteLock

- **锁分类**

  - **可重入锁**

    指在同一个线程在外层方法获取锁的时候，进入内层方法会自动获取锁。JDK 中基本都是可重入锁，避免死锁的发生。

  - **公平锁**

    多个线程按照申请锁的顺序来获取锁。如 java.util.concurrent.lock.ReentrantLock.FairSync

  - **非公平锁**

    多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程先获得锁。如 synchronized、java.util.concurrent.lock.ReentrantLock.NonfairSync

  - **独享锁**

    锁一次只能被一个线程所持有。synchronized、java.util.concurrent.locks.ReentrantLock 都是独享锁

  - **共享锁**

    锁可被多个线程所持有。ReadWriteLock 返回的 ReadLock 就是共享锁

  - **悲观锁**

    一律会对代码块进行加锁，如 synchronized、java.util.concurrent.locks.ReentrantLock，适合写操作较多的场景

  - **乐观锁**

    默认不会进行并发修改，通常采用 CAS 算法不断尝试更新，适合读操作较多的场景

  - **粗粒度锁**

    把执行的代码块都锁定（粗粒度锁和细粒度锁是相对的，没什么标准）

  - **细粒度锁**

    锁住尽可能小的代码块，java.util.concurrent.ConcurrentHashMap 中的分段锁就是一种细粒度锁

  - **锁升级机制**（偏向锁/轻量级锁/重量级锁）

    - JDK 1.5 之后新增锁的升级机制，提升性能。
    - 通过 synchronized 加锁后，一段同步代码一直被同一个线程所访问，那么该线程获取的就是偏向锁
    - 偏向锁被一个其他线程访问时，Java 对象的偏向锁就会升级为轻量级锁
    - 再有其他线程会以自旋的形式尝试获取锁，不会阻塞，自旋一定次数仍然未获取到锁，就会膨胀为重量级锁

  - **自旋锁**

    自旋锁是指尝试获取锁的线程**不会立即阻塞**，而是采用**循环**的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环占有、浪费 CPU 资源

---

### synchronized锁的升级原理是什么?

**锁的级别从低到高：**

无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁

**锁分级别原因：**

没有优化以前，synchronized 是重量级锁（悲观锁），使用 wait 和 notify、notifyAll 来切换线程状态非常消耗系统资源；线程的挂起和唤醒间隔很短暂，这样很浪费资源，影响性能。所以 JVM对 synchronized 关键字进行了优化，把锁分为无锁、偏向锁、轻量级锁、重量级锁状态。

锁升级的目的是为了减低锁带来的性能消耗，在 Java 6 之后优化 synchronized 为此方式。

- **无锁：**没有对资源进行锁定，所有的线程都能访问并修改同一个资源，但同时只有一个线程能修改成功，其他修改失败的线程会不断重试直到修改成功。

- **偏向锁：**对象的代码一直被同一线程执行，不存在多个线程竞争，该线程在后续的执行中自动获取锁，降低获取锁带来的性能开销。偏向锁，指的就是偏向第一个加锁线程，该线程是不会主动释放偏向锁的，只有当其他线程尝试竞争偏向锁才会被释放。

  偏向锁的撤销，需要在某个时间点上没有字节码正在执行时，先暂停拥有偏向锁的线程，然后判断锁对象是否处于被锁定状态。如果线程不处于活动状态，则将对象头设置成无锁状态，并撤销偏向锁；

  如果线程处于活动状态，升级为轻量级锁的状态。

- **轻量级锁：**轻量级锁是指当锁是偏向锁的时候，被第二个线程 B 所访问，此时偏向锁就会升级为轻量级锁，线程 B 会通过自旋的形式尝试获取锁，线程不会阻塞，从而提高性能。

  当前只有一个等待线程，则该线程将通过自旋进行等待。但是当自旋超过一定的次数时，轻量级锁便会升级为重量级锁；当一个线程已持有锁，另一个线程在自旋，而此时又有第三个线程来访时，轻量级锁也会升级为重量级锁。

- **重量级锁：**指当有一个线程获取锁之后，其余所有等待获取该锁的线程都会处于阻塞状态。

  重量级锁通过对象内部的监视器（monitor）实现，而其中 monitor 的本质是依赖于底层操作系统的 Mutex Lock 实现，操作系统实现线程之间的切换需要从用户态切换到内核态，切换成本非常高。

![img](https://www.javanav.com/aimgs/image__20191014193357.png)

**synchronized 锁升级的过程：**

- 在锁对象的对象头里面有一个 threadid 字段，未访问时 threadid 为空
- 第一次访问 jvm 让其持有偏向锁，并将 threadid 设置为其线程 id
- 再次访问时会先判断 threadid 是否与其线程 id 一致。如果一致则可以直接使用此对象；如果不一致，则升级偏向锁为轻量级锁，通过自旋循环一定次数来获取锁
- 执行一定次数之后，如果还没有正常获取到要使用的对象，此时就会把锁从轻量级升级为重量级锁