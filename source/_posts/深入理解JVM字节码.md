# 1. 深入剖析class文件结构

## 1.1 初探class文件

​	Java是平台无关的语言，但JVM却不是跨平台的，不同平台的JVM帮我们屏蔽了平台的差异。通过这些虚拟机加载和执行同一种平台无关的字节码，我们的源代码就不用根据不同平台编译成不同的二进制可执行文件，Java平台无关性如图1-2所示。

![](https://pic.imgdb.cn/item/619d9ae42ab3f51d9165c398.jpg)

![](https://pic.imgdb.cn/item/619d9c7b2ab3f51d916642a1.jpg)

## 1.2 class文件结构剖析

​	Java虚拟机规定用u1、u2、u4三种数据结构来表示1、2、4字节无符号整数，相同类型的若干条数据集合用表（table）的形式来存储。表是一个变长的结构，由代表长度的表头n和紧随着的n个数据项组成。class文件采用类似C语言的结构体来存储数据，如下所示。

```c++
classFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

​	class文件由下面十个部分组成：

* 魔数（Magic Number）
* 版本号（Minor&Major Version）
* 常量池（Constant Pool）
* 类访问标记（Access Flag）
* 类索引（This Class）
* 超类索引（Super Class）
* 接口表索引（Interface）
* 字段表（Field）
* 方法表（Method）
* 属性表（Attribute）



​	Optimizing Java的作者编了一句顺口溜帮忙记住上面这十部分：My Very Cute Animal Turns Savage In Full Moon Areas。如图1-4所示。

![](https://pic.imgdb.cn/item/619d9d862ab3f51d916691fc.jpg)

### 1.2.1 魔数

​	魔数0xCAFEBABE是JVM识别.class文件的标志，虚拟机在加载类文件之前会先检查这4个字节，如果不是0xCAFEBABE，则会抛出java.lang.ClassFormatError异常。我们可以把前面的class文件的4个字节改为0xCAFEBABA来模拟这种情况，使用Java运行这个修改过的class文件，会出现预期的异常，如图1-6所示。

![](https://pic.imgdb.cn/item/619d9e012ab3f51d9166befc.jpg)

### 1.2.2 版本号

![](https://pic.imgdb.cn/item/619da56f2ab3f51d9169b19f.jpg)

​	这里的主版本号是52（0x34），虚拟机解析这个类时就知道这是一个Java 8编译出的类，如果类文件的版本号高于JVM自身的版本号，加载该类会被直接抛出java.lang.UnsupportedClassVersionError异常，如图1-8所示。

![](https://pic.imgdb.cn/item/619da5912ab3f51d9169c5bf.jpg)

### 1.2.3 常量池

​	紧随版本号之后的是常量池数据区域，常量池是类文件中最复杂的数据结构。对于JVM字节码来说，如果操作数是很常用的数字，比如0，这些操作数是内嵌到字节码中的。如果是字符串常量和较大的整数等，class文件则会把这些操作数存储在常量池（Constant Pool）中，当使用这些操作数时，会根据常量池的索引位置来查找。

```c++
struct {
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
}
```

​	1）常量池大小（cp_info_count）：常量池是class文件中第一个出现的变长结构。既然是池，就有大小，常量池大小由两个字节表示。假设常量池大小为n，常量池真正有效的索引是1～n-1。也就是说，如果constant_pool_count等于10，constant_pool数组的有效索引值是1～9。0属于保留索引，可供特殊情况使用。

​	2）常量池项（cp_info）集合：最多包含n-1个元素。为什么是最多呢？long和double类型的常量会占用两个索引位置，如果常量池包含了这两种类型的元素，实际的常量池项的元素个数比n-1要小。

![](https://pic.imgdb.cn/item/619da6c92ab3f51d916a74b2.jpg)

![](https://pic.imgdb.cn/item/619da6e02ab3f51d916a8425.jpg)

```java
javap -v HelloWorld
Constant pool:
   #1 = Methodref      #6.#15     // java/lang/Object."<init>":()V
   #2 = Fieldref       #16.#17    // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String         #18        // Hello, World
  ...
  #27 = Utf8           println
  #28 = Utf8           (Ljava/lang/String;)V
```

**1.CONSTANT_Integer_info和CONSTANT_Float_info**

```c++
CONSTANT_Integer_info {
    u1 tag;
    u4 bytes;
}

CONSTANT_Float_info {
    u1 tag;
    u4 bytes;
}
```

​	Java语言规范还定义了boolean、byte、short和char类型的变量，在常量池中都会被当作int来处理，以下面的代码清单1-2为例。

```java
public class MyConstantTest {
    public final boolean bool = true; //  1(0x01)
    public final char c = 'A';        // 65(0x41)
    public final byte b = 66;         // 66(0x42)
    public final short s = 67;        // 67(0x43)
    public final int i = 68;          // 68(0x44)
}
```

**2.CONSTANT_Long_info和CONSTANT_Double_info**

​	CONSTANT_Long_info和CONSTANT_Double_info这两种结构分别用来表示long和double类型的常量，二者都用8个字节表示具体的常量数值，它们的结构如下面的代码所示。

```c++
CONSTANT_Long_info {
    u1 tag;
    u4 high_bytes;
    u4 low_bytes;
}
CONSTANT_Double_info {
    u1 tag;
    u4 high_bytes;
    u4 low_bytes;
}
```

**3.CONSTANT_Utf8_info**

```c++
CONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```

​	它由三部分构成：第一个字节是tag，值为固定值1；tag之后的两个字节length并不是表示字符串有多少个字符，而是表示第三部分byte数组的长度；第三部分是采用MUTF-8编码的长度为length的字节数组。

![](https://pic.imgdb.cn/item/619da8a22ab3f51d916b1250.jpg)

​	MUTF-8编码与标准的UTF-8编码在大部分情况下是相同的，但也有一些细微的区别，为了能搞清楚MUTF-8，需要知道UTF-8编码是如何实现的。UTF-8是一种变长编码方式，使用1～4个字节表示一个字符，规则如下。

​	1）对于传统的ASCII编码字符（0x0001~0x007F），UTF-8用一个字节来表示，如下所示。

```
0000 0001 ~ 0000 007F -> 0xxxxxxx
```

因此英文字母的ASCII编码和UTF-8编码的结果一样。
2）对于0080~07FF范围的字符，UTF-8用2个字节来表示，如下所示。

```
0000 0080 ~ 0000 07FF -> 110xxxxx 10xxxxxx
```

​	程序在遇到这种字符的时候，会把第一个字节的110和第二个字节的10去掉，再把剩下的bit组成新的两字节数据。

​	3）对于0000 0800~0000 FFFF范围的字符，UTF-8用3个字节表示，如下所示。

```
0000 0800 ~ 0000 FFFF -> 1110xxxx 10xxxxxx 10xxxxxx
```

​	程序在遇到这种字符的时候，会把第一个字节的1110、第二和第三个字节的10去掉，再把剩下的bit组成新的3字节数据。

​	4）对于0001 0000~0010 FFFF范围的字符，UTF-8用4个字节表示，如下所示。

```
0001 0000-0010 FFFF -> 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
```

​	那MUTF-8有什么不一样呢？它们之间的区别如下。

1）MUTF-8里用两个字节表示空字符（"\0"），把前面介绍的双字节表示格式110xxxxx 10xxxxxx中的x全部填0，也即0xC080，而在标准UTF-8编码中只用一个字节0x00表示。这样做的原因是在其他语言中（比如C语言）会把空字符当作字符串的结束，而MUTF-8这种处理空字符的方式保证字符串中不会出现空字符，在C语言处理时不会意外截断。

2）MUTF-8只用到了标准UTF-8编码中的单字节、双字节、三字节表示方式，没有用到4字节表示方式。编码在U+FFFF之上的字符，Java使用“代理对”（surrogate pair）通过2个字符表示，比如“![](https://pic.imgdb.cn/item/619da9f42ab3f51d916b8de9.jpg)”的代理对为\ud83d\ude02。

**4.CONSTANT_String_info**

​	CONSTANT_String_info用来表示java.lang.String类型的常量对象。它与CONSTANT_Utf8_info的区别是CONSTANT_Utf8_info存储了字符串真正的内容，而CONSTANT_String_info并不包含字符串的内容，仅仅包含一个指向常量池中CONSTANT_Utf8_info常量类型的索引。

​	CONSTANT_String_info的结构由两部分构成，第一个字节是tag，值为8，tag后面的两个字节是一个名为string_index的索引值，指向常量池中的CONSTANT_Utf8_info，这个CONSTANT_Utf8_info中存储的才是真正的字符串常量内容，如下所示。

```java
CONSTANT_String_info {
    u1 tag;
    u2 string_index;
}
```

以下面代码中的字符串a为例。

```java
public class Hello {
    private String a = "hello";
}
```

![](https://pic.imgdb.cn/item/619dab302ab3f51d916c1dff.jpg)

![](https://pic.imgdb.cn/item/619dab402ab3f51d916c26bb.jpg)

**5.CONSTANT_Class_info**

​	CONSTANT_Class_info结构用来表示类或接口，它的结构与CONSTANT_String_info非常类似，可用下面的伪代码表示。

```java
CONSTANT_Class_info {
    u1 tag;
    u2 name_index;
}
```

​	它由两部分组成，第一个字节是tag，值固定为7，tag后面的两个字节name_index是一个常量池索引，指向CONSTANT_Utf8_info常量，这个字符串存储的是类或接口的全限定名，如图1-19所示。

![](https://pic.imgdb.cn/item/619daba32ab3f51d916c521f.jpg)

**6.CONSTANT_NameAndType_info**

​	CONSTANT_NameAndType_info结构用来表示字段或者方法，可以用下面的伪代码表示。

```
CONSTANT_NameAndType_info{
    u1 tag;
    u2 name_index;
    u2 descriptor_index;
}
```

​	CONSTANT_NameAndType_info结构由三部分组成，第一部分tag值固定为12，后面的两个部分name_index和descriptor_index都指向常量池中的CONSTANT_Utf8_info的索引，name_index表示字段或方法的名字，descriptor_index是字段或方法的描述符，用来表示一个字段或方法的类型，字段和方法描述符在本章后面会有详细介绍。

​	以下面代码中的testMethod为例。

```java
public void testMethod(int id, String name) {
}
```

​	对应的CONSTANT_NameAndType_info的结构布局示意图如图1-20所示。

![](https://pic.imgdb.cn/item/619dac5e2ab3f51d916ca01c.jpg)

I => int

Ljava/lang/String => String

V => void

**7.CONSTANT_Fieldref_info、CONSTANT_Methodref_info和CONSTANT_InterfaceMethodref_info**

```java
CONSTANT_Fieldref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}

CONSTANT_Methodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}

