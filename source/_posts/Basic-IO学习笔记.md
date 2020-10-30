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