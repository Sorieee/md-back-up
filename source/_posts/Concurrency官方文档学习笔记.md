---
title: Concurrency官方文档学习笔记
date: 2020-11-04 11:37:58
tags: [java, 并发编程]
---

https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html

​	本课介绍平台的基本并发支持，并总结了java.util.concurrent包。

# Processes and Threads

​	在并发编程中，有两个基础的执行单位：进程和线程。在Java语言中，并发基本和线程相关。但是进程也同样重要。

## Processes

​	一个进程包含一个独立的执行环境。进程通常有一组完整的、私有的基本运行时资源。特别是，每个进程都有自己的内存空间。

​	进程通常被视为程序或应用程序的同义词。然而，用户所看到的单个应用程序实际上可能是一组协作的进程。为了促进进程间交流，大多数系统支持*Inter Process Communication* (IPC) ，比如pipes和sockets。IPC不仅用于同一系统上进程之间的通信，而且用于不同系统上的进程之间的通信。

​	Java虚拟机的大多数实现都是作为单个进程运行的。Java应用程序可以使用ProcessBuilder对象创建其他进程。多进程应用程序超出了本课的讨论范围。

## Threads

​	线程有时候叫做轻量级进程。进程和线程都是创造一个执行环境，但是创建一个线程比创建一个进程所消耗的资源更少。

​	线程存在于进程中。一个进程至少有一个线程。线程共享进程的资源，包括内存和打开文件。这有助于有效的沟通，但可能存在问题。

​	多线程是Java平台一个重要特性。每个应用有一个到多个线程，如果你计算那些执行内存管理和信号处理之类的“系统”线程。但是从应用程序程序员的角度来看，您只从一个线程开始，称为主线程。这个线程能够创建额外的线程，我们将在下一节中演示。

# Thread Objects

​	每个线程和Thread类的实例相关。使用线程对象创建并发应用程序有两种基本策略。

* 要直接控制线程的创建和管理，只需在应用程序每次需要启动异步任务时实例化线程。
* 要从应用程序的其余部分抽象线程管理，请将应用程序的任务传递给执行器。

## Defining and Starting a Thread

​	创建线程实例有两种方法。

* 提供一个Runnable对象。

```java
public class HelloRunnable implements Runnable {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new Thread(new HelloRunnable())).start();
    }

}
```

* 继承Thread。

```java
public class HelloThread extends Thread {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new HelloThread()).start();
    }

}
```

​	注意两种都要调用`Thread.start`去启动一个新线程。

​	你应该用哪一个？第一个用法使用`Runnable`，它更通用，因为可运行对象可以作为线程以外的类的子类。

​	Thread类定义了许多对线程管理有用的方法。其中包括静态方法，它们提供有关调用方法的线程的信息或影响其状态。其他方法从管理线程和线程对象的其他线程调用。我们将在下面几节中研究其中一些方法。

## Pausing Execution with Sleep

​	Thread.sleep在指定时间段内暂停执行。有两个版本的sleep方法。一个是指定停顿的毫秒，另外一个是纳秒。然而，这些睡眠时间不能保证精确，因为它们受到底层操作系统提供的功能的限制。另外，睡眠期可以通过中断来终止，我们将在后面的章节中看到。在任何情况下，都不能假定调用sleep将在指定的时间段内挂起线程。

```java
public class SleepMessages {
    public static void main(String args[])
        throws InterruptedException {
        String importantInfo[] = {
            "Mares eat oats",
            "Does eat oats",
            "Little lambs eat ivy",
            "A kid will eat ivy too"
        };

        for (int i = 0;
             i < importantInfo.length;
             i++) {
            //Pause for 4 seconds
            Thread.sleep(4000);
            //Print a message
            System.out.println(importantInfo[i]);
        }
    }
}
```

​	注意main声明它抛出InterruptedException。当睡眠处于活动状态时，另一个线程中断当前线程时，sleep会抛出这个异常。由于这个应用程序没有定义另一个线程来引起中断，所以它不必费心去捕捉InterruptedException。

## Interrupts

​	中断表明线程应该停止正在做的事情然后去做其他事情。由程序员来决定线程如何响应中断，但是线程终止是很常见的。这是本课强调的用法。

​	线程通过调用`interrupt`方法去发送一个中断信号。为了使中断机制正常工作，被中断的线程必须支持自己的中断。

### Supporting Interruption

​	如果线程经常调用抛出InterruptedException的方法，那么它只需在捕捉到该异常之后从run方法返回。

```java
for (int i = 0; i < importantInfo.length; i++) {
    // Pause for 4 seconds
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        // We've been interrupted: no more messages.
        return;
    }
    // Print a message
    System.out.println(importantInfo[i]);
}
```

​	许多抛出InterruptedException的方法，如sleep，都被设计成取消当前操作，并在收到中断时立即返回。

​	如果一个线程运行了很长一段时间而没有调用抛出InterruptedException的方法怎么办？然后它必须周期性地调用线程中断，如果接收到中断，则返回true。例如

```java
for (int i = 0; i < inputs.length; i++) {
    heavyCrunch(inputs[i]);
    if (Thread.interrupted()) {
        // We've been interrupted: no more crunching.
        return;
    }
}
```

在这个简单的例子中，代码只是测试中断并在收到中断时退出线程。在更复杂的应用程序中，抛出InterruptedException可能更有意义：

```java
if (Thread.interrupted()) {
    throw new InterruptedException();
}
```

这使得中断处理代码可以集中在catch子句中。

### The Interrupt Status Flag

​	中断机制是使用一个称为中断状态的内部标志来实现的。调用`Thread.interrupt`设置此标志。当线程检测到有一个中断后，中断状态会清除。一个线程使用非静态isInterrupted方法来查询另一个线程的中断状态，它不会更改中断状态标志。