CONSTANT_InterfaceMethodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```

​	下面以CONSTANT_Methodref_info为例来进行讲解，它用来描述一个方法。它由三部分组成：第一部分是tag值，固定为10；第二部分是class_index，是一个指向CONSTANT_Class_info的常量池索引值，表示方法所在的类信息；第三部分是name_and_type_index，是一个指向CONSTANT_NameAndType_info的常量池索引值，表示方法的方法名、参数和返回值类型。以下面的代码清单1-3为例。

```java
public class HelloWorldMain {
    public static void main(String[] args) {
        new HelloWorldMain().testMethod(1, "hi");
    }
    public void testMethod(int id, String name) {
    }
}

Constant pool:
   #2 = Class       #18      // HelloWorldMain
   #5 = Methodref   #2.#20   // HelloWorldMain.testMethod:(ILjava/lang/String;)V
  #20 = NameAndType #13:#14  // testMethod:(ILjava/lang/String;)V
```

​	testMethod对应的Methodref的class_index为2，指向类名为“HelloWorldMain”的类，name_and_type_index为20，指向常量池中下标为20的NameAndType索引项，对应的方法名为“testMethod”，方法类型为“（ILjava/lang/String；）V”。

​	testMethod的Methodref信息可以用图1-21表示。

![](https://pic.imgdb.cn/item/619dade32ab3f51d916d42b9.jpg)

**8.CONSTANT_MethodType_info、CONSTANT_MethodHandle_info和CONSTANT_InvokeDynamic_info**

​	从JDK1.7开始，为了更好地支持动态语言调用，新增了3种常量池类型（CONSTANT_MethodType_info、CONSTANT_MethodHandle_info和CONSTANT_InvokeDynamic_info）。以CONSTANT_InvokeDynamic_info为例，CONSTANT_InvokeDynamic_info的主要作用是为invokedynamic指令提供启动引导方法，它的结构如下所示。

```
CONSTANT_InvokeDynamic_info {
    u1 tag;
    u2 bootstrap_method_attr_index;
    u2 name_and_type_index;
}
```

​	第一部分为tag，值固定为18；第二部分为bootstrap_method_attr_index，是指向引导方法表bootstrap_methods[]数组的索引。第三部分为name_and_type_index，是指向索引类常量池里的CONSTANT_NameAndType_info的索引，表示方法描述符。以下面的代码清单1-4为例。

```java
public void foo() {
    new Thread (()-> {
        System.out.println("hello");
    }).start();
}

javap 输出的常量池的部分如下：

Constant pool:
   #3 = InvokeDynamic      #0:#25         // #0:run:()Ljava/lang/Runnable;
   ...
  #25 = NameAndType        #37:#38        // run:()Ljava/lang/Runnable;

BootstrapMethods:
  0: #22 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #23 ()V
      #24 invokestatic HelloWorldMain.lambda$foo$0:()V
      #23 ()V
```

​	整体的结构如图1-22所示。

![](https://pic.imgdb.cn/item/619f04db2ab3f51d91f35699.jpg)

### 1.2.4 AccessFlags

​	紧随常量池之后的区域是访问标记（Access flags），用来标识一个类为final、abstract等，由两个字节表示，总共有16个标记位可供使用，目前只使用了其中的8个，如图1-23所示。

​	完整的访问标记含义如表1-3所示。

![](https://pic.imgdb.cn/item/619f05222ab3f51d91f3755f.jpg)

​	这些访问标记并不是可以随意组合的，比如ACC_PUBLIC、ACC_PRIVATE、ACC_PROTECTED不能同时设置，ACC_FINAL和ACC_ABSTRACT也不能同时设置，否则会违背语义。更多的规则可以在javac源码的com.sun.tools.javac.comp.Check.java文件中找到。

### 1.2.5 this_class、super_name、interfaces

​	这三部分用来确定类的继承关系，this_class表示类索引，super_name表示直接父类的索引，interfaces表示类或者接口的直接父接口。

```java
public class Hello {
    public static void main(String[] args) {
    }
}

Constant pool:
   // ...
   #2 = Class              #13            // Hello
   // ...
  #13 = Utf8               Hello
