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

