## 单元测试

在之前的测试方法中，几乎都能看到以下的两行代码：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("xxx.xml");
Xxxx xxx = context.getBean(Xxxx.class);
```

这两行代码的作用是创建Spring容器，最终获取到对象，但是每次测试都需要重复编写。

针对上述问题，我们需要的是程序能自动帮我们创建容器。我们都知道`JUnit`无法知晓我们是否使用了 Spring 框架，更不用说帮我们创建 Spring 容器了。Spring提供了一个运行器，可以读取配置文件（或注解）来创建容器。我们只需要告诉它配置文件位置就可以了。这样一来，我们通过Spring整合`JUnit`可以使程序创建spring容器了。



### 整合JUnit5



##### 添加配置文件

`bean.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="bean"/>
</beans>
```

##### 添加类

```java
@Component
public class User {

    public User() {
        System.out.println("USer construct...");
    }
}
```

##### 测试类

```java
@SpringJUnitConfig(locations = "classpath:bean.xml")
public class UserTest {
    @Autowired
    private User user;

    @Test
    public void testUser() {
        System.out.println(user);
    }
}
```

另外还有一种稍显繁琐的方式：

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:bean.xml")
public class UserTest {
    @Autowired
    private User user;

    @Test
    public void testUser() {
        System.out.println(user);
    }
}
```



### 整合JUnit4

引入`JUnit4`：

```xml
<!-- junit4 测试 -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
</dependency>
```

测试类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class TestClass {
    @Autowired
    private User user;

    @Test
    public void test() {
        System.out.println(user);
    }
}
```

