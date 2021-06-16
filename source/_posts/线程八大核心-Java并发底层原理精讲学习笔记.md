---
title: 线程八大核心+Java并发底层原理精讲学习笔记
date: 2020-05-30 18:04:45
tags: [java，并发]
---

学习视频：https://coding.imooc.com/learn/list/362.html

思维导图：https://naotu.baidu.com/file/89fb28b05e3395800f9dc2d332d2b198?token=9b45e08e55281667

八大核心基础思维导图：https://naotu.baidu.com/file/07f437ff6bc3fa7939e171b00f133e17?token=6744a1c6ca6860a0

内存模型思维导图：https://naotu.baidu.com/file/60a0bdcaca7c6b92fcc5f796fe6f6bc9?token=bcdbae34bb3b0533

死锁内存导图：https://naotu.baidu.com/file/ec7748c253f4fc9d88ac1cc1e47814f3?token=bb71b5895a747d67

# 实现多线程的正确姿势

## 如何创建新线程

有两种方法可以创建新的执行线程。一种是将类声明为Thread子类，重写run方法。

一种是实现Runnable的类，将该类的实例作为参数传递给Thread

```java
/***
 * 用Runnable创建线程
 */
public class RunnableStyle implements Runnable {
    public static void main(String[] args) {
        Thread thread = new Thread(new RunnableStyle());
        thread.start();
    }

    @Override
    public void run() {
        System.out.println("runnable创建");
    }
}
```

```java
/***
 * 用Thread继承方式去创建线程
 */
public class ThreadStyle extends Thread {
    public static void main(String[] args) {
        ThreadStyle threadStyle = new ThreadStyle();
        threadStyle.start();
    }

    @Override
    public void run() {
        System.out.println("Thread创建");
    }
}
```

## 两种方法的区别

runnable接口方法更好。

线程应该和执行的代码解耦

继承Thread每次要新开任务都只能去新建一个独立的线程，损耗比较大，Runnable可以使用线程池。

继承了Thread无法继承其他类。

本质区别：

Thread::run

```java
public void run() {
    if (target != null) {
        target.run();
    }
}
```

runable接口是执行targetRun

Thread是重写这个代码。

## 同时用两种方法会怎么样？

因为覆盖了run，所以只会输出Thread

```java
public class BothRunnableThreadStyle  {

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("runnable");
            }
        }) {
            @Override
            public void run() {
                System.out.println("Thread");
            }
        }.start();
    }
}
```



# 启动线程的正确姿势

应该用start，run直接是在main线程执行。

# 线程停止

### 如何正确停止线程？

使用interrupt来通知，而不是强制。



通常线程会在什么情况下停止（普通情况）？

run方法运行完毕

未捕获的异常



### 普通中断方式

```java
public class RightWayStopThreadWithoutSleep implements Runnable {
    @Override
    public void run() {
        int num = 0;
        while (!Thread.currentThread().isInterrupted() && num <= Integer.MAX_VALUE / 2) {
            if (num % 10000 == 0) {
                System.out.println(num);
            }
            num++;
        }
        System.out.println("end");
    }

    public static void main(String[] args) throws InterruptedException{
        Thread thread = new Thread(new RightWayStopThreadWithoutSleep());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}
```



### 阻塞线程的中断

```java
public class RightWayStopThreadWithSleep {


    public static void main(String[] args) throws InterruptedException{
        Runnable runnable = () -> {
            int num = 0;
            while (!Thread.currentThread().isInterrupted() && num <= 300) {
                if (num % 100 == 0) {
                    System.out.println(num);
                }
                num++;
            }
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("end");

        };
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(500);
        thread.interrupt();
    }
}
output:
0
100
200
300
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at cn.sorie.threadcore.createthreads.stopthread.RightWayStopThreadWithSleep.lambda$main$0(RightWayStopThreadWithSleep.java:19)
	at java.lang.Thread.run(Thread.java:745)
end
```



### 每次迭代后都阻塞

