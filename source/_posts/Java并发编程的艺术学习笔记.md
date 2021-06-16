---
title: Java并发编程的艺术学习笔记
date: 2020-11-06 11:22:52
tags: [Java, 并发编程]
---

# 第1章 并发编程的挑战

## 1.1 上下文切换

​	CPU会不断切换线程执行，每个时间片一般是几十毫秒。执行一个时间片后，会保存上一个任务的状态，然后切换到下一个任务。任务的保存到加载的过程就是一次上下文切换。

### 1.1.1 多线程一定快么？

​	看例子ConcurrencyTest。

![](https://pic.imgdb.cn/item/5fa4c3c41cd1bbb86b92a423.jpg)

### 1.1.2 测试上下文切换的次数和时长

* 使用Lmbench3来测量切换时长
* 用vmstat(linux)测量上下文切换的次数。

![](https://pic.imgdb.cn/item/5fa4c5891cd1bbb86b930150.jpg)

https://www.cnblogs.com/ftl1012/p/vmstat.html

Procs（进程）：

>  r: 运行队列中进程数量
>
>  b： 等待IO的进程数量

Memory（内存）：

>  swpd: 使用虚拟内存大小
>
>  free: 可用内存大小
>
>  buff: 用作缓冲的内存大小
>
>  cache: 用作缓存的内存大小

Swap：

 si: 每秒从交换区写到内存的大小

>  so: 每秒写入交换区的内存大小

IO：（现在的Linux版本块的大小为1024bytes）

>  bi: 每秒读取的块数
>
>  bo: 每秒写入的块数

系统：

> in: 每秒中断数，包括时钟中断。【interrupt】
>
> cs: 每秒上下文切换数。    【count/second】

CPU（以百分比表示）：

>  us: 用户进程执行时间(user time)
>
>  sy: 系统进程执行时间(system time)
>
>  id: 空闲时间(包括IO等待时间),中央处理器的空闲时间 。以百分比表示。
>
>  wa: 等待IO时间

### 1.1.3 如何减少上下文切换

* 无锁并发编程。
* CAS算法。
* 使用最少线程。
* 协程。

### 1.1.4 减少上下文切换实战

​	通过减少线上大量WAITTING的线程，来减少上下文切换次数。

​	第一步： 使用jstack命令dump线程信息。

```
sudo -u admin /opt/ ifeve/ java/ bin/ jstack 31177 > /home/ tengfei. fangtf/ dump17
```

​	第二步：统计所有线程处于什么状态，发现300多个线程处于WAITING状态。

```
grep java. lang. Thread. State dump17 | awk '{print $ 2$ 3$ 4$ 5}' | sort | uniq -c
```

​	第三步：打开dump文件查看处于WATING的线程都在做什么

​	第四步：减少对应的工作线程数。

​	第五步：重启应用，然后重新统计WAITING线程。

## 1.2 死锁

避免死锁的几个常见方法。

* 避免一个线程同时获取多个锁。
* 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源。
* 尝试使用定时锁，使用lock.tryLock(timeout)来替代使用内部锁机制。
* 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会解锁失败。

## 1.3 资源限制的挑战

略

# 第2章 Java并发机制的底层实现原理

## 2.1 vlolatile的应用

**1.volatile的定义与实现原理**

​	![](https://pic.imgdb.cn/item/5fa4d0571cd1bbb86b95617c.jpg)

![](https://pic.imgdb.cn/item/5fa4d08f1cd1bbb86b956d1b.jpg)

Lock前缀的指令在多个处理器下会引发两件事情：

1) 将当前处理器缓存行的数据写回到系统内存。

2) 这个写回内存的操作会使其他CPU里混存了该内存地址的数据无效。

​	每个处理器通过嗅探在总显示传播的数据来检查自己的缓存的值是不是过期了，当发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置为无效状态。

volatile的两条实现原则：

1) Lock前缀指令会引起处理器缓存会写到内存。

​	Lock 前缀 指令 导致 在 执行 指令 期间， 声言 处理器 的 LOCK# 信号。 在 多 处理器 环境 中， LOCK# 信号 确保 在 声言 该 信号 期间， 处理器 可以 独占 任何 共享 内存[ 2]。 但是， 在最 近的 处理器 里， LOCK＃ 信号 一般 不 锁 总线， 而是 锁 缓存， 毕竟 锁 总线 开销 的 比较 大。 在 8. 1. 4 节 有 详细 说明 锁定 操作 对 处理器 缓存 的 影响， 对于 Intel486 和 Pentium 处理器， 在 锁 操作 时， 总是 在 总线 上 声言 LOCK# 信号。 但在 P6 和 目前 的 处理器 中， 如果 访问 的 内存 区域 已经 缓存 在 处理器 内部， 则 不会 声言 LOCK# 信号。 相反， 它 会 锁定 这块 内存 区域 的 缓存 并 回 写到 内存， 并使 用 缓存 一致性 机制 来 确保 修改 的 原子 性， 此 操作 被称为“ 缓存 锁定”， 缓存 一致性 机制 会 阻止 同时 修改 由 两个 以上 处理器 缓存 的 内存 区域 数据。

2) 一个处理器的缓存回写到内存会导致其他处理器的缓存无效。

​	IA- 32 处理器 和 Intel 64 处理器 使用 MESI（ 修改、 独占、 共享、 无效） 控制 协议 去 维护 内部 缓存 和 其他 处理器 缓存 的 一致性。 在 多 核 处理器 系统 中进 行 操作 的 时候， IA- 32 和 Intel 64 处理器 能 嗅探 其他 处理器 访问 系统 内存 和 它们 的 内部 缓存。 处理器 使用 嗅探 技术 保证 它的 内部 缓存、 系统 内存 和 其他 处理器 的 缓存 的 数据 在 总线 上 保持一致。 例如， 在 Pentium 和 P6 family 处理器 中， 如果 通过 嗅探 一个 处理器 来 检测 其他 处理器 打算 写 内存 地址， 而这 个 地址 当前 处于 共享 状态， 那么 正在 嗅探 的 处理器 将使 它的 缓存 行 无效， 在下 次 访问 相同 内存 地址 时， 强制 执行 缓存 行 填充。

**2. volatile的使用优化**

​	Doug lea在JDK 7 并发包立新增了一个队列集合类`LinkedTransferQueue`。它使用volatile变量时，通过一种追加字节的方式来优化队列出队和入队的性能。

​	jdk7中没看到这个代码。。。，所以相关笔记不记录。应该在com.google.code.yanf4j.util下。

