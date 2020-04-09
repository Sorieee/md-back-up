---
title: HashMap源码分析
date: 2020-04-06 22:16:37
tags: [java,java容器,Map,HashMap,java源代码]
---

学习视频：https://www.bilibili.com/video/BV1FE411t7M7

# 1.HashMap介绍

​	基于哈希表的实现的`Map`接口，是以key-value存储形式存在，即主要用来存放键值对。HashMap的实现不是同步的，这意味着它不是线程安全的。它的key和value都可以为null。此外，HashMap中的映射不是有序的。

​	JDK1.8之前HasmMap是由数字+链表组成的。数组是HashMap的主体，链表则是主要为了解决哈希冲突（**两个对象调用的hashCode方法计算的Hash码值一致导致计算的数组的索引值相同**)而存在("拉链法"解决冲突)。JDK1.8以后在解决哈希冲突时有了比较大的变化。**当链表长度大于阈值(或者红黑树的边界值，默认为8)并且当前数组长度大于64时，此时此索引位置上的所有数据改为使用红黑树存储**。

​	补充：将链表转换成红黑树前会判断，即使阈值大于8，但是当前数组长度小于64，此时并不会将链表变为红黑树，而是选择进行数组扩容。

​	这样做的目的是因为数组比较小，尽量避开红黑树结构，这种情况下变为红黑树结构，反而会降低效率，因为红黑树需要进行左旋，右旋，变色这些操作来保持平衡。数组长度小于64时，搜索时间相对要快些，所以综上所述，为了提高性能和减少搜索时间，底层在阈值大于8并且数组长度大于64时，链表才转换为红黑树，具体可以参考 treeifyBin方法。

![](https://raw.githubusercontent.com/Ewasong/picBed/master/20200407231049.png)

特点：

1. 存取无序的
2. 键和值位置都可以是null，但是键位置只能是一个null
3. 键位置是唯一的，底层的数据结构控制键的
4. jdk1.8前数据结构是：链表+数组，jdk1.8之后是：链表+数组+红黑树
5. 阈值（边界值）> 8并且数组长度大于64，才将链表转换为红黑树，变为红黑树的目的是为了高效查询。

# 2.HashMap集合底层的数据结构

## 2.1数据结构概念

​	数据结构是计算机存储，组织数据的方式，数据结构是指相互之间存在一种或者多种特定关系的数据元素的集合。通常情况下，精心选择的数据结构可以带来更高的运行或者存储效率。数据结构往往同高效的检测算法和索引技术有关。

在JDK1.8之前HashMap由数组+链表 数据结构组成

在JDK1.8以及之后HashMap由数组+链表+红黑树数据结构组成

## 2.2 HashMap底层的数据结构存储过程

1. HashMap<String, Integer> hm = new HashMap();

   当创建HashMap集合数据的时候，在jdk8以前，构造方法创造一个一个长度是16的Entry[] table 用来存储键值对数据的。 在jdk8后不是在HashMap的构造方法底层创建数组了，而是第一次调用put方法时创建数组 Node[] table

2. 假设向哈希表存储<柳岩,18>数据，根据柳岩调用String类中重写之后的HashCode()方法计算出值，然后结合数组长度采用某种算法计算出Node数组中存储数据的空间的索引值。如果计算出的索引空间没有值，则直接将数据存储到数组中。

   * 面试题：哈希表底层采用何种算法计算hash值？还有哪些算法可以计算出hash值？

     底层采用key的hashCode方法的值结合数组长度进行无符号右移(>>>)，按位抑或(^),按位与(&)计算出索引。

     还可以采用：平方取中法，取余数，伪随机数。

3. 假设刘德华的key也是3，假设刘德华计算出的hashCode结合数组长度计算出的索引和柳岩一样，那么此时数组空间不是null，此时底层会比较柳岩和刘德华的hash值是否一致，如果不一致，则在此空间上划出一个节点来存储键值对数据<刘德华,40>

   这种方式称为拉链法

4. 假设向哈希表存储数据<柳岩,20>，首先计算出的索引值肯定一样，如果hash值相等，此时发生hash碰撞，那么底层会先判断hashCode一致，一致就发生哈希碰撞，如果相等，就覆盖；如果都不相等，此空间上划出一个节点来存储键值对数据

![image-20200407235604451](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200407235604451.png)

	5. 在不断添加数据的过程中，会涉及到扩容问题，当超出临界值时(且要存放的位置非空)，扩容。默认扩容方式：扩容为原来的两倍，并将原有数据复制过来。
 	6. 通过上述描述，当一个链表中元素较多，即hash值相等但是内容不相等时，通过key值一次查询的效率较低。而JDK1.8中，哈希表存储采用是数组+链表+红黑树实现，当链表长度超过8且当前数组长度>64时，将链表转换为空黑叔，这样大大减少了查找时间。jdk8在哈希表中引入红黑树的原因只是为了查找效率更高。

**问题：传统hashMap的缺点,1.8为什么引入红黑树？这样的结构的话不是更麻烦了吗，为何发阈值大于8换成红黑树？**

​	JDK1.8以前HashMap的实现是数组+链表，即使哈希函数取得再好，也很难达到元素百分百均匀分布。当HashMap中有大量的元素都存放到同一个桶中时，这个桶下有一条常常的链表，这个时候HashMap就相当于一个单链表，假如单链表有n个元素，遍历的时间复杂度就是O(n)，完全失去了它的优势。针对这种情况，JDK1.8才引入了红黑树（查找时间复杂度为O(logn))来优化这个问题。当链表长度很小时，即使遍历，速度也非常快，但是当链表不断变长，肯定会对查询性能有一定影响，所以才需要转成数。