​	按照惯例，任何通过抛出`InterruptedException`退出的方法都会在执行此操作时清除中断状态。然而，总是有可能由另一个调用中断的线程再次设置中断状态。

## Joins

​	`join`方法可以使得一个线程等待另一个线程完成。如果t是一个线程对象，其线程当前正在执行

```
t.join();
```

​	会导致当前线程停止知道t线程结束。他允许程序员指定一个等待时间。然而，和sleep一样。这个等待时间不精确。

​	和sleep一样，join通过使用InterruptedException退出来响应中断。

## The SimpleThreads Example

​	下面的样例有两个线程，一个是主线程，主线程通过Runnable对象执行一个循环打印。如果循环打印时间太长，主线程会中断它。

​	MessageLoop线程打印出一系列消息。如果在打印完所有消息之前被中断，MessageLoop线程将打印一条消息并退出。

```java
public class SimpleThreads {

    // Display a message, preceded by
    // the name of the current thread
    static void threadMessage(String message) {
        String threadName =
            Thread.currentThread().getName();
        System.out.format("%s: %s%n",
                          threadName,
                          message);
    }

    private static class MessageLoop
        implements Runnable {
        public void run() {
            String importantInfo[] = {
                "Mares eat oats",
                "Does eat oats",
                "Little lambs eat ivy",
                "A kid will eat ivy too"
            };
            try {
                for (int i = 0;
                     i < importantInfo.length;
                     i++) {
                    // Pause for 4 seconds
                    Thread.sleep(4000);
                    // Print a message
                    threadMessage(importantInfo[i]);
                }
            } catch (InterruptedException e) {
                threadMessage("I wasn't done!");
            }
        }
    }

    public static void main(String args[])
        throws InterruptedException {

        // Delay, in milliseconds before
        // we interrupt MessageLoop
        // thread (default one hour).
        long patience = 1000 * 60 * 60;

        // If command line argument
        // present, gives patience
        // in seconds.
        if (args.length > 0) {
            try {
                patience = Long.parseLong(args[0]) * 1000;
            } catch (NumberFormatException e) {
                System.err.println("Argument must be an integer.");
                System.exit(1);
            }
        }

        threadMessage("Starting MessageLoop thread");
        long startTime = System.currentTimeMillis();
        Thread t = new Thread(new MessageLoop());
        t.start();

        threadMessage("Waiting for MessageLoop thread to finish");
        // loop until MessageLoop
        // thread exits
        while (t.isAlive()) {
            threadMessage("Still waiting...");
            // Wait maximum of 1 second
            // for MessageLoop thread
            // to finish.
            t.join(1000);
            if (((System.currentTimeMillis() - startTime) > patience)
                  && t.isAlive()) {
                threadMessage("Tired of waiting!");
                t.interrupt();
                // Shouldn't be long now
                // -- wait indefinitely
                t.join();
            }
        }
        threadMessage("Finally!");
    }
}
```

# Synchronization

​	线程主要通过共享对字段和引用字段引用的对象的访问进行通信。这种形式的通信非常高效，但是可能会产生两种错误：线程干扰和内存一致性错误。预防这些错误的工具是同步。

​	但是，同步可能会引入线程争用：当两个或多个线程尝试同时访问同一资源并导致Java运行时执行一个或多个线程的速度较慢时发生，甚至中止执行。