```

​	本例中this_class为0x0002，指向常量池中下标为2的元素，这个元素是CONSTANT_Class_info类型，它的name_index指向常量池中下标为13、类型为CONSTANT_Utf8_info的元素，表示类名为“Hello”，如图1-25所示。

![](https://pic.imgdb.cn/item/619f05f82ab3f51d91f3c958.jpg)

### 1.2.6 字段表

​	紧随接口索引表之后的是字段表（fields），类中定义的字段会被存储到这个集合中，包括静态和非静态的字段，它的结构可以用下面的伪代码表示。

```c
{
    u2             fields_count;
    field_info     fields[fields_count];
}
```

​	字段表也是一个变长的结构，fields_count表示field的数量，接下来的fields表示字段集合，共有fields_count个，每一个字段用field_info结构表示，稍后会进行介绍。

**1.字段field_info结构**

​	每个字段field_info的格式如下所示。

```c
field_info {
    u2             access_flags; 
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

​	字段结构分为4个部分：第一部分access_flags表示字段的访问标记，用来标识是public、private还是protected，是否是static，是否是final等；第二部分name_index用来表示字段名，指向常量池的字符串常量；第三部分descriptor_index是字段描述符的索引，指向常量池的字符串常量；最后的attributes_count、attribute_info表示属性的个数和属性集合。如图1-26所示。

![](https://pic.imgdb.cn/item/619f08df2ab3f51d91f5449c.jpg)

**2.字段访问标记**

![](https://pic.imgdb.cn/item/619f09192ab3f51d91f5668d.jpg)

​	如果在类中定义了字段public static final int DEFAULT_SIZE=128，编译后DEFAULT_SIZE字段在类文件中存储的访问标记值为0x0019，则它的访问标记为ACC_PUBLIC|ACC_STATIC|ACC_FINAL，表示它是一个public static final类型的变量，如图1-27所示。

![](https://pic.imgdb.cn/item/619f09712ab3f51d91f5b45d.jpg)

**3.字段描述符**

	字段描述符（field descriptor）用来表示某个field的类型，在JVM中定义一个int类型的字段时，类文件中存储的类型并不是字符串int，而是更精简的字母I。
根据类型的不同，字段描述符分为三大类。
	1）原始类型，byte、int、char、float等这些简单类型使用一个字符来表示，比如J对应long类型，B对应byte类型。
	2）引用类型使用L；的方式来表示，为了防止多个连续的引用类型描述符出现混淆，引用类型描述符最后都加了一个“；”作为结束，比如字符串类型String的描述符为“Ljava/lang/String；”。
	3）JVM使用一个前置的“[”来表示数组类型，如int[]类型的描述符为“[I”，字符串数组String[]的描述符为“[Ljava/lang/String；”。而多维数组描述符只是多加了几个“[”而已，比如`Object[][][]`类型的描述符为“[[[Ljava/lang/Object；”。
	完整的字段类型描述符映射表如表1-5所示。

![](https://pic.imgdb.cn/item/619f0a0e2ab3f51d91f640a1.jpg)

**4.字段属性**

​	与字段相关的属性包括ConstantValue、Synthetic、Signature、Deprecated、Runtime-Visible-Annotations和RuntimeInvisibleAnnotations这6个，比较常见的是ConstantValue属性，用来表示一个常量字段的值，具体将在1.2.8节展开介绍。

### 1.2.7 方法表

```java
{
    u2             methods_count;
    method_info    methods[methods_count];
}
```

​	其中methods_count表示方法的数量，接下来的methods表示方法的集合，共有methods_count个，每一个方法用method_info结构表示。

**1.方法method_info结构**

​	对于每个方法method_info而言，它的结构如下所示。

```java
method_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

​	方法method_info结构分为四部分：第一部分access_flags表示方法的访问标记，用来标记是public、private还是protected，是否是static，是否是final等；接下来的name_index、descriptor_index分别表示方法名和方法描述符的索引值，指向常量池的字符串常量；attributes_count和attribute_info表示方法相关属性的个数和属性集合，包含了很多有用的信息，比如方法内部的字节码就存放在Code属性中。

![](https://pic.imgdb.cn/item/619f39962ab3f51d9110dd15.jpg)

**2.方法访问标记**

​	方法的访问标记比类和字段的访问标记类型更丰富，一共有12种，完整的映射表如表1-6所示。

![](https://pic.imgdb.cn/item/619f3a112ab3f51d9111206f.jpg)

​	以下面的代码为例：

```java
private static synchronized void foo() {
}
```

​	生成的类文件中，foo方法的访问标记等于0x002a（ACC_PRIVATE|ACC_STATIC|ACC_SYNCHRONIZED），表示这是一个private static synchronized的方法，如图1-29所示。

![](https://pic.imgdb.cn/item/619f3a7e2ab3f51d91114749.jpg)

​	同前面的字段访问标记一样，不是所有的方法访问标记都可以随意组合设置，比如ACC_ABSTRACT、ACC_FINAL在方法描述符中不能同时设置，ACC_ABSTRACT和ACC_SYNCHRONIZED也不能同时设置。

**3.方法名与描述符**

​	紧随方法访问标记的是方法名索引name_index，指向常量池中CONSTANT_Utf8_info类型的字符串常量，比如有这样一个方法定义private void foo（），编译器会生成一个类型为CONSTANT_Utf8_info的字符串常量项，里面存储了“foo”，方法名索引name_index指向了这个常量项。

​	方法描述符索引descriptor_index也是指向常量池中类型为CONSTANT_Utf8_info的字符串常量项。方法描述符用来表示一个方法所需的参数和返回值，格式如下：

```
(参数1类型 参数2类型 参数3类型 ...)返回值类型
```

​	比如，方法Object foo（int i，double d，Thread t）的描述符为“（IDLjava/lang/Thread；）Ljava/lang/Object；”，其中“I”表示第一个参数i的参数类型int，“D”表示第二个参数d的类型double，“Ljava/lang/Thread；”表示第三个参数t的类型Thread，“Ljava/lang/Object；”表示返回值类型Object，如图1-30所示。

![](https://pic.imgdb.cn/item/619f3b072ab3f51d911179da.jpg)

**4.方法属性表**

​	方法属性表是method_info结构的最后一部分。前面介绍了方法的访问标记和方法签名，还有一些重要的信息没有出现，如方法声明抛出的异常，方法的字节码，方法是否被标记为deprecated等，属性表就是用来存储这些信息的。与方法相关的属性有很多，其中比较重要的是Code和Exceptions属性，其中Code属性存放方法体的字节码指令，Exceptions属性用于存储方法声明抛出的异常。属性的细节我们将在1.2.8节中进行介绍。

### 1.2.8 属性表

​	在方法表之后的结构是class文件的最后一部分——属性表。属性出现的地方比较广泛，不只出现在字段和方法中，在顶层的class文件中也会出现。相比于常量池只有14种固定的类型，属性表的类型更加灵活，不同的虚拟机实现厂商可以自定义属性，属性表的结构如下所示。

```java
{
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

​	与其他结构类似，属性表使用两个字节表示属性的个数attributes_count，接下来是若干个属性项的集合，可以看作是一个数组，数组的每一项都是一个属性项attribute_info，数组的大小为attributes_count。每个属性项的attribute_info的结构如下所示。

```java
attribute_info{
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

​	attribute_name_index是指向常量池的索引，根据这个索引可以得到attribute的名字，接下来的两部分表示info数组的长度和具体byte数组的内容。

​	虚拟机预定义了20多种属性，下面我们挑选字段表相关的ConstantValue属性和方法表相关的Code属性进行介绍。

**1.ConstantValue属性**

​	ConstantValue属性出现在字段field_info中，用来表示静态变量的初始值，它的结构如下所示。

```java
ConstantValue_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 constantvalue_index;
}
```

​	其中attribute_name_index是指向常量池中值为“ConstantValue”的字符串常量项，attribute_length值固定为2，因为接下来的具体内容只会有两个字节大小。constantvalue_index指向常量池中具体的常量值索引，根据变量的类型不同，constantvalue_index指向不同的常量项。如果变量为long类型，则constantvalue_index指向CONSTANT_Long_info类型的常量项。

​	以代码public static final int DEFAULT_SIZE=128为例，字段对应的class文件如图1-31高亮部分所示。

![](https://pic.imgdb.cn/item/619f3be92ab3f51d9111e121.jpg)

![](https://pic.imgdb.cn/item/619f3c062ab3f51d9111f10f.jpg)

**2.Code属性**

​	Code属性是类文件中最重要的组成部分，它包含方法的字节码，除native和abstract方法以外，每个method都有且仅有一个Code属性，它的结构如下。

```java
Code_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 max_stack;
    u2 max_locals;
    u4 code_length;
    u1 code[code_length];
    u2 exception_table_length;
    {   u2 start_pc;
        u2 end_pc;
        u2 handler_pc;
        u2 catch_type;
    } exception_table[exception_table_length];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```

下面开始介绍Code属性表的各个字段含义。
	1）属性名索引（attribute_name_index）占2个字节，指向常量池中CONSTANT_Utf8_info常量，表示属性的名字，比如这里对应的常量池的字符串常量“Code”。
	2）属性长度（attribute_length）占用2个字节，表示属性值长度大小。
	3）max_stack表示操作数栈的最大深度，方法执行的任意期间操作数栈的深度都不会超过这个值。它的计算规则是：有入栈的指令stack增加，有出栈的指令stack减少，在整个过程中stack的最大值就是max_stack的值，增加和减少的值一般都是1，但也有例外：LONG和DOUBLE相关的指令入栈stack会增加2，VOID相关的指令则为0。
	4）max_locals表示局部变量表的大小，它的值并不等于方法中所有局部变量的数量之和。当一个局部作用域结束，它内部的局部变量占用的位置就可以被接下来的局部变量复用了。
	5）code_length和code用来表示字节码相关的信息。其中，code_length表示字节码指令的长度，占用4个字节；code是一个长度为code_length的字节数组，存储真正的字节码指令。
	6）exception_table_length和exception_table用来表示代码内部的异常表信息，如我们熟知的try-catch语法就会生成对应的异常表。exception_table_length表示接下来exception_table数组的长度，	每个异常项包含四个部分，可以用下面的结构表示。

```java
{
    u2 start_pc;
    u2 end_pc;
    u2 handler_pc;
    u2 catch_type;
}
```

​	其中start_pc、end_pc、handler_pc都是指向code字节数组的索引值，start_pc和end_pc表示异常处理器覆盖的字节码开始和结束的位置，是左闭右开区间[start_pc，end_pc），包含start_pc，不包含end_pc。handler_pc表示异常处理handler在code字节数组的起始位置，异常被捕获以后该跳转到何处继续执行。

​	catch_type表示需要处理的catch的异常类型是什么，它用两个字节表示，指向常量池中类型为CONSTANT_Class_info的常量项。如果catch_type等于0，则表示可处理任意异常，可用来实现finally语义。

​	当JVM执行到这个方法[start_pc，end_pc）范围内的字节码发生异常时，如果发生的异常是这个catch_type对应的异常类或者它的子类，则跳转到code字节数组handler_pc处继续处理。

​	7）attributes_count和attributes[]用来表示Code属性相关的附属属性，Java虚拟机规定Code属性只能包含这四种可选属性：LineNumberTable、LocalVariableTable、LocalVariableTypeTable、StackMapTable。以LineNumberTable为例，LineNumberTable用来存放源码行号和字节码偏移量之间的对应关系，属于调试信息，不是类文件运行的必需属性，默认情况下都会生成。如果没有这个属性，那么在调试时就没有办法在源码中设置断点，也没有办法在代码抛出异常时在错误堆栈中显示出错的行号信息。

​	接下来以代码清单1-6为例来看Code属性。

​	代码清单1-6　Code属性代码示例

```java
public class HelloWorldMain {
    public static void main(String[] args) {
        try {
            foo();
        } catch (NullPointerException e) {
            System.out.println(e);
        } catch (IOException e) {
            System.out.println(e);
        }

        try {
            foo();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
    public static void foo() throws IOException {
    }
}
```

![](https://pic.imgdb.cn/item/619f3d582ab3f51d9112a10a.jpg)

​	其中attribute_name_index为0x0C，指向常量池中下标为12的字符串“Code”。attribute_length等于154（0x9A），表示属性值的长度大小。max_stack和max_locals都等于2，表示最大栈深度和局部变量表的大小都等于2，code_length等于40（0x28），表示接下来code字节数组的长度为40。exception_table_length等于3（0x03），表示接下来会有3个异常表项目。最后的attributes_count为2，表示接下来会有2个相关的属性项，这里是LineNumberTable和StackMapTable。根据前面的介绍，可以画出的Code属性结构如图1-34所示。

![](https://pic.imgdb.cn/item/619f3f932ab3f51d91137df8.jpg)

## 1.3 使用javap查看类文件

​	前面零星介绍了使用javap来查看类文件，本节将详细介绍javap命令。由前面的小节也可以看出class文件是二进制块，想直接与它打交道比较艰难，好在JDK提供了专门用来分析类文件的工具javap以窥探class文件内部的细节。它的使用方式如下：

```java
javap [options]  <classes>
```

​	不加任何参数运行javap的输出如下所示。

```java
javap HelloWorld

Compiled from "HelloWorld.java"
public class HelloWorld {
  public HelloWorld();
  public static void main(java.lang.String[]);
}
```

​	默认情况下，javap会显示访问权限为public、protected和默认级别的方法。如果想要显示private方法和字段，就需要加上-p选项。

​	javap还有一个好用的选项-s，可以输出类型描述符签名信息，HelloWorld.java所有的方法签名如下所示。

```java
javap -s HelloWorld

Compiled from "HelloWorld.java"
public class HelloWorld {
  public HelloWorld();
    descriptor: ()V

public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
}
```

​	加上-c选项可以对类文件进行反编译，可以显示出方法内的字节码，加上-c选项以后的输出如下所示。

```java
javap -c HelloWorld

public static void main(java.lang.String[]);
Code:
   0: getstatic     #2       // Field java/lang/System.out:Ljava/io/PrintStream;
   3: ldc           #3       // String hello, world
   5: invokevirtual #4       // Method java/io/PrintStream.println:(Ljava/lang/String;)V
   8: return
```

​	加上-v选项可以显示更加详细的内容，比如版本号、类访问权限、常量池相关的信息，是一个非常有用的参数，如下所示。

```java
javap -v HelloWorld 

public class HelloWorld
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref      #6.#15     // java/lang/Object."<init>":()V
   #2 = Fieldref       #16.#17    // java/lang/System.out:Ljava/io/PrintStream;
   ...
```

​	还有一个比较少用的-l选项，可以用来显示行号表和局部变量表，实测并没有输出局部变量表，只显示了行号表，如下所示。

```java
javap -l HelloWorld     

public static void main(java.lang.String[]);
LineNumberTable:
  line 3: 0
  line 4: 8
```

​	原因是要想显示局部变量表，需要在javac编译的时候加-g选项，生成所有的调试信息选项，加上-g选项编译javac-g HelloWorld.java以后重新执行javap-l命令就可以看到局部变量表（LocalVariableTable）了，如下所示。

```java
javap -l HelloWorld
public static void main(java.lang.String[]);
LineNumberTable:
  line 3: 0
  line 4: 8
LocalVariableTable:
  Start  Length  Slot  Name   Signature
      0       9     0  args   [Ljava/lang/String;
```

# 2. 字节码基础

* 基于寄存器和基于栈虚拟机实现的优缺点
* 字节码的分类
* 类型转换指令
* for循环的字节码实现
* switch-case的tableswitch和lookupswitch两种实现
* String的switch实现原理
* ++i与i++的字节码原理
* Java异常处理原理
* finally语句块一定会执行的原因
* try-with-resources语法糖背后的原理
* 对象创建、类初始化相关的字节码指令

## 2.1 字节码概述

​	Java虚拟机的指令由一个字节长度的操作码（opcode）和紧随其后的可选的操作数（operand）构成，如下所示。

```
<opcode> [<operand1>, <operand2>]
```

​	字节码使用大端序（Big-Endian）表示，即高位在前，低位在后的方式，比如字节码getfield 00 02，表示的是getfiled 0x00<<8|0x02（getfield#2）。

​	字节码并不是某种虚拟CPU的机器码，而是一种介于源码和机器码中间的一种抽象表示方法，不过字节码通过JIT（Just In Time）技术可以被进一步编译成机器码。

​	根据字节码的不同用途，字节码指令可以大概分为如下几类：

* ·加载和存储指令，比如iload将一个整型值从局部变量表加载到操作数栈；
* 控制转移指令，比如条件分支ifeq；
* 对象操作，比如创建类实例的指令new；
* 方法调用，比如invokevirtual指令用于调用对象的实例方法；
* 运算指令和类型转换，比如加法指令iadd；
* 线程同步，比如monitorenter和monitorexit这两条指令用于支持synchronized关键字的语义
* 异常处理，比如athrow显式抛出异常。

## 2.2 Java虚拟机栈和栈帧

​	虚拟机常见的实现方式有两种：基于栈（Stack based）和基于寄存器（Register based）。典型的基于栈的虚拟机有Hotspot JVM、.net CLR，而典型的基于寄存器的虚拟机有Lua语言虚拟机LuaVM和Google开发的Android虚拟机DalvikVM。

​	两者有什么不同呢？举一个计算两数相加的例子：c=a+b，Java源码如下所示。

```java
int my_add(int a, int b) {
    return a + b;
}
```

使用javap查看对应的字节，如下所示。

```java
0: iload_1 // 将 a 压入操作数栈
1: iload_2 // 将 b 压入操作数栈
2: iadd    // 将栈顶两个值出栈相加，然后将结果放回栈顶
3: ireturn // 将栈顶值返回
```

实现相同功能对应的lua代码如下。

```java
local function my_add(a, b)
    return a + b;
end
```

使用luac-l-l-v-s test.lua命令查看lua的字节码，如下所示。

```java
[1] ADD       R2 R0 R1     ; R2 := R0 + R1
[2] RETURN    R2 2         ; return R2
[3] RETURN    R0 1         ; return 
```

​	第1行调用ADD指令将R0寄存器和R1寄存器中的值相加存储到寄存器R2中。第2行返回R2寄存器的值。第3行是lua的一个特殊处理，为了防止有分支漏掉了return语句，lua始终在最后插入一行return语句。
​	以7+20为例，基于栈和基于寄存器的执行过程对比如图2-1所示。

![](https://pic.imgdb.cn/item/619f50472ab3f51d911e1be3.jpg)

​	基于栈和基于寄存器的指令集架构各有优缺点，具体如下所示。

* 基于栈的指令集架构的优点是移植性更好、指令更短、实现简单，但是不能随机访问堆栈中的元素，完成相同功能所需的指令数一般比寄存器架构多，需要频繁地入栈出栈，不利于代码优化。
* 基于寄存器的指令集架构的优点是速度快，可以充分利用寄存器，有利于程序做运行速度优化，但操作数需要显式指定，指令较长。

#### 栈帧

​	Hotspot JVM是一个基于栈的虚拟机，每个线程都有一个虚拟机栈用来存储栈帧，每次方法调用都伴随着栈帧的创建、销毁。Java虚拟机栈的释义如图2-2所示。

![](https://pic.imgdb.cn/item/619f508f2ab3f51d911e4f95.jpg)

​	当线程请求分配的栈容量超过Java虚拟机栈允许的最大容量时，Java虚拟机将会抛出StackOverflowError异常，可以用JVM命令行参数-Xss来指定线程栈的大小，比如-Xss：256k用于将栈的大小设置为256 KB。

​	每个线程都拥有自己的Java虚拟机栈，一个多线程的应用会拥有多个Java虚拟机栈，每个栈拥有自己的栈帧，如图2-3所示。

![](https://pic.imgdb.cn/item/619f51192ab3f51d911eacc4.jpg)

​	栈帧是用于支持虚拟机进行方法调用和方法执行的数据结构，随着方法调用而创建，随着方法结束而销毁。栈帧的存储空间分配在Java虚拟机栈中，每个栈帧拥有自己的局部变量表（Local Variable）、操作数栈（Operand Stack）和指向常量池的引用，如图2-4所示。

![](https://pic.imgdb.cn/item/619f513f2ab3f51d911eba8c.jpg)

**1.局部变量表**

​	每个栈帧内部都包含一组称为局部变量表的变量列表，局部变量表的大小在编译期间就已经确定，对应class文件中方法Code属性的max_locals字段，Java虚拟机会根据max_locals字段来分配方法执行过程中需要分配的最大的局部变量表容量。代码示例如下。

```java
public class MyJVMTest {
    public void foo(int id, String name) {
        String tmp = "A";
    }
}
```

​	使用javac-g MyJVMTest.java进行编译，然后执行javap-c-v-l MyJVMTest查看字节码，结果如下。

```java
public void foo(int, java.lang.String);
Code:
  stack=1, locals=4, args_size=3
     0: ldc           #2                  // String A
     2: astore_3
     3: return
  LocalVariableTable:
    Start  Length  Slot  Name   Signature
        0       4     0  this   LMyJVMTest;
        0       4     1    id   I
        0       4     2  name   Ljava/lang/String;
        3       1     3   tmp   Ljava/lang/String;
```

​	可以看到foo方法只有两个参数，args_size却等于3。当一个实例方法（非静态方法）被调用时，第0个局部变量是调用这个实例方法的对象的引用，也就是我们所说的this。调用方法foo（2019，"hello"）实际上是调用foo（this，2019，"hello"）。

​	LocalVariableTable输出显示了局部变量表的4个槽（slot），布局如表2-1所示。

![](https://pic.imgdb.cn/item/619f52ea2ab3f51d911fad7d.jpg)

​	javap输出中的locals=4表示局部变量表的大小等于4。局部变量表的大小并不是方法中所有局部变量的数量之和，它与变量的类型和变量作用域有关。当一个局部作用域结束，它内部的局部变量占用的位置就可以被接下来的局部变量复用了，以下面的静态foo方法为例。

```java
public static void foo() {
    // locals=0, max_locals=0
    if (true) {
        // locals=1, max_locals=1
        String a = "a";
    }
    // locals=0, max_locals=1
    if (true) {
        // locals=1, max_locals=1
        String b = "b";
    }
    // locals=0, max_locals=1
}
```

​	foo方法对应的局部变量表的大小等于1，因为是静态方法，局部变量表不用自动添加this为局部变量表的第一个元素，a和b共用同一个slot等于0的局部变量表位置。

​	当包含long和double类型的变量时，这些变量会占用两个局部变量表的slot，以下面的代码为例。

```java
public void foo() {
    double a = 1L;
    int b = 10;
}
```

![](https://pic.imgdb.cn/item/619f53372ab3f51d911fe8c8.jpg)

**2.操作数栈**

​	每个栈帧内部都包含一个称为操作数栈的后进先出（LIFO）栈，栈的大小同样也是在编译期间确定。Java虚拟机提供的很多字节码指令用于从局部变量表或者对象实例的字段中复制常量或者变量到操作数栈，也有一些指令用于从操作数栈取走数据、操作数据和把操作结果重新入栈。在方法调用时，操作数栈也用于准备调用方法的参数和接收方法返回的结果。

​	比如iadd指令用于将两个int型的数值相加，它要求执行之前操作数栈已经存在两个int型数值，在iadd指令执行时，两个int型数值从操作数栈中出栈，相加求和，然后将求和的结果重新入栈。1+2对应的指令执行过程，如图2-6所示。

![](https://pic.imgdb.cn/item/619f54692ab3f51d91208ffe.jpg)

​	整个JVM指令执行的过程就是局部变量表与操作数栈之间不断加载、存储的过程，如图2-7所示。

![](https://pic.imgdb.cn/item/619f54842ab3f51d9120a4aa.jpg)

​	那么，如何计算操作数栈的最大值？操作数栈容量最大值对应方法Code属性的max_stack，表示当前方法的操作数栈在执行过程中任何时间点的最大深度。调用一个成员方法会将this和所有参数入栈，调用完毕this和参数都会出栈。如果方法有返回值，会将返回值入栈。代码示例如下。

```java
public void foo() {
    bar(1, 1, 2);
}
public void bar(int a, int b, int c) {
}
```

​	foo方法的max_stack等于4，因为调用bar方法会将this、1、1、2这四个变量压栈到栈上，栈的深度为4，调用完后全部出栈。

​	如果bar方法后面再调用一个参数个数小于3的方法，比如下面代码中的bar1，foo方法的max_stack还是等于4，因为方法调用过程中，操作数栈的最大深度还是调用bar方法产生的。

```java
public void foo() {
    // stack=4, max_stack=4
    bar(1, 1, 2);
    // stack=2, max_stack=4
    bar1(1);
}

public void bar(int a, int b, int c) {
}
public void bar1(int a) {
}
```

​	如果bar方法后面再调用一个参数个数大于3的方法，比如下面代码中的bar2，会将this、1、2、3、4、5入栈，max_stack变为6。

```java
public void foo() {
    // stack=4, max_stack=4
    bar(1, 1, 2);
    // stack=2, max_stack=4
    bar1(1);
    // stack=6, max_stack=6
    bar2(1, 2, 3, 4, 5);
}
public void bar(int a, int b, int c) {
}
public void bar1(int a) {
}
public void bar2(int a, int b, int c ,int d, int e) {
}
```

​	计算stack的方式如下：遇到入栈的字节码指令，stack+=1或者stack+=2（根据不同的指令类型），遇到出栈的字节码指令，stack则相应减少，这个过程中stack的最大值就是max_stack，也就是javap输出的stack的值，计算过程的伪代码如下所示。

```java
push(Type t) {
    stack = stack + width(t);
    if (stack > max_stack) max_stack = stack;
}

pop(Type t) {
    stack = stack - width(t);
}
```

​	有了栈帧的概念，接下来理解字节码指令就容易很多了，从下一节开始，我们将分类型讲解字节码指令。

## 2.3 字节码指令

### 2.3.1　加载和存储指令

​	加载（load）和存储（store）相关的指令是使用得最频繁的指令，分为load类、store类、常量加载这三种。

​	1）load类指令是将局部变量表中的变量加载到操作数栈，比如iload_0将局部变量表中下标为0的int型变量加载到操作数栈上，根据不同的数据变量类型还有lload、fload、dload、aload这些指令，分别表示加载局部变量表中long、float、double、引用类型的变量。

​	2）store类指令是将栈顶的数据存储到局部变量表中，比如istore_0将操作数栈顶的元素存储到局部变量表中下标为0的位置，这个位置的元素类型为int，根据不同的数据变量类型还有lstore、fstore、dstore、astore这些指令。

​	3）常量加载相关的指令，常见的有const类、push类、ldc类。const、push类指令是将常量值直接加载到操作数栈顶，比如iconst_0是将整数0加载到操作数栈上，bipush 100是将int型常量100加载到操作数栈上。ldc指令是从常量池加载对应的常量到操作数栈顶，比如ldc#10是将常量池中下标为10的常量数据加载到操作数栈上。

​	为什么同是int型常量，加载需要分这么多类型呢？这是为了使字节码更加紧凑，int型常量值根据值n的范围，使用的指令按照如下的规则。

* 若n在[-1，5]范围内，使用iconst_n的方式，操作数和操作码加一起只占一个字节。比如iconst_2对应的十六进制为0x05。-1比较特殊，对应的指令是iconst_m1（0x02）。
* 若n在[-128，127]范围内，使用bipush n的方式，操作数和操作码一起只占两个字节。比如n值为100（0x64）时，bipush 100对应十六进制为0x1064。
* 若n在[-32768，32767]范围内，使用sipush n的方式，操作数和操作码一起只占三个字节，比如n值为1024（0x0400）时，对应的字节码为sipush 1024（0x110400）。
* 若n在其他范围内，则使用ldc的方式，这个范围的整数值被放在常量池中，比如n值为40000时，40000被存储到常量池中，加载的指令为ldc#i，i为常量池的索引值。

* 若n在其他范围内，则使用ldc的方式，这个范围的整数值被放在常量池中，比如n值为40000时，40000被存储到常量池中，加载的指令为ldc#i，i为常量池的索引值。

![](https://pic.imgdb.cn/item/619f57832ab3f51d9122649c.jpg)

### 2.3.2 操作数栈指令

​	常见的操作数栈指令有pop、dup和swap。pop指令用于将栈顶的值出栈，一个常见的场景是调用了有返回值的方法，但是没有使用这个返回值，比如下面的代码。

```java
public String foo() {
    return "";
}
public void bar() {
    foo();
}
```

​	对应字节码如下所示。

```java
0: aload_0
1: invokevirtual #13                 // Method foo:()Ljava/lang/String;
4: pop
5: return
```

​	第4行有一个pop指令用于弹出调用bar方法的返回值。
​	dup指令用来复制栈顶的元素并压入栈顶，后面讲到创建对象的时候会用到dup指令。swap用于交换栈顶的两个元素，如图2-8所示。

![](https://pic.imgdb.cn/item/619f59c52ab3f51d91239a96.jpg)

​	还有几个稍微复杂一点的栈操作指令：dup_x1、dup2_x1和dup2_x2。下面以dup_x1为例来讲解。dup_x1是复制操作数栈栈顶的值，并插入栈顶以下2个值，看起来很绕，把它拆开来看其实分为了五步，如图2-9所示。

```java
v1 = stack.pop(); // 弹出栈顶的元素，记为 v1
v2 = stack.pop(); // 再次弹出栈顶的元素，记为 v2
state.push(v1);   // 将 v1 入栈
state.push(v2);   // 将 v2 入栈
state.push(v1);   // 再次将 v1 入栈
```

![](https://pic.imgdb.cn/item/619f59f12ab3f51d9123b788.jpg)

```java
public class Hello {
    private int id;
    public int incAndGetId() {
        return ++id;
    }
}
```

incAndGetId方法对应的字节码如下。

```java
public int incAndGetId();
     0: aload_0
     1: dup
     2: getfield      #2                  // Field id:I
     5: iconst_1
     6: iadd
     7: dup_x1
     8: putfield      #2                  // Field id:I
    11: ireturn
```

​	假如id的初始值为42，调用incAndGetId方法执行过程中操作数栈的变化如图2-10所示。

![](https://pic.imgdb.cn/item/619f5b732ab3f51d91248d3d.jpg)

​	第0行：aload_0将this加载到操作数栈上。
​	第1行：dup指令将复制栈顶的this，现在操作数栈上有两个this，栈上的元素是[this，this]。
​	第2行：getfield#2指令将42加载到栈上，同时将一个this出栈，栈上的元素变为[this，42]。

​	第5行：iconst_1将常量1加载到栈上，栈中元素变为[this，42，1]。
​	第6行：iadd将栈顶的两个值出栈相加，并将结果43放回栈上，现在栈中的元素是[this，43]。
​	第7行：dup_x1将栈顶的元素43插入this之下，栈中元素变为[43，this，43]。
​	第8行：putfield#2将栈顶的两个元素this和43出栈，现在栈中元素只剩下栈顶的[43]，最后的ireturn指令将栈顶的43出栈返回。

![](https://pic.imgdb.cn/item/619f5bbc2ab3f51d9124b287.jpg)

### 2.3.3 运算和类型转换指令

![](https://pic.imgdb.cn/item/61a1b6a52ab3f51d9105b82b.jpg)

​	如果需要进行运算的数据类型不一样，会涉及类型转换（cast），比如下面的浮点数1.0与整数1相加的运算。

```java
1.0 + 1
```

​	按照直观的想法，加法操作对应的字节码指令如下所示。

```java
fconst_1 // 将 1.0 入栈
iconst_1 // 将 1 入栈
fadd
```

​	但fadd指令值只支持对两个float类型的数据做相加操作，为了支持这种运算，JVM会先把两个数据类型转换为一样，但精度可能出问题。为了能将1.0和1相加，int型数据需要转为float型数据，然后调用fadd指令进行相加，如下面的代码所示。

```java
fconst_1 // 将 1.0 入栈
iconst_1 // 将 1 入栈
i2f      // 将栈顶的 1 的 int 转为 float
fadd     // 两个 float 值相加
```

​	虽然在Java语言层面，boolean、char、byte、short是不同的数据类型，但是在JVM层面它们都被当作int来处理，不需要显式转为int，字节码指令上也没有对应转换的指令。

​	有多种类型数据混合运算时，系统会自动将数据转为范围更大的数据类型，这种转换被称为宽化类型转换（widening）或自动类型转换，如图2-11所示。

![](https://pic.imgdb.cn/item/61a1b70b2ab3f51d9105d4c2.jpg)

​	自动类型转换并不意味着不丢失精度，比如下面代码中将int值“123456789”转为float就出现了精度丢失的情况。

```java
int n = 123456789;
float f = n; // f = 1.23456792E8
```

​	相对的，如果把大范围数据类型的数据强制转换为小范围数据类型，这种转换称为窄化类型转换（narrowing），比如把long转为int，double转为float，如图2-12所示。

![](https://pic.imgdb.cn/item/61a1b7382ab3f51d9105e034.jpg)

![](https://pic.imgdb.cn/item/61a1b7422ab3f51d9105e2f3.jpg)

### 2.3.4 控制转移指令

​	控制转移指令用于有条件和无条件的分支跳转，常见的if-then-else、三目表达式、for循环、异常处理等都属于这个范畴。对应的指令集包括：

* 条件转移：ifeq、iflt、ifle、ifne、ifgt、ifge、ifnull、ifnonnull、if_icmpeq、if_icmpne、if_icmplt，if_icmpgt、if_icmple、if_icmpge、if_acmpeq和if_acmpne。
* 复合条件转移：tableswitch、lookupswitch。
* 无条件转移：goto、goto_w、jsr、jsr_w、ret。

下面代码中的isPositive方法为例，它的作用是判断一个整数是否为正数。

```java
public int isPositive(int n) {
    if (n > 0) {
        return 1;
    } else {
        return 0;
    }
}
```

​	对应的字节码如下所示。

```java
0: iload_1
1: ifle 6
4: iconst_1  
5: ireturn  
6: iconst_0 
7: ireturn
```

​	根据我们之前的分析，isPositive方法局部变量表的大小为2，第一个元素是this，第二个元素是参数n，接下来逐行解释上面的字节码。

​	第0行：iload_1的作用是将局部变量表中下标为1的整型变量加载到操作数栈上，也就是加载参数n。其中iload_1中的i表示要加载的变量是一个int类型。同时注意到iload_1后面跟了一个数字1，它们的作用都是把栈顶元素存入局部变量表的下标为1的位置，它属于iload_<i>指令组，其中i只能是0、1、2、3。其实把iload_1写成iload 1也能获取正确的结果，但是编译的字节码会变长，在字节码执行时也需要获取和解析1这个额外的操作数。
​	第1行：ifle指令的作用是将操作数栈顶元素出栈跟0进行比较，如果小于等于0则跳转到特定的字节码处，如果大于0则继续执行接下来的字节码。ifle可以看作“if less or equal”的缩写，比较的值是0。如果想要比较的值不是0，需要用新的指令if_icmple表示“if int compare less or equal xx”。
​	第4～5行：对应代码“return 1；”，iconst_1指令的作用是把常量1加载到操作数栈上，ireturn指令的作用是将栈顶的整数1出栈返回，方法调用结束。
​	第6～7行：对应代码“return 0；”，第6行iconst_0指令的作用是将常量0加载到操作数栈上，ireturn指令的作用是将栈顶的整数1出栈返回，方法调用结束。
假设n等于20，调用isPositive方法操作数栈的变化情况如图2-13所示。
​	控制转移指令完整的列表如表2-6所示。

![](https://pic.imgdb.cn/item/61a1b8622ab3f51d910633fd.jpg)

![](https://pic.imgdb.cn/item/61a1b89f2ab3f51d9106491a.jpg)

### 2.3.5　for语句的字节码原理

​	纵观所有的字节码指令，并没有与for名字相关的指令，那for循环是如何实现的呢？接下来以sum相加求和的例子来看for循环的实现细节，代码如下所示。

```java
public int sum(int[] numbers) {
    int sum = 0;
    for (int number : numbers) {
        sum += number;
    }
    return sum;
}
```

```java
 0: iconst_0
 1: istore_2
 2: aload_1
 3: astore_3
 4: aload_3
 5: arraylength
 6: istore        4
 8: iconst_0
 9: istore        5
11: iload         5
13: iload         4
15: if_icmpge     35
18: aload_3
19: iload         5
21: iaload
22: istore        6
24: iload_2
25: iload         6
27: iadd
28: istore_2
29: iinc          5, 1
32: goto          11
35: iload_2
36: ireturn
```

![](https://pic.imgdb.cn/item/61a1b9b82ab3f51d9106acf3.jpg)

​	第0～1行：把常量0加载到操作数栈上，随后通过istore_2指令将0出栈赋值给局部变量表下标为2的元素，也就是给局部变量sum赋值为0，如图2-15所示。

![](https://pic.imgdb.cn/item/61a1b9e92ab3f51d9106bb22.jpg)

​	第2～3行：aload_1指令的作用是加载局部变量表中下标为1的变量（参数numbers），astore_3指令的作用是将栈顶元素存储到局部变量下标为3的位置上，记为$array，如图2-16所示。

![](https://pic.imgdb.cn/item/61a1ba7e2ab3f51d9106f70f.jpg)

​	第4～6行：计算数组的长度，astore_3加载`$array`到栈顶，调用arraylength指令获取数组长度存储到栈顶，随后调用istore 4将数组长度存储到局部变量表的第4个位置，这个变量是表示数组的长度值，记为`$len`，过程如图2-17所示。

![](https://pic.imgdb.cn/item/61a1bab22ab3f51d91070669.jpg)

​	第8～9行：初始化数组遍历的下标初始值。iconst_0将0加载到操作数栈上，随后istore 5将栈顶的0存储到局部变量表中的第5个位置，这个局部变量是数组遍历的下标初始值，记为$i，如图2-18所示。

![](https://pic.imgdb.cn/item/61a1bacd2ab3f51d91070c5f.jpg)

​	11～32行是真正的循环体，详细介绍如下。

​	第11～15行的作用是判断循环能否继续。这部分的字节码如下所示。

```java
11: iload         5
13: iload         4
15: if_icmpge     35
```

​	首先通过iload 5和iload 4指令加载$i和$len到栈顶，然后调用if_icmpge进行比较，如果$i>=$len，直接跳转到第35行指令处，for循环结束；如果$i<$len则继续往下执行循环体，可以用如下伪代码表示。

```java
if ($i >= $len) goto 35;
```

​	过程如图2-19所示。

![](https://pic.imgdb.cn/item/61a1bb502ab3f51d91073a55.jpg)	第18～22行的作用是把$array[$i]赋值给number。aload_3加载$array到栈上，iload 5加载$i到栈上，然后iaload指令把下标为$i的数组元素加载到操作数栈上，随后istore 6将栈顶元素存储到局部变量表下标为6的位置上，过程如图2-20所示。

![](https://pic.imgdb.cn/item/61a1bb9d2ab3f51d9107556f.jpg)

​	第24～28行：iload_2和iload 6指令把sum和number值加载到操作数栈上，然后执行iadd指令进行整数相加，过程如图2-21所示。
​	第29行：“iinc 5，1”指令对执行循环后的$i加一。iinc指令比较特殊，之前介绍的指令都是基于操作数栈来实现功能，它则是直接对局部变量进行自增，不用先入栈、执行加一操作，再将结果出栈存储到局部变量表，因此效率非常高，适合循环结构，如图2-22所示。
​	第32行：goto 11指令的作用是跳转到第11行继续进行循环条件的判断。

![](https://pic.imgdb.cn/item/61a1bbb82ab3f51d910762bf.jpg)

![](https://pic.imgdb.cn/item/61a1bbda2ab3f51d91076e59.jpg)

```java
@start: if ($i >= $len) return;
$item = $array[$i];
sum += $item;
++ $i
goto @start
```

​	整段代码的逻辑看起来非常熟悉，可以用下面的Java代码表示。

```java
int sum = 0;
for (int i = 0; i < numbers.length; i++) {
    sum += numbers[i];
}
return sum;
```

​	由此可见，for（item：array）就是一个语法糖，字节码会让它现出原形，回归它的本质。

### 2.3.6　switch-case底层实现原理

​	如果让我们来实现一个switch-case语法，会如何做呢？是通过一个个if-else语句来判断吗？这样明显效率非常低。通过分析switch-case的字节码，可以知道编译器使用了tableswitch和lookupswitch两条指令来生成switch语句的编译代码。为什么会有两条指令呢？这是基于效率的考量，接下来进行详细分析。代码示例如下。

```java
int chooseNear(int i) {
    switch (i) {
        case 100: return 0;
        case 101: return 1;
        case 104: return 4;
        default: return -1;
    }
}
```

对应的字节码如下所示。

```java
: iload_1
1: tableswitch   { // 100 to 104
         100: 36
         101: 38
         102: 42
         103: 42
         104: 40
     default: 42
}

42: iconst_m1
43: i
```

​	细心的同学会发现，代码的case中并没有出现102、103，但字节码中却出现了。原因是编译器会对case的值做分析，如果case的值比较“紧凑”，中间有少量断层或者没有断层，会采用tableswitch来实现switch-case；如果case值有大量断层，会生成一些虚假的case帮忙补齐，这样可以实现O（1）时间复杂度的查找。case值已经被补齐为连续的值，通过下标就可以一次找到，这部分伪代码如下所示。

```java
int val = pop();                // pop an int from the stack
if (val < low || val > high) {  // if its less than <low> or greater than <high>,
    pc += default;              // branch to default 
} else {                        // otherwise
    pc += table[val - low];     // branch to entry in table
}
```

​	再来看一个case值断层严重的例子，代码如下所示。

```java
0: iload_1
1: lookupswitch  { // 3
           1: 36
          10: 38
         100: 41
     default: 44
}
```

​	如果还是采用前面tableswitch补齐的方式，就会生成上百个假case项，class文件会爆炸式增长，这种做法显然不合理。为了解决这个问题，可以使用lookupswitch指令，它的键值都是经过排序的，在查找上可以采用二分查找的方式，时间复杂度为O（log n）。

​	从上面的介绍可以知道，switch-case语句在case比较“稀疏”的情况下，编译器会使用lookupswitch指令来实现，反之，编译器会使用tableswitch来实现。我们在第4章会介绍编译器是如何来判断case值的稀疏程度的。

### 2.3.7　String的switch-case实现的字节码原理

​	前面我们已经知道switch-case依据case值的稀疏程度，分别由两个指令——tableswitch和lookupswitch实现，但这两个指令都只支持整型值，那编译器是如何让String类型的值也支持switch-case的呢？本节我们将介绍这背后的实现细节，以下面的代码为例。

```java
public int test(String name) {
    switch (name) {
        case "Java":
            return 100;
        case "Kotlin":
            return 200;
        default:
            return -1;
    }
}
```

​	对应的字节码如下所示。

```java
 0: aload_1
 1: astore_2
 2: iconst_m1
 3: istore_3
 
 4: aload_2
 5: invokevirtual #2                  // Method java/lang/String.hashCode:()I
 8: lookupswitch  { // 2
     -2041707231: 50 // 对应 "Kotlin".hashCode()
         2301506: 36 // 对应 "Java".hashCode()
         default: 61
    }
    
36: aload_2
37: ldc           #3                  // String Java
39: invokevirtual #4                  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
42: ifeq          61
45: iconst_0
46: istore_3
47: goto          61

50: aload_2
51: ldc           #5                  // String Kotlin
53: invokevirtual #4                  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
56: ifeq          61
59: iconst_1
60: istore_3

61: iload_3
62: lookupswitch  { // 2
               0: 88
               1: 91
         default: 95
    }
    
// 88 ~ 90
88: bipush        100
90: ireturn

91: sipush        200
94: ireturn

95: iconst_m1
96: ireturn
```

​	为了方便理解，这里先画出了局部变量表的布局图，如图2-23所示。
​	第0～3行：做初始化操作，把入参name赋值给局部变量表下标为2的变量，记为tmpName，初始化局部变量表中位置为3的变量为-1，记为matchIndex。

![](https://pic.imgdb.cn/item/61a61dc62ab3f51d910d0d15.jpg)

​	第4～8行：调用tmpName的hashCode方法，得到一个整型值。因为哈希值一般都比较离散，所以没有选用tableswitch而是用lookupswitch来作为switch-case的实现。

​	第36～47行：如果hashCode等于字符串"Java"的hashCode会跳转到第36行继续执行。首先调用字符串的equals方法进行比较，看是否相等。判断是否相等使用的指令是ifeq，它的含义是如果等于0则跳转到对应字节码行处，实际上是等于false时跳转。这里如果相等则把matchIndex赋值为0。

​	第61～96行：进行最后的case分支执行。这一段比较好理解，不再继续分析。

​	结合上面的字节码解读，可以推演出对应的Java代码实现，如代码清单2-1所示。

​	代码清单2-1　String的switch-case等价实现代码

```java
public int test_translate(String name) {
    String tmpName = name;
    int matchIndex = -1;
    switch (tmpName.hashCode()) {
        case -2041707231:
            if (tmpName.equals("Kotlin")) {
                matchIndex = 1;
            }
            break;
        case 2301506:
            if (tmpName.equals("Java")) {
                matchIndex = 0;
            }
            break;
        default:
            break;
    }
    switch (matchIndex) {
        case 0:
            return 100;
        case 1:
            return 200;
        default:
            return -1;
    }
}
```

​	看到这里细心的读者可能会问，字符串的hashCode冲突时要怎样处理，比如“Aa”和“BB”的hashCode都是2112。以下面的代码为例，学习case值hashCode相同时编译器是如何处理的。

```java
public int testSameHash(String name) {
    switch (name) {
        case "Aa":
            return 100;
        case "BB":
            return 200;
        default:
            return -1;
    }
}
```

对应的字节码如代码清单2-2所示。
代码清单2-2　相同hashCode值的String switch-case字节码

```java
public int testSameHash(java.lang.String);
descriptor: (Ljava/lang/String;)I
flags: ACC_PUBLIC
Code:
  stack=2, locals=4, args_size=2
     0: aload_1
     1: astore_2
     2: iconst_m1
     3: istore_3
     
     4: aload_2
     5: invokevirtual #2         // Method java/lang/String.hashCode:()I
     8: lookupswitch  { // 1
                2112: 28
             default: 53
        }
        
    28: aload_2
    29: ldc           #3         // String BB
    31: invokevirtual #4         // Method java/lang/String.equals:(Ljava/lang/Object;)Z
    34: ifeq          42
    37: iconst_1
    38: istore_3
    39: goto          53
    
    42: aload_2
    43: ldc           #5         // String Aa
    45: invokevirtual #4         // Method java/lang/String.equals:(Ljava/lang/Object;)Z
    48: ifeq          53
    51: iconst_0
    52: istore_3
    
    53: iload_3
    54: lookupswitch  { // 2
                   0: 80
                   1: 83
             default: 87
        }
    80: bipush        100
    82: ireturn
    83: sipush        200
    86: ireturn
    87: iconst_m1
    88: ireturn
```

​	可以看到34行在hashCode冲突的情况下，编译器的处理不过是多一次调用字符串equals判断相等的比较。与BB不相等的情况，会继续判断是否等于Aa，翻译为Java源代码如代码清单2-3所示。

```java
public int testSameHash_translate(String name) {
    String tmpName = name;
    int matchIndex = -1;

    switch (tmpName.hashCode()) {
        case 2112:
            if (tmpName.equals("BB")) {
                matchIndex = 1;
            } else if (tmpName.equals("Aa")) {
                matchIndex = 0;
            }
            break;
        default:
            break;
    }

    switch (matchIndex) {
        case 0:
            return 100;
        case 1:
            return 200;
        default:
            return -1;
    }
}
```

​	前面介绍了String的swich-case实现，里面用到了字符串的hashCode方法，那如何快速构造两个hashCode相同的字符串呢？这要从hashCode的源码说起，String类hashCode的代码如代码清单2-4所示。

​	代码清单2-4　String的hashCode源码

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

​	假设要构造的字符串只有两个字符，用“ab”和“cd”，上面的代码就变成了如果两个hashCode相等，则满足下面的公式。

```java
a * 31 + b = c * 31 + d
31*(a-c)=d-b
```

​	其中一个特殊解是a-c=1，d-b=31，也就是只有两个字符的字符串“ab”与“cd”满足a-c=1，d-b=31，这两个字符串的hashCode就一定相等，比如“Aa”和“BB”，“Ba”和“CB”，“Ca”和“DB”，依次类推。

### 2.3.8 ++i和i++的字节码原理

​	在面试中经常会被问到++i与i++相关的陷阱题，关于i++和++i的区别，我自己在刚学编程时是比较困惑的，下面我们从字节码的角度来看++i与i++到底是如何实现的。
​	首先来看一段i++的陷阱题，如代码清单2-5所示。
​	代码清单2-5　i++代码示例

```java
public static void foo() {
    int i = 0;
    for (int j = 0; j < 50; j++) {
        i = i++;
    }
    System.out.println(i);
}
```

​	执行上述代码输出结果是0，而不是50，源码“i=i++；”对应的字节码如下所示。

```java
...
10: iload_0
11: iinc          0, 1
14: istore_0
...
```

​	第10行：iload_0把局部变量表slot=0的变量（i）加载到操作数栈上。
​	第11行：“iinc 0，1”对局部变量表slot=0的变量（i）直接加1，但是这时候栈顶的元素没有变化，还是0。
​	第14行：istore_0将栈顶元素出栈赋值给局部变量表slot=0的变量，也就是i。此时，局部变量i又被赋值为0，前面的iinc指令对i的加一操作被覆盖，如图2-24所示。

![](https://pic.imgdb.cn/item/61a621ae2ab3f51d910fdecb.jpg)

可以用下面的伪代码来表示i=i++的执行过程。

```java
tmp = i;
i = i + 1;
i = tmp;
```

​	因此可以得知，“j=i++；”在字节码层面是先把i的值加载到操作数栈上，随后才对局部变量i执行加一操作，留在操作数栈顶的还是i的旧值。如果把栈顶值赋值给j，则这个变量得到的是i加一之前的值。
​	把上面的代码稍作修改，将i++改为++i，如代码清单2-6所示。
​	代码清单2-6　++i代码示例

```java
public static void foo() {
    int i = 0;
    for (int j = 0; j < 50; j++)
        i = ++i;
    System.out.println(i);
}
```

代码对应的字节码如下所示。

```java
...
10: iinc          0, 1
13: iload_0
14: istore_0
...
```

​	i=++i对应的字节码还是第10～14行，可以看出“i=++i；”先对局部变量表下标为0的变量加1，然后才把它加载到操作

​	数栈上，随后又从操作数栈上出栈赋值给局部变量表中下标为0的变量。

​	整个过程的局部变量表和操作数栈的变化如图2-25所示。

![](https://pic.imgdb.cn/item/61a621fa2ab3f51d911020b4.jpg)

​	i=++i可以用下面的伪代码来表示。

```java
i = i + 1;
tmp = i;
i = tmp;
```

​	因此可以得知，“j=++i；”实际上是对局部变量i做了加一操作，然后才把最新的i值加载到操作数上，随后赋值给变量j。

​	再来看一道难一点的题目，完整的代码如下所示。

```java
public static void bar() {
    int i = 0;
    i = i++ + ++i;
    System.out.println("i=" + i);
}
```

​	对应的字节码如下。

```java
 0: iconst_0
 1: istore_0
 
 2: iload_0
 3: iinc          0, 1
 
 6: iinc          0, 1
 9: iload_0
10: iadd
11: istore_0
```

![](https://pic.imgdb.cn/item/61a623202ab3f51d91110f07.jpg)

​	整个过程可以用下面的伪代码来表示。

```java
i = 0;

tmp1 = i;
i = i + 1;

i = i + 1
tmp2 = i;

tmpSum = tmp1 + tmp2;

i = tmpSum;
```

### 2.3.9 try-catch-finally的字节码原理

​	Java中有一个非常重要的内容是try-catch-finally的执行顺序和返回值问题，大部分书里说过finally一定会执行，但是为什么是这样？下面来看看try-catch-finally这个语法糖背后的实现原理。

**1.try-catch字节码分析**

​	下面是一个简单的try-catch的例子。

```java
public class TryCatchFinallyDemo {
    public void foo() {
        try {
            tryItOut1();
        } catch (MyException1 e) {
            handleException(e);
        }
    }
}
```

​	对应的字节码如下所示。

```java
 0: aload_0
 1: invokevirtual #2        // Method tryItOut1:()V
 4: goto          13
 7: astore_1
 8: aload_0
 9: aload_1
10: invokevirtual #4        // Method handleException:(Ljava/lang/Exception;)V
13: return
Exception table:
 from    to  target type
     0     4     7   Class MyException1
```

​	第0～1行：aload_0指令加载this，随后使用invokevirtual指令调用tryItOut1方法，关于invokevirtual的详细用法在第3章会介绍，这里只需要知道invokevirtual是方法调用指令即可。

​	第4行：goto语句是如果tryItOut1方法不抛出异常就会跳转到第13行继续执行return指令，方法调用结束。如果有异常抛出，将如何处理呢？

​	从第1章的内容可以知道，当方法包含try-catch语句时，在编译单元生成的方法的Code属性中会生成一个异常表（Exception table），每个异常表项表示一个异常处理器，由from指针、to指针、target指针、所捕获的异常类型type四部分组成。这些指针的值是字节码索引，用于定位字节码。其含义是在[from，to）字节码范围内，如果抛出了异常类型为type的异常，就会跳转到target指针表示的字节码处继续执行。

​	上面例子中的Exception table表示，在0到4之间（不包含4）如果抛出了类型为MyException1或其子类异常，就跳转到7继续执行。

​	值得注意的是，当抛出异常时，Java虚拟机会自动将异常对象加载到操作数栈栈顶。
​	第7行：astore_1将栈顶的异常对象存储到局部变量表中下标为1的位置。
​	第8～10行：aload_0和aload_1分别加载this和异常对象到栈上，最后执行invokevirtual#4指令调用handleException方法。
​	异常处理逻辑的操作数栈和局部变量表变化过程如图2-27所示。
​	下面我们来看在有多个catch语句的情况下，虚拟机是如何处理的，以代码清单2-7为例。

```java
public void foo() {
    try {
        tryItOut2();
    } catch (MyException1 e) {
        handleException1(e);
    } catch (MyException2 e) {
        handleException2(e);
    }
}
```

​	对应字节码如下。

```java
 0: aload_0
 1: invokevirtual #5 // Method tryItOut2:()V
 4: goto          22
 // 第一个 catch 部分内容
 7: astore_1
 8: aload_0
 9: aload_1
10: invokevirtual #6 // Method handleException1:(Ljava/lang/Exception;)V
13: goto          22
// 第二个 catch 部分内容
16: astore_1
17: aload_0
18: aload_1
19: invokevirtual #8 // Method handleException2:(Ljava/lang/Exception;)V
22: return

Exception table:
from    to  target type
   0     4     7   Class MyException1
   0     4    16   Class MyException2
```

​	可以看到，多一个catch语句处理分支，异常表里面就会多一条记录，当程序出现异常时，Java虚拟机会从上至下遍历异常表中所有的条目。当触发异常的字节码索引值在某个异常条目的[from，to）范围内，则会判断抛出的异常是否是想捕获的异常或其子类。

​	如果异常匹配，Java虚拟机会将控制流跳转到target指向的字节码继续执行；如果不匹配，则继续遍历异常表。如果遍历完所有的异常表还未找到匹配的异常处理器，那么该异常将继续抛到调用方（caller）中重复上述的操作。

![](https://pic.imgdb.cn/item/61a623e72ab3f51d91117bee.jpg)

**2.finally字节码分析**

​	很多Java学习资料中都有写：finally语句块保证一定会执行。这一句简单的规则背后却不简单，之前我一直以为finally的实现是用简单的跳转来实现的，实际上并非如此。接下来我们一步步分析finally语句的底层原理，以代码清单2-8为例。

​	代码清单2-8　finally语句示例

```java
public void foo() {
    try {
        tryItOut1();
    } catch (MyException1 e) {
        handleException(e);
    } finally {
        handleFinally();
    }
}
```

​	对应的字节码如下所示。

```java
 0: aload_0
 1: invokevirtual #2 // Method tryItOut1:()V
 //  添加 finally 语句块
 4: aload_0
 5: invokevirtual #9 // Method handleFinally:()V
 8: goto          31
 ---
11: astore_1
12: aload_0
13: aload_1
14: invokevirtual #4 // Method handleException:(Ljava/lang/Exception;)V
//  添加 finally 语句块
17: aload_0
18: invokevirtual #9 // Method handleFinally:()V
21: goto          31
---
24: astore_2
25: aload_0
26: invokevirtual #9 // Method handleFinally:()V
29: aload_2
30: athrow
31: return
Exception table:
  from    to  target type
     0     4    11   Class MyException1
     0     4    24   any
    11    17    24   any
```

​	可以看到字节码中出现了三次调用handleFinally方法的invokevirtual#9，都是在程序正常return和异常throw之前，其中两处在try-catch语句调用return之前，一处是在异常抛出throw之前。
​	第0～3行：执行tryItOut1方法。如果没有异常，就继续执行handleFinally方法；如果有异常，则根据异常表的映射关系跳转到对应的字节码处执行。
​	第11~14行：执行catch语句块中的handleException方法，如果没有异常就继续执行handleFinally方法，如果有异常则跳转到第24行继续执行。
​	第24～30行：负责处理tryItOut1方法抛出的非MyException1异常和handleException方法抛出的异常。
​	不用finally语句，只用try-catch语句实现的等价代码如代码清单2-9所示。
​	代码清单2-9　finally语句等价实现

```java
public void foo() {
    try {
        tryItOut1();
        handleFinally();
    } catch (MyException1 e) {
        try {
            handleException(e);
        } catch (Throwable e2) {
            handleFinally();
            throw e2;
        }
    } catch (Throwable e) {
        handleFinally();
        throw e;
    }
}
```

​	由代码可知，现在的Java编译器采用复制finally代码块的方式，并将其内容插入到try和catch代码块中所有正常退出和异常退出之前。这样就解释了我们一直以来被灌输的观点，finally语句块一定会执行。

​	有了上面的基础，就很容易理解在finally语句块中有return语句会发生什么。因为finally语句块插入在try和catch返回指令之前，finally语句块中的return语句会“覆盖”其他的返回（包括异常），以代码清单2-10为例。
​	代码清单2-10　finally语句块中有return语句的情况

```java
public int foo() {
    try {
        int a = 1 / 0;
        return 0;
    } catch (Exception e) {
        int b = 1 / 0;
        return 1;
    } finally {
        return 2;
    }
}
```

```java
...
 8: astore_1
 9: iconst_1
10: iconst_0
11: idiv
12: istore_2
13: iconst_1
14: istore_3
15: iconst_2
16: ireturn
...

Exception table:
 from    to  target type
     0     6     8   Class java/lang/Exception
     0     6    17   any
     8    15    17   any
    17    19    17   any
```

​	第8～12行字节码相当于源码中的“int b=1/0；”，第13行字节码iconst_1将整型常量值1加载到栈上，第14行字节码istore_3将栈顶的数值1暂存到局部变量中下标为3的元素中，第15行字节码iconst_2将整型常量2加载到栈上，第16行随后调用ireturn将其出栈值返回，方法调用结束。
​	可以看到，受finally语句return的影响，虽然catch语句中有“return 1；”，在字节码层面只是将1暂存到临时变量中，没有机会执行返回，本例中foo方法的返回值为2。
​	上面的代码在语义上与下面的代码清单2-11等价。
​	代码清单2-11　finally语句包含return的等价实现

```java
public int foo() {
    try {
        int a = 1 / 0;
        int tmp = 0;
        return 2;
    } catch (Exception e) {
        try {
            int b = 1 / 0;
            int tmp = 1;
            return 2;
        } catch (Throwable e1) {
            return 2;
        }
    } catch (Throwable e) {
        return 2;
    }
}
```

​	接下来看看，在finally语句中修改return的值会发生什么，可以想想代码清单2-12中foo方法返回值是100还是101。
​	代码清单2-12　finally语句修改了return值

```java
public int foo() {
    int i = 100;
    try {
        return i;
    } finally {
        ++i;
    }
}
```

​	前面介绍过，在finally语句中包含return语句时，会将返回值暂存到临时变量中，这个finally语句中的++i操作只会影响i的值，不会影响已经暂存的临时变量的值。foo返回值为100。foo方法对应的字节码如下所示。

```java
 0: bipush        100
 2: istore_1
 3: iload_1
 4: istore_2
 5: iinc          1, 1
 8: iload_2
 9: ireturn
10: astore_3
11: iinc          1, 1
14: aload_3
15: athrow

Exception table:
 from    to  target type
     3     5    10   any
```

​	第0～2行的作用是初始化i值为100，bipush 100的作用是将整数100加载到操作数栈上。第3～4行的作用是加载i值并将其存储到局部变量表中位置为2的变量中，这个变量在源代码中是没有出现的，是return之前的暂存值，记为tmpReturn，此时tmpReturn的值为100。第5行的作用是对局部变量表中i直接自增加一，这次自增并不会影响局部变量tmpReturn的值。第8～9行加载tmpReturn的值并返回，方法调用结束。第10～15行是第3～4行抛出异常时执行的分支。

​	整个过程如图2-28所示。

![](https://pic.imgdb.cn/item/61a626042ab3f51d911291e9.jpg)

类似的陷阱题如代码清单2-13所示。
代码清单2-13　finally陷阱题

```java
public String foo() {
    String s = "hello";
    try {
        return s;
    } finally {
        s = null;
    }
}
```

对应的字节码如下所示。

```java
 0: ldc           #2                  // String hello
 2: astore_1
 3: aload_1
 4: astore_2
 5: ldc           #3                  // String xyz
 7: astore_1
 8: aload_2
 9: areturn
10: astore_3
11: ldc           #3                  // String xyz
13: astore_1
14: aload_3
15: athrow
```

​	可以看到，第0～2行字节码加载字符串常量“hello”的引用赋值给局部变量s。第3～4行将局部变量s的引用加载到栈上，随后赋值给局部变量表中位置为2的元素，这个变量在代码中并不存在，是一个临时变量，这里记为tmp。第5～7行将字符串常量“xyz”的引用加载到栈上，随后赋值给局部变量s。第8~9行加载局部变量tmp到栈顶，tmp指向的依旧是字符串“hello”，随后areturn将栈顶元素返回。上述过程如图2-29所示。

![](https://pic.imgdb.cn/item/61a6263f2ab3f51d9112b07d.jpg)

```java
public String foo() {
    String s = "hello";
    String tmp = s;
    s = "xyz";
    return tmp;
}
```