![](https://pic.imgdb.cn/item/5fa4d78d1cd1bbb86b970919.jpg)



## 2.2 synchronized的实现原理与引用

* 普通同步方法，锁时当前实例对象。
* 对于静态同步方法，锁时当前类的Class对象。
* 对于同步代码块，锁时Synchronized括号里配置的对象。

​	代码块同步是使用monitor enter和monitor exit指令实现的。方法同步是使用另外一种方式实现的(规范里没有详细说明，但是也可以通过这两个指令来实现)。

### 2.2.1 Java对象头

​	synchronized锁时存在Java对象头里面。如果是数组类型，虚拟机用3个字宽的对象头，如果是非数组类型，则用2字宽对象头(32位虚拟机)。

​	![](https://pic.imgdb.cn/item/5fa4dcc01cd1bbb86b982ffe.jpg)

![](https://pic.imgdb.cn/item/5fa4dce51cd1bbb86b9838ec.jpg)

### 2.2.2 锁的升级与对比

​	Java 1.6 为了减少获得锁和释放锁带来的性能小号，引入了"偏向锁"和"轻量级锁"。

​	无锁 > 偏向锁 > 轻量级锁 > 重量级锁。锁只能升级不能降级。

**1. 偏向锁**

​	锁不仅不存在多线程竞争，而且总是由同一线程多次获得。所以引入偏向锁。当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID, 以后该线程在进入和退出代码块时，不需要CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否存储指向当前线程的偏向锁。如果测试成功，就代表获取了锁。失败时，需要测试一下Mark Word中偏向锁的表示是否设置为1：如果没有设置，则使用CAS竞争锁；如果设置了，则尝试使用CAS将对象的偏向锁指向当前线程。

​	(1) 偏向锁的撤销

​	当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。

​	它 会首 先 暂停 拥有 偏向 锁 的 线程， 然后 检查 持有 偏向 锁 的 线程 是否 活着， 如果 线程 不 处于 活动 状态， 则 将对 象头 设置 成 无 锁 状态； 如果 线程 仍然 活着， 拥有 偏向 锁 的 栈 会被 执行， 遍历 偏向 对象 的 锁 记录， 栈 中的 锁 记录 和 对 象头 的 Mark Word 要么 重新 偏向 于 其他 线程， 要么 恢复 到 无 锁 或者 标记 对象 不适合 作为 偏向 锁， 最后 唤醒 暂停 的 线程。

​	![](https://pic.imgdb.cn/item/5fa5f52a1cd1bbb86bd24e80.jpg)

​	(2) 关闭偏向锁

在jdk6和7中默认启用偏向锁，但是要程序启动几秒之后才激活。可以关闭延迟：`- XX: `BiasedLockingStartupDelay= 0`。如果大部分情况下锁处于竞争状态，可以关闭偏向锁：

`- XX:- UseBiasedLocking= false`, 那么会默认进入到轻量级锁。

**2. 轻量级锁**

​	(1) 轻量级锁加锁

​	JVM在执行同步块前会在当前线程栈帧中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中。然后尝试使用CAS将对象头中的Mark Word替换成指向锁记录的指针。如果成功，当前线程获得锁，失败，代表有竞争。那么使用自旋来获取锁。

​	(2) 轻量级锁解锁

​	解锁时，会使用原子的CAS操作将Mark Word替换回到对象头。成功，没有竞争发生。如果失败，表示由竞争，锁就会膨胀成重量级锁。

![](https://pic.imgdb.cn/item/5fa5f8381cd1bbb86bd2e8e6.jpg)

​	因为自旋会消耗CPU，为了避免无用的自旋。一旦升级成重量级锁，不会降级为轻量级锁。当 锁 处于 这个 状态 下， 其他 线程 试图 获取 锁 时， 都会 被 阻塞 住， 当 持有 锁 的 线程 释放 锁 之后 会 唤醒 这些 线程， 被 唤醒 的 线程 就会 进行 新 一轮 的 夺 锁 之争。

**3.锁的优点和缺点**

![](https://pic.imgdb.cn/item/5fa5fa2d1cd1bbb86bd33687.jpg)

## 2.3 原子操作的实现原理

**1.术语定义**

![](https://pic.imgdb.cn/item/5fa605641cd1bbb86bd4dafe.jpg)

**2.处理器如何实现原子操作**

​	32位IA-32处理器使用基于对缓存加锁或对总线加锁的方式来实现多处理器之间的原子操作。处理器自动保证基本的内存操作的原子性。意思是当一个处理器读取一个字节时，其他处理器不能访问这个字节的内存地址。Pentium 6最新的处理器可以自动保证单处理器对同一缓存行里进行16/32/64位的操作是原子的，但是复杂的内存操作处理器不能自动保证其原子性，比如跨总线宽度、跨多个缓存行的跨页表的访问。处理器提供总线锁定和缓存锁定两个机制来保证复杂内存操作的原子性。

​	(1) 使用总线锁保证原子性

​	总线锁就是使用处理器提供的一个LOCK#信号，当一个处理器在总线上输出此型号，其他处理器的请求将被阻塞住，那么该处理器可以独占共享锁。

​	(2) 使用缓存锁保证原子性

​	在同一时刻，只需要保证某个内存地址的操作是原子性的。频繁使用的内存会缓存在处理器的L1, L2, L3的高速缓存里，那么原子操作就可以直接在处理器内部缓存中进行。在Pentium 6和目前的处理器中可以使用"缓存锁定"的方式来实现复杂的原子性。

​	所谓“ 缓存 锁定” 是指 内存 区域 如果 被 缓存 在 处理器 的 缓存 行中， 并且 在 Lock 操作 期间 被 锁定， 那么 当它 执行 锁 操作 回 写到 内存 时， 处理器 不在 总线 上 声言 LOCK＃ 信号， 而是 修改 内部 的 内存 地址， 并 允许 它的 缓存 一致性 机制 来 保证 操作 的 原子 性， 因为 缓存 一致性 机制 会 阻止 同时 修改 由 两个 以上 处理器 缓存 的 内存 区域 数据， 当 其他 处理器 回 写 已被 锁定 的 缓存 行的 数据 时， 会使 缓存 行 无效，

​	有两种情况不会使用缓存锁定。

* 操作数据不能被缓存在处理器内部，或操作的数据跨多个缓存行时。
* 有些处理器不支持缓存。



​	针对以上两条机制，可以通过Intel处理器提供了很多Lock前缀的指令来实现。

**3. Java如何实现原子操作**

​	(1) 使用循环CAS实现原子操作。

​	JVM的CAS操作利用了处理器提供的CMPXCHG指令实现的。基本思路是循环进行CAS操作直到成功为止。

```java
public class Counter {

    private AtomicInteger atomicI = new AtomicInteger(0);
    private int           i       = 0;

    public static void main(String[] args) {
        final Counter cas = new Counter();
        List<Thread> ts = new ArrayList<Thread>(600);
        long start = System.currentTimeMillis();
        for (int j = 0; j < 100; j++) {
            Thread t = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int i = 0; i < 10000; i++) {
                        cas.count();
                        cas.safeCount();
                    }
                }
            });
            ts.add(t);
        }
        for (Thread t : ts) {
            t.start();

        }
        // �ȴ������߳�ִ�����
        for (Thread t : ts) {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
        System.out.println(cas.i);
        System.out.println(cas.atomicI.get());
        System.out.println(System.currentTimeMillis() - start);
    }

    /**
     * ʹ��CASʵ���̰߳�ȫ������
     */
    private void safeCount() {
        for (;;) {
            int i = atomicI.get();
            boolean suc = atomicI.compareAndSet(i, ++i);
            if (suc) {
                break;
            }
        }
    }

    /**
     * ���̰߳�ȫ������
     */
    private void count() {
        i++;
    }

}
```

(2) CAS实现原子操作的三大问题

* ABA问题：原来是A，变成了B，后面有变成了A。可以使用版本号来解决。1A->2B>3A。JDK的Atomic包提供了一个类`AtomicStampedReference`来解决ABA问题。
* 循环时间长开销大：如果JVM能支持处理器提供的pause指令，效率会有一定提升。第一，可以延迟流水线执行指令，使CPU不会消耗过多的执行资源，在一些处理器上延迟时间是0; 第二，可以避免在退出循环时，因内存顺序冲突引起的CPU流水线被清空，从而提高CPU的执行效率。
* 只能保证一个共享变量的原子操作。

(3) 使用锁机制来实现原子操作

​	锁机制保证了只有获得锁的线程才能够操作锁定的内存区域。除了偏向锁，JVM实现锁的方式都用了循环CAS。

# 第3章 Java内存模型

## 3.1 Java内存模型的基础

### 3.1.1 并发编程模型的两个关键维问题。

​	并发编程需要处理两个关键问题：线程之间如何通信以及线程之间如何同步的问题。在命令式编程汇总，线程之间通信的机制有两种：共享内存和消息传递。

​	在共享内存的并发模型里，线程之间共享程序的公共状态，通过写-读内存中的公共状态进行隐式通信。在消息传递的模型里面，线程之间没有公共状态，线程之间必须通过发送消息来进行显示通信。

​	同步是指程序中用于控制不同线程之间操作发生相对顺序的机制。在共享内存的模型中，同步是显示进行的。在消息传递的模型中，消息发送必须在消息接收之前是隐式的。

​	Java的并发采用的是共享内存模型。线程之间的通信是隐式进行的，但是对程序员完全透明。

### 3.1.2 Java内存模型的抽象结构

​	局部变量、方法定义参数和异常处理器参数不会在线程之间共享。

​	Java线程之间的通信由JMM控制：线程之间的共享变量存储在主内存中，每个线程都有一个私有的本地内存，本地内存中存储了该现场以读/写共享的变量的副本。

![](https://pic.imgdb.cn/item/5fa614cb1cd1bbb86bd77a9d.jpg)

线程AB要通信。

1) 线程A把本地内存A汇总更新过的共享变量刷新到主内存中去。

2) 线程B到主内存中去读取线程A之前已经更新过的共享变量。

![](https://pic.imgdb.cn/item/5fa625d61cd1bbb86bda9f9b.jpg)

### 3.1.3 从源代码到指令荀烈的重排序。

​	重排序分3种：

1) 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。

2) 指令集并行的重排序。现在处理器采用了指令集并行技术来讲多条指令重叠执行。如果不存在数据依赖项，处理器可以改变语句对应的机器指令的执行顺序。

3) 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，使得加载和存储操作看上去可能在乱序执行。

![](https://pic.imgdb.cn/item/5fa629a71cd1bbb86bdb72e6.jpg)





​	1属于编译器重排序，2和3处于处理器重排序。对于编译器，JMM的编译重排序规则会禁止特定类型的编译器重排序。对于处理器重排序，JMM的处理器重排序规则会要求Java编译器在生成指令集序列时，插入特定类型的内存屏障。

### 3.1.4 并发编程模型的分类

​		现代处理器使用写缓冲区临时保存向内存才能写入数据。每个处理器上的写缓冲区仅仅对它所在的处理器可见：处理器对内存的读/写的执行顺序，不一定与内存的发生的读/写顺序一样。

![](https://pic.imgdb.cn/item/5fa639fe1cd1bbb86bdf18d4.jpg)

原因如下图所示：

![](https://pic.imgdb.cn/item/5fa63a551cd1bbb86bdf29cd.jpg)



​	从内存操作实际发生的顺序来看，知道处理器A执行A3来刷新自己的写缓存区，写操作A1才算真正执行了。

![](https://pic.imgdb.cn/item/5fa63c3d1cd1bbb86bdf8801.jpg)

​	常见的处理器都允许Store-Load重排序。常见的都不允许对存在数据依赖的操作做重排序。sparc-TSO和x86有用较强的处理器内存模型，他们仅允许对写-读操作做重排序。

​	为了保证内存可见性，Java会插入内存屏障。

![](https://pic.imgdb.cn/item/5fa63cff1cd1bbb86bdfabe4.jpg)

​	StoreLoad是一个全能型的屏障，现代的多处理器大多支持。执行该屏障开销会很昂贵，因为要把写缓冲区的数据全部刷新到内存中。

### 3.1.5 happens-before简介

​	从JDK5开始，Java使用新的JSR-133内存模型。它用happens-before的概念来阐述操作之间的内存可见性。在JMM中，如果一个操作的执行的结果需要对另一个操作可见，那么就必须存在happens-before关系。

* 程序顺序规则：一个线程中的每个操作，happens-before与该线程中的任意后续操作。
* 监视锁规则：一个锁的解锁，happens-before于随后对这个锁的加锁。
* volatile规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
* 传递性：如果A happens-before B， B happens-before C，那么 A happens-before C。

![](https://pic.imgdb.cn/item/5fa640621cd1bbb86be0585d.jpg)

## 3.2 重排序

### 3.2.1 数据依赖性

​	如果两个操作访问同一个变量，且两个操作中有一个为写操作，此时两个操作之间就存在数据依赖性：

![](https://pic.imgdb.cn/item/5fa641bd1cd1bbb86be0a828.jpg)

### 3.2.2 as-if-serial语义

​	意思是：不管怎么重排序，(单线程)程序的执行结果不会被改变。

### 3.2.3 程序顺序规则

1） A 　 happens- before B。 
2） B 　 happens- before C。 
3） A 　 happens- before C。



​	这里的A happens-before B，但是实际执行的B却可以排在A之前执行。JMM不一定要求A一定要在B之前执行，仅仅要求前一个操作的执行结果对后一个操作可见，且前一个操作按顺序排在第二个操作之前。如果操作A的执行结果不需要对B可见，而且重排序后的操作A和操作B的执行结果一致，就允许这种重排序。

### 3.2.4 重排序对多线程的影响

​	略

## 3.3 顺序一致性

### 3.3.1  数据竞争与顺序一致性

​	数据未正确同步，就可能存在数据竞争。数据竞争定义如下：

​	在一个线程中写一个变量，

​	在另一个线程读一个变量，

​	而且写和读没有通过同步来排序。

​	JMM对正确同步的多线程程序的内存一致性做了如下保证：

​	如果程序时正确同步的，程序的执行将具有一致性——即程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同。

### 3.3.2 顺序一致性内存模型

1) 一个线程中所有操作必须按照程序的顺序来执行。

2）所有线程只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程看见。

![](https://pic.imgdb.cn/item/5fa64a0d1cd1bbb86be26c82.jpg)

​	概念上，顺序一致性模型有一个单一的全局内存，这个内存通过一个开关可以连接到任意一个线程，同时每个线程必须按程序的顺序进行内存读写。(在顺序一致性模型中，所有操作具有全序关系)。

​	有两个线程A,B。操作是A1->A2->A3, B1->B2->B3。

​	如果正确同步，A操作后释放监视器锁，然后B线程获取同一个监视器锁。

![](https://pic.imgdb.cn/item/5fa64bd51cd1bbb86be2cb32.jpg)



如果没有做同步。

![](https://pic.imgdb.cn/item/5fa64bf21cd1bbb86be2d1d5.jpg)

### 3.3.3 同步程序的顺序一致性效果

​	![](https://pic.imgdb.cn/item/5fa64fdb1cd1bbb86be3bf01.jpg)

![](https://pic.imgdb.cn/item/5fa6500c1cd1bbb86be3c766.jpg)

### 3.3.4 未同步程序的执行特性

​	对于未同步或者未正确同步的多线程程序，JMM只提供最小安全性：读取到的值，要么是某个线程写入的值，要么是默认值。为了实现最小安全性，JVM在堆上分配对象时，首先会对内存空间进行清零，然后才会在上面分配对象。

​	JMM不保证未同步的执行结果与该程序在顺序一致性模型中执行的结果一致。如果要一致，JMM要禁用大量的处理器和编译器优化。

​	

​	1） 顺序 一致性 模型 保证 单 线程 内 的 操作 会 按 程序 的 顺序 执行， 而 JMM 不保 证 单 线程 内 的 操作 会 按 程序 的 顺序 执行。

​	 2） 顺序 一致性 模型 保证 所有 线程 只能 看到 一致 的 操作 执行 顺序， 而 JMM 不保 证 所有 线程 能看 到 一致 的 操作 执行 顺序。

​	3） JMM 不 保证 对 64 位 的 long 型 和 double 型 变量 的 写 操作 具有 原子 性， 而 顺序 一致性 模型 保证 对 所有 的 内存 读/ 写 操作 都 具有 原子 性。

​	和处理器的总线工作机制有关。总线会同步试图并发使用总线的事务。在每一个处理器执行总线事务期间，总线会禁用其他处理器和I/O设备执行内存的读/写。

![](https://pic.imgdb.cn/item/5fa652231cd1bbb86be43675.jpg)

​	如果A,B,C同时发起总线事务，总线仲裁会对竞争做出裁决。

​	这个工作机制可以把所有处理器对内存的访问以串行化来执行。在任意时间点，最多只有一个处理器可以访问你内存。这个特性确保了单个总线事务中的内存读/写操作的原子性。当JVM在32位处理器上允许时，可能会把一个64位long/double变量的写操作拆分为两个32位的写操作来执行。这两个写操作可能分配到不同的总线事务中执行，此时对这个64位变量的写操作将不具有原子性。

​	![](https://pic.imgdb.cn/item/5fa653321cd1bbb86be47467.jpg)



​	在JSR-133之前的旧内存模型中，64位的读写操作都可以拆分成两个操作。JSR-133开始，只允许拆分写操作。

## 3.4 volatile的内存语义

### 3.4.1 volatile的特性

​	理解volatile特性的一个好方法是把volatile变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步。

* 可见性：对一个volatile变量的读，总是能看到(任意线程)对这个volatile变量最后的写入。
* 原子性：对任意单个volatile变量的读/写具有原子性，但类似于a++这种复合操作不具有原子性。

### 3.4.2 volatile写-读简历的happens-before关系

​	对程序员来说，volatile对线程的内存可见性的影响比volatile自身的特性更为重要。

​	JSR-133开始, volatile变量的写-读可以实现线程之间的通信。volatile的写和锁的释放由相同的内存语义；读和锁的获取有相同语义。

​	![](https://pic.imgdb.cn/item/5fa65cc01cd1bbb86be68238.jpg)

假设线程A执行writer()之后，线程B执行reader()方法。

1） 根据 程序 次序 规则， 1 happens- before 2; 3 happens- before 4。

 2） 根据 volatile 规则， 2 happens- before 3。 

3） 根据 happens- before 的 传递性 规则， 1 happens- before 4。

![](https://pic.imgdb.cn/item/5fa65d091cd1bbb86be68f74.jpg)

### 3.4.3 volatile写-读的内存意义

​	当写一个volatile变量时，JMM会把该线程对应的本地内存的共享变量值刷新到主内存。

![](https://pic.imgdb.cn/item/5fa65da51cd1bbb86be6aab7.jpg)



volatile读的内存语义如下：

​	当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来从主内存中读取共享变量。

​	对volatile写和读的内存语义做个总结

* 线程A写一个volatile变量，实际上是线程A向接下来要读这个volatile变量的某个线程发出了消息。
* 线程B读一个volatile变量，实际上是线程B接收了某个之前线程发出的消息。
* 线程A写一个volatile变量，虽有线程B读这个变量，实际上是线程A通过主内存向线程B发送消息。

![](https://pic.downk.cc/item/5faa73e51cd1bbb86bbd2709.jpg)

### 3.4.4 volatile内存语义的实现

​	![](https://pic.downk.cc/item/5faa741b1cd1bbb86bbd3bf3.jpg)

* 当第二个操作是volatile写，不管第一个操作是什么，都不能重排序。
* 当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。
* 当第一个操作是volatile写，第二个操作是volatile读/写时，不能重排序。



​	一个最优不止来最小化插入屏障总数几乎不可能。为此,JMM采取保守策略。

* 每一个volatile写的前面，插入一个StoreStore屏障。
* 每一个volatile写后面，插入StoreLoad屏障。
* 在每个volatile读操作的后面插入一个LoadLoad屏障。
* 在每个volatile读操作后面插入一个LoadStore屏障。

![](https://pic.downk.cc/item/5faa76bd1cd1bbb86bbe0f65.jpg)

​	volatile写后面的StoreLoad屏障。编译器无法准确判断volatile写后面是否需要插入一个StoreLoad屏障。JMM采取保守策略：要么在volatile写后面或者每个volatile读前面插入一个StoreLoad屏障。最后选择了写后面加。

![](https://pic.downk.cc/item/5faa773a1cd1bbb86bbe2cc8.jpg)

示例：

![](https://pic.downk.cc/item/5faa778b1cd1bbb86bbe42cd.jpg)

![](https://pic.downk.cc/item/5faa77a91cd1bbb86bbe4d7a.jpg)

​	上面 的 优化 针对 任意 处理器 平台， 由于 不同 的 处理器 有 不同“ 松紧 度” 的 处理器 内存 模型， 内存 屏障 的 插入 还可以 根据 具体 的 处理器 内存 模型 继续 优化。 以 X86 处理器 为例， 图  中 除 最后 的 StoreLoad 屏障 外， 其他 的 屏障 都会 被 省略。

​	前文 提 到过， X86 处理器 仅 会对 写- 读 操作 做 重 排序。 X86 不 会对 读- 读、 读- 写 和 写- 写 操作 做 重 排序， 因此 在 X86 处理器 中会 省略 掉 这 3 种 操作 类型 对应 的 内存 屏障。 在 X86 中， JMM 仅 需 在 volatile 写 后面 插入 一个 StoreLoad 屏障 即可 正确 实现 volatile 写- 读 的 内存 语义。 这 意味着 在 X86 处理器 中， volatile 写的 开销 比 volatile 读 的 开销 会 大 很多。

### 3.4.5 JSR-133为什么要增强volatile内存语义

​	为了 提供 一种 比 锁 更轻 量级 的 线程 之间 通信 的 机制， JSR- 133 专家组 决定 增强 volatile 的 内存 语义。如果 读者 想在 程序 中用 volatile 代替 锁， 请 一定 谨慎， 具体 详情 请参阅 Brian Goetz 的 文章《 Java 理论 与 实践： 正确 使用 Volatile 变量》。

## 3.5 锁的内存语义

### 3.5.1 锁的释放-获取简历的happens-before关系

​	锁除了让临界区互斥执行之外，还可以让释放锁的线程向获取同一个锁的线程发送消息。

​	![](https://pic.downk.cc/item/5faa95a21cd1bbb86bc58371.jpg)

假设 线程 A 执行 writer() 方法， 随后 线程 B 执行 reader() 方法。 

根据 happens- before 规则， 这个 过程 包含 的 happens- before 关系 可以 分为 3 类。

 1） 根据 程序 次序 规则， 1 happens- before 2, 2 happens- before 3; 4 happens- before 5, 5 happens- before 6。 

2） 根据 监视器 锁 规则， 3 happens- before 4。 

3） 根据 happens- before 的 传递性， 2 happens- before 5。



![](https://pic.downk.cc/item/5faa95b81cd1bbb86bc587fc.jpg)

### 3.5.2 锁的释放和获取的内存语义

​	![](https://pic.downk.cc/item/5faa96331cd1bbb86bc5aa9a.jpg)

​	当线程获取锁时，JMM会把该现场对应的本地内存置为无效。

![](https://pic.downk.cc/item/5faa96c61cd1bbb86bc5cee9.jpg)

* 线程 A 释放 一个 锁， 实质上 是 线程 A 向 接下来 将要 获取 这个 锁 的 某个 线程 发出 了（ 线程 A 对 共享 变量 所做 修改 的） 消息。 
* 线程 B 获取 一个 锁， 实质上 是 线程 B 接收 了 之前 某个 线程 发出 的（ 在 释放 这个 锁 之前 对 共享 变量 所做 修改 的） 消息。 
* 线程 A 释放 锁， 随后 线程 B 获取 这个 锁， 这个 过程 实质上 是 线程 A 通过 主 内存 向 线程 B 发送 消息。

### 3.5.3 锁内存语义的实现

​	![](https://pic.downk.cc/item/5faa98ac1cd1bbb86bc649f4.jpg)

![](https://pic.downk.cc/item/5faa98c71cd1bbb86bc65086.jpg)

ReentrantLock 的 实现 依赖于 Java 同步 器 框架 AbstractQueuedSynchronizer（ 本文 简称 之为 AQS）。 AQS 使用 一个 整型 的 volatile 变量（ 命名为 state） 来 维护 同步 状态， 马上 我们 会 看到， 这个 volatile 变量 是 ReentrantLock 内存 语义 实现 的 关键。

ReentrantLock分为公平锁和非公平锁。

公平锁加锁轨迹如下：

1) ReetrantLock:lock();

2) FairSync:lock();

3) AbstractQueuedSynchronizer:acquire(int arg);

4) ReetrantLock:tryAcquire(int acquires);

第四步才真正开始加锁。

![](https://pic.downk.cc/item/5faa99421cd1bbb86bc66c17.jpg)

加锁时先读volatile变量state。

公平锁的释放如下：

1） ReentrantLock:unlock()。 

2） AbstractQueuedSynchronizer:release( int arg)。 

3） Sync:tryRelease( int releases)。

第三步才是真正释放锁。

![](https://pic.downk.cc/item/5faa9a341cd1bbb86bc6a63c.jpg)



非公平锁加锁：

1） ReentrantLock:lock()。

2） NonfairSync:lock()。 

3） AbstractQueuedSynchronizer:compareAndSetState( int expect, int update)。

 <img src="https://pic.downk.cc/item/5faa9bb71cd1bbb86bc70a61.jpg" style="zoom:200%;" />

​	编译器 不 会对 volatile 读 与 volatile 读后面 的 任意 内存 操作 重 排序； 编译器 不 会对 volatile 写 与 volatile 写 前面 的 任意 内存 操作 重 排序。意味着 为了 同时 实现 volatile 读 和 volatile 写的 内存 语义， 编译器 不 能对 CAS 与 CAS 前面 和 后面 的 任意 内存 操作 重 排序。

​	对 应于 intel X86 处理器 的 C++源 代码 的 片段。

![](https://pic.downk.cc/item/5faa9c461cd1bbb86bc72bfa.jpg)

intel 的 手册 对 lock 前缀 的 说明 如下。

 1） 确保 对 内存 的 读- 改- 写 操作 原子 执行。 在 Pentium 及 Pentium 之前 的 处理器 中， 带有 lock 前缀 的 指令 在 执行 期间 会 锁住 总线， 使得 其他 处理器 暂时 无法 通过 总线 访问 内存。 很 显然， 这 会 带来 昂贵 的 开销。 从 Pentium 4、 Intel Xeon 及 P6 处理器 开始， Intel 使用 缓存 锁定（ Cache Locking） 来 保证 指令 执行 的 原子 性。 缓存 锁定 将 大大 降低 lock 前缀 指令 的 执行 开销。 

2） 禁止 该 指令， 与 之前 和 之 后的 读 和 写 指令 重 排序。

 3） 把 写 缓冲区 中的 所有 数据 刷新 到 内存 中。

* 公平 锁 和 非 公平 锁 释放 时， 最后 都要 写 一个 volatile 变量 state。

* 

* 公平 锁 获取 时， 首先 会 去 读 volatile 变量。

* 非 公平 锁 获取 时， 首先 会用 CAS 更新 volatile 变量， 这个 操作 同时 具有 volatile 读 和 volatile 写的 内存 语义。

  

对 ReentrantLock 的 分析 可以 看出， 锁 释放- 获取 的 内存 语义 的 实现 至少 有 下面 两种 方式。 

1） 利用 volatile 变量 的 写- 读 所 具有 的 内存 语义。

 2） 利用 CAS 所 附带 的 volatile 读 和 volatile 写的 内存 语义。

### 3.5.4 concurrent包的实现

现在Java线程通信有了下面4种方式：

* A线程写volatile变量，随后B线程读这个volatile变量。
* A线程写volatile变量，随后B线程用CAS更新这个volatile变量。
* A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。
* A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。



​	现代处理器都会去支持某种能对内存执行原子性读-改-写操作的原子指令。把这些整合在一起，就形成了整个concurrent包得以实现的基石。

​	concurrent包有一个通用化的实现模式。

1. 声明共享变量为volatile。

2. 然后使用CAS原子条件更新来实现线程之间的同步。

3. 配合以volatile的读/写和CAS所具有的volatile读和谐的内存语义来试下你先吃之间的通信。

   ![](https://pic.imgdb.cn/item/5fab69941cd1bbb86bec096d.jpg)

## 3.6 final域的内存语义

### 3.6.1 final域的重排序规则

​	编译器和处理器要遵循两个重排序规则

1. 在构造器内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作不能重排序。
2. 除此读一个包含final域对象的引用，与随后初次读这个对象的final域，不能重排序。

### 3.6.2 写final域的重排序规则

1. JMM禁止编译把final域的写重排序到构造函数之外。
2. 编译器会在final域写之后，构造器return之前，插入一个StoreStore屏障。这个屏障禁止fi把final域的写重排序到构造函数之外。

写 final 域 的 重排 序 规则 可以 确保： 在 对象 引用 为 任意 线程 可见 之前， 对象 的 final 域 已经 被 正确 初始化 过了。

### 3.6.3 读final域的重排序规则

​		编译器会在读final域操作前面加入一个LoadLoad屏障。

​		初次读对应引用与初次读该对象包含的final域，这两个操作之间存在间接依赖关系。大多数处理器不会重排序这两个操作。但有少数处理器会(比如alpha处理器)，这个规则就是专门用来针对这种处理器的。

​	这个规则可以确保：在读一个对象final域之前，一定会先读这个final域对象的引用。

### 3.6.4 fiinal域为引用类型

​	对于 引用 类型， 写 final 域 的 重排 序 规则 对 编译器 和 处理器 增加 了 如下 约束： 在 构造 函数 内 对 一个 final 引用 的 对象 的 成员 域 的 写入， 与 随后 在 构造 函数 外 把 这个 被 构造 对象 的 引用 赋值 给 一个 引用 变量， 这 两个 操作 之间 不能 重 排序。

![](https://pic.imgdb.cn/item/5fab71431cd1bbb86beda2f8.jpg)

​	假设 首先 线程 A 执行 writerOne() 方法， 执行 完 后 线程 B 执行 writerTwo() 方法， 执行 完 后 线程 C 执行reader() 方法。

​	1 是对 final 域 的 写入， 2 是对 这个 final 域 引用 的 对象 的 成员 域 的 写入， 3 是把 被 构造 的 对象 的 引用 赋值 给 某个 引用 变量。 这里 除了 前面 提到 的 1 不能 和 3 重 排序 外， 2 和 3 也不能 重 排序。

​	JMM 可以 确保 读 线程 C 至少 能 看到 写 线程 A 在 构造 函数 中 对 final 引用 对象 的 成员 域 的 写入。 即 C 至少 能看 到数 组 下标 0 的 值 为 1。 而 写 线程 B 对数 组 元素 的 写入， 读 线程 C 可能 看得 到， 也可 能看 不到。

### 3.6.5 为什么final引用不能从构造函数内"逸出"

​	在 构造 函数 返回 前， 被 构造 对象 的 引用 不能 为 其他 线程 所见， 因为 此时 的 final 域 可能 还没有 被 初始化。 在 构造 函数 返回 后， 任意 线程 都将 保证 能看 到 final 域 正确 初始化 之 后的 值。

### 3.6.6 final语义在处理器中的实现

​	由于 X86 处理器 不 会对 写- 写 操作 做 重 排序， 所以 在 X86 处理器 中， 写 final 域 需要 的 StoreStore 屏障会被 省略 掉。 同样， 由于 X86 处理器 不会 对 存在 间接 依赖 关系 的 操作 做 重 排序， 所以 在 X86 处理器 中， 读 final 域 需要 的 LoadLoad 屏障 也会 被 省略 掉。 也就是说， 在 X86 处理器 中， final 域 的 读/ 写 不会 插入 任何 内存 屏障！

### 3.6.7 JSR-133为什么要增强final的语义

在 旧的 Java 内存 模型 中， 一个 最 严重 的 缺陷 就是 线程 可能 看到 final 域 的 值 会 改变。 比如， 一个 线程 当前 看到 一个 整型 final 域 的 值 为 0（ 还未 初始化 之前 的 默认值）， 过 一段时间 之后 这个 线程 再去 读 这个 final 域 的 值 时， 却 发现 值 变为 1（ 被 某个 线程 初始化 之 后的 值）。最 常见 的 例子 就是 在 旧的 Java 内存 模型 中， String 的 值 可能 会 改变。
	只要 对象 是 正确 构造 的（ 被 构造 对象 的 引用 在 构造 函数 中 没有“ 逸出”）， 那么 不需要 使用 同步（ 指 lock 和 volatile 的 使用） 就可以 保证 任意 线程 都能 看到 这个 final 域 在 构造 函数 中 被 初始化 之 后的 值。

##  3.7 happens-before

### 3.7.1 JMM的设计

​	需要考虑两个关键因素：

* 程序员对内存模型的使用。易于理解，易于编程。希望一个强内存模型。
* 编译器和处理器对内存模型的实现。希望约束少，可以做更多优化。希望一个弱内存模型。



​	JMM采取如下设计：

* 对于 会 改变 程序 执行 结果 的 重 排序， JMM 要求 编译器 和 处理器 必须 禁止 这种 重 排序。 
* 对于 不会 改变 程序 执行 结果 的 重 排序， JMM 对 编译器 和 处理器 不做 要求（ JMM 允许 这种 重 排序）。

![](https://pic.imgdb.cn/item/5fab76e61cd1bbb86beeee05.jpg)

* JMM 向 程序员 提供 的 happens- before 规则 能 满足 程序员 的 需求。

* JMM 对 编译器 和 处理器 的 束缚 已经 尽可能 少。

  

### 3.7.2 happens-before的定义

《JSR- 133: Java Memory Model and Thread Specification》 对 happens- before 关系 的 定义 如下。

1） 如果 一个 操作 happens- before 另一个 操作， 那么 第一个 操作 的 执行 结果 将对 第二个 操作 可见， 而且 第一个 操作 的 执行 顺序 排在 第二个 操作 之前。

 2） 两个 操作 之间 存在 happens- before 关系， 并不 意味着 Java 平台 的 具体 实现 必须 要 按照 happens- before 关系 指定 的 顺序 来 执行。 如果 重排 序 之后 的 执行 结果， 与 按 happens- before 关系 来 执行 的 结果 一致， 那么 这种 重排 序 并不 非法（ 也就是说， JMM 允许 这种 重 排序）。

### 3.7.3 happens-before规则

1） 程序 顺序 规则： 一个 线程 中的 每个 操作， happens- before 于 该 线程 中的 任意 后续 操作。

2） 监视器 锁 规则： 对 一个 锁 的 解 锁， happens- before 于 随后 对 这个 锁 的 加锁。 3） volatile 变量 规则： 对 一个 volatile 域 的 写， happens- before 于 任意 后续 对 这个 volatile 域 的 读。

4） 传递性： 如果 A happens- before B， 且 B happens- before C， 那么 A happens- before C。 

5） start() 规则： 如果 线程 A 执行 操作 ThreadB. start()（ 启动 线程 B）， 那么 A 线程 的 ThreadB. start() 操作 happens- before 于 线程 B 中的 任意 操作。

 6） join() 规则： 如果 线程 A 执行 操作 ThreadB. join() 并 成功 返回， 那么 线程 B 中的 任意 操作 happens- before 于 线程 A 从 ThreadB. join() 操作 成功 返回。

## 3.8 双重检查锁定与延迟初始化

​	双重 检查 锁定 是 常见 的 延迟 初始化 技术， 但它 是一 个 错误 的 用法。 本文 将 分析 双重 检查 锁定 的 错误 根源， 以及 两种 线程 安全 的 延迟 初始化 方案。

### 3.8.1  双重检查锁定的由来

![](https://pic.imgdb.cn/item/5fab78281cd1bbb86bef2f77.jpg)

* 多个线程视图在同一时间创建对象，会通过加锁来保证只有一个线程能创建对象。
* 在对象创建好之后，执行getInstance()方法不需要获取锁，直接返回已创建好的对象。

### 3.8.2 问题的根源

​	第7行创建对象代码可以分解为如下三行伪代码：

![](https://pic.imgdb.cn/item/5fab78f21cd1bbb86bef538d.jpg)

​	2和3之间可能会发生重排序。

### 3.8.3 预计volatile的解决方案

​	![](https://pic.imgdb.cn/item/5fab79da1cd1bbb86bef80d8.jpg)

​	注意该解决方案需要基于JDK5及以上

### 3.8.4 基于类初始化的解决方案

​	![](https://pic.imgdb.cn/item/5fab7a521cd1bbb86bef9fba.jpg)

![](https://pic.imgdb.cn/item/5fab7a761cd1bbb86befb006.jpg)

​	Java 语言 规范 规定， 对于 每一个 类 或 接口 C， 都有 一个 唯一 的 初始化 锁 LC 与之 对应。 从 C 到 LC 的 映射， 由 JVM 的 具体 实现 去 自由 实现。 JVM 在 类 初始化 期间 会 获取 这个 初始化 锁， 并且 每个 线程 至少 获取 一次 锁 来 确保 这个 类 已经 被 初始化 过了（ 事实上， Java 语言 规范 允许 JVM 的 具体 实现 在这里 做 一些 优化， 见 后文 的 说明）。

根据 Java 语言 规范， 在 首次 发生 下列 任意 一种 情况 时， 一个 类 或 接口 类型 T 将被 立即 初始化。 

1） T 是 一个 类， 而且 一个 T 类型 的 实例 被 创建。 

2） T 是 一个 类， 且 T 中 声明 的 一个 静态 方法 被 调用。 

3） T 中 声明 的 一个 静态 字段 被 赋值。 

4） T 中 声明 的 一个 静态 字段 被 使用， 而且 这个 字段 不是 一个 常量 字段。

5） T 是 一个 顶 级 类（ Top Level Class， 见 Java 语言 规范 的 § 7. 6）， 而且 一个 断言 语句 嵌套 在 T 内部 被 执行。

​	第 1 阶段： 通过 在 Class 对象 上 同步（ 即 获取 Class 对象 的 初始化 锁）， 来 控制 类 或 接口 的 初始化。 这个 获取 锁 的 线程 会 一直 等待， 直到 当前 线程 能够 获取 到 这个 初始化 锁。

​	第 2 阶段： 线程 A 执行 类 的 初始化， 同时 线程 B 在 初始化 锁 对应 的 condition 上 等待。

​	第 3 阶段： 线程 A 设置 state= initialized， 然后 唤醒 在 condition 中 等待 的 所有 线程。

​	第 4 阶段： 线程 B 结束 类 的 初始化 处理。

​	第 5 阶段： 线程 C 执行 类 的 初始化 的 处理。

但 基于 volatile 的 双重 检查 锁定 的 方案 有一个 额外 的 优势： 除了 可以 对 静态 字段 实现 延迟 初始化 外， 还可 以对 实例 字段 实现 延迟 初始化。

## 3.9 Java内存模型综述

### 3.9.1 处理器内存的模型

​	常见处理器的内存模型划分为如下几种类型。

* 放松 程序 中写- 读 操作 的 顺序， 由此 产生了 Total Store Ordering 内存 模型（ 简称 为 TSO）。

* 在上面的基础上，继续放松程序中写-写操作的顺序，因此产生了Partial StoreOrder内存模型(简称PSO)。

* 在前面 两条 的 基础上， 继续 放松 程序 中 读- 写 和 读- 读 操作 的 顺序， 由此 产生了 Relaxed Memory Order 内存 模型（ 简称 为 RMO） 和 PowerPC 内存 模型。

  ![](https://pic.imgdb.cn/item/5fab80421cd1bbb86bf0e8f8.jpg)

  

![](https://pic.imgdb.cn/item/5fab80771cd1bbb86bf0f47d.jpg)

### 3.9.2 各种内存模型之间的关系

​	![](https://pic.imgdb.cn/item/5fab81491cd1bbb86bf126ba.jpg)

​	JMM 是一 个 语言 级 的 内存 模型， 处理器 内存 模型 是 硬件 级 的 内存 模型， 顺序 一致性 内存 模型 是一 个 理论 参考 模型。

### 3.9.3 JMM的内存可见性

* 单线 程 程序。 单线 程 程序 不会 出现 内存 可见 性 问题。 编译器、 runtime 和 处理器 会 共同 确保 单线 程 程序 的 执行 结果 与 该 程序 在 顺序 一致性 模型 中的 执行 结果 相同。 
* 正确 同步 的 多 线程 程序。 正确 同步 的 多 线程 程序 的 执行 将 具有 顺序 一致性（ 程序 的 执行 结果 与 该 程序 在 顺序 一致性 内存 模型 中的 执行 结果 相同）。 这是 JMM 关注 的 重点， JMM 通过 限制 编译器 和 处理器 的 重 排序 来 为 程序员 提供 内存 可见 性 保证。 
* 未 同步/ 未 正确 同步 的 多 线程 程序。 JMM 为 它们 提供 了 最小 安全性 保障： 线程 执行 时 读取 到 的 值， 要么 是 之前 某个 线程 写入 的 值， 要么 是 默认值（ 0、 null、 false）。

![](https://pic.imgdb.cn/item/5fab82931cd1bbb86bf17c2c.jpg)

### 3.9.4 JSR-133对旧内存模型的修补

​	主要有两个：

* 增强 volatile 的 内存 语义。 旧 内存 模型 允许 volatile 变量 与 普通 变量 重 排序。 JSR- 133 严格 限制 volatile 变量 与 普通 变量 的 重 排序， 使 volatile 的 写- 读 和 锁 的 释放- 获取 具有 相同 的 内存 语义。 
* 增强 final 的 内存 语义。 在 旧 内存 模型 中， 多次 读取 同一个 final 变量 的 值 可能 会 不 相同。 为此， JSR- 133 为 final 增加 了 两个 重排 序 规则。 在 保证 final 引用 不会 从 构造 函数 内 逸出 的 情况下， final 具有 了 初始化 安全性。

# 第4章 Java并发编程基础

## 4.1 线程简介

### 4.1.1 什么是线程

​	现代操作系统调度的最小单元是线程，也叫轻量级进程。

```java
public class MultiThread {

    public static void main(String[] args) {
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
       
        for (ThreadInfo threadInfo : threadInfos) {
            System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
        }
    }
}
/**
[8] JDWP Command Reader
[7] JDWP Event Helper Thread
[6] JDWP Transport Listener: dt_socket
[5] Attach Listener
[4] Signal Dispatcher
[3] Finalizer
[2] Reference Handler
[1] main
*/
```

### 4.1.2 为什么要使用多线程

(1) 更多的处理器核心

(2) 更快的响应时间

(3) 更好的变成模型

### 4.1.3 线程优先级

​	在Java线程中，通过一个整型成员变量priority来控制优先级，范围1~10。默认是5。针对频繁阻塞(休眠或IO操作)的线程需要设置较高优先级，而偏重计算的线程则设置较低的优先级。在不同的JVM以及操作系统上，线程规划会存在差异，有些操作系统甚至会忽略对线程优先级的设定。

```java
public class Priority {
    private static volatile boolean notStart = true;
    private static volatile boolean notEnd   = true;

    public static void main(String[] args) throws Exception {
        List<Job> jobs = new ArrayList<Job>();
        for (int i = 0; i < 10; i++) {
            int priority = i < 5 ? Thread.MIN_PRIORITY : Thread.MAX_PRIORITY;
            Job job = new Job(priority);
            jobs.add(job);
            Thread thread = new Thread(job, "Thread:" + i);
            thread.setPriority(priority);
            thread.start();
        }
        notStart = false;
        Thread.currentThread().setPriority(8);
        System.out.println("done.");
        TimeUnit.SECONDS.sleep(10);
        notEnd = false;

        for (Job job : jobs) {
            System.out.println("Job Priority : " + job.priority + ", Count : " + job.jobCount);
        }

    }

    static class Job implements Runnable {
        private int  priority;
        private long jobCount;

        public Job(int priority) {
            this.priority = priority;
        }

        public void run() {
            while (notStart) {
                Thread.yield();
            }
            while (notEnd) {
                Thread.yield();
                jobCount++;
            }
        }
    }
}
/**
done.
Job Priority : 1, Count : 3944787
Job Priority : 1, Count : 3957283
Job Priority : 1, Count : 4208014
Job Priority : 1, Count : 4160023
Job Priority : 1, Count : 3975817
Job Priority : 10, Count : 4528591
Job Priority : 10, Count : 4712696
Job Priority : 10, Count : 4685421
Job Priority : 10, Count : 4534104
Job Priority : 10, Count : 4546863
*/
```



​	比较接近，说明程序的正确性不能依赖线程的优先级高低。

### 4.1.4 线程的状态

![](https://pic.imgdb.cn/item/5fabe1ac1cd1bbb86b0958ef.jpg)

下面使用jstack 尝试查看示例代码运行时的线程状态

```java
public class ThreadState {

    private static Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        new Thread(new TimeWaiting(), "TimeWaitingThread").start();
        new Thread(new Waiting(), "WaitingThread").start();
        // ʹ������Blocked�̣߳�һ����ȡ���ɹ�����һ��������
        new Thread(new Blocked(), "BlockedThread-1").start();
        new Thread(new Blocked(), "BlockedThread-2").start();
        new Thread(new Sync(), "SyncThread-1").start();
        new Thread(new Sync(), "SyncThread-2").start();
    }

    /**
     * ���̲߳��ϵĽ���˯��
     */
    static class TimeWaiting implements Runnable {
        @Override
        public void run() {
            while (true) {
                SleepUtils.second(100);
            }
        }
    }

    /**
     * ���߳���Waiting.classʵ���ϵȴ�
     */
    static class Waiting implements Runnable {
        @Override
        public void run() {
            while (true) {
                synchronized (Waiting.class) {
                    try {
                        Waiting.class.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    /**
     * ���߳���Blocked.classʵ���ϼ����󣬲����ͷŸ���
     */
    static class Blocked implements Runnable {
        public void run() {
            synchronized (Blocked.class) {
                while (true) {
                    SleepUtils.second(100);
                }
            }
        }
    }

    static class Sync implements Runnable {

        @Override
        public void run() {
            lock.lock();
            try {
                SleepUtils.second(100);
            } finally {
                lock.unlock();
            }

        }

    }
}
```

输入jps

![](https://pic.imgdb.cn/item/5fabe4271cd1bbb86b0a1eb9.jpg)

然后输入jstack 18576

![](https://pic.imgdb.cn/item/5fabe4721cd1bbb86b0a2d2f.jpg)

Java线程状态变迁如图

![](https://pic.imgdb.cn/item/5fabe49c1cd1bbb86b0a35e9.jpg)

### 4.1.5 Daemon线程

​	是一种支持性线程。当一个虚拟机中不存在非Daemon线程，Java虚拟机才会退出。可以调用Thread.setDaemon(true)将线程设置为Daemon线程。

​	Daemon 线程 被用 作 完成 支持 性 工作， 但是 在 Java 虚拟 机 退出 时 Daemon 线程 中的 finally 块 并不 一定 会 执行。

```java
public class Daemon {

    public static void main(String[] args) {
        Thread thread = new Thread(new DaemonRunner());
        thread.setDaemon(true);
        thread.start();
    }

    static class DaemonRunner implements Runnable {
        @Override
        public void run() {
            try {
                SleepUtils.second(100);
            } finally {
                System.out.println("DaemonThread finally run.");
            }
        }
    }
}
```

## 4.2 启动和终止线程

### 4.2.1 构造线程

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc) {
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }

    this.name = name;

    Thread parent = currentThread();
    SecurityManager security = System.getSecurityManager();
    if (g == null) {
        /* Determine if it's an applet or not */

        /* If there is a security manager, ask the security manager
           what to do. */
        if (security != null) {
            g = security.getThreadGroup();
        }

        /* If the security doesn't have a strong opinion of the matter
           use the parent thread group. */
        if (g == null) {
            g = parent.getThreadGroup();
        }
    }

    /* checkAccess regardless of whether or not threadgroup is
       explicitly passed in. */
    g.checkAccess();

    /*
     * Do we have the required permissions?
     */
    if (security != null) {
        if (isCCLOverridden(getClass())) {
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }

    g.addUnstarted();

    this.group = g;
    this.daemon = parent.isDaemon();
    this.priority = parent.getPriority();
    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    this.inheritedAccessControlContext =
            acc != null ? acc : AccessController.getContext();
    this.target = target;
    setPriority(priority);
    if (parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;

    /* Set thread ID */
    tid = nextThreadID();
}
```

一个 新 构造 的 线程 对象 是 由其 parent 线程 来 进行 空间 分配 的， 而 child 线程 继承 了 parent 是否 为 Daemon、 优先级 和 加载 资源 的 contextClassLoader 以及 可继承 的 ThreadLocal， 同时 还会 分配 一个 唯一 的 ID 来 标识 这个 child 线程。

### 4.2.2 启动线程。

​	调用start()方法。

> 注意 　 启动 一个 线程 前， 最好 为 这个 线程 设置 线程 名称， 因为 这样 在 使用 jstack 分析 程序 或者 进行 问题 排 查 时， 就会 给 开发 人员 提供 一些 提示， 自定义 的 线程 最好 能够 起 个 名字。

### 4.2.3 理解中断

​	中断可以理解为线程的一个标识位属性，它表示一个运行中的线程是否被其他线程进行了中断操作。

​	可以通过方法isInterruptted()来判断是否被中断，也可以调用静态方法Thread.interrupted()对当前线程的中断标识位复位。如果 该 线程 已经 处于 终结 状态， 即使 该 线程 被 中断 过， 在 调用 该 线程 对象 的 isInterrupted() 时 依旧 会 返回 false。

```java
public class Interrupted {

    public static void main(String[] args) throws Exception {
        // sleepThread不停的尝试睡眠
        Thread sleepThread = new Thread(new SleepRunner(), "SleepThread");
        sleepThread.setDaemon(true);
        // busyThread不停的运行
        Thread busyThread = new Thread(new BusyRunner(), "BusyThread");
        busyThread.setDaemon(true);
        sleepThread.start();
        busyThread.start();
        // 休眠5秒，让sleepThread和busyThread充分运行
        TimeUnit.SECONDS.sleep(5);
        sleepThread.interrupt();
        busyThread.interrupt();
        System.out.println("SleepThread interrupted is " + sleepThread.isInterrupted());
        System.out.println("BusyThread interrupted is " + busyThread.isInterrupted());
        // 防止sleepThread和busyThread立刻退出
        TimeUnit.SECONDS.sleep(2);
    }

    static class SleepRunner implements Runnable {
        @Override
        public void run() {
            while (true) {
                SleepUtils.second(10);
            }
        }
    }

    static class BusyRunner implements Runnable {
        @Override
        public void run() {
            while (true) {
            }
        }
    }
}
/**
SleepThread interrupted is false
BusyThread interrupted is true
**/
```

​	从 结果 可以 看出， 抛出 InterruptedException 的 线程 SleepThread， 其中 断 标识 位 被 清除 了， 而 一直 忙碌 运作 的 线程 BusyThread， 中断 标识 位 没有 被 清除。

### 4.2.4 过期的suspend()、resume()、和stop()

​	暂停、恢复和停止操作。

```java
public class Deprecated {
    @SuppressWarnings("deprecation")
    public static void main(String[] args) throws Exception {
        DateFormat format = new SimpleDateFormat("HH:mm:ss");
        Thread printThread = new Thread(new Runner(), "PrintThread");
        printThread.setDaemon(true);
        printThread.start();
        TimeUnit.SECONDS.sleep(3);
        // 将PrintThread进行暂停，输出内容工作停止
        printThread.suspend();
        System.out.println("main suspend PrintThread at " + format.format(new Date()));
        TimeUnit.SECONDS.sleep(20);
        // 将PrintThread进行恢复，输出内容继续
        printThread.resume();
        System.out.println("main resume PrintThread at " + format.format(new Date()));
        TimeUnit.SECONDS.sleep(3);
        // 将PrintThread进行终止，输出内容停止
        printThread.stop();
        System.out.println("main stop PrintThread at " + format.format(new Date()));
        TimeUnit.SECONDS.sleep(3);
    }

    static class Runner implements Runnable {
        @Override
        public void run() {
            DateFormat format = new SimpleDateFormat("HH:mm:ss");
            while (true) {
                System.out.println(Thread.currentThread().getName() + " Run at " + format.format(new Date()));
                SleepUtils.second(1);
            }
        }
    }
}
```

​	在执行过程中，PrintThread运行了3秒，随后被暂停，3秒后恢复，最后3秒被终止。

​	不建议使用的原因主要有：以suspend()为例，在调用后，线程不会释放已经占有的资源。容易引发死锁。

### 4.2.5 安全地终止线程

​	可以中断或者利用一个boolean变量来控制是否是否需要停止任务并终止该线程。

```java
public class Shutdown {
    public static void main(String[] args) throws Exception {
        Runner one = new Runner();
        Thread countThread = new Thread(one, "CountThread");
        countThread.start();
        // 睡眠1秒，main线程对CountThread进行中断，使CountThread能够感知中断而结束
        TimeUnit.SECONDS.sleep(1);
        countThread.interrupt();
        Runner two = new Runner();
        countThread = new Thread(two, "CountThread");
        countThread.start();
        // 睡眠1秒，main线程对Runner two进行取消，使CountThread能够感知on为false而结束
        TimeUnit.SECONDS.sleep(1);
        two.cancel();
    }

    private static class Runner implements Runnable {
        private long             i;

        private volatile boolean on = true;

        @Override
        public void run() {
            while (on && !Thread.currentThread().isInterrupted()) {
                i++;
            }
            System.out.println("Count i = " + i);
        }

        public void cancel() {
            on = false;
        }
    }
}
```

## 4.3 线程间通信

### 4.3.1 volatile和synchronized关键字

​	volatile关键字可以让所有线程感知到变化。过多使用会降低程序的执行效率。

​	关键字synchronized保证了线程对变量访问的可见性和排他性。中， 对于 同步 块 的 实现 使用 了 monitorenter 和 monitorexit 指令， 而 同步 方法 则是 依靠 方法 修饰 符 上 的 ACC_SYNCHRONIZED 来 完成 的。

![](https://pic.imgdb.cn/item/5fac92e41cd1bbb86b340eb7.jpg)

### 4.3.2 等待/通知机制

​	以下代码可以完成消费者的工作

```java
while (value != desire) { 
	Thread. sleep( 1000); 
} 
doSomething();
```

​	存在如下问题

​	1) 难以确保及时性。

​	2) 难以降低开销。

​	使用Object的通知/等待方法可以解决该问题。

![](https://pic.imgdb.cn/item/5fac950b1cd1bbb86b348bb8.jpg)

```java
public class WaitNotify {
    static boolean flag = true;
    static Object  lock = new Object();

    public static void main(String[] args) throws Exception {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        TimeUnit.SECONDS.sleep(1);

        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();
    }

    static class Wait implements Runnable {
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 当条件不满足时，继续wait，同时释放了lock的锁
                while (flag) {
                    try {
                        System.out.println(Thread.currentThread() + " flag is true. wait @ "
                                           + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                        lock.wait();
                    } catch (InterruptedException e) {
                    }
                }
                // 条件满足时，完成工作
                System.out.println(Thread.currentThread() + " flag is false. running @ "
                                   + new SimpleDateFormat("HH:mm:ss").format(new Date()));
            }
        }
    }

    static class Notify implements Runnable {
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 获取lock的锁，然后进行通知，通知时不会释放lock的锁，
                // 直到当前线程释放了lock后，WaitThread才能从wait方法中返回
                System.out.println(Thread.currentThread() + " hold lock. notify @ " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                lock.notifyAll();
                flag = false;
                SleepUtils.second(5);
            }
            // 再次加锁
            synchronized (lock) {
                System.out.println(Thread.currentThread() + " hold lock again. sleep @ "
                                   + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                SleepUtils.second(5);
            }
        }
    }
}
/**
Thread[WaitThread,5,main] flag is true. wait @ 09:58:16
Thread[NotifyThread,5,main] hold lock. notify @ 09:58:17
Thread[NotifyThread,5,main] hold lock again. sleep @ 09:58:22
Thread[WaitThread,5,main] flag is false. running @ 09:58:27
**/
```

1） 使用 wait()、 notify() 和 notifyAll() 时 需要 先 对调 用 对象 加锁。 

2） 调用 wait() 方法 后， 线程 状态 由 RUNNING 变为 WAITING， 并将 当前 线程 放置 到 对象 的 等待 队列。 

3） notify() 或 notifyAll() 方法 调 用后， 等待 线程 依旧 不会 从 wait() 返回， 需要 调用 notify() 或 notifAll() 的 线程 释放 锁 之后， 等待 线程 才有 机会 从 wait() 返回。 

4） notify() 方法 将 等待 队列 中的 一个 等待 线程 从 等待 队列 中 移到 同步 队列 中， 而 notifyAll() 方法 则是 将 等待 队列 中 所有 的 线程 全部 移到 同步 队列， 被 移动 的 线程 状态 由 WAITING 变为 BLOCKED。 

5） 从 wait() 方法 返回 的 前提 是 获得 了 调用 对象 的 锁。

### 4.3.3 等待/通知的经典范式

等待 方 遵循 如下 原则。 

1） 获取 对象 的 锁。 

2） 如果 条件 不满足， 那么 调用 对象 的 wait() 方法， 被 通知 后仍 要 检查 条件。 

3） 条件 满足 则 执行 对应 的 逻辑。

![](https://pic.imgdb.cn/item/5fac99901cd1bbb86b359aaf.jpg)

通知 方 遵循 如下 原则。 

1） 获得 对象 的 锁。 

2） 改变 条件。 

3） 通知 所有 等待 在 对象 上 的 线程。

![](https://pic.imgdb.cn/item/5fac99af1cd1bbb86b35a529.jpg)

### 4.3.4 管道输入/输出流

​	管道 输入/ 输出 流 和 普通 的 文件 输入/ 输出 流 或者 网络 输入/ 输出 流 不同 之处 在于， 它 主要 用于 线程 之间 的 数据 传输， 而 传输 的 媒介 为 内存。

```java
public class Piped {

    public static void main(String[] args) throws Exception {
        PipedWriter out = new PipedWriter();
        PipedReader in = new PipedReader();
        // 将输出流和输入流进行连接，否则在使用时会抛出IOException
        out.connect(in);

        Thread printThread = new Thread(new Print(in), "PrintThread");
        printThread.start();
        int receive = 0;
        try {
            while ((receive = System.in.read()) != -1) {
                out.write(receive);
            }
        } finally {
            out.close();
        }
    }

    static class Print implements Runnable {
        private PipedReader in;

        public Print(PipedReader in) {
            this.in = in;
        }

        public void run() {
            int receive = 0;
            try {
                while ((receive = in.read()) != -1) {
                    System.out.print((char) receive);
                }
            } catch (IOException ex) {
            }
        }
    }
}
```

### 4.3.5 Thread.join()的使用

​	含义是：当前线程A等待thread终止之后才从thread.join()返回。还提供了`joing(long mills)`和`join(long millis, int nanos)`。两个超时方法表示，如果线程thread在给定的超时时间里没有终止，那么将会从该超时方法中返回。

```java
public class Join {
    public static void main(String[] args) throws Exception {
        Thread previous = Thread.currentThread();
        for (int i = 0; i < 10; i++) {
            // 每个线程拥有前一个线程的引用，需要等待前一个线程终止，才能从等待中返回
            Thread thread = new Thread(new Domino(previous), String.valueOf(i));
            thread.start();
            previous = thread;
        }

        TimeUnit.SECONDS.sleep(5);
        System.out.println(Thread.currentThread().getName() + " terminate.");
    }

    static class Domino implements Runnable {
        private Thread thread;

        public Domino(Thread thread) {
            this.thread = thread;
        }

        public void run() {
            try {
                thread.join();
            } catch (InterruptedException e) {
            }
            System.out.println(Thread.currentThread().getName() + " terminate.");
        }
    }
}
/**
main terminate.
0 terminate.
1 terminate.
2 terminate.
3 terminate.
4 terminate.
5 terminate.
6 terminate.
7 terminate.
8 terminate.
9 terminate.
**/
```

每个线程终止的前提是前驱线程的终止。

```java
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

当线程终止时，会调用线程自身的notifyAll()方法，会通知所有等待在该现场对象上的线程。

### 4.3.6 ThreadLocal的使用

​	ThreadLocal，即线程变量，是一个以ThreadLocal对象为键、任意对象为值的存储结构。这个结构被附带在线程上。

```java
public class Profiler {
    // 第一次get()方法调用时会进行初始化（如果set方法没有调用），每个线程会调用一次
    private static final ThreadLocal<Long> TIME_THREADLOCAL = new ThreadLocal<Long>() {
                                                                protected Long initialValue() {
                                                                    return System.currentTimeMillis();
                                                                }
                                                            };

    public static final void begin() {
        TIME_THREADLOCAL.set(System.currentTimeMillis());
    }

    public static final long end() {
        return System.currentTimeMillis() - TIME_THREADLOCAL.get();
    }

    public static void main(String[] args) throws Exception {
        Profiler.begin();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("Cost: " + Profiler.end() + " mills");
    }
}
```

## 4.4 线程应用实例

### 4.4.1 等待超时模式

​	如果方法能够在给定的时间段内得到结果，那么立刻放回，反之，超时返回默认结果。

![](https://pic.imgdb.cn/item/5facd2571cd1bbb86b449194.jpg)

​	等待超时模式就是在等待/通知范式基础上加了超时控制。

### 4.4.2 一个简单的数据库连接池示例	

​	通过构造函数初始化连接的最大上限，通过一个双向队列来维护连接。调用放需要先调用fetchConnection(long) 方法来指定在多少毫秒内超时获取连接，当连接使用完成后，需要调用releaseConnection(Connection)方法将连接放回线程池。

```java
public class ConnectionPool {

    private LinkedList<Connection> pool = new LinkedList<Connection>();

    public ConnectionPool(int initialSize) {
        if (initialSize > 0) {
            for (int i = 0; i < initialSize; i++) {
                pool.addLast(ConnectionDriver.createConnection());
            }
        }
    }

    public void releaseConnection(Connection connection) {
        if (connection != null) {
            synchronized (pool) {
                // 添加后需要进行通知，这样其他消费者能够感知到链接池中已经归还了一个链接
                pool.addLast(connection);
                pool.notifyAll();
            }
        }
    }

    // 在mills内无法获取到连接，将会返回null
    public Connection fetchConnection(long mills) throws InterruptedException {
        synchronized (pool) {
            // 完全超时
            if (mills <= 0) {
                while (pool.isEmpty()) {
                    pool.wait();
                }

                return pool.removeFirst();
            } else {
                long future = System.currentTimeMillis() + mills;
                long remaining = mills;
                while (pool.isEmpty() && remaining > 0) {
                    pool.wait(remaining);
                    remaining = future - System.currentTimeMillis();
                }

                Connection result = null;
                if (!pool.isEmpty()) {
                    result = pool.removeFirst();
                }
                return result;
            }
        }
    }
}
```

### 4.4.3 线程池技术及其示例

​	![](https://pic.imgdb.cn/item/5facda8a1cd1bbb86b46c915.jpg)

```java
public class DefaultThreadPool<Job extends Runnable> implements ThreadPool<Job> {
    // 线程池最大限制数
    private static final int      MAX_WORKER_NUMBERS     = 10;
    // 线程池默认的数量
    private static final int      DEFAULT_WORKER_NUMBERS = 5;
    // 线程池最小的数量
    private static final int      MIN_WORKER_NUMBERS     = 1;
    // 这是一个工作列表，将会向里面插入工作
    private final LinkedList<Job> jobs                   = new LinkedList<Job>();
    // 工作者列表
    private final List<Worker>    workers                = Collections.synchronizedList(new ArrayList<Worker>());
    // 工作者线程的数量
    private int                   workerNum              = DEFAULT_WORKER_NUMBERS;
    // 线程编号生成
    private AtomicLong            threadNum              = new AtomicLong();

    public DefaultThreadPool() {
        initializeWokers(DEFAULT_WORKER_NUMBERS);
    }

    public DefaultThreadPool(int num) {
        workerNum = num > MAX_WORKER_NUMBERS ? MAX_WORKER_NUMBERS : num < MIN_WORKER_NUMBERS ? MIN_WORKER_NUMBERS : num;
        initializeWokers(workerNum);
    }

    public void execute(Job job) {
        if (job != null) {
            // 添加一个工作，然后进行通知
            synchronized (jobs) {
                jobs.addLast(job);
                jobs.notify();
            }
        }
    }

    public void shutdown() {
        for (Worker worker : workers) {
            worker.shutdown();
        }
    }

    public void addWorkers(int num) {
        synchronized (jobs) {
            // 限制新增的Worker数量不能超过最大值
            if (num + this.workerNum > MAX_WORKER_NUMBERS) {
                num = MAX_WORKER_NUMBERS - this.workerNum;
            }
            initializeWokers(num);
            this.workerNum += num;
        }
    }

    public void removeWorker(int num) {
        synchronized (jobs) {
            if (num >= this.workerNum) {
                throw new IllegalArgumentException("beyond workNum");
            }
            // 按照给定的数量停止Worker
            int count = 0;
            while (count < num) {
                workers.get(count).shutdown();
                count++;
            }
            this.workerNum -= count;
        }
    }

    public int getJobSize() {
        return jobs.size();
    }

    // 初始化线程工作者
    private void initializeWokers(int num) {
        for (int i = 0; i < num; i++) {
            Worker worker = new Worker();
            workers.add(worker);
            Thread thread = new Thread(worker, "ThreadPool-Worker-" + threadNum.incrementAndGet());
            thread.start();
        }
    }

    // 工作者，负责消费任务
    class Worker implements Runnable {
        // 是否工作
        private volatile boolean running = true;

        public void run() {
            while (running) {
                Job job = null;
                synchronized (jobs) {
                    // 如果工作者列表是空的，那么就wait
                    while (jobs.isEmpty()) {
                        try {
                            jobs.wait();
                        } catch (InterruptedException ex) {
                            // 感知到外部对WorkerThread的中断操作，返回
                            Thread.currentThread().interrupt();
                            return;
                        }
                    }
                    // 取出一个Job
                    job = jobs.removeFirst();
                }
                if (job != null) {
                    try {
                        job.run();
                    } catch (Exception ex) {
                        // 忽略Job执行中的Exception
                    }
                }
            }
        }

        public void shutdown() {
            running = false;
        }
    }
}
```

​	线程 池 的 本质 就是 使用 了 一个 线程 安全 的 工作 队列 连接 工作者 线程 和 客户 端线 程， 客户 端 线程 将 任务 放入 工作队 列 后 便 返回， 而 工作者 线程 则 不断 地 从 工作队 列 上 取出 工作 并 执行。

### 4.4.4 一个机遇线程池技术的简单Web服务器

```java
public class SimpleHttpServer {
    // 处理HttpRequest的线程池
    static ThreadPool<HttpRequestHandler> threadPool = new DefaultThreadPool<HttpRequestHandler>(11);
    // SimpleHttpServer的根路径
    static String                         basePath;
    static ServerSocket                   serverSocket;
    // 服务监听端口
    static int                            port       = 8080;

    public static void setPort(int port) {
        if (port > 0) {
            SimpleHttpServer.port = port;
        }
    }

    public static void setBasePath(String basePath) {
        if (basePath != null && new File(basePath).exists() && new File(basePath).isDirectory()) {
            SimpleHttpServer.basePath = basePath;
        }
    }

    // 启动SimpleHttpServer
    public static void start() throws Exception {
        serverSocket = new ServerSocket(port);
        Socket socket = null;
        while ((socket = serverSocket.accept()) != null) {
            // 接收一个客户端Socket，生成一个HttpRequestHandler，放入线程池执行
            threadPool.execute(new HttpRequestHandler(socket));
        }
        serverSocket.close();
    }

    static class HttpRequestHandler implements Runnable {

        private Socket socket;

        public HttpRequestHandler(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            String line = null;
            BufferedReader br = null;
            BufferedReader reader = null;
            PrintWriter out = null;
            InputStream in = null;
            try {
                reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                String header = reader.readLine();
                // 由相对路径计算出绝对路径
                String filePath = basePath + header.split(" ")[1];
                out = new PrintWriter(socket.getOutputStream());
                // 如果请求资源的后缀为jpg或者ico，则读取资源并输出
                if (filePath.endsWith("jpg") || filePath.endsWith("ico")) {
                    in = new FileInputStream(filePath);
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    int i = 0;
                    while ((i = in.read()) != -1) {
                        baos.write(i);
                    }

                    byte[] array = baos.toByteArray();
                    out.println("HTTP/1.1 200 OK");
                    out.println("Content-Type: image/jpeg");
                    out.println("Content-Length: " + array.length);
                    out.println("");
                    socket.getOutputStream().write(array, 0, array.length);
                } else {
                    br = new BufferedReader(new InputStreamReader(new FileInputStream(filePath)));
                    out = new PrintWriter(socket.getOutputStream());
                    out.println("HTTP/1.1 200 OK");
                    out.println("Content-Type: text/html; charset=UTF-8");
                    out.println("");
                    while ((line = br.readLine()) != null) {
                        out.println(line);
                    }
                }
                out.flush();
            } catch (Exception ex) {
                out.println("HTTP/1.1 500");
                out.println("");
                out.flush();
            } finally {
                close(br, in, reader, out, socket);
            }
        }
    }

    // 关闭流或者Socket
    private static void close(Closeable... closeables) {
        if (closeables != null) {
            for (Closeable closeable : closeables) {
                try {
                    closeable.close();
                } catch (Exception ex) {
                    // 忽略
                }
            }
        }
    }
}
```

# 第5章 Java中的锁

## 5.1 Lock接口

```java
public class LockUseCase {
    public void lock() {
        Lock lock = new ReentrantLock();
        lock.lock();
        try {
        } finally {
            lock.unlock();
        }
    }
}
```

![](https://pic.imgdb.cn/item/5facdc951cd1bbb86b4757f4.jpg)

LockAPI如表所示：

![](https://pic.imgdb.cn/item/5facdcf71cd1bbb86b477557.jpg)

## 5.2 队列同步器

//todo 不是很明白

​	队列同步器AbstractQueuedSynchronizer, 是用来构建锁或其他同步组件的框架。它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。

​	同步 器 的 主要 使用 方式 是 继承， 子类 通过 继承 同步 器 并 实现 它的 抽象 方法 来 管理 同步 状态， 在 抽象 方法 的 实现 过程中 免不了 要对 同步 状态 进行 更改， 这时 就 需要 使用 同步 器 提供 的 3 个 方法（ getState()、 setState( int newState) 和 compareAndSetState( int expect, int update)） 来 进行 操作， 因为 它们 能够 保证 状态 的 改变 是 安全 的。

### 5.2.1 队列同步器的接口与示例

​	同步器的设计是基于模板模式的。

​	重写 同步 器 指定 的 方法 时， 需要 使用 同步 器 提供 的 如下 3 个 方法 来 访问 或 修改 同步 状态。 

* getState()： 获取 当前 同步 状态。 
* setState( int newState)： 设置 当前 同步 状态。 
* compareAndSetState( int expect, int update)： 使用 CAS 设置 当前 状态， 该 方法 能够 保证 状态 设置 的 原子 性。

![](https://pic.imgdb.cn/item/5face15f1cd1bbb86b48bf2e.jpg)





![](https://pic.imgdb.cn/item/5face1711cd1bbb86b48c384.jpg)

​	同步 器 提供 的 模板 方法 基本上 分为 3 类： 独占 式 获取 与 释放 同步 状态、 共享 式 获取 与 释放 同步 状态 和 查询 同步 队列 中的 等待 线程 情况。 自定义 同步 组件 将使 用 同步 器 提供 的 模板 方法 来 实现 自己的 同步 语义。

​	顾名思义， 独占 锁 就是 在 同一 时刻 只能 有一个 线程 获取 到 锁， 而 其他 获取 锁 的 线程 只能 处于 同步 队列 中 等待， 只有 获取 锁 的 线程 释 放了 锁， 后继 的 线程 才能 够 获取 锁。

​	

```java
public class Mutex implements Lock {
    // 静态内部类，自定义同步器
    private static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -4387327721959839431L;

        // 是否处于占用状态
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // 当状态为0的时候获取锁
        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // Otherwise unused
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        // 释放锁，将状态设置为0
        protected boolean tryRelease(int releases) {
            assert releases == 1; // Otherwise unused
            if (getState() == 0)
                throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        // 返回一个Condition，每个condition都包含了一个condition队列
        Condition newCondition() {
            return new ConditionObject();
        }
    }

    // 仅需要将操作代理到Sync上即可
    private final Sync sync = new Sync();

    public void lock() {
        sync.acquire(1);
    }

    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    public void unlock() {
        sync.release(1);
    }

    public Condition newCondition() {
        return sync.newCondition();
    }

    public boolean isLocked() {
        return sync.isHeldExclusively();
    }

    public boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
```

### 5.2.2 队列同步器的实现分析

**1.同步队列**

​	![](https://pic.imgdb.cn/item/5facec7d1cd1bbb86b4bcfc2.jpg)

![](https://pic.imgdb.cn/item/5facec8c1cd1bbb86b4bd483.jpg)

**2.独占式同步状态获取与释放**

​	通过 调用 同步 器 的 acquire( int arg) 方法 可以 获取 同步 状态， 该 方法 对 中断 不 敏感， 也就是 由于 线程 获取 同步 状态 失败 后进 入 同步 队列 中， 后续 对 线程 进行 中断 操作 时， 线程 不会 从 同步 队列 中 移出。

![](https://pic.imgdb.cn/item/5facee981cd1bbb86b4c8d6a.jpg)

![](https://pic.imgdb.cn/item/5faceed11cd1bbb86b4c9e82.jpg)

在 enq( final Node node) 方法 中， 同步 器 通过“ 死 循环” 来 保证 节点 的 正确 添加， 在“ 死 循环” 中 只有 通过 CAS 将 节点 设置 成为 尾 节点 之后， 当前 线程 才能 从 该 方法 返回， 否则， 当前 线程 不断 地 尝试 设置。

![](https://pic.imgdb.cn/item/5facef931cd1bbb86b4cd4cd.jpg)

​	在 acquireQueued( final Node node, int arg) 方法 中， 当前 线程 在“ 死 循环” 中 尝试 获取 同步 状态， 而 只有 前驱 节点 是 头 节点 才能 够 尝试 获取 同步 状态，

​	第一， 头 节点 是 成功 获取 到 同步 状态 的 节点， 而 头 节点 的 线程 释放 了 同步 状态 之后， 将会 唤醒 其后 继 节点， 后继 节点 的 线程 被 唤醒 后 需要 检查 自己的 前驱 节点 是否 是 头 节点。

​	第二， 维护 同步 队列 的 FIFO 原则。 该 方法 中， 节点 自 旋 获取 同步 状态 的 行为 如图所示。

![](https://pic.imgdb.cn/item/5facf1cf1cd1bbb86b4d7336.jpg)

 acquire( int arg) 方法 调用 流程如下：

![](https://pic.imgdb.cn/item/5facf2791cd1bbb86b4da082.jpg)

release代码如下：

![](https://pic.imgdb.cn/item/5facf3031cd1bbb86b4dcde8.jpg)

​	在 获取 同步 状态 时， 同步 器 维护 一个 同步 队列， 获取 状态 失败 的 线程 都会 被 加入 到 队列 中 并在 队列 中 进行 自 旋； 移出 队列（ 或 停止 自 旋） 的 条件 是 前驱 节点 为 头 节点 且 成功 获取 了 同步 状态。 在 释放 同步 状态 时， 同步 器 调用 tryRelease( int arg) 方法 释放 同步 状态， 然后 唤醒 头 节点 的 后继 节点。

**3. 共享式同步状态获取与释放**

​	共享 式 获取 与 独占 式 获取 最主要 的 区别 在于 同一 时刻 能否 有多 个 线程 同时 获取 到 同步 状态。

![](https://pic.imgdb.cn/item/5facf43e1cd1bbb86b4e3d7e.jpg)

通过 调用 同步 器 的 acquireShared( int arg) 方法 可以 共享 式 地 获取 同步 状态。

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
}
```

​	在 acquireShared( int arg) 方法 中， 同步 器 调用 tryAcquireShared( int arg) 方法 尝试 获取 同步 状态， tryAcquireShared( int arg) 方法 返回 值 为 int 类型， 当 返回 值 大于 等于 0 时， 表示 能够 获取 到 同步 状态。 因此， 在 共享 式 获 取的 自 旋 过程中， 成功 获取 到 同步 状态 并 退 出自 旋 的 条件 就是 tryAcquireShared( int arg) 方法 返回 值 大于 等于 0。 可以 看到， 在 doAcquireShared( int arg) 方法 的 自 旋 过程中， 如果 当前 节点 的 前驱 为 头 节点 时， 尝试 获取 同步 状态， 如果 返回 值 大于 等于 0， 表示 该 次 获取 同步 状态 成功 并从 自 旋 过程中 退出。

​	releaseShared也类似。

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

**4.独占式超时获取同步状态**

​	通过 调用 同步 器 的 doAcquireNanos( int arg, long nanosTimeout) 方法 可以 超时 获取 同步 状态， 即在 指定 的 时间 段 内 获取 同步 状态， 如果 获取 到 同步 状态 则 返回 true， 否则， 返回 false。 该 方法 提供 了 传统 Java 同步 操作（ 比如 synchronized 关键字） 所不 具备 的 特性。

```java
rivate boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return true;
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

![](https://pic.imgdb.cn/item/5facf7be1cd1bbb86b4f5235.jpg)

**5. 自定义同步组件——TwinsLock**

​	只允许两个线程同时访问，超过两个线程将被阻塞。

```java
public class TwinsLock implements Lock {
    private final Sync sync = new Sync(2);

    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -7889272986162341211L;

        Sync(int count) {
            if (count <= 0) {
                throw new IllegalArgumentException("count must large than zero.");
            }
            setState(count);
        }

        public int tryAcquireShared(int reduceCount) {
            for (;;) {
                int current = getState();
                int newCount = current - reduceCount;
                if (newCount < 0 || compareAndSetState(current, newCount)) {
                    return newCount;
                }
            }
        }

        public boolean tryReleaseShared(int returnCount) {
            for (;;) {
                int current = getState();
                int newCount = current + returnCount;
                if (compareAndSetState(current, newCount)) {
                    return true;
                }
            }
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }
    }

    public void lock() {
        sync.acquireShared(1);
    }

    public void unlock() {
        sync.releaseShared(1);
    }

    public void lockInterruptibly() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    public boolean tryLock() {
        return sync.tryAcquireShared(1) >= 0;
    }

    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(time));
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

## 5.3 重入锁

​	ReentrantLock 虽然 没能 像 synchronized 关键字 一样 支持 隐式 的 重 进入， 但是 在 调用 lock() 方法 时， 已经 获取 到 锁 的 线程， 能够 再次 调用 lock() 方法 获取 锁 而 不被 阻塞。

​	事实上， 公平 的 锁 机制 往往 没有 非 公平 的 效率高， 但是， 并不是 任何 场景 都是 以 TPS 作为 唯一 的 指标， 公平 锁 能够 减少“ 饥饿” 发生 的 概率， 等待 越 久 的 请求 越是 能够 得到 优先 满足。

**1. 实现重进入**

​	需要解决如下问题：

1） 线程再次获取锁。

2） 锁的最终释放。

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

**2. 公平与非公平获取锁的区别**

​	公平 性 与否 是 针对 获取 锁 而言 的， 如果 一个 锁 是 公平 的， 那么 锁 的 获取 顺序 就应 该 符合 请求 的 绝对 时间 顺序， 也就是 FIFO。

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

## 5.4 读写锁

​	之前提到的都是排他锁。读写锁可以在同一时刻允许多个读线程访问。读写锁维护了一对锁，一个读锁，一个写锁，通过分离读锁和写锁，使得并发性相比一般的排他锁有了很大提升。

​	Java并发包提供读写锁的实现是ReentrantReadWriteLock

![](https://pic.imgdb.cn/item/5fad04291cd1bbb86b531ea5.jpg)

### 5.4.1 读写锁的接口与示例

![](https://pic.imgdb.cn/item/5fad04421cd1bbb86b532627.jpg)

​	

```java
public class Cache {
    private static final Map<String, Object>    map = new HashMap<String, Object>();
    private static final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    private static final Lock                   r   = rwl.readLock();
    private static final Lock                   w   = rwl.writeLock();

    public static final Object get(String key) {
        r.lock();
        try {
            return map.get(key);
        } finally {
            r.unlock();
        }
    }

    public static final Object put(String key, Object value) {
        w.lock();
        try {
            return map.put(key, value);
        } finally {
            w.unlock();
        }
    }

    public static final void clear() {
        w.lock();
        try {
            map.clear();
        } finally {
            w.unlock();
        }
    }
}
```

​	需要 获取 读 锁， 这使 得 并发 访问 该 方法 时不 会被 阻塞。 写 操作 put( String key, Object value) 方法 和 clear() 方法， 在 更新 HashMap 时必 须 提前 获取 写 锁， 当 获取 写 锁 后， 其他 线程 对于 读 锁 和 写 锁 的 获取 均被 阻塞， 而 只 有写 锁 被 释放 之后， 其他 读写 操作 才能 继续。

### 5.4.2 读写锁的实现分析

**1. 读写状态的设计**

​	读写锁将变量切分成了两个部分，高16位表示读，低16位表示写。

![](https://pic.imgdb.cn/item/5fad073b1cd1bbb86b53f4d8.jpg)

**2. 写锁的获取与释放**

```java
protected final boolean tryAcquire(int acquires) {
    /*
     * Walkthrough:
     * 1. If read count nonzero or write count nonzero
     *    and owner is a different thread, fail.
     * 2. If count would saturate, fail. (This can only
     *    happen if count is already nonzero.)
     * 3. Otherwise, this thread is eligible for lock if
     *    it is either a reentrant acquire or
     *    queue policy allows it. If so, update state
     *    and set owner.
     */
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

**3. 读锁的获取与释放**

```java
protected final int tryAcquireShared(int unused) {
            /*
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
            Thread current = Thread.currentThread();
            int c = getState();
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            int r = sharedCount(c);
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    firstReaderHoldCount++;
                } else {
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            return fullTryAcquireShared(current);
        }
```

​	在 tryAcquireShared( int unused) 方法 中， 如果 其他 线程 已经 获取 了 写 锁， 则 当前 线程 获取 读 锁 失败， 进入 等待 状态。 如果 当前 线程 获取 了 写 锁 或者 写 锁 未被 获取， 则 当前 线程（ 线程 安全， 依靠 CAS 保证） 增加 读 状态， 成功 获取 读 锁。 读 锁 的 每次 释放（ 线程 安全 的， 可能有 多个 读 线程 同时 释放 读 锁） 均 减少 读 状态， 减 少的 值 是（ 1<< 16）。

**4. 锁降级**

​	锁降级指的是写锁降级为读锁。如果 当前 线程 拥 有写 锁， 然后 将其 释放， 最后 再 获取 读 锁， 这种 分段 完成 的 过程 不能 称之为 锁 降级。 锁 降级 是指 把持 住（ 当前 拥有 的） 写 锁， 再 获取 到 读 锁， 随后 释放（ 先前 拥有 的） 写 锁 的 过程。

​	RentrantReadWriteLock 不支持 锁 升级（ 把持 读 锁、 获取 写 锁， 最后 释放 读 锁 的 过程）。

## 5.5 LockSupport工具

​	![](https://pic.imgdb.cn/item/5fad0fbb1cd1bbb86b565de9.jpg)

## 5.6 Condition接口

![](https://pic.imgdb.cn/item/5fad10b91cd1bbb86b56b653.jpg)

### 5.6.1 Condition接口与示例

```java
public class ConditionUseCase {
    Lock      lock      = new ReentrantLock();
    Condition condition = lock.newCondition();

    public void conditionWait() throws InterruptedException {
        lock.lock();
        try {
            condition.await();
        } finally {
            lock.unlock();
        }
    }

    public void conditionSignal() throws InterruptedException {
        lock.lock();
        try {
            condition.signal();
        } finally {
            lock.unlock();
        }
    }
}
```

![](https://pic.imgdb.cn/item/5fad12241cd1bbb86b5712cf.jpg)

通过BoundedQueue来深入了解Condition的使用方式

```java
public class BoundedQueue<T> {
    private Object[]  items;
    // 添加的下标，删除的下标和数组当前数量
    private int       addIndex, removeIndex, count;
    private Lock      lock     = new ReentrantLock();
    private Condition notEmpty = lock.newCondition();
    private Condition notFull  = lock.newCondition();

    public BoundedQueue(int size) {
        items = new Object[size];
    }

    // 添加一个元素，如果数组满，则添加线程进入等待状态，直到有“空位”
    public void add(T t) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length)
                notFull.await();
            items[addIndex] = t;
            if (++addIndex == items.length)
                addIndex = 0;
            ++count;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    // 由头部删除一个元素，如果数组空，则删除线程进入等待状态，直到有新添加元素
    @SuppressWarnings("unchecked")
    public T remove() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await();
            Object x = items[removeIndex];
            if (++removeIndex == items.length)
                removeIndex = 0;
            --count;
            notFull.signal();
            return (T) x;
        } finally {
            lock.unlock();
        }
    }
}
```

### 5.6.2 Condition的实现分析

ConditionObject 是 同步 器 AbstractQueuedSynchronizer 的 内 部类， 因为 Condition 的 操作 需要 获取 相 关联 的 锁， 所以 作为 同步 器 的 内 部类 也 较为 合理。

**1. 等待队列**

​	等待 队列 是 一个 FIFO 的 队列， 在 队列 中的 每个 节点 都 包含 了 一个 线程 引用， 该 线程 就是 在 Condition 对象 上 等待 的 线程， 如果 一个 线程 调用 了 Condition. await() 方法， 那么 该 线程 将会 释放 锁、 构造 成 节点 加入 等待 队列 并进 入 等待 状态。

![](https://pic.imgdb.cn/item/5fad14301cd1bbb86b57aa66.jpg)

![](https://pic.imgdb.cn/item/5fad14591cd1bbb86b57b2fe.jpg)

**2. 等待**

​	调用 该 方法 的 线程 成功 获取 了 锁 的 线程， 也就是 同步 队列 中的 首 节点， 该 方法 会 将 当前 线程 构造 成 节点 并 加入 等待 队列 中， 然后 释放 同步 状态， 唤醒 同步 队列 中的 后继 节点， 然后 当前 线程 会 进入 等待 状态。

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

**3. 通知**

![](https://pic.imgdb.cn/item/5fad15801cd1bbb86b57f3bb.jpg)

```java
public final void signal() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```

Condition 的 signalAll() 方法， 相当于 对等 待 队列 中的 每个 节点 均 执行 一次 signal() 方法， 效果 就是 将 等待 队列 中 所有 节点 全部 移动 到 同步 队列 中， 并 唤醒 每个 节点 的 线程。

# 第6章 Java并发容器和框架

## 6.1 ConcurrentHashMap的实现原理与使用

### 6.1.1 为什么要使用ConcurrentHashMap

(1) 线程不安全的hashMap

​	在JDK1.7中，hashmap在并发情况下容易引起死循环。

(2) 效率低下的HashTable

(3) ConcurrentHashMap的锁分段技术(jdk1.7)可有效提升并发访问率

### 6.1.2 ConcurrentHashMap的结构

![](https://pic.imgdb.cn/item/5fadd3e41cd1bbb86b7c47a9.jpg)

### 6.1.3 ConcurrentHashMap的初始化

​	初始化方法是通过`initialCapacity`,`loadFactor`和`concurrencyLevel`几个参数来初始化segement数组、段偏移量segmentShift、段掩码segmentMask和每个segment的HashEntry数组。

**1. 初始化segments数组**

![](https://pic.imgdb.cn/item/5fadd5471cd1bbb86b7c7a59.jpg)

​	由上 面的 代码 可知， segments 数组 的 长度 ssize 是 通过 concurrencyLevel 计算 得出 的。 为了 能 通过 按 位 与 的 散 列 算法 来 定位 segments 数组 的 索引， 必须 保证 segments 数组 的 长度 是 2 的 N 次方（ power- of- two size）， 所以 必须 计算 出 一个 大于 或 等于 concurrencyLevel 的 最小 的 2 的 N 次方 值 来作 为 segments 数组 的 长度。 假如 concurrencyLevel 等于 14、 15 或 16， ssize 都会 等于 16， 即 容器 里 锁 的 个数 也是 16。

**2. 初始化segmentShift和segmentMask**


​	这 两个 全局 变量 需要 在 定位 segment 时 的 散 列 算法 里 使用， sshift 等于 ssize 从 1 向左 移位 的 次数， 在 默认 情况下 concurrencyLevel 等于 16， 1 需要 向左 移位 移动 4 次， 所以 sshift 等于 4。4。 segmentShift 用于 定位 参与 散 列 运算 的位数， segmentShift 等于 32 减 sshift， 所以 等于 28， 这里 之所 以用 32 是因为 ConcurrentHashMap 里 的 hash() 方法 输出 的 最大数 是 32 位 的， 后面 的 测试 中 我们 可以 看到 这点。 segmentMask 是 散 列 运算 的 掩 码， 等于 ssize 减 1， 即 15， 掩 码 的 二进制 各 个位 的 值 都是 1。 因为 ssize 的 最大 长度 是 65536， 所以 segmentShift 最大值 是 16， segmentMask 最大值 是 65535， 对应 的 二进制 是 16 位， 每个 位 都是 1。

**3. 初始化每个segment**

![](https://pic.imgdb.cn/item/5fadd6ca1cd1bbb86b7cb21c.jpg)

### 6.1.4 定位Segment

​	ConcurrentHashMap 会首 先使 用 Wang/ Jenkins hash 的 变种 算法 对 元素 的 hashCode 进行 一次 再 散 列。

![](https://pic.imgdb.cn/item/5fadd74a1cd1bbb86b7cc7f8.jpg)

​	通过以下散列算法定位segment

![](https://pic.imgdb.cn/item/5fadd8351cd1bbb86b7cead8.jpg)

### 6.1.5 ConcurrentHashMap的操作

**1. get操作**

![](https://pic.imgdb.cn/item/5fadd8ab1cd1bbb86b7cfd21.jpg)

​	效率很高，不需要加锁，除非读到的值为空。 get 方法 里 将要 使用 的 共享 变量 都 定义 成 volatile 类型， 如用 于 统计 当前 Segement 大小 的 count 字段 和 用于 存储 值 的 HashEntry 的 value。 定义 成 volatile 的 变量， 能够 在 线程 之间 保持 可见 性， 能够 被 多 线程 同时 读， 并且 保证 不会 读到 过期 的 值， 但是 只能 被单 线程 写（ 有 一种 情况 可以 被 多 线程 写， 就是 写入 的 值 不 依赖于 原值）， 在 get 操作 里 只 需要 读 不需要 写 共享 变量 count 和 value， 所以 可以 不用 加锁。

**2. put操作**

(1) 是否需要扩容

​	超过阈值就进行扩容。

(2) 如何扩容

​	在 扩容 的 时候， 首先 会 创建 一个 容量 是 原来 容量 两倍 的 数组， 然后 将 原 数组 里 的 元素 进行 再 散 列 后 插入 到 新的 数组 里。 为了 高效， ConcurrentHashMap 不会 对 整个 容器 进行 扩容， 而 只对 某个 segment 进行 扩容。

**3. size操作**

​	最 安全 的 做法 是在 统计 size 的 时候 把 所有 Segment 的 put、 remove 和 clean 方法 全部 锁住， 但是 这种 做法 显然 非常 低效。

​	因为 在 累加 count 操作 过程中， 之前 累加 过 的 count 发生 变化 的 几率 非常 小， 所以 ConcurrentHashMap 的 做法 是 先 尝试 2 次 通过 不 锁住 Segment 的 方式 来 统计 各个 Segment 大小， 如果 统计 的 过程中， 容器 的 count 发生了 变化， 则 再 采用 加锁 的 方式 来 统计 所有 Segment 的 大小。

https://www.jianshu.com/p/c0642afe03e0

## 6.2 ConcurrentLinkedQueue

​	实现一个线程安全的队列有两种方式：

* 阻塞算法：可以用出队入队都一个锁或者单独的两个锁来实现。

* 非阻塞算法：使用循环CAS的方式来实现。

  

​	ConcurrentLinkedQueue 是一 个 基于 链接 节点 的 无 界线 程 安全 队列， 它 采用 先进 先出 的 规则 对 节点 进行 排序， 当 我们 添加 一个 元素 的 时候， 它 会 添加 到 队列 的 尾部； 当 我们 获取 一个 元素 时， 它 会 返回 队列 头部 的 元素。 它 采用 了“ wait- free” 算法（ 即 CAS 算法） 来 实现， 该 算法 在 Michael& Scott 算法 上进 行了 一些 修改。

### 6.2.1 ConcurrentLinkedQueue的结构

![](https://pic.imgdb.cn/item/5fadec351cd1bbb86b80a56b.jpg)

​	有head节点和tail节点组成。默认情况head存储的元素为空，tail节点等于head节点。

![](https://pic.imgdb.cn/item/5fadee271cd1bbb86b810924.jpg)

### 6.2.2 入队列

**1. 入队列的过程**

​	入队 列 就是 将 入队 节点 添加 到 队列 的 尾部。

![](https://pic.imgdb.cn/item/5fadef7f1cd1bbb86b814905.jpg)

![](https://pic.imgdb.cn/item/5fadf03d1cd1bbb86b8170aa.jpg)

**2. 定位尾结点**

​	tail节点并不是总是尾结点，所以每次入队都必须通过tail来找到尾结点。获取 tail 节点 的 next 节点 需要 注意 的 是 p 节点 等于 p 的 next 节点 的 情况， 只有 一种 可能 就是 p 节点 和 p 的 next 节点 都 等于 空， 表示 这个 队列 刚 初始化， 正 准备 添加 节点， 所以 需要 返回 head 节点。 获取 p 节点 的 next 节点 代码 如下。

​	

**3. 设置入队节点为尾节点**

​	p. casNext（ null， n） 方法 用于 将 入队 节点 设置 为 当前 队列 尾 节点 的 next 节点， 如果 p 是 null， 表示 p 是 当前 队列 的 尾 节点， 如果 不为 null， 表示 有其 他 线程 更新 了 尾 节点， 则需 要 重新 获取 当前 队列 的 尾 节点。

**4. HOPS的设计意图**

​	doug lea的将入队节点设置为尾节点的代码还是有点复杂。如下方式也是否可行？

![](https://pic.imgdb.cn/item/5fae0c3a1cd1bbb86b876ded.jpg)

​	但是这么做有个缺点，每次都需要使用CAS更新tail节点。如果能减少CAS更新tail的次数，就能提高入队的效率。所以使用hops变量来控制并减少tail节点的更新频率，并不是每次节点入队都将tail节点更新成尾节点，而是当tail节点和尾节点的距离大于等于常量HOPS的值(默认为1)才更新tail节点。

### 6.2.3 出队列

​	![](https://pic.imgdb.cn/item/5fae10a61cd1bbb86b88642b.jpg)

```java
public E poll() {
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            E item = p.item;

            if (item != null && p.casItem(item, null)) {
                // Successful CAS is the linearization point
                // for item to be removed from this queue.
                if (p != h) // hop two nodes at a time
                    updateHead(h, ((q = p.next) != null) ? q : p);
                return item;
            }
            else if ((q = p.next) == null) {
                updateHead(h, p);
                return null;
            }
            else if (p == q)
                continue restartFromHead;
            else
                p = q;
        }
    }
}
```

​	首先 获取 头 节点 的 元素， 然后 判断 头 节点 元素 是否 为 空， 如 果为 空， 表示 另外 一个 线程 已经 进行 了 一次 出 队 操作 将 该 节点 的 元素 取走， 如果不 为 空， 则 使用 CAS 的 方式 将 头 节点 的 引用 设置 成 null， 如果 CAS 成功， 则 直接 返回 头 节点 的 元素， 如果 不成功， 表示 另外 一个 线程 已经 进行 了 一次 出 队 操作 更新 了 head 节点， 导致 元素 发生了 变化， 需要 重新 获取 头 节点。

## 6.3 Java中的阻塞队列

### 6.3.1 什么是阻塞队列

​	支持两个附加操作的队列：

1) 支持阻塞的插入方法：意思是当队列满时，会阻塞插入元素的线程，直到队列不满。

2) 支持阻塞的移除方法：意思是在队列为空时，获取元素的线程会等待队列变为非空。

![](https://pic.imgdb.cn/item/5fae12c91cd1bbb86b88cd37.jpg)

### 6.3.2 Java的阻塞队列

java7提供了7个阻塞队列：

* ArrayBlockingQueue： 一个 由 数组 结构 组成 的 有 界 阻塞 队列。 
* LinkedBlockingQueue： 一个 由 链 表 结构 组成 的 有 界 阻塞 队列。 
* PriorityBlockingQueue： 一个 支持 优先级 排序 的 无 界 阻塞 队列。 
* DelayQueue： 一个 使用 优先级 队列 实现 的 无 界 阻塞 队列。 
* SynchronousQueue： 一个 不 存储 元素 的 阻塞 队列。 
* LinkedTransferQueue： 一个 由 链 表 结构 组成 的 无 界 阻塞 队列。 
* LinkedBlockingDeque： 一个 由 链 表 结构 组成 的 双向 阻塞 队列。

**1. ArrayBlockingQueue**

​	默认 情况下 不保 证 线程 公平 的 访问 队列， 所谓 公平 访问 队列 是指 阻塞 的 线程， 可以 按照 阻塞 的 先后 顺序 访问 队列， 即 先 阻塞 线程 先 访问 队列。 非 公平 性 是对 先 等待 的 线程 是非 公平 的， 当 队列 可用 时， 阻塞 的 线程 都可以 争夺 访问 队列 的 资格， 有可能 先 阻塞 的 线程 最后 才 访问 队列。 为了 保证 公平 性， 通常 会 降低 吞吐量。 我们 可以 使用 以下 代码 创建 一个 公平 的 阻塞 队列。

![](https://pic.imgdb.cn/item/5fae13ec1cd1bbb86b8904e0.jpg)

**2. LinkedBlockingQueue**

​	此 队列 的 默认 和 最大 长度 为 Integer. MAX_ VALUE。 此 队列 按照 先进 先出 的 原则 对 元素 进行 排序。

**3. PriorityBlockingQueue**

​	默认 情况下 元素 采取 自然 顺序 升序 排列。 也可以 自定义 类 实现 compareTo() 方法 来 指定 元素 排序 规则， 或者 初始化 PriorityBlockingQueue 时， 指定 构造 参数 Comparator 来 对 元素 进行 排序。 需要 注意 的 是 不能 保证 同 优先级 元素 的 顺序。

**4. DelayQueue**

​	DelayQueue 是一 个 支持 延时 获取 元素 的 无 界 阻塞 队列。 队列 使用 PriorityQueue 来 实现。 队列 中的 元素 必须 实现 Delayed 接口， 在 创建 元素 时 可以 指定 多久 才能 从 队列 中 获取 当前 元素。 只有 在 延迟 期满 时 才能 从 队列 中 提取 元素。

* 缓存系统的设计：能获取到就代表缓存有效期过了。
* 定时任务调度。

(1) 如何实现Delayed接口

​	有3步

​	第一步： 在 对象 创建 的 时候， 初始化 基本 数据。 使用 time 记录 当前 对象 延迟 到 什么时候 可以 使用， 使用 sequenceNumber 来 标识 元素 在 队列 中的 先后 顺序。

![](https://pic.imgdb.cn/item/5fae22d71cd1bbb86b8c443f.jpg)

​	第二步：实现getDelay方法，该方法返回当前元素还需要延时多长时间，单位是纳秒。

![](https://pic.imgdb.cn/item/5fae22d71cd1bbb86b8c443f.jpg)

​	第三步：实现compareTo方法来指定元素顺序。

(2) 如何实现延时阻塞队列

​	延时 阻塞 队列 的 实现 很 简单， 当 消费者 从 队列 里 获取 元素 时， 如果 元素 没有 达到 延时 时间， 就 阻塞 当前 线程。

![](https://pic.imgdb.cn/item/5fae24bc1cd1bbb86b8cc4c5.jpg)

**5. SynchronousQueue **

​	它是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加。

​	它 支持 公平 访问 队列。 默认 情况下 线程 采用 非 公平 性 策略 访问 队列。 使用 以下 构造 方法 可以 创建 公平 性 访问 的 SynchronousQueue， 如果 设置 为 true， 则 等待 的 线程 会 采用 先进 先出 的 顺序 访问 队列。

![](https://pic.imgdb.cn/item/5fae25031cd1bbb86b8cd20d.jpg)

**6. LinkedTransferQueue**

​	LinkedTransferQueue 是 一个 由 链 表 结构 组成 的 无 界 阻塞 TransferQueue 队列。 相对于 其他 阻塞 队列， LinkedTransferQueue 多了 tryTransfer 和 transfer 方法。

(1) transfer方法

​	如果 当前 有 消费者 正在 等待 接收 元素（ 消费者 使用 take() 方法 或 带 时间 限制 的 poll() 方法 时）， transfer 方法 可以 把 生产者 传入 的 元素 立刻 transfer（ 传输） 给 消费者。 如果 没有 消费者 在等 待 接收 元素， transfer 方法 会 将 元素 存放 在 队列 的 tail 节点， 并 等到 该 元素 被 消费者 消费 了 才 返回。

(2) tryTransfer方法

​	tryTransfer 方法 是 用来 试探 生产者 传入 的 元素 是否 能 直接 传给 消费者。 如果 没有 消费者 等待 接收 元素， 则 返回 false。 和 transfer 方法 的 区别 是 tryTransfer 方法 无论 消费者 是否 接收， 方法 立即 返回， 而 transfer 方法 是 必须 等到 消费者 消费 了 才 返回。

​	对于 带有 时间 限制 的 tryTransfer（ E e， long timeout， TimeUnit unit） 方法， 试图 把 生产者 传入 的 元素 直接 传给 消费者， 但是 如果 没有 消费者 消费 该 元素 则 等待 指定 的 时间 再 返回， 如果 超时 还没 消费 元素， 则 返回 false， 如果 在 超时 时间 内 消费 了 元素， 则 返回 true。

**7. LinkedBlockingDeque**

​	LinkedBlockingDeque 是 一个 由 链 表 结构 组成 的 双向 阻塞 队列。 所谓 双向 队列 指的 是 可以 从 队列 的 两端 插入 和 移出 元素。 双向 队列 因为 多了 一个 操作 队列 的 入口， 在 多 线程 同时 入队 时， 也就 减少 了 一半 的 竞争。 相比 其他 的 阻塞 队列， LinkedBlockingDeque 多了 addFirst、 addLast、 offerFirst、 offerLast、 peekFirst 和 peekLast 等 方法， 以 First 单词 结尾 的 方法， 表示 插入、 获取（ peek） 或 移 除 双端 队列 的 第一个 元素。 以 Last 单词 结尾 的 方法，表示 插入、 获取 或 移 除 双端 队列 的 最后 一个 元素。 另外， 插入 方法 add 等 同于 addLast， 移 除 方法 remove 等效 于 removeFirst。 但是 take 方法 却 等 同于 takeFirst， 不知道 是不是 JDK 的 bug， 使用 时 还是 用带 有 First 和 Last 后缀 的 方法 更 清楚。 

​	在 初始化 LinkedBlockingDeque 时 可以 设置 容量 防止 其 过度 膨胀。 另外， 双向 阻塞 队列 可以 运用 在“ 工作 窃取” 模式 中。

### 6.3.3 阻塞队列的实现原理

**使用通知模式实现**

​	所谓 通知 模式， 就是 当 生产者 往 满的 队列 里 添加 元素 时会 阻塞 住 生产者， 当 消费者 消费 了 一个 队列 中的 元素 后， 会 通知 生产者 当前 队列 可用。 通过 查看 JDK 源 码 发现 ArrayBlockingQueue 使用 了 Condition 来 实现，

```java
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
 public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

```

其他略

## 6.4 Fork/Join框架

### 6.4.1 什么是Fork/Join框架

​	 Fork 就是 把 一个 大 任务 切分 为 若干 子 任务 并行 的 执行， Join 就是 合并 这些 子 任务 的 执行 结果， 最后 得到 这个 大 任务 的 结果。

![](https://pic.imgdb.cn/item/5fae31921cd1bbb86b9170e8.jpg)

### 6.4.2 工作窃取算法

​	在 这时 它们 会 访问 同一个 队列， 所以 为了 减少 窃取 任务 线程 和 被窃 取 任务 线程 之间 的 竞争， 通常 会使 用 双端 队列， 被窃 取 任务 线程 永远 从 双端 队列 的 头部 拿 任务 执行， 而 窃取 任务 的 线程 永远 从 双端 队列 的 尾部 拿 任务 执行。

![](https://pic.imgdb.cn/item/5fae39e01cd1bbb86b939135.jpg)

​	工作 窃取 算法 的 优点： 充分 利用 线程 进行 并行 计算， 减少 了 线程 间的 竞争。 

​	工作 窃取 算法 的 缺点： 在 某些 情况下 还是 存在 竞争， 比如 双端 队列 里 只有 一个 任务 时。 并且 该 算法 会 消耗 了 更多 的 系统 资源， 比如 创建 多个 线程 和 多个 双端 队列。

### 6.4.3 Fork/Join框架的设计

1. 分割任务。
2. 执行任务合并结果。



​	Fork/Join使用两个类来完成以上两件事情。

* ForkJoinTask: 我们 要 使用 ForkJoin 框架， 必须 首先 创建 一个 ForkJoin 任务。 它 提供 在 任务 中 执行 fork() 和 join() 操作 的 机制。 通常 情况下， 我们 不需要 直接 继承 ForkJoinTask 类， 只需 要 继承 它的 子类， Fork/ Join 框架 提供 了 以下 两个 子类。
  * RecursiveAction： 用于 没有 返回 结果 的 任务。 
  * RecursiveTask： 用于 有 返回 结果 的 任务。
* ForkJoinPool: ForkJoinTask需要通过ForkJoinPool。

### 6.4.4 使用Fork/Join框架

 

```java
/**
 * 计数器任务
 * 
 * @author tengfei.fangtf
 * @version $Id: CountTask.java, v 0.1 2015-8-1 上午12:00:29 tengfei.fangtf Exp $
 */
public class CountTask extends RecursiveTask<Integer> {

    private static final int THRESHOLD = 2; // 阈值
    private int              start;
    private int              end;

    public CountTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;

        // 如果任务足够小就计算任务
        boolean canCompute = (end - start) <= THRESHOLD;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务计算
            int middle = (start + end) / 2;
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle + 1, end);
            //执行子任务
            leftTask.fork();
            rightTask.fork();
            //等待子任务执行完，并得到其结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();
            //合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        // 生成一个计算任务，负责计算1+2+3+4
        CountTask task = new CountTask(1, 4);
        // 执行一个任务
        Future<Integer> result = forkJoinPool.submit(task);
        try {
            System.out.println(result.get());
        } catch (InterruptedException e) {
        } catch (ExecutionException e) {
        }
    }

}
```

### 6.4.5 Fork/Join框架的异常处理

​	ForkJoinTask 提供 了 isCompletedAbnormally() 方法 来 检查 任务 是否 已经 抛出 异常 或 已经 被 取消 了， 并且 可以 通过 ForkJoinTask 的 getException 方法 获取 异常。

![](https://pic.imgdb.cn/item/5fae3cc01cd1bbb86b942c4a.jpg)

### 6.4.6 Fork/Join框架的实现原理

​	ForkJoinPool 由 ForkJoinTask 数组 和 ForkJoinWorkerThread 数组 组成， ForkJoinTask 数组 负责 将 存放 程序 提 交给 ForkJoinPool 的 任务， 而 ForkJoinWorkerThread 数组 负责 执行 这些 任务。

(1) ForkJoinTask的fork方法实现原理

​	![](https://pic.imgdb.cn/item/5fae3e841cd1bbb86b94a739.jpg)

![](https://pic.imgdb.cn/item/5fae3e9a1cd1bbb86b94abbc.jpg)

​	pushTask 方法 把 当前任务 存放 在 ForkJoinTask 数组 队列 里。 然后 再 调用 ForkJoinPool 的 signalWork() 方法 唤醒 或 创建 一个 工作 线程 来 执行任务。

(2) ForkJoinTask的join方法实现原理

​	![](https://pic.imgdb.cn/item/5fae3f3c1cd1bbb86b94cb8c.jpg)

首先， 它 调用 了 doJoin() 方法， 通过 doJoin() 方法 得到 当前任务 的 状态 来 判断 返回 什么 结果， 任务 状态 有 4 种： 已完成（ NORMAL）、 被 取消（ CANCELLED）、 信号（ SIGNAL） 和 出现 异常（ EXCEPTIONAL）。 

* 如果 任务 状态 是 已完成， 则 直接 返回 任务 结果。 
* 如果 任务 状态 是 被 取消， 则 直接 抛出 CancellationException。 
* 如果 任务 状态 是 抛出 异常， 则 直接 抛出 对应 的 异常。

```java
private int doJoin() {
    int s; Thread t; ForkJoinWorkerThread wt; ForkJoinPool.WorkQueue w;
    return (s = status) < 0 ? s :
        ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
        (w = (wt = (ForkJoinWorkerThread)t).workQueue).
        tryUnpush(this) && (s = doExec()) < 0 ? s :
        wt.pool.awaitJoin(w, this, 0L) :
        externalAwaitDone();
}
```

​	在 doJoin() 方法 里， 首先 通过 查看 任务 的 状态， 看 任务 是否 已经 执行 完成， 如果 执行 完成， 则 直接 返回 任务 状态； 如果 没有 执行 完， 则从 任务 数组 里 取出 任务 并 执行。 如果 任务 顺利 执行 完成， 则 设置 任务 状态 为 NORMAL， 如果 出现 异常， 则 记录 异常， 并将 任务 状态 设置 为 EXCEPTIONAL。

# 第7章 Java的13个原子操作类

## 7.1 原子更新基本类

使用 原子 的 方式 更新 基本 类型， Atomic 包 提供 了 以下 3 个 类。 

* AtomicBoolean： 原子 更新 布尔 类型。 
* AtomicInteger： 原子 更新 整型。 
* AtomicLong： 原子 更新 长整型。

常用方法如下：

![](https://pic.imgdb.cn/item/5fae5f421cd1bbb86b9d1991.jpg)

http://ifeve.com/how-does-atomiclong-lazyset-work/。

```java
public class AtomicIntegerTest {

    static AtomicInteger ai = new AtomicInteger(1);

    public static void main(String[] args) {
        System.out.println(ai.getAndIncrement());
        System.out.println(ai.get());
    }

}
```

通过 代码， 我们 发现 Unsafe 只 提供 了 3 种 CAS 方法： compareAndSwapObject、 compare- AndSwapInt 和 compareAndSwapLong， 再看 AtomicBoolean 源 码， 发现 它是 先把 Boolean 转换 成 整型， 再 使用 compareAndSwapInt 进行 CAS， 所以 原子 更新 char、 float 和 double 变量 也可 以用 类似 的 思路 来 实现。

## 7.2 原子更新数组

​	通过 原子 的 方式 更新 数组 里 的 某个 元素， Atomic 包 提供 了 以下 4 个 类。

* AtomicIntegerArray： 原子 更新 整型 数组 里 的 元素。 
* AtomicLongArray： 原子 更新 长整型 数组 里 的 元素。 
* AtomicReferenceArray： 原子 更新 引用 类型 数组 里 的 元素。 
* AtomicIntegerArray 类 主要 是 提供 原子 的 方式 更新 数组 里 的 整型， 其 常用 方法 如下。 
  * int addAndGet（ int i， int delta）： 以 原子 方式 将 输入 值 与 数组 中 索引 i 的 元素 相加。 
  * boolean compareAndSet（ int i， int expect， int update）： 如果 当前 值 等于 预期 值， 则以 原子 方式 将 数组 位置 i 的 元素 设置 成 update 值。

```java
public class AtomicIntegerArrayTest {

    static int[]              value = new int[] { 1, 2 };

    static AtomicIntegerArray ai    = new AtomicIntegerArray(value);

    public static void main(String[] args) {
        ai.getAndSet(0, 3);
        System.out.println(ai.get(0));
        System.out.println(value[0]);
    }
	
}

/**
3
1
**/
```

​	需要 注意 的 是， 数组 value 通过 构造 方法 传递 进去， 然后 AtomicIntegerArray 会 将 当前 数组 复制 一份， 所 以当 AtomicIntegerArray 对 内部 的 数组 元素 进行 修改 时， 不会 影响 传入 的 数组。

## 7.3 原子更新引用类型

* AtomicReference： 原子 更新 引用 类型。 
* AtomicReferenceFieldUpdater： 原子 更新 引用 类型 里 的 字段。 
* AtomicMarkableReference： 原子 更新 带有 标记 位 的 引用 类型。 可以 原子 更新 一个 布尔 类型 的 标记 位 和 引用 类型。 构造 方法 是 AtomicMarkableReference（ V initialRef， boolean initialMark）。

```java
public class AtomicReferenceTest {

    public static AtomicReference<User> atomicUserRef = new AtomicReference<User>();

    public static void main(String[] args) {
        User user = new User("conan", 15);
        atomicUserRef.set(user);
        User updateUser = new User("Shinichi", 17);
        atomicUserRef.compareAndSet(user, updateUser);
        System.out.println(atomicUserRef.get().getName());
        System.out.println(atomicUserRef.get().getOld());
    }

    public static class User {
        private String name;
        private int    old;

        public User(String name, int old) {
            this.name = name;
            this.old = old;
        }

        public String getName() {
            return name;
        }

        public int getOld() {
            return old;
        }
    }
}
```

## 7.4 原子更新字段类

* AtomicIntegerFieldUpdater： 原子 更新 整型 的 字段 的 更新 器。 
* AtomicLongFieldUpdater： 原子 更新 长整型 字段 的 更新 器。 
* AtomicStampedReference： 原子 更新 带有 版本 号的 引用 类型。 该类 将 整 数值 与 引用 关联 起来， 可用 于 原子 的 更新 数据 和 数据 的 版 本号， 可以 解决 使用 CAS 进行 原子 更新 时 可能 出现 的 ABA 问题。

```java
public class AtomicIntegerFieldUpdaterTest {

    private static AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.newUpdater(User.class, "old");

    public static void main(String[] args) {
        User conan = new User("conan", 10);
        System.out.println(a.getAndIncrement(conan));
        System.out.println(a.get(conan));
    }

    public static class User {
        private String      name;
        public volatile int old;

        public User(String name, int old) {
            this.name = name;
            this.old = old;
        }

        public String getName() {
            return name;
        }

        public int getOld() {
            return old;
        }
    }
}
/**
10
11
**/
```

# 第8章 Java中的并发工具类

## 8.1 等待多线程完成的CountDownLatch

​	允许一个或多个线程等待其他线程完成操作。

​	解析多个Excel里的多个Sheet数据。最简单的做法是使用join()。

```java
public class JoinCountDownLatchTest {

    public static void main(String[] args) throws InterruptedException {
        Thread parser1 = new Thread(new Runnable() {
            @Override
            public void run() {
            }
        });

        Thread parser2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("parser2 finish");
            }
        });

        parser1.start();
        parser2.start();
        parser1.join();
        parser2.join();
        System.out.println("all parser finish");
    }

}
```

​	使用CounDownLatch也可以实现join的功能，并且功能更多。

```java
public class CountDownLatchTest {

    static CountDownLatch c = new CountDownLatch(2);

    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(1);
                c.countDown();
                System.out.println(2);
                c.countDown();
            }
        }).start();

        c.await();
        System.out.println("3");
    }

}
```

## 8.2 同步屏障CyclicBarrier

​	让一组线程达到一个屏障时同时被阻塞，直到最后一个线程到达屏障时，屏障才会开门。

### 8.3.1 CyclicBarrier简介

```java
public class CyclicBarrierTest {

    static CyclicBarrier c = new CyclicBarrier(2);

    public static void main(String[] args) {
        new Thread(new Runnable() {

            @Override
            public void run() {
                try {
                    c.await();
                } catch (Exception e) {

                }
                System.out.println(1);
            }
        }).start();

        try {
            c.await();
        } catch (Exception e) {

        }
        System.out.println(2);
    }
}
```

还提供一个更高级的构造参数`CyclicBarrier(int parties, Runnable barrierAction)`, 用于在线程到达屏障时，优先执行barrierAction。

```java
public class CyclicBarrierTest2 {

    static CyclicBarrier c = new CyclicBarrier(2, new A());

    public static void main(String[] args) {
        new Thread(new Runnable() {

            @Override
            public void run() {
                try {
                    c.await();
                } catch (Exception e) {

                }
                System.out.println(1);
            }
        }).start();

        try {
            c.await();
        } catch (Exception e) {

        }
        System.out.println(2);
    }

    static class A implements Runnable {

        @Override
        public void run() {
            System.out.println(3);
        }

    }

}
```

### 8.2.2 CyclicBarrier的应用场景

​	多线程计算，最后合并结果。

### 8.2.3 CyclicBarrier和ContDownLatch的区别。

​	ContDownLatch只能使用一次，CyclicBarrier的计算器可以使用reset()重置。

​	还提供了：`getNumberWaiting`获取CyclicBarrier阻塞的线程数量，`isBroken()`用来了解阻塞线程室友被中断。

```java
public class CyclicBarrierTest3 {

    static CyclicBarrier c = new CyclicBarrier(2);

    public static void main(String[] args) throws InterruptedException, BrokenBarrierException {
        Thread thread = new Thread(new Runnable() {

            @Override
            public void run() {
                try {
                    c.await();
                } catch (Exception e) {
                }
            }
        });
        thread.start();
        thread.interrupt();
        try {
            c.await();
        } catch (Exception e) {
            System.out.println(c.isBroken());
        }
    }
}
```

## 8.3 控制并发线程数的Semaphore

​	用来控制同时访问特定资源的线程数量。

**1. 应用场景**

​	可以用于做流量控制，特别是公用资源有限的应用场景，比如数据库连接。

```java
public class SemaphoreTest {

    private static final int       THREAD_COUNT = 30;

    private static ExecutorService threadPool   = Executors.newFixedThreadPool(THREAD_COUNT);

    private static Semaphore       s            = new Semaphore(10);

    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        s.acquire();
                        System.out.println("save data");
                        s.release();
                    } catch (InterruptedException e) {
                    }
                }
            });
        }

        threadPool.shutdown();
    }
}
```

**2. 其他方法**

* intavailablePermits()： 返回 此 信号 量 中 当前 可用 的 许可证 数。 ·
* intgetQueueLength()： 返回 正在 等待 获取 许可证 的 线程 数。 
* booleanhasQueuedThreads()： 是否 有线 程 正在 等待 获取 许可证。 
* void reducePermits（ int reduction）： 减少 reduction 个 许可证， 是个 protected 方法。 
* Collection getQueuedThreads()： 返回 所有 等待 获取 许可证 的 线程 集合， 是个 protected 方法。

## 8.4 线程间交换数据的Exchanger

​	用于线程间数据交换。提供一个同步点，在这个同步点，两个线程可以交换批次的数据。

​	可以用于遗传算法，也可以用于校对工作。

```java
public class ExchangerTest {

