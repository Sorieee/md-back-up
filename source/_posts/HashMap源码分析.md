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

![](https://pic.imgdb.cn/item/60a9f2e535c5199ba7baabe5.jpg)

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

**转换红黑树的边界值为什么是8？**

因为树节点内存大概是普通内存的2倍，所以只在链表上有足够节点（8）才转换为红黑树。当节点变得很小时(6)又转换成为链表。理想情况下，在随机哈希码下，链表中节点服从泊松分布。

选择8是因为泊松分布，

>```
>* Because TreeNodes are about twice the size of regular nodes, we
>* use them only when bins contain enough nodes to warrant use
>* (see TREEIFY_THRESHOLD). And when they become too small (due to
>* removal or resizing) they are converted back to plain bins.  In
>* usages with well-distributed user hashCodes, tree bins are
>* rarely used.  Ideally, under random hashCodes, the frequency of
>* nodes in bins follows a Poisson distribution
>* (http://en.wikipedia.org/wiki/Poisson_distribution) with a
>* parameter of about 0.5 on average for the default resizing
>* threshold of 0.75, although with a large variance because of
>* resizing granularity. Ignoring variance, the expected
>* occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
>* factorial(k)). The first values are:
>*
>* 0:    0.60653066
>* 1:    0.30326533
>* 2:    0.07581633
>* 3:    0.01263606
>* 4:    0.00157952
>* 5:    0.00015795
>* 6:    0.00001316
>* 7:    0.00000094
>* 8:    0.00000006
>* more: less than 1 in ten million
>```



6.树退化为链表的阈值

```java
static final int UNTREEIFY_THRESHOLD = 6;
```

7.最小树化的数组容量

当Map中的数组容量超过这个值时， 表中的桶才会进行树化，否则桶元素太多时会进行扩容，而不是树化。为了避免扩容、树化的选择冲突，这个值不能小于4 * TREEIFY_THRESHOLD(8)

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

说明：

1. 加载因子是用来衡量HashMap满的程度，表示HashMap的疏密程度，影响hash操作到同一个数组位置的概率。计算HashMap的实时加载因子的方法为：size/capacity,而不是占用桶的数量去除以capacity。capacity是桶的数量，也就是table的长度length
2. loadFactor太大导致查找元素效率低，太小导致数组的利用率低，存放的数据会很分散。loadFactor的默认值为0.75f是官方给的一个比较好的边界值。
3. 当HashMap里面容纳的元素已经达到75%数组长度时，表示HashMap太挤了，需要扩容，而扩容这个过程涉及到rehash，复制数据等操作，非常消耗性能。所以开发中尽量减少扩容的次数，可以通过创建HashMap集合对象时初始容量时指定初始容量来尽量避免。

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
 	//这里不等于capacity*loadFactor
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

## HashMap(Map<? extends K, ? extends V> m) 

```java
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        //如果table没有数据，那么是初始化
        if (table == null) { // pre-size
            //计算容量 +1是为了减少扩容
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            //不为0就计算边界值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        //如果有数据，并且大于边界值，就扩容
        else if (s > threshold)
            resize();
        //然后把数据放进去
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

# 6. 增加方法

putVal方法是比较复杂的

1. 先通过hash值计算放到哪个桶中

2. 如果桶上没有碰撞冲突，就直接插入

3. 如果碰撞冲突了，则需要冲突冲突

   a. 如果该桶使用红黑树处理冲突，则调用红黑树插入数据

   b. 否则采用传统的链式方法插入，如果链的长度达到边界值，则把链转变为红黑树

4. 如果同种存在重复键，则替换该键为新值value

5. 如果size大于阈值threshold，则进行扩容。

主要参数

* hashKey的hash值
* key 原始key
* value 要存放的值
* onlyIfAbsent 如果true代表不更改现有值
* evict 如果为false表示table为创建状态

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}


final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    HashMap.Node<K,V>[] tab; HashMap.Node<K,V> p; int n, i;
    //table没有数据，就扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //(n-1)&hash等于 n%hash
    if ((p = tab[i = (n - 1) & hash]) == null)
        //创建链表的第一个节点
        tab[i] = newNode(hash, key, value, null);
    else {
       
        HashMap.Node<K,V> e; K k;
        //如果hash并且第一个key相等，直接替换
        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //判断是否是树节点
        else if (p instanceof HashMap.TreeNode)
            e = ((HashMap.TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //是链表，就逐个比对
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    //如果找不到，就把数据添加到链表最后
                    p.next = newNode(hash, key, value, null);
                    //如果大于8就转换为树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            //存在就执行替换操作 
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```



## hash方法分析

* 搞16bit不变，低16bit和高16bit做了一个抑或

为什么要这么操作？

如果当n即数组长度很小，假设为16的话，那么n-1为--->1111,这样的值和hashCode()做按位与操作，实际上只使用了哈希值的后4位，如果当hash值的高位变化很大，低位变化很小，就容易造成hash冲突，所以这里把高低位都利用起来，从而解决这个问题。

```java
static final int hash(Object key) {
   int h;
   return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

## treeifyBin(Node<K,V>[] tab, int hash)

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    //当少于容量64，只是扩容
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            //遍历链表，然后转换为树节点
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        //构建红黑树
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

## 扩容方法 resize

想了解HashMap的扩容机制要有这两个问题

* 1.什么时候才需要扩容
* 2.HashMap的扩容是什么

1. 什么时候才需要扩容

当HashMap中的元素个数超过数组大小（数组长度）* loadFactor时，就会进行数组扩容，loadFactor的默认值是0.75，这是一个这种的取值。也就是说，默认情况下，数组大小为16，那么HashMap中的个数超过16 * 0.75 = 12，就把数组的大小扩展为2 * 16， 即扩大一倍。 然后重新计算每个元素在数组中的位置，而这是一个非常小号性能的操作，所以如果我们已经预知HashMap中的元素个数，那么与预知个数能够有效提高HashMap的性能。

补充：

当HashMap中的一个链表对象个数达到9个，次数数组长度没有达到64，那么HashMap先会扩容处理，如果已经达到了64，那么这个链表会变成红黑树，阶段类型由Node变成TreeNode类型。当然，如果映射关系被移除后，下次执行resize方法判断树的节点个数低于6，也会再把树转换为链表。

2. HashMap的扩容是什么？

进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有元素，是非常耗时的，在编写程序中，要尽量避免resize。

HashMap在进行扩容时，使用的rehash方式非常巧妙，因为每次扩容都是翻倍，与元来计算(n-1)&hash的结果项目，只是多了一个bit位，所以节点要么就在原来的位置，要么就被分配到**原位置+旧容量**这个位置

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```



