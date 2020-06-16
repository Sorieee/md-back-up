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
namesrvAddr=rocketmq-nameserver:9876;rocketmq-nameserver2:9876
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
namesrvAddr=rocketmq-nameserver:9876;rocketmq-nameserver2:9876
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
namesrvAddr=rocketmq-nameserver:9876;rocketmq-nameserver2:9876
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
namesrvAddr=rocketmq-nameserver:9876;rocketmq-nameserver2:9876
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
tail -500f ~/logs/rocketmqlogs/namesrv.log
# broker日志
tail -500f ~/logs/rocketmqlogs/broker.log
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












