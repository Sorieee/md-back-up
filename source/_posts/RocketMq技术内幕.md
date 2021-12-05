# 1. 阅读源代码准备

## 1.1 获取和调试

**1.启动NameServer**

​	Step1：展开namesrv模块，右键NamesrvStartup.java，移动到Debug As，选中Debug Configurations，弹出Debug Configurations对话框，如图1-14所示。

​	Step2：选中Java Application条目并单击右键，选择New弹出Debug Configurations对话框，如图1-15所示。

​	Step3：设置RocketMQ运行主目录。选择Environment选项卡，添加环境变量ROCKET_HOME。

​	Step4：在RocketMQ运行主目录中创建conf、logs、store三个文件夹，如图1-16所示。

​	Step5：从RocketMQ distribution部署目录中将broker.conf、logback_broker.xml文件复制到conf目录中，logback_namesrv.xml文件则只需修改日志文件的目录，broker.conf文件内容如下所示。

```conf
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
#nameServer地址，分号分割
namesrvAddr=127.0.0.1:9844
deleteWhen=04
fileReservedTime=48
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
#存储路径
storePathRootDir=E:\\rocketmqTest\\store
#commitLog 存储路径
storePathCommitLog=E:\\rocketmqTest\\store\\commitlog
#消费队列存储路径
storePathConsumeQueue=E:\\rocketmqTest\\store\\consumequeue
#消息索引存储路径
storePathIndex=E:\\rocketmqTest\\store\\index
#checkpoint 文件存储路径
storeCheckpoint=E:\\rocketmqTest\\store\\checkpoint
#abort 文件存储路径
abortFile=E:\\rocketmqTest\\store\\a
```

**2. 启动Broker**

​	Step1：展开broker模块，右键BrokerStartup.java，移动到Debug As，选中Debug Configurations，弹出如图1-17所示的对话框，选择arguments选项卡，配置-c属性指定broker配置文件路径。

​	Step2：切换选项卡Environment，配置RocketMQ主目录，如图1-18所示。

​	Step3：以Debug模式运行BrokerStartup.java，查看${ROCKET_HOME}/logs/broker.log文件，未报错则表示启动成功。

## 1.2 RocketMQ源代码的目录结构

RocketMQ核心目录说明如下。
1）broker：broker模块（broker启动进程）。
2）client：消息客户端，包含消息生产者、消息消费者相关类。
3）common：公共包。
4）dev：开发者信息（非源代码）。
5）distribution：部署实例文件夹（非源代码）。
6）example：RocketMQ示例代码。
7）filter：消息过滤相关基础类。
8）filtersrv：消息过滤服务器实现相关类（Filter启动进程）。
9）logappender：日志实现相关类。
10）namesrv：NameServer实现相关类（NameServer启动进程）。
11）openmessaging：消息开放标准，正在制定中。
12）remoting：远程通信模块，基于Netty。
13）srvutil：服务器工具类。
14）store：消息存储实现相关类。
15）style：checkstyle相关实现。
16）test：测试相关类。
17）tools：工具类，监控命令相关实现类。

## 1.3 RocketMQ的设计理念和目标

### 1.3.1 设计理念

​	RocketMQ设计基于主题的发布与订阅模式，其核心功能包括消息发送、消息存储（Broker）、消息消费，整体设计追求简单与性能第一，主要体现在如下三个方面。

​	首先，NameServer设计极其简单，摒弃了业界常用的使用Zookeeper充当信息管理的“注册中心”，而是自研NameServer来实现元数据的管理（Topic路由信息等）。从实际需求出发，因为Topic路由信息无须在集群之间保持强一致，追求最终一致性，并且能容忍分钟级的不一致。正是基于此种情况，RocketMQ的NameServer集群之间互不通信，极大地降低了NameServer实现的复杂程度，对网络的要求也降低了不少，但是性能相比较Zookeeper有了极大的提升。

​	其次是高效的IO存储机制。RocketMQ追求消息发送的高吞吐量，RocketMQ的消息存储文件设计成文件组的概念，组内单个文件大小固定，方便引入内存映射机制，所有主题的消息存储基于顺序写，极大地提供了消息写性能，同时为了兼顾消息消费与消息查找，引入了消息消费队列文件与索引文件。

​	最后是容忍存在设计缺陷，适当将某些工作下放给RocketMQ使用者。消息中间件的实现者经常会遇到一个难题：如何保证消息一定能被消息消费者消费，并且保证只消费一次。RocketMQ的设计者给出的解决办法是不解决这个难题，而是退而求其次，只保证消息被消费者消费，但设计上允许消息被重复消费，这样极大地简化了消息中间件的内核，使得实现消息发送高可用变得非常简单与高效，消息重复问题由消费者在消息消费时实现幂等。

