# JDK9

https://www.runoob.com/java/java9-new-features.html

https://docs.oracle.com/javase/9/whatsnew/toc.htm#JSNEW-GUID-C23AFD78-C777-460B-8ACE-58BE5EA681F6

​	Java 9 发布于 2017 年 9 月 22 日，带来了很多新特性，其中最主要的变化是已经实现的模块化系统。接下来我们会详细介绍 Java 9 的新特性。

* **模块系统**：模块是一个包的容器，Java 9 最大的变化之一是引入了模块系统（Jigsaw 项目）。
* **REPL (JShell)**：交互式编程环境。
* **HTTP 2 客户端**：HTTP/2标准是HTTP协议的最新版本，新的 HTTPClient API 支持 WebSocket 和 HTTP2 流以及服务器推送特性。
* **改进的 Javadoc**：Javadoc 现在支持在 API 文档中的进行搜索。另外，Javadoc 的输出现在符合兼容 HTML5 标准。
* **多版本兼容 JAR 包**：多版本兼容 JAR 功能能让你创建仅在特定版本的 Java 环境中运行库程序时选择使用的 class 版本。
* **集合工厂方法**：List，Set 和 Map 接口中，新的静态工厂方法可以创建这些集合的不可变实例。
* **私有接口方法**：在接口中使用private私有方法。我们可以使用 private 访问修饰符在接口中编写私有方法。
* **进程 API**: 改进的 API 来控制和管理操作系统进程。引进 java.lang.ProcessHandle 及其嵌套接口 Info 来让开发者逃离时常因为要获取一个本地进程的 PID 而不得不使用本地代码的窘境。
* **改进的 Stream API**：改进的 Stream API 添加了一些便利的方法，使流处理更容易，并使用收集器编写复杂的查询。
* **改进 try-with-resources**：如果你已经有一个资源是 final 或等效于 final 变量,您可以在 try-with-resources 语句中使用该变量，而无需在 try-with-resources 语句中声明一个新变量。
* **改进的弃用注解 @Deprecated**：注解 @Deprecated 可以标记 Java API 状态，可以表示被标记的 API 将会被移除，或者已经破坏。
* **改进钻石操作符(Diamond Operator)** ：匿名类可以使用钻石操作符(Diamond Operator)。
* **改进 Optional 类**：java.util.Optional 添加了很多新的有用方法，Optional 可以直接转为 stream。
* **多分辨率图像 API**：定义多分辨率图像API，开发者可以很容易的操作和展示不同分辨率的图像了。
* **改进的 CompletableFuture API** ： CompletableFuture 类的异步机制可以在 ProcessHandle.onExit 方法退出时执行操作。
* **轻量级的 JSON API**：内置了一个轻量级的JSON API
* **响应式流（Reactive Streams) API**: Java 9中引入了新的响应式流 API 来支持 Java 9 中的响应式编程。

## Java 9 模块系统

​	Java 9 最大的变化之一是引入了模块系统（Jigsaw 项目）。

​	模块就是代码和数据的封装体。模块的代码被组织成多个包，每个包中包含Java类和接口；模块的数据则包括资源文件和其他静态信息。

​	Java 9 模块的重要特征是在其工件（artifact）的根目录中包含了一个描述模块的 module-info.class 文 件。 工件的格式可以是传统的 JAR 文件或是 Java 9 新增的 JMOD 文件。这个文件由根目录中的源代码文件 module-info.java 编译而来。该模块声明文件可以描述模块的不同特征。