不需要每次都去检测Thread.currentThread().isInterrupted()

```java
public class RightWayStopThreadWithSleep {


    public static void main(String[] args) throws InterruptedException{
        Runnable runnable = () -> {
            int num = 0;
            while (!Thread.currentThread().isInterrupted() && num <= 300) {
                if (num % 100 == 0) {
                    System.out.println(num);
                }
                num++;
            }
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("end");

        };
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(500);
        thread.interrupt();
    }
}
```

### 自动清除中断信号

线程一旦响应中断异常，就会清除掉中断标记位。

```java
public class CanInterrupt {
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
            int num = 0;
            while (num <= 10000 && !Thread.currentThread().isInterrupted()) {
                if (num%100 == 0) {
                    System.out.println(num + "是100的倍数");
                }
                num++;
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(5000);
        thread.interrupt();

    }
}
output:
0是100的倍数
100是100的倍数
200是100的倍数
300是100的倍数
400是100的倍数
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at cn.sorie.threadcore.stopthread.CanInterrupt.lambda$main$0(CanInterrupt.java:13)
	at java.lang.Thread.run(Thread.java:745)
500是100的倍数
600是100的倍数
```

### 最佳实践

* 优先选择：传递中断
* 不想或无法传递：恢复中断
* 不应屏蔽中断。

#### 传递中断

错误方式：忽略了中断

```java
/***
 * catch住了InterruptedException之后，应该优先抛出该异常
 */
public class RightWayStopThreadInProd implements Runnable{
    @Override
    public void run() {
        while (true) {
            System.out.println("go");
            throwInMethod();
        }
    }

    private void throwInMethod() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException{
        Thread thread = new Thread(new RightWayStopThreadInProd());
        thread.start();
        thread.sleep(1000);
        thread.interrupt();
    }
}
output:
go
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at cn.sorie.threadcore.stopthread.RightWayStopThreadInProd.throwInMethod(RightWayStopThreadInProd.java:17)
	at cn.sorie.threadcore.stopthread.RightWayStopThreadInProd.run(RightWayStopThreadInProd.java:11)
	at java.lang.Thread.run(Thread.java:745)
go
go
go
go
go
```

run方法无法抛出异常

![](https://pic.imgdb.cn/item/5ed90909c2a9a83be59ac506.jpg)



正确方式：应该抛出给最顶层的方法去做处理。

```java
/***
 * catch住了InterruptedException之后，应该优先抛出该异常
 */
public class RightWayStopThreadInProd implements Runnable{
    @Override
    public void run() {
        while (true && !Thread.currentThread().isInterrupted()) {
            System.out.println("go");
            try {
                throwInMethod();
            } catch (InterruptedException e) {
                System.out.println("保存日志");
                e.printStackTrace();
            }
        }
    }

    private void throwInMethod() throws InterruptedException{

        Thread.sleep(2000);

    }

    public static void main(String[] args) throws InterruptedException{
        Thread thread = new Thread(new RightWayStopThreadInProd());
        thread.start();
        thread.sleep(1000);
        thread.interrupt();
    }
}
```

#### 恢复中断

```java
/***
 * 描述： 最佳实践2：在catch子语句中调用Thread.currentThread().interrupt()来恢复设置中断状态，以便于在后续的执行中，
 * 依然能够检查到刚才发生了的中断
 *  回到刚才RightWayStopThreadInProd2补上中断，让它跳出
 */
public class RightWayStopThreadInProd2 implements Runnable{
    @Override
    public void run() {
        while (true) {
            if (Thread.currentThread().isInterrupted()) {
                System.out.println("Interrupted，程序运行结束");
                break;
            }
            reInterrupt();
        }
    }

    private void reInterrupt() {

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            e.printStackTrace();
        }

    }

    public static void main(String[] args) throws InterruptedException{
        Thread thread = new Thread(new RightWayStopThreadInProd2());
        thread.start();
        thread.sleep(1000);
        thread.interrupt();
    }
}
```