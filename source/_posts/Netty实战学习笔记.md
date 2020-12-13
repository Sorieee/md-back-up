---
title: Netty实战学习笔记
date: 2020-11-02 17:51:00
tags: [Netty, java, 网络编程]
---

# 第1章 Netty——异步和事件驱动

主要内容：

* Java网络编程
* Netty简介
* Netty的核心组件。

> netty.io
>
> Netty是一款异步的时间驱动的网络应用程序框架，支持快速地开发可维护的高性能的面向协议的服务器和客户端。

## 1.1 Java网络编程

​	

```java
public abstract class BlockingIoExample {

    public void serve(int portNumber) throws IOException {
        ServerSocket serverSocket = new ServerSocket(portNumber);
        Socket clientSocket = serverSocket.accept();
        BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream()));
        PrintWriter out =
                new PrintWriter(clientSocket.getOutputStream(), true);
        String request, response;
        while ((request = in.readLine()) != null) {
            if ("Done".equals(request)) {
                break;
            }
        }
        response = processRequest(request);
        out.println(response);
    }

    protected abstract String processRequest(String request);
}
```

* ServerSocket的accept方法会阻塞到一个连接建立，随后返回一个新的Socket用于客户端和服务器之间的通信。
* BufferedReader和PrintWriter都衍生自socket的输入输出流。前者从一个字符输入流中读取文本，后者打印对象的格式化的表示到文本输出流。
* readLine()方法将会阻塞，直到有换行符或者回车符结尾的字符串被读取。
* 客户端请求已经被处理。

这段代码只能同时处理一个连接，要管理多个并发客户端，需要未每个新的客户端Socket创建一个新的Thread。

