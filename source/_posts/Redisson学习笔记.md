https://github.com/redisson/redisson/wiki/1.-%E6%A6%82%E8%BF%B0

# 概述

R	edisson是一个在Redis的基础上实现的Java驻内存数据网格（In-Memory Data Grid）。它不仅提供了一系列的分布式的Java常用对象，还提供了许多分布式服务。其中包括(`BitSet`, `Set`, `Multimap`, `SortedSet`, `Map`, `List`, `Queue`, `BlockingQueue`, `Deque`, `BlockingDeque`, `Semaphore`, `Lock`, `AtomicLong`, `CountDownLatch`, `Publish / Subscribe`, `Bloom filter`, `Remote service`, `Spring cache`, `Executor service`, `Live Object service`, `Scheduler service`) Redisson提供了使用Redis的最简单和最便捷的方法。Redisson的宗旨是促进使用者对Redis的关注分离（Separation of Concern），从而让使用者能够将精力更集中地放在处理业务逻辑上。

**以下是Redisson的结构：**

* **[Redisson作为独立节点](https://github.com/redisson/redisson/wiki/12.-独立节点模式)** 可以用于独立执行其他节点发布到[分布式执行服务](https://github.com/redisson/redisson/wiki/9.-分布式服务/#93-分布式执行服务executor-service) 和 [分布式调度任务服务](https://github.com/redisson/redisson/wiki/9.-分布式服务/#94-分布式调度任务服务scheduler-service) 里的远程任务。

![](https://pic.imgdb.cn/item/60d44902844ef46bb2056652.jpg)

​	Redisson底层采用的是[Netty](http://netty.io/) 框架。支持[Redis](http://redis.cn/) 2.8以上版本，支持Java1.6+以上版本。

​	欢迎试用高性能[Redisson PRO](https://redisson.pro/)版。

# 配置方法

https://github.com/redisson/redisson/wiki/2.-%E9%85%8D%E7%BD%AE%E6%96%B9%E6%B3%95

## 程序化配置方法

​	Redisson程序化的配置方法是通过构建`Config`对象实例来实现的。例如

```
Config config = new Config();
config.setTransportMode(TransportMode.EPOLL);
config.useClusterServers()
      //可以用"rediss://"来启用SSL连接
      .addNodeAddress("redis://127.0.0.1:7181");
```

遇到问题

https://blog.csdn.net/weixin_34150830/article/details/89009880

windows上调整为

```java
	    private static final String redisUrl = "redis://192.168.170.200:6379";

	@Test
    void config1() {
        Config config = new Config();
        config.setTransportMode(TransportMode.NIO);

        config.useSingleServer()
                //可以用"rediss://"来启用SSL连接
                .setAddress(redisUrl);
        RedissonClient client = Redisson.create(config);
    }	
```

## 文件方式配置

​	Redisson既可以通过用户提供的YAML格式的文本文件来配置

####  通过YAML格式配置

​	Redisson的配置文件可以是或YAML格式。 也通过调用`config.fromYAML`方法并指定一个`File`实例来实现读取YAML格式的配置：

Redisson的配置文件可以是或YAML格式。 也通过调用`config.fromYAML`方法并指定一个`File`实例来实现读取YAML格式的配置：

```
Config config = Config.fromYAML(new File("config-file.yaml"));
RedissonClient redisson = Redisson.create(config);
```

调用`config.toYAML`方法可以将一个`Config`配置实例序列化为一个含有YAML数据类型的字符串：

```
Config config = new Config();
// ... 省略许多其他的设置
String jsonFormat = config.toYAML();
```

## 常用设置

​	以下是关于`org.redisson.Config`类的配置参数，它适用于所有Redis组态模式（单机，集群和哨兵）

##### codec（编码）

​	默认值: `org.redisson.codec.JsonJacksonCodec`

​	Redisson的对象编码类是用于将对象进行序列化和反序列化，以实现对该对象在Redis里的读取和存储。Redisson提供了以下几种的对象编码应用，以供大家选择：

| 编码类名称                                      | 说明                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `org.redisson.codec.JsonJacksonCodec`           | [Jackson JSON](https://github.com/FasterXML/jackson) 编码 **默认编码** |
| `org.redisson.codec.AvroJacksonCodec`           | [Avro](http://avro.apache.org/) 一个二进制的JSON编码         |
| `org.redisson.codec.SmileJacksonCodec`          | [Smile](http://wiki.fasterxml.com/SmileFormatSpec) 另一个二进制的JSON编码 |
| `org.redisson.codec.CborJacksonCodec`           | [CBOR](http://cbor.io/) 又一个二进制的JSON编码               |
| `org.redisson.codec.MsgPackJacksonCodec`        | [MsgPack](http://msgpack.org/) 再来一个二进制的JSON编码      |
| `org.redisson.codec.IonJacksonCodec`            | [Amazon Ion](https://amzn.github.io/ion-docs/) 亚马逊的Ion编码，格式与JSON类似 |
| `org.redisson.codec.KryoCodec`                  | [Kryo](https://github.com/EsotericSoftware/kryo) 二进制对象序列化编码 |
| `org.redisson.codec.SerializationCodec`         | JDK序列化编码                                                |
| `org.redisson.codec.FstCodec`                   | [FST](https://github.com/RuedigerMoeller/fast-serialization) 10倍于JDK序列化性能而且100%兼容的编码 |
| `org.redisson.codec.LZ4Codec`                   | [LZ4](https://github.com/jpountz/lz4-java) 压缩型序列化对象编码 |
| `org.redisson.codec.SnappyCodec`                | [Snappy](https://github.com/xerial/snappy-java) 另一个压缩型序列化对象编码 |
| `org.redisson.client.codec.JsonJacksonMapCodec` | 基于Jackson的映射类使用的编码。可用于避免序列化类的信息，以及用于解决使用`byte[]`遇到的问题。 |
| `org.redisson.client.codec.StringCodec`         | 纯字符串编码（无转换）                                       |
| `org.redisson.client.codec.LongCodec`           | 纯整长型数字编码（无转换）                                   |
| `org.redisson.client.codec.ByteArrayCodec`      | 字节数组编码                                                 |
| `org.redisson.codec.CompositeCodec`             | 用来组合多种不同编码在一起                                   |

##### threads（线程池数量）

​	默认值: `当前处理核数量 * 2`

这个线程池数量被所有`RTopic`对象监听器，`RRemoteService`调用者和`RExecutorService`任务共同共享。

##### nettyThreads （Netty线程池数量）

​	默认值: `当前处理核数量 * 2`

​	这个线程池数量是在一个Redisson实例内，被其创建的所有分布式数据类型和服务，以及底层客户端所一同共享的线程池里保存的线程数量。

##### executor（线程池）

​	单独提供一个用来执行所有`RTopic`对象监听器，`RRemoteService`调用者和`RExecutorService`任务的线程池（ExecutorService）实例。

​	用于特别指定一个EventLoopGroup. EventLoopGroup是用来处理所有通过Netty与Redis服务之间的连接发送和接受的消息。每一个Redisson都会在默认情况下自己创建管理一个EventLoopGroup实例。因此，如果在同一个JVM里面可能存在多个Redisson实例的情况下，采取这个配置实现多个Redisson实例共享一个EventLoopGroup的目的。

只有`io.netty.channel.epoll.EpollEventLoopGroup`或`io.netty.channel.nio.NioEventLoopGroup`才是允许的类型

##### transportMode（传输模式）

​	默认值：`TransportMode.NIO`

可选参数：
`TransportMode.NIO`,
`TransportMode.EPOLL` - 需要依赖里有`netty-transport-native-epoll`包（Linux） `TransportMode.KQUEUE` - 需要依赖里有 `netty-transport-native-kqueue`包（macOS）

##### lockWatchdogTimeout（监控锁的看门狗超时，单位：毫秒）

​	默认值：`30000`

监控锁的看门狗超时时间单位为毫秒。该参数只适用于分布式锁的加锁请求中未明确使用`leaseTimeout`参数的情况。如果该看门口未使用`lockWatchdogTimeout`去重新调整一个分布式锁的`lockWatchdogTimeout`超时，那么这个锁将变为失效状态。这个参数可以用来避免由Redisson客户端节点宕机或其他原因造成死锁的情况。

##### keepPubSubOrder（保持订阅发布顺序）

​	默认值：`true`

​	通过该参数来修改是否按订阅发布消息的接收顺序出来消息，如果选否将对消息实行并行处理，该参数只适用于订阅发布消息的情况。

##### performanceMode（高性能模式）

​	默认值：`HIGHER_THROUGHPUT`

用来指定高性能引擎的行为。由于该变量值的选用与使用场景息息相关（`NORMAL`除外）我们建议对每个参数值都进行尝试。

​	*该参数仅限于[Redisson PRO](https://redisson.pro/)版本。*

可选模式：
	`HIGHER_THROUGHPUT` - 将高性能引擎切换到 **高通量** 模式。 `LOWER_LATENCY_AUTO` - 将高性能引擎切换到 **低延时** 模式并自动探测最佳设定。 `LOWER_LATENCY_MODE_1` - 将高性能引擎切换到 **低延时** 模式并调整到预设模式1。 `LOWER_LATENCY_MODE_2` - 将高性能引擎切换到 **低延时** 模式并调整到预设模式2。 `NORMAL` - 将高性能引擎切换到 **普通** 模式

## 集群模式

集群模式除了适用于Redis集群环境，也适用于任何云计算服务商提供的集群模式，例如[AWS ElastiCache集群版](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Clusters.html)、[Azure Redis Cache](https://azure.microsoft.com/en-us/services/cache/)和[阿里云（Aliyun）的云数据库Redis版](https://cn.aliyun.com/product/kvstore)。

程序化配置集群的用法:

```
Config config = new Config();
config.useClusterServers()
    .setScanInterval(2000) // 集群状态扫描间隔时间，单位是毫秒
    //可以用"rediss://"来启用SSL连接
    .addNodeAddress("redis://127.0.0.1:7000", "redis://127.0.0.1:7001")
    .addNodeAddress("redis://127.0.0.1:7002");

RedissonClient redisson = Redisson.create(config);
```

### 集群设置

​	介绍配置Redis集群组态的文档在[这里](http://www.redis.cn/topics/cluster-tutorial.html)。 Redis集群组态的最低要求是必须有三个主节点。Redisson的集群模式的使用方法如下：

```
ClusterServersConfig clusterConfig = config.useClusterServers();
```

`ClusterServersConfig` 类的设置参数如下：

##### nodeAddresses（添加节点地址）

可以通过`host:port`的格式来添加Redis集群节点的地址。多个节点可以一次性批量添加。

##### scanInterval（集群扫描间隔时间）

默认值： `1000`

对Redis集群节点状态扫描的时间间隔。单位是毫秒。

##### slots（分片数量）

默认值： `231` 用于指定数据分片过程中的分片数量。支持数据分片/框架结构有：[集（Set）](https://github.com/redisson/redisson/wiki/7.-分布式集合#732--集set数据分片sharding)、[映射（Map）](https://github.com/redisson/redisson/wiki/7.-分布式集合#711-映射map的元素淘汰eviction本地缓存localcache和数据分片sharding)、[BitSet](https://github.com/redisson/redisson/wiki/6.-分布式对象#641-bitset数据分片sharding分布式roaringbitmap)、[Bloom filter](https://github.com/redisson/redisson/wiki/6.-分布式对象#681-布隆过滤器数据分片sharding), [Spring Cache](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#1421-spring-cache---本地缓存和数据分片)和[Hibernate Cache](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#1431-hibernate二级缓存---本地缓存和数据分片)等.

##### readMode（读取操作的负载均衡模式）

默认值： `SLAVE`（只在从服务节点里读取）

注：在从服务节点里读取的数据说明已经至少有两个节点保存了该数据，确保了数据的高可用性。

设置读取操作选择节点的模式。 可用值为： `SLAVE` - 只在从服务节点里读取。 `MASTER` - 只在主服务节点里读取。 `MASTER_SLAVE` - 在主从服务节点里都可以读取。

##### subscriptionMode（订阅操作的负载均衡模式）

默认值：`SLAVE`（只在从服务节点里订阅）

设置订阅操作选择节点的模式。 可用值为： `SLAVE` - 只在从服务节点里订阅。 `MASTER` - 只在主服务节点里订阅。

##### loadBalancer（负载均衡算法类的选择）

默认值： `org.redisson.connection.balancer.RoundRobinLoadBalancer`

在多Redis服务节点的环境里，可以选用以下几种负载均衡方式选择一个节点： `org.redisson.connection.balancer.WeightedRoundRobinBalancer` - 权重轮询调度算法 `org.redisson.connection.balancer.RoundRobinLoadBalancer` - 轮询调度算法 `org.redisson.connection.balancer.RandomLoadBalancer` - 随机调度算法

##### subscriptionConnectionMinimumIdleSize（从节点发布和订阅连接的最小空闲连接数）

默认值：`1`

多从节点的环境里，**每个** 从服务节点里用于发布和订阅连接的最小保持连接数（长连接）。Redisson内部经常通过发布和订阅来实现许多功能。长期保持一定数量的发布订阅连接是必须的。

##### subscriptionConnectionPoolSize（从节点发布和订阅连接池大小）

默认值：`50`

多从节点的环境里，**每个** 从服务节点里用于发布和订阅连接的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### slaveConnectionMinimumIdleSize（从节点最小空闲连接数）

默认值：`32`

多从节点的环境里，**每个** 从服务节点里用于普通操作（**非** 发布和订阅）的最小保持连接数（长连接）。长期保持一定数量的连接有利于提高瞬时读取反映速度。

##### slaveConnectionPoolSize（从节点连接池大小）

默认值：`64`

多从节点的环境里，**每个** 从服务节点里用于普通操作（**非** 发布和订阅）连接的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### masterConnectionMinimumIdleSize（主节点最小空闲连接数）

默认值：`32`

多节点的环境里，**每个** 主节点的最小保持连接数（长连接）。长期保持一定数量的连接有利于提高瞬时写入反应速度。

##### masterConnectionPoolSize（主节点连接池大小）

默认值：`64`

多主节点的环境里，**每个** 主节点的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### idleConnectionTimeout（连接空闲超时，单位：毫秒）

默认值：`10000`

如果当前连接池里的连接数量超过了最小空闲连接数，而同时有连接空闲时间超过了该数值，那么这些连接将会自动被关闭，并从连接池里去掉。时间单位是毫秒。

##### connectTimeout（连接超时，单位：毫秒）

默认值：`10000`

同任何节点建立连接时的等待超时。时间单位是毫秒。

##### timeout（命令等待超时，单位：毫秒）

默认值：`3000`

等待节点回复命令的时间。该时间从命令发送成功时开始计时。

##### retryAttempts（命令失败重试次数）

默认值：`3`

如果尝试达到 **retryAttempts（命令失败重试次数）** 仍然不能将命令发送至某个指定的节点时，将抛出错误。如果尝试在此限制之内发送成功，则开始启用 **timeout（命令等待超时）** 计时。

##### retryInterval（命令重试发送时间间隔，单位：毫秒）

默认值：`1500`

在某个节点执行相同或不同命令时，**连续** 失败 **failedAttempts（执行失败最大次数）** 时，该节点将被从可用节点列表里清除，直到 **reconnectionTimeout（重新连接时间间隔）** 超时以后再次尝试。

##### password（密码）

默认值：`null`

用于节点身份验证的密码。

##### subscriptionsPerConnection（单个连接最大订阅数量）

默认值：`5`

每个连接的最大订阅数量。

##### clientName（客户端名称）

默认值：`null`

在Redis节点里显示的客户端名称。

##### sslEnableEndpointIdentification（启用SSL终端识别）

默认值：`true`

开启SSL终端识别能力。

##### sslProvider（SSL实现方式）

默认值：`JDK`

确定采用哪种方式（JDK或OPENSSL）来实现SSL连接。

##### sslTruststore（SSL信任证书库路径）

默认值：`null`

指定SSL信任证书库的路径。

##### sslTruststorePassword（SSL信任证书库密码）

默认值：`null`

指定SSL信任证书库的密码。

##### sslKeystore（SSL钥匙库路径）

默认值：`null`

指定SSL钥匙库的路径。

##### sslKeystorePassword（SSL钥匙库密码）

默认值：`null`

指定SSL钥匙库的密码。

### 通过YAML文件配置集群模式

配置集群模式可以通过指定一个YAML格式的文件来实现。以下是YAML格式的配置文件样本。文件中的字段名称必须与`ClusterServersConfig`和`Config`对象里的字段名称相符。

```yml
---
clusterServersConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  slaveSubscriptionConnectionMinimumIdleSize: 1
  slaveSubscriptionConnectionPoolSize: 50
  slaveConnectionMinimumIdleSize: 32
  slaveConnectionPoolSize: 64
  masterConnectionMinimumIdleSize: 32
  masterConnectionPoolSize: 64
  readMode: "SLAVE"
  nodeAddresses:
  - "redis://127.0.0.1:7004"
  - "redis://127.0.0.1:7001"
  - "redis://127.0.0.1:7000"
  scanInterval: 1000
threads: 0
nettyThreads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
"transportMode":"NIO"
```

## 云托管模式

​	略

## 单Redis节点模式

程序化配置方法：

```
// 默认连接地址 127.0.0.1:6379
RedissonClient redisson = Redisson.create();

Config config = new Config();
config.useSingleServer().setAddress("myredisserver:6379");
RedissonClient redisson = Redisson.create(config);
```

### 单节点设置

Redis程序的配置和架设文档在[这里](http://www.redis.cn/topics/config.html)。Redisson的单Redis节点模式的使用方法如下： `SingleServerConfig singleConfig = config.useSingleServer();`

`SingleServerConfig` 类的设置参数如下：

##### address（节点地址）

可以通过`host:port`的格式来指定节点地址。

##### subscriptionConnectionMinimumIdleSize（发布和订阅连接的最小空闲连接数）

默认值：`1`

用于发布和订阅连接的最小保持连接数（长连接）。Redisson内部经常通过发布和订阅来实现许多功能。长期保持一定数量的发布订阅连接是必须的。

##### subscriptionConnectionPoolSize（发布和订阅连接池大小）

默认值：`50`

用于发布和订阅连接的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### connectionMinimumIdleSize（最小空闲连接数）

默认值：`32`

最小保持连接数（长连接）。长期保持一定数量的连接有利于提高瞬时写入反应速度。

##### connectionPoolSize（连接池大小）

默认值：`64`

在启用该功能以后，Redisson将会监测DNS的变化情况。

##### dnsMonitoringInterval（DNS监测时间间隔，单位：毫秒）

默认值：`5000`

监测DNS的变化情况的时间间隔。

##### idleConnectionTimeout（连接空闲超时，单位：毫秒）

默认值：`10000`

如果当前连接池里的连接数量超过了最小空闲连接数，而同时有连接空闲时间超过了该数值，那么这些连接将会自动被关闭，并从连接池里去掉。时间单位是毫秒。

##### connectTimeout（连接超时，单位：毫秒）

默认值：`10000`

同节点建立连接时的等待超时。时间单位是毫秒。

##### timeout（命令等待超时，单位：毫秒）

默认值：`3000`

等待节点回复命令的时间。该时间从命令发送成功时开始计时。

##### retryAttempts（命令失败重试次数）

默认值：`3`

如果尝试达到 **retryAttempts（命令失败重试次数）** 仍然不能将命令发送至某个指定的节点时，将抛出错误。如果尝试在此限制之内发送成功，则开始启用 **timeout（命令等待超时）** 计时。

##### retryInterval（命令重试发送时间间隔，单位：毫秒）

默认值：`1500`

在某个节点执行相同或不同命令时，**连续** 失败 **failedAttempts（执行失败最大次数）** 时，该节点将被从可用节点列表里清除，直到 **reconnectionTimeout（重新连接时间间隔）** 超时以后再次尝试。

##### database（数据库编号）

默认值：`0`

尝试连接的数据库编号。

##### password（密码）

默认值：`null`

用于节点身份验证的密码。

##### subscriptionsPerConnection（单个连接最大订阅数量）

默认值：`5`

每个连接的最大订阅数量。

##### clientName（客户端名称）

默认值：`null`

在Redis节点里显示的客户端名称。

##### sslEnableEndpointIdentification（启用SSL终端识别）

默认值：`true`

开启SSL终端识别能力。

##### sslProvider（SSL实现方式）

默认值：`JDK`

确定采用哪种方式（JDK或OPENSSL）来实现SSL连接。

##### sslTruststore（SSL信任证书库路径）

默认值：`null`

指定SSL信任证书库的路径。

##### sslTruststorePassword（SSL信任证书库密码）

默认值：`null`

指定SSL信任证书库的密码。

##### sslKeystore（SSL钥匙库路径）

默认值：`null`

指定SSL钥匙库的路径。

##### sslKeystorePassword（SSL钥匙库密码）

默认值：`null`

指定SSL钥匙库的密码。

### 通过YAML文件配置集群模式

配置单节点模式可以通过指定一个YAML格式的文件来实现。以下是YAML格式的配置文件样本。文件中的字段名称必须与`SingleServerConfig`和`Config`对象里的字段名称相符。

```
---
singleServerConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  address: "redis://127.0.0.1:6379"
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 32
  connectionPoolSize: 64
  database: 0
  dnsMonitoringInterval: 5000
threads: 0
nettyThreads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
"transportMode":"NIO"
```

## 哨兵模式

程序化配置哨兵模式的方法如下：

```
Config config = new Config();
config.useSentinelServers()
    .setMasterName("mymaster")
    //可以用"rediss://"来启用SSL连接
    .addSentinelAddress("127.0.0.1:26389", "127.0.0.1:26379")
    .addSentinelAddress("127.0.0.1:26319");

RedissonClient redisson = Redisson.create(config);
```

### 哨兵模式设置

配置Redis哨兵服务的官方文档在[这里](http://redis.cn/topics/sentinel.html)。Redisson的哨兵模式的使用方法如下： `SentinelServersConfig sentinelConfig = config.useSentinelServers();`

`SentinelServersConfig` 类的设置参数如下：

##### dnsMonitoringInterval（DNS监控间隔，单位：毫秒）

默认值：`5000`

用来指定检查节点DNS变化的时间间隔。使用的时候应该确保JVM里的DNS数据的缓存时间保持在足够低的范围才有意义。用`-1`来禁用该功能。

##### masterName（主服务器的名称）

主服务器的名称是哨兵进程中用来监测主从服务切换情况的。

##### addSentinelAddress（添加哨兵节点地址）

可以通过`host:port`的格式来指定哨兵节点的地址。多个节点可以一次性批量添加。

##### readMode（读取操作的负载均衡模式）

默认值： `SLAVE`（只在从服务节点里读取）

注：在从服务节点里读取的数据说明已经至少有两个节点保存了该数据，确保了数据的高可用性。

设置读取操作选择节点的模式。 可用值为： `SLAVE` - 只在从服务节点里读取。 `MASTER` - 只在主服务节点里读取。 `MASTER_SLAVE` - 在主从服务节点里都可以读取。

##### subscriptionMode（订阅操作的负载均衡模式）

默认值：`SLAVE`（只在从服务节点里订阅）

设置订阅操作选择节点的模式。 可用值为： `SLAVE` - 只在从服务节点里订阅。 `MASTER` - 只在主服务节点里订阅。

##### loadBalancer（负载均衡算法类的选择）

默认值： `org.redisson.connection.balancer.RoundRobinLoadBalancer`

在使用多个Redis服务节点的环境里，可以选用以下几种负载均衡方式选择一个节点： `org.redisson.connection.balancer.WeightedRoundRobinBalancer` - 权重轮询调度算法 `org.redisson.connection.balancer.RoundRobinLoadBalancer` - 轮询调度算法 `org.redisson.connection.balancer.RandomLoadBalancer` - 随机调度算法

##### subscriptionConnectionMinimumIdleSize（从节点发布和订阅连接的最小空闲连接数）

默认值：`1`

多从节点的环境里，**每个** 从服务节点里用于发布和订阅连接的最小保持连接数（长连接）。Redisson内部经常通过发布和订阅来实现许多功能。长期保持一定数量的发布订阅连接是必须的。

##### subscriptionConnectionPoolSize（从节点发布和订阅连接池大小）

默认值：`50`

多从节点的环境里，**每个** 从服务节点里用于发布和订阅连接的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### slaveConnectionMinimumIdleSize（从节点最小空闲连接数）

默认值：`32`

多从节点的环境里，**每个** 从服务节点里用于普通操作（**非** 发布和订阅）的最小保持连接数（长连接）。长期保持一定数量的连接有利于提高瞬时读取反映速度。

##### slaveConnectionPoolSize（从节点连接池大小）

默认值：`64`

多从节点的环境里，**每个** 从服务节点里用于普通操作（**非** 发布和订阅）连接的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### masterConnectionMinimumIdleSize（主节点最小空闲连接数）

默认值：`32`

多从节点的环境里，**每个** 主节点的最小保持连接数（长连接）。长期保持一定数量的连接有利于提高瞬时写入反应速度。

##### masterConnectionPoolSize（主节点连接池大小）

默认值：`64`

主节点的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### idleConnectionTimeout（连接空闲超时，单位：毫秒）

默认值：`10000`

如果当前连接池里的连接数量超过了最小空闲连接数，而同时有连接空闲时间超过了该数值，那么这些连接将会自动被关闭，并从连接池里去掉。时间单位是毫秒。

##### connectTimeout（连接超时，单位：毫秒）

默认值：`10000`

同任何节点建立连接时的等待超时。时间单位是毫秒。

##### timeout（命令等待超时，单位：毫秒）

默认值：`3000`

等待节点回复命令的时间。该时间从命令发送成功时开始计时。

##### retryAttempts（命令失败重试次数）

默认值：`3`

如果尝试达到 **retryAttempts（命令失败重试次数）** 仍然不能将命令发送至某个指定的节点时，将抛出错误。如果尝试在此限制之内发送成功，则开始启用 **timeout（命令等待超时）** 计时。

##### retryInterval（命令重试发送时间间隔，单位：毫秒）

默认值：`1500`

在某个节点执行相同或不同命令时，**连续** 失败 **failedAttempts（执行失败最大次数）** 时，该节点将被从可用节点列表里清除，直到 **reconnectionTimeout（重新连接时间间隔）** 超时以后再次尝试。

##### database（数据库编号）

默认值：`0`

尝试连接的数据库编号。

##### password（密码）

默认值：`null`

用于节点身份验证的密码。

##### subscriptionsPerConnection（单个连接最大订阅数量）

默认值：`5`

每个连接的最大订阅数量。

##### clientName（客户端名称）

默认值：`null`

在Redis节点里显示的客户端名称。

##### sslEnableEndpointIdentification（启用SSL终端识别）

默认值：`true`

开启SSL终端识别能力。

##### sslProvider（SSL实现方式）

默认值：`JDK`

确定采用哪种方式（JDK或OPENSSL）来实现SSL连接。

##### sslTruststore（SSL信任证书库路径）

默认值：`null`

指定SSL信任证书库的路径。

##### sslTruststorePassword（SSL信任证书库密码）

默认值：`null`

指定SSL信任证书库的密码。

##### sslKeystore（SSL钥匙库路径）

默认值：`null`

指定SSL钥匙库的路径。

##### sslKeystorePassword（SSL钥匙库密码）

默认值：`null`

指定SSL钥匙库的密码。

### 通过YAML文件配置集群模式

配置哨兵模式可以通过指定一个YAML格式的文件来实现。以下是YAML格式的配置文件样本。文件中的字段名称必须与`SentinelServersConfig`和`Config`对象里的字段名称相符。

```
---
sentinelServersConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  slaveSubscriptionConnectionMinimumIdleSize: 1
  slaveSubscriptionConnectionPoolSize: 50
  slaveConnectionMinimumIdleSize: 32
  slaveConnectionPoolSize: 64
  masterConnectionMinimumIdleSize: 32
  masterConnectionPoolSize: 64
  readMode: "SLAVE"
  sentinelAddresses:
  - "redis://127.0.0.1:26379"
  - "redis://127.0.0.1:26389"
  masterName: "mymaster"
  database: 0
threads: 0
nettyThreads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
"transportMode":"NIO"
```

## 主从模式

程序化配置主从模式的用法:

```
Config config = new Config();
config.useMasterSlaveServers()
    //可以用"rediss://"来启用SSL连接
    .setMasterAddress("redis://127.0.0.1:6379")
    .addSlaveAddress("redis://127.0.0.1:6389", "redis://127.0.0.1:6332", "redis://127.0.0.1:6419")
    .addSlaveAddress("redis://127.0.0.1:6399");

RedissonClient redisson = Redisson.create(config);
```

### 主从模式设置

介绍配置Redis主从服务组态的文档在[这里](http://redis.cn/topics/replication.html). Redisson的主从模式的使用方法如下： `MasterSlaveServersConfig masterSlaveConfig = config.useMasterSlaveServers();`

`MasterSlaveServersConfig` 类的设置参数如下：

##### dnsMonitoringInterval（DNS监控间隔，单位：毫秒）

默认值：`5000`

用来指定检查节点DNS变化的时间间隔。使用的时候应该确保JVM里的DNS数据的缓存时间保持在足够低的范围才有意义。用`-1`来禁用该功能。

##### masterAddress（主节点地址）

可以通过`host:port`的格式来指定主节点地址。

##### addSlaveAddress（添加从主节点地址）

可以通过`host:port`的格式来指定从节点的地址。多个节点可以一次性批量添加。

##### readMode（读取操作的负载均衡模式）

默认值： `SLAVE`（只在从服务节点里读取）

注：在从服务节点里读取的数据说明已经至少有两个节点保存了该数据，确保了数据的高可用性。

设置读取操作选择节点的模式。 可用值为： `SLAVE` - 只在从服务节点里读取。 `MASTER` - 只在主服务节点里读取。 `MASTER_SLAVE` - 在主从服务节点里都可以读取。

##### subscriptionMode（订阅操作的负载均衡模式）

默认值：`SLAVE`（只在从服务节点里订阅）

设置订阅操作选择节点的模式。 可用值为： `SLAVE` - 只在从服务节点里订阅。 `MASTER` - 只在主服务节点里订阅。

##### loadBalancer（负载均衡算法类的选择）

默认值： `org.redisson.connection.balancer.RoundRobinLoadBalancer`

在使用多个Redis服务节点的环境里，可以选用以下几种负载均衡方式选择一个节点： `org.redisson.connection.balancer.WeightedRoundRobinBalancer` - 权重轮询调度算法 `org.redisson.connection.balancer.RoundRobinLoadBalancer` - 轮询调度算法 `org.redisson.connection.balancer.RandomLoadBalancer` - 随机调度算法

##### subscriptionConnectionMinimumIdleSize（从节点发布和订阅连接的最小空闲连接数）

默认值：`1`

多从节点的环境里，**每个** 从服务节点里用于发布和订阅连接的最小保持连接数（长连接）。Redisson内部经常通过发布和订阅来实现许多功能。长期保持一定数量的发布订阅连接是必须的。

##### subscriptionConnectionPoolSize（从节点发布和订阅连接池大小）

默认值：`50`

多从节点的环境里，**每个** 从服务节点里用于发布和订阅连接的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### slaveConnectionMinimumIdleSize（从节点最小空闲连接数）

默认值：`32`

多从节点的环境里，**每个** 从服务节点里用于普通操作（**非** 发布和订阅）的最小保持连接数（长连接）。长期保持一定数量的连接有利于提高瞬时读取反映速度。

##### slaveConnectionPoolSize（从节点连接池大小）

默认值：`64`

多从节点的环境里，**每个** 从服务节点里用于普通操作（**非** 发布和订阅）连接的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### masterConnectionMinimumIdleSize（主节点最小空闲连接数）

默认值：`32`

多从节点的环境里，**每个** 主节点的最小保持连接数（长连接）。长期保持一定数量的连接有利于提高瞬时写入反应速度。

##### masterConnectionPoolSize（主节点连接池大小）

默认值：`64`

主节点的连接池最大容量。连接池的连接数量自动弹性伸缩。

##### idleConnectionTimeout（连接空闲超时，单位：毫秒）

默认值：`10000`

如果当前连接池里的连接数量超过了最小空闲连接数，而同时有连接空闲时间超过了该数值，那么这些连接将会自动被关闭，并从连接池里去掉。时间单位是毫秒。

##### connectTimeout（连接超时，单位：毫秒）

默认值：`10000`

同任何节点建立连接时的等待超时。时间单位是毫秒。

##### timeout（命令等待超时，单位：毫秒）

默认值：`3000`

等待节点回复命令的时间。该时间从命令发送成功时开始计时。

##### retryAttempts（命令失败重试次数）

默认值：`3`

如果尝试达到 **retryAttempts（命令失败重试次数）** 仍然不能将命令发送至某个指定的节点时，将抛出错误。如果尝试在此限制之内发送成功，则开始启用 **timeout（命令等待超时）** 计时。

##### retryInterval（命令重试发送时间间隔，单位：毫秒）

默认值：`1500`

在某个节点执行相同或不同命令时，**连续** 失败 **failedAttempts（执行失败最大次数）** 时，该节点将被从可用节点列表里清除，直到 **reconnectionTimeout（重新连接时间间隔）** 超时以后再次尝试。

##### database（数据库编号）

默认值：`0`

尝试连接的数据库编号。

##### password（密码）

默认值：`null`

用于节点身份验证的密码。

##### subscriptionsPerConnection（单个连接最大订阅数量）

默认值：`5`

每个连接的最大订阅数量。

##### clientName（客户端名称）

默认值：`null`

在Redis节点里显示的客户端名称。

##### sslEnableEndpointIdentification（启用SSL终端识别）

默认值：`true`

开启SSL终端识别能力。

##### sslProvider（SSL实现方式）

默认值：`JDK`

确定采用哪种方式（JDK或OPENSSL）来实现SSL连接。

##### sslTruststore（SSL信任证书库路径）

默认值：`null`

指定SSL信任证书库的路径。

##### sslTruststorePassword（SSL信任证书库密码）

默认值：`null`

指定SSL信任证书库的密码。

##### sslKeystore（SSL钥匙库路径）

默认值：`null`

指定SSL钥匙库的路径。

##### sslKeystorePassword（SSL钥匙库密码）

默认值：`null`

指定SSL钥匙库的密码。

### 通过YAML文件配置集群模式

配置主从模式可以通过指定一个YAML格式的文件来实现。以下是YAML格式的配置文件样本。文件中的字段名称必须与`MasterSlaveServersConfig`和`Config`对象里的字段名称相符。

```
---
masterSlaveServersConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  failedAttempts: 3
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  slaveSubscriptionConnectionMinimumIdleSize: 1
  slaveSubscriptionConnectionPoolSize: 50
  slaveConnectionMinimumIdleSize: 32
  slaveConnectionPoolSize: 64
  masterConnectionMinimumIdleSize: 32
  masterConnectionPoolSize: 64
  readMode: "SLAVE"
  slaveAddresses:
  - "redis://127.0.0.1:6381"
  - "redis://127.0.0.1:6380"
  masterAddress: "redis://127.0.0.1:6379"
  database: 0
threads: 0
nettyThreads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
"transportMode":"NIO"
```

# 程序接口调用方式

​	`RedissonClient`、`RedissonReactiveClient`和`RedissonRxClient`实例本身和Redisson提供的所有分布式对象都是线程安全的。

Redisson为每个操作都提供了自动重试策略，当某个命令执行失败时，Redisson会自动进行重试。自动重试策略可以通过修改`retryAttempts`（默认值：3）参数和`retryInterval`（默认值：1000毫秒）参数来进行优化调整。当等待时间达到`retryInterval`指定的时间间隔以后，将自动重试下一次。全部重试失败以后将抛出错误。

Redisson框架提供的几乎所有对象都包含了`同步`和`异步`相互匹配的方法。这些对象都可以通过`RedissonClient`接口获取。同时还为大部分Redisson对象提供了满足[`异步流处理标准`](http://reactive-streams.org/)的程序接口`RedissonReactiveClient`。除此外还提供了`RxJava2`规范的`RedissonRxClient`程序接口。

以下是关于使用`RAtomicLong`对象的范例：

```java
RedissonClient client = Redisson.create(config);
RAtomicLong longObject = client.getAtomicLong('myLong');
// 同步执行方式
longObject.compareAndSet(3, 401);
// 异步执行方式
RFuture<Boolean> result = longObject.compareAndSetAsync(3, 401);

RedissonReactiveClient client = Redisson.createReactive(config);
RAtomicLongReactive longObject = client.getAtomicLong('myLong');
// 异步流执行方式
Mono<Boolean> result = longObject.compareAndSet(3, 401);
RedissonRxClient client = Redisson.createRx(config);
RAtomicLongRx longObject= client.getAtomicLong("myLong");
// RxJava2方式
Flowable<Boolean result = longObject.compareAndSet(3, 401);
```

## 异步执行方式

几乎所有的Redisson对象都实现了一个异步接口，异步接口提供的方法名称与其同步接口的方法名称相互匹配。例如：

```
// RAtomicLong接口继承了RAtomicLongAsync接口
RAtomicLongAsync longObject = client.getAtomicLong("myLong");
RFuture<Boolean> future = longObject.compareAndSetAsync(1, 401);
```

异步执行的方法都会返回一个实现了`RFuture`接口的对象。该对象同时提供了`java.util.concurrent.CompletionStage`和`java.util.concurrent.Future`两个异步接口。

```java
future.whenComplete((res, exception) -> {
    // ...
});
// 或者
future.thenAccept(res -> {
    // 处理返回
}).exceptionally(exception -> {
    // 处理错误
});
```

##  异步流执行方式

Redisson为大多数分布式数据结构提供了满足[Reactor](http://projectreactor.io/)项目的[异步流处理标准](http://reactive-streams.org/)的程序接口。该接口通过两种方式实现：

1. 基于[Project Reactor](http://projectreactor.io/)标准的实现方式。使用范例如下：

```
RedissonReactiveClient client = Redisson.createReactive(config);
RAtomicLongReactive atomicLong = client.getAtomicLong("myLong");
Mono<Boolean> cs = longObject.compareAndSet(10, 91);
Mono<Long> get = longObject.get();

Publisher<Long> getPublisher = longObject.get();
```

1. 基于[RxJava2](https://github.com/ReactiveX/RxJava)标准的实现方式。使用范例如下：

```
RedissonRxClient client = Redisson.createRx(config);
RAtomicLongRx atomicLong = client.getAtomicLong("myLong");
Single<Boolean> cs = longObject.compareAndSet(10, 91);
Single<Long> get = longObject.get();
```

# 数据序列化

Redisson的对象编码类是用于将对象进行序列化和反序列化，以实现对该对象在Redis里的读取和存储。Redisson提供了以下几种的对象编码应用，以供大家选择：

| 编码类名称                                      | 说明                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `org.redisson.codec.JsonJacksonCodec`           | [Jackson JSON](https://github.com/FasterXML/jackson) 编码 **默认编码** |
| `org.redisson.codec.AvroJacksonCodec`           | [Avro](http://avro.apache.org/) 一个二进制的JSON编码         |
| `org.redisson.codec.SmileJacksonCodec`          | [Smile](http://wiki.fasterxml.com/SmileFormatSpec) 另一个二进制的JSON编码 |
| `org.redisson.codec.CborJacksonCodec`           | [CBOR](http://cbor.io/) 又一个二进制的JSON编码               |
| `org.redisson.codec.MsgPackJacksonCodec`        | [MsgPack](http://msgpack.org/) 再来一个二进制的JSON编码      |
| `org.redisson.codec.IonJacksonCodec`            | [Amazon Ion](https://amzn.github.io/ion-docs/) 亚马逊的Ion编码，格式与JSON类似 |
| `org.redisson.codec.KryoCodec`                  | [Kryo](https://github.com/EsotericSoftware/kryo) 二进制对象序列化编码 |
| `org.redisson.codec.SerializationCodec`         | JDK序列化编码                                                |
| `org.redisson.codec.FstCodec`                   | [FST](https://github.com/RuedigerMoeller/fast-serialization) 10倍于JDK序列化性能而且100%兼容的编码 |
| `org.redisson.codec.LZ4Codec`                   | [LZ4](https://github.com/jpountz/lz4-java) 压缩型序列化对象编码 |
| `org.redisson.codec.SnappyCodec`                | [Snappy](https://github.com/xerial/snappy-java) 另一个压缩型序列化对象编码 |
| `org.redisson.client.codec.JsonJacksonMapCodec` | 基于Jackson的映射类使用的编码。可用于避免序列化类的信息，以及用于解决使用`byte[]`遇到的问题。 |
| `org.redisson.client.codec.StringCodec`         | 纯字符串编码（无转换）                                       |
| `org.redisson.client.codec.LongCodec`           | 纯整长型数字编码（无转换）                                   |
| `org.redisson.client.codec.ByteArrayCodec`      | 字节数组编码                                                 |
| `org.redisson.codec.CompositeCodec`             | 用来组合多种不同编码在一起                                   |

# 单个集合数据分片（Sharding）

在集群模式下，Redisson为单个Redis集合类型提供了自动分片的功能。

Redisson提供的所有数据结构都支持在集群环境下使用，但每个数据结构只被保存在一个固定的槽内。[Redisson PRO](https://redisson.pro/)提供的自动分片功能能够将单个数据结构拆分，然后均匀的分布在整个集群里，而不是被挤在单一一个槽里。自动分片功能的优势主要有以下几点：

1. 单个数据结构可以充分利用整个集群内存资源，而不是被某一个节点的内存限制。
2. 将单个数据结构分片以后分布在集群中不同的节点里，不仅可以大幅提高读写性能，还能够保证读写性能随着集群的扩张而自动提升。

Redisson通过自身的分片算法，将一个大集合拆分为若干个片段（**默认231个，分片数量范围是3 - 16834**），然后将拆分后的片段均匀的分布到集群里各个节点里，保证每个节点分配到的片段数量大体相同。比如在默认情况下231个片段分到含有4个主节点的集群里，每个主节点将会分配到大约57个片段，同样的道理如果有5个主节点，每个节点会分配到大约46个片段。

目前支持的数据结构类型和服务包括[集（Set）](https://github.com/redisson/redisson/wiki/7.-分布式集合#732--集set数据分片sharding)、[映射（Map）](https://github.com/redisson/redisson/wiki/7.-分布式集合#数据分片功能sharding)、[BitSet](https://github.com/redisson/redisson/wiki/6.-分布式对象#641-bitset数据分片sharding分布式roaringbitmap)、[布隆过滤器（Bloom Filter）](https://github.com/redisson/redisson/wiki/6.-分布式对象#681-布隆过滤器数据分片sharding)、[Spring Cache](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#1421-spring-cache---本地缓存和数据分片)和[Hibernate Cache](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#1431-hibernate二级缓存---本地缓存和数据分片)。

*该功能仅限于[Redisson PRO](https://redisson.pro/)版本。*

# 分布式对象

​	每个Redisson对象实例都会有一个与之对应的Redis数据实例，可以通过调用`getName`方法来取得Redis数据实例的名称（key）。

```
RMap map = redisson.getMap("mymap");
map.getName(); // = mymap
```

​	所有与Redis key相关的操作都归纳在[`RKeys`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RKeys.html)这个接口里：

```java
RKeys keys = redisson.getKeys();

Iterable<String> allKeys = keys.getKeys();
Iterable<String> foundedKeys = keys.getKeysByPattern('key*');
long numOfDeletedKeys = keys.delete("obj1", "obj2", "obj3");
long deletedKeysAmount = keys.deleteByPattern("test?");
String randomKey = keys.randomKey();
long keysAmount = keys.count();
```

## 通用对象桶（Object Bucket）

​	Redisson的分布式[`RBucket`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBucket.html)Java对象是一种通用对象桶可以用来存放任类型的对象。 除了同步接口外，还提供了异步（[Async](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBucketAsync.html)）、反射式（[Reactive](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBucketReactive.html)）和[RxJava2](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBucketRx.html)标准的接口。

```java
RBucket<AnyObject> bucket = redisson.getBucket("anyObject");
bucket.set(new AnyObject(1));
AnyObject obj = bucket.get();

bucket.trySet(new AnyObject(3));
bucket.compareAndSet(new AnyObject(4), new AnyObject(5));
bucket.getAndSet(new AnyObject(6));
```

​	还可以通过[RBuckets](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBuckets.html)接口实现批量操作多个[RBucket](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBucket.html)对象：

```java
RBuckets buckets = redisson.getBuckets();
List<RBucket<V>> foundBuckets = buckets.find("myBucket*");
Map<String, V> loadedBuckets = buckets.get("myBucket1", "myBucket2", "myBucket3");

Map<String, Object> map = new HashMap<>();
map.put("myBucket1", new MyObject());
map.put("myBucket2", new MyObject());

// 利用Redis的事务特性，同时保存所有的通用对象桶，如果任意一个通用对象桶已经存在则放弃保存其他所有数据。
buckets.trySet(map);
// 同时保存全部通用对象桶。
buckets.set(map);
```

## 二进制流（Binary Stream）

Redisson的分布式[`RBinaryStream`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBinaryStream.html) Java对象同时提供了`InputStream`接口和`OutputStream`接口的实现。流的最大容量受Redis主节点的内存大小限制。

```java
RBinaryStream stream = redisson.getBinaryStream("anyStream");
byte[] content = ...
stream.set(content);

InputStream is = stream.getInputStream();
byte[] readBuffer = new byte[512];
is.read(readBuffer);

OutputStream os = stream.getOuputStream();
byte[] contentToWrite = ...
os.write(contentToWrite);
```

## 地理空间对象桶（Geospatial Bucket）

Redisson的分布式[`RGeo`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RGeo.html) Java对象是一种专门用来储存与地理位置有关的对象桶。除了同步接口外，还提供了异步（[Async](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RGeoAsync.html)）、反射式（[Reactive](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RGeoReactive.html)）和[RxJava2](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RGeoRx.html)标准的接口。

```java
RGeo<String> geo = redisson.getGeo("test");
geo.add(new GeoEntry(13.361389, 38.115556, "Palermo"),
        new GeoEntry(15.087269, 37.502669, "Catania"));
geo.addAsync(37.618423, 55.751244, "Moscow");

Double distance = geo.dist("Palermo", "Catania", GeoUnit.METERS);
geo.hashAsync("Palermo", "Catania");
Map<String, GeoPosition> positions = geo.pos("test2", "Palermo", "test3", "Catania", "test1");
List<String> cities = geo.radius(15, 37, 200, GeoUnit.KILOMETERS);
Map<String, GeoPosition> citiesWithPositions = geo.radiusWithPosition(15, 37, 200, GeoUnit.KILOMETERS);
```

## BitSet

Redisson的分布式[`RBitSet`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBitSet.html)Java对象采用了与`java.util.BiteSet`类似结构的设计风格。可以理解为它是一个分布式的可伸缩式位向量。需要注意的是`RBitSet`的大小受Redis限制，最大长度为`4 294 967 295`。除了同步接口外，还提供了异步（[Async](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBitSetAsync.html)）、反射式（[Reactive](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBitSetReactive.html)）和[RxJava2](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBitSetRx.html)标准的接口。

```java
RBitSet set = redisson.getBitSet("simpleBitset");
set.set(0, true);
set.set(1812, false);
set.clear(0);
set.addAsync("e");
set.xor("anotherBitset");
```

### BitSet数据分片（Sharding）（分布式RoaringBitMap）

基于Redis的Redisson集群分布式BitSet通过`RClusteredBitSet`接口，为集群状态下的Redis环境提供了BitSet数据分片的功能。通过优化后更加有效的分布式RoaringBitMap算法，突破了原有的BitSet大小限制，达到了集群物理内存容量大小。在[这里](https://github.com/redisson/redisson/wiki/5.-单个集合数据分片（Sharding）)可以获取更多的内部信息。

```java
RClusteredBitSet set = redisson.getClusteredBitSet("simpleBitset");
set.set(0, true);
set.set(1812, false);
set.clear(0);
set.addAsync("e");
set.xor("anotherBitset");
```

*该功能仅限于[Redisson PRO](https://redisson.pro/)版本。*

## 原子整长形（AtomicLong）

Redisson的分布式整长形[`RAtomicLong`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RAtomicLong.html)对象和Java中的`java.util.concurrent.atomic.AtomicLong`对象类似。除了同步接口外，还提供了异步（[Async](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RAtomicLongAsync.html)）、反射式（[Reactive](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RAtomicLongReactive.html)）和[RxJava2](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RAtomicLongRx.html)标准的接口。

```
RAtomicLong atomicLong = redisson.getAtomicLong("myAtomicLong");
atomicLong.set(3);
atomicLong.incrementAndGet();
atomicLong.get();
```

## 原子双精度浮点（AtomicDouble）

edisson还提供了分布式原子双精度浮点[`RAtomicDouble`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RAtomicDouble.html)，弥补了Java自身的不足。除了同步接口外，还提供了异步（[Async](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RAtomicDoubleAsync.html)）、反射式（[Reactive](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RAtomicDoubleReactive.html)）和[RxJava2](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RAtomicDoubleRx.html)标准的接口。

```java
RAtomicDouble atomicDouble = redisson.getAtomicDouble("myAtomicDouble");
atomicDouble.set(2.81);
atomicDouble.addAndGet(4.11);
atomicDouble.get();
```

## 话题（订阅分发）

Redisson的分布式话题[`RTopic`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RTopic.html对象实现了发布、订阅的机制。除了同步接口外，还提供了异步（[Async](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RTopicAsync.html)）、反射式（[Reactive](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RTopicReactive.html)）和[RxJava2](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RTopicRx.html)标准的接口。

```java
RTopic topic = redisson.getTopic("anyTopic");
topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(String channel, SomeObject message) {
        //...
    }
});

// 在其他线程或JVM节点
RTopic topic = redisson.getTopic("anyTopic");
long clientsReceivedMessage = topic.publish(new SomeObject());
```

在Redis节点故障转移（主从切换）或断线重连以后，所有的话题监听器将自动完成话题的重新订阅。

#### 模糊话题

​	Redisson的模糊话题[`RPatternTopic`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RPatternTopic.html)对象可以通过正式表达式来订阅多个话题。除了同步接口外，还提供了异步（[Async](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RPatternTopicAsync.html)）、反射式（[Reactive](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RPatternTopicReactive.html)）和[RxJava2](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RPatternTopicRx.html)标准的接口。

```java
// 订阅所有满足`topic1.*`表达式的话题
RPatternTopic topic1 = redisson.getPatternTopic("topic1.*");
int listenerId = topic1.addListener(Message.class, new PatternMessageListener<Message>() {
    @Override
    public void onMessage(String pattern, String channel, Message msg) {
         Assert.fail();
    }
});
```

在Redis节点故障转移（主从切换）或断线重连以后，所有的模糊话题监听器将自动完成话题的重新订阅。

## 布隆过滤器（Bloom Filter）

Redisson利用Redis实现了Java分布式[布隆过滤器（Bloom Filter）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RBloomFilter.html)。所含最大比特数量为`2^32`。

```java
RBloomFilter<SomeObject> bloomFilter = redisson.getBloomFilter("sample");
// 初始化布隆过滤器，预计统计元素数量为55000000，期望误差率为0.03
bloomFilter.tryInit(55000000L, 0.03);
bloomFilter.add(new SomeObject("field1Value", "field2Value"));
bloomFilter.add(new SomeObject("field5Value", "field8Value"));
bloomFilter.contains(new SomeObject("field1Value", "field8Value"));
```

#### 布隆过滤器数据分片（Sharding）

基于Redis的Redisson集群分布式布隆过滤器通过`RClusteredBloomFilter`接口，为集群状态下的Redis环境提供了布隆过滤器数据分片的功能。 通过优化后更加有效的算法，通过压缩未使用的比特位来释放集群内存空间。每个对象的状态都将被分布在整个集群中。所含最大比特数量为`2^64`。在[这里](https://github.com/redisson/redisson/wiki/5.-单个集合数据分片（Sharding）)可以获取更多的内部信息。

```java
RClusteredBloomFilter<SomeObject> bloomFilter = redisson.getClusteredBloomFilter("sample");
// 采用以下参数创建布隆过滤器
// expectedInsertions = 255000000
// falseProbability = 0.03
bloomFilter.tryInit(255000000L, 0.03);
bloomFilter.add(new SomeObject("field1Value", "field2Value"));
bloomFilter.add(new SomeObject("field5Value", "field8Value"));
bloomFilter.contains(new SomeObject("field1Value", "field8Value"));
```

*该功能仅限于[Redisson PRO](https://redisson.pro/)版本。*

## 基数估计算法（HyperLogLog）

Redisson利用Redis实现了Java分布式[基数估计算法（HyperLogLog）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RHyperLogLog.html)对象。该对象可以在有限的空间内通过概率算法统计大量的数据。除了同步接口外，还提供了异步（[Async](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RHyperLogLogAsync.html)）、反射式（[Reactive](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RHyperLogLogReactive.html)）和[RxJava2](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RHyperLogLogRx.html)标准的接口。

```java
RHyperLogLog<Integer> log = redisson.getHyperLogLog("log");
log.add(1);
log.add(2);
log.add(3);

log.count();
```

## 整长型累加器（LongAdder）

基于Redis的Redisson分布式[整长型累加器（LongAdder）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLongAdder.html)采用了与`java.util.concurrent.atomic.LongAdder`类似的接口。通过利用客户端内置的LongAdder对象，为分布式环境下递增和递减操作提供了很高得性能。据统计其性能最高比分布式`AtomicLong`对象快 **12000** 倍。完美适用于分布式统计计量场景。

```
RLongAdder atomicLong = redisson.getLongAdder("myLongAdder");
atomicLong.add(12);
atomicLong.increment();
atomicLong.decrement();
atomicLong.sum();
```

当不再使用整长型累加器对象的时候应该自行手动销毁，如果Redisson对象被关闭（shutdown）了，则不用手动销毁。

```java
RLongAdder atomicLong = ...
atomicLong.destroy();
```

## 双精度浮点累加器（DoubleAdder）

基于Redis的Redisson分布式[双精度浮点累加器（DoubleAdder）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RDoubleAdder.html)采用了与`java.util.concurrent.atomic.DoubleAdder`类似的接口。通过利用客户端内置的DoubleAdder对象，为分布式环境下递增和递减操作提供了很高得性能。据统计其性能最高比分布式`AtomicDouble`对象快 **12000** 倍。完美适用于分布式统计计量场景。

```
RLongDouble atomicDouble = redisson.getLongDouble("myLongDouble");
atomicDouble.add(12);
atomicDouble.increment();
atomicDouble.decrement();
atomicDouble.sum();
```

当不再使用双精度浮点累加器对象的时候应该自行手动销毁，如果Redisson对象被关闭（shutdown）了，则不用手动销毁。

```java
RLongDouble atomicDouble = ...

_b6d2063_
atomicDouble.destroy();
```

## 限流器（RateLimiter）

基于Redis的分布式[限流器（RateLimiter）](http://static.javadoc.io/org.redisson/redisson/3.10.6/org/redisson/api/RRateLimiter.html)可以用来在分布式环境下现在请求方的调用频率。既适用于不同Redisson实例下的多线程限流，也适用于相同Redisson实例下的多线程限流。该算法不保证公平性。除了同步接口外，还提供了异步（[Async](http://static.javadoc.io/org.redisson/redisson/3.10.6/org/redisson/api/RRateLimiterAsync.html)）、反射式（[Reactive](http://static.javadoc.io/org.redisson/redisson/3.10.6/org/redisson/api/RRateLimiterReactive.html)）和[RxJava2](http://static.javadoc.io/org.redisson/redisson/3.10.6/org/redisson/api/RRateLimiterRx.html)标准的接口。

```java
RRateLimiter rateLimiter = redisson.getRateLimiter("myRateLimiter");
// 初始化
// 最大流速 = 每1秒钟产生10个令牌
rateLimiter.trySetRate(RateType.OVERALL, 10, 1, RateIntervalUnit.SECONDS);

CountDownLatch latch = new CountDownLatch(2);
limiter.acquire(3);
// ...

Thread t = new Thread(() -> {
    limiter.acquire(2);
    // ...        
});
```

# 分布式集合

## 映射（Map）

​	基于Redis的Redisson的分布式映射结构的[`RMap`](http://static.javadoc.io/org.redisson/redisson/3.10.6/org/redisson/api/RMap.html) Java对象实现了`java.util.concurrent.ConcurrentMap`接口和`java.util.Map`接口。与HashMap不同的是，RMap保持了元素的插入顺序。该对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

​	除了同步接口外，还提供了[异步（Async）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RMapAsync.html)、[反射式（Reactive）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RMapReactive.html)和[RxJava2标准](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RMapRx.html)的接口。如果你想用Redis Map来保存你的POJO的话，可以考虑使用[分布式实时对象（Live Object）](https://github.com/redisson/redisson/wiki/9.-分布式服务#92-分布式实时对象live-object服务)服务。

​	在特定的场景下，映射缓存（Map）上的高度频繁的读取操作，使网络通信都被视为瓶颈时，可以使用Redisson提供的带有[本地缓存](https://github.com/redisson/redisson/wiki/7.-分布式集合#712-本地缓存映射localcachedmap)功能的映射。

```java
RMap<String, SomeObject> map = redisson.getMap("anyMap");
SomeObject prevObject = map.put("123", new SomeObject());
SomeObject currentObject = map.putIfAbsent("323", new SomeObject());
SomeObject obj = map.remove("123");

map.fastPut("321", new SomeObject());
map.fastRemove("321");

RFuture<SomeObject> putAsyncFuture = map.putAsync("321");
RFuture<Void> fastPutAsyncFuture = map.fastPutAsync("321");

map.fastPutAsync("321", new SomeObject());
map.fastRemoveAsync("321");
```

映射的字段锁的用法：

```java
RMap<MyKey, MyValue> map = redisson.getMap("anyMap");
MyKey k = new MyKey();
RLock keyLock = map.getLock(k);
keyLock.lock();
try {
   MyValue v = map.get(k);
   // 其他业务逻辑
} finally {
   keyLock.unlock();
}

RReadWriteLock rwLock = map.getReadWriteLock(k);
rwLock.readLock().lock();
try {
   MyValue v = map.get(k);
   // 其他业务逻辑
} finally {
   keyLock.readLock().unlock();
}
```

### 映射（Map）的元素淘汰（Eviction），本地缓存（LocalCache）和数据分片（Sharding）

Redisson提供了一系列的映射类型的数据结构，这些结构按特性主要分为三大类：

- **元素淘汰（Eviction）** 类 -- 带有元素淘汰（Eviction）机制的映射类允许针对一个映射中每个元素单独设定 **有效时间** 和 **最长闲置时间** 。
- **本地缓存（LocalCache）** 类 -- 本地缓存（Local Cache）也叫就近缓存（Near Cache）。这类映射的使用主要用于在特定的场景下，映射缓存（MapCache）上的高度频繁的读取操作，使网络通信都被视为瓶颈的情况。Redisson与Redis通信的同时，还将部分数据保存在本地内存里。这样的设计的好处是它能将读取速度提高最多 **45倍** 。 所有同名的本地缓存共用一个订阅发布话题，所有更新和过期消息都将通过该话题共享。
- **数据分片（Sharding）** 类 -- 数据分片（Sharding）类仅适用于Redis集群环境下，因此带有数据分片（Sharding）功能的映射也叫集群分布式映射。它利用[分库的原理](https://github.com/redisson/redisson/wiki/5.-单个集合数据分片)，将单一一个映射结构切分为若干个小的映射，并均匀的分布在集群中的各个槽里。这样的设计能使一个单一映射结构突破Redis自身的容量限制，让其容量随集群的扩大而增长。在扩容的同时，还能够使读写性能和元素淘汰处理能力随之成线性增长。

以下列表是Redisson提供的所有映射的名称及其特性：

| 接口名称 中文名称                                            | RedissonClient 对应的构造方法      | 本地缓存功能 Local Cache | 数据分片功能 Sharding | 元素淘汰功能 Eviction |
| ------------------------------------------------------------ | ---------------------------------- | ------------------------ | --------------------- | --------------------- |
| RMap 映射                                                    | getMap()                           | No                       | No                    | No                    |
| RMapCache 映射缓存                                           | getMapCache()                      | No                       | No                    | **Yes**               |
| RLocalCachedMap 本地缓存映射                                 | getLocalCachedMap()                | **Yes**                  | No                    | No                    |
| RLocalCachedMap Cache 本地缓存映射缓存 *仅限于[Redisson PRO](https://redisson.pro/)版本* | getLocalCachedMapCache()           | **Yes**                  | No                    | **Yes**               |
| RClusteredMap 集群分布式映射存 *仅限于[Redisson PRO](https://redisson.pro/)版本* | getClusteredMap()                  | No                       | **Yes**               | No                    |
| RClusteredMapCache 集群分布式映射缓存存 *仅限于[Redisson PRO](https://redisson.pro/)版本* | getClusteredMapCache()             | No                       | **Yes**               | **Yes**               |
| RClusteredLocal CachedMap 集群分布式本地缓存映射存 *仅限于[Redisson PRO](https://redisson.pro/)版本* | getClusteredLocal CachedMap()      | **Yes**                  | **Yes**               | No                    |
| RClusteredLocal CachedMapCache 集群分布式本地缓存映射缓存存 *仅限于[Redisson PRO](https://redisson.pro/)版本* | getClusteredLocal CachedMapCache() | **Yes**                  | **Yes**               | **Yes**               |

除此以外，Redisson还提供了[Spring Cache](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#142-spring-cache整合)和[JCache](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#144-java缓存标准规范jcache-api-jsr-107)的实现。

### 元素淘汰功能（Eviction）

​	Redisson的分布式的[`RMapCache`](http://static.javadoc.io/org.redisson/redisson/3.10.6/org/redisson/api/RMapCache.html) Java对象在基于`RMap`的前提下实现了针对单个元素的淘汰机制。同时仍然保留了元素的插入顺序。由于`RMapCache`是基于`RMap`实现的，使它同时继承了`java.util.concurrent.ConcurrentMap`接口和`java.util.Map`接口。Redisson提供的[Spring Cache整合](https://github.com/mrniko/redisson/wiki/14.-第三方框架整合#141-spring-cache整合)以及[JCache](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#144-java缓存标准规范jcache-api-jsr-107)正是基于这样的功能来实现的。

​	目前的Redis自身并不支持散列（Hash）当中的元素淘汰，因此所有过期元素都是通过`org.redisson.EvictionScheduler`实例来实现定期清理的。为了保证资源的有效利用，每次运行最多清理300个过期元素。任务的启动时间将根据上次实际清理数量自动调整，间隔时间趋于1秒到1小时之间。比如该次清理时删除了300条元素，那么下次执行清理的时间将在1秒以后（最小间隔时间）。一旦该次清理数量少于上次清理数量，时间间隔将增加1.5倍。

```java
RMapCache<String, SomeObject> map = redisson.getMapCache("anyMap");
// 有效时间 ttl = 10分钟
map.put("key1", new SomeObject(), 10, TimeUnit.MINUTES);
// 有效时间 ttl = 10分钟, 最长闲置时间 maxIdleTime = 10秒钟
map.put("key1", new SomeObject(), 10, TimeUnit.MINUTES, 10, TimeUnit.SECONDS);

// 有效时间 = 3 秒钟
map.putIfAbsent("key2", new SomeObject(), 3, TimeUnit.SECONDS);
// 有效时间 ttl = 40秒钟, 最长闲置时间 maxIdleTime = 10秒钟
map.putIfAbsent("key2", new SomeObject(), 40, TimeUnit.SECONDS, 10, TimeUnit.SECONDS);
```

### 本地缓存功能（Local Cache）

在特定的场景下，映射（Map）上的高度频繁的读取操作，使网络通信都被视为瓶颈时，使用Redisson提供的带有本地缓存功能的分布式本地缓存映射`RLocalCachedMap`Java对象会是一个很好的选择。它同时实现了`java.util.concurrent.ConcurrentMap`和`java.util.Map`两个接口。本地缓存功能充分的利用了JVM的自身内存空间，对部分常用的元素实行就地缓存，这样的设计让读取操作的性能较[分布式映射](https://github.com/redisson/redisson/wiki/7.-分布式集合#71-映射map)相比提高最多 **45倍** 。以下配置参数可以用来创建这个实例：

```
LocalCachedMapOptions options = LocalCachedMapOptions.defaults()
      // 用于淘汰清除本地缓存内的元素
      // 共有以下几种选择:
      // LFU - 统计元素的使用频率，淘汰用得最少（最不常用）的。
      // LRU - 按元素使用时间排序比较，淘汰最早（最久远）的。
      // SOFT - 元素用Java的WeakReference来保存，缓存元素通过GC过程清除。
      // WEAK - 元素用Java的SoftReference来保存, 缓存元素通过GC过程清除。
      // NONE - 永不淘汰清除缓存元素。
     .evictionPolicy(EvictionPolicy.NONE)
     // 如果缓存容量值为0表示不限制本地缓存容量大小
     .cacheSize(1000)
      // 以下选项适用于断线原因造成了未收到本地缓存更新消息的情况。
      // 断线重连的策略有以下几种：
      // CLEAR - 如果断线一段时间以后则在重新建立连接以后清空本地缓存
      // LOAD - 在服务端保存一份10分钟的作废日志
      //        如果10分钟内重新建立连接，则按照作废日志内的记录清空本地缓存的元素
      //        如果断线时间超过了这个时间，则将清空本地缓存中所有的内容
      // NONE - 默认值。断线重连时不做处理。
     .reconnectionStrategy(ReconnectionStrategy.NONE)
      // 以下选项适用于不同本地缓存之间相互保持同步的情况
      // 缓存同步策略有以下几种：
      // INVALIDATE - 默认值。当本地缓存映射的某条元素发生变动时，同时驱逐所有相同本地缓存映射内的该元素
      // UPDATE - 当本地缓存映射的某条元素发生变动时，同时更新所有相同本地缓存映射内的该元素
      // NONE - 不做任何同步处理
     .syncStrategy(SyncStrategy.INVALIDATE)
      // 每个Map本地缓存里元素的有效时间，默认毫秒为单位
     .timeToLive(10000)
      // 或者
     .timeToLive(10, TimeUnit.SECONDS)
      // 每个Map本地缓存里元素的最长闲置时间，默认毫秒为单位
     .maxIdle(10000)
      // 或者
     .maxIdle(10, TimeUnit.SECONDS);
RLocalCachedMap<String, Integer> map = redisson.getLocalCachedMap("test", options);

String prevObject = map.put("123", 1);
String currentObject = map.putIfAbsent("323", 2);
String obj = map.remove("123");

// 在不需要旧值的情况下可以使用fast为前缀的类似方法
map.fastPut("a", 1);
map.fastPutIfAbsent("d", 32);
map.fastRemove("b");

RFuture<String> putAsyncFuture = map.putAsync("321");
RFuture<Void> fastPutAsyncFuture = map.fastPutAsync("321");

map.fastPutAsync("321", new SomeObject());
map.fastRemoveAsync("321");
```

当不再使用Map本地缓存对象的时候应该手动销毁，如果Redisson对象被关闭（shutdown）了，则不用手动销毁。

```java
RLocalCachedMap<String, Integer> map = ...
map.destroy();
```

**如何通过加载数据的方式来降低过期淘汰事件发布信息对网络的影响**

代码范例:

```java
public void loadData(String cacheName, Map<String, String> data) {
    RLocalCachedMap<String, String> clearMap = redisson.getLocalCachedMap(cacheName, 
            LocalCachedMapOptions.defaults().cacheSize(1).syncStrategy(SyncStrategy.INVALIDATE));
    RLocalCachedMap<String, String> loadMap = redisson.getLocalCachedMap(cacheName, 
            LocalCachedMapOptions.defaults().cacheSize(1).syncStrategy(SyncStrategy.NONE));
    
    loadMap.putAll(data);
    clearMap.clearLocalCache();
}
```

Map数据分片是Redis集群模式下的一个功能。Redisson提供的分布式集群映射`RClusteredMap` Java对象也是基于`RMap`实现的。它同时实现了`java.util.concurrent.ConcurrentMap`和`java.util.Map`两个接口。在[这里](https://github.com/redisson/redisson/wiki/5.-单个集合数据分片（Sharding）)可以获取更多的内部信息。

```
RClusteredMap<String, SomeObject> map = redisson.getClusteredMap("anyMap");

SomeObject prevObject = map.put("123", new SomeObject());
SomeObject currentObject = map.putIfAbsent("323", new SomeObject());
SomeObject obj = map.remove("123");

map.fastPut("321", new SomeObject());
map.fastRemove("321");
```

#### 数据分片功能（Sharding）

Map数据分片是Redis集群模式下的一个功能。Redisson提供的分布式集群映射`RClusteredMap` Java对象也是基于`RMap`实现的。它同时实现了`java.util.concurrent.ConcurrentMap`和`java.util.Map`两个接口。在[这里](https://github.com/redisson/redisson/wiki/5.-单个集合数据分片（Sharding）)可以获取更多的内部信息。

```
RClusteredMap<String, SomeObject> map = redisson.getClusteredMap("anyMap");

SomeObject prevObject = map.put("123", new SomeObject());
SomeObject currentObject = map.putIfAbsent("323", new SomeObject());
SomeObject obj = map.remove("123");

map.fastPut("321", new SomeObject());
map.fastRemove("321");
```

### 映射持久化方式（缓存策略）

Redisson供了将映射中的数据持久化到外部储存服务的功能。主要场景有一下几种：

1. 将Redisson的分布式映射类型作为业务和外部储存媒介之间的缓存。
2. 或是用来增加Redisson映射类型中数据的持久性，或是用来增加已被驱逐的数据的寿命。
3. 或是用来缓存数据库，Web服务或其他数据源的数据。

##### Read-through策略

通俗的讲，如果一个被请求的数据不存在于Redisson的映射中的时候，Redisson将通过预先配置好的`MapLoader`对象加载数据。

##### Write-through（数据同步写入）策略

在遇到映射中某条数据被更改时，Redisson会首先通过预先配置好的`MapWriter`对象写入到外部储存系统，然后再更新Redis内的数据。

##### Write-behind（数据异步写入）策略

对映射的数据的更改会首先写入到Redis，然后再使用异步的方式，通过`MapWriter`对象写入到外部储存系统。在并发环境下可以通过`writeBehindThreads`参数来控制写入线程的数量，已达到对外部储存系统写入并发量的控制。

以上策略适用于所有实现了`RMap`、`RMapCache`、`RLocalCachedMap`和`RLocalCachedMapCache`接口的对象。

##### 配置范例：

```
MapOptions<K, V> options = MapOptions.<K, V>defaults()
                              .writer(myWriter)
                              .loader(myLoader);

RMap<K, V> map = redisson.getMap("test", options);
// 或
RMapCache<K, V> map = redisson.getMapCache("test", options);
// 或
RLocalCachedMap<K, V> map = redisson.getLocalCachedMap("test", options);
// 或
RLocalCachedMapCache<K, V> map = redisson.getLocalCachedMapCache("test", options);
```

#### 映射监听器（Map Listener）

Redisson为所有实现了`RMapCache`或`RLocalCachedMapCache`接口的对象提供了监听以下事件的监听器：

事件 | 监听器 元素 **添加** 事件 | `org.redisson.api.map.event.EntryCreatedListener`
元素 **过期** 事件 | `org.redisson.api.map.event.EntryExpiredListener`
元素 **删除** 事件 | `org.redisson.api.map.event.EntryRemovedListener`
元素 **更新** 事件 | `org.redisson.api.map.event.EntryUpdatedListener`

使用范例：

```java
RMapCache<String, Integer> map = redisson.getMapCache("myMap");
// 或
RLocalCachedMapCache<String, Integer> map = redisson.getLocalCachedMapCache("myMap", options);

int updateListener = map.addListener(new EntryUpdatedListener<Integer, Integer>() {
     @Override
     public void onUpdated(EntryEvent<Integer, Integer> event) {
          event.getKey(); // 字段名
          event.getValue() // 新值
          event.getOldValue() // 旧值
          // ...
     }
});

int createListener = map.addListener(new EntryCreatedListener<Integer, Integer>() {
     @Override
     public void onCreated(EntryEvent<Integer, Integer> event) {
          event.getKey(); // 字段名
          event.getValue() // 值
          // ...
     }
});

int expireListener = map.addListener(new EntryExpiredListener<Integer, Integer>() {
     @Override
     public void onExpired(EntryEvent<Integer, Integer> event) {
          event.getKey(); // 字段名
          event.getValue() // 值
          // ...
     }
});

int removeListener = map.addListener(new EntryRemovedListener<Integer, Integer>() {
     @Override
     public void onRemoved(EntryEvent<Integer, Integer> event) {
          event.getKey(); // 字段名
          event.getValue() // 值
          // ...
     }
});

map.removeListener(updateListener);
map.removeListener(createListener);
map.removeListener(expireListener);
map.removeListener(removeListener);
```

#### LRU有界映射

Redisson提供了基于Redis的以LRU为驱逐策略的分布式LRU有界映射对象。顾名思义，分布式LRU有界映射允许通过对其中元素按使用时间排序处理的方式，主动移除超过规定容量限制的元素。

```java
RMapCache<String, String> map = redisson.getMapCache("map");
// 尝试将该映射的最大容量限制设定为10
map.trySetMaxSize(10);

// 将该映射的最大容量限制设定或更改为10
map.setMaxSize(10);

map.put("1", "2");
map.put("3", "3", 1, TimeUnit.SECONDS);
```

## 多值映射（Multimap）

​	基于Redis的Redisson的分布式`RMultimap` Java对象允许Map中的一个字段值包含多个元素。 字段总数受Redis限制，每个Multimap最多允许有`4 294 967 295`个不同字段。

### 基于集（Set）的多值映射（Multimap）

```java
RSetMultimap<SimpleKey, SimpleValue> map = redisson.getSetMultimap("myMultimap");
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("0"), new SimpleValue("2"));
map.put(new SimpleKey("3"), new SimpleValue("4"));

Set<SimpleValue> allValues = map.get(new SimpleKey("0"));

List<SimpleValue> newValues = Arrays.asList(new SimpleValue("7"), new SimpleValue("6"), new SimpleValue("5"));
Set<SimpleValue> oldValues = map.replaceValues(new SimpleKey("0"), newValues);

Set<SimpleValue> removedValues = map.removeAll(new SimpleKey("0"));
```

### 基于列表（List）的多值映射（Multimap）

基于List的Multimap在保持插入顺序的同时允许一个字段下包含重复的元素。

```java
RListMultimap<SimpleKey, SimpleValue> map = redisson.getListMultimap("test1");
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("0"), new SimpleValue("2"));
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("3"), new SimpleValue("4"));

List<SimpleValue> allValues = map.get(new SimpleKey("0"));

Collection<SimpleValue> newValues = Arrays.asList(new SimpleValue("7"), new SimpleValue("6"), new SimpleValue("5"));
List<SimpleValue> oldValues = map.replaceValues(new SimpleKey("0"), newValues);

List<SimpleValue> removedValues = map.removeAll(new SimpleKey("0"));
```

### 多值映射（Multimap）淘汰机制（Eviction）

Multimap对象的淘汰机制是通过不同的接口来实现的。它们是`RSetMultimapCache`接口和`RListMultimapCache`接口，分别对应的是Set和List的Multimaps。

所有过期元素都是通过`org.redisson.EvictionScheduler`实例来实现定期清理的。为了保证资源的有效利用，每次运行最多清理100个过期元素。任务的启动时间将根据上次实际清理数量自动调整，间隔时间趋于1秒到2小时之间。比如该次清理时删除了100条元素，那么下次执行清理的时间将在1秒以后（最小间隔时间）。一旦该次清理数量少于上次清理数量，时间间隔将增加1.5倍。

RSetMultimapCache的使用范例：

```java
RSetMultimapCache<String, String> multimap = redisson.getSetMultimapCache("myMultimap");
multimap.put("1", "a");
multimap.put("1", "b");
multimap.put("1", "c");

multimap.put("2", "e");
multimap.put("2", "f");

multimap.expireKey("2", 10, TimeUnit.MINUTES);
```

## 集（Set）

基于Redis的Redisson的分布式Set结构的`RSet` Java对象实现了`java.util.Set`接口。通过元素的相互状态比较保证了每个元素的唯一性。该对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```
RSet<SomeObject> set = redisson.getSet("anySet");
set.add(new SomeObject());
set.remove(new SomeObject());
```

[Redisson PRO](https://redisson.pro/)版本中的Set对象还可以在Redis集群环境下支持[单集合数据分片](https://github.com/redisson/redisson/wiki/5.-单个集合数据分片sharding)。

### 集（Set）淘汰机制（Eviction）

基于Redis的Redisson的分布式`RSetCache` Java对象在基于`RSet`的前提下实现了针对单个元素的淘汰机制。由于`RSetCache`是基于`RSet`实现的，使它还集成了`java.util.Set`接口。

目前的Redis自身并不支持Set当中的元素淘汰，因此所有过期元素都是通过`org.redisson.EvictionScheduler`实例来实现定期清理的。为了保证资源的有效利用，每次运行最多清理100个过期元素。任务的启动时间将根据上次实际清理数量自动调整，间隔时间趋于1秒到2小时之间。比如该次清理时删除了100条元素，那么下次执行清理的时间将在1秒以后（最小间隔时间）。一旦该次清理数量少于上次清理数量，时间间隔将增加1.5倍。

```java
RSetCache<SomeObject> set = redisson.getSetCache("anySet");
// ttl = 10 seconds
set.add(new SomeObject(), 10, TimeUnit.SECONDS);
```

#### 集（Set）数据分片（Sharding）

Set数据分片是Redis集群模式下的一个功能。Redisson提供的分布式`RClusteredSet` Java对象也是基于`RSet`实现的。在[这里](https://github.com/redisson/redisson/wiki/5.-单个集合数据分片（Sharding）)可以获取更多的信息。

```
RClusteredSet<SomeObject> set = redisson.getClusteredSet("anySet");
set.add(new SomeObject());
set.remove(new SomeObject());
```

除了`RClusteredSet`以外，Redisson还提供了另一种集群模式下的分布式集（Set），它不仅提供了透明的数据分片功能，还为每个元素提供了淘汰机制。`RClusteredSetCache `类分别同时提供了`RClusteredSet `和`RSetCache `这两个接口的实现。当然这些都是基于`java.util.Set`的接口实现上的。

*该功能仅限于[Redisson PRO](https://redisson.pro/)版本。*

## 有序集（SortedSet）

基于Redis的Redisson的分布式`RSortedSet` Java对象实现了`java.util.SortedSet`接口。在保证元素唯一性的前提下，通过比较器（Comparator）接口实现了对元素的排序。

```java
RSortedSet<Integer> set = redisson.getSortedSet("anySet");
set.trySetComparator(new MyComparator()); // 配置元素比较器
set.add(3);
set.add(1);
set.add(2);

set.removeAsync(0);
set.addAsync(5);
```

## 计分排序集（ScoredSortedSet）

基于Redis的Redisson的分布式`RScoredSortedSet` Java对象是一个可以按插入时指定的元素评分排序的集合。它同时还保证了元素的唯一性。

```java
RScoredSortedSet<SomeObject> set = redisson.getScoredSortedSet("simple");

set.add(0.13, new SomeObject(a, b));
set.addAsync(0.251, new SomeObject(c, d));
set.add(0.302, new SomeObject(g, d));

set.pollFirst();
set.pollLast();

int index = set.rank(new SomeObject(g, d)); // 获取元素在集合中的位置
Double score = set.getScore(new SomeObject(g, d)); // 获取元素的评分
```

## 字典排序集（LexSortedSet）

基于Redis的Redisson的分布式`RLexSortedSet` Java对象在实现了`java.util.Set<String>`接口的同时，将其中的所有字符串元素按照字典顺序排列。它公式还保证了字符串元素的唯一性。

```java
RLexSortedSet set = redisson.getLexSortedSet("simple");
set.add("d");
set.addAsync("e");
set.add("f");

set.lexRangeTail("d", false);
set.lexCountHead("e");
set.lexRange("d", true, "z", false);
```

## 列表（List）

基于Redis的Redisson分布式列表（List）结构的`RList` Java对象在实现了`java.util.List`接口的同时，确保了元素插入时的顺序。该对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```java
RList<SomeObject> list = redisson.getList("anyList");
list.add(new SomeObject());
list.get(0);
list.remove(new SomeObject());
```

## 队列（Queue）

基于Redis的Redisson分布式无界队列（Queue）结构的`RQueue` Java对象实现了`java.util.Queue`接口。尽管`RQueue`对象无初始大小（边界）限制，但对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```java
RQueue<SomeObject> queue = redisson.getQueue("anyQueue");
queue.add(new SomeObject());
SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
```

## 双端队列（Deque）

基于Redis的Redisson分布式无界双端队列（Deque）结构的`RDeque` Java对象实现了`java.util.Deque`接口。尽管`RDeque`对象无初始大小（边界）限制，但对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```java
RDeque<SomeObject> queue = redisson.getDeque("anyDeque");
queue.addFirst(new SomeObject());
queue.addLast(new SomeObject());
SomeObject obj = queue.removeFirst();
SomeObject someObj = queue.removeLast();
```

## 阻塞队列（Blocking Queue）

基于Redis的Redisson分布式无界阻塞队列（Blocking Queue）结构的`RBlockingQueue` Java对象实现了`java.util.concurrent.BlockingQueue`接口。尽管`RBlockingQueue`对象无初始大小（边界）限制，但对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```java
RBlockingQueue<SomeObject> queue = redisson.getBlockingQueue("anyQueue");
queue.offer(new SomeObject());

SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
SomeObject ob = queue.poll(10, TimeUnit.MINUTES);
```

`poll`, `pollFromAny`, `pollLastAndOfferFirstTo`和`take`方法内部采用话题订阅发布实现，在Redis节点故障转移（主从切换）或断线重连以后，内置的相关话题监听器将自动完成话题的重新订阅。

## 有界阻塞队列（Bounded Blocking Queue）

基于Redis的Redisson分布式有界阻塞队列（Bounded Blocking Queue）结构的`RBoundedBlockingQueue` Java对象实现了`java.util.concurrent.BlockingQueue`接口。该对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。队列的初始容量（边界）必须在使用前设定好。

```java
RBoundedBlockingQueue<SomeObject> queue = redisson.getBoundedBlockingQueue("anyQueue");
// 如果初始容量（边界）设定成功则返回`真（true）`，
// 如果初始容量（边界）已近存在则返回`假（false）`。
queue.trySetCapacity(2);

queue.offer(new SomeObject(1));
queue.offer(new SomeObject(2));
// 此时容量已满，下面代码将会被阻塞，直到有空闲为止。
queue.put(new SomeObject());

SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
SomeObject ob = queue.poll(10, TimeUnit.MINUTES);
```

`poll`, `pollFromAny`, `pollLastAndOfferFirstTo`和`take`方法内部采用话题订阅发布实现，在Redis节点故障转移（主从切换）或断线重连以后，内置的相关话题监听器将自动完成话题的重新订阅。

## 阻塞双端队列（Blocking Deque）

​	基于Redis的Redisson分布式无界阻塞双端队列（Blocking Deque）结构的`RBlockingDeque` Java对象实现了`java.util.concurrent.BlockingDeque`接口。尽管`RBlockingDeque`对象无初始大小（边界）限制，但对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```java
RBlockingDeque<Integer> deque = redisson.getBlockingDeque("anyDeque");
deque.putFirst(1);
deque.putLast(2);
Integer firstValue = queue.takeFirst();
Integer lastValue = queue.takeLast();
Integer firstValue = queue.pollFirst(10, TimeUnit.MINUTES);
Integer lastValue = queue.pollLast(3, TimeUnit.MINUTES);
```

## 阻塞公平队列（Blocking Fair Queue）

基于Redis的Redisson分布式无界阻塞公平队列（Blocking Fair Queue）结构的`RBlockingFairQueue` Java对象在实现[Redisson分布式无界阻塞队列（Blocking Queue）结构`RBlockingQueue`](https://github.com/redisson/redisson/wiki/7.-分布式集合#710-阻塞队列blocking-queue)接口的基础上，解决了多个队列消息的处理者在复杂的网络环境下，网络延时的影响使“较远”的客户端最终收到消息数量低于“较近”的客户端的问题。从而解决了这种现象引发的个别处理节点过载的情况。

以分布式无界阻塞队列为基础，采用公平获取消息的机制，不仅保证了`poll`、`pollFromAny`、`pollLastAndOfferFirstTo`和`take`方法获取消息的先入顺序，还能让队列里的消息被均匀的发布到处在复杂分布式环境中的各个处理节点里。

```
RBlockingFairQueue queue = redisson.getBlockingFairQueue("myQueue");
queue.offer(new SomeObject());

SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
SomeObject ob = queue.poll(10, TimeUnit.MINUTES);
```

*该功能仅限于[Redisson PRO](https://redisson.pro/)版本。*

## 阻塞公平双端队列（Blocking Fair Deque）

基于Redis的Redisson分布式无界阻塞公平双端队列（Blocking Fair Deque）结构的`RBlockingFairDeque` Java对象在实现[Redisson分布式无界阻塞双端队列（Blocking Deque）结构`RBlockingDeque`](https://github.com/redisson/redisson/wiki/7.-分布式集合#712-阻塞双端队列blocking-deque)接口的基础上，解决了多个队列消息的处理者在复杂的网络环境下，网络延时的影响使“较远”的客户端最终收到消息数量低于“较近”的客户端的问题。从而解决了这种现象引发的个别处理节点过载的情况。

以分布式无界阻塞双端队列为基础，采用公平获取消息的机制，不仅保证了`poll`、`take`、`pollFirst`、`takeFirst`、`pollLast`和`takeLast`方法获取消息的先入顺序，还能让队列里的消息被均匀的发布到处在复杂分布式环境中的各个处理节点里。

```
RBlockingFairDeque deque = redisson.getBlockingFairDeque("myDeque");
deque.offer(new SomeObject());

SomeObject firstElement = queue.peekFirst();
SomeObject firstElement = queue.pollFirst();
SomeObject firstElement = queue.pollFirst(10, TimeUnit.MINUTES);
SomeObject firstElement = queue.takeFirst();

SomeObject lastElement = queue.peekLast();
SomeObject lastElement = queue.pollLast();
SomeObject lastElement = queue.pollLast(10, TimeUnit.MINUTES);
SomeObject lastElement = queue.takeLast();
```

*该功能仅限于[Redisson PRO](https://redisson.pro/)版本。*

## 延迟队列（Delayed Queue）

基于Redis的Redisson分布式延迟队列（Delayed Queue）结构的`RDelayedQueue` Java对象在实现了`RQueue`接口的基础上提供了向队列按要求延迟添加项目的功能。该功能可以用来实现消息传送延迟按几何增长或几何衰减的发送策略。

```
RQueue<String> distinationQueue = ...
RDelayedQueue<String> delayedQueue = getDelayedQueue(distinationQueue);
// 10秒钟以后将消息发送到指定队列
delayedQueue.offer("msg1", 10, TimeUnit.SECONDS);
// 一分钟以后将消息发送到指定队列
delayedQueue.offer("msg2", 1, TimeUnit.MINUTES);
```

在该对象不再需要的情况下，应该主动销毁。仅在相关的Redisson对象也需要关闭的时候可以不用主动销毁。

```java
RDelayedQueue<String> delayedQueue = ...
delayedQueue.destroy();
```

## 优先队列（Priority Queue）

基于Redis的Redisson分布式优先队列（Priority Queue）Java对象实现了`java.util.Queue`的接口。可以通过比较器（Comparator）接口来对元素排序。

```java
RPriorityQueue<Integer> queue = redisson.getPriorityQueue("anyQueue");
queue.trySetComparator(new MyComparator()); // 指定对象比较器
queue.add(3);
queue.add(1);
queue.add(2);

queue.removeAsync(0);
queue.addAsync(5);

queue.poll();
```

## 优先双端队列（Priority Deque）

​	基于Redis的Redisson分布式优先双端队列（Priority Deque）Java对象实现了`java.util.Deque`的接口。可以通过比较器（Comparator）接口来对元素排序。

```java
RPriorityDeque<Integer> queue = redisson.getPriorityDeque("anyQueue");
queue.trySetComparator(new MyComparator()); // 指定对象比较器
queue.addLast(3);
queue.addFirst(1);
queue.add(2);

queue.removeAsync(0);
queue.addAsync(5);

queue.pollFirst();
queue.pollLast();
```

## 优先阻塞队列（Priority Blocking Queue）

基于Redis的分布式无界优先阻塞队列（Priority Blocking Queue）Java对象的结构与`java.util.concurrent.PriorityBlockingQueue`类似。可以通过比较器（Comparator）接口来对元素排序。`PriorityBlockingQueue`的最大容量是`4 294 967 295`个元素。

```java
RPriorityBlockingQueue<Integer> queue = redisson.getPriorityBlockingQueue("anyQueue");
queue.trySetComparator(new MyComparator()); // 指定对象比较器
queue.add(3);
queue.add(1);
queue.add(2);

queue.removeAsync(0);
queue.addAsync(5);

queue.take();
```

当Redis服务端断线重连以后，或Redis服务端发生主从切换，并重新建立连接后，断线时正在使用`poll`，`pollLastAndOfferFirstTo`或`take`方法的对象Redisson将自动再次为其订阅相关的话题。

### 优先阻塞双端队列（Priority Blocking Deque）

基于Redis的分布式无界优先阻塞双端队列（Priority Blocking Deque）Java对象实现了`java.util.Deque`的接口。`addLast`、 `addFirst`、`push`方法不能再这个对里使用。`PriorityBlockingDeque`的最大容量是`4 294 967 295`个元素。

```java
RPriorityBlockingDeque<Integer> queue = redisson.getPriorityBlockingDeque("anyQueue");
queue.trySetComparator(new MyComparator()); // 指定对象比较器
queue.add(2);

queue.removeAsync(0);
queue.addAsync(5);

queue.pollFirst();
queue.pollLast();
queue.takeFirst();
queue.takeLast();
```

当Redis服务端断线重连以后，或Redis服务端发生主从切换，并重新建立连接后，断线时正在使用`poll`，`pollLastAndOfferFirstTo`或`take`方法的对象Redisson将自动再次为其订阅相关的话题。

# 分布式锁和同步器

## 可重入锁（Reentrant Lock）

基于Redis的Redisson分布式可重入锁[`RLock`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLock.html) Java对象实现了`java.util.concurrent.locks.Lock`接口。同时还提供了[异步（Async）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockAsync.html)、[反射式（Reactive）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockReactive.html)和[RxJava2标准](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockRx.html)的接口。

```
RLock lock = redisson.getLock("anyLock");
// 最常见的使用方法
lock.lock();
```

大家都知道，如果负责储存这个分布式锁的Redisson节点宕机以后，而且这个锁正好处于锁住的状态时，这个锁会出现锁死的状态。为了避免这种情况的发生，Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期。默认情况下，看门狗的检查锁的超时时间是30秒钟，也可以通过修改[Config.lockWatchdogTimeout](https://github.com/redisson/redisson/wiki/2.-配置方法#lockwatchdogtimeout监控锁的看门狗超时单位毫秒)来另行指定。

另外Redisson还通过加锁的方法提供了`leaseTime`的参数来指定加锁的时间。超过这个时间后锁便自动解开了。

```java
// 加锁以后10秒钟自动解锁
// 无需调用unlock方法手动解锁
lock.lock(10, TimeUnit.SECONDS);

// 尝试加锁，最多等待100秒，上锁以后10秒自动解锁
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
if (res) {
   try {
     ...
   } finally {
       lock.unlock();
   }
}
```

Redisson同时还为分布式锁提供了异步执行的相关方法：

```
RLock lock = redisson.getLock("anyLock");
lock.lockAsync();
lock.lockAsync(10, TimeUnit.SECONDS);
Future<Boolean> res = lock.tryLockAsync(100, 10, TimeUnit.SECONDS);
```

`RLock`对象完全符合Java的Lock规范。也就是说只有拥有锁的进程才能解锁，其他进程解锁则会抛出`IllegalMonitorStateException`错误。但是如果遇到需要其他进程也能解锁的情况，请使用[分布式信号量`Semaphore`](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#86-信号量semaphore) 对象.

##  公平锁（Fair Lock）

基于Redis的Redisson分布式可重入公平锁也是实现了`java.util.concurrent.locks.Lock`接口的一种`RLock`对象。同时还提供了[异步（Async）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockAsync.html)、[反射式（Reactive）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockReactive.html)和[RxJava2标准](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockRx.html)的接口。它保证了当多个Redisson客户端线程同时请求加锁时，优先分配给先发出请求的线程。所有请求线程会在一个队列中排队，当某个线程出现宕机时，Redisson会等待5秒后继续下一个线程，也就是说如果前面有5个线程都处于等待状态，那么后面的线程会等待至少25秒。

```java
RLock fairLock = redisson.getFairLock("anyLock");
// 最常见的使用方法
fairLock.lock();
```

大家都知道，如果负责储存这个分布式锁的Redis节点宕机以后，而且这个锁正好处于锁住的状态时，这个锁会出现锁死的状态。为了避免这种情况的发生，Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期。默认情况下，看门狗的检查锁的超时时间是30秒钟，也可以通过修改[Config.lockWatchdogTimeout](https://github.com/redisson/redisson/wiki/2.-配置方法#lockwatchdogtimeout监控锁的看门狗超时单位毫秒)来另行指定。

另外Redisson还通过加锁的方法提供了`leaseTime`的参数来指定加锁的时间。超过这个时间后锁便自动解开了。

```java
// 10秒钟以后自动解锁
// 无需调用unlock方法手动解锁
fairLock.lock(10, TimeUnit.SECONDS);

// 尝试加锁，最多等待100秒，上锁以后10秒自动解锁
boolean res = fairLock.tryLock(100, 10, TimeUnit.SECONDS);
...
fairLock.unlock();
```

Redisson同时还为分布式可重入公平锁提供了异步执行的相关方法：

```java
RLock fairLock = redisson.getFairLock("anyLock");
fairLock.lockAsync();
fairLock.lockAsync(10, TimeUnit.SECONDS);
Future<Boolean> res = fairLock.tryLockAsync(100, 10, TimeUnit.SECONDS);
```

## 联锁（MultiLock）

基于Redis的Redisson分布式联锁[`RedissonMultiLock`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/RedissonMultiLock.html)对象可以将多个`RLock`对象关联为一个联锁，每个`RLock`对象实例可以来自于不同的Redisson实例。

```
RLock lock1 = redissonInstance1.getLock("lock1");
RLock lock2 = redissonInstance2.getLock("lock2");
RLock lock3 = redissonInstance3.getLock("lock3");

RedissonMultiLock lock = new RedissonMultiLock(lock1, lock2, lock3);
// 同时加锁：lock1 lock2 lock3
// 所有的锁都上锁成功才算成功。
lock.lock();
...
lock.unlock();
```

大家都知道，如果负责储存某些分布式锁的某些Redis节点宕机以后，而且这些锁正好处于锁住的状态时，这些锁会出现锁死的状态。为了避免这种情况的发生，Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期。默认情况下，看门狗的检查锁的超时时间是30秒钟，也可以通过修改[Config.lockWatchdogTimeout](https://github.com/redisson/redisson/wiki/2.-配置方法#lockwatchdogtimeout监控锁的看门狗超时单位毫秒)来另行指定。

另外Redisson还通过加锁的方法提供了`leaseTime`的参数来指定加锁的时间。超过这个时间后锁便自动解开了。

```java
RedissonMultiLock lock = new RedissonMultiLock(lock1, lock2, lock3);
// 给lock1，lock2，lock3加锁，如果没有手动解开的话，10秒钟后将会自动解开
lock.lock(10, TimeUnit.SECONDS);

// 为加锁等待100秒时间，并在加锁成功10秒钟后自动解开
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
...
lock.unlock();
```

## 红锁（RedLock）

基于Redis的Redisson红锁`RedissonRedLock`对象实现了[Redlock](http://redis.cn/topics/distlock.html)介绍的加锁算法。该对象也可以用来将多个`RLock`对象关联为一个红锁，每个`RLock`对象实例可以来自于不同的Redisson实例。

```java
RLock lock1 = redissonInstance1.getLock("lock1");
RLock lock2 = redissonInstance2.getLock("lock2");
RLock lock3 = redissonInstance3.getLock("lock3");

RedissonRedLock lock = new RedissonRedLock(lock1, lock2, lock3);
// 同时加锁：lock1 lock2 lock3
// 红锁在大部分节点上加锁成功就算成功。
lock.lock();
...
lock.unlock();
```

大家都知道，如果负责储存某些分布式锁的某些Redis节点宕机以后，而且这些锁正好处于锁住的状态时，这些锁会出现锁死的状态。为了避免这种情况的发生，Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期。默认情况下，看门狗的检查锁的超时时间是30秒钟，也可以通过修改[Config.lockWatchdogTimeout](https://github.com/redisson/redisson/wiki/2.-配置方法#lockwatchdogtimeout监控锁的看门狗超时单位毫秒)来另行指定。

另外Redisson还通过加锁的方法提供了`leaseTime`的参数来指定加锁的时间。超过这个时间后锁便自动解开了。

```java
RedissonRedLock lock = new RedissonRedLock(lock1, lock2, lock3);
// 给lock1，lock2，lock3加锁，如果没有手动解开的话，10秒钟后将会自动解开
lock.lock(10, TimeUnit.SECONDS);

// 为加锁等待100秒时间，并在加锁成功10秒钟后自动解开
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
...
lock.unlock();
```

## 读写锁（ReadWriteLock）

基于Redis的Redisson分布式可重入读写锁[`RReadWriteLock`](http://static.javadoc.io/org.redisson/redisson/3.4.3/org/redisson/api/RReadWriteLock.html) Java对象实现了`java.util.concurrent.locks.ReadWriteLock`接口。其中读锁和写锁都继承了[RLock](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#81-可重入锁reentrant-lock)接口。

分布式可重入读写锁允许同时有多个读锁和一个写锁处于加锁状态。

```java
RReadWriteLock rwlock = redisson.getReadWriteLock("anyRWLock");
// 最常见的使用方法
rwlock.readLock().lock();
// 或
rwlock.writeLock().lock();
```

大家都知道，如果负责储存这个分布式锁的Redis节点宕机以后，而且这个锁正好处于锁住的状态时，这个锁会出现锁死的状态。为了避免这种情况的发生，Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期。默认情况下，看门狗的检查锁的超时时间是30秒钟，也可以通过修改[Config.lockWatchdogTimeout](https://github.com/redisson/redisson/wiki/2.-配置方法#lockwatchdogtimeout监控锁的看门狗超时单位毫秒)来另行指定。

另外Redisson还通过加锁的方法提供了`leaseTime`的参数来指定加锁的时间。超过这个时间后锁便自动解开了。

```java
// 10秒钟以后自动解锁
// 无需调用unlock方法手动解锁
rwlock.readLock().lock(10, TimeUnit.SECONDS);
// 或
rwlock.writeLock().lock(10, TimeUnit.SECONDS);

// 尝试加锁，最多等待100秒，上锁以后10秒自动解锁
boolean res = rwlock.readLock().tryLock(100, 10, TimeUnit.SECONDS);
// 或
boolean res = rwlock.writeLock().tryLock(100, 10, TimeUnit.SECONDS);
...
lock.unlock();
```

## 信号量（Semaphore）

基于Redis的Redisson的分布式信号量（[Semaphore](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphore.html)）Java对象`RSemaphore`采用了与`java.util.concurrent.Semaphore`相似的接口和用法。同时还提供了[异步（Async）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphoreAsync.html)、[反射式（Reactive）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphoreReactive.html)和[RxJava2标准](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphoreRx.html)的接口。

```java
RSemaphore semaphore = redisson.getSemaphore("semaphore");
semaphore.acquire();
//或
semaphore.acquireAsync();
semaphore.acquire(23);
semaphore.tryAcquire();
//或
semaphore.tryAcquireAsync();
semaphore.tryAcquire(23, TimeUnit.SECONDS);
//或
semaphore.tryAcquireAsync(23, TimeUnit.SECONDS);
semaphore.release(10);
semaphore.release();
//或
semaphore.releaseAsync();
```

### 可过期性信号量（PermitExpirableSemaphore）

基于Redis的Redisson可过期性信号量（[PermitExpirableSemaphore](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RPermitExpirableSemaphore.html)）是在`RSemaphore`对象的基础上，为每个信号增加了一个过期时间。每个信号可以通过独立的ID来辨识，释放时只能通过提交这个ID才能释放。它提供了[异步（Async）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RPermitExpirableSemaphoreAsync.html)、[反射式（Reactive）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RPermitExpirableSemaphoreReactive.html)和[RxJava2标准](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RPermitExpirableSemaphoreRx.html)的接口。

```java
RPermitExpirableSemaphore semaphore = redisson.getPermitExpirableSemaphore("mySemaphore");
String permitId = semaphore.acquire();
// 获取一个信号，有效期只有2秒钟。
String permitId = semaphore.acquire(2, TimeUnit.SECONDS);
// ...
semaphore.release(permitId);
```

## 闭锁（CountDownLatch）

基于Redisson的Redisson分布式闭锁（[CountDownLatch](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RCountDownLatch.html)）Java对象`RCountDownLatch`采用了与`java.util.concurrent.CountDownLatch`相似的接口和用法。

```java
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.trySetCount(1);
latch.await();

// 在其他线程或其他JVM里
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.countDown();
```

# 分布式服务(略)





# 额外功能

## 对Redis节点的操作

Redisson的`NodesGroup`对象提供了许些对Redis节点的操作。

```
NodesGroup nodesGroup = redisson.getNodesGroup();
nodesGroup.addConnectionListener(new ConnectionListener() {
    public void onConnect(InetSocketAddress addr) {
       // Redis节点连接成功
    }

    public void onDisconnect(InetSocketAddress addr) {
       // Redis节点连接断开
    }
});
```

也可以用来PING单个Redis节点或全部节点。

```java
NodesGroup nodesGroup = redisson.getNodesGroup();
Collection<Node> allNodes = nodesGroup.getNodes();
for (Node n : allNodes) {
    n.ping();
}
// 或者
nodesGroup.pingAll();
```

## 复杂多维对象结构和对象引用的支持

Redisson突破了Redis数据结构维度的限制，通过一个特殊引用对象的帮助，Redisson允许以任意的组合方式构建多维度的复杂对象结构，实现了对象之间的类似传统数据库里的关联关系。使用范例如下：

```
RMap<RSet<RList>, RList<RMap>> map = redisson.getMap("myMap");
RSet<RList> set = redisson.getSet("mySet");
RList<RMap> list = redisson.getList("myList");

map.put(set, list);
// 在特殊引用对象的帮助下，我们甚至可以构建一个循环引用，这是通过普通序列化方式实现不了的。
set.add(list);
list.add(map);
```

可能您已经注意到了，在map包含的元素发生改变以后，我们无需再次“保存/持久”这些对象。因为map对象所记录的并不是序列化以后的值，而是元素对象的引用。这让Redisson提供的对象在使用方法上，与普通Java对象的使用方法一致。从而让Redis成为内存的一部分，而不仅仅是一个储存空间。

以上范例中，一共创建了三个Redis数据结构：一个Redis HASH，一个Redis SET和一个Redis LIST。

## 命令的批量执行

多个连续命令可以通过`RBatch`对象在一次网络会话请求里合并发送，这样省去了产生多个请求消耗的时间和资源。这在Redis中叫做[管道](http://redis.cn/topics/pipelining.html#redis-pipelining)。

用户可以通过以下方式调整通过管道方式发送命令的方式：

```java
BatchOptions options = BatchOptions.defaults()
// 指定执行模式
//
// ExecutionMode.REDIS_READ_ATOMIC - 所有命令缓存在Redis节点中，以原子性事务的方式执行。
//
// ExecutionMode.REDIS_WRITE_ATOMIC - 所有命令缓存在Redis节点中，以原子性事务的方式执行。
//
// ExecutionMode.IN_MEMORY - 所有命令缓存在Redisson本机内存中统一发送，但逐一执行（非事务）。默认模式。
//
// ExecutionMode.IN_MEMORY_ATOMIC - 所有命令缓存在Redisson本机内存中统一发送，并以原子性事务的方式执行。
//
.executionMode(ExecutionMode.IN_MEMORY)

// 告知Redis不用返回结果（可以减少网络用量）
.skipResult()

// 将写入操作同步到从节点
// 同步到2个从节点，等待时间为1秒钟
.syncSlaves(2, 1, TimeUnit.SECONDS)

// 处理结果超时为2秒钟
.responseTimeout(2, TimeUnit.SECONDS)

// 命令重试等待间隔时间为2秒钟
.retryInterval(2, TimeUnit.SECONDS);

// 命令重试次数。仅适用于未发送成功的命令
.retryAttempts(4);
```

使用方式如下：

```java
RBatch batch = redisson.createBatch();
batch.getMap("test").fastPutAsync("1", "2");
batch.getMap("test").fastPutAsync("2", "3");
batch.getMap("test").putAsync("2", "5");
batch.getAtomicLongAsync("counter").incrementAndGetAsync();
batch.getAtomicLongAsync("counter").incrementAndGetAsync();

BatchResult res = batch.execute();
// 或者
Future<BatchResult> asyncRes = batch.executeAsync();
List<?> response = res.getResponses();
res.getSyncedSlaves();
```

在集群模式下，所有的命令会按各个槽所在的节点，筛选分配到各个节点并同时发送。每个节点返回的结果将会汇总到最终的结果列表里。

## Redisson事务

Redisson为`RMap`、`RMapCache`、`RLocalCachedMap`、`RSet`、`RSetCache`和`RBucket`这样的对象提供了具有ACID属性的事务功能。Redisson事务通过分布式锁保证了连续写入的原子性，同时在内部通过操作指令队列实现了Redis原本没有的`提交`与`滚回`功能。当`提交`与`滚回`遇到问题的时候，将通过`org.redisson.transaction.TransactionException`告知用户。

目前支持的环境如下： `SINGLE`, `MASTER/SLAVE`, `SENTINEL`, `ELASTICACHE REPLICATED`, `AZURE CACHE`, `RLEC`。

Redisson事务支持的事务隔离等级为: `READ_COMMITTED`，即仅读取提交后的结果。

另见 [Spring事务管理器](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#147-spring事务管理器spring-transaction-manager) 和本章 [XA事务（XA Transactions）](https://github.com/redisson/redisson/wiki/10.-额外功能#105-xa事务xa-transactions)。

以下选项可以用来配置事务属性：

```java
TransactionOptions options = TransactionOptions.defaults()
// 设置参与本次事务的主节点与其从节点同步的超时时间。
// 默认值是5秒。
.syncSlavesTimeout(5, TimeUnit.SECONDS)

// 处理结果超时。
// 默认值是3秒。
.responseTimeout(3, TimeUnit.SECONDS)

// 命令重试等待间隔时间。仅适用于未发送成功的命令。
// 默认值是1.5秒。
.retryInterval(2, TimeUnit.SECONDS)

// 命令重试次数。仅适用于未发送成功的命令。
// 默认值是3次。
.retryAttempts(3)

// 事务超时时间。如果规定时间内没有提交该事务则自动滚回。
// 默认值是5秒。
.timeout(5, TimeUnit.SECONDS);
```

代码范例：

```java
RTransaction transaction = redisson.createTransaction(TransactionOptions.defaults());

RMap<String, String> map = transaction.getMap("myMap");
map.put("1", "2");
String value = map.get("3");
RSet<String> set = transaction.getSet("mySet")
set.add(value);

try {
   transaction.commit();
} catch(TransactionException e) {
   transaction.rollback();
}
```

## XA事务（XA Transactions）

Redisson提供了[XAResource](https://docs.oracle.com/javaee/7/api/javax/transaction/xa/XAResource.html)标准的实现。该实现可用于JTA事务中。

另见本章[Redisson事务](https://github.com/redisson/redisson/wiki/10.-额外功能#104-redisson事务)和[Spring事务管理器](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#147-spring事务管理器spring-transaction-manager)。

*该功能仅适用于[Redisson PRO](https://redisson.pro/)版本*

代码范例：

```java
// Transaction对象可以从所有兼容JTA接口的事务管理器中获取。
Transaction globalTransaction = transactionManager.getTransaction();

RXAResource xaResource = redisson.getXAResource();
globalTransaction.enlistResource(xaResource);

RTransaction transaction = xaResource.getTransaction();
RBucket<String> bucket = transaction.getBucket("myBucket");
bucket.set("simple");
RMap<String, String> map = transaction.getMap("myMap");
map.put("myKey", "myValue");

transactionManager.commit();
```

## 脚本执行

```java
redisson.getBucket("foo").set("bar");
String r = redisson.getScript().eval(Mode.READ_ONLY,
   "return redis.call('get', 'foo')", RScript.ReturnType.VALUE);

// 通过预存的脚本进行同样的操作
RScript s = redisson.getScript();
// 首先将脚本保存到所有的Redis主节点
String res = s.scriptLoad("return redis.call('get', 'foo')");
// 返回值 res == 282297a0228f48cd3fc6a55de6316f31422f5d17

// 再通过SHA值调用脚本
Future<Object> r1 = redisson.getScript().evalShaAsync(Mode.READ_ONLY,
   "282297a0228f48cd3fc6a55de6316f31422f5d17",
   RScript.ReturnType.VALUE, Collections.emptyList());
```

## 底层Redis客户端

Redisson在底层采用了高性能异步非阻塞式Java客户端，它同时支持异步和同步两种通信模式。如果有哪些命令Redisson还没提供支持，也可以直接通过调用底层Redis客户端来实现。Redisson支持的命令在[Redis命令和Redisson对象匹配列表](https://github.com/redisson/redisson/wiki/11.-Redis命令和Redisson对象匹配列表)里做了详细对比参照。

```java
// 在使用多个客户端的情况下可以共享同一个EventLoopGroup
EventLoopGroup group = new NioEventLoopGroup();

RedisClientConfig config = new RedisClientConfig();
config.setAddress("redis://localhost:6379") // 或者用rediss://使用加密连接
      .setPassword("myPassword")
      .setDatabase(0)
      .setClientName("myClient")
      .setGroup(group);

RedisClient client = RedisClient.create(config);
RedisConnection conn = client.connect();
// 或
RFuture<RedisConnection> connFuture = client.connectAsync();

conn.sync(StringCodec.INSTANCE, RedisCommands.SET, "test", 0);
// 或
conn.async(StringCodec.INSTANCE, RedisCommands.GET, "test");

conn.close()
// 或
conn.closeAsync()

client.shutdown();
// 或
client.shutdownAsync();
```

# Redis命令和Redisson对象匹配列表

| Redis命令         | Redisson对象方法                                             |
| ----------------- | ------------------------------------------------------------ |
| AUTH              | Config.setPassword();                                        |
| APPEND            | RBinaryStream.getOutputStream().write()                      |
| BITCOUNT          | RBitSet.cardinality(), RBitSet.cardinalityAsync(), RBitSetReactive.cardinality() |
| BITOP             | RBitSet.or(), RBitSet.orAsync(), RBitSetReactive.or(); RBitSet.and(), RBitSet.andAsync(), RBitSetReactive.and(); RBitSet.not(); RBitSet.xor(), RBitSet.xorAsync(), RBitSetReactive.xor() |
| BITPOS            | RBitSet.length(), RBitSet.lengthAsync(), RBitSetReactive.length() |
| BLPOP             | RBlockingQueue.take(), RBlockingQueue.takeAsync(), RBlockingQueueReactive.take(); RBlockingQueue.poll(), RBlockingQueue.pollAsync(), RBlockingQueueReactive.poll(); RBlockingQueue.pollFromAny(), RBlockingQueue.pollFromAnyAsync(), RBlockingQueueReactive.pollFromAny(); |
| BRPOP             | RBlockingDeque.takeLast(), RBlockingDeque.takeLastAsync(), RBlockingDequeReactive.takeLast(); |
| BRPOPLPUSH        | RBlockingQueue.pollLastAndOfferFirstTo(), RBlockingQueue.pollLastAndOfferFirstToAsync(), RBlockingQueueReactive.pollLastAndOfferFirstTo(); |
| COPY              | RObject.copy, RObject.copyAsync, RObjectReactive.copy();     |
| CLIENT SETNAME    | Config.setClientName();                                      |
| CLUSTER INFO      | ClusterNode.info();                                          |
| CLUSTER KEYSLOT   | RKeys.getSlot(), RKeys.getSlotAsync(), RKeysReactive.getSlot(); |
| CLUSTER NODES     | Used in ClusterConnectionManager                             |
| DUMP              | RObject.dump(), RObject.dumpAsync(), RObjectReactive.dump(); |
| DBSIZE            | RKeys.count(), RKeys.countAsync(), RKeysReactive.count();    |
| DECR              | RAtomicLong.decrementAndGet(), RAtomicLong.decrementAndGetAsync(), RAtomicLongReactive.decrementAndGetAsync(); |
| DEL               | RObject.delete(), RObject.deleteAsync(), RObjectReactive.delete(); RKeys.delete(), RKeys.deleteAsync(); |
| STRLEN            | RBucket.size(), RBucket.sizeAsync(), RBucketReactive.size(); |
| EVAL              | RScript.eval(), RScript.evalAsync(), RScriptReactive.eval(); |
| CLIENT REPLY      | RBatch.executeSkipResult();                                  |
| EVALSHA           | RScript.evalSha(), RScript.evalShaAsync(), RScriptReactive.evalSha(); |
| EXEC              | RBatch.execute(), RBatch.executeAsync(), RBatchReactive.execute(); |
| EXISTS            | RObject.isExists(), RObject.isExistsAsync(), RObjectReactive.isExists(); |
| FLUSHALL          | RKeys.flushall(), RKeys.flushallAsync(), RKeysReactive.flushall(); |
| FLUSHDB           | RKeys.flushdb(), RKeys.flushdbAsync(), RKeysReactive.flushdb(); |
| GEOADD            | RGeo.add(), RGeo.addAsync(), RGeoReactive.add();             |
| GEODIST           | RGeo.dist(), RGeo.distAsync(), RGeoReactive.dist();          |
| GEOHASH           | RGeo.hash(), RGeo.hashAsync(), RGeoReactive.hash();          |
| GEOPOS            | RGeo.pos(), RGeo.posAsync(), RGeoReactive.pos();             |
| GEORADIUS         | RGeo.radius(), RGeo.radiusAsync(), RGeoReactive.radius(); RGeo.radiusWithDistance(), RGeo.radiusWithDistanceAsync(), RGeoReactive.radiusWithDistance(); RGeo.radiusWithPosition(), RGeo.radiusWithPositionAsync(), RGeoReactive.radiusWithPosition(); |
| GEORADIUSBYMEMBER | RGeo.radius(), RGeo.radiusAsync(), RGeoReactive.radius(); RGeo.radiusWithDistance(), RGeo.radiusWithDistanceAsync(), RGeoReactive.radiusWithDistance(); RGeo.radiusWithPosition(), RGeo.radiusWithPositionAsync(), RGeoReactive.radiusWithPosition(); |
| GET               | RBucket.get(), RBucket.getAsync(), RBucketReactive.get();    |
| GETBIT            | RBitSet.get(), RBitSet.getAsync(), RBitSetReactive.get();    |
| GETSET            | RBucket.getAndSet(), RBucket.getAndSetAsync(), RBucketReactive.getAndSet(); RAtomicLong.getAndSet(), RAtomicLong.getAndSetAsync(), RAtomicLongReactive.getAndSet(); RAtomicDouble.getAndSet(), RAtomicDouble.getAndSetAsync(), RAtomicDoubleReactive.getAndSet(); |
| HDEL              | RMap.fastRemove(), RMap.fastRemoveAsync(), RMapReactive.fastRemove(); |
| HEXISTS           | RMap.containsKey(), RMap.containsKeyAsync(), RMapReactive.containsKey(); |
| HGET              | RMap.get(), RMap.getAsync(), RMapReactive.get();             |
| HSTRLEN           | RMap.valueSize(), RMap.valueSizeAsync(), RMapReactive.valueSize(); |
| HGETALL           | RMap.readAllEntrySet(), RMap.readAllEntrySetAsync(), RMapReactive.readAllEntrySet(); |
| HINCRBY           | RMap.addAndGet(), RMap.addAndGetAsync(), RMapReactive.addAndGet(); |
| HINCRBYFLOAT      | RMap.addAndGet(), RMap.addAndGetAsync(), RMapReactive.addAndGet(); |
| HKEYS             | RMap.readAllKeySet(), RMap.readAllKeySetAsync(), RMapReactive.readAllKeySet(); |
| HLEN              | RMap.size(), RMap.sizeAsync(), RMapReactive.size();          |
| HMGET             | RMap.getAll(), RMap.getAllAsync(), RMapReactive.getAll();    |
| HMSET             | RMap.putAll(), RMap.putAllAsync(), RMapReactive.putAll();    |
| HSET              | RMap.put(), RMap.putAsync(), RMapReactive.put();             |
| HSETNX            | RMap.fastPutIfAbsent(), RMap.fastPutIfAbsentAsync, RMapReactive.fastPutIfAbsent(); |
| HVALS             | RMap.readAllValues(), RMap.readAllValuesAsync(), RMapReactive.readAllValues(); |
| INCR              | RAtomicLong.incrementAndGet(), RAtomicLong.incrementAndGetAsync(), RAtomicLongReactive.incrementAndGet(); |
| INCRBY            | RAtomicLong.addAndGet(), RAtomicLong.addAndGetAsync(), RAtomicLongReactive.addAndGet(); |
| KEYS              | RKeys.findKeysByPattern(), RKeys.findKeysByPatternAsync(), RKeysReactive.findKeysByPattern(); RedissonClient.findBuckets(); |
| LINDEX            | RList.get(), RList.getAsync(), RListReactive.get();          |
| LLEN              | RList.size(), RList.sizeAsync(), RListReactive.Size();       |
| LPOP              | RQueue.poll(), RQueue.pollAsync(), RQueueReactive.poll();    |
| LPUSH             | RDeque.addFirst(), RDeque.addFirstAsync(); RDequeReactive.addFirst(), RDeque.offerFirst(), RDeque.offerFirstAsync(), RDequeReactive.offerFirst(); |
| LRANGE            | RList.readAll(), RList.readAllAsync(), RListReactive.readAll(); |
| LREM              | RList.fastRemove(), RList.fastRemoveAsync(), RList.remove(), RList.removeAsync(), RListReactive.remove(); RDeque.removeFirstOccurrence(), RDeque.removeFirstOccurrenceAsync(), RDequeReactive.removeFirstOccurrence(); RDeque.removeLastOccurrence(), RDeque.removeLastOccurrenceAsync(), RDequeReactive.removeLastOccurrence(); |
| LSET              | RList.fastSet(), RList.fastSetAsync(), RListReactive.fastSet(); |
| LTRIM             | RList.trim(), RList.trimAsync(), RListReactive.trim();       |
| LINSERT           | RList.addBefore(), RList.addBeforeAsync(), RList.addAfter(), RList.addAfterAsync(), RListReactive.addBefore(), RListReactive.addAfter(); |
| MULTI             | RBatch.execute(), RBatch.executeAsync(), RBatchReactive.execute(); |
| MGET              | RedissonClient.loadBucketValues();                           |
| MIGRATE           | RObject.migrate(), RObject.migrateAsync();                   |
| MOVE              | RObject.move(), RObject.moveAsync();                         |
| MSET              | RedissonClient.saveBuckets();                                |
| PERSIST           | RExpirable.clearExpire(), RExpirable.clearExpireAsync(), RExpirableReactive.clearExpire(); |
| PEXPIRE           | RExpirable.expire(), RExpirable.expireAsync(), RExpirableReactive.expire(); |
| PEXPIREAT         | RExpirable.expireAt(), RExpirable.expireAtAsync(), RExpirableReactive.expireAt(); |
| PFADD             | RHyperLogLog.add(), RHyperLogLog.addAsync(), RHyperLogLogReactive.add(); RHyperLogLog.addAll(), RHyperLogLog.addAllAsync(), RHyperLogLogReactive.addAll(); |
| PFCOUNT           | RHyperLogLog.count(), RHyperLogLog.countAsync(), RHyperLogLogReactive.count(); RHyperLogLog.countWith(), RHyperLogLog.countWithAsync(), RHyperLogLogReactive.countWith(); |
| PFMERGE           | RHyperLogLog.mergeWith(), RHyperLogLog.mergeWithAsync(), RHyperLogLogReactive.mergeWith(); |
| PING              | Node.ping(); NodesGroup.pingAll();                           |
| PSUBSCRIBE        | RPatternTopic.addListener();                                 |
| PTTL              | RExpirable.remainTimeToLive(), RExpirable.remainTimeToLiveAsync(), RExpirableReactive.remainTimeToLive(); |
| PUBLISH           | RTopic.publish                                               |
| PUNSUBSCRIBE      | RPatternTopic.removeListener();                              |
| RANDOMKEY         | RKeys.randomKey(), RKeys.randomKeyAsync(), RKeysReactive.randomKey(); |
| RESTORE           | RObject.restore(), RObject.restoreAsync, RObjectReactive.restore(); |
| RENAME            | RObject.rename(), RObject.renameAsync(), RObjectReactive.rename(); |
| RENAMENX          | RObject.renamenx(), RObject.renamenxAsync(), RObjectReactive.renamenx(); |
| RPOP              | RDeque.pollLast(), RDeque.pollLastAsync(), RDequeReactive.pollLast(); RDeque.removeLast(), RDeque.removeLastAsync(), RDequeReactive.removeLast(); |
| RPOPLPUSH         | RDeque.pollLastAndOfferFirstTo(), RDeque.pollLastAndOfferFirstToAsync(); |
| RPUSH             | RList.add(), RList.addAsync(), RListReactive.add();          |
| SADD              | RSet.add(), RSet.addAsync(), RSetReactive.add();             |
| SCARD             | RSet.size(), RSet.sizeAsync(), RSetReactive.size();          |
| SCRIPT EXISTS     | RScript.scriptExists(), RScript.scriptExistsAsync(), RScriptReactive.scriptExists(); |
| SCRIPT FLUSH      | RScript.scriptFlush(), RScript.scriptFlushAsync(), RScriptReactive.scriptFlush(); |
| SCRIPT KILL       | RScript.scriptKill(), RScript.scriptKillAsync(), RScriptReactive.scriptKill(); |
| SCRIPT LOAD       | RScript.scriptLoad(), RScript.scriptLoadAsync(), RScriptReactive.scriptLoad(); |
| SDIFFSTORE        | RSet.diff(), RSet.diffAsync(), RSetReactive.diff();          |
| SELECT            | Config.setDatabase();                                        |
| SET               | RBucket.set(); RBucket.setAsync(); RBucketReactive.set();    |
| SETBIT            | RBitSet.set(); RBitSet.setAsync(); RBitSet.clear(); RBitSet.clearAsync(); |
| SETEX             | RBucket.set(); RBucket.setAsync(); RBucketReactive.set();    |
| SETNX             | RBucket.trySet(); RBucket.trySetAsync(); RBucketReactive.trySet(); |
| SISMEMBER         | RSet.contains(), RSet.containsAsync(), RSetReactive.contains(); |
| SINTERSTORE       | RSet.intersection(), RSet.intersectionAsync(), RSetReactive.intersection(); |
| SINTER            | RSet.readIntersection(), RSet.readIntersectionAsync(), RSetReactive.readIntersection(); |
| SMEMBERS          | RSet.readAll(), RSet.readAllAsync(), RSetReactive.readAll(); |
| SMOVE             | RSet.move(), RSet.moveAsync(), RSetReactive.move();          |
| SPOP              | RSet.removeRandom(), RSet.removeRandomAsync(), RSetReactive.removeRandom(); |
| SREM              | RSet.remove(), RSet.removeAsync(), RSetReactive.remove();    |
| SUBSCRIBE         | RTopic.addListener(), RTopicReactive.addListener();          |
| SUNION            | RSet.readUnion(), RSet.readUnionAsync(), RSetReactive.readUnion(); |
| SUNIONSTORE       | RSet.union(), RSet.unionAsync(), RSetReactive.union();       |
| TTL               | RExpirable.remainTimeToLive(), RExpirable.remainTimeToLiveAsync(), RExpirableReactive.remainTimeToLive(); |
| TYPE              | RKeys.getType();                                             |
| UNSUBSCRIBE       | RTopic.removeListener(), RTopicReactive.removeListener();    |
| WAIT              | RBatch.syncSlaves, RBatchReactive.syncSlaves();              |
| ZADD              | RScoredSortedSet.add(), RScoredSortedSet.addAsync(), RScoredSortedSetReactive.add(); |
| ZCARD             | RScoredSortedSet.size(), RScoredSortedSet.sizeAsync(), RScoredSortedSetReactive.size(); |
| ZCOUNT            | RScoredSortedSet.count(), RScoredSortedSet.countAsync();     |
| ZINCRBY           | RScoredSortedSet.addScore(), RScoredSortedSet.addScoreAsync(), RScoredSortedSetReactive.addScore(); |
| ZLEXCOUNT         | RLexSortedSet.lexCount(), RLexSortedSet.lexCountAsync(), RLexSortedSetReactive.lexCount(); RLexSortedSet.lexCountHead(), RLexSortedSet.lexCountHeadAsync(), RLexSortedSetReactive.lexCountHead(); RLexSortedSet.lexCountTail(), RLexSortedSet.lexCountTailAsync(), RLexSortedSetReactive.lexCountTail(); |
| ZRANGE            | RScoredSortedSet.valueRange(), RScoredSortedSet.valueRangeAsync(), RScoredSortedSetReactive.valueRange(); |
| ZREVRANGE         | RScoredSortedSet.valueRangeReversed(), RScoredSortedSet.valueRangeReversedAsync(), RScoredSortedSetReactive.valueRangeReversed(); |
| ZUNIONSTORE       | RScoredSortedSet.union(), RScoredSortedSet.unionAsync(), RScoredSortedSetReactive.union(); |
| ZINTERSTORE       | RScoredSortedSet.intersection(), RScoredSortedSet.intersectionAsync(), RScoredSortedSetReactive.intersection(); |
| ZRANGEBYLEX       | RLexSortedSet.lexRange(), RLexSortedSet.lexRangeAsync(), RLexSortedSetReactive.lexRange(); RLexSortedSet.lexRangeHead(), RLexSortedSet.lexRangeHeadAsync(), RLexSortedSetReactive.lexRangeHead(); RLexSortedSet.lexRangeTail(), RLexSortedSet.lexRangeTailAsync(), RLexSortedSetReactive.lexRangeTail(); |
| ZRANGEBYSCORE     | RScoredSortedSet.valueRange(), RScoredSortedSet.valueRangeAsync(), RScoredSortedSetReactive.valueRange(); RScoredSortedSet.entryRange(), RScoredSortedSet.entryRangeAsync(), RScoredSortedSetReactive.entryRange(); |
| TIME              | RedissonClient.getNodesGroup().getNode().time(), RedissonClient.getClusterNodesGroup().getNode().time(); |
| ZRANK             | RScoredSortedSet.rank(), RScoredSortedSet.rankAsync(), RScoredSortedSetReactive.rank(); |
| ZREM              | RScoredSortedSet.remove(), RScoredSortedSet.removeAsync(), RScoredSortedSetReactive.remove(); RScoredSortedSet.removeAll(), RScoredSortedSet.removeAllAsync(), RScoredSortedSetReactive.removeAll(); |
| ZREMRANGEBYLEX    | RLexSortedSet.removeRangeByLex(), RLexSortedSet.removeRangeByLexAsync(), RLexSortedSetReactive.removeRangeByLex(); RLexSortedSet.removeRangeHeadByLex(), RLexSortedSet.removeRangeHeadByLexAsync(), RLexSortedSetReactive.removeRangeHeadByLex(); RLexSortedSet.removeRangeTailByLex(), RLexSortedSet.removeRangeTailByLexAsync(), RLexSortedSetReactive.removeRangeTailByLex(); |
| ZREMRANGEBYLEX    | RScoredSortedSet.removeRangeByRank(), RScoredSortedSet.removeRangeByRankAsync(), RScoredSortedSetReactive.removeRangeByRank(); |
| ZREMRANGEBYSCORE  | RScoredSortedSet.removeRangeByScore(), RScoredSortedSet.removeRangeByScoreAsync(), RScoredSortedSetReactive.removeRangeByScore(); |
| ZREVRANGEBYSCORE  | RScoredSortedSet.entryRangeReversed(), RScoredSortedSet.entryRangeReversedAsync(), RScoredSortedSetReactive.entryRangeReversed(), RScoredSortedSet.valueRangeReversed(), RScoredSortedSet.valueRangeReversedAsync(), RScoredSortedSetReactive.valueRangeReversed(); |
| ZREVRANK          | RScoredSortedSet.revRank(), RScoredSortedSet.revRankAsync(), RScoredSortedSetReactive.revRank(); |
| ZSCORE            | RScoredSortedSet.getScore(), RScoredSortedSet.getScoreAsync(), RScoredSortedSetReactive.getScore(); |
| SCAN              | RKeys.getKeys(), RKeysReactive.getKeys();                    |
| SSCAN             | RSet.iterator(), RSetReactive.iterator();                    |
| HSCAN             | RMap.keySet().iterator(), RMap.values().iterator(), RMap.entrySet().iterator(), RMapReactive.keyIterator(), RMapReactive.valueIterator(), RMapReactive.entryIterator(); |
| ZSCAN             | RScoredSortedSet.iterator(), RScoredSortedSetReactive.iterator(); |

# 独立节点模式

Rui Gu edited this page on 12 Jan 2019 · [7 revisions](https://github.com/redisson/redisson/wiki/12.-独立节点模式/_history)

## 概述

Redisson Node指的是Redisson在分布式运算环境中作为独立节点运行的一种模式。Redisson Node的功能可以用来执行通过[分布式执行服务](https://github.com/redisson/redisson/wiki/9.-分布式服务#93-分布式执行服务executor-service)或[分布式调度执行服务](https://github.com/redisson/redisson/wiki/9.-分布式服务#94-分布式调度任务服务scheduler-service)发送的远程任务，也可以用来为[分布式远程服务](https://github.com/redisson/redisson/wiki/9.-分布式服务#91-分布式远程服务remote-service)提供远端服务。 所有这些功能全部包含在一个JAR包里，您可以从[这里](https://repository.sonatype.org/service/local/artifact/maven/redirect?r=central-proxy&g=org.redisson&a=redisson-all&v=LATEST&e=jar)下载

## 配置方法

### 配置参数

Redisson Node采用的是与Redisson框架同样的[配置方法](https://github.com/redisson/redisson/wiki/2.-配置方法)，并同时还增加了以下几个专用参数。值得注意的是`ExecutorService`使用的线程数量可以通过`threads`参数来设定。

##### mapReduceWorkers （MapReduce的工作者数量）

默认值：0 用来指定执行MapReduce任务的工作者的数量 `0 代表当前CPU核的数量`

##### executorServiceWorkers（执行服务的工作者数量）

默认值：null 用一个Map结构来指定某个服务的工作者数量，Map的Key是服务名称，用value指定数量。

##### redissonNodeInitializer（初始化监听器）

默认值：null

Redisson Node启动完成后调用的初始化监听器。

###  通过JSON和YAML配置文件配置独立节点

以下是JSON格式的配置文件范例，该范例是在集群模式配置方法基础上，增加了Redisson Node的配置参数。

```
{
   "clusterServersConfig":{
      "nodeAddresses":[
         "//127.0.0.1:7004",
         "//127.0.0.1:7001",
         "//127.0.0.1:7000"
      ],
   },
   "threads":0,

   "executorServiceThreads": 0,
   "executorServiceWorkers": {"myExecutor1":3, "myExecutor2":5},
   "redissonNodeInitializer": {"class":"org.mycompany.MyRedissonNodeInitializer"}
}
```

以下是YAML格式的配置文件范例，该范例是在集群模式配置方法基础上，增加了Redisson Node的配置参数。

```
---
clusterServersConfig:
  nodeAddresses:
  - "//127.0.0.1:7004"
  - "//127.0.0.1:7001"
  - "//127.0.0.1:7000"
  scanInterval: 1000
threads: 0

executorServiceThreads: 0
executorServiceWorkers:
  myService1: 123
  myService2: 421
redissonNodeInitializer: !<org.mycompany.MyRedissonNodeInitializer> {}
```

## 初始化监听器

Redisson Node提供了在启动完成后，执行`RedissonNodeInitializer`指定的初始化监听器的机制。这个机制可以用在启动完成时执行注册在类路径（classpath）中分布式远程服务的实现，或其他必要业务逻辑。比如，通知其他订阅者关于一个新节点上线的通知：

```java
public class MyRedissonNodeInitializer implements RedissonNodeInitializer {

    @Override
    public void onStartup(RedissonNode redissonNode) {
        RMap<String, Integer> map = redissonNode.getRedisson().getMap("myMap");
        // ...
        // 或
        redisson.getRemoteService("myRemoteService").register(MyRemoteService.class, new MyRemoteServiceImpl(...));
        // 或
        reidsson.getTopic("myNotificationTopic").publish("New node has joined. id:" + redissonNode.getId() + " remote-server:" + redissonNode.getRemoteAddress());
    }

}
```

## 嵌入式运行方法

Redisson Node也可以以嵌入式方式运行在其他应用当中。

```
// Redisson程序化配置代码
Config config = ...
// Redisson Node 程序化配置方法
RedissonNodeConfig nodeConfig = new RedissonNodeConfig(config);
Map<String, Integer> workers = new HashMap<String, Integer>();
workers.put("test", 1);
nodeConfig.setExecutorServiceWorkers(workers);

// 创建一个Redisson Node实例
RedissonNode node = RedissonNode.create(nodeConfig);
// 或者通过指定的Redisson实例创建Redisson Node实例
RedissonNode node = RedissonNode.create(nodeConfig, redisson);

node.start();

//...

node.shutdown();
```

## 命令行运行方法

1. [下载](https://repository.sonatype.org/service/local/artifact/maven/redirect?r=central-proxy&g=org.redisson&a=redisson-all&v=LATEST&e=jar)Redisson Node的JAR包。
2. 编写一个JSON或YAML格式的配置文件。
3. 通过以下方式之一运行Redisson Node： `java -jar redisson-all.jar config.json` 或 `java -jar redisson-all.jar config.yaml`

另外不要忘记添加`-Xmx`或`-Xms`之类的参数。

## Docker方式运行方法

#### 无现有Redis环境：

1. 首先运行Redis： `docker run -d --name redis-node redis`

2. 再运行Redisson Node： `docker run -d --network container:redis-node -e JAVA_OPTS="<java-opts>" -v <path-to-config>:/opt/redisson-node/redisson.conf redisson/redisson-node`

   `<path-to-config>` - Redisson Node的JSON或YAML配置文件路径 `<java-opts>` - JAVA虚拟机的运行参数

#### 有现有Redis环境：

1. 运行Redisson Node： `docker run -d -e JAVA_OPTS="<java-opts>" -v <path-to-config>:/opt/redisson-node/redisson.conf redisson/redisson-node`

   `<path-to-config>` - Redisson Node的JSON或YAML配置文件路径 `<java-opts>` - JAVA虚拟机的运行参数

# 工具

### 集群管理工具

Redisson集群管理工具提供了通过程序化的方式，像`redis-trib.rb`脚本一样方便地管理Redis集群的工具。

### 创建集群

以下范例展示了如何创建三主三从的Redis集群。

```
ClusterNodes clusterNodes = ClusterNodes.create()
.master("127.0.0.1:7000").withSlaves("127.0.0.1:7001", "127.0.0.1:7002")
.master("127.0.0.1:7003").withSlaves("127.0.0.1:7004")
.master("127.0.0.1:7005");
ClusterManagementTool.createCluster(clusterNodes);
```

主节点`127.0.0.1:7000`的从节点有`127.0.0.1:7001`和`127.0.0.1:7002`。

主节点`127.0.0.1:7003`的从节点是`127.0.0.1:7004`。

主节点`127.0.0.1:7005`没有从节点。

### 踢出节点

以下范例展示了如何将一个节点踢出集群。

```
ClusterManagementTool.removeNode("127.0.0.1:7000", "127.0.0.1:7002");
// 或
redisson.getClusterNodesGroup().removeNode("127.0.0.1:7002");
```

将从节点`127.0.0.1:7002`从其主节点`127.0.0.1:7000`里踢出。

### 数据槽迁移

以下范例展示了如何将数据槽在集群的主节点之间迁移。

```
ClusterManagementTool.moveSlots("127.0.0.1:7000", "127.0.0.1:7002", 23, 419, 4712, 8490);
// 或
redisson.getClusterNodesGroup().moveSlots("127.0.0.1:7000", "127.0.0.1:7002", 23, 419, 4712, 8490);
```

将番号为`23`，`419`，`4712`和`8490`的数据槽从`127.0.0.1:7002`节点迁移至`127.0.0.1:7000`节点。

以下范例展示了如何将一个范围的数据槽在集群的主节点之间迁移。

```
ClusterManagementTool.moveSlotsRange("127.0.0.1:7000", "127.0.0.1:7002", 51, 9811);
// 或
redisson.getClusterNodesGroup().moveSlotsRange("127.0.0.1:7000", "127.0.0.1:7002", 51, 9811);
```

将番号范围在[51, 9811]（含）之间的数据槽从`127.0.0.1:7002`节点移动到`127.0.0.1:7000`节点。

### 添加从节点

以下范例展示了如何向集群中添加从节点。

```
ClusterManagementTool.addSlaveNode("127.0.0.1:7000", "127.0.0.1:7003");
// 或
redisson.getClusterNodesGroup().addSlaveNode("127.0.0.1:7003");
```

将`127.0.0.1:7003`作为从节点添加至`127.0.0.1:7000`所在的集群里。

### 添加主节点

以下范例展示了如何向集群中添加主节点。

```
ClusterManagementTool.addMasterNode("127.0.0.1:7000", "127.0.0.1:7004");
// 或
redisson.getClusterNodesGroup().addMasterNode("127.0.0.1:7004");
```

将`127.0.0.1:7004`作为主节点添加至`127.0.0.1:7000`所在的集群里。 Adds master node `127.0.0.1:7004` to cluster where `127.0.0.1:7000` participate in

*该功能仅限于[Redisson PRO](https://redisson.pro/)版本。*

# 第三方框架整合(略)

## Spring Boot Starter

Integrates Redisson with Spring Boot library. Depends on [Spring Data Redis](https://github.com/redisson/redisson/tree/master/redisson-spring-data#spring-data-redis-integration) module.

Supports Spring Boot 1.3.x - 2.4.x

## Usage

###  Add `redisson-spring-boot-starter` dependency into your project:

Maven

```
     <dependency>
         <groupId>org.redisson</groupId>
         <artifactId>redisson-spring-boot-starter</artifactId>
         <version>3.15.6</version>
     </dependency>
```

Gradle

```
     compile 'org.redisson:redisson-spring-boot-starter:3.15.6'
```

Downgrade `redisson-spring-data` module if necessary to support required Spring Boot version:

| redisson-spring-data module name | Spring Boot version |
| -------------------------------- | ------------------- |
| redisson-spring-data-16          | 1.3.x               |
| redisson-spring-data-17          | 1.4.x               |
| redisson-spring-data-18          | 1.5.x               |
| redisson-spring-data-20          | 2.0.x               |
| redisson-spring-data-21          | 2.1.x               |
| redisson-spring-data-22          | 2.2.x               |
| redisson-spring-data-23          | 2.3.x               |
| redisson-spring-data-24          | 2.4.x               |

### Add settings into `application.settings` file

Using common spring boot settings:

```
spring:
  redis:
    database: 
    host:
    port:
    password:
    ssl: 
    timeout:
    cluster:
      nodes:
    sentinel:
      master:
      nodes:
```

Using Redisson settings:

```
spring:
  redis:
   redisson: 
      file: classpath:redisson.yaml
      config: |
        clusterServersConfig:
          idleConnectionTimeout: 10000
          connectTimeout: 10000
          timeout: 3000
          retryAttempts: 3
          retryInterval: 1500
          failedSlaveReconnectionInterval: 3000
          failedSlaveCheckInterval: 60000
          password: null
          subscriptionsPerConnection: 5
          clientName: null
          loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
          subscriptionConnectionMinimumIdleSize: 1
          subscriptionConnectionPoolSize: 50
          slaveConnectionMinimumIdleSize: 24
          slaveConnectionPoolSize: 64
          masterConnectionMinimumIdleSize: 24
          masterConnectionPoolSize: 64
          readMode: "SLAVE"
          subscriptionMode: "SLAVE"
          nodeAddresses:
          - "redis://127.0.0.1:7004"
          - "redis://127.0.0.1:7001"
          - "redis://127.0.0.1:7000"
          scanInterval: 1000
          pingConnectionInterval: 0
          keepAlive: false
          tcpNoDelay: false
        threads: 16
        nettyThreads: 32
        codec: !<org.redisson.codec.MarshallingCodec> {}
        transportMode: "NIO"
```

###  Available Spring Beans:

- `RedissonClient`
- `RedissonRxClient`
- `RedissonReactiveClient`
- `RedisTemplate`
- `ReactiveRedisTemplate`

Consider **[Redisson PRO](https://redisson.pro/)** version for **ultra-fast performance** and **support by SLA**.

