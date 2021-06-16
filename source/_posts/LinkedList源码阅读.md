---
title: LinkedList源码阅读
date: 2020-04-04 19:51:48
tags: [java,java容器,List,LinkedList,java源代码]
---

# 1.ArrayList引发的思考

* 优点：查询快

* 缺点：

  1. 增删慢，消耗cpu性能

     ​	情况一： 指定索引上的添加

     ​	情况二：如果原数组中元素已经不够了

  2.  比较浪费内存空间

* 有没有一种数据结构可以用多少空间就申请多少空间，并且又能够提高他的增删速度呢？

# 2.Linked集合底层数据结构

## 2.1链表

链表的分类：单链表，双链表，循环链表

* 链表：由链将一个个元素连接，每一个元素我们通常将其称之为Node 节点

* Node节点： 由两部分组成

  1. 数据值的变量

  2. Node next用来存放下一个节点的Node对象

  3. Node pre用来存放上一个节点的Node对象

* 链表和数组的区别

  1. 链表查询慢(因为链表没有索引)，但是增删快

**LinkedList底层实现采用双链表**

# 2.2 LinkedList介绍

双链表实现了`List`和`Deque`接口。 实现所有可选列表操作，并允许所有元素（包括`null` ）。

所有的操作都能像双向列表一样预期。 索引到列表中的操作将从开始或结束遍历列表，以更接近指定的索引为准。

**请注意，此实现不同步。**

# 3.成员变量

```java
 transient int size = 0;

 transient Node<E> first;

 transient Node<E> last;
```



# 4.继承关系



```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
}
```



## Deque接口

Deque接口的链表实现



# 5.构造方法

| 方法名                                       | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| public LinkedList()                          | 构造一个空列表                                               |
| public LinkedList(Collection<? extends E> c) | 构造一个包含指定元素集合的列表，按照它们由集合迭代器返回的顺序 |



## public LinkedList()

```java
public LinkedList() {
}
```

## public LinkedList(Collection<? extends E> c)

* 调用默认构造器
* 使用addAll方法将集合c的元素添加到该List数据中

```java
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```



# 6.Node内部类

以下摘抄自:https://www.cnblogs.com/aademeng/articles/6192944.html

```
一、非静态内部类：

1、变量和方法不能声明为静态的。(类的编译顺序：外部类--静态方法或属性--内部类，如果内部类声明为静态的，造成编译顺序冲突。个人理解)

2、实例化的时候需要依附在外部类上面。比如：B是A的非静态内部类，实例化B，则：A.B b = new A().new B();

3、内部类可以引用外部类的静态或者非静态属性或者方法。

二、静态内部类：

1、属性和方法可以声明为静态的或者非静态的。

2、实例化静态内部类：比如：B是A的静态内部类，A.B b = new A.B();

3、内部类只能引用外部类的静态的属性或者方法。

4、如果属性或者方法声明为静态的，那么可以直接通过类名直接使用。比如B是A的静态内部类，b（）是B中的一个静态属性，则可以：A.B.b();
```

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

# 7.添加方法

| 方法名                                                      | 描述                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| public boolean add(E e)                                     | 将指定的元素追加到此列表的末尾。                             |
| public void add(int index, E element)                       | 在此列表中的指定位置插入指定的元素。                         |
| public boolean addAll(Collection<? extends E> c)            | 按照指定集合的迭代器返回的顺序将指定集合中的所有元素追加到此列表的末尾。 |
| public boolean addAll(int index, Collection<? extends E> c) | 将指定集合中的所有元素插入到此列表中，从指定的位置开始。     |
| public void addFirst(E e)                                   | 在该列表开头插入指定的元素。                                 |
| public void addLast(E e)                                    | 将指定的元素追加到此列表的末尾。                             |

## public boolean add(E e) 

直接调用linkLast

