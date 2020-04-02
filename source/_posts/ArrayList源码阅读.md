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

//todo 未完待续