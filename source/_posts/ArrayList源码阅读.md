---
title: ArrayList源码阅读
date: 2020-04-01 22:36:10
tags: [java,java容器,List,ArrayList,java源代码]
---

学习视频:https://www.bilibili.com/video/BV1gE411A7H8

[TOC]



# 1.ArrayList集合底层数据结构

## 1.1List接口的可调整大小的数据实现

​	数组：一旦初始化长度就不可以发生改变

## 1.2数组结构介绍

* 增删慢：每次删除元素，就需要更改数组长度，拷贝以及移动元素位置
* 查询快：由于数组在内存中是一块连续空间，因此可以根据地址+索引的方式快速获取对应位置上的元素

# 2.ArrayList继承关系

## 2.1 Serializable标记性接口

1. **介绍** 类的序列化由java.io.Serializable接口的类启用。不实现此接口的类将不会使用任何状态序列化和反序列化。可序列化类的所有子类都是可序列化的。序列化接口没有方法或者字段，仅用于标志可串行化的语义。

   序列化： 将对象的数据写入到文件

   反序列化：将文件的数据读取数据

2. **Serializable** 源码

```java
public interface Serializable {
}
```

## 2.2 Cloneable 标记性接口

1. **介绍** 一个类实现Cloneable接口来执行Object.clone()方法，该方法对于该类的实例进行字段的复制是合法的。在不识闲Cloneable接口的实例上调用对象的克隆方法会导致异常CloneNotSupportedException被抛出。简言之：克隆就是依据已经有的数据，创造出一份新的完全一样的数据拷贝。
2. 源代码

```java
public interface Cloneable {
}
```

3. 克隆的前提

   * 被克隆的对象必须实现Cloneable接口
   * 必须重写clone方法


## 2.3 Clone 源码分析

* 浅拷贝

```java
/**
     * Returns a shallow copy of this <tt>ArrayList</tt> instance.  (The
     * elements themselves are not copied.)
     *
     * @return a clone of this <tt>ArrayList</tt> instance
     */
    public Object clone() {
        try {
            //这里是Object.clone(),因为AbstractList里面没有clone方法
            ArrayList<?> v = (ArrayList<?>) super.clone();
            //浅拷贝List，新建了一个List
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }
```

## 2.4 RandomAccess接口

1. **介绍** 标记接口由List实现使用， 以表明它们支持快速(通常为恒定时间)随机访问

   此接口的主要目的是允许通用算法更改其行为，以便在**随机访问列表**或者**顺序访问列表**时提供良好的性能。

   如果实现此接口， 循环随机访问比迭代器循环访问快

## 2.5 AbstractList 抽象类

​	此类提供的骨干实现的[`List`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/List.html)接口以最小化来实现该接口由一个“随机访问”数据存储备份所需的工作（如阵列）。 对于顺序存取的数据（如链接列表）， [`AbstractSequentialList`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/AbstractSequentialList.html)应优先使用此类。

