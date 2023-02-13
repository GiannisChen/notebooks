## 设计模式

![Go 设计模式](../Golang/Images/go.png)

**设计模式**是软件设计中常见问题的典型解决方案。 它们就像能根据需求进行调整的预制蓝图， 可用于解决代码中反复出现的设计问题。

设计模式与方法或库的使用方式不同， 你很难直接在自己的程序中套用某个设计模式。 模式并不是一段特定的代码， 而是解决特定问题的一般性概念。 你可以根据模式来实现符合自己程序实际所需的解决方案。

人们常常会混淆模式和算法， 因为两者在概念上都是已知特定问题的典型解决方案。 但算法总是明确定义达成特定目标所需的一系列步骤， 而模式则是对解决方案的更高层次描述。 同一模式在两个不同程序中的实现代码可能会不一样。

算法更像是菜谱： 提供达成目标的明确步骤。 而模式更像是蓝图： 你可以看到最终的结果和模式的功能， 但需要自己确定实现步骤。



### 创建型模式

创建型模式提供了创建对象的机制， 能够提升已有代码的灵活性和可复用性。



##### 工厂模式（Factory）

> 在父类中提供一个创建对象的接口以允许子类决定实例化对象的类型。

抽象接口 + 具体实现，通过对不同场景下实例的抽象，形成一个统一的接口，实例实现接口。对上层而言不关心底层实现。

- **优点**
  1. 你可以避免创建者和具体产品之间的紧密耦合。
  2.  单一职责原则。 你可以将产品创建代码放在程序的单一位置， 从而使得代码更容易维护。
  3.  开闭原则。 无需更改现有客户端代码， 你就可以在程序中引入新的产品类型。
- **缺点**
  1. 应用工厂方法模式需要引入许多新的子类， 代码可能会因此变得更复杂。 最好的情况是将该模式引入创建者类的现有层次结构中。

工厂方法定义了一个方法， 且必须使用该方法代替通过直接调用构造函数来创建对象 （ `new`操作符） 的方式。 子类可重写该方法来更改将被创建的对象所属类。

###### 代码

```go
type iGun interface {
    setName(name string)
    getName() string
}

type AK47 struct {
    name string
}

func (a *AK47) setName(name string) {
    ...
}

func (a *AK47) getName() string {
    ...
}
```



##### 抽象工厂模式（Abstract Factory）

> 让你能创建一系列相关的对象， 而无需指定其具体类。

抽象工厂定义了用于创建不同产品的接口， 但将实际的创建工作留给了具体工厂类。 每个工厂类型都对应一个特定的产品变体。

在创建产品时， 客户端代码调用的是工厂对象的构建方法， 而不是直接调用构造函数 （ `new`操作符）。 由于一个工厂对应一种产品变体， 因此它创建的所有产品都可相互兼容。

客户端代码仅通过其抽象接口与工厂和产品进行交互。 该接口允许同一客户端代码与不同产品进行交互。 你只需创建一个具体工厂类并将其传递给客户端代码即可。

###### 代码

让我们假设一下， 如果你想要购买一组运动装备， 比如一双鞋与一件衬衫这样由两种不同产品组合而成的套装。 相信你会想去购买同一品牌的商品， 这样商品之间能够互相搭配起来。

如果我们把这样的行为转换成代码的话， 帮助我们创建此类产品组的工具就是抽象工厂， 便于产品之间能够相互匹配。

```go
type SportsBrand interface {
    f()
}

func GetSportsBrand(choice int) SportsBrand {
    switch choice {
    case 0:
        return &A{}
    case 1:
        return &B{}
    ...
    }
}

type A struct {
    ...
} 

func (a *A) f() {
    ...
}
```



##### 生成器模式（Builder）

> 使你能够分步骤创建复杂对象。

当所需产品较为复杂且需要多个步骤才能完成时， 也可以使用生成器模式。 在这种情况下， 使用多个构造方法比仅仅使用一个复杂可怕的构造函数更简单。 分为多个步骤进行构建的潜在问题是， 构建不完整的和不稳定的产品可能会被暴露给客户端。 生成器模式能够在产品完成构建之前使其处于私密状态。

###### 代码

```go
type IBuilder interface {
    setWindowType()
    setDoorType()
    setNumFloor()
    getHouse() House
}

func getBuilder(builderType string) IBuilder {
    switch builderType {
    case "xxx":
        return newABuilder()
    case "yyy":
        return newBBuilder()
    ...
    }
}
```