![](https://pic.imgdb.cn/item/5f9fd8d81cd1bbb86b75f2e2.jpg)

### 1.1.1  Java NIO

​	本地套接字库很早就提供了非阻塞调用。

* 可以使用setsocket方法配置套接字，以便读/写调用在没有数据的时候立即返回。
* 可以使用操作视同的事件通知API注册一组非阻塞套接字，已确定它们中是否有任何的套接字已经有数据可供读写。

![](https://pic.imgdb.cn/item/5f9fd9eb1cd1bbb86b768119.jpg)

### 1.1.2 选择器

![](https://pic.imgdb.cn/item/5f9fda1c1cd1bbb86b769263.jpg)

`java.nio.channels.Selector`是Java的非阻塞I/O实现的关键。

这种模型提供了更好的资源管理：

* 使用较少的线程可以处理许多连接，因此也可以减少内存管理和上下文切换所带来的开销。
* 当没有I/O操作需要处理时，线程也可以用于其他任务。

## 1.2 Netty简介

​	Netty特性如下：

![](https://pic.imgdb.cn/item/5f9fef161cd1bbb86b7b8fb5.jpg)

![](https://pic.imgdb.cn/item/5f9fef2d1cd1bbb86b7b9510.jpg)

### 1.2.1 谁在使用Netty

略

### 1.2.2 异步和事件驱动

​	异步代表在等待的同时可以做别的事。

​	异步和可伸缩性之间的联系：

* 非阻塞网络调用使得我们可以不必等待一个操作的完成。完全异步的I/O正是基于这个特性构建的:异步方法会立即返回，并在在它完成时，会直接或在稍后某个时间点通知用户。
* 选择器使得我们能够通过较少的线程便可监视许多连接上的时间。

## 1.3 Netty的核心组件

​	主要构件块：

* Channel;
* 回调;
* Future;
* 事件和ChannekHandler。

### 1.3.1 Channel

​	它是Java NIO的一个基本构造。

> 它代表一个实体的开放连接，如读操作和写操作。

​	可以把Channel看做是传入或传出数据的载体。因此它可以被打开或关闭，连接或断开连接。



### 1.3.2 回调

​	回调就是一个方法，一个指向已经被提供给另一个方法的方法引用。这使得后者可以在适当的时候调用前者。

![](https://pic.imgdb.cn/item/5f9ff2071cd1bbb86b7c2e28.jpg)

### 1.3.3 Future

​	它提供了另一种在操作完成时通知应用程序的方式。可以看做是一个异步操作结果的占位符, 它将在未来某个时刻完成，并提供其结果的访问。

​	JDK预支了interface java.util.concurrent.Future。但是只能手动检查操作是否完成，或者一直阻塞到它完成。Netty提供了自己的实现——ChannelFuture。

​	ChannelFutre提供了几种额外的方法，是的我们可以注册一个到多个ChannelFutureListener实例。监听器的回调方法operationComplete()，将会在操作完成时调用。

​	每个Netty的出站I/O操作都将返回一个ChannelFuture，意味着它们不会阻塞。

```java
public class ConnectExample {

    public static void connect(Channel channel) {
        // Does not block
        ChannelFuture future = channel.connect(
                new InetSocketAddress("192.168.0.1", 25));
        future.addListener(new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture future) {
            if (future.isSuccess()) {
                ByteBuf buffer = Unpooled.copiedBuffer(
                        "Hello", Charset.defaultCharset());
                ChannelFuture wf = future.channel().writeAndFlush(buffer);
                // ...
            } else {
                Throwable cause = future.cause();
                cause.printStackTrace();
            }
        }
        });

    }
}
```

​	如果把ChannelFutureListener看做回调的精细化版本是正确的。事实上，他们相互补充; 相互结合，构成了Netty本身的关键构建块之一。

### 1.3.4 事件和ChannelHandler

​	Netty使用不同事件来通知我们状态的改变或者操作的状态：

* 记录日志;
* 数据转换;
* 流控制;
* 应用程序逻辑。



​	Netty是一个网络编程框架，所以事件是按照他们的入站或出站的数据流的相关性进行分类的。

​	入站事件：

* 连接已被激活或者连接失活;
* 数据读取;
* 用户事件;
* 错误事件。



​	出站事件包括:

* 打开或关闭到远程节点的连接;
* 将数据写到或冲刷到套接字。

![](https://pic.imgdb.cn/item/5f9ffa151cd1bbb86b7e5c1d.jpg)

### 1.3.5 把它们放在一起

#### 1. Future、回调和ChannelHandler

​	Netty异步变成模型建立在Future和回调纸上，将事件分派到ChannelHandler的方法在更深层次。结合在一起，提供了一个处理环境，使得应用程序可以独立于任何网络操作相关的顾虑而独立地演变。

#### 2. 选择器、事件和EventLoop

​	通过触发时间将Selector从程序中抽象出来，消除派发代码。在内部，为每个Channel分配一个EventLoop，用以处理所有事件：

* 注册感兴趣的事件;
* 将事件派发给ChannelHandler;
* 安排进一步的动作。



​	EventLoop本身只由一个线程驱动, 其处理了一个Channel的所有I/O事件, 并且在该EventLoop的整个生命周期内都不会发生改变。

## 第2章 你的第一款Netty应用程序

主要内容：

* 设置开发环境
* 编写Echo服务器和客户端
* 构建并测试应用程序

## 2.1 设置开发环境

​	JDK和Apache Maven

### 2.2.1 获取并安装Java开发工具包

​	略

### 2.1.2 下载并安装IDEA

​	略

### 2.1.3 下载并安装Apache Maven

​	略。

### 2.1.4 配置工具集

​	略

## 2.2 Netty客户端/服务器概览

​	![](https://pic.imgdb.cn/item/5f9ffe9f1cd1bbb86b7f550a.jpg)

## 2.3 编写Echo服务器

​	都需要以下两部分：

* 至少一个ChannelHandler——该组件实现了服务器对客户端接收的数据的处理，即业务逻辑。
* 引导——配置服务器的启动代码。至少会将服务器绑定到它要监听连接请求的端口上。

### 2.3.1 ChannelHandler和业务逻辑

因为Echo服务器会响应传入的消息，所以需要实现`ChannelInboundHandler`接口，来定义响应入站事件的方法。`ChannelInboundHandlerAdapter`提供了上述接口的默认实现。

​	感兴趣的方法是:

* channelRead()——对于每个传入的消息都要调用;
* channelReadComplete()——通知`ChannelInboundHandler`最后一次channelRead()调用时当前批量读取中的最后一条消息;
* exceptionCaught()——在读取操作期间，有异常抛出时会调用。

```java
@Sharable
public class EchoServerHandler extends
        ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx,
        Object msg) {
        ByteBuf in = (ByteBuf) msg;
        System.out.println("Server received: " + in.toString(CharsetUtil.UTF_8));
        ctx.write(in);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,
        Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

> **如果不捕获异常，会发生什么？**
>
> 