* 取出last赋值为l
* 新建一个node对象，l作为新node的pre，e作为新node的element，null作为新node的next
* 如果l为null代表集合为空，那么设置第一个也为新node
* 如果l不为空，将l的下一个设置为新node
* size自增
* 修改次数自增

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}
void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
}
```

## public void add(int index, E element)

* 越界检查
* 如果是加在末尾，调用linkLast方法
* 如果是加在中间，首先需要查找到index节点的node，然后把要加的数据加载该节点前面

**Node<E> node(int index)**  

* 如果index < size的一半，从前往后查
* 如果index >= size的一半，从后往前查

```java
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
Node<E> node(int index) {
        // assert isElementIndex(index);
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
}
```

## addAll

只有集合参数的addAll直接调用index+集合参数的addAll，所以只分析后者。

* 边界检查
* 将集合c转为数组，如果该数组为空，直接return false
* 如果不为空，先求出这个数组第一个node的前驱和最后node的后驱
* 从前驱开始，把数组一个一个添加到list中
* 最后把后驱拼接给数组的最后一个node
* 最后改变Size
* modCount自增(和ArrayList不一样)

```java
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```


# 8.修改方法

* 越界检查
* 查找index的node为x
* 赋值x的数据到oldVal中
* 修改x的数据
* 返回oldVal

```java
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```

# 9.获取方法

* 越界检查
* 调用node(index)，然后返回其数据

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

# 10.contains和indexOf

**indexOf**

* 遍历整个链表，找到第一个o.equals数据的位置返回index
* 特别的，如果o等于null，返回第一个null出现的index

**contains**

* 调用indexOf，然后判断是否为-1

```java
public boolean contains(Object o) {
    return indexOf(o) != -1;
}
  public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
}
```

# 11.迭代器

和ArrayList一样会引发并发修改异常，再此不再赘述，移步博文ArrayList源码阅读即可。

```java
private class ListItr implements ListIterator<E> {
    private Node<E> lastReturned; //上一个返回值
    private Node<E> next; //下一个
    private int nextIndex; //下一个的index
    private int expectedModCount = modCount; //预期修改值

    ListItr(int index) {
        // assert isPositionIndex(index);
        //构造器只初始化next和nextIndex
        next = (index == size) ? null : node(index);
        nextIndex = index;
    }

    public boolean hasNext() {
        return nextIndex < size;
    }

    public E next() {
        //预期修改数和实际修改数检查，
        checkForComodification();
        if (!hasNext())
            throw new NoSuchElementException();

        lastReturned = next;
        next = next.next;
        nextIndex++;
        return lastReturned.item;
    }

    public boolean hasPrevious() {
        return nextIndex > 0;
    }

    public E previous() {
        checkForComodification();
        if (!hasPrevious())
            throw new NoSuchElementException();

        lastReturned = next = (next == null) ? last : next.prev;
        nextIndex--;
        return lastReturned.item;
    }

    public int nextIndex() {
        return nextIndex;
    }

    public int previousIndex() {
        return nextIndex - 1;
    }

    public void remove() {
        checkForComodification();
        if (lastReturned == null)
            throw new IllegalStateException();

        Node<E> lastNext = lastReturned.next;
        unlink(lastReturned);
        if (next == lastReturned)
            next = lastNext;
        else
            nextIndex--;
        lastReturned = null;
        expectedModCount++;
    }

    public void set(E e) {
        if (lastReturned == null)
            throw new IllegalStateException();
        checkForComodification();
        lastReturned.item = e;
    }

    public void add(E e) {
        checkForComodification();
        lastReturned = null;
        if (next == null)
            linkLast(e);
        else
            linkBefore(e, next);
        nextIndex++;
        expectedModCount++;
    }

    public void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (modCount == expectedModCount && nextIndex < size) {
            action.accept(next.item);
            lastReturned = next;
            next = next.next;
            nextIndex++;
        }
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```



# 12.常见面试题

//todo





