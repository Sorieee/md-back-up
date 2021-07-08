---
title: Basic IO学习笔记
date: 2020-10-23 15:06:08
tags: [Java, I/O]

---

https://docs.oracle.com/javase/tutorial/essential/io/index.html

# 概览

该课程包括基础I/O的Java平台类。首先关注I/O Streams, 其次关注序列化, 最后这个课程还会介绍文件I/O以及文件系统操作, 包括随机访问文件。

# I/O Streams

一个I/O Stream 代表一个输入源或输出源。stream可以代表多个不同的来源或输出, 包括文件, 设备, 其他程序, 内存数组。

Stream也支持很多数据类型, 包括简单字节, 基本数据结构, 字符和对象等。一些Stream只是简单的传递数据, 另外一些有效地可以控制和传输这些数据。

不论他们内部如何工作, 所有的stream都是基于一个简单的模型: stream就是序列化的数据。

读取信息到程序

![](https://pic.imgdb.cn/item/5f924e3a1cd1bbb86be74d17.jpg)

​	

从程序写入数据

![](https://pic.imgdb.cn/item/5f924e971cd1bbb86be75da6.jpg)

在这节课中, 我们将看到stream可以处理所有的类型, 从基本类型到类实例都可以。

## Byte Streams

​	使用byte Streams可以执行按字节(8-bit)的输入输出。所有的stream类都继承[`InputStream`](https://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html) and [`OutputStream`](https://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html)。

​	有许多byte stream class, 为了演示byte streams如何工作，我们主要把中心放在[`FileInputStream`](https://docs.oracle.com/javase/8/docs/api/java/io/FileInputStream.html) and [`FileOutputStream`](https://docs.oracle.com/javase/8/docs/api/java/io/FileOutputStream.html)上。其他类型的byte stream也是相同的方式使用;主要是构造时不太一样。

### Using Byte Streams

​	

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class CopyBytes {
    public static void main(String[] args) throws IOException {
        System.out.println(System.getProperty("user.dir"));
        FileInputStream in = null;
        FileOutputStream out = null;

        try {
            in = new FileInputStream("src/io/bytestream/xanadu.txt");
            out = new FileOutputStream("src/io/bytestream/outagain.txt");
            int c;

            while ((c = in.read()) != -1) {
                out.write(c);
            }
        } finally {
            if (in != null) {
                in.close();
            }
            if (out != null) {
                out.close();
            }
        }
    }
}
```

​	CopyBytes大部分时间都在读取inputStream和写入到output stream。每次一个字节。

![](https://pic.imgdb.cn/item/5f9291aa1cd1bbb86bf4a3b7.jpg)

### Always Close Streams

​	不再需要流时，关闭它非常重要。CopyBytes在finally中保证关闭了两个流，就算有错误发生。这个做法有助于避免严重的资源泄漏。

​	第一个可能的错误是不能打开其中一个或者两个文件。当这种情况发生时，文件不会从它初始null值改变(不太明白 TBD)，这也是为什么要保证每个stream都要调用close方法。

### When Not to Use Byte Streams

​	CopyBytes看上去是一个正常的程序，但是实际上它代表了一种low-level的I/O，应该避免这样写程序。因为`xanadu.txt`包含了字符数据，最好的方法是使用[character streams](https://docs.oracle.com/javase/tutorial/essential/io/charstreams.html)。

## Character Streams

​	Java平台使用Unicode存储字符。Character stream自动将此内部格式与本地字符集进行转换。在西方地区，本地字符集主要是ASCII的8-bit超集。

​	在大部分应用中， character streams不会比用byte streams做I/O更复杂。使用Character streams可以很好的适配国际化。如果目前不关心国际化，可以使用它，避免字符集问题。如果关系国家化。See the [Internationalization](https://docs.oracle.com/javase/tutorial/i18n/index.html) trail for more information。

### 使用Character Streams

	所有的Character Streams都继承Reader和Writer。如果是文件I/O，那么就是：[`FileReader`](https://docs.oracle.com/javase/8/docs/api/java/io/FileReader.html) and [`FileWriter`](https://docs.oracle.com/javase/8/docs/api/java/io/FileWriter.html)。

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class CopyCharacters {
    public static void main(String[] args) throws IOException {

        FileReader inputStream = null;
        FileWriter outputStream = null;

        try {
            inputStream = new FileReader("xanadu.txt");
            outputStream = new FileWriter("characteroutput.txt");

            int c;
            while ((c = inputStream.read()) != -1) {
                outputStream.write(c);
            }
        } finally {
            if (inputStream != null) {
                inputStream.close();
            }
            if (outputStream != null) {
                outputStream.close();
            }
        }
    }
}
```

最大的不同就是一个用 `FileReader` and `FileWriter` 另外一个用`FileInputStream` and `FileOutputStream`。注意，两者都使用`int`变量进行读写。然而，在`CopyCharacters`中存的是16位字符，在 `CopyBytes`中存储的是8位的byte数据。

#### 使用Byte Streams的Character Streams

Characters streams经常是由byte streams包装而来。character stream使用 byte stream去进行实际的物理I/O，然后character stream 处理字符和字节的转换工作。比如`FileReader`使用`FileInputStream`, `FileWriter`使用`FileOutputStream`。

存在两个多用于的 byte-2-character的"桥"stream:`InputStreamReader`和`OutputStreamWriter`。如果没有需要的character stream class，可以用他们来创建我们自己的character stream。可以在 [sockets lesson](https://docs.oracle.com/javase/tutorial/networking/sockets/readingWriting.html) in the [networking trail](https://docs.oracle.com/javase/tutorial/networking/index.html)中查看如何通过byte stream创建 character stream。

### 面向行的I/O

​	在面向行的IO中我们使用[`BufferedReader`](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedReader.html) and [`PrintWriter`](https://docs.oracle.com/javase/8/docs/api/java/io/PrintWriter.html)两个类。

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.BufferedReader;
import java.io.PrintWriter;
import java.io.IOException;

public class CopyLines {
    public static void main(String[] args) throws IOException {

        BufferedReader inputStream = null;
        PrintWriter outputStream = null;

        try {
            inputStream = new BufferedReader(new FileReader("xanadu.txt"));
            outputStream = new PrintWriter(new FileWriter("characteroutput.txt"));

            String l;
            while ((l = inputStream.readLine()) != null) {
                outputStream.println(l);
            }
        } finally {
            if (inputStream != null) {
                inputStream.close();
            }
            if (outputStream != null) {
                outputStream.close();
            }
        }
    }
}
```

## Buffered Streams

​	到目前为止，大多数样例都是unbuffered I/O。这意味着每次读写都要底层系统直接处理。这使程序非常低效，因为每次都要进行磁盘访问, 网络活动以及一些代价不菲的操作。

​	为了减少这种代价，实现了 buffered I/O streams。它从一块叫做缓冲区的内存读取数据， 当缓冲区为空时，本地输入API才会被调用。相似的，buffer out streams写入数据到缓冲区，只有当缓冲区满了才会调用本地输出API。

​	可以用一个包装器将一个unbuffered stream转换成一个buffered stream。

```java
inputStream = new BufferedReader(new FileReader("xanadu.txt"));
outputStream = new BufferedWriter(new FileWriter("characteroutput.txt"));
```

​	有四个buffered stream被用于包装unbuffered stream：

* [`BufferedInputStream`](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedInputStream.html) and [`BufferedOutputStream`](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedOutputStream.html) create buffered byte streams
* [`BufferedReader`](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedReader.html) and [`BufferedWriter`](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedWriter.html) create buffered character streams

### Flushing Buffered Streams

​	通常来说, 不等缓冲区填满, 在一个临界点写出缓冲区。叫做刷新缓冲区。

​	一些buffered output class支持自动flush, 由一个可选的构造器参数指定。当authflush启用时，某些key events会导致缓冲区被刷新。比如`PrintWriter`会每次调用`println`或`format`时刷新缓冲区。See [Formatting](https://docs.oracle.com/javase/tutorial/essential/io/formatting.html) for more on these methods.

​	也可以手动调用`flush`方法。

## Scanning And Formatting

​	I/O变成通常涉及到格式化数据。scanner API将输入数据分割成独立的。formatting API将数据组装成人类可阅读的漂亮的格式。

### Scanning 

​	Scanner通常在将格式化输入分解成独立数据方面比较有用

#### Beaking Input into Tokens

​	默认情况下，scanner根据空白符分割。

```java
import java.io.*;
import java.util.Scanner;

public class ScanXan {
    public static void main(String[] args) throws IOException {

        Scanner s = null;

        try {
            s = new Scanner(new BufferedReader(new FileReader("xanadu.txt")));

            while (s.hasNext()) {
                System.out.println(s.next());
            }
        } finally {
            if (s != null) {
                s.close();
            }
        }
    }
}
```

​	注意它在使用完毕有调用了close方法。虽然它不是一个流，但是需要关闭它以表明处理完它底层的流。

​	如果想使用其他分割符。

```
s.useDelimiter(",\\s*");
```

#### Translating Individual Tokens

​	Scanner也支持所有java基础类型(除了char)以及BigInteger和BigDecimal。当然，也可以处理数字的千位分隔符。因此，在US locale下，Scanner可以正确读取"32,767"到Integer中去。

```java
import java.io.FileReader;
import java.io.BufferedReader;
import java.io.IOException;
import java.util.Scanner;
import java.util.Locale;

public class ScanSum {
    public static void main(String[] args) throws IOException {

        Scanner s = null;
        double sum = 0;

        try {
            s = new Scanner(new BufferedReader(new FileReader("usnumbers.txt")));
            s.useLocale(Locale.US);

            while (s.hasNext()) {
                if (s.hasNextDouble()) {
                    sum += s.nextDouble();
                } else {
                    s.next();
                }   
            }
        } finally {
            s.close();
        }

        System.out.println(sum);
    }
}
```

nsnumbers.txt内容如下：

```
8.5
32,767
3.14159
1,000,000.1
```

输出是"1032778.74159"

##  Formatting

​	实现格式化的流对象要么是`PrintWriter`要么是`PrintStream`。

> 唯一可能需要`PrintStream`的是`System.out`和`System.err`,当你需要创建格式化输出流，使用`PrintWritter`。

- `print` and `println` format individual values in a standard way.
- `format` formats almost any number of values based on a format string, with many options for precise formatting.

#### The `print` and `println` Methods

```java
public class Root {
    public static void main(String[] args) {
        int i = 2;
        double r = Math.sqrt(i);
        
        System.out.print("The square root of ");
        System.out.print(i);
        System.out.print(" is ");
        System.out.print(r);
        System.out.println(".");

        i = 5;
        r = Math.sqrt(i);
        System.out.println("The square root of " + i + " is " + r + ".");
    }
}
```

#### The `format` Method

```java
public class Root2 {
    public static void main(String[] args) {
        int i = 2;
        double r = Math.sqrt(i);
        
        System.out.format("The square root of %d is %f.%n", i, r);
    }
}
//The square root of 2 is 1.414214.
```

- `d` formats an integer value as a decimal value.
- `f` formats a floating point value as a decimal value.
- `n` 输出平台特定的换行符。

- `x` formats an integer as a hexadecimal value.
- `s` formats any value as a string.
- `tB` formats an integer as a locale-specific month name.

> 除%%和%n外，所有格式说明符都必须与参数匹配。如果没有，则抛出异常。在Java中吗, \n始终生成换行符。除非您特别需要换行符，否则不要使用\n。若要为本地平台获取正确的行分隔符，请使用%n

还有一些其他规则

![](https://pic.imgdb.cn/item/5f9fb4361cd1bbb86b6afd47.jpg)



```java
public class Format {
    public static void main(String[] args) {
        System.out.format("%f, %1$+015.3f %,d %-10f %n", 34444.3232323, 3232323, 123.234);
    }
}
34444.323232, +0000034444.323 3,232,323 123.234000 
```

* **Precision**: 小数精度，如果有必要，该值会被右阶段。

* **Width**: 格式化的最小width。如果有必要会填充。默认会左填充空格

* **Flags**： +指定数字的格式应始终使用符号(正数+，负数-)。0代表用0填充。

  -代表用右填充。如果是逗号(代表按千分隔符分割)。

  -+可以一起用, +-不能。

  -0也不能是0

* **Argument Index**：允许显示的指定匹配的位置。也可以使用<代表上一个匹配。比如`System.out.format("%f, %<+020.10f %n", Math.PI);`

## I/O from the Command Line

### Standard Streams

​	标准输出输出默认从键盘读入，写出到展示上，也支持文件和程序间的IO。但是这个特性是从命令行解释器控制的而不是程序。

​	Java支持3种标准输入输出。

> *Standard Input*, accessed through `System.in`; *Standard Output*, accessed through `System.out`; and *Standard Error*, accessed through `System.err`.

​	这个对象是自定义的，不需要打开。Standard Output和Standard Error都是输出。单独的错误输出允许用户将常规输出转移到一个文件中，并且仍然能够读取错误消息。

​	你可能希望标准流是character streams，但是由于历史原因，他们是byte streams。System.out和System.err都被定义为PrintStream对象。尽管严格意义来说是byte stream，但是Print Stream 利用内部字符流对象来模拟字符流的许多特性。

​	相比之下, `System.in`是一个没有任何Character stream特性的byte stream。为了想character stream一样使用标准输入，可以用InputStreamReader包装System.in。

```java
InputStreamReader cin = new InputStreamReader(System.in);
```

### The Console

​	标准流一个更高级的替代方案是控制台。是一个预定义好的`Console`类型的具有大多数标准流提供特性的对象。Console对安全密码输入特别有用。它提供了character stream作为输入输出流(通过reader和writer方法)。

​	必须使用`Stystem.console()`尝试获取Console对象。如果可以获取，会返回这个对象，否则会返回NULL。如果是NULL，代表Console操作不被允许。要么是因为操作系统不支持它们，要么是因为程序是在非交互环境中启动的。

​	Console对象通过`readPassword`方法支持安全密码输入。这个方法有两种方式支持安全密码输入。第一种是suppresses echoing， 密码不会在用户屏幕上显示出来。第二种，`readPassword`会返回一个character array， 而不是`String`, 所以密码会被覆盖，并且一旦密码不再需要，会马上从内存中删除掉。

​	

```java
import java.io.Console;
import java.util.Arrays;
import java.io.IOException;

public class Password {
    
    public static void main (String args[]) throws IOException {

        Console c = System.console();
        if (c == null) {
            System.err.println("No console.");
            System.exit(1);
        }

        String login = c.readLine("Enter your login: ");
        char [] oldPassword = c.readPassword("Enter your old password: ");

        if (verify(login, oldPassword)) {
            boolean noMatch;
            do {
                char [] newPassword1 = c.readPassword("Enter your new password: ");
                char [] newPassword2 = c.readPassword("Enter new password again: ");
                noMatch = ! Arrays.equals(newPassword1, newPassword2);
                if (noMatch) {
                    c.format("Passwords don't match. Try again.%n");
                } else {
                    change(login, newPassword1);
                    c.format("Password for %s changed.%n", login);
                }
                Arrays.fill(newPassword1, ' ');
                Arrays.fill(newPassword2, ' ');
            } while (noMatch);
        }

        Arrays.fill(oldPassword, ' ');
    }
    
    // Dummy change method.
    static boolean verify(String login, char[] password) {
        // This method always returns
        // true in this example.
        // Modify this method to verify
        // password according to your rules.
        return true;
    }

    // Dummy change method.
    static void change(String login, char[] password) {
        // Modify this method to change
        // password according to your rules.
    }
}
```

## DataStream

​	Data Stream 支持二进制的基础类型的I/O(boolean, char, byte, short, int, long, float, double)以及String。所有data stream 要么实现 `DataInput`接口, 要么实现了DataOutput接口。本节重点介绍这些接口最广泛使用的实现，`DataInputStream`和`DataOutputStream`。

​	DataStreams示例通过写出一组数据记录，然后再次读取它们来演示数据流。

​	每条记录由3个关联的值组成。

| Order in record | Data type | Data description | Output Method                  | Input Method                 | Sample Value     |
| --------------- | --------- | ---------------- | ------------------------------ | ---------------------------- | ---------------- |
| 1               | `double`  | Item price       | `DataOutputStream.writeDouble` | `DataInputStream.readDouble` | `19.99`          |
| 2               | `int`     | Unit count       | `DataOutputStream.writeInt`    | `DataInputStream.readInt`    | `12`             |
| 3               | `String`  | Item description | `DataOutputStream.writeUTF`    | `DataInputStream.readUTF`    | `"Java T-Shirt"` |

第一步，程序定义了一些包含data file名称以及要写入的数据。

```java
static final String dataFile = "invoicedata";

static final double[] prices = { 19.99, 9.99, 15.99, 3.99, 4.99 };
static final int[] units = { 12, 8, 13, 29, 50 };
static final String[] descs = {
    "Java T-shirt",
    "Java Mug",
    "Duke Juggling Dolls",
    "Java Pin",
    "Java Key Chain"
};
```

DataStream 打开输出流，由于DataOutputStream只能创建为现有字节流对象的包装器，因此DataStream提供缓冲文件输出字节流。

```java
out = new DataOutputStream(new BufferedOutputStream(
              new FileOutputStream(dataFile)));
```

DataStream输出记录并且关闭输出流。

```java
for (int i = 0; i < prices.length; i ++) {
    out.writeDouble(prices[i]);
    out.writeInt(units[i]);
    out.writeUTF(descs[i]);
}
```

writeUTF方法以UTF-8的修改形式写出字符串值。这是一种可变宽度字符编码，普通的西方字符只需要一个字节。

现在DataStream要把数据重新读回来。第一步必须要提供一个inputStream。以及装入输入数据的变量。

```java
in = new DataInputStream(new
            BufferedInputStream(new FileInputStream(dataFile)));

double price;
int unit;
String desc;
double total = 0.0;
```

​	注意DataStream捕捉EOFException来结束文件读取，而不是返回一个不合法的值。DataInput方法的所有实现都使用EOFException而不是返回值。

​	还请注意，数据流中的每个专门写入都与相应的专用读取完全匹配。程序员要确保输出类型和输入类型以这种方式匹配：输入流由简单的二进制数据组成，没有任何内容指示单个值的类型或它们在流中的起始位置。

​	DataStreams使用了一个非常糟糕的编程技巧：使用浮点数表示金额。一般来说，浮点数不利于存储精确值。对于十进制小数尤其不利，因为普通值（如0.1）没有二进制表示。

​	应该使用[`java.math.BigDecimal`](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html)。但是它没法在DataStream中使用。但是它可以在object stream中使用。下一节会介绍。

## Object Streams	

​	data Stream只支持基础类型的I/O, object stream 支持对象的I/O。大多数（但不是所有）标准类都支持对象的序列化。他们都实现了接口`Serializable`。

​	object stream类是`ObjectInputStream`和`ObjectOutputStream`。他们实现了`ObjectInput`和`ObjectOutput`接口。它们是`DataInput`和`DataOutput`的子接口。这意味着所有Data Stream包含的基础类型也在object stream中实现了。所以Object Stream 既可以处理基础类型也可以处理对象类型。exmaple的简单描述如下。ObjectStream创建一个和DataStream相似的程序。但是有两点不同，第一,prices使用`BigDecimal`对象，第二点，`Calander`对象被写入data file中，表示发票日期。

​	如果readObject不能返回期望的object类型，尝试将其强制转换为正确的类型可能会引发ClassNotFoundException。在例子中，它不会发生，所以不需要捕获异常。相反，我们通过在主方法的throws子句中添加ClassNotFoundException来通知编译器我们已经注意到了这个问题。

#### Output and Input of Complex Objects

​	writeObject和readObject方法使用简单，但它们包含一些非常复杂的对象管理逻辑。这对于Calendar这样的类并不重要，因为Calendar只是封装了原始值。但是许多对象引用了其他对象。如果readObject从流中重组一个对象，它必须能够重建原始对象引用的所有对象。额外的对象也可能有他们自己的引用，并因此类推。在这种情况下，`writeObject`遍历对象引用的整个网络并将该w网络中的所有对象写入流。因此，对writeObject的一次调用会导致大量对象被写入流中。

​	如下图所示，writeObject被调用于写入对象a。a包含b和c，b包含d和e。调用writeobject(a)不只是写入a，还有所有重组a需要的对象。当a被`readObject`方法读回时，其他四个对象也会被都会。

![](https://pic.imgdb.cn/item/5f9acf8d1cd1bbb86be68dfa.jpg)

​	如果同一流中的两个对象都包含对单个对象的引用，会发生什么情况？是否读回时, 他们是否是同一个对象？答案是"yes"。一个流只会对同一个对象只会包含一个副本，尽管可能有好些对它的引用。因此，你可以将一个对象写入流两次。

```
Object ob = new Object();
out.writeObject(ob);
out.writeObject(ob);
```

然后将它们读回。

```java
Object ob1 = in.readObject();
Object ob2 = in.readObject();
```

结果的两个变量ob1和ob2,它们都是同一个对象的引用。

但是，如果一个对象被写入两个不同的流，那么它就被有效地复制了——一个将两个流读回的程序将看到两个不同的对象。

# File I/O(Featuring NIO.2)

> 注意：本教程反映了jdk7版本中引入的文件I/O机制。

​	`java.nio.file`包以及其关联包`java.nio.file.attribute`, 提供了访问默认文件系统的file I/O的全部支持。尽管这个API有很多class，但是你可以只关注一些切入点。这些API会让你觉得顾名思义，非常易用。

​	本教程首先提出一个问题，什么是路径？然后，介绍包的主要入口点Path类。解释了Path类中与语法操作相关的方法。然后，本教程继续讨论包中的另一个主要类，Files类，它包含处理文件操作的方法。首先，介绍了许多文件操作中常见的一些概念。本教程随后介绍了检查、删除、复制和移动文件的方法。

​	教程展示了在metadata怎么管理，然后再转到文件I/O和目录I/O。对随机存取文件进行了解释，并检查了符号和硬链接特有的问题。

​	接下来，我们将介绍一些非常强大但更高级的主题。首先，演示了递归遍历文件树的能力，然后介绍了如何使用通配符搜索文件。接下来，将解释和演示如何监视目录中的更改。然后，对不适合其他地方的方法给予一些关注。

​	最后，如果您有在JavaSE7发行版之前编写的文件I/O代码，则会有一个从旧API到新API的映射，以及有关文件.toPath方法，用于希望在不重写现有代码的情况下利用新API的开发人员。

## What Is a Path? (And Other File System Facts)

​	文件系统在某种形式的介质（通常是一个或多个硬盘驱动器）上存储和组织文件，以便于检索。目前发多数文件系统都按树结构(或层级结构)存储文件。在树顶端是一个到多个根节点。在根节点下，是文件或者目录。每个目录包含其他文件和子目录，如此递归,可能达到无限的深度。

### What Is a Path

​	下图展示了一个包含在单个根节点的文件树样例。Windows支持多根节点。Solaris系统支持单个节点。

![](https://pic.imgdb.cn/item/5f9b7e491cd1bbb86b0dc28a.jpg)

文件通过文件系统的路径标识，从根节点开始。例如，在Solaris操作系统中，上图中的statusReport文件由以下符号描述：

```
/home/sally/statusReport
```

在Windows系统中：

```
C:\home\sally\statusReport
```

两者文件分隔符不一样。

forward slash (`/`),和 backslash slash (`\`)。

### Relative or Absolute?

​	路径可以是相对的，也可以是绝对的。绝对路径始终包含根元素和定位文件所需的完整目录列表。例如，/home/sally/statusReport是一个绝对路径。定位文件所需的所有信息都包含在路径字符串中。

​	一个相对路径需要与另一个路径相结合才能访问文件。例如，joe/foo是相对路径。如果没有更多信息，程序就无法在文件系统中可靠地定位joe/foo目录。

### Symbolic Links

​	文件系统对象通常是目录或文件。一些文件系统支持symbolic links的概念。它也称为*symlink* or  *soft link*。

​	Symbolic Links是一个指向其他文件的的连接。大多数情况下，符号链接对应用程序是透明的。符号链接上的操作将自动重定向到链接的目标。例外情况是删除或重命名符号链接，在这种情况下，链接本身被删除，或重命名，而不是链接的目标。

​	

![](https://pic.imgdb.cn/item/5f9b854c1cd1bbb86b0f5995.jpg)

​	符号链接通常对用户是透明的。读取或写入符号链接与读取或写入任何其他文件或目录相同。

​	词组解析链接意味着用文件系统中的实际位置替换符号链接。

​	在实际场景中，大多数文件系统都会自由地使用符号链接。偶尔，不小心创建的符号链接会导致循环引用。当链接的目标指向原始链接时，会发生循环引用。循环引用可能是间接的：目录a指向目录b，目录b指向目录c，其中包含指向目录a的子目录。当程序递归遍历目录结构时，循环引用可能会造成严重破坏。但是，这个场景已经考虑到了，不会导致程序无限循环。

## The Path Class

​	在JavaSE7发行版中引入的Path类是java.nio.file文件包中的关键切入点。如果你的程序想使用文件I/O，你应该想知道这个类强大的特性。

> **Version Note:** If you have pre-JDK7 code that uses `java.io.File`, you can still take advantage of the `Path` class functionality by using the [`File.toPath`](https://docs.oracle.com/javase/8/docs/api/java/io/File.html#toPath--) method. See [Legacy File I/O Code](https://docs.oracle.com/javase/tutorial/essential/io/legacy.html) for more information.

​	顾名思义，Path类是文件系统中路径的程序化表示。一个`Path`对象包含用于构造路径的文件名和目录列表， 用于检查、定位和操纵文件。

​	路径实例反映了底层平台。In the Solaris OS, a `Path` uses the Solaris syntax (`/home/joe/foo`) and in Microsoft Windows, a `Path` uses the Windows syntax (`C:\home\joe\foo`)。

​	与路径对应的文件或目录可能不存在。您可以创建一个Path实例并以各种方式操作它：可以附加到它，提取它的片段，将其与另一个路径进行比较。在适当的时候，可以使用Files类中的方法来检查路径对应的文件是否存在、创建文件、打开文件、删除文件、更改其权限等。

### Path Operation

#### Creating A Path

可以按如下方法创建Path类实例。

```java
Path p1 = Paths.get("/tmp/foo");
Path p2 = Paths.get(args[0]);
Path p3 = Paths.get(URI.create("file:///Users/joe/FileTest.java"));
```

这个路径.get方法是以下代码的简写

```java
Path p4 = FileSystems.getDefault().getPath("/users/sally");
```

The following example creates `/u/joe/logs/foo.log` assuming your home directory is `/u/joe`, or `C:\joe\logs\foo.log` if you are on Windows.

```java
Path p5 = Paths.get(System.getProperty("user.home"),"logs", "foo.log");
```

#### Retrieving Information about a Path

​	可以将Path看为是存储名字元素的的序列。目录结构中最高的元素将位于索引0处。目录结构中最低的元素位于索引[n-1]。 方法可用于使用这些索引检索单个元素或路径的子序列

![](https://pic.imgdb.cn/item/5f9baba71cd1bbb86b1a6a60.jpg)

```java
// None of these methods requires that the file corresponding
// to the Path exists.
// Microsoft Windows syntax
Path path = Paths.get("C:\\home\\joe\\foo");

// Solaris syntax
Path path = Paths.get("/home/joe/foo");

System.out.format("toString: %s%n", path.toString());
System.out.format("getFileName: %s%n", path.getFileName());
System.out.format("getName(0): %s%n", path.getName(0));
System.out.format("getNameCount: %d%n", path.getNameCount());
System.out.format("subpath(0,2): %s%n", path.subpath(0,2));
System.out.format("getParent: %s%n", path.getParent());
System.out.format("getRoot: %s%n", path.getRoot());
```

| Method Invoked | Returns in the Solaris OS | Returns in Microsoft Windows | Comment                                                      |
| -------------- | ------------------------- | ---------------------------- | ------------------------------------------------------------ |
| `toString`     | `/home/joe/foo`           | `C:\home\joe\foo`            | Returns the string representation of the `Path`. If the path was created using `Filesystems.getDefault().getPath(String)` or `Paths.get` (the latter is a convenience method for `getPath`), the method performs minor syntactic cleanup. For example, in a UNIX operating system, it will correct the input string `//home/joe/foo` to `/home/joe/foo`. |
| `getFileName`  | `foo`                     | `foo`                        | Returns the file name or the last element of the sequence of name elements. |
| `getName(0)`   | `home`                    | `home`                       | Returns the path element corresponding to the specified index. The 0th element is the path element closest to the root. |
| `getNameCount` | `3`                       | `3`                          | Returns the number of elements in the path.                  |
| `subpath(0,2)` | `home/joe`                | `home\joe`                   | Returns the subsequence of the `Path` (not including a root element) as specified by the beginning and ending indexes. |
| `getParent`    | `/home/joe`               | `\home\joe`                  | Returns the path of the parent directory.                    |
| `getRoot`      | `/`                       | `C:\`                        | Returns the root of the path.                                |

前面的示例显示了绝对路径的输出。在以下示例中，指定了相对路径：

```java
// Solaris syntax
Path path = Paths.get("sally/bar");
or
// Microsoft Windows syntax
Path path = Paths.get("sally\\bar");
```

| Method Invoked | Returns in the Solaris OS | Returns in Microsoft Windows |
| -------------- | ------------------------- | ---------------------------- |
| `toString`     | `sally/bar`               | `sally\bar`                  |
| `getFileName`  | `bar`                     | `bar`                        |
| `getName(0)`   | `sally`                   | `sally`                      |
| `getNameCount` | `2`                       | `2`                          |
| `subpath(0,1)` | `sally`                   | `sally`                      |
| `getParent`    | `sally`                   | `sally`                      |
| `getRoot`      | `null`                    | `null`                       |

#### Removing Redundancies From a Path

很多文件系统使用"."表示当前目录, ".."表示父目录。你有可能向从path去掉"/."。

```
/home/./joe/foo
/home/sally/../joe/foo
```

`normalize`方法是删除所有冗余元素，包括任何“.”或“directory/.”出现。前面的两个示例都规范化为/home/joe/foo。

​	需要注意的是，normalize在清理路径时不会检查文件系统。这是一个纯粹的句法操作。在第二个示例中，如果sally是一个符号链接，则删除sally/。。可能导致路径不再定位目标文件。

​	要在确保结果找到正确文件的同时清理路径，可以使用toRealPath方法。

#### Converting a Path

​	有3种方法convert Path。如果需要将路径转换为可从浏览器打开的字符串：

```java
Path p1 = Paths.get("/home/logfile");
// Result is file:///home/logfile
System.out.format("%s%n", p1.toUri());
```

​	toAbsolutePath方法将一个path 转换成一个absolute path。如果本身已经是absolute了，会返回相同的对象。toAbsolutePath方法转换用户输入并返回在查询时返回有用值的路径。此方法不需要存在该文件

```java
public class FileTest {
    public static void main(String[] args) {

        if (args.length < 1) {
            System.out.println("usage: FileTest file");
            System.exit(-1);
        }

        // Converts the input string to a Path object.
        Path inputPath = Paths.get(args[0]);

        // Converts the input Path
        // to an absolute path.
        // Generally, this means prepending
        // the current working
        // directory.  If this example
        // were called like this:
        //     java FileTest foo
        // the getRoot and getParent methods
        // would return null
        // on the original "inputPath"
        // instance.  Invoking getRoot and
        // getParent on the "fullPath"
        // instance returns expected values.
        Path fullPath = inputPath.toAbsolutePath();
    }
}
```

toRealPath方法返回现有文件的实际路径。这个方法会有有如下几个操作

> - If `true` is passed to this method and the file system supports symbolic links, this method resolves any symbolic links in the path.
> - If the `Path` is relative, it returns an absolute path.
> - If the `Path` contains any redundant elements, it returns a path with those elements removed.

当文件不存在或不可访问时，这个方法会抛出异常。

```java
try {
    Path fp = path.toRealPath();
} catch (NoSuchFileException x) {
    System.err.format("%s: no such" + " file or directory%n", path);
    // Logic for case when file doesn't exist.
} catch (IOException x) {
    System.err.format("%s%n", x);
    // Logic for other sort of file error.
}
```

#### Joining Two Paths

```java
// Solaris
Path p1 = Paths.get("/home/joe/foo");
// Result is /home/joe/foo/bar
System.out.format("%s%n", p1.resolve("bar"));

or

// Microsoft Windows
Path p1 = Paths.get("C:\\home\\joe\\foo");
// Result is C:\home\joe\foo\bar
System.out.format("%s%n", p1.resolve("bar"));
```

将绝对路径传给resolve会返回传入的路径：

```java
// Result is /home/joe
Paths.get("foo").resolve("/home/joe");
```

#### Creating a Path Between Two Paths

​	在编写文件I/O代码时，一个常见的需求是能够构造从文件系统中的一个位置到另一个位置的路径。你可以使用`relativize`方法。

```java
Path p1 = Paths.get("joe");
Path p2 = Paths.get("sally");
// Result is ../sally
Path p1_to_p2 = p1.relativize(p2);
// Result is ../joe
Path p2_to_p1 = p2.relativize(p1);

Path p1 = Paths.get("home");
Path p3 = Paths.get("home/sally/bar");
// Result is sally/bar
Path p1_to_p3 = p1.relativize(p3);
// Result is ../..
Path p3_to_p1 = p3.relativize(p1);
```

#### Comparing Two Paths

```java
Path path = ...;
Path otherPath = ...;
Path beginning = Paths.get("/home");
Path ending = Paths.get("foo");

if (path.equals(otherPath)) {
    // equality logic here
} else if (path.startsWith(beginning)) {
    // path begins with "/home"
} else if (path.endsWith(ending)) {
    // path ends with "foo"
}
```

`Path`也实现了迭代器：

```kjava
Path path = ...;
for (Path name: path) {
    System.out.println(name);
}
```

Path类还实现了`Comparable`的接口。可以使用compareTo来比较路径对象，这对排序很有用。

当你想确认两个Path对象是否定位到相同文件，可以使用isSameFile方法。

## File Operations

Files类是另外一个java.nio.file包的基础点。它提供了丰富的读写以及操作文件和文件夹的静态方法集合。`Files`方法在`Path`对象实例上进行工作。在继续学习其余部分之前，您应该熟悉以下常见概念。

### Releasing System Resources

​	很多resource使用这个API，比如stream和channels，都实现或继承了`java.io.Closeable`接口。可关闭资源的一个要求是，当不再需要时，必须调用close方法来释放资源。忽略关闭资源可能会对应用程序的性能产生负面影响。不过`try-with-resources`语句可以帮你做这件事。

### Cathing Exception

​	在进行I/O时可能会遇到很多异常或者错误。所有访问文件系统的方法都会抛出`IOException`,最好是通过`try-with-resources`语句来处理他们。

```java
Charset charset = Charset.forName("US-ASCII");
String s = ...;
try (BufferedWriter writer = Files.newBufferedWriter(file, charset)) {
    writer.write(s, 0, s.length());
} catch (IOException x) {
    System.err.format("IOException: %s%n", x);
}
```

​	你也可以手动关闭他们。

```java
Charset charset = Charset.forName("US-ASCII");
String s = ...;
BufferedWriter writer = null;
try {
    writer = Files.newBufferedWriter(file, charset);
    writer.write(s, 0, s.length());
} catch (IOException x) {
    System.err.format("IOException: %s%n", x);
} finally {
    if (writer != null) writer.close();
}
```

除了IOException，许多特定的异常都扩展了FileSystemException。这个类有一些有用的方法，它们返回所涉及的文件（getFile）、详细的消息字符串（getMessage）、文件系统操作失败的原因（getReason）和涉及的“其他”文件（如果有）（getOtherFile）。

```java
try (...) {
    ...    
} catch (NoSuchFileException x) {
    System.err.format("%s does not exist\n", x.getFile());
}
```

后续的I/O example不会包含异常处理，但是你自己的代码要始终进行异常处理。

### Var args

当指定标志时，有几个Files方法接受任意数量的参数。比如

```java
Path Files.move(Path, Path, CopyOption...)
```

example如下：

```java
import static java.nio.file.StandardCopyOption.*;

Path source = ...;
Path target = ...;
Files.move(source,
           target,
           REPLACE_EXISTING,
           ATOMIC_MOVE);
```

### Atomic Opertions

一些文件方法（如move）可以在某些文件系统中以原子方式执行某些操作。

### Method Chaining

​	许多文件操作支持方法链。

```java
String value = Charset.defaultCharset().decode(buf).toString();
UserPrincipal group =
    file.getFileSystem().getUserPrincipalLookupService().
         lookupPrincipalByName("me");
```

### What is a Glob

​	您可以使用glob语法来指定模式匹配行为。

​	glob模式被指定为字符串，并与其他字符串（如目录或文件名）相匹配。Glob语法遵循几个简单规则：

* `*`：任意字符
* `**`: 两个星号的工作原理与`*`相似，但跨越了目录边界。此语法通常用于匹配完整路径.
* ?: 匹配任意一个字符
* {sun,moon,stars}: 代表匹配其中一个字符即可。
* 可以使用-, 比如：
  - `[aeiou]` matches any lowercase vowel.
  - `[0-9]` matches any digit.
  - `[A-Z]` matches any uppercase letter.
  - `[a-z,A-Z]` matches any uppercase or lowercase letter.
* 可以使用`\*`,`\?`,`\\`等表示特殊字符。

glob语法功能强大，易于使用。但是，如果它不足以满足您的需要，您还可以使用正则表达式。

### Link Awareness

Files类是“link-aware”。每个Files方法要么检测遇到符号链接时要做什么，要么提供一个选项，使您能够配置遇到符号链接时的行为。

## Checking a File or Directory

您有一个表示文件或目录的路径实例，但该文件是否存在于文件系统中？它可读吗？可写？可执行文件？

### Verifying the Existence of a File or Directory

可以使用 [`exists(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#exists-java.nio.file.Path-java.nio.file.LinkOption...-) and the [`notExists(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#notExists-java.nio.file.Path-java.nio.file.LinkOption...-)来检查文件的存在性。有几种可能

* 文件存在
* 文件不存在
* 未知状态，当程序无法访问该文件时，可能会出现此结果。

如果当exists和notExists都返回了false，那么无法知晓文件是否存在。

### Checking File Accessibility

可以使用[`isReadable(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isReadable-java.nio.file.Path-), [`isWritable(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isWritable-java.nio.file.Path-), and [`isExecutable(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isExecutable-java.nio.file.Path-)方法。

```java
Path file = ...;
boolean isRegularExecutableFile = Files.isRegularFile(file) &
         Files.isReadable(file) & Files.isExecutable(file);
```

> 注意：一旦这些方法完成，就不能保证可以访问该文件。在许多应用程序中，一个常见的安全缺陷是执行检查，然后访问该文件。有关详细信息，请使用您最喜欢的搜索引擎查找`TOCTTOU`

>  TOCTTOU是指计算机系统的资料与权限等状态的检查与使用之间，因为某特定状态在这段时间已改变所产生的[软件漏洞](https://baike.baidu.com/item/软件漏洞/3879396)。TOCTTOU是[竞争危害](https://baike.baidu.com/item/竞争危害) (race hazard) 又名[竞态条件](https://baike.baidu.com/item/竞态条件) (race condition)的一种。

### Checking Whether Two Paths Locate the Same File

```java
Path p1 = ...;
Path p2 = ...;

if (Files.isSameFile(p1, p2)) {
    // Logic when the paths locate the same file
}
```

## Deleting a File or Directory

您可以删除文件、目录或链接。对于符号链接，链接将被删除，而不是链接的目标。对于目录，目录必须为空，否则删除失败。

```java
try {
    Files.delete(path);
} catch (NoSuchFileException x) {
    System.err.format("%s: no such" + " file or directory%n", path);
} catch (DirectoryNotEmptyException x) {
    System.err.format("%s not empty%n", path);
} catch (IOException x) {
    // File permission problems are caught here.
    System.err.println(x);
}
```

The [`deleteIfExists(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#deleteIfExists-java.nio.file.Path-) method also deletes the file, but if the file does not exist, no exception is thrown.

## Copying a File or Directory

​	您可以使用`copy（Path，Path，CopyOption…）`方法复制文件或目录。如果目标文件存在，则复制失败，除非指定了`REPLACE_EXISTING`选项。

​	可以复制目录。但是，不会复制目录中的文件，因此即使原始目录包含文件，新目录也是空的.

​	复制符号链接时，将复制链接的目标。如果要复制链接本身，而不是复制链接的内容，请指定NOFOLLOW_LINKS或REPLACE_EXISTING选项。

​	持以下StandardCopyOption和LinkOption枚举

- `REPLACE_EXISTING` – 即使目标文件已存在，也执行复制。如果目标是符号链接，则复制链接本身（而不是链接的目标）。如果目标是非空目录，则复制失败，并出现FileAlreadyExistsException异常。
- `COPY_ATTRIBUTES` – 将与文件关联的文件属性复制到目标文件。支持的确切文件属性取决于文件系统和平台，但不同平台都支持上次修改时间，并将其复制到目标文件。
- `NOFOLLOW_LINKS` –表示不应遵循符号链接。如果要复制的文件是符号链接，则复制链接（而不是链接的目标）。

除了文件复制之外，Files类还定义了可用于在文件和流之间进行复制的方法。

`copy（InputStream，Path，CopyOptions…）`方法可用于将输入流中的所有字节复制到文件中。`copy（Path，OutputStream）`方法可用于将文件中的所有字节复制到输出流中。

​	 [`Copy`](https://docs.oracle.com/javase/tutorial/essential/io/examples/Copy.java)示例使用Copy和Files.walkFileTree方法来支持递归副本。

## Moving a File or Directory

可以使用 [`move(Path, Path, CopyOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#move-java.nio.file.Path-java.nio.file.Path-java.nio.file.CopyOption...-)来移动文件。如果目标文件存在，则移动将失败，除非指定了REPLACE_EXISTING选项。

​	可以移动空目录。如果目录不为空，则允许在不移动目录内容的情况下移动目录。在UNIX系统上，在同一分区内移动目录通常包括重命名该目录。在这种情况下，即使目录包含文件，此方法也可以工作。

the following `StandardCopyOption` enums are supported:

- `REPLACE_EXISTING` – 即使目标文件已存在，也执行移动。如果目标是符号链接，则会替换符号链接，但它指向的内容不受影响。
- `ATOMIC_MOVE` – 将移动作为原子文件操作执行。如果文件系统不支持原子移动，则会引发异常。通过原子移动，您可以将文件移动到目录中，并保证任何监视该目录的进程都可以访问完整的文件。

```java
import static java.nio.file.StandardCopyOption.*;
...
Files.move(source, target, REPLACE_EXISTING);
```

尽管可以在单个目录上实现move方法，如图所示，但该方法通常与文件树递归机制一起使用.

## Managing Metadata (File and File Store Attributes)

Files包含如下可以用于获取或者设置文件单个属性的方法。

| Methods                                                      | Comment                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`size(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#size-java.nio.file.Path-) | Returns the size of the specified file in bytes.             |
| [`isDirectory(Path, LinkOption)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isDirectory-java.nio.file.Path-java.nio.file.LinkOption...-) | Returns true if the specified `Path` locates a file that is a directory. |
| [`isRegularFile(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isRegularFile-java.nio.file.Path-java.nio.file.LinkOption...-) | Returns true if the specified `Path` locates a file that is a regular file. |
| [`isSymbolicLink(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isSymbolicLink-java.nio.file.Path-) | Returns true if the specified `Path` locates a file that is a symbolic link. |
| [`isHidden(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#isHidden-java.nio.file.Path-) | Returns true if the specified `Path` locates a file that is considered hidden by the file system. |
| [`getLastModifiedTime(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#getLastModifiedTime-java.nio.file.Path-java.nio.file.LinkOption...-) [`setLastModifiedTime(Path, FileTime)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#setLastModifiedTime-java.nio.file.Path-java.nio.file.attribute.FileTime-) | Returns or sets the specified file's last modified time.     |
| [`getOwner(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#getOwner-java.nio.file.Path-java.nio.file.LinkOption...-) [`setOwner(Path, UserPrincipal)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#setOwner-java.nio.file.Path-java.nio.file.attribute.UserPrincipal-) | Returns or sets the owner of the file.                       |
| [`getPosixFilePermissions(Path, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#getPosixFilePermissions-java.nio.file.Path-java.nio.file.LinkOption...-) [`setPosixFilePermissions(Path, Set)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#setPosixFilePermissions-java.nio.file.Path-java.util.Set-) | Returns or sets a file's POSIX file permissions.             |
| [`getAttribute(Path, String, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#getAttribute-java.nio.file.Path-java.lang.String-java.nio.file.LinkOption...-) [`setAttribute(Path, String, Object, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#setAttribute-java.nio.file.Path-java.lang.String-java.lang.Object-java.nio.file.LinkOption...-) | Returns or sets the value of a file attribute.               |

如果一个程序同时需要多个文件属性，那么使用检索单个属性的方法可能效率低下。

| Method                                                       | Comment                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`readAttributes(Path, String, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#readAttributes-java.nio.file.Path-java.lang.String-java.nio.file.LinkOption...-) | Reads a file's attributes as a bulk operation. The `String` parameter identifies the attributes to be read. |
| [`readAttributes(Path, Class, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#readAttributes-java.nio.file.Path-java.lang.Class-java.nio.file.LinkOption...-) | Reads a file's attributes as a bulk operation. The `Class<A>` parameter is the type of attributes requested and the method returns an object of that class. |

​	在展示readAttributes方法的示例之前，应该提到不同的文件系统对于应该跟踪哪些属性有不同的概念。因此，相关的文件属性被分组到视图中。视图映射到特定的文件系统实现，如POSIX或DOS，或映射到一个公共功能，如文件所有权。

- [`BasicFileAttributeView`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/attribute/BasicFileAttributeView.html) – 提供所有文件系统实现都需要支持的基本属性的视图。
- [`DosFileAttributeView`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/attribute/DosFileAttributeView.html) – 使用支持DOS属性的文件系统支持的标准四位扩展基本属性视图
- [`PosixFileAttributeView`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/attribute/PosixFileAttributeView.html) – 使用支持POSIX标准系列（如UNIX）的文件系统支持的属性扩展基本属性视图。这些属性包括文件所有者、组所有者和九个相关的访问权。
- [`FileOwnerAttributeView`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/attribute/FileOwnerAttributeView.html) – 支持任何支持文件所有者概念的文件系统实现。
- [`AclFileAttributeView`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/attribute/AclFileAttributeView.html) – 支持读取或更新文件的访问控制列表（ACL）。支持NFSv4 ACL模型。任何ACL模型，例如Windows ACL模型，都可能支持到NFSv4模型的定义良好的映射
- [`UserDefinedFileAttributeView`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/attribute/UserDefinedFileAttributeView.html) – 支持用户定义的元数据。此视图可以映射到系统支持的任何扩展机制。例如，在Solaris操作系统中，可以使用此视图存储文件的MIME类型。

### Basic File Attributes

```java
Path file = ...;
BasicFileAttributes attr = Files.readAttributes(file, BasicFileAttributes.class);

System.out.println("creationTime: " + attr.creationTime());
System.out.println("lastAccessTime: " + attr.lastAccessTime());
System.out.println("lastModifiedTime: " + attr.lastModifiedTime());

System.out.println("isDirectory: " + attr.isDirectory());
System.out.println("isOther: " + attr.isOther());
System.out.println("isRegularFile: " + attr.isRegularFile());
System.out.println("isSymbolicLink: " + attr.isSymbolicLink());
System.out.println("size: " + attr.size());
```

除了本例中显示的访问器方法外，还有一个fileKey方法，它返回唯一标识文件的对象，如果没有文件密钥，则返回null。

### Setting Time Stamps

```java
Path file = ...;
BasicFileAttributes attr =
    Files.readAttributes(file, BasicFileAttributes.class);
long currentTime = System.currentTimeMillis();
FileTime ft = FileTime.fromMillis(currentTime);
Files.setLastModifiedTime(file, ft);
}
```

### DOS File Attributes

```java
Path file = ...;
try {
    DosFileAttributes attr =
        Files.readAttributes(file, DosFileAttributes.class);
    System.out.println("isReadOnly is " + attr.isReadOnly());
    System.out.println("isHidden is " + attr.isHidden());
    System.out.println("isArchive is " + attr.isArchive());
    System.out.println("isSystem is " + attr.isSystem());
} catch (UnsupportedOperationException x) {
    System.err.println("DOS file" +
        " attributes not supported:" + x);
}
```

However, you can set a DOS attribute using the [`setAttribute(Path, String, Object, LinkOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#setAttribute-java.nio.file.Path-java.lang.String-java.lang.Object-java.nio.file.LinkOption...-)

```java
Path file = ...;
Files.setAttribute(file, "dos:hidden", true);
```

### POSIX File Permissions

```java
Path file = ...;
PosixFileAttributes attr =
    Files.readAttributes(file, PosixFileAttributes.class);
System.out.format("%s %s %s%n",
    attr.owner().getName(),
    attr.group().getName(),
    PosixFilePermissions.toString(attr.permissions()));
```

PosixFilePermissions提供了几个有用的方法：

- The `toString` method, used in the previous code snippet, converts the file permissions to a string (for example, `rw-r--r--`).
- The `fromString` method accepts a string representing the file permissions and constructs a `Set` of file permissions.
- The `asFileAttribute` method accepts a `Set` of file permissions and constructs a file attribute that can be passed to the `Path.createFile` or `Path.createDirectory` method.

```java
Path sourceFile = ...;
Path newFile = ...;
PosixFileAttributes attrs =
    Files.readAttributes(sourceFile, PosixFileAttributes.class);
FileAttribute<Set<PosixFilePermission>> attr =
    PosixFilePermissions.asFileAttribute(attrs.permissions());
Files.createFile(file, attr);
```

 要将文件的权限设置为硬编码字符串表示的值，可以使用以下代码

```java
Path file = ...;
Set<PosixFilePermission> perms =
    PosixFilePermissions.fromString("rw-------");
FileAttribute<Set<PosixFilePermission>> attr =
    PosixFilePermissions.asFileAttribute(perms);
Files.setPosixFilePermissions(file, perms);
```

### Setting a File or Group Owner

要将名称转换为可以存储为文件所有者或组所有者的对象，可以使用UserPrincipalLookupService。

```java
Path file = ...;
UserPrincipal owner = file.GetFileSystem().getUserPrincipalLookupService()
        .lookupPrincipalByName("sally");
Files.setOwner(file, owner);
```

Files类中没有用于设置组所有者的专用方法。但是，直接执行此操作的安全方法是通过POSIX文件属性视图。

```java
Path file = ...;
GroupPrincipal group =
    file.getFileSystem().getUserPrincipalLookupService()
        .lookupPrincipalByGroupName("green");
Files.getFileAttributeView(file, PosixFileAttributeView.class)
     .setGroup(group);
```

### User-Defined File Attributes

​	一些实现将这个概念映射到诸如NTFS替代数据流和ext3和ZFS等文件系统上的扩展属性。大多数实现对值的大小施加限制，例如ext3将大小限制为4KB。

```java
Path file = ...;
UserDefinedFileAttributeView view = Files
    .getFileAttributeView(file, UserDefinedFileAttributeView.class);
view.write("user.mimetype",
           Charset.defaultCharset().encode("text/html");
```

```java
Path file = ...;
UserDefinedFileAttributeView view = Files
.getFileAttributeView(file,UserDefinedFileAttributeView.class);
String name = "user.mimetype";
ByteBuffer buf = ByteBuffer.allocate(view.size(name));
view.read(name, buf);
buf.flip();
String value = Charset.defaultCharset().decode(buf).toString();
```

The [`Xdd`](https://docs.oracle.com/javase/tutorial/essential/io/examples/Xdd.java) example shows how to get, set, and delete a user-defined attribute.

> 在Linux中，您可能需要启用扩展属性才能使用户定义的属性正常工作。如果在尝试访问用户定义的属性视图时收到UnsupportedOperationException，则需要重新装载文件系统。下面的命令为ext3文件系统重新装载具有扩展属性的根分区。如果此命令不适用于您的Linux版本，请参阅文档。
>
> ```
> $ sudo mount -o remount,user_xattr /
> ```
>
> 如果要使更改永久化，请在/etc/fstab中添加一个条目。

### File Store Attributes

您可以使用`FileStore`类来了解有关文件存储的信息，例如有多少可用空间。

以下代码段打印特定文件所在的文件存储区的空间使用情况

```java
Path file = ...;
FileStore store = Files.getFileStore(file);

long total = store.getTotalSpace() / 1024;
long used = (store.getTotalSpace() -
             store.getUnallocatedSpace()) / 1024;
long avail = store.getUsableSpace() / 1024;
```

The [`DiskUsage`](https://docs.oracle.com/javase/tutorial/essential/io/examples/DiskUsage.java) example uses this API to print disk space information for all the stores in the default file system.

## Reading, Writing, and Creating Files

为了帮助理解API，下图按复杂性排列文件I/O方法。

![](https://pic.imgdb.cn/item/5f9e40171cd1bbb86bfb2e4f.jpg)



​	图的最左边是实用程序方法readAllBytes、readAllLines和write方法，它们是为简单的常见情况而设计的。右侧是用于迭代一个或多行文本的方法，例如newBufferedReader、newBufferedWriter、newInputStream和newOutputStream。这些方法可与java.io包互补。右边是处理bytechannel、seekablebytechannes和ByteBuffers的方法，比如newByteChannel方法。最后，在最右边是使用FileChannel处理需要文件锁定或内存映射I/O的高级应用程序的方法。

### The `OpenOptions` Parameter

The following `StandardOpenOptions` enums are supported:

- `WRITE` – 打开文件用于写访问.
- `APPEND` – 将新数据追加到文件末尾。此选项与`WRITE`或`CREATE`选项一起使用。
- `TRUNCATE_EXISTING` – 将文件截断为零字节。此选项与“写入”选项一起使用。
- `CREATE_NEW` – 创建一个新文件，如果文件已存在，则引发异常。
- `CREATE` – 打开文件（如果存在）或创建新文件（如果不存在）。
- `DELETE_ON_CLOSE` – 关闭流时删除文件。此选项对于临时文件很有用。
- `SPARSE` – 提示新创建的文件将是稀疏的。此高级选项在某些文件系统（如NTFS）上很受欢迎，在NTFS中，具有数据“间隙”的大文件可以以更高效的方式存储，而这些空白空间不会占用磁盘空间。
- `SYNC` – 使文件（内容和元数据）与基础存储设备保持同步。
- `DSYNC` – 使文件内容与基础存储设备保持同步。

### Methods for Unbuffered Streams and Interoperable with `java.io` APIs

#### Reading a File by Using Stream I/O

```java
Path file = ...;
try (InputStream in = Files.newInputStream(file);
    BufferedReader reader =
      new BufferedReader(new InputStreamReader(in))) {
    String line = null;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException x) {
    System.err.println(x);
}
```

#### Creating and Writing a File by Using Stream I/O

```java
import static java.nio.file.StandardOpenOption.*;
import java.nio.file.*;
import java.io.*;

public class LogFileTest {

  public static void main(String[] args) {

    // Convert the string to a
    // byte array.
    String s = "Hello World! ";
    byte data[] = s.getBytes();
    Path p = Paths.get("./logfile.txt");

    try (OutputStream out = new BufferedOutputStream(
      Files.newOutputStream(p, CREATE, APPEND))) {
      out.write(data, 0, data.length);
    } catch (IOException x) {
      System.err.println(x);
    }
  }
}
```

### Methods for Channels and `ByteBuffers`

#### Reading and Writing Files by Using Channel I/O

​	当Stream I/O一次读取一个字符时，Channel  I/O一次读取一个缓冲区。ByteChannel接口提供基本的读写功能。SeekableByteChannel是一个ByteChannel，它能够在通道中保持一个位置并更改该位置。SeekableByteChannel还支持截断与通道关联的文件并查询文件的大小。

​	移动到文件中的不同点，然后从该位置读写的能力使得随机访问文件成为可能。

There are two methods for reading and writing channel I/O.

- [`newByteChannel(Path, OpenOption...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#newByteChannel-java.nio.file.Path-java.nio.file.OpenOption...-)
- [`newByteChannel(Path, Set, FileAttribute...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#newByteChannel-java.nio.file.Path-java.util.Set-java.nio.file.attribute.FileAttribute...-)

> newByteChannel方法返回SeekableByteChannel的实例。对于默认的文件系统，您可以将这个可查找的字节通道强制转换为一个`FileChannel`，从而提供对更高级功能的访问，例如将文件的某个区域直接映射到内存中以实现更快的访问,锁定文件的某个区域以使其他进程无法访问该区域，或者在不影响通道当前位置的情况下从绝对位置读取和写入字节。

指定READ打开读取通道。指定WRITE或APPEND将打开写入通道。如果没有指定这些选项，则打开通道进行读取。

```java
// Defaults to READ
    try (SeekableByteChannel sbc = Files.newByteChannel(file)) {
    ByteBuffer buf = ByteBuffer.allocate(10);

    // Read the bytes with the proper encoding for this platform.  If
    // you skip this step, you might see something that looks like
    // Chinese characters when you expect Latin-style characters.
    String encoding = System.getProperty("file.encoding");
    while (sbc.read(buf) > 0) {
        buf.rewind();
        System.out.print(Charset.forName(encoding).decode(buf));
        buf.flip();
    }
} catch (IOException x) {
    System.out.println("caught exception: " + x);
```

以下为UNIX和其他POSIX文件系统编写的示例创建了一个具有特定文件权限集的日志文件。此代码将创建一个日志文件，如果日志文件已存在，则将其附加到日志文件中。日志文件是使用所有者的读/写权限和组的只读权限创建的。

```java
import static java.nio.file.StandardOpenOption.*;
import java.nio.*;
import java.nio.channels.*;
import java.nio.file.*;
import java.nio.file.attribute.*;
import java.io.*;
import java.util.*;

public class LogFilePermissionsTest {

  public static void main(String[] args) {
  
    // Create the set of options for appending to the file.
    Set<OpenOption> options = new HashSet<OpenOption>();
    options.add(APPEND);
    options.add(CREATE);

    // Create the custom permissions attribute.
    Set<PosixFilePermission> perms =
      PosixFilePermissions.fromString("rw-r-----");
    FileAttribute<Set<PosixFilePermission>> attr =
      PosixFilePermissions.asFileAttribute(perms);

    // Convert the string to a ByteBuffer.
    String s = "Hello World! ";
    byte data[] = s.getBytes();
    ByteBuffer bb = ByteBuffer.wrap(data);
    
    Path file = Paths.get("./permissions.log");

    try (SeekableByteChannel sbc =
      Files.newByteChannel(file, options, attr)) {
      sbc.write(bb);
    } catch (IOException x) {
      System.out.println("Exception thrown: " + x);
    }
  }
}
```

### Methods for Creating Regular and Temporary Files

#### Creating Files

​	您可以使用[`createFile(Path, FileAttribute)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#createFile-java.nio.file.Path-java.nio.file.attribute.FileAttribute...-)方法创建一个新的空文件，并且初始化一些属性进去。如果文件已经存在，createFile将抛出一个异常。

```java
Path file = ...;
try {
    // Create the empty file with default permissions, etc.
    Files.createFile(file);
} catch (FileAlreadyExistsException x) {
    System.err.format("file named %s" +
        " already exists%n", file);
} catch (IOException x) {
    // Some other sort of failure, such as permissions.
    System.err.format("createFile error: %s%n", x);
}
```

也可以使用newOutputStream方法创建新文件，如使用流I/O创建和写入文件中所述。如果打开新的输出流并立即关闭它，则会创建一个空文件。

#### Creating Temporary Files

可以使用以下createTempFile方法之一创建临时文件：



- [`createTempFile(Path, String, String, FileAttribute)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#createTempFile-java.nio.file.Path-java.lang.String-java.lang.String-java.nio.file.attribute.FileAttribute...-)
- [`createTempFile(String, String, FileAttribute)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#createTempFile-java.lang.String-java.lang.String-java.nio.file.attribute.FileAttribute...-)

```java
try {
    Path tempFile = Files.createTempFile(null, ".myapp");
    System.out.format("The temporary file" +
        " has been created: %s%n", tempFile);
} catch (IOException x) {
    System.err.format("IOException: %s%n", x);
}
//The temporary file has been created: /tmp/509668702974537184.myapp
```

## Random Access Files

​	随机访问文件允许对文件内容进行非连续或随机访问。要随机访问文件，请打开该文件，查找特定位置，然后从该文件中读取或写入。

​	SeekableByteChannel接口使用当前位置的概念扩展了通道I/O。

- [`position`](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SeekableByteChannel.html#position--) – Returns the channel's current position
- [`position(long)`](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SeekableByteChannel.html#position-long-) – Sets the channel's position
- [`read(ByteBuffer)`](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SeekableByteChannel.html#read-java.nio.ByteBuffer-) – Reads bytes into the buffer from the channel
- [`write(ByteBuffer)`](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SeekableByteChannel.html#write-java.nio.ByteBuffer-) – Writes bytes from the buffer to the channel
- [`truncate(long)`](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SeekableByteChannel.html#truncate-long-) – Truncates the file (or other entity) connected to the channel

[Reading and Writing Files With Channel I/O](https://docs.oracle.com/javase/tutorial/essential/io/file.html#channelio) shows that the `Path.newByteChannel` methods return an instance of a `SeekableByteChannel`

​	在默认文件系统上，您可以按原样使用该通道，也可以将其强制转换为一个文件通道，以便您访问更高级的功能，例如将文件的某个区域直接映射到内存中以实现更快的访问、锁定文件的某个区域，或者从绝对位置读写字节而不影响通道的当前位置。

```java
String s = "I was here!\n";
byte data[] = s.getBytes();
ByteBuffer out = ByteBuffer.wrap(data);

ByteBuffer copy = ByteBuffer.allocate(12);

try (FileChannel fc = (FileChannel.open(file, READ, WRITE))) {
    // Read the first 12
    // bytes of the file.
    int nread;
    do {
        nread = fc.read(copy);
    } while (nread != -1 && copy.hasRemaining());

    // Write "I was here!" at the beginning of the file.
    fc.position(0);
    while (out.hasRemaining())
        fc.write(out);
    out.rewind();

    // Move to the end of the file.  Copy the first 12 bytes to
    // the end of the file.  Then write "I was here!" again.
    long length = fc.size();
    fc.position(length-1);
    copy.flip();
    while (copy.hasRemaining())
        fc.write(copy);
    while (out.hasRemaining())
        fc.write(out);
} catch (IOException x) {
    System.out.println("I/O Exception: " + x);
}
```

## Creating and Reading Directories

### Listing a File System's Root Directories

```java
Iterable<Path> dirs = FileSystems.getDefault().getRootDirectories();
for (Path name: dirs) {
    System.err.println(name);
}
```

### Creating a Directory

 [`createDirectory(Path, FileAttribute)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#createDirectory-java.nio.file.Path-java.nio.file.attribute.FileAttribute...-)

```java
Path dir = ...;
Files.createDirectory(path)
```

```java
Set<PosixFilePermission> perms =
    PosixFilePermissions.fromString("rwxr-x---");
FileAttribute<Set<PosixFilePermission>> attr =
    PosixFilePermissions.asFileAttribute(perms);
Files.createDirectory(file, attr);
```

当一个或多个父目录可能还不存在时，要创建几个级别的目录，可以使用方便的方法`createDirectories(Path,FileAttribute<?>)`。

```java
Files.createDirectories(Paths.get("foo/bar/test"));
```

此方法可能在创建部分（但不是全部）父目录后失败。

### Creating a Temporary Directory

- [`createTempDirectory(Path, String, FileAttribute...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#createTempDirectory-java.nio.file.Path-java.lang.String-java.nio.file.attribute.FileAttribute...-)
- [`createTempDirectory(String, FileAttribute...)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#createTempDirectory-java.lang.String-java.nio.file.attribute.FileAttribute...-)

第一个方法允许代码指定临时目录的位置，第二个方法在默认的临时fle目录中创建一个新目录。

### Listing a Directory's Contents

​	You can list all the contents of a directory by using the [`newDirectoryStream(Path)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#newDirectoryStream-java.nio.file.Path-) method。返回的对象实现DirectoryStream接口，也实现了Iterable，因此您可以遍历目录流，读取所有对象。这种方法可以很好地扩展到非常大的目录。

```=java
Path dir = ...;
try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir)) {
    for (Path file: stream) {
        System.out.println(file.getFileName());
    }
} catch (IOException | DirectoryIteratorException x) {
    // IOException can never be thrown by the iteration.
    // In this snippet, it can only be thrown by newDirectoryStream.
    System.err.println(x);
}
```

此方法返回目录的全部内容：文件、链接、子目录和隐藏文件。如果您想对检索到的内容更具选择性，可以使用其他newDirectoryStream方法。

请注意，如果在目录迭代期间发生异常，那么DirectoryIteratorException将被抛出，原因是IOException。迭代器方法不能引发异常。

### Filtering a Directory Listing By Using Globbing

​	 [`newDirectoryStream(Path, String)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#newDirectoryStream-java.nio.file.Path-java.lang.String-) method, which provides a built-in glob filter。

```java
Path dir = ...;
try (DirectoryStream<Path> stream =
     Files.newDirectoryStream(dir, "*.{java,class,jar}")) {
    for (Path entry: stream) {
        System.out.println(entry.getFileName());
    }
} catch (IOException x) {
    // IOException can never be thrown by the iteration.
    // In this snippet, it can // only be thrown by newDirectoryStream.
    System.err.println(x);
}
```

### Writing Your Own Directory Filter

```java
DirectoryStream.Filter<Path> filter =
    new DirectoryStream.Filter<Path>() {
    public boolean accept(Path file) throws IOException {
        try {
            return (Files.isDirectory(path));
        } catch (IOException x) {
            // Failed to determine if it's a directory.
            System.err.println(x);
            return false;
        }
    }
};
```

```java
Path dir = ...;
try (DirectoryStream<Path>
                       stream = Files.newDirectoryStream(dir, filter)) {
        for (Path entry: stream) {
            System.out.println(entry.getFileName());
        }
} catch (IOException x) {
    System.err.println(x);
}
```

此方法仅用于筛选单个目录。但是，如果您想在一个文件树中找到所有的子目录，您可以使用遍历文件树的机制。

## Links, Symbolic or Otherwise

一些文件系统也支持硬链接。硬链接比符号链接更具限制性，如下所示。

- 链接的目标必须存在。
- 不允许目录
- 不允许硬链接跨分区或卷。因此，它们不能跨文件系统存在。
- 硬链接的外观和行为与常规文件类似，因此很难找到它们。
- 硬链接无论出于何种目的，都是与原始文件相同的实体。它们具有相同的文件权限、时间戳等。所有属性都相同。

### Creating a Symbolic Link

```java
Path newLink = ...;
Path target = ...;
try {
    Files.createSymbolicLink(newLink, target);
} catch (IOException x) {
    System.err.println(x);
} catch (UnsupportedOperationException x) {
    // Some file systems do not support symbolic links.
    System.err.println(x);
}
```

### Creating a Hard Link

```java
Path newLink = ...;
Path existingFile = ...;
try {
    Files.createLink(newLink, existingFile);
} catch (IOException x) {
    System.err.println(x);
} catch (UnsupportedOperationException x) {
    // Some file systems do not
    // support adding an existing
    // file to a directory.
    System.err.println(x);
}
```

### Detecting a Symbolic Link

```java
Path file = ...;
boolean isSymbolicLink =
    Files.isSymbolicLink(file);
```

### Finding the Target of a Link

```java
Path link = ...;
try {
    System.out.format("Target of link" +
        " '%s' is '%s'%n", link,
        Files.readSymbolicLink(link));
} catch (IOException x) {
    System.err.println(x);
}
```

If the `Path` is not a symbolic link, this method throws a `NotLinkException`.

## Walking the File Tree

### The FileVisitor Interface

​	为了遍历文件树，首先要实现`FileVisitor`。它指定遍历过程中关键点处所需的行为:when a file is visited, before a directory is accessed, after a directory is accessed, or when a failure occurs.

- [`preVisitDirectory`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileVisitor.html#preVisitDirectory-T-java.nio.file.attribute.BasicFileAttributes-) – Invoked before a directory's entries are visited.
- [`postVisitDirectory`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileVisitor.html#postVisitDirectory-T-java.io.IOException-) – Invoked after all the entries in a directory are visited. If any errors are encountered, the specific exception is passed to the method.
- [`visitFile`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileVisitor.html#visitFile-T-java.nio.file.attribute.BasicFileAttributes-) – Invoked on the file being visited. The file's `BasicFileAttributes` is passed to the method, or you can use the [file attributes](https://docs.oracle.com/javase/tutorial/essential/io/fileAttr.html) package to read a specific set of attributes. For example, you can choose to read the file's `DosFileAttributeView` to determine if the file has the "hidden" bit set.
- [`visitFileFailed`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileVisitor.html#visitFileFailedy-T-java.io.IOException-) – Invoked when the file cannot be accessed. The specific exception is passed to the method. You can choose whether to throw the exception, print it to the console or a log file, and so on.

如果不要实现这四个方法，可以继承 [`SimpleFileVisitor`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/SimpleFileVisitor.html)。这个类实现FileVisitor接口，访问树中的所有文件，并在遇到错误时抛出IOError。

```java
import static java.nio.file.FileVisitResult.*;

public static class PrintFiles
    extends SimpleFileVisitor<Path> {

    // Print information about
    // each type of file.
    @Override
    public FileVisitResult visitFile(Path file,
                                   BasicFileAttributes attr) {
        if (attr.isSymbolicLink()) {
            System.out.format("Symbolic link: %s ", file);
        } else if (attr.isRegularFile()) {
            System.out.format("Regular file: %s ", file);
        } else {
            System.out.format("Other: %s ", file);
        }
        System.out.println("(" + attr.size() + "bytes)");
        return CONTINUE;
    }

    // Print each directory visited.
    @Override
    public FileVisitResult postVisitDirectory(Path dir,
                                          IOException exc) {
        System.out.format("Directory: %s%n", dir);
        return CONTINUE;
    }

    // If there is some error accessing
    // the file, let the user know.
    // If you don't override this method
    // and an error occurs, an IOException 
    // is thrown.
    @Override
    public FileVisitResult visitFileFailed(Path file,
                                       IOException exc) {
        System.err.println(exc);
        return CONTINUE;
    }
}
```

### Kickstarting the Process

- [`walkFileTree(Path, FileVisitor)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#walkFileTree-java.nio.file.Path-java.nio.file.FileVisitor-)
- [`walkFileTree(Path, Set, int, FileVisitor)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#walkFileTree-java.nio.file.Path-java.util.Set-int-java.nio.file.FileVisitor-)

如果要确保此方法遍历整个文件树，可以指定Integer.MAX_值对于最大深度参数。

您可以指定FileVisitOption枚举FOLLOW_LINKS，这表示应该遵循符号链接。

```java
import static java.nio.file.FileVisitResult.*;

Path startingDir = ...;

EnumSet<FileVisitOption> opts = EnumSet.of(FOLLOW_LINKS);

Finder finder = new Finder(pattern);
Files.walkFileTree(startingDir, opts, Integer.MAX_VALUE, finder);
```

### Considerations When Creating a FileVisitor

​	文件树首先进行深度遍历，但不能对访问子目录的迭代顺序做出任何假设。

​	如果您的程序要更改文件系统，则需要仔细考虑如何实现FileVisitor。

​	如果要编写递归删除，请先删除目录中的文件，然后再删除目录本身。在本例中，删除postVisitDirectory中的目录。

​	如果您正在编写递归副本，请先在preVisitDirectory中创建新目录，然后再尝试将文件复制到该目录（在visitFiles中）。如果要保留源目录的属性（类似于unixcp-p命令），则需要在复制完文件后，在postVisitDirectory中执行此操作。The [`Copy`](https://docs.oracle.com/javase/tutorial/essential/io/examples/Copy.java) example shows how to do this.

​	如果要编写文件搜索，请在visitFile方法中执行比较。此方法查找所有符合条件的文件，但找不到目录。如果要同时查找文件和目录，还必须在preVisitDirectory或postVisitDirectory方法中执行比较。The [`Find`](https://docs.oracle.com/javase/tutorial/essential/io/examples/Find.java) example shows how to do this.

​	您需要决定是否要遵循符号链接。例如，如果要删除文件，则不建议使用以下符号链接。如果要复制文件树，可能需要允许它。默认情况下，walkFileTree不跟随符号链接。

​	对文件调用visitFile方法。如果指定了FOLLOW_LINKS选项，并且文件树具有指向父目录的循环链接，则在visitFileFailed方法中报告循环目录，并显示FileSystemLoopException。下面的代码片段演示了如何捕获循环链接：

```java
@Override
public FileVisitResult
    visitFileFailed(Path file,
        IOException exc) {
    if (exc instanceof FileSystemLoopException) {
        System.err.println("cycle detected: " + file);
    } else {
        System.err.format("Unable to copy:" + " %s: %s%n", file, exc);
    }
    return CONTINUE;
}
```

### Controlling the Flow

当你在寻找一个特定的目录树时，也许你想要终止一个目录。也许你想跳过特定的目录。

您可以中止文件遍历过程，也可以控制在FileVisitor方法中返回的值是否访问目录：

- `CONTINUE` – 指示应继续文件遍历。如果preVisitDirectory方法返回CONTINUE，则访问该目录。
- `TERMINATE` – 立即中止文件遍历。返回此值后，不再调用进一步的文件遍历方法。
- `SKIP_SUBTREE` – 当preVisitDirectory返回此值时，将跳过指定的目录及其子目录。这树枝是从树上“剪掉”的。
- `SKIP_SIBLINGS` – 当preVisitDirectory返回此值时，不访问指定的目录，不调用postVisitDirectory，也不访问其他未访问的同级目录。如果从postVisitDirectory方法返回，则不会访问其他同级。本质上，在指定的目录中不会再发生任何事情。

下面的代码跳过了所有叫SCCS的目录或文件。

```java
import static java.nio.file.FileVisitResult.*;

public FileVisitResult
     preVisitDirectory(Path dir,
         BasicFileAttributes attrs) {
    (if (dir.getFileName().toString().equals("SCCS")) {
         return SKIP_SUBTREE;
    }
    return CONTINUE;
}
```

在这个代码片段中，一旦找到特定的文件，文件名就被打印到标准输出，文件遍历终止。

```
import static java.nio.file.FileVisitResult.*;

// The file we are looking for.
Path lookingFor = ...;

public FileVisitResult
    visitFile(Path file,
        BasicFileAttributes attr) {
    if (file.getFileName().equals(lookingFor)) {
        System.out.println("Located file: " + file);
        return TERMINATE;
    }
    return CONTINUE;
}
```

### Examples

- [`Find`](https://docs.oracle.com/javase/tutorial/essential/io/examples/Find.java) – Recurses a file tree looking for files and directories that match a particular glob pattern. This example is discussed in [Finding Files](https://docs.oracle.com/javase/tutorial/essential/io/find.html).
- [`Chmod`](https://docs.oracle.com/javase/tutorial/essential/io/examples/Chmod.java) – Recursively changes permissions on a file tree (for POSIX systems only).
- [`Copy`](https://docs.oracle.com/javase/tutorial/essential/io/examples/Copy.java) – Recursively copies a file tree.
- [`WatchDir`](https://docs.oracle.com/javase/tutorial/essential/io/examples/WatchDir.java) – Demonstrates the mechanism that watches a directory for files that have been created, deleted or modified. Calling this program with the `-r` option watches an entire tree for changes. For more information about the file notification service, see [Watching a Directory for Changes](https://docs.oracle.com/javase/tutorial/essential/io/notification.html).

# Finding Files

这个java.nio.file文件包为模式匹配特性提供了编程支持。每个文件系统实现都提供了一个路径匹配器。可以使用FileSystem类中的getPathMatcher（String）方法检索文件系统的PathMatcher。以下代码段获取默认文件系统的路径匹配器：

```java
String pattern = ...;
PathMatcher matcher =
    FileSystems.getDefault().getPathMatcher("glob:" + pattern);
```

```java
PathMatcher matcher =
    FileSystems.getDefault().getPathMatcher("glob:*.{java,class}");

Path filename = ...;
if (matcher.matches(filename)) {
    System.out.println(filename);
}
```

### Recursive Pattern Matching

​	搜索与特定模式匹配的文件与遍历文件树密切相关。有多少次你知道一个文件在文件系统的某个地方，但是在哪里？或者，您可能需要在文件树中查找具有特定文件扩展名的所有文件。

```java
/**
 * Sample code that finds files that match the specified glob pattern.
 * For more information on what constitutes a glob pattern, see
 * https://docs.oracle.com/javase/tutorial/essential/io/fileOps.html#glob
 *
 * The file or directories that match the pattern are printed to
 * standard out.  The number of matches is also printed.
 *
 * When executing this application, you must put the glob pattern
 * in quotes, so the shell will not expand any wild cards:
 *              java Find . -name "*.java"
 */

import java.io.*;
import java.nio.file.*;
import java.nio.file.attribute.*;
import static java.nio.file.FileVisitResult.*;
import static java.nio.file.FileVisitOption.*;
import java.util.*;


public class Find {

    public static class Finder
        extends SimpleFileVisitor<Path> {

        private final PathMatcher matcher;
        private int numMatches = 0;

        Finder(String pattern) {
            matcher = FileSystems.getDefault()
                    .getPathMatcher("glob:" + pattern);
        }

        // Compares the glob pattern against
        // the file or directory name.
        void find(Path file) {
            Path name = file.getFileName();
            if (name != null && matcher.matches(name)) {
                numMatches++;
                System.out.println(file);
            }
        }

        // Prints the total number of
        // matches to standard out.
        void done() {
            System.out.println("Matched: "
                + numMatches);
        }

        // Invoke the pattern matching
        // method on each file.
        @Override
        public FileVisitResult visitFile(Path file,
                BasicFileAttributes attrs) {
            find(file);
            return CONTINUE;
        }

        // Invoke the pattern matching
        // method on each directory.
        @Override
        public FileVisitResult preVisitDirectory(Path dir,
                BasicFileAttributes attrs) {
            find(dir);
            return CONTINUE;
        }

        @Override
        public FileVisitResult visitFileFailed(Path file,
                IOException exc) {
            System.err.println(exc);
            return CONTINUE;
        }
    }

    static void usage() {
        System.err.println("java Find <path>" +
            " -name \"<glob_pattern>\"");
        System.exit(-1);
    }

    public static void main(String[] args)
        throws IOException {

        if (args.length < 3 || !args[1].equals("-name"))
            usage();

        Path startingDir = Paths.get(args[0]);
        String pattern = args[2];

        Finder finder = new Finder(pattern);
        Files.walkFileTree(startingDir, finder);
        finder.done();
    }
}

//java Find . -name "*.html"
```

## Watching a Directory for Changes

### Watch Service Overview

​	watchservice api是相当低的级别，允许您自定义它。您可以按原样使用它，也可以选择在这个机制之上创建一个高级API，以便它适合您的特定需求。

Here are the basic steps required to implement a watch service:

- 为文件系统创建一个WatchService“watcher”。
- 对于要监视的每个目录，请向监视程序注册它。注册目录时，指定要通知的事件类型。您将收到注册的每个目录的WatchKey实例。
- 实现无限循环以等待传入事件。当发生事件时，会发出信号，并将该密钥放入监视者的队列中。
- 从观察者的队列中检索密钥。您可以从密钥获取文件名。
- 检索密钥的每个挂起事件（可能有多个事件）并根据需要进行处理。
- 重置键，并继续等待事件。
- 关闭服务：监视服务在线程退出或关闭时退出（通过调用其closed方法）。

监视键是线程安全的，可以与java.nio.concurrent包一起使用。您可以将线程池专用于此工作。

### Try It Out

由于此API更高级，请在继续之前试用它。保存你的WatchDir实例（[`WatchDir`](https://docs.oracle.com/javase/tutorial/essential/io/examples/WatchDir.java)）。创建一个将传递给示例的测试目录。WatchDir使用一个线程来处理所有事件，因此它在等待事件时阻止键盘输入。

```
java WatchDir test &
```

在test目录中创建、删除和编辑文件。当这些事件发生时，会向控制台输出一条消息。完成后，删除test目录并退出WatchDir。或者，如果您愿意，您可以手动终止进程。

### Creating a Watch Service and Registering for Events

第一步是创建一个新的WatchService。

```java
WatchService watcher = FileSystems.getDefault().newWatchService();
```

​	接下来，向监视服务注册一个或多个对象。任何实现可监视接口的对象都可以注册。Path类实现了可监视的接口，因此要监视的每个目录都注册为Path对象。

​	向监视服务注册对象时，指定要监视的事件类型。支持的StandardWatchEventKinds事件类型如下：

- `ENTRY_CREATE` – A directory entry is created.
- `ENTRY_DELETE` – A directory entry is deleted.
- `ENTRY_MODIFY` – A directory entry is modified.
- `OVERFLOW` – Indicates that events might have been lost or discarded. You do not have to register for the `OVERFLOW` event to receive it.

```java
import static java.nio.file.StandardWatchEventKinds.*;

Path dir = ...;
try {
    WatchKey key = dir.register(watcher,
                           ENTRY_CREATE,
                           ENTRY_DELETE,
                           ENTRY_MODIFY);
} catch (IOException x) {
    System.err.println(x);
}
```

### Processing Events

处理时间的步骤如下：

1. Get a watch key. Three methods are provided:
   - [`poll`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/WatchService.html#poll--) – Returns a queued key, if available. Returns immediately with a `null` value, if unavailable.
   - [`poll(long, TimeUnit)`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/WatchService.html#poll-long-java.util.concurrent.TimeUnit-) – Returns a queued key, if one is available. If a queued key is not immediately available, the program waits until the specified time. The `TimeUnit` argument determines whether the specified time is nanoseconds, milliseconds, or some other unit of time.
   - [`take`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/WatchService.html#take--) – Returns a queued key. If no queued key is available, this method waits.
2. Process the pending events for the key. You fetch the `List` of [`WatchEvents`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/WatchEvent.html)from the [`pollEvents`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/WatchKey.html#pollEvents--) method.
3. Retrieve the type of event by using the [`kind`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/WatchEvent.html#kind--) method. No matter what events the key has registered for, it is possible to receive an `OVERFLOW` event. You can choose to handle the overflow or ignore it, but you should test for it.
4. Retrieve the file name associated with the event. The file name is stored as the context of the event, so the [`context`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/WatchEvent.html#context--) method is used to retrieve it.
5. After the events for the key have been processed, you need to put the key back into a `ready` state by invoking [`reset`](https://docs.oracle.com/javase/8/docs/api/java/nio/file/WatchEvent.html#reset--). If this method returns `false`, the key is no longer valid and the loop can exit. This step is very **important**. If you fail to invoke `reset`, this key will not receive any further events.

watch key 有三个状态

- `Ready` 指示密钥已准备好接受事件。第一次创建时，密钥处于就绪状态。
- `Signaled` 指示一个或多个事件已排队。一旦发出信号，键就不再处于就绪状态，直到调用reset方法为止。
- `Invalid`指示该键不再处于活动状态。当下列事件之一发生时，就会发生这种状态：
  - 进程使用cancel方法显式地取消键。
  - 目录无法访问。
  - watch服务已关闭。

```java
for (;;) {

    // wait for key to be signaled
    WatchKey key;
    try {
        key = watcher.take();
    } catch (InterruptedException x) {
        return;
    }

    for (WatchEvent<?> event: key.pollEvents()) {
        WatchEvent.Kind<?> kind = event.kind();

        // This key is registered only
        // for ENTRY_CREATE events,
        // but an OVERFLOW event can
        // occur regardless if events
        // are lost or discarded.
        if (kind == OVERFLOW) {
            continue;
        }

        // The filename is the
        // context of the event.
        WatchEvent<Path> ev = (WatchEvent<Path>)event;
        Path filename = ev.context();

        // Verify that the new
        //  file is a text file.
        try {
            // Resolve the filename against the directory.
            // If the filename is "test" and the directory is "foo",
            // the resolved name is "test/foo".
            Path child = dir.resolve(filename);
            if (!Files.probeContentType(child).equals("text/plain")) {
                System.err.format("New file '%s'" +
                    " is not a plain text file.%n", filename);
                continue;
            }
        } catch (IOException x) {
            System.err.println(x);
            continue;
        }

        // Email the file to the
        //  specified email alias.
        System.out.format("Emailing file %s%n", filename);
        //Details left to reader....
    }

    // Reset the key -- this step is critical if you want to
    // receive further watch events.  If the key is no longer valid,
    // the directory is inaccessible so exit the loop.
    boolean valid = key.reset();
    if (!valid) {
        break;
    }
}
```

### Retrieving the File Name

 The [`Email`](https://docs.oracle.com/javase/tutorial/essential/io/examples/Email.java) example retrieves the file name with this code

```java
WatchEvent<Path> ev = (WatchEvent<Path>)event;
Path filename = ev.context();
```

When you compile the `Email` example, it generates the following error:

```java
Note: Email.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```

此错误是由于代码行将`WatchEvent<T>`强制转换为`WatchEvent<Path>`。The [`WatchDir`](https://docs.oracle.com/javase/tutorial/essential/io/examples/WatchDir.java) example avoids this error by creating a utility `cast` method that suppresses the unchecked warning, as follows:

```java
@SuppressWarnings("unchecked")
static <T> WatchEvent<T> cast(WatchEvent<?> event) {
    return (WatchEvent<Path>)event;
}
```

### When to Use and Not Use This API

​	监视服务API是为需要通知文件更改事件的应用程序而设计的。它非常适合于任何应用程序，如编辑器或IDE，这些应用程序可能有许多打开的文件，并且需要确保这些文件与文件系统同步。它还非常适合监视目录的应用程序服务器，可能等待.jsp或.jar文件删除，以便部署它们。

​	此API不是为索引硬盘而设计的。大多数文件系统实现对文件更改通知具有本机支持。监视服务API在可用的情况下利用了这种支持。但是，当文件系统不支持此机制时，监视服务将轮询文件系统，等待事件。

## Other Useful Methods

### Determining MIME Type

```java
try {
    String type = Files.probeContentType(filename);
    if (type == null) {
        System.err.format("'%s' has an" + " unknown filetype.%n", filename);
    } else if (!type.equals("text/plain") {
        System.err.format("'%s' is not" + " a plain text file.%n", filename);
        continue;
    }
} catch (IOException x) {
    System.err.println(x);
}
```

Note that `probeContentType` returns null if the content type cannot be determined.

这种方法的实现是高度特定于平台的，并且不是绝对正确的。内容类型由平台的默认文件类型检测器确定。

### Default File System

```java
PathMatcher matcher =
    FileSystems.getDefault().getPathMatcher("glob:*.*");
```

### Path String Separator

```java
String separator = File.separator;
String separator = FileSystems.getDefault().getSeparator();
```

### File System's File Stores

一个文件系统有一个或多个文件存储来保存它的文件和目录。文件存储表示底层存储设备。在UNIX操作系统中，每个装载的文件系统都由一个文件存储区表示。在Microsoft Windows中，每个卷都由一个文件存储表示：C:、D:，依此类推。

```java
for (FileStore store: FileSystems.getDefault().getFileStores()) {
   ...
}
```

```java
Path file = ...;
FileStore store= Files.getFileStore(file);
```

The [`DiskUsage`](https://docs.oracle.com/javase/tutorial/essential/io/examples/DiskUsage.java) example uses the `getFileStores` method.

## Legacy File I/O Code

### Interoperability With Legacy Code

在JavaSE7发行版之前java.io.File文件类是用于文件I/O的机制，但它有几个缺点：

- 许多方法在失败时不会抛出异常，因此不可能获得有用的错误消息。例如，如果文件删除失败，程序将收到“删除失败”消息，但不知道是因为文件不存在、用户没有权限还是存在其他问题。
- rename方法在不同平台上的工作并不一致。
- 没有真正支持符号链接。
- 需要对元数据的更多支持，例如文件权限、文件所有者和其他安全属性。
- 访问文件元数据效率低下。
- 许多文件方法无法扩展。在服务器上请求大目录列表可能会导致挂起。大目录还可能导致内存资源问题，从而导致拒绝服务。
- 如果有一个可靠的循环树不能正确地响应，那么它就不能递归地写一个循环链接。

### Mapping java.io.File Functionality to java.nio.file

| java.io.File Functionality                                   | java.nio.file Functionality                                  | Tutorial Coverage                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `java.io.File`                                               | `java.nio.file.Path`                                         | [The Path Class](https://docs.oracle.com/javase/tutorial/essential/io/pathClass.html) |
| `java.io.RandomAccessFile`                                   | The `SeekableByteChannel` functionality.                     | [Random Access Files](https://docs.oracle.com/javase/tutorial/essential/io/rafs.html) |
| `File.canRead`, `canWrite`, `canExecute`                     | `Files.isReadable`, `Files.isWritable`, and `Files.isExecutable`. On UNIX file systems, the [Managing Metadata (File and File Store Attributes)](https://docs.oracle.com/javase/tutorial/essential/io/fileAttr.html) package is used to check the nine file permissions. | [Checking a File or Directory](https://docs.oracle.com/javase/tutorial/essential/io/check.html) [Managing Metadata](https://docs.oracle.com/javase/tutorial/essential/io/fileAttr.html) |
| `File.isDirectory()`, `File.isFile()`, and `File.length()`   | `Files.isDirectory(Path, LinkOption...)`, `Files.isRegularFile(Path, LinkOption...)`, and `Files.size(Path)` | [Managing Metadata](https://docs.oracle.com/javase/tutorial/essential/io/fileAttr.html) |
| `File.lastModified()` and `File.setLastModified(long)`       | `Files.getLastModifiedTime(Path, LinkOption...)` and `Files.setLastMOdifiedTime(Path, FileTime)` | [Managing Metadata](https://docs.oracle.com/javase/tutorial/essential/io/fileAttr.html) |
| The `File` methods that set various attributes: `setExecutable`, `setReadable`, `setReadOnly`, `setWritable` | These methods are replaced by the `Files` method `setAttribute(Path, String, Object, LinkOption...)`. | [Managing Metadata](https://docs.oracle.com/javase/tutorial/essential/io/fileAttr.html) |
| `new File(parent, "newfile")`                                | `parent.resolve("newfile")`                                  | [Path Operations](https://docs.oracle.com/javase/tutorial/essential/io/pathOps.html) |
| `File.renameTo`                                              | `Files.move`                                                 | [Moving a File or Directory](https://docs.oracle.com/javase/tutorial/essential/io/move.html) |
| `File.delete`                                                | `Files.delete`                                               | [Deleting a File or Directory](https://docs.oracle.com/javase/tutorial/essential/io/delete.html) |
| `File.createNewFile`                                         | `Files.createFile`                                           | [Creating Files](https://docs.oracle.com/javase/tutorial/essential/io/file.html#createFile) |
| `File.deleteOnExit`                                          | Replaced by the `DELETE_ON_CLOSE` option specified in the `createFile` method. | [Creating Files](https://docs.oracle.com/javase/tutorial/essential/io/file.html#createFile) |
| `File.createTempFile`                                        | `Files.createTempFile(Path, String, FileAttributes<?>)`, `Files.createTempFile(Path, String, String, FileAttributes<?>)` | [Creating Files](https://docs.oracle.com/javase/tutorial/essential/io/file.html#createFile) [Creating and Writing a File by Using Stream I/O](https://docs.oracle.com/javase/tutorial/essential/io/file.html#createStream) [Reading and Writing Files by Using Channel I/O](https://docs.oracle.com/javase/tutorial/essential/io/file.html#channelio) |
| `File.exists`                                                | `Files.exists` and `Files.notExists`                         | [Verifying the Existence of a File or Directory](https://docs.oracle.com/javase/tutorial/essential/io/check.html) |
| `File.compareTo` and `equals`                                | `Path.compareTo` and `equals`                                | [Comparing Two Paths](https://docs.oracle.com/javase/tutorial/essential/io/pathOps.html#compare) |
| `File.getAbsolutePath` and `getAbsoluteFile`                 | `Path.toAbsolutePath`                                        | [Converting a Path](https://docs.oracle.com/javase/tutorial/essential/io/pathOps.html#convert) |
| `File.getCanonicalPath` and `getCanonicalFile`               | `Path.toRealPath` or `normalize`                             | [Converting a Path (`toRealPath`)](https://docs.oracle.com/javase/tutorial/essential/io/pathOps.html#convert) [Removing Redundancies From a Path (`normalize`)](https://docs.oracle.com/javase/tutorial/essential/io/pathOps.html#normal) |
| `File.toURI`                                                 | `Path.toURI`                                                 | [Converting a Path](https://docs.oracle.com/javase/tutorial/essential/io/pathOps.html#convert) |
| `File.isHidden`                                              | `Files.isHidden`                                             | [Retrieving Information About the Path](https://docs.oracle.com/javase/tutorial/essential/io/pathOps.html#info) |
| `File.list` and `listFiles`                                  | `Path.newDirectoryStream`                                    | [Listing a Directory's Contents](https://docs.oracle.com/javase/tutorial/essential/io/dirs.html#listdir) |
| `File.mkdir` and `mkdirs`                                    | `Files.createDirectory`                                      | [Creating a Directory](https://docs.oracle.com/javase/tutorial/essential/io/dirs.html#create) |
| `File.listRoots`                                             | `FileSystem.getRootDirectories`                              | [Listing a File System's Root Directories](https://docs.oracle.com/javase/tutorial/essential/io/dirs.html#listall) |
| `File.getTotalSpace`, `File.getFreeSpace`, `File.getUsableSpace` | `FileStore.getTotalSpace`, `FileStore.getUnallocatedSpace`, `FileStore.getUsableSpace`, `FileStore.getTotalSpace` | [File Store Attributes](https://docs.oracle.com/javase/tutorial/essential/io/fileAttr.html#store) |



