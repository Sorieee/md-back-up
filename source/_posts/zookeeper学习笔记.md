---
title: zookeeper学习笔记
date: 2020-12-13 14:38:16
tags: [zookeeper]
---

# 《Zookeeper: 分布式过程协同技术详解》

# 第1章 简介

​	ZooKeeper是一种用于分布式应用程序的高性能协调服务。它在一个简单的界面中公开了常见的服务，如命名、配置管理、同步和组服务，这样您就不必从头开始编写它们了。您可以使用它来实现共识、组管理、领导人选举和出席协议。你可以根据自己的具体需要来构建它。

​	要实现主-从模式的系统，必须解决以下三个关键问题：

* 主节点崩溃。
* 从节点崩溃。
* 通信故障。



​	可以得到主-从架构的需求:

* 主节点枚举。
* 崩溃检测。
* 组成员关系管理。
* 元数据管理。



**注意：拜占庭将军问题**

​	拜 占 庭 将军 问题（ Byzantine Faults） 是指 可能 导致 一个 组件 发生 任意 行为（ 常常 是 意料之外 的） 的 故障。 这个 故障 的 组件 可能 会破 坏 应用 的 状态， 甚至 是 恶意 行为。 系统 是 建立 在 假设 会 发生 这些 故障， 需要 更高 程度 的 复制 并使 用 安全 原 语 的 基础上。 尽管 我们 从 学术 文献 中 知道， 针对 拜 占 庭 将军 问题 技术 发展 已经 取得 了 巨大 进步， 我们 还是 觉得 没有 必要 在 ZooKeeper 中 采用 这些 技术， 因此， 我们 也 避免 代码 库 中 引入 额外 的 复杂性。

​	CAP表示一致性，可用性和分区容错性。没有一个系统同时满足这三种属性。Zookeeper的设计尽可能满足一致性和可用性。在发生了网络分区时Zookeeper也停工了只读能力。

# 第2章 了解Zookeeper

## Zookeeper基础

​	很多设计一个用于协作需求的方法是提供原语列表，暴露出每个原语的实例化调用方法，并直接控制这些事例。比如可以说分布式锁机制组成了一个重要的原语，同时暴露出创建、获取和释放三个调用方法。

​	这种 设计 存在 一些 重大 的 缺陷： 首先， 我们 要么 预先 提出 一份 详尽 的 原 语 列表， 要么 提供 API 的 扩展， 以便 引入 新的 原 语； 其次， 以 这种 方式 实现 原 语 的 服务 使得 应用 丧失了 灵活性。

​	ZooKeeper 并不 直接 暴露 原 语， 取而代之， 它 暴露 了 由 一 小部分 调用 方法 组成 的 类似 文件 系统 的 API， 以便 允许 应用 实现 自己的 原 语。 我们 通常 使用 菜谱（ recipes） 来 表示 这些 原 语 的 实现。 菜谱 包括 ZooKeeper 操作 和 维护 一个 小型 的 数据 节点， 这些 节点 被称为 znode， 采用 类似于 文件 系统 的 层级 树状 结构 进行 管理。

