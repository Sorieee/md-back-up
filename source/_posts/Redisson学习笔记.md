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

### 程序化配置方法

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

### 文件方式配置

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