##### 原型模式（Prototype）

> 使你能够复制对象， 甚至是复杂对象， 而又无需使代码依赖它们所属的类。

所有的原型类都必须有一个通用的接口， 使得即使在对象所属的具体类未知的情况下也能复制对象。 原型对象可以生成自身的完整副本， 因为相同类的对象可以相互访问对方的私有成员变量。

###### 代码

```go
type Inode interface {
    print(string)
    clone() Inode
}

type File struct {
    name string
}

func (f *File) print(indentation string) {
    fmt.Println(indentation + f.name)
}

func (f *File) clone() Inode {
    return &File{name: f.name + "_clone"}
}

type Folder struct {
    children []Inode
    name     string
}

func (f *Folder) print(indentation string) {
    fmt.Println(indentation + f.name)
    for _, i := range f.children {
        i.print(indentation + indentation)
    }
}

func (f *Folder) clone() Inode {
    cloneFolder := &Folder{name: f.name + "_clone"}
    var tempChildren []Inode
    for _, i := range f.children {
        _copy := i.clone()
        tempChildren = append(tempChildren, _copy)
    }
    cloneFolder.children = tempChildren
    return cloneFolder
}
```



#####  单例模式（Singleton）

> 让你能够保证一个类只有一个实例， 并提供一个访问该实例的全局节点。

单例拥有与全局变量相同的优缺点。 尽管它们非常有用， 但却会破坏代码的模块化特性。

###### 代码

```go
var lock = &sync.Mutex{}

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        lock.Lock()
        defer lock.Unlock()
        if singleInstance == nil {
            fmt.Println("Creating single instance now.")
            singleInstance = &single{}
        } else {
            fmt.Println("Single instance already created.")
        }
    } else {
        fmt.Println("Single instance already created.")
    }

    return singleInstance
}
```

或者使用内置的 `sync.Once`

```go
var once sync.Once

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        once.Do(
            func() {
                fmt.Println("Creating single instance now.")
                singleInstance = &single{}
            })
    } else {
        fmt.Println("Single instance already created.")
    }

    return singleInstance
}
```

---





### 结构型模式

结构型模式介绍如何将对象和类组装成较大的结构， 并同时保持结构的灵活和高效。



##### 适配器模式（Adapter）

> 让接口不兼容的对象能够相互工作。
>

适配器可担任两个对象间的封装器， 它会接收对于一个对象的调用， 并将其转换为另一个对象可识别的格式和接口。

###### 代码

这里有一段客户端代码， 用于接收一个对象 （Lightning 接口） 的部分功能， 不过我们还有另一个名为 *adaptee* 的对象 （Windows 笔记本）， 可通过不同的接口 （USB 接口） 实现相同的功能。

```go
type Client struct {
}

func (c *Client) InsertLightningConnectorIntoComputer(com Computer) {
    com.InsertIntoLightningPort()
}

type Computer interface {
    InsertIntoLightningPort()
}

type Mac struct {
}

func (m *Mac) InsertIntoLightningPort() {
    ...
}

type Windows struct{}

func (w *Windows) insertIntoUSBPort() {
	...
}

type WindowsAdapter struct {
    windowMachine *Windows
}

func (w *WindowsAdapter) InsertIntoLightningPort() {
    ...
    w.windowMachine.insertIntoUSBPort()
}
```



##### 桥接模式（Bridge）

> 可将一个大类或一系列紧密相关的类拆分为抽象和实现两个独立的层次结构， 从而能在开发时分别使用。
>

层次结构中的第一层 （通常称为抽象部分） 将包含对第二层 （实现部分） 对象的引用。 抽象部分将能将一些 （有时是绝大部分） 对自己的调用委派给实现部分的对象。 所有的实现部分都有一个通用接口， 因此它们能在抽象部分内部相互替换。

###### 代码

假设你有两台电脑： 一台 Mac 和一台 Windows。 还有两台打印机： 爱普生和惠普。 这两台电脑和打印机可能会任意组合使用。 客户端不应去担心如何将打印机连接至计算机的细节问题。

- 抽象层： 代表计算机
- 实施层： 代表打印机

```go
type Computer interface {
    Print()
}

type Printer interface {
    PrintFile()
}
```

```go
type Mac struct {
    printer Printer
}

func (m *Mac) Print() {
    m.printer.PrintFile()
}

type Windows struct {
    printer Printer
}

func (w *Windows) Print() {
    w.printer.PrintFile()
}
```