![](https://pic.imgdb.cn/item/5fd5bdcd3ffa7d37b33a97a5.jpg)

​	针对 一个 znode， 没有 数据 常常 表达 了 重要的 信息。 比如， 在 主- 从 模式 的 例子 中， 主 节点 的 znode 没有 数据， 表示 当前 还没有 选 举出 主 节点。 而 图 中 涉及 的 一些 其他 znode 节 点在 主- 从 模式 的 配置 中非 常有 用： 

* /workers 节点 作为 父 节点， 其下 每个 znode 子 节点 保存 了 系统 中 一个 可用 从 节点 信息。 如图 2- 1 所示， 有一个 从 节点（ foot. com： 2181）。 
* /tasks 节点 作为 父 节点， 其下 每个 znode 子 节点 保存 了 所有 已经 创建 并 等待 从 节点 执行 的 任务 的 信息， 主- 从 模式 的 应用 的 客户 端 在/ tasks 下 添加 一个 znode 子 节点， 用来 表示 一个 新任务， 并 等待 任务 状态 的 znode 节点。
* /assign 节点 作为 父 节点， 其下 每个 znode 子 节点 保存 了 分配 到 某个 从 节点 的 一个 任务 信息， 当 主 节点 为某个从节点分配了一个任务，就会在/assign下增加一个子节点。

### API概述

​	ZooKeeper 的 API 暴露 了 以下 方法： 

​	**create/path data** 

​	创建 一个 名为/path 的 znode 节点， 并 包含 数据 data。 

​	**delete/path** 

​	删除 名为/path 的 znode。 

​	**exists/path** 

​	检查 是否 存在 名为/path 的 节点。 

​	**setData/path data** 

​	设置 名为/path 的 znode 的 数据 为 data。 

​	**getData/path** 

​	返回 名为/path 节点 的 数据 信息。 

​	**getChildren/path**

​	返回 所有/ path 节点 的 所有 子 节点 列表。

### znode的不同类型

​	新建znode的时候，还需要指定该结点的类型

**持久节点和临时节点**

​	持久的节点，只能调用delete删除。临时znide与之相反，当创建该节点的客户端崩溃或关闭了与ZooKeeper的连接时，阶段就会被删除。

**有序节点**

​	一个 znode 还可以 设置 为 有序（ sequential） 节点。 一个 有序 znode 节点 被 分配 唯一 个 单调 递增 的 整数。 当 创建 有序 节点 时， 一个 序号 会被 追加 到 路径 之后。 例如， 如果 一个 客户 端 创建 了 一个 有序 znode 节点， 其 路径 为/ tasks/ task-， 那么 ZooKeeper 将会 分配 一个 序号， 如 1， 并将 这个 数字 追加 到 路径 之后， 最后 该 znode 节点 为/ tasks/ task- 1。 有序 znode 通过 提供 了 创建 具有 唯一 名称 的 znode 的 简单 方式。 同时 也 通过 这种 方式 可以 直观 地 查看 znode 的 创建 顺序。

​		总之， znode 一 共有 4 种类 型： 持久 的（ persistent）、 临时 的（ ephemeral）、 持久 有序 的（ persistent_ sequential） 和 临时 有序 的（ ephemeral_ sequential）。

### 监视与通知

​	为了替换客户端轮询，ZooKeeper选择了基于通知的机制：客户端向ZooKeeper注册需要接受通知的znode，通过对znode设置监视点来接受通知。监视点是一个单词触发的操作，即监视点会触发一个通知。为了接受多个通知，客户端必须在每次通知后设置一个新的监视点。

![](https://pic.imgdb.cn/item/5fd5c1cc3ffa7d37b33edf4e.jpg)

​	通知 机制 的 一个 重要 保障 是， 对 同一个 znode 的 操作， 先向 客户 端 传送 通知， 然 后再 对 该 节点 进行 变更。 如果 客户 端 对 一个 znode 设置 了 监视 点， 而 该 znode 发生了 两个 连续 更新。 第一次 更新 后， 客户 端 在 观察 第二次 变化 前 就 接收 到了 通知， 然后 读取 znode 中的 数据。 我们 认为 主要 特性 在于 通知 机制 阻止 了 客户 端 所 观察 的 更新 顺序。 虽然 ZooKeeper 的 状态 变化 传播 给 某些 客户 端 时 更 慢， 但我 们 保障 客户 端 以 全局 的 顺序 来 观察 ZooKeeper 的 状态。

###  版本

​	每个znode都有一个版本号，随着每次变化自增。用于解决缓存不一致的情况。

## ZooKeeper架构

​	服务端运行于两种模式下：独立模式(standalone)和仲裁模式(quorum)。独立模式：有一个单独的服务器，ZooKeeper状态无法复制。在仲裁模式下，具有一组ZooKeeper服务器，我们称为ZooKeeper集合(ZooKeeper ensemble), 它们之间可以进行状态的复制，并同时为服务于客户端的请求。

![](https://pic.imgdb.cn/item/5fd5c65d3ffa7d37b344d8aa.jpg)

### ZooKeeper仲裁

​	客户端不必等待每个服务器完成数据保护后再继续，只要满足保存客户端数据的服务器的最小个数即可。

​	桶过 使用 多数 方案， 我们 就可以 容许 f 个 服务器 的 崩溃， 在这里， f 为 小于 集合 中 服务器 数量 的 一半。 例如， 如果 有 5 个 服务器， 可以 容许 最多 f= 2 个 崩溃。 在 集合 中， 服务器 的 个数 并不是 必须 为 奇数， 只是 使用 偶数 会使 得 系统 更加 脆弱。 假设 在 集合 中 使用 4 个 服务器， 那么 多数 原则 对应 的 数量 为 3 个 服务器。 然而， 这个 系统 仅能 容许 1 个 服务器 崩溃， 因为 两个 服务器 崩溃 就会 导致 系统 失去 多数 原则 的 状态。 因此， 在 4 个 服务器 的 情况下， 我们 仅能 容许 一个 服务器 崩溃， 而 法定人数 现在 却 更大， 这 意味着 对 每个 请求， 我们 需要 更多 的 确认 操作。 底线 是我 们 需要 争取 奇 数个 服务器。

### 会话

​	在执行任何请求前，必须先建立会话。会话提供了顺序保障，同一个会话的请求会以FIFO顺序执行。通常一个客户端只能打开一个会话。

## 开始使用Zookeeper

​	下载zookeeper。http://zookeeper.apache.org

​	用命令解压：

```sh
tar zxvf apache-zookeeper-3.6.2-bin.tar.gz
```

### 第一个ZooKeeper会话

​	以独立模式运行一个会话。首先重命名配置文件。

```sh
 mv conf/zoo_sample.cfg conf/zoo.cfg
```

​	最好把data目录移除/temp目录，以防止ZooKeeper填满根分区。在zoo.cfg中修改这个目录的位置。

```sh
dataDir=/usr/sorie/zookeeper
```

​	最后，启动服务器.

```sh
bin/zkServer.sh start
```

​	如果想查看服务器输出，可以运行如下命令

```sh
bin/zkServer.sh start-foreground
```

​	在另一个shell进入项目根目录，运行以下命令

```sh
bin/zkCli.sh
```

1. 客户 端 启动 程序 来 建立 一个 会话。 
2. 客户 端 尝试 连接 到 localhost/ 127. 0. 0. 1： 2181。
3. 客户 端 连接 成功， 服务器 开始 初始化 这个 新 会话。
4. 会话 初始化 成功 完成。 
5.  服务器 向 客户 端 发送 一个 SyncConnected 事件。



​	我们列出根节点下所有的znode,然后创建一个znode。首先要确认znode树为空。处理/zookeeper之外，该节点标记了ZookKeeper服务所需的元数据树。

```sh
$ ls /
[zookeeper]
$ create /workers ""
create / workers ""
$ ls / 
[workers, zookeeper]
```

​	创建`/workers`节点后，指定了一个空字符串，说明我们此刻不希望在这个znode中保存数据。

​	删除znode然后退出。

```SH
$ delete /workers
$ ls /
$ quit
$ bin/zkServer.sh stop
```

### 会话状态和生命周期

![](https://pic.imgdb.cn/item/5fded5423ffa7d37b34088e0.jpg)

​	一个会话从NOT_CONNECTED开始，客户端初始化完成后就会转换到CONNECTING状态。

​	成功连接，就会到CONNECTED。

​	断开连接或者无法接收到服务器响应时，就会转换会CONNECTING状态，并尝试发现其他ZooKeeper服务器。如果可以发现另外一个服务器或者重连到原来服务器，确认会话有效之后又会转回到CONNECTED状态，否则将会声明会话过期，转换到CLOSED状态。也可以显式关闭会话。

**注意： 发生 网络 分区 时 等待 CONNECTING**

​	如果 一个 客户 端 与 服务器 因 超时 而 断开 连接， 客户 端 仍然 保持 CONNECTING 状态。 如果 因 网络 分区 问题 导致 客户 端 与 ZooKeeper 集合 被 隔离 而 发生 连接 断开， 那么 其 状态 将会 一直 保持， 直到 显 式 地 关闭 这个 会话， 或者 分区 问题 修复 后， 客户 端 能够 获悉 ZooKeeper 服务器 发送 的 会话 已经 过期。 发生 这种 行为 是因为 ZooKeeper 集合 对 声明 会话 超时 负责， 而 不是 客户 端 负责。 直到 客户 端 获悉 ZooKeeper 会话 过期， 否则 客户 端 不能 声明 自己的 会话 过期。 然而， 客户 端 可以 选择 关闭 会话。

**注意： 客户 端 会 尝试 连接 哪一个 服务器？**

​	在 仲裁 模式 下， 客户 端 有 多个 服务器 可以 连接， 而在 独立 模式 下， 客户 端 只能 尝试 重新 连接 单个 服务器。 在 仲裁 模式 中， 应用 需要 传递 可用 的 服务器 列表 给 客户 端， 告知 客户 端 可以 连接 的 服务器 信息 并 选择 一个 进行 连接。

​	当 尝试 连接 到 一个 不同 的 服务器 时， 非常 重要的 是， 这个 服务器 的 ZooKeeper 状态 要与 最后 连接 的 服务器 的 ZooKeeper 状态 保持 最新。 客户 端 不能 连接 到 这样 的 服务器： 它 未发现 更新 而 客户 端 却已 经 发现 的 更新。 ZooKeeper 通过 在 服务 中 排序 更新 操作 来 决定 状态 是否 最新。 ZooKeeper 确保 每一个 变化 相对于 所有 其他 已 执行 的 更新 是 完全 有序 的。 因此， 如果 一个 客户 端 在 位置 i 观察 到 一个 更新， 它 就不能 连 接到 只 观察 到 i'< i 的 服务器 上。

![](https://pic.imgdb.cn/item/5fdeda543ffa7d37b344dad8.jpg)

### ZooKeeper与仲裁模式

​	处于可靠性，我们需要运行多个服务器。幸运的是，我们仅仅需要做的便是配置一个更复杂的配置文件。

```properties
# 1
initLimit=10
syncLimit=5
dataDir=/usr/local/zk/zk1/data
dataLogDir=/usr/local/zk/zk1/log
clientPort=2183
server.1=127.0.0.1:2222:2223
server.2=127.0.0.1:3333:3334
server.3=127.0.0.1:4444:4445
# 2
initLimit=10
syncLimit=5
dataDir=/usr/local/zk/zk2/data
dataLogDir=/usr/local/zk/zk2/log
clientPort=2183
server.1=127.0.0.1:2222:2223
server.2=127.0.0.1:3333:3334
server.3=127.0.0.1:4444:4445
# 3
initLimit=10
syncLimit=5
dataDir=./data
dataLogDir=./log
clientPort=2183
server.1=127.0.0.1:2222:2223
server.2=127.0.0.1:3333:3334
server.3=127.0.0.1:4444:4445
```

​	每一个server.n指定了编号为n的ZooKeeper服务器使用的地址和端口号。第一部分是IP/主机名，第二部分和第三部分为TCP端口号，分别用于仲裁通信和群首选举。

​	还需要分别设置data目录。

```sh
mkdir z1 
mkdir z1/data 
mkdir z2
mkdir z2/data
mkdir z2/logs
mkdir z3 
mkdir z3/data
mkdir z3/logs
```

​	当启动一个服务器时，我们需要知道启动的是哪个服务器。一个服务器通过读取data目录下一个名为myid的文件来获取服务器ID信息。

```sh
echo 1 > z1/data/myid 
echo 2 > z2/data/myid 
echo 3 > z3/data/myid
```

​	服务器启动时，服务器通过配置文件中的dataDir参数来查找data目录配置。然后启动服务器。

```sh
cd z1 
{PATH_TO_ZK}/bin/zkServer.sh start ./z1.cfg
cd z2 
{PATH_TO_ZK}/bin/zkServer.sh start ./z2.cfg
cd z3
{PATH_TO_ZK}/bin/zkServer.sh start ./z3.cfg
```

​	使用zkCli.sh来访问集群

```sh
bin/zkCli.sh -server 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
```

使用检查

```sh
bin/zkServer.sh status z1/z1.cfg 
bin/zkServer.sh status z2/z2.cfg 
bin/zkServer.sh status z3/z3.cfg 
```

**注意事项**

1. 每项配置后不要添加空格。
2. server.n中n好像只能为数字。

### 实现一个原语：通过ZooKeeper实现锁