​	至于为什么阈值是9，我们看了源代码再来回答。

1. size代表HashMap中KV的实时数量，不等于数组长度
2. threshold(临界值) = capacity*loadFactor（加载因子）。size超过这个临界值就重新扩容，扩容后容量是之前两倍。

# 3.继承关系

![](https://raw.githubusercontent.com/Ewasong/picBed/master/20200408214748.png)

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

说明：

* Cloneable表示可以克隆
* Serializable表示可以序列化
* AbstractMap父类提供了Map实现接口，以最大限度减少实现此接口所需要的工作。

补充：通过上述继承关系可以发现一个很奇怪的现象，就是HashMao已经继承了AbstractMap而AbstractMap已经实现了Map接口，那为什么HashMap还要再实现Mao接口呢？同样在ArrayList和LinkedList都是这种结构。

>据Java集合框架创世人Josh Blosh描述，这样的写法是一个失误。在Java框架中，类似的写法很多，最开始写Java集合的时候，他认为这样写，在某些地方可能是有价值的，直到他意识到错了。显然的，JDK的维护者后台不认为这个小小的失误只能去修改，所以就这样存在下来了。

# 4.HashMap集合类的成员变量

1.序列化版本号

```java
private static final long serialVersionUID = 362498820763181265L;
```

2.默认容量(必须是二的n次幂)

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

问：为什么必须是2的n次幂？如果输入值不是2的幂比如10会怎么样？

```java
public HashMap(int initialCapacity) 
```

根据上述讲解我们已经知道，当向HashMap添加一个元素时，需要根据key的hash值，去确认其在数组中的具体位置，HashMap为了存储高效，要尽量减少碰撞，就是要尽量把数据分配均匀，每个链表长度大致相同，这个实现就是吧数据存储到哪个链表中的算法。

这个算法实际就是取模，hash%length，计算机直接求余的效率不如位运算效率高，所以源码中做了优化，使用hash&(length-1),实际上hash%length等于hash&(length-1)的前提是length是2的n次幂。

​	为什么这样能均匀缝补减少碰撞呢？2的n次方就是1后面n个0,，2的n次方-1吗，实际上就是n个1。

3.最大容量

```java
static final int MAXIMUM_CAPACITY = 1 << 30;
```

4.默认加载因子

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

5.变树链表长度阈值

```java
static final int TREEIFY_THRESHOLD = 8;
```

6.树退化为链表的阈值

```java
static final int UNTREEIFY_THRESHOLD = 6;
```

7.最小树化的数组容量

```java
static final int MIN_TREEIFY_CAPACITY = 64;
```

8.数据存放的table

```java
transient Node<K,V>[] table;
```

9.缓存的EntrySet

```java
transient Set<Map.Entry<K,V>> entrySet;
```

10.数据大小

```java
transient int size;
```

11.结构化修改次数

```java
transient int modCount;
```

12.扩容的阈值(capacity * load factor)

```java
int threshold;
```

13.加载因子

```java
final float loadFactor;
```

# 5.构造器

| 构造方法                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **[HashMap](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/HashMap.html#HashMap--)**() | 构造一个空的 `HashMap` ，默认初始容量（16）和默认负载因子（0.75） |
| **[HashMap](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/HashMap.html#HashMap-int-)**(int initialCapacity) | 构造一个空的 `HashMap`具有指定的初始容量和默认负载因子（0.75） |
| **[HashMap](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/HashMap.html#HashMap-int-float-)**(int initialCapacity, float loadFactor) | 构造一个空的 `HashMap`具有指定的初始容量和负载因子           |
| **[HashMap](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/HashMap.html#HashMap-java.util.Map-)**([Map](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/Map.html)<? extends [K](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/HashMap.html),? extends [V](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/HashMap.html)> m) | 构造一个新的 `HashMap`与指定的相同的映射 Map                 |

## HashMap()

容量在第一次put时初始化。

```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

## HashMap(int initialCapacity)

```java
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

## HashMap(int initialCapacity, float loadFactor)

这里主要分析tableSizeFor方法

这和方法主要给刚好大于等于cap的最小2的n次幂

1. 首先对cap执行-1操作

   为了防止cap已经是2的幂，如果cap已经是2的幂，又没有执行这个-1操作，则执行完几条无符号右移之后，返回的capacity将是这个cap的2倍。

2. 如果n为0，经过几次无符号右移依然是0，最后返回的值是1，因为后面有一个n+1操作。

Tips1：

几次位运算可以将n的进制表示的最高位的1后面的值全部全部置为1,然后在+1就可以求出刚好比他大的2的n次幂

* 得到这个capacity被赋值给threshold

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

