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

### 2.3.10 try-with-resources的字节码原理

​	try-with-resources是Java7 Project Coin提案中引入的新的资源释放机制，Project Coin的提交者声称JDK源码中close用法在释放资源时存在bug。try-with-resources的出现既可以减少代码出错的概率，也可以使代码更加简洁。下面以代码清单2-14为例开始介绍这一小节的内容。

```java
public static void foo() throws IOException {
    try (FileOutputStream in = new FileOutputStream("test.txt")) {
        in.write(1);
    }
}
```

​	在不用try-with-resources的情况下，我们很容易写出下面这种try-finally包裹起来的看似等价的代码。

​	看起来好像没有什么问题，但是仔细想一下，如果in.write（）抛出了异常，in.close（）也抛出了异常，调用者会收到哪个呢？我们回顾一下Java基础中try-catch-finally的内容，以代码清单2-15中的bar方法为例。

```java
public static void bar() {
    try {
        throw new RuntimeException("in try");
    } finally {
        throw new RuntimeException("in finally");
    }
}
```

​	调用bar（）方法会抛出的异常如下所示。

```java
Exception in thread "main" java.lang.RuntimeException: in finally
```

​	也就是说，try中抛出的异常被finally抛出的异常淹没了，这也很好理解，从上一节介绍的内容可知finally中的代码块会在try抛出异常之前插入，即try抛出的异常被finally抛出的异常捷足先登先返回了

​	因此在上面foo方法中in.write（）和in.close（）都抛出异常的情况下，调用方收到的是in.close（）抛出的异常，in.write（）抛出的重要异常消失了，这往往不是我们想要的，那么怎样在抛出try中的异常同时又不丢掉finally中的异常呢？

​	接下来，我们来学习try-with-resources是怎么解决这个问题的。使用javap查看foo方法的字节码，部分输出如代码清单2-16所示。

​	代码清单2-16　try-with-resources字节码

```java
...
17: aload_0
18: invokeinterface #4,  1  // InterfaceMethod java/lang/AutoCloseable.close:()V
23: goto          86
...
26: astore_2
27: aload_1
28: aload_2
29: invokevirtual #6     // Method java/lang/Throwable.addSuppressed:(Ljava/lang/Throwable;)V
32: goto          86
...
86: return
Exception table:
    from    to  target  type
      17    23    26    Class java/lang/Throwable
       6     9    44    Class java/lang/Throwable
       6     9    49    any
      58    64    67    Class java/lang/Throwable
      44    50    49    any
```

​	可以看到，第29行出现了一个源代码中并没有出现的Throwable.addSuppressed方法调用，接下来我们来看这里面的玄机。

​	Java 7中为Throwable类增加了addSuppressed方法。当一个异常被抛出的时候，可能有其他异常因为该异常而被抑制，从而无法正常抛出，这时可以通过addSuppressed方法把被抑制的异常记录下来，这些异常会出现在抛出的异常的堆栈信息中；也可以通过getSuppressed方法来获取这些异常。这样做的好处是不会丢失任何异常，方便开发人员进行调试。
​	根据上述概念，对代码进行再次改写，如代码清单2-17所示。

```java
public static void foo() throws Exception {
    FileOutputStream in = null;
    Exception tmpException = null;
    try {
        in = new FileOutputStream("test.txt");
        in.write(1);
    } catch (Exception e) {
        tmpException = e;
        throw e;
    } finally {
        if (in != null) {
            if (tmpException != null) {
                try {
                    in.close();
                } catch (Exception e) {
                    tmpException.addSuppressed(e);
                }
            } else {
                in.close();
            }
        }
    }
}
```

​	上面的代码中如果in.close（）发生异常，这个异常不会覆盖原来的异常，只是放到原异常的Suppressed异常中。

​	本节介绍了try-with-resources语句块的底层字节码实现，一起来回顾一下要点：第一，try-with-resources语法并不是简单地在finally中加入closable.close（）方法，因为finally中的close方法如果抛出了异常会淹没真正的异常；第二，引入了Suppressed异常，既可以抛出真正的异常又可以调用addSuppressed附带上suppressed的异常。

### 2.3.11 对象相关的字节码指令

**`1.<init>方法`**

​	`<init>`方法是对象初始化方法，类的构造方法、非静态变量的初始化、对象初始化代码块都会被编译进这个方法中。比如：

```java
 public class Initializer {
    // 初始化变量
    private int a = 10; 
    // 构造器方法
    public Initializer() {
        int c = 30;
    }
    // 对象初始化代码块
    {
        int b = 20;
    }
}
```

​	对应的字节码为：

```java
public Initializer();
descriptor: ()V
flags: ACC_PUBLIC
Code:
  stack=2, locals=2, args_size=1
     0: aload_0
     1: invokespecial #1          // Method java/lang/Object."<init>":()V
     4: aload_0
     5: bipush        10
     7: putfield      #2          // Field a:I
    10: bipush        20
    12: istore_1
    13: bipush        30
    15: istore_1
    16: return
```

​	javap输出的字节码中Initializer（）方法对应`<init>`对象初始化方法，其中5～7行将成员变量a赋值为10，10～12行将b赋值为10，13～15行将c赋值为30。

​	可以看到，虽然Java语法上允许我们把成员变量初始化和初始化语句块写在构造器方法之外，最终在编译以后都会统一编译进`<init>`方法。为了加深印象，可以来看一个在变量初始化可能抛出异常的情况，如下面的代码所示。

```java
public class Hello {
    private FileOutputStream outputStream = new FileOutputStream("test.txt");

    public Hello()  {
    }
}
```

​	编译上面的代码会报如下的错误。

```java
javac Hello.java 
Hello.java:8: error: unreported exception FileNotFoundException; must be caught or declared to be thrown
    private FileOutputStream outputStream = new FileOutputStream("test.txt");
                                            ^
```

​	为了能使上面的代码编译通过，需要在默认构造器方法抛出FileNotFoundException异常，如下面的代码所示。

```java
public class Hello {
    private FileOutputStream outputStream = new FileOutputStream("test.txt");

    public Hello() throws FileNotFoundException {
    }
}
```

**2.new、dup、invokespecial对象创建三条指令**

​		在Java中new是一个关键字，在字节码中也有一个叫new的指令，但是两者不是一回事。当我们创建一个对象时，背后发生了哪些事情呢？以下面的代码为例。

```java
ScoreCalculator calculator = new ScoreCalculator();
```

​	对应的字节码如下所示。

```java
0: new           #2                  // class ScoreCalculator
3: dup
4: invokespecial #3                  // Method ScoreCalculator."<init>":()V
7: astore_1
```

​	一个对象创建需要三条指令，new、dup、`<init>`方法的invokespecial调用。在JVM中，类的实例初始化方法是`<init>`，调用new指令时，只是创建了一个类实例引用，将这个引用压入操作数栈顶，此时还没有调用初始化方法。使用invokespecial调用`<init>`方法后才真正调用了构造器方法，那中间的dup指令的作用是什么？

​	invokespecial会消耗操作数栈顶的类实例引用，如果想要在invokespecial调用以后栈顶还有指向新建类对象实例的引用，就需要在调用invokespecial之前复制一份类对象实例的引用，否则调用完`<init>`方法以后，类实例引用出栈以后，就再也找不回刚刚创建的对象引用了。有了栈顶的新建对象的引用，就可以使用astore指令将对象引用存储到局部变量表，如图2-30所示。

