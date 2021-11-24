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