##### 组合模式（Composite）

> 你可以使用它将对象组合成树状结构， 并且能像使用独立对象一样使用它们。
>

对于绝大多数需要生成树状结构的问题来说， 组合都是非常受欢迎的解决方案。 组合最主要的功能是在整个树状结构上递归调用方法并对结果进行汇总。

###### 代码

让我们试着用一个操作系统文件系统的例子来理解组合模式。 文件系统中有两种类型的对象： 文件和文件夹。 在某些情形中， 文件和文件夹应被视为相同的对象。 这就是组合模式发挥作用的时候了。

```go
type Component interface {
    search(string)
}

type File struct {
    name string
}

func (f *File) search(keyword string) {
    fmt.Printf("Searching for keyword %s in file %s\n", keyword, f.name)
}

type Folder struct {
    components []Component
    name       string
}

func (f *Folder) search(keyword string) {
    fmt.Printf("Serching recursively for keyword %s in folder %s\n", keyword, f.name)
    for _, composite := range f.components {
        composite.search(keyword)
    }
}
```



##### 装饰模式（Decorator）

> 允许你通过将对象放入包含行为的特殊封装对象中来为原对象绑定新的行为。（**Wrapper**）
>

由于目标对象和装饰器遵循同一接口， 因此你可用装饰来对对象进行无限次的封装。 结果对象将获得所有封装器叠加而来的行为。

###### 代码

```go
type IPizza interface {
    getPrice() int
}

type VeggeMania struct {}

func (p *VeggeMania) getPrice() int {
    return 15
}

type TomatoTopping struct {
    pizza IPizza
}

func (c *TomatoTopping) getPrice() int {
    pizzaPrice := c.pizza.getPrice()
    return pizzaPrice + 7
}
```



##### 外观模式（Facade）

> 能为程序库、 框架或其他复杂类提供一个简单的接口。
>

尽管外观模式降低了程序的整体复杂度， 但它同时也有助于将不需要的依赖移动到同一个位置。

###### 代码

https://refactoringguru.cn/design-patterns/facade/go/example



##### 享元模式（Flyweight）

> 它摒弃了在每个对象中保存所有数据的方式， 通过共享多个对象所共有的相同状态， 让你能在有限的内存容量中载入更多对象。
>

模式通过共享多个对象的部分状态来实现上述功能。 换句话来说， 享元会将不同对象的相同数据进行缓存以节省内存。

###### 代码

https://refactoringguru.cn/design-patterns/flyweight/go/example



##### 代理模式（Proxy）

> 让你能够提供对象的替代品或其占位符。 代理控制着对于原对象的访问， 并允许在将请求提交给对象前后进行一些处理。
>

代理对象拥有和服务对象相同的接口， 这使得当其被传递给客户端时可与真实对象互换。

###### 代码

https://refactoringguru.cn/design-patterns/proxy/go/example

---



### 行为模式

行为模式负责对象间的高效沟通和职责委派。



##### 责任链模式（Chain of Responsiblity）

> 允许你将请求沿着处理者链进行发送。 收到请求后， 每个处理者均可对请求进行处理， 或将其传递给链上的下个处理者。
>

该模式允许多个对象来对请求进行处理， 而无需让发送者类与具体接收者类相耦合。 链可在运行时由遵循标准处理者接口的任意处理者动态生成。

###### 代码

让我们来看看一个医院应用的责任链模式例子。 医院中会有多个部门， 如：

- 前台
- 医生
- 药房
- 收银

病人来访时， 他们首先都会去前台， 然后是看医生、 取药， 最后结账。 也就是说， 病人需要通过一条部门链， 每个部门都在完成其职能后将病人进一步沿着链条输送。

```go
type Department interface {
    execute(*Patient)
    setNext(Department)
}

type Department interface {
    execute(*Patient)
    setNext(Department)
}

type Reception struct {
    next Department
}

func (r *Reception) execute(p *Patient) {
    if p.registrationDone {
        fmt.Println("Patient registration already done")
        r.next.execute(p)
        return
    }
    fmt.Println("Reception registering patient")
    p.registrationDone = true
    r.next.execute(p)
}

func (r *Reception) setNext(next Department) {
    r.next = next
}
```



##### 命令模式（Command）

