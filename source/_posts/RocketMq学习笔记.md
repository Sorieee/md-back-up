---
4title: RocketMq学习笔记
date: 2020-06-01 10:46:56
tags: [MQ,RocketMq]
---

https://www.bilibili.com/video/BV1RE411r75d

# MQ介绍

消息队列是一种“先进先出”的数据结构。

![](https://pic.imgdb.cn/item/5ed46cb3c2a9a83be56c0d15.jpg)



应用场景主要包括以下3个方面

* 应用解耦

  替代RPC，降低耦合

* 流量削峰

  缓存请求，避免请求压垮服务器

* 数据分发

## 1.2 MQ的优点和缺点

优点：解耦、削峰、数据分发。

缺点：

* 系统可用性降低

  外部依赖越多，稳定性越差，一旦MQ宕机，就会对业务造成影响。

* 系统复杂度提高

* 一致性问题

## 1.3 MQ产品比较

https://www.cnblogs.com/nov5026/archive/2018/08/22/9518520.html

# RocketMq快速入门

## 2.1准备工作

### 下载RocketMq

http://rocketmq.apache.org/

版本4.7.0

### 系统环境

* Linux64位环境
* JDK1.8
* 源码安装需要安装Maven 3.2.x

## 2.2 安装RocketMQ

### 安装步骤

以二进制包进行安装

1. 解压安装包
2. 进入安装目录

```sh
unzip rocketmq-all-4.7.0-bin-release.zip
```



### 目录介绍

* bin: 启动脚本，包括shell脚本和cmd脚本
* conf：实例配置文件，包括broker配置文件，logback配置文件等
* lib：依赖jar包，包括netty，commons-lang,FastJson等

## 2.3启动RocketMQ

1. 启动NameServer

```sh
# 1.启动NameServer
nohup sh bin/mqnamesrv &
# 2.查看启动日志
tail -f ~/logs/rocketmqlogs/namesrv.log
```

2. 启动broker

```sh
# 1.启动broker
nohup sh bin/mqbroker -n localhost:9876 &
# 2.查看启动日志
tail -f ~/logs/rocketmqlogs/broker.log
```

* 问题描述

  RocketMQ默认的虚拟机内存较大，启动Broker如果因为内存不足失败，需要编辑如下两个配置文件，修改JVM内存大小

```sh
vim runbroker.sh
vim runserver.sh
```

* 参考设置

```sh
JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx256m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

## 2.4测试RocketMQ

### 消息发送

```sh
# 1.设置环境变量
export NAMESRV_ADDR=localhost:9876
# 2.使用安装包的Demo发送信息
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```

### 接受消息

```sh
# 1.设置环境变量
export NAMESRV_ADDR=localhost:9876
# 2.接受消息
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Comsumer
```



## 2.5 关闭RocketMQ

```sh
# 1.关闭NameServer
sh bin/mqshutdown namesrv
# 2.关闭Broker
sh bin/mqshutdown broker
```

# RocketMQ集群搭建

## 3.1各角色介绍

* Producer：消息的发送者。
* Consumer：消息的接收者。
* Broker：暂存和传递消息。
* NameServer：管理Broker。
* Topic：区分消息的分类；一个发送者发送消息给一个或者多个Topic；一个消息的接收者可以订阅一个或多个Topic消息。
* MessageQueue：相当于是Topic的分区；用于并行发送和接收消息。

![](https://pic.imgdb.cn/item/5ed6606cc2a9a83be5e5e6c1.jpg)

## 3.2 集群搭建方式

### 集群特点

* NameServer是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。
* Broker部署相对复杂，Broker分为Master和Slave，一个Master可以对应多个Slave，但是一个Slave只能对应一个Master，Master和Slave的对应关系通过指定相同的BrokerName，不同的BrokerId来定义，BrokerId为0代表Master，非0代表Slave。Master也可以部署多个。每个Broker与NameServer集群中的所有节点建立长连接，定时注册Topic信息到所有NameServer。
* Producer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer取Topic路由信息，并向提供Topic服务的Master建立长连接，且定时向Master发送心跳。Producer完全无状态，可集群部署。
* Consumer与NameServer集群中其中一个节点（随机选择）建立长连接，定期从NameServer取路由信息，并向提供Topic服务器的Master和Slave建立长连接，且定时向Master和Slave发送心跳。Consumer既可以从Master订阅信息，也可以从Slave订阅信息，订阅规则由Broker决定。

### 集群模式

**单Master模式**

这种方式风险较大，一旦Broker重启或者宕机时，会导致整个服务不可用。不建议线上环境使用，可用于本地测试。

**多Master模式**

一个集群无Slave，全是Master，例如2个或者3个Master。

* 优点：配置简单，单个Master宕机或重启维护对应用无影响，在磁盘配置为RAID10时，即使机器宕机不可恢复情况下，由于RAID10磁盘非常可靠，消息也不会丢失（异步刷盘丢失少了信息，同步刷盘一条不丢），性能最高。
* 缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受到影响。

**多Master多Slave模式（异步）**

每个Master配置一个Slave，有多对Master-Slave，HA采用异步复制方式，主备有短暂消息延迟（毫秒级），这种模式的优缺点如下：

* 优点：即使磁盘损坏，消息丢失非常少，且消息实时性不受影响，同时Master宕机后，消费者仍然从Slave消费，而且此过程对应用透明，不需要人工干预，性能同多Master模式几乎一样。
* 缺点：Master宕机，磁盘损坏情况下会丢失少量信息。

**多Master多Slave模式（同步）**

每个Master配置一个Slave，有多对Master-Slave，HA采用同步双写方式，即主备都写成功，才向应用返回成功。

* 优点：数据与服务都无单点故障，Master宕机情况下，消息无延迟，服务可用性与数据可用性都非常高。
* 缺点：性能比异步复制模式略低（大约低10%左右），发送单个消息的RT略高，且在目前版本主节点宕机后，备机不能自动切换为主机。

## 3.3 双主双从集群搭建

### 总体架构

![](https://pic.imgdb.cn/item/5ed8f998c2a9a83be58351d9.jpg)



### 集群工作流程

1.启动NameServer，NameServer起来后监听端口，等待Broker、Producer、Consumer连上来，相当于一个路由控制中心。

2.Broker启动，跟所有的NameServer保持长连接，定时发送心跳包。心跳包中包含当前Broker信息（IP+端口等）以及存储所有Topic信息。注册成功后，NameServer集群中就有Topic跟Broker的映射关系。

3.收发消息前，先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，也可以在发送消息时自动创建Topic。

4.Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic存在哪些Broker上，轮询从队列列表中选择一个队列，然后与队列所在的Broker建立长连接从而向Broker发送消息。

5.Consumer跟Producer类似，跟其中一台NameServer建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，开始消费消息。

### 服务器环境

| 序号 | IP             | 角色              | 架构模式        |
| ---- | -------------- | ----------------- | --------------- |
| 1    | 192.168.25.xxx | nameserver,broker | Master1,Slave2  |
| 2    | 192.168.25.yyy | nameserver,broker | Master2，Salve1 |

### Host添加信息

```sh
vim /etc/hosts
```

配置如下

```sh
# nameserver
192.168.1.6 rocketmq-nameserver1
192.168.1.7 rocketmq-nameserver2
# broker
192.168.1.6 rocketmq-master1
192.168.1.6 rocketmq-slave2
192.168.1.7 rocketmq-master2
192.168.1.7 rocketmq-slave1
```

完成后重启网卡,centos8是

```sh
nmcli c reload
```

### 防火墙配置

宿主机需要远程访问虚拟机的rocketmq服务和web服务，需要开放相关的端口号，简单粗暴的方式是直接关闭防火墙。

```sh
# 关闭防火墙
systemclt stop firewalld.service
# 查看防火墙状态
firewall-cmd --state
# 禁止firewall开机启动
systemctl disable firewalld.service
```

或者为了安全，只开放特定端口，RocketMQ默认使用3个端口:9876、10911、11011。如果防火墙没有关闭的话，那么防火墙必须开放这些端口。

* nameserver 默认使用9876端口
* master 默认使用10911端口
* slave 默认使用11011端口

执行以下命令：

```sh
# 开放name server默认端口
firewall-cmd --remove-port=9876/tcp --permanet
# 开放master默认端口
firewall-cmd --remove-port=10911/tcp --permanet
# 开放slave默认端口(当前集群模式可不开启)
firewall-cmd --remove-port=11011/tcp --permanet
# 重启防火墙
firewall-cmd --reload
```

### 环境变量配置

```sh
vim /etc/profile
```

在profile文件的末尾加入如下命令：

```sh
#set rocketmq
ROCKETMQ_HOME=/opt/rocketmq-all-4.7.0-bin-release
PATH=$PATH:$ROCKETMQ_HOME/bin
export ROCKETMQ_HOME PATH
```

输入:wq保存并退出，并使得配置立即生效

```sh
source /etc/profile
```

### 创建消息存储路径

```sh
mkdir /opt/rocketmq/store
mkdir /opt/rocketmq/store/commitlog
mkdir /opt/rocketmq/store/consumequeue
mkdir /opt/rocketmq/store/index
```

### broker配置文件

**master1**

服务器：192.168.1.6

```sh
vim /opt/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-a.properties
```

修改配置如下：

```sh
# 所属集群名字
brokerClusterName=rocketmq-cluster
# broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master， >0 表示 slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨4点
deletewhen=04
#文件保留时间，默认 48 小时
fileReserveredTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30w条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/opt/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/opt/rocketmq/store/commitlog
#消费队列存储路径
storePathConsumeQueue=/opt/rocketmq/store/consumequeue
#消费索引存储路径
storePathIndex=/opt/rocketmq/store/index
# checkpoint 文件存储路径
storeCheckpoint=/opt/rocketmq/store/checkpoint
#abort 文件存储地址
abortFile=/opt/rocketmq/store/abort
#限制消息的大小
maxMessageSize=65536
# Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写MASTER
# - SLAVE
brokerRole=SYNC_MASTER
# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH

```

**Slave2**

服务器：192.168.1.6

```sh
vim /opt/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-b-s.properties
```

修改配置如下：

```sh
# 所属集群名字
brokerClusterName=rocketmq-cluster
# broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示 Master， >0 表示 slave
brokerId=1
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=11011
#删除文件时间点，默认凌晨4点
deletewhen=04
#文件保留时间，默认 48 小时
fileReserveredTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30w条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/opt/rocketmq/store1
#commitLog 存储路径
storePathCommitLog=/opt/rocketmq/store1/commitlog
#消费队列存储路径
storePathConsumeQueue=/opt/rocketmq/store1/consumequeue
#消费索引存储路径
storePathIndex=/opt/rocketmq/store1/index
# checkpoint 文件存储路径
storeCheckpoint=/opt/rocketmq/store1/checkpoint
#abort 文件存储地址
abortFile=/opt/rocketmq/store/abort
#限制消息的大小
maxMessageSize=65536
# Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写MASTER
# - SLAVE
brokerRole=SLAVE
# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

```

**Master2**

服务器：192.168.1.7

```sh
vim /opt/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-b.properties
```

修改配置如下：

```sh
# 所属集群名字
brokerClusterName=rocketmq-cluster
# broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示 Master， >0 表示 slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨4点
deletewhen=04
#文件保留时间，默认 48 小时
fileReserveredTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30w条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/opt/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/opt/rocketmq/store/commitlog
#消费队列存储路径
storePathConsumeQueue=/opt/rocketmq/store/consumequeue
#消费索引存储路径
storePathIndex=/opt/rocketmq/store/index
# checkpoint 文件存储路径
storeCheckpoint=/opt/rocketmq/store/checkpoint
#abort 文件存储地址
abortFile=/opt/rocketmq/store/abort
#限制消息的大小
maxMessageSize=65536
# Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写MASTER
# - SLAVE
brokerRole=SYNC_MASTER
# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
```

**Slave2**

服务器：192.168.1.7

```sh
vim /opt/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-a-s.properties
```

修改配置如下：

```sh
# 所属集群名字
brokerClusterName=rocketmq-cluster
# broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master， >0 表示 slave
brokerId=1
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=11011
#删除文件时间点，默认凌晨4点
deletewhen=04
#文件保留时间，默认 48 小时
fileReserveredTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30w条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/opt/rocketmq/store1
#commitLog 存储路径
storePathCommitLog=/opt/rocketmq/store1/commitlog
#消费队列存储路径
storePathConsumeQueue=/opt/rocketmq/store1/consumequeue
#消费索引存储路径
storePathIndex=/opt/rocketmq/store1/index
# checkpoint 文件存储路径
storeCheckpoint=/opt/rocketmq/store1/checkpoint
#abort 文件存储地址
abortFile=/opt/rocketmq/store1/abort
#限制消息的大小
maxMessageSize=65536
# Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写MASTER
# - SLAVE
brokerRole=SLAVE
# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

```

### 修改启动脚本文件

**runbroker.sh**

```sh
vim /bin/runbroker.sh
```

需要根据内存大小进行适当的对JVM参数进行调整：

```sh
# 开发环境配置 JVM Configuration
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```

**runserver.sh**

```sh
vim /bin/runserver.sh
```

需要根据内存大小进行适当的对JVM参数进行调整：

```sh
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

### 服务启动

**启动NameServer集群**

分别在两台集群启动NameServer

```sh
cd /bin
nohup sh mqnamesrv &
```

**启动Broker集群**

master1：

```sh
cd /bin
nohup sh mqbroker -c /opt/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-a.properties &
```

slave2：

```sh
cd /bin
nohup sh mqbroker -c /opt/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-b-s.properties &
```

master2：

```sh
cd /bin
nohup sh mqbroker -c /opt/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-b.properties &
```

slave1：

```sh
cd /bin
nohup sh mqbroker -c /opt/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-a-s.properties &
```

### 查看日志

```sh
# nameSrv日志
tail -f ~/logs/rocketmqlogs/namesrv.log
# broker日志
tail -f ~/logs/rocketmqlogs/broker.log
```

**易错点**

中间遇到无法启动broker的情况，是因为使用了同一个storePath，改一下就好。

## 3.4 mqadmin 管理工具

### 使用方式

进入mq安装位置，在bin目录下执行 ./mqadmin {command} {args}

### 命令介绍

略

### 3.5 集群监控平台搭建

### 概述

我们需要自己对rocketmq-console进行编译打包运行。

https://github.com/apache/rocketmq-externals

#### 3.5.2 下载并编译打包

```
git clone https://github.com/apache/rocketmq-externals
cd rocketmq-console
mvn clean package -Dmaven.test.skip=true
```

注意：打包前在rocket-console中配置namesrv集群地址：

```
rocketmq.config.namesrvAddr=192.168.1.6:9876;192.168.1.7:9876;
```

上传到linux服务器上，启动rocketmq-console：

```
java -jar rocketmq-console-ng-1.0.0.jar
```

启动成功，我们就可以通过浏览器http://localhost:8080进入控制台界面了。

# 消息发送样例

* 创建maven项目并导入客户端依赖

  ```xml
  <!-- https://mvnrepository.com/artifact/org.apache.rocketmq/rocketmq-client -->
  <dependency>
      <groupId>org.apache.rocketmq</groupId>
      <artifactId>rocketmq-client</artifactId>
      <version>4.7.0</version>
  </dependency>
  ```

* 消息发送者步骤分析

```tex
1. 创建消息生产者producer,并指定生产者组名。
2. 指定NameServer地址
3. 启动producer
4. 创建消息对象，指定主题Topic、Tag和消息体
5. 发送消息
6. 关闭生产者Producer
```

* 消息消费者步骤分析

```tex
1. 创建消费者Consumer，指定消费者组名
2. 指定NameServer地址
3. 订阅主题Topic和Tag
4. 设置回调函数，处理消息
5. 启动消费者Consumer
```

## 4.1 基本样例

### 4.1.1 消息发送

#### 1）发送同步消息

这种可靠性同步地发送方式使用比较广泛，比如：重要的消息通知，短信通知

```java
package cn.sorie.mq.rocketmq.base;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.client.producer.SendStatus;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

import java.util.concurrent.TimeUnit;

public class SyncProducer {
    public static void main(String[] args) throws Exception {
        //实例化生产者producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 设置NameServer的地址
        producer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //启动producer
        producer.start();
        try {
            for (int i = 0; i < 10; i++) {
                //创建消息 指定topic,Tag和消息体
                Message msg = new Message("base",
                        "Tag1", ("Hello MQ" + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                //发送消息到一个broker
                SendResult sendResult = producer.send(msg);
                SendStatus status = sendResult.getSendStatus();
                String msgId = sendResult.getMsgId();
                int queueId = sendResult.getMessageQueue().getQueueId();

                //发送消息是否成功送达
                System.out.printf("发送状态：%s 消息id：%s 队列id:%d\n", status, msgId, queueId);
                TimeUnit.SECONDS.sleep(1);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            producer.shutdown();
        }
    }
}
```

#### 2)  发送异步消息

异步消息通常用在响应时间敏感的业务场景，即发送端不能容忍长时间等待Broker的响应。

```java
package cn.sorie.mq.rocketmq.base;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.client.producer.SendStatus;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

import java.util.concurrent.TimeUnit;

public class AsyncProducer {
    public static void main(String[] args) throws Exception {
        //实例化生产者producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 设置NameServer的地址
        producer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //启动producer
        producer.start();
        producer.setRetryTimesWhenSendAsyncFailed(0);
        try {
            for (int i = 0; i < 10; i++) {
                //创建消息 指定topic,Tag和消息体
                final int index = i;
                Message msg = new Message("base",
                        "Tag1", ("Async Hello MQ" + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                //发送消息到一个broker
                producer.send(msg, new SendCallback() {
                    public void onSuccess(SendResult sendResult) {
                        SendStatus status = sendResult.getSendStatus();
                        String msgId = sendResult.getMsgId();
                        int queueId = sendResult.getMessageQueue().getQueueId();
                        System.out.printf("发送状态：%s 消息id：%s 队列id:%d\n", status, msgId, queueId);
                    }

                    public void onException(Throwable e) {
                        System.out.printf("%-10d Exception %s %n", index, e);
                    }
                });

                //发送消息是否成功送达
                TimeUnit.SECONDS.sleep(1);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            producer.shutdown();
        }
    }
}
```

#### 3) 单向消息

不在乎消息的发送状态，比如日志等消息。

```java
package cn.sorie.mq.rocketmq.base;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.client.producer.SendStatus;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

import java.util.concurrent.TimeUnit;

public class OneWayProducer {
    public static void main(String[] args) throws Exception {
        //实例化生产者producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 设置NameServer的地址
        producer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //启动producer
        producer.start();
        try {
            for (int i = 0; i < 10; i++) {
                //创建消息 指定topic,Tag和消息体
                Message msg = new Message("base",
                        "Tag1", ("Hello MQ 单向消息" + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                //发送消息到一个broker
                producer.sendOneway(msg);


                //发送消息是否成功送达
                System.out.printf("发送状态：%n\n", i);
                TimeUnit.SECONDS.sleep(5);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            producer.shutdown();
        }
    }
}
```

### 4.1.2 消费消息

```java
package cn.sorie.mq.rocketmq.base;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.client.producer.SendStatus;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

import java.util.concurrent.TimeUnit;

public class OneWayProducer {
    public static void main(String[] args) throws Exception {
        //实例化生产者producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 设置NameServer的地址
        producer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //启动producer
        producer.start();
        try {
            for (int i = 0; i < 10; i++) {
                //创建消息 指定topic,Tag和消息体
                Message msg = new Message("base",
                        "Tag2", ("Hello MQ 单向消息" + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                //发送消息到一个broker
                producer.sendOneway(msg);


                //发送消息是否成功送达
                System.out.printf("发送状态：%n\n", i);
                TimeUnit.SECONDS.sleep(5);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            producer.shutdown();
        }
    }
}
```

#### 1） 负载均衡模式

消费者采用负载均衡模式方法消费消息，多个消费者共同消费队列消息，每个消费者处理的消息不同。 默认为负载均衡。

```java
package cn.sorie.mq.rocketmq.base.consumer;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;

import java.util.List;

public class ClusterConsumer {
    public static void main(String[] args) throws MQClientException {
        //1. 创建消费者Consumer，指定消费者组名
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        //2. 指定NameServer地址
        consumer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //3. 订阅主题Topic和Tag
        consumer.subscribe("base", "Tag1");
        //设置负载均衡
        consumer.setMessageModel(MessageModel.CLUSTERING);
        //4. 设置回调函数，处理消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            //接收消息内容
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for (MessageExt per : list) {
                    System.out.println(" 负载均衡" + new String(per.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //5. 启动消费者Consumer
        consumer.start();
    }
}
```



#### 2）广播模式

消费者采用广播的方式消费消息，每个消费者的消息是相同的。

```java
package cn.sorie.mq.rocketmq.base.consumer;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;

import java.util.List;

public class ClusterConsumer {
    public static void main(String[] args) throws MQClientException {
        //1. 创建消费者Consumer，指定消费者组名
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        //2. 指定NameServer地址
        consumer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //3. 订阅主题Topic和Tag
        consumer.subscribe("base", "Tag1");
        //设置负载均衡
        consumer.setMessageModel(MessageModel.BROADCASTING);
        //4. 设置回调函数，处理消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            //接收消息内容
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for (MessageExt per : list) {
                    System.out.println(" 负载均衡" + new String(per.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //5. 启动消费者Consumer
        consumer.start();
    }
}
```

## 4.2 顺序消息

​	消息有序指的是可以按照消息的发送顺序来消费消息(FIFO)。RocketMq可以严格的保证消息有序。可以分为分区有序或者全局有序。

​	顺序消费的原理解析，在默认情况下消息发送会采取Round Robin轮询方式把消息发送到不同的queue(分区队列)；而消费消息时从多个queue上拉取消息，这种情况发送和消费是不能保证顺序。但是如果控制发送的顺序只依次发送到同一个queue中，消费的时候只从这个queue依次拉取，则保证了顺序。当发送和消费参与的queue只有一个，则是全局有序；如果有多个queue参与，则是分区有序，即相对于每个queue，消息都是有序的。

### 4.2.1 顺序消息生产

​	下面用订单进行分区有序的示例。一个订单的顺序：创建、付款、推送、完成。订单相同的消息会被先后发送到同一个队列中，消费时，同一个OrderId获取到的肯定是同一个队列。

```java
package cn.sorie.mq.rocketmq.order;

import org.apache.rocketmq.client.producer.*;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.remoting.common.RemotingHelper;

import java.util.List;
import java.util.concurrent.TimeUnit;

public class Producer {
    public static void main(String[] args) throws Exception {
        //实例化生产者producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 设置NameServer的地址
        producer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //启动producer
        producer.start();

        //构建消息集合
        List<OrderStep> orderStepList = OrderStep.buildOrders();
        try {
            //发送消息
            int i = 0;
            for (OrderStep per : orderStepList) {
                String body = per + "";
                Message msg = new Message("OrderTopic", "Order", "i" + i, body.getBytes());
                //参数一：消息对象
                //参数二：消息队列选择器
                //参数三：选择队列的业务标识（订单id）

                SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                    /**
                     * @param list    : 队列集合
                     * @param message : 消息对象
                     * @param o 业务标识参数
                     * @return
                     * @author soriee
                     * @date 2020/6/30 22:03
                     */
                    public MessageQueue select(List<MessageQueue> list, Message message, Object o) {
                        long orderId = (Long) o;
                        int index = (int) (orderId % list.size());
                        System.out.println("队列 " + index);
                        return list.get(index);
                    }
                }, per.getOrderId());
                System.out.println("发送成功: " + sendResult.toString());
                i++;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            producer.shutdown();
        }
    }
}
```

### 4.2.2 顺序消息消费

```java
package cn.sorie.mq.rocketmq.order;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.*;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class Consumer {
    public static void main(String[] args) throws Exception{
        //1. 创建消费者Consumer，指定消费者组名
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        //2. 指定NameServer地址
        consumer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //3. 订阅主题Topic和Tag
        consumer.subscribe("OrderTopic", "*");
        //4. 设置回调函数，处理消息
        consumer.registerMessageListener(new MessageListenerOrderly() {
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext consumeOrderlyContext) {
                for (MessageExt per : list) {
                    System.out.println("线程名称【" + Thread.currentThread().getName() + "】 消费消息：" + new String(per.getBody()));
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });
        //5. 启动消费者Consumer
        consumer.start();
    }
}
```

## 4.3 延时消息

​	比如电商里，提交了一个订单就可以发送一个延时消息，1h后去检查这个订单的状态，如果还是未付款就取消订单释放库存。

### 4.3.1 启动消息消费者

```java
package cn.sorie.mq.rocketmq.delay;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerOrderly;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class Consumer {
    public static void main(String[] args) throws Exception{
        //1. 创建消费者Consumer，指定消费者组名
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        //2. 指定NameServer地址
        consumer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //3. 订阅主题Topic和Tag
        consumer.subscribe("topicDelay", "*");
        //4. 设置回调函数，处理消息
        consumer.registerMessageListener(new MessageListenerOrderly() {
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext consumeOrderlyContext) {
                for (MessageExt per : list) {
                    System.out.println("线程名称【" + Thread.currentThread().getName() + "】 消费消息：" + new String(per.getBody()) + System.currentTimeMillis());
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });
        //5. 启动消费者Consumer
        consumer.start();
    }
}
```

### 4.3.2 发送延时消息

```java
package cn.sorie.mq.rocketmq.delay;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.client.producer.SendStatus;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

import java.util.concurrent.TimeUnit;

public class Producer {
    public static void main(String[] args) throws Exception {
        //实例化生产者producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        // 设置NameServer的地址
        producer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");
        //启动producer
        producer.start();
        try {
            for (int i = 0; i < 10; i++) {
                //创建消息 指定topic,Tag和消息体
                Message msg = new Message("topicDelay",
                        "Tag1", ("Hello MQ 延时消息" + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                //发送消息到一个broker
                msg.setDelayTimeLevel(2);
                SendResult sendResult = producer.send(msg);
                SendStatus status = sendResult.getSendStatus();
                String msgId = sendResult.getMsgId();
                int queueId = sendResult.getMessageQueue().getQueueId();

                //发送消息是否成功送达
                System.out.printf("发送状态：%s 消息id：%s 队列id:%d\n", status, msgId, queueId);
                TimeUnit.SECONDS.sleep(1);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            producer.shutdown();
        }
    }
}
```

### 4.3.3 验证

您将会看到消息的消费比存储时间晚10秒

### 4.3.4 使用限制

```java
// org.apache.rocketmq/store/config/MessageStoreConfig.java
private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h"
```

现在RocketMq并不支持任意时间的延时，需要设置几个固定延时等级，从1s到2h对应等级1到18

## 4.4 批量消息

​	批量消息能够显著提高传递小消息的性能，限制是这些批量消息应该有相同的topic，相同的waitStoreMsgOK，而且不能是延时消息。此外，这一批的消息总大小应不超过4MB。

### 4.4.1 发送批量消息

​	如果您每次只发送4MB不到的消息，则很容易批处理，样例如下：

```java
String topic = "BatchTest";
List<Message> msgs = new ArrayList<>();
msgs.add(new Message(topic), "TagA", "OrderID001", "Hello World 0".getBytes);
msgs.add(new Message(topic), "TagA", "OrderID001", "Hello World 0".getBytes);
msgs.add(new Message(topic), "TagA", "OrderID001", "Hello World 0".getBytes);
try {
    producer.send(msgs);
} catch (Exception e) {
    e.printTrace();
}
```

​	如果消费的总长度可能大于4MB时，这时候最好把消费进行分割。

创建一个分割器

一个消息的长度=topic长度和body长度以及消息额外的属性以及日志开销20字节。

![](https://pic.imgdb.cn/item/5efde28e14195aa594a1ce17.jpg)



## 4.5 过滤消息

​	在大多数情况下,TAG是一个简单而有用的，其可以选择您想要的消息。例如：

```java
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("CID_EXAMPLE");
consumer.subscribe("TOPIC", "TAGA || TAGB || TAGC");
```

​	消费者将接收TAGA或TAGB或TAGC的消息。但是限制是一个消息只能有一个标签，这对复杂的场景可能不起作用。在这种情况下，可以使用SQL表达式筛选我消息。SQL特性可以通过发送消息时的属性来进行计算。在RocketMQ定义的语法下，实现一些简单的逻辑。下面是一个例子：

​	

```
----------
|message   |
|----------|  a > 5 AND b = 'abc'  
|a=10      |   --->Gotten
|b='abc'   |
|c=true    |
------------

----------
|message   |  a > 5 AND b = 'abc'  
|----------|  ---> Missing
|a=1      |
|b='abc'   |
|c=true    |
------------
```

### 4.5.1 SQL基本语法

RocketMQ只定义了一些基本语法来支持这个特性，你也可以很容易地扩展它。

* 数值比较, >, >= , <, <=, BETWEEN, =;
* 字符比较,比如：=，<>, IN;
* IS NULL或者 IS NOT NULL
* 逻辑符号 AND OR NOT；

支持的类型有：

* 数值，整数和小数
* 字符，必须用单引号括起来
* NULL, 特殊的常量
* 布尔值，TRUE或者FALSE

只有使用push模式的消费者才能够使用SQL92标准的sql语句

```java
public void subcribe(final String topic, final MessageSelector messageSelector)
```


### 4.5.2 消息生产者
发送消息时，可以通过putUserProperty来设置消息的属性。

```java
msg.setUserProperty("i", String.valueOf(i));
```



### 4.5.3 消息消费者

```java
consumer.subcribe("FilterSqlTopic", MessageSelector.bySql("i>5"));
```

## 4.6 事务消息

### 4.6.1  流程分析

![](https://pic.imgdb.cn/item/5eff3c6414195aa59424388d.jpg)



上图大致说明了事务消息的大致方案，分为两个流程：正常事务消息的发送及提交、事务消息的补偿流程

#### 1）事务消息发送及提交

1. 发送消息（half消息）。
2. 服务端响应消息写入结果。
3. 根据发送结果执行本地事务（如果写入失败，此时half消息对业务不可见，本地逻辑不执行）。
4. 根据本地事务状态Commit或者RollBack(Commit操作生成消息索引，消息对消费者可见)。

#### 2）消息补偿

1. 对没有Commit和Rollback的事务消息(pending状态消息)，从服务端发起一次回查
2. Producer收到回查消息，检查回查消息的本地事务的状态
3. 根据本地事务状态，重新Commit或者RollBack

其中，补偿阶段用户解决消息Commit或者RollBack发生超时或者失败的情况

#### 3）事务消息状态

事务消息共有三种状态：提交状态、回滚状态、中间状态：

* TransactionStatus.CommitTransaction:提交事务，它允许消费者消费此消息
* TransactionStatus.RollBackTransaction: 回滚事务，它代表该消息将被删除，不允许被消费
* TransactionStatus.UnKnown：中间状态，它代表需要检查消息队列来确定状态

### 4.6.2 发送事务消息

#### 1) 创建事务性生产者

​	使用TransactionMQProducer类创建生产者，并指定唯一的ProducerGroup，就可以设置自定义线程池来处理这些检查要求。执行本地事务后，需要根据执行结果对消息队列进行回复。回传的事务状态在请参考前一节。

​	//好像事务消息发送者关闭太快回查无法触发

```java
package cn.sorie.mq.rocketmq.transaction;

import org.apache.commons.lang3.StringUtils;
import org.apache.rocketmq.client.producer.*;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.remoting.common.RemotingHelper;

import java.util.concurrent.TimeUnit;

public class Producer {
    public static void main(String[] args) throws Exception {
        //实例化生产者producer
        TransactionMQProducer producer = new TransactionMQProducer("group5");
        // 设置NameServer的地址
        producer.setNamesrvAddr("192.168.1.6:9876;192.168.1.7:9876");

        //设置消息事务的消息监听器
        producer.setTransactionListener(new TransactionListener() {

            //该方法中去执行本地事务
            public LocalTransactionState executeLocalTransaction(Message message, Object o) {
                if (StringUtils.equals("TAGA", message.getTags())) {
                    return LocalTransactionState.COMMIT_MESSAGE;
                } else if (StringUtils.equals("TAGB", message.getTags())) {
                    return LocalTransactionState.ROLLBACK_MESSAGE;
                } else {
                    return LocalTransactionState.UNKNOW;
                }
            }
            //MQ进行消息事务状态的回查
            public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
                System.out.println(messageExt.getTags());
                return LocalTransactionState.COMMIT_MESSAGE;
            }
        });
        //启动producer
        String[] tags = {"TAGA", "TAGB", "TAGC"};
        producer.start();
        try {
            for (int i = 0; i < 3; i++) {
                //创建消息 指定topic,Tag和消息体
                Message msg = new Message("TransactionTopic",
                        tags[i], ("Hello MQ 事务消息" + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                //发送消息到一个broker
                SendResult sendResult = producer.sendMessageInTransaction(msg, null);
                SendStatus status = sendResult.getSendStatus();
                String msgId = sendResult.getMsgId();
                int queueId = sendResult.getMessageQueue().getQueueId();

                //发送消息是否成功送达
                System.out.printf("发送状态：%s 消息id：%s 队列id:%d, %s\n", status, msgId, queueId, tags[i]);
                TimeUnit.SECONDS.sleep(2);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
//            int n = 100000;
//            while (true) {
//                
//            }
            producer.shutdown();
        }
    }
}
```

### 4.6.3 使用限制

1. 事务消息不支持延时消息和批量消息
2. 为了避免单个消息检查太多次而导致半队列累计，我们默认将单个消息的检查次数限制为15次，但是用户可以通过Broker配置文件的trancationCheckMax参数修改此限制。如果已经检查某条消息超过N次的话，则丢弃此消息，并在默认情况下同时打印错误日志。用户可以通过重写AbstractTransactionCheckListener类来修改这个行为。
3. 事务消息将在Broker配置文件中的参数transactionMsgTimeout这样的特定时间长度之后被检查。当发送事务消息时，用户还可以通过设置用户属性CHECK_IMMUNITY_TIME_IN_SECONDS来改变这个限制，该参数优先于transactionMsgTimeout参数。
4. 事务消息可能不止一次被检查或消费。
5. 提交给用户的目标主题消息可能会失败，目前这一日志的记录而定。它的高可用性RocketMQ本身的高可用机制来保证，如果希望确保事务消息不丢失、并且事务完整性得到保证，建议使用同步双重写入机制。
6. 事务消息的生产者ID不能与其他类型消息的生产者ID共享。与其他类型的消息不同，事务消息允许反向查询、MQ服务器通过它们的生产者ID查询到消费者。

# 案例介绍

## 5.1 业务分析

​	模拟电商网站购物场景下的下单和支付业务。

### 1）下单

![](https://pic.imgdb.cn/item/5f00846714195aa594a01a01.jpg)



1. 用户请求订单系统下单
2. 订单系统通过RPC调用订单服务下单
3. 订单服务调用优惠券服务，扣减优惠券
4. 订单服务调用库存服务，校验并扣减库存
5. 订单服务调用用户服务，扣减用户余额
6. 订单完成确认订单。

### 2）支付

![](https://pic.imgdb.cn/item/5f00853414195aa594a077df.jpg)



1. 用户请求支付系统
2. 支付系统调用第三方支付平台API发起支付流程
3. 用户通过第三方支付平台支付成功后，第三方支付平台回调支付系统
4. 支付系统调用订单服务修改订单状态
5. 支付系统调用积分服务添加积分
6. 支付系统调用日志服务记录日志

## 5.2 问题分析

### 问题1

​	用户提交订单后，扣减库存成功、扣减优惠券成功、使用余额成功，但是在确认订单操作失败，需要对库存、优惠券、余额进行回退。如何保证数据的完整性。

![](https://pic.imgdb.cn/item/5f00872014195aa594a15f0f.jpg)

### 问题2

​	用户通过第三方支付平台支付成功后，第三方支付平台要回调API异步通知商家支付系统用户支付结果，支付系统根据支付结果修改订单状态、记录支付日志和给用户增加积分。

​	支付系统如何保证在收到第三方支付平台的异步通知时，如果快速给第三方支付凭条做出回应。

​	![](https://pic.imgdb.cn/item/5f00883114195aa594a1e2af.jpg)

通过MQ进行数据分发，提高系统处理性能。

![](https://pic.imgdb.cn/item/5f00885414195aa594a1f3f9.jpg)

# 技术分析

## 6.1 技术选型

* Spring Boot
* Dubbo
* Zookeeper
* RocketMQ
* MySql

![](https://pic.imgdb.cn/item/5f00893914195aa594a27427.jpg)



## 6.2 SpringBoot整合RocketMQ

​	下载rocketmq-spring 项目

​	https://github.com/apache/rocketmq-spring

​	将rocketmq-spring安装到本地仓库

```bash
mvn install -Dmaven.skip.test = true
```

### 6.2.1 消息生产者

#### 1) 添加依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>cn.sorie</groupId>
    <artifactId>rocketmq-producer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>rocketmq-producer</name>
    <description>rocketmq</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.rocketmq/rocketmq-spring-boot-starter -->
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

#### 2) 发送消息测试者

```java
package cn.sorie.rocketmqproducer;

import lombok.extern.slf4j.Slf4j;
import org.apache.rocketmq.spring.core.RocketMQTemplate;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = {RocketmqProducerApplication.class})
@Slf4j
class RocketmqProducerApplicationTests {

    @Autowired
    private RocketMQTemplate rocketMQTemplate;

    @Test
    public void testSendMessage() {
        rocketMQTemplate.convertAndSend("springboot-rocketmq", "Hello Springboot RocketMq");
        log.info("发送消息");
    }
}
```

#### 3）application.properties

```properties
rocketmq.name-server=192.168.1.6:9876;192.168.1.7:9876;
rocketmq.producer.group=my-grp
```

### 6.2.2 消息消费者

#### 1) 添加依赖

同上

#### 2) 消费者

```java
package cn.sorie.rocketmqconsumer.listener;

import lombok.extern.slf4j.Slf4j;
import org.apache.rocketmq.spring.annotation.RocketMQMessageListener;
import org.apache.rocketmq.spring.core.RocketMQListener;
import org.springframework.stereotype.Component;

@RocketMQMessageListener(topic = "springboot-rocketmq",
        consumerGroup = "${rocketmq.producer.group}")
@Slf4j
@Component
public class Consumer implements RocketMQListener<String> {
    @Override
    public void onMessage(String s) {
        System.out.println("接收消息" + s);
    }
}
```

#### 3）application.properties

```properties
rocketmq.name-server=192.168.1.6:9876;192.168.1.7:9876;
rocketmq.producer.group=my-grp
```

## 6.3 Spring Boot整合Dubbo

rpc远程调用

下载dubbo-spring-boot-starter依赖包

安装到本地仓库

```shell
mvn install -Dmaven.skip.test = true
```

![](https://pic.imgdb.cn/item/5f01d3af14195aa59457b14c.jpg)

### 6.3.1 搭建Zookeeper集群

#### 1）准备工作

1. 安装JDK

2. 将Zookeeper上传服务器

3. 解压Zookeeper，并创建data目录，将conf下的zoo_sample.cfg文件名改为zoo.cfg

4. 建立/user/local/zookeeper-cluster, 将解压后的Zookeeper复制到以下三个目录

```
/user/local/zookeeper-cluster/zookeeper-1
/user/local/zookeeper-cluster/zookeeper-2
/user/local/zookeeper-cluster/zookeeper-3
```

5. 配置每一个Zookeeper的dataDir(zoo.cfg)clientPort 分别为 2181, 2182, 2183

修改/user/local/zookeeper-cluster/zookeeper-1/conf/zoo.cfg

```
clientPort=2181
dataDir=/zookeeper-cluster/zookeeper-1/data
```

​	其他也做如上改动

#### 2) 配置集群

1. 在每个zookeeper的data目录下创建一个myid文件，内容分别为1、2、3。这个文件就是记录每个服务器的id
2. 在每个zookeeper的zoo.cfg配置客户端访问端口(clientPort)和集群服务器IP列表

```
server.1=192.168.1.6:2881:3881
server.1=192.168.1.6:2882:3882
server.1=192.168.1.6:2883:3883
```

解释：server.服务器 ID = 服务器IP地址:服务器之间通信端口:服务器之间投票选举端口

#### 3) 启动集群

启动集群就是分别启动每个实例

```shell
zkServer.sh start
```

### 6.3.2 RPC服务接口

项目结构

<img src="https://pic.imgdb.cn/item/5f03265414195aa5940d489b.jpg" style="zoom:200%;" />

在interface下添加接口

```java
package cn.sorie.springbootdubboprovider.service;

public interface UserService {
    public String sayHello();
}
```

### 6.2.3 服务提供者

#### 1）依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>cn.sorie</groupId>
    <artifactId>springboot-dubbo-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-dubbo-provider</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <dubbo.version>2.7.7</dubbo.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.logging.log4j</groupId>
                    <artifactId>log4j-to-slf4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>cn.sorie</groupId>
            <artifactId>springboot-dubbo-interface</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>${dubbo.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <scope>import</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>javax.servlet</groupId>
                    <artifactId>servlet-api</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>




        <!-- https://mvnrepository.com/artifact/com.101tec/zkclient -->
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.11</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.9</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.3.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.3.0</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

#### 2）application.properties

```properties
spring.application.name=dubbo-demo-provider
dubbo.application.id=dubbo-demo-provider
dubbo.application.name=dubbo-demo-provider
dubbo.registry.address=zookeeper://192.168.1.6:2182;zookeeper://192.168.1.6:2181;zookeeper://192.168.1.6:2183;
dubbo.registry.client=curator
dubbo.protocol.name=dubbo
dubbo.protocol.port=20880
dubbo.config-center.timeout=10000
dubbo.scan.base-packages=cn.sorie.springbootdubboprovider.service

```

#### 3)启动类

```java
@EnableDubboConfig
@SpringBootApplication
public class SpringbootDubboProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootDubboProviderApplication.class, args);
    }

}
```

#### 4)服务实现

```java
package cn.sorie.springbootdubboprovider.service;


import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService{
    @Override
    public String sayHello() {
        return "hello";
    }
}
```

### 6.2.4 dubbo-admin搭建

下载tomcat和dubbo-admin.war

将war包放入tomcat的webapps目录下，启动tomcat即可

然后访问即可

http://192.168.1.6:8080/dubbo-admin

服务名和密码都是root

中间遇到8080端口被占用了，改为了8081端口

### 6.2.4 服务消费者

#### 1） 添加依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>cn.sorie</groupId>
    <artifactId>springboot-dubbo-consumer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-dubbo-consumer</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <dubbo.version>2.7.7</dubbo.version>

    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>${dubbo.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <scope>import</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>javax.servlet</groupId>
                    <artifactId>servlet-api</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.9</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.3.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.3.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

#### 2）配置文件

```yml
server:
  port: 8082
spring:
  application:
    name: dubbo-demo-consumer
dubbo:
  application:
    name: dubbo-demo-consumer
  registry:
    address: zookeeper://192.168.1.6:2182,192.168.1.6:2181,192.168.1.6:2183
```

#### 3）启动类

```java
package cn.sorie.springbootdubboconsumer;


import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


//@EnableAutoConfiguration
@SpringBootApplication
public class SpringbootDubboConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootDubboConsumerApplication.class, args);
    }

}
```

#### 4）Controller

```java
package cn.sorie.springbootdubboconsumer.controller;

import cn.sorie.springbootdubboprovider.service.UserService;
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {
    @DubboReference
    private UserService userService;

    @RequestMapping("/sayHello")
    public String sayHello() {
        return userService.sayHello();
    }
}
@RestController
@RequestMapping("/user")
public class UserController {
    @DubboReference
    private UserService userService;

    @RequestMapping("/sayHello")
    public String sayHello() {
        return userService.sayHello();
    }
}
```

### 7. 环境搭建

## 7.1 数据库

### 1）优惠券表

| Field        | Type                | Comment                  |
| ------------ | ------------------- | ------------------------ |
| coupon_id    | bigint(50) NOT NULL | 优惠券ID                 |
| coupon_price | decimal(10,2) NULL  | 优惠券金额               |
| user_id      | bigint(50) NULL     | 用户ID                   |
| order_id     | bigint(32) NULL     | 订单ID                   |
| is_used      | int(1) NULL         | 是否使用 0未使用 1已使用 |
| used_time    | timestamp NULL      | 使用时间                 |

### 2）商品表

| Field        | Type                | Comment  |
| ------------ | ------------------- | -------- |
| goods_id     | bigint(50) NOT NULL | 主键     |
| goods_name   | varchar(255) NULL   | 商品名称 |
| goods_number | int(11) NULL        | 商品库存 |
| goods_price  | decimal(10,2) NULL  | 商品价格 |
| goods_desc   | varchar(255) NULL   | 商品描述 |
| add_time     | timestamp NULL      | 添加时间 |

### 3）订单表

| Field           | Type                | Comment                                      |
| --------------- | ------------------- | -------------------------------------------- |
| order_id        | bigint(50) NOT NULL | 订单ID                                       |
| user_id         | bigint(50) NULL     | 用户ID                                       |
| order_status    | int(1) NULL         | 订单状态 0未确认 1已确认 2已取消 3无效 4退款 |
| pay_status      | int(1) NULL         | 支付状态 0未支付 1支付中 2已支付             |
| shipping_status | int(1) NULL         | 发货状态 0未发货 1已发货 2已退货             |
| address         | varchar(255) NULL   | 收货地址                                     |
| consignee       | varchar(255) NULL   | 收货人                                       |
| goods_id        | bigint(50) NULL     | 商品ID                                       |
| goods_number    | int(11) NULL        | 商品数量                                     |
| goods_price     | decimal(10,2) NULL  | 商品价格                                     |
| goods_amount    | decimal(10,0) NULL  | 商品总价                                     |
| shipping_fee    | decimal(10,2) NULL  | 运费                                         |
| order_amount    | decimal(10,2) NULL  | 订单价格                                     |
| coupon_id       | bigint(50) NULL     | 优惠券ID                                     |
| coupon_paid     | decimal(10,2) NULL  | 优惠券                                       |
| money_paid      | decimal(10,2) NULL  | 已付金额                                     |
| pay_amount      | decimal(10,2) NULL  | 支付金额                                     |
| add_time        | timestamp NULL      | 创建时间                                     |
| confirm_time    | timestamp NULL      | 订单确认时间                                 |
| pay_time        | timestamp NULL      | 支付时间                                     |

### 4）订单商品日志表

| Field        | Type                 | Comment  |
| ------------ | -------------------- | -------- |
| goods_id     | int(11) NOT NULL     | 商品ID   |
| order_id     | varchar(32) NOT NULL | 订单ID   |
| goods_number | int(11) NULL         | 库存数量 |
| log_time     | datetime NULL        | 记录时间 |

### 5）用户表

| Field         | Type                | Comment  |
| ------------- | ------------------- | -------- |
| user_id       | bigint(50) NOT NULL | 用户ID   |
| user_name     | varchar(255) NULL   | 用户姓名 |
| user_password | varchar(255) NULL   | 用户密码 |
| user_mobile   | varchar(255) NULL   | 手机号   |
| user_score    | int(11) NULL        | 积分     |
| user_reg_time | timestamp NULL      | 注册时间 |
| user_money    | decimal(10,0) NULL  | 用户余额 |

### 6）用户余额日志表

| Field          | Type                | Comment                       |
| -------------- | ------------------- | ----------------------------- |
| user_id        | bigint(50) NOT NULL | 用户ID                        |
| order_id       | bigint(50) NOT NULL | 订单ID                        |
| money_log_type | int(1) NOT NULL     | 日志类型 1订单付款 2 订单退款 |
| use_money      | decimal(10,2) NULL  | 操作金额                      |
| create_time    | timestamp NULL      | 日志时间                      |

### 7）订单支付表

| Field      | Type                | Comment            |
| ---------- | ------------------- | ------------------ |
| pay_id     | bigint(50) NOT NULL | 支付编号           |
| order_id   | bigint(50) NULL     | 订单编号           |
| pay_amount | decimal(10,2) NULL  | 支付金额           |
| is_paid    | int(1) NULL         | 是否已支付 1否 2是 |

### 8）MQ消息生产表

| Field       | Type                  | Comment             |
| ----------- | --------------------- | ------------------- |
| id          | varchar(100) NOT NULL | 主键                |
| group_name  | varchar(100) NULL     | 生产者组名          |
| msg_topic   | varchar(100) NULL     | 消息主题            |
| msg_tag     | varchar(100) NULL     | Tag                 |
| msg_key     | varchar(100) NULL     | Key                 |
| msg_body    | varchar(500) NULL     | 消息内容            |
| msg_status  | int(1) NULL           | 0:未处理;1:已经处理 |
| create_time | timestamp NOT NULL    | 记录时间            |

### 9）MQ消息消费表

| Field              | Type                  | Comment                          |
| ------------------ | --------------------- | -------------------------------- |
| msg_id             | varchar(50) NULL      | 消息ID                           |
| group_name         | varchar(100) NOT NULL | 消费者组名                       |
| msg_tag            | varchar(100) NOT NULL | Tag                              |
| msg_key            | varchar(100) NOT NULL | Key                              |
| msg_body           | varchar(500) NULL     | 消息体                           |
| consumer_status    | int(1) NULL           | 0:正在处理;1:处理成功;2:处理失败 |
| consumer_times     | int(1) NULL           | 消费次数                         |
| consumer_timestamp | timestamp NULL        | 消费时间                         |
| remark             | varchar(500) NULL     | 备注                             |

sql

```sql
/*
SQLyog Ultimate v8.32 
MySQL - 5.5.49 : Database - trade
*********************************************************************
*/

/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`trade` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `trade`;

/*Table structure for table `trade_coupon` */

DROP TABLE IF EXISTS `trade_coupon`;

CREATE TABLE `trade_coupon` (
  `coupon_id` bigint(50) NOT NULL COMMENT '优惠券ID',
  `coupon_price` decimal(10,2) DEFAULT NULL COMMENT '优惠券金额',
  `user_id` bigint(50) DEFAULT NULL COMMENT '用户ID',
  `order_id` bigint(32) DEFAULT NULL COMMENT '订单ID',
  `is_used` int(1) DEFAULT NULL COMMENT '是否使用 0未使用 1已使用',
  `used_time` timestamp NULL DEFAULT NULL COMMENT '使用时间',
  PRIMARY KEY (`coupon_id`),
  KEY `FK_trade_coupon` (`user_id`),
  KEY `FK_trade_coupon2` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `trade_coupon` */

insert  into `trade_coupon`(`coupon_id`,`coupon_price`,`user_id`,`order_id`,`is_used`,`used_time`) values (345988230098857984,'20.00',345963634385633280,352537369385242624,1,'2019-07-27 16:58:44');

/*Table structure for table `trade_goods` */

DROP TABLE IF EXISTS `trade_goods`;

CREATE TABLE `trade_goods` (
  `goods_id` bigint(50) NOT NULL AUTO_INCREMENT,
  `goods_name` varchar(255) DEFAULT NULL COMMENT '商品名称',
  `goods_number` int(11) DEFAULT NULL COMMENT '商品库存',
  `goods_price` decimal(10,2) DEFAULT NULL COMMENT '商品价格',
  `goods_desc` varchar(255) DEFAULT NULL COMMENT '商品描述',
  `add_time` timestamp NULL DEFAULT NULL COMMENT '添加时间',
  PRIMARY KEY (`goods_id`)
) ENGINE=InnoDB AUTO_INCREMENT=345959443973935105 DEFAULT CHARSET=utf8;

/*Data for the table `trade_goods` */

insert  into `trade_goods`(`goods_id`,`goods_name`,`goods_number`,`goods_price`,`goods_desc`,`add_time`) values (345959443973935104,'Javase课程',999,'1000.00','传智播客出品Java视频课程','2019-07-09 20:38:00');

/*Table structure for table `trade_goods_number_log` */

DROP TABLE IF EXISTS `trade_goods_number_log`;

CREATE TABLE `trade_goods_number_log` (
  `goods_id` bigint(50) NOT NULL COMMENT '商品ID',
  `order_id` bigint(50) NOT NULL COMMENT '订单ID',
  `goods_number` int(11) DEFAULT NULL COMMENT '库存数量',
  `log_time` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`goods_id`,`order_id`),
  KEY `FK_trade_goods_number_log2` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `trade_goods_number_log` */

insert  into `trade_goods_number_log`(`goods_id`,`order_id`,`goods_number`,`log_time`) values (345959443973935104,352537369385242624,-1,'2019-07-27 16:58:44');

/*Table structure for table `trade_mq_consumer_log` */

DROP TABLE IF EXISTS `trade_mq_consumer_log`;

CREATE TABLE `trade_mq_consumer_log` (
  `msg_id` varchar(50) DEFAULT NULL,
  `group_name` varchar(100) NOT NULL,
  `msg_tag` varchar(100) NOT NULL,
  `msg_key` varchar(100) NOT NULL,
  `msg_body` varchar(500) DEFAULT NULL,
  `consumer_status` int(1) DEFAULT NULL COMMENT '0:正在处理;1:处理成功;2:处理失败',
  `consumer_times` int(1) DEFAULT NULL,
  `consumer_timestamp` timestamp NULL DEFAULT NULL,
  `remark` varchar(500) DEFAULT NULL,
  PRIMARY KEY (`group_name`,`msg_tag`,`msg_key`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `trade_mq_consumer_log` */

/*Table structure for table `trade_mq_producer_temp` */

DROP TABLE IF EXISTS `trade_mq_producer_temp`;

CREATE TABLE `trade_mq_producer_temp` (
  `id` varchar(100) NOT NULL,
  `group_name` varchar(100) DEFAULT NULL,
  `msg_topic` varchar(100) DEFAULT NULL,
  `msg_tag` varchar(100) DEFAULT NULL,
  `msg_key` varchar(100) DEFAULT NULL,
  `msg_body` varchar(500) DEFAULT NULL,
  `msg_status` int(1) DEFAULT NULL COMMENT '0:未处理;1:已经处理',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `trade_mq_producer_temp` */

/*Table structure for table `trade_order` */

DROP TABLE IF EXISTS `trade_order`;

CREATE TABLE `trade_order` (
  `order_id` bigint(50) NOT NULL COMMENT '订单ID',
  `user_id` bigint(50) DEFAULT NULL COMMENT '用户ID',
  `order_status` int(1) DEFAULT NULL COMMENT '订单状态 0未确认 1已确认 2已取消 3无效 4退款',
  `pay_status` int(1) DEFAULT NULL COMMENT '支付状态 0未支付 1支付中 2已支付',
  `shipping_status` int(1) DEFAULT NULL COMMENT '发货状态 0未发货 1已发货 2已收货',
  `address` varchar(255) DEFAULT NULL COMMENT '收货地址',
  `consignee` varchar(255) DEFAULT NULL COMMENT '收货人',
  `goods_id` bigint(50) DEFAULT NULL COMMENT '商品ID',
  `goods_number` int(11) DEFAULT NULL COMMENT '商品数量',
  `goods_price` decimal(10,2) DEFAULT NULL COMMENT '商品价格',
  `goods_amount` decimal(10,0) DEFAULT NULL COMMENT '商品总价',
  `shipping_fee` decimal(10,2) DEFAULT NULL COMMENT '运费',
  `order_amount` decimal(10,2) DEFAULT NULL COMMENT '订单价格',
  `coupon_id` bigint(50) DEFAULT NULL COMMENT '优惠券ID',
  `coupon_paid` decimal(10,2) DEFAULT NULL COMMENT '优惠券',
  `money_paid` decimal(10,2) DEFAULT NULL COMMENT '已付金额',
  `pay_amount` decimal(10,2) DEFAULT NULL COMMENT '支付金额',
  `add_time` timestamp NULL DEFAULT NULL COMMENT '创建时间',
  `confirm_time` timestamp NULL DEFAULT NULL COMMENT '订单确认时间',
  `pay_time` timestamp NULL DEFAULT NULL COMMENT '支付时间',
  PRIMARY KEY (`order_id`),
  KEY `FK_trade_order` (`user_id`),
  KEY `FK_trade_order2` (`goods_id`),
  KEY `FK_trade_order3` (`coupon_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `trade_order` */

insert  into `trade_order`(`order_id`,`user_id`,`order_status`,`pay_status`,`shipping_status`,`address`,`consignee`,`goods_id`,`goods_number`,`goods_price`,`goods_amount`,`shipping_fee`,`order_amount`,`coupon_id`,`coupon_paid`,`money_paid`,`pay_amount`,`add_time`,`confirm_time`,`pay_time`) values (352537369385242624,345963634385633280,1,2,NULL,'北京',NULL,345959443973935104,1,'1000.00',NULL,'0.00','1000.00',345988230098857984,'20.00','100.00','880.00','2019-07-27 16:58:44','2019-07-27 16:58:44',NULL);

/*Table structure for table `trade_pay` */

DROP TABLE IF EXISTS `trade_pay`;

CREATE TABLE `trade_pay` (
  `pay_id` bigint(50) NOT NULL COMMENT '支付编号',
  `order_id` bigint(50) DEFAULT NULL COMMENT '订单编号',
  `pay_amount` decimal(10,2) DEFAULT NULL COMMENT '支付金额',
  `is_paid` int(1) DEFAULT NULL COMMENT '是否已支付 0:未付款 1正在付款 2已经付款',
  PRIMARY KEY (`pay_id`),
  KEY `FK_trade_pay` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `trade_pay` */

insert  into `trade_pay`(`pay_id`,`order_id`,`pay_amount`,`is_paid`) values (352542415984402432,352537369385242624,'880.00',2);

/*Table structure for table `trade_user` */

DROP TABLE IF EXISTS `trade_user`;

CREATE TABLE `trade_user` (
  `user_id` bigint(50) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `user_name` varchar(255) DEFAULT NULL COMMENT '用户姓名',
  `user_password` varchar(255) DEFAULT NULL COMMENT '用户密码',
  `user_mobile` varchar(255) DEFAULT NULL COMMENT '手机号',
  `user_score` int(11) DEFAULT NULL COMMENT '积分',
  `user_reg_time` timestamp NULL DEFAULT NULL COMMENT '注册时间',
  `user_money` decimal(10,0) DEFAULT NULL COMMENT '用户余额',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=345963634385633281 DEFAULT CHARSET=utf8;

/*Data for the table `trade_user` */

insert  into `trade_user`(`user_id`,`user_name`,`user_password`,`user_mobile`,`user_score`,`user_reg_time`,`user_money`) values (345963634385633280,'zs','123456','18888888888',100,'2020-10-10 00:00:00','900');

/*Table structure for table `trade_user_money_log` */

DROP TABLE IF EXISTS `trade_user_money_log`;

CREATE TABLE `trade_user_money_log` (
  `user_id` bigint(50) NOT NULL COMMENT '用户ID',
  `order_id` bigint(50) NOT NULL COMMENT '订单ID',
  `money_log_type` int(1) NOT NULL COMMENT '日志类型 1订单付款 2 订单退款',
  `use_money` decimal(10,2) DEFAULT NULL,
  `create_time` timestamp NULL DEFAULT NULL COMMENT '日志时间',
  PRIMARY KEY (`user_id`,`order_id`,`money_log_type`),
  KEY `FK_trade_user_money_log2` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `trade_user_money_log` */

insert  into `trade_user_money_log`(`user_id`,`order_id`,`money_log_type`,`use_money`,`create_time`) values (345963634385633280,352537369385242624,1,'100.00','2019-07-27 16:58:44');

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```



## 7.2 项目初始化

### 7.2.1 工程概览

![](https://pic.imgdb.cn/item/5f0b237314195aa59431dc47.jpg)

### 7.2.2 工程关系

![](https://pic.imgdb.cn/item/5f0b23ba14195aa59431f4a4.jpg)

![](https://pic.imgdb.cn/item/5f0b23d114195aa59431fcbb.jpg)

## 7.3 Mybatis逆向工程使用

 略

## 7.4 公共类介绍

* ID生成器

  IDWorker:Twitter雪花算法

* 异常处理类

  CustomerException：自定义异常类

  CastException：异常抛出类

* 常量类

  ShopCode：系统状态类

* 响应实体类

  Result：封装响应状态

# 8下单业务

##  8.1 下单基本流程

![](https://pic.imgdb.cn/item/5f0c69cd14195aa5948c60ab.jpg)

### 1）接口定义

* IOrderService

```java
public interface IOrderService {
    /**
     * 确认订单
     * @param order
     * @return Result
     */
    Result confirmOrder(TradeOrder order);
}
```

### 2）业务类实现

```java
@Slf4j
@Component
@Service(interfaceClass = IOrderService.class)
public class OrderServiceImpl implements IOrderService {

    @Override
    public Result confirmOrder(TradeOrder order) {
        //1.校验订单
       
        //2.生成预订单
       
        try {
            //3.扣减库存
            
            //4.扣减优惠券
           
            //5.使用余额
           
            //6.确认订单
            
            //7.返回成功状态
           
        } catch (Exception e) {
            //1.确认订单失败,发送消息
            
            //2.返回失败状态
        }

    }
}
```

### 3）校验订单

![](https://pic.imgdb.cn/item/5f145ba414195aa594e34985.jpg)

![](https://pic.imgdb.cn/item/5f145e6714195aa594e42def.jpg)

```java
private void checkOrder(TradeOrder order) {
        //1.校验订单是否存在
        if(order==null){
            CastException.cast(ShopCode.SHOP_ORDER_INVALID);
        }
        //2.校验订单中的商品是否存在
        TradeGoods goods = goodsService.findOne(order.getGoodsId());
        if(goods==null){
            CastException.cast(ShopCode.SHOP_GOODS_NO_EXIST);
        }
        //3.校验下单用户是否存在
        TradeUser user = userService.findOne(order.getUserId());
        if(user==null){
            CastException.cast(ShopCode.SHOP_USER_NO_EXIST);
        }
        //4.校验商品单价是否合法
        if(order.getGoodsPrice().compareTo(goods.getGoodsPrice())!=0){
            CastException.cast(ShopCode.SHOP_GOODS_PRICE_INVALID);
        }
        //5.校验订单商品数量是否合法
        if(order.getGoodsNumber()>=goods.getGoodsNumber()){
            CastException.cast(ShopCode.SHOP_GOODS_NUM_NOT_ENOUGH);
        }

        log.info("校验订单通过");
}
```

### 4）生成预订单

![](https://pic.imgdb.cn/item/5f145e9514195aa594e43d50.jpg)

![](https://pic.imgdb.cn/item/5f145eb714195aa594e44914.jpg)

![](https://pic.imgdb.cn/item/5f145eda14195aa594e45312.jpg)



```java
private Long savePreOrder(TradeOrder order) {
        //1.设置订单状态为不可见
        order.setOrderStatus(ShopCode.SHOP_ORDER_NO_CONFIRM.getCode());
        //2.订单ID
        order.setOrderId(idWorker.nextId());
        //核算运费是否正确
        BigDecimal shippingFee = calculateShippingFee(order.getOrderAmount());
        if (order.getShippingFee().compareTo(shippingFee) != 0) {
            CastException.cast(ShopCode.SHOP_ORDER_SHIPPINGFEE_INVALID);
        }
        //3.计算订单总价格是否正确
        BigDecimal orderAmount = order.getGoodsPrice().multiply(new BigDecimal(order.getGoodsNumber()));
        orderAmount.add(shippingFee);
        if (orderAmount.compareTo(order.getOrderAmount()) != 0) {
            CastException.cast(ShopCode.SHOP_ORDERAMOUNT_INVALID);
        }

        //4.判断优惠券信息是否合法
        Long couponId = order.getCouponId();
        if (couponId != null) {
            TradeCoupon coupon = couponService.findOne(couponId);
            //优惠券不存在
            if (coupon == null) {
                CastException.cast(ShopCode.SHOP_COUPON_NO_EXIST);
            }
            //优惠券已经使用
            if ((ShopCode.SHOP_COUPON_ISUSED.getCode().toString())
                .equals(coupon.getIsUsed().toString())) {
                CastException.cast(ShopCode.SHOP_COUPON_INVALIED);
            }
            order.setCouponPaid(coupon.getCouponPrice());
        } else {
            order.setCouponPaid(BigDecimal.ZERO);
        }

        //5.判断余额是否正确
        BigDecimal moneyPaid = order.getMoneyPaid();
        if (moneyPaid != null) {
            //比较余额是否大于0
            int r = order.getMoneyPaid().compareTo(BigDecimal.ZERO);
            //余额小于0
            if (r == -1) {
                CastException.cast(ShopCode.SHOP_MONEY_PAID_LESS_ZERO);
            }
            //余额大于0
            if (r == 1) {
                //查询用户信息
                TradeUser user = userService.findOne(order.getUserId());
                if (user == null) {
                    CastException.cast(ShopCode.SHOP_USER_NO_EXIST);
                }
            //比较余额是否大于用户账户余额
            if (user.getUserMoney().compareTo(order.getMoneyPaid().longValue()) == -1) {
                CastException.cast(ShopCode.SHOP_MONEY_PAID_INVALID);
            }
            order.setMoneyPaid(order.getMoneyPaid());
        }
    } else {
        order.setMoneyPaid(BigDecimal.ZERO);
    }
    //计算订单支付总价
    order.setPayAmount(orderAmount.subtract(order.getCouponPaid())
                       .subtract(order.getMoneyPaid()));
    //设置订单添加时间
    order.setAddTime(new Date());

    //保存预订单
    int r = orderMapper.insert(order);
    if (ShopCode.SHOP_SUCCESS.getCode() != r) {
        CastException.cast(ShopCode.SHOP_ORDER_SAVE_ERROR);
    }
    log.info("订单:["+order.getOrderId()+"]预订单生成成功");
    return order.getOrderId();
}
```

### 5）扣减库存

* 通过dubbo调用商品服务完成扣减库存

```java
private void reduceGoodsNum(TradeOrder order) {
        TradeGoodsNumberLog goodsNumberLog = new TradeGoodsNumberLog();
        goodsNumberLog.setGoodsId(order.getGoodsId());
        goodsNumberLog.setOrderId(order.getOrderId());
        goodsNumberLog.setGoodsNumber(order.getGoodsNumber());
        Result result = goodsService.reduceGoodsNum(goodsNumberLog);
        if (result.getSuccess().equals(ShopCode.SHOP_FAIL.getSuccess())) {
            CastException.cast(ShopCode.SHOP_REDUCE_GOODS_NUM_FAIL);
        }
        log.info("订单:["+order.getOrderId()+"]扣减库存["+order.getGoodsNumber()+"个]成功");
    }
```

* 商品服务GoodsService扣减库存

```java
@Override
public Result reduceGoodsNum(TradeGoodsNumberLog goodsNumberLog) {
    if (goodsNumberLog == null ||
            goodsNumberLog.getGoodsNumber() == null ||
            goodsNumberLog.getOrderId() == null ||
            goodsNumberLog.getGoodsNumber() == null ||
            goodsNumberLog.getGoodsNumber().intValue() <= 0) {
        CastException.cast(ShopCode.SHOP_REQUEST_PARAMETER_VALID);
    }
    TradeGoods goods = goodsMapper.selectByPrimaryKey(goodsNumberLog.getGoodsId());
    if(goods.getGoodsNumber()<goodsNumberLog.getGoodsNumber()){
        //库存不足
        CastException.cast(ShopCode.SHOP_GOODS_NUM_NOT_ENOUGH);
    }
    //减库存
    goods.setGoodsNumber(goods.getGoodsNumber()-goodsNumberLog.getGoodsNumber());
    goodsMapper.updateByPrimaryKey(goods);


    //记录库存操作日志
    goodsNumberLog.setGoodsNumber(-(goodsNumberLog.getGoodsNumber()));
    goodsNumberLog.setLogTime(new Date());
    goodsNumberLogMapper.insert(goodsNumberLog);

    return new Result(ShopCode.SHOP_SUCCESS.getSuccess(),ShopCode.SHOP_SUCCESS.getMessage());
}
```

### 6）扣减优惠券

* 通过dubbo完成扣减优惠券

```java
private void changeCoponStatus(TradeOrder order) {
    //判断用户是否使用优惠券
    if (!StringUtils.isEmpty(order.getCouponId())) {
        //封装优惠券对象
        TradeCoupon coupon = couponService.findOne(order.getCouponId());
        coupon.setIsUsed(ShopCode.SHOP_COUPON_ISUSED.getCode());
        coupon.setUsedTime(new Date());
        coupon.setOrderId(order.getOrderId());
        Result result = couponService.changeCouponStatus(coupon);
        //判断执行结果
        if (result.getSuccess().equals(ShopCode.SHOP_FAIL.getSuccess())) {
            //优惠券使用失败
            CastException.cast(ShopCode.SHOP_COUPON_USE_FAIL);
        }
        log.info("订单:["+order.getOrderId()+"]使用扣减优惠券["+coupon.getCouponPrice()+"元]成功");
    }

}
```

* 优惠券服务CouponService更改优惠券状态

```java
@Override
public Result changeCouponStatus(TradeCoupon coupon) {
    try {
        //判断请求参数是否合法
        if (coupon == null || StringUtils.isEmpty(coupon.getCouponId())) {
            CastException.cast(ShopCode.SHOP_REQUEST_PARAMETER_VALID);
        }
		//更新优惠券状态为已使用
        couponMapper.updateByPrimaryKey(coupon);
        return new Result(ShopCode.SHOP_SUCCESS.getSuccess(), ShopCode.SHOP_SUCCESS.getMessage());
    } catch (Exception e) {
        return new Result(ShopCode.SHOP_FAIL.getSuccess(), ShopCode.SHOP_FAIL.getMessage());
    }
}
```

### 7）扣减用户余额

* 通过用户服务完成扣减余额

```java
private void reduceMoneyPaid(TradeOrder order) {
    //判断订单中使用的余额是否合法
    if (order.getMoneyPaid() != null && order.getMoneyPaid().compareTo(BigDecimal.ZERO) == 1) {
        TradeUserMoneyLog userMoneyLog = new TradeUserMoneyLog();
        userMoneyLog.setOrderId(order.getOrderId());
        userMoneyLog.setUserId(order.getUserId());
        userMoneyLog.setUseMoney(order.getMoneyPaid());
        userMoneyLog.setMoneyLogType(ShopCode.SHOP_USER_MONEY_PAID.getCode());
        //扣减余额
        Result result = userService.changeUserMoney(userMoneyLog);
        if (result.getSuccess().equals(ShopCode.SHOP_FAIL.getSuccess())) {
            CastException.cast(ShopCode.SHOP_USER_MONEY_REDUCE_FAIL);
        }
        log.info("订单:["+order.getOrderId()+"扣减余额["+order.getMoneyPaid()+"元]成功]");
    }
}
```

* 用户服务UserService,更新余额

![](https://pic.imgdb.cn/item/5f145cd614195aa594e3a838.jpg)

![](https://pic.imgdb.cn/item/5f145cb614195aa594e39d7b.jpg)

```java
@Override
public Result changeUserMoney(TradeUserMoneyLog userMoneyLog) {
    //判断请求参数是否合法
    if (userMoneyLog == null
            || userMoneyLog.getUserId() == null
            || userMoneyLog.getUseMoney() == null
            || userMoneyLog.getOrderId() == null
            || userMoneyLog.getUseMoney().compareTo(BigDecimal.ZERO) <= 0) {
        CastException.cast(ShopCode.SHOP_REQUEST_PARAMETER_VALID);
    }

    //查询该订单是否存在付款记录
    TradeUserMoneyLogExample userMoneyLogExample = new TradeUserMoneyLogExample();
    userMoneyLogExample.createCriteria()
            .andUserIdEqualTo(userMoneyLog.getUserId())
            .andOrderIdEqualTo(userMoneyLog.getOrderId());
   int count = userMoneyLogMapper.countByExample(userMoneyLogExample);
   TradeUser tradeUser = new TradeUser();
   tradeUser.setUserId(userMoneyLog.getUserId());
   tradeUser.setUserMoney(userMoneyLog.getUseMoney().longValue());
   //判断余额操作行为
   //【付款操作】
   if (userMoneyLog.getMoneyLogType().equals(ShopCode.SHOP_USER_MONEY_PAID.getCode())) {
           //订单已经付款，则抛异常
           if (count > 0) {
                CastException.cast(ShopCode.SHOP_ORDER_PAY_STATUS_IS_PAY);
            }
       	   //用户账户扣减余额
           userMapper.reduceUserMoney(tradeUser);
       }
    //【退款操作】
    if (userMoneyLog.getMoneyLogType().equals(ShopCode.SHOP_USER_MONEY_REFUND.getCode())) {
         //如果订单未付款,则不能退款,抛异常
         if (count == 0) {
         CastException.cast(ShopCode.SHOP_ORDER_PAY_STATUS_NO_PAY);
     }
     //防止多次退款
     userMoneyLogExample = new TradeUserMoneyLogExample();
     userMoneyLogExample.createCriteria()
             .andUserIdEqualTo(userMoneyLog.getUserId())
                .andOrderIdEqualTo(userMoneyLog.getOrderId())
                .andMoneyLogTypeEqualTo(ShopCode.SHOP_USER_MONEY_REFUND.getCode());
     count = userMoneyLogMapper.countByExample(userMoneyLogExample);
     if (count > 0) {
         CastException.cast(ShopCode.SHOP_USER_MONEY_REFUND_ALREADY);
     }
     	//用户账户添加余额
        userMapper.addUserMoney(tradeUser);
    }


    //记录用户使用余额日志
    userMoneyLog.setCreateTime(new Date());
    userMoneyLogMapper.insert(userMoneyLog);
    return new Result(ShopCode.SHOP_SUCCESS.getSuccess(),ShopCode.SHOP_SUCCESS.getMessage());
}
```

### 8）确认订单 

```java
private void updateOrderStatus(TradeOrder order) {
    order.setOrderStatus(ShopCode.SHOP_ORDER_CONFIRM.getCode());
    order.setPayStatus(ShopCode.SHOP_ORDER_PAY_STATUS_NO_PAY.getCode());
    order.setConfirmTime(new Date());
    int r = orderMapper.updateByPrimaryKey(order);
    if (r <= 0) {
        CastException.cast(ShopCode.SHOP_ORDER_CONFIRM_FAIL);
    }
    log.info("订单:["+order.getOrderId()+"]状态修改成功");
}
```

### 9）小结

```java
@Override
public Result confirmOrder(TradeOrder order) {
    //1.校验订单
    checkOrder(order);
    //2.生成预订单
    Long orderId = savePreOrder(order);
    order.setOrderId(orderId);
    try {
        //3.扣减库存
        reduceGoodsNum(order);
        //4.扣减优惠券
        changeCoponStatus(order);
        //5.使用余额
        reduceMoneyPaid(order);
        //6.确认订单
        updateOrderStatus(order);
        log.info("订单:["+orderId+"]确认成功");
        return new Result(ShopCode.SHOP_SUCCESS.getSuccess(), ShopCode.SHOP_SUCCESS.getMessage());
    } catch (Exception e) {
        //确认订单失败,发送消息
        ...
        return new Result(ShopCode.SHOP_FAIL.getSuccess(), ShopCode.SHOP_FAIL.getMessage());
    }
}
```

## 8.2 失败补偿机制

### 8.2.1 消息发送方

* 配置RocketMQ属性值

```properties
rocketmq.name-server=192.168.25.135:9876;192.168.25.138:9876
rocketmq.producer.group=orderProducerGroup

mq.order.consumer.group.name=order_orderTopic_cancel_group
mq.order.topic=orderTopic
mq.order.tag.confirm=order_confirm
mq.order.tag.cancel=order_cancel
```

* 注入模板类和属性值信息

```java
 @Autowired
 private RocketMQTemplate rocketMQTemplate;

 @Value("${mq.order.topic}")
 private String topic;

 @Value("${mq.order.tag.cancel}")
 private String cancelTag;
```

* 发送下单失败消息

```java
@Override
public Result confirmOrder(TradeOrder order) {
    //1.校验订单
    //2.生成预订
    try {
        //3.扣减库存
        //4.扣减优惠券
        //5.使用余额
        //6.确认订单
    } catch (Exception e) {
        //确认订单失败,发送消息
        CancelOrderMQ cancelOrderMQ = new CancelOrderMQ();
        cancelOrderMQ.setOrderId(order.getOrderId());
        cancelOrderMQ.setCouponId(order.getCouponId());
        cancelOrderMQ.setGoodsId(order.getGoodsId());
        cancelOrderMQ.setGoodsNumber(order.getGoodsNumber());
        cancelOrderMQ.setUserId(order.getUserId());
        cancelOrderMQ.setUserMoney(order.getMoneyPaid());
        try {
            sendMessage(topic, 
                        cancelTag, 
                        cancelOrderMQ.getOrderId().toString(), 
                    JSON.toJSONString(cancelOrderMQ));
    } catch (Exception e1) {
        e1.printStackTrace();
            CastException.cast(ShopCode.SHOP_MQ_SEND_MESSAGE_FAIL);
        }
        return new Result(ShopCode.SHOP_FAIL.getSuccess(), ShopCode.SHOP_FAIL.getMessage());
    }
}
```

```java
private void sendMessage(String topic, String tags, String keys, String body) throws Exception {
    //判断Topic是否为空
    if (StringUtils.isEmpty(topic)) {
        CastException.cast(ShopCode.SHOP_MQ_TOPIC_IS_EMPTY);
    }
    //判断消息内容是否为空
    if (StringUtils.isEmpty(body)) {
        CastException.cast(ShopCode.SHOP_MQ_MESSAGE_BODY_IS_EMPTY);
    }
    //消息体
    Message message = new Message(topic, tags, keys, body.getBytes());
    //发送消息
    rocketMQTemplate.getProducer().send(message);
}
```

### 8.2.2 消费接收方

* 配置RocketMQ属性值

```properties
rocketmq.name-server=192.168.25.135:9876;192.168.25.138:9876
mq.order.consumer.group.name=order_orderTopic_cancel_group
mq.order.topic=orderTopic
```

* 创建监听类，消费消息

```java
@Slf4j
@Component
@RocketMQMessageListener(topic = "${mq.order.topic}", 
                         consumerGroup = "${mq.order.consumer.group.name}",
                         messageModel = MessageModel.BROADCASTING)
public class CancelOrderConsumer implements RocketMQListener<MessageExt>{

    @Override
    public void onMessage(MessageExt messageExt) {
        ...
    }
}
```

#### 1）回退库存

* 流程分析

![](https://pic.imgdb.cn/item/5f145d0b14195aa594e3b848.jpg)

![](https://pic.imgdb.cn/item/5f145d5f14195aa594e3d3b8.jpg)

![](https://pic.imgdb.cn/item/5f145d4c14195aa594e3ce57.jpg)

* 消息消费者

```java
@Slf4j
@Component
@RocketMQMessageListener(topic = "${mq.order.topic}",consumerGroup = "${mq.order.consumer.group.name}",messageModel = MessageModel.BROADCASTING )
public class CancelMQListener implements RocketMQListener<MessageExt>{


    @Value("${mq.order.consumer.group.name}")
    private String groupName;

    @Autowired
    private TradeGoodsMapper goodsMapper;

    @Autowired
    private TradeMqConsumerLogMapper mqConsumerLogMapper;

    @Autowired
    private TradeGoodsNumberLogMapper goodsNumberLogMapper;

    @Override
    public void onMessage(MessageExt messageExt) {
        String msgId=null;
        String tags=null;
        String keys=null;
        String body=null;
        try {
            //1. 解析消息内容
            msgId = messageExt.getMsgId();
            tags= messageExt.getTags();
            keys= messageExt.getKeys();
            body= new String(messageExt.getBody(),"UTF-8");

            log.info("接受消息成功");

            //2. 查询消息消费记录
            TradeMqConsumerLogKey primaryKey = new TradeMqConsumerLogKey();
            primaryKey.setMsgTag(tags);
            primaryKey.setMsgKey(keys);
            primaryKey.setGroupName(groupName);
            TradeMqConsumerLog mqConsumerLog = mqConsumerLogMapper.selectByPrimaryKey(primaryKey);

            if(mqConsumerLog!=null){
                //3. 判断如果消费过...
                //3.1 获得消息处理状态
                Integer status = mqConsumerLog.getConsumerStatus();
                //处理过...返回
                if(ShopCode.SHOP_MQ_MESSAGE_STATUS_SUCCESS.getCode().intValue()==status.intValue()){
                    log.info("消息:"+msgId+",已经处理过");
                    return;
                }

                //正在处理...返回
                if(ShopCode.SHOP_MQ_MESSAGE_STATUS_PROCESSING.getCode().intValue()==status.intValue()){
                    log.info("消息:"+msgId+",正在处理");
                    return;
                }

                //处理失败
                if(ShopCode.SHOP_MQ_MESSAGE_STATUS_FAIL.getCode().intValue()==status.intValue()){
                    //获得消息处理次数
                    Integer times = mqConsumerLog.getConsumerTimes();
                    if(times>3){
                        log.info("消息:"+msgId+",消息处理超过3次,不能再进行处理了");
                        return;
                    }
                    mqConsumerLog.setConsumerStatus(ShopCode.SHOP_MQ_MESSAGE_STATUS_PROCESSING.getCode());

                    //使用数据库乐观锁更新
                    TradeMqConsumerLogExample example = new TradeMqConsumerLogExample();
                    TradeMqConsumerLogExample.Criteria criteria = example.createCriteria();
                    criteria.andMsgTagEqualTo(mqConsumerLog.getMsgTag());
                    criteria.andMsgKeyEqualTo(mqConsumerLog.getMsgKey());
                    criteria.andGroupNameEqualTo(groupName);
                    criteria.andConsumerTimesEqualTo(mqConsumerLog.getConsumerTimes());
                    int r = mqConsumerLogMapper.updateByExampleSelective(mqConsumerLog, example);
                    if(r<=0){
                        //未修改成功,其他线程并发修改
                        log.info("并发修改,稍后处理");
                    }
                }

            }else{
                //4. 判断如果没有消费过...
                mqConsumerLog = new TradeMqConsumerLog();
                mqConsumerLog.setMsgTag(tags);
                mqConsumerLog.setMsgKey(keys);
                mqConsumerLog.setConsumerStatus(ShopCode.SHOP_MQ_MESSAGE_STATUS_PROCESSING.getCode());
                mqConsumerLog.setMsgBody(body);
                mqConsumerLog.setMsgId(msgId);
                mqConsumerLog.setConsumerTimes(0);

                //将消息处理信息添加到数据库
                mqConsumerLogMapper.insert(mqConsumerLog);
            }
            //5. 回退库存
            MQEntity mqEntity = JSON.parseObject(body, MQEntity.class);
            Long goodsId = mqEntity.getGoodsId();
            TradeGoods goods = goodsMapper.selectByPrimaryKey(goodsId);
            goods.setGoodsNumber(goods.getGoodsNumber()+mqEntity.getGoodsNum());
            goodsMapper.updateByPrimaryKey(goods);

            //记录库存操作日志
            TradeGoodsNumberLog goodsNumberLog = new TradeGoodsNumberLog();
            goodsNumberLog.setOrderId(mqEntity.getOrderId());
            goodsNumberLog.setGoodsId(goodsId);
            goodsNumberLog.setGoodsNumber(mqEntity.getGoodsNum());
            goodsNumberLog.setLogTime(new Date());
            goodsNumberLogMapper.insert(goodsNumberLog);

            //6. 将消息的处理状态改为成功
            mqConsumerLog.setConsumerStatus(ShopCode.SHOP_MQ_MESSAGE_STATUS_SUCCESS.getCode());
            mqConsumerLog.setConsumerTimestamp(new Date());
            mqConsumerLogMapper.updateByPrimaryKey(mqConsumerLog);
            log.info("回退库存成功");
        } catch (Exception e) {
            e.printStackTrace();
            TradeMqConsumerLogKey primaryKey = new TradeMqConsumerLogKey();
            primaryKey.setMsgTag(tags);
            primaryKey.setMsgKey(keys);
            primaryKey.setGroupName(groupName);
            TradeMqConsumerLog mqConsumerLog = mqConsumerLogMapper.selectByPrimaryKey(primaryKey);
            if(mqConsumerLog==null){
                //数据库未有记录
                mqConsumerLog = new TradeMqConsumerLog();
                mqConsumerLog.setMsgTag(tags);
                mqConsumerLog.setMsgKey(keys);
                mqConsumerLog.setConsumerStatus(ShopCode.SHOP_MQ_MESSAGE_STATUS_FAIL.getCode());
                mqConsumerLog.setMsgBody(body);
                mqConsumerLog.setMsgId(msgId);
                mqConsumerLog.setConsumerTimes(1);
                mqConsumerLogMapper.insert(mqConsumerLog);
            }else{
                mqConsumerLog.setConsumerTimes(mqConsumerLog.getConsumerTimes()+1);
                mqConsumerLogMapper.updateByPrimaryKeySelective(mqConsumerLog);
            }
        }

    }
}
```

#### 2）回退优惠券

```java
@Slf4j
@Component
@RocketMQMessageListener(topic = "${mq.order.topic}",consumerGroup = "${mq.order.consumer.group.name}",messageModel = MessageModel.BROADCASTING )
public class CancelMQListener implements RocketMQListener<MessageExt>{


    @Autowired
    private TradeCouponMapper couponMapper;

    @Override
    public void onMessage(MessageExt message) {

        try {
            //1. 解析消息内容
            String body = new String(message.getBody(), "UTF-8");
            MQEntity mqEntity = JSON.parseObject(body, MQEntity.class);
            log.info("接收到消息");
            //2. 查询优惠券信息
            TradeCoupon coupon = couponMapper.selectByPrimaryKey(mqEntity.getCouponId());
            //3.更改优惠券状态
            coupon.setUsedTime(null);
            coupon.setIsUsed(ShopCode.SHOP_COUPON_UNUSED.getCode());
            coupon.setOrderId(null);
            couponMapper.updateByPrimaryKey(coupon);
            log.info("回退优惠券成功");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            log.error("回退优惠券失败");
        }

    }
}
```

#### 3）回退余额

```java
@Slf4j
@Component
@RocketMQMessageListener(topic = "${mq.order.topic}",consumerGroup = "${mq.order.consumer.group.name}",messageModel = MessageModel.BROADCASTING )
public class CancelMQListener implements RocketMQListener<MessageExt>{


    @Autowired
    private IUserService userService;

    @Override
    public void onMessage(MessageExt messageExt) {

        try {
            //1.解析消息
            String body = new String(messageExt.getBody(), "UTF-8");
            MQEntity mqEntity = JSON.parseObject(body, MQEntity.class);
            log.info("接收到消息");
            if(mqEntity.getUserMoney()!=null && mqEntity.getUserMoney().compareTo(BigDecimal.ZERO)>0){
                //2.调用业务层,进行余额修改
                TradeUserMoneyLog userMoneyLog = new TradeUserMoneyLog();
                userMoneyLog.setUseMoney(mqEntity.getUserMoney());
                userMoneyLog.setMoneyLogType(ShopCode.SHOP_USER_MONEY_REFUND.getCode());
                userMoneyLog.setUserId(mqEntity.getUserId());
                userMoneyLog.setOrderId(mqEntity.getOrderId());
                userService.updateMoneyPaid(userMoneyLog);
                log.info("余额回退成功");
            }
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            log.error("余额回退失败");
        }

    }
}
```

#### 4）取消订单

```java
@Override
    public void onMessage(MessageExt messageExt) {
        String body = new String(messageExt.getBody(), "UTF-8");
        String msgId = messageExt.getMsgId();
        String tags = messageExt.getTags();
        String keys = messageExt.getKeys();
        log.info("CancelOrderProcessor receive message:"+messageExt);
        CancelOrderMQ cancelOrderMQ = JSON.parseObject(body, CancelOrderMQ.class);
        TradeOrder order = orderService.findOne(cancelOrderMQ.getOrderId());
		order.setOrderStatus(ShopCode.SHOP_ORDER_CANCEL.getCode());
        orderService.changeOrderStatus(order);
        log.info("订单:["+order.getOrderId()+"]状态设置为取消");
        return order;
    }
```

## 8.3 测试

### 1）准备测试环境

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = ShopOrderServiceApplication.class)
public class OrderTest {

    @Autowired
    private IOrderService orderService;
}
```

### 2）准备测试数据

* 用户数据
* 商品数据
* 优惠券数据

### 3）测试下单成功流程

```java
@Test    
public void add(){
    Long goodsId=XXXL;
    Long userId=XXXL;
    Long couponId=XXXL;

    TradeOrder order = new TradeOrder();
    order.setGoodsId(goodsId);
    order.setUserId(userId);
    order.setGoodsNumber(1);
    order.setAddress("北京");
    order.setGoodsPrice(new BigDecimal("5000"));
    order.setOrderAmount(new BigDecimal("5000"));
    order.setMoneyPaid(new BigDecimal("100"));
    order.setCouponId(couponId);
    order.setShippingFee(new BigDecimal(0));
    orderService.confirmOrder(order);
}
```

执行完毕后,查看数据库中用户的余额、优惠券数据，及订单的状态数据

### 4）测试下单失败流程

代码同上。

执行完毕后，查看用户的余额、优惠券数据是否发生更改，订单的状态是否为取消。

# 9.支付业务

## 9.1 创建支付订单

![](https://pic.imgdb.cn/item/5f145da214195aa594e3e91e.jpg)

```java
public Result createPayment(TradePay tradePay) {
    //查询订单支付状态
    try {
        TradePayExample payExample = new TradePayExample();
        TradePayExample.Criteria criteria = payExample.createCriteria();
        criteria.andOrderIdEqualTo(tradePay.getOrderId());
        criteria.andIsPaidEqualTo(ShopCode.SHOP_ORDER_PAY_STATUS_IS_PAY.getCode());
        int count = tradePayMapper.countByExample(payExample);
        if (count > 0) {
            CastException.cast(ShopCode.SHOP_ORDER_PAY_STATUS_IS_PAY);
        }

        long payId = idWorker.nextId();
        tradePay.setPayId(payId);
        tradePay.setIsPaid(ShopCode.SHOP_ORDER_PAY_STATUS_NO_PAY.getCode());
        tradePayMapper.insert(tradePay);
        log.info("创建支付订单成功:" + payId);
    } catch (Exception e) {
        return new Result(ShopCode.SHOP_FAIL.getSuccess(), ShopCode.SHOP_FAIL.getMessage());
    }
    return new Result(ShopCode.SHOP_SUCCESS.getSuccess(), ShopCode.SHOP_SUCCESS.getMessage());
}
```

## 9.2 支付回调 

###  9.2.1 流程分析

![](https://pic.imgdb.cn/item/5f145df714195aa594e4085c.jpg)

![](https://pic.imgdb.cn/item/5f145e3214195aa594e419e7.jpg)

### 9.2.2 代码实现

```java
public Result callbackPayment(TradePay tradePay) {

    if (tradePay.getIsPaid().equals(ShopCode.SHOP_ORDER_PAY_STATUS_IS_PAY.getCode())) {
        tradePay = tradePayMapper.selectByPrimaryKey(tradePay.getPayId());
        if (tradePay == null) {
            CastException.cast(ShopCode.SHOP_PAYMENT_NOT_FOUND);
        }
        tradePay.setIsPaid(ShopCode.SHOP_ORDER_PAY_STATUS_IS_PAY.getCode());
        int i = tradePayMapper.updateByPrimaryKeySelective(tradePay);
        //更新成功代表支付成功
        if (i == 1) {
            TradeMqProducerTemp mqProducerTemp = new TradeMqProducerTemp();
            mqProducerTemp.setId(String.valueOf(idWorker.nextId()));
            mqProducerTemp.setGroupName("payProducerGroup");
            mqProducerTemp.setMsgKey(String.valueOf(tradePay.getPayId()));
            mqProducerTemp.setMsgTag(topic);
            mqProducerTemp.setMsgBody(JSON.toJSONString(tradePay));
            mqProducerTemp.setCreateTime(new Date());
            mqProducerTempMapper.insert(mqProducerTemp);
            TradePay finalTradePay = tradePay;
            executorService.submit(new Runnable() {
                @Override
                public void run() {
                    try {
                        SendResult sendResult = sendMessage(topic, 
                                                            tag, 
                                                            finalTradePay.getPayId(), 
                                                            JSON.toJSONString(finalTradePay));
                        log.info(JSON.toJSONString(sendResult));
                        if (SendStatus.SEND_OK.equals(sendResult.getSendStatus())) {
                            mqProducerTempMapper.deleteByPrimaryKey(mqProducerTemp.getId());
                            System.out.println("删除消息表成功");
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            });
        } else {
            CastException.cast(ShopCode.SHOP_PAYMENT_IS_PAID);
        }
    }
    return new Result(ShopCode.SHOP_SUCCESS.getSuccess(), ShopCode.SHOP_SUCCESS.getMessage());
}
```

#### 线程池优化消息发送逻辑

* 创建线程池对象

```java
@Bean
public ThreadPoolTaskExecutor getThreadPool() {

    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();

    executor.setCorePoolSize(4);

    executor.setMaxPoolSize(8);

    executor.setQueueCapacity(100);

    executor.setKeepAliveSeconds(60);

    executor.setThreadNamePrefix("Pool-A");

    executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());

    executor.initialize();

    return executor;

}
```

* 使用线程池

```java
@Autowired
private ThreadPoolTaskExecutor executorService;

executorService.submit(new Runnable() {
    @Override
    public void run() {
        try {
            SendResult sendResult = sendMessage(topic, tag, finalTradePay.getPayId(), JSON.toJSONString(finalTradePay));
            log.info(JSON.toJSONString(sendResult));
            if (SendStatus.SEND_OK.equals(sendResult.getSendStatus())) {
                mqProducerTempMapper.deleteByPrimaryKey(mqProducerTemp.getId());
                System.out.println("删除消息表成功");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```



### 9.2.3 处理消息

支付成功后，支付服务payService发送MQ消息，订单服务、用户服务、日志服务需要订阅消息进行处理

1. 订单服务修改订单状态为已支付
2. 日志服务记录支付日志
3. 用户服务负责给用户增加积分

以下用订单服务为例说明消息的处理情况

#### 1）配置RocketMQ属性值

```properties
mq.pay.topic=payTopic
mq.pay.consumer.group.name=pay_payTopic_group
```

#### 2）消费消息

* 在订单服务中，配置公共的消息处理类

```java
public class BaseConsumer {

    public TradeOrder handleMessage(IOrderService 
                                    orderService, 
                                    MessageExt messageExt,Integer code) throws Exception {
        //解析消息内容
        String body = new String(messageExt.getBody(), "UTF-8");
        String msgId = messageExt.getMsgId();
        String tags = messageExt.getTags();
        String keys = messageExt.getKeys();
        OrderMQ orderMq = JSON.parseObject(body, OrderMQ.class);
        
        //查询
        TradeOrder order = orderService.findOne(orderMq.getOrderId());

        if(ShopCode.SHOP_ORDER_MESSAGE_STATUS_CANCEL.getCode().equals(code)){
            order.setOrderStatus(ShopCode.SHOP_ORDER_CANCEL.getCode());
        }

        if(ShopCode.SHOP_ORDER_MESSAGE_STATUS_ISPAID.getCode().equals(code)){
            order.setPayStatus(ShopCode.SHOP_ORDER_PAY_STATUS_IS_PAY.getCode());
        }
        orderService.changeOrderStatus(order);
        return order;
    }

}
```

* 接受订单支付成功消息

```java
@Slf4j
@Component
@RocketMQMessageListener(topic = "${mq.pay.topic}", 
                         consumerGroup = "${mq.pay.consumer.group.name}")
public class PayConsumer extends BaseConsumer implements RocketMQListener<MessageExt> {

    @Autowired
    private IOrderService orderService;

    @Override
    public void onMessage(MessageExt messageExt) {
        try {
            log.info("CancelOrderProcessor receive message:"+messageExt);
            TradeOrder order = handleMessage(orderService, 
                                             messageExt, 
                                             ShopCode.SHOP_ORDER_MESSAGE_STATUS_ISPAID.getCode());
            log.info("订单:["+order.getOrderId()+"]支付成功");
        } catch (Exception e) {
            e.printStackTrace();
            log.error("订单支付失败");
        }
    }
}
```

# 10. 整体联调

通过Rest客户端请求shop-order-web和shop-pay-web完成下单和支付操作

## 10.1 准备工作

### 1）配置RestTemplate类

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @ConditionalOnMissingBean({ RestOperations.class, RestTemplate.class })
    public RestTemplate restTemplate(ClientHttpRequestFactory factory) {

        RestTemplate restTemplate = new RestTemplate(factory);

        // 使用 utf-8 编码集的 conver 替换默认的 conver（默认的 string conver 的编码集为"ISO-8859-1"）
        List<HttpMessageConverter<?>> messageConverters = restTemplate.getMessageConverters();
        Iterator<HttpMessageConverter<?>> iterator = messageConverters.iterator();
        while (iterator.hasNext()) {
            HttpMessageConverter<?> converter = iterator.next();
            if (converter instanceof StringHttpMessageConverter) {
                iterator.remove();
            }
        }
        messageConverters.add(new StringHttpMessageConverter(Charset.forName("UTF-8")));

        return restTemplate;
    }

    @Bean
    @ConditionalOnMissingBean({ClientHttpRequestFactory.class})
    public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        // ms
        factory.setReadTimeout(15000);
        // ms
        factory.setConnectTimeout(15000);
        return factory;
    }
}
```

### 2）配置请求地址

* 订单系统

```properties
server.host=http://localhost
server.servlet.path=/order-web
server.port=8080
shop.order.baseURI=${server.host}:${server.port}${server.servlet.path}
shop.order.confirm=/order/confirm
```

* 支付系统

```properties
server.host=http://localhost
server.servlet.path=/pay-web
server.port=9090
shop.pay.baseURI=${server.host}:${server.port}${server.servlet.path}
shop.pay.createPayment=/pay/createPayment
shop.pay.callbackPayment=/pay/callbackPayment
```

## 10.2 下单测试

 ```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = ShopOrderWebApplication.class)
@TestPropertySource("classpath:application.properties")
public class OrderTest {

    @Autowired
    private RestTemplate restTemplate;

    @Value("${shop.order.baseURI}")
    private String baseURI;

    @Value("${shop.order.confirm}")
    private String confirmOrderPath;

    @Autowired
    private IDWorker idWorker;
   
   /**
     * 下单
     */
    @Test
    public void confirmOrder(){
        Long goodsId=XXXL;
        Long userId=XXXL;
        Long couponId=XXXL;

        TradeOrder order = new TradeOrder();
        order.setGoodsId(goodsId);
        order.setUserId(userId);
        order.setGoodsNumber(1);
        order.setAddress("北京");
        order.setGoodsPrice(new BigDecimal("5000"));
        order.setOrderAmount(new BigDecimal("5000"));
        order.setMoneyPaid(new BigDecimal("100"));
        order.setCouponId(couponId);
        order.setShippingFee(new BigDecimal(0));

        Result result = restTemplate.postForEntity(baseURI + confirmOrderPath, order, Result.class).getBody();
        System.out.println(result);
    }

}
 ```

## 10.3 支付测试

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = ShopPayWebApplication.class)
@TestPropertySource("classpath:application.properties")
public class PayTest {

    @Autowired
    private RestTemplate restTemplate;

    @Value("${shop.pay.baseURI}")
    private String baseURI;

    @Value("${shop.pay.createPayment}")
    private String createPaymentPath;

    @Value("${shop.pay.callbackPayment}")
    private String callbackPaymentPath;

    @Autowired
    private IDWorker idWorker;

   /**
     * 创建支付订单
     */
    @Test
    public void createPayment(){

        Long orderId = 346321587315814400L;
        TradePay pay = new TradePay();
        pay.setOrderId(orderId);
        pay.setPayAmount(new BigDecimal(4800));

        Result result = restTemplate.postForEntity(baseURI + createPaymentPath, pay, Result.class).getBody();
        System.out.println(result);
    }
   
    /**
     * 支付回调
     */
    @Test
    public void callbackPayment(){
        Long payId = 346321891507720192L;
        TradePay pay = new TradePay();
        pay.setPayId(payId);
        pay.setIsPaid(ShopCode.SHOP_ORDER_PAY_STATUS_IS_PAY.getCode());
        Result result = restTemplate.postForEntity(baseURI + callbackPaymentPath, pay, Result.class).getBody();
        System.out.println(result);

    }

}
```

# 11.高级功能

## 11.1消息存储

分布式队列因为有高可靠性，所以数据要进行持久化存储

![](https://pic.imgdb.cn/item/5f10556f14195aa594b1f6fa.jpg)

1. 消息生产者发送消息

2. MQ收到消息，将消息进行持久化，在存储中新增一条记录

3. 返回ACK给生产者

4. MQ push消息给对应的消费者，然后等待消费者返回ACK
5. 如果消息消费者在指定时间内成功返回ack，那么MQ认为消息消费成功，在存储中删除消息，即执行第6步；如果MQ在指定时间内没有收到ACK，则认为消息消费失败，会尝试重新push消息,重复执行4、5、6步骤
6. MQ删除消息

### 11.1.1 存储介质

* 关系型数据库DB

Apache下开源的另外一款MQ—ActiveMQ（默认采用的KahaDB做消息存储）可选用JDBC的方式来做消息持久化，通过简单的xml配置信息即可实现JDBC消息存储。由于，普通关系型数据库（如Mysql）在单表数据量达到千万级别的情况下，其IO读写性能往往会出现瓶颈。在可靠性方面，该种方案非常依赖DB，如果一旦DB出现故障，则MQ的消息就无法落盘存储会导致线上故障。

![](https://pic.imgdb.cn/item/5f10575214195aa594b28e20.jpg)

- 文件系统

  目前业界较为常用的几款产品（RocketMQ/Kafka/RabbitMQ）均采用的是消息刷盘至所部署虚拟机/物理机的文件系统来做持久化（刷盘一般可以分为异步刷盘和同步刷盘两种模式）。消息刷盘为消息存储提供了一种高效率、高可靠性和高性能的数据持久化方式。除非部署MQ机器本身或是本地磁盘挂了，否则一般是不会出现无法持久化的故障问题。

![](https://pic.imgdb.cn/item/5f10578414195aa594b29cc1.jpg)

### 11.1.2 性能对比

文件系统>关系型数据库DB

### 11.1.3 消息的存储和发送

#### 1）消息存储

磁盘如果使用得当，磁盘的速度完全可以匹配上网络 的数据传输速度。目前的高性能磁盘，顺序写速度可以达到600MB/s， 超过了一般网卡的传输速度。但是磁盘随机写的速度只有大概100KB/s，和顺序写的性能相差6000倍！因为有如此巨大的速度差别，好的消息队列系统会比普通的消息队列系统速度快多个数量级。RocketMQ的消息用顺序写,保证了消息存储的速度。

#### 2）消息发送

Linux操作系统分为【用户态】和【内核态】，文件操作、网络操作需要涉及这两种形态的切换，免不了进行数据复制。

一台服务器 把本机磁盘文件的内容发送到客户端，一般分为两个步骤：

1）read；读取本地文件内容； 

2）write；将读取的内容通过网络发送出去。

这两个看似简单的操作，实际进行了4 次数据复制，分别是：

1. 从磁盘复制数据到内核态内存；
2. 从内核态内存复 制到用户态内存；(省去的是这一步)
3. 然后从用户态 内存复制到网络驱动的内核态内存；
4. 最后是从网络驱动的内核态内存复 制到网卡中进行传输。

![](https://pic.imgdb.cn/item/5f1058a114195aa594b2eeb5.jpg)

RocketMQ充分利用了上述特性，也就是所谓的“零拷贝”技术，提高消息存盘和网络发送的速度。

> 这里需要注意的是，采用MappedByteBuffer这种内存映射的方式有几个限制，其中之一是一次只能映射1.5~2G 的文件至用户态的虚拟内存，这也是为何RocketMQ默认设置单个CommitLog日志数据文件为1G的原因了

### 9.1.4 消息存储结构

RocketMQ消息的存储是由ConsumeQueue和CommitLog配合完成 的，消息真正的物理存储文件是CommitLog，ConsumeQueue是消息的逻辑队列，类似数据库的索引文件，存储的是指向物理存储的地址。每 个Topic下的每个Message Queue都有一个对应的ConsumeQueue文件。

![](https://pic.imgdb.cn/item/5f10596f14195aa594b32a01.jpg)

* CommitLog：存储消息的元数据
* ConsumerQueue：存储消息在CommitLog的索引
* IndexFile：为了消息查询提供了一种通过key或时间区间来查询消息的方法，这种通过IndexFile来查找消息的方法不影响发送与消费消息的主流程

### 9.1.5 刷盘机制

RocketMQ的消息是存储到磁盘上的，这样既能保证断电后恢复， 又可以让存储的消息量超出内存的限制。RocketMQ为了提高性能，会尽可能地保证磁盘的顺序写。消息在通过Producer写入RocketMQ的时 候，有两种写磁盘方式，分布式同步刷盘和异步刷盘。

![](https://pic.imgdb.cn/item/5f145ef914195aa594e45c89.jpg)

#### 1）同步刷盘

在返回写成功状态时，消息已经被写入磁盘。具体流程是，消息写入内存的PAGECACHE后，立刻通知刷盘线程刷盘， 然后等待刷盘完成，刷盘线程执行完成后唤醒等待的线程，返回消息写 成功的状态。

#### 2）异步刷盘

在返回写成功状态时，消息可能只是被写入了内存的PAGECACHE，写操作的返回快，吞吐量大；当内存里的消息量积累到一定程度时，统一触发写磁盘动作，快速写入。

#### 3）配置

**同步刷盘还是异步刷盘，都是通过Broker配置文件里的flushDiskType 参数设置的，这个参数被配置成SYNC_FLUSH、ASYNC_FLUSH中的 一个。**

## 11.2 高可用性机制

![](https://pic.imgdb.cn/item/5f105b5a14195aa594b3bca4.jpg)

RocketMQ分布式集群是通过Master和Slave的配合达到高可用性的。

Master和Slave的区别：在Broker的配置文件中，参数 brokerId的值为0表明这个Broker是Master，大于0表明这个Broker是 Slave，同时brokerRole参数也会说明这个Broker是Master还是Slave。

Master角色的Broker支持读和写，Slave角色的Broker仅支持读，也就是 Producer只能和Master角色的Broker连接写入消息；Consumer可以连接 Master角色的Broker，也可以连接Slave角色的Broker来读取消息。

### 11.2.1 消息消费高可用

在Consumer的配置文件中，并不需要设置是从Master读还是从Slave 读，当Master不可用或者繁忙的时候，Consumer会被自动切换到从Slave 读。有了自动切换Consumer这种机制，当一个Master角色的机器出现故障后，Consumer仍然可以从Slave读取消息，不影响Consumer程序。这就达到了消费端的高可用性。

### 11.2.2 消息发送高可用

在创建Topic的时候，把Topic的多个Message Queue创建在多个Broker组上（相同Broker名称，不同 brokerId的机器组成一个Broker组），这样当一个Broker组的Master不可 用后，其他组的Master仍然可用，Producer仍然可以发送消息。 RocketMQ目前还不支持把Slave自动转成Master，如果机器资源不足， 需要把Slave转成Master，则要手动停止Slave角色的Broker，更改配置文 件，用新的配置文件启动Broker。



### 11.2.3 消息主从复制

如果一个Broker组有Master和Slave，消息需要从Master复制到Slave 上，有同步和异步两种复制方式。

#### 1）同步复制

同步复制方式是等Master和Slave均写 成功后才反馈给客户端写成功状态；

在同步复制方式下，如果Master出故障， Slave上有全部的备份数据，容易恢复，但是同步复制会增大数据写入 延迟，降低系统吞吐量。

#### 2）异步复制 

异步复制方式是只要Master写成功 即可反馈给客户端写成功状态。

在异步复制方式下，系统拥有较低的延迟和较高的吞吐量，但是如果Master出了故障，有些数据因为没有被写 入Slave，有可能会丢失；

#### 3）配置

同步复制和异步复制是通过Broker配置文件里的brokerRole参数进行设置的，这个参数可以被设置成ASYNC_MASTER、 SYNC_MASTER、SLAVE三个值中的一个。

#### 4）总结

![](https://pic.imgdb.cn/item/5f105de114195aa594b477c7.jpg)



实际应用中要结合业务场景，合理设置刷盘方式和主从复制方式， 尤其是SYNC_FLUSH方式，由于频繁地触发磁盘写动作，会明显降低 性能。通常情况下，应该把Master和Save配置成ASYNC_FLUSH的刷盘 方式，主从之间配置成SYNC_MASTER的复制方式，这样即使有一台 机器出故障，仍然能保证数据不丢，是个不错的选择。

## 11.3 负载均衡

### 11.3.1 Producer负载均衡

Producer端，每个实例在发消息的时候，默认会轮询所有的message queue发送，以达到让消息平均落在不同的queue上。而由于queue可以散落在不同的broker，所以消息就发送到不同的broker下，如下图：

![](https://pic.imgdb.cn/item/5f105e2214195aa594b48cca.jpg)

图中箭头线条上的标号代表顺序，发布方会把第一条消息发送至 Queue 0，然后第二条消息发送至 Queue 1，以此类推。

### 11.3.2 Consumer负载均衡

#### 1）集群模式

在集群消费模式下，每条消息只需要投递到订阅这个topic的Consumer Group下的一个实例即可。RocketMQ采用主动拉取的方式拉取并消费消息，在拉取的时候需要明确指定拉取哪一条message queue。

而每当实例的数量有变更，都会触发一次所有实例的负载均衡，这时候会按照queue的数量和实例的数量平均分配queue给每个实例。

默认的分配算法是AllocateMessageQueueAveragely，如下图：

![](https://pic.imgdb.cn/item/5f105e7d14195aa594b4aa8b.jpg)

还有另外一种平均的算法是AllocateMessageQueueAveragelyByCircle，也是平均分摊每一条queue，只是以环状轮流分queue的形式，如下图：

![](https://pic.imgdb.cn/item/5f105ec514195aa594b4bfa4.jpg)

需要注意的是，集群模式下，queue都是只允许分配只一个实例，这是由于如果多个实例同时消费一个queue的消息，由于拉取哪些消息是consumer主动控制的，那样会导致同一个消息在不同的实例下被消费多次，所以算法上都是一个queue只分给一个consumer实例，一个consumer实例可以允许同时分到不同的queue。

通过增加consumer实例去分摊queue的消费，可以起到水平扩展的消费能力的作用。而有实例下线的时候，会重新触发负载均衡，这时候原来分配到的queue将分配到其他实例上继续消费。

但是如果consumer实例的数量比message queue的总数量还多的话，多出来的consumer实例将无法分到queue，也就无法消费到消息，也就无法起到分摊负载的作用了。所以需要控制让queue的总数量大于等于consumer的数量。

#### 2）广播模式

由于广播模式下要求一条消息需要投递到一个消费组下面所有的消费者实例，所以也就没有消息被分摊消费的说法。

在实现上，其中一个不同就是在consumer分配queue的时候，所有consumer都分到所有的queue。

![](https://pic.imgdb.cn/item/5f145f0f14195aa594e461fe.jpg)



## 11.4 消息重试

### 11.4.1 顺序消息的重试

对于顺序消息，当消费者消费消息失败后，消息队列 RocketMQ 会自动不断进行消息重试（每次间隔时间为 1 秒），这时，应用会出现消息消费被阻塞的情况。因此，在使用顺序消息时，务必保证应用能够及时监控并处理消费失败的情况，避免阻塞现象的发生。

### 11.4.2 无序消息的重试

对于无序消息（普通、定时、延时、事务消息），当消费者消费消息失败时，您可以通过设置返回状态达到消息重试的结果。

无序消息的重试只针对集群消费方式生效；广播方式不提供失败重试特性，即消费失败后，失败消息不再重试，继续消费新的消息。

#### 1）重试次数

消息队列 RocketMQ 默认允许每条消息最多重试 16 次，每次重试的间隔时间如下：

| 第几次重试 | 与上次重试的间隔时间 | 第几次重试 | 与上次重试的间隔时间 |
| :--------: | :------------------: | :--------: | :------------------: |
|     1      |        10 秒         |     9      |        7 分钟        |
|     2      |        30 秒         |     10     |        8 分钟        |
|     3      |        1 分钟        |     11     |        9 分钟        |
|     4      |        2 分钟        |     12     |       10 分钟        |
|     5      |        3 分钟        |     13     |       20 分钟        |
|     6      |        4 分钟        |     14     |       30 分钟        |
|     7      |        5 分钟        |     15     |        1 小时        |
|     8      |        6 分钟        |     16     |        2 小时        |

如果消息重试 16 次后仍然失败，消息将不再投递。如果严格按照上述重试时间间隔计算，某条消息在一直消费失败的前提下，将会在接下来的 4 小时 46 分钟之内进行 16 次重试，超过这个时间范围消息将不再重试投递。

**注意：** 一条消息无论重试多少次，这些重试消息的 Message ID 不会改变。

#### 2）配置方式

**消费失败后，重试配置方式**

集群消费方式下，消息消费失败后期望消息重试，需要在消息监听器接口的实现中明确进行配置（三种方式任选一种）：

- 返回 Action.ReconsumeLater （推荐）
- 返回 Null
- 抛出异常

```java
public class MessageListenerImpl implements MessageListener {
    @Override
    public Action consume(Message message, ConsumeContext context) {
        //处理消息
        doConsumeMessage(message);
        //方式1：返回 Action.ReconsumeLater，消息将重试
        return Action.ReconsumeLater;
        //方式2：返回 null，消息将重试
        return null;
        //方式3：直接抛出异常， 消息将重试
        throw new RuntimeException("Consumer Message exceotion");
    }
}
```

**消费失败后，不重试配置方式**

集群消费方式下，消息失败后期望消息不重试，需要捕获消费逻辑中可能抛出的异常，最终返回 Action.CommitMessage，此后这条消息将不会再重试。

```java
public class MessageListenerImpl implements MessageListener {
    @Override
    public Action consume(Message message, ConsumeContext context) {
        try {
            doConsumeMessage(message);
        } catch (Throwable e) {
            //捕获消费逻辑中的所有异常，并返回 Action.CommitMessage;
            return Action.CommitMessage;
        }
        //消息处理正常，直接返回 Action.CommitMessage;
        return Action.CommitMessage;
    }
}
```

**自定义消息最大重试次数**

消息队列 RocketMQ 允许 Consumer 启动的时候设置最大重试次数，重试时间间隔将按照如下策略：

- 最大重试次数小于等于 16 次，则重试时间间隔同上表描述。
- 最大重试次数大于 16 次，超过 16 次的重试时间间隔均为每次 2 小时。

```java
Properties properties = new Properties();
//配置对应 Group ID 的最大消息重试次数为 20 次
properties.put(PropertyKeyConst.MaxReconsumeTimes,"20");
Consumer consumer =ONSFactory.createConsumer(properties);
```

> 注意：
>
> - 消息最大重试次数的设置对相同 Group ID 下的所有 Consumer 实例有效。
> - 如果只对相同 Group ID 下两个 Consumer 实例中的其中一个设置了 MaxReconsumeTimes，那么该配置对两个 Consumer 实例均生效。
> - 配置采用覆盖的方式生效，即最后启动的 Consumer 实例会覆盖之前的启动实例的配置

**获取消息重试次数**

消费者收到消息后，可按照如下方式获取消息的重试次数：

```java
public class MessageListenerImpl implements MessageListener {
    @Override
    public Action consume(Message message, ConsumeContext context) {
        //获取消息的重试次数
        System.out.println(message.getReconsumeTimes());
        return Action.CommitMessage;
    }
}
```

## 11.5 死信队列

当一条消息初次消费失败，消息队列 RocketMQ 会自动进行消息重试；达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时，消息队列 RocketMQ 不会立刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中。

在消息队列 RocketMQ 中，这种正常情况下无法被消费的消息称为死信消息（Dead-Letter Message），存储死信消息的特殊队列称为死信队列（Dead-Letter Queue）。

### 11.5.1 死信特性

死信消息具有以下特性

- 不会再被消费者正常消费。
- 有效期与正常消息相同，均为 3 天，3 天后会被自动删除。因此，请在死信消息产生后的 3 天内及时处理。

死信队列具有以下特性：

- 一个死信队列对应一个 Group ID， 而不是对应单个消费者实例。
- 如果一个 Group ID 未产生死信消息，消息队列 RocketMQ 不会为其创建相应的死信队列。
- 一个死信队列包含了对应 Group ID 产生的所有死信消息，不论该消息属于哪个 Topic。

### 11.5.2 查看死信信息

1. 在控制台查询出现死信队列的主题信息

![](https://pic.imgdb.cn/item/5f10608f14195aa594b54889.jpg)

2. 在消息界面根据主题查询死信消息

![](https://pic.imgdb.cn/item/5f145f2c14195aa594e4694c.jpg)

3. 选择重新发送消息

一条消息进入死信队列，意味着某些因素导致消费者无法正常消费该消息，因此，通常需要您对其进行特殊处理。排查可疑因素并解决问题后，可以在消息队列 RocketMQ 控制台重新发送该消息，让消费者重新消费一次。

## 11.6 消费幂等

消息队列 RocketMQ 消费者在接收到消息以后，有必要根据业务上的唯一 Key 对消息做幂等处理的必要性。

### 11.6.1 消费幂等的必要性

在互联网应用中，尤其在网络不稳定的情况下，消息队列 RocketMQ 的消息有可能会出现重复，这个重复简单可以概括为以下情况：

- 发送时消息重复

  当一条消息已被成功发送到服务端并完成持久化，此时出现了网络闪断或者客户端宕机，导致服务端对客户端应答失败。 如果此时生产者意识到消息发送失败并尝试再次发送消息，消费者后续会收到两条内容相同并且 Message ID 也相同的消息。

- 投递时消息重复

  消息消费的场景下，消息已投递到消费者并完成业务处理，当客户端给服务端反馈应答的时候网络闪断。 为了保证消息至少被消费一次，消息队列 RocketMQ 的服务端将在网络恢复后再次尝试投递之前已被处理过的消息，消费者后续会收到两条内容相同并且 Message ID 也相同的消息。

- 负载均衡时消息重复（包括但不限于网络抖动、Broker 重启以及订阅方应用重启）

  当消息队列 RocketMQ 的 Broker 或客户端重启、扩容或缩容时，会触发 Rebalance，此时消费者可能会收到重复消息。

### 11.6.2 处理方式

因为 Message ID 有可能出现冲突（重复）的情况，所以真正安全的幂等处理，不建议以 Message ID 作为处理依据。 最好的方式是以业务唯一标识作为幂等处理的关键依据，而业务的唯一标识可以通过消息 Key 进行设置：

```java
Message message = new Message();
message.setKey("ORDERID_100");
SendResult sendResult = producer.send(message);
```

订阅方收到消息时可以根据消息的 Key 进行幂等处理：

```java
consumer.subscribe("ons_test", "*", new MessageListener() {
    public Action consume(Message message, ConsumeContext context) {
        String key = message.getKey()
        // 根据业务唯一标识的 key 做幂等处理
    }
});
```