![](https://pic.imgdb.cn/item/614e8c6c2ab3f51d913eafe6.jpg)

​	在 module-info.java 文件中，我们可以用新的关键词module来声明一个模块，如下所示。下面给出了一个模块com.mycompany.mymodule的最基本的模块声明。



```
module top.sorie.jdk9.modules {
}
```

module-info.java 用于创建模块。这一步我们创建了 top.sorie.jdk9.modules模块。

**第三步**

在模块中添加源代码文件，创建文件 ModuleTest.java，代码如下：

```java
package top.sorie.jdk9.modules;

public class ModuleTest {
    public static void main(String[] args) {
        System.out.println("Hello, jdk9 modules.");
    }
}

```

**第四步**

src目录同级创建文件夹 `mods`，然后在该目录下创建 top.sorie.jdk9.modules 文件夹，编译模块到这个目录下：

```
javac -d mods/top.sorie.jdk9.modules src/top.sorie.jdk9.modules/module-info.java  src/top.sorie.jdk9.modules/top/sorie/jdk9/modules/ModuleTest.java
```

**第五步**

执行模块，查看输出结果：

```
java --module-path mods -m top.sorie.jdk9.modules/top.sorie.jdk9.modules.ModuleTest
Hello World!
```

**module-path** 指定了模块所在的路径。

**-m** 指定主要模块。

![](https://pic.imgdb.cn/item/614e8e712ab3f51d914151b5.jpg)

## Java 9 REPL (JShell)

REPL(Read Eval Print Loop)意为交互式的编程环境。

JShell 是 Java 9 新增的一个交互式的编程环境工具。它允许你无需使用类或者方法包装来执行 Java 语句。它与 Python 的解释器类似，可以直接 输入表达式并查看其执行结果。

**执行 JSHELL**

```
$ jshell
|  Welcome to JShell -- Version 9-ea
|  For an introduction type: /help intro
jshell>
```

**查看 JShell 命令**

输入 **/help** 可以查看 JShell相关的命令：

```
jshell> /help
|  Type a Java language expression, statement, or declaration.
|  Or type one of the following commands:
|  /list [<name or id>|-all|-start]
|  list the source you have typed
|  /edit <name or id>
|  edit a source entry referenced by name or id
|  /drop <name or id>
|  delete a source entry referenced by name or id
|  /save [-all|-history|-start] <file>
|  Save snippet source to a file.
|  /open <file>
|  open a file as source input
|  /vars [<name or id>|-all|-start]
|  list the declared variables and their values
|  /methods [<name or id>|-all|-start]
|  list the declared methods and their signatures
|  /types [<name or id>|-all|-start]
|  list the declared types
|  /imports 
|  list the imported items
```

**执行 JShell 命令**

**/imports** 命令用于查看已导入的包：

```
jshell> /imports
|    import java.io.*
|    import java.math.*
|    import java.net.*
|    import java.nio.file.*
|    import java.util.*
|    import java.util.concurrent.*
|    import java.util.function.*
|    import java.util.prefs.*
|    import java.util.regex.*
|    import java.util.stream.*
jshell>
```

**JShell 执行计算**

以下实例执行 JShell 简单计算：

```
jshell> 3+1
$1 ==> 4
jshell> 13%7
$2 ==> 6
jshell> $2
$2 ==> 6
jshell>
```

**JShell 创建与使用函数**

创建一个函数 doubled() ，将传入的整型参数乘于 2 后返回：

```
jshell> int doubled(int i){ return i*2;}
|  created method doubled(int)
jshell> doubled(6)
$3 ==> 12
jshell>
```

**退出 JShell**

输入 /exit 命令退出 jshell：

```
jshell> /exit
| Goodbye 
```

## Java 9 改进 Javadoc

javadoc 工具可以生成 Java 文档， Java 9 的 javadoc 的输出现在符合兼容 HTML5 标准。

**Java 9 之前的旧版本文档**

考虑以下文件代码 C:/JAVA/Tester.java:

**实例**

```java
/**
  * @author MahKumar
  * @version 0.1
*/
public class Tester {
   /**
      * Default method to be run to print 
      * <p>Hello world</p>
      * @param args command line arguments
   */
   public static void main(String []args) {
      System.out.println("Hello World");
   }
}
```

使用 jdk 7 的 javadoc 生成文档：

```
C:\JAVA>javadoc -d C:/JAVA Tester.java
Loading source file tester.java...
Constructing Javadoc information...
Standard Doclet version 1.7.0_21
Building tree for all the packages and classes...
Generating C:\JAVA\Tester.html...
Generating C:\JAVA\package-frame.html...
Generating C:\JAVA\package-summary.html...
Generating C:\JAVA\package-tree.html...
Generating C:\JAVA\constant-values.html...
Building index for all the packages and classes...
Generating C:\JAVA\overview-tree.html...
Generating C:\JAVA\index-all.html...
Generating C:\JAVA\deprecated-list.html...
Building index for all classes...
Generating C:\JAVA\allclasses-frame.html...
Generating C:\JAVA\allclasses-noframe.html...
Generating C:\JAVA\index.html...
Generating C:\JAVA\help-doc.html...
```

执行以上命令会再 C:/JAVA 命令下生成文档页面，如下图所示：

![img](https://www.runoob.com/wp-content/uploads/2018/06/1528617613-1757-javadoc-output.jpg)

**Java 9 生成的文档兼容 HTML5 标准**

使用 jdk 9 javadoc 命令中的 **-html5** 参数可以让生成的文档支持 HTML5 标准：

```
C:\JAVA> javadoc -d C:/JAVA -html5 Tester.java
Loading source file Tester.java...
Constructing Javadoc information...
Standard Doclet version 9.0.1
Building tree for all the packages and classes...
Generating C:\JAVA\Tester.html...
Generating C:\JAVA\package-frame.html...
Generating C:\JAVA\package-summary.html...
Generating C:\JAVA\package-tree.html...
Generating C:\JAVA\constant-values.html...
Building index for all the packages and classes...
Generating C:\JAVA\overview-tree.html...
Generating C:\JAVA\index-all.html...
Generating C:\JAVA\deprecated-list.html...
Building index for all classes...
Generating C:\JAVA\allclasses-frame.html...
Generating C:\JAVA\allclasses-frame.html...
Generating C:\JAVA\allclasses-noframe.html...
Generating C:\JAVA\allclasses-noframe.html...
Generating C:\JAVA\index.html...
Generating C:\JAVA\help-doc.html...
```

执行以上命令会再 C:/JAVA 命令下生成文档页面，如下图所示：

![img](https://www.runoob.com/wp-content/uploads/2018/06/1528617610-9326-javadoc-output1.jpg)

## Java 9 多版本兼容 jar 包

多版本兼容 JAR 功能能让你创建仅在特定版本的 Java 环境中运行库程序时选择使用的 class 版本。

通过 **--release** 参数指定编译版本。

具体的变化就是 META-INF 目录下 MANIFEST.MF 文件新增了一个属性：

```
Multi-Release: true
```

然后 META-INF 目录下还新增了一个 versions 目录，如果是要支持 java9，则在 versions 目录下有 9 的目录。

```
multirelease.jar
├── META-INF
│   └── versions
│       └── 9
│           └── multirelease
│               └── Helper.class
├── multirelease
    ├── Helper.class
    └── Main.class
```

在以下实例中，我们使用多版本兼容 JAR 功能将 Tester.java 文件生成了两个版本的 jar 包, 一个是 jdk 7，另一个是 jdk 9，然后我们再不同环境下执行。

**第一步**

创建文件夹 c:/test/java7/com/runoob，并在该文件夹下创建 Test.java 文件，代码如下：

```
package com.runoob;

public class Tester {
   public static void main(String[] args) {
      System.out.println("Inside java 7");
   }
}
```

**第二步**

创建文件夹 c:/test/java9/com/runoob，并在该文件夹下创建 Test.java 文件，代码如下：

```
package com.runoob;

public class Tester {
   public static void main(String[] args) {
      System.out.println("Inside java 9");
   }
}
```

编译源代码：

```
C:\test > javac --release 9 java9/com/runoob/Tester.java

C:\JAVA > javac --release 7 java7/com/runoob/Tester.java
```

创建多版本兼容 jar 包

```
C:\JAVA > jar -c -f test.jar -C java7 . --release 9 -C java9.
Warning: entry META-INF/versions/9/com/runoob/Tester.java, 
   multiple resources with same name
```

使用 JDK 7 执行：

```
C:\JAVA > java -cp test.jar com.runoob.Tester
Inside Java 7
```

使用 JDK 9 执行：

```
C:\JAVA > java -cp test.jar com.runoob.Tester
Inside Java 9
```

![](https://pic.imgdb.cn/item/614e91622ab3f51d914546ce.jpg)

## Java 9 集合工厂方法

Java 9 List，Set 和 Map 接口中，新的静态工厂方法可以创建这些集合的不可变实例。

这些工厂方法可以以更简洁的方式来创建集合。

**旧方法创建集合**

**实例**

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;
 
public class Tester {
   public static void main(String []args) {
      Set<String> set = new HashSet<>();
      set.add("A");
      set.add("B");
      set.add("C");
      set = Collections.unmodifiableSet(set);
      System.out.println(set);
      List<String> list = new ArrayList<>();
 
      list.add("A");
      list.add("B");
      list.add("C");
      list = Collections.unmodifiableList(list);
      System.out.println(list);
      Map<String, String> map = new HashMap<>();
 
      map.put("A","Apple");
      map.put("B","Boy");
      map.put("C","Cat");
      map = Collections.unmodifiableMap(map);
      System.out.println(map);
   }
}
```



执行输出结果为：

```
[A, B, C]
[A, B, C]
{A=Apple, B=Boy, C=Cat}
```

**新方法创建集合**

Java 9 中，以下方法被添加到 List，Set 和 Map 接口以及它们的重载对象。

```
static <E> List<E> of(E e1, E e2, E e3);
static <E> Set<E>  of(E e1, E e2, E e3);
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3);
static <K,V> Map<K,V> ofEntries(Map.Entry<? extends K,? extends V>... entries)
```

* List 和 Set 接口, of(...) 方法重载了 0 ~ 10 个参数的不同方法 。
* Map 接口, of(...) 方法重载了 0 ~ 10 个参数的不同方法 。
* Map 接口如果超过 10 个参数, 可以使用 ofEntries(...) 方法。

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.AbstractMap;
import java.util.Map;
import java.util.Set;
 
public class Tester {
 
   public static void main(String []args) {
      Set<String> set = Set.of("A", "B", "C");      
      System.out.println(set);
      List<String> list = List.of("A", "B", "C");
      System.out.println(list);
      Map<String, String> map = Map.of("A","Apple","B","Boy","C","Cat");
      System.out.println(map);
  
      Map<String, String> map1 = Map.ofEntries (
         new AbstractMap.SimpleEntry<>("A","Apple"),
         new AbstractMap.SimpleEntry<>("B","Boy"),
         new AbstractMap.SimpleEntry<>("C","Cat"));
      System.out.println(map1);
   }
}
```

输出结果为：

```java
[A, B, C]
[A, B, C]
{A=Apple, B=Boy, C=Cat}
{A=Apple, B=Boy, C=Cat}
```

## Java 9 私有接口方法

在 Java 8之前，接口可以有常量变量和抽象方法。

我们不能在接口中提供方法实现。如果我们要提供抽象方法和非抽象方法（方法与实现）的组合，那么我们就得使用抽象类。

```java
public class Tester {
   public static void main(String []args) {
      LogOracle log = new LogOracle();
      log.logInfo("");
      log.logWarn("");
      log.logError("");
      log.logFatal("");
      LogMySql log1 = new LogMySql();
      log1.logInfo("");
      log1.logWarn("");
      log1.logError("");
      log1.logFatal("");
   }
}
final class LogOracle implements Logging {
   @Override
   public void logInfo(String message) {
      getConnection();
      System.out.println("Log Message : " + "INFO");
      closeConnection();
   }
   @Override
   public void logWarn(String message) {
      getConnection();
      System.out.println("Log Message : " + "WARN");
      closeConnection();
   }
   @Override
   public void logError(String message) {
      getConnection();
      System.out.println("Log Message : " + "ERROR");
      closeConnection();
   }
   @Override
   public void logFatal(String message) {
      getConnection();
      System.out.println("Log Message : " + "FATAL");
      closeConnection();
   }
   @Override
   public void getConnection() {
      System.out.println("Open Database connection");
   }
   @Override
   public void closeConnection() {
      System.out.println("Close Database connection");
   }
}
final class LogMySql implements Logging {
   @Override
   public void logInfo(String message) {
      getConnection();
      System.out.println("Log Message : " + "INFO");
      closeConnection();
   }
   @Override
   public void logWarn(String message) {
      getConnection();
      System.out.println("Log Message : " + "WARN");
      closeConnection();
   }
   @Override
   public void logError(String message) {
      getConnection();
      System.out.println("Log Message : " + "ERROR");
      closeConnection();
   }
   @Override
   public void logFatal(String message) {
      getConnection();
      System.out.println("Log Message : " + "FATAL");
      closeConnection();
   }
   @Override
   public void getConnection() {
      System.out.println("Open Database connection");
   }
   @Override
   public void closeConnection() {
      System.out.println("Close Database connection");
   }
}
interface Logging {
   String ORACLE = "Oracle_Database";
   String MYSQL = "MySql_Database";
 
   void logInfo(String message);
   void logWarn(String message);
   void logError(String message);
   void logFatal(String message);
 
   void getConnection();
   void closeConnection();
}
```

以上实例执行输出结果为：

```java
Open Database connection
Log Message : INFO
Close Database connection
Open Database connection
Log Message : WARN
Close Database connection
Open Database connection
Log Message : ERROR
Close Database connection
Open Database connection
Log Message : FATAL
Close Database connection
```

在上面的例子中，每个日志方法都有自己的实现。

在 Java 8 接口引入了一些新功能——默认方法和静态方法。我们可以在Java SE 8的接口中编写方法实现，仅仅需要使用 **default** 关键字来定义它们。

在 Java 8 中，一个接口中能定义如下几种变量/方法：

* **常量**
* **抽象方法**
* **默认方法**
* **静态方法**

```java
public class Tester {
   public static void main(String []args) {
      LogOracle log = new LogOracle();
      log.logInfo("");
      log.logWarn("");
      log.logError("");
      log.logFatal("");
      
      LogMySql log1 = new LogMySql();
      log1.logInfo("");
      log1.logWarn("");
      log1.logError("");
      log1.logFatal("");
   }
}
final class LogOracle implements Logging { 
}
final class LogMySql implements Logging { 
}
interface Logging {
   String ORACLE = "Oracle_Database";
   String MYSQL = "MySql_Database";
 
   default void logInfo(String message) {
      getConnection();
      System.out.println("Log Message : " + "INFO");
      closeConnection();
   }
   default void logWarn(String message) {
      getConnection();
      System.out.println("Log Message : " + "WARN");
      closeConnection();
   }
   default void logError(String message) {
      getConnection();
      System.out.println("Log Message : " + "ERROR");
      closeConnection();
   }
   default void logFatal(String message) {
      getConnection();
      System.out.println("Log Message : " + "FATAL");
      closeConnection();
   }
   static void getConnection() {
      System.out.println("Open Database connection");
   }
   static void closeConnection() {
      System.out.println("Close Database connection");
   }
}
```

以上实例执行输出结果为：

```
Open Database connection
Log Message : INFO
Close Database connection
Open Database connection
Log Message : WARN
Close Database connection
Open Database connection
Log Message : ERROR
Close Database connection
Open Database connection
Log Message : FATAL
Close Database connection
```

Java 9 不仅像 Java 8 一样支持接口默认方法，同时还支持私有方法。

在 Java 9 中，一个接口中能定义如下几种变量/方法：

* **常量**
* **抽象方法**
* **默认方法**
* **静态方法**
* **私有方法**
* **私有静态方法**

以下实例提取了冗余到通用方法，看起来明显更简洁：

```java
public class Tester {
   public static void main(String []args) {
      LogOracle log = new LogOracle();
      log.logInfo("");
      log.logWarn("");
      log.logError("");
      log.logFatal("");
      
      LogMySql log1 = new LogMySql();
      log1.logInfo("");
      log1.logWarn("");
      log1.logError("");
      log1.logFatal("");
   }
}
final class LogOracle implements Logging { 
}
final class LogMySql implements Logging { 
}
interface Logging {
   String ORACLE = "Oracle_Database";
   String MYSQL = "MySql_Database";
 
   private void log(String message, String prefix) {
      getConnection();
      System.out.println("Log Message : " + prefix);
      closeConnection();
   }
   default void logInfo(String message) {
      log(message, "INFO");
   }
   default void logWarn(String message) {
      log(message, "WARN");
   }
   default void logError(String message) {
      log(message, "ERROR");
   }
   default void logFatal(String message) {
      log(message, "FATAL");
   }
   private static void getConnection() {
      System.out.println("Open Database connection");
   }
   private static void closeConnection() {
      System.out.println("Close Database connection");
   }
}
```

以上实例执行输出结果为：

```java
Open Database connection
Log Message : INFO
Close Database connection
Open Database connection
Log Message : WARN
Close Database connection
Open Database connection
Log Message : ERROR
Close Database connection
Open Database connection
Log Message : FATAL
Close Database connection
```

## Java 9 改进的进程 API

​	在 Java 9 之前，Process API 仍然缺乏对使用本地进程的基本支持，例如获取进程的 PID 和所有者，进程的开始时间，进程使用了多少 CPU 时间，多少本地进程正在运行等。

​	Java 9 向 Process API 添加了一个名为 ProcessHandle 的接口来增强 java.lang.Process 类。

​	ProcessHandle 接口的实例标识一个本地进程，它允许查询进程状态并管理进程。

​	ProcessHandle 嵌套接口 Info 来让开发者逃离时常因为要获取一个本地进程的 PID 而不得不使用本地代码的窘境。

​	我们不能在接口中提供方法实现。如果我们要提供抽象方法和非抽象方法（方法与实现）的组合，那么我们就得使用抽象类。

​	ProcessHandle 接口中声明的 onExit() 方法可用于在某个进程终止时触发某些操作。

```java
import java.time.ZoneId;
import java.util.stream.Stream;
import java.util.stream.Collectors;
import java.io.IOException;
 
public class Tester {
   public static void main(String[] args) throws IOException {
      ProcessBuilder pb = new ProcessBuilder("notepad.exe");
      String np = "Not Present";
      Process p = pb.start();
      ProcessHandle.Info info = p.info();
      System.out.printf("Process ID : %s%n", p.pid());
      System.out.printf("Command name : %s%n", info.command().orElse(np));
      System.out.printf("Command line : %s%n", info.commandLine().orElse(np));
 
      System.out.printf("Start time: %s%n",
         info.startInstant().map(i -> i.atZone(ZoneId.systemDefault())
         .toLocalDateTime().toString()).orElse(np));
 
      System.out.printf("Arguments : %s%n",
         info.arguments().map(a -> Stream.of(a).collect(
         Collectors.joining(" "))).orElse(np));
 
      System.out.printf("User : %s%n", info.user().orElse(np));
   } 
}
```

以上实例执行输出结果为：

```
Process ID : 5800
Command name : C:\Windows\System32\notepad.exe
Command line : Not Present
Start time: 2017-11-04T21:35:03.626
Arguments : Not Present
User: administrator
```

## Java 9 改进的 Stream API

​	

Java 9 改进的 Stream API 添加了一些便利的方法，使流处理更容易，并使用收集器编写复杂的查询。

Java 9 为 Stream 新增了几个方法：dropWhile、takeWhile、ofNullable，为 iterate 方法新增了一个重载方法。

### takeWhile 方法

**语法**

```
default Stream<T> takeWhile(Predicate<? super T> predicate)
```

takeWhile() 方法使用一个断言作为参数，返回给定 Stream 的子集直到断言语句第一次返回 false。如果第一个值不满足断言条件，将返回一个空的 Stream。

takeWhile() 方法在有序的 Stream 中，takeWhile 返回从开头开始的尽量多的元素；在无序的 Stream 中，takeWhile 返回从开头开始的符合 Predicate 要求的元素的子集。

**实例**

```java
import java.util.stream.Stream;
 
public class Tester {
   public static void main(String[] args) {
      Stream.of("a","b","c","","e","f").takeWhile(s->!s.isEmpty())
         .forEach(System.out::print);      
   } 
}
```



以上实例 takeWhile 方法在碰到空字符串时停止循环输出，执行输出结果为：

```
abc
```

### dropWhile 方法

**语法**

```
default Stream<T> dropWhile(Predicate<? super T> predicate)
```

​	dropWhile 方法和 takeWhile 作用相反的，使用一个断言作为参数，直到断言语句第一次返回 false 才返回给定 Stream 的子集。

```java
import java.util.stream.Stream;
 
public class Tester {
   public static void main(String[] args) {
      Stream.of("a","b","c","","e","f").dropWhile(s-> !s.isEmpty())
         .forEach(System.out::print);
   } 
}
```

以上实例 dropWhile 方法在碰到空字符串时开始循环输出，执行输出结果为：

```
ef
```

### iterate 方法

**语法**

```
static <T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next)
```

​	方法允许使用初始种子值创建顺序（可能是无限）流，并迭代应用指定的下一个方法。 当指定的 hasNext 的 predicate 返回 false 时，迭代停止。

```java
java.util.stream.IntStream;
 
public class Tester {
   public static void main(String[] args) {
      IntStream.iterate(3, x -> x < 10, x -> x+ 3).forEach(System.out::println);
   } 
}
```

执行输出结果为：

```
3
6
9
```

### OfNullable

**语法**

```
static <T> Stream<T> ofNullable(T t)
```

ofNullable 方法可以预防 NullPointerExceptions 异常， 可以通过检查流来避免 null 值。

如果指定元素为非 null，则获取一个元素并生成单个元素流，元素为 null 则返回一个空流。

**实例**

```java
import java.util.stream.Stream;
 
public class Tester {
   public static void main(String[] args) {
      long count = Stream.ofNullable(100).count();
      System.out.println(count);
  
      count = Stream.ofNullable(null).count();
      System.out.println(count);
   } 
}
```

执行输出结果为：

```
1
0
```

## Java 9 改进的 try-with-resources

​	try-with-resources 是 JDK 7 中一个新的异常处理机制，它能够很容易地关闭在 try-catch 语句块中使用的资源。所谓的资源（resource）是指在程序完成后，必须关闭的对象。try-with-resources 语句确保了每个资源在语句结束时关闭。所有实现了 java.lang.AutoCloseable 接口（其中，它包括实现了 java.io.Closeable 的所有对象），可以使用作为资源。

​	try-with-resources 声明在 JDK 9 已得到改进。如果你已经有一个资源是 final 或等效于 final 变量,您可以在 try-with-resources 语句中使用该变量，而无需在 try-with-resources 语句中声明一个新变量。

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.Reader;
import java.io.StringReader;
 
public class Tester {
   public static void main(String[] args) throws IOException {
      System.out.println(readData("test"));
   } 
   static String readData(String message) throws IOException {
      Reader inputString = new StringReader(message);
      BufferedReader br = new BufferedReader(inputString);
      try (BufferedReader br1 = br) {
         return br1.readLine();
      }
   }
}
```

输出结果为：

```
test
```

以上实例中我们需要在 try 语句块中声明资源 br1，然后才能使用它。

在 Java 9 中，我们不需要声明资源 br1 就可以使用它，并得到相同的结果。

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.Reader;
import java.io.StringReader;
 
public class Tester {
   public static void main(String[] args) throws IOException {
      System.out.println(readData("test"));
   } 
   static String readData(String message) throws IOException {
      Reader inputString = new StringReader(message);
      BufferedReader br = new BufferedReader(inputString);
      try (br) {
         return br.readLine();
      }
   }
}
```

执行输出结果为：

```
test
```

​	在处理必须关闭的资源时，使用try-with-resources语句替代try-finally语句。 生成的代码更简洁，更清晰，并且生成的异常更有用。 try-with-resources语句在编写必须关闭资源的代码时会更容易，也不会出错，而使用try-finally语句实际上是不可能的。

## Java 9 改进的 @Deprecated 注解

注解 @Deprecated 可以标记 Java API 状态，可以是以下几种：

* 使用它存在风险，可能导致错误
* 可能在未来版本中不兼容
* 可能在未来版本中删除
* 一个更好和更高效的方案已经取代它。

Java 9 中注解增加了两个新元素：**since** 和 **forRemoval**。

* **since**: 元素指定已注解的API元素已被弃用的版本。
* **forRemoval**: 元素表示注解的 API 元素在将来的版本中被删除，应该迁移 API。

以下实例为 Java 9 中关于 Boolean 类的说明文档，文档中 @Deprecated 注解使用了 since 属性：[Boolean Class](https://docs.oracle.com/javase/9/docs/api/java/lang/Boolean.html#Boolean-boolean-)。

![img](https://www.runoob.com/wp-content/uploads/2018/06/1528695298-2284-boolean.jpg)

以下实例为在 Java 9 中关于系统类的说明文档，文档中 @Deprecated 注解使用了 forRemoval 属性：[System Class](https://docs.oracle.com/javase/9/docs/api/java/lang/System.html#runFinalizersOnExit-boolean-)。

![img](https://www.runoob.com/wp-content/uploads/2018/06/1528695298-6362-system.jpg)

## Java 9 钻石操作符(Diamond Operator)

​	钻石操作符是在 java 7 中引入的，可以让代码更易读，但它不能用于匿名的内部类。

​	在 java 9 中， 它可以与匿名的内部类一起使用，从而提高代码的可读性。

​	考虑以下 Java 9 之前的代码：

```java
public class Tester {
   public static void main(String[] args) {
      Handler<Integer> intHandler = new Handler<Integer>(1) {
         @Override
         public void handle() {
            System.out.println(content);
         }
      };
      intHandler.handle();
      Handler<? extends Number> intHandler1 = new Handler<Number>(2) {
         @Override
         public void handle() {
            System.out.println(content);
         }
      };
      intHandler1.handle();
      Handler<?> handler = new Handler<Object>("test") {
         @Override
         public void handle() {
            System.out.println(content);
         }
      };
      handler.handle();    
   }  
}
abstract class Handler<T> {
   public T content;
 
   public Handler(T content) {
      this.content = content; 
   }
   
   abstract void handle();
}
```

执行输出结果为：

```
1
2
Test
```

在 Java 9 中，我们可以在匿名类中使用 <> 操作符，如下所示：

```java
public class Tester {
   public static void main(String[] args) {
      Handler<Integer> intHandler = new Handler<>(1) {
         @Override
         public void handle() {
            System.out.println(content);
         }
      };
      intHandler.handle();
      Handler<? extends Number> intHandler1 = new Handler<>(2) {
         @Override
         public void handle() {
            System.out.println(content);
         }
      };
      intHandler1.handle();
      Handler<?> handler = new Handler<>("test") {
         @Override
         public void handle() {
            System.out.println(content);
         }
      };
 
      handler.handle();    
   }  
}
 
abstract class Handler<T> {
   public T content;
 
   public Handler(T content) {
      this.content = content; 
   }
   
   abstract void handle();
}
```

执行输出结果为：

```java
1
2
Test
```

## Java 9 改进的 Optional 类

Optional 类在 Java 8 中引入，Optional 类的引入很好的解决空指针异常。。在 java 9 中, 添加了三个方法来改进它的功能：

* stream()
* ifPresentOrElse()
* or()

### stream() 方法

**语法**

```
public Stream<T> stream()
```

​	stream 方法的作用就是将 Optional 转为一个 Stream，如果该 Optional 中包含值，那么就返回包含这个值的 Stream，否则返回一个空的 Stream（Stream.empty()）。

```java
import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;
 
public class Tester {
public static void main(String[] args) {
   List<Optional<String>> list = Arrays.asList (
      Optional.empty(), 
      Optional.of("A"), 
      Optional.empty(), 
      Optional.of("B"));
 
      //filter the list based to print non-empty values
  
      //if optional is non-empty, get the value in stream, otherwise return empty
      List<String> filteredList = list.stream()
         .flatMap(o -> o.isPresent() ? Stream.of(o.get()) : Stream.empty())
         .collect(Collectors.toList());
 
      //Optional::stream method will return a stream of either one 
      //or zero element if data is present or not.
      List<String> filteredListJava9 = list.stream()
         .flatMap(Optional::stream)
         .collect(Collectors.toList());
 
      System.out.println(filteredList);
      System.out.println(filteredListJava9);
   }  
}
```

执行输出结果为：

```
[A, B]
[A, B]
```

### ifPresentOrElse() 方法

**语法**

```
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)
```

​	ifPresentOrElse 方法的改进就是有了 else，接受两个参数 Consumer 和 Runnable。

​	ifPresentOrElse 方法的用途是，如果一个 Optional 包含值，则对其包含的值调用函数 action，即 action.accept(value)，这与 ifPresent 一致；与 ifPresent 方法的区别在于，ifPresentOrElse 还有第二个参数 emptyAction —— 如果 Optional 不包含值，那么 ifPresentOrElse 便会调用 emptyAction，即 emptyAction.run()。

```java
import java.util.Optional;
 
public class Tester {
   public static void main(String[] args) {
      Optional<Integer> optional = Optional.of(1);
 
      optional.ifPresentOrElse( x -> System.out.println("Value: " + x),() -> 
         System.out.println("Not Present."));
 
      optional = Optional.empty();
 
      optional.ifPresentOrElse( x -> System.out.println("Value: " + x),() -> 
         System.out.println("Not Present."));
   }  
}
```

执行输出结果为：

```
Value: 1
Not Present.
```

### or() 方法

**语法**

```
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)
```

如果值存在，返回 Optional 指定的值，否则返回一个预设的值。

```java
import java.util.Optional;
import java.util.function.Supplier;
 
public class Tester {
   public static void main(String[] args) {
      Optional<String> optional1 = Optional.of("Mahesh");
      Supplier<Optional<String>> supplierString = () -> Optional.of("Not Present");
      optional1 = optional1.or( supplierString);
      optional1.ifPresent( x -> System.out.println("Value: " + x));
      optional1 = Optional.empty();    
      optional1 = optional1.or( supplierString);
      optional1.ifPresent( x -> System.out.println("Value: " + x));  
   }  
}
```

执行输出结果为：

```
Value: Mahesh
Value: Not Present
```

## Java 9 多分辨率图像 API

Java 9 定义多分辨率图像 API，开发者可以很容易的操作和展示不同分辨率的图像了。

以下是多分辨率图像的主要操作方法：

* **Image getResolutionVariant(double destImageWidth, double destImageHeight)** − 获取特定分辨率的图像变体-表示一张已知分辨率单位为DPI的特定尺寸大小的逻辑图像，并且这张图像是最佳的变体。。
* ``**List<Image> getResolutionVariants()**`` − 返回可读的分辨率的图像变体列表。

```java
import java.io.IOException;
import java.net.URL;
import java.net.MalformedURLException;
import java.util.ArrayList;
import java.util.List;
import java.awt.Image;
import java.awt.image.MultiResolutionImage;
import java.awt.image.BaseMultiResolutionImage;
 
import javax.imageio.ImageIO;
 
public class Tester {
   public static void main(String[] args) throws IOException, MalformedURLException {
 
      List<String> imgUrls = List.of("http://www.runoob.com/wp-content/themes/runoob/assets/img/runoob-logo@2x.png",
         "http://www.runoob.com/wp-content/themes/runoob/assets/img/runoob-logo.png",
         "http://www.runoob.com/wp-content/themes/runoob/assets/images/qrcode.png");
 
      List<Image> images = new ArrayList<Image>();
 
      for (String url : imgUrls) {
         images.add(ImageIO.read(new URL(url)));
      }
 
      // 读取所有图片
      MultiResolutionImage multiResolutionImage = 
         new BaseMultiResolutionImage(images.toArray(new Image[0]));
 
      // 获取图片的所有分辨率
      List<Image> variants = multiResolutionImage.getResolutionVariants();
 
      System.out.println("Total number of images: " + variants.size());
 
      for (Image img : variants) {
         System.out.println(img);
      }
 
      // 根据不同尺寸获取对应的图像分辨率
      Image variant1 = multiResolutionImage.getResolutionVariant(156, 45);
      System.out.printf("\nImage for destination[%d,%d]: [%d,%d]", 
         156, 45, variant1.getWidth(null), variant1.getHeight(null));
 
      Image variant2 = multiResolutionImage.getResolutionVariant(311, 89);
      System.out.printf("\nImage for destination[%d,%d]: [%d,%d]", 311, 89, 
         variant2.getWidth(null), variant2.getHeight(null));
 
      Image variant3 = multiResolutionImage.getResolutionVariant(622, 178);
      System.out.printf("\nImage for destination[%d,%d]: [%d,%d]", 622, 178, 
         variant3.getWidth(null), variant3.getHeight(null));
 
      Image variant4 = multiResolutionImage.getResolutionVariant(300, 300);
      System.out.printf("\nImage for destination[%d,%d]: [%d,%d]", 300, 300, 
         variant4.getWidth(null), variant4.getHeight(null));
   }  
}
```

## Java 9 改进的 CompletableFuture API

Java 8 引入了 **CompletableFuture<T>** 类，可能是 **java.util.concurrent.Future<T>** 明确的完成版（设置了它的值和状态），也可能被用作**java.util.concurrent.CompleteStage** 。支持 future 完成时触发一些依赖的函数和动作。Java 9 引入了一些**CompletableFuture** 的改进：

Java 9 对 **CompletableFuture** 做了改进：

* 支持 delays 和 timeouts
* 提升了对子类化的支持
* 新的工厂方法

### 支持 delays 和 timeouts

```
public CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)
```

在 **timeout**（单位在 **java.util.concurrent.Timeunits units** 中，比如 MILLISECONDS ）前以给定的 value 完成这个 CompletableFutrue。返回这个 CompletableFutrue。

```
public CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)
```

如果没有在给定的 timeout 内完成，就以 java.util.concurrent.TimeoutException 完成这个 CompletableFutrue，并返回这个 CompletableFutrue。

### 增强了对子类化的支持

做了许多改进使得 **CompletableFuture** 可以被更简单的继承。比如，你也许想重写新的 **public Executor defaultExecutor()** 方法来代替默认的 **executor**。

另一个新的使子类化更容易的方法是：

```
public <U> CompletableFuture<U> newIncompleteFuture()
```

### 新的工厂方法

Java 8引入了 `<U> CompletableFuture<U> completedFuture(U value)`工厂方法来返回一个已经以给定 value 完成了的 **CompletableFuture**。Java 9以 一个新的 `<U> CompletableFuture<U> failedFuture(Throwable ex)` 来补充了这个方法，可以返回一个以给定异常完成的 **CompletableFuture**。

除此以外，Java 9 引入了下面这对 stage-oriented 工厂方法，返回完成的或异常完成的 completion stages:

* ``**<U> CompletionStage<U> completedStage(U value)**: `返回一个新的以指定 value 完成的**CompletionStage** ，并且只支持 **CompletionStage** 里的接口。
* `**<U> CompletionStage<U> failedStage(Throwable ex)**`: 返回一个新的以指定异常完成的`**CompletionStage** `，并且只支持 **CompletionStage** 里的接口。

# JDK10

JDK10 包含 12 个JEP (改善提议）：

- [【286】局部变量类型推断](http://openjdk.java.net/jeps/286) ：对于开发者来说，这是 JDK10 唯一的真正特性。它向 Java 中引入在其他语言中很常见的 *var*  ，比如 JavaScript 。只要编译器可以推断此种类型，你不再需要专门声明一个局部变量的类型。一个简单的例子是：

      ```
      var x = new ArrayList<String>();
      ```

  这就消除了我们之前必须执行的` ArrayList<String>` 类型定义的重复。我鼓励你们去读 JEP ，因为上面有一些关于这个句法是否能用的规则。

  有趣的是，需要注意 *var* 不能成为一个关键字，而是一个保留字。这意味着你仍然可以使用 var 作为一个变量，方法或包名，但是现在（尽管我确定你绝不会）你不能再有一个类被调用。

- [[310\]应用类数据共享](http://openjdk.java.net/jeps/310)(CDS) ：CDS 在 JDK5 时被引进以改善 JVM 启动的表现，同时减少当多个虚拟机在同一个物理或虚拟的机器上运行时的资源占用。

  JDK10 将扩展 CDS 到允许内部系统的类加载器、内部平台的类加载器和自定义类加载器来加载获得的类。之前，CDS 的使用仅仅限制在了 bootstrap 的类加载器。

- [[314\]额外的 Unicode 语言标签扩展](http://openjdk.java.net/jeps/314)：这将改善 java.util.Locale 类和相关的 API 以实现额外 BCP 47 语言标签的 Unicode 扩展。尤其是，货币类型，一周的第一天，区域覆盖和时区等标签现在将被支持。

- [[322\]基于时间的版本控制](http://openjdk.java.net/jeps/322)：正如我在之前的博客中所讨论的，我们的 JDK 版本字符串格式几乎与 JDK 版本一样多。有幸的是，这是最后需要使用到的，我们可以坚持用它。这种格式使用起来很像 JDK9 中介绍的提供一个更加语义的形式。有一件困扰我的事是包含了一个 INTERIM 元素，正如 JEP 提议中所说，“永远是0”。好吧，如果永远是0，那它有什么意义呢？他们说这是为未来使用做保留，但我仍不是很赞同。我认为，这有些冗余繁杂。

  这也消除了在 JDK9 中有过的相当奇怪的情形。第一次更新是 JDK 9.0.1 , 非常符合逻辑。第二次更新是 JDK 9.0.4 ，不合逻辑。原因是，在 JDK9 的版本计数模式下，需要留下空白以便应急或不在预期安排的更新使用。但既然没有更新是必须的，为什么不简单称之为 JDK 9.0.2 呢？

- [[319\]根证书](http://openjdk.java.net/jeps/319)：在 JDK 中将提供一套默认的 CA 根证书。关键的安全部件，如 TLS ，在 OpenJDK 构建中将默认有效。这是 Oracle 正在努力确保 OpenJDK 二进制和 Oracle JDK 二进制功能上一样的工作的一部分，是一项有用的补充内容。

-  [[307\] 并行全垃圾回收器 G1](http://openjdk.java.net/jeps/307) : G1 是设计来作为一种低延时的垃圾回收器（但是如果它跟不上旧的堆碎片产生的提升速率的话，将仍然采用完整压缩集合）。在 JDK9 之前，默认的收集器是并行，吞吐，收集器。为了减少在使用默认的收集器的应用性能配置文件的差异，G1 现在有一个并行完整收集机制。

- [[313\]移除 Native-Header 自动生成工具](http://openjdk.java.net/jeps/313)：Java9 开始了一些对 JDK 的家务管理，这项特性是对它的延续。当编译 JNI 代码时，已不再需要单独的工具来生成头文件，因为这可以通过 javac 完成。在未来的某一时刻，JNI 将会被 Panama 项目的结果取代，但是何时发生还不清楚。

- [[304\]垃圾回收器接口](http://openjdk.java.net/jeps/304): 这不是让开发者用来控制垃圾回收的接口；而是一个在 JVM 源代码中的允许另外的垃圾回收器快速方便的集成的接口。

- [[312\]线程-局部变量管控](http://openjdk.java.net/jeps/312)：这是在 JVM 内部相当低级别的更改，现在将允许在不运行全局虚拟机安全点的情况下实现线程回调。这将使得停止单个线程变得可能和便宜，而不是只能启用或停止所有线程。

- [[316\]在备用存储装置上的堆分配](http://openjdk.java.net/jeps/316)：硬件技术在持续进化，现在可以使用与传统 DRAM 具有相同接口和类似性能特点的非易失性 RAM 。这项 JEP 将使得 JVM 能够使用适用于不同类型的存储机制的堆。

- [[317\] 试验性的基于 Java 的 JIT 编译器](http://openjdk.java.net/jeps/317)：最近宣布的 Metropolis 项目，提议用 Java 重写大部分 JVM 。乍一想，觉得很奇怪。如果 JVM 是用 Java 编写的，那么是否需要一个 JVM 来运行 JVM ？ 相应的，这导致了一个很好的镜像类比。 现实情况是，使用 Java 编写 JVM 并不意味着必须将其编译为字节码，你可以使用 AOT 编译，然后在运行时编译代码以提高性能。

  这项 JEP 将 Graal 编译器研究项目引入到 JDK 中。并给将 Metropolis 项目成为现实，使 JVM 性能与当前 C++ 所写版本匹敌（或有幸超越）提供基础。

- [[296\]: 合并 JDK 多个代码仓库到一个单独的储存库中](http://openjdk.java.net/jeps/296)：在 JDK9 中，有 8 个仓库： root、corba、hotspot、jaxp、jaxws、jdk、langtools 和 nashorn 。在 JDK10 中这些将被合并为一个，使得跨相互依赖的变更集的存储库运行 atomic commit （原子提交）成为可能。

### 新 API

有 73 项新增内容添加到了标准类库中。

- **java.awt.Toolkit
  int getMenuShortcutKeyMaskEx()**: 确定哪个扩展修饰符键是菜单快捷键的适当加速键。
- **java.awt.geom.Path2D**: 
  **void trimToSize()**: 将此 Path2D 实例的容量计算到它当前的大小。应用可使用此操作将路径的存储空间最小化。这个方法也被添加到 Path2D.Double 和 Path2D.Float 类。
- **java.io.ByteArrayOutputStream:
  String toString(Charset)**: 重载 toString()，通过使用指定的字符集解码字节，将缓冲区的内容转换为字符串。
- **java****.io.PrintStream:
  lang.io.PrintWriter:**
  这两个类都有三个新的构造函数，它们需要额外的 Charset 参数。
- **java.io.Reader:
  long transferTo(Writer):** 从这个 Reader 中读取所有字符，并按照所读的顺序将字符写入给定的 Writer 。
- **java.lang.Runtime.Version:**
  有四种新方法返回新（JEP 322）版本字符串字段的整数值: feature()、interim()、patch() 和 update() 。

- **java.lang.StackWalker.StackFrame**:
  **String getDescriptor()**: 按照 JVM 标准返回此堆栈帧所代表的方法的描述符。
  **String getMethodType()**:返回此堆栈帧所代表的方法类型，描述参数类型和返回值类型。
- **java.lang.invoke.MethodType:
  Class<?> lastParameterType():**返回这个方法类型的最后一个参数类型。如果这个方法类型没有参数，则返回空类型作为岗哨值（*Sentinel* *Value*）。
- **java.lang.management.RuntimeMXBean:
  long getPid():** R 返回正在运行的 JVM 的进程 ID 。
- **java.lang.management.ThreadMXBean:
  ThreadInfo[] dumpAllThreads(boolean, boolean, int):** 返回所有活动线程的线程信息，其中有指定的最大元素数量和同步信息的堆栈跟踪。
  **ThreadInfo[] getThreadInfo(long[], boolean, boolean, int):** 返回每个线程的线程信息，这些线程的标识位于输入数组中，其中有指定的最大元素数量和同步信息的堆栈跟踪。
- **java.lang.reflect.MalformedParameterizedTypeException:** 添加了一个新的构造函数，它以字符串的形式作为参数来获取详细信息。
- **java.net.URLDecoder:
  java.net.URLEncoder:**
  这两个类都有新的重载的解码和编码方法，将 charset 作为附加参数。

- **java.nio.channels.Channels:**
  两个新的静态重载方法，允许使用 Charset 的 newReader（ReadByteChannel，Charset）和newWriter（WriteByteChannel，Charset）。
- **java.nio.file.FileStore:
  long getBlockSize():** 在这个文件存储中返回每个块的字节数。
- **java.time.chrono:** 这个包里有三个类，HijrahEra、MiinguoEra 和 ThaiBuddhistEra ，都有同样的方法。
  **String getDisplayName(TextStyle, Locale):** 这将返回用于识别 era 的文本名称，适合于向用户展示。
- **java.time.format.DateTimeFormatter:
  localizedBy(Locale):** 返回指定格式器的一个副本，其中包含地区、日历、区域、小数和/或时区的本地化值，这将取代该格式器中的值。
- **java.util**: DoubleSummaryStatistics、IntSummaryStatistics 和 LongSummaryStatistics 都有一个新的构造函数，它包含 4 个数值。它使用指定的计数、最小值、最大值和总和构造一个非空实例。
- **java.util.List:
  java.util.Map:
  java.util.Set:** 这些接口中的每一个都增加了一个新的静态方法，copyOf(Collection）。这些函数按照其迭代顺序返回一个不可修改的列表、映射或包含给定集合的元素的集合。
- **java.util.Optional:
  java.util.OptionalDouble:
  java.util.OptionalInt:
  java.util.OptionalLong:** 每一个类都有一个新的方法，orElseThrow() ，它本质上和 get() 一样，也就是说，如果 Optional 有值则返回。否则，将抛出 NoSuchElementException 。
- **java**.**util.Formatter:** 
  **java.util.Scanner:**
  这两个类都有三个新的构造函数，除了其他参数之外，它们都带有一个 charset 参数。

- **java.util.Properties**: 这有一个新的构造函数，它接受一个 int 参数。这将创建一个没有默认值的空属性列表，并且指定初始大小以容纳指定的元素数量，而无需动态调整大小。还有一个新的重载的 replace 方法，接受三个 Object 参数并返回一个布尔值。只有在当前映射到指定值时，才会替换指定键的条目。
- **java**.**SplittableRandom**: 
  **void nextBytes(byte[]):** 用生成的伪随机字节填充一个用户提供的字节数组。
- **java.util.concurrent.FutureTask**: 添加了 toString() 方法，该方法返回一个标识 FutureTask 的字符串，以及它的完成状态。在括号中，状态包含如下字符串中的一个，“Completed Normally” 、“Completed Exceptionally”、 “Cancelled” 或者 “Not completed”。
- **java.util.concurrent.locks.StampedLock**:
  **boolean isLockStamp(long)**: 返回一个标记戳表示是否持有一个锁。
  **boolean isOptimisticReadStamp(long)**: 返回一个标记戳代表是否成功的进行了乐观读（optimistic read）。
  **boolean isReadLockStamp(long):** 返回一个标记戳表示是否持有一个非独占锁（即 read lock ）。
  **boolean isWriteLockStamp(long)**: 返回一个标记戳表示是否持有一个独占锁（即 write lock ）。
- **java**.**jar.JarEntry:**
  **String getRealName()**: 返回这个 JarEntry 的真实名称。如果这个 JarEntry 是一个多版本 jar 文件的入口，它被配置为这样处理，这个方法返回的名字是 JarEntry 所代表的版本条目的入口，而不是 ZipEntry.getName（） 返回的基本条目的路径名。如果 JarEntry 不代表一个多版本 jar 文件的版本化条目或者 jar 文件没有被配置为作为一个多版本 jar 文件进行处理，这个方法将返回与 ZipEntry.getName（） 返回的相同名称。
- **java.util.jar.JarFile:**
  **Stream<JarEntry> versionedStream()**: 返回 jar 文件中指定版本的入口对应 Stream 。与 JarEntry 的 getRealName 方法类似，这与多版本 jar 文件有关。
- **java.util.spi.LocaleNameProvider:
  getDisplayUnicodeExtensionKey(String, Locale):** 为给定的 Unicode 扩展键返回一个本地化名称。
  **getDisplayUnicodeExtensionType(String, String, Locale):** 为给定的 Unicode 扩展键返回一个本地化名称。
- **java.util.stream.Collectors:
  toUnmodifiableList():
  toUnmodifiableSet():
  toUnmodifiableMap(Function, Function):** 
  **toUnmodifiableMap(Function, Function, BinaryOperator):** 这四个新方法都返回 Collectors ，将输入元素聚集到适当的不可修改的集合中。

- **java.lang.model.SourceVersion**: 现在有了一个字段，它代表了 JDK 10 的版本。
- **java.lang.model.util.TypeKindVisitor6**:
  **javax.lang.model.util.TypeKindVisitor9:**
  （我必须承认，我从来没听说过这些类）
  **R visitNoTypeAsModule(NoType, P)**: 访问一个 MODULE 的 pseudo-type 。我不确定为什么只有这两个类得到这个方法，因为还有 Visitor7 和 Visitor8 变量。
- **javax.remote.management.rmi.RMIConnectorServer**:
  这个类已经添加了两个字段： CREDENTIALS_FILTER_PATTERN 和 SERIAL_FILTER_PATTERN 。
- **javax**.**ButtonModel**：看，Swing 还在更新！
  **ButtonGroup getGroup():** 返回按钮所属的组。通常用于单选按钮，它们在组中是互斥的。
- **javax**.**plaf.basic.BasicMenuUI**:
  **Dimension getMinimumSize(JComponent):** 返回指定组件适合观感的最小大小。

### JVM 规范改动

这些改动相当小：

- 4.6节：类文件格式（第99页）。在方法访问标志方面有小的改动。
- 4.7节：模块属性（第169页）。如果模块不是 java.base ，则 JDK 10 不再允许设置 ACC_TRANSITIVE 或 ACC_STATIC_PHASE 。
- 4.10节：类文件的校验（第252页）。dup2 指令已改变了 typesafe form 1 的定义，颠倒了 canSafleyPushList 一节中类型的顺序（你需要仔细查看才能发现它）。
- 5.2节：Java 虚拟机启动（第350页）。该描述添加了在创建初始类或接口时可使用用户定义的类加载器（ bootstrap 类加载器除外）。

### 对 Java 语言规范的更改

这里还有一些更改，但主要是为了支持局部变量类型推断。

- 第3.8节：标识符（第23页）。在忽略了可忽略的字符之后，标识符的等价性现在被考虑了。这似乎是合乎逻辑的。
  （第24页）一个新的 Token，TypeIdentifier，它支持对局部变量类型推断的新用法，而 var 的使用不是关键字，而是一个具有特殊含义的标识符，作为局部变量声明的类型。
- 第4.10.5节：类型预测（第76页）。这是一个相当复杂的部分，它涉及到捕获变量、嵌套类以及如何使用局部变量类型推断。我建议你阅读规范中的这一部分，而不是试图解释它。
- 第6.1节：声明（第134页）。一个反映使用 TypeIdentifier 来支持局部变量类型的推断的小改动。
- 第6.5节：确定名字的含义（第153页，第158页和第159页）。根据类型标识符的使用而更改类类型。
- 第6.5.4.1:简单的 PackageOrTypeNames（第160页）
- 第6.5.4.2节：合规的 PackageOrTypeNames（第160页）。这两种方式都与使用 TypeIdentifier 有细微的变化。
- 第7.5.3:单静态导入声明（第191页）。这改变了导入具有相同名称的静态类型的规则。除非类型是相同的，否则这将成为一个错误，在这种情况下，重复被忽略。
- 第7.7.1:依赖（第198页）。如果你明确声明一个模块需要 java.base ，那在必要的关键字之后，你就不能再使用修饰符（例如静态）了。
- 第8部分：正式参数（第244页）。接收者参数可能只出现在一个实例方法的 formalparameters 列表，或者是一个内部类的构造函数中，其中内部类没有在静态上下文中声明。
- 第9.7.4节：注释可能出现的地方（第335页）。有一个与局部变量类型推断相关的变更。
- 第14.4部分：局部变量声明语句（第433页）。实现局部变量类型推断所需的大量更改。
- 第14节：增强的 for 语句（第455页）。这个结构已经更新，包括对局部变量类型推断的支持。
- 第14.20.3节:try-with-resources（474页）。这个结构已经更新，包括对局部变量类型推断的支持。

最后，第 19 章有多处语法更新，反映了应更多使用 TypeIdentifier 类型标识符，而不仅仅是 Identifier 标识符，以支持局部变量类型推断。 
