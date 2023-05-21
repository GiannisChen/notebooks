

## Java 基础



### SPI机制



#### 什么是SPI机制？

SPI（Service Provider Interface），是JDK内置的一种 服务提供发现机制，可以用来启用框架扩展和替换组件，主要是被框架的开发人员使用，比如java.sql.Driver接口，其他不同厂商可以针对同一接口做出不同的实现，MySQL和PostgreSQL都有不同的实现提供给用户，而Java的SPI机制可以为某个接口寻找服务实现。Java中SPI机制主要思想是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要，其核心思想就是 **解耦**。

SPI整体机制图如下：

![img](../java-images/java-advanced-spi-8.jpg)

当服务的提供者提供了一种接口的实现之后，需要在classpath下的`META-INF/services/`目录里创建一个以服务接口命名的文件，这个文件里的内容就是这个接口的具体的实现类。当其他的程序需要这个服务的时候，就可以通过查找这个jar包（一般都是以jar包做依赖）的`META-INF/services/`中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。JDK中查找服务的实现的工具类是：`java.util.ServiceLoader`。



#### SPI机制的应用？

- SPI机制 - JDBC DriverManager

  在JDBC4.0之前，我们开发有连接数据库的时候，通常会用Class.forName("com.mysql.jdbc.Driver")这句先加载数据库相关的驱动，然后再进行获取连接等的操作。**而JDBC4.0之后不需要用Class.forName("com.mysql.jdbc.Driver")来加载驱动，直接获取连接就可以了，现在这种方式就是使用了Java的SPI扩展机制来实现**。

- JDBC接口定义

  首先在java中定义了接口`java.sql.Driver`，并没有具体的实现，具体的实现都是由不同厂商来提供的。

- mysql实现

  在mysql的jar包`mysql-connector-java-6.0.6.jar`中，可以找到`META-INF/services`目录，该目录下会有一个名字为`java.sql.Driver`的文件，文件内容是`com.mysql.cj.jdbc.Driver`，这里面的内容就是针对Java中定义的接口的实现。

- postgresql实现

  同样在postgresql的jar包`postgresql-42.0.0.jar`中，也可以找到同样的配置文件，文件内容是`org.postgresql.Driver`，这是postgresql对Java的`java.sql.Driver`的实现。

- 使用方法

  上面说了，现在使用SPI扩展来加载具体的驱动，我们在Java中写连接数据库的代码的时候，不需要再使用`Class.forName("com.mysql.jdbc.Driver")`来加载驱动了，而是直接使用如下代码：

	```java
	String url = "jdbc:xxxx://xxxx:xxxx/xxxx";
	Connection conn = DriverManager.getConnection(url,username,password);
	.....
	```



#### SPI机制的简单示例？

我们现在需要使用一个内容搜索接口，搜索的实现可能是基于文件系统的搜索，也可能是基于数据库的搜索。

- 先定义好接口

    ```java
    public interface Search {
        public List<String> searchDoc(String keyword);   
    }
    ```

- 文件搜索实现

    ```java
    public class FileSearch implements Search{
        @Override
        public List<String> searchDoc(String keyword) {
            System.out.println("文件搜索 "+keyword);
            return null;
        }
    }
    ```

- 数据库搜索实现

    ```java
    public class DatabaseSearch implements Search{
        @Override
        public List<String> searchDoc(String keyword) {
            System.out.println("数据搜索 "+keyword);
            return null;
        }
    }
    ```

- resources 接下来可以在resources下新建META-INF/services/目录，然后新建接口全限定名的文件：`com.cainiao.ys.spi.learn.Search`，里面加上我们需要用到的实现类：`com.cainiao.ys.spi.learn.FileSearch`

- 测试方法

    ```java
    public class TestCase {
        public static void main(String[] args) {
            ServiceLoader<Search> s = ServiceLoader.load(Search.class);
            Iterator<Search> iterator = s.iterator();
            while (iterator.hasNext()) {
               Search search =  iterator.next();
               search.searchDoc("hello world");
            }
        }
    }
    ```

可以看到输出结果：

```shell
文件搜索 hello world
```

如果在`com.cainiao.ys.spi.learn.Search`文件里写上两个实现类，那最后的输出结果就是两行了。

这就是因为`ServiceLoader.load(Search.class)`在加载某接口时，会去`META-INF/services`下找接口的全限定名文件，再根据里面的内容加载相应的实现类。

这就是spi的思想，接口的实现由provider实现，provider只用在提交的jar包里的`META-INF/services`下根据平台定义的接口新建文件，并添加进相应的实现类内容就好。



### null值得存放问题

- `HashTable`因为判断要调用`hashCode()`这一个方法，所以会产生**NPE**的问题，故key不能存放`null`；
- `HashMap`可以存放`null`，key也可以为`null`，这是因为计算哈希时做了特殊处理，遇到`null`直接短路返回0；
- `HashSet`底层是`HashMap`，由于要判断是否存在在`set`里，所以填入`HashMap`的占位值不能是`null`，否则存在和不存在`HashMap`的`put`方法返回的都是`null`，无法判断；



