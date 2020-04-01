---
title: ArrayList扩容机制
date: 2020-03-28 13:05:07
tags: [java,java容器,List,ArrayList,java源代码]
---

[TOC]

# ensureCapacity方法

## 来自于某个CSDN博客

//来源于 https://blog.csdn.net/wei_chong_chong/article/details/44419197

我们在使用Arraylist时，经常要对它进行初始化工作，在使用add()方法增加新的元素时，如果要初始化的数据量很大，应该使用ensureCapacity()方法，该方法的作用是增加Arraylist的大小，这样可以大大提高初始化速度。



![image-20200328130605117](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200328130605117.png)

## 源代码解读

1. 首先求得minExpand,ArrayList为空时为默认值10

   ​	因为如果为空，add时会将List的容量设置为默认容量10, 所以如果当空List扩容小于10就无需扩容了。

```java
/**
     * Increases the capacity of this <tt>ArrayList</tt> instance, if
     * necessary, to ensure that it can hold at least the number of elements
     * specified by the minimum capacity argument.
     *
     * @param   minCapacity   the desired minimum capacity
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
```

2. 然后直接扩容

   * 首先将异动次数+1
   * 然后如果所需最小容量小于当前elementData的容量就不扩容了
   * 否则扩容

   ```java
     private void ensureExplicitCapacity(int minCapacity) {
           modCount++;
   
           // overflow-conscious code
           if (minCapacity - elementData.length > 0)
               grow(minCapacity);
       }
   ```

   

# group方法(扩容机制)

1. 首先尝试扩容1.5倍，如果还是小于最小所需容量，则直接设置为最小所需容量(这里也可以判断溢出)

2. 然后如果新的容量比最大容量(MAX_ARRAY_SIZE)还大，则设置为int的最大值，否则返回MAX_ARRAY_SIZE
3. 最后一句就是说，最小容量通常和size的容量差不多，所以这样就ok

```java
 /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```