> 它可将请求转换为一个包含与请求相关的所有信息的独立对象。 该转换让你能根据不同的请求将方法参数化、 延迟请求执行或将其放入队列中， 且能实现可撤销操作。
>

此类转换让你能够延迟进行或远程执行请求， 还可将其放入队列中。

###### 代码

```go
type Device interface {
    on()
    off()
}

type Tv struct {
    isRunning bool
}

func (t *Tv) on() {
    t.isRunning = true
    fmt.Println("Turning tv on")
}

func (t *Tv) off() {
    t.isRunning = false
    fmt.Println("Turning tv off")
}
```



##### 迭代器模式（Iterator）

> 让你能在不暴露集合底层表现形式 （列表、 栈和树等） 的情况下遍历集合中所有的元素。
>

在迭代器的帮助下， 客户端可以用一个迭代器接口以相似的方式遍历不同集合中的元素。

###### 代码

```go
type Iterator interface {
    hasNext() bool
    getNext() *User
}

type UserIterator struct {
    index int
    users []*User
}

func (u *UserIterator) hasNext() bool {
    if u.index < len(u.users) {
        return true
    }
    return false

}
func (u *UserIterator) getNext() *User {
    if u.hasNext() {
        user := u.users[u.index]
        u.index++
        return user
    }
    return nil
}

type UserCollection struct {
    users []*User
}
```



##### 中介者模式（Mediator）

> 能让你减少对象之间混乱无序的依赖关系。 该模式会限制对象之间的直接交互， 迫使它们通过一个中介者对象进行合作。
>

中介者能使得程序更易于修改和扩展， 而且能更方便地对独立的组件进行复用， 因为它们不再依赖于很多其他的类。

###### 代码

```go
type Train interface {
    arrive()
    depart()
    permitArrival()
}

type Mediator interface {
    canArrive(Train) bool
    notifyAboutDeparture()
}

type PassengerTrain struct {
    mediator Mediator
}

...
```



##### 备忘录模式（Memento）

> 允许在不暴露对象实现细节的情况下保存和恢复对象之前的状态。
>

备忘录不会影响它所处理的对象的内部结构， 也不会影响快照中保存的数据。

###### 代码

https://refactoringguru.cn/design-patterns/memento/go/example



##### 观察者模式（Observer）

> 允许你定义一种订阅机制， 可在对象事件发生时通知多个 “观察” 该对象的其他对象。
>

观察者模式提供了一种作用于任何实现了订阅者接口的对象的机制， 可对其事件进行订阅和取消订阅。

###### 代码

```go
type Subject interface {
    register(observer Observer)
    deregister(observer Observer)
    notifyAll()
}

type Item struct {
    observerList []Observer
    name         string
    inStock      bool
}
```



##### 状态模式（State）

> 让你能在一个对象的内部状态变化时改变其行为， 使其看上去就像改变了自身所属的类一样。（有限状态机）
>

该模式将与状态相关的行为抽取到独立的状态类中， 让原对象将工作委派给这些类的实例， 而不是自行进行处理。

###### 代码

https://refactoringguru.cn/design-patterns/state/go/example



##### 策略模式（Strategy）

> 它能让你定义一系列算法， 并将每种算法分别放入独立的类中， 以使算法的对象能够相互替换。
>

原始对象被称为上下文， 它包含指向策略对象的引用并将执行行为的任务分派给策略对象。 为了改变上下文完成其工作的方式， 其他对象可以使用另一个对象来替换当前链接的策略对象。

###### 代码

思考一下构建内存缓存的情形。 由于处在内存中， 故其大小会存在限制。 在达到其上限后， 一些条目就必须被移除以留出空间。 此类操作可通过多种算法进行实现。 一些流行的算法有：

- **最少最近使用 （LRU）**： 移除最近使用最少的一条条目。
- **先进先出 （FIFO）**： 移除最早创建的条目。
- **最少使用 （LFU）**： 移除使用频率最低一条条目。

```go
type EvictionAlgo interface {
    evict(c *Cache)
}

type Fifo struct {}
type Lru struct {}
type Lfu struct {}
```



##### 模板方法模式（Template Method）

> 它在超类中定义了一个算法的框架， 允许子类在不修改结构的情况下重写算法的特定步骤。
>

###### 代码

https://refactoringguru.cn/design-patterns/template-method/go/example



##### 访问者模式（Visitor）

> 它能将算法与其所作用的对象隔离开来。

https://refactoringguru.cn/design-patterns/visitor/go/example