![](https://pic.imgdb.cn/item/61aac4e62ab3f51d91060590.jpg)

​	从本质上来理解导致必须要有dup指令的原因是`<init>`方法没有返回值，如果`<init>`方法把新建的引用对象作为返回值，也不会存在这个问题。

**3.`<clinit>`方法**

```java
public class Initializer {
    private static int a = 0;

    static {
        System.out.println("static");
    }
}
```

​	对应的字节码如下所示。

```java
static {};
descriptor: ()V
flags: ACC_STATIC
Code:
  stack=2, locals=0, args_size=0
     0: iconst_0
     1: putstatic     #2     // Field a:I
     4: getstatic     #3     // Field java/lang/System.out:Ljava/io/PrintStream;
     7: ldc           #4     // String static
     9: invokevirtual #5     // Method java/io/PrintStream.println:(Ljava/lang/String;)V
    12: return
```

​	javap输出字节码中的static{}表示`<clinit>`方法。``<clinit>``不会直接被调用，它在四个指令触发时被调用（new、getstatic、putstatic和invokestatic），比如下面的场景：

* 创建类对象的实例，比如new、反射、反序列化等；
* 访问类的静态变量或者静态方法；
* 访问类的静态字段或者对静态字段赋值（final的字段除外）；
* 初始化某个类的子类。

# 3. 字节码进阶

​	上一章我们介绍了常见的字节码指令，这一章我们来探究字节码的进阶知识，通过阅读本章你会学到以下内容。

* 5条方法调用指令的联系和区别
* JVM方法分派机制与vtable、itable原理
* 通过HSDB来查看JVM运行时数据
* invokedynamic指令的介绍及在Lambda表达式中的作用
* 从字节码角度理解泛型擦除
* 反射的底层实现原理

## 3.1 方法调用指令

​	JVM的方法调用指令都以invoke开头，这5条指令如下所示。

* invokestatic：用于调用静态方法。
* invokespecial：用于调用私有实例方法、构造器方法以及使用super关键字调用父类的实例方法等。
* invokevirtual：用于调用非私有实例方法。
* invokeinterface：用于调用接口方法。
* invokedynamic：用于调用动态方法。



### 3.1.1 invokestatic指令

​	invokestatic用来调用静态方法，也就是使用static关键字修饰的方法。它要调用的方法在编译期间确定，且运行期不会修改，属于静态绑定。调用invokestatic不需要将对象加载到操作数栈，只需要将所需要的参数入栈就可以执行invokestatic指令了。例如Integer.valueOf（"42"）对应字节码如下所示。

```java
0: ldc           #2      // String 42
2: invokestatic  #3      // Method java/lang/Integer.valueOf:(Ljava/lang/String;) Ljava/lang/Integer;
```

### 3.1.2 invokevirtual指令

​	invokevirtual指令用于调用普通实例方法，它调用的目标方法在运行时才能根据对象实际的类型确定，在编译期无法知道，类似于C++中的虚方法。

​	在调用invokevirtual指令之前，需要将对象引用、方法参数入栈，调用结束对象引用、方法参数都会出栈，如果方法有返回值，返回值会入栈到栈顶。比如file.toString（）对应的字节码如下所示。

```java
10: aload_1
11: invokevirtual #5      // Method java/io/File.toString:()Ljava/lang/String;
```

​	下面以一个实际的例子来讲解invokevirtual指令的用法，如代码清单3-1所示。
​	代码清单3-1　invokevirtual示例

```java
package invoketest;
class Color {
    public void printColorName()  {
        System.out.println("Color name from parent");
    }
}
class Red extends Color {
    @Override
    public void printColorName() {
        System.out.println("Color name is Red");
    }
}
class Yellow extends Color {
    @Override
    public void printColorName() {
        System.out.println("Color name is Yellow");
    }
}
public class InvokeVirtualTest {
    public static void main(String[] args) {
        Color yellowColor = new Yellow();
        Color redColor = new Red();
        printColorName(yellowColor);
        printColorName(redColor);
    }
    public static void foo(Color color) {
        color.printColorName();
    }
}
```

​	foo方法的字节码如下所示。

```java
0: aload_0
1: invokevirtual #7        // Method invoketest/Color.printColorName:()V
4: return
```

​	foo方法使用invokevirtual指令调用Color类的printColorName方法，但它们最终调用的目标方法却不同，可能是Yellow类的printColorName方法，也有可能是Red类的printColorName方法。invokevirtual根据对象的实际类型进行分派（虚方法分派），在运行时动态选择执行具体子类的实现方法。

### 3.1.3 invokespecial指令

​	invokespecial，顾名思义，它用来调用“特殊”的实例方法，包括如下三种：

* 实例构造器方法`<init>`；
* private修饰的私有实例方法；
* 使用super关键字调用的父类方法。



​	比如new File（"test.txt"）对应的字节码如下所示，invokespecial用来调用File类的构造器方法`<init>`。

```java
0: new           #2     // class java/io/File
3: dup
4: ldc           #3     // String test.txt
6: invokespecial #4     // Method java/io/File."<init>":(Ljava/lang/String;)V
```

​	看到这里细心的读者可能会想为什么有了invokevirtual指令还需要invokespecial指令呢？这是出于效率的考虑，invokespecial调用的方法可以在编译期间确定，在JDK 1.0.2之前，invokespecial指令曾被命名为invokenonvirtual，以区别于invokevirtual。例如private方法不会因为继承被子类覆写，在编译期间就可以确定，所以private方法的调用使用invokespecial指令。

### 3.1.4 invokeinterface指令

​	invokeinterface用于调用接口方法，同invokevirtual一样，也是需要在运行时根据对象的类型确定目标方法，以下面的代码为例。

```java
private AutoCloseable autoCloseable;
public void foo() throws Exception {
    autoCloseable.close();
}
```

​	foo方法对应的字节码如下所示。

```java
0: aload_0
1: getfield      #2       // Field autoCloseable:Ljava/lang/AutoCloseable;
4: invokeinterface #3,  1 // InterfaceMethod java/lang/AutoCloseable.close:()V
```

​	那它与invokevirtual有什么区别，为什么不用invokevirtual来实现接口方法的调用呢？这就要说到Java方法分派的实现原理。

**1.方法分派原理**

​	Java的设计受到很多C++的影响，方法分派的思路参考了C++的实现，下面我们先来看C++中虚方法的实现。

​	当C++类中包含虚方法时，编译器会为这个类生成一个虚方法表（vtable），每个类都有一个指向虚方法表的指针vptr。虚方法表是方法指针的数组，用来实现多态。这里先来看看C++单继承的场景，新建一个main.cpp文件，如代码清单3-2所示。

```c++
class A {
 public:
  virtual void method1();
  virtual void method2();
  virtual void method3();
};

void A::method1() { std::cout << "method1 in A" << std::endl; }
void A::method2() { std::cout << "method2 in A" << std::endl; }
void A::method3() { std::cout << "method3 in A" << std::endl; }

class B : public A {
 public:
  void method2() override;
  virtual void method4();
  void method5();
};
void B::method2() { std::cout << "method2 in B" << std::endl; }
void B::method4() { std::cout << "method4 in B" << std::endl; }
void B::method5() { std::cout << "method5 in B" << std::endl; }
```

​	在命令行中使用g++-std=c++11-fdump-class-hierarchy test.cpp会输出A和B的虚方法表，输出结果如下所示。

```c++
Vtable for A
A::_ZTV1A: 5u entries
0     (int (*)(...))0
8     (int (*)(...))(& _ZTI1A)
16    (int (*)(...))A::method1
24    (int (*)(...))A::method2
32    (int (*)(...))A::method3

Vtable for B
B::_ZTV1B: 6u entries
0     (int (*)(...))0
8     (int (*)(...))(& _ZTI1B)
16    (int (*)(...))A::method1
24    (int (*)(...))B::method2
32    (int (*)(...))A::method3
40    (int (*)(...))B::method4
```

​	vtable除了包含虚方法以外，还包含了两个额外元素，这里暂时不用关心。重点看offset 16开始的虚方法。可以看到在单继承的情况下，子类B的虚方法的顺序与父类A保持一致，B类虚方法表中覆写方法method2指向B的实现，B新增的方法method4按顺序添加到虚方法表的末尾。

​	单继承的方法分派非常简单，比如有对象A*a调用method2方法时，如下所示。

```java
A *a
a->method2
```

​	我们并不知道a指针所指向对象的真正类型，不确定它是A类还是B类抑或是其他A的子类，但可以确定每个method2方法都被放在虚函数表的offset为24的位置上，不会随类型的影响而不同。

​	在C++的单继承中，这种虚函数的方式实现非常高效。Java类只支持单继承，在实现上与C++的虚方法表非常类似，也是使用了一个名为vtable的结构，如代码清单3-3所示。

```java
class A {
    public void method1() { }
    public void method2() { }
    public void method3() { }
}

class B extends A {
    public void method2() { } // overridden from BaseClass
    public void method4() { }
}
```

![](https://pic.imgdb.cn/item/61aacb0c2ab3f51d9109dafb.jpg)

​	可以看到B类的虚方法表保留了父类A中虚方法表的顺序，只是覆盖了method2指向的方法链接和新增了method4。假设这时需要调用method2方法，invokevirtual只需要直接去找虚方法表位置为2的方法引用就可以了。

​	Java的单继承看起来是规避了C++多继承带来的复杂性，但支持实现多个接口与多继承没有本质上的区别，下面来看看Java是如何实现的。

​	除了虚方法表vtable，JVM提供了名为itable（interface method table）的结构来支持多接口实现，itable由偏移量表（offset table）和方法表（method table）两部分组成。itable组成结构在hotspot源码的注释如下所示。

```java
// Format of an itable
//
//    ---- offset table ---
//    Klass* of interface 1             \
//    offset to vtable from start of oop  / offset table entry
//    ...
//    Klass* of interface n             \
//    offset to vtable from start of oop  / offset table entry
//    --- vtable for interface 1 ---
//    Method*                             \
//    compiler entry point                / method table entry
//    ...
//    Method*                             \
//    compiler entry point                / method table entry
//    -- vtable for interface 2 ---
//    ...
//
```

​	在需要调用某个接口方法时，虚拟机会在itable的offset table中查找到对应的方法表位置和方法位置，随后在method table中查找具体的方法实现。hotspot这部分源代码如代码清单3-4所示。

![](https://pic.imgdb.cn/item/61aacb512ab3f51d910a0e28.jpg)

```c++
InstanceKlass* int2 = (InstanceKlass*) rcvr->klass();
itableOffsetEntry* ki = (itableOffsetEntry*) int2->start_of_itable();
int i;
for ( i = 0 ; i < int2->itable_length() ; i++, ki++ ) {
    if (ki->interface_klass() == iclass) break;
}
// If the interface isn't found, this class doesn't implement this
// interface.  The link resolver checks this but only for the first
// time this interface is called.
if (i == int2->itable_length()) {
  VM_JAVA_ERROR(vmSymbols::java_lang_IncompatibleClassChangeError(), "");
}
int mindex = cache->f2_as_index();
itableMethodEntry* im = ki->first_method_entry(rcvr->klass());
callee = im[mindex].method();
if (callee == NULL) {
  VM_JAVA_ERROR(vmSymbols::java_lang_AbstractMethodError(), "");
}

istate->set_callee(callee);
istate->set_callee_entry_point(callee->from_interpreted_entry());
```

​	有了itable的知识，接下来看看invokevirtual和invokeinterface指令的区别。前面介绍过invokevirtual的实现依赖于Java的单继承特性，子类的虚方法表保留了父类虚方法表的顺序，但是因为Java的多接口实现，这一特性无法使用。以下面的代码清单3-5为例。

​	代码清单3-5　invokeinterface示例

```java
interface A {
    void method1();
    void method2();
}
interface B {
    void method3();
}
class D implements A, B {
    @Override
    public void method1() { }
    @Override
    public void method2() { }
    @Override
    public void method3() { }
}
class E implements B {
    @Override
    public void method3() { }
}
```

![](https://pic.imgdb.cn/item/61aacba92ab3f51d910a53fd.jpg)

​	当有下面这样的调用时：

```java
public void foo(B b) {
    b.method3();
}
```

​	D类中method3在itable的第三个位置，E类中method3在itable的第一个位置，如果要用invokevirtual调用method3就不能直接从固定的索引位置取得对应的方法。只能搜索整个itable来找到对应方法，使用invokeinterface指令进行调用。

**2.使用HSDB探究多态的原理**

​	HSDB全称是Hotspot Debugger，它是一个内置的JVM工具，可以用来深入分析JVM运行时的内部状态。HSDB位于JDK安装目录下的lib/sa-jdi.jar中，启动HSDB的方式如下所示。

```java
sudo java -cp sa-jdi.jar sun.jvm.hotspot.HSDB
```

​	执行上面的命令会弹出如图3-4的界面。

![](https://pic.imgdb.cn/item/61aacc5a2ab3f51d910ad0d0.jpg)

​	在File菜单中可以选择“attach到一个Hotspot JVM进程”“打开一个core文件”或者“连接到一个远程的debug server”。“attach到一个JVM进程”是最常用的选项。获取进程号可以用系统自带的ps命令，也可以用jps命令。在弹出的输入框中输入进程号以后默认展示当前线程列表，如图3-5所示。

![](https://pic.imgdb.cn/item/61aacc732ab3f51d910ae0bf.jpg)

![](https://pic.imgdb.cn/item/61aacc802ab3f51d910aea7a.jpg)

```java
public abstract class A {
    public void printMe() {
        System.out.println("I love vim");
    }
    public abstract void sayHello();
}
public class B extends A {
    @Override
    public void sayHello() {
        System.out.println("hello, i am child B");
    }
}

public class MyTest {
    public static void main(String[] args) throws IOException {
        A obj = new B();
        System.in.read()
        System.out.println(obj);
    }
}
```

​	运行MyTest，在命令行中执行jps找到MyTest的进程ID，这里为97169。

```java
jps
97169 MyTest
```

​	在HSDB的界面中选择File→Attach to Hotspot process，输入进程号，然后选择Tools→Class Browser可以找到对象列表，其中B对象的内存指针地址如下所示。

​	B@0x00000007c0060418

​	随后选择Tools→Inspector输入B的内存指针地址，可以显示B类的对象布局，如图3-7所示。

​	可以看到它的vtable的长度为7。有5个是上帝类Object的5个可以被继承的方法，1个是B覆写的sayHello方法，1个是继承A的printMe方法，接下来我们来验证这个结论。

​	vtable分配在instanceKlass对象实例的内存末尾，instanceKlass在64位系统的大小为0x1B8，因此vtable的起始地址等于instanceKlass的内存首地址加上0x1B8等于0x00000007C00605D0，如下所示。

```java
0x00000007c0060418 + 0x1B8 = 0x00000007C00605D0
```

![](https://pic.imgdb.cn/item/61aacd7f2ab3f51d910b9a07.jpg)

​	在HSDB的console输入mem查看实际的内存分布。mem命令需要两个参数，一个是起始地址，另一个是长度。输入mem 0x7C00605D0 7可以查看vtable内存起始地址开始的7个方法指针地址，如图3-8所示。

![](https://pic.imgdb.cn/item/61aacd912ab3f51d910ba4d0.jpg)

​	可以看到vtable的前5条与java.lang.Object下面的这5个方法一一对应，vtable里存储的是指向方法内存的指针。

```java
void finalize()
boolean equals(java.lang.Object) 
java.lang.String toString()
int hashCode()
java.lang.Object clone()
```

​	我们继续看剩下的两个方法地址，如图3-9所示

![](https://pic.imgdb.cn/item/61aacdc62ab3f51d910bc602.jpg)



* vtable、itable机制是实现Java多态的基础。
* 子类会继承父类的vtable。因为Java类都会继承Object类，Object中有5个方法可以被继承，所以一个空Java类的vtable的大小也等于5。
* 被final和static修饰的方法不会出现在vtable中，因为没有办法被继承重写，同理可以知道private修饰的方法也不会出现在vtable中。
* 接口方法的调用使用invokeinterface指令，Java使用itable来支持多接口实现，itable由offset table和method table两部分组成。在调用接口方法时，会先在offset table中查找method table的偏移量位置，随后在method table查找具体的接口实现。

![](https://pic.imgdb.cn/item/61aace102ab3f51d910bf2d0.jpg)



### 3.1.5 invokedynamic指令

​	Java虚拟机的指令集从1.0开始到JDK7之间的十余年间没有新增任何指令，这期间基于JVM的语言百花齐放，出现了JRuby、Groovy、Scala等很多运行在JVM的语言。因为JVM有诸多的限制，大部分情况下这些非Java的语言需要很多额外的调教才能在JVM上高效运行。随着JDK7的发布，字节码指令集新增了一个重量级指令invokedynamic，这个指令为多语言在JVM上的实现提供了技术支撑。这个小节我们来介绍invokedynamic指令背后的原理。

​	JDK7虽然在指令集中新增了这个指令，但是javac并不会生成invokedynamic指令，直到JDK8 Lambda表达式的出现，在Java中才第一次用上了这个指令。

​	对于JVM而言，不管是什么语言都是强类型语言，它会在编译时检查传入参数的类型和返回值的类型，比如下面这段代码。

```java
obj.println("hello world"); 
```

​	Java语言中，对应字节码如下所示。

```java
Constant pool:
#1 = Methodref      #6.#15      // java/lang/Object."<init>":()V
#4 = Methodref      #19.#20     // java/io/PrintStream.println:(Ljava/lang/String;)V
   
0: getstatic     #2             // Field java/lang/System.out:Ljava/io/PrintStream;
3: ldc           #3             // String Hello World
5: invokevirtual #4       // Method java/io/PrintStream.println:(Ljava/lang/String;)V
```

​	可以看到println方法要求如下。

* 要调用的对象类需要是java.io.PrintStream
* 方法名必须是println
* 方法参数必须是（String）
* 函数返回值必须是void



​	如果当前类找不到符合条件的函数，会在其父类中继续查找，如果obj所属的类与PrintStream没有继承关系，就算obj所属的类有符合条件的函数，上述调用也不会成功，因为类型检查不会通过。

​	但相同的代码在Groovy或者JavaScript等语言中表现不一样，无论obj是何种类型，只要所属类包含函数名为println，函数参数为（String）的函数，那么上述调用就会成功。这也是我们下面要讲到的鸭子类型。

​	鸭子类型概念的名字来源于由James Whitcomb Riley提出的鸭子测试，可以这样表述：当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。在鸭子类型中，关注点在于对象的行为，能做什么，而不是关注对象所属的类型，不关注对象的继承关系。

​	以下面这段Groovy脚本为例。

```java
class Duck {
    void fly() { println "duck flying" }
}


class Airplane {
    void fly() { println "airplane flying" }
}

class Whale {
    void swim() { println "Whale swim" }
}

def liftOff(entity) { entity.fly() }

liftOff(new Duck())
liftOff(new Airplane())
liftOff(new Whale())
```

​	输出如下所示：

```java
duck flying
airplane flying
groovy.lang.MissingMethodException: No signature of method: Whale.fly() is applicable for argument types: () values: []
```

​	可以看到liftOff方法调用了一个传入对象的fly方法，但是它并不知道这个对象的类型，也不知道这个对象是否包含了fly方法。

​	开始讲解invokedynamic之前需要先介绍一个核心的概念方法句柄（MethodHandle）。MethodHandle又称为方法句柄或方法指针，是java.lang.invoke包中的一个类，它的出现使得Java可以像其他语言一样把函数当作参数进行传递。MethodHandle类似于反射中的Method类，但它比Method类要更加灵活和轻量级。下面以一个实际的例子来看MethodHandle的用法，如下所示。

```java
public class Foo {
    public void print(String s) {
        System.out.println("hello, " + s);
    }
    public static void main(String[] args) throws Throwable {
        Foo foo = new Foo();

        MethodType methodType = MethodType.methodType(void.class, String.class);
        MethodHandle methodHandle = MethodHandles.lookup().findVirtual(Foo.class, "print", methodType);
        methodHandle.invokeExact(foo, "world");
    }
}
```

​	可以看到liftOff方法调用了一个传入对象的fly方法，但是它并不知道这个对象的类型，也不知道这个对象是否包含了fly方法

​	开始讲解invokedynamic之前需要先介绍一个核心的概念方法句柄（MethodHandle）。MethodHandle又称为方法句柄或方法指针，是java.lang.invoke包中的一个类，它的出现使得Java可以像其他语言一样把函数当作参数进行传递。MethodHandle类似于反射中的Method类，但它比Method类要更加灵活和轻量级。下面以一个实际的例子来看MethodHandle的用法，如下所示。

```java
public class Foo {
    public void print(String s) {
        System.out.println("hello, " + s);
    }
    public static void main(String[] args) throws Throwable {
        Foo foo = new Foo();

        MethodType methodType = MethodType.methodType(void.class, String.class);
        MethodHandle methodHandle = MethodHandles.lookup().findVirtual(Foo.class, "print", methodType);
        methodHandle.invokeExact(foo, "world");
    }
}
```

​	运行上面的代码就可以看到输出了“hello，world”。使用MethodHandle方法的步骤如下所示。

​	1）创建MethodType对象，MethodType用来表示方法签名，每个MethodHandle都有一个MethodType实例，用来指定方法的返回值类型和各个参数类型。

​	2）调用MethodHandles.lookup静态方法返回MethodHandles.Lookup对象，这个对象表示查找的上下文，根据方法的不同类型通过findStatic、findSpecial、findVirtual等方法查找方法签名为MethodType的方法句柄。

​	3）拿到方法句柄以后就可以调用具体的方法了，通过传入目标方法的参数，使用invoke或者invokeExact进行方法的调用。

​	前面介绍的4条invoke*指令的方法分派规则固化在虚拟机中，invokedynamic则把如何查找目标方法的决定权从虚拟机下放到具体的用户代码中。JRuby作者就给invokedynamic下过一个定义：“Invokedynamic is a user-definable bytecode，You decide how the JVM implements it”，说的也是这个道理。

​	invokedynamic指令的调用流程如下。

* JVM首次执行invokedynamic指令时会调用引导方法（Bootstrap Method）。
* 引导方法返回一个CallSite对象，CallSite内部根据方法签名进行目标方法查找。它的getTarget方法返回方法句柄（MethodHandle）对象。
* 在CallSite没有变化的情况下，MethodHandle可以一直被调用，如果CallSite有变化，重新查找即可。以def add（a，b）{a+b}为例，如果在代码中一开始使用两个int入参进行调用，那么极有可能后面很多次调用还会继续使用两个int，这样就不用每次都重新选择目标方法了。



​	它们之间的关系如图3-12所示。

![](https://pic.imgdb.cn/item/61aad17e2ab3f51d910e1d4f.jpg)

​	接下来我们来介绍invokedynamic在Groovy语言上的应用，新建一个Test.groovy文件，源码如下所示。

```java
def add(a, b) {
    new Exception().printStackTrace()
    return a + b
}

add("hello", "world")
```

​	默认情况下invokedynamic在Groovy中并未启用，需要加上--indy选项启用。使用groovy--indy Test.groovy编译上面的groovy代码，会生成Test.class文件，它对应的字节码如代码清单3-7所示。

```java
public java.lang.Object run();
descriptor: ()Ljava/lang/Object;
flags: ACC_PUBLIC
Code:
  stack=3, locals=1, args_size=1
     0: aload_0
     1: ldc           #44                 // String hello
     3: ldc           #46                 // String world
     5: invokedynamic #52,  0             // InvokeDynamic #1:invoke:(LTest;Ljava/lang/String;Ljava/lang/String;)Ljava/lang/Object;
    10: areturn
------------------  
Constant pool:
#1 = Utf8               Test
...省略掉部分字节码...
#52 = InvokeDynamic      #1:#51     // #1:invoke:(LTest;Ljava/lang/String;Ljava/lang/String;)Ljava/lang/Object;
------------------    
BootstrapMethods:
...省略掉部分字节码...
1: #34 invokestatic org/codehaus/groovy/vmplugin/v7/IndyInterface.bootstrap: (Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/String;I)Ljava/lang/invoke/CallSite;
Method arguments:
  #48 add
  #49 2
```

​	可以看到add（"hello"，"world"）调用被翻译为了invokedynamic指令，第一个参数是常量池中的#52，这个条目又指向了BootstrapMethods中的#1的元素，调用静态方IndyInterface.bootstrap，返回值是一个CallSite对象，这个函数签名如下所示。

```java
public static CallSite bootstrap(
    Lookup caller, // the caller
    String callType, // the type of the call
    MethodType type, // the MethodType
    String name, // the real method name
    int flags // call flags
    ) {
}
```

​	其中callType为调用类型，是枚举类CALL_TYPES的一种，这里为CALL_TYPES.METHOD（"invoke"），name为实际调用的方法名，这里为“add”。这个方法内部调用了realBootstrap方法，realBootstrap方法的返回值为CallSite对象，而CallSite的目标方法句柄（MethodHandle）真正调用了selectMethod方法，这个方法在运行期选择合适的方式进行调用，如图3-13所示。

​	简化上述过程为伪代码如代码清单3-8所示。

![](https://pic.imgdb.cn/item/61aad1e92ab3f51d910e6ff0.jpg)

​	Groovy采用invokedynamic指令有哪些好处呢？第一是标准化，使用Bootstrap Method、CallSite、MethodHandle机制使得动态调用的方式得到统一。第二是保持了字节码层的统一和向后兼容，把动态方法的分派逻辑下放到语言实现层，未来版本可以很方便地进行优化、修改。第三是高性能，拥有接近原生Java调用的性能，也可以享受到JIT优化等。

​	invokedynamic指令并不是只能用在动态语言上，它是一种方法动态分派的方式，除了用于动态语言还有很多其他用途，比如下面我们要介绍的Lambda表达式。

​	代码清单3-8　MethodHandle示例

```java
public static void main(String[] args) throws Throwable {
    MethodHandles.Lookup lookup = MethodHandles.lookup();
    MethodType mt = MethodType.methodType(Object.class,
            Object.class, Object.class);
    CallSite callSite =
            IndyInterface.bootstrap(lookup, "invoke", mt, "add", 0);
    MethodHandle mh = callSite.getTarget();
    
    mh.invokeExact(obj, "hello", "world");
}
```

## 3.2 Lambda表达式的原理

```java
public static void main(String[] args) {   
    Runnable r1 = new Runnable() {
        @Override
        public void run() {
            System.out.println("hello, inner class");
        }
    };
    r1.run();
}
```

​	使用javac编译Test.java会生成两个class文件，Test.class和Test$1.class，反编译以后的代码如下所示。

```java
static final class Test$1 implements Runnable {
    @Override
    public void run() {
        System.out.println("hello, inner class");
    }
}
public class Test {
    public static void main(String[] args) {
        Runnable r1 = new Test$1();
        r1.run();
    }
}
```

​	可以看到匿名内部类是在编译期间生成新的class文件来实现的。接下来我们来看Lambda表达式的实现原理，修改上面的代码为如下的Lambda表达式方式。

```java
public static void main(String[] args) {
    Runnable r = ()->{
        System.out.println("hello, lambda");
    };
    r.run();
}
```

​	使用javac编译上面的代码会发现只生成了一个Test.class类文件，并没有生成匿名内部类，使用javap查看对应字节码如下所示。

```java
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
Code:
  stack=1, locals=2, args_size=1
     0: invokedynamic #2,  0       // InvokeDynamic #0:run:()Ljava/lang/Runnable;
     5: astore_1
     6: aload_1
     7: invokeinterface #3,  1    // InterfaceMethod java/lang/Runnable.run:()V
    12: return

private static void lambda$main$0();
Code:
     0: getstatic     #4      // Field java/lang/System.out:Ljava/io/PrintStream;
     3: ldc           #5      // String hello, lambda
     5: invokevirtual #6      // Method java/io/PrintStream.println:(Ljava/lang/String;)V
     8: return
```

​	字节码中出现了一个名为lambda$main$0的静态方法，这段字节码比较简单，翻译为源码如下。

```java
private static void lambda$main$0() {
    System.out.println("hello, lambda");
}
```

​	这里main方法中出现了invokedynamic指令，第0行中#2表示常量池中#2的元素，这个元素又指向了#0：#23。Test类的部分常量池如下所示。

```java
Constant pool:
   #1 = Methodref          #8.#18         // java/lang/Object."<init>":()V
   #2 = InvokeDynamic      #0:#23         // #0:run:()Ljava/lang/Runnable;
   ...
   #23 = NameAndType        #35:#36        // run:()Ljava/lang/Runnable;

BootstrapMethods:
  0: #20 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #21 ()V
      #22 invokestatic Test.lambda$main$0:()V
      #21 ()V
```

​	其中#0是一个特殊的查找，对应BootstrapMethods中的0行，可以看到这是对静态方法LambdaMetafactory.metafactory（）的调用，它的返回值是java.lang.invoke.CallSite对象，这个对象的getTarget方法返回了目标方法句柄。核心的metafactory方法定义如下所示。

```java
public static CallSite metafactory(
    MethodHandles.Lookup caller,
    String invokedName,
    MethodType invokedType,
    MethodType samMethodType,
    MethodHandle implMethod,
    MethodType instantiatedMethodType
)
```

​	接下来介绍各个参数的含义。

* caller：表示JVM提供的查找上下文。
* invokedName：表示调用函数名，在本例中invokedName为“run”。
* samMethodType：表示函数式接口定义的方法签名（参数类型和返回值类型），本例中run方法的签名为“（）void”。
* implMethod：表示编译时生成的Lambda表达式对应的静态方法invokestatic Test.lambda$main$0。
* instantiatedMethodType：一般和samMethodType是一样或是它的一个特例，在本例中是“（）void”。



​	metafactory方法的内部细节是整个Lambda表达式最复杂的地方。它的源码如图3-14所示。

![](https://pic.imgdb.cn/item/61aad5042ab3f51d911075b6.jpg)

![](https://pic.imgdb.cn/item/61aad5112ab3f51d91107c52.jpg)

​	使用打印的方式看一下具体生成的类名。

```java
Runnable r = ()->{
    System.out.println("hello, lambda");
};

System.out.println(r.getClass().getName());
```

​	输出结果如下所示。

```java
Test$$Lambda$1/1108411398
```

​	其中斜杠/后面的数字1108411398是类对象的hashcode值。
​	InnerClassLambdaMetafactory这个类有一个静态初始化方法块，里面有一个开关可以选择是否把生成的类dump到文件中，这部分代码如下所示。

```java
// For dumping generated classes to disk, for debugging purposes
private static final ProxyClassesDumper dumper;
static {
    final String key = "jdk.internal.lambda.dumpProxyClasses";
    String path = AccessController.doPrivileged(
            new GetPropertyAction(key), null,
            new PropertyPermission(key , "read"));
    dumper = (null == path) ? null : ProxyClassesDumper.getInstance(path);
}
```

​	使用java-Djdk.internal.lambda.dumpProxyClasses=.Test运行Test类会发现在运行期间生成了一个新的内部类Test$$Lambda$1.class。这个类正是由InnerClassLambdaMetafactory使用ASM字节码技术动态生成的。它实现了Runnable接口，并在run方法里调用了Test类的静态方法lambda$main$0（），如下面的代码所示。

```java
final class Test$$Lambda$1 implements Runnable {
    @Override
    public void run() {
        Test.lambda$main$0();
    }
}
```

​	这部分的内容总结如下。

* Lambda表达式声明的地方会生成一个invokedynamic指令，同时编译器生成一个对应的引导方法（Bootstrap Method）。
* 第一次执行invokedynamic指令时，会调用对应的引导方法（Bootstrap Method），该引导方法会调用LambdaMetafactory.metafactory方法动态生成内部类。
* 引导方法会返回一个动态调用CallSite对象，这个CallSite会最终调用实现了Runnable接口的内部类。
* Lambda表达式中的内容会被编译成静态方法，前面动态生成的内部类会直接调用该静态方法。
* 真正执行lambda调用的还是用invokeinterface指令。



​	为什么Java 8的Lambda表达式要基于invokedynamic指令来实现呢？Oracle的开发者专门写了一篇文章介绍了Java 8 Lambda设计时的考虑以及实现方法。文中提到Lambda表达式可以通过内部类、method handle、dynamic proxies等方式实现，但是这些方法各有优劣。实现Lambda表达式需要达成两个目标，为未来的优化提供最大的灵活性，且能保持类文件字节码格式的稳定。使用invokedynamic可以很好地兼顾这两个目标。invokedynamic与之前四个invoke指令最大的不同就在于它把方法分派的逻辑从虚拟机层面下放到程序语言。

​	Lambda表达式采用的方式并不是在编译期间生成匿名内部类，而是提供一个稳定的字节码二进制表示规范，对用户而言看到的只有invokedynamic这样一个非常简单的指令。用invokedynamic来实现把方法翻译的逻辑隐藏在JDK的实现中，后续想替换实现方式非常简单，只用修改LambdaMetafactory.metafactory里面的逻辑就可以了。这种方法把Lambda翻译的策略由编译期间推迟到运行时，未来的JDK会怎样实现Lambda表达式可能还会有变化。

## 3.3 泛型与字节码

​	Java泛型是JDK5引进的新特性，对于泛型的引入社区褒贬不一，好的地方是泛型可以在编译期帮我们发现一些明显的问题，不好的地方是泛型在设计上因为考虑兼容性等原因，留下了不少缺陷。Java泛型更像是一个语法糖，接下来我们将从字节码的角度分析泛型，以下面的泛型类Pair为例。

```java
public class Pair<T> {

    public T first;
    public T second;

    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }
}

public void foo(Pair<String> pair) {
    String left = pair.left;
}
```

​	foo方法对应的字节码如下所示。

```java
0: aload_1
1: getfield      #2                  // Field left:Ljava/lang/Object;
4: checkcast     #4                  // class java/lang/String
7: astore_2
8: return
```

​	上面的字节码解释如下。

* 第0行：加载参数pair到操作数栈上。
* 第1行：调用getfield指令把left值加载到操作栈上，可以看到left字段的字段类型为Object，并不是String。
* 第4行：checkcast指令用来检查对象是否符合给定类型，这里是判断left值是否是java/lang/String类型。如果类型不匹配，checkcast指令会抛出java.lang.ClassCastException异常。
* 第7行：将栈顶值left存储到局部变量left中。



​	接下来我们来看看当泛型遇到方法重载时会发生什么，以下面的代码为例。

```java
public void print(List<String> list)  { }
public void print(List<Integer> list) { }
```

​	上面的代码编译时会报错，提示“name clash：`print（List<Integer>）and print（List<String>）have the same erasure`”，这两个方法编译后对应的字节码一样，如下所示。

```java
descriptor: (Ljava/util/List;)V
Code:
  stack=0, locals=2, args_size=2
     0: return
  LocalVariableTable:
    Start  Length  Slot  Name   Signature
        0       1     0  this   LMyClass;
        0       1     1  list   Ljava/util/List;
}
```

​	理解泛型概念的最重要的是理解类型擦除。Java的泛型是在javac编译器中实现的，在生成的字节码中，已经不包含泛型信息。这种在泛型使用时加上类型参数，在编译时再抹掉的过程被称为泛型擦除。

​	比如在代码中类型`List<String>`与`List<Integer>`在编译以后都变成了List，而JVM不允许相同签名的方法在一个类中同时存在，所以上面代码编译会失败。

​	由泛型附加的类型信息对JVM来说是不可见的，Java编译器会在编译时尽可能地发现可能出错的地方，但也不是万能的。很多奇怪的泛型语法规定都与类型擦除有关，比如下面这些。

​	1）泛型类并没有自己独有的Class类对象，比如并不存在`List<String>.class`或是`List<Integer>.class`，而只有List.class。

​	2）泛型不能用primary原始类型来实例化类型参数，比如在Java中不能出现`ArrayList<int>`，只有`ArrayList<Integer>`。因为泛型类型会被擦除为Object，而原始类型int不能存储到对象类型Object中。

​	3）Java也不能捕获泛型异常，以下面的代码为例。

```java
public <T extends Throwable> void foo() {
    try {
    } catch (T e) {
    }
}
```

​	因为JVM的异常处理是通过异常表来实现的，如果要捕获的异常在编译期无法确定，就无法生成对应的异常表。

```java
Pair<Integer>[] t = new Pair<Integer>[10];
error: generic array creation
```

​	在Java中，Object[]数组可以是任何数组的父类，假设我们可以创建泛型数组，上面代码中的t的类型`Pair<Integer>[]`在编译以后被擦除为Pair[]，可以被转换赋值为类型为Object[]的变量objArray，在语法上可以往objArray中存放任意类型的数据。这样原本定义只能存储元素类型为`Pair<Integer>`的数组，结果却可以存放任意类型的数据，如下所示。

```java
Pair<Integer>[] t = new Pair<Integer>[10];
Object[] objArray = t;
objArray[0] = new Thread();
```

​	泛型的内容到这里就告一段落了，接下来我们来看看synchronized关键字的实现原理。

## 3.4 synchronized的实现原理

​	synchronized是Java中的关键字，用于定义一个临界区（critical section）。临界区是指一次只能被一个线程执行的代码片段。synchronized保证方法和代码块在同一时刻只有一个线程可以进入临界区。本节将深入分析synchronized关键字在字节码层面的实现原理。以下面的代码为例。

```java
public class Counter {
    private int count = 0;
    public void increase() {
        ++count;
    }
    public int getCount() {
        return count;
    }
}
```

​	Counter类的increase方法并非线程安全，这个方法对应的字节码如下所示。

```java
 0: aload_0
 1: dup
 2: getfield      #2                  // Field count:I
 5: iconst_1
 6: iadd
 7: putfield      #2                  // Field count:I
10: return
```

​	重点看第2～7行，getfield#2指令获取字段count的值，iconst_1和iadd将取出的值加一，putfield#2指令将更新之后的值写回到字段count中，这是一个典型的read-modify-write的模式。

​	如果有两个线程同时调用了increase方法，各自通过getfield#2获得了count的值，随后执行了加1，最后各自将更新以后的值写回count中，count就只被加1，丢失了一次加1操作。

​	一个简单的解决办法就是给increase方法增加synchronized关键字修饰。

```java
public synchronized void increase() {
    ++count;
}
```

​	JVM不会使用特殊的字节码指令来处理同步方法，JVM会解析方法的访问标记，判断方法是不是同步的，也就是检查方法ACC_SYNCHRONIZED标记位是否被设置为1。如果方法有ACC_SYNCHRONIZED修饰，执行线程会先尝试获取锁。如果是实例方法，JVM会把当前实例对象this作为隐式的监视器。如果是类方法，JVM会把当前类的类对象作为隐式的监视器。在同步方法完成以后，不管是正常返回还是异常返回，都会释放锁。上面的代码在功能上等价于下面的代码。

```java
public void increase() {
    synchronized (this) {
        ++count;
    }
}
```

​	它对应的字节码如下所示。

```java
 0: aload_0
 1: dup
 2: astore_1
 3: monitorenter
 4: aload_0
 5: dup
 6: getfield      #2                  // Field count:I
 9: iconst_1
10: iadd
11: putfield      #2                  // Field count:I
14: aload_1
15: monitorexit
16: goto          24
19: astore_2
20: aload_1
21: monitorexit
22: aload_2
23: athrow
24: return
Exception table:
 from    to  target type
     4    16    19   any
    19    22    19   any
```

​	逐行介绍上面的代码如下。

* 第0～2行：将this对象引用入栈，使用dup指令复制栈顶元素，并将它存入局部变量表位置为1的地方，现在栈上还剩下一个this对象引用。
* 第3行：monitorenter指令尝试获取栈顶this对象的监视器锁，如果成功则继续往下执行，如果已经有其他线程的线程持有，则进入等待状态。
* 第4～11行：执行++count。
* 第14～15行：将this对象入栈，调用monitorexit释放锁。
* 第19～23行：执行异常处理，我们代码中本来没有try-catch的代码，为什么字节码会加上这段逻辑呢？我们稍后再来讲解。



​	每个Java对象都可以作为一个实现同步的锁，这些锁被称为监视器锁（Monitor），有三种不同的表现形态。

* synchronized修饰的非静态方法，监视器锁是当前对象。
* 用synchronized修饰的静态方法，监视器锁是当前类的类对象。
* synchronized（lock）{}同步代码块，监视器锁是lock对象。



​	Java虚拟机保证一个monitor一次最多只能被一个线程占有。monitorenter和monitorexit是两个与监视器相关的字节码指令。当线程执行到monitorenter指令时，会尝试获取栈顶对象对应监视器（monitor）的所有权，也就是尝试获取对象的锁。如果此时monitor没有其他线程占有，当前线程会成功获取锁，monitor计数器置为1。如果当前线程已经拥有了monitor的所有权，monitorenter指令也会顺利执行，monitor计数器加1。如果其他线程拥有了monitor的所有权，当前线程会阻塞，直到monitor计数器变为0。

​	当线程执行monitorexit时，会将监视器计数器减1，计时器值等于0时，锁被释放，其他等待这个锁的线程可以尝试去获取monitor的所有权。

​	编译器必须保证无论同步代码块中的代码以何种方式结束（正常退出或异常退出），代码中每次调用monitorenter必须执行对应的monitorexit指令。如果执行了monitorenter指令但没有执行monitorexit指令，monitor一直被占有，则其他线程没有办法获取锁。如果执行monitorexit的线程原本并没有这个monitor的所有权，那monitorexit指令在执行时将抛出IllegalMonitorStateException异常。

​	为了保证这一点，编译器会自动生成一个异常处理器，这个异常处理器确保了方法正常退出和异常退出都能正常释放锁。可理解为下面这样的一段Java代码。

```java
public void _foo() throws Throwable {
    monitorenter(lock);
    try {
        bar();
    } finally {
        monitorexit(lock);
    }
}
```

​	根据之前介绍的try-catch-finally的字节码实现原理，finally语句块会被编译器复制到方法正常退出和异常退出的地方，从语义上等价于下面的代码。

```java
public void _foo() throws Throwable {
    monitorenter(lock);
    try {
        bar();
        monitorexit(lock);
    } catch (Throwable e) {
        monitorexit(lock);
        throw e;
    }
}
```

​	这就是我们在上面字节码中看到只有一个monitorenter指令却有两个monitorexit的原因。

## 3.5　反射的实现原理

​	反射是Java的核心特性之一，很多框架都大量使用反射来实现强大的功能，比如Spring、Mybatis等。Java的反射机制允许我们运行时动态地调用某个对象的方法、新建对象实例、获取对象的属性等。接下来我们来看看反射背后的实现原理，以下面的代码清单3-10为例。

```java
public class ReflectionTest {
    private static int count = 0;
    public static void foo() {
        new Exception("test#" + (count++)).printStackTrace();
    }
    public static void main(String[] args) throws Exception {
        Class<?> clz = Class.forName("ReflectionTest");
        Method method = clz.getMethod("foo");
        for (int i = 0; i < 20; i++) {
            method.invoke(null);
        }
    }
}
```

![](https://pic.imgdb.cn/item/61aadbde2ab3f51d911584db.jpg)

​	可以看到同一段代码，运行的堆栈结果与执行次数有关，在第0～15次时调用方式为sun.reflect.NativeMethodAccessorImpl.invoke0，从第16次开始调用方式变为了sun.reflect.GeneratedMethodAccessor1.invoke。接下来看看这背后的原理。

### 3.5.1　反射方法源码分析

![](https://pic.imgdb.cn/item/61aadc0e2ab3f51d9115a9ab.jpg)

​	Method.invoke方法调用了MethodAccessor.invoke方法，MethodAccessor是一个接口，它的源码如下所示。

```java
public interface MethodAccessor {
    public Object invoke(Object obj, Object[] args)
        throws IllegalArgumentException, InvocationTargetException;
}
```

​	从输出的堆栈可以看到MethodAccessor的实现类是委托类DelegatingMethodAcDelegatingMethodAccessorImpl，它的invoke方法非常简单，就是把调用委托给了真正的MethodAccessorImpl实现类。

```java
class DelegatingMethodAccessorImpl extends MethodAccessorImpl {
    private MethodAccessorImpl delegate;
    public Object invoke(Object obj, Object[] args)
        throws IllegalArgumentException, InvocationTargetException
    {
        return delegate.invoke(obj, args);
    }
```

​	通过堆栈可以看到在第0～15次调用中，实现类是NativeMethodAccessorImpl，从第16次调用开始，实现类是GeneratedMethodAccessor1，玄机就在NativeMethodAccessor-Impl的invoke方法中，如图3-18所示。

![](https://pic.imgdb.cn/item/61aadd6b2ab3f51d9116cd2d.jpg)

​	前0～15次都会调用invoke0，这是一个native的函数，代码如下所示。

```java
private static native Object invoke0(Method m, Object obj, Object[] args);
```

​	15次调用以后会使用新的逻辑，利用GeneratedMethodAccessor1来调用反射的方法。MethodAccessorGenerator的作用是通过ASM生成新的类sun.reflect.GeneratedMethod-Accessor1。为了查看生成的类的内容，可以使用阿里的arthas工具。修改上面的代码，在main函数的最后加上“System.in.read（）；”让JVM进程不要退出。执行arthas工具中的./as.sh，会要求输入JVM进程，如图3-19所示。

![![](https://pic.imgdb.cn/item/61aaddb02ab3f51d91170c0e.jpg)](https://pic.imgdb.cn/item/61aadda12ab3f51d91170130.jpg)

![](https://pic.imgdb.cn/item/61aaddb02ab3f51d91170c0e.jpg)

​	翻译上面这段字节码，忽略掉异常处理以后的代码如下所示。

```java
public class GeneratedMethodAccessor1 extends MethodAccessorImpl {
    @Override
    public Object invoke(Object obj, Object[] args)
            throws IllegalArgumentException, InvocationTargetException {
        ReflectionTest.foo();
        return null;
    }
}
```

​	为什么要设置为0～15次使用native方式来调用，15次以后使用ASM新生成的类来处理反射的调用呢？

​	这是基于性能的考虑，JNI native调用的方式要比动态生成类调用的方式慢20倍左右，但是由于第一次字节码生成的过程比较慢，如果反射仅调用一次的话，采用生成字节码的方式反而比native调用的方式慢3~4倍。为了权衡这两种方式的利弊，Java引入了inflation机制，接下来我们来看看inflation的细节。

### 3.5.2　反射的inflation机制

​	很多情况下，反射只会调用一两次，JVM于是想了一个办法，设置了一个sun.reflect.inflationThreshold阈值，默认等于15。当反射方法调用超过15次时（从0开始计算），会使用ASM生成新类，保证后面的调用比native要快。调用次数小于15次的情况下，直接用native的方式来调用，没有额外类的生成、校验、加载的开销。这种方式被称为inflation机制。

​	JVM与inflation相关的属性有两个，一个是刚提到的阈值sun.reflect.inflationThreshold，还有一个是是否禁用inflation的属性sun.reflect.noInflation，默认值为false。如果把sun.reflect.noInflation这个值设置成true，那么从第0次开始就使用动态生成类的方式来调用反射方法了，而不会使用native的方式。增加noInflation选项重新执行上述Java代码，如下所示。

```java
java -cp . -Dsun.reflect.noInflation=true ReflectionTest
```

```java
java.lang.Exception: test#0
        at ReflectionTest.foo(ReflectionTest.java:10)
        at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source)
        at java.lang.reflect.Method.invoke(Method.java:497)
        at ReflectionTest.main(ReflectionTest.java:18)
java.lang.Exception: test#1
        at ReflectionTest.foo(ReflectionTest.java:10)
        at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source)
        at java.lang.reflect.Method.invoke(Method.java:497)
        at ReflectionTest.main(ReflectionTest.java:18)
```

​	可以看到，从第0次开始就已经没有使用native方法来调用反射方法了。

# 4. javac编译原理简介

​	编译原理中，源代码到机器指令的过程如图4-1所示。

![](https://pic.imgdb.cn/item/61adcbc12ab3f51d91cb8b1b.jpg)

​	javac这种将源文件转为字节码的过程在编译原理上属于前端编译，不涉及目标机器码相关的代码的生成和优化。JDK中的javac本身是用Java语言编写的，在某种意义上实现了javac语言的自举。javac没有使用类似YACC、Lex这样的生成器工具，所有的词法分析、语法分析等功能都是自己实现，代码比较精简和高效。

​	javac的源码比较复杂，如果没有扎实的编译原理基础，阅读起来就会比较吃力，调试源码是一个比较好的方式，可以帮助我们更好地理解实现细节。下面先来看看如何在IDEA中调试javac的源码。

## 4.1 javac源码调试

1）首先下载并导入javac的源码。从OpenJDK的网站下载javac的源码，导入IntelliJ IDEA中。

2）找到javac主函数入口，代码在src/com/sun/tools/javac/Main.java。运行main方法，正常情况下应该会在控制台输出如图4-2所示的内容。

![](https://pic.imgdb.cn/item/61adcc312ab3f51d91cbc957.jpg)

​	新建一个空的HelloWorld.java文件，在启动配置的Program arguments里加入HelloWorld.java的绝对路径，如图4-3所示。
​	再次运行Main.java，可以看到在HelloWorld.java的同级目录生成了HelloWorld.class文件。

3）打开Project Structure页面（File→Project Structure），选择Dependencies选项卡，把`<Moudle source>`顺序调整到项目JDK上面，如图4-4所示。

![](https://pic.imgdb.cn/item/61adcd402ab3f51d91cc7b64.jpg)

![](https://pic.imgdb.cn/item/61adcd972ab3f51d91ccc197.jpg)

​	在Main.java中打上断点，开始调试就可以进入项目源码中的断点处了。接下来看一个实际的例子，以int整型常量加载指令的选择过程为例，测试代码如下所示。

```java
public static void foo() {
    int a = 0;
    int b = 6;
    int c = 130;
    int d = 33000;
}
```

在com/sun/tools/javac/jvm/Items.java的load（）方法加上断点，如图4-5所示。

![](https://pic.imgdb.cn/item/61adcdc52ab3f51d91cce55b.jpg)

* -1～5，选择iconst_x指令。
* -128～127，选择bipush指令。
* -32768～32767，选择sipush指令。
* 其他范围，选择ldc指令。

## 4.2　javac的七个阶段

![](https://pic.imgdb.cn/item/61adcdec2ab3f51d91cd06d5.jpg)

### 4.2.1 parse阶段

​	parse阶段的主要作用是读取.java源文件并做词法分析和语法分析。

​	词法分析（lexical analyze）将源代码拆分为一个个词法记号（Token），这个过程又被称为扫描（scan），比如代码i=1+2在词法分析时会被拆分为五部分：i、=、1、+、2。这个过程会将空格、空行、注释等对程序执行没有意义的部分排除。词法分析与我们理解英语的过程类似，比如英语句子“you are handsome”在我们大脑中会被拆分为you、are、handsome三个单词。

​	javac中的词法分析由com.sun.tools.javac.parser.Scanner实现，以语句“int k=i+j；”为例，引入Scanner类的源码做实际的测试，以下面的代码清单4-1为例。

```java
import com.sun.tools.javac.parser.Scanner;
import com.sun.tools.javac.parser.ScannerFactory;
import com.sun.tools.javac.util.Context;

public class MyTest {
    public static void main(String[] args) {
        ScannerFactory factory = ScannerFactory.instance(new Context());
        Scanner scanner = factory.newScanner("int k = i + j;", false);

        scanner.nextToken();
        System.out.println(scanner.token().kind);   // int
        scanner.nextToken();
        System.out.println(scanner.token().name()); // j
        scanner.nextToken();
        System.out.println(scanner.token().kind);   // =
        scanner.nextToken();
        System.out.println(scanner.token().name()); // i
        scanner.nextToken();
        System.out.println(scanner.token().kind);   // +
        scanner.nextToken();
        System.out.println(scanner.token().name()); // j
        scanner.nextToken();
        System.out.println(scanner.token().kind);   // ;
    }
}
```

​	Scanner会读取源文件中的内容，将其解析为Java语言的Token序列，这个过程如图4-7所示。
​	词法分析之后是进行语法分析（syntax analyzing），语法分析是在词法分析的基础上分析单词之间的关系，将其转换为计算机易于理解的形式，生成抽象语法树（Abstract Syntax Tree，AST）。AST是一个树状结构，树的每个节点都是一个语法单元，抽象语法树是后续的语义分析、语法校验、代码生成的基础。

![](https://pic.imgdb.cn/item/61adcef02ab3f51d91cdbd5a.jpg)

​	与其他大多数语言一样，javac也是使用递归下降法（recursive descent）来生成抽象语法树。主要功能由com.sun.tools.javac.parser.JavacParser类完成，语句“int k=i+j；”对应的AST如图4-8所示。

![](https://pic.imgdb.cn/item/61adcf072ab3f51d91cdcbf7.jpg)

​	词法分析和语法分析没有我们想的那么遥远，解析CSV、JSON、XML本质上也是语法分析和一部分语义分析的过程。

### 4.2.2 enter阶段

​	enter阶段的主要作用是解析和填充符号表（symbol table），主要由com.sun.tools.javac.comp.Enter和com.sun.tools.javac.comp.MemberEnter类来实现。符号表是由标识符、标识符类型、作用域等信息构成的记录表。在遍历抽象语法树遇到类型、变量、方法定义时，会将它们的信息存储到符号表中，方便后续进行快速查询。

```java
public class HelloWorld { // 定义类 HelloWorld
    int x = 5; // 定义 int 型字段 x，初始化值为 5
    char y = 'A'; // 定义 char 型字段 y，初始化值为 'A'
    // 定义 add 方法，返回类型为 long，参数个数为 2，类型都为 long
    public long add(long a, long b) {
        return a + b;
    }
}
```

​	javac使用Symbol类来表示符号，每个符号都包含名称、类别和类型这三个关键属性，如下所示。

* name表示符号名，比如上面代码中的x、y、add都是符号名。
* kind表示符号类型，上面代码中，x的符号类型是Kinds.VAR，表示这是一个变量符号。add的符号类型是Kinds.MTH，表示这是一个方法符号。
* type表示符号类型，上面代码中，x的符号类型是int，y的符号类型为char，add方法的符号类型为null，对于Java这种静态类型的语言来说，在编译期就会确定变量的类型。



​	Symbol类是一个抽象类，常见的实现类有VarSymbol、MethodSymbol等，如图4-9所示。

![](https://pic.imgdb.cn/item/61add0072ab3f51d91ce5f26.jpg)

​	Symbol定义了符号是什么，作用域（Scope）则指定了符号的有效范围，由com.sun.tools.javac.code.Scope类表示。以下面的代码为例。

```java
public void foo() {
    int x = 0; // x 在 foo 方法作用域内
    System.out.println(x);
}

public void bar() {
    int x = 0; // x 在 bar 方法作用域内
    System.out.println(x);
}
```

​	foo和bar函数都定义了一个名为x的int类型的变量，这两个变量能独立使用且不会互相影响，在超出各自的方法体作用域以后就对外不可见了，外部也访问不到。

​	作用域也可以进行嵌套，如下面的代码所示。

```java
public class MyClass {
    int x = 0;
    public void foo() {
        int i = 0; // foo 方法作用域
        {
            int y = 0; // 第一层嵌套作用域
            {
                int z = 0; // 第二层嵌套作用域
            }
        }
        int j = 0; // foo 方法作用域
    }
}
```

​	符号表查找的过程是先在当前作用域中查找，如果找到，就直接返回；如果没有找到，那么它会向上在外层的作用域中继续查找，直到找到或者到达顶层作用域为止。

​	enter阶段除了上述生成符号表，还会在类文件中没有默认构造方法的情况下，添加<init>构造方法等。

### 4.2.3 process阶段

​	process用来做注解的处理，这个步骤由com.sun.tools.javac.processing.JavacProcessing-Environment类完成。从JDK6开始，javac支持在编译阶段允许用户自定义处理注解，大名鼎鼎的lombok框架就是利用了这个特性，通过注解处理的方式生成目标class文件，比在运行时反射调用性能明显提升。这一部分的内容比较多，会在后面的章节单独进行介绍，这里不再详细展开。

### 4.2.4 attr阶段

​	attr阶段是语义分析的一部分，主要由com.sun.tools.javac.comp.Attr类实现，这个阶段会做语义合法性检查、常量折叠等，由com.sun.tools.javac.comp包下的Check、Resolve、ConstFold、Infer几个类辅助实现。

​	com.sun.tools.javac.comp.Check类的主要作用是检查变量类型、方法返回值类型是否合法，是否有重复的变量、类定义等，比如下面这些场景。

**Check**

​	1）检查方法返回值是否与方法声明的返回值类型一致，以下面的代码为例。

```java
public int foo() {
    return "hello";
}
```

​	这个检查由com.sun.tools.javac.comp.Check类的checkType完成，这个方法的定义如下。

```java
Type checkType(final DiagnosticPosition pos, final Type found, final Type req, final CheckContext checkContext){
}
```

​	其中found参数值表示当前语法节点的类型，req参数表示当前语义环境需要的类型，这个方法内部会调用checkContext.compatible方法检查found与req是否匹配。当抽象语法树遍历到"return hello；"对应的节点时，得到的found参数值为对象类型String，req类型为int，返回值类型和方法声明不一致，错误日志如下所示。

```java
error: incompatible types: String cannot be converted to int
        return "hello";
               ^
1 error
```

​	2）检查是否有重复的定义，比如检查同一个类中是否存在相同签名的方法。校验的逻辑由Check类的checkUnique方法实现，以下面的代码为例。

```java
class HelloWorld {
    public void foo() {}
    public void foo() {}
}
```

​	在编译时会出现重复定义的错误，如下所示

```java
error: method foo() is already defined in class HelloWorld
    public void foo() {}
                ^
```

​	在编译时会出现重复定义的错误，如下所示。

```java
error: method foo() is already defined in class HelloWorld
    public void foo() {}
                ^
```

**Resolve**

​	com.sun.tools.javac.comp.Resolve类的主要作用是：

* 检查变量、方法、类访问是否合法，比如private方法的访问是否在方法所在类中访问等。
* 为重载方法调用选择最具体（most specific）的方法。



​	以下面的代码为例：

```java
public static void method(Object obj) {
    System.out.println("method # Object");
}

public static void method(String obj) {
    System.out.println("method # String");
}

public static void main(String[] args) {
    method(null);
}
```

​	在Java中允许方法重载（overload），但要求方法签名不能一样。调用method（null）实际是调用第二个方法输出“method#String”，javac在编译时会推断出最具体的方法，方法的选择在Resolve类的mostSpecific方法中完成。第二个方法的入参类型String是第一个方法的入参类型Object的子类，会被javac认为更具体。

​	将上面的例子稍作修改，增加一个入参类型为Integer的method方法。

```java
public static void method(Integer obj) {
    System.out.println("method # Integet");
}

public static void method(String obj) {
    System.out.println("method # String");
}

public static void main(String[] args) {
    method(null);
}
```

​	在编译时会报错，错误信息如下：

```java
error: reference to method is ambiguous
        method(null);
        ^
  both method method(Integer) in HelloWorld and method method(String) in HelloWorld match
```

​	null可以被赋值给String类型，也可以被赋值给Integer类型，此时编译器无法决定该调用哪个方法。



**ConstFold**

​	com.sun.tools.javac.comp.ConstFold类的主要作用是在编译期将可以合并的常量合并，比如常量字符串相加，常量整数运算等，以下面的代码为例。

```java
public void foo() {
    int x = 1 + 2;
    String y = "hel" + "lo";
    int z = 100 / 2;
}
```



​	对应的字节码如下所示：

```java
0: iconst_3
1: istore_1
2: ldc           #2                  // String hello
4: astore_2
5: bipush        50
7: istore_3
8: return
```

​	可以看到编译后的字节码已经将常量进行了合并，javac没有理由把这些可以在编译期完成的计算留到运行时。

​	com.sun.tools.javac.comp.Infer类的主要作用是推导泛型方法的参数类型，比较简单，这里不再赘述。

### 4.2.5 flow阶段

​	flow阶段主要用来处理数据流分析，主要由com.sun.tools.javac.comp.Flow类实现，很多编译期的校验在这个阶段完成，下面列举几个常见的场景。

1）检查非void方法是否所有的退出分支都有返回值，以下面的代码为例：

```java
public boolean foo(int x) {
    if (x == 0) {
        return true;
    }
    // 注释掉这个 return
    // return false; 
}
```

​	上面的代码编译报错如下：

```java
error: missing return statement
    }
    ^
```

​	2）检查受检异常（checked exception）是否被捕获或者显式抛出，以下面的代码为例。

```java
public void foo() {
    throw new FileNotFoundException();
}
```

​	上面的代码编译报错如下：

```java
error: unreported exception FileNotFoundException; must be caught or declared to be thrown
    throw new FileNotFoundException();
    ^
```

​	3）检查局部变量使用前是否被初始化。Java中的成员变量在未赋值的情况下会赋值为默认值，但是局部变量不会，在使用前必须先赋值，以下面的代码为例。

```java
public void foo() {
    int x;
    int y = x + 1;
    System.out.println(y);
}
```

​	上面的代码编译报错如下：

```java
error: variable x might not have been initialized
        int y = x + 1;
                ^
```

4）检查final变量是否有重复赋值，保证final的语义，以下面的代码为例。

```java
public void foo(final int x) {
    x = 2;
    System.out.println(x);
}
```

​	上面的代码编译报错如下：

```java
error: final parameter x may not be assigned
        x = 2;
        ^
```

5）检查是否有语句不可达，比如在return之后的不可达代码，以下面的代码为例。

```java
public int foo() {
    System.out.println("Hello");
    return 1;
    System.out.println("World");
}
```

​	上面的代码编译报错如下。

```java
	error: unreachable statement
    System.out.println("World");
    ^
```

​	这个逻辑判断是在Flow.java的AliveAnalyzer中完成的，在遇到return以后，会回调markDead方法，把alive变量设置为false，表示后面的代码块将不可达，源代码如下所示。

```java
void markDead(JCTree tree) {
    alive = false;
}
```

​	继续往下处理第2个println时，回调AliveAnalyzer的scanStat方法，这里会判定当前语句是否已经不可达，如果不可达，则输出错误日志，如下所示。

```java
void scanStat(JCTree tree) {
    // 如果已经不可达，tree 代表第二次 println 语句，不为 null
    if (!alive && tree != null) {
        // 打印 "error: unreachable statement"
        log.error(tree.pos(), "unreachable.stmt");
        if (!tree.hasTag(SKIP)) alive = true;
    }
    scan(tree);
}
```

### 4.2.6 desugar阶段

​	Java的语法糖没有Kotlin和Scala那么丰富，每次随着新版本的发布都会加入非常多的语法糖。下面这些某种意义上来说都算是语法糖：泛型、内部类、try-with-resources、foreach语句、原始类型和包装类型之间的隐式转换、字符串和枚举的switch-case实现、后缀和前缀运算符（i++和++i）、变长参数等。

​	desugar的过程就是解除语法糖，主要由com.sun.tools.javac.comp.TransTypes类和com.sun.tools.javac.comp.Lower类完成。TransTypes类用来擦除泛型和插入相应的类型转换代码，Lower类用来处理除泛型以外其他的语法糖。下面是列举的几个常见的Desugar例子。

​	1）在desugar阶段泛型会被擦除，在有需要时自动为原始类和包装类型转换添加拆箱、装箱代码，以下面的代码为例。

```java
public void foo() {
    List<Long> idList = new ArrayList<>();
    idList.add(1L);
    long firstId = idList.get(0);
}
```

​	对应的字节码如下所示：

```java
// 执行 new ArrayList<>()
 0: new           #2             // class java/util/ArrayList
 3: dup
 4: invokespecial #3             // Method java/util/ArrayList."<init>":()V
 7: astore_1
 8: aload_1
 // 把原始类型 1 自动装箱为 Long 类型
 9: lconst_1
10: invokestatic  #4             // Method java/lang/Long.valueOf:(J)Ljava/lang/Long;
// 执行 add 调用
13: invokeinterface #5,  2       // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z
18: pop
19: aload_1
// 执行 get(0) 调用
20: iconst_0
21: invokeinterface #6,  2       // InterfaceMethod java/util/List.get:(I)Ljava/lang/Object;
// 检查 Object 对象是否是 Long 类型
26: checkcast     #7             // class java/lang/Long
// 自动拆箱为原始类型
29: invokevirtual #8             // Method java/lang/Long.longValue:()J
32: lstore_2
33: return
```

把上面的代码转换为等价的Java代码，如下所示：

```java
public void foo() {
    List idList = new ArrayList();
    // 原始类型自动装箱
    idList.add(Long.valueOf(1L));
    // 插入强制类型转换，保持泛型语义，自动拆箱转为原始类型
    long firstId = ((Long) idList.get(0)).longValue();
}
```

​	2）去除逻辑死代码，也就是不可能进入的代码块，比如if（false）{}，以下面的代码为例。

```java
public void foo() {
    if (false) {
        System.out.println("string #1 false");
    }  else {
        System.out.println("string #2 true");
    }
}
```

​	对应的字节码如下所示：

```java
public void foo();
     0: getstatic     #2     // Field java/lang/System.out:Ljava/io/PrintStream;
     3: ldc           #3     // String string #2 true
     5: invokevirtual #4     // Method java/io/PrintStream.println: (Ljava/lang/ String;)V
     8: return
```

​	可以看到，在编译后的字节码中if（false）分支的代码已经不存在了，javac这部分的逻辑在Lower类的visitIf方法中处理，如下面的代码清单4-2所示。

​	代码清单4-2　逻辑死分支剔除

```java
public void visitIf(JCIf tree) {
    JCTree cond = tree.cond = translate(tree.cond, syms.booleanType);
    if (cond.type.isTrue()) {
        result = translate(tree.thenpart);
        addPrunedInfo(cond);
    } else if (cond.type.isFalse()) {
        if (tree.elsepart != null) {
            result = translate(tree.elsepart);
        } else {
            result = make.Skip();
        }
        addPrunedInfo(cond);
    } else {
        // Condition is not a compile-time constant.
        tree.thenpart = translate(tree.thenpart);
        tree.elsepart = translate(tree.elsepart);
        result = tree;
    }
}
```

​	在碰到if语句块时，javac会先判断if条件值在编译期是否是一个固定值。如果条件值恒等于true，则会保留if条件部分，去掉else部分；反之，如果条件值恒等于false，则会去掉if部分，保留else部分。如果条件值在编译期不恒等于true和false，则会保留if和else两部分。

​	3）String类、枚举类的switch-case也是在Desugar阶段进行的，以下面的枚举类switch代码为例。

```java
Color color = Color.BLUE;
switch (color) {
    case RED:
        System.out.println("red");
        break;
    case BLUE:
        System.out.println("blue");
        break;
    default:
        System.out.println("default");
        break;
}
```

​	javac为枚举的每一个switch都会生成一个中间类，这个类包含了一个称为“SwitchMap”的数组，这个SwitchMap数组维护了枚举ordinal值与递增整数序列的映射关系。上面的代码会转为如代码清单4-3所示的实现形式。

​	代码清单4-3　枚举switch的等价实现

```java
class Outer$0 {
    synthetic static final int[] $SwitchMap$Color = new int[Color.values().length];

    static {
        try {
            $SwitchMap$Color[Color.RED.ordinal()] = 1;
        } catch (NoSuchFieldError ex) {
        }
        try {
            $SwitchMap$Color[Color.BLUE.ordinal()] = 2;
        } catch (NoSuchFieldError ex) {
        }
    }
}

public void bar(Color color) {
    switch (Outer$0.$SwitchMap$Color[color.ordinal()]) {
        case 1:
            System.out.println("red");
            break;
        case 2:
            System.out.println("blue");
            break;
        default:
            System.out.println("default");
            break;
    }
}
```

​	为什么不直接用ordinal值来作为case值呢？这是为了提供更好的性能，case值中的ordinal值不一定是连续的，通过SwitchMap数组可以把不连续的ordinal值转为连续的case值，编译成更高效的tableswitch指令。

### 4.2.7 generate阶段

​	generate阶段的主要作用是遍历抽象语法树生成最终的Class文件，由com.sun.tools.javac.jvm.Gen类实现，下面列举了几个常见的场景。

​	1）初始化块代码并收集到`<init>`和`<clinit>`中，以下面的代码为例。

```java
public class MyInit {
    {
        System.out.println("hello");
    }
    public int a = 100;
}
```

​	生成的构造器方法`<init>`对应的字节码如下所示。

```java
public MyInit();
     0: aload_0
     1: invokespecial #1          // Method java/lang/Object."<init>":()V
     4: getstatic     #2          // Field java/lang/System.out:Ljava/io/PrintStream;
     7: ldc           #3          // String hello
     9: invokevirtual #4          // Method java/io/PrintStream.println:(Ljava/lang/String;)V
    12: aload_0
    13: bipush        100
    15: putfield      #5          // Field a:I
    18: return
```

​	可以看到，编译器会自动帮忙生成一个构造器方法`<init>`，没有写在构造器方法中的字段初始化、初始化代码块都被收集到了构造器方法中，翻译为Java源代码如下所示。

```java
public class MyInit {
    public int a;
    
    public MyInit() {
        System.out.println("hello");
        this.a = 100;
    }
}
```

​	与static修饰的静态初始化的逻辑一样，javac会将静态初始化代码块和静态变量初始化收集到`<clinit>`方法中。

​	2）把字符串拼接语句转换为StringBuilder.append的方式来实现，比如下面的字符串x和y的拼接代码。

```java
public void foo(String x, String y) {
    String ret = x + y;
    System.out.println(ret);
}
```

​	在generate阶段会被转换为下面的代码。

```java
public void foo(String x, String y) {
    String ret = new StringBuilder().append(s).append(s2).toString();
    System.out.println(ret);
}
```

​	3）为synchronized关键字生成异常表，保证monitorenter、monitorexit指令可以成对调用。

​	4）switch-case实现中tableswitch和lookupswitch指令的选择。

​	第2章介绍过，switch-case的实现会根据case值的稀疏程度选择tableswitch或者lookupswitch指令来实现，以下面的代码为例。

```java
public static void foo() {
    int a = 0;
    switch (a) {
        case 0:
            System.out.println("#0");
            break;
        case 1:
            System.out.println("#1");
            break;
        default:
        System.out.println("default");
            break;
    }
}
```

​	对应的字节码如下所示。

```java
public static void foo();
 0: iconst_0
 1: istore_0
 2: iload_0
 3: lookupswitch  { // 2
               0: 28
               1: 39
         default: 50
    }
```

​	可以看到，上面的代码是采用lookupswitch而不是tableswitch来实现，难道case值0和1还不够紧凑吗？

​	我们来分析原因，这两个指令的选择逻辑在com/sun/tools/javac/jvm/Gen.java中，如下所示。

```java
long table_space_cost = 4 + ((long) hi - lo + 1); // words
long table_time_cost = 3; // comparisons
long lookup_space_cost = 3 + 2 * (long) nlabels;
long lookup_time_cost = nlabels;
int opcode =
    nlabels > 0 &&
    table_space_cost + 3 * table_time_cost <=
    lookup_space_cost + 3 * lookup_time_cost
    ?
    tableswitch : lookupswitch;
```

​	在上面的例子中，nlables等于case值的个数，等于2，hi表示case值的最大值1，lo表示case值的最小值0，因此可以计算出使用tableswitch和lookupswitch的时间和空间代价，如下所示。

```java
// table_space_cost 表示 tableswitch的空间代价
table_space_cost = 4 + (1 - 0 + 1) = 6
// table_time_cost 表示 tableswitch的时间代价，恒等于 3
table_time_cost = 3
// lookup_space_cost 表示 lookupswitch 的空间代价 
lookup_space_cost = 3 + 2 * 2 = 7
// lookup_time_cost 表示 lookupswitch 的时间代价 
lookup_time_cost = 2
```

​	tableswitch和lookupswitch的总代价计算公式如下。

```java
代价 = 空间代价 + 3 * 时间代价
```

​	因此在case值为0、1时，tableswitch的代价为6+3*3=15，lookupswitch的代价为7+3*2=13，lookupswitch的代价更小，javac选择了lookupswitch作为switch-case的实现指令。

​	如果case值变多为0、1、2时，nlables等于3，hi等于2，lo等于0，因此计算出空间代价和时间代价如下。

```java
table_space_cost = 7
table_time_cost = 3
lookup_space_cost = 3 + 2 * 3 = 9
lookup_time_cost = 3
table_space_cost + 3 * table_time_cost  = 7 + 3 * 3 = 16
lookup_space_cost + 3 * lookup_time_cost = 9 + 3 * 3 = 18
```

​	这种情况下，table_space_cost的代价更小，选择tableswitch作为switch-case的实现指令。

​	通过上面的源码分析，可以彻底搞清楚switch-case实现指令选择的依据：在一般情况下，当case值较稀疏时，使用tableswitch的空间代价较大，会选择lookupswitch指令来实现；当case值较密集时，lookupswitch的时间代价比tableswitch高，会选择tableswitch指令。

# 6. ASM和Javassist字节码操作工具

## 6.1 ASM介绍

​	当需要对一个class文件做修改时，我们可以选择自己解析这个class文件，在符合Java字节码规范的前提下进行字节码改造。如果你写过class文件的解析代码，就会发现这个过程极其烦琐，更别提增加方法、手动计算max_stack等操作了。

​	ASM最开始是2000年Eric Bruneton在INRIA（法国国立计算机及自动化研究院）读博士期间完成的一个作品。那个时候包含java.lang.reflect.Proxy包的JDK 1.3还没发布，ASM被用作代码生成器生成动态代理的代理类。经过多年的发展，ASM被诸多框架采用，成为字节码操作领域事实上的标准。

​	简单的API背后ASM自动帮我们做了很多事情，比如维护常量池的索引、计算最大栈大小max_stack、局部变量表大小max_locals等，除此之外还有下面这些优点。

* 架构设计精巧，使用方便。
* 更新速度快，支持最新的Java版本。
* 速度非常快，在动态代理class的生成和转换时，尽可能确保运行中的应用不会被ASM拖慢。
* 非常可靠、久经考验，有很多著名的开源框架都在使用，如cglib、MyBatis、Fastjson等。



​	其他字节码操作框架在操作字节码的过程中会生成很多中间类和对象，耗费大量的内存且运行缓慢，ASM提供了两种生成和转换类的方法：基于事件触发的core API和基于对象的Tree API，这两种方式可以用XML解析的SAX和DOM方式来对照。

​	SAX解析XML文件采用的是事件驱动，它不需要一次解析完整个文档，而是按内容顺序解析文档，如果解析时符合特定的事件则回调一些函数来处理事件。SAX运行时是单向的、流式的，解析过的部分无法在不重新开始的情况下再次读取，ASM的Core API与这种方式类似。

​	DOM解析方式则会将整个XML作为类似树结构的方式读入内存中以便操作及解析，ASM的Tree API与这种方式类似。

​	以下面的XML文件为例。

```xml
<Order>
    <Customer>Arthur</Customer>
    <Product>
        <Name>Birdsong Clock</Name>
        <Quantity>12</Quantity>
        <Price currency="USD">21.95</Price >
    </Product>
</Order>
```

​	对应的SAX和DOM解析方式如图6-1所示。

![](https://pic.imgdb.cn/item/61af4def2ab3f51d919f15d2.jpg)

### 6.1.1　ASM Core API核心类

​	ASM核心包由Core API、Tree API、Commons、Util、XML几部分组成，如图6-2所示。

![](https://pic.imgdb.cn/item/61af4e152ab3f51d919f3071.jpg)

​	Core API中最重要的三个类是ClassReader、ClassVisitor、ClassWriter，字节码操作都是跟这个三个类打交道。

​	ClassReader是字节码读取和分析引擎，负责解析class文件。采用类似于SAX的事件读取机制，每当有事件发生时，触发相应的ClassVisitor、MethodVisitor等做相应的处理。

​	ClassVisitor是一个抽象类，使用时需要继承这个类，ClassReader的accept（）方法需要传入一个ClassVisitor对象。ClassReader在解析class文件的过程中遇到不同的节点时会调用ClassVisitor不同的visit方法，比如visitAttribute、visitInnerClass、visitField、visitMethod、visitEnd方法等。

​	在上述visit的过程中还会产生一些子过程，比如visitAnnotation会触发Annotation-Visitor的调用、visitMethod会触发MethodVisitor的调用。正是在这些visit的过程中，我们得以有机会去修改各个子节点的字节码。

​	ClassVisitor类中的visit方法按照以下的顺序被调用执行。

```java
visit
[visitSource]
[visitOuterClass] 
(visitAnnotation | visitAttribute)*
(visitInnerClass | visitField | visitMethod)* 
visitEnd
```

​	visit方法最先被调用，接着调用零次或一次visitSource方法，调用零次或一次visitOuter-Class方法，接下来按任意顺序调用任意多次visitAnnotation和visitAttribute方法，再按任意顺序调用任意多次visitInnerClass、visitField、visitMethod方法，最后调用visitEnd。

​	调用时序图如图6-3所示。
​	ClassWriter类是ClassVisitor抽象类的一个实现类，在ClassVisitor的visit方法中可以对原始的字节码做修改，ClassWriter的toByteArray方法则把最终修改的字节码以byte数组的形式返回。

​	一个最简单的用法如下面的代码清单6-1所示。

![](https://pic.imgdb.cn/item/61af4f222ab3f51d919fdc1a.jpg)

 	**代码清单6-1　ASM核心类使用示例**

```java
public class FooClassVisitor extends ClassVisitor {
    ...
    // visitXXX() 函数
    ...
}

ClassReader cr = new ClassReader(bytes);
ClassWriter cw = new ClassWriter(cr,
        ClassWriter.COMPUTE_MAXS | ClassWriter.COMPUTE_FRAMES);
ClassVisitor cv = new FooClassVisitor(cw);
cr.accept(cv, 0);
```

​	上面的代码中，ClassReader负责读取类文件字节数组，accept调用之后ClassReader会把解析Class文件过程中的事件源源不断地通知给ClassVisitor对象调用不同的visit方法，ClassVisitor可以在这些visit方法中对字节码进行修改，ClassWriter可以生成最终修改过的字节码。这三个核心类的关系如图6-4所示。

![](https://pic.imgdb.cn/item/61b0222a2ab3f51d91080fda.jpg)

## 6.1.2　ASM操作字节码示例

​	接下来我们用几个简单的例子来演示如何使用ASM的API操作字节码。

### **1.访问类的方法和字段**

​	ASM的visitor设计模式使我们可以很方便地访问类文件中感兴趣的部分，比如类文件的字段和方法列表，以下面的MyMain类为例。

```java
public class MyMain {
    public int a = 0;
    public int b = 1;
    public void test01() {
    }
    public void test02() {
    }
}   
```

​	使用javac将其编译为class文件，可以用下面的ASM代码来输出类的方法和字段列表。

```java
byte[] bytes  = getBytes(); // MyMain.class 文件的字节数组
ClassReader cr = new ClassReader(bytes);
ClassWriter cw = new ClassWriter(0);
ClassVisitor cv = new ClassVisitor(ASM5, cw) {
    @Override
    public FieldVisitor visitField(int access, String name, String desc, String signature, Object value) {
        System.out.println("field: " + name);
        return super.visitField(access, name, desc, signature, value);
    }

    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
          System.out.println("method: " + name);
          return super.visitMethod(access, name, desc, signature, exceptions);
    }
};
cr.accept(cv, ClassReader.SKIP_CODE | ClassReader.SKIP_DEBUG);
```

​	这样就实现了输出类中所有字段和方法的效果，输出结果如下所示。

```java
field: a
field: b
method: <init>
method: test01
method: test02
```

​	值得注意的是，ClassReader类accept方法的第二个参数flags是一个位掩码（bit-mask），可以选择组合的值有下面这些。

* SKIP_DEBUG：跳过类文件中的调试信息，比如行号信息（LineNumberTable）等。
* SKIP_CODE：跳过方法体中的Code属性（方法字节码、异常表等）。
* EXPAND_FRAMES：展开StackMapTable属性。
* SKIP_FRAMES：跳过StackMapTable属性。



​	本例中的flags值为SKIP_DEBUG|SKIP_CODE，因为这里只要求输出字段名和方法名，不需要解析方法Code属性和调试信息。
​	前面有提到ClassVisitor是一个抽象类，我们可以选择关心的事件进行处理，比如例子中覆写了visitField和visitMethod方法，仅对字段和方法进行处理，对于不感兴趣的事件可以选择不覆写。
​	前面介绍的都是Core API的用法，使用Tree API的方式也可以实现同样的效果，代码如下所示。

```java
byte[] bytes = getBytes();

ClassReader cr = new ClassReader(bytes);
ClassNode cn = new ClassNode();
cr.accept(cn, ClassReader.SKIP_DEBUG | ClassReader.SKIP_CODE);

List<FieldNode> fields = cn.fields;
for (int i = 0; i < fields.size(); i++) {
    FieldNode fieldNode = fields.get(i);
    System.out.println("field: " + fieldNode.name);
}
List<MethodNode> methods = cn.methods;
for (int i = 0; i < methods.size(); ++i) {
    MethodNode method = methods.get(i);
    System.out.println("method: " + method.name);
}
ClassWriter cw = new ClassWriter(0);
cr.accept(cn, 0);
byte[] bytesModified = cw.toByteArray();
```

​	Tree API的方式相比Core API使用起来更简单，但是处理速度会慢30%左右，同时会消耗更多的内存，在实际使用过程中可以根据场景做取舍。

### 2.新增一个字段

​	在实际字节码转换中，经常需要给类新增一个字段以存储额外的信息，在ASM中给类新增一个字段非常简单，以下面的MyMain类为例，使用javac编译为class文件。

```java
public class MyMain {
}
```

​	这里采用visitEnd方法中进行添加字段的操作，使用下面的代码可以给MyMain新增一个String类型的xyz字段。

```java
byte[] bytes = FileUtils.readFileToByteArray(new File("./MyMain.class"));
ClassReader cr = new ClassReader(bytes);
ClassWriter cw = new ClassWriter(0);
ClassVisitor cv = new ClassVisitor(ASM5, cw) {
    @Override
    public void visitEnd() {
        super.visitEnd();
        FieldVisitor fv = cv.visitField(Opcodes.ACC_PUBLIC, "xyz", "Ljava/lang/String;", null, null);
        if (fv != null) fv.visitEnd();
    }
};
cr.accept(cv, ClassReader.SKIP_CODE | ClassReader.SKIP_DEBUG);
byte[] bytesModified = cw.toByteArray();
FileUtils.writeByteArrayToFile(new File("./MyMain2.class"), bytesModified);
```

​	使用javap查看MyMain2的字节码，可以看到已经多了一个类型为String的xyz变量，部分字节码如下所示。

```java
public java.lang.String xyz;
descriptor: Ljava/lang/String;
flags: ACC_PUBLIC
```

​	在实际的使用中，为了避免添加的字段名与已有字段重名，一般会增加一个特殊的后缀或者前缀。
​	使用Tree API也可以同样实现这个功能，不用担心调用时机的问题，代码如下所示。

```java
byte[] bytes = FileUtils.readFileToByteArray(new File("./MyMain.class"));
ClassReader cr = new ClassReader(bytes);
ClassNode cn = new ClassNode();
cr.accept(cn, ClassReader.SKIP_DEBUG | ClassReader.SKIP_CODE);

FieldNode fn = new FieldNode(ACC_PUBLIC, "xyz", "Ljava/lang/String;", null, null);
cn.fields.add(fn);

ClassWriter cw = new ClassWriter(0);
cn.accept(cw);
byte[] bytesModified = cw.toByteArray();
FileUtils.writeByteArrayToFile(new File("./MyMain2.class"), bytesModified);
```

​	在实际的使用中，为了避免添加的字段名与已有字段重名，一般会增加一个特殊的后缀或者前缀。
​	使用Tree API也可以同样实现这个功能，不用担心调用时机的问题，代码如下所示。

```java
byte[] bytes = FileUtils.readFileToByteArray(new File("./MyMain.class"));
ClassReader cr = new ClassReader(bytes);
ClassNode cn = new ClassNode();
cr.accept(cn, ClassReader.SKIP_DEBUG | ClassReader.SKIP_CODE);

FieldNode fn = new FieldNode(ACC_PUBLIC, "xyz", "Ljava/lang/String;", null, null);
cn.fields.add(fn);

ClassWriter cw = new ClassWriter(0);
cn.accept(cw);
byte[] bytesModified = cw.toByteArray();
FileUtils.writeByteArrayToFile(new File("./MyMain2.class"), bytesModified);
```

### 3.新增方法

​	这里同样以MyMain类为例，给这个类新增一个xyz方法，如下所示。

```java
public void xyz(int a, String b) {
}
```

​	根据前面的知识可以知道xyz方法的签名为（ILjava/lang/String；）V，使用ASM新增xyz方法的代码如下所示。

```java
byte[] bytes = FileUtils.readFileToByteArray(new File("./MyMain.class"));
ClassReader cr = new ClassReader(bytes);
ClassWriter cw = new ClassWriter(0);
ClassVisitor cv = new ClassVisitor(ASM5, cw) {
    @Override
    public void visitEnd() {
        super.visitEnd();
        MethodVisitor mv = cv.visitMethod(Opcodes.ACC_PUBLIC, "xyz", "(ILjava/lang/String;)V", null, null);
        if (mv != null) mv.visitEnd();
    }
};
cr.accept(cv, ClassReader.SKIP_CODE | ClassReader.SKIP_DEBUG);
byte[] bytesModified = cw.toByteArray();
FileUtils.writeByteArrayToFile(new File("./MyMain2.class"), bytesModified);
```

使用javap查看生成的MyMain2类，确认xyz方法已经生成，方法签名如下所示。

```java
public void xyz(int, java.lang.String);
descriptor: (ILjava/lang/String;)V
flags: ACC_PUBLIC
```

​	Tree API生成方法的方式与生成字段类似，这里不再赘述。

### 4.移除方法和字段

​	前面介绍了如何利用ASM给class文件新增方法和字段，接下来看看如何使用ASM删掉方法和字段。以下面的MyMain类为例，删掉abc字段和xyz方法。

```java
public class MyMain {
    private int abc = 0;
    private int def = 0;
    public void foo() {
    }
    public int xyz(int a, String b) {
        return 0;
    }
}
```

​	如果仔细观察ClassVisitor类的visit方法，会发现visitField、visitMethod等方法是有返回值的，如果这些方法直接返回null，这些字段、方法将从类中被移除，代码如下所示。

```java
byte[] bytes = FileUtils.readFileToByteArray(new File("./MyMain.class"));
ClassReader cr = new ClassReader(bytes);
ClassWriter cw = new ClassWriter(0);
ClassVisitor cv = new ClassVisitor(ASM5, cw) {
    @Override
    public FieldVisitor visitField(int access, String name, String desc, String signature, Object value) {
        if ("abc".equals(name)) {
            return null; // 返回 null
        }
        return super.visitField(access, name, desc, signature, value);
    }

    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
        if ("xyz".equals(name)) {
            return null; // 返回 null
        }
        return super.visitMethod(access, name, desc, signature, exceptions);
    }
};

cr.accept(cv, ClassReader.SKIP_CODE | ClassReader.SKIP_DEBUG);
byte[] bytesModified = cw.toByteArray();
FileUtils.writeByteArrayToFile(new File("./MyMain2.class"), bytesModified);
```

​	使用javap查看MyMain2的字节码，可以看到abc字段和xyz方法已经移除，只剩下def字段和foo方法。

### 5.修改方法内容

​	前面有接触到MethodVisitor类，这个类用来处理访问一个方法触发的事件，与ClassVisitor一样，它也有很多visit方法，这些visit方法也有一定的调用时序，常用的如下所示。

```java
(visitParameter)* 
[visitAnnotationDefault] 
(visitAnnotation | visitParameterAnnotation | visitAttribute)*
[
    visitCode 
    (visitFrame | visit<i>X</i>Insn | visitLabel | visitInsnAnnotation | visitTryCatchBlock | visitTryCatchAnnotation | visitLocalVariable | visitLocalVariableAnnotation | visitLineNumber )* 
    visitMaxs
]
visitEnd
```

​	其中visitCode和visitMaxs可以作为方法体中字节码的开始和结束，visitEnd是MethodVisitor所有事件的结束。

​	以下面的foo方法为例，把方法体的返回值改为a+100。

```java
public class MyMain {
    public static void main(String[] args) {
        System.out.println(new MyMain().foo(1));
    }

    public int foo(int a) {
        return a; // 修改为 return a + 100;
    }
}
```

​	方法体内容“return 100；”对应的字节码指令如下所示。

```java
0: iload_1
1: bipush        100
3: iadd
4: ireturn
```

​	为了替换foo的方法体，一个可选的做法是在ClassVisitor的visitMethod方法返回null以删除原foo方法，然后在visitEnd方法中新增一个foo方法，如下面的代码所示：

```java
byte[] bytes = FileUtils.readFileToByteArray(new File("./MyMain.class"));
ClassReader cr = new ClassReader(bytes);
ClassWriter cw = new ClassWriter(0);
ClassVisitor cv = new ClassVisitor(ASM7, cw) {
    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {

        if ("foo".equals(name)) {
            // 删除 foo 方法
            return null;
        }
        return super.visitMethod(access, name, desc, signature, exceptions);
    }

    @Override
    public void visitEnd() {
        // 新增 foo 方法
        MethodVisitor mv = cv.visitMethod(Opcodes.ACC_PUBLIC, "foo", "(I)I", null, null);

        mv.visitCode();
        mv.visitVarInsn(Opcodes.ILOAD, 1);
        mv.visitIntInsn(Opcodes.BIPUSH, 100);
        mv.visitInsn(Opcodes.IADD);
        mv.visitInsn(Opcodes.IRETURN);
        mv.visitEnd();
    }
};
cr.accept(cv, 0);
byte[] bytesModified = cw.toByteArray();
FileUtils.writeByteArrayToFile(new File("./MyMain.class"), bytesModified);
```

​	使用javap查看生成的foo方法字节码，可以看到方法字节码已经被替换，如下所示。

```java
public int foo();
descriptor: ()I
flags: ACC_PUBLIC
Code:
stack=0, locals=0, args_size=1
 0: iload_1
 1: bipush        100
 3: iadd
 4: ireturn
```

​	使用java运行MyMain，会发现抛出了ClassFormatError异常，提示入参无法放到局部变量表locals中，详细的错误信息如下所示。

```java
java -cp . MyMain    
Error: A JNI error has occurred, please check your installation and try again
Exception in thread "main" java.lang.ClassFormatError: Arguments can't fit into locals in class file MyMain
```

​	再回过头来查看生成的字节码，会发现它的stack和locals都等于0，从前面的内容可以知道Java虚拟机根据字节码中stack和locals的值来分配操作数栈和局部变量表的空间，如果两个值都等于0则不能加载操作数和存储方法参数。

​	从源代码可以分析出，最大栈的大小为2（a，100），局部变量表的大小为2（this，a）。一个可选的办法是在“mv.visitEnd（）；”代码之前新增“mv.visitMaxs（2，2）；”以手动指定stack和locals的大小。

​	另一个方法是让ASM自动计算stack和locals，这与ClassWriter构造器方法参数有关，如下所示。

* new ClassWriter（0）：这种方式不会自动计算操作数栈和局部变量表的大小，需要我们手动指定。
* new ClassWriter（ClassWriter.COMPUTE_MAXS）：这种方式会自动计算操作数栈和局部变量表的大小，前提是需要调用visitMaxs方法来触发计算上述两个值，参数值可以随便指定。
* new ClassWriter（ClassWriter.COMPUTE_FRAMES）：不仅会计算操作数栈和局部变量表，还会自动计算StackMapFrames。在Java 6之后JVM在class文件的Code属性中引入了StackMapTable属性，作用是为了提高JVM在类型检查时验证过程的效率，里面记录的是一个方法中操作数栈与局部变量区的类型在一些特定位置的状态。



​	虽然COMPUTE_FRAMES隐式地包含了COMPUTE_MAXS，一般在使用中还是会同时指定，调用的代码如下所示。

```java
new ClassWriter(ClassWriter.COMPUTE_MAXS | ClassWriter.COMPUTE_FRAMES)
```

​	因为stack和locals的计算复杂、容易出错，在正常使用中强烈建议使用ASM的COMPUTE_MAXS和COMPUTE_FRAMES，虽然有一点点性能损耗，但是代码更加清晰易懂，且更易于维护。

```java
byte[] bytes = FileUtils.readFileToByteArray(new File("./MyMain.class"));
ClassReader cr = new ClassReader(bytes);
// 指定 ClassWriter 自动计算
ClassWriter cw = new ClassWriter(ClassWriter.COMPUTE_MAXS | ClassWriter.COMPUTE_FRAMES);
ClassVisitor cv = new ClassVisitor(ASM7, cw) {
    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {

        if ("foo".equals(name)) {
            // 删除 foo 方法
            return null;
        }
        return super.visitMethod(access, name, desc, signature, exceptions);
    }

    @Override
    public void visitEnd() {
        // 新增 foo 方法
        MethodVisitor mv = cv.visitMethod(Opcodes.ACC_PUBLIC, "foo", "(I)I", null, null);

        mv.visitCode();
        mv.visitVarInsn(Opcodes.ILOAD, 1);
        mv.visitIntInsn(Opcodes.BIPUSH, 100);
        mv.visitInsn(Opcodes.IADD);
        mv.visitInsn(Opcodes.IRETURN);
        // 触发计算
        mv.visitMaxs(0, 0);
        mv.visitEnd();
    }
};
cr.accept(cv, 0);
byte[] bytesModified = cw.toByteArray();
FileUtils.writeByteArrayToFile(new File("./MyMain.class"), bytesModified);
```

​	这个时候使用java执行MyMain，就可以正常输出结果，如下所示。

```java
java -cp . MyMain
101
```

### 6.AdviceAdapter使用

​	AdviceAdapter是一个抽象类，继承自MethodVisitor，可以很方便地在方法的开始和结束前插入代码，它的两个核心方法介绍如下所示。

* onMethodEnter：方法开始或者构造器方法中父类的构造器调用以后被回调。
* onMethodExit：正常退出和异常退出时被调用。正常退出指的是遇到RETURN、ARETURN、LRETURN等方法正常返回的情况。异常退出指的是遇到ATHROW指令，有异常抛出方法返回的情况。

```java
public void foo() {
    System.out.println("hello foo");
}
```

​	接下来用AdviceAdapter在函数开始和结束的时候都加上一行打印。

```java
byte[] bytes = FileUtils.readFileToByteArray(new File("./MyMain.class"));

ClassReader cr = new ClassReader(bytes);
ClassWriter cw = new ClassWriter(ClassWriter.COMPUTE_MAXS | ClassWriter.COMPUTE_FRAMES);
ClassVisitor cv = new ClassVisitor(ASM7, cw) {
    @Override
    public MethodVisitor visitMethod(int access, final String name, String desc, String signature, String[] exceptions) {

        MethodVisitor mv = super.visitMethod(access, name, desc, signature, exceptions);
        if (!"foo".equals(name)) return mv;
        
        return new AdviceAdapter(ASM7, mv, access, name, desc) {
            @Override
            protected void onMethodEnter() {
                // 新增 System.out.println("enter " +  name);
                super.onMethodEnter();
                mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
                mv.visitLdcInsn("enter " + name);
                mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
            }

            @Override
            protected void onMethodExit(int opcode) {
               // 新增 System.out.println("[normal,err] exit " +  name);
                super.onMethodExit(opcode);
                mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
                if (opcode == Opcodes.ATHROW) {
                    mv.visitLdcInsn("err exit " + name);
                } else {
                    mv.visitLdcInsn("normal exit " + name);
                }
                mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
            }
        };
    }
};
cr.accept(cv, 0);
byte[] bytesModified = cw.toByteArray();
FileUtils.writeByteArrayToFile(new File("./MyMain.class"), bytesModified);
```

​	运行生成的MyMain类，就可以看到enter和exit语句已经生效了，运行结果如下所示。

```java
java -cp . MyMain    
enter foo
hello foo
normal exit foo
```

### 7.给方法加上try catch

​	很显然上一个小节的代码无法在代码抛出未捕获异常时输出err quit，比如把foo代码做细微修改，如下所示。

```java
public void foo() {
    System.out.println("step 1");
    int a = 1 / 0;
    System.out.println("step 2");
}
```

​	经过上个例子的字节码改写以后，执行的结果输出如下所示。

```java
java -cp . MyMain  
enter foo
step 1
Exception in thread "main" java.lang.ArithmeticException: / by zero
        at MyMain.foo(MyMain.java:24)
        at MyMain.main(MyMain.java:8)
```

​	可以看到并没有如期输出“err exit foo”，因为在字节码中并没有出现显式的ATHROW指令抛出异常，自然无法添加相应的输出语句。为了达到这个效果，需要把方法体用try/finally语句块包裹起来。

​	这里需要介绍ASM的Label类，与它的英文含义一样，可以给字节码指令地址打标签，标记特定的字节码位置，用于后续跳转等。新增一个Label可以用MethodVisitor的visitLabel方法，如下所示。

```java
Label startLabel = new Label();
mv.visitLabel(startLabel);
```

​	前面章节介绍过，JVM的异常处理是通过异常表来实现的，try-catch-finally语句块实际上是标定了异常处理的范围。ASM中可以用visitTryCatchBlock方法来给一段代码块增加异常表，它的方法签名如下所示。

```java
public void visitTryCatchBlock(Label start, Label end, Label handler, String type)
```

​	其中start、end表示异常表开始和结束的位置，handler表示异常发生后需要跳转到哪里继续执行，可以理解为catch语句块开始的位置，type是异常的类型。

​	为了给整个方法体包裹try-catch语句，start Label应该放在方法visitCode之后，end Label则放在visitMaxs调用之前，代码如下所示。

```java
Label startLabel = new Label();

@Override
protected void onMethodEnter() {
    super.onMethodEnter();
    mv.visitLabel(startLabel);

    mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
    mv.visitLdcInsn("enter " + name);
    mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
}

@Override
public void visitMaxs(int maxStack, int maxLocals) {
    // 生成异常表
    Label endLabel = new Label();
    mv.visitTryCatchBlock(startLabel, endLabel, endLabel, null);
    mv.visitLabel(endLabel);
    
    // 生成异常处理代码块
    finallyBlock(ATHROW);
    mv.visitInsn(ATHROW);
    super.visitMaxs(maxStack, maxLocals);
}

private void finallyBlock(int opcode) {
    mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
    if (opcode == Opcodes.ATHROW) {
        mv.visitLdcInsn("err exit " + name);
    } else {
        mv.visitLdcInsn("normal exit " + name);
    }
    mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
}

@Override
protected void onMethodExit(int opcode) {
    super.onMethodExit(opcode);
    // 处理正常返回的场景
    if (opcode != ATHROW) finallyBlock(opcode);
}
```

​	前面介绍过onMethodExit在方法正常退出和异常退出时都会被调用。添加完异常处理表以后，程序异常退出时都会进入异常代码处理模块，为了避免重复处理，在onMethodExit中只会处理正常退出的情况，不必处理ATHROW指令。
​	执行使用Java修改过的MyMain类，可以看到已经有“err exit foo”输出了。

```java
java -cp . MyMain 
enter foo
step 1
err exit foo
Exception in thread "main" java.lang.ArithmeticException: / by zero
        at MyMain.foo(MyMain.java:24)
        at MyMain.main(MyMain.java:8)
```

## 6.2　Javassist介绍

​	前面介绍的ASM入门门槛还是挺高的，需要跟底层的字节码指令打交道，优点是小巧、性能好。Javassist是一个性能比ASM稍差但使用起来简单很多的字节码操作库，不需要使用者掌握字节码指令，由东京工业大学的数学和计算机科学系的教授Shigeru Chiba开发。

​	本节将分为两个部分来讲解，第一部分是Javassist核心API介绍，第二部分是Javassist操作class文件的代码示例。

### 6.2.1 Javassist核心API

​	在Javassist中每个需要编辑的class都对应一个CtClass实例，CtClass的含义是编译时的类（compile time class），这些类会存储在ClassPool中。ClassPool是一个容器，存储了一系列CtClass对象。

​	Javassist的API与Java反射API比较相似，Java类包含的字段、方法在Javassist中分别对应CtField和CtMethod，通过CtClass对象就可以给类新增字段、修改方法了。Javassist核心API如图6-5所示。

![](https://pic.imgdb.cn/item/61b4b57d2ab3f51d911d9572.jpg)

​	CtClass的writeFile可以将生成的class文件输出到指定目录中，比如新建一个Hello类可以用下面的代码实现。

```java
ClassPool cp = ClassPool.getDefault();
CtClass ct = cp.makeClass("ya.me.Hello");
ct.writeFile("./out");
```

​	运行上面的代码，out目录下就生成了一个Hello类，内容如下所示。

```java
package ya.me;

public class Hello {
    public Hello() {
    }
}
```

### 6.2.2　Javassist操作字节码示例

#### 1.给已有类新增方法

​	新建一个MyMain类，如下所示。

```java
public class MyMain {
}
```

​	接下来用Javassist给它新增一个foo方法，调用CtClass的addMethod可以实现给类新增方法的功能，代码如下所示。

```java
ClassPool cp = ClassPool.getDefault();
cp.insertClassPath("/path/to/MyMain.class");
CtClass ct = cp.get("MyMain");
CtMethod method = new CtMethod(
        CtClass.voidType,
        "foo",
        new CtClass[]{CtClass.intType, CtClass.intType},
        ct);
method.setModifiers(Modifier.PUBLIC);
ct.addMethod(method);
ct.writeFile("./out");
```

​	查看生成MyMain类可以看到生成了foo方法。

#### **2.修改方法体**

​	CtMethod提供了几个实用的方法来修改方法体：

* setBody方法用来替换整个方法体，它接收一段源代码字符串，Javassist会将这段源码字符串编译为字节码，替换原有的方法体。
* insertBefore、insertAfter方法可以实现在方法开始和结束的地方插入语句。



​	如果想把foo方法体修改为“return 0；”，可以做如下修改。

```java
CtMethod method = ct.getMethod("foo", "(II)I");
method.setBody("return 0;");
```

​	如果想把foo方法体的“a+b；”修改为“a*b；”，比较直观的想法是调用`method.setBody（"return a*b；"）`。实际运行时会报错，提示找不到字段a，错误堆栈如下所示。

```java
Exception in thread "main" javassist.CannotCompileException: [source error] no such field: a
    at javassist.CtBehavior.setBody(CtBehavior.java:474)
    at javassist.CtBehavior.setBody(CtBehavior.java:440)
    at JavassistTest2.main(JavassistTest2.java:17)
Caused by: compile error: no such field: a
```

​	这是因为源代码在javac编译以后，抹去了局部变量名字，只留下类型和局部变量表的位置，比如上面的a和b对应局部变量表1和2的位置。在Javassist中访问方法参数使用$0$1...，而不是直接使用变量名，将上面的代码修改为如下所示。

```java
method.setBody("return $1 * $2;");
```

生成新的MyMain类中的foo方法如下所示。

```java
public int foo(int var1, int var2) {
    return var1 * var2;
}
```

​	除了方法的参数，Javassist定义了以$开头的特殊标识符，如表6-1所示。

![](https://pic.imgdb.cn/item/61b4b7ca2ab3f51d911ee61f.jpg)

​	下面来逐一介绍。

（1）$0$1$2...参数

​	$0$1$2等表示方法参数，非静态方法0对应于this，如果是静态方法$0不可用，从$1开始依次表示方法参数。

（2）$args参数

​	$args变量表示所有参数的数组，它是一个Object类型的数组，如果参数中有原始类型，会被转为对应的包装类型，比如上面foo（int a，int b）对应的$args如下所示。

```java
new Object[]{ new Integer(a), new Integer(b) }
```

（3）$$参数

​	$$参数表示所有的参数的展开，参数直接用逗号分隔，foo（$$）相当于foo（$1，$2，...）。

（4）$cflow参数

​	$cflow是“control flow”的缩写，这是一个只读的属性，表示某方法递归调用的深度。一个典型的使用场景是监控某递归方法执行的时间，如果只想记录一次最顶层调用的时间，可以使用$cflow来判断当前递归调用的深度，如果不是最顶层调用则忽略记录时间。比如下面的计算fibonacci数列的方法。

​	如果只想在第一次调用的时候执行打印，可以对字节码做如下修改。

```java
CtMethod method = ct.getMethod("fibonacci", "(I)J");
method.useCflow("fibonacci");
method.insertBefore(
        "if ($cflow(fibonacci) == 0) {" +
            "System.out.println(\"fibonacci init \" + $1);" +
        "}"
);
```

​	执行生成的MyMain，可以看到只输出了一次打印：

```java
java -cp /path/to/javassist.jar:. MyMain
fibonacci init 10
```

（5）$_参数

​	CtMethod的insertAfter（）方法在目标方法的末尾插入一段代码。$_用来表示方法的返回值，在insertAfter方法中可以引用，以下面的代码为例。

```java
method.insertAfter("System.out.println(\"result: \"  + $_);");
```

​	生成的class文件反编译结果如下所示。

```java
public int foo(int a, int b) {
    int var4 = a + b;
    System.out.println("result: " + var4);
    return var4;
}
```

​	细心的读者看到这里会有疑问，如果是方法异常退出，它的方法返回值是什么呢？以下面foo代码为例。

```java
public int foo(int a, int b) {
    int c = 1 / 0;
    return a + b;
}
```

​	执行上面的改写后，反编译以后代码如下所示。

```java
public int foo(int a, int b) {
    int c = 1 / 0;
    int var5 = a + b;
    System.out.println(var5);
    return var5;
}
```

​	在这种情况下，代码块抛出异常时是无法执行插入语句的。如果想代码抛出异常的时候也能执行，就需要把insertAfter的第二个参数asFinally设置为true，如下面代码所示。

```java
method.insertAfter("System.out.println(\"result: \"  + $_);", true);
```

​	执行输出结果如下，可以看到在抛出异常的情况下也输出了预期的打印语句。

```java
result: 0
Exception in thread "main" java.lang.ArithmeticException: / by zero
        at MyMain.foo(MyMain.java:9)
        at MyMain.main(MyMain.java:6)
```

（6）其他参数

​	还有几个Javassist提供的内置变量（$r等）用的非常少，这里不再介绍，具体可以查看Javassist的官网。

# 7.Java Instrumentation原理

这一章我们来介绍Java中强大的Instrumentation机制，Instrumentation看起来比较神秘，很少有书会详细介绍。日常工作中用到的很多工具其实都是基于Instrumentation来实现的，比如下面这些：

* APM产品：Pinpoint、SkyWalking、newrelic等。
* 热部署工具：Intellij idea的HotSwap、Jrebel等。
* Java诊断工具：Arthas等。

## 7.1　Java Instrumentation简介

​	JDK从1.5版本开始引入了java.lang.instrument包，开发者可以更方便的实现字节码增强。其核心功能由java.lang.instrument.Instrumentation提供，这个接口的方法提供了注册类文件转换器、获取所有已加载的类等功能，允许我们在对已加载和未加载的类进行修改，实现AOP、性能监控等功能。

​	Instrumentation接口的常用方法如下所示。

```java
void addTransformer(ClassFileTransformer transformer, boolean canRetransform);

void retransformClasses(Class<?>... classes) throws UnmodifiableClassException;

Class[] getAllLoadedClasses()

boolean isRetransformClassesSupported();
```

​	它的addTransformer方法给Instrumentation注册一个类型为ClassFileTransformer的类文件转换器。ClassFileTransformer接口只有一个transform方法，接口定义如下所示。

```java
public interface ClassFileTransformer {
    byte[] transform(
            ClassLoader loader,
            String className,
            Class<?> classBeingRedefined,
            ProtectionDomain protectionDomain,
            byte[] classfileBuffer
    ) throws IllegalClassFormatException;
}
```

​	其中className参数表示当前加载类的类名，classfileBuffer参数是待加载类文件的字节数组。调用addTransformer注册transformer以后，后续所有JVM加载类都会被它的transform方法拦截，这个方法接收原类文件的字节数组，在这个方法中可以做任意的类文件改写，最后返回转换过的字节数组，由JVM加载这个修改过的类文件。如果transform方法返回null，表示不对此类做处理，如果返回值不为null，JVM会用返回的字节数组替换原来类的字节数组。

​	Instrumentation接口的retransformClasses方法对JVM已经加载的类重新触发类加载。getAllLoadedClasses方法用于获取当前JVM加载的所有类对象。isRetransform-ClassesSupported方法返回一个boolean值表示当前JVM配置是否支持类重新转换的特性。

​	Instrumentation有两种使用方式：第一种方式是在JVM启动的时候添加一个Agent的jar包；第二种方式是在JVM运行以后在任意时刻通过Attach API远程加载Agent的jar包。接下来分开进行介绍。

## 7.2　Instrumentation与-javaagent启动参数

​	Instrumentation的第一种使用方式是通过JVM的启动参数-javaagent来启动，一个典型的使用方式如下所示。

```java
java -javaagent:myagent.jar MyMain
```

​	为了能让JVM识别到Agent的入口类，需要在jar包的MANIFEST.MF文件中指定Premain-Class等信息，一个典型的生成好的MANIFEST.MF内容如下所示。

```java
Premain-Class: me.geek01.javaagent.AgentMain
Agent-Class: me.geek01.javaagent.AgentMain
Can-Redefine-Classes: true
Can-Retransform-Classes: true
```

​	其中AgentMain类有一个静态的premain方法，JVM在类加载时会先执行AgentMain类的premain方法，再执行Java程序本身的main方法，这就是premain名字的来源。在premain方法中可以对class文件进行修改。这种机制可以认为是虚拟机级别的AOP，无须对原有应用做任何修改就可以实现类的动态修改和增强。

​	premain方法签名如下所示。

```java
public static void premain(String agentArgument, Instrumentation instrumentation) throws Exception
```

​	第一个参数agentArgument是agent的启动参数，可以在JVM启动时指定。以下面的启动方式为例，它的agentArgument的值为"appId：agent-demo，agentType：singleJar"。

```java
java -javaagent:<jarfile>=appId:agent-demo,agentType:singleJar test.jar
```

![](https://pic.imgdb.cn/item/61c0723c2ab3f51d911dd04e.jpg)

​	下面使用Instrumentation的方式实现一个简单的方法调用栈跟踪，在每个方法进入和结束的地方都打印一行日志，实现调用过程的追踪效果，测试代码如下面代码清单7-1所示。

```java
public class MyTest {
    public static void main(String[] args) {
        new MyTest().foo();
    }
    public void foo() {
        bar1();
        bar2();
    }

    public void bar1() {
    }

    public void bar2() {
    }
}
```

​	这里需要用到前面介绍的ASM相关的知识。新建一个AdviceAdapter的子类MyMethodVisitor，覆写onMethodEnter、onMethodExit方法，核心逻辑如下面代码清单7-2所示。

```java
public class MyMethodVisitor extends AdviceAdapter {
    @Override
    protected void onMethodEnter() {
        // 在方法开始处插入 <<<enter xxx
        mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
        mv.visitLdcInsn("<<<enter " + this.getName());
        mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
        super.onMethodEnter();
    }
    @Override
    protected void onMethodExit(int opcode) {
        super.onMethodExit(opcode);
        // 在方法结束处插入 >>>exit xxx
        mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
        mv.visitLdcInsn(">>>exit " + this.getName());
        mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
    }
}	
```

​	新建一个ClassVisitor的子类MyClassVisitor，覆写visitMethod方法，返回自定义的MethodVisitor，如下面代码清单7-3所示。

```java
public class MyClassVisitor extends ClassVisitor {

    public MyClassVisitor(ClassVisitor classVisitor) {
        super(ASM7, classVisitor);
    }
    @Override
    public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {
        MethodVisitor mv = super.visitMethod(access, name, descriptor, signature, exceptions);
        if (name.equals("<init>")) return mv;
        return new MyMethodVisitor(mv, access, name, descriptor);
    }
}
```

​	接下来新建一个MyClassFileTransformer，这个类实现了ClassFileTransformer接口，在transform接口中使用ASM实现class文件的转换，如下面代码清单7-4所示。

​	代码清单7-4　自定义ClassFileTransformer

```java
public class MyClassFileTransformer implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                            ProtectionDomain protectionDomain, byte[] bytes) throws IllegalClassFormatException {
        if (!"MyTest".equals(className)) return bytes;
        ClassReader cr = new ClassReader(bytes);
        ClassWriter cw = new ClassWriter(cr, ClassWriter.COMPUTE_FRAMES);
        ClassVisitor cv = new MyClassVisitor(cw);
        cr.accept(cv, ClassReader.SKIP_FRAMES | ClassReader.SKIP_DEBUG);
        return cw.toByteArray();
    }
}
```

​	接下来新建一个AgentMain类，在其中实现premain方法的逻辑，调用addTransformer方法将MyClassFileTransformer注册到Instrumentation中，如下面代码清单7-5所示。

```java
public class AgentMain {
    public static void premain(String agentArgs, Instrumentation inst) throws ClassNotFoundException, UnmodifiableClassException {
        inst.addTransformer(new MyClassFileTransformer(), true);
    }
}
```

​	接下来将上面的代码打包为Agent的jar包，使用maven-jar-plugin插件可以比较方便地实现这个功能，在manifestEntries属性中指定Premain-Class为前面新建的AgentMain类名，plugin的配置xml如下面代码清单7-6所示。

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <configuration>
        <archive>
          <manifestEntries>
            <Agent-Class>me.geek01.javaagent.AgentMain</Agent-Class>
            <Premain-Class>me.geek01.javaagent.AgentMain</Premain-Class>
            <Can-Redefine-Classes>true</Can-Redefine-Classes>
            <Can-Retransform-Classes>true</Can-Retransform-Classes>
          </manifestEntries>
        </archive>
      </configuration>
    </plugin>
  </plugins>
</build>
```

​	执行mvn clean package生成jar包，添加-javaagent参数执行MyTest类，如下所示。

```
java -javaagent:/path/to/agent.jar MyTest
```

​	可以看到终端输出了方法调用的打印语句，结果如下所示。

```java
<<<enter main
<<<enter foo
<<<enter bar1
>>>exit bar1
<<<enter bar2
>>>exit bar2
>>>exit foo
>>>exit main
```

​	通过上面的方式，我们在不修改MyTest类源码的情况下实现了调用链跟踪的效果，这个例子是APM实现的原型，更加完善的调用链跟踪实现会在后面的APM章节中介绍。接下来我们来看第二种Instrumentation使用方式。

## 7.3　JVM Attach API介绍

​	在JDK5中，开发者只能在JVM启动时指定一个javaagent，在premain中操作字节码，这种Instrumentation方式仅限于main方法执行前，存在很大的局限性。从JDK6开始引入了动态Attach Agent的方案，可以在JVM启动后任意时刻通过Attach API远程加载Agent的jar包，比如大名鼎鼎的arthas工具就是基于Attach API实现的。

​	加载Agent的jar包只是Attach API的功能之一，我们常用jstack、jps、jmap工具都是利用Attach API来实现的。这个小节会先介绍Attach API的使用，随后会结合跨进程通信中的信号和UNIX域套接字来看Attach API的实现原理。

### 7.3.1　JVM Attach API基本使用

​	下面以一个实际的例子来演示动态Attach API的使用，测试代码中有一个main方法，每3秒输出foo方法的返回值100，接下来动态Attach上MyTestMain进程，修改foo的字节码，让foo方法返回50，测试代码如下面代码清单7-7所示。

```java
public class MyTestMain {
    public static void main(String[] args) throws InterruptedException {
        while (true) {
            System.out.println(foo());
            TimeUnit.SECONDS.sleep(3);
        }
    }

    public static int foo() {
        return 100; // 修改后 return 50;
    }
}
```

具体分为下面这几个步骤。
1）编写实现自定义ClassFileTransformer，通过ASM对foo方法做注入，核心代码如下面代码清单7-8所示。
代码清单7-8　foo方法字节码注入

```java
public class MyMethodVisitor extends AdviceAdapter {
    @Override
    protected void onMethodEnter() {
        // 在方法开始插入 return 50;
        mv.visitIntInsn(BIPUSH, 50);
        mv.visitInsn(IRETURN);
    }
}
public class MyClassVisitor extends ClassVisitor {
    @Override
    public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {
        MethodVisitor mv = super.visitMethod(access, name, descriptor, signature, exceptions);
        // 只转换 foo 方法
        if ("foo".equals(name)) {
            return new MyMethodVisitor(mv, access, name, descriptor);
        }
        return mv;
    }
}
public class MyClassFileTransformer implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                            ProtectionDomain protectionDomain, byte[] bytes) throws IllegalClassFormatException {
        if (!"MyTestMain".equals(className)) return bytes;
        ClassReader cr = new ClassReader(bytes);
        ClassWriter cw = new ClassWriter(cr, ClassWriter.COMPUTE_FRAMES);
        ClassVisitor cv = new MyClassVisitor(cw);
        cr.accept(cv, ClassReader.SKIP_FRAMES | ClassReader.SKIP_DEBUG);
        return cw.toByteArray();
    }
}
```

​	2）实现agentmain方法。前面介绍的启动时加载agent会调用premain方法，动态Attach的agent会执行agentmain方法，方法参数和含义跟premain类似，agentmain的实现代码如下面代码清单7-9所示。

```java
public class AgentMain {
    public static void agentmain(String agentArgs, Instrumentation inst) throws ClassNotFoundException, UnmodifiableClassException {
        System.out.println("agentmain called");
        inst.addTransformer(new MyClassFileTransformer(), true);
        Class classes[] = inst.getAllLoadedClasses();
        for (int i = 0; i < classes.length; i++) {
            if (classes[i].getName().equals("MyTestMain")) {
                System.out.println("Reloading: " + classes[i].getName());
                inst.retransformClasses(classes[i]);
                break;
            }
        }
    }
}
```

​	3）因为是跨进程通信，Attach的发起端是一个独立的java程序，这个java程序会调用VirtualMachine.attach方法开始和目标JVM进行跨进程通信。

```java
public class MyAttachMain {
    public static void main(String[] args) throws Exception {
        VirtualMachine vm = VirtualMachine.attach(args[0]);
        try {
            vm.loadAgent("/path/to/agent.jar");
        } finally {
            vm.detach();
        }
    }
}
```

​	使用jps查询到MyTestMain的进程pid，使用java执行MyAttachMain类，如下所示。

```java
java -cp /path/to/your/tools.jar:. MyAttachMain pid
```

```java
java -cp . MyTestMain

100
100
100
agentmain called
Reloading: MyTestMain
50
50
50
```

### 7.3.2　JVM Attach API的底层原理

​	JVM Attach API的实现主要基于信号和UNIX域套接字，接下来详细介绍这两部分的内容。

​	信号是某事件发生时对进程的通知机制，也被称为“软件中断”。信号可以看作一种非常轻量级的进程间通信，信号由一个进程发送给另外一个进程，只不过是经由内核作为一个中间人转发，信号最初的目的是用来指定杀死进程的不同方式。

​	每个信号都有一个名字，以“SIG”开头，最熟知的信号应该是SIGINT，我们在终端执行某个应用程序的过程中按下Ctrl+C一般会终止正在执行的进程，这是因为按下Ctrl+C会发送SIGINT信号给目标程序。每个信号都有一个唯一的数字标识，从1开始，常见的信号量列表如表7-1所示。

![](https://pic.imgdb.cn/item/61c079052ab3f51d9120970f.jpg)

​	在Linux中，一个前台进程可以使用Ctrl+C进行终止，对于后台进程需要使用kill加进程号的方式来终止，kill命令是通过发送信号给目标进程来实现终止进程的功能。默认情况下，kill命令发送的是编号为15的SIGTERM信号，这个信号可以被进程捕获，选择忽略或正常退出。目标进程如果没有自定义处理这个信号就会被终止。对于那些忽略SIGTERM信号的进程，可以指定编号为9的SIGKILL信号强行杀死进程，SIGKILL信号不能被忽略也不能被捕获和自定义处理。

​	新建一个signal.c文件，自定义处理SIGQUIT、SIGINT、SIGTERM信号，如下面代码清单7-10所示。

​	代码清单7-10　信号处理示例

```java
static void signal_handler(int signal_no) {
    if (signal_no == SIGQUIT) {
        printf("quit signal receive: %d\n", signal_no);
    } else if (signal_no == SIGTERM) {
        printf("term signal receive: %d\n", signal_no);
    } else if (signal_no == SIGINT) {
        printf("interrupt signal receive: %d\n", signal_no);
    }
}
int main() {
    signal(SIGQUIT, signal_handler);
    signal(SIGINT, signal_handler);
    signal(SIGTERM, signal_handler);
    for (int i = 0;; i++) {
        printf("%d\n", i);
        sleep(3);
    }
}
```

​	使用gcc编译上面的signal.c文件，然后执行生成的signal可执行文件，如下所示。

```java
gcc signal.c -o signal
./signal
```

​	这种情况下在终端中输入Ctrl+C，kill-3，kill-15都没有办法杀掉这个进程，只能用kill-9，如下所示。

```java
0
^Cinterrupt signal receive: 2     // Ctrl+C
1
2
term signal receive: 15           // kill pid
3
4
5
quit signal receive: 3             // kill -3 
6
7
8
[1]    46831 killed     ./signal  // kill -9 成功杀死进程
```

​	JVM对SIGQUIT的默认行为是打印所有运行线程的堆栈信息，在类UNIX系统中，可以通过使用命令kill-3 pid来发送SIGQUIT信号。运行上面的MyTestMain，使用jps找到这个JVM的进程pid，执行kill-3 pid，在终端就可以看到打印了所有线程的调用栈信息。

```java
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.51-b03 mixed mode):

"Service Thread" #8 daemon prio=9 os_prio=31 tid=0x00007fe060821000 nid=0x4403 runnable [0x0000000000000000]
    java.lang.Thread.State: RUNNABLE
...
"Signal Dispatcher" #4 daemon prio=9 os_prio=31 tid=0x00007fe061008800 nid=0x3403 waiting on condition [0x0000000000000000]
    java.lang.Thread.State: RUNNABLE
"main" #1 prio=5 os_prio=31 tid=0x00007fe060003800 nid=0x1003 waiting on condition [0x000070000d203000]
    java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at java.lang.Thread.sleep(Thread.java:340)
        at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
        at MyTestMain.main(MyTestMain.java:10)
```

​	接下来我们来看UNIX域套接字（UNIX Domain Socket）的概念。使用TCP和UDP进行socket通信是一种广为人知的socket使用方式，除了这种方式以外还有一种称为UNIX域套接字的方式，可以实现同一主机上的进程间通信。虽然使用127.0.01环回地址也可以通过网络实现同一主机的进程间通信，但UNIX域套接字更可靠、效率更高。Docker守护进程（Docker daemon）使用了UNIX域套接字，容器中的进程可以通过它与Docker守护进程进行通信。MySQL同样提供了域套接字进行访问的方式。

​	在UNIX世界，一切皆文件，UNIX域套接字也是一个文件，下面是在某文件夹下执行ls-l命令的输出结果。

```java
ls -l
srwxrwxr-x. 1 ya ya        0 9月   8 00:26 tmp.sock
drwxr-xr-x. 5 ya ya    4096 11月  29 09:18 tmp
-rw-r--r--. 1 ya ya   14919 10月  31 2017  test1.png
```

​	ls-l命令输出的第一个字符表示文件的类型，“s”表示这是一个UNIX域套接字，“d”表示这个是一个文件夹，“-”表示这是一个普通文件。

​	两个进程通过读写这个文件就实现了进程间的信息传递。文件的拥有者和权限决定了谁可以读写这个套接字。

​	UNIX域套接字与普通套接字的对比和区别是什么？

* UNIX域套接字更加高效，它不用进行协议处理，不需要计算序列号，也不需要发送确认报文，只需要读写数据即可。
* UNIX域套接字是可靠的，不会丢失数据，普通套接字是为不可靠通信设计的。
* UNIX域套接字的代码可以非常简单地修改为普通套接字。



	.
	├── client.c
	└── server.c
​	其中server.c充当UNIX域套接字服务器，启动后会在当前目录生成一个名为tmp.sock的UNIX域套接字文件，它读取客户端写入的内容并输出到终端。server.c的代码如下面代码清单7-11所示。

```java
int main() {
    int fd = socket(AF_UNIX, SOCK_STREAM, 0);
    struct sockaddr_un addr;
    memset(&addr, 0, sizeof(addr));
    addr.sun_family = AF_UNIX;
    strcpy(addr.sun_path, "tmp.sock");
    int ret = bind(fd, (struct sockaddr *) &addr, sizeof(addr));
    listen(fd, 5)
    
    int accept_fd;
    char buf[100];
    while (1) {
        accept_fd = accept(fd, NULL, NULL)) == -1);
        while ((ret = read(accept_fd, buf, sizeof(buf))) > 0) {
            // 输出客户端传过来的数据
            printf("receive %u bytes: %s\n", ret, buf);
        }
}
```

客户端client.c代码如下面代码清单7-12所示。
代码清单7-12　UNIX域套接字客户端代码

```java
int main() {
    int fd = socket(AF_UNIX, SOCK_STREAM, 0);
    struct sockaddr_un addr;
    memset(&addr, 0, sizeof(addr));
    addr.sun_family = AF_UNIX;
    strcpy(addr.sun_path, "tmp.sock");

    connect(fd, (struct sockaddr *) &addr, sizeof(addr));
    
    int rc;
    char buf[100];
    // 读取终端标准输入的内容，写入 UNIX 域套接字文件中
    while ((rc = read(STDIN_FILENO, buf, sizeof(buf))) > 0) {
        write(fd, buf, rc);
    }
}
```

​	在命令行中进行编译和执行，如下所示。

```java
gcc server.c -o server
gcc client.c -o client
```

![](https://pic.imgdb.cn/item/61c07bb82ab3f51d9121b747.jpg)

​	接下来介绍JVM Attach API的执行过程。使用java执行本节开头的MyAttachMain，当指定一个不存在的JVM进程时，会出现如下的错误。

```java
java -cp /path/to/your/tools.jar:. MyAttachMain 1234
Exception in thread "main" java.io.IOException: No such process
    at sun.tools.attach.LinuxVirtualMachine.sendQuitTo(Native Method)
    at sun.tools.attach.LinuxVirtualMachine.<init>(LinuxVirtualMachine.java:91)
    at sun.tools.attach.LinuxAttachProvider.attachVirtualMachine(LinuxAttachProvider.java:63)
    at com.sun.tools.attach.VirtualMachine.attach(VirtualMachine.java:208)
    at MyAttachMain.main(MyAttachMain.java:8)
```

​	可以看到VirtualMachine.attach最终调用了sendQuitTo方法，这是一个native方法，底层就是发送了SIGQUIT号给目标JVM进程。

​	前面信号部分我们介绍过，JVM对SIGQUIT的默认行为是dump当前的线程堆栈，那为什么调用VirtualMachine.attach没有输出调用栈堆栈呢？

假设目标进程为12345，Attach的详细过程如下。
1）Attach端先检查临时文件目录是否有.java_pid12345文件。
	这个文件是一个UNIX域套接字文件，由Attach成功以后的目标JVM进程生成。如果这个文件存在，说明正在Attach中，可以用这个socket进行下一步的通信。如果这个文件不存在则创建一个.attach_pid12345文件，这部分的伪代码如下所示。

```java
String tmpdir = "/tmp";
File socketFile = new File(tmpdir,  ".java_pid" + pid);
if (socketFile.exists()) {
    File attachFile = new File(tmpdir, ".attach_pid" + pid);
    createAttachFile(attachFile.getPath());
}
```

​	2）Attach端检查发现如果没有.java_pid12345文件，就会创建.attach_pid12345文件，然后发送SIGQUIT信号给目标JVM。接下来每隔200ms检查一次socket文件是否已经生成，如果生成则进行socket通信，5s以后还没有生成则退出。
​	3）对于目标JVM进程而言，它的Signal Dispatcher线程收到SIGQUIT信号以后，会检查.attach_pid12345文件是否存在。

* 目标JVM如果发现.attach_pid12345不存在，则认为这不是一个attach操作，执行默认行为输出当前所有线程的堆栈。
* 目标JVM如果发现.attach_pid12345存在，则认为这是一个attach操作，会启动Attach Listener线程，负责处理Attach请求，同时创建名为.java_pid12345的socket文件。



​	源码中/hotspot/src/share/vm/runtime/os.cpp这一部分处理的逻辑如下面代码清单7-13所示。

```java
#define SIGBREAK SIGQUIT

static void signal_thread_entry(JavaThread* thread, TRAPS) {
  while (true) {
    int sig;
    {
    switch (sig) {
      case SIGBREAK: { 
        // Check if the signal is a trigger to start the Attach Listener - in that
        // case don't print stack traces.
        if (!DisableAttachMechanism && AttachListener::is_init_trigger()) {
          continue;
        }
        ...
        // Print stack traces
    }
}
```

​	AttachListener的is_init_trigger方法在.attach_pid12345文件存在的情况下会新建.java_pid12345套接字文件，准备接收Attach端发送数据。

​	那Attach端和目标进程用socket传递了什么数据呢？可以通过strace命令看到Attach端究竟向socket中写了什么数据。执行命令，输出结果如下所示。

```java
sudo strace -f java -cp /usr/local/jdk/lib/tools.jar:. MyAttachMain 12345  2> strace.out

...
5841 [pid  3869] socket(AF_LOCAL, SOCK_STREAM, 0) = 5
5842 [pid  3869] connect(5, {sa_family=AF_LOCAL, sun_path="/tmp/.java_pid12345"}, 110)      = 0
5843 [pid  3869] write(5, "1", 1)            = 1
5844 [pid  3869] write(5, "\0", 1)           = 1
5845 [pid  3869] write(5, "load", 4)         = 4
5846 [pid  3869] write(5, "\0", 1)           = 1
5847 [pid  3869] write(5, "instrument", 10)  = 10
5848 [pid  3869] write(5, "\0", 1)           = 1
5849 [pid  3869] write(5, "false", 5)        = 5
5850 [pid  3869] write(5, "\0", 1)           = 1
5855 [pid  3869] write(5, "/home/ya/agent.jar"..., 18 <unfinished ...>
```

​	可以看到向socket写入的内容如下所示。

```java
1
\0
load
\0
instrument
\0
false
\0
/home/ya/agent.jar
\0
```

​	数据之间用\0字符分隔，第一行的1表示协议版本，接下来是发送指令“load instrument false/home/ya/agent.jar”给目标JVM，目标JVM收到这些数据以后就可以加载相应的agent jar包进行字节码的改写。
​	如果从socket的角度来看，VirtualMachine.attach方法相当于三次握手建立连接，Virtual-Machine.loadAgent则是握手成功之后发送数据，VirtualMachine.detach相当于四次挥手断开连接。

![](https://pic.imgdb.cn/item/61c07da92ab3f51d91227ff8.jpg)

# 8. JSR 269插件化注解处理原理

## 8.1 JSR 269简介

​	注解（Annotation）第一次是在JDK1.5中被引入进来的，当时开发者只能在运行期处理注解。JDK1.6引入了JSR 269规范，允许开发者在编译期间对注解进行处理，可以读取、修改、添加抽象语法树中的内容。只要有足够的想象力，利用JSR269可以完成很多Java语言不支持的特性，甚至创造新的语法糖。

![](https://pic.imgdb.cn/item/61c937d22ab3f51d91155a95.jpg)

​	现注解处理器的第一步是继承AbstractProcessor类，实现它的process方法，如下面的代码清单8-1所示。

```java
@SupportedAnnotationTypes("me.ya.anno.Data")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class DataAnnoProcessor extends AbstractProcessor {

    private JavacTrees javacTrees;
    private TreeMaker treeMaker;
    private Names names;

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        Context context = ((JavacProcessingEnvironment) processingEnv).getContext();
        javacTrees = JavacTrees.instance(processingEnv);
        treeMaker = TreeMaker.instance(context);
        names = Names.instance(context);
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
    }
}
```

​	`@SupportedSourceVersion（value=SourceVersion.RELEASE_8）`注解表示最高支持JDK8编译出来的类文件，``@SupportedAnnotationTypes（{"me.ya.annotation.MyBuilder"}）``注解表示只处理类全限定名为``me.ya.annotation.MyBuilder`的注解。

​	AbstractProcessor类有两个核心方法init和process。init方法用来完成一些初始化的操作，比如初始化核心的JavacTrees、TreeMaker、Names等，这三个类在后面的代码中会频繁用到。process方法用来做语法树的修改。

​	编译阶段的注解处理过程实际上就是操作抽象语法树的过程，接下来我们来操作抽象语法树有关的类。

## 8.2 抽象语法树操作API

​	语法树核心操作的核心类是Names、JCTree、TreeMaker。

* Names类提供了访问标识符的方法
* JCTree类是语法树元素的基类
* TreeMaker类封装了创建语法树节点的方法

### 8.2.1 Names介绍

​	Names类提供了访问标识符Name的方法，它最常用的方法是fromString，用来从一个字符串获取Name对象，它的方法定义如下所示。

```java
public Name fromString(String s) {
    return table.fromString(s);
}
```

​	比如，获取this名字标识符可以使用如下的代码。

```java
names.fromString("this")
```

### 8.2.2 JCTree介绍

​	JCTree是语法树元素的基类，实现了Tree接口，它有两个核心的字段pos和type。其中，pos表示当前节点在语法树中的位置，type表示节点的类型。JCTree的子类众多，常见的有JCStatement、JCExpression、JCMethodDecl和JCModifiers，接下来详细介绍这几个类及其常用的子类。

**1.JCStatement**

​	JCStatement类用来声明语句，常见的子类有JCReturn、JCBlock、JCClassDecl、JCVariable-Decl、JCTry、JCIf等。
​	JCReturn类用来表示return语句，它的部分源码如下所示。

```java
public static class JCReturn extends JCTree.JCStatement implements ReturnTree {
    public JCTree.JCExpression expr;
}
```

​	JCReturn的expr字段是一个JCExpression类型的变量，表示return语句表达式内容。
​	JCBlock类表示一个代码块，它的部分源代码如下所示。

```java
public static class JCBlock extends JCTree.JCStatement implements BlockTree {
    public long flags;
    public List<JCTree.JCStatement> stats;
}
```

​	其中，flags字段表示代码块的访问标记，stats字段是一个JCStatement类型的列表，表示代码块内的所有语句。

​	JCClassDecl类表示类定义语法树节点，它的部分源代码如下所示。

```java
public static class JCClassDecl extends JCTree.JCStatement implements ClassTree {
    public JCTree.JCModifiers mods;
    public Name name;
    public List<JCTree.JCTypeParameter> typarams;
    public JCTree.JCExpression extending;
    public List<JCTree.JCExpression> implementing;
    public List<JCTree> defs;
    public ClassSymbol sym;
}
```

​	它的字段说明如下。

* mods表示方法的访问修饰符，比如public、static等。
* name表示类名。
* typarams表示泛型参数列表。
* restype表示返回类型。
* extending表示继承的父类信息。
* implementing表示实现的接口列表。
* defs表示所有的变量和方法列表。
* sym表示包名和类名。

JCVariableDecl类用来表示变量语法树节点，它的定义如下所示。

```java
public static class JCVariableDecl extends JCStatement implements VariableTree {
    public JCModifiers mods; 
    public Name name;
    public JCExpression vartype;
    public JCExpression init;
    public VarSymbol sym;
    protected JCVariableDecl(JCModifiers mods,
                     Name name,
                     JCExpression vartype,
                     JCExpression init,
                     VarSymbol sym) {
        this.mods = mods;
        this.name = name;
        this.vartype = vartype;
        this.init = init;
        this.sym = sym;
    }
}
```

它的核心字段说明如下：

* mods：表示变量的访问修饰符，比如public、final、static等。
* name：表示变量名。
* vartype：表示变量类型。
* init是JCExpression类型的变量，表示变量的初始化语句，有可能是一个固定值，也可能是一个表达式。



​	JCTry类表示try-catch-finally语句，JCTry的源代码如下所示。

```java
public static class JCTry extends JCStatement implements TryTree {
    public JCBlock body;
    public List<JCCatch> catchers;
    public JCBlock finalizer;
}
```

​	其中，body表示try语句块；catchers是JCCatch对象列表，表示多个catch语句块；finalizer表示finally语句块。
​	JCIf类表示if-else代码块，它的类定义如下所示。

```java
public static class JCIf extends JCStatement implements IfTree {
    public JCExpression cond;
    public JCStatement thenpart;
    public JCStatement elsepart;
}
```

​	其中，cond表示条件语句，thenpart和elsepart分别表示if和else部分。
​	JCForLoop类表示一个for循环语句，它的源代码如下所示。

```java
public static class JCForLoop extends JCStatement implements ForLoopTree {
    public List<JCStatement> init;
    public JCExpression cond;
    public List<JCExpressionStatement> step;
    public JCStatement body;
}
```

​	一个典型的for循环语句格式如下所示。

```java
for (init ; cond ; step) {
    body
}
```

​	其中，init表示循环的初始化，cond表示循环的条件判断，step是每次循环后的操作表达式，body是for循环的循环体。

**2.JCExpression**

​	JCExpression类用来表示表达式语法树节点，常见的子类如下所示。

* JCAssign：赋值语句表达式。
* JCIdent：标识符表达式。
* JCBinary：二元运算符。
* JCLiteral：字面量运算符表达式。

​	JCAssign用来表示赋值语句表达式，比如语句“x=2”就是一个赋值语句。JCAssign类的部分源代码如下所示。

```java
public static class JCAssign extends JCTree.JCExpression implements AssignmentTree {
    public JCTree.JCExpression lhs;
    public JCTree.JCExpression rhs;
    // ...
}
```

​	lhs表示赋值语句的左边表达式，rhs表示赋值语句的右边表达式。
​	JCIdent用来表示标识符语法树节点，可以表示类、变量和方法，它的部分源码如下所示。

```java
public static class JCIdent extends JCTree.JCExpression implements IdentifierTree {
    public Name name;
    public Symbol sym;
}
```

​	其中name表示标识符的名字，sym表示标识符的其他标记，比如表示类时，sym表示类的包名和类名。
​	JCBinary用来表示二元操作符，加、减、乘、除，以及与或运算都属于二元运算符，比如语句“1+2”就是一个二元操作符。JCBinary简化过的部分源码如下所示。

```java
public class JCBinary extends JCTree.JCExpression implements BinaryTree {
    private JCTree.Tag opcode;
    public JCTree.JCExpression lhs;
    public JCTree.JCExpression rhs;
    // ...
}
```

​	其中，opcode表示二元操作符的运算符，lhs表示二元操作符的左半部分，rhs表示二元操作符的右半部分。opcode是一个JCTree.Tag类型的变量，JCTree.Tag是一个枚举类，常见的枚举类有下面这些。

```java
public static enum Tag {
    // ...
    PLUS,  // +
    MINUS, // -
    MUL,   // *
    DIV,   // /
    MOD,   // %
    // ...
}
```

​	JCLiteral类用来表示字面量表达式，它的部分源码如下所示。

```java
public static class JCLiteral extends JCTree.JCExpression implements LiteralTree {
    public TypeTag typetag;
    public Object value;
}
```

​	其中typetag字段表示常量的类型，value字段表示常量的值。typetag是一个TypeTag类型的变量，TypeTag是一个枚举类，常见的枚举如下所示。

```java
public enum TypeTag {
    BYTE(1, 125, true),
    CHAR(2, 122, true),
    SHORT(4, 124, true),
    LONG(16, 112, true),
    FLOAT(32, 96, true),
    INT(8, 120, true),
    DOUBLE(64, 64, true),
    BOOLEAN(0, 0, true),
    VOID,
    CLASS,
    // ...
}
```

**3.JCMethodDecl**

```java
public static class JCMethodDecl extends JCTree implements MethodTree {
    public JCModifiers mods;
    public Name name;
    public JCExpression restype;
    public List<JCTypeParameter> typarams;
    public List<JCVariableDecl> params;
    public List<JCExpression> thrown;
    public JCBlock body;
    public JCExpression defaultValue; // for annotation types
    public MethodSymbol sym;
    ...
}
```

常用的字段解释如下。

* mods：表示方法的访问修饰符，比如public、static、synchronized等。
* name：表示方法名。
* restype：表示返回类型。
* typarams：表示方法泛型参数列表。
* params：表示方法参数列表。
* thrown：方法异常抛出列表。
* body：表示方法体。

**4.JCModifiers**

```java
public static class JCModifiers {
    public long flags;
    public List<JCTree.JCAnnotation> annotations;
    // ...
}
```

​	flags字段表示访问标记，可以由com.sun.tools.javac.code.Flags定义的常量来表示，多个flag可以组合使用，以下面的代码为例。

```java
public static final int x = 1;
```

​	变量x的访问标记flags可以用Flags.PUBLIC+Flags.STATIC+Flags.FINAL值表示。

### 8.2.3 TreeMaker介绍

​	TreeMaker类封装了创建语法树节点的方法，是注解处理中最核心的类。前面8.2.2节介绍过，JCTree包含一个pos字段表示当前语法树节点在抽象语法树中的位置，所以我们不能用new关键字来创建JCTree，只能使用包含了语法树上下文的TreeMaker对象来构造JCTree。它常用的方法有下面这些。

* TreeMaker.Modifiers方法用于生成一个访问标记JCModifiers。
* TreeMaker.Binary方法用于生成二元操作符JCBinary。
* TreeMaker.Ident方法用于创建标识符语法树节点JCIdent。
* TreeMaker.Select方法用于创建一个字段或方法访问。
* TreeMaker.Return方法用于创建return语句语法树节点JCReturn。
* TreeMaker.Assign方法用于生成赋值语句的语法树节点。
* TreeMaker.Block方法用于生成语句块语法树节点JCBlock。
* TreeMaker.Exec创建执行这个语句的语法树节点返回一个JCExpressionStatement对象。
* TreeMaker.VarDef方法用于生成变量语法树节点JCVariableDecl。
* TreeMaker.MethodDef方法用于生成方法语法树节点JCMethodDecl。

接下来一一进行介绍。

**1.TreeMaker.Modifiers**
	TreeMaker.Modifiers方法用来生成一个访问标记JCModifiers，它的方法定义如下所示。

```java
public JCModifiers Modifiers(long flags) {
}
```

​	其中，flags是一个long型值，表示访问标记的组合，以下面的变量x为例。

```java
public static final int x = 1;
```

**2.TreeMaker.Binary**

​	TreeMaker.Binary方法用来生成二元操作符JCBinary，它的方法定义如下。

```java
public JCBinary Binary(
    int opcode,        // 二元操作符
    JCExpression lhs,  // 操作符左边表达式
    JCExpression rhs   // 操作符右边表达式
    ) {
}
```

​	以“1+2”为例，用TreeMaker来创建的代码如下所示。

```java
JCTree.JCBinary addJCBinary = treeMaker.Binary(
    JCTree.Tag.PLUS,       // +
    treeMaker.Literal(1),  // 1
    treeMaker.Literal(2)   // 2
);
```

​	其中JCTree.Tag.PLUS表示“+”操作符，treeMaker.Literal（1）生成了一个整型常量1作为二元操作的左半部分，treeMaker.Literal（2）生成了一个整型常量2作为二元操作的右半部分。

**3.TreeMaker.Ident**

​	TreeMaker.Ident方法用来创建类、变量、方法的标识符语法树节点JCIdent，它的方法定义如下所示。

```java
public JCIdent Ident(Name name) {
}
```

​	以获取变量x的标识符为例，用TreeMaker来创建的代码如下所示。

```java
Names names = Names.instance(context);
treeMaker.Ident(names.fromString("x"))
```

**4.TreeMaker.Select**

​	TreeMaker.Select方法用于创建一个字段或方法访问JCFieldAccess，它的方法定义如下所示。

```java
public JCFieldAccess Select(
    JCExpression selected,  // . 号左边的表达式
    Name selector           // . 号右边的表达式
) 
```

​	以语句“this.id”为例，用TreeMaker来创建的代码如下所示。

**5.TreeMaker.Return**

```java
public JCReturn Return(JCExpression expr) {
}
```

​	其中expr参数是返回内容的表达式语句，以下面的代码为例。

```java
return this.id;
```

​	用TreeMaker创建的代码如下所示。

```java
Names names = Names.instance(context);
JCTree.JCReturn returnStatement = treeMaker.Return(
        treeMaker.Select(
treeMaker.Ident(names.fromString("this")),
                names.fromString("id")
        )
);
```

**6.TreeMaker.Assign**

​	TreeMaker.Assign用来生成赋值语句的语法树节点JCAssign，它的方法定义如下所示。

```java
public JCAssign Assign(
    JCExpression lhs, // 等号左边表达式
    JCExpression rhs  // 等号右边表达式
    ) {
}
```

​	以“x=0；”为例，对应的用TreeMaker创建的语句如下所示。

```java
treeMaker.Assign(
    treeMaker.Ident(names.fromString("x")), // 等号左边 x
    treeMaker.Literal(0)                    // 等号右边 0
)
```

**7.TreeMaker.Block**

​	TreeMaker.Block方法用于生成语句块语法树节点JCBlock，可以用于方法体，它的定义如下所示。

```java
public JCBlock Block(long flags, List<JCStatement> stats) {
}
```

​	它的第一个参数flags表示访问标记，第二个参数stats是一个JCStatement列表，表示多条语句，常见的用法如下所示。

```java
JCStatement statement = // ...
ListBuffer<JCTree.JCStatement> statements = new ListBuffer<JCTree.JCStatement>().append(statement);
JCTree.JCBlock body = treeMaker.Block(0, statements.toList());
```

**8.TreeMaker.Exec**

​	前面介绍的TreeMaker.Assign返回了一个JCAssign对象，一般使用时会对它包装一层TreeMaker.Exec方法调用，使其返回一个JCExpressionStatement类型的对象。TreeMaker.Exec的方法定义如下所示。

```java
public JCExpressionStatement Exec(JCExpression expr) {
}
```

​	还是以“x=0；”语句为例，对应的用TreeMaker创建的代码如下所示。

```java
JCTree.JCExpressionStatement statement =
    treeMaker.Exec(
        treeMaker.Assign(
            treeMaker.Ident(names.fromString("x")), // 等号左边表达式
            treeMaker.Literal(0)                    // 等号右边表达式
        )
);
```

**9.TreeMaker.VarDef**

​	TreeMaker.VarDef方法用来生成变量语法树节点JCVariableDecl，它的方法定义如下所示。

```java
public JCVariableDecl VarDef(
    JCModifiers mods,     // 访问标记
    Name name,            // 变量名
    JCExpression vartype, // 变量类型
    JCExpression init     // 变量初始化表达式
)
```

​	以下面的Java语句为例。

```java
private int x = 1;
```

​	可以用如下TreeMaker语句来创建。

```java
JCTree.JCVariableDecl var = treeMaker.VarDef(
        treeMaker.Modifiers(Flags.PRIVATE), // JCModifiers
        names.fromString("x"),              // Name
        treeMaker.TypeIdent(TypeTag.INT),   // JCPrimitiveTypeTree
        treeMaker.Literal(1)                // JCLiteral
);
```

​	变量初始值除了可以是字面量、常量以外，还可以是一个初始化表达式，以下面代码为例。

```java
final int x = 1 + 2;
final int y = flag ? -1 : 1;
```

​	变量x的初始化init为一个JCBinary（1+2），表示这是一个二元操作符表达式。变量y的初始化init为一个JCConditional（flag？-1：1），表示这是一个条件语句。

**10.TreeMaker.MethodDef**
	TreeMaker.MethodDef用来创建一个方法语法树节点JCMethodDecl，它的方法定义如下所示。

```java
public JCMethodDecl MethodDef(
       JCModifiers mods,              // 方法访问级别修饰符
       Name name,                     // 方法名
       JCExpression restype,          // 返回值类型
       List<JCTypeParameter> typarams,// 泛型参数列表
       List<JCVariableDecl> params,   // 参数值列表
       List<JCExpression> thrown,     // 异常抛出列表
       JCBlock body,                  // 方法体
       JCExpression defaultValue)     // 默认值
```

​	下面8.2.4节的案例中会有TreeMaker.MethodDef定义方法的使用介绍，这里先不展开。
​	到这里基础的API介绍就告一段落，接下来我们来看自定义注解处理实战。

### 8.2.4 自定义注解处理实战

​	本节会演示一个实际的例子，使用JSR 269 API为类中的字段自动生成get、set方法。首先定义一个自定义注解类Data，如下所示。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Data {
}
```

​	接下来新建一个AbstractProcessor的子类DataAnnotationProcessor，实现init和process方法，如下面的代码清单8-2所示。

```java
@SupportedAnnotationTypes("me.ya.annotation.Data")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class DataAnnotationProcessor extends AbstractProcessor {

    private JavacTrees javacTrees;
    private TreeMaker treeMaker;
    private Names names;
 @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        Context context = ((JavacProcessingEnvironment) processingEnv).getContext();
        javacTrees = JavacTrees.instance(processingEnv);
        treeMaker = TreeMaker.instance(context);
        names = Names.instance(context);
    }
    
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        Set<? extends Element> set = roundEnv.getElementsAnnotatedWith(Data.class);
        for (Element element : set) {
            JCTree tree = javacTrees.getTree(element);
            tree.accept(new TreeTranslator() {
                @Override
                public void visitClassDef(JCTree.JCClassDecl jcClassDecl) {
                    jcClassDecl.defs.stream()
                            .filter(it -> it.getKind().equals(Tree.Kind.VARIABLE)) // 只处理变量类型
                            .map(it -> (JCTree.JCVariableDecl) it)                 // 强制转换为 JCVariableDecl 类型
                            .forEach(it -> {
                                jcClassDecl.defs = jcClassDecl.defs.prepend(genGetterMethod(it));
                                jcClassDecl.defs = jcClassDecl.defs.prepend(genSetterMethod(it));
                            });

                    super.visitClassDef(jcClassDecl);
                }
            });
        }
        return true;
    }
}
```

init方法比较简单，主要作用是从Context中初始化JavacTrees、TreeMaker、Names等关键类。process方法分为下面这几个步骤。

首先通过RoundEnvironment.getElementsAnnotatedWith方法获取被Data注解的类的集合，接下来开始逐个处理这些类。

* 通过JavacTrees.getTree获取当前处理类的抽象语法树。
* 调用JCTree.accept方法，传入一个TreeTranslator实例，覆写其中的visitClassDef方法，在遍历抽象语法树的过程中遇到相应的事件就会调用这个方法。
* 当visitClassDef方法被调用时，可以获取到JCClassDecl实例对象，可以遍历过滤出这个类的所有字段。
* 接下来遍历所有的字段列表，根据字段名和字段类型使用TreeMaker生成get和set方法。

对于字段id，对应的get方法代码如下所示。

```java
public int getId() {
    return this.id;
}
```

通过TreeMaker.MethodDef可以生成方法语法树节点，完整的代码如下面的代码清单8-3所示。
代码清单8-3　get方法生成

```java
private JCTree.JCMethodDecl genGetterMethod(JCTree.JCVariableDecl jcVariableDecl) {
    // 生成语句 return this.xxx;
    JCTree.JCReturn returnStatement = treeMaker.Return(
            treeMaker.Select(
            treeMaker.Ident(names.fromString("this")),
                    jcVariableDecl.getName())
    );

    ListBuffer<JCTree.JCStatement> statements = new ListBuffer<JCTree.JCStatement>().append(returnStatement);
    // public 方法访问级别修饰符
    JCTree.JCModifiers modifiers = treeMaker.Modifiers(Flags.PUBLIC);
    // 方法名(getXxx), 根据字段名生成首字母大写的 get 方法
    Name getMethodName = getMethodName(jcVariableDecl.getName());
    // 返回值类型，get 方法的返回值类型与字段类型一样
    JCTree.JCExpression returnMethodType = jcVariableDecl.vartype;
    // 生成方法体
    JCTree.JCBlock body = treeMaker.Block(0, statements.toList());
    // 泛型参数列表
    List<JCTree.JCTypeParameter> methodGenericParamList = List.nil();
    // 参数值列表
    List<JCTree.JCVariableDecl> parameterList = List.nil();
    // 异常抛出列表
    List<JCTree.JCExpression> thrownCauseList = List.nil();

    // 生成方法定义语法树节点
    return treeMaker.MethodDef(
            modifiers,              // 方法访问级别修饰符
            getMethodName,          // get 方法名
            returnMethodType,       // 返回值类型
            methodGenericParamList, // 泛型参数列表
            parameterList,          // 参数值列表
            thrownCauseList,        // 异常抛出列表
            body,                   // 方法体
            null                    // 默认值
    );
}
```

set方法生成比get方法稍微复杂一点点，完整的代码如下面的代码清单8-4所示。
代码清单8-4　set方法生成

```java
private JCTree.JCMethodDecl genSetterMethod(JCTree.JCVariableDecl jcVariableDecl) {

    // this.xxx = xxx;
    JCTree.JCExpressionStatement statement =
            treeMaker.Exec(
                    treeMaker.Assign(
                            treeMaker.Select(
                                    treeMaker.Ident(names.fromString("this")),
                                    jcVariableDecl.getName()
                            ),                                        // lhs
                            treeMaker.Ident(jcVariableDecl.getName()) // rhs
                    )
            );
    ListBuffer<JCTree.JCStatement> statements = new ListBuffer<JCTree.JCStatement>().append(statement);

    // set 方法参数
    JCTree.JCVariableDecl param = treeMaker.VarDef(
            treeMaker.Modifiers(Flags.PARAMETER, List.nil()), // 访问修饰符
            jcVariableDecl.name,                              // 变量名
            jcVariableDecl.vartype,                           // 变量类型
            null                                              // 变量初始值
    );

    // 方法访问修饰符 public
    JCTree.JCModifiers modifiers = treeMaker.Modifiers(Flags.PUBLIC);
    // 方法名(setXxx), 根据字段名生成首字母大写的 set 方法
    Name setMethodName = setMethodName(jcVariableDecl.getName());
    // 返回值类型，void
    JCTree.JCExpression returnMethodType = treeMaker.Type(new Type.JCVoidType());
    // 生成方法体
    JCTree.JCBlock body = treeMaker.Block(0, statements.toList());
    // 泛型参数列表
    List<JCTree.JCTypeParameter> methodGenericParamList = List.nil();
    // 参数值列表
    List<JCTree.JCVariableDecl> parameterList = List.of(param);
    // 异常抛出列表
    List<JCTree.JCExpression> thrownCauseList = List.nil();

    // 生成方法定义语法树节点
    return treeMaker.MethodDef(
            modifiers,              // 方法访问级别修饰符
            setMethodName,          // set 方法名
            returnMethodType,       // 返回值类型
            methodGenericParamList, // 泛型参数列表
            parameterList,          // 参数值列表
            thrownCauseList,        // 异常抛出列表
            body,                   // 方法体
            null                    // 默认值
    );
}
```

​	这个时候还没有进行注解处理生成get、set方法，如果进行编译一定会出错，先使用javac编译注解类，如下所示。

```java
javac -cp /your_jdk_path/lib/tools.jar src/main/java/me/ya/annotation/* -d ./out
```

​	接下来使用-processor选项编译User.java，如下所示。

```java
javac -cp ./out -d out -processor me.ya.annotation.DataAnnotationProcessor src/main/java/User.java
```

​	这时会在out目录生成User.class文件，运行java可以得到预期的输出。

```java
java User 
id: 1118        name: ya
```