​	线程饥饿和或所是线程争用的一种形式。See the section [Liveness](https://docs.oracle.com/javase/tutorial/essential/concurrency/liveness.html) for more information.

## Thread Interference

看看一下这个叫Counter的例子。

```java
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }

}
```

​	如果一个Counter被多个线程引用可能会导致一些错误。当在不同线程中运行但作用于同一数据的两个操作交错时，会发生干扰。这意味着这两个操作由多个步骤组成，并且步骤序列重叠

​	计数器实例上的操作似乎不可能交错，因为c上的两个操作都是单一的简单语句。但是c++可以被分解为以下三个步骤：

1. 取出c的当前值。
2. 加1。
3. 保存。

c--也是一样的效果。

如果一个线程调用c++,一个调用c--。如果初始值是0，它们的交叉操作可能遵循以下顺序：

1. Thread A: Retrieve c.
2. Thread B: Retrieve c.
3. Thread A: Increment retrieved value; result is 1.
4. Thread B: Decrement retrieved value; result is -1.
5. Thread A: Store result in c; c is now 1.
6. Thread B: Store result in c; c is now -1.



​	线程A的结果丢失，被线程B覆盖。这种特殊的交织只有一种可能。在不同的情况下，可能是线程B的结果丢失，或者根本没有错误。因为线程干扰是不可预测的，所以很难检测和修复它们。

## Memory Consistency Errors

​	当不同线程对应该是相同数据的视图不一致时，会发生内存一致性错误。内存一致性错误的原因很复杂，超出了本教程的范围。幸运的是，程序员不需要详细了解这些原因。所需要的只是一个避免它们的策略。

​	避免内存一致性错误的关键是理解`happens-before`的关系。这种关系只是一种保证，由一个特定语句写入的内存对另一个特定语句可见。要了解这一点，请考虑下面的示例。假设定义并初始化了一个简单的int字段：

```
int counter = 0;
```

counter字段在两个线程A和B之间共享。假设线程A对它进行自增：

```
counter++;
```

然后，不久之后，线程B打印出它：

```java
System.out.println(counter);
```

​	两个语句在同一个线程中执行，可以假定打印出来的值是“1”。但是如果这两个语句在不同的线程中执行，那么打印出来的值很可能是“0”，因为不能保证线程A对计数器的更改对线程B是可见的——除非程序员在这两个语句之间建立了一个“发生在前”的关系。

​	有几个动作会创造`happens-before`关系。其中之一是同步，我们将在下面几节中看到。

* 当语句调用线程启动，与该语句具有`happens-before`关系的每个语句也与新线程执行的每个语句都有一个`happens-before`关系。导致创建新线程的代码的效果对新线程是可见的。
* 当一个线程结束导致另外一个线程调用的`Thread.join()`返回，然后，终止线程执行的所有语句与成功联接后的所有语句都有一个`happens-before`关系.

//todo

For a list of actions that create happens-before relationships, refer to the [Summary page of the `java.util.concurrent` package.](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html#MemoryVisibility).

## Synchronized Methods

​	java提供了两条基本的同步机制：同步方法和同步语句。

​	如果想使用同步方法，在方法声明中添加synchronized。

```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```

如果count是SynchronizedCounter的实例，则使这些方法同步有两个效果：

* 首先，在同一个对象上对同步方法的两次调用不可能交叉进行。当一个线程正在为一个对象执行同步方法时，为同一对象调用同步方法的所有其他线程都会阻塞（挂起执行），直到第一个线程处理完该对象为止。
* 第二，当同步方法退出时，它会自动与同一对象的同步方法的任何后续调用建立`happens-before`关系。这保证了对对象状态的更改对所有线程都是可见的。



​	注意, 构造器不能使用synchronized关键词修饰。同步构造函数没有意义，因为只有创建对象的线程在构造对象时才有权访问它。

> **警告**
>
> 当构造一个将在线程之间共享的对象时，要非常小心，对象的引用不会过早地“泄漏”。例如，假设您想要维护一个名为instances的列表，其中包含类的每个实例。您可能会想在构造函数中添加以下行：`instances.add(this);`
>
> 但在对象构造完成之前，其他线程可以使用实例访问该对象。

​	同步方法支持防止线程干扰和内存一致性错误的简单策略：如果一个对象对多个线程可见，则对该对象变量的所有读或写操作都是通过同步方法完成的。（一个重要的例外：final字段在对象构造之后不能修改，一旦对象被构造，就可以通过非同步的方法安全地读取）这个策略是有效的，但是可能会带来活跃性问题，我们将在本课后面的部分中看到。

## Intrinsic Locks and Synchronization

​	同步是围绕一个称为内置锁或监视器锁的内部实体构建的。内置锁在同步的放个方面发挥作用：

* 强制对对象状态的独占访问
* 建立对可见性非常重要的`happens-before`关系。

### Locks In Synchronized Methods

​	当线程调用同步方法时，它会自动获取该方法对象的内部锁，并在方法返回时释放它。即使返回是由未捕获的异常引起的，也会发生锁释放。

​	您可能想知道当调用静态同步方法时会发生什么，因为静态方法与类关联，而不是与对象关联。在这种情况下，线程获取与类关联的类对象的内部锁。因此，对类的静态字段的访问是由一个锁控制的，该锁不同于类的任何实例的锁。

### Synchronized Statements

​	和同步方法不一样，同步语句必须指定提供内置锁的对象。

```java
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }
    nameList.add(name);
}
```

### Reentrant Synchronization

​	允许一个线程多次获取同一个锁可以实现可重入同步。

​	这描述了一种情况，即同步代码直接或间接调用也包含同步代码的方法，并且两组代码使用相同的锁。如果没有可重入同步，同步代码将不得不采取许多额外的预防措施，以避免线程导致自身阻塞。

## Atomic Access

​	在编程中，原子性操作意味着一次性完成的事情。不可以在中间停顿。要么完成，要么不完成。原子作用的影响在作用完成之前是看不见的。

​	我们之前看的递增操作，c++，不是一个原子操作。即使是非常简单的表达式也可以定义可以分解为其他操作的复杂操作。

* 对于引用变量和大多数基本变量（除了long和double之外的所有类型），读和写都是原子的
* 对于所有声明为volatile的变量（包括long和double变量），读和写都是原子的。



​	原子操作不能交错，因此可以使用它们而不必担心线程干扰。但是，这并不能消除同步原子操作的所有需要，因为内存一致性错误仍然是可能的。使用volatile变量可以降低内存一致性错误的风险，因为对volatile变量的任何写入都会在后续读取同一变量之前建立一个`happens-before`关系。这意味着改变一个`volatile`变量对其他线程始终可见。这也意味着，当线程读取volatile变量时，它不仅可以看到volatile的最新更改，还可以看到另外一侧的代码的影响。

​	使用简单的原子变量访问比通过同步代码访问这些变量更有效，但是需要程序员更加小心以避免内存一致性错误。额外的工作是否值得，取决于应用程序的大小和复杂性。

​	一些java.util.concurrent包下的类提供不依赖同步的原子方法。我们将在高级并发对象一节中讨论它们。

# Liveness

​	并发应用程序及时执行的能力称为活动性(`liveness`)。这个章节主要描述了最常见的活跃性问题——死锁，以及其他两个活跃性问题：线程饥饿和活锁。

## Deadlock

​	死锁描述了两个以及更多的线程永远阻塞，互相等待。

```java
public class Deadlock {
    static class Friend {
        private final String name;
        public Friend(String name) {
            this.name = name;
        }
        public String getName() {
            return this.name;
        }
        public synchronized void bow(Friend bower) {
            System.out.format("%s: %s"
                + "  has bowed to me!%n", 
                this.name, bower.getName());
            bower.bowBack(this);
        }
        public synchronized void bowBack(Friend bower) {
            System.out.format("%s: %s"
                + " has bowed back to me!%n",
                this.name, bower.getName());
        }
    }

    public static void main(String[] args) {
        final Friend alphonse =
            new Friend("Alphonse");
        final Friend gaston =
            new Friend("Gaston");
        new Thread(new Runnable() {
            public void run() { alphonse.bow(gaston); }
        }).start();
        new Thread(new Runnable() {
            public void run() { gaston.bow(alphonse); }
        }).start();
    }
}
```

当死锁运行时，两个线程极有可能在尝试调用bowBack时阻塞。两个块都不会结束，因为每个线程都在等待另一个线程退出`bow`。

## Starvation and Livelock

​	饥饿和livelock比死锁更不常见，但仍然是每个并发软件设计者都可能遇到的问题。

### Starvation

​	饥饿描述了一种情况，即线程无法获得对共享资源的定期访问，并且无法取得进展。当共享资源被“贪婪”线程长时间占用时，就会发生这种情况。例如，假设一个对象提供了一个通常需要很长时间才能返回的同步方法。如果一个线程频繁调用此方法，那么其他需要频繁同步访问同一对象的线程通常会被阻塞。

### Livelock

​	一个线程经常响应另一个线程的操作。如果另一个线程的操作也是对另一个线程操作的响应，则可能会导致livelock。

​	与死锁一样，livelocked线程无法取得进一步的进展。但是，线程并没有被阻塞——它们只是忙于相互响应而无法继续工作。这相当于两个人在走廊里试图从对方身边经过：阿尔方斯向左移动让加斯顿通过，而加斯顿则向右移动让阿尔方斯通过。看到他们仍在互相阻拦，阿尔方斯向右移动，而加斯顿则向左移动。他们还在互相阻拦，所以。。。

# Guarded Blocks

​	线程之间经常需要互相协作。最常见的协作是`guarded block`。这样的块首先轮询一个条件，该条件必须为真，然后才能继续执行。要正确执行此操作，需要遵循许多步骤。

​	例如，假设guardedJoy是一个方法，它必须在另一个线程设置共享变量joy之前继续。理论上，这样的方法可以简单地循环直到满足条件，但是这个循环是浪费的，因为它在等待时连续执行。

```java
public void guardedJoy() {
    // Simple loop guard. Wastes
    // processor time. Don't do this!
    while(!joy) {}
    System.out.println("Joy has been achieved!");
}
```

一个更有效率的守卫对象。等等挂起当前线程。在另一个线程发出可能发生某些特殊事件的通知之前，wait的调用不会返回，但不一定是该线程正在等待的事件：

```java
public synchronized void guardedJoy() {
    // This guard only loops once for each special event, which may not
    // be the event we're waiting for.
    while(!joy) {
        try {
            wait();
        } catch (InterruptedException e) {}
    }
    System.out.println("Joy and efficiency have been achieved!");
}
```

​	像许多挂起执行的方法一样，wait可以抛出InterruptedException。在这个例子中，我们可以忽略它——我们只关心joy的值。

​	为什么这个版本是同步的？假设d是我们用来调用wait的对象。当一个线程调用d.wait时，它必须拥有d的内在锁，否则会抛出一个错误。在同步方法中调用wait是获取内部锁的简单方法。

```java
public synchronized notifyJoy() {
    joy = true;
    notifyAll();
}
```

​	当wait被调用时，线程释放锁并暂停执行。在将来的某个时候，另一个线程将获得相同的锁并调用Object.notifyAll()。通知所有等待该锁的线程发生了重要的事情。

​	在第二个线程释放锁之后的一段时间内，第一个线程重新获取锁并通过调用wait返回而继续。

> 还有第二种通知方法notify，它唤醒单个线程。因为notify不允许您指定被唤醒的线程，它只在大规模并行应用程序中有用，也就是说，具有大量线程的程序，所有这些程序都在做类似的工作。在这样的应用程序中，您不关心哪个线程被唤醒。

​	让我们使用保护块来创建生产者-消费者应用程序。这种应用程序在两个线程之间共享数据：生产者负责创建数据，消费者负责处理数据。这两个线程使用共享对象进行通信。协调是必不可少的：使用者线程在生产者线程传递数据之前不得尝试检索数据，如果使用者尚未检索到旧数据，生产者线程也不得尝试传递新数据

```java
public class Drop {
    // Message sent from producer
    // to consumer.
    private String message;
    // True if consumer should wait
    // for producer to send message,
    // false if producer should wait for
    // consumer to retrieve message.
    private boolean empty = true;

    public synchronized String take() {
        // Wait until message is
        // available.
        while (empty) {
            try {
                wait();
            } catch (InterruptedException e) {}
        }
        // Toggle status.
        empty = true;
        // Notify producer that
        // status has changed.
        notifyAll();
        return message;
    }

    public synchronized void put(String message) {
        // Wait until message has
        // been retrieved.
        while (!empty) {
            try { 
                wait();
            } catch (InterruptedException e) {}
        }
        // Toggle status.
        empty = false;
        // Store message.
        this.message = message;
        // Notify consumer that status
        // has changed.
        notifyAll();
    }
}
```

producer中定义的producer线程发送一系列消息。字符串“DONE”表示所有消息都已发送。为了模拟实际应用程序的不可预测性，生产者线程在消息之间随机暂停。

```java
import java.util.Random;

public class Producer implements Runnable {
    private Drop drop;

    public Producer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        String importantInfo[] = {
            "Mares eat oats",
            "Does eat oats",
            "Little lambs eat ivy",
            "A kid will eat ivy too"
        };
        Random random = new Random();

        for (int i = 0;
             i < importantInfo.length;
             i++) {
            drop.put(importantInfo[i]);
            try {
                Thread.sleep(random.nextInt(5000));
            } catch (InterruptedException e) {}
        }
        drop.put("DONE");
    }
}
```

​	consumer中定义的consumer线程只检索消息并将其打印出来，直到它检索到“DONE”字符串。此线程也会随机暂停一段时间。

```java
import java.util.Random;

public class Consumer implements Runnable {
    private Drop drop;

    public Consumer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        Random random = new Random();
        for (String message = drop.take();
             ! message.equals("DONE");
             message = drop.take()) {
            System.out.format("MESSAGE RECEIVED: %s%n", message);
            try {
                Thread.sleep(random.nextInt(5000));
            } catch (InterruptedException e) {}
        }
    }
}
```

主线程如下：



```java
public class ProducerConsumerExample {
    public static void main(String[] args) {
        Drop drop = new Drop();
        (new Thread(new Producer(drop))).start();
        (new Thread(new Consumer(drop))).start();
    }
}
```

# Immutable Objects

​	如果对象的状态在构造后无法更改，则认为它是不可变的。最大限度地依赖不可变对象被广泛接受为创建简单、可靠代码的合理策略。

​	不可变对象在并发应用程序中特别有用。因为它们不能更改状态，所以它们不能被线程干扰损坏或以不一致的状态被观察到。

## A Synchronized Class Example

```java
public class SynchronizedRGB {

    // Values must be between 0 and 255.
    private int red;
    private int green;
    private int blue;
    private String name;

    private void check(int red,
                       int green,
                       int blue) {
        if (red < 0 || red > 255
            || green < 0 || green > 255
            || blue < 0 || blue > 255) {
            throw new IllegalArgumentException();
        }
    }

    public SynchronizedRGB(int red,
                           int green,
                           int blue,
                           String name) {
        check(red, green, blue);
        this.red = red;
        this.green = green;
        this.blue = blue;
        this.name = name;
    }

    public void set(int red,
                    int green,
                    int blue,
                    String name) {
        check(red, green, blue);
        synchronized (this) {
            this.red = red;
            this.green = green;
            this.blue = blue;
            this.name = name;
        }
    }

    public synchronized int getRGB() {
        return ((red << 16) | (green << 8) | blue);
    }

    public synchronized String getName() {
        return name;
    }

    public synchronized void invert() {
        red = 255 - red;
        green = 255 - green;
        blue = 255 - blue;
        name = "Inverse of " + name;
    }
}
```

必须小心使用SynchronizedRGB，以避免在不一致的状态下被看到。例如，假设一个线程执行以下代码：

```java
ynchronizedRGB color =
    new SynchronizedRGB(0, 0, 0, "Pitch Black");
...
int myColorInt = color.getRGB();      //Statement 1
String myColorName = color.getName(); //Statement 2
```

另外一个线程在语句1和语句2之间调用color.set，那么myColorInt和myColorName不匹配。

```java
synchronized (color) {
    int myColorInt = color.getRGB();
    String myColorName = color.getName();
} 
```

为了避免这种情况，加锁。

```java
synchronized (color) {
    int myColorInt = color.getRGB();
    String myColorName = color.getName();
} 
```

## A Strategy for Defining Immutable Objects

* 不提供setter。
* 每个字段都定义为`final`以及`private`
* 不允许子类重写方法。最简单的方法是将类定义为`final`。更复杂的方法是使构造函数私有化并在工厂方法中构造实例。
* 如果实例字段包含对可变对象的引用，则不允许更改这些对象
  * 不提供方法去改变这些可变对象。
  * 不要共享对可变对象的引用。切勿存储传递给构造函数的外部、可变对象的引用；如果需要，请创建副本并存储对副本的引用。同样，在必要时创建内部可变对象的副本，以避免在方法中返回原始对象。

```java
final public class ImmutableRGB {

    // Values must be between 0 and 255.
    final private int red;
    final private int green;
    final private int blue;
    final private String name;

    private void check(int red,
                       int green,
                       int blue) {
        if (red < 0 || red > 255
            || green < 0 || green > 255
            || blue < 0 || blue > 255) {
            throw new IllegalArgumentException();
        }
    }

    public ImmutableRGB(int red,
                        int green,
                        int blue,
                        String name) {
        check(red, green, blue);
        this.red = red;
        this.green = green;
        this.blue = blue;
        this.name = name;
    }


    public int getRGB() {
        return ((red << 16) | (green << 8) | blue);
    }

    public String getName() {
        return name;
    }

    public ImmutableRGB invert() {
        return new ImmutableRGB(255 - red,
                       255 - green,
                       255 - blue,
                       "Inverse of " + name);
    }
}
```

# High Level Concurrency Objects

​	到目前为止，只介绍了比较低级的API。这些API对非常基础的任务来说足够了，但是，更高级的任务需要更高层次的构建块。

## Lock Objects

​	同步代码块展示了了非常简单的一种重入锁。这个锁易用，但是有很多限制。java.util.concurrent.lock包支持更多复杂的锁。不会很深入和详细，把注意力放在基础interface, `Lock`上。

​	锁对象的工作方式非常类似于同步代码使用的隐式锁。与隐式锁一样，一次只能有一个线程拥有一个锁对象。锁对象还通过其关联的条件对象支持等待/通知机制。

​	与隐式锁相比，Lock对象的最大优点是它们能够退出获取锁的尝试。如果锁不能立即使用或在超时过期（如果指定）之前，tryLock方法将退出。如果另一个线程在获取锁之前发送了一个中断，那么lockInterruptly方法将退出。

​	可以用Lock对象去解决死锁问题。

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.Random;

public class Safelock {
    static class Friend {
        private final String name;
        private final Lock lock = new ReentrantLock();

        public Friend(String name) {
            this.name = name;
        }

        public String getName() {
            return this.name;
        }

        public boolean impendingBow(Friend bower) {
            Boolean myLock = false;
            Boolean yourLock = false;
            try {
                myLock = lock.tryLock();
                yourLock = bower.lock.tryLock();
            } finally {
                if (! (myLock && yourLock)) {
                    if (myLock) {
                        lock.unlock();
                    }
                    if (yourLock) {
                        bower.lock.unlock();
                    }
                }
            }
            return myLock && yourLock;
        }
            
        public void bow(Friend bower) {
            if (impendingBow(bower)) {
                try {
                    System.out.format("%s: %s has"
                        + " bowed to me!%n", 
                        this.name, bower.getName());
                    bower.bowBack(this);
                } finally {
                    lock.unlock();
                    bower.lock.unlock();
                }
            } else {
                System.out.format("%s: %s started"
                    + " to bow to me, but saw that"
                    + " I was already bowing to"
                    + " him.%n",
                    this.name, bower.getName());
            }
        }

        public void bowBack(Friend bower) {
            System.out.format("%s: %s has" +
                " bowed back to me!%n",
                this.name, bower.getName());
        }
    }

    static class BowLoop implements Runnable {
        private Friend bower;
        private Friend bowee;

        public BowLoop(Friend bower, Friend bowee) {
            this.bower = bower;
            this.bowee = bowee;
        }
    
        public void run() {
            Random random = new Random();
            for (;;) {
                try {
                    Thread.sleep(random.nextInt(10));
                } catch (InterruptedException e) {}
                bowee.bow(bower);
            }
        }
    }
            

    public static void main(String[] args) {
        final Friend alphonse =
            new Friend("Alphonse");
        final Friend gaston =
            new Friend("Gaston");
        new Thread(new BowLoop(alphonse, gaston)).start();
        new Thread(new BowLoop(gaston, alphonse)).start();
    }
}
```

## Executors

​	在前面所有的例子中，由一个新线程完成的任务（由它的Runnable对象定义）和线程本身（由thread对象定义）之间有着密切的联系。这对于小型应用程序很有效，但是在大型应用程序中，将线程管理和创建与应用程序的其余部分分开是有意义的。

### Executor Interfaces

`java.util.concurrent`定义了3个executor interfaces:

* `Executor`, 启动新任务的简单接口。
* `ExecutorSevice`, `Executor`的子接口，添加了帮助管理生命周期的特性，包括任务和执行者本身。
* `ScheduledExecutorService`, `ExecutorService`子接口，支持未来和/或定期执行任务。

#### The `Executor` Interface

​	它提供了`execute`方法。

 replace

```
(new Thread(r)).start();
```

with

```
e.execute(r);
```

​	它更可能使用现有的工作线程来运行r，或者将r放入队列中等待工作线程可用。

#### The `ExecutorService` Interface

​	`ExecutorService` 额外提供了一个`submit`方法。`submit`方法支持接收`Runnable`对象，也支持`Callable`对象。其中`Callable`对象允许任务返回值。`submit`方法返回一个`Future`对象。用于检索可调用的返回值，并管理可调用和可运行任务的状态。

​	ExecutorService还提供了提交大型`Callable`对象集合的方法。最后，ExecutorService提供了许多方法来管理executor的关闭。为了支持立即关闭，任务应该正确处理中断。

#### The `ScheduledExecutorService` Interface

`ScheduledExecutorService` 有额外提供了`schedule`方法，可以在指定的延迟后执行`Runnable`或者`Callable`任务。此外，该接口定义scheduleAtFixedRate和scheduleWithFixedDelay，它们以定义的间隔重复执行指定的任务。

### Thread Pools

​	在java.util.concurrent中的大多数executor实现使用线程池。

https://blog.csdn.net/weixin_40682142/article/details/88313860

**FixedThreadPool**：这类线程池的特点就是里面全是核心线程，没有非核心线程，也没有超时机制，任务大小也是没有限制的，数量固定，即使是空闲状态，线程不会被回收，除非线程池被关闭，从构造方法也可以看出来，只有两个参数，一个是指定的核心线程数，一个是线程工厂，keepAliveTime无效。任务队列采用了无界的阻塞队列LinkedBlockingQueue，执行execute方法的时候，运行的线程没有达到corePoolSize就创建核心线程执行任务，否则就阻塞在任务队列中，有空闲线程的时候去取任务执行。由于该线程池线程数固定，且不被回收，线程与线程池的生命周期同步，所以适用于任务量比较固定但耗时长的任务。

**CachedThreadPool**：这类线程池的特点就是里面没有核心线程，全是非核心线程，其maximumPoolSize设置为Integer.MAX_VALUE，线程可以无限创建，当线程池中的线程都处于活动状态的时候，线程池会创建新的线程来处理新任务，否则会用空闲的线程来处理新任务，这类线程池的空闲线程都是有超时机制的，keepAliveTime在这里是有效的，时长为60秒，超过60秒的空闲线程就会被回收，当线程池都处于闲置状态时，线程池中的线程都会因为超时而被回收，所以几乎不会占用什么系统资源。任务队列采用的是SynchronousQueue，这个队列是无法插入任务的，一有任务立即执行，所以CachedThreadPool比较适合任务量大但耗时少的任务。
**ScheduledThreadPool**：这类线程池核心线程数量是固定的，好像和FixThreadPool有点像，但是它的非核心线程是没有限制的，并且非核心线程一闲置就会被回收，keepAliveTime同样无效，因为核心线程是不会回收的，当运行的线程数没有达到corePoolSize的时候，就新建线程去DelayedWorkQueue中取ScheduledFutureTask然后才去执行任务，否则就把任务添加到DelayedWorkQueue，DelayedWorkQueue会将任务排序，按新建一个非核心线程顺序执行，执行完线程就回收，然后循环。任务队列采用的DelayedWorkQueue是个无界的队列，延时执行队列任务。综合来说，这类线程池适用于执行定时任务和具体固定周期的重复任务。
**SingleThreadPool**：这类线程池顾名思义就是一个只有一个核心线程的线程池，从构造方法来看，它可以单独执行，也可以与周期线程池结合用。其任务队列是LinkedBlockingQueue，这是个无界的阻塞队列，因为线程池里只有一个线程，就确保所有的任务都在同一个线程中顺序执行，这样就不需要处理线程同步的问题。这类线程池适用于多个任务顺序执行的场景。

**WorkStealingPool**： 1.8新引入, 它是一个并行的线程池，参数中传入的是一个线程并发的数量，这里和之前就有很明显的区别，前面4种线程池都有核心线程数、最大线程数等等，而这就使用了一个并发线程数解决问题。

构造方法：

//其它几个构造方法都会调用这个构造方法

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

参数含义：

- **corePoolSize**：核心线程数量（最少线程数），在没有用的时候，也不会被回收
- **maximumPoolSize**：线程池中可以容纳的最大线程的数量
- **keepAliveTime**：程池中非核心线程最长可以保留的时间
- **util**：非核心线程最长可以保留的时间的单位（TimeUnit的DAYS、HOURS、MINUTES、SECONDS、MILLISECONDS、MICROSECONDS、NANOSECONDS）
- **workQueue**：等待队列，任务可以储存在任务队列中等待被执行
- **handler**：线程池的拒绝策略

常用的几种**BlockingQueue**接口的实现类：

- **ArrayBlockingQueue**（inti）:基于数组的阻塞队列实现，ArrayBlockingQueue内部维护了一个定长的数组，以便缓存队列中的数据对象，其内部**没实现读写分离**，也就意味着生产和消费者不能完全并行。长度是需要定义的，可以指定先进先出或者先进后出，因为长度是需要定义的，所以也叫**有界队列**，在很多场合非常适合使用。
- **LinkedBlockingQueue**（）或者（int i）:基于链表的阻塞队列，同ArrayBlockingQueue类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），LinkedBlockingQueue之所以能够高效地处理并发数据，是因为其内部实现采用分离锁（**读写分离两个锁**），从而实现生产者和消费者操作完全并行运行。
- **PriorityBlockingQueue**（）或者（int i）:基于优先级别的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定，也就是说传入队列的对象必须实现Comparable接口），在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁。
- **SynchronousQueue**（）:一种没有缓冲的队列，生产者产生的数据直接会被消费者获取并且立刻消费。`SynchronousQueue`是一个内部只能包含一个元素的队列。插入元素到队列的线程被阻塞，直到另一个线程从队列中获取了队列中存储的元素。同样，如果线程尝试获取元素并且当前不存在任何元素，则该线程将被阻塞，直到线程将元素插入队列。（https://www.jianshu.com/p/d5e2e3513ba3）

- **DelayQueue**：带有延迟时间的Queue，其中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue中的元素必须先实现**Delayed**接口，DelayQueue是一个没有大小限制的队列，应用场景很多，比如对缓存超时的数据进行移除、任务超时处理、空闲连接的关闭等等。

**线程池的四种拒绝策略：**（ThreadPoolExecutor的静态内部类）

- **AbortPolicy**：不执行新任务，直接抛出异常，提示线程池已满。**默认策略**
- **DisCardPolicy**：不执行新任务，也不抛出异常。
- **DisCardOldSetPolicy**：将消息队列中的第一个任务替换为当前新进来的任务执行。
- **CallerRunsPolicy**：直接在 execute 方法的调用线程中运行被拒绝的任务；如果执行程序已关闭，则会丢弃该任务。

**线程池工厂：**（Executors的静态内部类）
**DefaultThreadFactory**：默认缺省线程工厂，规定好了前缀名，不能设置守护线程。
PrivilegedThreadFactory： DefaultThreadFactory的子类，与class loader有关，后面再研究？？？？？？

**线程池调优：**
1、设置最大线程数和最小线程数。
2、选择合适的任务队列和拒绝策略。

**在尝试获得最好的性能时，可以应用KISS原则"Keep it simple，stupid"。可以将最小线程数和最大线程数设置为相同。
在任务队列方面，如果适合无界队列，则选择LinkedBlockingQueue；如果适合有界队列，则选择ArrayBlockingQueue**

线程池必须需仔细调优，盲目的向池中添加新线程，在某些情况下对性能会有不利的影响。
使用ThreadPoolExecutor时，选择更简单选项通常会带来最好的、最能预见的性能

### Fork/Join

​	fork/join框架是ExecutorService接口的实现，它可以帮助您利用多进程。它是为可以递归地分解成更小的块的工作而设计的。目标是使用所有可用的处理能力来增强应用程序的性能。

​	fork/join框架将任务分配给线程池中的工作线程。fork/join框架是不同的，因为它使用了一个工作窃取算法。工作线程用完了要做的事情，可以从其他仍在忙的线程中窃取任务。

​      fork/join框架的中心是[`	ForkJoinPool`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html)类，继承自`AbstractExecutorService`。ForkJoinPool实现核心工作窃取算法，可以执行ForkJoinTask进程。

#### Basic Use

​	使用fork/join编写框架的第一步是执行代码的第一步。您的代码应该类似于以下伪代码：	

```java
if (my portion of the work is small enough)
  do the work directly
else
  split my work into two pieces
  invoke the two pieces and wait for the results
```

​	将此代码包装在ForkJoinTask子类中，通常使用它的一个更专门的类型，RecursiveTask（可以返回结果）或RecursiveAction。

​	在ForkJoinTask子类准备好之后，创建一个表示要完成的所有工作的对象，并将其传递给ForkJoinPool实例的invoke（）方法。

#### Blurring for Clarity

​	以下例子帮助理解`fork/join`框架的工作方式。

```java
public class ForkBlur extends RecursiveAction {
    private int[] mSource;
    private int mStart;
    private int mLength;
    private int[] mDestination;
  
    // Processing window size; should be odd.
    private int mBlurWidth = 15;
  
    public ForkBlur(int[] src, int start, int length, int[] dst) {
        mSource = src;
        mStart = start;
        mLength = length;
        mDestination = dst;
    }

    protected void computeDirectly() {
        int sidePixels = (mBlurWidth - 1) / 2;
        for (int index = mStart; index < mStart + mLength; index++) {
            // Calculate average.
            float rt = 0, gt = 0, bt = 0;
            for (int mi = -sidePixels; mi <= sidePixels; mi++) {
                int mindex = Math.min(Math.max(mi + index, 0),
                                    mSource.length - 1);
                int pixel = mSource[mindex];
                rt += (float)((pixel & 0x00ff0000) >> 16)
                      / mBlurWidth;
                gt += (float)((pixel & 0x0000ff00) >>  8)
                      / mBlurWidth;
                bt += (float)((pixel & 0x000000ff) >>  0)
                      / mBlurWidth;
            }
          
            // Reassemble destination pixel.
            int dpixel = (0xff000000     ) |
                   (((int)rt) << 16) |
                   (((int)gt) <<  8) |
                   (((int)bt) <<  0);
            mDestination[index] = dpixel;
        }
    }
  
  ...
```

然后实现`compute`方法。

```java
protected static int sThreshold = 100000;

protected void compute() {
    if (mLength < sThreshold) {
        computeDirectly();
        return;
    }
    
    int split = mLength / 2;
    
    invokeAll(new ForkBlur(mSource, mStart, split, mDestination),
              new ForkBlur(mSource, mStart + split, mLength - split,
                           mDestination));
}
```

​	如果前面的方法在RecursiveAction类的子类中，那么将任务设置为在ForkJoinPool中运行是很简单的，并且包括以下步骤:

1. 成创建任务：

   ```java
   // source image pixels are in src
   // destination image pixels are in dst
   ForkBlur fb = new ForkBlur(src, 0, src.length, dst);
   ```

2. 创建ForkJoinPool`ForkJoinPool pool = new ForkJoinPool();`

3. 运行任务：

   ```
   pool.invoke(fb);
   ```

see the [`ForkBlur`](https://docs.oracle.com/javase/tutorial/essential/concurrency/examples/ForkBlur.java) example. //todo

#### Standard Implementations

​	除了使用fork/join框架为要在多处理器系统上并发执行的任务实现自定义算法（例如ForkBlur.java版在上一节的示例中），javase中有一些通常有用的特性，这些特性已经使用fork/join框架实现了。JavaSE8中引入的一个这样的实现是由java.util.Arrays类的parallelSort（）方法。这些方法类似于sort（），但通过fork/join框架利用并发性。在多处理器系统上运行时，大数组的并行排序比顺序排序快。但是，这些方法究竟是如何利用fork/join框架的，不在Java教程的范围之内

​	另外一个实现是使用`java.util.streams`,see the [Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) section。

## Concurrent Collections

​	 `java.util.concurrent` 包包含了一些额外的Java集合。通过提供的集合接口可以很容易地对它们进行分类：

​	[`BlockingQueue`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html)：定义一个先进先出的数据结构，当您试图添加到完整队列或从空队列中检索时，该结构将阻塞或超时。

[`ConcurrentMap`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentMap.html)：是java.util.Map的子接口，它定义了有用的原子操作。这些操作仅在key存在时移除或替换键值对，或者仅在key不存在时添加键值对。使这些操作原子化有助于避免同步。ConcurrentMap的标准通用实现是ConcurrentHashMap，它是HashMap的并发模拟。

[`ConcurrentNavigableMap`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentNavigableMap.html) ：是ConcurrentMap的一个子接口，支持近似匹配。ConcurrentNavigableMap的标准通用实现是ConcurrentSkipListMap，它是TreeMap的并发模拟。

​	以上这些容器都有助于避免内存一致性错误。

## Atomic Variables

`java.util.concurrent.atomic`包定义了对单个变量进行原子性操作的类。

​	但是对于更复杂的类，我们可能希望避免不必要的同步对活跃度的影响。将int字段替换为AtomicInteger可以防止线程干扰，而不必求助于同步。

```java
import java.util.concurrent.atomic.AtomicInteger;

class AtomicCounter {
    private AtomicInteger c = new AtomicInteger(0);

    public void increment() {
        c.incrementAndGet();
    }

    public void decrement() {
        c.decrementAndGet();
    }

    public int value() {
        return c.get();
    }

}
```

## Concurrent Random Numbers

​	对于希望使用来自多个线程或ForkJoinTasks的随机数的应用程序， 可以使用[`ThreadLocalRandom`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadLocalRandom.html)。

​	对于并发访问，使用ThreadLocalRandom而不是`Math.random()`减少了争用，最终提高了性能。

```
	int r = ThreadLocalRandom.current() .nextInt(4, 77);
```

# For Further Reading

- *Concurrent Programming in Java: Design Principles and Pattern (2nd Edition)* by Doug Lea. A comprehensive work by a leading expert, who's also the architect of the Java platform's concurrency framework.
- *Java Concurrency in Practice* by Brian Goetz, Tim Peierls, Joshua Bloch, Joseph Bowbeer, David Holmes, and Doug Lea. A practical guide designed to be accessible to the novice.
- *Effective Java Programming Language Guide (2nd Edition)* by Joshua Bloch. Though this is a general programming guide, its chapter on threads contains essential "best practices" for concurrent programming.
- *Concurrency: State Models & Java Programs (2nd Edition)*, by Jeff Magee and Jeff Kramer. An introduction to concurrent programming through a combination of modeling and practical examples.
- *[Java Concurrent Animated](http://sourceforge.net/projects/javaconcurrenta/):* Animations that show usage of concurrency features.