    private static final Exchanger<String> exgr       = new Exchanger<String>();

    private static ExecutorService         threadPool = Executors.newFixedThreadPool(2);

    public static void main(String[] args) {

        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String A = "银行流水A";// A录入银行流水数据
                    exgr.exchange(A);
                } catch (InterruptedException e) {
                }
            }
        });

        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String B = "银行流水B";// B录入银行流水数据
                    String A = exgr.exchange("B");
                    System.out.println("A和B数据是否一致：" + A.equals(B) + "，A录入的是：" + A + "，B录入是：" + B);
                } catch (InterruptedException e) {
                }
            }
        });

        threadPool.shutdown();

    }
}
```

可以设置第二个参数来设置最大等待时长：`exgr.exchange("B", 10, TimeUnit.SECONDS)`。

# 第9章 Java中的线程池

​	3个好处：

* 降低资源消耗。
* 提高响应速度。
* 提高线程的可管理性。

## 9.1 线程池的实现原理

​	处理流程如下：

1. 线程池判断核心线程池里的线程是否都在执行任务。如果不是，则创建一个新的线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下一个线程。
2. 线程池判断工作队列是否已经满了。如果工作队列没满，则将新提交的任务存储在这个队列里。如果满了，进入下个流程。
3. 线程池判断线程池的线程是否都处于工作状态，如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。

![](https://pic.imgdb.cn/item/5fbbab68b18d627113fabf16.jpg)

![](https://pic.imgdb.cn/item/5fbbab94b18d627113facbd9.jpg)

execute方法分为4种情况：

* 当先运行线程少于`corePoolSize`，则创建新线程来执行任务。

* 如果大于等于`corePoolSize`,则将任务加入`BlockingQueue`。

* 如果无法将任务加入`BlockingQueue`，则创建新的线程来处理任务(执行这一步骤需要获取全局锁)。

* 如果创建新建村将使当前运行的线程超出`maximumPoolSize`, 任务将被拒绝，并调`用RejectedExecutionHanlder.rejectedExecution()`方法。

  ![](https://pic.imgdb.cn/item/5fbbac71b18d627113fb0e2f.jpg)

  

工作线程：线程池创建线程时，会将线程封装成工作线程Worker，Work在执行完任务后，还会循环获取工作队列里的任务来执行。

![](https://pic.imgdb.cn/item/5fbbacfbb18d627113fb3b4a.jpg)

线程执行任务示意图如下：

![](https://pic.imgdb.cn/item/5fbbad15b18d627113fb4356.jpg)

分两种情况：

1. 在`execute()`方法中创建一个线程时，会让这个线程执行当前任务。
2. 这个线程执行完上图的1任务后，会反复从BlockingQueue获取任务来执行。

## 9.2 线程池的使用

### 9.2.1 线程池的创建

​	![](https://pic.imgdb.cn/item/5fbbae86b18d627113fbbc0a.jpg)

1） corePoolSize（ 线程 池 的 基本 大小）： 当 提交 一个 任务 到 线程 池 时， 线程 池 会 创建 一个 线程 来 执行任务， 即使 其他 空闲 的 基本 线程 能够 执行 新任务 也会 创建 线程， 等到 需要 执行 的 任务 数 大于 线程 池 基本 大小 时 就不 再 创建。 如果 调用 了 线程 池 的 prestartAllCoreThreads() 方法， 线程 池 会 提前 创建 并 启动 所有 基本 线程。

2） runnableTaskQueue（ 任务 队列）： 用于 保存 等待 执行 的 任务 的 阻塞 队列。 可以 选择 以下 几个 阻塞 队列。 

* ArrayBlockingQueue： 是一 个 基于 数组 结构 的 有 界 阻塞 队列， 此 队列 按 FIFO（ 先进 先出） 原则 对 元素 进行 排序。 
* LinkedBlockingQueue： 一个 基于 链 表 结构 的 阻塞 队列， 此 队列 按 FIFO 排序 元素， 吞吐量 通常 要 高于 ArrayBlockingQueue。 静态 工厂 方法 Executors. newFixedThreadPool() 使用 了 这个 队列。 
* SynchronousQueue： 一个 不 存储 元素 的 阻塞 队列。 每个 插入 操作 必须 等到 另一个 线程 调用 移 除 操作， 否则 插入 操作 一直 处于 阻塞 状态， 吞吐量 通常 要 高于 Linked- BlockingQueue， 静态 工厂 方法 Executors. newCachedThreadPool 使用 了 这个 队列。 
* PriorityBlockingQueue： 一个 具有 优先级 的 无限 阻塞 队列。

3） maximumPoolSize（ 线程 池 最大 数量）： 线程 池 允许 创建 的 最大 线程 数。 如果 队列 满了， 并且 已 创建 的 线程数 小于 最大 线程 数， 则 线程 池 会 再创 建 新的 线程 执行任务。 值得注意 的 是， 如果 使 用了 无 界 的 任务 队列 这个 参数 就 没什么 效果。 4） ThreadFactory： 用于 设置 创建 线程 的 工厂， 可以 通过 线程 工厂 给 每个 创建 出来 的 线程 设置 更有意义 的 名字。 使用 开源 框架 guava 提供 的 ThreadFactoryBuilder 可以 快速 给 线程 池 里 的 线程 设置 有意义 的 名字， 代码 如下。

​	![](https://pic.imgdb.cn/item/5fbbaf24b18d627113fbf81d.jpg)

5） RejectedExecutionHandler（ 饱和 策略）： 当 队列 和 线程 池 都 满了， 说明 线程 池 处于 饱和状态， 那么 必须 采取 一种 策略 处理 提交 的 新任务。 这个 策略 默认 情况下 是 AbortPolicy， 表示 无法 处理 新任 务时 抛出 异常。 在 JDK 1. 5 中 Java 线程 池 框架 提供 了 以下 4 种 策略。

* AbortPolicy： 直接 抛出 异常。 
* CallerRunsPolicy： 只用 调用 者 所在 线程 来 运行 任务。 
* DiscardOldestPolicy： 丢弃 队列 里 最近 的 一个 任务， 并 执行 当前任务。 ·
* DiscardPolicy： 不 处理， 丢弃 掉。

 当然， 也可以 根据 应用 场景 需要 来 实现 RejectedExecutionHandler 接口 自定义 策略。 如 记录 日志 或 持久 化 存储 不能 处理 的 任务。

* keepAliveTime（ 线程 活动 保持 时间）： 线程 池 的 工作 线程 空闲 后， 保持 存活 的 时间。 所以， 如果 任务 很多， 并且 每个 任务 执行 的 时间 比 较短， 可以 调 大 时间， 提高 线程 的 利用率。 
* TimeUnit（ 线程 活动 保持 时间 的 单位）： 可选 的 单位 有天（ DAYS）、 小时（ HOURS）、 分钟（ MINUTES）、 毫秒（ MILLISECONDS）、 微秒（ MICROSECONDS， 千分之一 毫秒） 和 纳秒（ NANOSECONDS， 千分之一 微秒）。

### 9.2.2 向线程池提交任务

​	分为execute()和submit()方法。

​	execute()用于提交不需要返回值的任务，无法判断是否执行成功。

![](https://pic.imgdb.cn/item/5fbbafd4b18d627113fc363b.jpg)

​	submit()方法用于提交需要返回值的任务，会返回一个future对象。

![](https://pic.imgdb.cn/item/5fbbaffeb18d627113fc45d5.jpg)

### 9.2.3 关闭线程池

​	通过`shutDown`或`shutDownNow`方法来关闭。原理是遍历工作线程，逐个调用线程的`intterrupt`方法来中断线程，所以无法响应中断的任务可能永远无法终止。区别， shutdownNow 首先 将 线程 池 的 状态 设置 成 STOP， 然后 尝试 停止 所有 的 正在 执行 或 暂停 任务 的 线程， 并 返回 等待 执行任务 的 列表， 而 shutdown 只是 将 线程 池 的 状态 设置 成 SHUTDOWN 状态， 然后 中断 所有 没有 正在 执行任务 的 线程。

​	只要 调 用了 这 两个 关闭 方法 中的 任意 一个， isShutdown 方法 就会 返回 true。 当 所有 的 任务 都已 关闭 后， 才 表示 线程 池 关闭 成功， 这时 调用 isTerminaed 方法 会 返回 true。

### 9.2.4 合理地配置线程池

​	首先需要分析任务特性。

* 任务性质：CPU密集型任务、IO密集型任务和混合型任务。
* 任务的优先级：高、中和低。
* 任务的执行时间：长、中和短。
* 任务的依赖性：是否依赖其他系统资源。



​	性质 不同 的 任务 可以 用不 同 规模 的 线程 池 分开 处理。 CPU 密集型 任务 应 配置 尽可能 小的 线程， 如 配置 Ncpu+ 1 个 线程 的 线程 池。 由于 IO 密集型 任务 线程 并不是 一直 在 执行任务， 则应 配置 尽可能 多的 线程， 如 2* Ncpu。 混合 型 的 任务， 如果 可以 拆分， 将其 拆分 成 一个 CPU 密集型 任务 和 一个 IO 密集型 任务， 只要 这 两个 任务 执行 的 时间 相差 不是 太大， 那么 分解 后 执行 的 吞吐量 将 高于 串行 执行 的 吞吐量。 如果 这 两个 任务 执行 时间 相差 太大， 则 没 必要 进行 分解。 可以 通过 Runtime. getRuntime(). availableProcessors() 方法 获得 当前 设备 的 CPU 个数。

​	优先级 不同 的 任务 可以 使用 优先级 队列 PriorityBlockingQueue 来 处理。 它 可以 让 优先级 高的 任务 先 执行。

* 建议使用有界队列。有 界 队列 能 增加 系统 的 稳定性 和 预警 能力， 可以 根据 需要 设 大一 点儿， 比如 几千。

  

### 9.2.5 线程池的监控

​	可以通过线程池提供的参数进行监控。

* taskCount： 线程 池 需要 执行 的 任务 数量。 
* completedTaskCount： 线程 池 在 运行 过程中 已完成 的 任务 数量， 小于 或 等于 taskCount。 
* largestPoolSize： 线程 池 里 曾经 创建 过 的 最大 线程 数量。 通过 这个 数据 可以 知道 线程 池 是否 曾经 满 过。 如 该数 值 等于 线程 池 的 最大 大小， 则 表示 线程 池 曾经 满 过。 
* getPoolSize： 线程 池 的 线程 数量。 如果 线程 池 不 销毁 的 话， 线程 池 里 的 线程 不会 自动 销毁， 所以 这个 大小 只 增 不减。 
* getActiveCount： 获取 活动 的 线程 数。 通过 扩展 线程 池 进行 监控。 可以 通过 继承 线程 池 来自 定义 线程 池， 重写 线程 池 的 beforeExecute、 afterExecute 和 terminated 方法， 也可 以在 任务 执行 前、 执行 后 和 线程 池 关闭 前 执行 一些 代码 来 进行 监控。 例如， 监控 任务 的 平均 执行 时间、 最大 执行 时间 和 最小 执行 时间等。 这 几个 方法 在 线程 池 里 是 空 方法。

. 

# 第10章 Executor框架

​	JDK5开始，把工作单元与执行机制分离开来。工作单元包括Runnable和Callable，而执行机制由Executor框架提供。

## 10.1 Executor框架简介

### 10.1.1 Executor框架的两级调度模型

​	在HotSpot VM线程模型中，Java线程被一对一映射为本地操作系统线程。Java线程启动时会创建一个本地操作系统线程；当该线程终止时，操作系统线程也会被回收。操作系统会调度所有线程并将它们分配给可用的CPU。

### 10.1.2 Executor框架的结构与成员

![](https://pic1.imgdb.cn/item/5fbd1c85b18d6271134fc50f.jpg)

**1. Executor框架的结构**

​	主要由3大部分组成：

* 任务。 包括 被 执行任务 需要 实现 的 接口： Runnable 接口 或 Callable 接口。 
* 任务 的 执行。 包括 任务 执行 机制 的 核心 接口 Executor， 以及 继承 自 Executor 的 ExecutorService 接口。 Executor 框架 有两 个 关键 类 实现 了 ExecutorService 接口（ ThreadPoolExecutor 和 ScheduledThreadPoolExecutor）。 
* 异步 计算 的 结果。 包括 接口 Future 和 实现 Future 接口 的 FutureTask 类。



​	下面是这些类和接口的简介。

* Executor 是一 个 接口， 它是 Executor 框架 的 基础， 它将 任务 的 提交 与 任务 的 执行 分离 开来。 
* ThreadPoolExecutor 是 线程 池 的 核心 实现 类， 用来 执行 被 提交 的 任务。 
* ScheduledThreadPoolExecutor 是一 个 实现 类， 可以 在给 定的 延迟 后 运行 命令， 或者 定期 执行 命令。 ScheduledThreadPoolExecutor 比 Timer 更 灵活， 功能 更 强大。 
* Future 接口 和 实现 Future 接口 的 FutureTask 类， 代表 异步 计算 的 结果。 
* Runnable 接口 和 Callable 接口 的 实现 类， 都可以 被 ThreadPoolExecutor 或 ScheduledThreadPoolExecutor 执行。

![](https://pic1.imgdb.cn/item/5fbd1d20b18d6271134ff727.jpg)

![](https://pic1.imgdb.cn/item/5fbd1d92b18d6271135019f6.jpg)

​	工具类Executors可以把一个Runnable对象封装为一个Callable对象对象（ Executors. callable（ Runnable task） 或 Executors. callable（ Runnable task， Object resule））。

​	如果 执行 ExecutorService. submit（…）， ExecutorService 将 返回 一个 实现 Future 接口 的 对象。由于 FutureTask 实现 了 Runnable， 程序员 也可以 创建 FutureTask， 然后 直接 交给 ExecutorService 执行。

**2. Executor框架的成员**

​	ThreadPoolExecutor、 ScheduledThreadPoolExecutor、 Future 接口、 Runnable 接口、 Callable 接口 和 Executors。

（1） ThreadPoolExecutor

​	ThreadPoolExecutor 通常 使用 工厂 类 Executors 来 创建。 Executors 可以 创建 3 种 类型 的 ThreadPoolExecutor： SingleThreadExecutor、 FixedThreadPool 和 CachedThreadPool。

​	Executors 可以 创建 3 种 类型 的 ThreadPoolExecutor： SingleThreadExecutor、 FixedThreadPool 和 CachedThreadPool。

（2） ScheduledThreadPoolExecutor

ScheduledThreadPoolExecutor 通常 使用 工厂 类 Executors 来 创建。 Executors 可以 创建 2 种 类型 的 ScheduledThreadPoolExecutor， 如下。 

* ScheduledThreadPoolExecutor。 包含 若干个 线程 的 ScheduledThreadPoolExecutor。 
* SingleThreadScheduledExecutor。 只 包含 一个 线程 的 ScheduledThreadPoolExecutor。

（3） Future 接口

​	Future 接口 和 实现 Future 接口 的 FutureTask 类 用来 表示 异步 计算 的 结果。 当 我们 把 Runnable 接口 或 Callable 接口 的 实现 类 提交（ submit） 给 ThreadPoolExecutor 或 ScheduledThreadPoolExecutor 时， ThreadPoolExecutor 或 ScheduledThreadPoolExecutor 会 向我 们 返回 一个 FutureTask 对象。

（4） Runnable 接口 和 Callable 接口

​	Runnable 接口 和 Callable 接口 的 实现 类， 都可以 被 ThreadPoolExecutor 或 Scheduled- ThreadPoolExecutor 执行。 它们 之间 的 区别 是 Runnable 不会 返回 结果， 而 Callable 可以 返回 结果。

## 10.2 ThreadPoolExecutor详解

​	最核心的类是ThreadPoolExecutor。是线程池的实现类

​	Executor 框架 最 核心 的 类 是 ThreadPoolExecutor， 它是 线程 池 的 实现 类， 主要 由 下列 4 个 组件 构成。 

* corePool： 核心 线程 池 的 大小。 
* maximumPool： 最大 线程 池 的 大小。 
* BlockingQueue： 用来 暂时 保存 任务 的 工作 队列。 
* RejectedExecutionHandler： 当 ThreadPoolExecutor 已经 关闭 或 ThreadPoolExecutor 已经 饱和 时（ 达到 了 最大 线程 池 大小 且 工作 队列 已满）， execute() 方法 将要 调用 的 Handler。



### 10.2.1 FixedThreadPool详解

![](https://pic1.imgdb.cn/item/5fbd20a2b18d6271135100dd.jpg)

![](https://pic1.imgdb.cn/item/5fbd20c4b18d627113510a9b.jpg)

1） 如果 当前 运行 的 线程 数 少于 corePoolSize， 则 创建 新 线程 来 执行任务。 

2） 在 线程 池 完成 预热 之后（ 当前 运行 的 线程 数 等于 corePoolSize）， 将 任务 加入 LinkedBlockingQueue。 

3） 线程 执行 完 1 中的 任务 后， 会在 循环 中 反复 从 LinkedBlockingQueue 获取 任务 来 执行。

FixedThreadPool 使用 无 界 队列 LinkedBlockingQueue 作为 线程 池 的 工作 队列（ 队列 的 容量 为 Integer. MAX_ VALUE）。

1） 当 线程 池 中的 线程 数 达到 corePoolSize 后， 新任务 将 在 无 界 队列 中 等待， 因此 线程 池 中的 线程 数 不会 超过 corePoolSize。 

2） 由于 1， 使用 无 界 队列 时 maximumPoolSize 将是 一个 无效 参数。 

3） 由于 1 和 2， 使用 无 界 队列 时 keepAliveTime 将是 一个 无效 参数。 

4） 由于 使用 无 界 队列， 运行 中的 FixedThreadPool（ 未 执行 方法 shutdown() 或 shutdownNow()） 不会 拒绝 任务（ 不会 调用 RejectedExecutionHandler. rejectedExecution 方法）。

### 10.2.2 SingleThreadExecutor详解

​	

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

1） 如果 当前 运行 的 线程 数 少于 corePoolSize（ 即 线程 池 中 无 运行 的 线程）， 则 创建 一个 新 线程 来 执行任务。 

2） 在 线程 池 完成 预热 之后（ 当前 线程 池 中有 一个 运行 的 线程）， 将 任务 加入 Linked- BlockingQueue。 

3） 线程 执行 完 1 中的 任务 后， 会在 一个 无限 循环 中 反复 从 LinkedBlockingQueue 获取 任务 来 执行。

### 10.2.3 CachedThreadPool详解

![](https://pic1.imgdb.cn/item/5fbfbdc815e771908423f18c.jpg)

​	CachedThreadPool 的 corePoolSize 被 设置 为 0， 即 corePool 为 空； maximumPoolSize 被 设置 为 Integer. MAX_ VALUE， 即 maximumPool 是 无 界 的。 这里 把 keepAliveTime 设置 为 60L， 意味着 CachedThreadPool 中的 空闲 线程 等待 新任务 的 最长 时间 为 60 秒， 空闲 线程 超过 60 秒 后 将会 被 终止。 

​	FixedThreadPool 和 SingleThreadExecutor 使用 无 界 队列 LinkedBlockingQueue 作为 线程 池 的 工作 队列。 CachedThreadPool 使用 没有 容量 的 SynchronousQueue 作为 线程 池 的 工作 队列， 但 CachedThreadPool 的 maximumPool 是 无 界 的。 这 意味着， 如果 主线 程 提交 任务 的 速度 高于 maximumPool 中线 程 处理 任务 的 速度 时， CachedThreadPool 会 不断 创建 新 线程。 极端 情况下， CachedThreadPool 会 因为 创建 过多 线程 而 耗尽 CPU 和 内存 资源。

![](https://pic1.imgdb.cn/item/5fbfbe3315e7719084240d0c.jpg)

1） 首先 执行 SynchronousQueue. offer（ Runnable task）。 如果 当前 maximumPool 中有 空闲 线程 正在 执行 SynchronousQueue. poll（ keepAliveTime， TimeUnit. NANOSECONDS）， 那么 主线 程 执行 offer 操作 与 空闲 线程 执行 的 poll 操作 配对 成功， 主 线程 把 任务 交给 空闲 线程 执行， execute() 方法 执行 完成； 否则 执行 下面 的 步骤 2）。

2） 当初 始 maximumPool 为 空， 或者 maximumPool 中 当前 没有 空闲 线程 时， 将 没有 线程 执行 SynchronousQueue. poll（ keepAliveTime， TimeUnit. NANOSECONDS）。 这种 情况下， 步骤 1） 将 失败。 此时 CachedThreadPool 会 创建 一个 新 线程 执行任务， execute() 方法 执行 完成。

3） 在 步骤 2） 中 新 创建 的 线程 将 任务 执行 完 后， 会 执行 SynchronousQueue. poll（ keepAliveTime， TimeUnit. NANOSECONDS）。 这个 poll 操作 会 让 空闲 线程 最多 在 SynchronousQueue 中 等待 60 秒钟。 如果 60 秒钟 内主 线程 提交 了 一个 新任务（ 主线 程 执行 步骤 1））， 那么 这个 空闲 线程 将 执行 主线 程 提交 的 新任务； 否则， 这个 空闲 线程 将 终止。 由于 空闲 60 秒 的 空闲 线程 会被 终止， 因此 长时间 保持 空闲 的 CachedThreadPool 不会 使用 任何 资源。

## 10.3 ScheduledThreadPoolExecutor

​	它 主要 用来 在给 定的 延迟 之后 运行 任务， 或者 定期 执行任务。 ScheduledThreadPoolExecutor 的 功能 与 Timer 类似， 但 ScheduledThreadPoolExecutor 功能 更 强大、 更 灵活。

### 10.3.1 ScheduledThreadPoolExecutor的运作机制

​	DelayQueue 是 一个 无 界 队列， 所以 ThreadPoolExecutor 的 maximumPoolSize 在 Scheduled- ThreadPoolExecutor 中 没有 什么 意义。

1） 当 调用 ScheduledThreadPoolExecutor 的 scheduleAtFixedRate() 方法 或者 scheduleWith- FixedDelay() 方法 时， 会 向 ScheduledThreadPoolExecutor 的 DelayQueue 添加 一个 实现 了 RunnableScheduledFutur 接口 的 ScheduledFutureTask。 

2） 线程 池 中的 线程 从 DelayQueue 中 获取 ScheduledFutureTask， 然后 执行任务。

![](https://pic1.imgdb.cn/item/5fbfc23115e771908425b1ed.jpg)

### 10.3.2 ScheduledThreadPoolExecutor的实现

3个成员变量：

* long型成员变量time，表示这个任务将要被执行的具体时间。
* long型成员变量sequenceNumber，便是这个任务被添加到ScheduledThreadPoolExecutor中的序号。
* long型成员变量period，表示任务执行的间隔周期。

DelayQueue 封装 了 一个 PriorityQueue， 这个 PriorityQueue 会对 队列 中的 Scheduled- FutureTask 进行 排序。 排序 时， time 小的 排在 前面（ 时间 早的 任务 将被 先 执行）。 如果 两个 ScheduledFutureTask 的 time 相同， 就 比较 sequenceNumber， sequenceNumber 小的 排在 前面（ 也就是说， 如果 两个 任务 的 执行 时间 相同， 那么 先 提交 的 任务 将被 先 执行）。

![](https://pic1.imgdb.cn/item/5fbfc32a15e77190842652e7.jpg)

1） 线程 1 从 DelayQueue 中 获取 已 到期 的 ScheduledFutureTask（ DelayQueue. take()）。 到期 任务 是指 ScheduledFutureTask 的 time 大于 等于 当前 时间。 

2） 线程 1 执行 这个 ScheduledFutureTask。

3） 线程 1 修改 ScheduledFutureTask 的 time 变量 为 下次 将要 被 执行 的 时间。

4） 线程 1 把这 个 修改 time 之后 的 ScheduledFutureTask 放回 DelayQueue 中（ Delay- Queue. add()）。

![](https://pic1.imgdb.cn/item/5fbfc39615e7719084269dba.jpg)

![](https://pic1.imgdb.cn/item/5fc111c615e77190847cc20c.jpg)

获取任务分为3大步骤

1) 获取Lock

2）获取周期任务

* 如果 PriorityQueue 为 空， 当前 线程 到 Condition 中 等待； 否则 执行 下面 的 2. 2。

* 如果 PriorityQueue 的 头 元素 的 time 时间 比 当前 时间 大， 到 Condition 中等 待到 time 时间； 否则 执行 下面 的 2. 3。

* 获取 PriorityQueue 的 头 元素（ 2. 3. 1）； 如果 PriorityQueue 不为 空， 则 唤醒 在 Condition 中 等待 的 所有 线程（ 2. 3. 2）。

3）释放Lock。

  ScheduledThreadPoolExecutor 在 一个 循环 中 执行 步骤 2， 直到 线程 从 PriorityQueue 获取 到 一个 元素 之后（ 执行 2. 3. 1 之后）， 才会 退出 无限 循环（ 结束 步骤 2）。

![](https://pic1.imgdb.cn/item/5fc1125015e77190847cecf2.jpg)

![](https://pic1.imgdb.cn/item/5fc1127815e77190847cf865.jpg)

​	添加 任务 分为 3 大 步骤。 

1） 获取 Lock。 

2） 添加 任务。 

* 向 PriorityQueue 添加 任务。 
* 如果 在上面 2. 1 中 添加 的 任务 是 PriorityQueue 的 头 元素， 唤醒 在 Condition 中 等待 的 所有 线程。

3） 释放锁。

## 10.4 FutureTask详解

​	Future 接口 和 实现 Future 接口 的 FutureTask 类， 代表 异步 计算 的 结果。

### 10.4.1 FutureTask简介

​	除了实现Future接口，害实现了Runnable接口，因此可以交给Executor执行。也可以调用线程直接执行。

1） 未 启动。 FutureTask. run() 方法 还没 有被 执行 之前， FutureTask 处于 未 启动 状态。 当 创建 一个 FutureTask， 且 没有 执行 FutureTask. run() 方法 之前， 这个 FutureTask 处于 未 启动 状态。 

2） 已 启动。 FutureTask. run() 方法 被 执行 的 过程中， FutureTask 处于 已 启动 状态。 3） 已完成。 FutureTask. run() 方法 执行 完 后 正常 结束， 或 被 取消（ FutureTask. cancel（…））， 或 执行 FutureTask. run() 方法 时 抛出 异常 而异 常 结束， FutureTask 处于 已完成 状态。

![](https://pic1.imgdb.cn/item/5fc1155e15e77190847dc668.jpg)

​	当 FutureTask 处于 未 启动 或 已 启动 状态 时， 执行 FutureTask. get() 方法 将 导致 调用 线程 阻塞； 当 FutureTask 处于 已完成 状态 时， 执行 FutureTask. get() 方法 将 导致 调用 线程 立即 返回 结果 或 抛出 异常。

​	当 FutureTask 处于 未 启动 状态 时， 执行 FutureTask. cancel() 方法 将 导致 此 任务 永远 不会 被 执行； 当 FutureTask 处于 已 启动 状态 时， 执行 FutureTask. cancel（ true） 方法 将以 中断 执行 此 任务 线程 的 方式 来试 图 停止 任务； 当 FutureTask 处于 已 启动 状态 时， 执行 FutureTask. cancel（ false） 方法 将不 会对 正在 执行 此 任务 的 线程 产生 影响（ 让 正在 执行 的 任务 运行 完成）； 当 FutureTask 处于 已完成 状态 时， 执行 FutureTask. cancel（…） 方法 将 返回 false。

![](https://pic1.imgdb.cn/item/5fc1159915e77190847dd9aa.jpg)

### 10.4.2 FutureTask的使用

​	可以 把 FutureTask 交给 Executor 执行； 也可以 通过 ExecutorService. submit（…） 方法 返回 一个 FutureTask， 然后 执行 FutureTask. get() 方法 或 FutureTask. cancel（…） 方法。 除此以外， 还可以 单独 使用 FutureTask。

​	当 一个 线程 需要 等待 另一个 线程 把 某个 任务 执行 完 后 它 才能 继续 执行， 此时 可以 使用 FutureTask。 假设 有多 个 线程 执行 若干 任务， 每个 任务 最多 只能 被 执行 一次。 当 多个 线程 试图 同时 执行 同一个 任务 时， 只 允许 一个 线程 执行任务， 其他 线程 需要 等待 这个 任务 执行 完 后才 能 继续 执行。

```java
public class ConcurrentTask {

