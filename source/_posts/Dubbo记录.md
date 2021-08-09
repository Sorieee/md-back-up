# 语雀

https://item.jd.com/12545055.html

https://www.yuque.com/apache-dubbo/dubbo3/pyrcyr

https://www.yuque.com/apache-dubbo

# XSD教程

https://www.w3school.com.cn/schema/schema_intro.asp1

# 架构图

![](https://pic.imgdb.cn/item/60dd67ed5132923bf88c418f.jpg)

![](https://pic.imgdb.cn/item/60e084ed5132923bf88eb35b.jpg)

![](https://pic.imgdb.cn/item/60e086675132923bf89cada0.jpg)

**服务的暴露过程**

​	首先，服务器端（服务提供者）在框架启动时，会初始化服务实例，通过Proxy组件调
用具体协议（Protocol ）,把服务端要暴露的接口封装成Invoker （真实类型是
AbstractProxylnvoker）,然后转换成Exporter,这个时候框架会打开服务端口等并记录服务实例
到内存中，最后通过Registry把服务元数据注册到注册中心。

* Proxy组件：我们知道，Dubbo中只需要引用一个接口就可以调用远程的服务，并且
  只需要像调用本地方法一样调用即可。其实是Dubbo框架为我们生成了代理类，调用
  的方法其实是Proxy组件生成的代理方法，会自动发起远程/本地调用，并返回结果,
  整个过程对用户完全透明。
* Protocol：顾名思义，协议就是对数据格式的一种约定。它可以把我们对接口的配置,根据不同的协议转换成不同的Invoker对象。例如：用DubboProtocol可以把XML文件中一个远程接口的配置转换成一个Dubbolnvokero
* Exporter：用于暴露到注册中心的对象，它的内部属性持有了Invoker对象，我们可以认为它在Invoker上包了一层。
* Registry：把Exporter注册到注册中心。

![](https://pic.imgdb.cn/item/60e087385132923bf8a480bb.jpg)

​	首先，调用过程也是从一个Proxy开始的，Proxy持有了一个Invoker对象。然后触发invoke调用。在invoke调用过程中，需要使用Cluster,Cluster负责容错，如调用失败的重试。

​	Cluster在调用之前会通过Directory获取所有可以调用的远程服务Invoker列表（一个接口可能有多个节点提供服务）。由于可以调用的远程服务有很多，此时如果用户配置了路由规则（如指定某些方法只能调用某个节点），那么还会根据路由规则将Invoker列表过滤一遍。

​	然后，存活下来的Invoker可能还会有很多，此时要调用哪一个呢？于是会继续通过LoadBalance方法做负载均衡，最终选出一个可以调用的Invokero这个Invoker在调用之前又会经过一个过滤器链，这个过滤器链通常是处理上下文、限流、计数等。

​	接着，会使用Client做数据传输，如我们常见的NettyClient等。传输之前肯定要做一些私有协议的构造，此时就会用到Codec接口。构造完成后，就对数据包做序列化（Serialization）,然后传输到服务提供者端。服务提供者收到数据包，也会使用Codec处理协议头及一些半包、粘包等。处理完成后再对完整的数据报文做反序列化处理。

​	随后，这个Request会被分配到线程池（ThreadPool）中进行处理oServer会处理这些Request,根据请求查找对应的Exporter（它内部持有了Invoker）0Invoker是被用装饰器模式一层一层套了非常多Filter的，因此在调用最终的实现类之前，又会经过一个服务提供者端的过滤器链。

​	随后，这个Request会被分配到线程池（ThreadPool）中进行处理oServer会处理这些Request,根据请求查找对应的Exporter（它内部持有了Invoker）0Invoker是被用装饰器模式一层一层套了非常多Filter的，因此在调用最终的实现类之前，又会经过一个服务提供者端的过滤器链。

​	最终，我们得到了具体接口的真实实现并调用，再原路把结果返回。

![](https://pic.imgdb.cn/item/60e088655132923bf8b044e6.jpg)

# 注册中心

## 注册中心概述

* 动态加入。一个服务提供者通过注册中心可以动态地把自己暴露给其他消费者，无须
  消费者逐个去更新配置文件。
* 动态发现。一个消费者可以动态地感知新的配置、路由规则和新的服务提供者，无须
  重启服务使之生效。
* 动态调整。注册中心支持参数的动态调整，新参数自动更新到所有相关服务节点。
* 动态调整。注册中心支持参数的动态调整，新参数自动更新到所有相关服务节点。



​	Dubbo的注册中心源码在模块dubbo-registry中，里面包含了五个子模块，如表3-1所示。

![](https://pic.imgdb.cn/item/60e509815132923bf84aaff5.jpg)

​	其中ZooKeeper是官方推荐的注册中心，在生产环境中有过实际使用，具体的实现在Dubbo
源码的dubbo-registry-zookeeper模块中。阿里内部并没有使用Redis作为注册中心，Redis
注册中心并没有经过长时间运行的可靠性验证，其稳定性依赖于Redis本身。Simple注册中心
是一个简单的基于内存的注册中心实现，它本身就是一个标准的RPC服务，不支持集群，也可能出现单点故障。Multicast模式则不需要启动任何注册中心，只要通过广播地址，就可以互相发现。服务提供者启动时，会广播自己的地址。消费者启动时，会广播订阅请求，服务提供者收到订阅请求，会根据配置广播或单播给订阅者。不建议在生产环境使用。



​	注册中心的总体流程比较简单，Dubbo官方也有比较详细的说明，总体流程如图3-1所示。

* 服务提供者启动时，会向注册中心写入自己的元数据信息，同时会订阅配置元数据信息。
* 消费者启动时，也会向注册中心写入自己的元数据信息，并订阅服务提供者、路由和
  配置元数据信息。
* 服务治理中心(dubbo-admin)启动时，会同时订阅所有消费者、服务提供者、路由和
  配置元数据信息。
* 当有服务提供者离开或有新的服务提供者加入时，注册中心服务提供者目录会发生变
  化，变化信息会动态通知给消费者、服务治理中心。
* 当消费方发起服务调用时,会异步将调用、统计信息等上报给监控中心（dubbo-monitor・simple）。

![](https://pic.imgdb.cn/item/60e50a465132923bf84e97ae.jpg)

### ZooKeeper 原理概述

​	ZooKeeper是树形结构的注册中心，每个节点的类型分为持久节点、持久顺序节点、临时
节点和临时顺序节点。

* 持久节点：服务注册后保证节点不会丢失，注册中心重启也会存在。
* 持久顺序节点：在持久节点特性的基础上增加了节点先后顺序的能力。
* 临时节点：服务注册后连接丢失或session超时，注册的节点会自动被移除。
* 临时顺序节点：在临时节点特性的基础上增加了节点先后顺序的能力。



​	Dubbo使用ZooKeeper作为注册中心时，只会创建持久节点和临时节点两种，对创建的顺
序并没有要求。

```
+ /dubbo
+-- service
+-- providers
+-- consumers
+-- routers
+-- configurators
```

​	树形结构的关系:

​	(1) 树的根节点是注册中心分组，下面有多个服务接口，分组值来自用户配置
`<dubbo:registry>`中的 group 属性，默认是/dubbo。

​	(2) 服务接口下包含4类子目录，分别是providers、consumers、 routers、configurators
这个路径是持久节点。

​	(3) 服务提供者目录(/dubbo/service/providers)下面包含的接口有多个服务者元数据信息。

​	(4) 服务消费者目录(/dubbo/service/consumers)下面包含的接口有多个消费者元数据信息。

​	(5) 路由配置目录(/dubbo/service/providers)下面包含有多个用于消费者路由策略URL元数据信息。

​	(6) 动态配置目录(/dubbo/service/configurations)下面包含多个用于服务者动态配置URL元数据信息。

![](https://pic.imgdb.cn/item/60e50d0f5132923bf85d65e2.jpg)

​	在Dubbo框架启动时，会根据用户配置的服务，在注册中心中创建4个目录，在providers
和consumers目录中分别存储服务提供方、消费方元数据信息，主要包括IP、端口、权重和应用名等数据。

​	在Dubbo框架进行服务调用时，用户可以通过服务治理平台(dubbo-admin)下发路由配置。如果要在运行时改变服务参数，则用户可以通过服务治理平台(dubbo-admin)下发动态配置。服务器端会通过订阅机制收到属性变更，并重新更新已经暴露的服务。

![](https://pic.imgdb.cn/item/60e50d915132923bf8602721.jpg)

​	服务元数据中的所有参数都是以键值对形式存储的。以服务元数据为例：
dubbo://192.168.0.1.20880/com.alibaba.demo.Service?category=provider&name=demo-provider&..服务元数据中包含2个键值对，第1个key为category, key关联的值为provider。

```xml
<beans>
<!--适用于ZooKeeper —个集群有多个节点，多个IP和端口用逗号分隔-->
<dubbo:registry protocol="zookeeper" address="ip:port>ip:port" />
<!--适用于ZooKeeper多个集群有多个节点，多个IP和端口用竖线分隔-->
<dubbo:registry protocol="zookeeper" address="ip:port|ip:port" />
</beans>
```

### Redis原理概述

​	Redis注册中心也沿用了 Dubbo抽象的Root、Service> Type> URL四层结构。但是由于Redis属于NoSQL数据库，数据都是以键值对的形式保存的，并不能像ZooKeeper-样直接实现树形目录结构。因此，Redis使用了 key/Map结构实现了这个需求，Root、Service、Type组合成Redis的keyo Redis的value是一个Map结构，URL作为Map的key,超时时间作为Map的value,如图3-3所示。

![](https://pic.imgdb.cn/item/60e510125132923bf86dfb35.jpg)

​	数据结构的组装逻辑在 org.apache.dubbo. registry. redis. RedisRegistry#doRegister
(URL url)方法中，如代码清单3-1所示，无关代码已经省略。

​	![](https://pic.imgdb.cn/item/60e5104c5132923bf86f396f.jpg)

## 订阅/发布

​	当一个已有服务提供者节点下线，或者一个新的服务提供者节点加入微服务环境时，订阅对应接口的消费者和服务治理中心都能及时收到注册中心的通知，并更新本地的配置信息。如此一来，后续的服务调用就能避免调用已经下线的节点，或者能调用到新的节点。整个过程都是自动完成的，不需要人工参与。

### ZooKeeper 的实现

**1.发布的实现**

​	服务提供者和消费者都需要把自己注册到注册中心。服务提供者的注册是为了让消费者感知服务的存在，从而发起远程调用；也让服务治理中心感知有新的服务提供者上线。消费
者的发布是为了让服务治理中心可以发现自己。ZooKeeper发布代码非常简单，只是调用了
ZooKeeper的客户端库在注册中心上创建一个目录，如代码清单3-2所示。

**代码清单3-2 zkClient创建目录源码**

```java
zkClient.create(toUrlPath(url), 
           		url.getParameter(Constants.DYNAMIJKEY, true));
```

​	取消发布也很简单，只是把ZooKeeper注册中心上对应的路径删除，如代码清单3-3所示。
**代码清单3-3 zkClient删除路径源码**

```java
zkClient.delete(toUrlPath(url));
```

**2.订阅的实现**

​	订阅通常有pull和push两种方式，一种是客户端定时轮询注册中心拉取配置，另一种是注
册中心主动推送数据给客户端。这两种方式各有利弊，目前Dubbo采用的是第一次启动拉取方式，后续接收事件重新拉取数据。

​	在服务暴露时，服务端会订阅configurators用于监听动态配置，在消费端启动时，消费端会订阅providers、routers和configurators这三个目录，分别对应服务提供者、路由和动态配置变更通知。

> **Dubbo中有哪些ZooKeeper客户端实现？**
>
> ​	无论服务提供者还是消费者，或者是服务治理中心，任何一个节点连接到ZooKeeper注册中心都需要使用一个客户端，Dubbo在dubbo-remoting-zookeeper模块中实现了ZooKeeper客户端的统一封装，定义了统一的Client API,并用两种不同的ZooKeeper开源客户端库实现了这个接口：
>
> * Apache Curator；
> * zkClient 
>
> 用户可以在＜dubbo: registry＞的client属性中设置curator、zkclient来使用不同的客户端实现库，如果不设置则默认使用Curator作为实现。



​	ZooKeeper注册中心采用的是“事件通知” + “客户端拉取”的方式，客户端在第一次连接
上注册中心时，会获取对应目录下全量的数据。并在订阅的节点上注册一个watcher,客户端与注册中心之间保持TCP长连接，后续每个节点有任何数据变化的时候，注册中心会根watcher的回调主动通知客户端（事件通知），客户端接到通知后，会把对应节点下的全量数据都拉取过来（客户端拉取），这一点在`NotifyListener#notify（List<URL> urls）`接口上就有约束的注释说明。全量拉取有一个局限，当微服务节点较多时会对网络造成很大的压力。

​	ZooKeeper的每个节点都有一个版本号，当某个节点的数据发生变化（即事务操作）时，
该节点对应的版本号就会发生变化，并触发watcher事件，推送数据给订阅方。版本号强调的是变更次数，即使该节点的值没有变化，只要有更新操作，依然会使版本号变化。

> **什么操作会被认为是事务操作？**
>
> ​	客户端任何新增、删除、修改、会话创建和失效操作,都会被认为是事务操作，会由ZooKeeper集群中的leader执行。即使客户端连接的是非leader节点,请求也会被转发给leader执行，以此来保证所有事物操作的全局时序性。由于每个节点都有一个版本号，因此可以通过CAS操作比较版本号来保证该节点数据操作的原子性。

​	客户端第一次连上注册中心，订阅时会获取全量的数据，后续则通过监听器事件进行更新。服务治理中心会处理所有service层的订阅，service被设置成特殊值*。此外，服务治理中心除了订阅当前节点，还会订阅这个节点下的所有子节点，核心代码来自ZookeeperRegistry,如代码清单3-4所示。

![](https://pic.imgdb.cn/item/60e514eb5132923bf888b4d5.jpg)

![](https://pic.imgdb.cn/item/60e515095132923bf88952f5.jpg)

​	从代码清单3-4可以得知，此处主要支持Dubbo服务治理平台(dubbo-admin),平台在启动时会订阅全量接口，它会感知每个服务的状态。

​	接下来，我们看一下普通消费者的订阅逻辑。首先根据URL的类别得到一组需要订阅的路径。如果类别是*，则会订阅四种类型的路径(providers、routers、consumers、configurators),否则只订阅providers路径，如代码清单3-5所示。

![](https://pic.imgdb.cn/item/60e516f05132923bf89341c9.jpg)

![](https://pic.imgdb.cn/item/60e517025132923bf8939adb.jpg)

​	注意，此处会根据URL中的category属性值获取具体的类别：providers、routers、
consumers、configurators,然后拉取直接子节点的数据进行通知(notify)。如果是providers
类别的数据，则订阅方会更新本地Directory管理的Invoker服务列表；如果是routers分类，则订阅方会更新本地路由规则列表；如果是configuators类别，则订阅方会更新或覆盖本地动态参数列表。

### Redis 的实现

#### **1 .总体流程**

​	使用Redis作为注册中心，其订阅发布实现方式与ZooKeeper不同。我们在Redis注册中心
的数据结构中已经了解到，Redis订阅发布使用的是过期机制和publish/subscribe通道。服务提供者发布服务，首先会在Redis中创建一个key,然后在通道中发布一条register事件消息。但服务的key写入Redis后，发布者需要周期性地刷新key过期时间，在RedisRegistry构造方法中会启动一个expireExecutor定时调度线程池，不断调用deferExpired()方法去延续key的超时时间。如果服务提供者服务宕机，没有续期，则key会因为超时而被Redis删除，服务也就会被认定为下线，如代码清单3.6所示。

**代码清单3-6 Redis续期key**

![](https://pic.imgdb.cn/item/60e518e95132923bf89d6946.jpg)

​	订阅方首次连接上注册中心，会获取全量数据并缓存在本地内存中。后续的服务列表变化
则通过publish/subscribe通道广播，当有服务提供者主动下线的时候，会在通道中广播一条
unregister事件消息，订阅方收到后则从注册中心拉取数据，更新本地缓存的服务列表。新服务提供者上线也是通过通道事件触发更新的。

​	但是，Redis的key超时是不会有动态消息推送的，如果服务提供者宕机而不是主动下线，
则造成没有广播unregister事件消息，订阅方是如何知道服务的发布方已经下线了呢？另外，Redis的publish/subscribe通道并不是消息可靠的，如果Dubbo注册中心使用了 failover的集群容错模式，并且消费者订阅了从节点，但是主节点并没有完成数据同步给从节点就宕机，后续订阅方要如何知道服务发布方己经下线呢？

​	如果使用Redis作为服务注册中心，会依赖于服务治理中心。如果服务治理中心定时调度，
则还会触发清理逻辑：获取Redis上所有的key并进行遍历，如果发现key已经超时，则删除Redis上对应的key。清除完后，还会在通道中发起对应key的unregister事件，其他消费者监听到取消注册事件后会删除本地对应服务的数据，从而保证数据的最终一致，如代码清单3.7所示。

![](https://pic.imgdb.cn/item/60e51a3a5132923bf8a430b1.jpg)

由上面的机制可以得出整个Redis注册中心的工作流程，如图3-4所示。

![](https://pic.imgdb.cn/item/60e51a6c5132923bf8a53731.jpg)

​	Redis客户端初始化的时候，需要先初始化Redis的连接池jedisPools,此时如果配置注册
中心的集群模式为`<dubbo:registry cluster=,,replicate"/>`，则服务提供者在发布服务的时候,需要同时向Redis集群中所有的节点都写入，是多写的方式。但读取还是从一个节点中读取。在这种模式下，Redis集群可以不配置数据同步，一致性由客户端的多写来保证。

​	如果设置为failover或不设置，则只会读取和写入任意一个Redis节点，失败的话再尝试
下一个Redis节点。这种模式需要Redis自行配置数据同步。

​	另外，在初始化阶段，还会初始化一个定时调度线程池expireExecutor,它主要的任务是延长key的过期时间和删除过期的keyo线程池调度的时间间隔是超时时间的一半。

#### **2.发布的实现**

​	服务提供者和消费者都会使用注册功能，Redis注册部分的关键源码如代码清单3-8所示。

**代码清单3-8 Redis注册代码**

![](https://pic.imgdb.cn/item/60e51c485132923bf8aee39d.jpg)

#### **3.订阅的实现**

​	服务消费者、服务提供者和服务治理中心都会使用注册中心的订阅功能。在订阅时，如果
是首次订阅，则会先创建一个Notifier内部类，这是一个线程类，在启动时会异步进行通道的订阅。在启动Notifier线程的同时，主线程会继续往下执行，全量拉一次注册中心上所有的服务信息。后续注册中心上的信息变更则通过Notifier线程订阅的通道推送事件来实现。下面是Notifier线程中通道订阅的逻辑，如代码清单3-9所示。

![](https://pic.imgdb.cn/item/60e51d9d5132923bf8b591e8.jpg)

![](https://pic.imgdb.cn/item/60e51dbc5132923bf8b6305d.jpg)

## 缓存机制

​	Dubbo的注册中心实现了通用的缓存机制，在抽象类AbstractRegistry中实现。AbstractRegistry类结构关系如图3-5所示。

![](https://pic.imgdb.cn/item/60e51ee45132923bf8bbffc2.jpg)

​	消费者或服务治理中心获取注册信息后会做本地缓存。内存中会有一份，保存在Properties对象里，磁盘上也会持久化一份文件，通过file对象引用。在AbstractRegistry抽象类中有如下定义,如代码清单3-10所示。

![](https://pic.imgdb.cn/item/60e51f125132923bf8bce8ae.jpg)

​	内存中的缓存notified是ConcurrentHashMap里面又嵌套了一个Map,外层Map的key
是消费者的 URL,内层 Map 的 key 是分类，包含 providers> consumers> routes> configurators四种。value则是对应的服务列表，对于没有服务提供者提供服务的URL,它会以特殊的empty://前缀开头。

### 缓存的加载

​	在服务初始化的时候，AbstractRegistry构造函数里会从本地磁盘文件中把持久化的注册数据读到Properties对象里，并加载到内存缓存中，如代码清单3-11所示。

![](https://pic.imgdb.cn/item/60e520ad5132923bf8c548dc.jpg)

​	Properties保存了所有服务提供者的URL,使用URL#serviceKey()作为key,提供者列表、
路由规则列表、配置规则列表等作为value。由于value是列表，当存在多个的时候使用空格隔开。还有一个特殊的key.registies,保存所有的注册中心的地址。如果应用在启动过程中，注册中心无法连接或宕机，则Dubbo框架会自动通过本地缓存加载Invokerso

### 缓存的保存与更新

​	缓存的保存有同步和异步两种方式。异步会使用线程池异步保存，如果线程在执行过程中出现异常，则会再次调用线程池不断重试，如代码清单3.12所示。

![](https://pic.imgdb.cn/item/60e5218e5132923bf8c9b457.jpg)

​	AbstractRegistry#notify方法中封装了更新内存缓存和更新文件缓存的逻辑。当客户端第一次订阅获取全量数据，或者后续由于订阅得到新数据时，都会调用该方法进行保存。

## 重试机制

​	由图 3-5 我们可以得知 com.alibaba.dubbo.registry.support.FailbackRegistry 继承了
AbstractRegistry,并在此基础上增加了失败重试机制作为抽象能力。ZookeeperRegistry和
RedisRegistry继承该抽象方法后，直接使用即可。

​	FailbackRegistry抽象类中定义了一个ScheduledExecutorService,每经过固定间隔(默认为5秒)调用FailbackRegistry#retry()方法。另外，该抽象类中还有五个比较重要的集合，如表3-3所示。

![](https://pic.imgdb.cn/item/60e522625132923bf8cdd684.jpg)

​	在定时器中调用retry方法的时候，会把这五个集合分别遍历和重试，重试成功则从集合中移除。FailbackRegistry实现了 subscribe> unsubscribe等通用方法，里面调用了未实现模板方法，会由子类实现。通用方法会调用这些模板方法，如果捕获到异常，则会把URL添加到对应的重试集合中，以供定时器去重试。

## 设计模式

​	Dubbo注册中心拥有良好的扩展性，用户可以在其基础上，快速开发出符合自己业务需求
的注册中心。这种扩展性和Dubbo中使用的设计模式密不可分，本节将介绍注册中心模块使用的设计模式。学习完本节后，能降低读者对注册中心源码阅读的门槛。

### 模板模式

​	![](https://pic.imgdb.cn/item/60e523275132923bf8d1a03f.jpg)

​	AbstractRegistry实现了 Registry接口中的注册、订阅、查询、通知等方法，还实现了磁盘文件持久化注册信息这一通用方法。但是注册、订阅、查询、通知等方法只是简单地URL加入对应的集合，没有具体的注册或订阅逻辑。

​	FailbackRegistry又继承了 AbstractRegistry,重写了父类的注册、订阅、查询和通知等方法，并且添加了重试机制。此外，还添加了四个未实现的抽象模板方法，如代码清单3-13所示。

**代码清单3-13未实现的抽象模板方法**

```java
protected abstract void doRegister(URL url);
protected abstract void dollnregister(URL url);
protected abstract void doSubscribe(URL url. NotifyListener listener);
protected abstract void doUnsubscribe(URL url. NotifyListener listener);
```

​	以订阅为例，FailbackRegistry重写了 subscribe方法，但只实现了订阅的大体逻辑及异常处理等通用性的东西。具体如何订阅，交给继承的子类实现。这就是模板模式的具体实现，如代码清单3.14所示。

![](https://pic.imgdb.cn/item/60e526945132923bf8e2044b.jpg)

### 工厂模式

​	所有的注册中心实现，都是通过对应的工厂创建的。工厂类之间的关系如图3.7所示。

![](https://pic.imgdb.cn/item/60e526fb5132923bf8e3c629.jpg)

​	AbstractRegistryFactory 实现了 RegistryFactory 接口的 getRegistry(URL url)方法，是一个通用实现，主要完成了加锁，以及调用抽象模板方法createRegistry(URL url)创建具体实现等操作，并缓存在内存中。抽象模板方法会由具体子类继承并实现，如代码清单3-15所示。

![](https://pic.imgdb.cn/item/60e53be55132923bf8613904.jpg)

​	虽然每种注册中心都有自己具体的工厂类，但是在什么地方判断，应该调用哪个工厂类实现呢？代码中并没有看到显式的判断。答案就在RegistryFactory接口中，该接口里有一个Registry getRegistry(URL url)方法，该方法上有@Adaptive({"protocol"))注解，如代码清单3-16所示。

![](https://pic.imgdb.cn/item/60e53c605132923bf864a44e.jpg)

​	了解AOP的读者就会很容易理解，这个注解会自动生成代码实现一些逻辑，它的value参数会从URL中获取protocol键的值，并根据获取的值来调用不同的工厂类。例如，当url.protocol = redis时，获得RedisRegistryFactory实现类。具体Adaptive注解的实现原会在第4章Dubbo加载机制中讲解。

## 小结

略

# Dubbo 扩展点加载机制

## 加载机制概述

​	Dubbo良好的扩展性与两个方面是密不可分的，一是整个框架中针对不同的场景，恰到好处地使用了各种设计模式，二就是本章要介绍的加载机制。基于Dubbo SPI加载机制，让整个框架的接口和具体实现完全解耦，从而奠定了整个框架良好可扩展性的基础。

​	Dubbo SPI没有直接使用J ava SPL而是在它的思想上又做了一定的改进，形成了一套自己的配置规范和特性。同时，Dubbo SPI又兼容Java SPL服务在启动的时候，Dubbo就会查这些扩展点的所有实现，Dubbo具体的启动流程和实现原理会在第5章Dubbo启停原理解析讲解。本章聚焦扩展机制的实现原理。

### Java SPI

​	在讲解Dubbo SPI之前，我们先了解一下Java SPI是怎么使用的。SPI的全称是Service Provider Interface,起初是提供给厂商做插件开发的。

​	Java SPI使用了策略模式，一个接口多种实现。我们只声明接口，具体的实现并不在程序中直接确定，而是由程序之外的配置掌控，用于具体实现的装配。具体步骤如下：

​	(1) 定义一个接口及对应的方法。

​	(2) 编写该接口的一个实现类。

​	(3) 在META-INF/services/目录下，创建一个以接口全路径命名的文件，如com.test.spi.PrintService。

​	(4) 文件内容为具体实现类的全路径名，如果有多个，则用分行符分隔。

​	(5) 在代码中通过java.util.ServiceLoader来加载具体的实现类。

![](https://pic.imgdb.cn/item/60e540625132923bf88125dc.jpg)

![](https://pic.imgdb.cn/item/60e541135132923bf8862eba.jpg)

### 扩展点加载机制的改进

​	与Java SPI相比，Dubbo SPI做了一定的改进和优化，官方文档中有这么一段：

> 1. JDK标准的SPI会一次性实例化扩展点所有实现，如果有扩展实现则初始化很耗时，如果没用上也加载，则浪费资源。
> 2. 如果扩展加载失败，则连扩展的名称都蕤取不到了。比如JDK标准的ScriptEngine,通过getName ()获取脚本类型的名称，如果RubyScriptEngine因为所依赖的jruby.jar不存在，导致RubyScriptEngine类加载失败，这个失败原因被“吃掉” 了和Ruby对应不起来，当用户执行Ruby脚本时，会报不支持Ruby,而不是真正失败的原因。
> 3. 增加了对扩展IoC和AOP的支持，一个扩展可以直接setter注入其他扩展。在Java SPI的使用示例章节(代码清单4-1 )中已经看到，java.util.ServiceLoader会一次把Printservice接口下的所有实现类全部初始化，用户直接调用即可oDubbo SPI只是加载配置文件中的类，并分成不同的种类缓存在内存中，而不会立即全部初始化，在性能上有更好的表现。具体的实现原理会在后面讲解，此处演示一个使用示例。我们把代码清单4-1中的Printservice改造成Dubbo SPI的形式，如代码清单4-2所示。

![](https://pic.imgdb.cn/item/60e542425132923bf88e8bd0.jpg)

​	Java SPI加载失败，可能会因为各种原因导致异常信息被“吞掉”，导致开发人员问题追踪比较困难。Dubbo SPI在扩展加载失败的时候会先抛出真实异常并打印日志。扩展点在被动加载的时候，即使有部分扩展加载失败也不会影响其他扩展点和整个框架的使用。

​	Dubbo SPI自己实现了 IoC和AOP机制。一个扩展点可以通过setter方法直接注入其他扩展的方法，T injectExtension(T instance)方法实现了这个功能，后面会专门讲解。另外Dubbo支持包装扩展类，推荐把通用的抽象逻辑放到包装类中，用于实现扩展点的AOP特性。举个例子，我们可以看到ProtocolFilterWrapper包装扩展了 DubboProtocol类，一些通用的判断逻辑全部放在了 ProtocolFilterWrapper 类的 export方法中，但最终会调用 DubboProtocol#export方法。这和Spring的动态代理思想一样，在被代理类的前后插入自己的逻辑进行增强，最终调用被代理类。下面是ProtocolFilterWrapper#export方法，如代码清单4-3所示。

![](https://pic.imgdb.cn/item/60e543115132923bf89442ce.jpg)

### 扩展点的配置规范

​	Dubbo SPI和Java SPI类似，需要在META-INF/dubbo/下放置对应的SPI配置文件，文件名称需要命名为接口的全路径名。配置文件的内容为key=扩展点实现类全路径名，如果有多个实现类则使用换行符分隔。其中，key会作为DubboSPI注解中的传入参数。另外，Dubbo SPI还兼容了 Java SPI的配置路径和内容配置方式。在Dubbo启动的时候，会默认扫这三个目录下的配置文件：META-INF/services/、META-INF/dubbo/、META-INF/dubbo/internal/,如表 4-1 所示。

![](https://pic.imgdb.cn/item/60e543ac5132923bf8987dbc.jpg)

### 扩展点的分类与缓存

​	Dubbo SPI可以分为Class缓存、实例缓存。这两种缓存又能根据扩展类的种类分为普通扩展类、包装扩展类（Wrapper类）、自适应扩展类（Adaptive类）等。

* Class缓存：Dubbo SPI获取扩展类时，会先从缓存中读取。如果缓存中不存在，则加载配置文件，根据配置把Class缓存到内存中，并不会直接全部初始化。
* 实例缓存：基于性能考虑，Dubbo框架中不仅缓存Class,也会缓存Class实例化后的对象。每次获取的时候，会先从缓存中读取，如果缓存中读不到，则重新加载并缓存起来。这也是为什么Dubbo SPI相对Java SPI性能上有优势的原因，因为Dubbo SPI缓存的Class并不会全部实例化，而是按需实例化并缓存，因此性能更好。

被缓存的Class和对象实例可以根据不同的特性分为不同的类别：

* 普通扩展类。最基础的，配置在SPI配置文件中的扩展类实现。
* 包装扩展类。这种Wrapper类没有具体的实现，只是做了通用逻辑的抽象，并且需要在构造方法中传入一个具体的扩展接口的实现。属于Dubbo的自动包装特性，该特性会在4.1.5节中详细介绍。
* 自适应扩展类。一个扩展接口会有多种实现类，具体使用哪个实现类可以不写死在配置或代码中，在运行时，通过传入URL中的某些参数动态来确定。这属于扩展点的自适应特性，使用的@Adaptive注解也会在4.1.5节中详细介绍。
* 其他缓存，如扩展类加载器缓存、扩展名缓存等。



![](https://pic.imgdb.cn/item/60e5466d5132923bf8ac05a7.jpg)

![](https://pic.imgdb.cn/item/60e546a95132923bf8adbab6.jpg)

### 扩展点的特性

​	从Dubbo官方文档中可以知道，扩展类一共包含四种特性：自动包装、自动加载、自适应和自动激活。

**1.自动包装**

​	自动包装是在4.1.4节中提到的一种被缓存的扩展类，ExtensionLoader在加载扩展时，如果发现这个扩展类包含其他扩展点作为构造函数的参数，则这个扩展类就会被认为是Wrapper类，如代码清代4-4所示。

![](https://pic.imgdb.cn/item/60e546e45132923bf8af7357.jpg)

​	ProtocolFilterWrapper虽然继承了 Protocol接口，但是其构造函数中又注入了一个Protocol类型的参数。因此ProtocolFilterWrapper会被认定为Wrapper类。这是一种装饰器模式，把通用的抽象逻辑进行封装或对子类进行增强，让子类可以更加专注具体的实现。

**2.自动加载**

​	除了在构造函数中传入其他扩展实例，我们还经常使用setter方法设置属性值。如果某个扩展类是另外一个扩展点类的成员属性，并且拥有setter方法，那么框架也会自动注入对应的扩展点实例。ExtensionLoader在执行扩展点初始化的时候，会自动通过setter方法注入应的实现类。这里存在一个问题，如果扩展类属性是一个接口，它有多种实现，那么具体注入哪一个呢？这就涉及第三个特性一一自适应。

​	在Dubbo SPI中，我们使用@入(14?什作注解，可以动态地通过URL中的参数来确定要使用哪个具体的实现类。从而解决自动加载中的实例注入问题。©Adaptive注解使用示例如代码清单4-5所示。

![](https://pic.imgdb.cn/item/60e547635132923bf8b318c1.jpg)

​	@Adaptive传入了两个Constants中的参数，它们的值分别是"server”和“transporter”。当外部调用Transporter#bind方法时，会动态从传入的参数“URL”中提取key参数“server”的value值，如果能匹配上某个扩展实现类则直接使用对应的实现类；如果未匹配上，则继续通过第二个key参数“transporter”提取value值。如果都没匹配上，则抛出异常。也就是说，如果©Adaptive中传入了多个参数，则依次进行实现类的匹配，直到最后抛出异常。

​	这种动态寻找实现类的方式比较灵活，但只能激活一个具体的实现类，如果需要多个实现类同时被激活，如Filter可以同时有多个过滤器;或者根据不同的条件，同时激活多个实现类，如何实现？这就涉及最后一个特性一一自动激活。

**4. 自动激活**

​	使用@Activate注解，可以标记对应的扩展点默认被激活启用。该注解还可以通过传入不同的参数，设置扩展点在不同的条件下被自动激活。主要的使用场景是某个扩展点的多个实现类需要同时启用(比如Filter扩展点)。在4.2节中会详细介绍以上几种注解。

## 扩展点注解

### 扩展点注解：@SPI

​	@SPI注解可以使用在类、接口和枚举类上，Dubbo框架中都是使用在接口上。它的主要作用就是标记这个接口是一个Dubbo SPI接口，即是一个扩展点，可以有多个不同的内置或用户定义的实现。运行时需要通过配置找到具体的实现类。@SPI注解的源码如代码清单4-6所示。

![](https://pic.imgdb.cn/item/60e548635132923bf8bab20b.jpg)

​	我们可以看到SPI注解有一个value属性，通过这个属性，我们可以传入不同的参数来设
置这个接口的默认实现类。例如，我们可以看到Transporter接口使用Netty作为默认实现，
如代码清单4.7所示。

![](https://pic.imgdb.cn/item/60e548815132923bf8bbaa73.jpg)

### 扩展点自适应注解：@Adaptive

​	@Adaptive注解可以标记在类、接口、枚举类和方法上，但是在整个Dubbo框架中，只有几个地方使用在类级别上，如AdaptiveExtensionFactory和AdaptiveCompiler,其余都标注在方法上。如果标注在接口的方法上，即方法级别注解，则可以通过参数动态获得实现类，这一点已经在4.1.5节的自适应特性上说明。方法级别注解在第一次getExtension时，会自动生成和编译一个动态的Adaptive类，从而达到动态实现类的效果。

​	例如：Transporter接口在bind和connect两个方法上添加了@Adaptive注解，如代码清单4・5所示。Dubbo在初始化扩展点时，会生成一个Transporter$Adaptive类，里面会实现这两个方法，方法里会有一些抽象的通用逻辑，通过@Adaptive中传入的参数，找到并调用真正的实现类。熟悉装饰器模式的读者会很容易理解这部分的逻辑。具体实现原理会在4.4节讲解。

​	下面是自动生成的Transporter$Adaptive#bind实现代码，如代码清单4-8所示，已经省略了无关代码。

![](https://pic.imgdb.cn/item/60e5490a5132923bf8bfcb27.jpg)

​	当该注解放在实现类上，则整个实现类会直接作为默认实现，不再自动生成代码清单4-8中的代码。在扩展点接口的多个实现里，只能有一个实现上可以加@Adaptive注解。如果多个实现类都有该注解，则会抛出异常：More than 1 adaptive class found。@Adaptive注的源代码如代码清单4.9所示。

![](https://pic.imgdb.cn/item/60e549c65132923bf8c56b05.jpg)

​	该注解也可以传入value参数，是一个数组。我们在代码清单4.9中可以看到，Adaptive可以传入多个key值，在初始化Adaptive注解的接口时，会先对传入的URL进行key值匹配，第一个key没匹配上则匹配第二个，以此类推。直到所有的key匹配完毕，如果还没有匹配到，则会使用“驼峰规则”匹配，如果也没匹配到，则会抛出IllegalStateException异常。

​	什么是"驼峰规则”呢？如果包装类（Wrapper）没有用Adaptive指定key值，则Dubbo
会自动把接口名称根据驼峰大小写分开，并用符号连接起来，以此来作为默认实现类的名称，如 org.apache.dubbo.xxx.YyylnvokerWpappep 中的 YyylnvokerWrapper 会被转换为yyy.invoker.wrapper。

​	最后，为什么有些实现类上会标注@Adaptive呢？放在实现类上，主要是为了直接固定对
应的实现而不需要动态生成代码实现，就像策略模式直接确定实现类。在代码中的实现方式是：ExtensionLoader中会缓存两个与@Adaptive有关的对象，一个缓存在cachedAdaptiveClass中，即Adaptive具体实现类的Class类型；另外一个缓存在cachedAdaptivelnstance中，即Class的具体实例化对象。在扩展点初始化时，如果发现实现类有@Adaptive注解，则直接赋值给cachedAdaptiveClass ,后续实例化类的时候，就不会再动态生成代码，直接实例化cachedAdaptiveClass,并把实例缓存到cachedAdaptivelnstance中。如果注解在接口方法上，则会根据参数，动态获得扩展点的实现，会生成Adaptive类（如代码清单4-8所示），再缓存到cachedAdaptivelnstance 中。

### 扩展点自动激活注解：@Activate

​	@Activate可以标记在类、接口、枚举类和方法上。主要使用在有多个扩展点实现、需要根据不同条件被激活的场景中，如Filter需要多个同时激活，因为每个Filter实现的是不同的功能。@Activate可传入的参数很多，如表4-3所示。

![](https://pic.imgdb.cn/item/60e54b205132923bf8cf5849.jpg)

## ExtensionLoader 的工作原理

### 工作流程

​	ExtensionLoader 的逻辑入口可以分为 getExtension、getAdaptiveExtension、
getActivateExtension三个，分别是获取普通扩展类、获取自适应扩展类、获取自动激活的扩展类。总体逻辑都是从调用这三个方法开始的，每个方法可能会有不同的重载的方法，根据不同的传入参数进行调整，如图4-1所示。

![](https://pic.imgdb.cn/item/60e54bbc5132923bf8d3db8c.jpg)

​	三个入口中,getActivateExtension对getExtension 的依赖比较, getAdaptiveExtension则相对独立。

​	get Extension (String name)是整个扩展加载器中最核心的方法，实现了一个完整的普通扩展类加载过程。加载过程中的每一步，都会先检查缓存中是否己经存在所需的数据，如果存在则直接从缓存中读取，没有则重新加载。这个方法每次只会根据名称返回一个扩展点实现类。初始化的过程可以分为4步：

​	(1) 框架读取SPI对应路径下的配置文件，并根据配置加载所有扩展类并缓存(不初始化)。

​	(2) 根据传入的名称初始化对应的扩展类。

​	(3) 尝试查找符合条件的包装类：包含扩展点的setter方法，例如setProtocol(Protocol protocol)方法会自动注入protocol扩展点实现；包含与扩展点类型相同的构造函数，为其注入扩展类实例，例如本次初始化了一个Class A,初始化完成后，会寻找构造参数中需要Class A的包装类(Wrapper),然后注入Class A实例，并初始化这个包装类。

​	(4) 返回对应的扩展类实例。



​	getAdaptiveExtension也相对独立，只有加载配置信息部分与getExtension共用了同一个方法。和获取普通扩展类一样，框架会先检查缓存中是否有已经初始化化好的Adaptive实例，没有则调用createAdaptiveExtension重新初始化。初始化过程分为4步：

​	(1) 和getExtension 一样先加载配置文件。

​	(2) 生成自适应类的代码字符串。

​	(3) 获取类加载器和编译器，并用编译器编译刚才生成的代码字符串。Dubbo 一共有三种类型的编译器实现，这些内容会在4.4节讲解。

​	(4) 返回对应的自适应类实例。

### getExtension 的实现原理

​	getExtension的主要流程前面已经讲过了，本节主要会讲解每一步的实现原理。

​	当调用getExtension(String name)方法时，会先检查缓存中是否有现成的数据，没有则调用createExtension开始创建。这里有个特殊点，如果getExtension传入的name是true,则加载并返回默认扩展类。

​	在调用createExtension开始创建的过程中，也会先检查缓存中是否有配置信息，如果不存在扩展类，则会从 META-INF/services/> META-INF/dubbo/、META-INF/dubbo/internal/这几个路径中读取所有的配置文件，通过I/O读取字符流，然后通过解析字符串，得到配置文件中对应的扩展点实现类的全称(如com.alibaba.dubbo.common.extensionloader.activate.impl.GroupActivateExtImpl)o扩展点配置信息加载过程的源码如代码清单4-10所示。

![](https://pic.imgdb.cn/item/60e5517c5132923bf8fcfc1e.jpg)

​	加载完扩展点配置后，再通过反射获得所有扩展实现类并缓存起来。注意，此处仅仅是把Class加载到JVM中，但并没有做Class初始化。在加载Class文件时，会根据Class 上的注解来判断扩展点类型，再根据类型分类做缓存。缓存的分类已经在4.1.4节讲过。扩展类的缓存分类如代码清单4-11所示。

![](https://pic.imgdb.cn/item/60e5586c5132923bf82f4465.jpg)

最后，根据传入的name找到对应的类并通过Class.forName方法进行初始化，并为其注入
依赖的其他扩展类(自动加载特性)。当扩展类初始化后，会检查一次包装扩展类`Set<Class<?>>wrapperclasses`,查找包含与扩展点类型相同的构造函数，为其注入刚初始化的扩展类，如代码清单4-12所示。

![](https://pic.imgdb.cn/item/60e55f675132923bf85f1774.jpg)

​	在injectExtension方法中可以为类注入依赖属性,它使用了`ExtensionFactory#getExtension(Class<T> type, String name)`来获取对应的bean实例，这个工厂接口会在4.3.5节详细说明。我们先来了解一下注入的实现原理。

​	injectExtension方法总体实现了类似Spring的IoC机制，其实现原理比较简单：首先通
过反射获取类的所有方法，然后遍历以字符串set开头的方法，得到set方法的参数类型，再通过ExtensionFactory寻找参数类型相同的扩展类实例，如果找到，就设值进去，如代码清单4-13所示。

![](https://pic.imgdb.cn/item/60e55feb5132923bf8624c50.jpg)

## getAdaptiveExtension 的实现原理

​	由之前的流程我们可以知道，在getAdaptiveExtension()方法中，会为扩展点接口自动生成实现类字符串，实现类主要包含以下逻辑：为接口中每个有^Adaptive注解的方法生成默认实现(没有注解的方法则生成空实现)，每个默认实现都会从URL中提取Adaptive参数值，并以此为依据动态加载扩展点。然后，框架会使用不同的编译器，把实现类字符串编译为自适应类并返回。本节主要讲解字符串代码生成的实现原理。

​	生成代码的逻辑主要分为7步，具体步骤如下：

​	(1) 生成package> import、类名称等头部信息。此处只会引入一个类ExtensionLoader。为了不写其他类的import方法，其他方法调用时全部使用全路径。类名称会变为“接口名称+ SAdaptive ” 的格式。例如：Transporter 接口 会生成 Transporter$Adpative。

​	(2) 遍历接口所有方法，获取方法的返回类型、参数类型、异常类型等。为第(3)步判断是否为空值做准备。

​	(3) 生成参数为空校验代码，如参数是否为空的校验。如果有远程调用，还会添加Invocation参数为空的校验。

​	(4) 生成默认实现类名称。如果©Adaptive注解中没有设定默认值，则根据类名称生成，如YyylnvokerWrapper会被转换为yyy.invoker.wrappero生成的规则是不断找大写字母，并把它们用连接起来。得到默认实现类名称后，还需要知道这个实现是哪个扩展点的。

​	(5) 生成获取扩展点名称的代码。根据@Adaptive注解中配置的key值生成不同的获取代码，例如：如果是@Adaptive("protocol"),则会生成 url. getProtocol() 。

​	(6) 生成获取具体扩展实现类代码。最终还是通过getExtension(extName)方法获取自适应扩展类的真正实现。如果根据URL中配置的key没有找到对应的实现类，则会使用第(4)步中生成的默认实现类名称去找。

​	(7) 生成调用结果代码。

![](https://pic.imgdb.cn/item/60e562a85132923bf8736c4f.jpg)

![](https://pic.imgdb.cn/item/60e562e35132923bf874c9a6.jpg)

​	生成完代码之后就要对代码进行编译，生成一个新的Classo Dubbo中的编译器也是一个自适应接口，但@Adaptive注解是加在实现类AdaptiveCompiler上的。这样一来AdaptiveCompiler就会作为该自适应类的默认实现，不需要再做代码生成和编译就可以使用了。具体的编译器实现原理会在4.4节讲解。

​	如果一个接口上既有@SPI(”impl”)注解，方法上又有@Adaptive(”impl2”)注解，那么会以哪个key作为默认实现呢？由上面动态生成的SAdaptive类可以得知，最终动态生成的实现方法会是url.getParameter(nimpl2n, "impl”),即优先通过©Adaptive注解传入的key去查找扩展实现类；如果没找到，则通过@SPI注解中的key去查找；如果@SPI注解中没有默认值，则把类名转化为key,再去查找。

### getActivateExtension 的实现原理

​	接下来，我们讲解图4-1中的旧Activate的实现原理，先从它的入口方法说起。getActivateExtension(URL url. String key. String group)方法可以获取所有自动激活扩展点。参数分别是URL.URL中指定的key(多个则用逗号隔开)和URL中指定的组信息(group)。其实现逻辑非常简单，当调用该方法时，主线流程分为4步：

​	(1) 检查缓存，如果缓存中没有，则初始化所有扩展类实现的集合。

​	(2) 遍历整个©Activate注解集合，根据传入URL匹配条件(匹配group> name等)，得到所有符合激活条件的扩展类实现。然后根据@入"浦3七。中配置的before、after、order等参数进行排序，这些参数在4.2.3节中已经介绍过。

​	(3) 遍历所有用户自定义扩展类名称，根据用户URL配置的顺序，调整扩展点激活顺序(遵循用户在 URL 中配置的顺序，例如 URL 为 test ://localhost/test?ext=orderlJdefault,则扩展点ext的激活顺序会遵循先orderl再default,其中default代表所有有@Activate注解的扩展点)。

​	(4) 返回所有自动激活类集合。

​	获取Activate扩展类实现，也是通过getExtension得到的。因此，可以认为getExtension是其他两种Extension的基石。

​	此处有一点需要注意，如果URL的参数中传入了-default,则所有的默认@Activate都不会被激活，只有URL参数中指定的扩展点会被激活。如果传入了 符号开头的扩展点名，则该扩展点也不会被自动激活。例如：-xxxx,表示名字为xxxx的扩展点不会被激活。

### Extension Factory 的实现原理

​	经过前面的介绍，我们可以知道ExtensionLoader类是整个SPI的核心。但是，ExtensionLoader类本身又是如何被创建的呢？

​	我们知道Registry Factory工厂类通ii@Adaptive( ("protocol"})注解动态查找注册中心实现，根据URL中的protocol参数动态选择对应的注册中心工厂，并初始化具体的注册中心客户端。而实现这个特性的ExtensionLoader类，本身又是通过工厂方法ExtensionFactory创建的，并且这个工厂接口上也有SPI注解，还有多个实现。具体见代码清单4-15。

![](https://pic.imgdb.cn/item/60e565f65132923bf886ca90.jpg)

![](https://pic.imgdb.cn/item/60e566075132923bf88728de.jpg)

​	可以看到，除了 AdaptiveExtensionFactory,还有 SpiExtensionFactory 和 SpringExtensionFactory两个工厂。也就是说，我们除了可以从Dubbo SPI管理的容器中获取扩展点实例，还可以从Spring容器中获取。

​	那么Dubbo和Spring容器之间是如何打通的呢？我们先来看SpringExtensionFactory的实现，该工厂提供了保存Spring上下文的静态方法，可以把Spring上下文保存到Set集合中。当调用getExtension获取扩展类时，会遍历Set集合中所有的Spring上下文，先根据名字依次从每个Spring容器中进行匹配，如果根据名字没匹配到，则根据类型去匹配，如果还没匹配到则返回null,如代码清单4-16所示。

![](https://pic.imgdb.cn/item/60e566605132923bf8893117.jpg)

​	那么Spring的上下文又是在什么时候被保存起来的呢？我们可以通过代码搜索得知，在ReferenceBean和ServiceBean中会调用静态方法保存Spring上下文，即一个服务被发布被
引用的时候，对应的Spring 上下文会被保存下来。

​	我们再看一下SpiExtensionFactory,主要就是获取扩展点接口对应的Adaptive实现类。例如：某个扩展点实现类 ClassA 上有@Adaptive 注解，则调用SpiExtensionFactory#getExtension会直接返回ClassA实例，如代码清单4-17所示。

![](https://pic.imgdb.cn/item/60e567815132923bf88f9268.jpg)

​	经过一番流转，最终还是回到了默认实现AdaptiveExtensionFactory 上.,因为该工厂上有
@Adaptive注解。这个默认工厂在构造方法中就获取了所有扩展类工厂并缓存起来，包括	SpiExtensionFactory 和 SpringExtensionFactory0 AdaptiveExtensionFactory 构造方法如代码清单4-18所示。

![](https://pic.imgdb.cn/item/60e567ab5132923bf8906e2a.jpg)

​	被AdaptiveExtensionFactory缓存的工厂会通过TreeSet进行排序，SPI排在前面，Spring排在后面。当调用getExtension方法时，会遍历所有的工厂，先从SPI容器中获取扩展类；如果没找到，则再从Spring容器中查找。我们可以理解为，AdaptiveExtensionFactory持有了所有的具体工厂实现，它的getExtension方法中只是遍历了它持有的所有工厂，最终还是调用SPI或Spring工厂实现的getExtension方法。getExtension方法如代码清单4-19所示。

![](https://pic.imgdb.cn/item/60e567fd5132923bf8925982.jpg)

## 扩展点动态编译的实现

### 总体结构

​	Dubbo中有三种代码编译器，分别是JDK编译器、Javassist编译器和AdaptiveCompiler编译器。这几种编译器都实现了 Compiler接口，编译器类之间的关系如图4.3所示。

![](https://pic.imgdb.cn/item/60e568ac5132923bf895f91e.jpg)

​	从图4-3中可以看到, Compiler接口上含有一个SPI注解，注解的默认值是`@SPI(”javassist”)`,很明显，Javassist编译器将作为默认编译器。如果用户想改变默认编译器，则可以通过`<dubbo:application compiler="jdk" />`标签进行配置。

​	AdaptiveCompiler上面有^Adaptive注解，说明AdaptiveCompiler会固定为默认实现，这个Compiler的主要作用和AdaptiveExtensionFactory相似，就是为了管理其他Compiler,如代码清单4-20所示。

![](https://pic.imgdb.cn/item/60e5698e5132923bf89a8c98.jpg)

​	AdaptiveCompiler#setDefaultCompiler 方法会在 ApplicationConfig 中被调用，也就是 Dubbo在启动时，会解析配置中的<dubbo:application compiler="jdk" />标签，获取设置的值，初始化对应的编译器。如果没有标签设置，贝U使用@SPI(Hjavassistn)中的设置，即JavassistCompiler。

​	然后看一下AbstpactCompiler,它是一个抽象类，无法实例化，但在里面封装了通用的模板逻辑。还定义了一个抽象方法decompile ,留给子类来实现具体的编译逻辑。JavassistCompiler和JdkCompiler都实现了这个抽象方法。

​	主要抽象逻辑如下：

* 通过正则匹配出包路径、类名，再根据包路径、类名拼接出全路径类名。
* 尝试通过Class.forName加载该类并返回，防止重复编译。如果类加载器中没有这个
  类，则进入第3步。
* 调用doCompile方法进行编译。这个抽象方法由子类实现。

### Javassist动态代码编译

​	Java中动态生成Class的方式有很多，可以直接基于字节码的方式生成，常见的工具库有
CGLIB、ASM、Javassist等。而自适应扩展点使用了生成字符串代码再编译为Class的方式。

![](https://pic.imgdb.cn/item/60e56a3d5132923bf89e0359.jpg)

​	看完Javassist使用示例，其实Dubbo中DavassistCompiler的实现原理也很清晰了。由于我们之前已经生成了代码字符串，因此在JavassistCompiler中，就是不断通过正则表达式匹配不同部位的代码，然后调用Javassist库中的API生成不同部位的代码，最后得到一个完整的Class对象。具体步骤如下：

​	(1) 初始化Javassist,设置默认参数，如设置当前的classpath。

​	(2) 通过正则匹配出所有import的包，并使用Javassist添加import。

​	(3) 通过正则匹配出所有extends的包，创建Class对象，并使用Javassist添加extends。

​	(4) 通过正则匹配出所有implements包，并使用Javassist添加implements。

​	(5) 通过正则匹配出类里面所有内容，即得到｛｝中的内容，再通过正则匹配出所有方法,并使用Javassist添加类方法。

​	(6) 生成Class对象。

​	JavassistCompiler继承了抽象类Abstractcompiler,需要实现父类定义的一个抽象方法doCompileo以上步骤就是整个doCompile方法在JavassistCompiler中的实现。

### JDK动态代码编译

​	JdkCompiler是Dubbo编译器的另一种实现，使用了 JDK自带的编译器，原生JDK编译器包位于 javax. tools下。主要使用了三个东西：JavaFileObject 接口、ForwardingJavaFileManager 接口、DavaCompiler.CompilationTask 方法。整个动态编译过程可以简单地总结为：首先初始化一个JavaFileObject对象，并把代码字符串作为参数传入构造方法，然后调用JavaCompiler.CompilationTask方法编译出具体的类。JavaFileManager负责管理类文件的输入/输出位置。以下是每个接口/方法的简要介绍：

​	(1) JavaFileObject接口。字符串代码会被包装成一个文件对象，并提供获取二进制流
的接口。Dubbo框架中的JavaFileObjectlmpl类可以看作该接口一种扩展实现，构造方法中需要传入生成好的字符串代码，此文件对象的输入和输出都是ByteArray流。由于
SimpleDavaFileObject、JavaFileObject之间的关系属于JDK中的知识，因此在本章不深入讲解，有兴趣的读者可以自行查看JDK源码。

​	(2) DavaFileManager接口。主要管理文件的读取和输出位置。JDK中没有可以直接使用的实现类，唯一的实现类ForwardingDavaFileManager构造器又是protect类型。因此Dubbo中定制化实现了一个DavaFileManagerlmpl类，并通过一个自定义类加载器ClassLoaderlmpl完成资源的加载。

​	(3) DavaCompiler.CompilationTask 把 DavaFileObject 对象编译成具体的类。

## 小结

​	略。

# Dubbo启停原理分析

## 配置解析

​	目前Dubbo框架同时提供了 3种配置方式:XML配置、注解、属性文件(properties和ymal)配置，最常用的还是XML和注解两种方式。Dubbo 2.5.8以后重写了注解的逻辑，解决了一些遗留的bug,同时能更好地支持dubbo-spring-boot。

### 基于schema设计解析

​	Dubbo配置约束文件在dubbo-conf ig/dubbo-configspring/src/main/resources/dubbo.xsd 中，在 IntelliD IDEA 中能够自动查找这个文件，当用户使用属性时进行自动提示。

​	dubbo.xsd文件用来约束使用XML配置时的标签和对应的属性，比如Dubbo中的`＜dubbo:service＞和〈dubbo:reference〉`标签等。Spring 在解析到自定义的 namespace 标签时（比如＜dubbo:service＞标签），会查找对应的spring.schemas和spring.handlers文件，最终触发Dubbo的DubboNamespaceHandler类来进行初始化和解析。我们先看以下两个文件的内容：

![](https://pic.imgdb.cn/item/60eadb215132923bf800f423.jpg)

​	Dubbo设计的粒度很多都是针对方法级别设计的，比如方法级别的timeout、retries和mock特性。这里包含的模块详细用法可以参考文档：http://dubbo.apache.org/zh-cn/docs/user/references/xml/introduction.htmlo 在图 5-1 中，左边代表 schema 有继承关系的类型，右边是独立的类型。

![](https://pic.imgdb.cn/item/60eadb4c5132923bf8018fe1.jpg)

​	接下来我们看一个dubbo.xsd的真实的配置，以protocolType模块为例，如代码清单5-1所示。

![](https://pic.imgdb.cn/item/60eadc8a5132923bf8060e60.jpg)

​	在代码清单5-1中，我们可以简单理解其为协议定义约束字段，只有在这里定义的属性才会在Dubbo的XML配置文件中智能提示，当我们基于Dubbo做二次开发时，应该在schema中添加合适的字段，同时应该在dubbo-config-api对应的Config类中添加属性和get & set方法，这样用户在配置属性框架时会自动注入这个值。只有属性定义是不够的，为了让Spring正确解析标签，我们要定义element标签，与代码清单5-1中的protocolType进行绑定，这里以protocolType示例展示，如代码清单5-2所示。

![](https://pic.imgdb.cn/item/60eadc8a5132923bf8060e60.jpg)

![](https://pic.imgdb.cn/item/60eadd435132923bf808bb99.jpg)

​	目前绝大多数场景使用默认的Dubbo配置就足够了，如果新增特性，比如增加epoll特性，则只需要在 providerType> consumerType> ProviderConfig 和 ConsumerConfig 中增加 epoll属性和方法即可。如果使用已经存在schema类型(比如说protocolType),则只需要添加新属性即可，也不需要定义新的element标签。如果接口是级别通用的，一般我们只需要在
interfaceType中增加属性即可，继承自interfaceType的类型会拥有该字段。同理，在Dubbo对应类AbstractInterfaceConfig中增加属性和方法即可。

### 基于XML配置原理解析

![](https://pic.imgdb.cn/item/60eaddff5132923bf80b188f.jpg)

​	DubboNamespaceHandler主要把不同的标签关联至U解析实现类中o registerBeanDef initionParser方法约定了在Dubbo框架中遇到标签application> module和registry等都会委托给DubboBeanDefinitionParser处理。需要注意的是，在新版本中重写了注解实现，主要解决了以前实现的很多缺陷（比如无法处理AOP等），相关重写注解的逻辑会在后面讲解。

![](https://pic.imgdb.cn/item/60eade695132923bf80ca168.jpg)

​	前面的逻辑主要负责把标签解析成对应的Bean定义并注册到Spring ±下文中，同时保证了 Spring容器中相同id的Bean不会被覆盖。

接下来分析具体的标签是如何解析的，我们依次分析`<dubbo:service>>` `<dubbo:provider>`和`<dubbo:consumer>`标签，如代码清单5-5所示。

![](https://pic.imgdb.cn/item/60eb01135132923bf8bbf164.jpg)

​	通过对ServiceBean的解析我们可以看到只是特殊处理了 class属性取值，并且在解析过程中调用了 parseProperties方法，这个方法主要解析`〈dubbo:service〉`标签中的name、class和ref等属性。parseProperties方法会把key-value键值对提取出来放到BeanDefinition中，运行时Spring会自动处理注入值，因此ServiceBean就会包含用户配置的属性值了。

​	其中`〈dubbo:provider〉`和`〈dubbo:consumer〉`标签复用了解析代码(parseNested),主要逻辑是处理内部嵌套的标签，比如＜dubbo:provider＞内部可能嵌套了`〈dubbo:service〉`，如果使用了嵌套标签，则内部的标签对象会自动持有外层标签的对象，例如`＜dubbo:provider＞`内部定义了`＜dubbo:service＞`,解析内部的service并生成Bean的时候，会把外层provider实例对象注入service,这种设计方式允许内部标签直接获取外部标签属性。

​	当前标签的attribute是如何提取的呢？主要分为两种场景：

* 查找配置对象的get、set和is前缀方法，如果标签属性名和方法名称相同，则通过反射调用存储标签对应值。
* 如果没有和get、set和is前缀方法匹配，则当作parameters参数存储，parameters是一个Map对象。

![](https://pic.imgdb.cn/item/60ec4c005132923bf819eeea.jpg)

![](https://pic.imgdb.cn/item/60ec4c185132923bf81a90b2.jpg)

![](https://pic.imgdb.cn/item/60ec4c385132923bf81b7017.jpg)



Dubbo框架生成的BeanDefinition最终还是会委托Spring创建对应的Java对象,dubbo.xsd中定义的类型都会有与之对应的POJO, Dubbo承载对象和继承关系如图5-2所示。

![](https://pic.imgdb.cn/item/60ec4d355132923bf8224587.jpg)

### 基于注解配置原理解析

​	重启开源后,Dubbo的注解己经完全重写了，因为原来注解是基于AnnotationBean实现的，主要存在以下几个问题：

* 注解支持不充分，需要XML配置＜dubbo:annotation＞；
* @ServiceBean 不支持 Spring AOP；
* @Reference不支持字段继承性。



​	在原来实现思路的基础上无法解决历史遗留问题，但采用另外一种思路实现可以很好地修复并改进遗留问题。

​	详细原因和分析可以参考以下文章：

(1)https://github.com/mercyblitz/mercyblitz.github.io/blob/master/java/dubbo/Dubbo-Annotation-Driven.md

(2)https://zonghaishang.github.io/2018/10/01/Spring%E6%9D%82%E8%B0%88-%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E5%AF%BC%E8%87%B4Dubbo%E6%9C%8D%E5%8A%A1%E6%97%A0%E6%B3%95%E8%A2%AB%E6%AD%A3%E7%A1%AE%E4%BB%A3%E7%90%86/

​	注解处理逻辑主要包含3部分内容，第一部分是如果用户使用了配置文件，则框架按需生成对应Bean,第二部分是要将所有使用Dubbo的注解^Service的class提升为Bean,第三部分要为使用@Reference注解的字段或方法注入代理对象。我们先看一下@EnableDubbo注解，如代码清单5-7所示

![](https://pic.imgdb.cn/item/60ec4fcc5132923bf835cda7.jpg)

![](https://pic.imgdb.cn/item/60ec4fe15132923bf836765a.jpg)

​	当Spring容器启动的时候，如果注解上面使用.Import,则会触发其注解方法selectimports,
比如 EnableDubboConfig 注解中指定的 DubboConfigConfigurationSelector.class,会自动触发 DubboConfigConfigurationSelector#selectImports 方法。如果业务方配置了 Spring 的@PropertySource 或 XML 等价的配置(比如配置了框架 dubbo. registry .address 和
dubbo. application 等属性)，则 Dubbo 框架会在DubboConfigConfigurationSelector#selectlmports中自动生成相应的配置承载对象，比如Applicationconfig等。细心的读者可能发现DubboConfigConfiguration 里面标注了@EnableDubboConfigBindings, @EnableDubboConfigBindings同样指定了@Import(DubboConfigBindingsRegistrar.class)。因为@EnableDubboConfigBindings
允许指定多个@EnableDubboConfigBinding注解,Dubbo会根据用户配置属性自动填充这些承载的对象，如代码清单5.8所示。

![](https://pic.imgdb.cn/item/60ec51685132923bf842b2e8.jpg)

​	可以发现处理用户指定配置代码的逻辑比较简单，在DubboConfigBindingRegistrar实现中做了下面几件事情：

(1)如果用户配置了属性，比如dubbo.application.name,则会自动创建对应Spring Bean到容器。

(2)注册和配置对象Bean属性绑定处理器DubboConfigBindingBeanPostProcessor,委托
Spring做属性值绑定。

​	接下来我们看一下是如何对服务提供者通过注^Service进行暴露的，注解扫描也委托给Spring,本质上使用asm库进行字节码扫描注解元数据，感兴趣的读者可以参考Spring源代码
SimpleMetadataReadero 当用户使用注解旧DubboComponentScan 时，会激活DubboComponentScan-
Registrar, 同时生成 ServiceAnnotationBeanPostProcessor 和ReferenceAnnotationBeanPostProcessor两种处理器，通过名称很容易知道分别是处理服务注解和消费注解。我们首先分析服务注解逻辑，因为 ServiceAnnotationBeanPostProcessor 处理器实现了BeanDefinitionRegistryPostProcessor接口，Spring容器中所有Bean注册之后回调postProcessBeanDefinitionRegistry方法开始扫描旧Service注解并注入容器，如代码清单5-9所示。

![](https://pic.imgdb.cn/item/60ec52205132923bf8488c29.jpg)

![](https://pic.imgdb.cn/item/60ec52425132923bf8499f3a.jpg)

​	①：Dubbo框架首先会提取用户配置的扫描包名称，因为包名可能使用${...}占位符，因
此框架会调用Spring的占位符解析做进一步解码。②：开始真正的注解扫描，委托Spring对所有符合包名的.class文件做字节码分析，最终通过③配置扫描@Service注解作为过滤条件。在④中将仞Service标注的服务提升为不同的Bean,这里并没有设置beanClasso在⑤中主要根据注册的普通Bean生成ServiceBean的占位符，用于后面的属性注入逻辑。在⑥中会提取普通Bean上标注的Service注解生成新的RootBeanDefinition,用于Spring启动后的服务暴露，具体服务暴露的逻辑会在后面详细解析。

​	在实际使用过程中，我们会在旧Service注解的服务中注入旧Reference注解，这样就可以很方便地发起远程服务调用，Dubbo中做属性注入是通过ReferenceAnnotationBeanPost-Processor处理的，主要做以下几种事情(参考代码清单5-10处理引用注解)：

​	(1) 获取类中标注的@Reference注解的字段和方法。

​	(2) 反射设置字段或方法对应的引用。

![](https://pic.imgdb.cn/item/60ec53a55132923bf85501a3.jpg)

![](https://pic.imgdb.cn/item/60ec53b85132923bf855a050.jpg)

​	因为处理器 ReferenceAnnotationBeanPostProcessor 实现了InstantiationAwareBeanPostProcessor接口，所以在Spring的Bean中初始化前会触postProcessPropertyValues方法，该方法允许我们做进一步处理，比如增加属性和属性值修改等。在①中主要利用这个扩展点查找服务引用的字段或方法。在②中触发字段或反射方法值的注入，字段处理会调用findFieldReferenceMetadata方法，在③中会遍历类所有字

段，因为篇幅的原因，方法级别注入最终会调用findMethodReferenceMetadata方法处理上面的注解。在②中会触发字段或方法inject方法，使用泛化调用的开发人员可能用过ReferenceConfig创建引用对象，这里做注入用的是ReferenceBean类，它同样继承自ReferenceConfig,在此基础上增加了 Spring初始化等生命周期方法，比如触发afterPropertiesSet从容器中获取一些配置(protocol)等，当设置字段值的时候仅调用referenceBean.getObject()获取远程代理即可，具体服务消费会在5.3节讲解。

## 服务暴露的实现原理

### 配置承载初始化

​	不管在服务暴露还是服务消费场景下，Dubbo框架都会根据优先级对配置信息做聚合处理,目前默认覆盖策略主要遵循以下几点规则：

(1) -D 传递给 JVM 参数优先级最高，比如-Ddubbo. protocol.port=20880

(2) 代码或XML配置优先级次高，比如Spring中XML文件指定`<dubbo:protocolport="20880">`

3) 配置文件优先级最低，比如 dubbo.properties 文件指定 dubbo.protocol.port=20880。

一般推荐使用dubbo.properties作为默认值，只有XML没有配置时，dubbo.properties配置项才会生效，通常用于共享公共配置，比如应用名等。

Dubbo的配置也会受到provider的影响，这个属于运行期属性值影响，同样遵循以下几点规则：

(1) 如果只有provider端指定配置，则会自动透传到客户端(比如timeout)

(2) 如果客户端也配置了相应属性，则服务端配置会被覆盖(比如timeout)

### 远程服务的暴露机制

​	我们先看一下整体RPC的暴露原理，如图5-4所示。

![](https://pic.imgdb.cn/item/60ed87a45132923bf8ea8746.jpg)

在整体上看，Dubbo框架做服务暴露分为两大部分，第一步将持有的服务实例通过代理转换成Invoker,第二步会把Invoker通过具体的协议（比如Dubbo）转换成Exporter,框架做了
这层抽象也大大方便了功能扩展。这里的Invoker可以简单理解成一个真实的服务对象实例，是Dubbo框架实体域，所有模型都会向它靠拢，可向它发起invoke调用。它可能是一个本地的实现，也可能是一个远程的实现，还可能是一个集群实现。

Dubbo支持多注册中心同时写，如果配置了服务同时注册多个注册中心，则会在ServiceConfig#doExportUrls中依次暴露，如代码清单5-11所示。

![](https://pic.imgdb.cn/item/60ed88785132923bf8eed7cb.jpg)

Dubbo也支持相同服务暴露多个协议，比如同时暴露Dubbo和REST协议，框架内部会依次对使用的协议都做一次服务暴露，每个协议注册元数据都会写入多个注册中心。在①中会自动获取用户配置的注册中心，如果没有显示指定服务注册中心，则默认会用全局配置的注册中心。在②中处理多协议服务暴露的场景，真实服务暴露逻辑是在doExportUrlsForlProtocol方法中实现的，如代码清单5-12所示。

![](https://pic.imgdb.cn/item/60ed8a8b5132923bf8fa230d.jpg)

![](https://pic.imgdb.cn/item/60ed8a9d5132923bf8fa886e.jpg)

为了更容易地理解服务暴露与注册中心的关系，以下列表项分别展示有注册中心和无注册中心的URL：

* registry://host:port/com.alibaba.dubbo.registry.RegistryService?protocol=zo
  okeeper&export=dubbo://ip:port/xxx?..。
* dubbo://ip:host/xxx.Service?timeout=1000&..。

protocol实例会自动根据服务暴露URL自动做适配，有注册中心场景会取出具体协议，比如ZooKeeper,首先会创建注册中心实例，然后取出export对应的具体服务URL,最后用服务URL对应的协议(默认为Dubbo)进行服务暴露，当服务暴露成功后把服务数据注册到ZooKeeper。如果没有注册中心，则在⑦中会自动判断URL对应的协议(Dubbo)并直接暴露服务，从而有经过注册中心。

在将服务实例ref转换成Invoker之后，如果有注册中心时，则会通过RegistryProtocol#export进行更细粒度的控制，比如先进行服务暴露再注册服务元数据。注册中心在做服务暴露时依次做了以下几件事情(逻辑如代码清单5.13所示)。

(1) 委托具体协议(Dubbo)进行服务暴露，创建NettyServer监听端口和保存服务实例。

(2) 创建注册中心对象，与注册中心创建TCP连接。

(3) 注册服务元数据到注册中心。

(4) 订阅configurators节点，监听服务动态属性变更事件。

(5) 服务销毁收尾工作，比如关闭端口、反注册服务信息等。

![](https://pic.imgdb.cn/item/60ed8b7d5132923bf8ff517d.jpg)

![](https://pic.imgdb.cn/item/60ed8b8c5132923bf8ffa4a7.jpg)

通过代码清单5-13可以清楚地看到服务暴露的各个流程，当服务真实调用时会触发各种拦截器Filter,这个是在哪里初始化的呢？在①中进行服务暴露前，框架会做拦截器初始化Dubbo在加载 protocol 扩展点时会自动注入 Protocol Listenerwrapper 和ProtocolFilterWrapper。真实暴露时会按照图5-5所示的流程执行。

![](https://pic.imgdb.cn/item/60ed8bae5132923bf80061cd.jpg)

![](https://pic.imgdb.cn/item/60ed8bc55132923bf800df6d.jpg)

在构造调用拦截器之后会调用Dubbo协议进行服务暴露，请参考DubboProtocol#export（如代码清单5.15所示）实现。

![](https://pic.imgdb.cn/item/60ed8bf85132923bf801fed5.jpg)

### 本地服务的暴露机制

​	5.2.2节主要讲解了服务远程暴露的主流程，很多使用Dubbo框架的应用可能存在同一个JVM暴露了远程服务，同时同一个JVM内部又引用了自身服务的情况，Dubbo默认会把远程服务用injvm协议再暴露一份，这样消费方直接消费同一个JVM内部的服务，避免了跨网络进行远程通信

![](https://pic.imgdb.cn/item/60ed8c3d5132923bf803855a.jpg)

通过exportLocal实现可以发现，在①中显示Dubbo指定用injvm协议暴露服务，这个协议比较特殊，不会做端口打开操作，仅仅把服务保存在内存中而已。在②中会提取URL中的协议，在InjvmProtocol类中存储服务实例信息，它的实现也是非常直截了当的，直接返回InjvmExporter实例对象，构造函数内部会把当前Invoker加入exporterMap,如代码清单5-17
所示。

![](https://pic.imgdb.cn/item/60ed8c605132923bf8044bad.jpg)

## 服务消费的实现原理

### 单注册中心消费原理

![](https://pic.imgdb.cn/item/60ed8d135132923bf80845ee.jpg)

​	在整体上看，Dubbo框架做服务消费也分为两大部分，第一步通过持有远程服务实例生成Invoker,这个Invoker在客户端是核心的远程代理对象。第二步会把Invoker通过动态代理转换成实现用户接口的动态代理引用。这里的Invoker承载了网络连接、服务调用和重试等功能，在客户端，它可能是一个远程的实现，也可能是一个集群实现。

​	Dubbo支持多注册中心同时消费，如果配置了服务同时注册多个注册中心，则会在ReferenceConfig#createProxy 中合并成一个 Invoker,如代码清单 5-18 。

![](https://pic.imgdb.cn/item/60ed8d505132923bf8099ca9.jpg)

![](https://pic.imgdb.cn/item/60ed8d5e5132923bf809ecca.jpg)

​	当经过注册中心消费时，主要通过RegistryProtocol#refer触发数据拉取、订阅和服务Invoker转换等操作，其中最核心的数据结构是RegistryDirectory,如代码清单5-19所示。

![](https://pic.imgdb.cn/item/60ed8d985132923bf80b40f5.jpg)

![](https://pic.imgdb.cn/item/60ed8de75132923bf80d0f76.jpg)



​	具体远程Invoker是在哪里创建的呢？客户端调用拦截器又是在哪里构造的呢？当在⑦中第一次发起订阅时会进行一次数据拉取操作，同时触发RegistryDirectory#notify方法，这里的通知数据是某一个类别的全量数据，比如providers和routers类别数据。当通知providers数据时，在RegistryDirectory#toInvokers方法内完成Invoker转换。下面是具体细节，如代码清单5-20所示。

![](https://pic.imgdb.cn/item/60ed8e2a5132923bf80e916e.jpg)

![](https://pic.imgdb.cn/item/60ed8e385132923bf80ee647.jpg)

Dubbo框架允许在消费方配置只消费指定协议的服务，具体协议过滤在①中进行处理，支持消费多个协议，允许消费多个协议时，在配置Protocol值时用逗号分隔即可。在②中消费信息是客户端处理的，需要合并服务端相关信息，比如远程IP和端口等信息，通过注册中心获取这些信息，解耦了消费方强绑定配置。在③中消除重复推送的服务列表，防止重复引用。在④中使用具体的协议发起远程连接等操作。在真实远程连接建立后也会发起拦截器构建操作，可参考5.2.2节的图5-4,处理机制类似，只不过处理逻辑在ProtocolFilterWrapper#refer中触发链式构造。

​	DubboProtocol#refer内部会调用DubboProtocol#initClient负责建立客户端连接和初始化Handler,如代码清单5-21所示。

![](https://pic.imgdb.cn/item/60ed8e745132923bf8104464.jpg)

### 多注册中心消费原理

​	在实际使用过程中，我们更多遇到的是单注册中心场景，但是当跨机房消费时，Dubbo框架允许同时消费多个机房服务。默认Dubbo消费机房的服务顺序是按照配置注册中心的顺序决定的，配置靠前优先消费。

![](https://pic.imgdb.cn/item/60ed8e995132923bf81120b7.jpg)

### 直连服务消费原理

​	Dubbo可以绕过注册中心直接向指定服务(直接指定目标IP和端口)发起RPC调用，使用直连模式可以方便在某些场景下使用，比如压测指定机器等。Dubbo框架也支持同时指定直连多台机器进行服务调用，如代码清单5-23所示。

![](https://pic.imgdb.cn/item/60ed8eb85132923bf811d059.jpg)

![](https://pic.imgdb.cn/item/60ed8ec65132923bf8121e3c.jpg)



## 优雅停机原理解析

![](https://pic.imgdb.cn/item/60ed8cb65132923bf8063537.jpg)

Dubbo中实现的优雅停机机制主要包含6个步骤：

1)收到kill 9进程退出信号，Spring容器会触发容器销毁事件。

2) provider端会取消注册服务元数据信息。

3) consumer端会收到最新地址列表（不包含准备停机的地址）。

4) Dubbo协议会发送readonly事件报文通知consumer服务不可用。

5)服务端等待已经执行的任务结束并拒绝新任务执行。

# Dubbo远程调用

## Dubbo调用介绍

​	在讲解Dubbo中的RPC调用细节之前，我们先回顾一次调用过程经历了哪些处理步骤。如果我们动手写简单的RPC调用，则需要把服务调用信息传递到服务端，每次服务调用的一些用的信息包括服务调用接口、方法名、方法参数类型和方法参数值等，在传递方法参数值时需要先序列化对象并经过网络传输到服务端，在服务端需要按照客户端序列化顺序再做一次反序列化来读取信息，然后拼装成请求对象进行服务反射调用，最终将调用结果再传给客户端。

![](https://pic.imgdb.cn/item/60ed8f4e5132923bf8153e8a.jpg)

​	首先在客户端启动时会从注册中心拉取和订阅对应的服务列表，Cluster会把拉取的服务列表聚合成一个Invoker,每次RPC调用前会通过Directory#list获取providers地址（已经生成好的Invoker列表），获取这些服务列表给后续路由和负载均衡使用。对应图6.1,在①中主要是将多个服务提供者做聚合。在框架内部另外一个实现Directory接口是RegistryDirectory类，它和接口名是一对一的关系（每一个接口都有一个RegistryDirectory实例），主要负责拉取和订阅服务提供者、动态配置和路由项。

​	在Dubbo发起服务调用时，所有路由和负载均衡都是在客户端实现的。客户端服务调用首先会触发路由操作，然后将路由结果得到的服务列表作为负载均衡参数，经过负载均衡后会选出一台机器进行RPC调用，这3个步骤依次对应于②、③和④。客户端经过路由和负载均衡后，会将请求交给底层I/O线程池（比如Netty）处理，I/O线程池主要处理读写、序列化和反序列化等逻辑，因此这里一定不能阻塞操作，Dubbo也提供参数控制（decode.in.io）参数，在处理反序列化对象时会在业务线程池中处理。在⑤中包含两种类似的线程池，一种是I/O线程池（Netty）,另一种是Dubbo业务线程池（承载业务方法调用）。

​	目前Dubbo将服务调用和Telnet调用做了端口复用，在编解码层面也做了适配。在Telnet调用时，会新建立一个TCP连接，传递接口、方法和JSON格式的参数进行服务调用，在编解码层面简单读取流中的字符串（因为不是Dubbo标准头报文），最终交给Telnet对应的Handler去解析方法调用。如果是非Telnet调用，则服务提供方会根据传递过来的接口、分组和版本信息查找Invoker对应的实例进行反射调用。在⑦中进行了端口复用，如果是Telnet调用，则先找到对应的Invoker进行方法调用。Telnet和正常RPC调用不一样的地方是序列化和反序列化使用的不是Hessian方式，而是直接使用fastjson进行处理。如果读者对目前的流程没有完全理解也没有关系，后面会逐渐深入讲解。

## Dubbo协议详解

​	本节我们讲解Dubbo协议设计，其协议设计参考了现有TCP/IP协议。

![](https://pic.imgdb.cn/item/60ed90b65132923bf81da95c.jpg)

​	理解协议本身的内容对后面的编码器和解码器的实现非常重要，我们先逐字节、逐比特位
讲解协议内容(具体内容参考表6-l)。



![](https://pic.imgdb.cn/item/60ed90d75132923bf81e76f5.jpg)

![](https://pic.imgdb.cn/item/60ed90ef5132923bf81f0cb1.jpg)

​	在消息体中，客户端严格按照序列化顺序写入消息，服务端也会遵循相同的顺序读取消息，客户端发起请求的消息体依次保存下列内容：Dubbo版本号、服务接口名、服务接口版本、方法名、参数类型、方法参数值和请求额外参数(attachment)。

![](https://pic.imgdb.cn/item/60ed910c5132923bf81fc58f.jpg)

​	主要根据以下标记判断返回值，如表6-3所示。

![](https://pic.imgdb.cn/item/60ed91315132923bf820aac5.jpg)

​	我们知道在网络通信中（基于TCP）需要解决网络粘包/解包的问题，一些常用解决办法比如用回车、换行、固定长度和特殊分隔符等进行处理，通过对前面协议的理解，我们很容易发现Dubbo其实就是用特殊符号0xdabb魔法数来分割处理粘包问题的。

![](https://pic.imgdb.cn/item/60ed916b5132923bf8221810.jpg)

​	当客户端多个线程并发请求时，框架内部会调用DefaultFuture对象的get方法进行等待。在请求发起时，框架内部会创建Request对象，这个时候会被分配一个唯一 id, DefaultFuture可以从Request对象中获取id,并将关联关系存储到静态HashMap中，就是图6-3中的Futures集合。当客户端收到响应时，会根据Response对象中的id,从Futures集合中查找对应DefaultFuture对象，最终会唤醒对应的线程并通知结果。客户端也会启动一个定时扫描线程去探测超时没有返回的请求。

## 编解码器原理

![](https://pic.imgdb.cn/item/60f2de9a5132923bf8f31ebc.jpg)

​	在图6-4中,AbstractCodec主要提供基础能力，比如校验报文长度和查找具体编解码器等。TransportCodec主要抽象编解码实现，自动帮我们去调用序列化、反序列实现和自动cleanup流。
我们通过Dubbo编解码继承结构可以清晰看到，DubboCodec继承自ExchangeCodec,它又再次继承了 TelnetCodec实现。我们前面说过Telnet实现复用了 Dubbo协议端口，其实就是在这层编解码做了通用处理。因为流中可能包含多个RPC请求，Dubbo框架尝试一次性读取更多完整报文编解码生成对象，也就是图中的DubboCountCodec,它的实现思想比较简单，依次调用DubboCodec去解码，如果能解码成完整报文，贝I」加入消息列表，然后触发下一个Handler方法调用。

### Dubbo协议编码器

​	Dubbo中的编码器主要将Java对象编码成字节流返回给客户端，主要做两部分事情，构造报文头部，然后对消息体进行序列化处理。所有编解码层实现都应该继承自Exchangecodec,Dubbo协议编码器也不例外。当Dubbo协议编码请求对象时，会调用ExchangeCodec#encode方法。我们首先分析编码请求对象，如代码清单6.1所示。

![](https://pic.imgdb.cn/item/60f2e0ec5132923bf8045e68.jpg)

![](https://pic.imgdb.cn/item/60f2e1145132923bf805924a.jpg)

​	通过上面的请求编码器实现，在理解6.2节协议的基础上很容易理解这里的代码，在⑧中会调用encodeRequestData方法对Rpclnvocation调用进行编码，这部分主要就是对接口、方法、方法参数类型、方法参数等进行编码，在DubboCodec#encodeRequestData中重写了这个方法实现，如代码清单6-2所示。

![](https://pic.imgdb.cn/item/60f2e1565132923bf8078f99.jpg)

​	代码清单6-2的主要职责是将Dubbo方法调用参数和值编码成字节流。在编码消息体的时候，在①中主要先写入框架的版本，这里主要用于支持服务端版本隔离和服务端隐式参数透传给客户端的特性。在②中向服务端写入调用的接口。在③中指定接口的版本，默认版本为0.0.0,Dubbo允许同一个接口有多个实现，可以指定版本或分组来区分。在④中指定远程调用的接口方法。在⑤中将方法参数类型以Java类型方式传递给服务端。在⑥中循环对参数值进行序列化。在⑦中写入隐式参数HashMap,这里可能包含timeout和group等动态参数。

![](https://pic.imgdb.cn/item/60f2e59d5132923bf8295c1a.jpg)

![](https://pic.imgdb.cn/item/60f2e5b65132923bf82a27b6.jpg)

![](https://pic.imgdb.cn/item/60f2e5cc5132923bf82adb67.jpg)

​	代码清单6-3的主要职责是将Dubbo响应对象编码成字节流(包括协议报文头部)。在编码响应中，在①中获取用户指定或默认的序列化协议，在②中构造报文头部(16字节)。在③中同样将魔法数填入报文头部前2个字节。在④中会将服务端配置的序列化协议写入头部。在⑤中报文头部中status会保存服务端调用状态码。在⑥中会将请求唯一 id设置回响应头中。在⑦中空出16字节头部用于存储响应体报文。在⑧中会对服务端调用结果进行编码，后面会进行详细解释。在⑨中主要对响应报文大小做检查，默认校验是否超过8MB大小。在⑩中将消息体长度写入头部偏移量(第12个字节)，长度占用4个字节。在⑪中将buffer定位到报文头部开始，在⑫中将构造好的头部写入buffer。在⑬中再将buffer写入索引执行消息体结尾的下一个位置。在⑭中主要处理编码报错复位buffer,否则导致缓冲区中数据错乱。在⑥中会将异常响应返回到客户端，防止客户端只有等到超时才能感知服务调用返回。在⑯和⑰中主要对报错进行了细分，处理服务端报文超过限制和具体报错原因。为了防止报错对象无法在客户端反序列化，在服务端会将异常信息转成字符串处理。

![](https://pic.imgdb.cn/item/60f2e5f75132923bf82c452b.jpg)

​	代码清单6-4的主要职责是将Dubbo方法调用状态和返回值编码成字节流。编码响应体也是比较简单的，在①中判断客户端的版本是否支持隐式参数从服务端传递到客户端。在②和③中处理正常服务调用，并且返回值为null的场景，用一个字节标记。在④中处理方法正常调用并且有返回值，先写一个字节标记并序列化结果。在⑤中处理方法调用发生异常，写一个字节标记并序列化异常对象。在⑥中处理客户端支持隐式参数回传，记录服务端Dubbo版本，并返回服务端隐式参数。

### Dubbo协议解码器

​	6.3.1节主要完成了对编码的分析，本节聚焦解码的实现，相比较编码而言，解码要复杂一些。解码工作分为2部分，第1部分解码报文的头部(16字节)，第2部分解码报文体内容，以及如何把报文体转换成Rpclnvocation 0当服务端读取流进行解码时，会触发ExchangeCodec#decode方法，Dubbo协议解码继承了这个类实现，但是在解析消息体时，Dubbo协议重写了 decodeBody方法。我们先分析解码头部的部分，如代码清单6-5所示。

![](https://pic.imgdb.cn/item/60f2e74b5132923bf8377d0f.jpg)

![](https://pic.imgdb.cn/item/60f2e75b5132923bf8380b37.jpg)

​	整体实现解码过程中要解决粘包和半包问题。在①中最多读取Dubbo报文头部(16字节)，如果流中不足16字节，则会把流中数据读取完毕。在decode方法中会先判断流当前位置是不是Dubbo报文开始处，在流中判断报文分割点是通过②判断的(Oxdabb魔法数)。如果当前流中没有遇到完整Dubbo报文(在③中会判断流可读字节数)，在④中会为剩余可读流分配存储空间，在⑤中会将流中数据全部读取并追加在header数组中。当流被读取完后，会查找流中第一个Dubbo报文开始处的索引，在⑥中会将buffer索引指向流中第一个Dubbo报文开始处(Oxdabb)0在⑦中主要将流中从起始位置(初始buffer的readerindex)到第一个Dubbo报文开始处的数据保存在header中，用于⑧解码header数据，目前常用的场景有Telnet调用等。在正常场景中解析时，在⑨中首先判断当次读取的字节是否多于16字节，否则等待更多网络数据到来。在⑩中会判断Dubbo报文头部包含的消息体长度，然后校验消息体长度是否超过限制（默认为8MB）。在⑪中会校验这次解码能否处理整个报文。在⑫中处理消息体解码，这个是强协议相关的，因此Dubbo协议重写了这部分实现，我们先看一下在DubboCodec中是如
何处理的，如代码清单6-6所示。

![](https://pic.imgdb.cn/item/60f2e78d5132923bf839c129.jpg)

![](https://pic.imgdb.cn/item/60f2e7995132923bf83a2349.jpg)

​	站在解码器的角度，解码请求一定是通过标志判断类别的，否则不知道是请求还是响应，Dubbo报文16字节头部长度包含了 FLAG_REQUEST标志位。①：根据这个标志位创建请求对象，②：在I/O线程中直接解码(比如在Netty的I/O线程中)，然后简单调用decode解码，解码逻辑在后面会详细探讨。③：实际上不做解码，延迟到业务线程池中解码。④：将解码消息体作为Rpclnvocation放到请求数据域中。如果解码失败了，则会通过⑤标记，并把异常原因记录下来。这里没有提到的是心跳和事件的解码，这两种解码非常简单，心跳报文是没有消息体的，事件有消息体，在使用Hessian2协议的情况下默认会传递字符R,当优雅停机时会通过发送readonly事件来通知客户端服务端不可用。接下来，我们分析一下如何把消息体转换成Rpclnvocation对象，具体解码会触发DecodeableRpcInvocation#decode 方法，如代码清单 6-7 所示。

![](https://pic.imgdb.cn/item/60f2e7cb5132923bf83bd721.jpg)

![](https://pic.imgdb.cn/item/60f2e8535132923bf8407a29.jpg)

![](https://pic.imgdb.cn/item/60f2e86e5132923bf8416a3a.jpg)

​	在解码请求时，是严格按照客户端写数据顺序来处理的。在①中会读取远端传递的框架版本，在②中会读取调用接口全称，在③中会读取调用的服务版本，用来实现分组和版本隔离。在④中会读取调用方法的名称，在⑤中读取方法参数类型，通过类型能够解析出实际参数个数。在⑥中会对方法参数值依次读取，这里具体解析参数值是和序列化协议相关的。在⑦中读取隐式参数，比如同机房优先调用会读取其中的tag值。⑧是为了支持异步参数回调，因为参数是回调客户端方法，所以需要在服务端创建客户端连接代理。

![](https://pic.imgdb.cn/item/60f2e8855132923bf8422c6d.jpg)

![](https://pic.imgdb.cn/item/60f2e8965132923bf842cfdd.jpg)

​	在读取服务端响应报文时，先读取状态标志，然后根据状态标志判断后续的数据内容。在代码清单6・4编码响应对象中，响应结果首先会写一个字节标记位。在①中处理标记位代表返回值为Null的场景。②代表正常返回，首先判断请求方法的返回值类型，返回值类型方便底层反序列化正确读取，将读取的值存在result字段中。在④中处理服务端返回异常对象的场景，同时会将结果保存在exception字段中。在⑤中处理返回值为Null,并且支持服务端隐式参数透传给客户端，在客户端会继续读取保存在HashMap中的隐式参数值。当然，还有其他场景，比如RPC调用有返回值，RPC调用抛出异常时需要隐式参数给客户端的场景，可以举一反三，不再重复说明。

## Telnet调用原理

​	理解Telnet调用并不难，编解码器主要把Telnet当作明文字符串处理，按照Dubbo的调用规范，解析成调用命令格式，然后查找对应的Invoker,发起方法调用即可。

### Telnet指令解析原理

![](https://pic.imgdb.cn/item/60f2e8f05132923bf845e5b3.jpg)

​	通过这个扩展点的定义，能够解决扩展更多命令的诉求。message包含处理命令之外的所有字符串参数，具体如何使用这些参数及这些参数的定义全部交给命令实现者决定。

完成Telnet指令转发的核心实现类是TelnetHandlerAdapter,它的实现非常简单，首先将用户输入的指令识别成commandC比如invoke>Is和status),然后将剩余的内容解析成message,message会交给命令实现者去处理。实现代码类在TelnetHandlerAdapter#telnet中，如代码清单6-10所示。

![](https://pic.imgdb.cn/item/60f2e90e5132923bf846fa8b.jpg)

![](https://pic.imgdb.cn/item/60f2e9ab5132923bf84c7436.jpg)

​	理解编解码后，可以更好地理解上层的实现和原理，在①中提取Telnet一行消息的首个字符串作为命令，如果命令行有空格，则将后面的内容作为字符串，再通过②提取并存储到message中。在③中判断并加载是否有对应的扩展点，如果存在对应的Telnet扩展点，则会通过④加载具体的扩展点并调用其telnet方法，最后连同返回结果并追加消息结束符(在⑤中处理)返回给调用方。

​	在讲解完Telnet调用转发后，我们对常用命令Invoke调用进行探讨，在InvokeTelnetHandler中本地实现Telnet类调用，如代码清单6-11所示。

![](https://pic.imgdb.cn/item/60f2e9cf5132923bf84dd0a1.jpg)

![](https://pic.imgdb.cn/item/60f2e9de5132923bf84e5448.jpg)

![](https://pic.imgdb.cn/item/60f2e9ea5132923bf84ec328.jpg)

​	当本地没有客户端，想测试服务端提供的方法时，可以使用Telnet登录到远程服务器(Telnet IP port),根据invoke指令执行方法调用来获得结果。当用户输入invoke指令时，会被转发到代码清单6-11对应的Handlero在①中提取方法调用信息(去除参数信息)，在②中会提取调用括号内的信息作为参数值。在③中提取方法调用的接口信息，在④中提取接口调用的方法名称。在⑤中会将传递的JSON参数值转换成fastjson对象,然后在⑥中根据接口名称、方法和参数值查找对应的方法和Invoker对象。在真正方法调用前，需要通过⑦把fastjson对象转换成Java对象，在⑧中触发方法调用并返回结果值。

### Telnet实现健康监测

​	Telnet提供了健康检查的命令，可以在Telnet连接成功后执行status -1查看线程池、内存和注册中心等状态信息。为了完成线程池监控、内存和注册中心监控等诉求，Telnet提供了新的扩展Statuschecker,如代码清单6-12所示。

![](https://pic.imgdb.cn/item/60f2ea1b5132923bf85079d2.jpg)

​	当执行status命令时会触发StatusTelnetHandler#telnet调用，这个方法的实现也比较简单，它会加载所有实现Statuschecker扩展点的类，然后调用所有扩展点的check方法。因为这类扩展点的具体实现并不复杂，所以不会一一讲解，表6.4列出了健康监测对应的实现和作用。

![](https://pic.imgdb.cn/item/60f2ea335132923bf85152c7.jpg)

## ChannelHandler

​	如果读者熟悉Netty框架，那么很容易理解Dubbo内部使用的ChannlHandler组件的原理,Dubbo框架内部使用大量Handler组成类似链表，依次处理具体逻辑，比如编解码、心跳时间戳和方法调用Handler等。因为Netty每次创建Handler都会经过ChannelPipeline,大量的事件
经过很多Pipeline会有较多的开销，因此Dubbo会将多个Handler聚合为一个Handler0

### 核心Handler和线程模型

![](https://pic.imgdb.cn/item/60f2ea605132923bf852f002.jpg)

![](https://pic.imgdb.cn/item/60f2ea6c5132923bf8535eb9.jpg)

​	Dubbo中提供了大量的Handler去承载特性和扩展，这些Handler最终会和底层通信框架做关联，比如Netty等。一次完整的RPC调用贯穿了一系列的Handler,如果直接挂载到底层通信框架（Netty）,因为整个链路比较长，则需要触发大量链式查找和事件，不仅低效，而且浪费资源。

​	图6-5展示了同时具有入站和出站ChannelHandler的布局，如果有一个入站事件被触发，比如连接或数据读取，那么它会从ChannelPipeline头部开始一直传播到Channelpipeline的尾端。出站的I/O事件将从ChannelPipeline最右边开始，然后向左传播。当然，在ChannelPipeline传播事件时，它会测试入站是否实现了 ChannellnboundHandler接口，如果没有实现则会自动跳过，出站时会监测是否实现ChannelOutboundHandler,如果没有实现，那么也会自动跳过。在Dubbo框架中实现的这两个接口类主要是NettyServerHandler和NettyClientHandler0Dubbo通过装饰者模式层包装Handler,从而不需要将每个Handler都追加到Pipeline中。在NettyServer 和 NettyClient 中最多有 3 个 Handler,分别是编码、解码和 NettyServerHandler或 NettyClientHandlero

![](https://pic.imgdb.cn/item/60f2eab05132923bf855ca14.jpg)

![](https://pic.imgdb.cn/item/60f2eac15132923bf8566645.jpg)

​	经按照特定规则(端口、接口名、接口版本和接口分组)把实例Invoker存储到HashMap中，客户端调用过来时必须携带相同信息构造的key,找到对应Exporter然后调用。在①中查找当前已经暴露的服务，后面会继续分析这个方法实现。在②中主要包含实例的Filter和真实业务对象，当触发invoker#invoke方法时，就会执行具体的业务逻辑。在DubboProtocol中，我们继续跟踪getlnvoker调用,会发现在服务端唯一标识的服务是由4部分组成的：端口、接口名、接口版本和接口分组。服务端Invoker查找如代码清单6-14所示。

![](https://pic.imgdb.cn/item/60f2ead65132923bf85729e6.jpg)

​	为了理解关键原理，特意移除了异步参数回调逻辑，这部分内容会单独在第9章高级特性中探讨。在①中主要获取协议暴露的端口，比如Dubbo协议默认的端口为20880。在②中获取客户端传递过来的接口名称(大部分场景都是接口名)。在③中主要根据服务端口、接口名、接口分组和接口版本构造唯一的key。④：简单从HashMap中取出对应的Exporter并调用Invoker属性值。分析到这里，读者应该能理解RPC调用在服务端处理的逻辑了。

​	Dubb。为了编织这些Handler,适应不同的场景，提供了一套可以定制的线程模型。为了使概念更清晰，我们描述的I/O线程是指底层直接负责读写报文，比如Netty线程池。Dubbo中提供的线程池负责业务方法调用，我们称为业务线程。如果一些事件逻辑可以很快执行完成，比Dubb。为了编织这些Handler,适应不同的场景，提供了一套可以定制的线程模型。为了使概念更清晰，我们描述的I/O线程是指底层直接负责读写报文，比如Netty线程池。Dubbo中提供的线程池负责业务方法调用，我们称为业务线程。如果一些事件逻辑可以很快执行完成，比如做个标记而已，则可以直接在I/O线程中处理。如果事件处理耗时或阻塞，比如读写数据库操作等，则应该将耗时或阻塞的任务转到业务线程池执行。因为I/O线程用于接收请求，如果I/O线程饱和，则不会接收新的请求。

![](https://pic.imgdb.cn/item/60f2eb0d5132923bf85929b0.jpg)

​	在图6-6中，Dispatcher就是线程池派发器。这里需要注意的是，Dispatcher真实的职责是创建具有线程派发能力的 ChannelHandler,比如 AllChannelHandler > MessageOnlyChannel-Handler和ExecutionChannelHandler等，其本身并不具备线程派发能力。

![](https://pic.imgdb.cn/item/60f2eb4d5132923bf85b7dd9.jpg)

​	具体业务方需要根据使用场景启用不同的策略。建议使用默认策略即可，如果在TCP连接中需要做安全加密或校验，则可以使用ConnectionOrderedDispatcher策略。如果引入新的线程池，则不可避免地导致额外的线程切换，用户可在Dubbo配置中指定dispatcher属性让具体策略生效。

### Dubbo 请求响应 Handler

​	在Dubbo框架内部，所有方法调用会被抽象成Request/Response,每次调用（一次会话）都会创建一个请求Request,如果是方法调用则会返回一个Response对象。HeaderExchangeHandler用来处理这种场景，它主要负责以下4种事情。

(1) 更新发送和读取请求时间戳。
(2) 判断请求格式或编解码是否有错，并响应客户端失则的具体原因。
(3) 处理Request请求和Response正常响应。
(4) 支持Telnet调用。

我们首先看一下HeaderExchangeHandler#received实现，如代码清单6-15所示。

![](https://pic.imgdb.cn/item/60f2f0c25132923bf88de348.jpg)

![](https://pic.imgdb.cn/item/60f2f18d5132923bf894f447.jpg)

①：负责响应读取时间并更新时间戳，在Dubbo心跳处理中会使用当前值并判断是否超过空闲时间。②：主要处理事件类型，目前主要处理readonly事件，用于Dubbo优雅停机。当注册中心反注册元数据时，因为网络原因，客户端不能及时感知注册中心事件，服务端会发送readonly报文告知下线。④：处理收到的Response响应，告知业务调用方。⑤：校验客户端不支持Telnet调用，因为只有服务提供方暴露服务才有意义。这里有个小改进，因为客户端支持异步参数回调，但为什么这里不能支持Telnet调用呢？异步参数回调客户端实际上也会暴露一个服务，因此针对这种场景Telnet应该是允许调用的。⑥：触发Telnet调用，并将字符串返回给Telnet客户端，关于Telent调用在前面已经分析过了。

![](https://pic.imgdb.cn/item/60f2f1a05132923bf8959dda.jpg)

![](https://pic.imgdb.cn/item/60f2f1c55132923bf896f265.jpg)

​	在处理请求时，因为在编解码层报错会透传到Handler,所以在①中首先会判断是否是因为请求报文不正确，如果发生错误，则服务端会将具体异常包装成字符串返回，如果直接使用异常对象，则可能造成无法序列化的错误。在②中触发Dubbo协议方法调用，并且把方法调用返回值发送给客户端。如果调用发生未知错误，则会通过③做容错并返回。当发送请求时，会在DefaultFuture中保存请求对象并阻塞请求线程，在④中会唤醒阻塞线程并将Response中的结果通知调用方。

### Dubbo 心跳 Handler

​	Dubbo默认客户端和服务端都会发送心跳报文，用来保持TCP长连接状态。在客户端和服务端，Dubbo内部开启一个线程循环扫描并检测连接是否超时，在服务端如果发现超时则会主动关闭客户端连接，在客户端发现超时则会主动重新创建连接。默认心跳检测时间是60秒，具体应用可以通过heartbeat进行配置。

![](https://pic.imgdb.cn/item/60f2f59a5132923bf8baf8dc.jpg)

![](https://pic.imgdb.cn/item/60f3027e5132923bf837c0c5.jpg)

​	①：遍历所有的Channel,在服务端对应的是所有客户端连接，在客户端对应的是服务端连接。②：主要忽略已经关闭的Socket连接。®：判断当前TCP连接是否空闲，如果空闲就发送心跳报文。目前判断是否是空闲的，根据Channel是否有读或写来决定，比如1分钟内没有读或写就发送心跳报文。④：处理客户端超时重新建立TCP连接，目前的策略是检查是否在3分钟内(用户可以设置)都没有成功接收或发送报文。如果在服务端监测则会通过⑤主动关闭远程客户端连接。

# Dubbo集群容错

集群容错总体实现；

* 普通容错策略的实现；
* Directory的实现原理；
* Router的实现原理；
* LoadBalance的实现原理；
* Merger的实现原理；
* Mock的实现原理。

## Cluster 层概述

​	我们可以把Cluster看作一个集群容错层，该层中包含Cluster、Directory、Router、LoadBalance几大核心接口。注意这里要区分Cluster层和Cluster接口，Cluster层是抽象概念，表示的是对外的整个集群容错层；Cluster是容错接口，提供Failover、Failfast等容错策略。

​	由于Cluster层的实现众多，因此本节介绍的流程是一个基于Abstractclusterinvoker的全量流程，某些实现可能只使用了该流程的一小部分。Cluster的总体工作流程可以分为以下几步:

（1） 生成Invoker对象。不同的Cluster实现会生成不同类型的Clusterinvoker对象并返回。然后调用Clusterinvoker的Invoker方法，正式开始调用流程。

（2） 获得可调用的服务列表。首先会做前置校验，检查远程服务是否已被销毁。然后通过Directory#list方法获取所有可用的服务列表。接着使用Router接口处理该服务列表，根据路由规则过滤一部分服务，最终返回剩余的服务列表。

（3） 做负载均衡。在第2步中得到的服务列表还需要通过不同的负载均衡策略选出一个服务，用作最后的调用。首先框架会根据用户的配置，调用ExtensionLoader获取不同负载均衡策略的扩展点实现（具体负载均衡策略会在后面讲解）。然后做一些后置操作，如果是异步调用则设置调用编号。接着调用子类实现的dolnvoke方法（父类专门留了这个抽象方法让子类实现），子类会根据具体的负载均衡策略选出一个可以调用的服务。

（4） 做RPC调用。首先保存每次调用的Invoker到RPC上下文，并做RPC调用。然后处理调用结果，对于调用出现异常、成功、失败等情况，每种容错策略会有不同的处理方式。7.2节将介绍Cluster接口下不同的容错策略实现。

![](https://pic.imgdb.cn/item/60fe5a765132923bf82e6564.jpg)

## 容错机制的实现

​	Cluster接口一共有9种不同的实现，每种实现分别对应不同的Clusterlnvokero本节会介绍继承了 Abstractclusterinvoker 的 7 种 Clusterinvoker 实现,Merge 和Mock属于特殊机制，会在其他章节讲解。

### 容错机制概述

​	Dubbo容错机制能增强整个应用的鲁棒性，容错过程对上层用户是完全透明的，但用户也可以通过不同的配置项来选择不同的容错机制。每种容错机制又有自己个性化的配置项。Dubbo中现有 Failover> Failfast> Failsafe> Fallback> Forking> Broadcast 等容错机制，容错机制的特性如表7-1所示。

![](https://pic.imgdb.cn/item/60fe5c435132923bf8350419.jpg)

![](https://pic.imgdb.cn/item/60fe5cab5132923bf8367b39.jpg)

​	Cluseter的具体实现:用户可以在`<dubbo :service>><dubbo:reference>><dubbo:consumer>><dubbo:provider>`标签上通过cluster属性设置。

​	对于Failover容错模式，用户可以通过retries属性来设置最大重试次数。可以设置在dubbo: reference标签上，也可以设置在细粒度的方法标签dubbo:method上。

​	对于Forking容错模式，用户可通过forks="ft大并行数”属性来设置最大并行数。假设设置的forks数为们 可用的服务数为v,当n < v时，即可用的服务数大于配置的并行数，则并行请求n个服务；当n>v时，即可用的服务数小于配置的并行数，则请求所有可用的服务v。

​	对于Mergeable容错模式，用可以在dubbo:reference标签中通过merger="true"开启，合并时可以通过group="*"属性指定需要合并哪些分组的结果。默认会根据方法的返回值自动匹配合并器，如果同一个类型有两个不同的合并器实现，则需要在参数中指定合并器的名字（merger-“合并器名“）。例如:用户根据某List类型的返回结果实现了多个合并器，则需要手动指定合并器名称，否则框架不知道要用哪个。如果想调用返回结果的指定方法进行合并（如返回了一个Set,想调用Set#addAll方法），则可以通过merger=" .addAll'1配置来实现。官方Mergeable配置示例如代码清单7-1所示。

![](https://pic.imgdb.cn/item/60fe5fb25132923bf843a0a5.jpg)

### Cluster 接口关系

​	在微服务环境中，可能多个节点同时都提供同一个服务。当上层调用Invoker时，无论实际存在多少个Invoker,只需要通过Cluster层，即可完成整个调用的容错逻辑，包括获取服务列表、路由、负载均衡等，整个过程对上层都是透明的。当然，Cluster接口只是串联起整个逻辑，其中Clusterlnvokep只实现了容错策略部分，其他逻辑则是调用了 Directory、Router >LoadBalance 等接口 实现。

​	容错的接口主要分为两大类，第一类是Cluster类，第二类是Clusterinvoker类。Cluster和Clusterinvoker之间的关系也非常简单：Cluster接口下面有多种不同的实现，每种实现中都需要实现接口的join方法，在方法中会“new”一个对应的Clusterinvoker实现。我们以FailoverCluster实现为例进行说明，如代码清单7-2所示。

![](https://pic.imgdb.cn/item/60fe605b5132923bf846007b.jpg)



​	FailoverCluster是Cluster的其中一种实现，FailoverCluster中直接创建了一个新的FailoverClusterlnvoker 并返回。FailoverClusterlnvoker 继承的接口是Invoker。光看文字描述还是比较难懂。因此，在理解集群容错的详细原理之前，我们先从“上帝视角”看一下整个集群容错的接口关系。Cluster接口的类图关系如图7-2所示。

​	Cluster是最上层的接口，下面一共有9个实现类。Cluster接口上有SPI注解，也就是说，实现类是通过第4章中介绍的扩展机制动态生成的。每个实现类里都只有一个join方法，实现也很简单，直接“new” 一个对应的Clusterinvokero其中AvailableCluster例外，直接使用匿名内部类实现了所有功能。

![](https://pic.imgdb.cn/item/60fe60be5132923bf8475b1c.jpg)

![](https://pic.imgdb.cn/item/60fe60ce5132923bf84791f8.jpg)

​	Invoker 接口是最上层的接口，它下面分别有 AbstractClusterlnvoker> MockClusterlnvoker和 MergeableClusterlnvoker H个类。其中，AbstractClusterlnvoker 是一个抽象类，其封装了通用的模板逻辑，如获取服务列表、负载均衡、调用服务提供者等，并预留了一个dolnvoke方法需要子类自行实现。AbstractClusterlnvoker T面有7个子类，分别实现了不同的集群容错机制。

​	MockClusterlnvoker和MergeableClusterlnvoker由于并不适用于正常的集群容错逻辑，因此没有挂在AbstractClusterlnvoker下面，而是直接继承了 Invoker接口。

​	以上就是容错的接口，Directory、Router和LoadBalance的接口会在对应的章节讲解。

### Failover 策略

​	Cluster 接口上有 SPI 注W@SPI(FailoverCluster.NAME),即默认实现是 Failover。该策略的代码逻辑如下：

(1) 校验。校验从AbstractClusterlnvoker传入的Invoker列表是否为空。

(2) 获取配置参数。从调用URL中获取对应的retries重试次数。

(3) 初始化一些集合和对象。用于保存调用过程中出现的异常、记录调用了哪些节点(这个会在负载均衡中使用，在某些配置下，尽量不要一直调用同一个服务)。

(4) 使用for循环实现重试，for循环的次数就是重试的次数。成功则返回，否则继续循环。如果for循环完，还没有一个成功的返回，则抛出异常，把(3)中记录的信息抛出去。

​	前3步都是做一些校验、数据准备的工作。第4步开始真正的调用逻辑。以下步骤是for循环中的逻辑：

* 校验。如果for循环次数大于1，即有过一次失败，则会再次校验节点是否被销毁、传入的Invoker列表是否为空。
* 负载均衡。调用select方法做负载均衡，得到要调用的节点，并记录这个节点到步骤3的集合里，再把己经调用的节点信息放进RPC上下文中。
* 远程调用。调用invoker#invoke方法做远程调用，成功则返回，异常则记录异常信息，再做下次循环。



​	Failover流程如图7-4所示。

![](https://pic.imgdb.cn/item/60fe61c35132923bf84b1ee7.jpg)

### Failfast 策略

Failfast会在失败后直接抛出异常并返回，实现非常简单，步骤如下：

（1） 校验。校验从AbstractClusterlnvoker传入的Invoker列表是否为空。
（2） 负载均衡。调用select方法做负载均衡，得到要调用的节点。
（3） 进行远程调用。在try代码块中调用invoker#invoke方法做远程调用。如果捕获到异常，则直接封装成RpcException抛出。整个过程非常简短，也不会做任何中间信息的记录。

### Failsafe 策略

Failsafe调用时如果出现异常，则会直接忽略。实现也非常简单，步骤如下：
（1） 校验传入的参数。校验从AbstractClusterlnvoker传入的Invoker列表是否为空。

（2） 负载均衡。调用select方法做负载均衡，得到要调用的节点。

   (3)    远程调用。在try代码块中调用invoker#invoke方法做远程调用，“catch”到任何异
常都直接“吞掉”，返回一个空的结果集。

![](https://pic.imgdb.cn/item/60fe63905132923bf851e631.jpg)

### Fallback 策略

​	Fallback如果调用失败，则会定期重试。FailbackClusterlnvoker里面定义了一个ConcurrentHashMap,专门用来保存失败的调用。另外定义了一个定时线程池，默认每5秒把所有失败的调用拿出来，重试一次。如果调用重试成功，则会从ConcurrentHashMap中移除。dolnvoke的调用逻辑如下：

(1) 校验传入的参数。校验从AbstractClusterlnvoker传入的Invoker列表是否为空。
(2) 负载均衡。调用select方法做负载均衡，得到要调用的节点。
(3) 远程调用。在try代码块中调用invoker#invoke方法做远程调用，“catch”到异常后直接把invocation保存到重试的ConcurrentHashMap中，并返回一个空的结果集。
(4) 定时线程池会定时把ConcurrentHashMap中的失败请求拿出来重新请求，请求成功则从ConcurrentHashMap中移除。如果请求还是失败，则异常也会被“catch”住，不会影响ConcurrentHashMap中后面的重试。

![](https://pic.imgdb.cn/item/60fe64355132923bf854532c.jpg)



### Available 策略

​	Available是找到第一个可用的服务直接调用，并返回结果。步骤如下：

(1) 遍历从AbstractClusterlnvoker传入的Invoker列表，如果Invoker是可用的，则直接调用并返回。

(2) 如果遍历整个列表还没找到可用的Invoker,则抛出异常。

### Broadcast 策略

​	Broadcast会广播给所有可用的节点，如果任何一个节点报错，则返回异常。步骤如下：

​	(1) 前置操作。校验从AbstractClusterlnvoker传入的Invoker列表是否为空；在RPC上下文中设置Invoker列表；初始化一些对象，用于保存调用过程中产生的异常和结果信息等。

​	(2) 循环遍历所有Invoker,直接做RPC调用。任何一个节点调用出错，并不会中断整个广播过程，会先记录异常，在最后广播完成后再抛出。如果多个节点异常，则只有最后一个节点的异常会被抛出，前面的异常会被覆盖。

### Forking 策略

​	Forking可以同时并行请求多个服务，有任何一个返回，则直接返回。相对于其他调用策略，Forking的实现是最复杂的。其步骤如下：

(1) 准备工作。校验传入的Invoker列表是否可用；初始化一个Invoker集合，用于保存真正要调用的Invoker列表；从URL中得到最大并行数、超时时间。

(2) 获取最终要调用的Invoker列表。假设用户设置最大的并行数为n 实际可以调用的最大服务数为v。如果n<0或n>v,则说明可用的服务数小于用户的设置，因此最终要调用的Invoker H能有v个；如果n<v,则会循环调用负载均衡方法，不断得到可调用的Invoker,加入步骤1中的Invoker集合里。

​	这里有一点需要注意：在Invoker加入集合时，会做去重操作。因此，如果用户设置的负载均衡策略每次返回的都是同一个Invoker,那么集合中最后只会存在一个Invoker,也就是只会调用一个节点。

(3) 调用前的准备工作。设置要调用的Invoker列表到RPC上下文；初始化一个异常计数器；初始化一个阻塞队列，用于记录并行调用的结果。

(4) 执行调用。循环使用线程池并行调用，调用成功，则把结果加入阻塞队列；调用失败，则失败计数+1。如果所有线程的调用都失败了，即失败计数>所有可调用的Invoker时，则把异常信息加入阻塞队列。

​	这里有一点需要注意：并行调用是如何保证个别调用失败不返回异常信息，只有全部失败才返回异常信息的呢？因为有判断条件，当失败计数N所有可调用的Invoker时，才会把异常信息放入阻塞队列，所以只有当最后一个Invoker也调用失败时才会把异常信息保存到阻塞队列，从而达到全部失败才返回异常的效果。

(5) 同步等待结果。由于步骤4中的步骤是在线程池中执行的，因此主线程还会继续往下执行，主线程中会使用阻塞队列的poll(-超时时间“)方法，同步等待阻塞队列中的第一个结果，如果是正常结果则返回，如果是异常则抛出。

​	从上面步骤可以得知,Forking的超时是通过在阻塞队列的poll方法中传入超时时间实现的；线程池中的并发调用会获取第一个正常返回结果。只有所有请求都失败了，Forking才会失败。Forking调用流程如图7-5所示。

![](https://pic.imgdb.cn/item/60fe66c15132923bf860b941.jpg)

## Directory 的实现

​	整个容错过程中首先会使用Directory#list来获取所有的Invoker列表。Directory也有多种实现子类，既可以提供静态的Invoker列表，也可以提供动态的Invoker列表。静态列表是用户自己设置的Invoker列表；动态列表根据注册中心的数据动态变化，动态更新Invoker列表的数据，整个过程对上层透明

### 总体实现

![](https://pic.imgdb.cn/item/60fe68d45132923bf867f03b.jpg)

​	又是熟悉的“套路”，使用了模板模式。Directory是顶层的接口。AbstractDirectory封装了通用的实现逻辑。抽象类包含RegistryDirectory和StaticDirectory两个子类。下面分别介绍每个类的职责和工作：

(1) AbstractDirectoryo封装了通用逻辑，主要实现了四个方法：检测Invoker是否可用，销毁所有Invoker, list方法，还留了一个抽象的doList方法给子类自行实现。list方法是最主要的方法，用于返回所有可用的list,逻辑分为两步：

* 调用抽象方法doList获取所有Invoker列表，不同子类有不同的实现；
* 遍历所有的router,进行Invoker的过滤，最后返回过滤好的Invoker列表。



doList抽象方法则是返回所有的Invoker列表，由于是抽象方法，子类继承后必须要有自己的实现。

(2) RegistryDirectoryo属于Directory的动态列表实现，会自动从注册中心更新Invoker列表、配置信息、路由列表。

(3) StaticDirectoryo Directory的静态列表实现，即将传入的Invoker列表封装成静态的Directory对象，里面的列表不会改变。因为Cluste「#join(Directopy〈T> directory)方法需要传入Directory对象，因此该实现主要使用在一些上层已经知道自己要调用哪些Invoker,只需要包装一个 Directory对象返回即可的场景。在ReferenceConfig#createProxy和RegistryDirectory#toMergeMethodInvokerMap 中使用了 Cluster#join 方法。StaticDirectory 的逻辑非常简单，在构造方法中需要传入Invoker列表，doList方法则直接返回初始化时传入的列表。因此，不再详细说明。

### RegistryDirectory 的实现

​	RegistryDirectory中有两条比较重要的逻辑线，第一条，框架与注册中心的订阅，并动态更新本地Invoker列表、路由列表、配置信息的逻辑；第二条，子类实现父类的doList方法。

**1.订阅与动态更新**

​	我们先看一下订阅和动态更新逻辑。这个逻辑主要涉及subscribe、notify、refreshinvoker三个方法，其余是一些数据转换的辅助类方法，如toConfigurators、toRouters。

​	subscribe是订阅某个URL的更新信息。Dubbo在引用每个需要RPC调用的Bean的时候，会调用directory. subscribe来订阅这个Bean的各种URL的变化(Bean的配置在配置中心中都是以URL的形式存放的)。这个方法比较简单，只有两行代码，仅仅使用registry.subscribe订阅。订阅发布在前面章节已经有了详细的讲解，因此不在此展开。

​	notify就是监听到配置中心对应的URL的变化，然后更新本地的配置参数。监听的URL分为三类：配置configurators>路由规则router> Invoker列表。工作流程如下：

​	(1) 新建三个List,分别用于保存更新的Invoker URL、路由配置URL、配置URL。遍历监听返回的所有URL,分类后放入三个List中。

​	(2) 解析并更新配置参数。

* 对于router类参数，首先遍历所有router类型的URL,然后通过Router工厂把每个URL包装成路由规则，最后更新本地的路由信息。这个过程会忽略以empty开头的URL。
* 对于Configurator类的参数，管理员可以在dubbo-admin动态配置功能上修改生产者的参数，这些参数会保存在配置中心的configurators类目下。notify监听到URL配置参数的变化，会解析并更新本地的Configurator配置。
* 对于Invoker类型的参数，如果是empty协议的URL,则会禁用该服务，并销毁本地缓存的Invoker；如果监听到的Invoker类型URL都是空的，则说明没有更新，直接使用本地的老缓存；如果监听到的Invoker类型URL不为空，则把新的URL和本地老的URL合并，创建新的Invoker,找出差异的老Invoker并销毁。

![](https://pic.imgdb.cn/item/60fe6baf5132923bf8725846.jpg)

​	dubbo-admin上更新路由规则或参数是通过“override://”协议实现的，dubbo-admin的使用方法可以查看官方文档。override协议的URL会覆盖更新本地URL中对应的参数。如果是“empty://"协议的URL,则会清空本地的配置，这里会调用Configurator接口来实现该功能。override参数示例如图7-8所示，如果有其他的参数需要覆盖，则直接加在URL上即可。

![](https://pic.imgdb.cn/item/60fe81fd5132923bf8cceee6.jpg)



**2. doList的实现**

​	notify中更新的Invoker列表最终会转化为一个字典`Map<Stringj List<Invoker<T>>>methodlnvokerMap`。 key是对应的方法名称，value是整个Invoker列表。doList的最终目标就是在字典里匹配出可以调用的Invoker列表，并返回给上层。其主要步骤如下：

(1)检查服务是否被禁用。如果配置中心禁用了某个服务，则该服务无法被调用。如果服务被禁用则会抛出异常。

(2)根据方法名和首参数匹配Invokero这是一个比较奇特的特性。根据方法名和首参数查找对应的Invoker列表，暂时没看到相关的应用场景。首参数匹配Invoker使用示例如代码清单7-5所示。



![](https://pic.imgdb.cn/item/60ff79945132923bf840f15f.jpg)

如果在这一步没有匹配到Invoker列表，则进入第3步。
(3)根据方法名匹配Invokero以方法名为key去methodlnvokerMap中匹配Invoker列表,如果还是没有匹配到，则进入第4步。
(4)根据"*”匹配Invoker>用星号去匹配Invoker列表，如果还没有匹配到，则进入最后一步兜底操作。
(5)遍历methodlnvokerMap,找到第一个Invoker列表返回。如果还没有，则返回一个空列表。

## 路由的实现

​	通过7.3节的Directory获取所有Invoker列表的时候，就会调用到本节的路由接口。路由接口会根据用户配置的不同路由策略对Invoker列表进行过滤，只返回符合规则的Invoker。例如：如果用户配置了接口 A的所有调用，都使用IP为192.168.1.22的节点，则路由会过滤其他的 Invoker,只返回 IP 为 192.168.1.22 的 Invoker。

### 路由的总体结构

​	路由分为条件路由、文件路由、脚本路由，对应dubbo-admin中三种不同的规则配置方式。条件路由是用户使用Dubbo定义的语法规则去写路由规则;文件路由则需要用户提交一个文件,里面写着对应的路由规则，框架基于文件读取对应的规则；脚本路由则是使用JDK自身的脚本引擎解析路由规则脚本，所有JDK脚本引擎支持的脚本都能解析，默认是JavaScript。我们先来看一下接口之间的关系，如图7.9所示。

![](https://pic.imgdb.cn/item/60ff9ee85132923bf8dacfa5.jpg)

​	RouterFactory是一个SPI接口，没有设置默认值，但由于有@Adaptive("protocol")注解，因此它会根据URL中的protocol参数确定要初始化哪一个具体的Router实现。RouterFactory源码如代码清单7-6所示。

![](https://pic.imgdb.cn/item/60ff9f275132923bf8dbe13a.jpg)

​	我们在SPI的配置文件中能看到，URL中的protocol可以设置file、script、condition三种值，分别寻找对应的实现类，如代码清单7-7所示。

![](https://pic.imgdb.cn/item/60ff9f4e5132923bf8dc89ce.jpg)

​	RouterFactory的实现类也非常简单，就是直接“new” 一个对应的Router并返回。例如：ConditionRouterFactory 直接"new” 并返回一个 ConditionRouter。当然，FileRouterFactory除外，直接在工厂类中实现了所有逻辑。

### 条件路由的参数规则

​	条件路由使用的是 condition://协议，URL 形式是”condition:// 0.0.0.0/com.fbo.BarService?category=routers&dynamic=false&rule=n + URL.encode(nhost = 10.20.153.10 => host = 10.20.153.11")。我们可以看到，最后的路由规则会用URL.encode进行编码。下面来看一下官方文档中对每个参数含义的说明，如表7-2所示。

![](https://pic.imgdb.cn/item/60ff9ff55132923bf8df6e2b.jpg)

![](https://pic.imgdb.cn/item/60ffa0495132923bf8e0ddef.jpg)

![](https://pic.imgdb.cn/item/60ffa0775132923bf8e1a931.jpg)



* 这条配置说明所有调用find开头的方法都会被路由到IP为192.168.1.22的服务节点上。
* =＞之前的部分为消费者匹配条件，将所有参数和消费者的URL进行对比，当消费者满足匹配条件时，对该消费者执行后面的过滤规则。
* =＞之后的部分为提供者地址列表的过滤条件，将所有参数和提供者的URL进行对比,消费者最终只获取过滤后的地址列表。
* 如果匹配条件为空，则表示应用于所有消费方，如=＞ host != 192.168.1.22。
* 如果过滤条件为空，则表示禁止访问，如host = 192.168.1.22 =＞。



​	整个规则的表达式支持$protocol等占位符方式，也支持=、！=等条件。值可以支持多个，用逗号分隔，如host = 192.168.1.22,192.168.1.23;如果以`*`号结尾，则说明是通配符，如host = `192.168.1.*`表示匹配192.168.1.网段下所有的IP。

### 条件路由的实现

​	条件路由的具体实现类是ConditionRouter, Dubbo会根据自定义的规则语法来实现路由规则。我们主要需要关注其构造方法和实现父类接口的route方法。

**1. ConditionRouter构造方法的逻辑**

​	ConditionRouterFactory在初始化ConditionRouter的时候，其构造方法中含有规则解析的逻辑。步骤如下：

​	(1) 根据URL的键rule获取对应的规则字符串。以=>为界，把规则分成两段，前面部分为whenRule,即消费者匹配条件；后面部分为thenRule,即提供者地址列表的过滤条件。我们以代码清单7-8的规则为例，其会被解析为whenRule: method = find*和thenRule: host =192.168.1.22。

​	(2) 分别解析两个路由规则。调用parseRule方法，通过正则表达式不断循环匹配whenRule和thenRule字符串。解析的时候，会根据key-value之间的分隔符对key-value做分类(如果A=B,则分隔符为=)，支持的分隔符形式有：A=B、A&B、A!=B、A,B这4种形式。最终参数都会被封装成一个个MatchPair对象，放入Map中保存。Map的key是参数值，value是MatchPair对象。若以代码清单7-8的规则为例，则会生成以method为key的when Map,以host为key的 then Mapo value 则分别是包装了 find*和 192.168.1.22 的 MatchPair 对象。

​	MatchPair对象是用来做什么的呢？这个对象一共有两个作用。第一个作用是通配符的匹配和占位符的赋值。MatchPair对象是内部类，里面只有一个isMatch方法，用于判断值是否能匹配得上规则。规则里的$、`*`等通配符都会在MatchPair对象中进行匹配。其中$支持protoco、username、 password、host、port、path这几个动态参数的占位符。例如：规则中写了$protocol,则会自动从URL中获取protocol的值，并赋值进去。第二个作用是缓存规则。MatchPair对象中有两个Set集合，一个用于保存匹配的规则，如=`find*`；另一个则用于保存不匹配的规则，如!=`find*`。这两个集合在后续路由规则匹配的时候会使用到。

**2. route方法的实现原理**

   ConditionRouter继承了 Router接口，需要实现接口的route方法。该方法的主要功能是过滤出符合路由规则的Invoker列表，即做具体的条件匹配判断，其步骤如下：

​	(1) 校验。如果规则没有启用，则直接返回；如果传入的Invoker列表为空，则直接返回空；如果没有任何的whenRule匹配，即没有规则匹配，则直接返回传入的Invoker列表；如果whenRule有匹配的，但是thenRule为空，即没有匹配上规则的Invoker,则返回空。

​	(2) 遍历Invoker列表，通过thenRule找出所有符合规则的Invoker加入集合。例如：匹配规则中的method名称和当前URL中的method是不是相等。

​	(3) 返回结果。如果结果集不为空，则直接返回；如果结果集为空，但是规则配置了force=true,即强制过滤，那么就会返回空结果集；非强制则不过滤，即返回所有Invoker列表。

### 文件路由的实现

​	文件路由是把规则写在文件中，文件中写的是自定义的脚本规则，可以是JavaScript, Groovy等，URL中对应的key值填写的是文件的路径。文件路由主要做的就是把文件中的路由脚本读出来，然后调用路由的工厂去匹配对应的脚本路由做解析。由于逻辑比较简单，我们直接看代码清单7-9。

![](https://pic.imgdb.cn/item/60ffd9e35132923bf8cb3420.jpg)

### 脚本路由的实现

​	脚本路由使用JDK自带的脚本解析器解析脚本并运行，默认使用JavaScript解析器，其逻辑分为构造方法和route方法两大部分。构造方法主要负责一些初始化的工作，route方法则是具体的过滤逻辑执行的地方。我们先来看一段官方文档中的JavaScript脚本，如代码清单7-10所示。

![](https://pic.imgdb.cn/item/610009735132923bf85e82a6.jpg)

​	我们在写JavaScript脚本的时候需要注意，一个服务只能有一条规则，如果有多条规则，并且规则之间没有交集，则会把所有的Invoker都过滤。另外，脚本路由中也没看到沙箱约束,因此会有注入的风险。

​	下面我们来看一下脚本路由的构造方法逻辑：

(1) 初始化参数。获取规则的脚本类型、路由优先级。如果没有设置脚本类型，则默认设置为JavaScript类型，如果没有解析到任何规则，则抛出异常。
(2) 初始化脚本执行引擎。根据脚本的类型，通过Java的ScriptEngineManager创建不同的脚本执行器，并缓存起来。

​	route方法的核心逻辑就是调用脚本引擎，获取执行结果并返回。主要是JDK脚本引擎相关的知识，不会涉及具体的过滤逻辑，因为逻辑己经下沉到用户自定义的脚本中，如代码清单7-11所示。

![](https://pic.imgdb.cn/item/61000d965132923bf8703f74.jpg)

## 负载均衡的实现

​	在整个集群容错流程中，首先经过Directory获取所有Invoker列表，然后经过Router根据路由规则过滤Invoker,最后幸存下来的Invoker还需要经过负载均衡这一关，选出最终要调用的 Invoker。

### 包装后的负载均衡

​	7.2节介绍了 7种容错策略，发现在很多容错策略中都会使用负载均衡方法，并且所有的容错策略中的负载均衡都使用了抽象父类Abstractclusterinvoker中定义的`Invoker<T> select`方法，而并不是直接使用LoadBalance方法。因为抽象父类在LoadBalance的基础上又封装了一些新的特性：

​	(1) 粘滞连接。Dubbo中有一种特性叫粘滞连接，以下内容摘自官方文档：

​	粘滞连接用于有状态服务，尽可能让客户端总是向同一提供者发起调用，除非该提供者“挂了”，再连接另一台。

​	粘滞连接将自动开启延迟连接，以减少长连接数。
`<dubbo:protocol name=Hdubbo" sticky="true" />`

​	(2) 可用检测。Dubbo调用的URL中，如果含有cluster.availablecheck=false,则不会检测远程服务是否可用，直接调用。如果不设置，则默认会开启检查，对所有的服务都做是否可用的检查，如果不可用，则再次做负载均衡。

​	(3) 避免重复调用。对于已经调用过的远程服务，避免重复选择，每次都使用同一个节点。这种特性主要是为了避免并发场景下，某个节点瞬间被大量请求。

​	整个逻辑过程大致可以分为4步：

​	(1) 检查URL中是否有配置粘滞连接，如果有则使用粘滞连接的Invoker。如果没有配置粘滞连接，或者重复调用检测不通过、可用检测不通过，则进入第2步。

​	(2) 通过ExtensionLoader获取负载均衡的具体实现，并通过负载均衡做节点的选择。对选择出来的节点做重复调用、可用性检测，通过则直接返回，否则进入第3步。

​	(3) 进行节点的重新选择。如果需要做可用性检测，则会遍历Directory中得到的所有节点，过滤不可用和已经调用过的节点，在剩余的节点中重新做负载均衡；如果不需要做可用性检测，那么也会遍历Directory中得到的所有节点，但只过滤已经调用过的，在剩余的节点中重新做负载均衡。这里存在一种情况，就是在过滤不可用或已经调用过的节点时，节点全部被过滤，没有剩下任何节点，此时进入第4步。

​	(4) 遍历所有已经调用过的节点，选出所有可用的节点，再通过负载均衡选出一个节点并返回。如果还找不到可调用的节点，则返回null。

​	从上述逻辑中，我们可以得知，框架会优先处理粘滞连接。否则会根据可用性检测或重复调用检测过滤一些节点，并在剩余的节点中做负载均衡。如果可用性检测或重复调用检测把节点都过滤了，则兜底的策略是：在己经调用过的节点中通过负载均衡选择出一个可用的节点。

​	以上就是封装过的负载均衡的实现，下面讲解原始的LoadBalance是如何实现的。

### 负载均衡的总体结构

​	Dubbo现在内置了 4种负载均衡算法，用户也可以自行扩展，因为LoadBalance接口上有@SPI注解，如代码清单7.12所示。

![](https://pic.imgdb.cn/item/610014365132923bf88aa601.jpg)

​	从代码中我们可以知道默认的负载均衡实现就是RandomLoadBalance,即随机负载均衡。由于select方法上有@Adaptive ("loadbalance")注解，因此我们在URL中可以通过loadbalance=xxx来动态指定select时的负载均衡算法。下面我们来看一下官方文档中对所有负载均衡算法的说明，如表7-3所示。

![](https://pic.imgdb.cn/item/6100148f5132923bf88c0d4d.jpg)

​	4种负载均衡算法都继承自同一个抽象类，使用的也是模板模式，抽象父类中己经把通用的逻辑完成，留了一个抽象的doSelect方法给子类实现。负载均衡的接口关系如图7-10所示。

![](https://pic.imgdb.cn/item/6100153e5132923bf88ee04e.jpg)

​	抽象父类AbstractLoadBalance有两个权重相关的方法：calculateWarmupWeight和getWeighto getWeight方法就是获取当前Invoker的权重，calculateWarmupWeight是计算具体的权重。getWeight方法中会调用calculateWarmupWeight,如代码清单7-13所示。

![](https://pic.imgdb.cn/item/610016905132923bf894683b.jpg)

​	calculateWarmupWeight的计算逻辑比较简单，由于框架考虑了服务刚启动的时候需要有一个预热的过程，如果一启动就给予100%的流量，则可能会让服务崩溃，因此实现了calculateWarmupWeight方法用于计算预热时候的权重，计算逻辑是：(启动至今时间/给予的预热总时间)X权重。例如：假设我们设置A服务的权重是5,让它预热10分钟，则第一分钟的时候，它的权重变为(1/10) X5 = 0.5, 0.5/5 = 0.1,也就是只承担10%的流量；10分钟后，权重就变为(10/10) X5 = 5,也就是权重变为设置的100%,承担了所有的流量。

​	抽象父类的select方法是进行具体负载均衡逻辑的地方，这里只是做了一些判断并调用需要子类实现的doSelect方法，因此不再赘述。下面直接看一下不同子类实现doSelect的方式。

### Random负载均衡

​	Random负载均衡是按照权重设置随机概率做负载均衡的。这种负载均衡算法并不能精确地平均请求，但是随着请求数量的增加，最终结果是大致平均的。它的负载计算步骤如下：

​	(1) 计算总权重并判断每个Invoker的权重是否一样。遍历整个Invoker列表，求和总权重。在遍历过程中，会对比每个Invoker的权重，判断所有Invoker的权重是否相同。

​	(2) 如果权重相同，则说明每个Invoker的概率都一样，因此直接用nextlnt随机选一个Invoker返回即可。

​	(3) 如果权重不同，则首先得到偏移值，然后根据偏移值找到对应的Invoker,如代码清单7-14所示。

![](https://pic.imgdb.cn/item/610017845132923bf898767b.jpg)

​	看源码可能还没理解原理，下面做一个场景假设：假设有4个Invoker,它们的权重分别是1、2、3、4,则总权重是 1 +2+3+4=10。说明每个 Invoker 分别有 1/10、2/10、3/10、4/10 的概率会被选中。然后nextlnt(10)会返回0〜10之间的一个整数，假设为5。如果进行累减,则减到3后会小于0,此时会落入3的区间，即选择3号Invoker,如图7-11所示。

![](https://pic.imgdb.cn/item/610019285132923bf89f7c8c.jpg)

### RoundRobin 负载均衡

​	权重轮询负载均衡会根据设置的权重来判断轮询的比例。普通轮询负载均衡的好处是每个节点获得的请求会很均匀，如果某些节点的负载能力明显较弱，则这个节点会堆积比较多的请求。因此普通的轮询还不能满足需求，还需要能根据节点权重进行干预。权重轮询又分为普通权重轮询和平滑权重轮询。普通权重轮询会造成某个节点会突然被频繁选中，这样很容易突然让一个节点流量暴增。Nginx中有一种叫平滑轮询的算法(smooth weighted round-robin balancing),这种算法在轮询时会穿插选择其他节点，让整个服务器选择的过程比较均匀，不会“逮住”一个节点一直调用。Dubbo框架中最新的RoundRobin代码已经改为平滑权重轮询算法。

​	我们先来看一下Dubbo中RoundRobin负载均衡的工作步骤，如下：

​	(1 )初始化权重缓存Map。以每个Invoker的URL为key,对象WeightedRoundRobin为value生成一个 ConcurrentMap,并把这个 Map 保存到全局的 methodWeightMap 中：`ConcurrentMap<String, ConcurrentMap<StringJ WeightedRoundRobin>> `methodWeightMap。methodWeightMap
的key是每个接口 +方法名。这一步只会生成这个缓存Map,但里面是空的，第2步才会生成每个Invoker对应的键值。

​	WeightedRoundRobin封装了每个Invoker的权重，对象中保存了三个属性，如代玛清单7-15所示。

![](https://pic.imgdb.cn/item/61001bf75132923bf8ac1300.jpg)

​	(2) 遍历所有Invoker。首先，在遍历的过程中把每个Invoker的数据填充到第1步生成的权重缓存Map中。其次，获取每个Invoker的预热权重，新版的框架RoundRobin也支持预热，通过和Random负载均衡中相同的方式获得预热阶段的权重。如果预热权重和Invoker设置的权重不相等，则说明还在预热阶段，此时会以预热权重为准。然后，进行平滑轮询。每个Invoker会把权重加到自己的current属性上，并更新当前Invoker的lastUpdate。同时累加每个Invoker的权重到totalweighto最终，遍历完后，选出所有Invoker中current最大的作为最终要调用的节点。

​	(3) 清除已经没有使用的缓存节点。由于所有的Invoker的权重都会被封装成一个weightedRoundRobin对象，因此如果可调用的Invoker列表数量和缓存weightedRoundRobin对象的Map大小不相等，则说明缓存Map中有无用数据（有些Invoker己经不在了，但Map中还有缓存）。

​	为什么不相等就说明有老数据呢？如果Invoker列表比缓存Map大，则说明有没被缓存的Invoker,此时缓存Map会新增数据。因此缓存Map永远大于等于Invoker列表。

​	清除老旧数据时，各线程会先用CAS抢占锁（抢到锁的线程才做清除操作，抢不到的线程就直接跳过，保证只有一个线程在做清除操作），然后复制原有的Map到一个新的Map中，根据lastupdate清除新Map中的过期数据（默认60秒算过期），最后把Map从旧的Map引用修改到新的Map上面。这是一种CopyOnWrite的修改方式。

​	（4）返回Invoker。注意，返回之前会把当前Invoker的current减去总权重。这是平滑权重轮询中重要的一步。

​	看了这个逻辑，很多读者可能也没明白整个轮询过程，因为穿插了框架逻辑。因此这里把算法逻辑提取出来：

（1） 每次请求做负载均衡时，会遍历所有可调用的节点（Invoker列表）。对于每个Invoker,让它的current = current + weight。属性含义见weightedRoundRobin对象。同时累加每个Invoker 的 weight 到 totalWeight,即 totalweight = totalweight + weight。

（2） 遍历完所有Invoker后，current值最大的节点就是本次要选择的节点。最后，把该节点的 current 值减去 totalWeight,即 current = current - totalweight。

假设有3个Invoker： A、B、C,它们的权重分别为1、6、9,初始current都是0,则平滑权重轮询过程如表7-4所示。

![](https://pic.imgdb.cn/item/61001dcb5132923bf8b4f71e.jpg)

![](https://pic.imgdb.cn/item/61001e1e5132923bf8b6868e.jpg)

​	从这16次的负载均衡来看，我们可以清楚地得知，A刚好被调用了 1次，B刚好被调用了6次，C刚好被调用了 9次。符合权重轮询的策略，因为它们的权重比是1 ： 6 ： 90此外，C并没有被频繁地一直调用，其中会穿插B和A的调用。至于平滑权重轮询的数学原理，就不在本书讨论的范围内了，感兴趣的读者可以去进一步了解。

### LeastActive 负载均衡

​	LeastActive负载均衡称为最少活跃调用数负载均衡，即框架会记下每个Invoker的活跃数，每次只从活跃数最少的Invoker里选一个节点。这个负载均衡算法需要配合ActiveLimitFilter过滤器来计算每个接口方法的活跃数。最少活跃负载均衡可以看作Random负载均衡的“加强版”，因为最后根据权重做负载均衡的时候，使用的算法Random的一样。我们现在配合一些代码来看一下具体的运行逻辑，不重要的代码直接使用省略，如代码清单7-16所示。

![](https://pic.imgdb.cn/item/610167385132923bf8bbc5d0.jpg)

​	从代码清单7-16中我们可以得知其逻辑：遍历所有Invoker,不断寻找最小的活跃数(leastActive),如果有多个Invoker的活跃数都等于least Active,则把它们保存到同一个集合中，最后在这个Invoker集合中再通过随机的方式选出一个Invoker。

​	那最少活跃的计数又是如何知道的呢？

​	在ActiveLimitFilter中，只要进来一个请求，该方法的调用的计数就会原子性+1。整个Invoker调用过程会包在try-catch-finally中，无论调用结束或出现异常，finally中都会把计数原子-1。该原子计数就是最少活跃数。

### 一致性Hash负载均衡

​	一致性Hash负载均衡可以让参数相同的请求每次都路由到相同的机器上。这种负载均衡的方式可以让请求相对平均，相比直接使用Hash而言，当某些节点下线时，请求会平摊到其他服务提供者，不会引起剧烈变动。普通一致性Hash的简单示例如图7-12所示。

![](https://pic.imgdb.cn/item/610168e55132923bf8c32f97.jpg)

​	普通一致性Hash会把每个服务节点散列到环形上，然后把请求的客户端散列到环上，顺时针往前找到的第一个节点就是要调用的节点。假设客户端落在区域2,则顺时针找到的服务C就是要调用的节点。当服务C宕机下线，则落在区域2部分的客户端会自动迁移到服务D上。这样就避免了全部重新散列的问题。

​	普通的一致性Hash也有一定的局限性，它的散列不一定均匀，容易造成某些节点压力大。因此Dubbo框架使用了优化过的Ketama 一致性Hash。这种算法会为每个真实节点再创建多个虚拟节点，让节点在环形上的分布更加均匀，后续的调用也会随之更加均匀。

​	下面我们来看下一致性Hash的实现原理，如代码清单7-17所示。

![](https://pic.imgdb.cn/item/61016b565132923bf8ce651d.jpg)

整个逻辑的核心在ConsistentHashSelector中，因此我们继续来看ConsistentHashSelector是如何初始化的。ConsistentHashSelector初始化的时候会对节点进行散列，散列的环形是使用一个TreeMap实现的，所有的真实、虚拟节点都会放入TreeMapo把节点的IP+递增数字做“MD5”，以此作为节点标识，再对标识做“Hash”得到TreeMap的key,最后把可以调用的节点作为TreeMap的value,如代码清单7-18所示。

![](https://pic.imgdb.cn/item/61016dc45132923bf8d9acc3.jpg)

​	TreeMap实现一致性Hash：在客户端调用时候，只要对请求的参数也做“MD5”即可。虽然此时得到的MD5值不一定能对应到TreeMap中的一个key,因为每次的请求参数不同。但是由于TreeMap是有序的树形结构，所以我们可以调用TreeMap的ceilingEntry方法，用于返回一个至少大于或等于当前给定key的Entry,从而达到顺时针往前找的效果。如果找不到，则使用firstEntry返回第一个节点。

## Merger的实现

​	当一个接口有多种实现，消费者又需要同时引用不同的实现时，可以用group来区分不同的实现，如下所示。

![](https://pic.imgdb.cn/item/61016f5a5132923bf8e0dfa4.jpg)

​	框架中有一些默认的合并实现。Merger接口上有@SPI注解，没有默认值，属于SPI扩展点。用户可以基于Merger扩展点接口实现自己的自定义类型合并器。本节主要介绍现有抽象逻辑及已有实现。

### 总体结构

​	MergerCluster也是Cluster接口的一种实现，因此也遵循Cluster的设计模式，在invoke方法中完成具体逻辑。整个过程会使用Merger接口的具体实现来合并结果集。在使用的时候，通过MergerFactory获得各种具体的Merger实现oMerger的12种默认实现的关系如图7-14所示。

![](https://pic.imgdb.cn/item/61016fd85132923bf8e30b85.jpg)

​	如果开启了 Merger特性，并且未指定合并器(Merger的具体实现)，则框架会根据接口的返回类型自动匹配合并器。我们可以扩展属于自己的合并器，MergerFactory在加载具体实现的时候，会用ExtensionLoader把所有SPI的实现都加载到缓存中。后续使用时直接从缓存中读取，如果读不到则会重新全量加载一次SPIo内置的合并我们可以分为四类：Array、Set、List、Map,实现都比较简单，我们只列举MapMerger的实现，如代码清单7-19所示。

![](https://pic.imgdb.cn/item/610170455132923bf8e4f091.jpg)

​	整个实现的思路就是，在Merge中新建了一个Map,把返回的多个Map合并成一个。其他类型的合并器实现都是类似的，因此不再赘述。

### MergeableClusterlnvoker 机制

​	MergeableClusterlnvoker串起了整个合并器逻辑，在讲解MergeableClusterlnvoker的机制之前，我们先回顾一下整个调用的过程：MergeableCluster#join方法中直接生成并返回了MergeableClusterlnvoker, MergeableClusterInvoker#invoke 方法又通过 MergerFactory 工厂获取不同的Merger接口实现，完成了合并的具体逻辑。

​	MergeableCluster并没有继承抽象的Cluster实现，而是独立完成了自己的逻辑。因此，它的整个逻辑和之前的Failover等机制不同，其步骤如下：

​	(1) 前置准备。通过directory获取所有Invoker列表。

​	(2) 合并器检查。判断某个方法是否有合并器，如果没有，则不会并行调用多个group,找到第一个可以调用的Invoker直接调用就返回了。如果有合并器，则进入第3步。

​	(3) 获取接口的返回类型。通过反射获得返回类型，后续要根据这个返回值查找不同的合并器。

​	(4) 并行调用。把Invoker的调用封装成一个个Callable对象，放到线程池中执行，保存
线程池返回的future对象到HashMap中，用于等待后续结果返回。

​	(5) 等待fixture对象的返回结果。获取配置的超时参数，遍历(4)中得到的fixture对象，设置Future#get的超时时间，同步等待得到并行调用的结果。异常的结果会被忽略，正常的结果会被保存到list中。如果最终没有返回结果，则直接返回一个空的RpcResult；如果只有一个结果，那么也直接返回，不需要再做合并；如果返回类型是void,则说明没有返回值，也直接返回。

​	(6) 合并结果集。如果配置的是merger=".addAll",则直接通过反射调用返回类型中的.addAll方法合并结果集。例如：返回类型是Set,则调用Set.addAll来合并结果，如代码清单7.20所示。

![](https://pic.imgdb.cn/item/610171ae5132923bf8eb74a3.jpg)

![](https://pic.imgdb.cn/item/610171bb5132923bf8ebb4f6.jpg)

​	对于要调用合并器来合并的结果集，则使用以下逻辑，如代码清单7.21所示。

![](https://pic.imgdb.cn/item/610171ec5132923bf8ec947b.jpg)

## Mock

​	在Cluster中，还有最后一个MockClusterWrapper,由它实现了 Dubbo的本地伪装。这个功能的使用场景较多，通常会应用在以下场景中：服务降级；部分非关键服务全部不可用，希望主流程继续进行；在下游某些节点调用异常时，可以以Mock的结果返回。

### Mock常见的使用方式

​	Mock只有在拦截到RpcException的时候会启用，属于异常容错方式的一种。业务层面其实也可以用try-catch来实现这种功能，如果使用下沉到框架中的Mock机制，则可以让业务的实现更优雅。常见配置如下：
![](https://pic.imgdb.cn/item/6101731a5132923bf8f21c86.jpg)

​	当接口配置了 Mock,在RPC调用抛出RpcException时就会执行Mock方法。最后一种return null的配置方式通常会在想直接忽略异常的时候使用。

​	服务的降级是在dubbo-admin中通过override协议更新Invoker的Mock参数实现的。如果Mock参数设置为mock=force: return+null,则表明是强制Mock,强制Mock会让消费者对该服务的调用直接返回null,不再发起远程调用。通常使用在非重要服务己经不可用的时候，可以屏蔽下游对上游系统造成的影响。此外，还能把参数设置为印ock=fail:retupn+null,这样消费者还是会发起远程调用，不过失败后会返回null,但是不抛出异常。

​	最后，如果配置的参数是以throw开头的，即mock= throw,则直接抛出RpcException,不会发起远程调用。

### Mock的总体结构

​	Mock涉及的接口比较多,整个流程贯穿Cluster和Protocol层，接口之间的逻辑关系如图7-15所示。

![](https://pic.imgdb.cn/item/610174255132923bf8f719ed.jpg)

​	从图7-15我们可以得知，主要流程分为Cluster层和Protocol层。

* MockClusterWrapper是一个包装类，包装类会被自动注入合适的扩展点实现，它的逻辑很简单，只是把被包装扩展类作为初始化参数来创建并返回一个MockClusterlnvoker,因此本节就不再详细讲解。

* MockClusterlnvoker和其他的Clusterinvoker 一样,在Invoker方法中完成了主要逻辑。

* MocklnvokersSelector 是 Router 接口 的一种实现，用于过滤出 Mock 的 Invoker。

* MockProtocol根据用户传入的URL和类型生成一个Mockinvoker0

* Mockinvoker实现最终的Invoker逻辑。

  Mockinvoker与MockClusterlnvoker看起来都是Invoker,它们之间有什么区别呢？



​	首先，强制Mock、失败后返回Mock结果等逻辑是在MockClusterlnvoker里处理的；其次，MockClusterlnvoker在某些逻辑下，会生成Mockinvoker并进行调用；然后，在Mockinvoker里会处理 mock=,,return null"> mock="throw xxx"或 mock=com.xxService 这些配置逻辑。最后，Mockinvoker还会被 MockProtocol在引用远程服务的时候创建。我们可以认为，MockClusterlnvoker会处理一些Class级别的Mock逻辑，例如：选择调用哪些Mock类。Mockinvoker处理的是方法级别的Mock逻辑，如返回值。

### Mock的实现**原理**

**1. MockClusterlnvoker 的实现原理**

​	MockClusterWrapper是一个包装类，它在创建 MockClusterlnvoker的时候会把被包装的Invoker传入构造方法，因此MockClusterlnvoker内部天生就含有一个Invoker的引用。MockClusterlnvoker的invoke方法处理了主要逻辑，步骤如下：

​	(1) 获取Invoker的Mock参数。前面已经说过，该Invoker是在构造方法中传入的。如果该Invoker根本就没有配置Mock,则直接调用Invoker的invoke方法并把结果返回；如果配置了 Mock参数，则进入下一步。

​	(2) 判断参数是否以force开头，即判断是否强制Mock。如果是强制Mock,则进入doMocklnvoke逻辑，这部分逻辑在后面统一讲解。如果不以force开头，则进入失败后Mock的逻辑。

​	(3) 失败后调用doMocklnvoke逻辑返回结果。在try代码块中直接调用Invoker的invoke方法，如果抛出了异常，则在catch代码块中调用doMocklnvoke逻辑。

​	强制Mock和失败后Mock都会调用doMocklnvoke逻辑，其步骤如下：

​	(1) 通过 selectMocklnvoker 获得所有 Mock 类型的 Invoker。selectMocklnvoker 在对象的attachment属性中偷偷放进一个invocation.need.mock=true的标识。directory在list方法中列出所有Invoker的时候，如果检测到这个标识，则使用Mockinvokers Selector来过滤Invoker,而不是使用普通route实现，最后返回Mock类型的Invoker列表。如果一个Mock类型的Invoker都没有返回，则通过directory的URL新创建一个Mockinvoker；如果有Mock类型的Invoker,则使用第一个。

​	(2) 调用Mockinvoker的invoke方法。在try-catch中调用invoke方法并返回结果。如果出现了异常，并且是业务异常，则包装成一个RpcResult返回，否则返回RpcException异常。

**2. MocklnvokersSelector 的实现原理**

​	在 doMocklnvoke 的第 1 步中，directory 会使用 MocklnvokersSelector 来过滤出 Mock 类型的Invoker。MocklnvokersSelector是Router接口的其中一种实现。它路由时的具体逻辑如下：

​	(1) 判断是否需要做Mock过滤。如果attachment为空，或者没有invocation.need.mock=true的标识，'则认为不需要做Mock过滤，进入步骤2；如果找到这个标识，则进入步骤3。

​	(2) 获取非Mock类型的Invoker。遍历所有的Invoker,如果它们的protocol中都没有Mock参数，则整个列表直接返回。否则，把protocol中所有没有Mock标识的取出来并返回。

​	(3) 获取Mock类型的Invokero遍历所有的Invoker,如果它们的protocol中都没有Mock参数，则直接返回null。否则，把protocol中所有含有Mock标识的取出来并返回。

**3. MockProtocol 与 Mockinvoker 的实现原理**

​	MockProtocol也是协议的一种,主要是把注册中心的Mock URL转换为Mockinvoker对象。URL可以通过dubbo.admin或其他方式写入注册中心，它被定义为只能引用，不能暴露，如代码清单7-22所示。

![](https://pic.imgdb.cn/item/6101762c5132923bf800aca5.jpg)

​	例如，我们在注册中心/dubbo/com.test.xxxService/providers这个服务提供者的目录下，写入一个 Mock 的 URL： mock:// 192.168.0.123/com.test.xxxService。

​	在Mockinvoker的invoke方法中，主要处理逻辑如下：

​	(1) 获取Mock参数值。通过URL获取Mock配置的参数，如果为空则抛出异常。优先会获取方法级的Mock参数，例如：以methodName.mock为key去获取参数值；如果取不到，则尝试以mock为key获取对应的参数值。

​	(2) 处理参数值是return的配置。如果只配置了一个return,即mock=return,则返回一个空的RpcResult；如果return后面还跟了别的参数，则首先解析返回类型，然后结合Mock参数和返回类型，返回Mock值。现支持以下类型的参数：Mock参数值等于empty,根据返回类型返回new xxx()空对象；如果参数值是null> true> false,则直接返回这些值；如果是其他字符串，则返回字符串；如果是数字、List、Map类型，则返回对应的JSON串；如果都没匹配上，则直接返回Mock的参数值。

​	(3) 处理参数值是throw的配置。如果throw后面没有字符串，则包装成一个RpcException异常，直接抛出；如果throw后面有自定义的异常类，则使用自定义的异常类，并包装成一个RpcException 异常抛出。

​	(4) 处理Mock实现类。先从缓存中取，如果有则直接返回。如果缓存中没有，则先获取接口的类型，如果Mock的参数配置的是true或default,则尝试通过“接口名+Mock”查找Mock实现类，例如：TestService会查找Mock实现TestServiceMock0如果是其他配置方式，则通过Mock的参数值进行查找，例如：配置了 mock=com.xxx.testservice ,则会查找com.xxx.testservice。

# Dubbo扩展点

## Dubbo核心扩展点概述

​	我们经常会听到一句话：唯一不变的，就是变化本身。面对互联网领域日新月异的业务发展变化，作为一个分布式服务框架，既需要提供非常强大的功能，满足业务开发者日常开发的需求，也需要在框架的内在结构里，提供足够多的特定扩展能力，使得用户可以在不改动内部代码和结构的基础上，按照这些接口的约定，即可简单方便地定制出自己想要的功能。Dubbo使用了扩展点的方式来实现这种能力。Dubbo的总体流程和分层是抽象不变的，但是每一层都提供扩展接口，让用户可以自定义扩展每一层的功能。

### 扩展点的背景

​	Dubbo本身的各类功能组件也是按照这些扩展点的具体实现构建出来的，例如Dubbo默认的Dubbo协议、Hessian2序列化和协议、fastjson序列化、ZooKeeper注册中心等。更多的“非官方”扩展组件则是由广大开发者在自己的工作实践中创造并提交到Dubbo项目中的，最终一部分变成了目前的“官方”扩展组件，比如WebService协议、REST协议、Thrift协议、fst和kryo序列化等，另一部分变成了 Dubbo生态项目中的“准官方”扩展组件，例如JSON-RPC/XML-RPC协议、JMS协议、avro/gson序列化、etcd/nacos注册中心等。这些扩展的提交者也成为Dubbo项目的Committer或PMC成员。

### 扩展点整体架构

​	如果按照使用者和开发者两种类型来区分，Dubbo可以分为API层和SPI层。API层让用户只关注业务的配置，直接使用框架的API即可；SPI层则可以让用户自定义不同的实现类来扩展整个框架的功能。

​	如果按照逻辑来区分，那么又可以把Dubbo从上到下分为业务、RPC、Remote三个领域。由于业务层不属于SPI的扩展，因此不是本章关注的内容。可扩展的RPC和Remote层继续细分，又能分出7层，如图8-1所示。

​	另外，细分出来的每一层(Proxy.Registry…)的作用，已经在第1章中介绍，因此本章不再重复赘述。

![](https://pic.imgdb.cn/item/6102c3e55132923bf8e8d6bf.jpg)

## RPC层扩展点

​	按照完整的Dubbo结构分层，RPC层可以分为四层：Config、Proxy、Registry、Cluster。由于Config属于API的范畴，因此我们只基于Proxy、Registry、Cluster三层来介绍对应的扩展点。

### Proxy层扩展点

​	Proxy层主要的扩展接口是ProxyFactoryo我们在使用Dubbo框架的时候，明明调用的是一个本地的接口，为什么框架会自动帮我们发起远程请求，并把调用结果返回呢？整个远程调用的过程对开发者完全是透明的，就像本地调用一样。这正是由于ProxyFactory帮我们生成了代理类，当我们调用某个远程接口时，实际上使用的是代理类。代理类远程调用过程如图8.2所示。

​	图8-2中省略了很多细节，如序列化等，主要是为了说明整个代理调用过程。Dubbo中的ProxyFactory有两种默认实现:Javassist和JDK,用户可以自行扩展自己的实现，如CGLIB(CodeGeneration Library)o Dubbo选用Javassist作为默认字节码生成工具，主要是基于性能和使用的简易性考虑，Javassist的字节码生成效率相对于其他库更快，使用也更简单。下面我们来看一下ProxyFactory接口有哪些具体的方法，如代码清单8-1所示。

![](https://pic.imgdb.cn/item/6102c5c35132923bf8f1e561.jpg)

![](https://pic.imgdb.cn/item/6102c5d05132923bf8f2257d.jpg)

​	我们可以看到ProxyFactory接口有三个方法，每个方法上都有©Adaptive注解，并且方法会根据URL中的proxy参数决定使用哪种字节码生成工具。第二个方法的generic参数是为了标识这个代理是否是泛化调用。已有的扩展点实现如表8.1所示。

![](https://pic.imgdb.cn/item/6102c5e95132923bf8f2a28c.jpg)

​	stub比较特殊，它的作用是创建一个代理类，这个类可以在发起远程调用之前在消费者本地做一些事情，比如先读缓存。它可以决定要不要调用Proxy。

### Registry 层扩展点

​	Registry层可以理解为注册层，这一层中最重要的扩展点就是org. apache, dubbo. registry.RegistryFactoryo整个框架的注册与服务发现客户端都是由这个扩展点负责创建的。该扩展点有@Adaptive({"protocol"))注解，可以根据URL中的protocol参数创建不同的注册中心客户端。例如：protocol=redis,该工厂会创建基于Redis的注册中心客户端。因此，如果我们扩展了自定义的注册中心，那么只需要配置不同的Protocol即可。在第11章"Dubbo注册中心扩展实践”中使用的就是这个扩展点。RegistryFactory接口如代码清单8-2所示。

![](https://pic.imgdb.cn/item/6102c7225132923bf8f8c88c.jpg)

使用这个扩展点，还有一些需要遵循的“潜规则”：

* 如果URL中设置了 check-false,则连接不会被检查。否则，需要在断开连接时抛出异常。
* 需要支持通过usemame:password格式在URL中传递鉴权。
* 需要支持设置backup参数来指定备选注册集群的地址。
* 需要支持设置file参数来指定本地文件缓存。
* 需要支持设置timeout参数来指定请求的超时时间。
* 需要支持设置session参数来指定连接的超时或过期时间。



​	在Dubbo,有AbstractRegistryFactory已经抽象了一些通用的逻辑，用户可以直接继承该抽象类实现自定义的注册中心工厂。已有的RegistryFactory实现如表8-2所示。

![](https://pic.imgdb.cn/item/6102c7c45132923bf8fc1078.jpg)

### Cluster层扩展点

​	Cluster层负责了整个Dubbo框架的集群容错，涉及的扩展点较多，包括容错(Cluster)、路由(Router负载均衡(LoadBalance)> 配置管理工厂(ConfiguratorFactory)和合并器(Merger)。

**1. Cluster扩展点**

​	Cluster需要与Cluster层区分开，Cluster主要负责一些容错的策略，也是整个集群容错的入口。当远程调用失败后，由Cluster负责重试、快速失败等，整个过程对上层透明。整个集群容错层之间的关系，已经在第7章有比较详细的讲解了，因此不再赘述。

​	Cluster扩展点主要负责Dubbo框架中的容错机制，如Failover、Failfast等,默认使用Failover机制。Cluster扩展点接口如代码清单8-3所示。

![](https://pic.imgdb.cn/item/6102c8b75132923bf8010527.jpg)

​	Cluster接口只有一个join方法，并且有^Adaptive注解，说明会根据配置动态调用不同的容错机制。已有的Cluster实现如表8-3所示。

![](https://pic.imgdb.cn/item/6102c90c5132923bf802bfc9.jpg)

**2. RouterFactory 扩展点**

​	RouterFactory是一个工厂类，顾名思义，就是用于创建不同的Router。假设接口 A有多个服务提供者提供服务，如果配置了路由规则(某个消费者只能调用某个几个服务提供者)，则Router会过滤其他服务提供者，只留下符合路由规则的服务提供者列表。

​	现有的路由规则支持文件、脚本和自定义表达式等方式。接口上有@Adaptive("protocol1')注解，会根据不同的protocol自动匹配路由规则，如代码清单8-4所示。

![](https://pic.imgdb.cn/item/6102ca935132923bf80abc6a.jpg)

​	在2.7版本之后，路由模块会做出较大的更新，每个服务中每种类型的路由只会存在一个，它们会成为一个路由器链。因此新的运行模式需要关注2.7版本。已有的RouterFactory实现如表8-4所示。

![](https://pic.imgdb.cn/item/6102cace5132923bf80bf47e.jpg)

**3. LoadBalance 扩展点**

​	LoadBalance是Dubbo框架中的负载均衡策略扩展点，框架中已经内置随机(Random)、轮询(RoundRobin)、最小连接数(LeastActive)、一致性 Hash (ConsistentHash)这几种负载均衡的方式，默认使用随机负载均衡策略。LoadBalance主要负责在多个节点中，根据不同的负载均衡策略选择一个合适的节点来调用。由于在集群容错章节对负载均衡也有比较深入的讲解，因此也不再赘述。LoadBalance扩展点接口源码如代码清单8-5所示。

![](https://pic.imgdb.cn/item/6104b3c95132923bf8f5b80a.jpg)

**4. ConfiguratorFactory 扩展点**

​	ConfiguratorFactory是创建配置实例的工厂类，现有override和absent两种工厂实现，分别会创建OverrideConfigupatop和AbsentConfigurator两种配置对象。默认的两种实现，OverrideConfigurator会直接把配置中心中的参数覆盖本地的参数；AbsentConfigurator会先看本地是否存在该配置，没有则新增本地配置，如果己经存在则不会覆盖。ConfiguratorFactory扩展点源码如代码清单8-6所示。

![](https://pic.imgdb.cn/item/6104b3fd5132923bf8f6fcfa.jpg)

![](https://pic.imgdb.cn/item/6104b4dd5132923bf8f9fbe1.jpg)

**5. Merger扩展点**

​	Merger是合并器，可以对并行调用的结果集进行合并，例如：并行调用A、B两个服务都会返回一个List结果集,Merger可以把两个List合并为一个并返回给应用。默认已经支持map、set、list、byte等11种类型的返回值。用户可以基于该扩展点，添加自定义类型的合并器。Merger扩展点源码如代码清单8-7所示。

![](https://pic.imgdb.cn/item/6104b5a35132923bf8fc77f6.jpg)

## Remote层扩展点

​	Remote处于整个Dubbo框架的底层，涉及协议、数据的交换、网络的传输、序列化、线程池等，涵盖了一个远程调用的所有要素。

​	Remote层是对Dubbo传输协议的封装，内部再划为Transport传输层和Exchange信息交换层。其中Transport层只负责单向消息传输，是对Mina> Netty等传输工具库的抽象。而Exchange层在传输层之上实现了 Request-Response语义，这样我们可以在不同传输方式之上都能做到统一的请求/响应处理。Serialize层是RPC的一部分，决定了在消费者和服务提供者之间的二进制数据传输格式。不同的序列化库的选择会对RPC调用的性能产生重要影响，目前默认选择是Hessian2序列化。

### Protocol 层扩展点

​	Protocol 层主要包含四大扩展点，分别是 Protocol 、Filter 、ExporterListener 和 InvokerListener。其中Protocol. Filter这两个扩展点使用得最多。下面分别介绍每个扩展点。

**1. Protocol 扩展点**

​	Protocol是Dubbo RPC的核心调用层，具体的RPC协议都可以由Protocol点扩展。如果想增加一种新的RPC协议，则只需要扩展一个新的Protocol扩展点实现即可。Protocol扩展点接口如代码清单8-8所示。

![](https://pic.imgdb.cn/item/6104b8615132923bf805c2fb.jpg)

Protocol的每个接口会有一些“潜规则”，在实现自定义协议的时候需要注意。
export 方法：

​	(1) 协议收到请求后应记录请求源IP地址。通过RpcContext.getContext(). setRemote-Address()方法存入RPC上下文。

​	(2) export方法必须实现幕等，即无论调用多少次，返回的URL都是相同的。

​	(3) Invoker实例由框架传入，无须关心协议层。

refer方法：

​	(1) 当我们调用refer。方法返回Invoker对象的invoke()方法时，协议也需要相应地执行invoke()方法。这一点在设计自定义协议的Invoker时需要注意。

​	(2) 正常来说refer。方法返回的自定义Invoker需要继承Invoker接口。

​	(3) 当URL的参数有check=false时，自定义的协议实现必须不能抛出异常，而是在出现连接失败异常时尝试恢复连接。

destroy 方法：

​	(1) 调用destroy方法的时候，需要销毁所有本协议暴露和引用的方法。

​	(2) 需要释放所有占用的资源，如连接、端口等。

​	(3) 自定义的协议可以在被销毁后继续导出和引用新服务。

整个Protocol的逻辑由Protocol、 Exporter、 Invoker三个接口串起来：

* com.alibaba.dubbo.rpc.Protocol；
* com.alibaba.dubbo.rpc.Exporter；
* com. alibaba. dubbo .rpc. Invoker o



​	其中Protocol接口是入口，其实现封装了用来处理Exporter和Invoker的方法：

​	Exporter代表要暴露的远程服务引用，Protocol#export方法是将服务暴露的处理过程，Invoker代表要调用的远程服务代理对象，Protocol#refer方法通过服务类型和URL获得要调用的服务代理。

​	由于Protocol可以实现Invoker和Exporter对象的创建，因此除了作为远程调用对象的构造，还能用于其他用途，例如：可以在创建Invoker的时候对原对象进行包装增强，添加其他Filter进去，ProtocolFilterWrapper实现就是把Filter链加入Invokero如果对这一段并不是十分了解，则可以先了解设计模式中的装饰器模式，对最原始的Invoker进行增强。

​	因此，下面我们只列出框架中用于调用的协议，已有的Protocol扩展点实现如表8.8所示。

![](https://pic.imgdb.cn/item/6104c1d35132923bf824894b.jpg)

**2. Filter扩展点**

​	Filter是Dubbo的过滤器扩展点，可以自定义过滤器，在Invoker调用前后执行自定义的逻辑。在Filter的实现中，必须要调用传入的Invoker的invoke方法，否则整个链路就断了。Filter接口定义及实现的示例如代码清单8-9所示。

![](https://pic.imgdb.cn/item/6104c79b5132923bf83712b7.jpg)

​	可以看到，Filter接口使用了 JDK8的新特性，接口中有default方法onResponse,默认返回收到的结果。

​	由于Filter在后面有专门的章节去讲解，因此默认实现就不在本章再讲一遍了。

**3. ExporterListener/lnvokerListener 扩展点**

​	ExporterListener和InvokerListener这两个扩展点非常相似，ExporterListener是在暴露和取消暴露服务时提供回调；InvokerListener则是在服务的引用与销毁引用时提供回调。ExporterListener与InvokerListener扩展接口如代码清单8-10所示。

![](https://pic.imgdb.cn/item/6104c8795132923bf8399223.jpg)

### Exchange 层扩展点

​	Exchange层只有一个扩展点接口 Exchanger,这个接口主要是为了封装请求/响应模式，例如：把同步请求转化为异步请求。默认的扩展点实现是org.apache.dubbo.「emoting.exchange,support.header.HeaderExchanger o 每个方法上都有@Adaptive 注解，会根据 URL 中的Exchanger参数决定实现类。

![](https://pic.imgdb.cn/item/6104ca6b5132923bf83f3af9.jpg)

​	既然已经有了 Transport层来传输数据了，为什么还要有Exchange层呢？因为上层业务关注的并不是诸如Netty这样的底层Channel。上层一个Request只关注对应的Response,对于是同步还是异步请求，或者使用什么传输根本不关心。Transport层是无法满足这项需求的，Exchange层因此实现了 Request-Response模型，我们可以理解为基于Transport层做了更高层次的封装。

### Transport 层扩展点

​	Transport层为了屏蔽不同通信框架的异同，封装了统一的对外接口。主要的扩展点接口有Transporter、 Dispatcher、 Codec2 和 ChannelHandler。
​	其中，ChannelHandler主要处理连接相关的事件，例如：连接上、断开、发送消息、收到消息、出现异常等。虽然接口上有SPI注解，但是在框架中实现类的使用却是直接“new”的方式。因此不在本章做过多介绍。

**1. Transporter 扩展接口**

​	Transporter屏蔽了通信框架接口、实现的不同，使用统一的通信接口。Transporter扩展接口如代码清单8-12所示。

![](https://pic.imgdb.cn/item/6104caf25132923bf840c7a7.jpg)

​	bind方法会生成一个服务，监听来自客户端的请求；connect方法则会连接到一个服务。两个方法上都有^Adaptive注解，首先会根据URL中server的参数值去匹配实现类，如果匹配不到则根据transporter参数去匹配实现类。默认的实现是netty4o然后我们看一下框架中已有的扩展点实现，如表8-9所示。

![](https://pic.imgdb.cn/item/6104cc1e5132923bf8445e47.jpg)

**2. Dispatcher 扩展接口**

​	如果有些逻辑的处理比较慢，例如：发起I/O请求查询数据库、请求远程数据等，则需要使用线程池。因为I/O速度相对CPU是很慢的，如果不使用线程池，则线程会因为I/O导致同步阻塞等待。Dispatcher扩展接口通过不同的派发策略，把工作派发到不同的线程池，以此来应对不同的业务场景。Dispatcher扩展接口如代码清单8-13所示。

![](https://pic.imgdb.cn/item/6104d2fa5132923bf8593085.jpg)

**3. Codec2扩展接口**

​	Codec2主要实现对数据的编码和解码，但这个接口只是需要实现编码/解码过程中的通用逻辑流程，如解决半包、粘包等问题。该接口属于在序列化上封装的一层。Codec2扩展接口如代码清单8-14所示。

![](https://pic.imgdb.cn/item/6104d3315132923bf859e6ec.jpg)

**4. ThreadPool 扩展接口**

​	我们在Transport层由Dispatcher实现不同的派发策略，最终会派发到不同的ThreadPool中执行。ThreadPool扩展接口就是线程池的扩展。ThreadPool扩展接口如代码清单8-15所示。

![](https://pic.imgdb.cn/item/6104d3565132923bf85a5f32.jpg)

现阶段，框架中默认含有四种线程池扩展的实现，以下内容摘自官方文档：

* fixed,固定大小线程池，启动时建立线程，不关闭，一直持有。
* cached,缓存线程池，空闲一分钟自动删除，需要时重建。
* limited,可伸缩线程池，但池中的线程数只会增长不会收缩。只增长不收缩的目的是
  为了避免收缩时突然来了大流量引起的性能问题。
* eager,优先创建Worker线程池。在任务数量大于corePoolSize小于maximumPoolSize
  时，优先创建Worker来处理任务。当任务数量大于maximumPoolSize时，将任务放入
  阻塞队列。阻塞队列充满时抛出RejectedExecutionException (cached在任务数量超
  过maximumPoolSize时直接抛出异常而不是将任务放入阻塞队列)。

其接口实现的类如下：

* org.apache .dubbo .common.threadpool. support, fixed. F ixedThreadPool；
* org.apache.dubbo.common.threadpool.support.cached.CachedThreadPool；
* org.apache.dubbo.common.threadpool.support.limited.LimitedThreadPool；
* org.apache.dubbo.common.threadpool.support.eager.EagerThreadPool。

### Serialize 层扩展点

​	Serialize层主要实现具体的对象序列化，只有Serialization 一个扩展接口。Serialization是具体的对象序列化扩展接口，即把对象序列化成可以通过网络进行传输的二进制流。

**1. Serialization 扩展接口**

​	Serialization就是具体的对象序列化，Serialization扩展接口如代码清单8-16所示。

![](https://pic.imgdb.cn/item/6104d3b85132923bf85b9c9d.jpg)

![](https://pic.imgdb.cn/item/6104d3c55132923bf85bc6b9.jpg)

​	其中compactedjava是在Java原生序列化的基础上做了压缩，实现了自定义的类描写叙述符的写入和读取。在序列化的时候仅写入类名，而不是完整的类信息，这样在对象数量很多的情况下，可以有效压缩体积。

​	NativeJavaSerialization是原生的Java序列化的实现方式。

​	JavaSerialization是原生Java序列化及压缩的封装。

​	其他的序列化实现则封装了现在比较流行的各种序列化框架，如kryo> protostuff和fastjson等。

## 其他扩展点

​	还有其他的一些扩展点接口： TelnetHandler、StatusChecker、 Container、CacheFactory、Validation 、Logger Adapter和Compiler。由于平时使用得比较少，因此归类到其他扩展点中，下面简单介绍每个扩展点的用途。

**1. TelnetHandler 扩展点**

​	我们知道，Dubbo框架支持Telnet命令连接，TelnetHandler接口就是用于扩展新的Telnet命令的接口。已知的命令与接口实现之间的关系如下：

![](https://pic.imgdb.cn/item/6104d4705132923bf85e0a94.jpg)

**2. StatusChecker 扩展点**

​	通过这个扩展点，可以让Dubbo框架支持各种状态的检查，默认已经实现了内存和load的检查。用户可以自定义扩展，如硬盘、CPU等的状态检查。已有的实现如下所示。

![](https://pic.imgdb.cn/item/6104d4895132923bf85e65ac.jpg)

**3. Container 扩展点**

​	服务容器就是为了不需要使用外部的Tomcat、JBoss等Web容器来运行服务，因为有可能服务根本用不到它们的功能，只是需要简单地在Main方法中暴露一个服务即可。此时就可以使用服务容器。Dubbo中默认使用Spring作为服务容器。

**4. CacheFactory 扩展点**

​	我们可以通过dubbo:method配置每个方法的调用返回值是否进行缓存，用于加速数据访问速度。已有的缓存实现如下所示。

![](https://pic.imgdb.cn/item/6104d4b75132923bf85f0aee.jpg)

**5. Validation 扩展点**

​	该扩展点主要实现参数的校验，我们可以在配置中使用＜dubbo: service validation="校验实现名"/＞实现参数的校验。己知的扩展实现有org.apache.dubbo.validation.support.jvalidation.^Validation, 扩展 key 为 jvalidation。

**6. LoggerAdapter 扩展点**

​	日志适配器主要用于适配各种不同的日志框架，使其有统一的使用接口。已知的扩展点实现如下：

![](https://pic.imgdb.cn/item/6104d4dc5132923bf85f94b7.jpg)

**7. Compiler 扩展点**

​	我们在第4章讲Dubbo扩展点加载机制的时候就提到：@Adaptive注解会生成Java代码,然后使用编译器动态编译出新的Class。 Compiler接口就是可扩展的编译器，现有两个具体的实现（adaptive不算在内）：

![](https://pic.imgdb.cn/item/6104d54b5132923bf8611706.jpg)

# Dubbo高级特性

## Dubbo高级特性概述

​	目前Dubbo框架在支持RPC通信的基础上，提供了大量的高级特性，比如服务端Telnet调用、Telnet调用统计、服务版本和分组隔离、隐式参数、异步调用、泛化调用、上下文信息和结果缓存等特性。本章会对常用的高级特性原理做进一步分析，在表9-1中展示了目前Dubbo支持的高级特性

![](https://pic.imgdb.cn/item/610939b05132923bf8794c4c.jpg)

## 服务分组和版本

​	Dubbo中提供的服务分组和版本是强隔离的，如果服务指定了服务分组和版本，则消费方调用也必须传递相同的分组名称和版本名称。

​	假设我们有订单查询接口 com. alibaba. pay. order.QueryService,这个接口包含不同的版本实现，比如版本分别为1.0.0-stable和2.0.0,在服务端对应的实现名称分别为com.alibaba.pay.order.StableQueryService 和com.alibaba.pay.order.PerfomanceQueryServiceo 我彳门可以在服务暴露时指定配置，如代码清单9-1所示。

![](https://pic.imgdb.cn/item/610939ec5132923bf879f884.jpg)

​	在代码清单9.1中发现，服务暴露直接配置version属性即可，如果要为服务指定分组，则继续添加group属性即可。因为这个特性是强隔离的，消费方必须在配置文件中指定消费的版本。如果消费方式为泛化调用或注解引用，那么也需要指定对应的相同名称的版本号，如代码清单9-2所示。

![](https://pic.imgdb.cn/item/61093ad05132923bf87c9da4.jpg)

​	在消费方〈dubbo:reference〉标签中指定要消费的版本号时，在服务拉取时会在客户端做一次过滤。如果要消费指定的分组，那么还需要指定group属性。

​	当服务提供方进行服务暴露时，服务端会根据serviceGroup/serviceName:serviceversion:port组合成key,然后服务实现作为value保存在DubboProtocol类的exporterMap字段中。这个字段是一个HashMap对象，当服务消费调用时，根据消费方传递的服务分组、服务接口、版本号和服务暴露时的协议端口号重新构造这个key,然后从内存Map中查找对应实例进行调用。

​	当客户端指定了分组和版本时，在Dubbolnvoker构造函数中会将URL中包含的接口、分组、Token和timeout加入attachment,同时将接口上的版本号存储在version字段。当发起RPC请求时，通过DubboCodec把这些信息发送到服务器端，服务器端收到这些关键信息后重新组装成key,然后查找业务实现并调用。

​	Dubbo客户端启动时是如何获取指定分组和服务版本对应的调用列表的呢？当Dubbo客户端启动时，实际上会把调用接口所有的协议节点都拉取下来，然后根据本地URL配置的接口、category、分组和版本做过滤，具体过滤是在注册中心层面实现的。以ZooKeeper注册中心为例，当注册中心推送列表时，会调用ZookeeperRegistry#toUrlsWithoutEmpty方法，这个方法会把所有服务列表进行一次过滤，如代码清单9-3所示。

![](https://pic.imgdb.cn/item/61093b065132923bf87d315f.jpg)

​	Dubbo中接收服务列表是在RegistryDirectory中完成的，它收到的列表是全量的列表。RegistryDirectory主要将URL转换成可以调用的Invokerso在获取列表前会经过①把服务列表解码，用于解码被转译的字符。消费指定分组和版本关键逻辑在②中，它会将特定接口的全量列表和消费方URL进行匹配，匹配规则是校验接口名、类别、版本和分组是否一致。消费
方默认的类别是providers。

## 参数回调

​	Dubbo支持异步参数回调，当消费方调用服务端方法时，允许服务端在某个时间点回调回客户端的方法。在服务端回调到客户端时，服务端不会重新开启TCP连接，会复用已经建立的从客户端到服务端的TCP连接。在讲解参数回调前，我们给出一个参数回调的例子，如代码清单9-4所示，然后对其实现原理进行分析。

![](https://pic.imgdb.cn/item/61093c375132923bf880c884.jpg)

![](https://pic.imgdb.cn/item/61093c485132923bf880fce0.jpg)

​	要实现异步参数回调，我们首先定义一个服务提供者接口，这里举例为Callbackservice,注意其方法的第2个参数是接口，被回调的参数顺序不重要。第2个参数代表我们想回调的客户端CallbackListener接口，具体什么时候回调和调用哪个方法是由服务提供方决定的。对应到代码清单9-4中，①是我们定义的服务提供方服务接口，定义了 addListener方法，用于给客户端调用。②定义了客户端回调接口，这个接口实现在客户端完成。③对应普通Dubbo服务实现。当客户端调用addListener方法时，会将客户端回调实例加入listeners,用于服务端定时回调客户端。服务提供者在初始化时会开启一个线程，它轮询检查是否有回调加入，如果有则每隔5秒回调客户端。在⑤中每隔5秒处理多个回调方法。

​	当服务提供方完成后，我们需要编写消费方代码，用于调用服务提供者addListener方法，把客户端加入回调列表，如代码清单9-5所示。

![](https://pic.imgdb.cn/item/61093d005132923bf88326e7.jpg)

​	客户端调用也是非常简单的，主要是调用服务提供者服务并把自己加入回调列表，同时指定key和对应的回调方法。①会获取Spring的消费配置<dubbo:reference ...>实例，调用CallbackService#addListener,然后创建接口匿名类实现。

​	在服务暴露和消费代码写完后，接下来我们需要做适当配置，告诉Dubbo框架哪个参数是异步回调，如代码清单9.6所示。

![](https://pic.imgdb.cn/item/61093d275132923bf883a43e.jpg)

​	可以发现服务提供方要想实现回调，就需要指定回调方法参数是否为回调，对于客户端消费方来说没有任何区别。

​	实现异步回调的原理比较容易理解，客户端在启动时，会拉取服务Callbackservice元数据，因为服务端配置了异步回调信息，这些信息会透传给客户端。客户端在编码请求时，会发现第2个方法参数为回调对象。此时，客户端会暴露一个Dubbo协议的服务，服务暴露的端口号是本地TCP连接自动生成的端口。在客户端暴露服务时，会将客户端回调参数对象内存id存储在attachment中，对应的key为sys_callback_arg-回调参数索引。这个key在调用普通服务addListener时会传递给服务端，服务端回调客户端时，会把这个key对应的值再次放到attachment中传给客户端。从服务端回调到客户端的attachment会用keycallback.service,instid保存回调参数实例id,用于查找客户端暴露的服务。

​	客户端调用服务端方法时，并不会把第2个异步参数实例序列化并传给服务端。当服务端解码时，会先检查参数是不是异步回调参数。如果发现是异步参数回调，那么在服务端解码参数值时,会自动创建到消费方的代理。服务端创建回调代理实例Invoker类型是ChannelWrappedlnvoker,比较特殊的是，构造函数的service值是客户端暴露对象id,当回调发生时，会把keycallback, service, instid保存的对象id传给客户端，这样就能正确地找到客户端暴露的服务了。

## 隐式参数

​	Dubbo服务提供者或消费者启动时，配置元数据会生成URL, 一般是不可变的。在很多实际的使用场景中，在服务运行期需要动态改变属性值，在做动态路由和灰度发布场景中需要这个特性。Dubbo框架支持消费方在RpcContext#setAttachment方法中设置隐式参数，在服务端RpcContext#getAttachment方法中获取隐式传递。

​	当客户端发起调用前，设置隐藏参数，框架会在拦截器中把当前线程隐藏参数传递到Rpclnvocation的attachment中，服务端在拦截器中提取隐藏参数并设置到当前线程RpcContext中。隐式传参的详细原理如图9-1所示。

![](https://pic.imgdb.cn/item/61093dcd5132923bf885ad17.jpg)

![](https://pic.imgdb.cn/item/61093dde5132923bf885e138.jpg)

​	在消费方调用服务方传递隐式参数时，会在Abstractlnvoker#invoke方法调用中合并RpcContext#getAttachments()参数。用户的隐式参数会被合并到 Rpclnvocation 中的 attachment字段，这个字段发送给服务端。在服务提供方收到请求时，在ContextFilter#invoke中提取Rpclnvocation中的attachment信息，并设置到当前线程上下文中。因为后端业务方法调用和拦截器在同一个线程中执行，所以直接使用RpcContext.getContext().getAttachment获取值即可。在图9・1中会发现客户端在拦截器中(ConsumercontextFilter)执行setAttachements方法，这个主要支持服务端透传隐式参数给客户端。

## 异步调用

​	本节主要聚焦Dubbo在客户端支持异步调用方面的内容，在编写本书时，Dubbo还未支持服务端异步调用，2.7.0+版本才在服务端支持异步调用。在客户端实现异步调用非常简单，在消费接口时配置异步标识，在调用时从上下文中获取Future对象，在期望结果返回时再调用阻塞方法Future.get()即可。我们给出了在客户端实现异步调用的实例，如代码清单9-8所示。

![](https://pic.imgdb.cn/item/61093e155132923bf8867d9d.jpg)

​	通过代码清单9-8,我们知道在客户端发起异步调用时，应该在保存当前调用的Future后，再发起其他远程调用，否则前一次异步调用的结果可能丢失(异步Future对象会被上下文覆盖)。因为框架要明确知道用户意图，所以需要再明确开启使用异步特性，在<dubbo:reference ...>标签中指定async标记，如代码清单9・9所示。

![](https://pic.imgdb.cn/item/61093e2a5132923bf886b7d2.jpg)

​	站在Dubbo客户端角度来说，直接发起RPC调用端属于用户线程。用户线程①发起任意远程方法调用，最终会通过I/O线程发送网络报文。在真实发送报文前会在用户线程中设置当前异步请求Future (③)。因此在用户线程发起下一个远程方法调用前，需要先保存异步Future对象(④)o Dubbo框架会把异步请求对象保存在DefaultFuture类中，当服务端响应或超时时，被挂起的用户线程将被唤醒(⑤)。用户线程设置异步Future对象的逻辑在Dubbolnvoker#dolnvoke方法中完成，感兴趣的读者可以参阅Dubbolnvoker中对应的源码实现。

## 泛化调用

​	Dubbo泛化调用特性可以在不依赖服务接口 API包的场景中发起远程调用。这种特性特别适合框架集成和网关类应用开发。Dubbo在客户端发起泛化调用并不要求服务端是泛化暴露。假设我们调用服务端com.xxx.XxxService#sayHello方法，可以实现如代码清单方法9-10所示的方法。

![](https://pic.imgdb.cn/item/61093f6a5132923bf88a749f.jpg)

​	目前泛化调用必需的参数主要包括应用名称、注册中心(或者是直连调用地址)、真实接口名称和泛化标识。在发起远程服务调用时，GenericService方法参数类型分别为真实方法名、真实方法参数类型签名和真实参数值。这里有一个注意事项，每次动态创建的GenericService实例比较重，需要建立TCP连接，处理注册中心订阅和服务列表等计算，因此需要缓存ReferenceConfig对象进行复用。但是往往很多业务开发时，忘记设置ReferenceConfig对象的Check方法为false,导致在没有服务提供者时，触发框架抛出No provider available的异常，从而导致缓存命中失败。

​	其实泛化的实现原理相对比较好理解，服务端在处理服务调用时，在GenericFilter拦截器中先把Rpclnvocation中传递过来的参数类型和参数值提取出来，然后根据传递过来的接口名、方法名和参数类型查找服务端被调用的方法。获取真实方法后，主要提取真实方法参数类型(可能包含泛化类型)，然后将参数值做Java类型转换。最后用解析后的参数值构造新的Rpclnvocation对象发起调用。

## 上下文信息

​	Dubbo上下文信息的获取和存储同样是基于JDK的ThreadLocal实现的。上下文中存放的是当前调用过程中所需的环境信息。RpcContext是一个ThreadLocal的临时状态记录器，当收到或发送RPC时，当前线程关联的RpcContext状态都会变化。比如：A调用B, B再调用C,则在B机器上，在B调用C之前，RpcContext记录的是A调用B的信息，在B调用C之后，RpcContext记录的是B调用C的信息。

![](https://pic.imgdb.cn/item/61093fa15132923bf88b1e54.jpg)

​	在客户端和服务端分另U有一个拦截设置当前上下文信息，对应的分另U为ConsumerContextFilter和ContextFiltero在客户端拦截器实现中，因为Invoker包含远程服务信息，因此直接设置远程IP等信息。在服务端拦截器中主要设置本地地址，这个时候无法获取远程调用地址。设置远程地址主要在DubboProtocol#ExchangeHandlerAdapter.reply方法中完成，可以直接通过channel.getRemoteAddress 方法获取。

## Telnet 操作

​	目前Dubbo支持通过Telnet登录进行简单的运维，比如查看特定机器暴露了哪些服务、显示服务端口连接列表、跟踪服务调用情况、调用本地服务和服务健康状况等。

​	为了避免和前面章节讲解重复，本节我们主要讲解Is、ps> trace和count命令的实现和原理。

​	当服务发布时，如果注册中心没有对应的服务，那么我们可以初步使用Is命令检查Dubbo服务是否正确暴露了。Is主要提供了查询已经暴露的服务列表、查询服务详细信息和查询指定服务接口信息等功能。Is命令的用法如下：

![](https://pic.imgdb.cn/item/61093fd65132923bf88bca0b.jpg)

​	Is命令的实现主要基于ListTelnetHandler, Dubbo框架的Telnet调用只对Dubbo协议提供支持。它的原理非常简单，当服务端收到Is命令和参数时，会加载ListTelnetHandler并执行，然后触发DubboProtocol.getDubboProtocol().getExporters()方法获取所有已经暴露的服务，获取暴露的接口名和暴露服务别名(path属性)进行匹配，将匹配的结果进行输出。如果是查看服务暴露的方法，则框架会获取暴露接口名，然后反射获取所有方法并输出。

![](https://pic.imgdb.cn/item/61093fec5132923bf88c0e1f.jpg)

​	ps命令实现类对应PortTelnetHandler类，当Dubbo服务暴露时，会把关联端口的服务端实例加入DubboProtocol类的serverMap字段。当执行ps命令时，PortTelnetHandler类会通过DubboProtocol.getDubboProtocol().getServers()提取暴露的 server 实例。它持有了端口号和所有客户端连接信息等。当无法确认命令对应的后端实现时，可以查找和扩展点名称相同的文件,它包含扩展点所有的实现定义，比如 com.alibaba.dubbo.remoting.telnet.TelnetHandler。

![](https://pic.imgdb.cn/item/6109400c5132923bf88c795c.jpg)

​	如果在使用trace命令跟踪方法调用时指定了最大次数，则不需要重复执行trace命令，当服务接口方法调用超过了最大次数后，不会把调用结果信息推送给Telnet客户端。

​	trace命令对应的实现类是TraceTelnetHandler,它本身不会执行任何方法调用，首先根据传递的接口和方法查找对应的Invoker,然后把当前的Telnet连接（Channel）、接口、方法和最大执行次数信息记录在TraceFilter中，当接口方法被调用时,TraceFilter会取出对应的Telnet连接（Channel）,并把调用结果信息发送的Telnet客户端。

​	count命令也用于统计服务信息，但它主要统计方法调用成功数、失败数、正在并发执行数、平均耗时和最大耗时。如果在服务方暴露服务时配置了 executes属性，那么使用count命令可以统计并发调用信息。

![](https://pic.imgdb.cn/item/610940325132923bf88cf562.jpg)

​	count命令对应的实现类是CountTelnetHandler,每次执行count命令时在服务端会启动一个线程去循环统计当前调用次数。比如统计10次，在线程中每间隔1秒执行一次统计，直到达到统计次数时退出线程。框架会使用RpcStatus类记录并发调用信息，CountTelnetHandler负责提取这些统计信息并输出给Telnet客户端。

## Mock 调用

​	Dubbo提供服务容错的能力，通常用于服务降级，比如验权服务，当服务提供方“挂掉”后，客户端不抛出异常，而是通过Mock数据返回授权失败。

![](https://pic.imgdb.cn/item/610940865132923bf88dfe0d.jpg)

​	当Dubbo服务提供者调用抛出RpcException时，框架会降级到本地Mock伪装。以接口com.foo.BarService 例，第1种和第2种的使用方式是等价的，当直接指定mock=true时，客户端启动时会查找并加装com.foo. BarServiceMock类。查找规则根据接口名加Mock后缀组合成新的实现类，当然也可以使用自己的Mock实现类指定给Mock属性。

​	当在Mock中指定return null时,允许调用失败返回空值。当在Mock中指定throw或throw com. alibaba. XXXException 时，分另 U 会抛出即 cException 和用户自定义异常 com. alibaba.XXXException。2.6.5版本以前（包括当前版本），因为实现有缺陷，在使用方式4、5和6中需要更新后的版本支持。目前默认场景都是在没有服务提供者或调用失败时，触发Mock调用，如果不想发起RPC调用直接使用Mock数据，则需要在配置中指定force:语法（同样需要版本高2.6.5）。

​	这些Mock关键逻辑是在哪里处理的呢？处理Mock伪装对应的实现类是MockClusterlnvoker,因为MockClusterWrapper是对扩展点Cluster的包装，当框架在加载Cluster扩展点时会自动使用MockClusterWrapper 类对 Cluster 实例进行包装（默认是 FailoverCluster）。MockClusterlnvoker对应的实现如代码清单9-12所示。

![](https://pic.imgdb.cn/item/6109414d5132923bf8907dc4.jpg)

​	代码清单9-12中主要完成服务降级伪装。在①中如果没有配置Mock,则直接发起RPC调用。2.6.5版本虽然支持force特性，但因为有bug,②中的这段代码实际上并不会执行。在2.6.5版本以后，如果用户为Mock指定了 force,则直接在本地伪装而不发起RPC调用。在③中先处理正常RPC调用，如果调用出错则会降级到Mock调用。在④中具体Mock数据是由开发者自己编码完成的。Dubbo框架对常用的返回值做了支持，比如接口返回布尔值，可以直接在Mock中指定return true。

## 结果缓存

​	Dubbo框架提供了对服务调用结果进行缓存的特性，用于加速热门数据的访问速度，Dubbo提供声明式缓存，以减少用户加缓存的工作量。因为每次调用都会使用3SON.to3SONString方法将请求参数转换成字符串，然后拼装唯一的key,用于缓存唯一键。如果不能接受缓存造成的开销，则谨慎使用这个特性。

​	如果要使用缓存，则可以在消费方添加如下配置：

![](https://pic.imgdb.cn/item/610941975132923bf8915896.jpg)

## 小结

​	本章主要对Dubbo中的高级特性进行讲解，比如服务分组和版本、参数回调、隐式参数、异步调用、泛化调用、上下文信息、Telnet操作、Mock调用和结果缓存原理。虽然本章的知识点比较独立，但这些特性点能够解决实际业务场景中的很多问题。比如版本和分组能够解决业务资源隔离，防止整体资源被个别调用方拖垮，可以将某些调用分配一个隔离的资源池中，单独为它们提供服务。

# Dubbo过滤器

​	本章首先介绍Dubbo过滤器的总体概况，包括如何配置和使用一些框架自定义的规则约束,整个过滤器接口的总体结构，Dubbo框架中内置过滤器的不同用途；然后介绍众多的过滤器是如何初始化成一个过滤器链的；最后，由于有的过滤器会在服务提供者端生效，有的会在消费者端生效，因此我们会分为服务提供者和消费者两端来分别介绍各端的过滤器的实现原理。通过本章的阅读，读者可以了解整个Dubbo过滤器在框架中的实现原理，后续可以无障碍地自行扩展过滤器。

## Dubbo过滤器概述

​	做过Java Web开发的读者对过滤器应该都不会陌生,Dubbo中的过滤器和Web应用中的过滤器的概念是一样的，提供了在服务调用前后插入自定义逻辑的途径。过滤器是整个Dubbo框架中非常重要的组成部分，Dubbo中有很多功能都是基于过滤器扩展而来的。过滤器提供了服务提供者和消费者调用过程的拦截，即每次执行RPC调用的时候，对应的过滤器都会生效。虽然过滤器的功能强大，但由于每次调用时都会执行，因此在使用的时候需要注意它对性能的影响。

### 过滤器的使用

​	我们知道Dubbo中巳经有很多内置的过滤器，并且大多数都是默认启用的，如ContextFiltero对于自行扩展的过滤器，要如何启用呢？ 一种方式是使用@Activate注解默认启用；另一种方式是在配置文件中配置，下面是官方文档中的配置，如代码清单10・1所示。

![](https://pic.imgdb.cn/item/610942675132923bf893bf9c.jpg)

### 过滤器的总体结构

​	在了解过滤器的使用方式后，我们先从“上帝视角”看一下接口关系，如图10.1所示，让读者对过滤器有一个总体的了解。



![](https://pic.imgdb.cn/item/610942a85132923bf8948096.jpg)

​	从图10-1可以看到所有的内置过滤器中除了 CompatibleFilter特别突出，只继承了 Filter接口，即不会被默认激活，其他的内置过滤器都使用了旧Activate注解，即默认被激活。Filter接口上有也PI注解，说明过滤器是一个扩展点，用户可以基于这个扩展点接口实现自己的过滤器。所有的过滤器会被分为消费者和服务提供者两种类型，消费者类型的过滤器只会在服务引用时被加入Invoker,服务提供者类型的过滤器只会在服务暴露的时候被加入对应的Invoker。MonitorFilter比较特殊，它会同时在暴露和引用时被加入Invokero具体的过滤器加入Invoker的实现原理会在10.2节讲解。

​	我们来了解一下每个过滤器的作用，如表10-1所示。

![](https://pic.imgdb.cn/item/610942cc5132923bf894e939.jpg)

​	由于表10-1中的GenericFilter> EchoFilter等过滤器在“第9章Dubbo高级特性”中的泛化调用、回声测试等章节已经讲解过，因此本章不再赘述。还有一些过滤器的实现过于简单，如CompatibleFilter、 MonitorFilter等，只是做一些对象类型的转换、计数的统计等，没有深层次的原理，因此本章也不再赘述。感兴趣的读者可以自行查看源码。

​	每个过滤器的使用方不一样，有的是服务提供者使用，有的是消费者使用。Dubb。是如何保证服务提供者不会使用消费者的过滤器的呢？答案就在旧Activate注解上，该注解可以设置过滤器激活的条件和顺序，如旧Activate (group = Constants ・ PROVIDER, order = -110000)表示在服务提供端扩展点实现才有效，并且过滤器的顺序是-110000。

## 过滤器链初始化的实现原理

​	这么多默认的过滤器实现类都会在扩展点初始化的时候进行加载、排序等，这部分的实现原理在“第4章Dubbo扩展点加载机制”中已经讲解过，如果己经忘记，读者可以查看4.3.4节。使用过Filter的读者都知道，所有的Filter会连接成一个过滤器链，每个请求都会经过整个链路中的每一个Filtero那么这个过滤器链在Dubbo框架中是如何组装起来的呢？

​	我们在前面的章节己经了解过，服务的暴露与引用会使用Protocol层，而ProtocolFilterWrapper包装类则实现了过滤器链的组装。在服务的暴露与引用过程中，会使用ProtocolFilterWrapper#buildInvokerChain方法组装整个过滤器链，如代码清单10-2所示。

![](https://pic.imgdb.cn/item/6109431a5132923bf895cb21.jpg)

​	然后我们来关注一下buildlnvokerChain方法是如何构造调用链的，总的来说可以分为两步：

​	(1) 获取并遍历所有过滤器。通过ExtensionLoader#getActivateExtension方法获取所有的过滤器并遍历。

​	(2) 使用装饰器模式，增强原有Invoker,组装过滤器链。使用装饰器模式，像俄罗斯套娃一样，把过滤器一个又一个地“套”到Invoker上。

​	构造调用链源码如代码清单10-3所示。

![](https://pic.imgdb.cn/item/610943415132923bf89641f6.jpg)

​	源码中为什么要倒排遍历呢？因为是通过从里到外构造匿名类的方式构造Invoker的，所以只有倒排，最外层的Invoker才能是第一个过滤器。我们来看一个例子：

​	假设有过滤器A、B、C和Invoker,会按照C、B、A倒序遍历，过滤器链构建顺序为：C—Invoker, B—C—Invoker, A—B—C—Invoker。最终调用时的顺序就会变为A是第一个过滤器。

​	上面已经介绍了整个过滤器链在框架中组装的实现原理，下面针对每个过滤器的实现原理进行讲解。把过滤器分为服务提供者端和消费者端两大类分别进行讲解。

## 服务提供者过滤器的实现原理

​	在表10-1中，日Activate注解上可以设置group属性，从而设定某些过滤器只有在服务提供者端才生效。本章将详细介绍每一个在服务提供者端生效的过滤器，共8个。服务提供者端的过滤器数量明显比消费者端多。

### AccessLogFilter 的实现原理

**1. AccessLogFilter 的使用**

​	AccessLogFilter是一个日志过滤器，如果想记录服务每一次的请求日志，则可以开启这个过滤器。虽然AccessLogFilter有旧Activate注解，默认会被激活，但还是需要手动配置来开启日志的打印。我们有两种方式来配置开启AccessLogFilter,如代码清单10-4所示。

![](https://pic.imgdb.cn/item/610943a35132923bf89778d1.jpg)

**2. AccessLogFilter 的实现**

​	AccessLogFilter的实现步骤比较简短，主要分为构造方法和invoke方法两大逻辑。在AccessLogFilter的构造方法中会加锁并初始化一个定时线程池ScheduledThreadPoolo该线程池只有在指定了输出的log文件时才会用到,ScheduledThreadPool中的线程会定时把队列中的日志数据写入文件。在构造方法中主要是初始化线程池,而打印日志的逻辑主要在invoke方法中，其逻辑如下：

![](https://pic.imgdb.cn/item/610943c25132923bf897dd5f.jpg)

### ExecuteLimitFilter 的实现原理

​	ExecuteLimitFilter用于限制每个服务中每个方法的最大并发数，有接口级别和方法级别的配置方式。我们先看看官方文档中是如何配置的：

![](https://pic.imgdb.cn/item/610944215132923bf898fb31.jpg)

​	如果不设置，则默认不做限制；如果设置了小于等于0的数值，那么也会不做任何限制。、

​	其实现原理是：在框架中使用一个ConcurrentMap缓存了并发数的计数器。为每个请求URL生成一个Identity String,并以此为key；再以每个IdentityString生成一个RpcStatus对象，并以此为value。RpcStatus对象用于记录对应的并发数。在过滤器中，会以try-catch-finally的形式调用过滤器链的下一个节点。因此，在开始调用之前，会通过URL获得RpcStatus对象，把对象中的并发数计数器原子+1，在finally中再将原子-1。只要在计数器+1的时候，发现当前计数比设置的最大并发数大时，就会抛出异常，提示己经超过最大并发数，请求就会被终止并直接返回。

### ClassLoaderFilter 的实现原理

​	如果读者对Java的类加载机制不清楚的话，那么会感觉ClassLoaderFilter过滤器不好理解。ClassLoaderFilter主要的工作是：切换当前工作线程的类加载器到接口的类加载器，以便和接口的类加载器的上下文一起工作。我们先来看一下ClassLoaderFilter的源码，非常简短，如代码清单10-5所示。

![](https://pic.imgdb.cn/item/6109445a5132923bf899ab62.jpg)



​	代码很好理解，就是临时切换了一下上下文类加载器。为什么要这么做呢？首先读者需要理解Java的类加载机制和双亲委派模型等，这些概念在网上已经有非常多的资料，并且也不在本书内容的范围内，因此不清楚的读者可以先去学习一下这部分的知识，本书就不再赘述。我们先了解一下双亲委派模型在框架中会有哪些问题,先看代码清单10・6中的代码能否正常运行。

![](https://pic.imgdb.cn/item/610944765132923bf89a01cc.jpg)

​	如果ClassA和ClassB都是同一个类加载器加载的，则它们之间是可以互相访问的，ClassA的调用会输出ClassB。但是，如果ClassA和ClassB是不同的类加载器加载的呢？不同类加载器加载示例如图10-2所示。

![](https://pic.imgdb.cn/item/6109448b5132923bf89a452b.jpg)

​	假设ClassA由ClassLoaderA加载，ClassB由ClassLoaderB加载，此时ClassA是无法访问ClassB的。我们按照双亲委派模型来获取一下ClassB：首先，ClassA会从ClassLoaderA中查找ClassB,看是否已经加载。没找到则继续往父类加载器ParentClassLoader查找ClassB,没找到则继续往父类加载器查找 最终还没找到，会抛出ClassNotFoundException异常。

​	讲了这么多，和ClassLoaderFilter有什么关系呢？如果要实现违反双亲委派模型来查找Class,那么通常会使用上下文类加载器(ContextClassLoader)。当前框架线程的类加载器可能和Invoker接口的类加载器不是同一个，而当前框架线程中又需要获取Invoker的类加载器中的一些 Class,为 了避免出现 ClassNot Found Exception,此时只需要使用 Thread. currentThread().getContextClassLoader()就可以获取Invoker的类加载器，进而获得这个类加载器中的Class。

​	常见的使用例子有DubboProtocol#optimizeSerialization方法，会根据Invoker中配置的optimizer参数获取扩展的自定义序列化处理类，这些外部引入的序列化类在框架的类加载器中肯定没有，因此需要使用Invoker的类加载器获取对应的类。

### ContextFilter 的实现原理

​	ContextFilter主要记录每个请求的调用上下文。每个调用都有可能产生很多中间临时信息，我们不可能要求在每个接口上都加一个上下文的参数，然后一路往下传。通常做法都是放在ThreadLocal中，作为一个全局参数，当前线程中的任何一个地方都可以直接读/写上下文信息。

​	ContextFilter就是统一在过滤器中处理请求的上下文信息，它为每个请求维护一个RpcContext对象，该对象中维护两个InternalThreadLocal (它是优化过的ThreadLocal,具体可以搜索Netty的InternalThreadLocal做了什么优化)，分别记录local和server的上下文。每次收到或发起RPC调用的时候，上下文信息都会发生改变。例如：A调用B, B调用C。当A调用B且B还未调用C时，RpcContext中保存A调用B的上下文；当B开始调用C的时候，RpcContext中保存B调用C的上下文。发起调用时候的上下文是由ConsumerContextFilter实现的，这个是消费者端的过滤器，因此不在本节讲解oContextFilter保存的是收到的请求的上下文。

​	ContextFilter的主要逻辑如下：

![](https://pic.imgdb.cn/item/610944ec5132923bf89b7f2e.jpg)

### ExceptionFilter 的实现原理

​	光看ExceptionFilter很容易会认为它和我们平时写业务时统一处理异常的过滤器一样。它的关注点不在于捕获异常，而是为了找到那些返回的自定义异常，但异常类可能不存在于消费者端，从而防止消费者端序列化失败。对于所有没有声明Unchecked的方法抛出的异常，ExceptionFilter会把未引入的异常包装到RuntimeException中，并把异常原因字符串化后返回。

​	因此，ExceptionFilter的逻辑都在onResponse方法中。

​	ExceptionFilter过滤器会打印出ERROR级别的错误日志，但并不会处理泛化调用，即Invoker的接口是GenericService。

![](https://pic.imgdb.cn/item/610945125132923bf89c0989.jpg)

### TimeoutFilter 的实现原理

​	TimeoutFilter主要是日志类型的过滤器，它会记录每个Invoker的调用时间，如果超过了接口设置的timeout值，则会打印一条警告日志，并不会干扰业务的正常运行，如代码清单10-7所示。

![](https://pic.imgdb.cn/item/610945485132923bf89ce9b8.jpg)

### TokenFilter 的实现原理

​	在Dubbo中，如果某些服务提供者不想让消费者绕过注册中心直连自己，则可以使用令牌验证。总体的工作原理是，服务提供者在发布自己的服务时会生成令牌，与服务一起注册到注册中心。消费者必须通过注册中心才能获取有令牌的服务提供者的URL。TokenFilter是在服务提供者端生效的过滤器，它的工作就是对请求的令牌做校验。我们首先来看一下开启令牌校验的配置方式，代码清单10-8摘自官方文档。

![](https://pic.imgdb.cn/item/6109456d5132923bf89d9113.jpg)

我们来看一下整个令牌的工作流程：、

(1) 消费者从注册中心获得服务提供者包含令牌的URL。

(2) 消费者RPC调用设置令牌。具体是在Rpclnvocation的构造方法中，把服务提供者的令牌设置到附件(attachments)中一起请求服务提供者。、

(3) 服务提供者认证令牌。

​	这就是TokenFilter所做的工作。

​	TokenFilter的工作原理很简单，收到请求后，首先检查这个暴露出去的服务是否有令牌信息。如果有，则获取请求中的令牌，如果和接口的令牌匹配，则认证通过，否则认证失败并抛出异常。

### TpsLimitFilter 的实现原理

​	TpsLimitFilter主要用于服务提供者端的限流。我们会发现在org. apache, dubbo. rpc. Filter这个SPI配置文件中，并没有TpsLimitFilter的配置，因此如果需要使用，则用户要自己添加对应的配置。TpsLimitFilter的限流是基于令牌的，即一个时间段内只分配N个令牌，每个请求过来都会消耗一个令牌，耗完即止，后面再来的请求都会被拒绝。限流对象的维度支持分组、版本和接口级别，默认通过interface + group + version作为唯一标识来判断是否超过最大值。我们先看一下如何配置：

![](https://pic.imgdb.cn/item/610945ce5132923bf89f3d7d.jpg)

## 消费者过滤器的实现原理

### ActiveLimitFilter 的实现原理

​	和服务提供者端的ExecuteLimitFilter相似，ActiveLimitFilter是消费者端的过滤器，限制的是客户端的并发数。官方文档中使用的配置如下：

![](https://pic.imgdb.cn/item/610945f15132923bf89fce14.jpg)

​	如果`<dubbo:service>`和`<dubbo:reference>` 都配了 actives,贝`<dubbo:reference>`优先。如果设置了 actives小于等于0,则不做并发限制。下面我们看一下ActiveLimitFilter的具体实现逻辑：

(1) 获取参数。获取方法名、最大并发数等参数，为下面的逻辑做准备。

(2) 如果达到限流阈值，和服务提供者端的逻辑并不一样，并不是直接抛出异常，而是先等待直到超时，因为请求是有timeout属性的。当并发数达到阈值时，会先加锁抢占当前接口的RpcStatus对象，然后通过wait方法进行等待。此时会有两种结果，第一种是某个Invoker在调用结束后，并发把计数器原子-1并触发一个notify,会有一个在wait状态的线程被唤醒并继续执行逻辑。第二种是wait等待超时都没有被唤醒，此时直接抛出异常，如代码清单10・9所示。

![](https://pic.imgdb.cn/item/6109463c5132923bf8a11f7b.jpg)

![](https://pic.imgdb.cn/item/6109464b5132923bf8a161f0.jpg)

​	(3)如果满足调用阈值，则直接进行调用，成功或失败都会原子-1对应并发计数。最后会唤醒一个等待中的线程。详情见代码清单10-9的后半部分。

​	细心的读者肯定已经发现，这种方式的限流是有问题的。在大并发场景下容易出现超过限流阈值的情况。例如：当10个线程同时获取当前并发数时，都发现还差1个计数到达阈值，此时10个线程都符合要求并往下执行，即超了 9个限额。不过这个问题在新版本中己经修复。

### ConsumerContextFilter 的实现原理

​	ConsumerContextFilter会和ContextFilter配合使用。因为在微服务环境中，有很多链式调用，如A-B-C-D。收到请求时，当前节点可以被看作一个服务提供者，由ContextFilter设置上下文。当发起请求到下一个服务时，当前服务变为一个消费者，由ConsumerContextFilter设置上下文。其工作逻辑主要如下：

(1) 设置当前请求上下文，如Invoker信息、地址信息、端口信息等。
(2) 服务调用。清除Response ±下文，然后继续调用过滤器链的下一个节点。
(3) 清除上下文信息。每次服务调用完成，都会把附件上下文清除，如隐式传参。

### DeprecatedFilter 的实现原理

​	DeprecatedFilter会检查所有调用，如果方法已经通过dubbo:parameter设置了 deprecated=true,则会打印一段ERROR级别的日志。这个错误日志只会打印一次，判断是否打印过的key维度是：接口名+方法名。打印过的key都会被加入一个Set中保存，后续就不会再打印了，如代码清单10-10所示。

![](https://pic.imgdb.cn/item/610946835132923bf8a24ffa.jpg)

### FutureFilter 的实现原理

​	FutureFilter主要实现框架在调用前后出现异常时，触发调用用户配置的回调方法，如以下配置：

![](https://pic.imgdb.cn/item/610946b95132923bf8a33ba9.jpg)

​	调用前的回调实现就很简单了，由于整个逻辑是在过滤器链中执行的，FutureFilter在执行下一个节点的invoke方法前调用oninvoke回调方法就能实现调用前的回调。方法在服务引用初始化的时候就会把配置文件中的回调方法保存到ConsumerMethodModel中，后续使用的时候,直接取出来就可以调用。不过需要注意的是，oninvoke回调只会对异步调用有效。

​	当调用有返回结果的时候，会执行FutureFilter#onResponse的逻辑。对于同步调用的方法，则直接判断返回的result是否有异常，有异常则同步调用。nthrow回调方法，没有异常则同步调用onretum回调方法。对于异步调用，会通过CompletableFuture的thenApply方法来执行onthrow 或 onretum 的回调。CompletableFuture 是 JDK8 中的新特性。

## 小结

​	之前的章节已经涉及很多过滤器的讲解，因此本章只介绍了一些前面章节都没有涉及的过滤器。首先介绍了 Dubbo框架中所有过滤器的总体大图，讲解了每个过滤器的作用及归属，过滤器分为服务提供者端生效和消费者端生效两种，其中服务提供者端有11种过滤器，消费者端有5种过滤器。有一个特殊的Monitor过滤器在两端都会生效，还有一个CompatibleFilter过滤器并没有默认启用。然后，我们介绍了整个过滤器链串联起来的原理，框架在ProtocolFilterWrapper中为每个Invoker包上了一层又一层的过滤器，最终形成一个过滤器链。最后，我们分别详细介绍了服务提供者、消费者端的每个过滤器的实现原理。

## Dubbo注册中心扩展实践

## etcd背景介绍

​	etcd是一种分布式键值存储系统，它提供了可靠的集群存储数据的途径。它是开源的，可以在GitHub上找到它的源码。etcd使用了 Raft算法保证集群中数据的一致性，当leader节点下线时会自动触发新的leader选举，以此容忍机器的故障。应用可以在etcd中读写数据，例如：把一些参数性的信息通过key-value （键值对）形式写入etcd,这些数据可以被监听，当数据发生变化的时候，可以通知监听者。etcd还有其他高级特性，感兴趣的读者可以访问其官网查看，本书就不再赘述。

​	虽然Dubbo默认支持ZooKeeper和Redis等注册中心实现，但是生产环境中使用较多的还是ZooKeepero后起之秀etcd广泛应用于Kubernates中（用于服务发现），经过了生产环境的考验。相比于ZooKeeper实现，基于etcd实现的注册中心有很多优点，例如：不需要每次子节点变更都重新全量拉取节点数据，大大降低了网络的压力。

​	支持etcd注册中心的原因非常简单，可以让Dubbo支持更多的注册中心，丰富Dubbo生态。在正式讲解实现原理前，我们先看一下etcd注册中心的一些优点：

![](https://pic.imgdb.cn/item/6109491c5132923bf8ac0bb9.jpg)

## etcd数据结构设计

​	etcd数据结构设计如果仔细阅读前面章节，在理解ZooKeeper基础上，就很容易理解etcd的存储结构了。etcd注册中心只支持最新v3版本的APE在v3版本的etcd实现中，所有元数据信息都是基于key-value （键值对）存储的，和ZooKeeper中节点和子节点不同，etcd存储是通过前缀区分的。

​	在ZooKeeper中有临时节点的概念，它是通过TCP连接状态断开自动删除临时节点数据的。etcd注册中心也有临时节点的概念，但不是根据TCP连接状态，而是根据租约到期自动删除对应的key实现的。当provider和consumer上线时，会自动向注册中心写临时节点。可能有读者会有疑问，通过租约到期删除key会不会不可靠？当JVM关闭时,Dubbo会触发优雅停机逻辑，也会及时删除临时key,所以问题不大。

​	其实etcd3没有树的概念，etcd3里面都是平铺展开的键值对，我们可以把展开的键值对抽象成树的概念，使其与ZooKeeper的模型保持一致，降低其他开发者针对每个注册中心都必须重新理解一遍模型的成本。

​	在etcd3注册中心的存储结构中，我们会按照树状结构平铺展开来举例。

![](https://pic.imgdb.cn/item/610949685132923bf8ad1517.jpg)

​	这里有意地画出了树状结构，每个层级都代表etcd中的key,非临时节点默认存储的是Hash值，临时节点中存储的是key关联的租约id。

这里的模型做了简化，主要想表达在服务注册和发现过程中，服务提供者和消费者都会将自己的IP和	端口等相关信息写入对应的key-value中。存储到注册中心的任何特殊字符都会被编码，比如URL中包含的“/”字符，在etcd3注册中心内部已经调用URLEncode进行特殊字符处理了。

## 构建可运行的注册中心

​	在构建生产可用的版本前，要考虑的因素比较多，比如新的注册中心临时节点如何保活、如何可扩展、如何降低网络拉取压力和如何兼容服务治理平台等因素。在实现etcd注册中心前，首先要考虑使用哪种客户端与etcd server通信，目前虽然采用官方的jetcd作为默认客户端，但提供了 SPI扩展，为将来其他客户端替换它提供了可能(类似在使用Zookeeper时，将curator客户端替换成zkclient)。接下来，我们针对注册中心按照相关扩展逐个探讨。

### 扩展 Transporter 实现

​	为了支持未来采用其他客户端与etcd server交互，我们需要提供一个新的扩展点EtcdTransporter,在注册中心初始化时会通过这个transporter初始化真实的交互cliento我们先看一下这个扩展点定义，如代码清单11-1所示。

![](https://pic.imgdb.cn/item/610949b25132923bf8ae0a05.jpg)

​	在实现扩展点之后，还需要将扩展点通过SPI的方式配置在项目中，ExtensionLoader才能正确查找和加载，将这个实现放到 resource/META-INF/dubbo/internal/org.apache.dubbo.remoting.etcd. EtcdTransporter文件中，文件内容也是键值对形式，其中key代表扩展点名称，value代表扩展点具体实现类，文件内容如下：

​	jetcd=org.apache.dubbo.remoting.etcd.jetcd.JEtcdTransporter

​	因为这个是默认扩展点的实现，可以发现jetcd作为key和代码清单SPI(jetcd)值是一致的，默认会加载当前实现。

### 扩展 RegistryFactory 实现

​	具体扩展点实现之后，我们要考虑在哪里使用，当provider或consumer启动时，会创建注册中心实例RegistryProtocol#getRegistry,这里通过另外一个扩展点加载Factory,这个新的扩展点是RegistryFactory,所有注册中心必须提供一个对应实现，我们先看一下这个新扩展点定义，代码清单11-3所示。

![](https://pic.imgdb.cn/item/61094a335132923bf8afbbd7.jpg)

​	这个扩展点包含SPI(dubbo),代表默认使用基于内存的Dubbo注册中心实现，生产环境不会使用这个实现。这个扩展点只有一个接口方法，返回具体注册中心实例，因此，我们定义的EtcdTransporter 一定在这个接口内部使用。

​	所有扩展RegistryFactory扩展点都应该继承AbstractRegistryFactory类，因为它提供了注册中心创建的cache及JVM销毁时删除注册中心中的服务元数据方法。在AbstractRegistryFactory#getRegistry中会先检查是否已有实例，否则加锁创建具体注册中心。这里有两点值得注意，第一点，在生成cache key的时候，不应该调用注册中心host域名解析，否则可能因为DNS解析慢影响性能。第二点，不解析host是有好处的，运维人员维护etcd节点时，交给具体client处理域名转换的事情。

​	在 AbstractRegistryFactory#getRegistry 中通过 createRegistry 方法创建注册中心，这个是抽象方法，交给开发者实现，因此，在实现etcd时我们需要提供EtcdRegistryFactory对应实现，如代码清单11-4所示。

![](https://pic.imgdb.cn/item/61094a585132923bf8b038a6.jpg)

### 新增 JEtcdClient 实现

​	在讲解具体实现前，我们先看一下开发一个客户端需要具备的一些特性，比如要支持临时/永久节点的创建和删除、获取子节点、查询节点和保活机制等。表11-3列出了 API对应的功能。

![](https://pic.imgdb.cn/item/61094a745132923bf8b09672.jpg)

​	在实现注册中心客户端基础上，不仅要考虑增删改查，还需要考虑watch机制、连接状态变更，主要是为了方便接收etcd服务端推送的服务元数据和连接恢复需要重新注册的本地服务元数据。

​	为了循序渐进地弄明白etcd扩展实现的原理，我们先分析增删改查的处理，然后分析watch机制的实现。所有与注册中心进行交互都在OEtcdClientWrapper中，在构造函数中主要异步创建与etcd server之间的TCP连接，并且初始化RPC调用失败时的重试机制。以临时节点为例，代码清单11-5展示了临时节点的创建过程。

![](https://pic.imgdb.cn/item/61094a945132923bf8b10172.jpg)

​	在①中主要支持与etcd服务端交互的防御性容错，类似zkClient实现，失败时是允许重试的，默认策略是失败最多重试1次，每次重试休眠I秒，防止出现对etcd服务端的冲击，关于重试的设计会在后面分析。在②中进行客户端初始化检查，只有客户端正确初始化才会触发网络调用。在③中会将用户配置的session作为keep-alive时间，默认是30秒。在实现租约保活时,做了一次优化，对同一个应用临时节点做了租约复用。因为每次租约保活会触发一次gRPC调用，租约复用机制大大降低了 stream数量占用，同时降低了 etcd集群内存使用。etcd要求提供一个时间间隔，服务端会返回唯一标识来代表这个租约，需要线程池定期续租。jetcd触发续租逻辑的主要原理是通过2个线程池处理的，第1个线程池定期刷新TTL保持存活，第2个线程池定期检测本地是否过期。在jetcd 0.3.0之后提供了 keep-alive回调机制，允许保活失败执行开发者的自定义逻辑。在④中会构造key-value键值对，键对应Dubbo服务元数据，值是租约。写值时自动关联租约，如果provider节点宕机无法续租，则etcd服务端会自动将节点清除，因此无须担心有垃圾数据的问题。

​	接下来，我们需要解决如何获取直接子节点的问题，因为etcd是平铺的key-value。这里有两种解决办法，第一种是用key对应的value存储所有子节点的值，第二种是把所有元数据作为key,值实际上不存储有意义的元素。如果采用第一种方法，则不可避免地又回到了 ZooKeeper的数据结构，每次单个provider ±线都会触发所有客户端进行拉取，对网络资源消耗较大。因此，这里采用第二种办法，将元数据作为key,并且使用特定的前缀进行区分，比如服务提供者会采用 /dubbo/com.alibaba.demo.HelloService/providers 作为前缀，针对每个接口com. alibaba. demo. HelloService都采用这个前缀进行匹配。

​	这里有一个技巧，当拉取数据时，我们想要的数据是直接子节点的数据，比如接口com.alibaba.demo.HelloService服务提供者，我们需要过滤providers key后面的数据，可以取紧跟在“/”之后的值作为服务元数据，获取直接子节点如代码清单11・6所示。

![](https://pic.imgdb.cn/item/61094af95132923bf8b2619c.jpg)

![](https://pic.imgdb.cn/item/61094b075132923bf8b29300.jpg)

​	在①中将path作为查找直接子节点的开始索引，比如要获取com. alibaba.demo.HelloService对应的providers,这里的path其实对应/dubbo/com.alibaba.demo.HelloService/providers,因此，我们期望的初始索引指向 key /dubbo/com.alibaba.demo.HelloService/providers/最后一个字符“/”。在②中主要是获取服务端返回的key做并行stream计算，这里是安全的。在③中主要是探测返回的key是否是一级子节点，存储到注册中心的数据特殊字符“/”己经做了编码，因此不用担心会遇到这个字符，如果发现不是直接子节点也会快速失败。

​	为了保持完整性，我们需要继续完成当服务下线时节点的删除逻辑，删除节点的核心逻辑非常简单,主要就是利用jetcd客户端delete方法删除对应的key即可，如代码清单11-7所示。

![](https://pic.imgdb.cn/item/61094b365132923bf8b33527.jpg)

​	了解了前面常用的添加、查找和删除的逻辑，我们都会调用RetryLoops.invokeWithRetry进行失败重试，接下来我们看一下这个机制的逻辑结构，如图11.1所示。

![](https://pic.imgdb.cn/item/61094b455132923bf8b36c50.jpg)

![](https://pic.imgdb.cn/item/61094b565132923bf8b3a6bf.jpg)

​	为了防止给注册中心瞬间造成很大的压力，接口指明了是否允许休眠，并给出了每次触发重试策略shouldRetry己经发生的重试次数和收到的时间耗时。接下来我们要实现RetryLoops,主要的逻辑为控制失败是否应该重试，如果第一次就成功则应该立即返回。当发生异常时，经过重试策略判断是否重试，如果应该重试则再次循环执行，RetryLoops核心重试逻辑如代码清单11-9所示。

![](https://pic.imgdb.cn/item/61094b785132923bf8b4195c.jpg)

![](https://pic.imgdb.cn/item/61094b865132923bf8b44b3a.jpg)

​	①：在重试开始前会生成RetryLoops实例记录当前的状态，主要包括第一次重试时间戳、己经重试的次数和是否正常完成等状态。②：主要进入循环判断是否继续执行真实逻辑，第一次状态会放开执行。③：在调用方线程中执行具体逻辑，如果正常完成方法执行，则在下次调用时退出循环④。⑤：负责处理异常调用，是否重试由两部分决定，第1部分只有是可恢复异常才有机会重试，第2部分满足重试策略当前条件，比如最多重试2次，当前虽然调用失败没达到上限也应该继续重试。⑥：是否可恢复异常(比如当前leader正在选举)并且执行重试策略逻辑，如果允许当前失败则会默默丢弃。

​	JEtcdClientWrapper主要负责对etcd服务端进行增删改查，我们要实现的JEtcdClient需要支持watch机制，这个是服务发现重度依赖的特性。当编写本书时，使用的是官方的jetcd客户端(最新版本是0.3.0),因为提供的watch接口是阻塞的且不易于使用，因此借鉴了 gRPC的接口，利用它实现等价的watch功能o jetcd底层实现调用是gRPC协议(HTTP/2.0),因此会自动利用连接复用特性。相比原来etcdv2版本不支持连接复用，大大提升了性能。

​	Streamobserver接口是gRPC中面向stream的核心接口，当服务端推送watch事件时,onNext方法会被触发。因此我们需要实现这个接口用于监听并回调刷新最新的服务可用列表。自己扩展gRPC回调接口有一定的难度，需要理解内部通信机制，这里直接给出EtcdWatcher类的接收通知实现，如代码清单11-10所示。

![](https://pic.imgdb.cn/item/61094bb25132923bf8b4dd95.jpg)

![](https://pic.imgdb.cn/item/61094bbf5132923bf8b50ab0.jpg)

​	①：从gRPC响应中获取响应事件。②：如果是新增事件，则首先通过find判断是否是当前path直接子节点，如果满足则保存在URL列表中，等一批事件处理完后一次性通知。③：通过加锁将服务元数据保存在链表中，这里加锁是因为watch是异步的，由gRPC线程池触发。④：感知服务下线、动态配置或路由删除，同样会将服务从URL列表中移除，modified标志用于记录是否应该通知，这里用整型变量是为了消除ABA问题，如果用boolean变量保存并且发生了 put和delete事件，可能误认为不需要通知。⑤：从URL列表中移除服务元数据，通过⑥通知最新可用的服务，这里的listener 一般是RegistryDirectory,如果是dubbo-admin则对应RegistryServerSync。

​	如果读过ZooKeeper注册中心代码，那么会发现它每次接收事件变更都会触发getChildren拉取全量数据，etcd在这里做了一层cache,不存在也不需要每次拉取，因为通知的时间携带了key信息，而且key就是我们要的服务元数据。接下来我们需要实现发起watch的RPC调用和异常重连的场景，如代码清单11-11所示。

![](https://pic.imgdb.cn/item/61094be45132923bf8b58b7b.jpg)

![](https://pic.imgdb.cn/item/61094bf45132923bf8b5c343.jpg)

​	在①中做了幕等保证，防止对同一个path多次“watch”，如果多次“watch”，首先取消之前的watch,防止多次watcho在②中主要创建本地调用代理存根。在③中会将自己注册为回调通知，注册成功后服务端通知会执行前面的onNext方法接收事件。在④中会发起watch远程调用，在调用前会使用nextRequest创建当前path作为请求对象。我们监听的是前缀，利用了 etcd范围监听机制，使用比较简单，只需要将监听的path最后一个字符值加1即可。⑤：因为监听并不会拉取节点数据，所以第一次要手动拉取一次直接子节点数据。

​	如果RPC发生了异常没有正确处理，则可能会丢失watch,因此任何异常错误gRPC都会回调。nError方法，我们需要在这里保证健壮性，如代码清单11-12所示。

![](https://pic.imgdb.cn/item/61094c195132923bf8b63c1a.jpg)

​	当gRPC发生异常时，通过onError直接调用tryReconnect方法。①：处理严重故障时延迟重试。②：对常规异常进行快速重试。reconnect中的实现非常简单，直接关闭当前watch,然后重新发起一个新的RPC调用进行“watch”。比如正在选举leader,等待较长时间是合适的，采用随机延迟，防止在一瞬间对集群造成较大压力。目前对etcd注册中心创建节点、查询节点、删除节点和对watch机制的说明占用了较多的篇幅，接下来我们会对上层使用注册中心接口 API做进一步探讨。

### 扩展 FailbackRegistry 实现

​	实现具体注册中心时，我们应该最大复用现有注册中心的功能，比如注册失败和订阅失败应该重试，当注册中心“挂掉”后，应用启动应该自动使用cache文件等。这些功能现有的Dubbo框架巳经给我们提供好了，因此我们的注册中心只需要继承FailbackRegistry类并重写具体订阅和注册等逻辑即可。

​	扩展注册中心时，我们主要的聚焦点是实现doSubscribe> doUnsubscribe> doRegister和doUnregistero这四个方法的功能是支持订阅、取消订阅、注册服务元数据和取消注册服务元数据。如果读者理解了第3章ZooKeeper注册中心实现，那么会更容易理解etcd中的实现。

​	在实现数据订阅时，我们必须要兼容服务治理平台(dubbo-admin),服务治理平台启动时接口会传递星号(*),代表订阅所有接口数据。因此在处理服务治理场景中必须递归拉取所有接口，然后订阅接口中包含的服务数据或配置，etcd注册中心订阅逻辑如代码清单11.13所示(部分删减)。

![](https://pic.imgdb.cn/item/61094c565132923bf8b711f1.jpg)

![](https://pic.imgdb.cn/item/61094c735132923bf8b77a37.jpg)

![](https://pic.imgdb.cn/item/61094c895132923bf8b7c8b0.jpg)

​	我们首先分析dubbo-admin逻辑订阅场景，前面提到首先要拉取根节点下对应的所有接口，在①中主要接收根节点推送的接口数据。②：当收到推送数据时需要递归调用接口下的数据（provider、consumers'Configurators和 routers）。③:主要创建根节点数据（默认是/dubbo）。④：在第一次添加watcher监听时会返回当前节点的最新数据。⑤：处理第一次拉取数据并在⑥中发起递归调用监听。其中①和②处理异步推送回的接口信息，然后递归订阅每个接口服务元数据，⑤和⑥主要是对第一次订阅返回的数据进行处理。

​	在处理常规接口和服务治理平台订阅的接口时，首先会获取当前服务元数据URL对应的类别，默认类别是provideto©：接收注册中心主动推送的接口对应的全量数据，主要包括providers >动态配置和路由，但每次推送都是某一个类别的全量数据。⑧：精确匹配服务端元数据（接口、分组和版本等），客户端获取的Invoker也是在这里过滤的。⑨；创建某个接口包含的类别，就是对应在接口下方的直接目录，比如/providerso⑩：监听具体接口下面的分类，第一次监听会主动拉一次最新数据。@：主要实现对返回的数据做过滤和剔除前缀等逻辑。⑫：通知刷新本地服务列表、动态配置或路由，consumer端最常用的是RegistryDirectory实现。为了聚焦关注点，删除了对listener做缓存的代码。

​	在处理完服务订阅时，我们需要实现服务注册，服务注册比较简单，主要就是把URL转换成注册中心对应的path,需要区分当前节点是否是临时节点，如果是临时节点则调用前面的接口createEphemeral实现即可。在当前接口内部会自动封装保活逻辑，如果是永久节点则直接创建即可，不需要保活，etcd服务端会自动持久化。当JVM退出后进行反注册也比较简单，直接调用接口删除注册中心的key即可。

### 编写单元测试

​	不管是编写框架运行代码还是业务代码，详尽的测试再多也不为过。在编写完注册中心实现后，我们进行代码在正确用例和失败用例下的测试。

![](https://pic.imgdb.cn/item/61094cc35132923bf8b898ad.jpg)

## 搭建etcd集群并在Dubbo中运行

​	这里讲解的启动etcd3主要是为了方便本地运行单元测试跟踪内部细节，集群也是在本地启动的伪集群。

​	当前实现针对的是etcd3,需要指定环境变量，在Mac的-/.bash_profile中添加以下内容：

![](https://pic.imgdb.cn/item/61094ce85132923bf8b91e5a.jpg)

### 单机启动etcd

![](https://pic.imgdb.cn/item/61094cfd5132923bf8b96a57.jpg)

### 集群启动etcd

![](https://pic.imgdb.cn/item/61094d2a5132923bf8ba06d5.jpg)

![](https://pic.imgdb.cn/item/61094d395132923bf8ba39da.jpg)

## 小结

​	本章首先介绍了 etcd注册中心元数据的结构设计，etcd的结构比较简单，在v3版本中主要是平铺的key-value,说明了为什么选用元数据作为key的原因。然后介绍了扩展新注册中心要考虑扩展性，我们理解了为什么要新增扩展点并且如何使用扩展点，给出了与注册中心交互的实现，比如临时节点创建和保活、节点删除实现和重新实现watch的机制，详细讲解了 watch中可能出现的异常场景处理，在watch实现中优化了网络拉取。最后我们讲解了完整注册中心订阅的逻辑，需要同时适配服务治理平台、provider和consumer的订阅，受限于篇幅，没有把所有接口实现完整地展现出来，尽量关注最核心和较复杂的逻辑，剩余接口相对简单，读者可以翻阅代码参考。我们也给出了搭建etcd集群的方法，方便本地快速建立环境，动手调试是比较好的学习方式。

# Dubbo服务治理平台

## 服务治理平台总体结构

​	Dubbo有新旧两个服务治理平台，旧的服务治理平台在Dubbo 2.6.0以后就从源码中被移除了，现在已经没有继续维护。新的服务治理平台并没有包含在Dubbo源码中，而是独立的一个仓库，有兴趣的读者可以在GitHub上搜索dubbo-ops来了解详细信息。新的服务治理平台分为前端Web部分和后台部分，并做了前后端分离，前端可以独立启动部署。前端技术使用Vue +Vuetify的组合方式，后台则直接使用了 SpringBooto整个服务治理平台还在开发中，很多特性还未上线，因此本章会优先基于新的服务治理平台讲解，对于新服务治理平台还未实现的功能,将基于旧的服务治理平台讲解。下面看一下新版的服务治理平台的界面，如图12.1所示。

![](https://pic.imgdb.cn/item/61094d995132923bf8bb8069.jpg)

​	顾名思义，服务治理平台包含服务治理的功能，和一般的MVC项目一样，前端通过REST接口请求后端服务，后端Controller收到对应请求后调用Service处理具体逻辑。现有的Controller及作用如表12-1所示。

![](https://pic.imgdb.cn/item/61094dad5132923bf8bbc2b0.jpg)

![](https://pic.imgdb.cn/item/61094db85132923bf8bbe95d.jpg)

每个Impl的Service都有各自的接口，它们总体都继承了一个抽象接口 AbstractServiceo这个抽象类只实现了一个方法，就是返回本地的服务缓存数据。对于还未完成的接口实现，我们将基于旧的服务治理平台讲解，剩余三个实现则基于新的服务治理平台代码讲解。其中，ConfigServicelmpl和UserServicelmpl 在两个版本里都没实现，因此不进行讲解。ConsumerServicelmpl和ProviderServicelmpl的实现原理基本一样，只不过一个是消费者的管理类，另一个是生产者的管理类，因此后续只讲解ProviderServicelmpl。

## 服务治理平台的实现原理

**1 .服务搜索的实现**

​	服务搜索是通过不同的过滤条件，在本地的注册数据缓存里，查找出合适的结果集。因此，我们首先看一下抽象父类是如何获取到注册中心的数据并缓存到本地的。主要是通过一个工具类RegistryServerSync实现的，它继承了两个Spring接口和一个Dubbo注册中心接口，如图12-3所示。

![](https://pic.imgdb.cn/item/61094de85132923bf8bc90a9.jpg)

​	熟悉Spring的读者会知道，继承了 Spring的InitializingBean接口后需要实现afterPropertiesSet()方法，Spring 在所有 Bean 的属性被设置后，调用 RegistryServerSync 实现的 afterPropertiesSet 方法。继承 DisposableBean 接口则需要实现 destroy()方法，Spring容器在释放Bean的时候会调用该方法。

容器在释放Bean的时候会调用该方法。除此之外，RegistryServerSync还继承了 NotifyListener接口，这个是Dubbo注册中心的监听接口，说明RegistryServerSync在监听到注册中心的变化时，会调用自己实现的`notify(List<URL> urls)`方法更新本地的缓存数据。因此，整个流程如下：

​	(1) 在Bean初始化时，在afterPropertiesSet()方法中会订阅注册中心。直接调用注册中心模块的registryService#subscribe订阅，把this传入并作为监听者，因为RegistryServerSync也实现了监听接口。
​	(2) 监听到变化时候，通过`notify(List<URL> urls)`方法更新本地的缓存数据。对于empty协议的变更，如果服务配置的group和version (Dubbo支持一个接口多个版本)的值是*,则清空所有本地的这个节点;如果指定了特定的group和version,则只删除指定的节点。对于非empty协议的变更，则把数据按照类目、ServiceKey两种维度分别保存一份，更新本地缓存。ServiceKey的规则是：group + 7' + 接口名 + + version。
​	(3) Bean被Spring容器销毁时，在destroy()方法中会取消订阅注册中心，直接调用registryService#unsubscribe 取消订阅。获取注册中心的数据，并缓存到本地后，providerServicelmpl或ConsumepServicelmpl的查找就很好实现了，通过查询的参数遍历缓存，过滤出合适的结果即可。现有新版搜索支持根据Service名称、IP地址、服务名称进行搜索；旧版还支持创建、禁用、启用服务，这些特性是通过override特性实现的。

**2. override特性的实现**

​	override特性主要使用在动态参数的更新上，各个节点监听到注册中心的参数发生变化，从而更新本地的参数信息。override类型的URL是以override://开头的，允许整个URL中只有部分属性变化，监听者监听到变化后会做部分更新。override包含以下属性的配置，如表12-2所示。

![](https://pic.imgdb.cn/item/61094e365132923bf8bd9be6.jpg)

​	表12-2中的第一列是前端把数据传到后台时自动绑定到override对象的属性中；最后一列则是该属性保存到注册中心时对应的属性名。

下面我们来看一下override实现的具体操作逻辑：
(1) 新增。把 override 对象转换成 URL,通过 registryservice.register(url)把 URL注册到注册中心。
(2) 修改。根据Hash值找到老的URL,如果没找到则说明数据己经被修改，抛出异常；如果找到了，则先取消注册老的URL,再注册新的URLO
(3) 删除。先根据id获取老的URL,再直接取消注册老的URL。
(4) 启用/禁用。首先根据id获取老的URL,在新URL的params属性里把enabled设置为true或false,然后取消注册老的URL,最后注册新的URL。

**3. route的实现**

![](https://pic.imgdb.cn/item/61094e5e5132923bf8be28a7.jpg)

![](https://pic.imgdb.cn/item/61094e715132923bf8be672c.jpg)

​	最后的conditions的意思就是，调用com.test.xxService服务中所有以find、list、get> is开头的方法，都路由到172.22.3.94、172.22.3.95、172.22.3.96这三个地址中的一个。它们之间使用=>表示路由。最终，整个route对象会转换为以route://开头的URL。增删改查的实现逻辑与override相同，都使用“注册”、"取消注册”方法来实现。

**4. LoadBalance 的实现**

​	如果用户不做任何配置，则默认使用RandomLoadBalance,即加权随机负载算法。用户可以在服务治理平台里修改某个服务的负载均衡策略，其配置参数较少，如表12-4所示。

![](https://pic.imgdb.cn/item/61094e975132923bf8beeebd.jpg)

**5. Weight的实现**

![](https://pic.imgdb.cn/item/61094ea95132923bf8bf29e9.jpg)

​	本章内容较少，由于Dubbo的服务治理平台一直处于半成品状态，实现的方式也比较简单，因此可以讲解的原理不多。总的来说，Dubbo服务治理平台各种功能的实现，都是通过RegistryServerSync工具类把注册中心的数据缓存到本地，然后通过override协议更新到注册中心，订阅者得知URL变更后，自动更新本地的配置缓存，从而实现配置的下发。

# Dubbo未来展望

# Dubbo 未来生态

### 后续发展

​	框架的发展可以推动业务更高速地发展，业务的高速发展很快又会遇到众多新的问题，从而对框架提出新的要求。因此，我们可以从现在互联网业务的趋势来得知未来技术的趋势。
​	首先是业务规模的不断扩大，未来技术的必然趋势是单体应用向微服务的转化，为了方便各种不同语言开发的单体应用，能方便地迁移到分布式应用，Dubbo肯定会支持多语言。Dubbo也会变得更加轻量化，降低框架对业务应用的体积影响。其次，Spring Boot系列无疑是现在Java应用开发的首选，为了符合主流开发习惯，Dubbo还会支持REST及Spring Boot的集成。再次，企业为了进一步降低开发、运维的成本，软件上云会成为趋势。因此Dubbo也会在后续进一步适配Cloud Native, Dubbo Mesh也在探索当中。然后，Dubbo现在在服务化治理方面存在一定的短板，完善服务化治理整体方案，建立Dubbo的生态也会是今后的趋势。最后，高性能是一个框架的立身之本，性能的提升在任何时候都会是关注点，后续Dubbo会不断完善在大规模集群、大流量场景的性能表现，并建立异步编程、benchmark等机制。

​	对于Dubbo后续的发展，我们先从最近的2.7版本特性说起。然后介绍后续Dubbo核心功能的规划、Dubbo扩展生态的规划、Dubbo互通生态的规划和Dubbo云原生的规划。

**1. Dubbo 2.7.x 新特性**

​	2.7.x版本更新的新特性比较多，我们会介绍其核心特性，如JDK8特性的引入及repackage、异步化的支持、元数据的管理、动态配置的管理、路由规则的管理。对于细节性的特性，感兴趣的读者可以去GitHub上查看2.7版本的更新wiki: https://github.com/apache/incubator-dubbo/releases。

(1) JDK8特性的引入及repackage。

​	JDK8新特性的使用。JDK8己经逐渐成为主流使用的JDK, Dubbo 2.7.x中已经全面拥抱JDK8的各种特性。例如：框架接口中直接使用了 default method,不需要先使用一个抽象类来定义默认方法了；使用了 CompletableFuture, Future执行结束后直接触发回调，不需要再做同步等待；还有Optional、Lambda表达式、function等JDK8的各种新特性。

​	框架的repackage。首先，因为Dubbo己经捐献给了 Apache,所以Dubbo的Groupld或Package需要改为org.apache.dubbo。由于2.7版本将会作为Apache毕业的版本，而这些工作必须在毕业之前完成，因此2.7版本中对package进行了重命名。然后就引发出了新的问题，由于JDK8各种新特性的使用和包名的更改，用户使用的配置还是老的，直接升级会导致服务不可用。为了降低用户的升级成本，框架对一些核心API和SPI扩展向下做了兼容，并且生成了专门的兼容模块dubbo-compatibleo dubbo-compatible模块使用的还是老的包路径，其兼容方式主要是继承己经"repackage"的包，但构造函数、接口等保持低版本兼容。

(2) 异步化的支持。

​	2.6. x版本的异步调用比较奇怪，如果直接调用定义为异步的接口则会立即返回null,必须通过RPC上下文来获取Future对象，然后同步get,如代码清单13-1所示。

![](https://pic.imgdb.cn/item/61094f6f5132923bf8c1f3c0.jpg)

​	另外，2.6.x版本不支持服务提供者端的异步，2.7.x版本则支持设置为异步。

(3) 元数据的管理。

​	首先，现有Dubbo注册的元数据数据量很大，服务提供者端注册的参数有30多个，但接近一半是不需要通过注册中心传递的；消费者端注册的参数有20多个，只有个别需要传递给注册中心。其次，由于数据量大，有数据更新时，推送量也会相应增大。然后，Dubbo现在服务信息的更新会把某个接口下的所有服务提供者信息全量拉过来，如果集群规模较大，则整个网络传输量会瞬间激增，让整个网络的延迟增大。最后，Dubbo-OPS有新的需求，其服务测试也需要使用元数据。

基	于以上的问题，2.7.x版本对注册信息进行了简化，减少了注册中心的数据量；etcd注册中心的实现可以完成增量更新的特性。额外的元数据会写入Redis等第三方存储中间件，以此降低注册中心的压力，提升总体服务的性能。新旧元数据管理对比如图13.1所示。

![](https://pic.imgdb.cn/item/61094fa95132923bf8c2c3c1.jpg)

（4）动态配置的管理。

​	Dubbo现阶段的配置基本都是静态的，缺少动态配置的手段，也容易造成不同节点的配置不同的问题。例如：Dubbo配置通常都是写在本地配置文件中的，缺少像Spring Cloud Config 一样的远程配置托管的模式；服务治理平台只有服务级别的配置，SPI等也是预先写好在配置文件中的，不能动态添加、修改。

​	在2.7.x版本中，Dubbo新增了动态配置中心，实现类似Spring Cloud Config 一样的远程配置方式，配置优先级如图13.2所示。

![](https://pic.imgdb.cn/item/61094fcf5132923bf8c355a0.jpg)

​	从图13.2知道，优先级最高的是通过启动参数进行配置的，其次是通过XML或API的方式配置的，然后是本地dubbo.propertis配置文件，最后就是新增的远程配置中心。

​	新的动态配置中心支持配置的动态覆盖与新增，并且还支持应用级别和服务级别的配置，兼容override配置。此外，2.7.x版本中定义了新的SPI接口，开发者可以基于该接口自定义动态配置中心，默认支持Apollo> Nacos> ZooKeeper作为配置中心。

​	综合上面几个特性，我们可以得知，Dubbo从原来一个注册中心，分离出来了注册、配置、元数据三个中心，减轻了老注册中心数据容量、扩展困难的问题，如图13-3所示。

![](https://pic.imgdb.cn/item/61094ff25132923bf8c3d2d5.jpg)

（5）路由规则的管理。

​	路由规则在2.6.x版本中支持的力度不够，只支持服务粒度的路由；支持的方式也不足，如不支持tag类型的路由规则；路由规则还存储在注册中心，造成注册中心的臃肿；一个服务允许设置多条路由规则，导致路由结果非常复杂、难以排查，等等。

​	在2.7.x版本中，对路由规则进行了增强，支持应用级别的路由，也支持tag类型的路由规则；路由规则的存储已经随着三个中心的确立，从注册中心转移到配置中心；每个服务都能对应精确的路由规则等。

**2. Dubbo核心能力规划**

​	Dubbo核心能力主要会向六个方向发展：模块化、元数据、路由策略、大流量、大规模和异步化。

* 模块化。Dubbo现在的通信层与服务治理层的耦合比较严重，如Cluster层中的路由规
  贝叭软负载均衡等，都耦合在框架中，而集群容错层完全可以下沉到sidecar中，框架
  里只保留PRC通信。因此，在后续规划中，会让Dubbo更加模块化，使得框架各个
  层次的能力更加内聚，方便后续的拆分，也为Dubbo Mesh做好准备。
* 元数据。在2.6.x版本中，Dubbo注册中心里包含注册数据、元数据和配置数据。元数据也过于冗长，注册中心过于臃肿，水平扩展能力受限。因此在核心能力规划时，会把现有的注册中心拆分为三个中心：注册中心、元数据中心和配置中心。这一规划己经在2.7.x版本中大致实现。
* 路由策略。随着互联网业务的不断发展，同城多机房、异地多机房等巳经比较常见，服务的数量级也不断上升。Dubbo后续会引入在阿里内部实践广泛的路由策略：多机房、灰度、参数路由等智能化策略，以此来增强现有的路由模块。
* 大规模。业务的发展随之带来服务数量级的不断上升，在超大规模的服务集群中，服务的注册发现、内存占用、海量服务选址对CPU消耗等有很大的挑战。因此，后续Dubbo会对大规模服务集群的各种场景进行针对性的优化。
* 大流量。在大规模集群的同时，也会带来大流量的问题。现在的Dubbo框架还没有完善的熔断、隔离等机制来提升整个集群的总体稳定性。当集群出现问题时，定位故障节点也相对困难。Dubbo在后续的规划中会补齐这些短板。
* 异步化。2.6.x版本的Dubbo框架还未支持CompletableFuture,也没有跨进程的Reactive支持。虽然在2.7.x版本中已经支持了 CompletableFuture,但Reactive还未支持。因此在后续的规划中，会通过这些异步化的方式来提升分布式系统整体的吞吐量和CPU利用率。

**3. Dubbo生态的规划**

​	Dubbo使用微内核+富插件的模式，平等对待任何第三方扩展，因此可以很好地接收其他插件扩展，丰富自身的生态。首先，我们先看一下Dubbo对扩展点生态的未来规划，如图13-4所示，实线表示已经实现，虚线表示还未实现。

我们可以从图13-4得知，Dubbo每一层的扩展点接口将来都会引入或适配第三方的优秀扩
展，以此来丰富Dubbo自身的生态圈。

![](https://pic.imgdb.cn/item/6109506a5132923bf8c58cd6.jpg)

​	对于API层，会使用Reactive进行异步编程，增加框架的CPU使用率，提升整体性能；同时 Spring Cloud Alibaba 也在适配当中，使用 Dubbo 的 RPC 替代 Spring Cloud 的 OpenFeign 是一个不错的选择。
​	对于Registry层，会由现在主流的ZooKeeper、Redis等注册中心，全面支持市面上流行的所有注册中心，如Eureka> etcd> Consul等，让用户可以根据实际业务需求，对注册中心有更多的选择。
​	对于Config层，Dubbo 2.6.x的所有配置信息均写在注册中心，从2.7.x版本开始，配置中心将会独立出去，后续还会支持Apollo> Nacos等热门配置中心。

​	对于容错，Dubbo现在已经支持了 Failfast> Failover等容错策略，但对熔断、平滑限流等却未有良好的支持。虽然有限流的Filter,但只能根据令牌或请求计数，无法应对锯齿形的流量及计算一段时间内的平均请求量。因此后续Dubbo也会接入现在比较成熟的熔断组件Hystrix。

​	对于负载均衡，Dubbo后续会补齐Latency策略的负载均衡，通过计算请求服务器的往返延迟（RTT）,动态地选择延迟最低的节点来处理当前请求。

​	对于路由规则，2.6.x版本仅支持脚本、文件、条件表达式这三种路由规则的方式，能力比较弱小。在大规模应用中，灰度、同机房优先等策略也是必不可少的。因此后续Dubbo会不断
增强路由规则的能力。
​	对于Protocol层，现有的协议也基本能满足需求，但也会继续引入Avro、gRPC等协议的支持。
​	对于Transport层，新出现的HTTP/2、QUIC等协议也会支持。

![](https://pic.imgdb.cn/item/610950c75132923bf8c6e30d.jpg)

## 云原生

​	云原生意味着应用在设计之初，就把最佳运行实践设为在云端。云原生的定义在最近几年一直在不断变化，从Pivotal公司的Matt Stine最初提出来的12个因素，到后面六大特质，云原生的定义各不相同，感兴趣的读者可以自行搜索。

​	详情可见：https://github.com/cncf/toc/blob/master/DEFINITION.mdo
​	在微服务大行其道的今天，为什么又提出了云原生的概念呢？任何技术的演进都是为了解决当下存在的问题，我们先来看一下现在的框架面临了什么挑战。

### 面临的挑战

​	在过去几年，以微服务+容器为核心的互联网技术成为主要趋势，大量的企业开始落地微服务架构。微服务技术相对之前的单体应用，己经有了很先进的服务复用概念，对于可复用的模块还可以封装成jar,以类库的形式提供。在这种看似先进的理念下也存在众多的问题。

**1.类库内容众多，门槛较高**

​	我们以Dubbo框架为例，该框架一共分为十层。如果要用好整个框架，只使用API层是远远不够的，还需要理解下面组件的具体工作逻辑。例如，在集群环境中，用户根据不同的业务场景设置容错的策略，这就需要理解内部的容错机制；如果需要做软负载均衡，那么还需要了解LoadBalance相关的内容。整个开发团队除了要熟悉业务，还需要数量掌握整套框架的最佳实践。

​	业务团队的强项往往是对于业务的理解，并不是在底层框架。在当下，对框架有深入理解的团队，就能开发出稳定、高效的应用；而对于只擅长业务，不擅长底层框架的团队，可能开发出来的应用就比较差。所以说，现有的微服务框架的准入门槛还是比较高的。业务团队迫于业务上线的压力，根本没有足够的时间关心底层实现。业务团队的核心价值在于业务的实现与上线，框架应该服务于业务，而不是最终的目标。因此，一个好的框架，应该让业务团队更加专注于业务，提升其研发效率。

**2. 框架升级极其困难**

​	在现在互联网业务场景中，中大型的业务的服务端数以千计。如果算上客户端与跨语言端，使用同一个框架的地方更是数量惊人。当一个框架需要升级的时候，就会非常痛苦：新老版本依赖的兼容，框架不同版本API的兼容，等等。每次升级各种依赖的变更、错误的处理，常常让业务团队“痛不欲生二因此，更加轻量级、透明的升级方式是优秀框架的追求方向。

**3. 应用的臃肿**

​	随着框架能力的不断增强，一个又一个的依赖被加入应用。很快一个应用的体积就会暴增，其中很大一部分来自框架，而业务代码和框架SDK又会在同一个进程中运行，属于强耦合。即使开发一个很小的应用，也会带上一个非常笨重的框架，微服务一点也不“微”。因此，框架应该尽可能把和业务无关的东西抽离出来，减少整个SDK对业务的影响。

**4. 多语言支持维护成本高**

​	Dubbo现在主要使用在Java类项目的开发中，后续如果需要支持多语言，那么肯定会出现Go版、PHP版、Node版等Dubbo框架。如果按照现在的方式，任何一个新特性的出现或Bug的修复，都需要更新到所有语言的SDK中，开发和测试的工作量都是数倍的提升。因此，如果一个框架可以把一些通用特性抽离出来，每种语言的SDK客户端只保留最低限度的功能，并且这部分的功能很少会修改，则后续的维护与升级就会更高效。

**5. 新旧应用的共存**

​	一家中型传统公司，各种应用数以百计，新旧应用之间的交互非常困难，有可能根本不是基于同一套技术栈完成的。而企业永远都是人手不够，如果把旧应用全部都改造成新的架构，其成本之高显而易见。此外，一段时间后，现在的新架构也成为老架构，是否又要全部推倒重来？在几年之前，企业会通过ESB总线的方式来屏蔽这些接口之间的异同，但其缺点也比较明显。因此，后续的框架应该能让各种技术平台共同演进，并且不需要做很大的改造就能相互兼容工作。

**6. 技术人才的多样化**

​	每种语言都有它的局限性，大型互联网公司存在不同语言技术栈的人才，每种语言开发出来的服务，它们之间如何统一地交互、治理，是对框架的另一大考验。

**7. 服务治理的缺失**

​	在大型互联网公司中，如果技术栈比较零散，那么很容易造成每个团队各自为战，开发适合自己团队的服务治理工具。而这些工具的质量又良莠不齐，点状的服务治理很难做到及时、经济。

### Service Mesh 简介

Service Mesh是什么？我们看一下Linkerd CE0对其的定义：
	A service mesh is a dedicated infrastructure layer for handling service-to-service communication. It's responsible for the reliable delivery of requests through the complex topology of services that comprise a modern, cloud native application. In practice, the
service mesh is typically implemented as an array of lightweight network proxies that are deployed alongside application code, without the application needing to be aware. 

​	Service Mesh是一个用于处理服务间通信的基础设施层，它负责为构建复杂的云原生应用传递可靠的网络请求。在实践中，服务网格通常实现为一组和应用程序部署在一起的轻量级的网络代理，但对应用程序来说是透明的。

​	即框架把服务治理的能力下沉为平台的基础能力，只保留通信部分即可，由sidecar成为代理，负责现在框架大部分能力的管理。例如：服务的发现、集群容错、负载均衡等都从框架中剥离出来，应用不关注请求哪个服务，只需要将请求发送给sidecar,由sidecar来完成后续的全部逻辑。如此一来，支持多种语言的RPC成为可能，框架的升级、应用的瘦身、多语言的支持、服务化门槛的降低、新旧应用的共存等问题都迎刃而解。
​	我们可以把Service Mesh演化为一个层次化、规范化、体系化、无入侵的分布式服务治理平台。其层次化主要表现在对“数据平面”和“控制平面”的切分，我们熟悉的sidecar属于数据平面，而统一管理所有sidecar的控制面板则属于控制平面。其规范化主要表现在，数据平面与控制平面之间通过标准协议进行通信，应用与sidecar> sidecar与sidecar之间的互联互通协议都有对应的标准。其体系化主要表现在，服务发现、熔断、限流、灰度、安全等都统一管理，指标统计、日志等都是全局考虑的。其无入侵主要表现在，sidecar以独立进程的方式存在，不会影响应用进程。

![](https://pic.imgdb.cn/item/610951e25132923bf8cb0fa4.jpg)

​	传统服务通过一个臃肿的SDK进行服务发现和远程调用等。当服务网格化后，所有的请求都由sidecar进行代理，sidecar会控制服务的发现与远程调用，应用只需要使用一个轻量级的SDK完成与sidecar之间的通信即可。

### Dubbo Mesh

​	从官方的说明我们可以得知，Dubbo Mesh现在已经在紧张的研发中，我们在本节来了解一下官方对Dubbo Mesh后续的发展思路。首先，未来Kubernetes肯定是容器管理方面的绝对主流，因此Dubbo Mesh适配Kubernetes是大势所趋。然后，现在市面上己经有相对成熟的Istio和envoy方案，阿里官方也不会重复造轮子，会在其基础上进行二次开发，与主流的开源项目形成合力，源于开源、回馈开源。对于Istio,阿里巴巴会在其基础上封装出Dubbo Control；对于envoy,阿里巴巴会在其基础上完成一个Dubbo Proxyo最后，Dubbo Mesh的开源版本会和阿里巴巴集团内部使用的版本保持一致，不会出现“阉割”开源的情况。

![](https://pic.imgdb.cn/item/610951ff5132923bf8cb7f26.jpg)

## 小结

​	本章主要分析了 Dubbo未来生态的发展方向与云原生。首先介绍了 Dubbo重启开源一年多的现状，列举开源团队在这段时间所做出的努力与成果。其次介绍了最新发布的2.7.x版本的新特性，让读者对2.7.x版本的改动有一个大致的了解。然后介绍了后续Dubbo对于其核心能力的规划，一共分为六大方向。接着介绍了 Dubbo生态的后续规划，通过扩展点兼容第三方服务，丰富整个Dubbo的生态，让用户可以体验各种开箱即用的特性，降低研发成本。最后介绍了后续的云原生趋势，分析了现有框架所面临的挑战，后续Server Mesh会以什么样的方式来解决现有的问题，以及Dubbo Mesh现在的规划与发展。



# 个人源码阅读

## 启动

```java
DubboBootstrap bootstrap = DubboBootstrap.getInstance();
bootstrap.application(new ApplicationConfig("dubbo-demo-api-consumer"))
        .registry(new RegistryConfig("zookeeper://192.128.1.200:2181"))
        .reference(reference)
        .start();
```

### 设置applicationConfig

```java
// ConfigManager
final Map<String, Map<String, AbstractConfig>> configsCache = newMap();
protected <T extends AbstractConfig> T addConfig(AbstractConfig config, boolean unique) {
        if (config == null) {
            return null;
        }
        // ignore MethodConfig
        if (config instanceof MethodConfig) {
            return null;
        }
        return (T) write(() -> {
            Map<String, AbstractConfig> configsMap = configsCache.computeIfAbsent(getTagName(config.getClass()), type -> newMap());
            return addIfAbsent(config, configsMap, unique);
        });
    }
```

```java
// AbstractConfig
public static String getTagName(Class<?> cls) {
        String tag = cls.getSimpleName();
    	// 如果是以某些结尾，去掉后面的后缀 Config Bean ConfigBase
        for (String suffix : SUFFIXES) {
            if (tag.endsWith(suffix)) {
                tag = tag.substring(0, tag.length() - suffix.length());
                break;
            }
        }
        return StringUtils.camelToSplitName(tag, "-");
    }
```

```java
// StringUtils 把ApplicationWow-> application-wow
public static String camelToSplitName(String camelName, String split) {
        if (isEmpty(camelName)) {
            return camelName;
        }
        if (!isWord(camelName)) {
            // convert Ab-Cd-Ef to ab-cd-ef
            if (isSplitCase(camelName, split.charAt(0))) {
                return camelName.toLowerCase();
            }
            // not camel case
            return camelName;
        }

        StringBuilder buf = null;
        for (int i = 0; i < camelName.length(); i++) {
            char ch = camelName.charAt(i);
            if (ch >= 'A' && ch <= 'Z') {
                if (buf == null) {
                    buf = new StringBuilder();
                    if (i > 0) {
                        buf.append(camelName, 0, i);
                    }
                }
                if (i > 0) {
                    buf.append(split);
                }
                buf.append(Character.toLowerCase(ch));
            } else if (buf != null) {
                buf.append(ch);
            }
        }
        return buf == null ? camelName.toLowerCase() : buf.toString().toLowerCase();
    }

```

```java
// ConfigManager
private <C extends AbstractConfig> C addIfAbsent(C config, Map<String, C> configsMap, boolean unique)
            throws IllegalStateException {

        if (config == null || configsMap == null) {
            return config;
        }

        // find by value
        // TODO Is there any problem with ignoring duplicate and equivalent but different ReferenceConfig instances?
        Optional<C> prevConfig = configsMap.values().stream()
                .filter(val -> isEquals(val, config))
                .findFirst();
        if (prevConfig.isPresent()) {
            if (prevConfig.get() == config) {
                // the new one is same as existing one
                return prevConfig.get();
            }

            // ignore duplicated equivalent config
            if (logger.isInfoEnabled()) {
                logger.info("Ignore duplicated config: " + config);
            }
            return prevConfig.get();
        }

        // check unique config
        if (unique && configsMap.size() > 0) {
            C oldOne = configsMap.values().iterator().next();
            String configName = oldOne.getClass().getSimpleName();
            String msgPrefix = "Duplicate Configs found for " + configName + ", only one unique " + configName +
                " is allowed for one application. previous: " + oldOne + ", later: " + config + ". According to config mode [" + configMode + "], ";
            switch (configMode) {
                case STRICT: {
                    if (!isEquals(oldOne, config)) {
                        throw new IllegalStateException(msgPrefix + "please remove redundant configs and keep only one.");
                    }
                    break;
                }
                case IGNORE: {
                    // ignore later config
                    logger.warn(msgPrefix + "keep previous config and ignore later config: " + config);
                    return oldOne;
                }
                case OVERRIDE: {
                    // clear previous config, add new config
                    configsMap.clear();
                    logger.warn(msgPrefix + "override previous config with later config: " + config);
                    break;
                }
            }
        }

        String key = getId(config);
            if (key == null) {
                // generate key for non-default config compatible with API usages
                key = generateConfigId(config);
            }

        C existedConfig = configsMap.get(key);

        if (isEquals(existedConfig, config)) {
            String type = config.getClass().getSimpleName();
            throw new IllegalStateException(String.format("Duplicate %s found, there already has one default %s or more than two %ss have the same id, " +
                    "you can try to give each %s a different id, key: %s, prev: %s, new: %s", type, type, type, type, key, existedConfig, config));
        } else {
            configsMap.put(key, config);
        }
        return config;
    }
```

### 设置RegistryConfig

最终也会到这个方法中

```
protected <T extends AbstractConfig> T addConfig(AbstractConfig config, boolean unique);
```

### **.reference(reference)**

最终也会以ReferenceConfig落到addConfig方法中

### 启动

```java
public DubboBootstrap start() {
        if (started.compareAndSet(false, true)) {
            startup.set(false);
            shutdown.set(false);
            awaited.set(false);

            initialize();
            if (logger.isInfoEnabled()) {
                logger.info(NAME + " is starting...");
            }
            // 1. export Dubbo Services
            exportServices();

            // If register consumer instance or has exported services
            if (isRegisterConsumerInstance() || hasExportedServices()) {
                // 2. export MetadataService
                exportMetadataService();
                // 3. Register the local ServiceInstance if required
                registerServiceInstance();
            }

            referServices();
            if (asyncExportingFutures.size() > 0 || asyncReferringFutures.size() > 0) {
                new Thread(() -> {
                    try {
                        this.awaitFinish();
                    } catch (Exception e) {
                        logger.warn(NAME + " asynchronous export / refer occurred an exception.");
                    }
                    startup.set(true);
                    if (logger.isInfoEnabled()) {
                        logger.info(NAME + " is ready.");
                    }
                    onStart();
                }).start();
            } else {
                startup.set(true);
                if (logger.isInfoEnabled()) {
                    logger.info(NAME + " is ready.");
                }
                onStart();
            }

            if (logger.isInfoEnabled()) {
                logger.info(NAME + " has started.");
            }
        }
        return this;
    }
```

初始化框架

```java
// ApplicationModel
public static void initFrameworkExts() {
        Set<FrameworkExt> exts = ExtensionLoader.getExtensionLoader(FrameworkExt.class).getSupportedExtensionInstances();
        for (FrameworkExt ext : exts) {
            ext.initialize();
        }
    }
```

**ConfigManager初始化**

主要设置congMode

```java
// ConfigManager
@Override
    public void initialize() throws IllegalStateException {
        String configModeStr = null;
        try {
            configModeStr = (String) ApplicationModel.getEnvironment().getConfiguration().getProperty(DUBBO_CONFIG_MODE);
            if (StringUtils.hasText(configModeStr)) {
                this.configMode = ConfigMode.valueOf(configModeStr.toUpperCase());
            }
            logger.info("Dubbo config mode: " + configMode);
        } catch (Exception e) {
            String msg = "Illegal '" + DUBBO_CONFIG_MODE + "' config value [" + configModeStr + "], available values " + Arrays.toString(ConfigMode.values());
            logger.error(msg, e);
            throw new IllegalArgumentException(msg, e);
        }
    }
```

**startConfigCenter**

```java
private void startConfigCenter() {

    // load application config
    loadConfigs(ApplicationConfig.class);

    // load config centers
    loadConfigs(ConfigCenterConfig.class);

    useRegistryAsConfigCenterIfNecessary();

    // check Config Center
    Collection<ConfigCenterConfig> configCenters = configManager.getConfigCenters();
    if (CollectionUtils.isEmpty(configCenters)) {
        ConfigCenterConfig configCenterConfig = new ConfigCenterConfig();
        configCenterConfig.refresh();
        ConfigValidationUtils.validateConfigCenterConfig(configCenterConfig);
        if (configCenterConfig.isValid()) {
            configManager.addConfigCenter(configCenterConfig);
            configCenters = configManager.getConfigCenters();
        }
    } else {
        for (ConfigCenterConfig configCenterConfig : configCenters) {
            configCenterConfig.refresh();
            ConfigValidationUtils.validateConfigCenterConfig(configCenterConfig);
        }
    }

    if (CollectionUtils.isNotEmpty(configCenters)) {
        CompositeDynamicConfiguration compositeDynamicConfiguration = new CompositeDynamicConfiguration();
        for (ConfigCenterConfig configCenter : configCenters) {
            // Pass config from ConfigCenterBean to environment
            environment.updateExternalConfigMap(configCenter.getExternalConfiguration());
            environment.updateAppExternalConfigMap(configCenter.getAppExternalConfiguration());

            // Fetch config from remote config center
            compositeDynamicConfiguration.addConfiguration(prepareEnvironment(configCenter));
        }
        environment.setDynamicConfiguration(compositeDynamicConfiguration);
    }

    configManager.refreshAll();
}
```

## ServiceConfig#Export流程

* 初始化

* 如果没有刷新配置，就刷新

* doExport

  * doExportUrls

    * 往ServiceRepository注册provider

    * 然后根据协议依次注册Service

    * doExportUrlsFor1Protocol

      * ```
        //init serviceMetadata attachments
        serviceMetadata.getAttachments().putAll(map);
        ```

    * 不是远程就要暴露到本地exportLocal

    * DubboProtocol.export

      * openServer
      * ExchangeServer

    * RegistryProtocol.export

      * ZookeeperServiceDiscovery

  * exported

```java
//ServiceRepository
    // services
    private ConcurrentMap<String, ServiceDescriptor> services = new ConcurrentHashMap<>();

    // consumers
    private ConcurrentMap<String, ConsumerModel> consumers = new ConcurrentHashMap<>();

    // providers
    private ConcurrentMap<String, ProviderModel> providers = new ConcurrentHashMap<>();

    // useful to find a provider model quickly with serviceInterfaceName:version
    private ConcurrentMap<String, ProviderModel> providersWithoutGroup = new ConcurrentHashMap<>();
```

# 《深度剖析ApacheDubbo核心技术内幕》Start

基于dubbo2.7

# Dubbo基础

```java
public class ApiProvider {

   public static void main(String[] args) throws IOException {
      // 1.创建ServiceConfig实例
      ServiceConfig<GreetingService> serviceConfig = new ServiceConfig<GreetingService>();
      // 2.设置应用程序配置
      serviceConfig.setApplication(new ApplicationConfig("first-dubbo-provider"));

      // 3.设置服务注册中心信息
      RegistryConfig registryConfig = new RegistryConfig("zookeeper://192.168.170.200:2181");
      serviceConfig.setRegistry(registryConfig);
      // 4.设置接口与实现类
      serviceConfig.setInterface(GreetingService.class);
      serviceConfig.setRef(new GreetingServiceImpl());

      // 5.设置服务分组与版本 
      serviceConfig.setVersion("1.0.0");
      serviceConfig.setGroup("dubbo");

      // 6.设置线程池策略
//    HashMap<String, String> parameters = new HashMap<>();
//    parameters.put("threadpool", "mythreadpool");
//    serviceConfig.setParameters(parameters);

      // 7.导出服务
      serviceConfig.export();

      // 8.挂起线程，避免服务停止
      System.out.println("server is started");
      System.in.read();
   }
}
```

```java
public class APiConsumer {
   public static void main(String[] args) throws InterruptedException {
      // 10.创建服务引用对象实例
      ReferenceConfig<GreetingService> referenceConfig = new ReferenceConfig<GreetingService>();
      // 11.设置应用程序信息
      referenceConfig.setApplication(new ApplicationConfig("first-dubbo-consumer"));
      // 12.设置服务注册中心
      referenceConfig.setRegistry(new RegistryConfig("zookeeper://192.168.170.200:2181"));
      
      //直连测试
      //referenceConfig.setUrl("dubbo://192.168.0.109:20880");
      
      // 13.设置服务接口和超时时间
      referenceConfig.setInterface(GreetingService.class);
      referenceConfig.setTimeout(5000);
      
      // 14.设置自定义负载均衡策略与集群容错策略
//		 referenceConfig.setLoadbalance("myroundrobin");
//		 referenceConfig.setCluster("myCluster");
//		 RpcContext.getContext().set("ip", "30.10.67.231");

      // 15.设置服务分组与版本
      referenceConfig.setVersion("1.0.0");
      referenceConfig.setGroup("dubbo");

      // 16.引用服务
      GreetingService greetingService = referenceConfig.get();

      // 17. 设置隐式参数
      RpcContext.getContext().setAttachment("company", "alibaba");

      // 18调用服务
      System.out.println(greetingService.sayHello("world"));
      
      Thread.currentThread().join();
   }
}
```

**Dubbo 2.6.*版本提供的异步调用**

```java
public class APiAsyncConsumer {
   public static void main(String[] args) throws InterruptedException, ExecutionException {
      //1.创建引用实例，并设置属性
      ReferenceConfig<GreetingService> referenceConfig = new ReferenceConfig<GreetingService>();
      referenceConfig.setApplication(new ApplicationConfig("first-dubbo-consumer"));
      referenceConfig.setRegistry(new RegistryConfig("zookeeper://127.0.0.1:2181"));
      referenceConfig.setInterface(GreetingService.class);
      referenceConfig.setVersion("1.0.0");
      referenceConfig.setGroup("dubbo");
      
      //2. 设置为异步
      referenceConfig.setAsync(true);

      //3. 直接返回null
      GreetingService greetingService = referenceConfig.get();
      System.out.println(greetingService.sayHello("world"));

      //4.等待结果
      java.util.concurrent.Future<String> future = RpcContext.getContext().getFuture();
      System.out.println(future.get());

   }
}
```

​	上面介绍的基于从返回的future对象调用get（）方法实现异步的缺点是当业务线程调用get（）方法后业务线程会被阻塞，这不是我们想要的，所以Dubbo提供了在future对象上设置回调函数的方式，让我们实现真正的异步调用。下面是Consumer模块的APiAsyncConsumerForCallBack类：

```java
public class APiAsyncConsumerForCallBack {
   public static void main(String[] args) throws InterruptedException, ExecutionException {
      // 1.创建引用实例，并设置属性
      ReferenceConfig<GreetingService> referenceConfig = new ReferenceConfig<GreetingService>();
      referenceConfig.setApplication(new ApplicationConfig("first-dubbo-consumer"));
      referenceConfig.setRegistry(new RegistryConfig("zookeeper://127.0.0.1:2181"));
      referenceConfig.setInterface(GreetingService.class);
      referenceConfig.setTimeout(5000);
      referenceConfig.setVersion("1.0.0");
      referenceConfig.setGroup("dubbo");

      // 2. 设置为异步
      referenceConfig.setAsync(true);

      // 3. 直接返回null
      GreetingService greetingService = referenceConfig.get();
      System.out.println(greetingService.sayHello("world"));
      RpcContext.getContext().setAttachment("company", "alibaba");


      // 4.异步执行回调
      ((FutureAdapter) RpcContext.getContext().getFuture()).getFuture().setCallback(new ResponseCallback() {

         @Override
         public void done(Object response) {
            System.out.println("result:" + response);
         }

         @Override
         public void caught(Throwable exception) {
            System.out.println("error:" + exception.getLocalizedMessage());
         }
      });
      
      Thread.currentThread().join();
   }
}
```

**Dubbo 2.7.*版本提供的异步调用**

```java
public class APiAsyncConsumerForCompletableFuture2 {
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		// 1.创建服务引用对象，并设置数据
		ReferenceConfig<GreetingService> referenceConfig = new ReferenceConfig<GreetingService>();
		referenceConfig.setApplication(new ApplicationConfig("first-dubbo-consumer"));
		referenceConfig.setRegistry(new RegistryConfig("zookeeper://192.168.170.200:2181"));
		referenceConfig.setInterface(GreetingService.class);
		referenceConfig.setTimeout(30000);
		referenceConfig.setVersion("1.0.0");
		referenceConfig.setGroup("dubbo");

		// 2. 设置为异步
		referenceConfig.setAsync(true);
		RpcContext.getContext().setAttachment("company", "alibaba");

		// 3. 直接返回null
		GreetingService greetingService = referenceConfig.get();
		System.out.println(greetingService.sayHello("world"));

		// 4.异步执行回调
		CompletableFuture<String> future = RpcContext.getContext().getCompletableFuture();
		future.whenComplete((v, t) -> {
			if (t != null) {
				t.printStackTrace();
			} else {
				System.out.println(v);
			}

		});

		System.out.println("over");
		Thread.currentThread().join();
	}
}
```

## 服务提供端异步执行

**1.基于定义CompletableFuture签名的接口实现异步执行**

```java
public class GrettingServiceAsyncImpl implements GrettingServiceAsync {

   // 1.创建业务自定义线程池
   private final ThreadPoolExecutor bizThreadpool = new ThreadPoolExecutor(8, 16, 1, TimeUnit.MINUTES,
         new SynchronousQueue(), new NamedThreadFactory("biz-thread-pool"),
         new ThreadPoolExecutor.CallerRunsPolicy());

   // 2.创建服务处理接口，返回值为CompletableFuture
   @Override
   public CompletableFuture<String> sayHello(String name) {

      // 2.1 为supplyAsync提供自定义线程池bizThreadpool，避免使用JDK公用线程池(ForkJoinPool.commonPool())
      // 使用CompletableFuture.supplyAsync让服务处理异步化进行处理
      // 保存当前线程的上下文
      RpcContext context = RpcContext.getContext();

      return CompletableFuture.supplyAsync(() -> {
         try {
            Thread.sleep(2000);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
         System.out.println("async return ");
         return "Hello " + name + " " + context.getAttachment("company");
      }, bizThreadpool);
   }
}
```

```java
public class ApiProviderForAsync {

   public static void main(String[] args) throws IOException {

      // 1.创建服务发布实例，并设置
      ServiceConfig<GrettingServiceAsync> serviceConfig = new ServiceConfig<GrettingServiceAsync>();
      serviceConfig.setApplication(new ApplicationConfig("first-dubbo-provider"));
      serviceConfig.setRegistry(new RegistryConfig("zookeeper://192.168.170.200:2181"));
      serviceConfig.setInterface(GrettingServiceAsync.class);
      serviceConfig.setRef(new GrettingServiceAsyncImpl());
      serviceConfig.setVersion("1.0.0");
      serviceConfig.setGroup("dubbo");

      // 2.设置线程池策略
      // HashMap<String, String> parameters = new HashMap<>();
      // parameters.put("threadpool", "mythreadpool");
      // serviceConfig.setParameters(parameters);

      // 3.导出服务
      serviceConfig.export();

      // 4.阻塞线程
      System.out.println("server is started");
      System.in.read();
   }
}
```

```java
public class APiConsumerForProviderAsync {
   public static void main(String[] args) throws InterruptedException {
      //1.创建服务引用实例，并设置
      ReferenceConfig<GrettingServiceAsync> referenceConfig = new ReferenceConfig<GrettingServiceAsync>();
      referenceConfig.setApplication(new ApplicationConfig("first-dubbo-consumer"));
      referenceConfig.setRegistry(new RegistryConfig("zookeeper://127.0.0.1:2181"));
      referenceConfig.setInterface(GrettingServiceAsync.class);
      referenceConfig.setTimeout(5000);
      //referenceConfig.setCluster("myCluster");

      referenceConfig.setVersion("1.0.0");
      referenceConfig.setGroup("dubbo");
      
      //2.服务引用
      GrettingServiceAsync greetingService = referenceConfig.get();
      
      //3.设置隐士参数
      RpcContext.getContext().setAttachment("company", "alibaba");
      //RpcContext.getContext().set("ip", "30.39.148.197");
      
      //4.获取future并设置回调
      CompletableFuture<String> future = greetingService.sayHello("world");
      future.whenComplete((v, t) -> {
         if (t != null) {
            t.printStackTrace();
         } else {
            System.out.println(v);
         }

      });
      
      Thread.currentThread().join();

   }
}     
```

**使用AsyncContext实现异步执行**

```java
public class GrettingServiceAsyncContextImpl implements GrettingServiceRpcContext {

   // 1.创建业务自定义线程池
   private final ThreadPoolExecutor bizThreadpool = new ThreadPoolExecutor(8, 16, 1, TimeUnit.MINUTES,
         new SynchronousQueue(), new NamedThreadFactory("biz-thread-pool"),
         new ThreadPoolExecutor.CallerRunsPolicy());

   // 2.创建服务处理接口，返回值为CompletableFuture
   @Override
   public String sayHello(String name) {

      // 2.1开启异步
      final AsyncContext asyncContext = RpcContext.startAsync();
      bizThreadpool.execute(() -> {
         // 2.2 如果要使用上下文，则必须要放在第一句执行
         asyncContext.signalContextSwitch();
         try {
            Thread.sleep(500);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
         // 2.3写回响应
         asyncContext.write("Hello " + name + " " + RpcContext.getContext().getAttachment("company"));
      });

      return null;
   }
}
```

## 服务消费端泛化调用

​	泛化接口调用方式主要在服务消费端没有API接口类及模型类元（比如入参和出参的POJO类）的情况下使用，其参数及返回值中没有对应的POJO类，所以所有POJO参数均转换为Map表示。使用泛化调用时，服务消费模块不再需要引入SDK二方包。

​	在Dubbo中，根据序列化方式的不同，分为三种泛化调用，分别为true、bean和nativejava。

**1.generic=true方式**

​	在Consumer模块的APiGenericConsumerForTrue类中演示了这种方式，其代码如下：代码1创建引用实例，这里需要注意的是，在泛型调用时，泛型参数固定为GenericService；代码2设置泛化调用类型为true；代码3获取引用，注意泛化值类型固定为GenericService。

```java
public class APiGenericConsumerForTrue {
   public static void main(String[] args) throws IOException {
      // 1.泛型参数固定为GenericService
      ReferenceConfig<GenericService> referenceConfig = new ReferenceConfig<GenericService>();
      referenceConfig.setApplication(new ApplicationConfig("first-dubbo-consumer"));
      referenceConfig.setRegistry(new RegistryConfig("zookeeper://127.0.0.1:2181"));

      referenceConfig.setVersion("1.0.0");
      referenceConfig.setGroup("dubbo");

      // 2. 设置为泛化引用，类型为true
      referenceConfig.setInterface("com.books.dubbo.demo.api.GreetingService");
      referenceConfig.setGeneric(true);

      // 3.用org.apache.dubbo.rpc.service.GenericService替代所有接口引用
      GenericService greetingService = referenceConfig.get();

      // 4.泛型调用， 基本类型以及Date,List,Map等不需要转换，直接调用,如果返回值为POJO也将自动转成Map
      Object result = greetingService.$invoke("sayHello", new String[] { "java.lang.String" },
            new Object[] { "world" });

      System.out.println(JSON.json(result));

      // 5. POJO参数转换为map
      Map<String, Object> map = new HashMap<String, Object>();
      map.put("class", "com.books.dubbo.demo.api.PoJo");
      map.put("id", "1990");
      map.put("name", "jiaduo");

      // 6.发起泛型调用
      result = greetingService.$invoke("testGeneric", new String[] { "com.books.dubbo.demo.api.PoJo" },
            new Object[] { map });
      System.out.println((result));
   }
}
```

**2.generic=bean方式**

```java
public class APiGenericConsumerForBean {
   public static void main(String[] args) throws IOException {
      // 1.泛型参数固定为GenericService
      ReferenceConfig<GenericService> referenceConfig = new ReferenceConfig<GenericService>();
      referenceConfig.setApplication(new ApplicationConfig("first-dubbo-consumer"));
      referenceConfig.setRegistry(new RegistryConfig("zookeeper://192.168.170.200:2181"));

      referenceConfig.setVersion("1.0.0");
      referenceConfig.setGroup("dubbo");
      referenceConfig.setTimeout(30000);
      // 2. 设置为泛化引用，并且泛化类型为bean
      referenceConfig.setInterface("com.books.dubbo.demo.api.GreetingService");
      referenceConfig.setGeneric("bean");

      // 3.用org.apache.dubbo.rpc.service.GenericService替代所有接口引用
      GenericService greetingService = referenceConfig.get();

      // 4.泛型调用，参数使用JavaBean进行序列化
      JavaBeanDescriptor param = JavaBeanSerializeUtil.serialize("world");
      Object result = greetingService.$invoke("sayHello", new String[] { "java.lang.String" },
            new Object[] { param });

      // 5.结果反序列化
      result = JavaBeanSerializeUtil.deserialize((JavaBeanDescriptor) result);
      System.out.println(result);

   }
}
```

**3.generic=nativejava方式**

```java
public class APiGenericConsumerForNativeJava {
   public static void main(String[] args) throws IOException, ClassNotFoundException {
      // 1.泛型参数固定为GenericService
      ReferenceConfig<GenericService> referenceConfig = new ReferenceConfig<GenericService>();
      referenceConfig.setApplication(new ApplicationConfig("first-dubbo-consumer"));
      referenceConfig.setRegistry(new RegistryConfig("zookeeper://127.0.0.1:2181"));

      referenceConfig.setVersion("1.0.0");
      referenceConfig.setGroup("dubbo");

      // 2. 设置为泛化引用，并且泛化类型为nativejava
      referenceConfig.setInterface("com.books.dubbo.demo.api.GreetingService");
      referenceConfig.setGeneric("nativejava");

      // 3.用org.apache.dubbo.rpc.service.GenericService替代所有接口引用
      GenericService greetingService = referenceConfig.get();
      UnsafeByteArrayOutputStream out = new UnsafeByteArrayOutputStream();

      // 4.泛型调用， 需要把参数使用Java序列化为二进制
      ExtensionLoader.getExtensionLoader(Serialization.class)
            .getExtension(Constants.GENERIC_SERIALIZATION_NATIVE_JAVA).serialize(null, out).writeObject("world");

      Object result = greetingService.$invoke("sayHello", new String[] { "java.lang.String" },
            new Object[] { out.toByteArray() });

      // 5.打印结果，需要把二进制结果使用Java反序列为对象
      UnsafeByteArrayInputStream in = new UnsafeByteArrayInputStream((byte[]) result);
      System.out.println(ExtensionLoader.getExtensionLoader(Serialization.class)
            .getExtension(Constants.GENERIC_SERIALIZATION_NATIVE_JAVA).deserialize(null, in).readObject());

   }
}
```

## 服务消费端本地服务mock与服务降级

**1.本地服务mock**

```java
public class APiConsumerMock {
   public static void main(String[] args) throws InterruptedException {
      // 0.创建服务引用对象实例
      ReferenceConfig<GreetingService> referenceConfig = new ReferenceConfig<GreetingService>();
      // 1.设置应用程序信息
      referenceConfig.setApplication(new ApplicationConfig("first-dubbo-consumer"));
      // 2.设置服务注册中心
      referenceConfig.setRegistry(new RegistryConfig("zookeeper://192.168.170.200:2181"));
      // 3.设置服务接口和超时时间
      referenceConfig.setInterface(GreetingService.class);
      referenceConfig.setTimeout(5000);

      // 4.设置服务分组与版本
      referenceConfig.setVersion("1.0.0");
      referenceConfig.setGroup("dubbo");

      // 5设置启动时候不检查服务提供者是否可用
      referenceConfig.setCheck(false);
      referenceConfig.setMock("true");
      // 6.引用服务
      GreetingService greetingService = referenceConfig.get();

      // 7. 设置隐式参数
      RpcContext.getContext().setAttachment("company", "alibaba");

      // 8调用服务
      System.out.println(greetingService.sayHello("world"));

   }
}
```

​	需要注意的是，在执行mock服务实现类mock（）方法前，会先发起远程调用，当远程调用失败（比如服务不存在）时，才会降级执行mock功能。

**2.服务降级**

* force：return策略：当服务调用方设置某个接口的降级策略为这种方式时，服务调用方在调用该接口服务时会直接在客户端内返回设置的mock值，而不会在通过远程调用方式调用服务提供者，比如配置URL为override：//0.0.0.0/com.books.dubbo.demo.api.GreetingService？category=configurators&dynamic=false&application=first-dubbo-consumer&"+"mock="+type+"：return+null&group=dubbo&version=1.0.0"，其中mock=force：return+null表示服务调用方在调用该服务的方法时都直接返回mock的null值，而不发起远程调用。需要注意的是，在URL里要指明是对哪个接口的哪个分组的哪个版本的服务进行降级，另外category必须为configurators，application为你的服务调用方的应用名称，也就是ApplicationConfig的name值。override：//0.0.0.0/标识该降级策略对所有的服务消费者生效。该URL会被保存到ZooKeeper中，持久化存放该设置，当消费端启动时会获取到该配置。
* fail：return策略：表示服务调用方调用服务提供方的服务失败后再返回mock值，与force：return的区别是前者如果调用服务提供者成功，则返回正常的结果，如果调用失败则返回mock的值。这个功能和上一节讲解的本地服务mock功能一致。



​	在Demo的Consumer的APiConsumerMockResult类中演示了如何对一个服务进行降级设置：

```java
public class APiConsumerMockResult {

   public static void mockResult(String type) {
      // (1)获取服务注册中心工厂
      RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class)
            .getAdaptiveExtension();
      // (2)根据zk地址，获取具体的zk注册中心的客户端实例
      Registry registry2 = registryFactory.getRegistry(URL.valueOf("zookeeper://127.0.0.1:2181"));

      // directory.subscribe(subscribeUrl.addParameter(CATEGORY_KEY,
      // PROVIDERS_CATEGORY + "," + CONFIGURATORS_CATEGORY + "," + ROUTERS_CATEGORY));

      // (3)注册降级方案到zk
      registry2.register(URL.valueOf(
            "override://0.0.0.0/com.books.dubbo.demo.api.GreetingService?category=configurators&dynamic=false&application=first-dubbo-consumer&"
                  + "mock=" + type + ":return+null&group=dubbo&version=1.0.0"));

      //(4)取消配置
//    registry2.unregister(URL.valueOf(
//          "override://0.0.0.0/com.books.dubbo.demo.api.GreetingService?category=configurators&dynamic=false&application=first-dubbo-consumer&"
//                + "mock=" + type + ":return+null&group=dubbo&version=1.0.0"));
   }

   public static void main(String[] args) throws InterruptedException {

      // mock=force:result+null;
      mockResult("force");

      // mock=fail:result+null;
      // mockResult("fail");

   }
}
```

**隐式参数传递**

```
RpcContext.getContext().setAttachment("company", "alibaba");
```

## 本地服务暴露与引用

![](https://pic.imgdb.cn/item/610f8d895132923bf82a0b19.jpg)

​	其实Dubbo还提供了一种本地服务暴露与引用的方式，这在同一个JVM进程中同时发布与调用同一个服务时显得比较重要，因为如果当前JVM内要调用的服务在本JVM进程内有提供，则避免了一次远程过程调用，而是直接在JVM内进行通信，如图1.4所示。

![](https://pic.imgdb.cn/item/610f8da25132923bf82a3081.jpg)

```java
public class APiConsumerInJvm {

[...]

   static public void referService() {
      // 10.创建服务引用对象实例
      ReferenceConfig<GreetingService> referenceConfig = new ReferenceConfig<GreetingService>();
      // 12.设置服务注册中心
      referenceConfig.setRegistry(new RegistryConfig("zookeeper://192.168.170.200:2181"));

      // 13.设置服务接口和超时时间
      referenceConfig.setInterface(GreetingService.class);
      referenceConfig.setTimeout(5000);
      referenceConfig.setAsync(true);

      referenceConfig.setVersion("1.0.0");
      referenceConfig.setGroup("dubbo");
      referenceConfig.setCheck(false);

      // 16.引用服务
      GreetingService greetingService = referenceConfig.get();

      // 17. 设置隐式参数
      RpcContext.getContext().setAttachment("company", "alibaba");

      // 18调用服务
      System.out.println(greetingService.sayHello("world"));
   }

   public static void main(String[] args) throws InterruptedException {
      // 导出服务
      exportService();

      // 引用服务
      referService();

      Thread.currentThread().join();
   }
}
```

​	在上面的代码中，exportService（）方法导出服务（包含本地服务与远程服务），referService（）方法进行服务引用，由于在JVM内存在要引用的服务，所以实际是进行了本地调用，大家可以在InjvmInvoker的invoke（）方法内添加断点以便验证是否真的调用了本地服务。

# Dubbo框架内核原理剖析

![](https://pic.imgdb.cn/item/610f8ea05132923bf82bb75f.jpg)

* Service和Config层为API接口层，是为了让Dubbo使用方方便地发布服务和引用服务；对于服务提供方来说需要实现服务接口，然后使用ServiceConfig API来发布该服务；对于服务消费方来说需要使用ReferenceConfig对服务接口进行代理。Dubbo服务发布与引用方可以直接初始化配置类，也可以通过Spring配置自动生成配置类。
* 其他各层均为SPI（Service Provider Interface，服务提供者接口）层，SPI意味着下面各层都是组件化的，是可以被替换的，这也是Dubbo设计比较好的一点。Dubbo增强了JDK中提供的标准SPI功能，在Dubbo中除了Service和Config层，其他各层都是通过实现扩展点接口来提供服务的；Dubbo增强的SPI增加了对扩展点IoC和AOP的支持，一个扩展点可以直接使用setter（）方法注入其他扩展点，并且不会一次性实例化扩展点的所有实现类，这就避免了当扩展点实现类初始化很耗时，但当前还没用上它的功能时仍进行加载实例化这种浪费资源的情况；增强的SPI是在具体用某一个实现类的时候才对具体实现类进行实例化。后续会具体讲解Dubbo增强的SPI的实现原理。
* Proxy服务代理层：该层主要是对服务消费端使用的接口进行代理，把本地调用透明地转换为远程调用；另外对服务提供方的服务实现类进行代理，把服务实现类转换为Wrapper类，这是为了减少反射的调用，后面会具体讲解。Proxy层的SPI扩展接口为ProxyFactory，Dubbo提供的实现类主要有JavassistProxyFactory（默认使用）和JdkProxyFactory，用户可以实现ProxyFactory SPI接口，自定义代理服务层的实现。
* Registry服务注册中心层：服务提供者启动时会把服务注册到服务注册中心，消费者启动时会去服务注册中心获取服务提供者的地址列表，Registry层主要功能是封装服务地址的注册与发现逻辑，扩展接口Registry对应的扩展实现为ZookeeperRegistry、RedisRegistry、MulticastRegistry、DubboRegistry等。扩展接口RegistryFactory对应的扩展接口实现为DubboRegistryFactory、DubboRegistryFactory、RedisRegistryFactory、ZookeeperRegistryFactory。另外，该层扩展接口Directory实现类有RegistryDirectory、StaticDirectory，用来透明地把Invoker列表转换为一个Invoker；用户可以实现该层的一系列扩展接口，自定义该层的服务实现。
* Cluster路由层：封装多个服务提供者的路由规则、负载均衡、集群容错的实现，并桥接服务注册中心；扩展接口Cluster对应的实现类有FailoverCluster（失败重试）、FailbackCluster（失败自动恢复）、FailfastCluster（快速失败）、FailsafeCluster（失败安全）、ForkingCluster（并行调用）等；负载均衡扩展接口LoadBalance对应的实现类为RandomLoadBalance（随机）、RoundRobinLoadBalance（轮询）、LeastActiveLoadBalance（最小活跃数）、ConsistentHashLoadBalance（一致性Hash）等。用户可以实现该层的一系列扩展接口，自定义集群容错和负载均衡策略。
* Monitor监控层：用来统计RPC调用次数和调用耗时时间，扩展接口为MonitorFactory，对应的实现类为DubboMonitorFactroy。用户可以实现该层的MonitorFactory扩展接口，实现自定义监控统计策略。
* Protocol远程调用层：封装RPC调用逻辑，扩展接口为Protocol，对应实现有RegistryProtocol、DubboProtocol、InjvmProtocol等。
* Exchange信息交换层：封装请求响应模式，同步转异步，扩展接口为Exchanger，对应的扩展实现有HeaderExchanger等。
* Transport网络传输层：Mina和Netty抽象为统一接口。扩展接口为Channel，对应的实现有NettyChannel（默认）、MinaChannel等；扩展接口Transporter对应的实现类有GrizzlyTransporter、MinaTransporter、NettyTransporter（默认实现）；扩展接口Codec2对应的实现类有DubboCodec、ThriftCodec等。
* Serialize数据序列化层：提供可以复用的一些工具，扩展接口为Serialization，对应的扩展实现有DubboSerialization、FastJsonSerialization、Hessian2Serialization、JavaSerialization等，扩展接口ThreadPool对应的扩展实现有FixedThreadPool、CachedThreadPool、LimitedThreadPool等。

## Dubbo远程调用细节

**服务提供者暴露一个服务的概要过程**

![](https://pic.imgdb.cn/item/610f948e5132923bf83f3e84.jpg)

​	首先，ServiceConfig类引用对外提供服务的实现类ref（如GreetingServiceImpl），然后通过ProxyFactory接口的扩展实现类的getInvoker（）方法使用ref生成一个AbstractProxyInvoker实例，到这一步就完成了具体服务到Invoker的转化。接下来就是Invoker转换到Exporter的过程。Dubbo协议的Invoker转为Exporter发生在DubboProtocol类的export（）方法中，Dubbo处理服务暴露的关键就在Invoker转换到Exporter的过程中，在这个过程中会先启动Netty Server监听服务连接，然后将服务注册到服务注册中心。

![](https://pic.imgdb.cn/item/610f94d85132923bf8404ba3.jpg)

​	首先ReferenceConfig类的init（）方法调用Protocol扩展接口实现类的refer（）方法生成Invoker实例（如图2.3中的左上角部分），这是服务消费的关键。接下来把Invoker转换为客户端需要的接口（如GreetingService）。

​	Dubbo协议的Invoker转换为客户端需要的接口，发生在ProxyFactory接口的扩展实现类的getProxy（）方法中，它主要是使用代理对服务接口的调用转换为对Invoker的调用。

## Dubbo的适配器原理

​		首先我们看看什么是适配器模式，比如Dubbo提供的扩展接口Protocol，其定义如下：

![](https://pic.imgdb.cn/item/610f95155132923bf8411d3e.jpg)

​	Dubbo会使用我们下一节将要讲解的动态编译技术为接口Protocol生成一个适配器类Protocol$Adaptive的对象实例，在Dubbo框架中需要使用Protocol的实例时，实际上就是使用Protocol$Adaptive的对象实例来获取具体的SPI实现类的，其代码如下：

![](https://pic.imgdb.cn/item/610f95345132923bf8418b8c.jpg)

​	当调用protocol.export（wrapperInvoker）时，实际是调用Protocol$Adaptive的对象实例的export（）方法，然后后者根据wrapperInvoker中URL里面的协议类型参数执行；代码2使用Dubbo增强SPI方法getExtension（）获取对应的SPI实现类，然后调用代码3来执行具体SPI实现类的export（）方法。

​	适配器类Protocol$Adaptive会根据传递的协议参数的不同，加载不同的Protocol的SPI实现。

​	其实在Dubbo框架中，框架会给每个SPI扩展接口动态生成一个对应的适配器类，并根据参数来使用增强SPI以选择不同的SPI实现。比如扩展接口ProxyFactory的适配器类为ProxyFactory$Adaptive，其根据参数proxy来选择使用JdkProxyFactory还是使用JavassistProxyFactory做代理工厂；扩展接口Registry的适配器类Registry$Adaptive则根据参数register来决定使用ZookeeperRegistry、RedisRegistry、MulticastRegistry、DubboRegistry中的哪一个作为服务注册中心等。

## Dubbo的动态编译原理

​	上一节我们提到，在Dubbo框架中框架会给每个SPI扩展接口动态生成一个对应的适配器类，那么如何生成呢？这里就使用了动态编译技术，在Dubbo中提供了一个Compiler的SPI：

![](https://pic.imgdb.cn/item/610f960a5132923bf8446652.jpg)

​	我们打开ExtensionLoader的createAdaptiveExtensionClass（）方法，就是该方法将源文件动态编译为Class对象的，有了Class对象后，我们就可以使用newInstance（）方法创建对象实例了：

![](https://pic.imgdb.cn/item/610f96375132923bf844f376.jpg)

​	代码1是根据SPI扩展接口生成其对应的适配器类的源码，其返回的是一个字符串，比如对于Protocol扩展接口来说，返回的字符串内容为：

​	Dubbo框架会为每个扩展接口生成其对应的适配器类的源码，然后选择具体的动态编译类的扩展实现对源码进行编译以生成适配器类的Class对象，然后就可以调用Class对象的newInstance（）方法生成扩展接口对应的适配器类的实例。

## Dubbo增强SPI

### JDK标准SPI

​	JDK中的SPI是面向接口编程的，服务规则提供者会在JRE的核心API里提供服务访问接口，而具体实现则由其他开发商提供。

​	例如，如果规范制定者在rt.jar包里定义了数据库的驱动接口java.sql.Driver，那么MySQL实现的开发商则会在MySQL的驱动包的META-INF/services文件夹下建立名称为java.sql.Driver的文件，文件内容就是MySQL对java.sql.Driver接口的实现类，如图2.4所示。

**原理**

​	Java核心API（比如rt.jar包）是使用Bootstrap ClassLoader类加载器加载的，而用户提供的Jar包是由AppClassLoader加载的。如果一个类由类加载器加载，那么这个类依赖的类也是由相同的类加载器加载的。

​	用来搜索开发商提供的SPI扩展实现类的API类（ServiceLoader）是使用Bootstrap ClassLoader加载的，那么ServiceLoader里面依赖的类应该也是由Bootstrap ClassLoader加载的。而上面说了用户提供的包含SPI实现类的Jar包是由AppClassLoader加载的，所以这就需要一种违反双亲委派模型的方法，线程上下文类加载器ContextClassLoader就是用来解决这个问题的。

![](https://pic.imgdb.cn/item/610f98c45132923bf84d3d73.jpg)

​	下面我们看看ServiceLoader的load（）方法源码。

![](https://pic.imgdb.cn/item/610f99175132923bf84e58e4.jpg)

​	代码5获取了当前线程上下文加载器，这里是AppClassLoader。

​	代码6将该类加载器传递给新构造的ServiceLoader的成员变量loader。那么这个loader在什么时候使用呢？下面我们看看LazyIterator的next（）方法。

![](https://pic.imgdb.cn/item/610f99375132923bf84ed38d.jpg)

​	代码7使用loader也就是AppClassLoader加载具体的驱动实现类的Class对象，代码8则使用Class对象调用newInstance（）方法来创建对象实例。至于cn是怎么来的，读者可以参见LazyIterator的hasNext（）方法：

![](https://pic.imgdb.cn/item/610f99c65132923bf8509c15.jpg)

## 增强SPI原理

* JDK标准的SPI会一次性实例化扩展点的所有实现，如果有些扩展实现初始化很耗时，但又没用上，那么加载就很浪费资源。
* 如果扩展点加载失败，是不会友好地向用户通知具体异常的。比如：对于JDK标准的ScriptEngine来说，如果Ruby ScriptEngine因为所依赖的jruby.jar不存在，导致Ruby ScriptEngine类加载失败，那么这个失败原因就被隐藏了，当用户执行Ruby脚本时，会报空指针异常，而不是报Ruby ScriptEngine不存在。
* 增加了对扩展点IoC和AOP的支持，一个扩展点可以直接使用setter（）方法注入其他扩展点，也可以对扩展点使用Wrapper类进行功能增强。



​	本节我们结合服务提供者配置类ServiceConfig来讲解如何使用增强SPI加载扩展接口Protocol的实现类，在ServiceConfig类中，有如下代码：

![](https://pic.imgdb.cn/item/610f9a095132923bf8516047.jpg)



​	这里的ExtensionLoader类似JDK标准SPI里的ServiceLoader类，代码ExtensionLoader.getExtensionLoader（Protocol.class）.getAdaptiveExtension（）的作用是获取Protocol接口的适配器类，在Dubbo中每个扩展接口都有一个对应的适配器类，前面所述这个适配器类是动态生成的一个类，这里我们给出Protocol扩展接口对应的适配器类的代码：

![](https://pic.imgdb.cn/item/610f9a325132923bf851e4bc.jpg)

​	所以当我们调用protocol.export（invoker）方法的时候实际调用的是动态生成的Protocol$Adaptive实例的export（invoker）方法，其内部代码1首先获取参数里的URL对象，然后从URL对象里获取用户设置的协议（Protocol）的实现类的名称，然后调用代码2根据名称获取具体的Protocol协议的实现类（后面我们会知道获取的是被使用Wrapper类增强后的实现类），最后代码3具体调用Protocol协议的实现类的export（invoker）方法。

​	下面我们结合时序图（见图2.5）来讲解ExtensionLoader的getAdaptiveExtension（）方法是如何动态生成扩展接口对应的适配器类，以及getExtension（）方法如何根据扩展实现类的名称找到对应的扩展实现类的。

![](https://pic.imgdb.cn/item/610f9aca5132923bf853c425.jpg)

![](https://pic.imgdb.cn/item/610f9b9f5132923bf8565906.jpg)

​	从上面的代码可以看到，第一次访问某个扩展接口时需要新建一个对应的ExtensionLoader并放入缓存，后面就直接从缓存中获取。

​	步骤2获取当前扩展接口对应的适配器对象，getAdaptiveExtension（）方法的代码如下：

```java
public T getAdaptiveExtension() {
    Object instance = cachedAdaptiveInstance.get();
    if (instance == null) {
        if (createAdaptiveInstanceError != null) {
            throw new IllegalStateException("Failed to create adaptive instance: " +
                    createAdaptiveInstanceError.toString(),
                    createAdaptiveInstanceError);
        }

        synchronized (cachedAdaptiveInstance) {
            instance = cachedAdaptiveInstance.get();
            if (instance == null) {
                try {
                    instance = createAdaptiveExtension();
                    cachedAdaptiveInstance.set(instance);
                } catch (Throwable t) {
                    createAdaptiveInstanceError = t;
                    throw new IllegalStateException("Failed to create adaptive instance: " + t.toString(), t);
                }
            }
        }
    }

    return (T) instance;
}
```

​	上面的代码通过双重锁检查创建cachedAdaptiveInstance对象，接口对应的适配器对象就保存到这个对象里。

```java
private T createAdaptiveExtension() {
    try {
        return injectExtension((T) getAdaptiveExtensionClass().newInstance());
    } catch (Exception e) {
        throw new IllegalStateException("Can't create adaptive extension " + type + ", cause: " + e.getMessage(), e);
    }
}
```

​	这里首先调用了步骤4的getAdaptiveExtensionClass（）.newInstance（）以获取适配器对象的一个实例，然后调用步骤7的injectExtension（）方法进行扩展点相互依赖注入（扩展点之间的依赖自动注入）。下面首先看看步骤4的getAdaptiveExtensionClass（）方法是如何动态生成适配器类的Class对象的：

```java
private Class<?> getAdaptiveExtensionClass() {
    getExtensionClasses();
    if (cachedAdaptiveClass != null) {
        return cachedAdaptiveClass;
    }
    return cachedAdaptiveClass = createAdaptiveExtensionClass();
}
```

​	上面的代码首先调用了步骤5的getExtensionClasses（）方法获取了该扩展接口所有实现类的Class对象，然后调用了步骤6的createAdaptiveExtensionClass（）方法创建具体的适配器对象的Class对象。前面我们讲解过createAdaptiveExtensionClass（）方法，该方法根据字符串代码生成适配器的Class对象并返回，然后通过getAdaptiveExtensionClass（）.newInstance（）创建适配器类的一个对象实例。至此扩展接口的适配器对象已经创建完毕。

​	下面，我们在看步骤7之前，先看看步骤5的getExtensionClasses（）方法是如何加载扩展接口的所有实现类的Class对象的。其内部最终调用了loadExtensionClasses（）方法进行加载，loadExtensionClasses（）方法的代码如下：

```java
private Map<String, Class<?>> loadExtensionClasses() {
    cacheDefaultExtensionName();

    Map<String, Class<?>> extensionClasses = new HashMap<>();

    for (LoadingStrategy strategy : strategies) {
        loadDirectory(extensionClasses, strategy.directory(), type.getName(), strategy.preferExtensionClassLoader(), strategy.overridden(), strategy.excludedPackages());
        loadDirectory(extensionClasses, strategy.directory(), type.getName().replace("org.apache", "com.alibaba"), strategy.preferExtensionClassLoader(), strategy.overridden(), strategy.excludedPackages());
    }

    return extensionClasses;
}
private void cacheDefaultExtensionName() {
    final SPI defaultAnnotation = type.getAnnotation(SPI.class);
    if (defaultAnnotation == null) {
        return;
    }

    String value = defaultAnnotation.value();
    if ((value = value.trim()).length() > 0) {
        String[] names = NAME_SEPARATOR.split(value);
        if (names.length > 1) {
            throw new IllegalStateException("More than 1 default extension name on extension " + type.getName()
                    + ": " + Arrays.toString(names));
        }
        if (names.length == 1) {
            cachedDefaultName = names[0];
        }
    }
}
```

​	拿Protocol协议来说，这里SPI被注解为@SPI（"dubbo"），这里的cachedDefaultName就是dubbo。然后，loadDirectory（）方法到META-INF/dubbo/internal/、META-INF/dubbo/、META-INF/services/目录下去加载具体的扩展实现类。

​	步骤7的injectExtension（）方法进行扩展点实现类相互依赖自动注入（IoC功能）：

<<<<<<< HEAD
```java
private T injectExtension(T instance) {

    if (objectFactory == null) {
        return instance;
    }

    try {
        for (Method method : instance.getClass().getMethods()) {
            if (!isSetter(method)) {
                continue;
            }
            /**
             * Check {@link DisableInject} to see if we need auto injection for this property
             */
            if (method.getAnnotation(DisableInject.class) != null) {
                continue;
            }
            Class<?> pt = method.getParameterTypes()[0];
            if (ReflectUtils.isPrimitives(pt)) {
                continue;
            }

            try {
                String property = getSetterProperty(method);
                Object object = objectFactory.getExtension(pt, property);
                if (object != null) {
                    method.invoke(instance, object);
                }
            } catch (Exception e) {
                logger.error("Failed to inject via method " + method.getName()
                        + " of interface " + type.getName() + ": " + e.getMessage(), e);
            }

        }
    } catch (Exception e) {
        logger.error(e.getMessage(), e);
    }
    return instance;
}
```

​	至此，ExtensionLoader的getAdaptiveExtension（）方法是如何动态生成扩展接口对应的适配器类以及如何加载扩展接口的实现类的Class对象的就讲解完了，下面我们看看getExtension（）方法是如何根据扩展实现类的名称找到对应的实现类的：

```java
public T getExtension(String name, boolean wrap) {
    if (StringUtils.isEmpty(name)) {
        throw new IllegalArgumentException("Extension name == null");
    }
    if ("true".equals(name)) {
        return getDefaultExtension();
    }
    final Holder<Object> holder = getOrCreateHolder(name);
    Object instance = holder.get();
    if (instance == null) {
        synchronized (holder) {
            instance = holder.get();
            if (instance == null) {
                instance = createExtension(name, wrap);
                holder.set(instance);
            }
        }
    }
    return (T) instance;
}
```

```java
private T createExtension(String name, boolean wrap) {
    Class<?> clazz = getExtensionClasses().get(name);
    if (clazz == null || unacceptableExceptions.contains(name)) {
        throw findException(name);
    }
    try {
        T instance = (T) EXTENSION_INSTANCES.get(clazz);
        if (instance == null) {
            EXTENSION_INSTANCES.putIfAbsent(clazz, clazz.getDeclaredConstructor().newInstance());
            instance = (T) EXTENSION_INSTANCES.get(clazz);
        }
        injectExtension(instance);


        if (wrap) {

            List<Class<?>> wrapperClassesList = new ArrayList<>();
            if (cachedWrapperClasses != null) {
                wrapperClassesList.addAll(cachedWrapperClasses);
                wrapperClassesList.sort(WrapperComparator.COMPARATOR);
                Collections.reverse(wrapperClassesList);
            }

            if (CollectionUtils.isNotEmpty(wrapperClassesList)) {
                for (Class<?> wrapperClass : wrapperClassesList) {
                    Wrapper wrapper = wrapperClass.getAnnotation(Wrapper.class);
                    if (wrapper == null
                            || (ArrayUtils.contains(wrapper.matches(), name) && !ArrayUtils.contains(wrapper.mismatches(), name))) {
                        instance = injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance));
                    }
                }
            }
        }

        initExtension(instance);
        return instance;
    } catch (Throwable t) {
        throw new IllegalStateException("Extension instance (name: " + name + ", class: " +
                type + ") couldn't be instantiated: " + t.getMessage(), t);
    }
}
```

​	上面我们讲解的getExtension（String name）方法只会加载某一个扩展接口实现的Class对象的实例，但在有些情况下我们需要全部创建，比如ProtocolFilterWrapper类中的buildInvokerChain（）方法在建立Filter责任链时，需要把属于某一个group的所有Filter都放到责任链里，其是通过如下方式来获取属于某个组的Filter扩展实现类的：

![](https://pic.imgdb.cn/item/610fd2e75132923bf8e31132.jpg)

​	比如，当服务提供端启动时只会加载group为provider的Filter扩展实现类：

![](https://pic.imgdb.cn/item/610fd30d5132923bf8e37739.jpg)

​	当消费端启动时只会加载group为consumer的Filter扩展实现类：

![](https://pic.imgdb.cn/item/610fd32d5132923bf8e3c869.jpg)

​	另外还需要注意，并不是所有属于某个group的Filter都会被加载，还需要看其设置的value的值是否在URL里（用户是否设置了该value的属性），比如ActiveLimitFilter在默认情况下是不会在服务消费端加载到Filter链的，只有当消费端设置了并发活跃数actives属性时才会（设置后actives属性就会出现在URL里了），具体内容可以参见9.1节。这里我们以加载Filter为例看看getActivateExtension（）方法的实现原理：

```java
public List<T> getActivateExtension(URL url, String[] values, String group) {
    List<T> activateExtensions = new ArrayList<>();
    // solve the bug of using @SPI's wrapper method to report a null pointer exception.
    TreeMap<Class, T> activateExtensionsMap = new TreeMap<>(ActivateComparator.COMPARATOR);
    List<String> names = values == null ? new ArrayList<>(0) : asList(values);
    if (!names.contains(REMOVE_VALUE_PREFIX + DEFAULT_KEY)) {
        if (cachedActivateGroups.size() == 0) {
            synchronized (cachedActivateGroups) {
                if (cachedActivateGroups.size() == 0) {
                    // 获取Filter所有的扩展实现
                    getExtensionClasses();
                    for (Map.Entry<String, Object> entry : cachedActivates.entrySet()) {
                        // 遍历所有扩展实现
                        String name = entry.getKey();
                        Object activate = entry.getValue();

                        String[] activateGroup, activateValue;

                        if (activate instanceof Activate) {
                            // 如果含有注解@Activate，则获取注解的group和value值
                            activateGroup = ((Activate) activate).group();
                            activateValue = ((Activate) activate).value();
                        } else if (activate instanceof com.alibaba.dubbo.common.extension.Activate) {
                            activateGroup = ((com.alibaba.dubbo.common.extension.Activate) activate).group();
                            activateValue = ((com.alibaba.dubbo.common.extension.Activate) activate).value();
                        } else {
                            continue;
                        }
                        cachedActivateGroups.put(name, new HashSet<>(Arrays.asList(activateGroup)));
                        cachedActivateValues.put(name, activateValue);
                    }
                }
            }
        }

        cachedActivateGroups.forEach((name, activateGroup)->{
            // 如果扩展的实现组和传递的group与我们传递的匹配，value值在URL存在。
            if (isMatchGroup(group, activateGroup)
                    && !names.contains(name)
                    && !names.contains(REMOVE_VALUE_PREFIX + name)
                    && isActive(cachedActivateValues.get(name), url)) {

                activateExtensionsMap.put(getExtensionClass(name), getExtension(name));
            }
        });

        if (!activateExtensionsMap.isEmpty()) {
            activateExtensions.addAll(activateExtensionsMap.values());
        }
    }
    List<T> loadedExtensions = new ArrayList<>();
    for (int i = 0; i < names.size(); i++) {
        String name = names.get(i);
        if (!name.startsWith(REMOVE_VALUE_PREFIX)
                && !names.contains(REMOVE_VALUE_PREFIX + name)) {
            if (DEFAULT_KEY.equals(name)) {
                if (!loadedExtensions.isEmpty()) {
                    activateExtensions.addAll(0, loadedExtensions);
                    loadedExtensions.clear();
                }
            } else {
                loadedExtensions.add(getExtension(name));
            }
        }
    }
    if (!loadedExtensions.isEmpty()) {
        activateExtensions.addAll(loadedExtensions);
    }
    return activateExtensions;
}
```

​	如果是则还要看扩展接口实现的注解中的value值是否在URL中（这是isActive（）方法所做的事情），如下列代码所示：

```java
private boolean isActive(String[] keys, URL url) {
    if (keys.length == 0) {
        return true;
    }
    for (String key : keys) {
        // @Active(value="key1:value1, key2:value2")
        String keyValue = null;
        if (key.contains(":")) {
            String[] arr = key.split(":");
            key = arr[0];
            keyValue = arr[1];
        }

        String realValue = url.getParameter(key);
        if (StringUtils.isEmpty(realValue)) {
            realValue = url.getAnyMethodParameter(key);
        }
        if ((keyValue != null && keyValue.equals(realValue)) || (keyValue == null && ConfigUtils.isNotEmpty(realValue))) {
            return true;
        }
    }
    return false;
}
```

​	上面的代码遍历当前扩展实现的value值，如果发现某一个值在URL中，则返回true，否则返回false。

## 扩展点的自动包装

​	在Spring AOP中，我们可以使用多个切面对指定类的方法进行增强，在Dubbo中也提供了类似的功能。在Dubbo中你可以指定多个Wrapper类对指定的扩展点的实现类的方法进行增强。

​	如下面的代码所示，当执行protocol.export（wrapperInvoker）方法时，实际调用的是适配器Protocol$Adaptive的export（）方法，如果URL对象里面的protocol为dubbo，那么在没有扩展点自动包装时，protocol.export（）方法返回的就是DubboProtocol的对象。

![](https://pic.imgdb.cn/item/610fd5345132923bf8e8ca4f.jpg)

​	而在真正的情况下，Dubbo里使用ProtocolFilterWrapper、ProtocolListenerWrapper等Wrapper类对DubboProtocol对象进行包装增强。

​	ProtocolFilterWrapper、ProtocolListenerWrapper、DubboProtocol三个类都有一个拷贝构造函数，这个拷贝构造函数的参数就是扩展接口Protocol。所谓包装，其含义如下：

![](https://pic.imgdb.cn/item/610fd5875132923bf8e9997d.jpg)

​	比如这里会进行两次包装，第一次首先使用ProtocolListenerWrapper类对DubboProtocol进行包装，这时ProtocolListenerWrapper类里的impl就是DubboProtocol，然后第二次使用ProtocolFilterWrapper对ProtocolListenerWrapper进行包装，也就是说ProtocolFilterWrapper里的impl是ProtocolListenerWrapper，这时会调用适配器Protocol$Adaptive的export（）方法，如果URL对象里面的protocol为dubbo，那么在扩展点自动包装时，protocol.export返回的就是ProtocolFilterWrapper的实例了。

​	下面我们看看在Dubbo增强的SPI中如何去收集这些包装类，以及使用包装类对SPI实现类的自动包装（AOP功能）是如何实现的。

​	其实，上一节所讲的getExtensionClasses里面的loadDirectory（）方法除了加载扩展接口的所有实现类的Class对象，还对包装类（Wrapper类）进行了收集，下面我们看看loadDirectory-＞loadResource中的loadClass（）方法的代码：

```java
private void loadClass(Map<String, Class<?>> extensionClasses, java.net.URL resourceURL, Class<?> clazz, String name,
                       boolean overridden) throws NoSuchMethodException {
    // 类型不匹配则抛出异常
    if (!type.isAssignableFrom(clazz)) {
        throw new IllegalStateException("Error occurred when loading extension class (interface: " +
                type + ", class line: " + clazz.getName() + "), class "
                + clazz.getName() + " is not subtype of interface.");
    }
    // 适配类，调用cacheAdaptiveClass缓存
    if (clazz.isAnnotationPresent(Adaptive.class)) {
        cacheAdaptiveClass(clazz, overridden);
    } else if (isWrapperClass(clazz)) {
        // Wrapper类， 调用cacheWrapperClass缓存
        cacheWrapperClass(clazz);
    } else {
        // 剩下的则是扩展实现类
        clazz.getConstructor();
        if (StringUtils.isEmpty(name)) {
            name = findAnnotationName(clazz);
            if (name.length() == 0) {
                throw new IllegalStateException("No such extension name for the class " + clazz.getName() + " in the config " + resourceURL);
            }
        }

        String[] names = NAME_SEPARATOR.split(name);
        if (ArrayUtils.isNotEmpty(names)) {
            cacheActivateClass(clazz, names[0]);
            for (String n : names) {
                cacheName(clazz, n);
                saveInExtensionClass(extensionClasses, clazz, n, overridden);
            }
        }
    }
}
private boolean isWrapperClass(Class<?> clazz) {
        try {
            clazz.getConstructor(type);
            return true;
        } catch (NoSuchMethodException e) {
            return false;
        }
    }
```

​	上面的代码使用isWrapperClass（）方法方法判断clazz是否为Wrapper类，其中调用了clazz.getConstructor（type）方法来判断SPI实现类clazz是否有参数类型为type的构造函数。如果没有，则直接抛出异常NoSuchMethodException，该异常被catch块捕获了，然后返回false；如果有，则说明clazz类为Wrapper类，此时返回true并调用cacheWrapperClass（）方法将Wrapper类的Class对象收集起来放入cachedWrapperClasses集合。至此，Wrapper类收集完毕。

​	而对扩展实现类使用收集的Wrapper类进行自动包装是在createExtension（）方法里完成的：

![](https://pic.imgdb.cn/item/610fd6505132923bf8eba38a.jpg)

​	上面的代码遍历了所有Wrapper类，并使用injectExtension一层层对扩展实现类进行功能增强。

## Dubbo使用JavaAssist减少反射调用开销

​	Dubbo会给每个服务提供者的实现类生产一个Wrapper类，这个Wrapper类里面最终调用服务提供者的接口实现类，Wrapper类的存在是为了减少反射的调用。当服务提供方收到消费方发来的请求后，需要根据消费者传递过来的方法名和参数反射调用服务提供者的实现类，而反射本身是有性能开销的，Dubbo把每个服务提供者的实现类通过JavaAssist包装为一个Wrapper类以减少反射调用开销。那么Wrapper类为何能减少反射调用呢？

​	假设我们的服务提供方实现类为GreetingServiceImpl，其代码如下：

略

那么其对应的Wrapper类的源码如下：

![](https://pic.imgdb.cn/item/610fd7bc5132923bf8ef3f8e.jpg)

![](https://pic.imgdb.cn/item/610fd7ce5132923bf8ef6e05.jpg)

​	下面我们看看Dubbo是在哪里生成Wrapper类的。在Dubbo分层架构概述中，我们讲过Proxy层的SPI扩展接口为ProxyFactory，Dubbo提供的实现主要有JavassistProxyFactory（默认使用）和JdkProxyFactory，其实就是JavassistProxyFactory为每个服务提供者实现类生成了Wrapper类：

![](https://pic.imgdb.cn/item/610fd87b5132923bf8f10877.jpg)

# 远程服务发布与引用流程剖析

## Dubbo服务发布端启动流程剖析

![](https://pic.imgdb.cn/item/610fea985132923bf81e2a28.jpg)

​	在2.2节中我们提到，服务提供方需要使用ServiceConfig API发布服务，具体来说就是执行图3.1中步骤1的export（）方法来激活发布服务。export（）方法的核心代码如下：通过上面的代码可知，Dubbo的延迟发布是通过使用ScheduledExecutorService来实现的，可以通过调用ServiceConfig的setDelay（Integer delay）方法来设置延迟发布时间，其中shouldDelay（）方法的代码如下：

```java
public synchronized void export() {
    if (this.shouldExport() && !this.exported) {
        this.init();

        // check bootstrap state
        if (!bootstrap.isInitialized()) {
            throw new IllegalStateException("DubboBootstrap is not initialized");
        }

        if (!this.isRefreshed()) {
            this.refresh();
        }
		// 是否需要导出服务
        if (!shouldExport()) {
            return;
        }
		// 是否是延迟发布
        if (shouldDelay()) {
            DELAY_EXPORT_EXECUTOR.schedule(this::doExport, getDelay(), TimeUnit.MILLISECONDS);
        } else {
            // 直接发布
            doExport();
        }

        if (this.bootstrap.getTakeoverMode() == BootstrapTakeoverMode.AUTO) {
            this.bootstrap.start();
        }
    }
}
```

```java
public boolean shouldDelay() {
    Integer delay = getDelay();
    return delay != null && delay > 0;
}
```

​	图3.1中的步骤5是在doExportUrlsFor1Protocol（）方法内部首先把参数封装为URL（在Dubbo里会把所有参数封装到一个URL里），然后具体执行服务导出，其核心代码如下：代码1解析MethodConfig对象设置的方法级别的配置并保存到参数map中；代码2用来判断调用类型，如果为泛型调用，则设置泛型类型（true、nativejava或bean方式）；代码5用来导出服务，Dubbo服务导出分本地导出与远程导出，本地导出使用了injvm协议，是一个伪协议，它不开启端口，不发起远程调用，只在JVM内直接关联，但执行Dubbo的Filter链；在默认情况下，Dubbo同时支持本地导出与远程导出协议，可以通过ServiceConfig的setScope（）方法设置，其中配置为none表示不导出服务，为remote表示只导出远程服务，为local表示只导出本地服务。

```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    String name = protocolConfig.getName();
    if (StringUtils.isEmpty(name)) {
        name = DUBBO;
    }
	// 解析MethodConfig设置
    Map<String, String> map = new HashMap<String, String>();
    map.put(SIDE_KEY, PROVIDER_SIDE);

    ServiceConfig.appendRuntimeParameters(map);
    AbstractConfig.appendParameters(map, getMetrics());
    AbstractConfig.appendParameters(map, getApplication());
    AbstractConfig.appendParameters(map, getModule());
    // remove 'default.' prefix for configs from ProviderConfig
    // appendParameters(map, provider, Constants.DEFAULT_KEY);
    AbstractConfig.appendParameters(map, provider);
    AbstractConfig.appendParameters(map, protocolConfig);
    AbstractConfig.appendParameters(map, this);
    MetadataReportConfig metadataReportConfig = getMetadataReportConfig();
    if (metadataReportConfig != null && metadataReportConfig.isValid()) {
        map.putIfAbsent(METADATA_KEY, REMOTE_METADATA_STORAGE_TYPE);
    }
    if (CollectionUtils.isNotEmpty(getMethods())) {
        //TODO Improve method config processing
        for (MethodConfig method : getMethods()) {
            AbstractConfig.appendParameters(map, method, method.getName());
            String retryKey = method.getName() + ".retry";
            if (map.containsKey(retryKey)) {
                String retryValue = map.remove(retryKey);
                if ("false".equals(retryValue)) {
                    map.put(method.getName() + ".retries", "0");
                }
            }
            List<ArgumentConfig> arguments = method.getArguments();
            if (CollectionUtils.isNotEmpty(arguments)) {
                for (ArgumentConfig argument : arguments) {
                    // convert argument type
                    if (argument.getType() != null && argument.getType().length() > 0) {
                        Method[] methods = interfaceClass.getMethods();
                        // visit all methods
                        if (methods.length > 0) {
                            for (int i = 0; i < methods.length; i++) {
                                String methodName = methods[i].getName();
                                // target the method, and get its signature
                                if (methodName.equals(method.getName())) {
                                    Class<?>[] argtypes = methods[i].getParameterTypes();
                                    // one callback in the method
                                    if (argument.getIndex() != -1) {
                                        if (argtypes[argument.getIndex()].getName().equals(argument.getType())) {
                                            AbstractConfig.appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                                        } else {
                                            throw new IllegalArgumentException("Argument config error : the index attribute and type attribute not match :index :" + argument.getIndex() + ", type:" + argument.getType());
                                        }
                                    } else {
                                        // multiple callbacks in the method
                                        for (int j = 0; j < argtypes.length; j++) {
                                            Class<?> argclazz = argtypes[j];
                                            if (argclazz.getName().equals(argument.getType())) {
                                                AbstractConfig.appendParameters(map, argument, method.getName() + "." + j);
                                                if (argument.getIndex() != -1 && argument.getIndex() != j) {
                                                    throw new IllegalArgumentException("Argument config error : the index attribute and type attribute not match :index :" + argument.getIndex() + ", type:" + argument.getType());
                                                }
                                            }
                                        }
                                    }
                                    break;
                                }
                            }
                        }
                    } else if (argument.getIndex() != -1) {
                        AbstractConfig.appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                    } else {
                        throw new IllegalArgumentException("Argument config must set index or type attribute.eg: <dubbo:argument index='0' .../> or <dubbo:argument type=xxx .../>");
                    }

                }
            }
        } // end of methods for
    }
	// 如果是泛型调用，设置泛型类型
    if (ProtocolUtils.isGeneric(generic)) {
        map.put(GENERIC_KEY, generic);
        map.put(METHODS_KEY, ANY_VALUE);
    } else {
        // 正常调用设置拼接URL的参数
        String revision = Version.getVersion(interfaceClass, version);
        if (revision != null && revision.length() > 0) {
            map.put(REVISION_KEY, revision);
        }

        String[] methods = Wrapper.getWrapper(interfaceClass).getMethodNames();
        if (methods.length == 0) {
            logger.warn("No method found in service interface " + interfaceClass.getName());
            map.put(METHODS_KEY, ANY_VALUE);
        } else {
            map.put(METHODS_KEY, StringUtils.join(new HashSet<String>(Arrays.asList(methods)), ","));
        }
    }

    /**
     * Here the token value configured by the provider is used to assign the value to ServiceConfig#token
     */
    if (ConfigUtils.isEmpty(token) && provider != null) {
        token = provider.getToken();
    }

    if (!ConfigUtils.isEmpty(token)) {
        if (ConfigUtils.isDefault(token)) {
            map.put(TOKEN_KEY, UUID.randomUUID().toString());
        } else {
            map.put(TOKEN_KEY, token);
        }
    }
    //init serviceMetadata attachments
    serviceMetadata.getAttachments().putAll(map);

    // export service
    // 拼接URL对象
    String host = findConfigedHosts(protocolConfig, registryURLs, map);
    Integer port = findConfigedPorts(protocolConfig, name, map);
    URL url = new ServiceConfigURL(name, null, null, host, port, getContextPath(protocolConfig).map(p -> p + "/" + path).orElse(path), map);

    // You can customize Configurator to append extra parameters
    if (ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
            .hasExtension(url.getProtocol())) {
        url = ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
                .getExtension(url.getProtocol()).getConfigurator(url).configure(url);
    }
	// 导出服务， 本地服务， 远程服务
    String scope = url.getParameter(SCOPE_KEY);
    // don't export when none is configured
    if (!SCOPE_NONE.equalsIgnoreCase(scope)) {

        // export to local if the config is not remote (export to remote only when config is remote)
        // 如果不是SCOPE_REMOTE，就导出本地服务
        if (!SCOPE_REMOTE.equalsIgnoreCase(scope)) {
            exportLocal(url);
        }
        // export to remote if the config is not local (export to local only when config is local)
        // 如果不是SCOPE_LOCAL，就导出远程服务
        if (!SCOPE_LOCAL.equalsIgnoreCase(scope)) {
            if (CollectionUtils.isNotEmpty(registryURLs)) {
                // 如果有服务中心地址
                for (URL registryURL : registryURLs) {
                    //if protocol is only injvm ,not register
                    if (LOCAL_PROTOCOL.equalsIgnoreCase(url.getProtocol())) {
                        continue;
                    }
                    url = url.addParameterIfAbsent(DYNAMIC_KEY, registryURL.getParameter(DYNAMIC_KEY));
                    URL monitorUrl = ConfigValidationUtils.loadMonitor(this, registryURL);
                    if (monitorUrl != null) {
                        url = url.putAttribute(MONITOR_KEY, monitorUrl);
                    }
                    if (logger.isInfoEnabled()) {
                        if (url.getParameter(REGISTER_KEY, true)) {
                            logger.info("Register dubbo service " + interfaceClass.getName() + " url " + url.getServiceKey() + " to registry " + registryURL.getAddress());
                        } else {
                            logger.info("Export dubbo service " + interfaceClass.getName() + " to url " + url.getServiceKey());
                        }
                    }

                    // For providers, this is used to enable custom proxy to generate invoker
                    String proxy = url.getParameter(PROXY_KEY);
                    if (StringUtils.isNotEmpty(proxy)) {
                        registryURL = registryURL.addParameter(PROXY_KEY, proxy);
                    }
					// 直连方式
                    Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, registryURL.putAttribute(EXPORT_KEY, url));
                    DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                    Exporter<?> exporter = PROTOCOL.export(wrapperInvoker);
                    exporters.add(exporter);
                }
            } else {
                if (logger.isInfoEnabled()) {
                    logger.info("Export dubbo service " + interfaceClass.getName() + " to url " + url);
                }
                if (MetadataService.class.getName().equals(url.getServiceInterface())) {
                    MetadataUtils.saveMetadataURL(url);
                }
                Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, url);
                DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                Exporter<?> exporter = PROTOCOL.export(wrapperInvoker);
                exporters.add(exporter);
            }
			// 元数据存储
            MetadataUtils.publishServiceDefinition(url);
        }
    }
    this.urls.add(url);
}
```

​	如果使用MetadataReportConfig设置了元数据存储信息，代码5.2.3则将元数据保存到指定配置中心。在Dubbo 2.7.0中对服务元数据进行了改造，其把原来都保存到服务注册中心的元数据进行了分类存储，注册中心将只用于存储关键服务信息，比如服务提供者地址列表、完整的接口定义等。Dubbo 2.7.0使用更专业的配置中心，如Nacos、Apollo、Consul和Etcd等，提供更灵活、更丰富的配置规则，包括服务和应用不同粒度的配置、更丰富的路由规则和集中式管理的动态参数规则等。

​	另外，服务提供端导出服务具体使用的是下列代码：

```java
         Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, url);
                DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                Exporter<?> exporter = PROTOCOL.export(wrapperInvoker);
```

​	其中，proxyFactory和protocol的定义为：

```java
private static final Protocol PROTOCOL = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();

/**
 * A {@link ProxyFactory} implementation that will generate a exported service proxy,the JavassistProxyFactory is its
 * default implementation
 */
private static final ProxyFactory PROXY_FACTORY = ExtensionLoader.getExtensionLoader(ProxyFactory.class).getAdaptiveExtension();
```

​	由此可知，proxyFactory和protocol都是扩展接口的适配器类。

​	执行代码proxyFactory.getInvoker（ref，（Class）interfaceClass，url）时，我们发现实际上是首先执行扩展接口ProxyFactory的适配器类ProxyFactory$Adaptive的getInvoker（）方法，其内部根据URL里的proxy的类型选择具体的代理工厂，这里默认proxy类型为javassist，所以又调用了JavassistProxyFactory的getInvoker（）方法获取了代理类。

​	JavassistProxyFactory的getInvoker（）方法的代码如下：

```java
ublic <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
    // TODO Wrapper cannot handle this scenario correctly: the classname contains '$'
    final Wrapper wrapper = Wrapper.getWrapper(proxy.getClass().getName().indexOf('$') < 0 ? proxy.getClass() : type);
    return new AbstractProxyInvoker<T>(proxy, type, url) {
        @Override
        protected Object doInvoke(T proxy, String methodName,
                                  Class<?>[] parameterTypes,
                                  Object[] arguments) throws Throwable {
            return wrapper.invokeMethod(proxy, methodName, parameterTypes, arguments);
        }
    };
}
```

​	根据前面章节介绍的内容可知，这里首先把服务实现类转换为Wrapper类，是为了减少反射的调用，这里返回的是AbstractProxyInvoker对象，其内部重写doInvoke（）方法，并委托给Wrapper实现具体功能。到这里就完成了2.2节中讲解的服务提供方实现类到Invoker的转换。

​	另外，当执行protocol.export（wrapperInvoker）方法的时候，实际调用了Protocol的适配器类Protocol$Adaptive的export（）方法。如果为远程服务暴露，则其内部根据URL中Protocol的类型为registry，会选择Protocol的实现类RegistryProtocol。如果为本地服务暴露，则其内部根据URL中Protocol的类型为injvm，会选择Protocol的实现类InjvmProtocol。但由于Dubbo SPI的扩展点使用了Wrapper自动增强，这里就使用了ProtocolFilterWrapper、ProtocolListenerWrapper、QosProtocolWrapper对其进行了增强，所以需要一层层调用才会调用到RegistryProtocol的export（）方法。本节我们只讲解远程服务暴露流程。

​	图3.1中步骤10的RegistryProtocol中的export（）方法通过步骤11的doLocalExport启动了NettyServer进行监听服务，步骤12、步骤13则将当前服务注册到服务注册中心。到这里就完成了2.2节中谈到的Invoker到Exporter的转换。

​	下面我们首先看看doLocalExport是如何启动NettyServer的。doLocalExport内部主要调用DubboProtocol的export（）方法，下面看看如图3.2所示的时序图。

![](https://pic.imgdb.cn/item/611120825132923bf870a408.jpg)

​	从图3.2可以看到，在RegistryProtocol的doLocalExport（）方法内调用了Protocol的适配器类Protocol$Adaptive，这里URL内的协议类型是dubbo，所以返回的SPI扩展实现类是DubboProtocol，由于DubboProtocol也被Wrapper类增强了，所以也是一层层调用后，才执行到图3.2中的步骤8，即调用DubboProtocol的export（）方法。export（）方法的代码如下：

![](https://pic.imgdb.cn/item/611120ae5132923bf8710af4.jpg)

​	这里将Invoker转换为DubboExporter对象，并且把DubboExporter保存到了缓存exporterMap里（在服务提供方处理请求时会从中获取出来），然后执行图3.2中步骤10的openServer（）方法，其代码如下：

```java
private void openServer(URL url) {
    // find server.
    // 提供者机器的地址ip:host
    String key = url.getAddress();
    //client can export a service which's only for server to invoke
    // 只有服务器提供端才会启动监听
    boolean isServer = url.getParameter(IS_SERVER_KEY, true);
    if (isServer) {
        ProtocolServer server = serverMap.get(key);
        if (server == null) {
            synchronized (this) {
                server = serverMap.get(key);
                if (server == null) {
                    serverMap.put(key, createServer(url));
                }
            }
        } else {
            // server supports reset, use together with override
            server.reset(url);
        }
    }
}
    protected final Map<String, ProtocolServer> serverMap = new ConcurrentHashMap<>();

```

​	通过上面的代码可知，这里首先是获取当前机器地址信息（ip：port）并作为key，然后判断当前是否为服务提供端。如果是，则以此key为key查看缓存serverMap中是否有对应的Server，如果没有则调用createServer（）方法来创建，否则返回缓存中的value。由于每个机器的ip：port是唯一的，所以多个不同服务启动时只有第一个会被创建，后面的服务都是直接从缓存中返回的。

​	通过上面的代码可知，这里首先是获取当前机器地址信息（ip：port）并作为key，然后判断当前是否为服务提供端。如果是，则以此key为key查看缓存serverMap中是否有对应的Server，如果没有则调用createServer（）方法来创建，否则返回缓存中的value。由于每个机器的ip：port是唯一的，所以多个不同服务启动时只有第一个会被创建，后面的服务都是直接从缓存中返回的。

![](https://pic.imgdb.cn/item/611128685132923bf8829ff0.jpg)

​	在默认情况下，传输扩展实现选择的是Netty，而Netty对应的扩展实现为：

![](https://pic.imgdb.cn/item/611129015132923bf8840250.jpg)

​	下面，我们看看NettyTransporter的bind（）方法，其代码如下：

```java
@Override
public RemotingServer bind(URL url, ChannelHandler handler) throws RemotingException {
    return new NettyServer(url, handler);
```

NettyServer的构造函数代码如下：

```java
public NettyServer(URL url, ChannelHandler handler) throws RemotingException {
    super(ExecutorUtil.setThreadName(url, SERVER_THREAD_POOL_NAME), ChannelHandlers.wrap(handler, url));
}
```

​	在NettyServer的构造函数内部又调用了其父类AbstractServer的构造函数，后者构造函数的内容如下：

```java
public AbstractServer(URL url, ChannelHandler handler) throws RemotingException {
    super(url, handler);
    localAddress = getUrl().toInetSocketAddress();

    String bindIp = getUrl().getParameter(Constants.BIND_IP_KEY, getUrl().getHost());
    int bindPort = getUrl().getParameter(Constants.BIND_PORT_KEY, getUrl().getPort());
    if (url.getParameter(ANYHOST_KEY, false) || NetUtils.isInvalidLocalHost(bindIp)) {
        bindIp = ANYHOST_VALUE;
    }
    bindAddress = new InetSocketAddress(bindIp, bindPort);
    this.accepts = url.getParameter(ACCEPTS_KEY, DEFAULT_ACCEPTS);
    try {
        doOpen();
        if (logger.isInfoEnabled()) {
            logger.info("Start " + getClass().getSimpleName() + " bind " + getBindAddress() + ", export " + getLocalAddress());
        }
    } catch (Throwable t) {
        throw new RemotingException(url.toInetSocketAddress(), null, "Failed to bind " + getClass().getSimpleName()
                + " on " + getLocalAddress() + ", cause: " + t.getMessage(), t);
    }
    executor = executorRepository.createExecutorIfAbsent(url);
}
```

​	通过上面的代码可知，NettyServer的doOpen（）方法启动了服务监听，其中doOpen（）方法的代码如下：

```java
protected void doOpen() throws Throwable {
    NettyHelper.setNettyLoggerFactory();
    ExecutorService boss = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerBoss", true));
    ExecutorService worker = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerWorker", true));
    ChannelFactory channelFactory = new NioServerSocketChannelFactory(boss, worker, getUrl().getPositiveParameter(IO_THREADS_KEY, Constants.DEFAULT_IO_THREADS));
    // 创建ServerBootStrap
    bootstrap = new ServerBootstrap(channelFactory);
	// 配置NettyServer，添加handler到管线
    final NettyHandler nettyHandler = new NettyHandler(getUrl(), this);
    channels = nettyHandler.getChannels();
    // https://issues.jboss.org/browse/NETTY-365
    // https://issues.jboss.org/browse/NETTY-379
    // final Timer timer = new HashedWheelTimer(new NamedThreadFactory("NettyIdleTimer", true));
    bootstrap.setOption("child.tcpNoDelay", true);
    bootstrap.setOption("backlog", getUrl().getPositiveParameter(BACKLOG_KEY, Constants.DEFAULT_BACKLOG));
    bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
        @Override
        public ChannelPipeline getPipeline() {
            // 添加handler到接收链接的管线
            NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyServer.this);
            ChannelPipeline pipeline = Channels.pipeline();
            /*int idleTimeout = getIdleTimeout();
            if (idleTimeout > 10000) {
                pipeline.addLast("timer", new IdleStateHandler(timer, idleTimeout / 1000, 0, 0));
            }*/
            // 解码handler
            pipeline.addLast("decoder", adapter.getDecoder());
            // 编码handler
            pipeline.addLast("encoder", adapter.getEncoder());
            // 业务handler
            pipeline.addLast("handler", nettyHandler);
            return pipeline;
        }
    });
    // bind
    // 绑定本地端口，并启动监听服务
    channel = bootstrap.bind(getBindAddress());
}
```

​	至此，服务提供方的NettyServer已经启动了，这里需要注意是，将NettyServerHandler和编解码Handler注册到了每个接收链接的通道（channel）管理的管线中，这些Handler的详细说明在后面的章节会具体讲解。

​	我们先看看RegistryProtocol中export（）方法里的getRegistry（）方法是如何获取服务注册中心和注册服务的，首先看看如图3.4所示的时序图。

​	图3.4中的步骤2用来获取服务注册中心，其中RegistryFactory为扩展接口，所以这里通过适配器类来确定RegistryFactory的扩展实现为ZookeeperRegistryFactory，然后后者内部调用createRegistry（）方法创建了一个ZookeeperRegistry作为ZooKeeper注册中心。其中ZookeeperRegistry的getRegistry（）方法的代码如下：

​	图3.4

![](https://pic.imgdb.cn/item/61112bfb5132923bf88b2113.jpg)

```java
public Registry getRegistry(URL url) {

    Registry defaultNopRegistry = getDefaultNopRegistryIfDestroyed();
    if (null != defaultNopRegistry) {
        return defaultNopRegistry;
    }

    url = URLBuilder.from(url)
            .setPath(RegistryService.class.getName())
            .addParameter(INTERFACE_KEY, RegistryService.class.getName())
            .removeParameters(EXPORT_KEY, REFER_KEY, TIMESTAMP_KEY)
            .build();
    String key = createRegistryCacheKey(url);
    // Lock the registry access process to ensure a single instance of the registry
    // 独占锁保证只有一个线程实例创建服务注册实例
    LOCK.lock();
    try {
        // double check
        // fix https://github.com/apache/dubbo/issues/7265.
        defaultNopRegistry = getDefaultNopRegistryIfDestroyed();
        if (null != defaultNopRegistry) {
            return defaultNopRegistry;
        }
		
        Registry registry = REGISTRIES.get(key);
        if (registry != null) {
            return registry;
        }
        //create registry by spi/ioc
        // 创建服务器注册中心实例
        registry = createRegistry(url);
        if (registry == null) {
            throw new IllegalStateException("Can not create registry " + url);
        }
        REGISTRIES.put(key, registry);
        return registry;
    } finally {
        // Release the lock
        LOCK.unlock();
    }
}
	protected static final ReentrantLock LOCK = new ReentrantLock();

    // Registry Collection Map<RegistryAddress, Registry>
    protected static final Map<String, Registry> REGISTRIES = new HashMap<>();
public Registry createRegistry(URL url) {
        return new ZookeeperRegistry(url, zookeeperTransporter);
    }
```

​	在图3.4的步骤6中，register（）方法最终调用了zkClient的create（）方法将服务注册到ZooKeeper：

```java
// ZookeeperRegistry
public void doRegister(URL url) {
    try {
        zkClient.create(toUrlPath(url), url.getParameter(DYNAMIC_KEY, true));
    } catch (Throwable e) {
        throw new RpcException("Failed to register " + url + " to zookeeper " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```

​	其中，create（）方法的代码如下：

```java
public void create(String path, boolean ephemeral) {
    if (!ephemeral) {
        if (persistentExistNodePath.contains(path)) {
            return;
        }
        if (checkExists(path)) {
            persistentExistNodePath.add(path);
            return;
        }
    }
    int i = path.lastIndexOf('/');
    if (i > 0) {
        create(path.substring(0, i), false);
    }
    if (ephemeral) {
        createEphemeral(path);
    } else {
        createPersistent(path);
        persistentExistNodePath.add(path);
    }
}
```

​	比如，以com.books.dubbo.demo.api.GreetingService服务接口为例，则其URL为：

![](https://pic.imgdb.cn/item/61112ef15132923bf89243aa.jpg)

![](https://pic.imgdb.cn/item/61112f685132923bf8935b2c.jpg)

​	经过toUrlPath转换后为：

![](https://pic.imgdb.cn/item/61112fb05132923bf894055e.jpg)

​	create（）方法是递归函数，首先其调用了方法createPersistent（）分别创建了节点/dubbo、/dubbo/com.books.dubbo.demo.api.GreetingService和/dubbo/com.books.dubbo.demo.api.GreetingService/providers，然后调用createEphemeral（）方法创建了下列内容：

![](https://pic.imgdb.cn/item/61112fef5132923bf8947dd9.jpg)

​	服务注册到ZooKeeper后，ZooKeeper服务端的最终树图结构如图3.5所示。

![](https://pic.imgdb.cn/item/61112ffe5132923bf894b533.jpg)

​	通过图3.5可知，机器192.168.0.1作为com.books.dubbo.demo.api.GreetingService服务的提供者，第一层Root节点说明ZooKeeper的服务分组为Dubbo，第二层Service节点说明注册的服务为com.books.dubbo.demo.api.GreetingService接口，第三层Type节点说明是为服务提供者注册的服务，第四层URL记录服务提供者的地址信息（这里只是简单地显示了服务提供者的IP信息，对于真实情况则不只是IP信息）。

​	第一个服务提供者注册时需要ZooKeeper服务端创建第一层的Dubbo节点、第二层的Service节点、第三层的Type节点，但是同一个Service的其他机器在注册服务时因为上面三层节点已经存在了，所以只需在Providers下也就是第四层插入服务提供者信息节点就可以了。

​	服务注册到ZooKeeper后，消费端就可以在Providers节点下找到com.books.dubbo.demo.api.GreetingService服务的所有服务提供者，然后根据设置的负载均衡策略选择机器进行远程调用了。

## Dubbo服务提供方如何处理请求

​	我们先看看当消费端发起TCP链接并完成后，在接收消费端发来的请求时，服务提供方是如何处理的，其处理时序图如图3.6所示。

![](https://pic.imgdb.cn/item/611130635132923bf895b7aa.jpg)

​	让我们看看图3.6中的connected部分（步骤1～步骤11），消费端发起TCP链接并完成后，服务提供方的NettyServer的connected方法会被激活，该方法的执行是在Netty的I/O线程上执行的，为了可以及时释放I/O线程，Netty默认的线程模型为All，正如在第7章所介绍的，所有消息都派发到Dubbo内部的业务线程池，这些消息包括请求事件、响应事件、连接事件、断开事件、心跳事件等，这里对应的是AllChannelHandler类把I/O线程接收到的所有消息包装为ChannelEventRunnable任务并都投递到了线程池里。

​	线程池里的任务被执行后，最终会调用DubboProtocol的connected（）方法，其代码如下：

```java
public void connected(Channel channel) throws RemotingException {
    invoke(channel, ON_CONNECT_KEY);
}
```

​	其中，invoke（）是一个通用方法，其代码如下：

```java
private void invoke(Channel channel, String methodKey) {
    Invocation invocation = createInvocation(channel, channel.getUrl(), methodKey);
    if (invocation != null) {
        try {
            received(channel, invocation);
        } catch (Throwable t) {
            logger.warn("Failed to invoke event method " + invocation.getMethodName() + "(), cause: " + t.getMessage(), t);
        }
    }
}
public void received(Channel channel, Object message) throws RemotingException {
            if (message instanceof Invocation) {
                reply((ExchangeChannel) channel, message);

            } else {
                super.received(channel, message);
            }
        }
```

​	其中，createInvocation（）方法的代码如下：

```java
private Invocation createInvocation(Channel channel, URL url, String methodKey) {
    String method = url.getParameter(methodKey);
    if (method == null || method.length() == 0) {
        return null;
    }

    RpcInvocation invocation = new RpcInvocation(method, url.getParameter(INTERFACE_KEY), "", new Class<?>[0], new Object[0]);
    invocation.setAttachment(PATH_KEY, url.getPath());
    invocation.setAttachment(GROUP_KEY, url.getGroup());
    invocation.setAttachment(INTERFACE_KEY, url.getParameter(INTERFACE_KEY));
    invocation.setAttachment(VERSION_KEY, url.getVersion());
    if (url.getParameter(STUB_EVENT_KEY, false)) {
        invocation.setAttachment(STUB_EVENT_KEY, Boolean.TRUE.toString());
    }

    return invocation;
}
```

​	在这里的URL内不包含Constants.ON_CONNECT_KEY，所以直接返回了null。

​	图3.6中的received部分（步骤12～步骤22）类似于connected，received事件被投递到线程池后进行异步处理。线程池任务被激活后调用了HeaderExchangeHandler的received（）方法，其代码如下：

```java
public void received(Channel channel, Object message) throws RemotingException {
    final ExchangeChannel exchangeChannel = HeaderExchangeChannel.getOrAddChannel(channel);
    if (message instanceof Request) {
        // 处理请求
        // handle request.
        Request request = (Request) message;
        if (request.isEvent()) {
            handlerEvent(channel, request);
        } else {
            // 需要有返回值的请求req-res, twoway
            if (request.isTwoWay()) {
                handleRequest(exchangeChannel, request);
            } else {
                // 不需要有返回值的请求req,oneway
                handler.received(exchangeChannel, request.getData());
            }
        }
    } else if (message instanceof Response) {
        handleResponse(channel, (Response) message);
    } else if (message instanceof String) {
        if (isClientSide(channel)) {
            Exception e = new Exception("Dubbo client can not supported string message: " + message + " in channel: " + channel + ", url: " + channel.getUrl());
            logger.error(e.getMessage(), e);
        } else {
            String echo = handler.telnet(channel, (String) message);
            if (echo != null && echo.length() > 0) {
                channel.send(echo);
            }
        }
    } else {
        handler.received(exchangeChannel, message);
    }
}
```

​		如果请求不需要响应结果则直接调用DubboProtocol的received（）方法，否则执行handleRequest（）方法，后者代码如下：

```java
void handleRequest(final ExchangeChannel channel, Request req) throws RemotingException {
    Response res = new Response(req.getId(), req.getVersion());
    if (req.isBroken()) {
        Object data = req.getData();

        String msg;
        if (data == null) {
            msg = null;
        } else if (data instanceof Throwable) {
            msg = StringUtils.toString((Throwable) data);
        } else {
            msg = data.toString();
        }
        res.setErrorMessage("Fail to decode request due to: " + msg);
        res.setStatus(Response.BAD_REQUEST);

        channel.send(res);
        return;
    }
    // find handler by message class.
    Object msg = req.getData();
    try {
        // 调用reply方法
        CompletionStage<Object> future = handler.reply(channel, msg);
        // 等待结果回调
        future.whenComplete((appResult, t) -> {
            try {
                if (t == null) {
                    res.setStatus(Response.OK);
                    res.setResult(appResult);
                } else {
                    res.setStatus(Response.SERVICE_ERROR);
                    res.setErrorMessage(StringUtils.toString(t));
                }
                channel.send(res);
            } catch (RemotingException e) {
                logger.warn("Send result to consumer failed, channel is " + channel + ", msg is " + e);
            }
        });
    } catch (Throwable e) {
        res.setStatus(Response.SERVICE_ERROR);
        res.setErrorMessage(StringUtils.toString(e));
        channel.send(res);
    }
}
```

​	如果请求需要返回值则执行handleRequest（）方法，其也是委托给DubboProtocol的reply（）方法来执行的。如果执行结果已经完成，则直接将结果写回消费端，否则使用异步回调方式（避免当前线程被阻塞），等执行完毕并拿到结果后再把结果写回消费端。

​	通过图3.6可知，无论消费端是否需要执行结果，最终都是DubboProtocol类的reply（）方法来执行具体的服务，下面我们看看reply（）方法的代码：

```java
public CompletableFuture<Object> reply(ExchangeChannel channel, Object message) throws RemotingException {

    if (!(message instanceof Invocation)) {
        throw new RemotingException(channel, "Unsupported request: "
                + (message == null ? null : (message.getClass().getName() + ": " + message))
                + ", channel: consumer: " + channel.getRemoteAddress() + " --> provider: " + channel.getLocalAddress());
    }
	// 获取调用放到对应的Invoker
    Invocation inv = (Invocation) message;
    Invoker<?> invoker = getInvoker(channel, inv);
    // need to consider backward-compatibility if it's a callback
    if (Boolean.TRUE.toString().equals(inv.getObjectAttachments().get(IS_CALLBACK_SERVICE_INVOKE))) {
        String methodsStr = invoker.getUrl().getParameters().get("methods");
        boolean hasMethod = false;
        if (methodsStr == null || !methodsStr.contains(",")) {
            hasMethod = inv.getMethodName().equals(methodsStr);
        } else {
            String[] methods = methodsStr.split(",");
            for (String method : methods) {
                if (inv.getMethodName().equals(method)) {
                    hasMethod = true;
                    break;
                }
            }
        }
        if (!hasMethod) {
            logger.warn(new IllegalStateException("The methodName " + inv.getMethodName()
                    + " not found in callback service interface ,invoke will be ignored."
                    + " please update the api interface. url is:"
                    + invoker.getUrl()) + " ,invocation is :" + inv);
            return null;
        }
    }
	// 获取上下文对象，并设置对端地址
  RpcContext.getServiceContext().setRemoteAddress(channel.getRemoteAddress());
    // 执行invoker调用链
    Result result = invoker.invoke(inv);
    // 回写结果
    return result.thenApply(Function.identity());
}
```

​	代码7首先使用getInvoker（）方法获取调用方法对应的DubboExporter对象导出的Invoker对象，具体代码如下。在3.1节中我们介绍了导出的DubboExporter对象会保存到exporterMap中，这里的getInvoker获取的是RegistryProtocol$InvokerDelegate：

```java
Invoker<?> getInvoker(Channel channel, Invocation inv) throws RemotingException {
    boolean isCallBackServiceInvoke = false;
    boolean isStubServiceInvoke = false;
    int port = channel.getLocalAddress().getPort();
    String path = (String) inv.getObjectAttachments().get(PATH_KEY);

    // if it's callback service on client side
    isStubServiceInvoke = Boolean.TRUE.toString().equals(inv.getObjectAttachments().get(STUB_EVENT_KEY));
    if (isStubServiceInvoke) {
        port = channel.getRemoteAddress().getPort();
    }

    //callback
    isCallBackServiceInvoke = isClientSide(channel) && !isStubServiceInvoke;
    if (isCallBackServiceInvoke) {
        path += "." + inv.getObjectAttachments().get(CALLBACK_SERVICE_KEY);
        inv.getObjectAttachments().put(IS_CALLBACK_SERVICE_INVOKE, Boolean.TRUE.toString());
    }

    String serviceKey = serviceKey(
            port,
            path,
            (String) inv.getObjectAttachments().get(VERSION_KEY),
            (String) inv.getObjectAttachments().get(GROUP_KEY)
    );
    DubboExporter<?> exporter = (DubboExporter<?>) exporterMap.get(serviceKey);

    if (exporter == null) {
        throw new RemotingException(channel, "Not found exported service: " + serviceKey + " in " + exporterMap.keySet() + ", may be version or group mismatch " +
                ", channel: consumer: " + channel.getRemoteAddress() + " --> provider: " + channel.getLocalAddress() + ", message:" + getInvocationWithoutData(inv));
    }

    return exporter.getInvoker();
}
```

​	代码8把对端的地址设置到上下文对象中。

​	代码9执行导出的Invoker的invoke（）方法，这里有个调用链，经过调用链后最终调用了服务提供方启动时AbstractProxyInvoker代理类创建的invoke（）方法，其调用时序如图3.7所示。

![](https://pic.imgdb.cn/item/611137445132923bf8a77590.jpg)

​	图3.7显示的是调用DubboProtocol的reply（）方法的情况，在调用InvokerDelegate的invoke（）方法前会先经过Filter链（这里只列出来了Filter链中的一部分Filter），然后InvokerDelegate会调用服务提供方启动时AbstractProxyInvoker代理类的invoke（）方法，其代码如下：

```java
public Result invoke(Invocation invocation) throws RpcException {
    try {
        // 具体执行本地服务调用
        Object value = doInvoke(proxy, invocation.getMethodName(), invocation.getParameterTypes(), invocation.getArguments());
        CompletableFuture<Object> future = wrapWithFuture(value);
        CompletableFuture<AppResponse> appResponseFuture = future.handle((obj, t) -> {
            AppResponse result = new AppResponse(invocation);
            if (t != null) {
                if (t instanceof CompletionException) {
                    result.setException(t.getCause());
                } else {
                    result.setException(t);
                }
            } else {
                result.setValue(obj);
            }
            return result;
        });
        return new AsyncRpcResult(appResponseFuture, invocation);
    } catch (InvocationTargetException e) {
        if (RpcContext.getServiceContext().isAsyncStarted() && !RpcContext.getServiceContext().stopAsync()) {
            logger.error("Provider async started, but got an exception from the original method, cannot write the exception back to consumer because an async result may have returned the new thread.", e);
        }
        return AsyncRpcResult.newDefaultAsyncResult(null, e.getTargetException(), invocation);
    } catch (Throwable e) {
        throw new RpcException("Failed to invoke remote proxy method " + invocation.getMethodName() + " to " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
private CompletableFuture<Object> wrapWithFuture(Object value) {
        if (RpcContext.getServiceContext().isAsyncStarted()) {
            return ((AsyncContextImpl)(RpcContext.getServiceContext().getAsyncContext())).getInternalFuture();
        } else if (value instanceof CompletableFuture) {
            return (CompletableFuture<Object>) value;
        }
        return CompletableFuture.completedFuture(value);
    }
```

​	代码11获取上下文对象，代码11.1调用doInvoke（）方法，并使用JavaAssist来执行本地服务，以便减少反射调用，这里我们再回顾一下doInvoke（）方法的代码：

![](https://pic.imgdb.cn/item/61113a915132923bf8b1473e.jpg)

​	通过上面的代码可知，AbstractProxyInvoker的doInvoke（）方法委托Wrapper类的invokeMethod执行具体逻辑，后者则通过调用服务提供方接口的实现类来执行本地服务，细节可参考2.5.4小节。

​	需要注意的是，步骤11.2返回的结果是区分情况的，如果返回值类型为CompletableFuture，或者如果是使用RpcContext.startAsync（）开启异步执行的，则返回AsyncRpcResult对象，否则返回正常的RpcResult对象，我们在随后的11.2.1小节中会用到这些内容。

​	另外，如果代码10发现结果为AsyncRpcResult，则说明是服务提供方的异步执行，此时返回对应的CompletableFuture对象，如果返回的是正常的RpcResult结果，则使用CompletableFuture.completedFuture（result）；把结果转换为CompletableFuture对象（即同步转异步），以便统一处理，在随后的11.2.1小节中我们也会讲到。

## Dubbo服务消费方启动流程剖析

![](https://pic.imgdb.cn/item/611146f55132923bf8d5c1ac.jpg)

​	在2.2节中，我们提到服务消费方需要使用ReferenceConfigAPI来消费服务，这里就是执行图3.8中步骤1的get（）方法来生成远程调用代理类。get（）方法首先会执行init（）方法：其中，checkMock（）方法用来检查设置的mock是否正确（有关细节我们在第5章会展开讲解），然后通过调用createProxy（）方法来创建代理类，createProxy（）方法的核心代码如下：

```java
 // ReferenceConfig
 protected synchronized void init() {
        if (initialized) {
            return;
        }

        // Using DubboBootstrap API will associate bootstrap when registering reference.
        // Loading by Spring context will associate bootstrap in afterPropertiesSet() method.
        // Initializing bootstrap here only for compatible with old API usages.
        if (bootstrap == null) {
            bootstrap = DubboBootstrap.getInstance();
            bootstrap.initialize();
            bootstrap.reference(this);
        }

        // check bootstrap state
        if (!bootstrap.isInitialized()) {
            throw new IllegalStateException("DubboBootstrap is not initialized");
        }

        if (!this.isRefreshed()) {
            this.refresh();
        }

        //init serivceMetadata
        initServiceMetadata(consumer);
        serviceMetadata.setServiceType(getServiceInterfaceClass());
        // TODO, uncomment this line once service key is unified
        serviceMetadata.setServiceKey(URL.buildKey(interfaceName, group, version));

        ServiceRepository repository = ApplicationModel.getServiceRepository();
        ServiceDescriptor serviceDescriptor = repository.registerService(interfaceClass);
        repository.registerConsumer(
                serviceMetadata.getServiceKey(),
                serviceDescriptor,
                this,
                null,
                serviceMetadata);


        Map<String, String> map = new HashMap<String, String>();
        map.put(SIDE_KEY, CONSUMER_SIDE);

        ReferenceConfigBase.appendRuntimeParameters(map);
        if (!ProtocolUtils.isGeneric(generic)) {
            String revision = Version.getVersion(interfaceClass, version);
            if (revision != null && revision.length() > 0) {
                map.put(REVISION_KEY, revision);
            }

            String[] methods = Wrapper.getWrapper(interfaceClass).getMethodNames();
            if (methods.length == 0) {
                logger.warn("No method found in service interface " + interfaceClass.getName());
                map.put(METHODS_KEY, ANY_VALUE);
            } else {
                map.put(METHODS_KEY, StringUtils.join(new HashSet<String>(Arrays.asList(methods)), COMMA_SEPARATOR));
            }
        }
        map.put(INTERFACE_KEY, interfaceName);
        AbstractConfig.appendParameters(map, getMetrics());
        AbstractConfig.appendParameters(map, getApplication());
        AbstractConfig.appendParameters(map, getModule());
        // remove 'default.' prefix for configs from ConsumerConfig
        // appendParameters(map, consumer, Constants.DEFAULT_KEY);
        AbstractConfig.appendParameters(map, consumer);
        AbstractConfig.appendParameters(map, this);
        MetadataReportConfig metadataReportConfig = getMetadataReportConfig();
        if (metadataReportConfig != null && metadataReportConfig.isValid()) {
            map.putIfAbsent(METADATA_KEY, REMOTE_METADATA_STORAGE_TYPE);
        }
        Map<String, AsyncMethodInfo> attributes = null;
        if (CollectionUtils.isNotEmpty(getMethods())) {
            attributes = new HashMap<>();
            for (MethodConfig methodConfig : getMethods()) {
                AbstractConfig.appendParameters(map, methodConfig, methodConfig.getName());
                String retryKey = methodConfig.getName() + ".retry";
                if (map.containsKey(retryKey)) {
                    String retryValue = map.remove(retryKey);
                    if ("false".equals(retryValue)) {
                        map.put(methodConfig.getName() + ".retries", "0");
                    }
                }
                AsyncMethodInfo asyncMethodInfo = AbstractConfig.convertMethodConfig2AsyncInfo(methodConfig);
                if (asyncMethodInfo != null) {
//                    consumerModel.getMethodModel(methodConfig.getName()).addAttribute(ASYNC_KEY, asyncMethodInfo);
                    attributes.put(methodConfig.getName(), asyncMethodInfo);
                }
            }
        }

        String hostToRegistry = ConfigUtils.getSystemProperty(DUBBO_IP_TO_REGISTRY);
        if (StringUtils.isEmpty(hostToRegistry)) {
            hostToRegistry = NetUtils.getLocalHost();
        } else if (isInvalidLocalHost(hostToRegistry)) {
            throw new IllegalArgumentException(
                    "Specified invalid registry ip from property:" + DUBBO_IP_TO_REGISTRY + ", value:" + hostToRegistry);
        }
        map.put(REGISTER_IP_KEY, hostToRegistry);

        serviceMetadata.getAttachments().putAll(map);
		// 创建代理
        ref = createProxy(map);

        serviceMetadata.setTarget(ref);
        serviceMetadata.addAttribute(PROXY_CLASS_REF, ref);
        ConsumerModel consumerModel = repository.lookupReferredService(serviceMetadata.getServiceKey());
        consumerModel.setProxyObject(ref);
        consumerModel.init(attributes);

        initialized = true;

        checkInvokerAvailable();
    }
```

```java
private T createProxy(Map<String, String> map) {
    // 是否需要打开本地引用
    if (shouldJvmRefer(map)) {
        URL url = new ServiceConfigURL(LOCAL_PROTOCOL, LOCALHOST_VALUE, 0, interfaceClass.getName()).addParameters(map);
        invoker = REF_PROTOCOL.refer(interfaceClass, url);
        if (logger.isInfoEnabled()) {
            logger.info("Using injvm service " + interfaceClass.getName());
        }
    } else {
        urls.clear();
        // 用户是否指定服务提供方地址: 可以使服务提供方IP地址(直连方式)
        if (url != null && url.length() > 0) { // user specified URL, could be peer-to-peer address, or register center's address.
            String[] us = SEMICOLON_SPLIT_PATTERN.split(url);
            if (us != null && us.length > 0) {
                for (String u : us) {
                    URL url = URL.valueOf(u);
                    if (StringUtils.isEmpty(url.getPath())) {
                        url = url.setPath(interfaceName);
                    }
                    if (UrlUtils.isRegistry(url)) {
                        urls.add(url.putAttribute(REFER_KEY, map));
                    } else {
                        URL peerURL = ClusterUtils.mergeUrl(url, map);
                        peerURL = peerURL.putAttribute(PEER_KEY, true);
                        urls.add(peerURL);
                    }
                }
            }
        } else { // assemble URL from register center's configuration
            // if protocols not injvm checkRegistry
            // 根据服务注册中心信息装配URL对象
            if (!LOCAL_PROTOCOL.equalsIgnoreCase(getProtocol())) {
                checkRegistry();
                List<URL> us = ConfigValidationUtils.loadRegistries(this, false);
                if (CollectionUtils.isNotEmpty(us)) {
                    for (URL u : us) {
                        URL monitorUrl = ConfigValidationUtils.loadMonitor(this, u);
                        if (monitorUrl != null) {
                            u = u.putAttribute(MONITOR_KEY, monitorUrl);
                        }
                        urls.add(u.putAttribute(REFER_KEY, map));
                    }
                }
                if (urls.isEmpty()) {
                    throw new IllegalStateException(
                            "No such any registry to reference " + interfaceName + " on the consumer " + NetUtils.getLocalHost() +
                                    " use dubbo version " + Version.getVersion() +
                                    ", please config <dubbo:registry address=\"...\" /> to your spring config.");
                }
            }
        }
		// 只有一个服务中心的时候
        if (urls.size() == 1) {
            invoker = REF_PROTOCOL.refer(interfaceClass, urls.get(0));
        } else {
            // 多个服务中心的时候
            List<Invoker<?>> invokers = new ArrayList<Invoker<?>>();
            URL registryURL = null;
            for (URL url : urls) {
                // For multi-registry scenarios, it is not checked whether each referInvoker is available.
                // Because this invoker may become available later.
                invokers.add(REF_PROTOCOL.refer(interfaceClass, url));

                if (UrlUtils.isRegistry(url)) {
                    registryURL = url; // use last registry url
                }
            }

            if (registryURL != null) { // registry url is available
                // for multi-subscription scenario, use 'zone-aware' policy by default
                String cluster = registryURL.getParameter(CLUSTER_KEY, ZoneAwareCluster.NAME);
                // The invoker wrap sequence would be: ZoneAwareClusterInvoker(StaticDirectory) -> FailoverClusterInvoker(RegistryDirectory, routing happens here) -> Invoker
                invoker = Cluster.getCluster(cluster, false).join(new StaticDirectory(registryURL, invokers));
            } else { // not a registry url, must be direct invoke.
                String cluster = CollectionUtils.isNotEmpty(invokers)
                        ?
                        (invokers.get(0).getUrl() != null ? invokers.get(0).getUrl().getParameter(CLUSTER_KEY, ZoneAwareCluster.NAME) :
                                Cluster.DEFAULT)
                        : Cluster.DEFAULT;
                invoker = Cluster.getCluster(cluster).join(new StaticDirectory(invokers));
            }
        }
    }

    if (logger.isInfoEnabled()) {
        logger.info("Referred dubbo service " + interfaceClass.getName());
    }

    URL consumerURL = new ServiceConfigURL(CONSUMER_PROTOCOL, map.get(REGISTER_IP_KEY), 0, map.get(INTERFACE_KEY), map);
    MetadataUtils.publishServiceDefinition(consumerURL);
	// 创建服务代理
    // create service proxy
    return (T) PROXY_FACTORY.getProxy(invoker, ProtocolUtils.isGeneric(generic));
}
```

​	上面的代码比较简单，首先判断是否需要开启本地引用，如果需要则创建JVM协议的本地引用，然后加载服务注册中心信息，服务注册中心可以有多个。

​	在2.2节我们讲过，服务暴露第一步是调用Protocol扩展接口实现类的refer（）方法以生成Invoker实例，这正是代码4做所的事情，其中refprotocol的定义如下：

​	![](https://pic.imgdb.cn/item/61114b2e5132923bf8e1250f.jpg)

​	从上面的代码可知，refprotocol是Protocol扩展接口的适配器类，这里调用的refprotocol.refer（interfaceClass，urls.get（0））；实际上是Protocol$Adaptive的refer（）方法。

​	在Protocol$Adaptive的refer（）方法内部，当我们设置了服务注册中心后，可以发现当前协议类型为registry，也就是说这里需要调用RegistryProtocol的refer（）方法。但RegistryProtocol被QosProtocolWrapper、ProtocolFilterWrapper、ProtocolListenerWrapper三个Wrapper类增强了，所以这里经过一层层调用后，最后才调用到RegistryProtocol的refer（）方法，其内部主要是调用了doRefer（）方法，doRefer（）方法的代码如下：

```java
protected <T> Invoker<T> doRefer(Cluster cluster, Registry registry, Class<T> type, URL url, Map<String, String> parameters) {
    Map<String, Object> consumerAttribute = new HashMap<>(url.getAttributes());
    consumerAttribute.remove(REFER_KEY);
    URL consumerUrl = new ServiceConfigURL(parameters.get(PROTOCOL_KEY) == null ? DUBBO : parameters.get(PROTOCOL_KEY),
            null,
            null,
            parameters.get(REGISTER_IP_KEY),
            0, getPath(parameters, type),
            parameters,
            consumerAttribute);
    url = url.putAttribute(CONSUMER_URL_KEY, consumerUrl);
    ClusterInvoker<T> migrationInvoker = getMigrationInvoker(this, cluster, registry, type, url, consumerUrl);
    return interceptInvoker(migrationInvoker, url, consumerUrl, url);
}

protected <T> Invoker<T> interceptInvoker(ClusterInvoker<T> invoker, URL url, URL consumerUrl, URL registryURL) {
        List<RegistryProtocolListener> listeners = findRegistryProtocolListeners(url);
        if (CollectionUtils.isEmpty(listeners)) {
            return invoker;
        }

        for (RegistryProtocolListener listener : listeners) {
            listener.onRefer(this, invoker, consumerUrl, registryURL);
        }
        return invoker;
    }

```

![](https://pic.imgdb.cn/item/61114ba75132923bf8e258b6.jpg)

​	图3.9中的步骤2、步骤3和步骤4用来从ZooKeeper获取服务提供者的地址列表，等ZooKeeper返回地址列表后会调用RegistryDirectory的notify（）方法：

