## 面向切片：AOP



### 场景模拟

创建新的子模块`spring6-aop`，并声明一个接口用以使用（计算器接口`Calculator`）：

```java
public interface Calculator {
    int add(int i, int j);
    int sub(int i, int j);
    int mul(int i, int j);
    int div(int i, int j);
}
```

![image-20230424224459030](spring-imgs/image-20230424224459030.png)

实现：

```java
public class CalculatorImpl implements Calculator {
    
    @Override
    public int add(int i, int j) {
        int result = i + j;
        System.out.println("方法内部 result = " + result);
        return result;
    }
    
    @Override
    public int sub(int i, int j) {
        int result = i - j;
        System.out.println("方法内部 result = " + result);
        return result;
    }
    
    @Override
    public int mul(int i, int j) {
        int result = i * j;
        System.out.println("方法内部 result = " + result);
        return result;
    }
    
    @Override
    public int div(int i, int j) {
        int result = i / j;
        System.out.println("方法内部 result = " + result);
        return result;
    }
}
```

如果现在有需求要加上一个日志显示👇：

![image-20230424224632788](spring-imgs/image-20230424224632788.png)

```java
public class CalculatorLogImpl implements Calculator {
    
    @Override
    public int add(int i, int j) {
        System.out.println("[日志] add 方法开始了，参数是：" + i + "," + j);
        int result = i + j;
        System.out.println("方法内部 result = " + result);
        System.out.println("[日志] add 方法结束了，结果是：" + result);
        return result;
    }
    
    @Override
    public int sub(int i, int j) {
        System.out.println("[日志] sub 方法开始了，参数是：" + i + "," + j);
        int result = i - j;
        System.out.println("方法内部 result = " + result);
        System.out.println("[日志] sub 方法结束了，结果是：" + result);
        return result;
    }
    
    @Override
    public int mul(int i, int j) {
        System.out.println("[日志] mul 方法开始了，参数是：" + i + "," + j);
        int result = i * j;
        System.out.println("方法内部 result = " + result);
        System.out.println("[日志] mul 方法结束了，结果是：" + result);
        return result;
    }
    
    @Override
    public int div(int i, int j) {
        System.out.println("[日志] div 方法开始了，参数是：" + i + "," + j);
        int result = i / j;
        System.out.println("方法内部 result = " + result);
        System.out.println("[日志] div 方法结束了，结果是：" + result);
        return result;
    }
}
```

那么这些代码就有如下问题了：

1. 对核心业务功能有干扰，导致程序员在开发核心业务功能时分散了精力；
2. 附加功能分散在各个业务功能方法中，不利于统一维护；

所以，解决这两个问题，核心就是：**解耦**。我们需要把附加功能从业务功能代码中抽取出来。

但是，也存在解决问题的困难：

- 要抽取的代码在方法内部，靠以前把子类中的重复代码抽取到父类的方式没法解决。所以需要引入新的技术。



### 代理模式

- **代理**：将非核心逻辑剥离出来以后，封装这些非核心逻辑的类、对象、方法。
- **目标**：被代理“套用”了非核心逻辑代码的类、对象、方法。

二十三种设计模式中的一种，属于结构型模式。它的作用就是通过提供一个代理类，让我们在调用目标方法的时候，不再是直接对目标方法进行调用，而是通过代理类**间接**调用。让不属于目标方法核心逻辑的代码从目标方法中剥离出来——**解耦**。

调用目标方法时先调用代理对象的方法，减少对目标方法的调用和打扰，同时让附加功能能够集中在一起也有利于统一维护。**使用代理前**：

![image-20230424225741555](spring-imgs/image-20230424225741555.png)

**使用代理后**：

![image-20230424225759263](spring-imgs/image-20230424225759263.png)



#### 静态代理

```java
package calculator;

public class CalculatorStaticProxy implements Calculator {

    // 将被代理的目标对象声明为成员变量
    private Calculator target;

    public CalculatorStaticProxy(Calculator target) {
        this.target = target;
    }

    @Override
    public int add(int i, int j) {

        // 附加功能由代理类中的代理方法来实现
        System.out.println("[日志] add 方法开始了，参数是：" + i + "," + j);

        // 通过目标对象来实现核心业务逻辑
        int addResult = target.add(i, j);

        System.out.println("[日志] add 方法结束了，结果是：" + addResult);

        return addResult;
    }

    @Override
    public int sub(int i, int j) {

        // 附加功能由代理类中的代理方法来实现
        System.out.println("[日志] sub 方法开始了，参数是：" + i + "," + j);

        // 通过目标对象来实现核心业务逻辑
        int subResult = target.sub(i, j);

        System.out.println("[日志] sub 方法结束了，结果是：" + subResult);

        return subResult;
    }

    @Override
    public int mul(int i, int j) {

        // 附加功能由代理类中的代理方法来实现
        System.out.println("[日志] mul 方法开始了，参数是：" + i + "," + j);

        // 通过目标对象来实现核心业务逻辑
        int mulResult = target.mul(i, j);

        System.out.println("[日志] mul 方法结束了，结果是：" + mulResult);

        return mulResult;
    }

    @Override
    public int div(int i, int j) {

        // 附加功能由代理类中的代理方法来实现
        System.out.println("[日志] div 方法开始了，参数是：" + i + "," + j);

        // 通过目标对象来实现核心业务逻辑
        int divResult = target.div(i, j);

        System.out.println("[日志] div 方法结束了，结果是：" + divResult);

        return divResult;
    }
}
```

##### 缺陷

- 静态代理确实实现了解耦，但是由于代码都写死了，完全不具备任何的灵活性。就拿日志功能来说，将来其他地方也需要附加日志，那还得再声明更多个静态代理类，那就产生了大量重复的代码，日志功能还是分散的，没有统一管理。


- 提出进一步的需求：**将日志功能集中到一个代理类中**，将来有任何日志需求，都通过这一个代理类来实现。这就需要使用**动态代理**技术了。