    private final ConcurrentMap<Object, Future<String>> taskCache = new ConcurrentHashMap<Object, Future<String>>();

    private String executionTask(final String taskName) throws ExecutionException, InterruptedException {
        while (true) {
            Future<String> future = taskCache.get(taskName); //1.1,2.1
            if (future == null) {
                Callable<String> task = new Callable<String>() {
                    public String call() throws InterruptedException {
                        //......
                        return taskName;
                    }
                };
                //1.2创建任务
                FutureTask<String> futureTask = new FutureTask<String>(task);
                future = taskCache.putIfAbsent(taskName, futureTask); //1.3
                if (future == null) {
                    future = futureTask;
                    futureTask.run(); //1.4执行任务
                }
            }

            try {
                return future.get(); //1.5,2.2线程在此等待任务执行完成
            } catch (CancellationException e) {
                taskCache.remove(taskName, future);
            }
        }
    }

}
```

![](https://pic1.imgdb.cn/item/5fc37f1cd590d4788ac8e0fb.jpg)

### 10.4.3 FurtureTask的实现

​	实现基于AbstractQueuedSychronizer(以下简称AQS)。队列。 JDK 6 中 AQS 被 广泛 使用， 基于 AQS 实现 的 同步 器 包括： ReentrantLock、 Semaphore、 ReentrantReadWriteLock、 CountDownLatch 和 FutureTask。

​	每一个 基于 AQS 实现 的 同步 器 都会 包含 两种 类型 的 操作， 如下。 

* 至少 一个 acquire 操作。 这个 操作 阻塞 调用 线程， 除非/ 直到 AQS 的 状态 允许 这个 线程 继续 执行。 FutureTask 的 acquire 操作 为 get()/ get（ long timeout， TimeUnit unit） 方法 调用。 

* 至少 一个 release 操作。 这个 操作 改变 AQS 的 状态， 改变 后的 状态 可 允许 一个 或 多个 阻塞 线程 被 解除 阻塞。 FutureTask 的 release 操作 包括 run() 方法 和 cancel（…） 方法。 



​	基于“ 复合 优先于 继承” 的 原则， FutureTask 声明 了 一个 内部 私有 的 继承 于 AQS 的 子类 Sync， 对 FutureTask 所有 公有 方法 的 调用 都会 委托 给 这个 内部 子类。 

​	AQS 被 作为“ 模板 方法 模式” 的 基础 类 提 供给 FutureTask 的 内部 子类 Sync， 这个 内部 子类 只需 要 实现 状态 检查 和 状态 更新 的 方法 即可， 这些 方法 将 控制 FutureTask 的 获取 和 释放 操作。 具体来说， Sync 实现 了 AQS 的 tryAcquireShared（ int） 方法 和 tryReleaseShared（ int） 方法， Sync 通过 这 两个 方法 来 检查 和 更新 同步 状态。

![](https://pic1.imgdb.cn/item/5fc38245d590d4788ac9df75.jpg)

​	FutureTask. get() 方法 会 调用 AQS. acquireSharedInterruptibly（ int arg） 方法， 这个 方法 的 执行 过程 如下。 

​	1） 调用 AQS. acquireSharedInterruptibly（ int arg） 方法， 这个 方法 首先 会 回 调 在 子类 Sync 中 实现 的 tryAcquireShared() 方法 来 判断 acquire 操作 是否 可以 成功。 acquire 操作 可以 成功 的 条件 为： state 为 执行 完成 状态 RAN 或 已 取消 状态 CANCELLED， 且 runner 不为 null。 

​	2） 如果 成功 则 get() 方法 立即 返回。 如果 失败 则 到 线程 等待 队列 中去 等待 其他 线程 执行 release 操作。 

​	3） 当 其他 线程 执行 release 操作（ 比如 FutureTask. run() 或 FutureTask. cancel（…）） 唤醒 当前 线程 后， 当前 线程 再次 执行 tryAcquireShared() 将 返回 正值 1， 当前 线程 将 离开 线程 等待 队列 并 唤醒 它的 后继 线程（ 这里 会 产生 级 联 唤醒 的 效果， 后面 会 介绍）。 

​	4） 最后 返回 计算 的 结果 或 抛出 异常。 

​	FutureTask. run() 的 执行 过程 如下。 

​	1） 执行 在 构造 函数 中指 定的 任务（ Callable. call()）。 

​	2） 以 原子 方式 来 更新 同步 状态（ 调用 AQS. compareAndSetState（ int expect， int update）， 设置 state 为 执行 完成 状态 RAN）。 如果 这个 原子 操作 成功， 就 设置 代表 计算 结果 的 变量 result 的 值 为 Callable. call() 的 返回 值， 然后 调用 AQS. releaseShared（ int arg）。 

​	3） AQS. releaseShared（ int arg） 首先 会 回 调 在 子类 Sync 中 实现 的 tryReleaseShared（ arg） 来 执行 release 操作（ 设置 运行 任务 的 线程 runner 为 null， 然 会 返回 true）； AQS. releaseShared（ int arg）， 然后 唤醒 线程 等待 队列 中的 第一个 线程。 

​	4） 调用 FutureTask. done()。

![](https://pic1.imgdb.cn/item/5fc3898cd590d4788acbb683.jpg)

​	假设 开始时 FutureTask 处于 未 启动 状态 或 已 启动 状态， 等待 队列 中 已经 有 3 个 线程（ A、 B 和 C） 在 等待。 此时， 线程 D 执行 get() 方法 将 导致 线程 D 也 到 等待 队列 中去 等待。 

​	当 线程 E 执行 run() 方法 时， 会 唤醒 队列 中的 第一个 线程 A。 线程 A 被 唤醒 后， 首先 把 自己 从 队列 中 删除， 然后 唤醒 它的 后继 线程 B， 最后 线程 A 从 get() 方法 返回。 线程 B、 C 和 D 重复 A 线程 的 处理 流程。 最终， 在 队列 中 等待 的 所有 线程 都被 级 联 唤醒 并从 get() 方法 返回。

