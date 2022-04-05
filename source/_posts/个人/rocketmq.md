# 2.Nameserver

![](https://pic.imgdb.cn/item/6242c9c227f86abb2a0428ab.jpg)

# 3.消息发送

​	同步：发送者向MQ执行发送消息API时，同步等待，直到消息服务器返回发送结果。
​	异步：发送者向MQ执行发送消息API时，指定消息发送成功后的回掉函数，然后调用消息发送API后，立即返回，消息发送者线程不阻塞，直到运行结束，消息发送成功或失败的回调任务在一个新的线程中执行。
​	单向：消息发送者向MQ执行发送消息API时，直接返回，不等待消息服务器的结果，也不注册回调函数，简单地说，就是只管发，不在乎消息是否成功存储在消息服务器上。



​	消息发送流程主要的步骤：验证消息、查找路由、消息发送（包含异常处理机制）。

# 4. 存储

​	RocketMQ主要存储的文件包括Comitlog文件、ConsumeQueue文件、IndexFile文件。

![](https://pic.imgdb.cn/item/62465de427f86abb2ad004a8.jpg)

1）CommitLog：消息存储文件，所有消息主题的消息都存储在CommitLog文件中。

2）ConsumeQueue：消息消费队列，消息到达CommitLog文件后，将异步转发到消息消费队列，供消息消费者消费。

3）IndexFile：消息索引文件，主要存储消息Key与Offset的对应关系。

4）事务状态服务：存储每条消息的事务状态。

5）定时消息服务：每一个延迟级别对应一个消息消费队列，存储延迟队列的消息拉取进度。



​	消息存储实现类：org.apache.rocketmq.store.DefaultMessageStore，它是存储模块里面最重要的一个类，包含了很多对存储文件操作的API，其他模块对消息实体的操作都是通过DefaultMessageStore进行操作。

![](https://pic.imgdb.cn/item/62465f4e27f86abb2ad29304.jpg)

1）``MessageStoreConfig messageStoreConfig`：消息存储配置属性。

2）``CommitLog commitLog`:CommitLog文件的存储实现类。

3）``ConcurrentMap<String/* topic */, ConcurrentMap<Integer/* queueId */,Consume-Queue>> consumeQueueTable`：消息队列存储缓存表，按消息主题分组。

4）``FlushConsumeQueueService flushConsumeQueueService`：消息队列文件ConsumeQueue刷盘线程。

5）``CleanCommitLogService cleanCommitLogService`：清除CommitLog文件服务。

6）`CleanConsumeQueueService cleanConsumeQueueService`：清除ConsumeQueue文件服务。

7）`IndexService indexService`：索引文件实现类。

8）`AllocateMappedFileService allocateMappedFileService`：MappedFile分配服务。

9）`ReputMessageService reputMessageService`:CommitLog消息分发，根据CommitLog文件构建ConsumeQueue、IndexFile文件。

10）`HAService haService`：存储HA机制。

11）`TransientStorePool transientStorePool`：消息堆内存缓存。

12）`MessageArrivingListener messageArrivingListener`：消息拉取长轮询模式消息达到监听器。13）`BrokerConfig brokerConfig`:Broker配置属性。

14）`StoreCheckpoint storeCheckpoint`：文件刷盘检测点。

## 4.3 消息发送存储流程

​	消息存储入口：`org.apache.rocketmq.store.DefaultMessageStore#putMessage`。

​	Commitlog文件存储目录为`${ROCKET_HOME}/store/commitlog`目录，每一个文件默认1G，一个文件写满后再创建另外一个，以该文件中第一个偏移量为文件名，偏移量小于20位用0补齐。图4-3所示的第一个文件初始偏移量为0，第二个文件的1073741824，代表该文件中的第一条消息的物理偏移量为1073741824，这样根据物理偏移量能快速定位到消息。MappedFileQueue可以看作是`${ROCKET_HOME}/store/commitlog`文件夹，而MappedFile则对应该文件夹下一个个的文件。

![](https://pic.imgdb.cn/item/6246648a27f86abb2adc6f6b.jpg)

​	

​	DefaultAppendMessageCallback#doAppend只是将消息追加在内存中，需要根据是同步刷盘还是异步刷盘方式，将内存中的数据持久化到磁盘，关于刷盘操作后面会详细介绍。然后执行HA主从同步复制，主从同步将在第7章详细介绍。

## 4.4 存储文件组织与内存映射

​	RocketMQ通过使用内存映射文件来提高IO访问性能，无论是CommitLog、ConsumeQueue还是IndexFile，单个文件都被设计为固定长度，如果一个文件写满以后再创建一个新文件，文件名就为该文件第一条消息对应的全局物理偏移量。

![](https://pic.imgdb.cn/item/6246725d27f86abb2af3544d.jpg)

下面让我们一一来介绍MappedFileQueue的核心属性。

1）`String storePath`：存储目录。

2）`int mappedFileSize`：单个文件的存储大小。

3）`CopyOnWriteArrayList<MappedFile> mappedFiles`:MappedFile文件集合。

4）`AllocateMappedFileService allocateMappedFileService`：创建MappedFile服务类。

5）`long flushedWhere = 0`：当前刷盘指针，表示该指针之前的所有数据全部持久化到磁盘。6）`long committedWhere = 0`：当前数据提交指针，内存中ByteBuffer当前的写指针，该值大于等于flushedWhere。

## 4.5 RocketMQ存储文件

下面让我们一一介绍一下RocketMQ主要的存储文件夹。

1）commitlog：消息存储目录。

2）config：运行期间一些配置信息，主要包括下列信息。consumerFilter.json：主题消息过滤信息。consumerOffset.json：集群消费模式消息消费进度。delayOffset.json：延时消息队列拉取进度。subscriptionGroup.json：消息消费组配置信息。topics.json:topic配置属性。

3）consumequeue：消息消费队列存储目录。

4）index：消息索引文件存储目录。

5）abort：如果存在abort文件说明Broker非正常关闭，该文件默认启动时创建，正常退出之前删除。

6）checkpoint：文件检测点，存储commitlog文件最后一次刷盘时间戳、consumequeue最后一次刷盘时间、index索引文件最后一次刷盘时间戳。

### 4.5.1 Commitlog文件

![](https://pic.imgdb.cn/item/6246919927f86abb2a2d9304.jpg)

![](https://pic.imgdb.cn/item/6246a26e27f86abb2a4f6097.jpg)

![](https://pic.imgdb.cn/item/6246a2b727f86abb2a4febaf.jpg)

### 4.5.2 ConsumeQueue文件

​	RocketMQ基于主题订阅模式实现消息消费，消费者关心的是一个主题下的所有消息，但由于同一主题的消息不连续地存储在commitlog文件中，试想一下如果消息消费者直接从消息存储文件（commitlog）中去遍历查找订阅主题下的消息，效率将极其低下，RocketMQ为了适应消息消费的检索需求，设计了消息消费队列文件（Consumequeue），该文件可以看成是Commitlog关于消息消费的“索引”文件，consumequeue的第一级目录为消息主题，第二级目录为主题的消息队列，如图4-13所示。

![](https://pic.imgdb.cn/item/62469e4a27f86abb2a4695bd.jpg)

​	单个ConsumeQueue文件中默认包含30万个条目，单个文件的长度为30w×20字节，单个ConsumeQueue文件可以看出是一个ConsumeQueue条目的数组，其下标为Consume-Queue的逻辑偏移量，消息消费进度存储的偏移量即逻辑偏移量。ConsumeQueue即为Commitlog文件的索引文件，其构建机制是当消息到达Commitlog文件后，由专门的线程产生消息转发任务，从而构建消息消费队列文件与下文提到的索引文件。

### 4.5.3 Index索引文件

​	消息消费队列是RocketMQ专门为消息订阅构建的索引文件，提高根据主题与消息队列检索消息的速度，另外RocketMQ引入了Hash索引机制为消息建立索引，HashMap的设计包含两个基本点：Hash槽与Hash冲突的链表结构。RocketMQ索引文件布局如图4-15所示。

![](https://pic.imgdb.cn/item/62469f1b27f86abb2a4838b8.jpg)

### 4.5.4 checkpoint文件

![](https://pic.imgdb.cn/item/6246a0ba27f86abb2a4baf29.jpg)