​	要实现一个不可修改的列表，程序员只需要扩展这个类并提供[`get(int)`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/AbstractList.html#get-int-)和[`size()`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/List.html#size--)方法的实现。

​	要实现可修改的列表，程序员必须另外覆盖[`set(int, E)`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/AbstractList.html#set-int-E-)方法（否则会抛出一个`UnsupportedOperationException` ）。 如果列表是可变大小，则程序员必须另外覆盖[`add(int, E)`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/AbstractList.html#add-int-E-)和[`remove(int)`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/AbstractList.html#remove-int-)方法。

​	根据`Collection`接口规范中的建议，程序员通常应该提供一个void（无参数）和[集合](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/Collection.html)构造函数。

​	不像其他的抽象集合实现，程序员*不必*提供迭代器实现; 迭代器和列表迭代器由此类实现的，对的“随机访问”方法上： [`get(int)`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/AbstractList.html#get-int-) ， [`set(int, E)`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/AbstractList.html#set-int-E-) ， [`add(int, E)`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/AbstractList.html#add-int-E-)和[`remove(int)`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/util/AbstractList.html#remove-int-) 。

​	该类中每个非抽象方法的文档详细描述了其实现。 如果正在实施的集合承认更有效的实现，则可以覆盖这些方法中的每一种。

# 3. 源码分析

## 3.1成员变量

```java
private static final long serialVersionUID = 8683452581122892189L;
    private static final int DEFAULT_CAPACITY = 10;
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    transient Object[] elementData; // non-private to simplify nested class access
    private int size;
```



## 3.2 构造方法

| Constructor                                 | 描述                                                       |
| ------------------------------------------- | ---------------------------------------------------------- |
| public ArrayList(int initialCapacity)       | 构造具有初始化容量的空列表                                 |
| public ArrayList()                          | 构造一个初始化容量为10的空列表                             |
| public ArrayList(Collection<? extends E> c) | 构造一个包含指定集合元素的列表，按照他们由集合迭代器的顺序 |

### public ArrayList()

实际上是构造的一个默认空容量的List

```java
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

### public ArrayList(int initialCapacity)

注意 当参数为0， 这个空容量的List和默认空容量List不同，这个会影响add方法的行为。

```java
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
}
```

### public ArrayList(Collection<? extends E> c)

将集合转成数组，然后复制该数组到elementData中，如果容量是0，那么赋值EMPTY_ELEMENTDATA。

**JDK1.6聚合框架bug：c.toArray might (incorrectly) not return Object[] (see 6260652)**

https://blog.csdn.net/qing0706/article/details/50750220

```java
public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
}
```

## 3.3 添加方法

| 方法名                                                  | 描述                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| public boolean add(E e)                                 | 将指定元素添加到此列表的末尾。                               |
| public void add(int index, E element)                   | 在此列表中的制定位置插入制定的元素。                         |
| public boolean addAll(Collection<? extends E> c)        | 按制定集合的Iterator返回的顺序将制定集合中的所有元素追加到此列表的末尾 |
| public boolean AddAll(index, Collection<? extends E> c) | 将指定集合中的所有元素到此列表中，从制定位置开始             |

### public boolean add(E e)

* 扩容
* 在末尾添加元素

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

### public void add(int index, E element)

* 首先进行越界检查
* 扩容
* 将要插入位置以及之后的元素往后挪动一位
* 设置元素
* 增加size

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
            size - index);
    elementData[index] = element;
    size++;
}
```

### public boolean addAll(Collection<? extends E> c)

* 将集合转为数组
* 扩容
* 复制数组集合到elementData末尾
* 改变size
* 如果c长度为0返回false，否则返回true

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```



### public boolean AddAll(index, Collection<? extends E> c)

* 边界检查
* 集合c转数组a
* 校验扩容
* 移动元素
* 将a元素复制list中
* 调整size

```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

## 3.4 修改方法

* 越界检查
* 取出当前index的值
* 修改index位置的值
* 返回之前index位置的值

```java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

## 3.5 获取方法

* 越界检查
* 获取数组中元素

```java
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```

## 3.6 toString()方法

该方法继承于AbstractCollection

* 获取迭代器
* 如果没有数据返回"[]"
* 有数据则使用StringBuilder拼接,按逗号隔开
* 如果某个元素是collection本身，显示则值是"(this Collection)" ,否则是元素的toString()

```java
public String toString() {
    Iterator<E> it = iterator();
    if (! it.hasNext())
        return "[]";

    StringBuilder sb = new StringBuilder();
    sb.append('[');
    for (;;) {
        E e = it.next();
        sb.append(e == this ? "(this Collection)" : e);
        if (! it.hasNext())
            return sb.append(']').toString();
        sb.append(',').append(' ');
    }
}
```

## 3.7 迭代器

```java
public Iterator<E> iterator() {
    //创建一个内部类的对象
    return new Itr();
}

/**
 * An optimized version of AbstractList.Itr
 */
private class Itr implements Iterator<E> {
    int cursor;       // 光标，默认是0
    int lastRet = -1; //  记录-1
    int expectedModCount = modCount; //将集合实际修改次数赋值给预期修改次数

   	//判断集合是否有元素
    public boolean hasNext() {
        // 光标不等于size
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        //校验预期修改次数和实际修改集合次数一样
        checkForComodification();
        //光标复制为i
        int i = cursor;
        //判断，如果大于集合size就说明没有元素
        if (i >= size)
            throw new NoSuchElementException();
        //把集合存储数组地址赋值给局部变量
        Object[] elementData = ArrayList.this.elementData;
        //判断i是否大于集合容量
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        //光标自增
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

### 并发修改异常产生原因分析

以下代码抛出并发修改异常

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {//删除C之后，因为size发生改变，所以hasNext还是为true
    String s = iterator.next(); // checkForComodification失败 然后报异常
    if ("C".equals(s)) {
        list.remove("C");
    }
}
```

### 并发修改异常产生的特殊情况

以下代码不会产生并发修改异常

因为移除C之后，size变成2，判断hasNext的时候，光标2和size一样。

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("C");
list.add("B");
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String s = iterator.next();
    if ("C".equals(s)) {
        list.remove("C");
    }
}
```

### 迭代器默认remove方法

```java
public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    //校验是否并发修改异常
    checkForComodification();

    try {
        //删除元素
        ArrayList.this.remove(lastRet);
        //光标异动到最后记录位置
        cursor = lastRet;
        //最后记录位置变成-1
        lastRet = -1;
        //预期修改次数更新
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```

## 3.8 clear方法

* 修改次数自增
* 清空元素
* size设置为0

```java
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```

## 3.9 contains&indexOf方法

contains直接调用indexOf

indexOf方法:

* 如果为null，就找到一个为null的返回
* 否则用equals方法去查找相等的
* 如果都找不到返回-1

```java
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}

public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```



## 3.10 isEmpty方法

```java
public boolean isEmpty() {
    return size == 0;
}
```

## 3.11 其他方法

//todo 

# 4.常见面试题

## 4.1 ArrayList扩容机制

第一次扩容10,以后每次都是原容量的1.5倍。

## 4.2 如何解决扩容带来的性能问题

设置初始化容量或者使用public void ensureCapacity(int minCapacity)直接进行扩容。

## 4.3 ArrayList插入和删除元素一定比LinkedList慢么

LinkedList移除时需要先查找到节点，再删除，所以不一定更快。

## 4.4 ArrayList是线程安全的么？

不是，如果需要线程安全，可以用Vector或者或者用Collections.synchronizedList方法变成一个安全的集合。

```java
public static <T> List<T> synchronizedList(List<T> list) {
    return (list instanceof RandomAccess ?
            new SynchronizedRandomAccessList<>(list) :
            new SynchronizedList<>(list));
}
```

## 4.5 如何复制某个ArrayList到另外一个ArrayList中去

* clone
* 构造方法
* addAll

## 4.6 多线程读写问题

Q:已知成员变量集合存储N多用户名称，在多线程环境下，使用迭代器在读取数据集合的同事如何保证还可以正常的写入数据到集合。

A: 用CopyOnWriteArrayList，它所有可变操作都是对底层数组的最新的副本实现

## 4.7 ArrayList和LinkedList的区别

* ArrayList
  1. 基于动态数组的数据结构
  2. 对于随机访问get和set，ArrayList要优于LinkedList
  3. 对于随机操作的add和remove，ArrayList不一定比LinkedList慢（ArrayList底层优于是动态数组，因此并不是每次add和remove都需要创建新的数组)
* LinkedList
  1. 基于链表的数据结构
  2. 对于顺序操作，LinkedList不一定比ArrayList慢
  3. 对于随机操作，LinkedList效率明显较低



