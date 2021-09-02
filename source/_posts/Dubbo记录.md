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

```java
@Override
public synchronized void notify(List<URL> urls) {
    if (isDestroyed()) {
        return;
    }
	// 对元数据分类
    Map<String, List<URL>> categoryUrls = urls.stream()
            .filter(Objects::nonNull)
            .filter(this::isValidCategory)
            .filter(this::isNotCompatibleFor26x)
            .collect(Collectors.groupingBy(this::judgeCategory));
	// 获取配置信息
    List<URL> configuratorURLs = categoryUrls.getOrDefault(CONFIGURATORS_CATEGORY, Collections.emptyList());
    this.configurators = Configurator.toConfigurators(configuratorURLs).orElse(this.configurators);
	// 路由信息收集并保存
    List<URL> routerURLs = categoryUrls.getOrDefault(ROUTERS_CATEGORY, Collections.emptyList());
    toRouters(routerURLs).ifPresent(this::addRouters);

    // providers
   	// 服务提供者信息
    List<URL> providerURLs = categoryUrls.getOrDefault(PROVIDERS_CATEGORY, Collections.emptyList());
    /**
     * 3.x added for extend URL address
     */
    ExtensionLoader<AddressListener> addressListenerExtensionLoader = ExtensionLoader.getExtensionLoader(AddressListener.class);
    List<AddressListener> supportedListeners = addressListenerExtensionLoader.getActivateExtension(getUrl(), (String[]) null);
    if (supportedListeners != null && !supportedListeners.isEmpty()) {
        for (AddressListener addressListener : supportedListeners) {
            providerURLs = addressListener.notify(providerURLs, getConsumerUrl(),this);
        }
    }
    refreshOverrideAndInvoker(providerURLs);
}
```

​	图3.9中的步骤6、步骤7和步骤8根据获取的最新的服务提供者URL地址，将其转换为具体的invoker列表，也就是说每个提供者的URL会被转换为一个Invoker对象，具体转换在toInvokers（）方法中进行：

```java
private Map<URL, Invoker<T>> toInvokers(List<URL> urls) {
    Map<URL, Invoker<T>> newUrlInvokerMap = new ConcurrentHashMap<>();
    if (urls == null || urls.isEmpty()) {
        return newUrlInvokerMap;
    }
    String queryProtocols = this.queryMap.get(PROTOCOL_KEY);
    for (URL providerUrl : urls) {
        // If protocol is configured at the reference side, only the matching protocol is selected
        if (queryProtocols != null && queryProtocols.length() > 0) {
            boolean accept = false;
            String[] acceptProtocols = queryProtocols.split(",");
            for (String acceptProtocol : acceptProtocols) {
                if (providerUrl.getProtocol().equals(acceptProtocol)) {
                    accept = true;
                    break;
                }
            }
            if (!accept) {
                continue;
            }
        }
        if (EMPTY_PROTOCOL.equals(providerUrl.getProtocol())) {
            continue;
        }
        if (!ExtensionLoader.getExtensionLoader(Protocol.class).hasExtension(providerUrl.getProtocol())) {
            logger.error(new IllegalStateException("Unsupported protocol " + providerUrl.getProtocol() +
                    " in notified url: " + providerUrl + " from registry " + getUrl().getAddress() +
                    " to consumer " + NetUtils.getLocalHost() + ", supported protocol: " +
                    ExtensionLoader.getExtensionLoader(Protocol.class).getSupportedExtensions()));
            continue;
        }
        URL url = mergeUrl(providerUrl);

        // Cache key is url that does not merge with consumer side parameters, regardless of how the consumer combines parameters, if the server url changes, then refer again
        Map<URL, Invoker<T>> localUrlInvokerMap = this.urlInvokerMap; // local reference
        Invoker<T> invoker = localUrlInvokerMap == null ? null : localUrlInvokerMap.remove(url);
        if (invoker == null) { // Not in the cache, refer again
            try {
                boolean enabled = true;
                if (url.hasParameter(DISABLED_KEY)) {
                    enabled = !url.getParameter(DISABLED_KEY, false);
                } else {
                    enabled = url.getParameter(ENABLED_KEY, true);
                }
                if (enabled) {
                    // 调用Dubbo协议转换服务到invoker
                    invoker = protocol.refer(serviceType, url);
                }
            } catch (Throwable t) {
                logger.error("Failed to refer invoker for interface:" + serviceType + ",url:(" + url + ")" + t.getMessage(), t);
            }
            if (invoker != null) { // Put new invoker in cache
                newUrlInvokerMap.put(url, invoker);
            }
        } else {
            newUrlInvokerMap.put(url, invoker);
        }
    }
    return newUrlInvokerMap;
}
```

​	从上面的代码可知，将服务接口转换到invoker对象是通过调用protocol.refer（serviceType，url）来完成的，这里的protocol对象也是Protocol扩展接口的适配器对象，所以调用protocol.refer实际上是调用适配器Protocol$Adaptive的refer（）方法。在URL中，协议默认为是Dubbo，所以适配器里调用的应该是DubboProtocol的refer（）方法。

​	前面的章节已讲过，Dubbo默认提供了一系列Wrapper类对扩展实现类进行功能增强，当然这里也不例外，Dubbo使用了ProtocolListenerWrapper、ProtocolFilterWrapper等类对DubboProtocol进行了功能增强。所以，在这里经过一次次调用后才调用到DubboProtocol的refer（）方法，DubboProtocol的refer（）方法的代码如下：

```java
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
    return protocolBindingRefer(type, url);
}
public <T> Invoker<T> protocolBindingRefer(Class<T> serviceType, URL url) throws RpcException {
        optimizeSerialization(url);

        // create rpc invoker.
        DubboInvoker<T> invoker = new DubboInvoker<T>(serviceType, url, getClients(url), invokers);
        invokers.add(invoker);

        return invoker;
    }
```

​	从上面的代码可知，getClients（）方法创建服务消费端的NettyClient对象，其调用链的时序图如图3.10所示，其中NettyClient构造函数如下：

```java
public NettyClient(final URL url, final ChannelHandler handler) throws RemotingException {
        super(url, wrapChannelHandler(url, handler));
    }
protected static ChannelHandler wrapChannelHandler(URL url, ChannelHandler handler) {
        return ChannelHandlers.wrap(handler, url);
    }
public static ChannelHandler wrap(ChannelHandler handler, URL url) {
        return ChannelHandlers.getInstance().wrapInternal(handler, url);
    }

protected ChannelHandler wrapInternal(ChannelHandler handler, URL url) {
        return new MultiMessageHandler(new HeartbeatHandler(ExtensionLoader.getExtensionLoader(Dispatcher.class)
                .getAdaptiveExtension().dispatch(handler, url)));
    }
```

​	在ChannelHandlers.wrap函数内会确定消费端Dubbo内部的线程池模型（关于线程池模型，在后面的章节会做具体讲解）。

​	NettyClient的父类AbstractClient的构造函数如下：

```java
public AbstractClient(URL url, ChannelHandler handler) throws RemotingException {
    // 加载编解码器实现
    super(url, handler);
    // set default needReconnect true when channel is not connected
    needReconnect = url.getParameter(Constants.SEND_RECONNECT_KEY, true);

    initExecutor(url);

    try {
        // 调用子类的doOpen
        doOpen();
    } catch (Throwable t) {
        close();
        throw new RemotingException(url.toInetSocketAddress(), null,
                "Failed to start " + getClass().getSimpleName() + " " + NetUtils.getLocalAddress()
                        + " connect to the server " + getRemoteAddress() + ", cause: " + t.getMessage(), t);
    }

    try {
        // connect.
        // 发起远端连接
        connect();
        if (logger.isInfoEnabled()) {
            logger.info("Start " + getClass().getSimpleName() + " " + NetUtils.getLocalAddress() + " connect to the server " + getRemoteAddress());
        }
    } catch (RemotingException t) {
        if (url.getParameter(Constants.CHECK_KEY, true)) {
            close();
            throw t;
        } else {
            logger.warn("Failed to start " + getClass().getSimpleName() + " " + NetUtils.getLocalAddress()
                    + " connect to the server " + getRemoteAddress() + " (check == false, ignore and retry later!), cause: " + t.getMessage(), t);
        }
    } catch (Throwable t) {
        close();
        throw new RemotingException(url.toInetSocketAddress(), null,
                "Failed to start " + getClass().getSimpleName() + " " + NetUtils.getLocalAddress()
                        + " connect to the server " + getRemoteAddress() + ", cause: " + t.getMessage(), t);
    }
}
```

​	在上面的代码中，首先调用了NettyClient的doOpen（）方法：

```java
protected void doOpen() throws Throwable {
    NettyHelper.setNettyLoggerFactory();
    // 创建启动器并配置
    bootstrap = new ClientBootstrap(CHANNEL_FACTORY);
    // config
    // @see org.jboss.netty.channel.socket.SocketChannelConfig
    bootstrap.setOption("keepAlive", true);
    bootstrap.setOption("tcpNoDelay", true);
    bootstrap.setOption("connectTimeoutMillis", getConnectTimeout());
    // 创建业务handler
    final NettyHandler nettyHandler = new NettyHandler(getUrl(), this);
    // 添加handler到链接的管线
    bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
        @Override
        public ChannelPipeline getPipeline() {
            NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyClient.this);
            ChannelPipeline pipeline = Channels.pipeline();
            // 解码器
            pipeline.addLast("decoder", adapter.getDecoder());
            // 编码器
            pipeline.addLast("encoder", adapter.getEncoder());
            // 框架内部使用的handler
            pipeline.addLast("handler", nettyHandler);
            return pipeline;
        }
    });
}
```

​	上面的代码创建了一个启动NettyClient的bootstrap并对其进行设置，这里是把编解码器和自定义的nettyClientHandler添加到了链接Channel对应的管线里，在后面的第13章会做具体讲解。

​	在调用doOpen（）方法后会调用NettyClient的doConnect（）方法与服务提供者建立TCP链接，其中NettyClient的doConnect（）方法的代码如下：

```java
protected void doConnect() throws Throwable {
    long start = System.currentTimeMillis();
    // 发起连接
    ChannelFuture future = bootstrap.connect(getConnectAddress());
    try {
        boolean ret = future.awaitUninterruptibly(getConnectTimeout(), TimeUnit.MILLISECONDS);

        if (ret && future.isSuccess()) {
            Channel newChannel = future.getChannel();
            newChannel.setInterestOps(Channel.OP_READ_WRITE);
            try {
                // Close old channel
                Channel oldChannel = NettyClient.this.channel; // copy reference
                if (oldChannel != null) {
                    try {
                        if (logger.isInfoEnabled()) {
                            logger.info("Close old netty channel " + oldChannel + " on create new netty channel " + newChannel);
                        }
                        oldChannel.close();
                    } finally {
                        NettyChannel.removeChannelIfDisconnected(oldChannel);
                    }
                }
            } finally {
                if (NettyClient.this.isClosed()) {
                    try {
                        if (logger.isInfoEnabled()) {
                            logger.info("Close new netty channel " + newChannel + ", because the client closed.");
                        }
                        newChannel.close();
                    } finally {
                        NettyClient.this.channel = null;
                        NettyChannel.removeChannelIfDisconnected(newChannel);
                    }
                } else {
                    NettyClient.this.channel = newChannel;
                }
            }
        } else if (future.getCause() != null) {
            throw new RemotingException(this, "client(url: " + getUrl() + ") failed to connect to server "
                    + getRemoteAddress() + ", error message is:" + future.getCause().getMessage(), future.getCause());
        } else {
            throw new RemotingException(this, "client(url: " + getUrl() + ") failed to connect to server "
                    + getRemoteAddress() + " client-side timeout "
                    + getConnectTimeout() + "ms (elapsed: " + (System.currentTimeMillis() - start) + "ms) from netty client "
                    + NetUtils.getLocalHost() + " using dubbo version " + Version.getVersion());
        }
    } finally {
        if (!isConnected()) {
            future.cancel();
        }
    }
}
```

​	另外需要注意的是，在NettyClient的父类AbstractEndpoint中确定了编解码器，这里默认为DubboCodec：

```java
public AbstractEndpoint(URL url, ChannelHandler handler) {
    super(url, handler);
    this.codec = getChannelCodec(url);
    this.connectTimeout = url.getPositiveParameter(Constants.CONNECT_TIMEOUT_KEY, Constants.DEFAULT_CONNECT_TIMEOUT);
}

protected static Codec2 getChannelCodec(URL url) {
    String codecName = url.getProtocol(); // codec extension name must stay the same with protocol name
    if (ExtensionLoader.getExtensionLoader(Codec2.class).hasExtension(codecName)) {
        return ExtensionLoader.getExtensionLoader(Codec2.class).getExtension(codecName);
    } else {
        return new CodecAdapter(ExtensionLoader.getExtensionLoader(Codec.class)
                .getExtension(codecName));
    }
}
```

​	然后，在doOpen（）方法中，代码NettyCodecAdapteradapter=new NettyCodecAdapter（getCodec（），getUrl（），NettyClient.this）；使用getCodec（）方法获取了该编解码器并封装到NettyCodecAdapter适配器中，然后把编解码器设置到链接Channel的管线中。关于编解码器的使用，后面有专门的章节进行讲解。

​	这里需要注意三点：第一点，由于同一个服务提供者机器可以提供多个服务，那么消费者机器需要与同一个服务提供者机器提供的多个服务共享连接，还是与每个服务都建立一个连接？第二点，消费端是启动时就与服务提供者机器建立好连接吗？第三点，每个服务消费端与服务提供者集群中的所有机器都有连接吗？对于第三点，我们可以看看图3.9中的toRouters（）方法就可以找到答案，其内部是把具体服务的所有服务提供者的URL信息转换为了Invoker，也就是说服务消费端与服务提供者的所有机器都有连接。

​	为了回答上面的问题，我们看看getClients（）方法的代码：

```java
private ExchangeClient[] getClients(URL url) {
    // whether to share connection
	// 不同服务是否共享连接
    boolean useShareConnect = false;

    int connections = url.getParameter(CONNECTIONS_KEY, 0);
    List<ReferenceCountExchangeClient> shareClients = null;
    // if not configured, connection is shared, otherwise, one connection for one service
    // 如果没配置，则默认连接时共享的
    if (connections == 0) {
        useShareConnect = true;

        /*
         * The xml configuration should have a higher priority than properties.
         */
        // xml配置优先级高于属性配置
        String shareConnectionsStr = url.getParameter(SHARE_CONNECTIONS_KEY, (String) null);
        connections = Integer.parseInt(StringUtils.isBlank(shareConnectionsStr) ? ConfigUtils.getProperty(SHARE_CONNECTIONS_KEY,
                DEFAULT_SHARE_CONNECTIONS) : shareConnectionsStr);
        // 获取共享的NettyClient
        shareClients = getSharedClient(url, connections);
    }

    ExchangeClient[] clients = new ExchangeClient[connections];
    for (int i = 0; i < clients.length; i++) {
        if (useShareConnect) {
            // 共享则返回已存在的
            clients[i] = shareClients.get(i);

        } else {
            // 否则创建新的
            clients[i] = initClient(url);
        }
    }

    return clients;
}
```

​	通过上面的代码可知，在默认情况下当消费端引用同一个服务提供者机器上多个服务时，这些服务复用一个Netty连接，这里回答了第一个问题。

​	下面我们从initClient（）方法里看第二个问题的答案：

```java
private ExchangeClient initClient(URL url) {

    // client type setting.
    String str = url.getParameter(CLIENT_KEY, url.getParameter(SERVER_KEY, DEFAULT_REMOTING_CLIENT));

    url = url.addParameter(CODEC_KEY, DubboCodec.NAME);
    // enable heartbeat by default
    url = url.addParameterIfAbsent(HEARTBEAT_KEY, String.valueOf(DEFAULT_HEARTBEAT));

    // BIO is not allowed since it has severe performance issue.
    if (str != null && str.length() > 0 && !ExtensionLoader.getExtensionLoader(Transporter.class).hasExtension(str)) {
        throw new RpcException("Unsupported client type: " + str + "," +
                " supported client type is " + StringUtils.join(ExtensionLoader.getExtensionLoader(Transporter.class).getSupportedExtensions(), " "));
    }

    ExchangeClient client;
    try {
        // connection should be lazy
        // 为惰性连接
        if (url.getParameter(LAZY_CONNECT_KEY, false)) {
            client = new LazyConnectExchangeClient(url, requestHandler);

        } else {
            // 为及时连接
            client = Exchangers.connect(url, requestHandler);
        }

    } catch (RemotingException e) {
        throw new RpcException("Fail to create remoting client for service(" + url + "): " + e.getMessage(), e);
    }

    return client;
}
```

​	上面的代码默认lazy为false，所以当消费端启动时就与提供者建立了连接，这里回答了第二个问题。

​	另外，从DubboProtocol的refer（）方法可知，其内部返回了一个DubboInvoker，这就是原生的invoker对象，服务消费方远程服务转换就是为了这个invoker。图3.9中的步骤17则是对这个invoker进行装饰，即使用一系列Filter形成了责任链，invoker被放到责任链的末尾。下面我们看看ProtocolFilterWrapper的buildInvokerChain（）方法是如何形成责任链的，具体代码如下：

```java
public <T> Invoker<T> buildInvokerChain(final Invoker<T> originalInvoker, String key, String group) {
    Invoker<T> last = originalInvoker;
    // 获取所有激活的Filter，然后使用链表的方式形成责任链
    List<Filter> filters = ExtensionLoader.getExtensionLoader(Filter.class).getActivateExtension(originalInvoker.getUrl(), key, group);

    if (!filters.isEmpty()) {
        for (int i = filters.size() - 1; i >= 0; i--) {
            final Filter filter = filters.get(i);
            final Invoker<T> next = last;
            last = new FilterChainNode<>(originalInvoker, next, filter);
        }
    }

    return last;
}
```

其中，扩展接口Filter对应的实现类如下所示：

![](https://pic.imgdb.cn/item/611275015132923bf8bd612b.jpg)

![](https://pic.imgdb.cn/item/611275895132923bf8be7387.jpg)

​	其中，MonitorFilter和监控中心进行交互，FutureFilter用来实现异步调用，GenericFilter用来实现泛化调用，ActiveLimitFilter用来控制消费端最大并发调用量，ExecuteLimitFilter用来控制服务提供方最大并发处理量等，当然也可以写自己的Filter。

​	需要注意的是，消费端启动时并不是把所有的Filter扩展实现都放到责任链中，而是把group=consumer并且value值在URL里的才会放到责任链中，这一点在2.5节中提到过。

​	由于是责任链，所以ProtocolFilterWrapper的refer（）方法是将责任链头部的Filter返回到ProtocolListenerWrapper。ProtocolListenerWrapper的refer（）方法的代码如下：

```java
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
    if (UrlUtils.isRegistry(url)) {
        return protocol.refer(type, url);
    }

    Invoker<T> invoker = protocol.refer(type, url);
    if (StringUtils.isEmpty(url.getParameter(REGISTRY_CLUSTER_TYPE_KEY))) {
        invoker = new ListenerInvokerWrapper<>(invoker,
                Collections.unmodifiableList(
                        ExtensionLoader.getExtensionLoader(InvokerListener.class)
                                .getActivateExtension(url, INVOKER_LISTENER_KEY)));
    }
    return invoker;
}
```

​	至此可知，在图3.10中，在RegistryDirectory里维护了所有服务者的invoker列表，消费端发起远程调用时就是根据集群容错和负载均衡算法以及路由规则从invoker列表里选择一个进行调用的，当服务提供者宕机的时候，ZooKeeper会通知更新这个invoker列表。

![](https://pic.imgdb.cn/item/611276795132923bf8c06d08.jpg)

​	到这里，我们就讲完了图3.9中directory.subscribe（）方法是如何订阅服务提供者服务的，并且把服务提供者所有的URL信息转换为了invoker列表，并保存到RegistryDirectory里。下面，我们接着讲解图3.9中RegistryProtocol的doRefer（）方法中的cluster.join（directory）是如何使用集群容错扩展将Dubbo协议的invoker客户端转换为需要的接口的。在默认情况下，cluster的扩展接口实现为FailoverCluster，所以这里是调用FailoverCluster的join（）方法，FailoverCluster的join（）方法的代码如下：

```java
public class FailoverCluster extends AbstractCluster {

    public final static String NAME = "failover";

    @Override
    public <T> AbstractClusterInvoker<T> doJoin(Directory<T> directory) throws RpcException {
        return new FailoverClusterInvoker<>(directory);
    }

}
```

​	这里是把directory对象包裹到了FailoverClusterInvoker里，需要注意的是，directory就是上面讲解的RegistryDirectory，其内部维护了所有服务提供者的invoker列表，而FailoverCluster就是集群容错策略。

​	其实，Dubbo对cluster扩展接口实现类使用Wrapper类MockClusterWrapper进行增强，这一点从图3.11可以得到证明。

![](https://pic.imgdb.cn/item/6112771b5132923bf8c1bdfa.jpg)

​	实际上调用的时序图如图3.12所示。

![](https://pic.imgdb.cn/item/6112772a5132923bf8c1dd8f.jpg)

​	图3.12中的步骤3将FailbackClusterInvoker对象返回到步骤2，下面看看MockClusterWrapper类的代码：

```java
public class MockClusterWrapper implements Cluster {

    private Cluster cluster;

    public MockClusterWrapper(Cluster cluster) {
        this.cluster = cluster;
    }

    @Override
    public <T> Invoker<T> join(Directory<T> directory) throws RpcException {
        return new MockClusterInvoker<T>(directory,
                this.cluster.join(directory));
    }

}
```

​	从上面的代码可知，MockClusterWrapper类把FailoverClusterInvoker包装成了MockClusterInvoker实例，所以整个调用链最终调用返回的是MockClusterInvoker对象。也就是说，本节第一个时序图（见图3.8）中的步骤4返回的是MockClusterWrapper，然后执行图3.9中的步骤14以获取MockClusterInvoker的代理，实现invoker到客户端接口的转换，这里默认调用的是JavassistProxyFactory的getProxy（）方法，其代码如下：

```java
public <T> T getProxy(Invoker<T> invoker, Class<?>[] interfaces) {
    return (T) Proxy.getProxy(interfaces).newInstance(new InvokerInvocationHandler(invoker));
}
```

​	其中，InvokerInvocationHandler为具体拦截器。至此，我们按照逆序的方式把服务消费端启动流程讲解完了，下面一节讲解一次远程调用过程。

## Dubbo服务消费端一次远程调用过程

​	我们先看看整体时序图，如图3.13所示。

![](https://pic.imgdb.cn/item/611277c05132923bf8c324f3.jpg)

​	在3.3节中，我们提到服务消费端通过ReferenceConfig的get（）方法返回的是一个代理类，并且方法拦击器为InvokerInvocationHandler，所以当消费方调用了服务的接口方法后会被InvokerInvocationHandler拦截，执行图3.13所示的流程。

​	在图3.13所示的流程中，步骤1的代码如下：

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (method.getDeclaringClass() == Object.class) {
        return method.invoke(invoker, args);
    }
    String methodName = method.getName();
    Class<?>[] parameterTypes = method.getParameterTypes();
    if (parameterTypes.length == 0) {
        if ("toString".equals(methodName)) {
            return invoker.toString();
        } else if ("$destroy".equals(methodName)) {
            invoker.destroy();
            return null;
        } else if ("hashCode".equals(methodName)) {
            return invoker.hashCode();
        }
    } else if (parameterTypes.length == 1 && "equals".equals(methodName)) {
        return invoker.equals(args[0]);
    }
    RpcInvocation rpcInvocation = new RpcInvocation(method, invoker.getInterface().getName(), protocolServiceKey, args);
    String serviceKey = url.getServiceKey();
    rpcInvocation.setTargetServiceUniqueName(serviceKey);

    // invoker.getUrl() returns consumer url.
    RpcServiceContext.setRpcContext(url);

    if (consumerModel != null) {
        rpcInvocation.put(Constants.CONSUMER_MODEL, consumerModel);
        rpcInvocation.put(Constants.METHOD_MODEL, consumerModel.getMethodModel(method));
    }

    return invoker.invoke(rpcInvocation).recreate();
}
```

​	从上面的代码可知，这里创建了RpcInvocation，其中method为调用的方法，args为参数，这个RpcInvocation对象会一直传递，直到发起远程调用（下面对此会做讲解）。

​	这里需要注意的一点是，如果服务接口的方法的返回值为CompletableFuture类或者其子类，则需要设置标记future_returntype为true和async=true，这些属性需要设置到附加属性RpcInvocation里：（3.0.1版本没看到)

![](https://pic.imgdb.cn/item/611278f35132923bf8c5ba2a.jpg)

​	这些内容在11.2.1小节会做具体讲解。

​	步骤2和步骤3调用了默认的集群容错策略FailoverClusterInvoker，其内部首先根据设置的负载均衡策略LoadBalance的扩展实现，选择一个invoker作为FailoverClusterInvoker具体的远程调用者，如果调用发生异常，则根据FailoverClusterInvoker的策略重新选择一个invoker进行调用。

​	在FailoverClusterInvoker内每次调用invoker的invoke（）方法时，都会走到步骤8和步骤9，后面的步骤10、步骤11和步骤12是在ProtocolFilterWrapper内创建的责任链，最后调用了原生的DubboInvoker，其使用NettyClient与服务提供者进行交互，其中DubboInvoker的doInvoke（）方法的内容如下：

```java
protected Result doInvoke(final Invocation invocation) throws Throwable {
    // 设置附加
    RpcInvocation inv = (RpcInvocation) invocation;
    final String methodName = RpcUtils.getMethodName(invocation);
    inv.setAttachment(PATH_KEY, getUrl().getPath());
    inv.setAttachment(VERSION_KEY, version);
	
    // 获取远程调用Client
    ExchangeClient currentClient;
    if (clients.length == 1) {
        currentClient = clients[0];
    } else {
        currentClient = clients[index.getAndIncrement() % clients.length];
    }
    // 执行远程调用
    try {
        // 是否为oneWay，也就是不需要响应结果的请求
        boolean isOneway = RpcUtils.isOneway(getUrl(), invocation);
        // 超时等待时间
        int timeout = calculateTimeout(invocation, methodName);
        invocation.put(TIMEOUT_KEY, timeout);
        // 不需要响应的请求
        if (isOneway) {
            boolean isSent = getUrl().getMethodParameter(methodName, Constants.SENT_KEY, false);
            currentClient.send(inv, isSent);
            return AsyncRpcResult.newDefaultAsyncResult(invocation);
        } else {
            // 
            ExecutorService executor = getCallbackExecutor(getUrl(), inv);
            CompletableFuture<AppResponse> appResponseFuture =
                    currentClient.request(inv, timeout, executor).thenApply(obj -> (AppResponse) obj);
            // save for 2.6.x compatibility, for example, TraceFilter in Zipkin uses com.alibaba.xxx.FutureAdapter
            FutureContext.getContext().setCompatibleFuture(appResponseFuture);
            AsyncRpcResult result = new AsyncRpcResult(appResponseFuture, inv);
            result.setExecutor(executor);
            return result;
        }
    } catch (TimeoutException e) {
        throw new RpcException(RpcException.TIMEOUT_EXCEPTION, "Invoke remote method timeout. method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
    } catch (RemotingException e) {
        throw new RpcException(RpcException.NETWORK_EXCEPTION, "Failed to invoke remote method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```

​	从上面的代码可知，程序首先获取远程调用Client，然后判断调用是否为异步调用、是否请求响应。

​	如果请求不需要响应结果则直接使用远程Client发起请求调用，然后将RpcContext上下文的future设置为null，并且返回空的RpcResult。

​	如果请求是异步请求，则保存远程Client发起请求后返回的future对象，并且设置到RpcContext上下文中，这样，调用方就可以通过RpcContext上下文获取该future。

​	如果请求为同步请求，则首先设置RpcContext上下文中的future对象为null，然后使用远程Client发起请求，然后在返回的future对象上调用get（）方法，以同步等待远程调用结果的返回。

# Directory目录与Router路由服务

## Directory目录

​	Directory代表了多个invoker（对于消费端来说，每个invoker代表了一个服务提供者），其内部维护着一个List，并且这个List的内容是动态变化的，比如当服务提供者集群新增或者减少机器时，服务注册中心就会推送当前服务提供者的地址列表，然后Directory中的List就会根据服务提供者地址列表相应变化。

​	在Dubbo中，接口Directory的实现有RegistryDirectory和StaticDirectory两种，其中前者管理的invoker列表是根据服务注册中心的推送变化而变化的，而后者是当消费端使用了多注册中心时，其把所有服务注册中心的invoker列表汇集到一个invoker列表中。

本章我们主要从下面几个方面来探讨RegistryDirectory：

* 消费端启动时何时构建RegistryDirectory
* RegistryDirectory管理的invoker列表如何动态变化
* 路由信息是如何保存与变化的

## RegistryDirectory的创建

![](https://pic.imgdb.cn/item/61127c2d5132923bf8ccdf2b.jpg)

​	通过图4.1可知，RegistryDirectory是在服务消费端启动时创建的。

* 在2.1节中我们提到，ReferenceConfig代表一个要消费的服务的配置对象，调用ReferenceConfig的get（）方法，就意味着要创建一个对服务提供方的远程调用代理。
* 在图4.1中，步骤2在创建对远程服务提供者的代理时，第一步是调用RegistryProtocol类的refer（）方法，由于RegistryProtocol是一个SPI，所以这里是通过其适配器类Protocol$Adaptive进行间接调用的。另外，这里的ProtocolListenerWrapper、QosProtocolWrapper和ProtocolFilterWrapper是对RegistryProtocol类的功能的增强。

## RegistryDirectory中invoker列表的更新

​	上一节探讨了RegistryDirectory何时被创建，本节我们看看其管理的invoker列表是何时被创建与更新的。

​	下面看看RegistryDirectory中invoker列表的更新流程，如图4.2所示。

​	通过图4.2可知，创建完RegistryDirectory后，调用了其subscribe（）方法，这里假设使用的服务注册中心为ZooKeeper，这样就会去ZooKeeper订阅需要调用的服务提供者的地址列表，然后步骤3添加了一个监听器，当ZooKeeper服务端发现服务提供者地址列表发生变化后，会将地址列表推送到服务消费端，然后zkClient会回调该监听器的notify（）方法。

![](https://pic.imgdb.cn/item/61127c7d5132923bf8cda507.jpg)

​	在步骤3设置完监听器后，同步返回了订阅的服务地址列表、路由规则、配置信息等，然后同步调用了RegistryDirectory的notify（）方法：

```java
	步骤6从ZooKeeper返回的服务提供者的信息里获取对应的路由规则，并使用步骤7保存到RouterChain里，这个路由规则是通过管理控制台进行配置的，如图4.3所示。
        public synchronized void notify(List<URL> urls) {
    if (isDestroyed()) {
        return;
    }

    Map<String, List<URL>> categoryUrls = urls.stream()
            .filter(Objects::nonNull)
            .filter(this::isValidCategory)
            .filter(this::isNotCompatibleFor26x)
            .collect(Collectors.groupingBy(this::judgeCategory));

    List<URL> configuratorURLs = categoryUrls.getOrDefault(CONFIGURATORS_CATEGORY, Collections.emptyList());
    this.configurators = Configurator.toConfigurators(configuratorURLs).orElse(this.configurators);
	// 动态配置信息
    List<URL> routerURLs = categoryUrls.getOrDefault(ROUTERS_CATEGORY, Collections.emptyList());
    toRouters(routerURLs).ifPresent(this::addRouters);

    // providers
    List<URL> providerURLs = categoryUrls.getOrDefault(PROVIDERS_CATEGORY, Collections.emptyList());
    /**
     * 3.x added for extend URL address
     */
    ExtensionLoader<AddressListener> addressListenerExtensionLoader = ExtensionLoader.getExtensionLoader(AddressListener.class);
    List<AddressListener> supportedListeners = addressListenerExtensionLoader.getActivateExtension(getUrl(), (String[]) null);
    // 服务提供者列表信息
    if (supportedListeners != null && !supportedListeners.isEmpty()) {
        for (AddressListener addressListener : supportedListeners) {
            providerURLs = addressListener.notify(providerURLs, getConsumerUrl(),this);
        }
    }
    // 刷新invoker列表
    refreshOverrideAndInvoker(providerURLs);
}
```

![](https://pic.imgdb.cn/item/611280f75132923bf8d87000.jpg)

​	步骤9根据服务降级信息，重写URL（也就是把mock=returnnull等信息拼接到URL中）并保存到overrideDirectoryUrl中，然后执行步骤10，把服务提供者的URL列表转换为invoker列表，并保存到RouterChain里：

```java
private void refreshInvoker(List<URL> invokerUrls) {
    Assert.notNull(invokerUrls, "invokerUrls should not be null");
	// 只有一个服务提供者时
    if (invokerUrls.size() == 1
            && invokerUrls.get(0) != null
            && EMPTY_PROTOCOL.equals(invokerUrls.get(0).getProtocol())) {
        this.forbidden = true; // Forbid to access
        this.invokers = Collections.emptyList();
        routerChain.setInvokers(this.invokers);
        destroyAllInvokers(); // Close all invokers
    } else {
        // 多个提供者时
        this.forbidden = false; // Allow to access
        Map<URL, Invoker<T>> oldUrlInvokerMap = this.urlInvokerMap; // local reference
        if (invokerUrls == Collections.<URL>emptyList()) {
            invokerUrls = new ArrayList<>();
        }
        if (invokerUrls.isEmpty() && this.cachedInvokerUrls != null) {
            invokerUrls.addAll(this.cachedInvokerUrls);
        } else {
            this.cachedInvokerUrls = new HashSet<>();
            this.cachedInvokerUrls.addAll(invokerUrls);//Cached invoker urls, convenient for comparison
        }
        if (invokerUrls.isEmpty()) {
            return;
        }
        // URL转换为invoker对象
        Map<URL, Invoker<T>> newUrlInvokerMap = toInvokers(invokerUrls);// Translate url list to Invoker map

        /**
         * If the calculation is wrong, it is not processed.
         *
         * 1. The protocol configured by the client is inconsistent with the protocol of the server.
         *    eg: consumer protocol = dubbo, provider only has other protocol services(rest).
         * 2. The registration center is not robust and pushes illegal specification data.
         *
         */
        if (CollectionUtils.isEmptyMap(newUrlInvokerMap)) {
            logger.error(new IllegalStateException("urls to invokers error .invokerUrls.size :" + invokerUrls.size() + ", invoker.size :0. urls :" + invokerUrls
                    .toString()));
            return;
        }
		// 设置newInvokers到routerChain
        List<Invoker<T>> newInvokers = Collections.unmodifiableList(new ArrayList<>(newUrlInvokerMap.values()));
        // pre-route and build cache, notice that route cache should build on original Invoker list.
        // toMergeMethodInvokerMap() will wrap some invokers having different groups, those wrapped invokers not should be routed.
        routerChain.setInvokers(newInvokers);
        this.invokers = multiGroup ? toMergeInvokerList(newInvokers) : newInvokers;
        this.urlInvokerMap = newUrlInvokerMap;
		// 关闭无用的invokers
        try {
            destroyUnusedInvokers(oldUrlInvokerMap, newUrlInvokerMap); // Close the unused Invoker
        } catch (Exception e) {
            logger.warn("destroyUnusedInvokers error. ", e);
        }

        // notify invokers refreshed
        this.invokersChanged();
    }
}
```

​	另外，RouterChain里也保存了可用服务提供者对应的invokers列表和路由规则信息，当服务消费方的集群容错策略要获取可用服务提供者对应的invoker列表时，会调用RouterChain的route（）方法，其内部根据路由规则信息和invokers列表来提供服务，流程如图4.4所示。

![](https://pic.imgdb.cn/item/611283055132923bf8dd866c.jpg)

```java
public List<Invoker<T>> route(URL url, Invocation invocation) {

    AddrCache<T> cache = this.cache.get();
    if (cache == null) {
        throw new RpcException(RpcException.ROUTER_CACHE_NOT_BUILD, "Failed to invoke the method "
            + invocation.getMethodName() + " in the service " + url.getServiceInterface()
            + ". address cache not build "
            + " on the consumer " + NetUtils.getLocalHost()
            + " using the dubbo version " + Version.getVersion()
            + ".");
    }
    BitList<Invoker<T>> finalBitListInvokers = new BitList<>(invokers, false);
    for (StateRouter stateRouter : stateRouters) {
        if (stateRouter.isEnable()) {
            RouterCache<T> routerCache = cache.getCache().get(stateRouter.getName());
            finalBitListInvokers = stateRouter.route(finalBitListInvokers, routerCache, url, invocation);
        }
    }

    List<Invoker<T>> finalInvokers = new ArrayList<>(finalBitListInvokers.size());

    for(Invoker<T> invoker: finalBitListInvokers) {
        finalInvokers.add(invoker);
    }

    for (Router router : routers) {
        finalInvokers = router.route(finalInvokers, url, invocation);
    }
    return finalInvokers;
}
```

​	总结一下就是，在服务消费端应用中，每个需要消费的服务都被包装为ReferenceConfig，在应用启动时会调用每个服务对应的ReferenceConfig的get（）方法，然后会为每个服务创建一个自己的RegistryDirectory对象，每个RegistryDirectory管理该服务提供者的地址列表、路由规则、动态配置等信息，当服务提供者的信息发生变化时，RegistryDirectory会动态地得到变化通知，并自动更新。

​	至于RouterChain链是什么时候创建的，我们已经在3.3节提到过，即RouterChain链是在消费端启动过程中通过RegistryProtocol的doRefer（）方法调用RegistryDirectory的buildRouterChain（）方法创建的。

# Dubbo消费端服务mock与服务降级策略原理

## 服务降级原理

### 降级策略注册

​	在基础篇中，我们介绍了使用下面的代码可以将服务降级信息注册到ZooKeeper：

![](https://pic.imgdb.cn/item/611286855132923bf8e595a0.jpg)

​	当以参数force调用上述代码时，降级策略就会写入ZooKeeper服务器com.books.dubbo.demo.api.GreetingService子树中Type为configurators的下面，如图5.1所示。当然，也可以使用admin手动配置服务降级策略。

![](https://pic.imgdb.cn/item/611286ca5132923bf8e63f82.jpg)

​	当服务消费者启动时，会去订阅com.books.dubbo.demo.api.GreetingService子树中的信息，比如Providers（服务提供者列表）、Routes（路由信息）、Configurators（服务降级策略）等信息，这些内容在第4章已经讲过。

​	当服务消费者发起远程调用时，会看是否设置了force：return降级策略，如果设置了则不发起远程调用并直接返回mock值，否则发起远程调用。当远程调用结果OK时，直接返回远程调用返回的结果；如果远程调用失败了，则看当前是否设置了fail：return的降级策略，如果设置了，则直接返回mock值，否则返回调用远程服务失败的具体原因。

### 服务消费端使用降级策略

​	首先，我们看一幅简化的时序图，了解一下从消费端发起远程调用的流程，如图5.2所示。

![](https://pic.imgdb.cn/item/611287175132923bf8e6f895.jpg)

​	从图中可以看出，服务消费端是在MockClusterInvoker类的invoke（）方法里使用降级策略并在DubboInvoker的invoke（）方法里发起远程调用的，所以服务降级是在消费端还没有发起远程调用时完成的。

​	本节主要讲解服务降级，所以我们看看MockClusterInvoker类的代码：

```java
public Result invoke(Invocation invocation) throws RpcException {
    Result result = null;
	// 查看url里面是否有mock字段
    String value = getUrl().getMethodParameter(invocation.getMethodName(), MOCK_KEY, Boolean.FALSE.toString()).trim();
    // 如果没有，或者值为默认的false，则说明没有设置降级策略
    if (value.length() == 0 || "false".equalsIgnoreCase(value)) {
        //no mock
        // 没有mock。正常发起远程调用
        result = this.invoker.invoke(invocation);
    } else if (value.startsWith("force")) {
        // 设置了force:return降级策略
        if (logger.isWarnEnabled()) {
            logger.warn("force-mock: " + invocation.getMethodName() + " force-mock enabled , url : " + getUrl());
        }
        //force:direct mock
        result = doMockInvoke(invocation, null);
    } else {
        // 设置了fail-mock
        //fail-mock
        try {
            result = this.invoker.invoke(invocation);

            //fix:#4585
            if(result.getException() != null && result.getException() instanceof RpcException){
                RpcException rpcException= (RpcException)result.getException();
                if(rpcException.isBiz()){
                    throw  rpcException;
                }else {
                    result = doMockInvoke(invocation, rpcException);
                }
            }

        } catch (RpcException e) {
            if (e.isBiz()) {
                throw e;
            }

            if (logger.isWarnEnabled()) {
                logger.warn("fail-mock: " + invocation.getMethodName() + " fail-mock enabled , url : " + getUrl(), e);
            }
            result = doMockInvoke(invocation, e);
        }
    }
    return result;
}
```

​	通过上面的代码可知，其中directory.getUrl（）方法获取的就是上一节我们讲解overrideDirectoryUrl，代码1查看该URL里面是否含有mock的值，如果没有或者有但是值为false，说明没有设置降级策略，则执行代码2.1，也就是正常发起远程调用。

​	如果URL里面含有mock字段，并且其值以force开头，则说明设置了force：return降级策略，那么直接调用doMockInvoke（）方法（其内部会调用创建的MockInvoker的invoke（）方法），返回mock值，而不发起远程调用。

​	如果URL里面含有mock字段，并且其值以fail开头，则说明设置了fail：return降级策略，那么先发起远程调用。如果远程调用成功，则直接返回远程返回的结果（如果这里使用了默认的集群容错策略，则代码4.1会调用FailoverClusterInvokerd的invoke（）方法发起远程调用）；如果发起远程调用失败，则执行代码4.3，直接返回mock的值。

## 本地服务mock原理

### mock合法性检查

​	在3.3节我们讲解了在服务引用启动时会在ReferenceConfig的init（）方法内调用checkMock来检查设置的mock的正确性，下面我们看看相应的代码：

```java
public static void checkMock(Class<?> interfaceClass, AbstractInterfaceConfig config) {
    // 没有设置mock，则直接返回
    String mock = config.getMock();
    if (ConfigUtils.isEmpty(mock)) {
        return;
    }
	// 格式化mock方式
    String normalizedMock = MockInvoker.normalizeMock(mock);
    // 检查mock值是否合法，不合法则抛出异常
    if (normalizedMock.startsWith(RETURN_PREFIX)) {
        normalizedMock = normalizedMock.substring(RETURN_PREFIX.length()).trim();
        try {
            //Check whether the mock value is legal, if it is illegal, throw exception
            MockInvoker.parseMockValue(normalizedMock);
        } catch (Exception e) {
            throw new IllegalStateException("Illegal mock return in <dubbo:service/reference ... " +
                "mock=\"" + mock + "\" />");
        }
    } else if (normalizedMock.startsWith(THROW_PREFIX)) {
        normalizedMock = normalizedMock.substring(THROW_PREFIX.length()).trim();
        if (ConfigUtils.isNotEmpty(normalizedMock)) {
            try {
                //Check whether the mock value is legal
                MockInvoker.getThrowable(normalizedMock);
            } catch (Exception e) {
                throw new IllegalStateException("Illegal mock throw in <dubbo:service/reference ... " +
                    "mock=\"" + mock + "\" />");
            }
        }
    } else {
        //Check whether the mock class is a implementation of the interfaceClass, and if it has a default constructor
        MockInvoker.getMockObject(normalizedMock, interfaceClass);
    }
}
```

​	如果代码1没有设置mock规则，则直接返回，否则代码2对mock规则进行格式化：

![](https://pic.imgdb.cn/item/611289b35132923bf8ed2b60.jpg)

​	例如，如果设置mock为return，则格式化后为return null。

​	代码将检查mock值是否合法，不合法将抛出异常。由于我们的Demo是将mock设置为true，所以我们会看到代码4将检查mock接口的实现类是否符合规则，MockInvoker.getMockObject的代码如下：

```java
public static Object getMockObject(String mockService, Class serviceType) {
    // 如果mock类型为true或者default
    boolean isDefault = ConfigUtils.isDefault(mockService);
    if (isDefault) {
        mockService = serviceType.getName() + "Mock";
    }
	// 反射加载字节码创建Class对象
    Class<?> mockClass;
    try {
        mockClass = ReflectUtils.forName(mockService);
    } catch (Exception e) {
        if (!isDefault) {// does not check Spring bean if it is default config.
            ExtensionFactory extensionFactory =
                    ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension();
            Object obj = extensionFactory.getExtension(serviceType, mockService);
            if (obj != null) {
                return obj;
            }
        }
        throw new IllegalStateException("Did not find mock class or instance "
                + mockService
                + ", please check if there's mock class or instance implementing interface "
                + serviceType.getName(), e);
    }
    if (mockClass == null || !serviceType.isAssignableFrom(mockClass)) {
        throw new IllegalStateException("The mock class " + mockClass.getName() +
                " not implement interface " + serviceType.getName());
    }
	// 创建实例
    try {
        return mockClass.newInstance();
    } catch (InstantiationException e) {
        throw new IllegalStateException("No default constructor from mock class " + mockClass.getName(), e);
    } catch (IllegalAccessException e) {
        throw new IllegalStateException(e);
    }
}
```

​	如果代码5中的mock类型为true或者deafult，则mockService被设置为接口名称加上Mock，例如，如果接口为com.books.dubbo.demo.api.GreetingServic并且mock设置为true，则这里的mockService就是com.books.dubbo.demo.api.GreetingServicMock，然后代码6加载com.books.dubbo.demo.api.GreetingServicMock的字节码文件以创建Class对象，代码7则创建实例。上面这些代码的作用是查看用户程序的classpath下是否存在mock类的实现com.books.dubbo.demo.api.GreetingServicMock，如果不存在，则抛出异常。

### 服务消费端使用mock服务

​	mock服务与服务降级策略一样，也是在MockClusterInvoker中实现的，这里我们再看看其中的invoke（）方法：

```java
public Result invoke(Invocation invocation) throws RpcException {
        Result result = null;

        String value = getUrl().getMethodParameter(invocation.getMethodName(), MOCK_KEY, Boolean.FALSE.toString()).trim();
    	// 没有mock
        if (value.length() == 0 || "false".equalsIgnoreCase(value)) {
            //no mock
            result = this.invoker.invoke(invocation);
        } else if (value.startsWith("force")) {
            // force
            if (logger.isWarnEnabled()) {
                logger.warn("force-mock: " + invocation.getMethodName() + " force-mock enabled , url : " + getUrl());
            }
            //force:direct mock
            result = doMockInvoke(invocation, null);
        } else {
            //fail-mock
            // force:direct mock
            try {
                result = this.invoker.invoke(invocation);

                //fix:#4585
                if(result.getException() != null && result.getException() instanceof RpcException){
                    RpcException rpcException= (RpcException)result.getException();
                    if(rpcException.isBiz()){
                        throw  rpcException;
                    }else {
                        result = doMockInvoke(invocation, rpcException);
                    }
                }

            } catch (RpcException e) {
                if (e.isBiz()) {
                    throw e;
                }

                if (logger.isWarnEnabled()) {
                    logger.warn("fail-mock: " + invocation.getMethodName() + " fail-mock enabled , url : " + getUrl(), e);
                }
                result = doMockInvoke(invocation, e);
            }
        }
        return result;
    }
```

​	由于我们设置的mock为true，所以这里的value为true，代码会执行到步骤3也就是fail-mock阶段，这也说明了服务mock与服务降级的fail-mock功能相似，不同之处在于前者会设置mock服务实现类，而后者是返回设置的静态返回值。代码3会首先发起远程RPC，如果成功则不会执行mock操作，否则执行doMockInvoke。

```java
private Result doMockInvoke(Invocation invocation, RpcException e) {
    Result result = null;
    Invoker<T> minvoker;

    List<Invoker<T>> mockInvokers = selectMockInvoker(invocation);
    if (CollectionUtils.isEmpty(mockInvokers)) {
        minvoker = (Invoker<T>) new MockInvoker(getUrl(), directory.getInterface());
    } else {
        minvoker = mockInvokers.get(0);
    }
    try {
        result = minvoker.invoke(invocation);
    } catch (RpcException me) {
        if (me.isBiz()) {
            result = AsyncRpcResult.newDefaultAsyncResult(me.getCause(), invocation);
        } else {
            throw new RpcException(me.getCode(), getMockExceptionMessage(e, me), me.getCause());
        }
    } catch (Throwable me) {
        throw new RpcException(getMockExceptionMessage(e, me), me.getCause());
    }
    return result;
}
```

​	MockInvoker的invoke（）方法如下：

```java
public Result invoke(Invocation invocation) throws RpcException {
    if (invocation instanceof RpcInvocation) {
        ((RpcInvocation) invocation).setInvoker(this);
    }
    String mock = null;
    // mock类型
    if (getUrl().hasMethodParameter(invocation.getMethodName())) {
        mock = getUrl().getParameter(invocation.getMethodName() + "." + MOCK_KEY);
    }
    if (StringUtils.isBlank(mock)) {
        mock = getUrl().getParameter(MOCK_KEY);
    }

    if (StringUtils.isBlank(mock)) {
        throw new RpcException(new IllegalAccessException("mock can not be null. url :" + url));
    }
    // 格式化
    mock = normalizeMock(URL.decode(mock));
    // 不同的类型，返回不同的mock值
    if (mock.startsWith(RETURN_PREFIX)) {
        mock = mock.substring(RETURN_PREFIX.length()).trim();
        try {
            Type[] returnTypes = RpcUtils.getReturnTypes(invocation);
            Object value = parseMockValue(mock, returnTypes);
            return AsyncRpcResult.newDefaultAsyncResult(value, invocation);
        } catch (Exception ew) {
            throw new RpcException("mock return invoke error. method :" + invocation.getMethodName()
                    + ", mock:" + mock + ", url: " + url, ew);
        }
    } else if (mock.startsWith(THROW_PREFIX)) {
        mock = mock.substring(THROW_PREFIX.length()).trim();
        if (StringUtils.isBlank(mock)) {
            throw new RpcException("mocked exception for service degradation.");
        } else { // user customized class
            Throwable t = getThrowable(mock);
            throw new RpcException(RpcException.BIZ_EXCEPTION, t);
        }
    } else { //impl mock
        try {
            Invoker<T> invoker = getInvoker(mock);
            return invoker.invoke(invocation);
        } catch (Throwable t) {
            throw new RpcException("Failed to create mock implementation class " + mock, t);
        }
    }
}
```

​	通过上面的代码可知，根据mock类型的不同，返回不同的mock值。这里我们重点看看代码7，即mock实现类GreetingServiceMock如何返回mock值：

![](https://pic.imgdb.cn/item/61128c405132923bf8f338c7.jpg)

![](https://pic.imgdb.cn/item/61128c4c5132923bf8f3580b.jpg)

​	通过上面的代码可知，如果缓存里含有mock实现类对应的invoker对象，则直接返回，否则使用getMockObject（）方法创建对象实例，并进行代理后保存到缓存，然后返回。在调用代理的invoke（）方法后，就会调用mock实现类GreetingServiceMock的方法。

# Dubbo集群容错与负载均衡策略

## Dubbo集群容错策略概述

下面让我们看看Dubbo提供的集群容错模式。

* Failover Cluster：失败重试

​	当服务消费方调用服务提供者失败后，会自动切换到其他服务提供者服务器进行重试，这通常用于读操作或者具有幂等的写操作。需要注意的是，重试会带来更长延迟。可以通过retries="2"来设置重试次数（不含第1次）。

​	可以使用＜dubbo：reference retries="2"/＞来进行接口级别配置的重试次数，当服务消费方调用服务失败后，此例子会再重试两次，也就是说最多会做3次调用，这里的配置对该接口的所有方法生效。

​	当然你也可以针对某个方法配置重试次数，比如：

![](https://pic.imgdb.cn/item/6113c40d5132923bf8e77d9a.jpg)

* Failfast Cluster：快速失败

​	当服务消费方调用服务提供者失败后，立即报错，也就是只调用一次。通常，这种模式用于非幂等性的写操作。

* Failsafe Cluster：安全失败

  当服务消费者调用服务出现异常时，直接忽略异常。这种模式通常用于写入审计日志等操作。

* Failback Cluster：失败自动恢复

  当服务消费端调用服务出现异常后，在后台记录失败的请求，并按照一定的策略后期再进行重试。这种模式通常用于消息通知操作。

* Forking Cluster：并行调用

  当消费方调用一个接口方法后，Dubbo Client会并行调用多个服务提供者的服务，只要其中有一个成功即返回。这种模式通常用于实时性要求较高的读操作，但需要浪费更多服务资源。如下代码可通过forks="4"来设置最大并行数：

![](https://pic.imgdb.cn/item/6113c4935132923bf8e995bd.jpg)

* Broadcast Cluster：广播调用

  当消费者调用一个接口方法后，Dubbo Client会逐个调用所有服务提供者，任意一台服务器调用异常则这次调用就标志失败。这种模式通常用于通知所有提供者更新缓存或日志等本地资源信息。

​	可以根据Dubbo提供的扩展接口Cluster进行定制。

​	首先，我们来看看服务消费端发起远程调用的一个简化时序图，如图6.1所示。

![](https://pic.imgdb.cn/item/6113c4da5132923bf8eab327.jpg)

​	图6.1是采用默认的FailOver集群容错方法时的调用时序图，从中可以看到，调用集群容错是在服务降级策略后进行的，集群容错FailoverClusterInvoker内部首先会调用父类AbstractClusterInvoker的list（）方法来获取invoker列表，即从RegistryDirectory管理的RouterChain的route（）方法中获取保存的invoker列表（参见第4章）。

## Failfast Cluster策略源码分析

​	在Dubbo中，实现快速失败策略的是FailfastClusterInvoker类，这里我们看看具体实现，主要看看doInvoke（）方法的代码：

```java
public Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    checkInvokers(invokers, invocation);
    // 负载均衡策略选择一个服务提供者
    Invoker<T> invoker = select(loadbalance, invocation, invokers, null);
    try {
        // 执行远程调用
        return invokeWithContext(invoker, invocation);
    } catch (Throwable e) {
        // 出错则抛出异常
        if (e instanceof RpcException && ((RpcException) e).isBiz()) { // biz exception.
            throw (RpcException) e;
        }
        throw new RpcException(e instanceof RpcException ? ((RpcException) e).getCode() : 0,
                "Failfast invoke providers " + invoker.getUrl() + " " + loadbalance.getClass().getSimpleName()
                        + " for service " + getInterface().getName()
                        + " method " + invocation.getMethodName() + " on consumer " + NetUtils.getLocalHost()
                        + " use dubbo version " + Version.getVersion()
                        + ", but no luck to perform the invocation. Last error is: " + e.getMessage(),
                e.getCause() != null ? e.getCause() : e);
    }
}
```

## Failsafe Cluster策略源码分析

​	在Dubbo中，实现失败安全的是FailsafeClusterInvoker类，这里我们看看具体实现，主要看看doInvoke（）方法的代码：

```java
public Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    try {
        checkInvokers(invokers, invocation);
        Invoker<T> invoker = select(loadbalance, invocation, invokers, null);
        return invokeWithContext(invoker, invocation);
    } catch (Throwable e) {
        logger.error("Failsafe ignore exception: " + e.getMessage(), e);
        return AsyncRpcResult.newDefaultAsyncResult(null, null, invocation); // ignore
    }
}
```

​	代码1根据具体的负载均衡策略选择一个服务提供者对应的invoker对象。

​	代码2使用选择的invoker对象发起远程RPC调用。

​	与Failfast的不同之处在于，该策略不会把异常抛给调用者。

## Failover Cluster策略源码分析

​	在Dubbo中，实现失败重试的是FailoverClusterInvoker类，这里我们看看具体实现，主要看看doInvoke（）方法的代码：

```java
public Result doInvoke(Invocation invocation, final List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    // 所有服务提供者
    List<Invoker<T>> copyInvokers = invokers;
    checkInvokers(copyInvokers, invocation);
    String methodName = RpcUtils.getMethodName(invocation);
    // 重试次数
    int len = calculateInvokeTimes(methodName);
    // retry loop.
    RpcException le = null; // last exception.
    List<Invoker<T>> invoked = new ArrayList<Invoker<T>>(copyInvokers.size()); // invoked invokers.
    Set<String> providers = new HashSet<String>(len);
    for (int i = 0; i < len; i++) {
        // 重试时，进行重新选择，避免重试时invoker列表已发生变化
        // 注意: 如果列表发生了变化，那么invoked判断会失效，因为实例已经改变
        //Reselect before retry to avoid a change of candidate `invokers`.
        //NOTE: if `invokers` changed, then `invoked` also lose accuracy.
        if (i > 0) {
            // 实例销毁就抛出异常
            checkWhetherDestroyed();
            // 重新获取所有服务提供者
            copyInvokers = list(invocation);
            // check again
            // 重新检查一下
            checkInvokers(copyInvokers, invocation);
        }
        // 选择负载均衡策略
        Invoker<T> invoker = select(loadbalance, invocation, copyInvokers, invoked);
        invoked.add(invoker);
        RpcContext.getServiceContext().setInvokers((List) invoked);
        // 具体发起调用
        try {
            Result result = invokeWithContext(invoker, invocation);
            if (le != null && logger.isWarnEnabled()) {
                logger.warn("Although retry the method " + methodName
                        + " in the service " + getInterface().getName()
                        + " was successful by the provider " + invoker.getUrl().getAddress()
                        + ", but there have been failed providers " + providers
                        + " (" + providers.size() + "/" + copyInvokers.size()
                        + ") from the registry " + directory.getUrl().getAddress()
                        + " on the consumer " + NetUtils.getLocalHost()
                        + " using the dubbo version " + Version.getVersion() + ". Last error is: "
                        + le.getMessage(), le);
            }
            return result;
        } catch (RpcException e) {
            if (e.isBiz()) { // biz exception.
                throw e;
            }
            le = e;
        } catch (Throwable e) {
            le = new RpcException(e.getMessage(), e);
        } finally {
            providers.add(invoker.getUrl().getAddress());
        }
    }
    throw new RpcException(le.getCode(), "Failed to invoke the method "
            + methodName + " in the service " + getInterface().getName()
            + ". Tried " + len + " times of the providers " + providers
            + " (" + providers.size() + "/" + copyInvokers.size()
            + ") from the registry " + directory.getUrl().getAddress()
            + " on the consumer " + NetUtils.getLocalHost() + " using the dubbo version "
            + Version.getVersion() + ". Last error is: "
            + le.getMessage(), le.getCause() != null ? le.getCause() : le);
}
```

​	代码2从URL参数里获取设置的重试次数，如果用户没有设置重试次数，则取默认值，默认值是重试2次。这里需要注意的是，代码2将获取的配置重试次数又加1了，这说明总共调用次数=重试次数+1（1是正常调用），用户可以通过＜dubbo：reference retries="2"/＞设置重试次数。

## Failback Cluster策略源码分析

​	在Dubbo中，实现失败后定时重试的是FailbackClusterInvoker类，这里我们看看具体实现，主要看看doInvoke（）方法的代码：

```java
protected Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    Invoker<T> invoker = null;
    try {
        // 检查invoker合法性
        checkInvokers(invokers, invocation);
        // 复杂均衡选择调用
        invoker = select(loadbalance, invocation, invokers, null);
        return invokeWithContext(invoker, invocation);
    } catch (Throwable e) {
        logger.error("Failback to invoke method " + invocation.getMethodName() + ", wait for retry in background. Ignored exception: "
                + e.getMessage() + ", ", e);
        addFailed(loadbalance, invocation, invokers, invoker);
        return AsyncRpcResult.newDefaultAsyncResult(null, null, invocation); // ignore
    }
}
```

​	如果执行代码1和代码2抛出了异常，则执行代码3，即把当前请求上下文添加到定时器后，将一个空的RpcResult返回调用方。其中addFailed（）方法的代码如下：

```java
private void addFailed(LoadBalance loadbalance, Invocation invocation, List<Invoker<T>> invokers, Invoker<T> lastInvoker) {
    // 创建自定义定时器实例
    if (failTimer == null) {
        synchronized (this) {
            if (failTimer == null) {
                failTimer = new HashedWheelTimer(
                        new NamedThreadFactory("failback-cluster-timer", true),
                        1,
                        TimeUnit.SECONDS, 32, failbackTasks);
            }
        }
    }
    // 创建重试任务，并启动
    RetryTimerTask retryTimerTask = new RetryTimerTask(loadbalance, invocation, invokers, lastInvoker, retries, RETRY_FAILED_PERIOD);
    try {
        failTimer.newTimeout(retryTimerTask, RETRY_FAILED_PERIOD, TimeUnit.SECONDS);
    } catch (Throwable e) {
        logger.error("Failback background works error,invocation->" + invocation + ", exception: " + e.getMessage());
    }
}
```

​	代码4创建了一个自定义定时器，代码5创建了重试任务，并把任务添加到定时器，然后延迟5s执行定时任务，下面我们看看RetryTimerTask的任务内容：

```java
public void run(Timeout timeout) {
    try {
        Invoker<T> retryInvoker = select(loadbalance, invocation, invokers, Collections.singletonList(lastInvoker));
        lastInvoker = retryInvoker;
        invokeWithContext(retryInvoker, invocation);
    } catch (Throwable e) {
        logger.error("Failed retry to invoke method " + invocation.getMethodName() + ", waiting again.", e);
        // 不再重试
        if ((++retryTimes) >= retries) {
            logger.error("Failed retry times exceed threshold (" + retries + "), We have to abandon, invocation->" + invocation);
        } else {
            // 否则再次重试
            rePut(timeout);
        }
    }
}
 private void rePut(Timeout timeout) {
            if (timeout == null) {
                return;
            }

            Timer timer = timeout.timer();
            if (timer.isStop() || timeout.isCancelled()) {
                return;
            }

            timer.newTimeout(timeout.task(), tick, TimeUnit.SECONDS);
        }
```

## Forking Cluster策略源码分析

​	在Dubbo中，实现并行调用的是ForkingClusterInvoker类，这里我们看看具体实现，主要看看doInvoke（）方法的代码：

```java
public Result doInvoke(final Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    try {
        checkInvokers(invokers, invocation);
        final List<Invoker<T>> selected;
        // 获取配置参数
        final int forks = getUrl().getParameter(FORKS_KEY, DEFAULT_FORKS);
        final int timeout = getUrl().getParameter(TIMEOUT_KEY, DEFAULT_TIMEOUT);
        if (forks <= 0 || forks >= invokers.size()) {
            // 获取并行执行的invoker列表
            selected = invokers;
        } else {
            selected = new ArrayList<>(forks);
            while (selected.size() < forks) {
                // 在invoker列表(排除selected)后，如果没有选够，则存在重复循环问题
                Invoker<T> invoker = select(loadbalance, invocation, invokers, selected);
                if (!selected.contains(invoker)) {
                    //Avoid add the same invoker several times.
                    selected.add(invoker);
                }
            }
        }
        // 使用线程池让invoker列表的invoker并发执行
        RpcContext.getServiceContext().setInvokers((List) selected);
        final AtomicInteger count = new AtomicInteger();
        final BlockingQueue<Object> ref = new LinkedBlockingQueue<>();
        for (final Invoker<T> invoker : selected) {
            executor.execute(() -> {
                try {
                    Result result = invokeWithContext(invoker, invocation);
                    ref.offer(result);
                } catch (Throwable e) {
                    // 让所有的invoker都调用失败则记录错误
                    int value = count.incrementAndGet();
                    if (value >= selected.size()) {
                        ref.offer(e);
                    }
                }
            });
        }
        try {
            // 获取执行结果并返回
            Object ret = ref.poll(timeout, TimeUnit.MILLISECONDS);
            if (ret instanceof Throwable) {
                Throwable e = (Throwable) ret;
                throw new RpcException(e instanceof RpcException ? ((RpcException) e).getCode() : 0, "Failed to forking invoke provider " + selected + ", but no luck to perform the invocation. Last error is: " + e.getMessage(), e.getCause() != null ? e.getCause() : e);
            }
            return (Result) ret;
        } catch (InterruptedException e) {
            throw new RpcException("Failed to forking invoke provider " + selected + ", but no luck to perform the invocation. Last error is: " + e.getMessage(), e);
        }
    } finally {
        // clear attachments which is binding to current thread.
        RpcContext.getClientAttachment().clearAttachments();
    }
}
```

​	代码1获取并行执行的参数，其中forks为并行执行的invoker个数，timeout为调用线程获取执行结果的超时时间（默认为1000ms），这个超时时间与下面配置的接口的timeout超时时间一致：

![](https://pic.imgdb.cn/item/6113c9d85132923bf8fe27fc.jpg)

​	代码2根据设置的forks参数获取并行执行的invoker列表，如果forks参数值小于等于0或者大于服务提供者机器的个数，则设置forks为提供者机器个数。

## Broadcast Cluster策略源码分析

​	在Dubbo中，实现广播调用的是BroadcastClusterInvoker类，这里我们看看具体实现，主要看看doInvoke（）方法的代码：

```java
public Result doInvoke(final Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    checkInvokers(invokers, invocation);
    RpcContext.getServiceContext().setInvokers((List) invokers);
    RpcException exception = null;
    Result result = null;
    URL url = getUrl();
    // The value range of broadcast.fail.threshold must be 0～100.
    // 100 means that an exception will be thrown last, and 0 means that as long as an exception occurs, it will be thrown.
    // see https://github.com/apache/dubbo/pull/7174
    int broadcastFailPercent = url.getParameter(BROADCAST_FAIL_PERCENT_KEY, MAX_BROADCAST_FAIL_PERCENT);

    if (broadcastFailPercent < MIN_BROADCAST_FAIL_PERCENT || broadcastFailPercent > MAX_BROADCAST_FAIL_PERCENT) {
        logger.info(String.format("The value corresponding to the broadcast.fail.percent parameter must be between 0 and 100. " +
                "The current setting is %s, which is reset to 100.", broadcastFailPercent));
        broadcastFailPercent = MAX_BROADCAST_FAIL_PERCENT;
    }

    int failThresholdIndex = invokers.size() * broadcastFailPercent / MAX_BROADCAST_FAIL_PERCENT;
    int failIndex = 0;
    // 每个机器都进行调用，result是最后一个机器的结果
    for (Invoker<T> invoker : invokers) {
        try {
            result = invokeWithContext(invoker, invocation);
            if (null != result && result.hasException()) {
                Throwable resultException = result.getException();
                if (null != resultException) {
                    exception = getRpcException(result.getException());
                    logger.warn(exception.getMessage(), exception);
                    if (failIndex == failThresholdIndex) {
                        break;
                    } else {
                        failIndex++;
                    }
                }
            }
        } catch (Throwable e) {
            exception = getRpcException(e);
            logger.warn(exception.getMessage(), exception);
            if (failIndex == failThresholdIndex) {
                break;
            } else {
                failIndex++;
            }
        }
    }

    if (exception != null) {
        if (failIndex == failThresholdIndex) {
            logger.debug(
                    String.format("The number of BroadcastCluster call failures has reached the threshold %s", failThresholdIndex));
        } else {
            logger.debug(String.format("The number of BroadcastCluster call failures has not reached the threshold %s, fail size is %s",
                    failIndex));
        }
        throw exception;
    }

    return result;
}
```

## 如何基于扩展接口自定义集群容错策略

​	前面已经讲过，Dubbo本身提供了丰富的集群容错策略，但是如果你有定制化需求，可以根据Dubbo提供的扩展接口Cluster进行定制。

​	为了自定义扩展实现，首先需要实现Cluster接口：

![](https://pic.imgdb.cn/item/6113cae85132923bf8021b12.jpg)

​	然后，需要集成AbstractClusterInvoker类创建自己的ClusterInvoker类：

![](https://pic.imgdb.cn/item/6113caf85132923bf8025265.jpg)

​	通过上面的代码可知，doInvoke方法需要重写，在该方法内用户就可以实现自己的集群容错策略。

​	然后，在org.apache.dubbo.rpc.cluster.Cluster目录下创建文件，并在文件里添加myCluster=org.apache.dubbo.demo.cluster.MyCluster，如图6.2所示。

![](https://pic.imgdb.cn/item/6113cb355132923bf8031aff.jpg)

​	最后，使用下面的方法将集群容错模式切换为myCluster：

![](https://pic.imgdb.cn/item/6113cb475132923bf8035784.jpg)

* Random LoadBalance：随机策略。按照概率设置权重，比较均匀，并且可以动态调节提供者的权重。
* RoundRobin LoadBalance：轮循策略。轮循，按公约后的权重设置轮循比率。会存在执行比较慢的服务提供者堆积请求的情况，比如一个机器执行得非常慢，但是机器没有宕机（如果宕机了，那么当前机器会从ZooKeeper的服务列表中删除），当很多新的请求到达该机器后，由于之前的请求还没处理完，会导致新的请求被堆积，久而久之，消费者调用这台机器上的所有请求都会被阻塞。
*  LeastActive LoadBalance：最少活跃调用数。如果每个提供者的活跃数相同，则随机选择一个。在每个服务提供者里维护着一个活跃数计数器，用来记录当前同时处理请求的个数，也就是并发处理任务的个数。这个值越小，说明当前服务提供者处理的速度越快或者当前机器的负载比较低，所以路由选择时就选择该活跃度最底的机器。如果一个服务提供者处理速度很慢，由于堆积，那么同时处理的请求就比较多，也就是说活跃调用数目较大（活跃度较高），这时，处理速度慢的提供者将收到更少的请求。

*  ConsistentHash LoadBalance一致性Hash策略。一致性Hash，可以保证相同参数的请求总是发到同一提供者，当某一台提供者机器宕机时，原本发往该提供者的请求，将基于虚拟节点平摊给其他提供者，这样就不会引起剧烈变动。



​	如上所述，Dubbo提供了丰富的负载均衡策略，但是如果你有定制化需求，可以基于Dubbo提供的扩展接口LoadBalance进行定制。

​	AbstractLoadBalance实现了LoadBalance接口，上面的各种负载均衡策略实际上就是继承了AbstractLoadBalance方法，但重写了其doSelect（）方法，所以下面我们重点看看每种负载均衡策略的doSelect（）方法。

​	我们先看看从消费端发起请求到处理负载均衡的时序图，如图6.3所示。

![](https://pic.imgdb.cn/item/6113d11b5132923bf8158eb9.jpg)

​	从图6.3可以看到，消费端发起远程调用后会先经过步骤4进行服务降级检查，如果发现没有设置降级策略，则会执行步骤5进入集群容错invoker，其内部会先执行步骤6，以便从RegistryDirectory里获取经过服务路由规则过滤后的服务提供者的invokers列表。然后，执行步骤8～步骤11，以便根据具体负载均衡策略从invoker列表中选择一个invoker，最后使用所选的invoker，执行步骤12以发起远程调用。

## Random LoadBalance策略源码分析

```java
protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
    // Number of invokers
    int length = invokers.size();

    if (!needWeightLoadBalance(invokers,invocation)){
        return invokers.get(ThreadLocalRandom.current().nextInt(length));
    }

    // Every invoker has the same weight?
    boolean sameWeight = true;
    // the maxWeight of every invokers, the minWeight = 0 or the maxWeight of the last invoker
    int[] weights = new int[length];
    // The sum of weights
    int totalWeight = 0;
    for (int i = 0; i < length; i++) {
        int weight = getWeight(invokers.get(i), invocation);
        // Sum
        totalWeight += weight;
        // save for later use
        weights[i] = totalWeight;
        if (sameWeight && totalWeight != weight * (i + 1)) {
            sameWeight = false;
        }
    }
    if (totalWeight > 0 && !sameWeight) {
        // If (not every invoker has the same weight & at least one invoker's weight>0), select randomly based on totalWeight.
        int offset = ThreadLocalRandom.current().nextInt(totalWeight);
        // Return a invoker based on the random value.
        for (int i = 0; i < length; i++) {
            if (offset < weights[i]) {
                return invokers.get(i);
            }
        }
    }
    // If all invokers have the same weight value or totalWeight=0, return evenly.
    return invokers.get(ThreadLocalRandom.current().nextInt(length));
}
```

​	需要注意的是，这里没有使用Random而是使用了ThreadLocalRandom，这是出于性能上的考虑，因为Random在高并发下会导致大量线程竞争同一个原子变量，导致大量线程原地自旋，从而浪费CPU资源（参见笔者编写的《Java并发编程之美》一书）。

​	在代码7中，如果所有服务提供者权重不一样，那么在正常情况下应该选择权重最大的提供者来提供服务，但是Dubbo还考虑到另外一个因素，就是服务预热时间。如果服务提供者A的权重比服务提供者B的权重大，但服务提供者A是刚启动的，而服务提供者B已经服务了一些时间，则这时候Dubbo会选择服务提供者B而不是服务提供者A来进行调用。为此，我们可以看看计算权重的getWeight（）方法的代码：

```java
int getWeight(Invoker<?> invoker, Invocation invocation) {
    int weight;
    URL url = invoker.getUrl();
    // Multiple registry scenario, load balance among multiple registries.
    if (REGISTRY_SERVICE_REFERENCE_PATH.equals(url.getServiceInterface())) {
        weight = url.getParameter(REGISTRY_KEY + "." + WEIGHT_KEY, DEFAULT_WEIGHT);
    } else {
        weight = url.getMethodParameter(invocation.getMethodName(), WEIGHT_KEY, DEFAULT_WEIGHT);
        if (weight > 0) {
            long timestamp = invoker.getUrl().getParameter(TIMESTAMP_KEY, 0L);
            if (timestamp > 0L) {
                long uptime = System.currentTimeMillis() - timestamp;
                if (uptime < 0) {
                    return 1;
                }
                int warmup = invoker.getUrl().getParameter(WARMUP_KEY, DEFAULT_WARMUP);
                if (uptime > 0 && uptime < warmup) {
                    weight = calculateWarmupWeight((int)uptime, warmup, weight);
                }
            }
        }
    }
    return Math.max(weight, 0);
}
```

​	代码6.1获取用户对该服务提供者设置的权重，默认情况下权重都是100。

​	代码6.2获取该服务提供者发布服务时的时间timestamp。

​	代码6.3计算该服务已经发布了多少时间。

​	代码6.4获取用户设置的该服务的预热时间，默认是10分钟。

​	在代码6.5中，如果该服务提供者还没过预热期，则让该服务提供者的预热时间参与计算权重calculateWarmupWeight：

```java
static int calculateWarmupWeight(int uptime, int warmup, int weight) {
    int ww = (int) ( uptime / ((float) warmup / weight));
    return ww < 1 ? 1 : (Math.min(ww, weight));
}
```

​	上面代码中的int ww=（int）（（float）uptime/（（float）warmup/（float）weight））；等价于int ww=（int）（（float）uptime/（float）warmup*（float）weight）；，也就是说如果当前服务提供者还没过预热期，则用户设置的权重将通过（float）uptime/（float）warmup打折扣。如果一个服务提供者设置的权重很大，但是还没过预热时间，那么重新计算出来的权重就比实际的小。

​	如果服务提供者已经过了预热期，则这时就会按照用户设置的权重的大小来选择使用哪个服务提供者。

## RoundRobin LoadBalance策略源码分析

​	在Dubbo中，实现轮询选择策略的是RoundRobinLoadBalance类，这里我们看看具体实现，主要看看doSelect（）方法的代码：

```java
protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
    // 获取调用方法的key
    String key = invokers.get(0).getUrl().getServiceKey() + "." + invocation.getMethodName();
    // 获取该调用方法对应的每个服务提供者的WeightedRoundRobin组成的Map
    ConcurrentMap<String, WeightedRoundRobin> map = methodWeightMap.computeIfAbsent(key, k -> new ConcurrentHashMap<>());
    // 遍历所有提供者，计算总权重和权重最大的提供者
    int totalWeight = 0;
    long maxCurrent = Long.MIN_VALUE;
    long now = System.currentTimeMillis();
    Invoker<T> selectedInvoker = null;
    WeightedRoundRobin selectedWRR = null;
    for (Invoker<T> invoker : invokers) {
        String identifyString = invoker.getUrl().toIdentityString();
        int weight = getWeight(invoker, invocation);
        WeightedRoundRobin weightedRoundRobin = map.computeIfAbsent(identifyString, k -> {
            WeightedRoundRobin wrr = new WeightedRoundRobin();
            wrr.setWeight(weight);
            return wrr;
        });

        if (weight != weightedRoundRobin.getWeight()) {
            // 权重变化
            //weight changed
            weightedRoundRobin.setWeight(weight);
        }
        long cur = weightedRoundRobin.increaseCurrent();
        weightedRoundRobin.setLastUpdate(now);
        if (cur > maxCurrent) {
            maxCurrent = cur;
            selectedInvoker = invoker;
            selectedWRR = weightedRoundRobin;
        }
        totalWeight += weight;
    }
    if (invokers.size() != map.size()) {
        map.entrySet().removeIf(item -> now - item.getValue().getLastUpdate() > RECYCLE_PERIOD);
    }
    if (selectedInvoker != null) {
        selectedWRR.sel(totalWeight);
        return selectedInvoker;
    }
    // should not happen here
    return invokers.get(0);
}
```

​	RECYCLE_PERIOD是清理周期，如果服务提供者耗时RECYCLE_PERIOD还没有更新自己的WeightedRoundRobin对象，则会被自动回收；updateLock用来确保更新methodWeightMap的线程安全，这里使用了原子变量的CAS操作，减少了因为使用锁带来的开销。

​	com.books.dubbo.demo.api.GreetingService接口的String sayHello（String name）方法，并且每个机器设置的提供者的权重分别为1、2、3，下面我们看看如何实现轮询调用。

![](https://pic.imgdb.cn/item/6113e2385132923bf84b3420.jpg)

​	在消费端第一次调用服务时，服务提供者A、B、C对应的WeightedRoundRobin对象里的weight和current如图6.4中的第二列所示，代码3执行完毕后，对应的weight和current值如图6.4中的第三列所示，这时最大权重的机器是C，所以轮询算法选择了机器C来提供服务。

​	消费端第二次调用服务，如图6.5所示。

![](https://pic.imgdb.cn/item/6113e28a5132923bf84c2f05.jpg)

​	在消费端第二次调用服务时，服务提供者A、B、C对应的WeightedRoundRobin对象里的weight和current如图6.5中的第二列所示，代码3执行完毕后，对应的weight和current值如图6.5中的第三列所示，这时最大权重的机器是B，所以轮询算法选择了机器B来提供服务。

​	消费端第三次调用服务，如图6.6所示。

![](https://pic.imgdb.cn/item/6113e2ae5132923bf84c9a84.jpg)

​	在消费端第三次调用服务时，服务提供者A、B、C对应的WeightedRoundRobin对象里的weight和current如图6.6中的第二列所示，代码3执行完毕后，对应的weight和current值如图6.6中的第三列所示，这时最大权重的机器是A，所以轮询算法选择了机器A来提供服务。

​	消费端第四次调用服务，如图6.7所示。

![](https://pic.imgdb.cn/item/6113e2c65132923bf84ce116.jpg)

​	在消费端第四次调用服务时，服务提供者A、B、C对应的WeightedRoundRobin对象里的weight和current如图6.7中的第二列所示，代码3执行完毕后，对应的weight和current值如图6.7中的第三列所示，这时最大权重的机器是C，所以轮询算法选择了机器C来提供服务。

​	消费端第五次调用服务，如图6.8所示。

![](https://pic.imgdb.cn/item/6113e2d85132923bf84d1c68.jpg)

​	在消费端第五次调用服务时，服务提供者A、B、C对应的WeightedRoundRobin对象里的weight和current如图6.8中的第二列所示，代码3执行完毕后，对应的weight和current值如图6.8中的第三列所示，这时最大权重的机器是B，所以轮询算法选择了机器B来提供服务。

​	消费端第六次调用服务，如图6.9所示。

![](https://pic.imgdb.cn/item/6113e2f05132923bf84d617a.jpg)

​	在消费端第六次调用服务时，服务提供者A、B、C对应的WeightedRoundRobin对象里的weight和current如图6.9中的第二列所示，代码3执行完毕后，对应的weight和current值如图6.9中的第三列所示，这时最大权重的机器是C，所以轮询算法选择了机器C来提供服务。下一次再次调用时就又回到了第一次调用的情况，也就是说6次调用完成一次轮询。

​	这里的RR算法并不是严格意义上的RR，因为选择提供者时还参考了其设置的权重，如上所述，调用6次服务并不是严格按照A—B—C—A—B—C这样的次序提供服务的，这是因为我们设置的每台机器的权重不一样。如果大家把A、B、C的权重设置成一样的，那么就会发现将以顺序方式选中机器A、B、C作为服务提供者。

## LeastActive LoadBalance策略源码分析

​	在Dubbo中，实现最少活跃调用策略的是LeastActiveLoadBalance类，这里我们看看具体实现，主要看看doSelect（）方法的代码：

```java
protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
    // Number of invokers
    int length = invokers.size();
    // The least active value of all invokers
    int leastActive = -1;
    // The number of invokers having the same least active value (leastActive)
    int leastCount = 0;
    // The index of invokers having the same least active value (leastActive)
    int[] leastIndexes = new int[length];
    // the weight of every invokers
    int[] weights = new int[length];
    // The sum of the warmup weights of all the least active invokers
    int totalWeight = 0;
    // The weight of the first least active invoker
    int firstWeight = 0;
    // Every least active invoker has the same weight value?
    boolean sameWeight = true;


    // Filter out all the least active invokers
    for (int i = 0; i < length; i++) {
        Invoker<T> invoker = invokers.get(i);
        // Get the active number of the invoker
        int active = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName()).getActive();
        // Get the weight of the invoker's configuration. The default value is 100.
        int afterWarmup = getWeight(invoker, invocation);
        // save for later use
        weights[i] = afterWarmup;
        // If it is the first invoker or the active number of the invoker is less than the current least active number
        if (leastActive == -1 || active < leastActive) {
            // Reset the active number of the current invoker to the least active number
            leastActive = active;
            // Reset the number of least active invokers
            leastCount = 1;
            // Put the first least active invoker first in leastIndexes
            leastIndexes[0] = i;
            // Reset totalWeight
            totalWeight = afterWarmup;
            // Record the weight the first least active invoker
            firstWeight = afterWarmup;
            // Each invoke has the same weight (only one invoker here)
            sameWeight = true;
            // If current invoker's active value equals with leaseActive, then accumulating.
        } else if (active == leastActive) {
            // Record the index of the least active invoker in leastIndexes order
            leastIndexes[leastCount++] = i;
            // Accumulate the total weight of the least active invoker
            totalWeight += afterWarmup;
            // If every invoker has the same weight?
            if (sameWeight && afterWarmup != firstWeight) {
                sameWeight = false;
            }
        }
    }
    // Choose an invoker from all the least active invokers
    if (leastCount == 1) {
        // If we got exactly one invoker having the least active value, return this invoker directly.
        return invokers.get(leastIndexes[0]);
    }
    if (!sameWeight && totalWeight > 0) {
        // If (not every invoker has the same weight & at least one invoker's weight>0), select randomly based on 
        // totalWeight.
        int offsetWeight = ThreadLocalRandom.current().nextInt(totalWeight);
        // Return a invoker based on the random value.
        for (int i = 0; i < leastCount; i++) {
            int leastIndex = leastIndexes[i];
            offsetWeight -= weights[leastIndex];
            if (offsetWeight < 0) {
                return invokers.get(leastIndex);
            }
        }
    }
    // If all invokers have the same weight value or totalWeight=0, return evenly.
    return invokers.get(leastIndexes[ThreadLocalRandom.current().nextInt(leastCount)]);
}
```

​	如果最小调用次数的invoker有多个并且权重一样，则随机选择一个。

## ConsistentHash LoadBalance策略源码分析

### 一致性Hash负载均衡策略原理

​	分布式系统中，负载均衡的问题可以使用Hash算法让固定的一部分请求落到同一台服务器上，这样每台服务器就会固定处理一部分请求（并维护这些请求的信息），从而起到负载均衡的作用。

​	但是普通的余数Hash（用户ID）算法伸缩性很差，当新增或者下线服务器机器时，用户ID与服务器的映射关系会大量失效。一致性Hash则利用Hash环对其进行了改进。

​	为了能直观地理解一致性Hash原理，这里结合一个简单的例子来讲解。假设有4台服务器，地址为IP1、IP2、IP3、IP4。

* 一致性Hash，首先计算4个IP地址对应的Hash值：Hash（IP1）、Hash（IP2）、Hash（IP3）、Hash（IP4），计算出来的Hash值是0～最大正整数之间的一个值，这4个值在一致性Hash环上的呈现如图6.10所示。

![](https://pic.imgdb.cn/item/6113e5575132923bf8548c43.jpg)

* 在Hash环上按顺时针从整数0开始，一直到最大正整数，我们根据4个IP计算的Hash值肯定会落到这个Hash环上的某一个点，至此我们把服务器的4个IP映射到了一致性Hash环上。
* 当用户在客户端进行请求时，首先根据Hash（用户ID）计算路由规则（Hash值），然后看Hash值落到了Hash环的哪个地方，根据Hash值在Hash环上的位置顺时针找距离最近的IP作为路由IP，如图6.11所示。
* 通过图6.11可知，user1、user2的请求会落到服务器IP2进行处理，user3的请求会落到服务器IP3进行处理，user4的请求会落到服务器IP4进行处理，user5、user6的请求会落到服务器IP1进行处理。

![](https://pic.imgdb.cn/item/6113e5805132923bf854fd71.jpg)

​	下面考虑一下当IP2的服务器宕机时会出现什么情况。

​	当IP2的服务器宕机时，一致性Hash环大致如图6.12所示。

![](https://pic.imgdb.cn/item/6113e5965132923bf8553a77.jpg)

​	根据顺时针规则可知，user1、user2的请求会被服务器IP3进行处理，而其他用户的请求所对应的处理服务器不变，也就是只有之前被IP2处理的一部分用户的映射关系被破坏了，并且其负责处理的请求被顺时针的下一个节点处理。

​	下面考虑一下当有新增机器加入时会出现什么情况。

​	在新增一个IP5的服务器后，一致性Hash环大致如图6.13所示。

![](https://pic.imgdb.cn/item/6113e5ab5132923bf8557806.jpg)

​	根据顺时针规则可知，之前user5的请求应该被IP1服务器处理，现在被新增的IP5服务器处理，其他用户的请求处理服务器不变，也就是说距新增的服务器顺时针最近的服务器的一部分请求，会被新增的服务器所替代。

**一致性Hash的特性**

* 单调性（Monotonicity）：单调性是指如果已经有一些请求通过Hash分派到了相应的服务器进行处理，当又有新的服务器加入到系统中时，应保证原有的请求可以被映射到原有的或者新增的服务器上，而不会被映射到原来的其他服务器上。这一点通过上面新增的服务器IP5就可以证明，在新增了IP5后，原来被IP1处理的user6现在还是被IP1处理，原来被IP1处理的user5现在被新增的IP5处理。
* · 分散性（Spread）：在分布式环境中，当客户端请求时可能不知道所有服务器的存在，可能只知道其中一部分服务器，从客户端看来，它看到的部分服务器会形成一个完整的Hash环。如果多个客户端都把部分服务器作为一个完整Hash环，那么可能会导致同一个用户的请求被路由到不同的服务器进行处理。这种情况显然是应该避免的，因为它不能保证同一个用户的请求落到同一台服务器。所谓分散性是指上述情况发生的严重程度。
* 平衡性（Balance）：平衡性也就是指负载均衡，是指客户端Hash后的请求应该能够分散到不同的服务器上。一致性Hash可以做到每个服务器都进行处理请求，但是不能保证每个服务器处理的请求的数量大致相同，如图6.14所示。

![](https://pic.imgdb.cn/item/6113e5fe5132923bf8565f97.jpg)

​	服务器IP1、IP2、IP3经过Hash后落到了一致性Hash环上，从图中Hash值的分布可知，IP1会负责处理大概80%的请求，而IP2和IP3则只会负责处理大概20%的请求，虽然三个机器都在处理请求，但明显每个机器的负载不均衡，这样称为一致性Hash的倾斜，虚拟节点的出现就是为了解决这个问题。

**虚拟节点**

​	当服务器节点比较少的时候会出现上面所说的一致性Hash倾斜问题，一种解决方法是多加机器，但加机器是有成本的，那么就加虚拟节点，比如上面三台机器，每台机器引入1个虚拟节点后的一致性Hash环如图6.15所示。

![](https://pic.imgdb.cn/item/6113e62b5132923bf856e080.jpg)

​	图中的IP1-1是IP1的虚拟节点，IP2-1是IP2的虚拟节点，IP3-1是IP3的虚拟节点。

​	当物理机器数目为M，虚拟节点为N的时候，实际Hash环上节点个数为M*（N+1）。

**均匀一致性Hash**

​	上一小节我们使用虚拟节点后的图看起来比较均衡，但如果生成虚拟节点的算法不够好，则很可能会得到如图6.16所示的环。

![](https://pic.imgdb.cn/item/6113e6575132923bf8575810.jpg)

​	每个服务节点引入1个虚拟节点后，相比没有引入前均衡性有所改善，但是并不均衡。

​	均衡的一致性Hash应该如图6.17所示。

![](https://pic.imgdb.cn/item/6113e66d5132923bf85794e8.jpg)

​	均匀一致性Hash的目标是，如果服务器有N台，客户端的Hash值有M个，那么每台服务器应该处理大概M/N个用户的请求，也就是说每台服务器负载尽量均衡。

### 源码分析

​	在Dubbo中，实现一致性Hash策略的是ConsistentHashLoadBalance类，这里我们看看具体实现，主要看看doSelect（）方法的代码：

```java
protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
    String methodName = RpcUtils.getMethodName(invocation);
    String key = invokers.get(0).getUrl().getServiceKey() + "." + methodName;
    // using the hashcode of list to compute the hash only pay attention to the elements in the list
    int invokersHashCode = invokers.hashCode();
    ConsistentHashSelector<T> selector = (ConsistentHashSelector<T>) selectors.get(key);
    if (selector == null || selector.identityHashCode != invokersHashCode) {
        selectors.put(key, new ConsistentHashSelector<T>(invokers, methodName, invokersHashCode));
        selector = (ConsistentHashSelector<T>) selectors.get(key);
    }
    return selector.select(invocation);
}
```

​	在上面的代码中，一致性Hash策略主要是ConsistentHashSelector类实现，其构造函数如下：

```java
ConsistentHashSelector(List<Invoker<T>> invokers, String methodName, int identityHashCode) {
    this.virtualInvokers = new TreeMap<Long, Invoker<T>>();
    this.identityHashCode = identityHashCode;
    URL url = invokers.get(0).getUrl();
    // 设置虚拟节点的个数 默认160
    this.replicaNumber = url.getMethodParameter(methodName, HASH_NODES, 160);
    String[] index = COMMA_SPLIT_PATTERN.split(url.getMethodParameter(methodName, HASH_ARGUMENTS, "0"));
    argumentIndex = new int[index.length];
    for (int i = 0; i < index.length; i++) {
        argumentIndex[i] = Integer.parseInt(index[i]);
    }
    for (Invoker<T> invoker : invokers) {
        String address = invoker.getUrl().getAddress();
        for (int i = 0; i < replicaNumber / 4; i++) {
            byte[] digest = Bytes.getMD5(address + i);
            for (int h = 0; h < 4; h++) {
                long m = hash(digest, h);
                virtualInvokers.put(m, invoker);
            }
        }
    }
}
```

## 如何基于扩展接口自定义负载均衡策略

​	自定义扩展实现，首先需要实现LoadBalance接口，由于Dubbo本身提供了一个抽象类AbstractLoadBalance，所以我们可以直接继承该类：

![](https://pic.imgdb.cn/item/6113e84b5132923bf85c9446.jpg)

​	在上面的代码中，MyLoadBalance实现了LoadBalance的doSelect接口。然后，在org.apache.dubbo.rpc.cluster.LoadBalance目录下创建文件，并在文件里添加myLoadBalance=com.books.dubbo.demo.loadbalance.MyLoadBalance，如图6.18所示。

![](https://pic.imgdb.cn/item/6113e87c5132923bf85d0705.jpg)

# Dubbo线程模型与线程池策略

​	Dubbo默认的底层网络通信使用的是Netty，服务提供方NettyServer使用两级线程池，其中EventLoopGroup（boss）主要用来接收客户端的链接请求，并把完成TCP三次握手的连接分发给EventLoopGroup（worker）来处理，我们把boss和worker线程组称为I/O线程。

​	如果服务提供方的逻辑处理能迅速完成，并且不会发起新的I/O请求，那么直接在I/O线程上处理会更快，因为这样减少了线程池调度与上下文切换的开销。

​	但如果处理逻辑较慢，或者需要发起新的I/O请求，比如需要查询数据库，则I/O线程必须派发请求到新的线程池进行处理，否则I/O线程会被阻塞，导致不能接收其他请求。

​	根据请求的消息类是被I/O线程处理还是被业务线程池处理，Dubbo提供了下面几种线程模型。

* all（AllDispatcher类）：所有消息都派发到业务线程池，这些消息包括请求、响应、连接事件、断开事件、心跳事件等，如图7.1所示。

![](https://pic.imgdb.cn/item/61150b015132923bf80ffbc4.jpg)



* direct（DirectDispatcher类）：所有消息都不派发到业务线程池，全部在IO线程上直接执行，如图7.2所示。

![](https://pic.imgdb.cn/item/61150b1c5132923bf810304e.jpg)

* message（MessageOnlyDispatcher类）：只有请求响应消息派发到业务线程池，其他消息如连接事件、断开事件、心跳事件等，直接在I/O线程上执行，如图7.3所示。

![](https://pic.imgdb.cn/item/61150b345132923bf8105ee3.jpg)

* execution（ExecutionDispatcher类）：只把请求类消息派发到业务线程池处理，但是响应、连接事件、断开事件、心跳事件等消息直接在I/O线程上执行，如图7.4所示。

![](https://pic.imgdb.cn/item/61150b455132923bf810806e.jpg)

* connection（ConnectionOrderedDispatcher类）：在I/O线程上将连接事件、断开事件放入队列，有序地逐个执行，其他消息派发到业务线程池处理，如图7.5所示。

![](https://pic.imgdb.cn/item/61150c425132923bf8127303.jpg)

​	在Dubbo中，线程模型的扩展接口为Dispatcher，其提供的上述扩展实现都实现了该接口，其中all模型是默认的线程模型。

## AllDispatcher源码剖析

```java
public class AllDispatcher implements Dispatcher {
	// 线程模型名称
    public static final String NAME = "all";

    // 扩展接口具体实现
    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        return new AllChannelHandler(handler, url);
    }

}
```

```java
public class AllChannelHandler extends WrappedChannelHandler {

    public AllChannelHandler(ChannelHandler handler, URL url) {
        super(handler, url);
    }

    // 连接完成事件，交给业务线程池处理
    @Override
    public void connected(Channel channel) throws RemotingException {
        ExecutorService executor = getExecutorService();
        try {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CONNECTED));
        } catch (Throwable t) {
            throw new ExecutionException("connect event", channel, getClass() + " error when process connected event .", t);
        }
    }
	
    // 断开事件，交给业务线程池处理
    @Override
    public void disconnected(Channel channel) throws RemotingException {
        ExecutorService executor = getExecutorService();
        try {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.DISCONNECTED));
        } catch (Throwable t) {
            throw new ExecutionException("disconnect event", channel, getClass() + " error when process disconnected event .", t);
        }
    }

    // 请求响应事件，业务线程池处理
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        try {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
            if(message instanceof Request && t instanceof RejectedExecutionException){
                sendFeedback(channel, (Request) message, t);
                return;
            }
            throw new ExecutionException(message, channel, getClass() + " error when process received event .", t);
        }
    }
	
    // 异常处理事件
    @Override
    public void caught(Channel channel, Throwable exception) throws RemotingException {
        ExecutorService executor = getExecutorService();
        try {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CAUGHT, exception));
        } catch (Throwable t) {
            throw new ExecutionException("caught event", channel, getClass() + " error when process caught event .", t);
        }
    }
}
```

## DirectDispatcher源码剖析

​	DirectDispatcher对应的是direct线程模型的实现，其代码如下：

```java
public class DirectDispatcher implements Dispatcher {

    public static final String NAME = "direct";

    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        return new DirectChannelHandler(handler, url);
    }

}
```

​	

```java
public class DirectChannelHandler extends WrappedChannelHandler {

    public DirectChannelHandler(ChannelHandler handler, URL url) {
        super(handler, url);
    }

    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        if (executor instanceof ThreadlessExecutor) {
            try {
                executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
            } catch (Throwable t) {
                throw new ExecutionException(message, channel, getClass() + " error when process received event .", t);
            }
        } else {
            handler.received(channel, message);
        }
    }

}
```

## MessageOnlyDispatcher源码剖析

​	MessageOnlyDispatcher对应是message线程模型的实现，其代码如下：

```java
public class MessageOnlyDispatcher implements Dispatcher {

    public static final String NAME = "message";

    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        return new MessageOnlyChannelHandler(handler, url);
    }

}
```

```java
public class MessageOnlyChannelHandler extends WrappedChannelHandler {

    public MessageOnlyChannelHandler(ChannelHandler handler, URL url) {
        super(handler, url);
    }

    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        try {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
            if(message instanceof Request && t instanceof RejectedExecutionException){
                sendFeedback(channel, (Request) message, t);
                return;
            }
            throw new ExecutionException(message, channel, getClass() + " error when process received event .", t);
        }
    }

}
```

## ExecutionDispatcher源码剖析

```java
public class ExecutionDispatcher implements Dispatcher {

    public static final String NAME = "execution";

    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        return new ExecutionChannelHandler(handler, url);
    }

}
```

```java
	public class ExecutionChannelHandler extends WrappedChannelHandler {

    public ExecutionChannelHandler(ChannelHandler handler, URL url) {
        super(handler, url);
    }

    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);

        if (message instanceof Request) {
            try {
                executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
            } catch (Throwable t) {
                // FIXME: when the thread pool is full, SERVER_THREADPOOL_EXHAUSTED_ERROR cannot return properly,
                // therefore the consumer side has to wait until gets timeout. This is a temporary solution to prevent
                // this scenario from happening, but a better solution should be considered later.
                if (t instanceof RejectedExecutionException) {
                    sendFeedback(channel, (Request) message, t);
                }
                throw new ExecutionException(message, channel, getClass() + " error when process received event.", t);
            }
        } else if (executor instanceof ThreadlessExecutor) {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } else {
            handler.received(channel, message);
        }
    }
}
```

​	通过上面的代码可知，只把请求类消息派发到业务线程池处理，但是响应、连接事件、断开事件、心跳事件等消息则直接在I/O线程上执行。这里与message模型不同之处在于，响应类型的事件也是在I/O线程上执行的。

## ConnectionOrderedDispatcher源码剖析

```java
public class ConnectionOrderedDispatcher implements Dispatcher {

    public static final String NAME = "connection";

    @Override
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {
        return new ConnectionOrderedChannelHandler(handler, url);
    }

}
```

```java
public class ConnectionOrderedChannelHandler extends WrappedChannelHandler {

    protected final ThreadPoolExecutor connectionExecutor;
    private final int queuewarninglimit;

    public ConnectionOrderedChannelHandler(ChannelHandler handler, URL url) {
        super(handler, url);
        // 创建线程池
        String threadName = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
        connectionExecutor = new ThreadPoolExecutor(1, 1,
                0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>(url.getPositiveParameter(CONNECT_QUEUE_CAPACITY, Integer.MAX_VALUE)),
                new NamedThreadFactory(threadName, true),
                new AbortPolicyWithReport(threadName, url)
        );  // FIXME There's no place to release connectionExecutor!
        queuewarninglimit = url.getParameter(CONNECT_QUEUE_WARNING_SIZE, DEFAULT_CONNECT_QUEUE_WARNING_SIZE);
    }
	
    // 链接建立事件
    @Override
    public void connected(Channel channel) throws RemotingException {
        try {
            checkQueueLength();
            connectionExecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CONNECTED));
        } catch (Throwable t) {
            throw new ExecutionException("connect event", channel, getClass() + " error when process connected event .", t);
        }
    }

    @Override
    public void disconnected(Channel channel) throws RemotingException {
        try {
            checkQueueLength();
            connectionExecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.DISCONNECTED));
        } catch (Throwable t) {
            throw new ExecutionException("disconnected event", channel, getClass() + " error when process disconnected event .", t);
        }
    }
	
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        try {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
            if (message instanceof Request && t instanceof RejectedExecutionException) {
                sendFeedback(channel, (Request) message, t);
                return;
            }
            throw new ExecutionException(message, channel, getClass() + " error when process received event .", t);
        }
    }

    @Override
    public void caught(Channel channel, Throwable exception) throws RemotingException {
        ExecutorService executor = getExecutorService();
        try {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CAUGHT, exception));
        } catch (Throwable t) {
            throw new ExecutionException("caught event", channel, getClass() + " error when process caught event .", t);
        }
    }

    private void checkQueueLength() {
        if (connectionExecutor.getQueue().size() > queuewarninglimit) {
            logger.warn(new IllegalThreadStateException("connectionordered channel handler `queue size: " + connectionExecutor.getQueue().size() + " exceed the warning limit number :" + queuewarninglimit));
        }
    }
}
```

​	在代码1中，在构造函数内创建一个只含有一个线程的线程池和一个有限元素的队列，如果设置的参数connect.queue.capacity大于0，则设置为线程池队列容量，否则线程池队列容量为整数最大值，这个线程池用来实现把链接建立和链接断开事件进行顺序化处理。

​	代码2、代码3分别处理链接建立、链接断开事件，可以先使用代码9检查线程池队列的元素个数，个数超过阈值则打印日志，然后把事件放入线程池队列，并使用单线程进行处理。由于是单线程处理，所以其实是“多生产-单消费”模型，实现了把链接建立、链接断开事件的处理变为顺序化处理。

​	代码4、代码5则处理请求事件和异常事件，这里是直接交给了线程池（这个线程池不是connectionExecutor）进行异步处理。

## 线程模型的确定时机

​	前面介绍了Dubbo提供的线程模型，下面我们介绍的是何时确定使用哪种线程模型。在前面的3.1节提到，服务提供方会启动NettyServer来监听消费方的链接，其构造函数如下：

```java
public NettyServer(URL url, ChannelHandler handler) throws RemotingException {
    super(ExecutorUtil.setThreadName(url, SERVER_THREAD_POOL_NAME), ChannelHandlers.wrap(handler, url));
}
```

​	这里我们主要看看ChannelHandlers.wrap（handler，ExecutorUtil.setThreadName（url，SERVER_THREAD_POOL_NAME））这个代码，该代码加载了具体的线程模型，这是通过ChannelHandlers的wrapInternal方法完成的：

```java
protected ChannelHandler wrapInternal(ChannelHandler handler, URL url) {
    return new MultiMessageHandler(new HeartbeatHandler(ExtensionLoader.getExtensionLoader(Dispatcher.class)
            .getAdaptiveExtension().dispatch(handler, url)));
}
```

​	这里根据URL里的线程模型来选择具体的Dispatcher实现类。在此，我们再提一下Dubbo提供的Dispatcher实现类，其默认的实现类是all：

![](https://pic.imgdb.cn/item/6115103f5132923bf81a6ad8.jpg)

##  如何基于扩展接口自定义线程模型

​	首先，我们需要实现扩展接口Dispatcher，其代码如下：

![](https://pic.imgdb.cn/item/611510585132923bf81a9b7b.jpg)

​	然后，我们需要实现自己的Handler，这里的实例为MyChannelHandler，我们需要在项目的src/main/resources目录下创建目录/META-INF/dubbo，并在该目录下创建文件org.apache.dubbo.remoting.Dispatcher，文件内容为mydispatcher=com.books.dubbo.demo.provider.mydispatcher.MyDispatcher。

​	最后，在服务提供者启动时将线程模型设置为mydispatcher即可：

![](https://pic.imgdb.cn/item/611510745132923bf81ad756.jpg)

## Dubbo的线程池策略

​	我们在上面讲解Dubbo线程模型时提到，为了尽量早地释放Netty的I/O线程，某些线程模型会把请求投递到线程池进行异步处理，那么这里所谓的线程池是什么样的线程池呢？其实这里的线程池ThreadPool也是一个扩展接口SPI，Dubbo提供了该扩展接口的一些实现，具体如下。

* FixedThreadPool：创建一个具有固定个数线程的线程池。
* LimitedThreadPool：创建一个线程池，这个线程池中的线程个数随着需要量动态增加，但是数量不超过配置的阈值。另外，空闲线程不会被回收，会一直存在。
* EagerThreadPool：创建一个线程池，在这个线程池中，当所有核心线程都处于忙碌状态时，将创建新的线程来执行新任务，而不是把任务放入线程池阻塞队列。
* CachedThreadPool：创建一个自适应线程池，当线程空闲1分钟时，线程会被回收；当有新请求到来时，会创建新线程。

## FixedThreadPool源码剖析

​	FixedThreadPool的代码如下：

```java
public class FixedThreadPool implements ThreadPool {

    @Override
    public Executor getExecutor(URL url) {
        String name = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
        int threads = url.getParameter(THREADS_KEY, DEFAULT_THREADS);
        int queues = url.getParameter(QUEUES_KEY, DEFAULT_QUEUES);
        return new ThreadPoolExecutor(threads, threads, 0, TimeUnit.MILLISECONDS,
                queues == 0 ? new SynchronousQueue<Runnable>() :
                        (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                                : new LinkedBlockingQueue<Runnable>(queues)),
                new NamedInternalThreadFactory(name, true), new AbortPolicyWithReport(name, url));
    }

}
```

​	代码1获取用户设置的线程池中线程的名称前缀，如果没有设置，则使用默认名称Dubbo。

​	代码2获取用户设置的线程池中线程的个数，如果没有设置，则使用默认的数值200。

​	代码3获取用户设置的线程池的阻塞队列大小，如果没有设置，则使用默认的数值0。

​	代码4使用JUC包里的ThreadPoolExecutor创建线程池。关于ThreadPoolExecutor，读者可以参考笔者所著的《Java并发编程之美》一书。

​	这里把ThreadPoolExecutor的核心线程个数和最大线程个数都设置为threads，所以创建的线程池是固定线程个数的线程池。另外，当队列元素为0时，阻塞队列使用的是SynchronousQueue；当队列元素小于0时，使用的是无界阻塞队列LinkedBlockingQueue；当队列元素大于0时，使用的是有界的LinkedBlockingQueue。

​	并且，这里使用了自定义的线程工厂NamedInternalThreadFactory来为线程创建自定义名称，并将线程设置为deamon线程（关于用户线程与deamon线程的区别，可以参考笔者所著的《Java并发编程之美》一书）。

​	最后，线程池拒绝策略选择了AbortPolicyWithReport，意味着当线程池队列已满并且线程池中线程都忙碌时，新来的任务会被丢弃，并抛出RejectedExecutionException异常。

## LimitedThreadPool源码剖析

```java
public class LimitedThreadPool implements ThreadPool {

    @Override
    public Executor getExecutor(URL url) {
        String name = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
        int cores = url.getParameter(CORE_THREADS_KEY, DEFAULT_CORE_THREADS);
        int threads = url.getParameter(THREADS_KEY, DEFAULT_THREADS);
        int queues = url.getParameter(QUEUES_KEY, DEFAULT_QUEUES);
        return new ThreadPoolExecutor(cores, threads, Long.MAX_VALUE, TimeUnit.MILLISECONDS,
                queues == 0 ? new SynchronousQueue<Runnable>() :
                        (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                                : new LinkedBlockingQueue<Runnable>(queues)),
                new NamedInternalThreadFactory(name, true), new AbortPolicyWithReport(name, url));
    }

}
```

## EagerThreadPool源码剖析

```java
public class EagerThreadPool implements ThreadPool {

    @Override
    public Executor getExecutor(URL url) {
        String name = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
        int cores = url.getParameter(CORE_THREADS_KEY, DEFAULT_CORE_THREADS);
        int threads = url.getParameter(THREADS_KEY, Integer.MAX_VALUE);
        int queues = url.getParameter(QUEUES_KEY, DEFAULT_QUEUES);
        int alive = url.getParameter(ALIVE_KEY, DEFAULT_ALIVE);

        // init queue and executor
        TaskQueue<Runnable> taskQueue = new TaskQueue<Runnable>(queues <= 0 ? 1 : queues);
        EagerThreadPoolExecutor executor = new EagerThreadPoolExecutor(cores,
                threads,
                alive,
                TimeUnit.MILLISECONDS,
                taskQueue,
                new NamedInternalThreadFactory(name, true),
                new AbortPolicyWithReport(name, url));
        taskQueue.setExecutor(executor);
        return executor;
    }
}
```

​	代码6使用自定义线程池EagerThreadPoolExecutor和队列创建需要的线程池。

​	EagerThreadPoolExecutor与JUC包中的ThreadPoolExecutor不同之处在于，对于后者来说，当线程池核心线程个数达到设置的阈值时，新来的任务会被放入线程池队列，等队列满了以后，才会开启新线程来处理任务（前提是当前线程个数没有超过线程池最大线程个数）；而对于前者来说，当线程池核心线程个数达到设置的阈值时，新来的任务不会被放入线程池队列，而是会开启新线程来处理任务（前提是当前线程个数没有超过线程池最大线程个数），当线程个数达到最大线程个数时，才会把任务放入线程池队列。

​	由于EagerThreadPoolExecutor继承自ThreadPoolExecutor，所以在向EagerThreadPoolExecutor提交任务后，最终还是会调用ThreadPoolExecutor的execute（）方法，这里我们简单讲解一下EagerThreadPoolExecutor的功能是如何实现的：

```java
public void execute(Runnable command) {
    if (command == null) {
        throw new NullPointerException();
    }
    // do not increment in method beforeExecute!
    submittedTaskCount.incrementAndGet();
    try {
        super.execute(command);
    } catch (RejectedExecutionException rx) {
        // retry to offer the task into queue.
        final TaskQueue queue = (TaskQueue) super.getQueue();
        try {
            if (!queue.retryOffer(command, 0, TimeUnit.MILLISECONDS)) {
                submittedTaskCount.decrementAndGet();
                throw new RejectedExecutionException("Queue capacity is full.", rx);
            }
        } catch (InterruptedException x) {
            submittedTaskCount.decrementAndGet();
            throw new RejectedExecutionException(x);
        }
    } catch (Throwable t) {
        // decrease any way
        submittedTaskCount.decrementAndGet();
        throw t;
    }
}
```

​	在代码3中，当目前线程池线程个数大于等于核心线程个数时会执行代码4。在正常情况下，代码4会把任务添加到队列而不是开启新线程，但是EagerThreadPoolExecutor使用了自己的队列TaskQueue，下面我们看看TaskQueue的offer（）方法：

```java
public boolean offer(Runnable runnable) {
    if (executor == null) {
        throw new RejectedExecutionException("The task queue does not have executor!");
    }

    int currentPoolThreadSize = executor.getPoolSize();
    // have free worker. put task into queue to let the worker deal with task.
    if (executor.getSubmittedTaskCount() < currentPoolThreadSize) {
        return super.offer(runnable);
    }

    // return false to let executor create new worker.
    if (currentPoolThreadSize < executor.getMaximumPoolSize()) {
        return false;
    }

    // currentPoolThreadSize >= max
    return super.offer(runnable);
}
```

​	在代码7中，如果当前线程池线程个数小于线程池设置的最大个数，则返回false，然后执行代码5新开启线程来处理当前任务。

## CachedThreadPool源码剖析

```java
@Override
public Executor getExecutor(URL url) {
    String name = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
    int cores = url.getParameter(CORE_THREADS_KEY, DEFAULT_CORE_THREADS);
    int threads = url.getParameter(THREADS_KEY, Integer.MAX_VALUE);
    int queues = url.getParameter(QUEUES_KEY, DEFAULT_QUEUES);
    int alive = url.getParameter(ALIVE_KEY, DEFAULT_ALIVE);
    return new ThreadPoolExecutor(cores, threads, alive, TimeUnit.MILLISECONDS,
            queues == 0 ? new SynchronousQueue<Runnable>() :
                    (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                            : new LinkedBlockingQueue<Runnable>(queues)),
            new NamedInternalThreadFactory(name, true), new AbortPolicyWithReport(name, url));
}
```

​	代码6使用JUC包的ThreadPoolExecutor创建线程池，需要注意的是，这里设置了线程池中线程空闲时间，当线程空闲时间达到后，线程会被回收。

##  线程池的确定时机 todo

​	到这里介绍完了线程模型的加载位置，但线程模型中的线程池SPI扩展什么时候加载呢？这里以线程模型AllDispatcher为例做介绍。线程模型AllDispatcher对应的处理器是AllChannelHandler，其构造函数如下：

​	可能是

```java
public ExecutorService getSharedExecutorService() {
    ExecutorRepository executorRepository =
            ExtensionLoader.getExtensionLoader(ExecutorRepository.class).getDefaultExtension();
    ExecutorService executor = executorRepository.getExecutor(url);
    if (executor == null) {
        executor = executorRepository.createExecutorIfAbsent(url);
    }
    return executor;
}
```

## 如何基于扩展接口自定义线程池策略

​	首先，需要实现扩展接口ThreadPool，其代码如下：

![](https://pic.imgdb.cn/item/611510745132923bf81ad756.jpg)

![](https://pic.imgdb.cn/item/611516165132923bf8261868.jpg)

​	然后，需要在项目的src/main/resources目录下创建目录/META-INF/dubbo，并在该目录下创建文件org.apache.dubbo.common.threadpool.ThreadPool，文件内容为mythreadpool=com.books.dubbo.demo.provider.mydispatcher.MyThreadPool。

​	最后，在服务提供者启动时把线程池策略设置为mythreadpool即可：

![](https://pic.imgdb.cn/item/6115162c5132923bf82640fc.jpg)

# Dubbo如何实现泛化引用 todo

​	在基础篇中我们已经介绍了，基于Dubbo API搭建Dubbo服务时，服务消费端引入了一个SDK二方包，里面存放着服务提供端提供的所有接口类。

​	泛化接口调用方式主要是在服务消费端没有API接口类及模型类元（比如入参和出参的POJO类）的情况下使用，其参数及返回值中没有对应的POJO类，所以全部POJO均转换为Map表示。使用泛化调用时服务消费模块不再需要引入SDK二方包。

​	我们在本章中将探讨Dubbo如何实现泛化调用，主要内容包括：服务消费端如何使用GenericImplFilter拦截泛化调用，把泛化参数进行校验并发起远程调用；服务提供方如何使用GenericFilter拦截请求，并把泛化参数进行反序列化处理，然后把请求转发给具体的服务进行执行。调用流程图如图8.1所示。

![](https://pic.imgdb.cn/item/611516805132923bf826e489.jpg)

## 服务消费端GenericImplFilter源码分析 

​	在基础篇中介绍的泛化调用有三个测试类，分别为APiGenericConsumerForTrue、APiGenericConsumerForNativeJava和APiGenericConsumerForBean，三者分别对应三种泛化引用。这里我们使用APiGenericConsumer来替代这三个类进行讲解，首先看看如图8.2所示的时序图。

![](https://pic.imgdb.cn/item/6115170d5132923bf828080a.jpg)

​	通过图8.2可知，在消费端使用泛化调用发起请求后，请求会经过包装类ProtocolFilterWrapper创建的责任链。这里只展示了责任链中的GenericImplFilter过滤器，GenericImplFilter处理完后会把请求传递给被包装的DubboInvoker，然后DubboInvoker会发起远程调用，这里我们主要看看GenericImplFilter是如何实现泛化引用的：

```java
@Override
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    String generic = invoker.getUrl().getParameter(GENERIC_KEY);
    // calling a generic impl service
    // 是否返回引用
    if (isCallingGenericImpl(generic, invocation)) {
        RpcInvocation invocation2 = new RpcInvocation(invocation);

        /**
         * Mark this invocation as a generic impl call, this value will be removed automatically before passing on the wire.
         * See {@link RpcUtils#sieveUnnecessaryAttachments(Invocation)}
         */
        invocation2.put(GENERIC_IMPL_MARKER, true);

        String methodName = invocation2.getMethodName();
        Class<?>[] parameterTypes = invocation2.getParameterTypes();
        // 参数
        Object[] arguments = invocation2.getArguments();

        String[] types = new String[parameterTypes.length];
        for (int i = 0; i < parameterTypes.length; i++) {
            types[i] = ReflectUtils.getName(parameterTypes[i]);
        }

        Object[] args;
        if (ProtocolUtils.isBeanGenericSerialization(generic)) {
            args = new Object[arguments.length];
            for (int i = 0; i < arguments.length; i++) {
                args[i] = JavaBeanSerializeUtil.serialize(arguments[i], JavaBeanAccessor.METHOD);
            }
        } else {
            args = PojoUtils.generalize(arguments);
        }

        if (RpcUtils.isReturnTypeFuture(invocation)) {
            invocation2.setMethodName($INVOKE_ASYNC);
        } else {
            invocation2.setMethodName($INVOKE);
        }
        invocation2.setParameterTypes(GENERIC_PARAMETER_TYPES);
        invocation2.setParameterTypesDesc(GENERIC_PARAMETER_DESC);
        invocation2.setArguments(new Object[]{methodName, types, args});
        return invoker.invoke(invocation2);
    }
    // making a generic call to a normal service
    else if (isMakingGenericCall(generic, invocation)) {

        Object[] args = (Object[]) invocation.getArguments()[2];
        if (ProtocolUtils.isJavaGenericSerialization(generic)) {

            for (Object arg : args) {
                if (byte[].class != arg.getClass()) {
                    error(generic, byte[].class.getName(), arg.getClass().getName());
                }
            }
        } else if (ProtocolUtils.isBeanGenericSerialization(generic)) {
            for (Object arg : args) {
                if (!(arg instanceof JavaBeanDescriptor)) {
                    error(generic, JavaBeanDescriptor.class.getName(), arg.getClass().getName());
                }
            }
        }

        invocation.setAttachment(
                GENERIC_KEY, invoker.getUrl().getParameter(GENERIC_KEY));
    }
    return invoker.invoke(invocation);
}
```

## 服务提供端GenericFilter源码分析

​	在3.2节我们讲到，服务提供端默认会把接收到的请求封装为ChannelEventRunnable任务，并投递到业务线程池以便及时释放I/O线程，这里我们就从ChannelEventRunnable的执行开始，看看流程是如何走到GenericFilter的。服务提供端处理请求的时序图如图8.3所示。

![](https://pic.imgdb.cn/item/611517b95132923bf8295fee.jpg)

​	从图8.3可以看到，当异步任务被执行时，请求会经过包装类ProtocolFilterWrapper创建的责任链。这里只展示了责任链中的GenericFilter过滤器，GenericFilter处理完后会把请求传递给AbstractProxyInvoker，然后AbstractProxyInvoker会进行本地服务的执行。这里我们主要看看GenericFilter是如何实现泛化引用的：

```java
public Result invoke(Invoker<?> invoker, Invocation inv) throws RpcException {
    // 首付繁华请求
    if ((inv.getMethodName().equals($INVOKE) || inv.getMethodName().equals($INVOKE_ASYNC))
            && inv.getArguments() != null
            && inv.getArguments().length == 3
            && !GenericService.class.isAssignableFrom(invoker.getInterface())) {
        String name = ((String) inv.getArguments()[0]).trim();
        String[] types = (String[]) inv.getArguments()[1];
        Object[] args = (Object[]) inv.getArguments()[2];
        try {
            // 反射获取调用方法
            Method method = ReflectUtils.findMethodByMethodSignature(invoker.getInterface(), name, types);
            Class<?>[] params = method.getParameterTypes();
            if (args == null) {
                args = new Object[params.length];
            }

            if(types == null) {
                types = new String[params.length];
            }

            if (args.length != types.length) {
                throw new RpcException("GenericFilter#invoke args.length != types.length, please check your "
                        + "params");
            }
            String generic = inv.getAttachment(GENERIC_KEY);

            if (StringUtils.isBlank(generic)) {
                // 获取泛化引用方使用的泛化类型
                generic = RpcContext.getClientAttachment().getAttachment(GENERIC_KEY);
            }
			// 泛化类型为空，则使用generic=true的泛化方式
            if (StringUtils.isEmpty(generic)
                    || ProtocolUtils.isDefaultGenericSerialization(generic)
                    || ProtocolUtils.isGenericReturnRawResult(generic)) {
                try {
                    args = PojoUtils.realize(args, params, method.getGenericParameterTypes());
                } catch (IllegalArgumentException e) {
                    throw new RpcException(e);
                }
            } else if (ProtocolUtils.isGsonGenericSerialization(generic)) {
                args = getGsonGenericArgs(args, method.getGenericParameterTypes());
             // nativejava的方式
            } else if (ProtocolUtils.isJavaGenericSerialization(generic)) {
                Configuration configuration = ApplicationModel.getEnvironment().getConfiguration();
                if (!configuration.getBoolean(CommonConstants.ENABLE_NATIVE_JAVA_GENERIC_SERIALIZE, false)) {
                    String notice = "Trigger the safety barrier! " +
                            "Native Java Serializer is not allowed by default." +
                            "This means currently maybe being attacking by others. " +
                            "If you are sure this is a mistake, " +
                            "please set `" + CommonConstants.ENABLE_NATIVE_JAVA_GENERIC_SERIALIZE + "` enable in configuration! " +
                            "Before doing so, please make sure you have configure JEP290 to prevent serialization attack.";
                    logger.error(notice);
                    throw new RpcException(new IllegalStateException(notice));
                }

                for (int i = 0; i < args.length; i++) {
                    if (byte[].class == args[i].getClass()) {
                        try (UnsafeByteArrayInputStream is = new UnsafeByteArrayInputStream((byte[]) args[i])) {
                            args[i] = ExtensionLoader.getExtensionLoader(Serialization.class)
                                    .getExtension(GENERIC_SERIALIZATION_NATIVE_JAVA)
                                    .deserialize(null, is).readObject();
                        } catch (Exception e) {
                            throw new RpcException("Deserialize argument [" + (i + 1) + "] failed.", e);
                        }
                    } else {
                        throw new RpcException(
                                "Generic serialization [" +
                                        GENERIC_SERIALIZATION_NATIVE_JAVA +
                                        "] only support message type " +
                                        byte[].class +
                                        " and your message type is " +
                                        args[i].getClass());
                    }
                }
            } else if (ProtocolUtils.isBeanGenericSerialization(generic)) {
                for (int i = 0; i < args.length; i++) {
                    if (args[i] instanceof JavaBeanDescriptor) {
                        args[i] = JavaBeanSerializeUtil.deserialize((JavaBeanDescriptor) args[i]);
                    } else {
                        throw new RpcException(
                                "Generic serialization [" +
                                        GENERIC_SERIALIZATION_BEAN +
                                        "] only support message type " +
                                        JavaBeanDescriptor.class.getName() +
                                        " and your message type is " +
                                        args[i].getClass().getName());
                    }
                }
            } else if (ProtocolUtils.isProtobufGenericSerialization(generic)) {
                // as proto3 only accept one protobuf parameter
                if (args.length == 1 && args[0] instanceof String) {
                    try (UnsafeByteArrayInputStream is =
                                 new UnsafeByteArrayInputStream(((String) args[0]).getBytes())) {
                        args[0] = ExtensionLoader.getExtensionLoader(Serialization.class)
                                .getExtension(GENERIC_SERIALIZATION_PROTOBUF)
                                .deserialize(null, is).readObject(method.getParameterTypes()[0]);
                    } catch (Exception e) {
                        throw new RpcException("Deserialize argument failed.", e);
                    }
                } else {
                    throw new RpcException(
                            "Generic serialization [" +
                                    GENERIC_SERIALIZATION_PROTOBUF +
                                    "] only support one " + String.class.getName() +
                                    " argument and your message size is " +
                                    args.length + " and type is" +
                                    args[0].getClass().getName());
                }
            }

            RpcInvocation rpcInvocation =
                    new RpcInvocation(method, invoker.getInterface().getName(), invoker.getUrl().getProtocolServiceKey(), args,
                            inv.getObjectAttachments(), inv.getAttributes());
            rpcInvocation.setInvoker(inv.getInvoker());
            rpcInvocation.setTargetServiceUniqueName(inv.getTargetServiceUniqueName());

            return invoker.invoke(rpcInvocation);
        } catch (NoSuchMethodException | ClassNotFoundException e) {
            throw new RpcException(e.getMessage(), e);
        }
    }
    return invoker.invoke(inv);
}
```

# Dubbo并发控制

​	由于资源的限制，一般会在服务提供方和消费方限制接口调用的并发数，本章将探讨相关的原理。图9.1是原理的模型图。

![](https://pic.imgdb.cn/item/6115d6835132923bf84bf697.jpg)

## 服务消费端并发控制

​	在基础篇的Demo中，APiConsumerForActiveLimit类使用了消费端并发控制，其代码如下：

![](https://pic.imgdb.cn/item/6115d72c5132923bf84d12ba.jpg)

​	上面的代码对com.books.dubbo.demo.api.GreetingService接口中的所有方法进行了设置，每个方法最多同时并发请求10个。其实也可以只设置接口GreetingService中某一个方法的并发请求限制个数：

![](https://pic.imgdb.cn/item/6115d73f5132923bf84d31d1.jpg)

​	在服务消费端，具体进行并发控制的是ProtocolFilterWrapper类创建的Filter链中的ActiveLimitFilter（）方法，下面我们看看如图9.2所示的调用时序图。

![](https://pic.imgdb.cn/item/6115d7785132923bf84d8ed5.jpg)

​	在上面的代码中，在DubboInvoker的invoke（）方法真正向远端发起RPC请求前，请求会先经过ActiveLimitFilter的invoke（）方法，其代码如下：

```java
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    // 获取url
    URL url = invoker.getUrl();
    String methodName = invocation.getMethodName();
    // 获取最大可用并发数
    int max = invoker.getUrl().getMethodParameter(methodName, ACTIVES_KEY, 0);
    final RpcStatus rpcStatus = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName());
    // 判断是不是超过并发了
    if (!RpcStatus.beginCount(url, methodName, max)) {
        long timeout = invoker.getUrl().getMethodParameter(invocation.getMethodName(), TIMEOUT_KEY, 0);
        long start = System.currentTimeMillis();
        long remain = timeout;
        // 包裹并发阻塞当前线程timeout时间
        synchronized (rpcStatus) {
            while (!RpcStatus.beginCount(url, methodName, max)) {
                try {
                    rpcStatus.wait(remain);
                } catch (InterruptedException e) {
                    // ignore
                }
                long elapsed = System.currentTimeMillis() - start;
                remain = timeout - elapsed;
                // 超时了还没被唤醒则抛出异常
                if (remain <= 0) {
                    throw new RpcException(RpcException.LIMIT_EXCEEDED_EXCEPTION,
                            "Waiting concurrent invoke timeout in client-side for service:  " +
                                    invoker.getInterface().getName() + ", method: " + invocation.getMethodName() +
                                    ", elapsed: " + elapsed + ", timeout: " + timeout + ". concurrent invokes: " +
                                    rpcStatus.getActive() + ". max concurrent invoke limit: " + max);
                }
            }
        }
    }

    invocation.put(ACTIVELIMIT_FILTER_START_TIME, System.currentTimeMillis());

    return invoker.invoke(invocation);
}
public static boolean beginCount(URL url, String methodName, int max) {
        max = (max <= 0) ? Integer.MAX_VALUE : max;
        RpcStatus appStatus = getStatus(url);
        RpcStatus methodStatus = getStatus(url, methodName);
        if (methodStatus.active.get() == Integer.MAX_VALUE) {
            return false;
        }
        for (int i; ; ) {
            i = methodStatus.active.get();

            if (i == Integer.MAX_VALUE || i + 1 > max) {
                return false;
            }

            if (methodStatus.active.compareAndSet(i, i + 1)) {
                break;
            }
        }

        appStatus.active.incrementAndGet();

        return true;
    }
/**
     * @param url
     * @param elapsed
     * @param succeeded
     */
    public static void endCount(URL url, String methodName, long elapsed, boolean succeeded) {
        endCount(getStatus(url), elapsed, succeeded);
        endCount(getStatus(url, methodName), elapsed, succeeded);
    }

    private static void endCount(RpcStatus status, long elapsed, boolean succeeded) {
        status.active.decrementAndGet();
        status.total.incrementAndGet();
        status.totalElapsed.addAndGet(elapsed);

        if (status.maxElapsed.get() < elapsed) {
            status.maxElapsed.set(elapsed);
        }

        if (succeeded) {
            if (status.succeededMaxElapsed.get() < elapsed) {
                status.succeededMaxElapsed.set(elapsed);
            }

        } else {
            status.failed.incrementAndGet();
            status.failedElapsed.addAndGet(elapsed);
            if (status.failedMaxElapsed.get() < elapsed) {
                status.failedMaxElapsed.set(elapsed);
            }
        }
    }
```

​	通过上面的代码可知，在RpcStatus中维护了一个并发安全的METHOD_STATISTICS缓存，其中key为服务接口，value为该服务接口中所有方法的一个缓存，后者ConcurrentMap＜String，RpcStatus＞的key为具体的方法，value为RpcStatus对象。

​	如果beginCount（）方法返回了false，则让当前线程挂起timeout时间，然后当前线程会在timeout时间后再被唤醒，或者当其他线程调用了count的notifyAll（）方法时被唤醒。如果超过timeout时间还没被唤醒，则当前线程会自动被唤醒，然后抛出RpcException异常，也就是说远程调用还没到达服务提供方，调用方就抛出异常结束了。

​	综上所述，在客户端并发控制中，如果当激活并发量达到指定值后，当前客户端请求线程会被挂起。如果在等待超时期间激活并发请求量少了，那么阻塞的线程会被激活，然后发送请求到服务提供方；如果等待超时了，则直接抛出异常，这时服务根本就没有发送到服务提供方服务器。

## 服务提供端并发控制

​	在基础篇的Demo中，ApiProviderForExecuteLimit使用了服务提供端并发控制：

![](https://pic.imgdb.cn/item/6115dca15132923bf8569c14.jpg)

​	上面的代码对com.books.dubbo.demo.api.GreetingService接口中的所有方法进行了设置，每个方法最多同时处理10个请求。其实也可以只设置接口GreetingService中的某一个方法的并发处理限制个数：

![](https://pic.imgdb.cn/item/6115dcc75132923bf856dde9.jpg)

​	上面的代码只设置了sayHello（）方法的同时并发处理数目。

​	在服务提供端，具体进行并发控制的是ProtocolFilterWrapper类创建的Filter链中的ExecuteLimitFilter（）方法，下面我们看看如图9.3所示的调用时序图。

![](https://pic.imgdb.cn/item/6115dce55132923bf85713a6.jpg)

​	在上面的代码中，服务提供方的AbstractProxyInvoker的invoke（）方法在真正执行服务处理的前，请求会先经过ExecuteLimitFilter的invoke（）方法，其代码如下：

```java
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    URL url = invoker.getUrl();
    String methodName = invocation.getMethodName();
    int max = url.getMethodParameter(methodName, EXECUTES_KEY, 0);
    if (!RpcStatus.beginCount(url, methodName, max)) {
        throw new RpcException(RpcException.LIMIT_EXCEEDED_EXCEPTION,
                "Failed to invoke method " + invocation.getMethodName() + " in provider " +
                        url + ", cause: The service using threads greater than <dubbo:service executes=\"" + max +
                        "\" /> limited.");
    }

    invocation.put(EXECUTE_LIMIT_FILTER_START_TIME, System.currentTimeMillis());
    try {
        return invoker.invoke(invocation);
    } catch (Throwable t) {
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        } else {
            throw new RpcException("unexpected exception when ExecuteLimitFilter", t);
        }
    }
}
```

​	代码2获取设置的executes的值，代码3递增方法对应的激活并发数。如果并发数超过了设置最大值，则直接抛出异常，否则执行代码4继续Filter链的处理，正常执行服务处理，然后执行代码5将当前方法激活并发数减去1。

​	需要注意的是，服务提供方设置并发数后，如果同时请求数超过了设置的executes的值，则会抛出异常，而不是像消费端设置actives时那样去等待。

# Dubbo隐式参数传递

​	在基础篇中我们讲到Dubbo提供了隐式参数传递的功能，即服务调用方可以通过RpcContext.getContext（）.setAttachment（）方法设置附加属性键值对，然后设置的键值对可以在服务提供方服务方法内获取。

​	要实现隐式参数传递，首先需要在服务消费端的AbstractClusterInvoker类的invoke（）方法内，把附加属性键值对放入到RpcInvocation的attachments变量中，然后经过网络传递到服务提供端；服务提供端则使用ContextFilter对请求进行拦截，并从RpcInvocation中获取attachments中的键值对，然后使用RpcContext.getContext（）.setAttachment设置到上下文对象中，其模型图如图10.1所示。

![](https://pic.imgdb.cn/item/6115dd915132923bf85840d6.jpg)

## 服务消费端AbstractClusterInvoker原理剖析

​	AbstractClusterInvoker是服务集群容错策略的抽象类，在默认情况下，集群容错策略为FailoverClusterInvoker，其继承了AbstractClusterInvoker，下面我们首先看看如图10.2所示的时序图。

![](https://pic.imgdb.cn/item/6115de655132923bf859cb88.jpg)

​	从图中可以看到，服务消费端启动后最终会到达默认的集群容错策略FailoverClusterInvoker的invoke（）方法，其代码如下：

```java
@Override
    public Result invoke(final Invocation invocation) throws RpcException {
        checkWhetherDestroyed();

        // binding attachments into invocation.
//        Map<String, Object> contextAttachments = RpcContext.getClientAttachment().getObjectAttachments();
//        if (contextAttachments != null && contextAttachments.size() != 0) {
//            ((RpcInvocation) invocation).addObjectAttachmentsIfAbsent(contextAttachments);
//        }

        List<Invoker<T>> invokers = list(invocation);
        LoadBalance loadbalance = initLoadBalance(invokers, invocation);
        RpcUtils.attachInvocationIdIfAsync(getUrl(), invocation);
        return doInvoke(invocation, invokers, loadbalance);
    }
```

​	服务消费端什么时候会把里面设置的值清除掉呢？其实清除操作是在时序图中ConsumerContextFilter的invoke（）方法内完成的：

```java
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    RpcContext.getServiceContext()
            .setInvocation(invocation)
            .setLocalAddress(NetUtils.getLocalHost(), 0);

    RpcContext context = RpcContext.getClientAttachment();
    context.setAttachment(REMOTE_APPLICATION_KEY, invoker.getUrl().getApplication());
    if (invocation instanceof RpcInvocation) {
        ((RpcInvocation) invocation).setInvoker(invoker);
    }

    ExtensionLoader<PenetrateAttachmentSelector> selectorExtensionLoader = ExtensionLoader.getExtensionLoader(PenetrateAttachmentSelector.class);
    Set<String> supportedSelectors = selectorExtensionLoader.getSupportedExtensions();
    if (CollectionUtils.isNotEmpty(supportedSelectors)) {
        for (String supportedSelector : supportedSelectors) {
            Map<String, Object> selected = selectorExtensionLoader.getExtension(supportedSelector).select();
            if (CollectionUtils.isNotEmptyMap(selected)) {
                ((RpcInvocation) invocation).addObjectAttachmentsIfAbsent(selected);
            }
        }
    } else {
        ((RpcInvocation) invocation).addObjectAttachmentsIfAbsent(RpcContext.getServerAttachment().getObjectAttachments());
    }

    Map<String, Object> contextAttachments = RpcContext.getClientAttachment().getObjectAttachments();
    if (CollectionUtils.isNotEmptyMap(contextAttachments)) {
        /**
         * invocation.addAttachmentsIfAbsent(context){@link RpcInvocation#addAttachmentsIfAbsent(Map)}should not be used here,
         * because the {@link RpcContext#setAttachment(String, String)} is passed in the Filter when the call is triggered
         * by the built-in retry mechanism of the Dubbo. The attachment to update RpcContext will no longer work, which is
         * a mistake in most cases (for example, through Filter to RpcContext output traceId and spanId and other information).
         */
        ((RpcInvocation) invocation).addObjectAttachments(contextAttachments);
    }

    try {
        // pass default timeout set by end user (ReferenceConfig)
        Object countDown = context.getObjectAttachment(TIME_COUNTDOWN_KEY);
        if (countDown != null) {
            TimeoutCountDown timeoutCountDown = (TimeoutCountDown) countDown;
            if (timeoutCountDown.isExpired()) {
                return AsyncRpcResult.newDefaultAsyncResult(new RpcException(RpcException.TIMEOUT_TERMINATE,
                        "No time left for making the following call: " + invocation.getServiceName() + "."
                                + invocation.getMethodName() + ", terminate directly."), invocation);
            }
        }

        RpcContext.removeServerContext();
        return invoker.invoke(invocation);
    } finally {
        // 清除附加属性
        RpcContext.removeServiceContext();
        RpcContext.removeClientAttachment();
    }
}
```

​	当请求发出去后，会清除当前与调用线程关联的线程变量里面的附加属性。

## 服务提供方ContextFilter原理剖析

​	上一节讲解了服务消费端如何把隐式参数设置到Invocation参数中，本节我们看看服务提供端接收请求后如何把隐式参数设置到上下文对象中，以便让服务提供方在服务方法内获取，服务提供端ContextFilter过滤器的激活时序与8.1节所讲的GenericFilter的激活时序一致，这里不再列出时序图，我们直接看ContextFilter的代码：

```java
public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    // 获取附加属性map
        Map<String, Object> attachments = invocation.getObjectAttachments();
    //不为null就设置到上下文中    
    if (attachments != null) {
            Map<String, Object> newAttach = new HashMap<>(attachments.size());
            for (Map.Entry<String, Object> entry : attachments.entrySet()) {
                String key = entry.getKey();
                if (!UNLOADING_KEYS.contains(key)) {
                    newAttach.put(key, entry.getValue());
                }
            }
            attachments = newAttach;
        }

        RpcContext.getServiceContext().setInvoker(invoker)
                .setInvocation(invocation);

        RpcContext context = RpcContext.getServerAttachment();
//                .setAttachments(attachments)  // merged from dubbox
        context.setLocalAddress(invoker.getUrl().getHost(), invoker.getUrl().getPort());

        String remoteApplication = (String) invocation.getAttachment(REMOTE_APPLICATION_KEY);
        if (StringUtils.isNotEmpty(remoteApplication)) {
            RpcContext.getServiceContext().setRemoteApplicationName(remoteApplication);
        } else {
            RpcContext.getServiceContext().setRemoteApplicationName((String) context.getAttachment(REMOTE_APPLICATION_KEY));
        }

        long timeout = RpcUtils.getTimeout(invocation, -1);
        if (timeout != -1) {
            // pass to next hop
            RpcContext.getClientAttachment().setObjectAttachment(TIME_COUNTDOWN_KEY, TimeoutCountDown.newCountDown(timeout, TimeUnit.MILLISECONDS));
        }

        // merged from dubbox
        // we may already added some attachments into RpcContext before this filter (e.g. in rest protocol)
        if (attachments != null) {
            if (context.getObjectAttachments() != null) {
                context.getObjectAttachments().putAll(attachments);
            } else {
                context.setObjectAttachments(attachments);
            }
        }

        if (invocation instanceof RpcInvocation) {
            ((RpcInvocation) invocation).setInvoker(invoker);
        }

        try {
            context.clearAfterEachInvoke(false);
            return invoker.invoke(invocation);
        } finally {
            // 清除上下文对象，则附加属性也被回收
            context.clearAfterEachInvoke(true);
            RpcContext.removeServerAttachment();
            RpcContext.removeServiceContext();
            // IMPORTANT! For async scenario, we must remove context from current thread, so we always create a new RpcContext for the next invoke for the same thread.
            RpcContext.getClientAttachment().removeAttachment(TIME_COUNTDOWN_KEY);
            RpcContext.removeServerContext();
        }
    }
```

​	代码1首先从从invocation对象获取附加属性Map，代码2发现附加属性Map不为null，则添加附加属性到上下文对象的attachments对象里，这样在服务实现方法里就可以通过RpcContext.getContext（）.getAttachment来获取附加属性了。代码3等服务逻辑执行完毕后清除当前线程所关联的上下文对象，由于附加属性属于上下文对象，所以附加属性也会被回收。

# Dubbo全链路异步

## 服务消费端异步调用

​	正如Dubbo官网所介绍的，从2.7.0版本开始，Dubbo以CompletableFuture为基础支持所有异步编程接口，解决了2.7.0之前的版本异步调用的不便与功能缺失。

​	异步调用是基于NIO的非阻塞能力实现并行调用，服务消费端不需要启动多线程即可完成并行调用多个远程服务，相对多线程开销较小。图11.1显示的是Dubbo异步调用链路的流程图。

![](https://pic.imgdb.cn/item/6115e1385132923bf85ec270.jpg)

​	图11.1中的步骤1是当服务消费端发起RPC调用时使用的用户线程，用户线程首先使用步骤2创建一个Future对象，接着步骤3会把请求转换为I/O线程来执行，步骤3为异步过程，所以会马上返回，然后用户线程使用步骤4把其创建的Future对象设置到RpcContext中，其后用户线程就返回了。

​	在步骤5中，用户线程可以在某个时间点从RpcContext中获取设置的Future对象，并且使用步骤6来等待调用结果。

​	在步骤7中，当服务提供方返回结果后，调用方线程模型中的线程池中的线程则会把结果使用步骤8写入Future，这时用户线程就可以得到远程调用结果了。

​	图11.1中的实线箭头代表同步调用，虚线箭头表示异步调用。

### 2.7.0版本前的异步调用实现  (3.0todo)

​	2.7.0版本之前的异步调用能力比较弱，比如在基础篇介绍的APiAsyncConsumer类里使用下面的方式进行异步调用：

![](https://pic.imgdb.cn/item/6115e1d25132923bf85fcada.jpg)

​	代码2设置调用为异步方式，代码3直接调用sayHello（）方法会马上返回null。如果要想获取远程调用的真正结果，需要使用代码4获取future对象，并且调用future的get（）系列方法来获取真正的结果，下面我们看看这种方式是如何实现的。

​	在3.4节我们讲到，具体发起远程调用是在DubboInvoker的doInvoke（）方法内，其中会区分是否为异步调用：





​	如果发现是异步调用，代码1.6会调用currentClient.request（inv，timeout）执行远程调用，该调用不会阻塞，而是马上返回一个future对象，返回的future对象会被适配器类FutureAdapter做包装，接着把包装结果设置到RpcContext里，然后返回SimpleAsyncRpcResult，SimpleAsyncRpcResult的构造函数保存了上下文对象（这在后面会具体用到）：

![](https://pic.imgdb.cn/item/6115e3df5132923bf86355d1.jpg)

![](https://pic.imgdb.cn/item/6115e3ec5132923bf8636bcd.jpg)

​	如果发现是异步调用，代码1.6会调用currentClient.request（inv，timeout）执行远程调用，该调用不会阻塞，而是马上返回一个future对象，返回的future对象会被适配器类FutureAdapter做包装，接着把包装结果设置到RpcContext里，然后返回SimpleAsyncRpcResult，SimpleAsyncRpcResult的构造函数保存了上下文对象（这在后面会具体用到）：

![](https://pic.imgdb.cn/item/6115e4085132923bf8639abe.jpg)

另外，由于SimpleAsyncRpcResult的recreate（）方法的代码如下：

![](https://pic.imgdb.cn/item/6115e41e5132923bf863c083.jpg)

​	其内部会返回null。

​	为了探究2.7.0版本之前的异步调用的实现原理，我们需要看看代码1.6.1内到底做了什么。其中，currentClient是ReferenceCountExchangeClient的实例，我们首先看看如图11.2所示的时序图。

![](https://pic.imgdb.cn/item/6115e4b95132923bf864ca11.jpg)

​	通过图11.2可知，代码1.6.1最终调用了HeaderExchangeChannel的request（）方法，其代码如下：

![](https://pic.imgdb.cn/item/6115e4e15132923bf8650d95.jpg)

​	代码2创建了请求对象，然后代码3创建了一个future对象，代码4使用底层通信异步发送请求（使用Netty的I/O线程把请求写入远端）。因代码4是非阻塞的，所以会马上返回。

​	这里简单提一下，从用户线程发起远程调用到返回request，使用的都是用户线程。由于代码4channel.send（req）会马上返回，所以不会阻塞用户线程。

​	为了探究异步实现，我们需要看看代码3中的DefaultFuture，其类图如图11.3所示。

![](https://pic.imgdb.cn/item/6115e5015132923bf86541f1.jpg)

​	从图11.3可以看到，DefaultFuture中有3个静态变量：

![](https://pic.imgdb.cn/item/6115e5405132923bf865ade8.jpg)

​	由于是static变量，所以全部DefaultFuture实例共享这3个变量。

​	图11.3中的lock是一个独占锁，done是该锁的一个条件变量：

![](https://pic.imgdb.cn/item/6115e5585132923bf865da09.jpg)

​	lock和done是为了实现线程之间的通知等待模型，比如调用DefaultFuture的get（）方法的线程为了获取响应结果，内部会调用done.await（）方法挂起调用线程。当接收到响应结果后，调用方线程模型中线程池里的线程会调用received（）方法，其内部会把响应结果设置到DefaultFuture内，然后调用done的signal（）方法激活一个因调用done的await（）系列方法而挂起的线程（比如调用get（）方法被阻塞的线程）。

​	首先我们看看代码3是如何使用newFuture创建DefaultFuture对象的：

![](https://pic.imgdb.cn/item/6115e5735132923bf8660578.jpg)

​	其中，DefaultFuture构造函数如下：

![](https://pic.imgdb.cn/item/6115e5825132923bf8661e66.jpg)

![](https://pic.imgdb.cn/item/6115e58c5132923bf8663159.jpg)

​	DefaultFuture内部保存了请求的信息，包含请求ID、请求通道、请求内容、超时时间，并且把当前DefaultFuture对象保存到缓存中，通道也保存到缓存中，缓存的key为请求ID。

​	代码5创建完DefaultFuture后，代码6检查超时情况，其代码如下：

![](https://pic.imgdb.cn/item/6115e6745132923bf867c675.jpg)

​	代码6.1创建了一个可执行任务，代码6.2则当设置的超时时间到了之后执行创建的任务，在任务内会检查调用是否已经完成，具体代码如下：

![](https://pic.imgdb.cn/item/6115e68e5132923bf867f1aa.jpg)

​	当超时时间到了之后，上面的代码会检查任务是否已经完成，如果已经完成则直接返回，否则创建超时事件，并调用DefaultFuture.received把结果设置到future内。

​	到这里我们就讲完了newFuture（）方法的作用，具体来说就是创建一个DefaultFuture对象，并启动一个定时器，然后在超时时间后检查是否已经有响应结果，如果有则直接返回，否则返回超时信息。

​	下面我们看看received（Channel channel，Response response）方法：

![](https://pic.imgdb.cn/item/6115e6a75132923bf8681f45.jpg)

​	从上面的代码可知，当发起的请求的结果返回时或者超时时间到了之后，会调用received（）方法，其中代码7把请求ID对应的future从缓存中移除，然后调用doReceived（）方法把响应结果设置到DefaultFuture的结果变量response中：

![](https://pic.imgdb.cn/item/6115e6c05132923bf8684af6.jpg)

​	在上面的代码中，doReceived（）方法会把响应结果设置到response变量里，然后激活一个由于调用done的wait（）方法被阻塞的线程（比如由于调用了get（）系列方法导致线程阻塞的线程）。另外，如果在future上设置了回调，则会调用回调函数。

​	获取执行结果的get（）方法如下：

![](https://pic.imgdb.cn/item/6115e6d55132923bf8686e02.jpg)

​	在上面的代码中，如果超时时间＜=0，则使用默认值1000；如果任务还没完成（响应结果还是为null），则调用条件变量done的await（）方法，让当前线程挂起timeout时间——若在timeout时间内得到了响应结果（也就是说received（）方法被调用了），则当前线程会被激活；如果已经得到响应结果，则直接执行代码14返回结果。

​	简单总结一下：本节一开始讲解了Dubbo异步调用链路流程图，当服务消费端业务线程发起请求后，会创建一个DefaultFuture对象并设置到RpcContext中，然后在启动I/O线程发起请求后调用线程就返回了null的结果；当业务线程从RpcContext获取future对象并调用其get（）方法获取真实的响应结果后，当前线程会调用条件变量done的await（）方法而挂起；当服务提供端把结果写回调用方之后，调用方线程模型中线程池里的线程会把结果写入DefaultFuture对象内的结果变量中，接着调用条件变量的signal（）方法来激活业务线程，然后业务线程就会从get（）方法返回响应结果。

​	这里所讲的这种实现异步调用的方式基于从返回的future调用get（）方法，其缺点是，当业务线程调用get（）方法后业务线程会被阻塞，这不是我们想要的，所以Dubbo提供了在future对象上设置回调函数的方式，让我们实现真正的异步调用。先看看在基础篇中介绍的APiAsyncConsumerForCallBack类：

![](https://pic.imgdb.cn/item/6115e7055132923bf868bff0.jpg)

​	通过上面的代码可知，这种方式在业务线程获取了future对象后，在其上设置回调函数后马上就会返回，接着等服务提供端把响应结果写回调用方，然后调用方线程模型中线程池里的线程会把结果写入future对象，其后对回调函数进行回调。由此可知，这个过程中是不需要业务线程干预的，实现了真正的异步调用。

​	下面我们看看回调函数的异步方式具体是如何实现的。在代码1.6.1中，在发起远程调用后，把返回的future对象使用FutureAdapter进行了包装，通过RpcContext.getContext（）.getFuture（）获取的其实就是FutureAdapter对象，调用FutureAdapter的getFuture（）方法获取的其实就是DefaultFuture对象，所以代码17是把回调函数设置到了DefaultFuture内，下面我们看看setCallback（）方法：

![](https://pic.imgdb.cn/item/6115e7335132923bf8690c66.jpg)

​	在上面的代码中，如果当前任务已经完成则直接执行回调，否则设置回调并等有响应结果时再执行回调。

​	其中，invokeCallback（）方法的代码为：

![](https://pic.imgdb.cn/item/6115e74f5132923bf8693e92.jpg)

​	另外，回顾一下前面所讲的doReceived（）方法内的代码10，可知当接收到响应结果后，如果发现回调函数不为null，则也会调用invokeCallback（）方法。

​	上面我们介绍了2.7.0版本之前提供的异步调用方式，Future方式只支持阻塞式的get（）接口获取结果。虽然通过获取内置的ResponseFuture接口，可以设置回调，但获取ResponseFuture的API使用起来很不便，并且无法满足让多个Future协同工作的场景，功能比较单一。

### 2.7.0版本提供的异步调用实现

​	在基础篇中介绍的APiAsyncConsumerForCompletableFuture2类使用了Dubbo 2.7.0版本提供的基于CompletableFuture的异步调用：

![](https://pic.imgdb.cn/item/6115e7735132923bf8697d12.jpg)

​	代码4可以直接获取到CompletableFuture，然后设置回调，基于CompletableFuture已有的能力，我们可以对CompletableFuture对象进行一系列的操作，以及可以让多个请求的CompletableFuture对象之间进行运算（比如合并两个CompletableFuture对象的结果为一个CompletableFuture对象等）。

​	下面我们看看这是如何实现的，首先看看RpcContext.getContext（）.getCompletableFuture（）到底返回的是什么：

![](https://pic.imgdb.cn/item/6115e7b25132923bf869e7e8.jpg)

​	从上面的代码可知，返回的还是FutureAdapter对象，所以我们还要回到FutureAdapter类，该类继承了CompletableFuture类。我们需要看看FutureAdapter类是如何将DefaultFuture转换到CompletableFuture的，为此先看看FutureAdapter的构造函数：

![](https://pic.imgdb.cn/item/6115e7c55132923bf86a074c.jpg)

![](https://pic.imgdb.cn/item/6115e7d05132923bf86a1974.jpg)

​	从上面的代码可知，FutureAdapter继承自CompletableFuture，代码6对传递的DefaultFuture设置回调（在2.7.0版本前，基于回调的异步调用需要我们自己设置回调，为了让CompletableFuture适配上DefaultFuture，专门内建一个回调）。然后，在回调方法的done（）内，把结果写入FutureAdapter继承的CompletableFuture内。

​	简单总结一下：当业务线程发起远程调用时，会创建一个DefaultFuture实例，接着经过FutureAdapter把DefaultFuture转换为CompletableFuture实例，然后把CompletableFuture实例设置到RpcContext内。业务线程从RpcContext获取该CompletableFuture后，设置业务回调函数。

​	当服务提供方把结果写回调用方之后，调用方的线程模型中线程池里的线程会调用DefaultFuture的received（）方法，并把响应结果写入DefaultFuture，接着调用这里的代码6所设置的回调，回调内部的done（）方法再把结果写入CompletableFuture，然后CompletableFuture会调用业务设置的业务回调函数。

## 服务提供端异步执行

​	在Provider端非异步执行时，对调用方发来的请求的处理是在Dubbo内部线程模型的线程池里的线程中执行的（参见第7章和3.2节）。在Dubbo中，服务提供方提供的所有服务接口都是使用这一个线程池来执行的，所以当一个服务执行比较耗时时，可能会占用线程池中很多线程，从而导致其他服务的处理收到影响。

​	Provider端异步执行则将服务的处理逻辑从Dubbo内部线程池切换到业务自定义线程，避免Dubbo线程池中的线程被过度占用，有助于避免不同服务间的互相影响。

​	但需要注意的是，Provider端异步执行对节省资源和提升RPC响应性能是没有效果的，这是因为如果服务处理比较耗时，虽然不是使用Dubbo框架内部线程处理，但还是需要业务自己的线程来处理，另外还有副作用，即会新增一次线程上下文切换（从Dubbo内部线程池线程切换到业务线程），模型如图11.4所示。

![](https://pic.imgdb.cn/item/6115e8bc5132923bf86bbeb8.jpg)

​	从图11.4可知，Provider端在同步提供服务时是使用Dubbo内部线程池中的线程来进行处理的，在异步执行时则是使用业务自己设置的线程从Dubbo内部线程池中的线程接收请求进行处理的。

### 基于定义CompletableFuture签名的接口实现异步执行

​	在基础篇的Demo中，GrettingServiceAsyncImpl中的服务提供端实现了基于定义CompletableFuture签名的接口实现异步执行：

![](https://pic.imgdb.cn/item/6115e9b45132923bf86d67b8.jpg)

​	通过上面的代码可知，基于定义CompletableFuture签名的接口实现异步执行需要接口方法的返回值为CompletableFuture，并且方法内部使用CompletableFuture.supplyAsync让本来该由Dubbo内部线程池中的线程处理的服务，转换为由业务自定义线程池中的线程来处理，CompletableFuture.supplyAsync（）方法会马上返回一个CompletableFuture对象（所以Dubbo内部线程池中的线程会得到及时释放），传递的业务函数则由业务线程池bizThreadpool执行。

​	需要注意的是，调用sayHello（）方法的线程是Dubbo线程模型线程池中的线程，而业务处理是由bizThreadpool中的线程来处理的，所以代码2.1保存了RPC上下文对象，以便在业务处理线程中使用。

​	在3.4节我们提到，当消费端发起远程调用时，请求会被InvokerInvocationHandler拦截，并在其中创建RpcInvocation，并且如果调用方法的返回值为CompletableFuture或者其子类，则会把future_returntype为true和async=true的属性添加到RpcInvocation的附加属性Map中。

​	在3.1节讲到，在Dubbo内部线程池中的线程接收received请求后，当请求被处理时，如果发现请求是需要返回值的，则会调用HeaderExchangeHandler的handleRequest（）方法，其代码如下：

![](https://pic.imgdb.cn/item/6115e9ea5132923bf86dc170.jpg)

![](https://pic.imgdb.cn/item/6115e9f65132923bf86dd4bb.jpg)

​	在上面的代码中，代码3执行的handler.reply的代码为：

![](https://pic.imgdb.cn/item/6115ea165132923bf86e06db.jpg)

​	其中，代码3.1最终会调用AbstractProxyInvoker的invoke（）方法：

![](https://pic.imgdb.cn/item/6115ea265132923bf86e22d3.jpg)

![](https://pic.imgdb.cn/item/6115ea355132923bf86e3dbf.jpg)

​	从上面的代码可知，返回结果是CompletableFuture类型，或者使用RpcContext.startAsync（）方法开启异步执行，返回AsyncRpcResult。

​	在代码3.2中，如果发现结果为AsyncRpcResult，则说明是服务提供方的异步执行，此时返回CompletableFuture对象，否则为同步执行，并把结果转换为CompletableFuture对象（同步转异步）。所以，代码3返回的肯定是CompletableFuture对象，无论是同步执行还是异步执行。

​	在接收到CompletableFuture对象后，代码3判断请求处理是否完成。如果请求已经完成，则执行代码4来设置结果并写回到调用方，否则执行代码5，即调用CompletableFuture来设置一个回调，等请求处理完毕后使用业务自己设置的线程池来执行回调，在回调函数内把结果或者错误信息写回调用方。

### 使用AsyncContext实现异步执行

​	在基础篇的Demo中，GrettingServiceAsyncContextImpl使用了AsyncContext实现异步执行，代码如下：

![](https://pic.imgdb.cn/item/6115ea6e5132923bf86e9abb.jpg)

![](https://pic.imgdb.cn/item/6115ea795132923bf86eabaf.jpg)

​	代码2.1调用RpcContext.startAsync（）方法开启服务异步执行，然后返回一个asyncContext，然后把服务处理任务提交到业务线程池后该方法就直接返回了null。在异步任务内，首先执行代码2.2切换任务的上下文，然后休眠500ms充当任务执行，最后代码2.3把任务执行结果写入异步上下文，其实现是参考了Servlet 3.0的异步执行。

​	在这里，由于具体执行业务处理的逻辑不在sayHello（）方法所在的Dubbo内部线程池的线程中，所以不会被阻塞。

​	为了探究其原理，我们先看看RpcContext.startAsync（）方法：

![](https://pic.imgdb.cn/item/6115ea975132923bf86ee033.jpg)

​	上面这些代码的主要作用是为当前调用线程关联的RPC上下文对象关联AsyncContextImpl。AsyncContextImpl构造函数如下：



![](https://pic.imgdb.cn/item/6115eaac5132923bf86f0478.jpg)

​	这里把当前线程上下文对象保存到了AsyncContextImpl内部（这是因为ThreadLocal变量不能跨线程访问，可以参考《Java并发编程之美》一书）。

​	AsyncContextImpl创建完毕后会被启动，其中AsyncContextImpl的start（）方法为：

![](https://pic.imgdb.cn/item/6115eabf5132923bf86f2342.jpg)

​	通过上面的代码可知，这里是为AsyncContextImpl内的future对象创建一个CompletableFuture对象，其中started是原子性boolean变量，为的是避免重复创建CompletableFuture。

​	下面我们看看AsyncContextImpl的signalContextSwitch（）方法，该方法用来将保存在AsyncContextImpl内的上下文信息传递到业务线程池的线程中（也就是说业务线程池中的线程可以通过RpcContext来访问）：

![](https://pic.imgdb.cn/item/6115eae35132923bf86f625c.jpg)

​	需要注意的是，signalContextSwitch（）方法需要放在业务线程中的第一句来执行，以避免后面的业务处理使用RpcContext获取上下文信息时出错。

​	下面我们再看看AsyncContextImpl的write（）方法：

![](https://pic.imgdb.cn/item/6115eafc5132923bf86f8d93.jpg)

![](https://pic.imgdb.cn/item/6115eb065132923bf86f9e5b.jpg)

​	在上面的代码中，当业务线程中的服务处理完毕，会把执行结果写入start（）方法创建的CompletableFuture对象内。

​	简单总结一下：当Dubbo线程模型线程池中的线程执行sayHello（）方法时，在方法内通过RpcContext.startAsync（）创建一个AsyncContextImpl实例，接着调用其start（）方法创建一个CompletableFuture对象。然后，在sayHello（）方法把业务处理任务添加到线程池之后，直接返回null。接着，在返回null后，结合上一节在AbstractProxyInvoker的invoke（）方法内代码6.1也返回了null，然后代码6.2通过判断返回结果用RpcContext.startAsync（）开启异步执行，所以使用（（AsyncContextImpl）（rpcContext.getAsyncContext（）））.getInternalFuture（）获取AsyncContextImpl内的future对象，该future对象会被返回给上一节的代码3.1，然后代码3.2把future返回给上一节的代码3，后面的逻辑就与上一节一样了。

## 异步调用与执行引入的新问题

### Filter链

​	在2.7.0版本之前，在消费端采用异步调用后，由于异步结果在异步线程（Dubbo框架线程模型线程池中的线程）中单独执行，因此DubboInvoker的invoke（）方法在发起远端请求后，会将空的RpcResult对象返回Filter调用链，也就是说，Filter链上的所有Filter获取的远端调用结果都是null，最终null值也直接返回给调用方法。而真正的远端调用结果需要调用方从RpcContext获取future对象来获取，当真正远端结果返回时，已经不会再次走Filter链进行处理了。

​	在2.7.0版本之前，异步调用时序图如图11.5所示。

![](https://pic.imgdb.cn/item/611726eb5132923bf84e0922.jpg)

​	从图11.5可知，在2.7.0版本之前，服务消费端的调用在发起远程调用DubboInvoker的invoke（）方法前进入Filter链，如果是异步调用，则DubboInvoker的invoke（）方法会创建一个DefaultFuture对象并设置到RpcContext中，接着返回一个空的RpcResult，然后该RpcResult会一层层返回到Filter链中的Filter，最终返回到业务调用的sayHello（）方法，得到结果null。

​	当调用方要获取真正的响应结果时，则需要首先从RpcContext获取future对象，然后调用其get（）方法进行等待。当服务提供方把服务处理结果写回调用方后，调用方I/O处理线程会把结果传递给调用方线程模型中的线程，从而把结果写入future对象，接着调用方线程就会从future的get（）方法返回，以获取真正的执行结果。可以看到，当真正结果回来时并没有再次执行Filter链。

​	为了解决这个问题，在2.7.0版本中为Filter接口增加了回调接口onResponse：

```java
public interface BaseFilter {
    /**
     * Always call invoker.invoke() in the implementation to hand over the request to the next filter node.
     */
    Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException;

    interface Listener {

        void onResponse(Result appResponse, Invoker<?> invoker, Invocation invocation);

        void onError(Throwable t, Invoker<?> invoker, Invocation invocation);
    }
}
```

​	由此可知，所有实现了Filter扩展接口的实现类都会含有这个默认的onResponse（）方法，并且默认返回传递过来的result。如果有需要的话，实现类可以重写onResponse（）方法，比如FutureFilter：

```java
public class FutureFilter implements ClusterFilter, ClusterFilter.Listener {
}
```

​	在2.7.0版本中，在异步调用后，DubboInvoker的invoke（）方法返回的是SimpleAsyncRpcResult对象而不再是RpcResult，前者继承了AsyncRpcResult类，并且其中含有远程调用返回的DefaultFuture。

​	前面我们讲到，Filter链是在ProtocolFilterWrapper的buildInvokerChain（）方法中建立并使用Invoker串联的。在2.7.0版本之前，Invoker的invoke（）方法如下：

![](https://pic.imgdb.cn/item/611728de5132923bf855e758.jpg)

​	由上面的代码可知，Filter链中的前一个Filter的结果会直接返回给后一个，而在2.7.0版本中，其代码被修改如下：

![](https://pic.imgdb.cn/item/611729865132923bf8588830.jpg)

![](https://pic.imgdb.cn/item/611729905132923bf858b40e.jpg)

​	在Dubbo 2.7.0中，消费端同步调用会执行代码2，同步调用对应Filter的onResponse（）方法。异步调用则会执行代码1，上面提到了这里的result实际上是SimpleAsyncRpcResult，下面我们看看它的thenApplyWithContext（）方法：

![](https://pic.imgdb.cn/item/611729a65132923bf85909c9.jpg)

​	这里的resultFuture就是DubboInvoker中的invoke（）方法在执行异步调用返回后创建的FutureAdapter中的resultFuture，它是CompletableFuture对象。

​	其中，函数指针beforeContext的定义为：

![](https://pic.imgdb.cn/item/611729c75132923bf8598ddf.jpg)

![](https://pic.imgdb.cn/item/611729d15132923bf859b70c.jpg)

​	其中，函数指针afterContext的定义为：

![](https://pic.imgdb.cn/item/611729e95132923bf85a18e0.jpg)

​	thenApplyWithContext（）函数的作用是等得到响应结果后首先执行beforeContext（）函数，以恢复发起远程调用时线程对应的上下文对象，并保存响应线程对应的上下文对象，然后执行传递的fn（）方法，也就是执行filter.onResponse（r，invoker，invocation）来调用当前Filter的onResponse（）方法，最后再调用afterContext（），以恢复保存的响应线程中的上下文。

​	另外，上面的FutureFilter也是同样的逻辑。经过上面的改造后，异步调用流程是这样的：当服务提供端把结果写回调用方后，调用方的线程模型中的线程会把结果写入CompletableFuture对象，接着Filter链中的Filter会执行自己的onResponse（）方法，然后把结果以新的CompletableFuture对象形式返回给Filter链上的一个节点，最后把最终结果写入业务调用方调用的get（）方法的future对象内，这样就获得最终调用结果了。

​	同理，服务提供端异步执行也存在这样的问题，解决方法也是一样的，在此就不再赘述了。

### 上下文对象传递

​	在服务提供端使用异步调用之前，业务可以直接在服务提供方服务代码内使用RpcContext.getContext（）获取上下文对象，进而可以访问其中保存的内容：

![](https://pic.imgdb.cn/item/61172db95132923bf8698915.jpg)

​	但在服务提供端使用异步执行后，由于真正执行服务处理的是另外的线程，而RpcContext内部的ThreadLocal变量是不能跨线程访问的，因此在启动异步执行后，需要业务先保存上下文对象，然后在业务线程里再访问：

![](https://pic.imgdb.cn/item/611736695132923bf88d3da5.jpg)

​	由上面的代码可知，需要先使用代码2.1保存当前线程中的上下文对象，然后在业务线程中，使用代码2.2访问保存的上下文对象中的属性。

​	如果使用了AsyncContext方式的异步执行，则可以方便地使用signalContextSwitch（）方法来实现Context的切换：

![](https://pic.imgdb.cn/item/611736d05132923bf88ee2f0.jpg)

![](https://pic.imgdb.cn/item/611736e35132923bf88f3836.jpg)

​	代码2.2使用signalContextSwitch（）方法实现上下文的切换，然后在业务线程中就可以使用代码2.3的RpcContext.getContext（）方法获取上下文了。

# 本地服务暴露与引用原理

​	前面的第3章我们讲解了远程服务暴露与引用的流程，这种方式是提供方与消费方通过网络来进行通信的，如图12.1所示。

![](https://pic.imgdb.cn/item/61173cdb5132923bf8a78ada.jpg)

​	其实Dubbo还提供了一种本地服务暴露与引用的方式，在同一个JVM进程中同时发布与调用同一个服务时，这种方式显得比较重要，因为如果当前JVM内要调用的服务在本JVM进程内已有，则避免了一次远程过程调用，而是直接在JVM内进行通信，如图12.2所示。

![](https://pic.imgdb.cn/item/61173cee5132923bf8a7d98b.jpg)

## 本地服务暴露流程

​	在3.1节我们提到，本地导出使用了injvm协议，这是一个伪协议，它不开启端口，不发起远程调用，只在JVM内直接关联，但执行Dubbo的Filter链。

​	首先，我们通过时序图看看本地服务暴露流程，如图12.3所示。

![](https://pic.imgdb.cn/item/61173d7a5132923bf8aa29a6.jpg)

​	我们看看ServiceConfig的exportLocal方法是怎么导出服务的：

​	![](https://pic.imgdb.cn/item/611743d45132923bf8c421a8.jpg)

​	在代码1中，如果发现协议类型不是injvm，则创建一个新的URL对象，并且设置协议为injvm，然后设置Host为127.0.0.1，端口为0。

​	后面就是与远程服务暴露一样的流程。首先，调用proxyFactory.getInvoker获取代理的对象，其中的proxyFactory也是适配器类，默认ProxyFactory的扩展实现是JavassistProxyFactory，所以这里调用JavassistProxyFactory的getInvoker（）方法来获取代理服务实现类的invoker类。

​	然后，调用protocol.export导出服务，其中的protocol也是扩展接口，这里与远程服务暴露不同之处在于协议类型为injvm，所以会加载扩展实现类InjvmProtocol。由图12.3可知，InjvmProtocol也使用Wrapper类被增强了，并且在ProtocolFilterWrapper内也建立了Filter链。

​	需要注意的是，在InjvmProtocol内有一个static类型的private static InjvmProtocolINSTANCE，而在InjvmProtocol的构造函数内把当前对象赋值给了INSTANCE：

![](https://pic.imgdb.cn/item/611744095132923bf8c4fd77.jpg)

​	另外，由于增强SPI内会缓存每个扩展接口实现的Class对象，所以在整个JVM内对于每个扩展接口来说只会存在一个InjvmProtocol的实例，并且其中INSTANCE保存的就是这个实例对象。

​	下面我们看看InjvmProtocol的export（）方法做了什么：

![](https://pic.imgdb.cn/item/6117443a5132923bf8c5c2e5.jpg)

​	这里的key为服务key，比如对服务com.books.dubbo.demo.api.GreetingService来说，其key为dubbo/com.books.dubbo.demo.api.GreetingService：1.0.0。

​	InjvmExporter构造函数把自己放入InjvmProtocol管理的exporterMap缓存中去了，这个很重要，后面会讲到相关内容。

## 本地服务引用启动流程

​	如果在消费端没有指定scope类型，则启动时会检查当前JVM内是否有导出的服务，如果有则自动开启本地引用（也就是将协议类型修改为injvm）。

​	下面我们看看当没有指定scope类型时，消费端的启动流程，如图12.4所示。

![](https://pic.imgdb.cn/item/611744c15132923bf8c802c7.jpg)

​	由图12.4可知，消费端启动时会调用createProxy（）方法来创建代理：

![](https://pic.imgdb.cn/item/611744d65132923bf8c86223.jpg)

​	其中，由函数shouldJvmRefer（）决定是开启本地引用还是远程引用，然后其内部会调用InjvmProtocol的isInjvmRefer（）方法来进行判断，后者又会调用其getExporter（）方法从InjvmProtocol的缓存中查看是否有本地暴露的服务：

![](https://pic.imgdb.cn/item/611744e95132923bf8c8b309.jpg)

​	前面讲过，对应每个扩展接口，在整个JVM内只会有一个InjvmProtocol的实例，并且上一节我们介绍了在本地服务导出时会把导出的服务保存到InjvmProtocol的内部缓存exporterMap中，所以在同一JVM内进行本地引用时，可以根据服务key从exporterMap中找到导出的invoker并返回，所以代码1中的shouldJvmRefer（）方法会返回true。

​	执行代码1将创建一个本地URL，其中Host为127.0.0.1，Port为0，协议为injvm，这些参数与本地服务导出时一致。

​	装配好URL之后，调用refprotocol.refer（interfaceClass，url）进行服务引用，其中refprotocol为扩展接口。由于协议类型为injvm，因此这里会返回SPI中缓存的InjvmProtocol对象，并且该对象也是使用Wrapper包装后的，所以会经过一层层后才调用到InjvmProtocol的refer（）方法（如图12.4所示）。其refer（）方法的代码如下：

![](https://pic.imgdb.cn/item/61174bcc5132923bf8e84b65.jpg)

​	由代码可知，该方法返回了一个InjvmInvoker对象，并且保存了InjvmProtocol管理的exporterMap（其中保存了导出的本地服务）。

## 本地服务一次引用流程

​	上面两节我们分别讲解了本地服务如何暴露、本地服务引用的启动流程，本节我们介绍一下发起一次本地引用调用的流程。这里以Provider模块中的APiConsumerInJvm为例画出时序图，如图12.5和图12.6所示。

![](https://pic.imgdb.cn/item/61174c375132923bf8ea5243.jpg)

![](https://pic.imgdb.cn/item/61174c425132923bf8ea8967.jpg)

​	如图12.5所示，在发起本地调用后，会进入调用方的Filter链，并最终调用InjvmInvoker的invoke（）方法，其内部又从JVM内唯一的InjvmProtocol实例里获取导出服务对应的InjvmExporter实例，按照图12.6所示的时序，先进入本地服务提供方的Filter链，然后最终调用代理拦截器AbstractProxyInvoker的invoke（）方法以调用服务实现类的对应方法。

# Dubbo协议与网络传输

​	在前面的章节中，我们介绍了服务消费端远程服务调用流程与服务提供端远程服务处理流程，但还有一些东西是我们没有提到的，比如服务消费端如何把服务请求信息序列化为二进制数据、服务提供方如何把消费端发送的二进制数据反序列化为可识别的POJO对象、Dubbo的应用层协议是怎么样的，等等。本章我们就来介绍相关的内容。

## Dubbo协议

​	在TCP协议栈中，每层协议都有自己的协议报文格式，比如TCP是网络七层模型中的传输层，是TCP协议报文格式；在TCP上层是应用层（应用层协议常见的有HTTP协议等），Dubbo协议作为建立在TCP之上的一种应用层协议，自然也有自己的协议报文格式。Dubbo协议也是参考了TCP协议栈中的协议，协议内容由header和body两部分组成，其结构如图13.1所示。

![](https://pic.imgdb.cn/item/611753b85132923bf80f3420.jpg)

​	其中，协议头header格式如图13.2所示。

![](https://pic.imgdb.cn/item/611753c95132923bf80f8410.jpg)

​	由图13.2可知，header总包含了16字节的数据。其中，前两字节为魔数，类似Class类文件里的魔数，这里用来标识一个帧的开始，固定为0xdabb，第一字节固定为0xda，第二字节固定为0xbb。

​	紧跟其后的一字节是请求类型和序列化标记ID的组合结果：requst flag|serializationId。其中，高四位标示请求类型，其枚举值如下：

![](https://pic.imgdb.cn/item/611758355132923bf8255c5c.jpg)

​	低四位标示序列化方式，其枚举值如下：

![](https://pic.imgdb.cn/item/6117584a5132923bf825cbe4.jpg)

​	再后面的一字节是只在响应报文里才设置（在请求报文里不设置），用来标示响应的结果码，具体定义如下：

![](https://pic.imgdb.cn/item/6117587c5132923bf826cdb0.jpg)

​	其后的8字节是请求ID。

​	最后的4字节是body内容的大小，也就是指定在协议头header内容后的多少字节是协议body的内容。

## 服务消费方编码原理

​	在3.3节我们讲到，在消费端启动时，NettyCodecAdapter管理的编解码器被设置到Netty链接的Channel管线里。下面我们看看服务消费端如何对Dubbo协议内容进行编码。

​	另外，在3.4节我们讲到，消费端发起一次调用后，最终会通过DubboInvoker的doInvoke（）方法内的远程调用客户端对象currentClient的request（）方法把请求发送出去。当网络传输使用Netty时，实际上是把请求转换为任务并投递到了NettyClient对应的Channel管理的异步队列里，这样当前的业务线程就会返回了，Netty会使用I/O线程去异步地执行该任务，把请求通过TCP链接发送出去。Netty异步处理的写入时序图如图13.3所示。

![](https://pic.imgdb.cn/item/611758be5132923bf8281032.jpg)

​	在Netty中，每个Channel（NioSocketChannel）与NioEventLoopGroup中的某一个NioEventLoop固定关联，业务线程就是异步地把请求转换为任务，并写入与当前Channel关联的NioEventLoop内部管理的异步队列中，然后NioEventLoop关联的线程就会去异步执行任务，图13.3就是使用NioEventLoop关联的线程异步地把请求发送出去。

​	在图13.3中，NioEventLoop关联的线程会把请求任务进行传递，即传递给该Channel管理的管线中的每个Handler，其中的一个Handler就是编解码处理器，也就是图13.3中的InternalEncoder，它又把任务委托给DubboCodec对请求任务进行编码，编码完毕执行步骤11，让编码后的数据沿管线继续流转下去。这里我们主要看看DubboCodec是如何按照Dubbo协议对请求进行编码的。

​	先看看DubboCodec的子类ExchangeCodec的encode（）方法，其代码如下：

![](https://pic.imgdb.cn/item/611759235132923bf829f3da.jpg)

​	首先我们看encodeRequest如何对请求信息进行编码：

![](https://pic.imgdb.cn/item/6117596e5132923bf82b89f7.jpg)

![](https://pic.imgdb.cn/item/61175efc5132923bf8492f23.jpg)

​	代码1.1获取序列化扩展实现，默认为hession序列化方式。

​	代码1.2创建Dubbo协议扩展头字节数组，由于Dubbo协议的协议头部分为16字节，所以这里创建了16字节的byte数组。

​	代码1.3把魔数0xdabb写入协议头的前两字节。

​	代码1.4则把请求类型与序列化类型标记到协议头第3字节。

​	代码1.5把请求ID写入协议头的第5～12字节，由于是request类型，所以第4字节的响应码是不需要设置的。

​	代码1.6使用代码1.1获取的序列化方式对象数据部分进行编码，并把协议数据部分写入缓存（buffer）。

​	代码1.7刷新缓存。在代码1.8中，checkPayload检查协议数据部分是否超过了设置的大小，如果是则抛出异常：

![](https://pic.imgdb.cn/item/61175f305132923bf84a7ce3.jpg)

​	如果检查合法，则把协议数据部分的大小写入协议头的第12～16字节。

​	代码1.9则首先把缓存的写入下标移动到写入协议数据前的位置，然后把协议头写入缓存，这时缓存里存放了完整的Dubbo协议帧（协议头+协议数据体），最后把缓存的写入下标设置为写入Dubbo协议帧后面的位置。

​	其实，encodeResponse对响应信息进行编码与encodeRequest代码类似，不同之处在于，前者需要对协议头的第4字节写入响应类型：

![](https://pic.imgdb.cn/item/61175f465132923bf84b0906.jpg)

![](https://pic.imgdb.cn/item/61175f705132923bf84c1294.jpg)

## 服务发布方解码原理

​		大家都知道，在客户端与服务端进行网络通信时，客户端会通过socket把需要发送的内容序列化为二进制流后发送出去，接着二进制流通过网络流向服务器端，服务端接收到该请求后会解析该请求包，然后反序列化后对请求进行处理。这看似是一个很简单的过程，但细细想来却会发现没有那么简单。图13.4显示的是客户端与服务端交互的流程。

![](https://pic.imgdb.cn/item/61175fe55132923bf84f0b84.jpg)

​	由图13.4可知，在客户端发送数据时，实际是把数据写入TCP发送缓存里，如果发送的包的大小比TCP发送缓存的容量大，那么这个数据包就会被分成多个包，通过socket多次发送到服务端。而服务端获取数据是从接收缓存里获取的，假设服务端第一次从接收缓存里获取的数据是整个包的一部分，这时就产生了半包现象。半包不是说只收到了全包的一半，是说收到了全包的一部分。

​	服务器读取到半包数据后，会对读取的二进制流进行解析，一般情况下会把二进制流反序列化为对象，但由于服务器只读取了客户端序列化对象后的一部分，所以反序列化会报错。

​	同理，如果发送的数据包大小比TCP发送缓存的容量小，并且假设TCP缓存可以存放多个包，那么客户端和服务端的一次通信就可能传递了多个包，这时服务端就可能从接收缓存一下读取了多个包，这样就出现了粘包现象。由于服务端从接收缓存获取的二进制流是多个对象转换来的，所以在后续的反序列化时肯定也会出错。

​	其实，出现粘包和半包的原因是TCP层不知道上层业务的包的概念，它只是简单地传递流，所以需要上层的应用层协议来识别读取的数据是不是一个完整的包。

​	在3.1节我们提到，NettyServer启动时把NettyCodecAdapter内部管理的编解码器注入链接Channel的管线内。下面我们看看服务提供端是如何对Dubbo协议的内容进行解码的。

​	另外，在3.2节我们是直接从NettyServer的received（）方法开始讲起的，下面我们看看源头，也就是如何从Netty的NioEventLoop里从TCP缓存读取请求数据，然后经过解码操作后调用NettyServer的received（）方法的。首先我们看看如图13.5所示的时序图。

![](https://pic.imgdb.cn/item/611760735132923bf852c16b.jpg)

​	如前所述，在Netty中，每个Channel（NioSocketChannel）与NioEventLoopGroup中的某一个NioEventLoop固定关联，NettyServer会为每个接收的链接创建一个Channel对象，这个Channel对象也会与WorkerNioEventLoopGroup中的某一个NioEventLoop固定关联。另外，每个NioEventLoop会管理一个Selector对象和一个线程，线程会不断检查注册到该Selector对象上的Channel是否有读取事件，如果有，则从TCP缓存读取数据。

​	由图13.5可知，当有读取事件时，NioEventLoop关联的线程会从缓存读取数据，然后把数据传递给该Channel管理的管线中的handler，这里会把数据传递给NettyCodecAdapter的内部类InternalDecoder，下面我们看看它的channelRead（）方法：

![](https://pic.imgdb.cn/item/611760925132923bf8538fa7.jpg)

![](https://pic.imgdb.cn/item/611760c05132923bf854be14.jpg)

​	在上面的代码中，解码操作是在callDecode（）方法中完成的。callDecode（）方法的代码如下：

![](https://pic.imgdb.cn/item/611761b15132923bf85b2063.jpg)

​	其中，decodeRemovalReentryProtection内部调用decode（）方法，后者的代码如下：

![](https://pic.imgdb.cn/item/611762245132923bf85e3494.jpg)

![](https://pic.imgdb.cn/item/611762335132923bf85ea2e6.jpg)

​	代码0.1先将message缓存的当前的读取下标保存到变量saveReaderIndex中，然后代码0.1执行具体的解码操作。如果代码0.1返回了NEED_MORE_INPUT，则说明遇到了半包，此时将message的读取下标重置为保存的saveReaderIndex，然后退出循环。否则，执行代码0.3，说明解析出来了一个完整的Dubbo协议帧，并保存到out列表，然后如果message还可读（有数据可以读取），则再次循环执行代码0.1，直到没有数据可读取，就退出循环。

​	这里，我们具体看看codec.decode（DubboCodec的decode（）方法）的代码：

![](https://pic.imgdb.cn/item/611762515132923bf85f6d2b.jpg)![](https://pic.imgdb.cn/item/6117625b5132923bf85fb321.jpg)

​	上面的代码先试图从缓存中读取Dubbo协议帧的协议头部分，然后调用decode（）方法解析协议的数据部分。其中decode（）方法的代码如下：

![](https://pic.imgdb.cn/item/611762705132923bf8603c53.jpg)

![](https://pic.imgdb.cn/item/6117627c5132923bf8609094.jpg)

​	代码1.1首先检查魔数以便确定当前数据是Dubbo协议的数据。

​	代码1.2判断是否读取到了一个完整的Dubbo协议帧头，如果不是，则返回NEED_MORE_INPUT。

​	代码1.3解析出Dubbo协议帧数据部分（body）的大小，检查body的大小是否超过了设置的数据包大小，如果超过了，则抛出ExceedPayloadLimitException异常。

​	代码1.4判断是否遇到了半包问题，如果是，则直接返回NEED_MORE_INPUT。

​	代码1.5解析协议帧数据部分。

​	在上面的代码中，如果返回了NEED_MORE_INPUT，则说明遇到了半包的情况（数据不足以支撑一个完整的Dubbo协议帧），此时通过循环继续累加数据直到有至少一个完整协议帧。下面我们看看decodeBody（）方法是如何解析Dubbo协议帧数据部分的：

​	在上面的代码中，如果返回了NEED_MORE_INPUT，则说明遇到了半包的情况（数据不足以支撑一个完整的Dubbo协议帧），此时通过循环继续累加数据直到有至少一个完整协议帧。下面我们看看decodeBody（）方法是如何解析Dubbo协议帧数据部分的：

![](https://pic.imgdb.cn/item/611762d45132923bf862f98e.jpg)

![](https://pic.imgdb.cn/item/611763245132923bf8653ab3.jpg)

![](https://pic.imgdb.cn/item/611763525132923bf8667bc9.jpg)

​	上面的代码比较简单，其流程与编码时的流程是相反的。值得注意的是，decode（）方法在过程中使用“自定义协议header+body”的方式来解决粘包、半包问题，其中header记录了body的大小，这种方式便于协议的升级。另外需要注意的是，在读取header后，message的读取指针已经后移了，如果后面发现出现了半包现象，则需要把读取指针重置。

# 实践篇

## Arthas的简介与安装

Arthas是Alibaba开源的Java诊断工具，深受开发者喜爱。如果遇到以下类似问题而束手无策时，Arthas可以帮助你解决：

* 这个类是从哪个Jar包加载的？为什么会报各种类相关的Exception？
* 我改的代码为什么没有执行到？难道是我没提交？分支搞错了？
* 遇到问题无法在线上调试，难道只能通过加日志再重新发布吗？
*  线上遇到某个用户的数据处理有问题，但线上同样无法调试，线下无法重现！
* 是否能够通过一个全局视角来查看系统的运行状况？
* 有什么办法可以监控到JVM的实时运行状态？



​	Arthas的github地址如下：

​	https://github.com/alibaba/arthas

​	安装Arthas第一步需要下载arthas-boot.jar：

```
wget https://alibaba.github.io/arthas/arthas-boot.jar
```

​	接着会列出所有Java进程，让你选择进入哪个进程，选择1，就会进入服务提供者进程：

![](https://pic.imgdb.cn/item/611767995132923bf889e05b.jpg)

​	选择1之后，就会关联到进程1224：

![](https://pic.imgdb.cn/item/611767a95132923bf88a74e5.jpg)

## 查看扩展接口适配器类的源码

​	在第2章，我们讲解了Dubbo的适配器原理，了解到每个扩展接口对应一个适配器类，并且这个适配器类是使用动态编译技术生成的，一般情况下只有使用Debug才能看到适配器的源码，但是使用Arthas我们就可以在服务启动的情况下查看某个适配器源码。

​	比如我们想查看扩展接口org.apache.dubbo.rpc.Protocol的适配器类源码，在启动Dubbo服务和Arthas后，在Arthas的控制台执行下面的命令即可：

![](https://pic.imgdb.cn/item/611767d25132923bf88bff33.jpg)

![](https://pic.imgdb.cn/item/611767de5132923bf88c7106.jpg)

![](https://pic.imgdb.cn/item/611767ed5132923bf88cf994.jpg)

​	同理，其他扩展接口的适配器类源码也可以使用类似的方法得到。

## 查看服务提供端Wrapper类的源码

​	在2.5.4小节我们讲到，Dubbo会给每个服务提供者的实现生成一个Wrapper类，在这个Wrapper类里最终是调用服务提供者的接口实现类，Wrapper类的存在是为了减少反射的调用。那么我们可以使用jad命令方便地查看被包装后的某一个服务实现类，以便探究它是如何工作的。

​	此，在启动Dubbo服务提供端和Arthas后，在Arthas的控制台执行下面的命令即可：

![](https://pic.imgdb.cn/item/6117682b5132923bf88f4576.jpg)

​	其他略

## 查询Dubbo启动后都有哪些Filter

​	查询Dubbo启动后都有哪些Filter

​	在Dubbo中，Filter链是一个亮点，通过Filter链可以对服务请求和服务处理流程进行干预，有时候我们想要知道运行时到底有哪些Filter在工作，这时使用Arthas的trace命令显得比较重要。

​	可以在启动Dubbo服务提供端和Arthas后，在Arthas的控制台执行如下命令：

![](https://pic.imgdb.cn/item/611768595132923bf890f9b6.jpg)

![](https://pic.imgdb.cn/item/611768785132923bf8921563.jpg)

![](https://pic.imgdb.cn/item/611768845132923bf8928b30.jpg)

​	由此可知，当前运行的服务提供端的Filter链中的Filter包括：

![](https://pic.imgdb.cn/item/6117689c5132923bf8936c91.jpg)

## Demo验证RoundRobin LoadBalance负载均衡原理

​	在6.9节，我们使用图表方式演示了RR负载均衡流程，这里我们再提供一个Demo来让大家简单地验证一下RR的效果。在Consumer模块中有一个org.apache.dubbo.demo.loadbalance包，其中有三个类，结构如图14.1所示。

![](https://pic.imgdb.cn/item/611768c65132923bf894f461.jpg)

​	其中，TestRoundRobinLoadBalance为我们的测试入口类，代码如下：

![](https://pic.imgdb.cn/item/611768dc5132923bf895c50c.jpg)

![](https://pic.imgdb.cn/item/611768ea5132923bf8964d52.jpg)

​	代码1模拟（mock）三个invoker，名称分别为A、B、C，并设置权重分别为1、2、3。代码2创建模拟（mock）的RoundRobin；代码3模拟远程调用，使用RoundRobin来做机器选择。运行上面代码后的输出为：

![](https://pic.imgdb.cn/item/611769085132923bf89761d1.jpg)

​	可知执行结果符合我们在高级篇讲解的原理，即6次调用一个循环；另外，如果把A、B、C权重设置成一样的，比如都为1，再运行上面代码，则结果为：

![](https://pic.imgdb.cn/item/611769165132923bf897f355.jpg)

## 如何动态获取Dubbo服务提供方地址列表

### 场景概述

​	Dubbo框架本身提供了丰富的负载均衡策略，比如轮询、随机、最少活跃调用数、一致性Hash等，但是有时候我们需要自己根据业务指定某个IP来进行调用。

​	那么什么时候需要指定IP来调用呢？我们考虑一个并行任务处理系统，系统接受一个大任务后会切割为若干子任务，然后把子任务分派到不同的机器上去执行，这时就需要把子任务路由到指定的IP上去运行，如图14.2所示。

![](https://pic.imgdb.cn/item/611769925132923bf89ca801.jpg)

​	从图14.2可以看到，任务切割与调度系统首先把SDK传递的任务分为子任务1到子任务6，然后把任务1和任务2分配到任务处理系统的机器1来执行，把任务3和任务4分配到任务处理系统的机器2来执行，把任务5和任务6分配到任务处理系统的机器3来执行。

​	要指定IP进行调用，需先知道服务提供者的IP。本节我们先来探讨第一步，看看当服务注册中心使用ZooKeeper时如何获取某一个服务提供端的地址列表。

### 原理与实现

​	我们知道，当服务提供方启动时，Dubbo会将服务注册到服务注册中心，这里我们使用的服务注册中心是ZooKeeper，比如服务com.books.dubbo.demo.api.GreetingService注册到ZooKeeper后，其结构是如图14.3所示的树形结构。

![](https://pic.imgdb.cn/item/611769c45132923bf89e9308.jpg)

​	当消费端启动时，Dubbo会去ZooKeeper上订阅/dubbo/com.books.dubbo.demo.api.GreetingService/providers节点下面的信息，也就是服务提供者列表信息。所以，我们可以基于这个原理来获取服务提供者列表，然后对信息进行过滤加工从而获取服务提供者地址列表，并且注册ZooKeeper的一个监听器，当服务提供者机器增减后，动态更新保存的地址列表。

​	基于上面原理的实现代码如下：

![](https://pic.imgdb.cn/item/611769e35132923bf89fbcf8.jpg)

![](https://pic.imgdb.cn/item/611769ef5132923bf8a030ae.jpg)

![](https://pic.imgdb.cn/item/611769fe5132923bf8a0bbd2.jpg)

![](https://pic.imgdb.cn/item/61176a085132923bf8a11eaa.jpg)

​	在上面的代码中，main（）函数创建了一个ZooKeeperIpList对象，并且调用其init（）方法，参数分别为ZooKeeper地址、ZooKeeper分组、服务接口以及版本、服务分组。

​	在ZookeeperIpList的init（）方法内，首先执行代码1进行参数校验，然后执行代码2拼接要订阅的ZookeeperIpList的path，拼接完成后dataid为/dubbo/com.books.dubbo.demo.api.GreetingService/providers，然后代码3创建zkClient订阅该dataid对应的path，并且注册监听器，当path下的信息变化后会得到最新的列表。另外，这里使用parseIpList（）方法解析获取的地址列表，解析完毕后保存到ipList中。

​	本节介绍的这个简单的演示，是基于ZooKeeper获取服务提供者地址列表的方法。该方法并不一定适合生产环境，大家如果想在生产环境中使用，需要进行量身改造，以免出现故障。

## 根据IP动态路由调用Dubbo服务

​	在Dubbo中，集群容错策略Cluster是SPI扩展接口，Dubbo框架提供了丰富的集群容错策略实现。集群容错策略会先从RegistryDirectory获取所有服务提供者的invoker列表，然后使用负载均衡策略从中选择一个inovker来发起远程调用。基于此原理，我们可以实现自己的集群容错策略，一开始还是使用RegistryDirectory获取所有服务提供者的invoker列表，但是不执行负载均衡策略，而是使用我们在发起远程调用前与指定的IP相匹配的invoker来进行发远程调用。至于如何在每次调用前指定IP，可以使用RpcContext.getContext（）.set（"ip"，"30.10.67.231"）来完成。

​	基于上述思路，本节我们就基于扩展接口实现指定IP调用的功能，首先我们实现扩展接口Cluster：

![](https://pic.imgdb.cn/item/61176a3c5132923bf8a30ae5.jpg)

​	然后，我们看看自己实现的MyClusterInvoker：

![](https://pic.imgdb.cn/item/61176a655132923bf8a49b00.jpg)

![](https://pic.imgdb.cn/item/61176a725132923bf8a51352.jpg)

​	在上面的代码1中，我们从RpcContext.getContext（）方法中获取了属性值IP，如果该值不为空，则说明指定了IP。

​	代码2检查是否有可用的服务提供者，如果没有则抛出异常。

​	代码3遍历invokers列表查找指定IP对应的invoker，如果没有指定IP对应的invoker，则抛出异常。

​	代码4使用选择的invoker发起远程调用。

​	注意，我们还将框架的AbstractClusterInvoker修改为MyAbstractClusterInvoker：

![](https://pic.imgdb.cn/item/61176ad55132923bf8a8d8a8.jpg)

​	这里我们把LoadBalanceloadbalance=initLoadBalance（invokers，invocation）；修改为LoadBalanceloadbalance=null；，因为我们不需要负载均衡了。

​	扩展实现写好后，要把扩展实现配置到如图14.4所示的文件中。

![](https://pic.imgdb.cn/item/61176af35132923bf8a9fadf.jpg)

​	然后，在消费端调用时进行如下设置就可以指定IP调用了。让我们看看Demo的Consumer模块的APiConsumerSetIpcall类：

![](https://pic.imgdb.cn/item/61176b135132923bf8ab3772.jpg)

​	上面的代码4创建ZookeeperIpList对象并初始化，代码5则调用ZookeeperIpList的getIpList（）方法获取所有服务提供者地址列表，然后轮询指定IP进行调用，也就是我们实现了动态IP指定路由调用，每次发起远程调用前，可以指定那一台服务提供方来提供服务。

​	本节只是简单演示了如何指定IP动态路由，并不一定适合生产环境，大家如果想在生产环境中使用，需要进行量身改造，以免出现故障。

​	这里我们回顾一下，Dubbo本身提供了服务直连功能，也就是在创建引用对象时可以指定一个IP进行调用以便点对点进行测试，需要注意的是，这个服务直连只能在服务启动前指定一次，不能在每次发起远程调用前动态改变。

​	另外，Dubbo本身提供了广播的集群容错策略，从而可以给所有的提供者并发发送请求，但是该策略在广播时给每个提供者传递的参数是一样的，不能够同时给不同服务提供者传递不同的参数；此外，其返回结果不是所有提供者返回结果的聚合结果，而是其中一个提供者的结果。基于上面两点，本节才基于业务需要指定IP动态路由调用服务。

## 基于CompletableFuture和Netty模拟RPC同步与纯异步调用

​	Dubbo的服务消费端基于CompletableFuture实现了纯异步调用，其实还不单单是CompletableFuture的功劳，归根到底是Netty的NIO非阻塞功能提供的底层实现，本节我们就基于CompletableFuture和Netty来模拟一下如何异步发起远程调用，以便加深对Dubbo异步调用实现原理的理解。

### 协议帧定义

![](https://pic.imgdb.cn/item/61176be95132923bf8b359c6.jpg)

​	在图14.5中，帧格式的第一部分为消息体，也就是业务需要传递的内容，第二部分为“：”，第三部分为请求ID，这里使用“：”把消息体与请求ID分开，以便服务端可以方便地提取出来这两部分内容，需要注意，消息体内不能含有“：”；第四部分“|”标识一个协议帧的结束，因为本Demo使用Netty的DelimiterBasedFrameDecoder来解决半包粘包问题。对比Dubbo的协议帧，本节的协议帧的定义就是一个演示。

### RpcServer的实现

​	首先我们基于Netty开发一个简单的Demo用来模拟RpcServer，也就是服务提供方程序。RpcServer的代码如下：

![](https://pic.imgdb.cn/item/61176c0e5132923bf8b4c758.jpg)

![](https://pic.imgdb.cn/item/61176c1a5132923bf8b5393b.jpg)

​	如上代码是一个典型的NettyServer启动程序，首先代码0创建了NettyServer的boss与worker线程池，然后代码1创建了业务NettyServerHandler（后面会具体讲解）；代码1.1将DelimiterBasedFrameDecoder解码器添加到链接Channel的管道，以便使用“|”分隔符来确定一个协议帧的边界（避免半包粘包问题）；代码1.2添加字符串解码器，这样的话在服务端链接Channel接收到客户端发来的消息后，自动把消息内容转换为字符串；代码1.3设置字符串编码器，用于在服务端链接Channel向客户端写入数据时，对数据进行编码；代码1.4将业务handler添加到管线。

​	代码2启动服务，并且在端口12800监听客户端发来的链接；代码3同步等待服务监听套接字关闭；代码4关闭两级线程池，以便释放线程。

​	这里我们主要看看业务Handler的实现，服务端在接收客户端消息并且消息内容经过代码1.1和代码1.2的Hanlder处理后，流转到NettyServerHandler的就是一个完整的协议帧的字符串了。NettyServerHandler的代码如下：

![](https://pic.imgdb.cn/item/61176c435132923bf8b6d346.jpg)

​	在上面的代码中，@Sharable注解是让服务端所有接收的链接对应的Channel复用同一个NettyServerHandler的实例。这里可以使用@Sharable方式，因为NettyServerHandler内的处理是无状态的，不会存在线程安全问题。

​	当数据流转到NettyServerHandler时，会调用其channelRead（）方法进行处理，这里的msg已经是一个完整的协议帧了。代码6为了及时释放Netty的I/O线程，把任务投递到AllChannelHandler管理的线程池内执行（这里仿照了Dubbo的All线程模型）。

​	在异步任务内，代码6.1接收消息体的内容，然后根据协议格式，从中截取出请求ID，然后调用代码6.2拼接返回给客户端的协议帧。需要注意的是，这里需要把请求ID带回去，然后休眠2s模拟服务端任务处理，最后代码6.3把拼接好的协议帧写回客户端。

​	其中，AllChannelHandler的代码如下：

![](https://pic.imgdb.cn/item/61176c605132923bf8b7f16c.jpg)

### RpcClient的实现

​	首先我们基于Netty开发一个简单的Demo用来模拟RpcClient，也就是服务消费方程序。RpcClient的代码如下：

![](https://pic.imgdb.cn/item/611771d75132923bf8f150e6.jpg)

![](https://pic.imgdb.cn/item/611771e45132923bf8f1d87d.jpg)

![](https://pic.imgdb.cn/item/611772095132923bf8f36924.jpg)

![](https://pic.imgdb.cn/item/611772175132923bf8f4000c.jpg)

​	在上面的代码中，RpcClient的构造函数创建了一个NettyClient，它与NettyServer类似。需要注意的是，这里将业务的NettyClientHandler处理器注册到链接Channel的管线里，并且在与服务端完成TCP三次握手后把对应的Channel对象保存了下来。

* rpcSyncCall（）方法：该方法意在模拟同步远程调用，其中代码1创建了一个CompletableFuture对象；代码2使用原子变量生成一个请求ID；代码3把业务传递的msg消息体和请求ID组成协议帧；代码4调用sendMsg（）方法通过保存的Channel对象把协议帧异步发送出去，该方法是非阻塞的，会马上返回，所以不会阻塞业务线程；代码5把代码1创建的future对象保存到FutureMapUtil管理的并发缓存中，其中key为请求ID，value为创建的future。FutureMapUtil就是管理并发缓存的一个工具类，其代码如下：

![](https://pic.imgdb.cn/item/6117724d5132923bf8f63807.jpg)

​	然后，代码6调用future的get（）方法，同步等待future的complete（）方法设置完结果，调用get（）方法会阻塞业务线程，直到future的结果被设置。

* rpcAsyncCall（）方法：我们看看rpcAsyncCall（）方法的异步调用，其代码实现与同步的rpcSyncCall（）方法类似，只不过其没有同步等待future的结果值，而是直接将future返回给调用方，然后就直接返回了，该方法不会阻塞业务线程。



​	至此，我们讲解了业务调用时远程调用的发起，下面我们看看服务端将结果写回客户端后，客户端如何把接入写回对应的future。为此我们需要看以一下注册的NettyClientHandler，其代码如下：

![](https://pic.imgdb.cn/item/611772705132923bf8f7aeb7.jpg)

​	在上面的代码中，当NettyClientHandler的channelRead（）方法被调用时，其中msg已经是一个完整的协议帧了（因为DelimiterBasedFrameDecoder与StringDecoder已经做过解析），这里的channelRead（）方法把对msg消息的处理提交到AllChannelHandler管理的线程池，以便及时释放Netty的I/O线程。

​	在异步任务内，代码1首先根据协议帧格式，从消息msg内获取到请求ID，然后从FutureMapUtil管理的缓存内获取请求ID对应的future对象并移除；如果请求ID存在，代码2则从协议帧内获取服务端写回的数据，并调用future的complete（）方法把结果设置到future，这时由于调用future的get（）方法而被阻塞的线程就返回结果了。

### 实例

​	前面我们讲解了RpcClient与RpcServer的实现，下面我们通过两个例子看看具体如何完成。首先我们看看TestModelAsyncRpc类的代码：

![](https://pic.imgdb.cn/item/6117728c5132923bf8f8d43b.jpg)

​	在上面的代码中，在main（）函数内首先创建了一个rpcClient对象，然后代码1同步调用了其rpcSyncCall（）方法，由于是同步调用，因此在服务端执行返回结果前，当前的调用线程会被阻塞，直到服务端把结果写回客户端，客户端把结果写回对应的future对象后才会返回。

​	代码2调用了异步方法rpcAsyncCall（），其不会阻塞业务调用线程，而是马上返回一个CompletableFuture对象，然后我们在其上设置了一个回调函数，意在等future对象的结果被设置后进行回调，这就实现了真正意义上的异步。

​	我们再通过一个实例，演示一下如何基于CompletableFuture的能力，并发发起多次调用，然后对返回的多个CompletableFuture进行运算，为此我们看看TestModelAsyncRpc2类的代码：

![](https://pic.imgdb.cn/item/611772ac5132923bf8fa20e7.jpg)

​	代码1首先发起一次远程调用，该调用马上返回future1；然后代码2又发起一次远程调用，该调用也马上返回future2对象；代码3则基于CompletableFuture的能力，意在让future1和fuuture2都有结果后再基于两者的结果做一件事情（这里是拼接两者返回的结果），并返回一个获取回调结果的新future。

​	代码4基于新的future，在其结果产生后，执行新的回调函数，进行结果打印或者异常打印。

​	至此，我们基于CompletableFuture和Netty模拟RPC同步与纯异步调用的内容已经讲完了，相信大家通过这个模拟对Dubbo的异步调用实现会有一定的理解。如果大家对CompletableFuture不太熟悉，可以等待笔者的《Java并发编程之美》修订版上市，修订版里会新增一章专门讲解各种Future的使用方法及其原理。

# 《个人源码阅读》

# 框架设计

https://dubbo.apache.org/zh/docs/v2.7/dev/design/

# SPI机制

https://dubbo.apache.org/zh/docs/v2.7/dev/source/dubbo-spi/

  https://www.processon.com/diagraming/61179781e0b34d0b1a57b97a

![](https://pic.imgdb.cn/item/611798a65132923bf8fe37a9.jpg)

```java
public T getExtension(String name) {
    if (name == null || name.length() == 0)
        throw new IllegalArgumentException("Extension name == null");
    if ("true".equals(name)) {
        // 获取默认的拓展实现类
        return getDefaultExtension();
    }
    // Holder，顾名思义，用于持有目标对象
    Holder<Object> holder = cachedInstances.get(name);
    if (holder == null) {
        cachedInstances.putIfAbsent(name, new Holder<Object>());
        holder = cachedInstances.get(name);
    }
    Object instance = holder.get();
    // 双重检查
    if (instance == null) {
        synchronized (holder) {
            instance = holder.get();
            if (instance == null) {
                // 创建拓展实例
                instance = createExtension(name);
                // 设置实例到 holder 中
                holder.set(instance);
            }
        }
    }
    return (T) instance;
}
```

```java
private T createExtension(String name) {
    // 从配置文件中加载所有的拓展类，可得到“配置项名称”到“配置类”的映射关系表
    Class<?> clazz = getExtensionClasses().get(name);
    if (clazz == null) {
        throw findException(name);
    }
    try {
        T instance = (T) EXTENSION_INSTANCES.get(clazz);
        if (instance == null) {
            // 通过反射创建实例
            EXTENSION_INSTANCES.putIfAbsent(clazz, clazz.newInstance());
            instance = (T) EXTENSION_INSTANCES.get(clazz);
        }
        // 向实例中注入依赖
        injectExtension(instance);
        Set<Class<?>> wrapperClasses = cachedWrapperClasses;
        if (wrapperClasses != null && !wrapperClasses.isEmpty()) {
            // 循环创建 Wrapper 实例
            for (Class<?> wrapperClass : wrapperClasses) {
                // 将当前 instance 作为参数传给 Wrapper 的构造方法，并通过反射创建 Wrapper 实例。
                // 然后向 Wrapper 实例中注入依赖，最后将 Wrapper 实例再次赋值给 instance 变量
                instance = injectExtension(
                    (T) wrapperClass.getConstructor(type).newInstance(instance));
            }
        }
        return instance;
    } catch (Throwable t) {
        throw new IllegalStateException("...");
    }
}
```

createExtension 方法的逻辑稍复杂一下，包含了如下的步骤：

1. 通过 getExtensionClasses 获取所有的拓展类
2. 通过反射创建拓展对象
3. 向拓展对象中注入依赖
4. 将拓展对象包裹在相应的 Wrapper 对象中

## 获取扩展类

加载扩展类代码

```java
private Map<String, Class<?>> getExtensionClasses() {
    // 先从缓存获取
    Map<String, Class<?>> classes = cachedClasses.get();
    if (classes == null) {
        // 双重校验
        synchronized (cachedClasses) {
            classes = cachedClasses.get();
            if (classes == null) {
                // 缓存没有 从文件加载
                classes = loadExtensionClasses();
                cachedClasses.set(classes);
            }
        }
    }
    return classes;
}
private Map<String, Class<?>> loadExtensionClasses() {
        cacheDefaultExtensionName();

        Map<String, Class<?>> extensionClasses = new HashMap<>();
		// 从不同的目录加载类
        for (LoadingStrategy strategy : strategies) {
            loadDirectory(extensionClasses, strategy.directory(), type.getName(), strategy.preferExtensionClassLoader(), strategy.overridden(), strategy.excludedPackages());
            loadDirectory(extensionClasses, strategy.directory(), type.getName().replace("org.apache", "com.alibaba"), strategy.preferExtensionClassLoader(), strategy.overridden(), strategy.excludedPackages());
        }

        return extensionClasses;
    }
```

从文件夹加载

```java
private void loadDirectory(Map<String, Class<?>> extensionClasses, String dir, String type,
                           boolean extensionLoaderClassLoaderFirst, boolean overridden, String... excludedPackages) {
    String fileName = dir + type;
    try {
        Enumeration<java.net.URL> urls = null;
        // 找到ClassLoader
        ClassLoader classLoader = findClassLoader();

        // try to load from ExtensionLoader's ClassLoader first
        if (extensionLoaderClassLoaderFirst) {
            ClassLoader extensionLoaderClassLoader = ExtensionLoader.class.getClassLoader();
            if (ClassLoader.getSystemClassLoader() != extensionLoaderClassLoader) {
                urls = extensionLoaderClassLoader.getResources(fileName);
            }
        }

        if (urls == null || !urls.hasMoreElements()) {
            if (classLoader != null) {
                urls = classLoader.getResources(fileName);
            } else {
                urls = ClassLoader.getSystemResources(fileName);
            }
        }

        if (urls != null) {
            while (urls.hasMoreElements()) {
                java.net.URL resourceURL = urls.nextElement();
                loadResource(extensionClasses, classLoader, resourceURL, overridden, excludedPackages);
            }
        }
    } catch (Throwable t) {
        logger.error("Exception occurred when loading extension class (interface: " +
                type + ", description file: " + fileName + ").", t);
    }
}
```

加载资源

```java
private void loadResource(Map<String, Class<?>> extensionClasses, ClassLoader classLoader,
                          java.net.URL resourceURL, boolean overridden, String... excludedPackages) {
    try {
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(resourceURL.openStream(), StandardCharsets.UTF_8))) {
            String line;
            String clazz = null;
            while ((line = reader.readLine()) != null) {
                final int ci = line.indexOf('#');
                if (ci >= 0) {
                    line = line.substring(0, ci);
                }
                line = line.trim();
                if (line.length() > 0) {
                    try {
                        String name = null;
                        int i = line.indexOf('=');
                        if (i > 0) {
                            name = line.substring(0, i).trim();
                            clazz = line.substring(i + 1).trim();
                        } else {
                            clazz = line;
                        }
                        if (StringUtils.isNotEmpty(clazz) && !isExcluded(clazz, excludedPackages)) {
                            loadClass(extensionClasses, resourceURL, Class.forName(clazz, true, classLoader), name, overridden);
                        }
                    } catch (Throwable t) {
                        IllegalStateException e = new IllegalStateException("Failed to load extension class (interface: " + type + ", class line: " + line + ") in " + resourceURL + ", cause: " + t.getMessage(), t);
                        exceptions.put(line, e);
                    }
                }
            }
        }
    } catch (Throwable t) {
        logger.error("Exception occurred when loading extension class (interface: " +
                type + ", class file: " + resourceURL + ") in " + resourceURL, t);
    }
}
```

加载类

```java
private void loadClass(Map<String, Class<?>> extensionClasses, java.net.URL resourceURL, Class<?> clazz, String name,
                       boolean overridden) throws NoSuchMethodException {
    if (!type.isAssignableFrom(clazz)) {
        throw new IllegalStateException("Error occurred when loading extension class (interface: " +
                type + ", class line: " + clazz.getName() + "), class "
                + clazz.getName() + " is not subtype of interface.");
    }
    // Adaptive注解
    if (clazz.isAnnotationPresent(Adaptive.class)) {
        cacheAdaptiveClass(clazz, overridden);
    } else if (isWrapperClass(clazz)) {
        // Wrapper class
        cacheWrapperClass(clazz);
    } else {
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
```

## Dubbo IOC

​	是否解决循环注入问题

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

# SPI 自适应拓展

https://dubbo.apache.org/zh/docs/v2.7/dev/source/adaptive-extension/

​	在对自适应拓展生成过程进行深入分析之前，我们先来看一下与自适应拓展息息相关的一个注解，即 Adaptive 注解。该注解的定义如下：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Adaptive {
    String[] value() default {};
}
```

​	从上面的代码中可知，Adaptive 可注解在类或方法上。当 Adaptive 注解在类上时，Dubbo 不会为该类生成代理类。注解在方法（接口方法）上时，Dubbo 则会为该方法生成代理逻辑。Adaptive 注解在类上的情况很少，在 Dubbo 中，仅有两个类被 Adaptive 注解了，分别是 AdaptiveCompiler 和 AdaptiveExtensionFactory。此种情况，表示拓展的加载逻辑由人工编码完成。更多时候，Adaptive 是注解在接口方法上的，表示拓展的加载逻辑需由框架自动生成。Adaptive 注解的地方不同，相应的处理逻辑也是不同的。注解在类上时，处理逻辑比较简单，本文就不分析了。注解在接口方法上时，处理逻辑较为复杂，本章将会重点分析此块逻辑。

## 获取自适应拓展

​	getAdaptiveExtension 方法是获取自适应拓展的入口方法，因此下面我们从这个方法进行分析。相关代码如下：

```java
public T getAdaptiveExtension() {
    // 从缓存中获取自适应拓展
    Object instance = cachedAdaptiveInstance.get();
    if (instance == null) {    // 缓存未命中
        if (createAdaptiveInstanceError == null) {
            synchronized (cachedAdaptiveInstance) {
                instance = cachedAdaptiveInstance.get();
                if (instance == null) {
                    try {
                        // 创建自适应拓展
                        instance = createAdaptiveExtension();
                        // 设置自适应拓展到缓存中
                        cachedAdaptiveInstance.set(instance);
                    } catch (Throwable t) {
                        createAdaptiveInstanceError = t;
                        throw new IllegalStateException("fail to create adaptive instance: ...");
                    }
                }
            }
        } else {
            throw new IllegalStateException("fail to create adaptive instance:  ...");
        }
    }

    return (T) instance;
}
```

​	getAdaptiveExtension 方法首先会检查缓存，缓存未命中，则调用 createAdaptiveExtension 方法创建自适应拓展。下面，我们看一下 createAdaptiveExtension 方法的代码。

```java
private T createAdaptiveExtension() {
    try {
        // 获取自适应拓展类，并通过反射实例化
        return injectExtension((T) getAdaptiveExtensionClass().newInstance());
    } catch (Exception e) {
        throw new IllegalStateException("Can't create adaptive extension " + type + ", cause: " + e.getMessage(), e);
    }
}
```

createAdaptiveExtension 方法的代码比较少，但却包含了三个逻辑，分别如下：

1. 调用 getAdaptiveExtensionClass 方法获取自适应拓展 Class 对象
2. 通过反射进行实例化
3. 调用 injectExtension 方法向拓展实例中注入依赖



​	前两个逻辑比较好理解，第三个逻辑用于向自适应拓展对象中注入依赖。这个逻辑看似多余，但有存在的必要，这里简单说明一下。前面说过，Dubbo 中有两种类型的自适应拓展，一种是手工编码的，一种是自动生成的。手工编码的自适应拓展中可能存在着一些依赖，而自动生成的 Adaptive 拓展则不会依赖其他类。这里调用 injectExtension 方法的目的是为手工编码的自适应拓展注入依赖，这一点需要大家注意一下。关于 injectExtension 方法，前文已经分析过了，这里不再赘述。接下来，分析 getAdaptiveExtensionClass 方法的逻辑。

```javajava
private Class<?> getAdaptiveExtensionClass() {
    // 通过 SPI 获取所有的拓展类
    getExtensionClasses();
    // 检查缓存，若缓存不为空，则直接返回缓存
    if (cachedAdaptiveClass != null) {
        return cachedAdaptiveClass;
    }
    // 创建自适应拓展类
    return cachedAdaptiveClass = createAdaptiveExtensionClass();
}
```

getAdaptiveExtensionClass 方法同样包含了三个逻辑，如下：

1. 调用 getExtensionClasses 获取所有的拓展类
2. 检查缓存，若缓存不为空，则返回缓存
3. 若缓存为空，则调用 createAdaptiveExtensionClass 创建自适应拓展类



​	这三个逻辑看起来平淡无奇，似乎没有多讲的必要。但是这些平淡无奇的代码中隐藏了着一些细节，需要说明一下。首先从第一个逻辑说起，getExtensionClasses 这个方法用于获取某个接口的所有实现类。比如该方法可以获取 Protocol 接口的 DubboProtocol、HttpProtocol、InjvmProtocol 等实现类。在获取实现类的过程中，如果某个实现类被 Adaptive 注解修饰了，那么该类就会被赋值给 cachedAdaptiveClass 变量。此时，上面步骤中的第二步条件成立（缓存不为空），直接返回 cachedAdaptiveClass 即可。如果所有的实现类均未被 Adaptive 注解修饰，那么执行第三步逻辑，创建自适应拓展类。相关代码如下：

```java
private Class<?> createAdaptiveExtensionClass() {
    String code = new AdaptiveClassCodeGenerator(type, cachedDefaultName).generate();
    ClassLoader classLoader = findClassLoader();
    org.apache.dubbo.common.compiler.Compiler compiler = ExtensionLoader.getExtensionLoader(org.apache.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();
    return compiler.compile(code, classLoader);
}
```

​	createAdaptiveExtensionClass 方法用于生成自适应拓展类，该方法首先会生成自适应拓展类的源码，然后通过 Compiler 实例（Dubbo 默认使用 javassist 作为编译器）编译源码，得到代理类 Class 实例。接下来，我们把重点放在代理类代码生成的逻辑上，其他逻辑大家自行分析。

## 自适应拓展类代码生成 2.7

### Adaptive 注解检测

​	在生成代理类源码之前，createAdaptiveExtensionClassCode 方法首先会通过反射检测接口方法是否包含 Adaptive 注解。对于要生成自适应拓展的接口，Dubbo 要求该接口至少有一个方法被 Adaptive 注解修饰。若不满足此条件，就会抛出运行时异常。相关代码如下：

```java
// 通过反射获取所有的方法
Method[] methods = type.getMethods();
boolean hasAdaptiveAnnotation = false;
// 遍历方法列表
for (Method m : methods) {
    // 检测方法上是否有 Adaptive 注解
    if (m.isAnnotationPresent(Adaptive.class)) {
        hasAdaptiveAnnotation = true;
        break;
    }
}

if (!hasAdaptiveAnnotation)
    // 若所有的方法上均无 Adaptive 注解，则抛出异常
    throw new IllegalStateException("No adaptive method on extension ...");
```

### 生成类

​	通过 Adaptive 注解检测后，即可开始生成代码。代码生成的顺序与 Java 文件内容顺序一致，首先会生成 package 语句，然后生成 import 语句，紧接着生成类名等代码。整个逻辑如下：

```java
// 生成 package 代码：package + type 所在包
codeBuilder.append("package ").append(type.getPackage().getName()).append(";");
// 生成 import 代码：import + ExtensionLoader 全限定名
codeBuilder.append("\nimport ").append(ExtensionLoader.class.getName()).append(";");
// 生成类代码：public class + type简单名称 + $Adaptive + implements + type全限定名 + {
codeBuilder.append("\npublic class ")
    .append(type.getSimpleName())
    .append("$Adaptive")
    .append(" implements ")
    .append(type.getCanonicalName())
    .append(" {");

// ${生成方法}

codeBuilder.append("\n}");
```

​	这里使用 ${…} 占位符代表其他代码的生成逻辑，该部分逻辑将在随后进行分析。上面代码不是很难理解，下面直接通过一个例子展示该段代码所生成的内容。以 Dubbo 的 Protocol 接口为例，生成的代码如下：

```java
package com.alibaba.dubbo.rpc;
import com.alibaba.dubbo.common.extension.ExtensionLoader;
public class Protocol$Adaptive implements com.alibaba.dubbo.rpc.Protocol {
    // 省略方法代码
}
```

### 生成方法

​	一个方法可以被 Adaptive 注解修饰，也可以不被修饰。这里将未被 Adaptive 注解修饰的方法称为“无 Adaptive 注解方法”，下面我们先来看看此种方法的代码生成逻辑是怎样的。

#### 无 Adaptive 注解方法代码生成逻辑

​	对于接口方法，我们可以按照需求标注 Adaptive 注解。以 Protocol 接口为例，该接口的 destroy 和 getDefaultPort 未标注 Adaptive 注解，其他方法均标注了 Adaptive 注解。Dubbo 不会为没有标注 Adaptive 注解的方法生成代理逻辑，对于该种类型的方法，仅会生成一句抛出异常的代码。生成逻辑如下：

```java
for (Method method : methods) {
    
    // 省略无关逻辑

    Adaptive adaptiveAnnotation = method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    // 如果方法上无 Adaptive 注解，则生成 throw new UnsupportedOperationException(...) 代码
    if (adaptiveAnnotation == null) {
        // 生成的代码格式如下：
        // throw new UnsupportedOperationException(
        //     "method " + 方法签名 + of interface + 全限定接口名 + is not adaptive method!”)
        code.append("throw new UnsupportedOperationException(\"method ")
            .append(method.toString()).append(" of interface ")
            .append(type.getName()).append(" is not adaptive method!\");");
    } else {
        // 省略无关逻辑
    }
    
    // 省略无关逻辑
}
```

​	以 Protocol 接口的 destroy 方法为例，上面代码生成的内容如下：

```java
throw new UnsupportedOperationException(
            "method public abstract void com.alibaba.dubbo.rpc.Protocol.destroy() of interface com.alibaba.dubbo.rpc.Protocol is not adaptive method!");
```

#### 获取 URL 数据

​	前面说过方法代理逻辑会从 URL 中提取目标拓展的名称，因此代码生成逻辑的一个重要的任务是从方法的参数列表或者其他参数中获取 URL 数据。举例说明一下，我们要为 Protocol 接口的 refer 和 export 方法生成代理逻辑。在运行时，通过反射得到的方法定义大致如下：

```java
Invoker refer(Class<T> arg0, URL arg1) throws RpcException;
Exporter export(Invoker<T> arg0) throws RpcException;
```

对于 refer 方法，通过遍历 refer 的参数列表即可获取 URL 数据，这个还比较简单。对于 export 方法，获取 URL 数据则要麻烦一些。export 参数列表中没有 URL 参数，因此需要从 Invoker 参数中获取 URL 数据。获取方式是调用 Invoker 中可返回 URL 的 getter 方法，比如 getUrl。如果 Invoker 中无相关 getter 方法，此时则会抛出异常。整个逻辑如下：

```java
for (Method method : methods) {
    Class<?> rt = method.getReturnType();
    Class<?>[] pts = method.getParameterTypes();
    Class<?>[] ets = method.getExceptionTypes();

    Adaptive adaptiveAnnotation = method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        // ${无 Adaptive 注解方法代码生成逻辑}
    } else {
    	int urlTypeIndex = -1;
        // 遍历参数列表，确定 URL 参数位置
        for (int i = 0; i < pts.length; ++i) {
            if (pts[i].equals(URL.class)) {
                urlTypeIndex = i;
                break;
            }
        }
        
        // urlTypeIndex != -1，表示参数列表中存在 URL 参数
        if (urlTypeIndex != -1) {
            // 为 URL 类型参数生成判空代码，格式如下：
            // if (arg + urlTypeIndex == null) 
            //     throw new IllegalArgumentException("url == null");
            String s = String.format("\nif (arg%d == null) throw new IllegalArgumentException(\"url == null\");",
                                     urlTypeIndex);
            code.append(s);

            // 为 URL 类型参数生成赋值代码，形如 URL url = arg1
            s = String.format("\n%s url = arg%d;", URL.class.getName(), urlTypeIndex);
            code.append(s);
            
        // 参数列表中不存在 URL 类型参数
        } else {
            String attribMethod = null;

            LBL_PTS:
            // 遍历方法的参数类型列表
            for (int i = 0; i < pts.length; ++i) {
                // 获取某一类型参数的全部方法
                Method[] ms = pts[i].getMethods();
                // 遍历方法列表，寻找可返回 URL 的 getter 方法
                for (Method m : ms) {
                    String name = m.getName();
                    // 1. 方法名以 get 开头，或方法名大于3个字符
                    // 2. 方法的访问权限为 public
                    // 3. 非静态方法
                    // 4. 方法参数数量为0
                    // 5. 方法返回值类型为 URL
                    if ((name.startsWith("get") || name.length() > 3)
                        && Modifier.isPublic(m.getModifiers())
                        && !Modifier.isStatic(m.getModifiers())
                        && m.getParameterTypes().length == 0
                        && m.getReturnType() == URL.class) {
                        urlTypeIndex = i;
                        attribMethod = name;
                        
                        // 结束 for (int i = 0; i < pts.length; ++i) 循环
                        break LBL_PTS;
                    }
                }
            }
            if (attribMethod == null) {
                // 如果所有参数中均不包含可返回 URL 的 getter 方法，则抛出异常
                throw new IllegalStateException("fail to create adaptive class for interface ...");
            }

            // 为可返回 URL 的参数生成判空代码，格式如下：
            // if (arg + urlTypeIndex == null) 
            //     throw new IllegalArgumentException("参数全限定名 + argument == null");
            String s = String.format("\nif (arg%d == null) throw new IllegalArgumentException(\"%s argument == null\");",
                                     urlTypeIndex, pts[urlTypeIndex].getName());
            code.append(s);

            // 为 getter 方法返回的 URL 生成判空代码，格式如下：
            // if (argN.getter方法名() == null) 
            //     throw new IllegalArgumentException(参数全限定名 + argument getUrl() == null);
            s = String.format("\nif (arg%d.%s() == null) throw new IllegalArgumentException(\"%s argument %s() == null\");",
                              urlTypeIndex, attribMethod, pts[urlTypeIndex].getName(), attribMethod);
            code.append(s);

            // 生成赋值语句，格式如下：
            // URL全限定名 url = argN.getter方法名()，比如 
            // com.alibaba.dubbo.common.URL url = invoker.getUrl();
            s = String.format("%s url = arg%d.%s();", URL.class.getName(), urlTypeIndex, attribMethod);
            code.append(s);
        }
        
        // 省略无关代码
    }
    
    // 省略无关代码
}
```

​		上面代码有点多，需要耐心看一下。这段代码主要目的是为了获取 URL 数据，并为之生成判空和赋值代码。以 Protocol 的 refer 和 export 方法为例，上面的代码为它们生成如下内容（代码已格式化）：

```java
refer:
if (arg1 == null) 
    throw new IllegalArgumentException("url == null");
com.alibaba.dubbo.common.URL url = arg1;

export:
if (arg0 == null) 
    throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument == null");
if (arg0.getUrl() == null) 
    throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument getUrl() == null");
com.alibaba.dubbo.common.URL url = arg0.getUrl();
```

#### 获取 Adaptive 注解值

​	Adaptive 注解值 value 类型为 String[]，可填写多个值，默认情况下为空数组。若 value 为非空数组，直接获取数组内容即可。若 value 为空数组，则需进行额外处理。处理过程是将类名转换为字符数组，然后遍历字符数组，并将字符放入 StringBuilder 中。若字符为大写字母，则向 StringBuilder 中添加点号，随后将字符变为小写存入 StringBuilder 中。比如 LoadBalance 经过处理后，得到 load.balance。

```java
for (Method method : methods) {
    Class<?> rt = method.getReturnType();
    Class<?>[] pts = method.getParameterTypes();
    Class<?>[] ets = method.getExceptionTypes();

    Adaptive adaptiveAnnotation = method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        // ${无 Adaptive 注解方法代码生成逻辑}
    } else {
        // ${获取 URL 数据}
        
        String[] value = adaptiveAnnotation.value();
        // value 为空数组
        if (value.length == 0) {
            // 获取类名，并将类名转换为字符数组
            char[] charArray = type.getSimpleName().toCharArray();
            StringBuilder sb = new StringBuilder(128);
            // 遍历字节数组
            for (int i = 0; i < charArray.length; i++) {
                // 检测当前字符是否为大写字母
                if (Character.isUpperCase(charArray[i])) {
                    if (i != 0) {
                        // 向 sb 中添加点号
                        sb.append(".");
                    }
                    // 将字符变为小写，并添加到 sb 中
                    sb.append(Character.toLowerCase(charArray[i]));
                } else {
                    // 添加字符到 sb 中
                    sb.append(charArray[i]);
                }
            }
            value = new String[]{sb.toString()};
        }
        
        // 省略无关代码
    }
    
    // 省略无关逻辑
}
```

#### 检测 Invocation 参数

​	此段逻辑是检测方法列表中是否存在 Invocation 类型的参数，若存在，则为其生成判空代码和其他一些代码。相应的逻辑如下：

```java
for (Method method : methods) {
    Class<?> rt = method.getReturnType();
    Class<?>[] pts = method.getParameterTypes();    // 获取参数类型列表
    Class<?>[] ets = method.getExceptionTypes();

    Adaptive adaptiveAnnotation = method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        // ${无 Adaptive 注解方法代码生成逻辑}
    } else {
        // ${获取 URL 数据}
        
        // ${获取 Adaptive 注解值}
        
        boolean hasInvocation = false;
        // 遍历参数类型列表
        for (int i = 0; i < pts.length; ++i) {
            // 判断当前参数名称是否等于 com.alibaba.dubbo.rpc.Invocation
            if (pts[i].getName().equals("com.alibaba.dubbo.rpc.Invocation")) {
                // 为 Invocation 类型参数生成判空代码
                String s = String.format("\nif (arg%d == null) throw new IllegalArgumentException(\"invocation == null\");", i);
                code.append(s);
                // 生成 getMethodName 方法调用代码，格式为：
                //    String methodName = argN.getMethodName();
                s = String.format("\nString methodName = arg%d.getMethodName();", i);
                code.append(s);
                
                // 设置 hasInvocation 为 true
                hasInvocation = true;
                break;
            }
        }
    }
    
    // 省略无关逻辑
}
```

####  生成拓展名获取逻辑

​	本段逻辑用于根据 SPI 和 Adaptive 注解值生成“获取拓展名逻辑”，同时生成逻辑也受 Invocation 类型参数影响，综合因素导致本段逻辑相对复杂。本段逻辑可能会生成但不限于下面的代码：

```java
String extName = (url.getProtocol() == null ? "dubbo" : url.getProtocol());
```

或

```java
String extName = url.getMethodParameter(methodName, "loadbalance", "random");
```

亦或是

```java
String extName = url.getParameter("client", url.getParameter("transporter", "netty"));
```

本段逻辑复杂之处在于条件分支比较多，大家在阅读源码时需要知道每个条件分支的意义是什么，否则不太容易看懂相关代码。下面开始分析本段逻辑。

```java
for (Method method : methods) {
    Class<?> rt = method.getReturnType();
    Class<?>[] pts = method.getParameterTypes();
    Class<?>[] ets = method.getExceptionTypes();

    Adaptive adaptiveAnnotation = method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        // $无 Adaptive 注解方法代码生成逻辑}
    } else {
        // ${获取 URL 数据}
        
        // ${获取 Adaptive 注解值}
        
        // ${检测 Invocation 参数}
        
        // 设置默认拓展名，cachedDefaultName 源于 SPI 注解值，默认情况下，
        // SPI 注解值为空串，此时 cachedDefaultName = null
        String defaultExtName = cachedDefaultName;
        String getNameCode = null;
        
        // 遍历 value，这里的 value 是 Adaptive 的注解值，2.2.3.3 节分析过 value 变量的获取过程。
        // 此处循环目的是生成从 URL 中获取拓展名的代码，生成的代码会赋值给 getNameCode 变量。注意这
        // 个循环的遍历顺序是由后向前遍历的。
        for (int i = value.length - 1; i >= 0; --i) {
            // 当 i 为最后一个元素的坐标时
            if (i == value.length - 1) {
                // 默认拓展名非空
                if (null != defaultExtName) {
                    // protocol 是 url 的一部分，可通过 getProtocol 方法获取，其他的则是从
                    // URL 参数中获取。因为获取方式不同，所以这里要判断 value[i] 是否为 protocol
                    if (!"protocol".equals(value[i]))
                    	// hasInvocation 用于标识方法参数列表中是否有 Invocation 类型参数
                        if (hasInvocation)
                            // 生成的代码功能等价于下面的代码：
                            //   url.getMethodParameter(methodName, value[i], defaultExtName)
                            // 以 LoadBalance 接口的 select 方法为例，最终生成的代码如下：
                            //   url.getMethodParameter(methodName, "loadbalance", "random")
                            getNameCode = String.format("url.getMethodParameter(methodName, \"%s\", \"%s\")", value[i], defaultExtName);
                    	else
                    		// 生成的代码功能等价于下面的代码：
	                        //   url.getParameter(value[i], defaultExtName)
	                        getNameCode = String.format("url.getParameter(\"%s\", \"%s\")", value[i], defaultExtName);
                    else
                    	// 生成的代码功能等价于下面的代码：
                        //   ( url.getProtocol() == null ? defaultExtName : url.getProtocol() )
                        getNameCode = String.format("( url.getProtocol() == null ? \"%s\" : url.getProtocol() )", defaultExtName);
                    
                // 默认拓展名为空
                } else {
                    if (!"protocol".equals(value[i]))
                        if (hasInvocation)
                        	// 生成代码格式同上
                            getNameCode = String.format("url.getMethodParameter(methodName, \"%s\", \"%s\")", value[i], defaultExtName);
	                    else
	                    	// 生成的代码功能等价于下面的代码：
	                        //   url.getParameter(value[i])
	                        getNameCode = String.format("url.getParameter(\"%s\")", value[i]);
                    else
                    	// 生成从 url 中获取协议的代码，比如 "dubbo"
                        getNameCode = "url.getProtocol()";
                }
            } else {
                if (!"protocol".equals(value[i]))
                    if (hasInvocation)
                        // 生成代码格式同上
                        getNameCode = String.format("url.getMethodParameter(methodName, \"%s\", \"%s\")", value[i], defaultExtName);
	                else
	                	// 生成的代码功能等价于下面的代码：
	                    //   url.getParameter(value[i], getNameCode)
	                    // 以 Transporter 接口的 connect 方法为例，最终生成的代码如下：
	                    //   url.getParameter("client", url.getParameter("transporter", "netty"))
	                    getNameCode = String.format("url.getParameter(\"%s\", %s)", value[i], getNameCode);
                else
                    // 生成的代码功能等价于下面的代码：
                    //   url.getProtocol() == null ? getNameCode : url.getProtocol()
                    // 以 Protocol 接口的 connect 方法为例，最终生成的代码如下：
                    //   url.getProtocol() == null ? "dubbo" : url.getProtocol()
                    getNameCode = String.format("url.getProtocol() == null ? (%s) : url.getProtocol()", getNameCode);
            }
        }
        // 生成 extName 赋值代码
        code.append("\nString extName = ").append(getNameCode).append(";");
        // 生成 extName 判空代码
        String s = String.format("\nif(extName == null) " +
                                 "throw new IllegalStateException(\"Fail to get extension(%s) name from url(\" + url.toString() + \") use keys(%s)\");",
                                 type.getName(), Arrays.toString(value));
        code.append(s);
    }
    
    // 省略无关逻辑
}
```

​	上面代码比较复杂，不是很好理解。对于这段代码，建议大家写点测试用例，对 Protocol、LoadBalance 以及 Transporter 等接口的自适应拓展类代码生成过程进行调试。这里我以 Transporter 接口的自适应拓展类代码生成过程举例说明。首先看一下 Transporter 接口的定义，如下：

```java
@SPI("netty")
public interface Transporter {
	// @Adaptive({server, transporter})
    @Adaptive({Constants.SERVER_KEY, Constants.TRANSPORTER_KEY}) 
    Server bind(URL url, ChannelHandler handler) throws RemotingException;

    // @Adaptive({client, transporter})
    @Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})
    Client connect(URL url, ChannelHandler handler) throws RemotingException;
}
```

​	下面对 connect 方法代理逻辑生成的过程进行分析，此时生成代理逻辑所用到的变量如下：

```java
String defaultExtName = "netty";
boolean hasInvocation = false;
String getNameCode = null;
String[] value = ["client", "transporter"];
```

下面对 value 数组进行遍历，此时 i = 1, value[i] = “transporter”，生成的代码如下：

```java
getNameCode = url.getParameter("transporter", "netty");
```

接下来，for 循环继续执行，此时 i = 0, value[i] = “client”，生成的代码如下：

```java
getNameCode = url.getParameter("client", url.getParameter("transporter", "netty"));
```

for 循环结束运行，现在为 extName 变量生成赋值和判空代码，如下：

```java
String extName = url.getParameter("client", url.getParameter("transporter", "netty"));
if (extName == null) {
    throw new IllegalStateException(
        "Fail to get extension(com.alibaba.dubbo.remoting.Transporter) name from url(" + url.toString()
        + ") use keys([client, transporter])");
}
```

#### 生成拓展加载与目标方法调用逻辑

​	本段代码逻辑用于根据拓展名加载拓展实例，并调用拓展实例的目标方法。相关逻辑如下：

```java
for (Method method : methods) {
    Class<?> rt = method.getReturnType();
    Class<?>[] pts = method.getParameterTypes();
    Class<?>[] ets = method.getExceptionTypes();

    Adaptive adaptiveAnnotation = method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        // $无 Adaptive 注解方法代码生成逻辑}
    } else {
        // ${获取 URL 数据}
        
        // ${获取 Adaptive 注解值}
        
        // ${检测 Invocation 参数}
        
        // ${生成拓展名获取逻辑}
        
        // 生成拓展获取代码，格式如下：
        // type全限定名 extension = (type全限定名)ExtensionLoader全限定名
        //     .getExtensionLoader(type全限定名.class).getExtension(extName);
        // Tips: 格式化字符串中的 %<s 表示使用前一个转换符所描述的参数，即 type 全限定名
        s = String.format("\n%s extension = (%<s)%s.getExtensionLoader(%s.class).getExtension(extName);",
                        type.getName(), ExtensionLoader.class.getSimpleName(), type.getName());
        code.append(s);

		// 如果方法返回值类型非 void，则生成 return 语句。
        if (!rt.equals(void.class)) {
            code.append("\nreturn ");
        }

        // 生成目标方法调用逻辑，格式为：
        //     extension.方法名(arg0, arg2, ..., argN);
        s = String.format("extension.%s(", method.getName());
        code.append(s);
        for (int i = 0; i < pts.length; i++) {
            if (i != 0)
                code.append(", ");
            code.append("arg").append(i);
        }
        code.append(");");   
    }
    
    // 省略无关逻辑
}
```

​	以 Protocol 接口举例说明，上面代码生成的内容如下：

```java
com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol) ExtensionLoader
    .getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class).getExtension(extName);
return extension.refer(arg0, arg1);
```

#### 生成完整的方法

​	本节进行代码生成的收尾工作，主要用于生成方法定义的代码。相关逻辑如下：

```java
for (Method method : methods) {
    Class<?> rt = method.getReturnType();
    Class<?>[] pts = method.getParameterTypes();
    Class<?>[] ets = method.getExceptionTypes();

    Adaptive adaptiveAnnotation = method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    if (adaptiveAnnotation == null) {
        // $无 Adaptive 注解方法代码生成逻辑}
    } else {
        // ${获取 URL 数据}
        
        // ${获取 Adaptive 注解值}
        
        // ${检测 Invocation 参数}
        
        // ${生成拓展名获取逻辑}
        
        // ${生成拓展加载与目标方法调用逻辑}
    }
}
    
// public + 返回值全限定名 + 方法名 + (
codeBuilder.append("\npublic ")
    .append(rt.getCanonicalName())
    .append(" ")
    .append(method.getName())
    .append("(");

// 添加参数列表代码
for (int i = 0; i < pts.length; i++) {
    if (i > 0) {
        codeBuilder.append(", ");
    }
    codeBuilder.append(pts[i].getCanonicalName());
    codeBuilder.append(" ");
    codeBuilder.append("arg").append(i);
}
codeBuilder.append(")");

// 添加异常抛出代码
if (ets.length > 0) {
    codeBuilder.append(" throws ");
    for (int i = 0; i < ets.length; i++) {
        if (i > 0) {
            codeBuilder.append(", ");
        }
        codeBuilder.append(ets[i].getCanonicalName());
    }
}
codeBuilder.append(" {");
codeBuilder.append(code.toString());
codeBuilder.append("\n}");
```

以 Protocol 的 refer 方法为例，上面代码生成的内容如下：

```java
public com.alibaba.dubbo.rpc.Invoker refer(java.lang.Class arg0, com.alibaba.dubbo.common.URL arg1) {
    // 方法体
}
```

## 自适应拓展类代码生成 3.0

```java
public String generate() {
    // no need to generate adaptive class since there's no adaptive method found.
    if (!hasAdaptiveMethod()) {
        throw new IllegalStateException("No adaptive method exist on extension " + type.getName() + ", refuse to create the adaptive class!");
    }

    StringBuilder code = new StringBuilder();
    code.append(generatePackageInfo());
    code.append(generateImports());
    code.append(generateClassDeclaration());

    Method[] methods = type.getMethods();
    for (Method method : methods) {
        code.append(generateMethod(method));
    }
    code.append("}");

    if (logger.isDebugEnabled()) {
        logger.debug(code.toString());
    }
    return code.toString();
}
```

### 生成方法

```java
private String generateMethod(Method method) {
        String methodReturnType = method.getReturnType().getCanonicalName();
        String methodName = method.getName();
        String methodContent = generateMethodContent(method);
        String methodArgs = generateMethodArguments(method);
        String methodThrows = generateMethodThrows(method);
        return String.format(CODE_METHOD_DECLARATION, methodReturnType, methodName, methodArgs, methodThrows, methodContent);
    }
```

#### 生成方法内容

```java

private String generateMethodContent(Method method) {
    Adaptive adaptiveAnnotation = method.getAnnotation(Adaptive.class);
    StringBuilder code = new StringBuilder(512);
    // 查找是否有@Adaptive
    if (adaptiveAnnotation == null) {
        return generateUnsupported(method);
    } else {
        // 获取url参数
        int urlTypeIndex = getUrlTypeIndex(method);
        // found parameter in URL type
        if (urlTypeIndex != -1) {
            // Null Point check
            // 空指针代码
            code.append(generateUrlNullCheck(urlTypeIndex));
        } else {
            // did not find parameter in URL type
            code.append(generateUrlAssignmentIndirectly(method));
        }
	   // 获取@Adaptive
        String[] value = getMethodAdaptiveValue(adaptiveAnnotation);
	   // 检查参数是否有Invocation
        boolean hasInvocation = hasInvocationArgument(method);
	   // 空指针检查代码
        code.append(generateInvocationArgumentNullCheck(method));
	   // 扩展名获取
        code.append(generateExtNameAssignment(value, hasInvocation));
        // check extName == null?
        code.append(generateExtNameNullCheck(value));
		// 扩展赋值
        code.append(generateExtensionAssignment());
		// 返回语句
        // return statement
        code.append(generateReturnAndInvocation(method));
    }

    return code.toString();
}
```

**检查方法参数的类中标是否有url的getter**

```java
private String generateUrlAssignmentIndirectly(Method method) {
    Class<?>[] pts = method.getParameterTypes();

    Map<String, Integer> getterReturnUrl = new HashMap<>();
    // find URL getter method
    // 查找方法的参数中是否有类有URL
    for (int i = 0; i < pts.length; ++i) {
        for (Method m : pts[i].getMethods()) {
            String name = m.getName();
            if ((name.startsWith("get") || name.length() > 3)
                    && Modifier.isPublic(m.getModifiers())
                    && !Modifier.isStatic(m.getModifiers())
                    && m.getParameterTypes().length == 0
                    && m.getReturnType() == URL.class) {
                getterReturnUrl.put(name, i);
            }
        }
    }

    if (getterReturnUrl.size() <= 0) {
        // getter method not found, throw
        throw new IllegalStateException("Failed to create adaptive class for interface " + type.getName()
                + ": not found url parameter or url attribute in parameters of method " + method.getName());
    }

    Integer index = getterReturnUrl.get("getUrl");
    if (index != null) {
        return generateGetUrlNullCheck(index, pts[index], "getUrl");
    } else {
        Map.Entry<String, Integer> entry = getterReturnUrl.entrySet().iterator().next();
        return generateGetUrlNullCheck(entry.getValue(), pts[entry.getValue()], entry.getKey());
    }
}
```

**ExtName赋值**

```java
private String generateExtNameAssignment(String[] value, boolean hasInvocation) {
    // TODO: refactor it
    String getNameCode = null;
    for (int i = value.length - 1; i >= 0; --i) {
        if (i == value.length - 1) {
            if (null != defaultExtName) {
                if (!"protocol".equals(value[i])) {
                    if (hasInvocation) {
                        getNameCode = String.format("url.getMethodParameter(methodName, \"%s\", \"%s\")", value[i], defaultExtName);
                    } else {
                        getNameCode = String.format("url.getParameter(\"%s\", \"%s\")", value[i], defaultExtName);
                    }
                } else {
                    getNameCode = String.format("( url.getProtocol() == null ? \"%s\" : url.getProtocol() )", defaultExtName);
                }
            } else {
                if (!"protocol".equals(value[i])) {
                    if (hasInvocation) {
                        getNameCode = String.format("url.getMethodParameter(methodName, \"%s\", \"%s\")", value[i], defaultExtName);
                    } else {
                        getNameCode = String.format("url.getParameter(\"%s\")", value[i]);
                    }
                } else {
                    getNameCode = "url.getProtocol()";
                }
            }
        } else {
            if (!"protocol".equals(value[i])) {
                if (hasInvocation) {
                    getNameCode = String.format("url.getMethodParameter(methodName, \"%s\", \"%s\")", value[i], defaultExtName);
                } else {
                    getNameCode = String.format("url.getParameter(\"%s\", %s)", value[i], getNameCode);
                }
            } else {
                getNameCode = String.format("url.getProtocol() == null ? (%s) : url.getProtocol()", getNameCode);
            }
        }
    }

    return String.format(CODE_EXT_NAME_ASSIGNMENT, getNameCode);
}
```

**扩展赋值**

```java
    private static final String CODE_EXTENSION_ASSIGNMENT = "%s extension = (%<s)%s.getExtensionLoader(%s.class).getExtension(extName);\n";

private String generateExtensionAssignment() {
    return String.format(CODE_EXTENSION_ASSIGNMENT, type.getName(), ExtensionLoader.class.getSimpleName(), type.getName());
}
```

# 服务路由

https://dubbo.apache.org/zh/docs/v2.7/dev/source/router/

​	服务目录在刷新 Invoker 列表的过程中，会通过 Router 进行服务路由，筛选出符合路由规则的服务提供者。在详细分析服务路由的源码之前，先来介绍一下服务路由是什么。服务路由包含一条路由规则，路由规则决定了服务消费者的调用目标，即规定了服务消费者可调用哪些服务提供者。Dubbo 目前提供了三种服务路由实现，分别为条件路由 ConditionRouter、脚本路由 ScriptRouter 和标签路由 TagRouter。其中条件路由是我们最常使用的，标签路由是一个新的实现，暂时还未发布，该实现预计会在 2.7.x 版本中发布。本篇文章将分析条件路由相关源码，脚本路由和标签路由这里就不分析了。

## 源码分析

​	条件路由规则由两个条件组成，分别用于对服务消费者和提供者进行匹配。比如有这样一条规则：

```
host = 10.20.153.10 => host = 10.20.153.11
```

​	该条规则表示 IP 为 10.20.153.10 的服务消费者**只可**调用 IP 为 10.20.153.11 机器上的服务，不可调用其他机器上的服务。条件路由规则的格式如下：

`[服务消费者匹配条件] => [服务提供者匹配条件]`

​	如果服务消费者匹配条件为空，表示不对服务消费者进行限制。如果服务提供者匹配条件为空，表示对某些服务消费者禁用服务。官方文档中对条件路由进行了比较详细的介绍，大家可以参考下，这里就不过多说明了。

​	条件路由实现类 ConditionRouter 在进行工作前，需要先对用户配置的路由规则进行解析，得到一系列的条件。然后再根据这些条件对服务进行路由。本章将分两节进行说明，2.1节介绍表达式解析过程。2.2 节介绍服务路由的过程。下面，我们先从表达式解析过程看起。

###  表达式解析

​	条件路由规则是一条字符串，对于 Dubbo 来说，它并不能直接理解字符串的意思，需要将其解析成内部格式才行。条件表达式的解析过程始于 ConditionRouter 的构造方法，下面一起看一下：

```java
public ConditionRouter(URL url) {
    this.url = url;
    // 获取 priority 和 force 配置
    this.priority = url.getParameter(PRIORITY_KEY, 0);
    this.force = url.getParameter(FORCE_KEY, false);
    this.enabled = url.getParameter(ENABLED_KEY, true);
    if (enabled) {
        // 获取路由规则进行初始化
        init(url.getParameterAndDecoded(RULE_KEY));
    }
}

public void init(String rule) {
    try {
        if (rule == null || rule.trim().length() == 0) {
            throw new IllegalArgumentException("Illegal route rule!");
        }
        rule = rule.replace("consumer.", "").replace("provider.", "");
        // 定位 => 分隔符
        int i = rule.indexOf("=>");
        // 分别获取服务消费者和提供者匹配规则
        String whenRule = i < 0 ? null : rule.substring(0, i).trim();
        String thenRule = i < 0 ? rule.trim() : rule.substring(i + 2).trim();
        // 解析服务消费者匹配规则
        Map<String, MatchPair> when = StringUtils.isBlank(whenRule) || "true".equals(whenRule) ? new HashMap<String, MatchPair>() : parseRule(whenRule);
        // 解析服务提供者匹配规则
        Map<String, MatchPair> then = StringUtils.isBlank(thenRule) || "false".equals(thenRule) ? null : parseRule(thenRule);
        // NOTE: It should be determined on the business level whether the `When condition` can be empty or not.
        // 将解析出的匹配规则分别赋值给 whenCondition 和 thenCondition 成员变量
        this.whenCondition = when;
        this.thenCondition = then;
    } catch (ParseException e) {
        throw new IllegalStateException(e.getMessage(), e);
    }
}
```

​	如上，ConditionRouter 构造方法先是对路由规则做预处理，然后调用 parseRule 方法分别对服务提供者和消费者规则进行解析，最后将解析结果赋值给 whenCondition 和 thenCondition 成员变量。ConditionRouter 构造方法不是很复杂，这里就不多说了。下面我们把重点放在 parseRule 方法上，在详细介绍这个方法之前，我们先来看一个内部类。

```java
private static final class MatchPair {
    final Set<String> matches = new HashSet<String>();
    final Set<String> mismatches = new HashSet<String>();
}
```

​	MatchPair 内部包含了两个 Set 类型的成员变量，分别用于存放匹配和不匹配的条件。这个类两个成员变量会在 parseRule 方法中被用到，下面来看一下。

```java
private static Map<String, MatchPair> parseRule(String rule)
        throws ParseException {
    Map<String, MatchPair> condition = new HashMap<String, MatchPair>();
    if (StringUtils.isBlank(rule)) {
        return condition;
    }
    // Key-Value pair, stores both match and mismatch conditions
    MatchPair pair = null;
    // Multiple values
    Set<String> values = null;
     // 通过正则表达式匹配路由规则，ROUTE_PATTERN = ([&!=,]*)\s*([^&!=,\s]+)
    // 这个表达式看起来不是很好理解，第一个括号内的表达式用于匹配"&", "!", "=" 和 "," 等符号。
    // 第二括号内的用于匹配英文字母，数字等字符。举个例子说明一下：
    //    host = 2.2.2.2 & host != 1.1.1.1 & method = hello
    // 匹配结果如下：
    //     括号一      括号二
    // 1.  null       host
    // 2.   =         2.2.2.2
    // 3.   &         host
    // 4.   !=        1.1.1.1 
    // 5.   &         method
    // 6.   =         hello
    final Matcher matcher = ROUTE_PATTERN.matcher(rule);
    while (matcher.find()) { // Try to match one by one
        // 获取括号一内的匹配结果
        String separator = matcher.group(1);
        // 获取括号二内的匹配结果
        String content = matcher.group(2);
        // 分隔符为空，表示匹配的是表达式的开始部分
        // Start part of the condition expression.
        if (StringUtils.isEmpty(separator)) {
            // 创建 MatchPair 对象
            pair = new MatchPair();
            // 存储 <匹配项, MatchPair> 键值对，比如 <host, MatchPair>
            condition.put(content, pair);
        }
        // The KV part of the condition expression
        // 如果分隔符为 &，表明接下来也是一个条件
        else if ("&".equals(separator)) {
            // 尝试从 condition 获取 MatchPair
            if (condition.get(content) == null) {
                // 未获取到 MatchPair，重新创建一个，并放入 condition 中
                pair = new MatchPair();
                condition.put(content, pair);
            } else {
                pair = condition.get(content);
            }
        }
        // 分隔符为 =
        // The Value in the KV part.
        else if ("=".equals(separator)) {
            if (pair == null) {
                throw new ParseException("Illegal route rule \""
                        + rule + "\", The error char '" + separator
                        + "' at index " + matcher.start() + " before \""
                        + content + "\".", matcher.start());
            }

            values = pair.matches;
            // 将 content 存入到 MatchPair 的 matches 集合中
            values.add(content);
        }
        //  分隔符为 != 
        // The Value in the KV part.
        else if ("!=".equals(separator)) {
            if (pair == null) {
                throw new ParseException("Illegal route rule \""
                        + rule + "\", The error char '" + separator
                        + "' at index " + matcher.start() + " before \""
                        + content + "\".", matcher.start());
            }
			// 将 content 存入到 MatchPair 的 mismatches 集合中
            values = pair.mismatches;
            values.add(content);
        }
        // The Value in the KV part, if Value have more than one items.
        // 分隔符为 ,
        else if (",".equals(separator)) { // Should be separated by ','
            if (values == null || values.isEmpty()) {
                throw new ParseException("Illegal route rule \""
                        + rule + "\", The error char '" + separator
                        + "' at index " + matcher.start() + " before \""
                        + content + "\".", matcher.start());
            }
            // 将 content 存入到上一步获取到的 values 中，可能是 matches，也可能是 mismatches
            values.add(content);
        } else {
            throw new ParseException("Illegal route rule \"" + rule
                    + "\", The error char '" + separator + "' at index "
                    + matcher.start() + " before \"" + content + "\".", matcher.start());
        }
    }
    return condition;
}
```

```fallback
    括号一      括号二
1.  null       host
2.   =         2.2.2.2
3.   &         host
4.   !=        1.1.1.1
5.   &         method
6.   =         hello
```

 condition 的内容如下：

```json
{
    "host": {
        "matches": ["2.2.2.2"],
        "mismatches": ["1.1.1.1"]
    },
    "method": {
        "matches": ["hello"],
        "mismatches": []
    }
}
```

### 服务路由

​	服务路由的入口方法是 ConditionRouter 的 route 方法，该方法定义在 Router 接口中。实现代码如下：

```java
public <T> List<Invoker<T>> route(List<Invoker<T>> invokers, URL url, Invocation invocation)
        throws RpcException {
    if (!enabled) {
        return invokers;
    }

    if (CollectionUtils.isEmpty(invokers)) {
        return invokers;
    }
    try {
        // 先对服务消费者条件进行匹配，如果匹配失败，表明服务消费者 url 不符合匹配规则，
        // 无需进行后续匹配，直接返回 Invoker 列表即可。比如下面的规则：
        //     host = 10.20.153.10 => host = 10.0.0.10
        // 这条路由规则希望 IP 为 10.20.153.10 的服务消费者调用 IP 为 10.0.0.10 机器上的服务。
        // 当消费者 ip 为 10.20.153.11 时，matchWhen 返回 false，表明当前这条路由规则不适用于
        // 当前的服务消费者，此时无需再进行后续匹配，直接返回即可。
        if (!matchWhen(url, invocation)) {
            return invokers;
        }
        List<Invoker<T>> result = new ArrayList<Invoker<T>>();
        // 服务提供者匹配条件未配置，表明对指定的服务消费者禁用服务，也就是服务消费者在黑名单中
        if (thenCondition == null) {
            logger.warn("The current consumer in the service blacklist. consumer: " + NetUtils.getLocalHost() + ", service: " + url.getServiceKey());
            return result;
        }
        // 这里可以简单的把 Invoker 理解为服务提供者，现在使用服务提供者匹配规则对 
        // Invoker 列表进行匹配
        for (Invoker<T> invoker : invokers) {
            if (matchThen(invoker.getUrl(), url)) {
                result.add(invoker);
            }
        }
        // 返回匹配结果，如果 result 为空列表，且 force = true，表示强制返回空列表，
        // 否则路由结果为空的路由规则将自动失效
        if (!result.isEmpty()) {
            return result;
        } else if (force) {
            logger.warn("The route result is empty and force execute. consumer: " + NetUtils.getLocalHost() + ", service: " + url.getServiceKey() + ", router: " + url.getParameterAndDecoded(RULE_KEY));
            return result;
        }
    } catch (Throwable t) {
        logger.error("Failed to execute condition router rule: " + getUrl() + ", invokers: " + invokers + ", cause: " + t.getMessage(), t);
    }
    // 原样返回，此时 force = false，表示该条路由规则失效
    return invokers;
}
```

​	route 方法先是调用 matchWhen 对服务消费者进行匹配，如果匹配失败，直接返回 Invoker 列表。如果匹配成功，再对服务提供者进行匹配，匹配逻辑封装在了 matchThen 方法中。下面来看一下这两个方法的逻辑：

```javajava
boolean matchWhen(URL url, Invocation invocation) {
    // 服务消费者条件为 null 或空，均返回 true，比如：
    //     => host != 172.22.3.91
    // 表示所有的服务消费者都不得调用 IP 为 172.22.3.91 的机器上的服务
    return whenCondition == null || whenCondition.isEmpty() 
        || matchCondition(whenCondition, url, null, invocation);  // 进行条件匹配
}

private boolean matchThen(URL url, URL param) {
    // 服务提供者条件为 null 或空，表示禁用服务
    return !(thenCondition == null || thenCondition.isEmpty()) 
        && matchCondition(thenCondition, url, param, null);  // 进行条件匹配
}
```

​	这两个方法长的有点像，不过逻辑上还是有差别的，大家注意看。这两个方法均调用了 matchCondition 方法，但它们所传入的参数是不同的。这个需要特别注意一下，不然后面的逻辑不好弄懂。下面我们对这几个参数进行溯源。matchWhen 方法向 matchCondition 方法传入的参数为 [whenCondition, url, null, invocation]，第一个参数 whenCondition 为服务消费者匹配条件，这个前面分析过。第二个参数 url 源自 route 方法的参数列表，该参数由外部类调用 route 方法时传入。比如：

```java
private List<Invoker<T>> route(List<Invoker<T>> invokers, String method) {
    Invocation invocation = new RpcInvocation(method, new Class<?>[0], new Object[0]);
    List<Router> routers = getRouters();
    if (routers != null) {
        for (Router router : routers) {
            if (router.getUrl() != null) {
                // 注意第二个参数
                invokers = router.route(invokers, getConsumerUrl(), invocation);
            }
        }
    }
    return invokers;
}
```

​	上面这段代码来自 RegistryDirectory，第二个参数表示的是服务消费者 url。matchCondition 的 invocation 参数也是从这里传入的。

​	接下来再来看看 matchThen 向 matchCondition 方法传入的参数 [thenCondition, url, param, null]。第一个参数不用解释了。第二个和第三个参数来自 matchThen 方法的参数列表，这两个参数分别为服务提供者 url 和服务消费者 url。搞清楚这些参数来源后，接下来就可以分析 matchCondition 方法了。

```java
private boolean matchCondition(Map<String, MatchPair> condition, URL url, URL param, Invocation invocation) {
    // 将服务提供者或消费者 url 转成 Map
    Map<String, String> sample = url.toMap();
    boolean result = false;
    // 遍历 condition 列表
    for (Map.Entry<String, MatchPair> matchPair : condition.entrySet()) {
        // 获取匹配项名称，比如 host、method 等
        String key = matchPair.getKey();
        String sampleValue;
        // 如果 invocation 不为空，且 key 为 method(s)，表示进行方法匹配
        if (invocation != null && (Constants.METHOD_KEY.equals(key) || Constants.METHODS_KEY.equals(key))) {
            // 从 invocation 获取被调用方法的名称
            sampleValue = invocation.getMethodName();
        } else {
            // 从服务提供者或消费者 url 中获取指定字段值，比如 host、application 等
            sampleValue = sample.get(key);
            if (sampleValue == null) {
                // 尝试通过 default.xxx 获取相应的值
                sampleValue = sample.get(Constants.DEFAULT_KEY_PREFIX + key);
            }
        }
        
        // --------------------✨ 分割线 ✨-------------------- //
        
        if (sampleValue != null) {
            // 调用 MatchPair 的 isMatch 方法进行匹配
            if (!matchPair.getValue().isMatch(sampleValue, param)) {
                // 只要有一个规则匹配失败，立即返回 false 结束方法逻辑
                return false;
            } else {
                result = true;
            }
        } else {
            // sampleValue 为空，表明服务提供者或消费者 url 中不包含相关字段。此时如果 
            // MatchPair 的 matches 不为空，表示匹配失败，返回 false。比如我们有这样
            // 一条匹配条件 loadbalance = random，假设 url 中并不包含 loadbalance 参数，
            // 此时 sampleValue = null。既然路由规则里限制了 loadbalance 必须为 random，
            // 但 sampleValue = null，明显不符合规则，因此返回 false
            if (!matchPair.getValue().matches.isEmpty()) {
                return false;
            } else {
                result = true;
            }
        }
    }
    return result;
}
```

​	如上，matchCondition 方法看起来有点复杂，这里简单说明一下。分割线以上的代码实际上用于获取 sampleValue 的值，分割线以下才是进行条件匹配。条件匹配调用的逻辑封装在 isMatch 中，代码如下：

```java
private boolean isMatch(String value, URL param) {
    // 情况一：matches 非空，mismatches 为空
    if (!matches.isEmpty() && mismatches.isEmpty()) {
        // 遍历 matches 集合，检测入参 value 是否能被 matches 集合元素匹配到。
        // 举个例子，如果 value = 10.20.153.11，matches = [10.20.153.*],
        // 此时 isMatchGlobPattern 方法返回 true
        for (String match : matches) {
            if (UrlUtils.isMatchGlobPattern(match, value, param)) {
                return true;
            }
        }
        
        // 如果所有匹配项都无法匹配到入参，则返回 false
        return false;
    }

    // 情况二：matches 为空，mismatches 非空
    if (!mismatches.isEmpty() && matches.isEmpty()) {
        for (String mismatch : mismatches) {
            // 只要入参被 mismatches 集合中的任意一个元素匹配到，就返回 false
            if (UrlUtils.isMatchGlobPattern(mismatch, value, param)) {
                return false;
            }
        }
        // mismatches 集合中所有元素都无法匹配到入参，此时返回 true
        return true;
    }

    // 情况三：matches 非空，mismatches 非空
    if (!matches.isEmpty() && !mismatches.isEmpty()) {
        // matches 和 mismatches 均为非空，此时优先使用 mismatches 集合元素对入参进行匹配。
        // 只要 mismatches 集合中任意一个元素与入参匹配成功，就立即返回 false，结束方法逻辑
        for (String mismatch : mismatches) {
            if (UrlUtils.isMatchGlobPattern(mismatch, value, param)) {
                return false;
            }
        }
        // mismatches 集合元素无法匹配到入参，此时再使用 matches 继续匹配
        for (String match : matches) {
            // 只要 matches 集合中任意一个元素与入参匹配成功，就立即返回 true
            if (UrlUtils.isMatchGlobPattern(match, value, param)) {
                return true;
            }
        }
        
        // 全部失配，则返回 false
        return false;
    }
    
    // 情况四：matches 和 mismatches 均为空，此时返回 false
    return false;
}
```

​	isMatch 方法逻辑比较清晰，由三个条件分支组成，用于处理四种情况。这里对四种情况下的匹配逻辑进行简单的总结，如下：

|        | 条件                          | 过程                                                         |
| ------ | ----------------------------- | ------------------------------------------------------------ |
| 情况一 | matches 非空，mismatches 为空 | 遍历 matches 集合元素，并与入参进行匹配。只要有一个元素成功匹配入参，即可返回 true。若全部失配，则返回 false。 |
| 情况二 | matches 为空，mismatches 非空 | 遍历 mismatches 集合元素，并与入参进行匹配。只要有一个元素成功匹配入参，立即 false。若全部失配，则返回 true。 |
| 情况三 | matches 非空，mismatches 非空 | 优先使用 mismatches 集合元素对入参进行匹配，只要任一元素与入参匹配成功，就立即返回 false，结束方法逻辑。否则再使用 matches 中的集合元素进行匹配，只要有任意一个元素匹配成功，即可返回 true。若全部失配，则返回 false |
| 情况四 | matches 为空，mismatches 为空 | 直接返回 false                                               |

可以更简洁：

https://github.com/apache/dubbo/issues/8524

​	isMatch 方法是通过 UrlUtils 的 isMatchGlobPattern 方法进行匹配，因此下面我们再来看看 isMatchGlobPattern 方法的逻辑。

```java
public static boolean isMatchGlobPattern(String pattern, String value, URL param) {
    if (param != null && pattern.startsWith("$")) {
        // 引用服务消费者参数，param 参数为服务消费者 url
        pattern = param.getRawParameter(pattern.substring(1));
    }
    // 调用重载方法继续比较
    return isMatchGlobPattern(pattern, value);
}

public static boolean isMatchGlobPattern(String pattern, String value) {
    // 对 * 通配符提供支持
    if ("*".equals(pattern))
        // 匹配规则为通配符 *，直接返回 true 即可
        return true;
    if ((pattern == null || pattern.length() == 0)
            && (value == null || value.length() == 0))
        // pattern 和 value 均为空，此时可认为两者相等，返回 true
        return true;
    if ((pattern == null || pattern.length() == 0)
            || (value == null || value.length() == 0))
        // pattern 和 value 其中有一个为空，表明两者不相等，返回 false
        return false;

    // 定位 * 通配符位置
    int i = pattern.lastIndexOf('*');
    if (i == -1) {
        // 匹配规则中不包含通配符，此时直接比较 value 和 pattern 是否相等即可，并返回比较结果
        return value.equals(pattern);
    }
    // 通配符 "*" 在匹配规则尾部，比如 10.0.21.*
    else if (i == pattern.length() - 1) {
        // 检测 value 是否以“不含通配符的匹配规则”开头，并返回结果。比如:
        // pattern = 10.0.21.*，value = 10.0.21.12，此时返回 true
        return value.startsWith(pattern.substring(0, i));
    }
    // 通配符 "*" 在匹配规则头部
    else if (i == 0) {
        // 检测 value 是否以“不含通配符的匹配规则”结尾，并返回结果
        return value.endsWith(pattern.substring(i + 1));
    }
    // 通配符 "*" 在匹配规则中间位置
    else {
        // 通过通配符将 pattern 分成两半，得到 prefix 和 suffix
        String prefix = pattern.substring(0, i);
        String suffix = pattern.substring(i + 1);
        // 检测 value 是否以 prefix 开头，且以 suffix 结尾，并返回结果
        return value.startsWith(prefix) && value.endsWith(suffix);
    }
}
```

​	以上就是 isMatchGlobPattern 两个重载方法的全部逻辑，这两个方法分别对普通的匹配过程，以及”引用消费者参数“和通配符匹配等特性提供了支持。这两个方法的逻辑不是很复杂，且代码中也进行了比较详细的注释，因此就不多说了。

## 总结

​	本篇文章对条件路由的表达式解析和服务路由过程进行了较为细致的分析。总的来说，条件路由的代码还是有一些复杂的，需要静下心来看。在阅读条件路由代码的过程中，要多调试。一般的框架都会有单元测试，Dubbo 也不例外，因此大家可以直接通过 ConditionRouterTest 对条件路由进行调试，无需重头构建测试用例。

# 服务导出

https://dubbo.apache.org/zh/docs/v2.7/dev/source/export-service/

## 简介

​	本篇文章，我们来研究一下 Dubbo 导出服务的过程。Dubbo 服务导出过程始于 Spring 容器发布刷新事件，Dubbo 在接收到事件后，会立即执行服务导出逻辑。整个逻辑大致可分为三个部分，第一部分是前置工作，主要用于检查参数，组装 URL。第二部分是导出服务，包含导出服务到本地 (JVM)，和导出服务到远程两个过程。第三部分是向注册中心注册服务，用于服务发现。本篇文章将会对这三个部分代码进行详细的分析。

## 2.7源码分析

​	服务导出的入口方法是 ServiceBean 的 onApplicationEvent。onApplicationEvent 是一个事件响应方法，该方法会在收到 Spring 上下文刷新事件后执行服务导出操作。方法代码如下：

```java
public void onApplicationEvent(ContextRefreshedEvent event) {
    // 是否有延迟导出 && 是否已导出 && 是不是已被取消导出
    if (isDelay() && !isExported() && !isUnexported()) {
        // 导出服务
        export();
    }
}
```

​	这个方法首先会根据条件决定是否导出服务，比如有些服务设置了延时导出，那么此时就不应该在此处导出。还有一些服务已经被导出了，或者当前服务被取消导出了，此时也不能再次导出相关服务。注意这里的 isDelay 方法，这个方法字面意思是“是否延迟导出服务”，返回 true 表示延迟导出，false 表示不延迟导出。但是该方法真实意思却并非如此，当方法返回 true 时，表示无需延迟导出。返回 false 时，表示需要延迟导出。与字面意思恰恰相反，这个需要大家注意一下。下面我们来看一下这个方法的逻辑。

```java
// -☆- ServiceBean
private boolean isDelay() {
    // 获取 delay
    Integer delay = getDelay();
    ProviderConfig provider = getProvider();
    if (delay == null && provider != null) {
        // 如果前面获取的 delay 为空，这里继续获取
        delay = provider.getDelay();
    }
    // 判断 delay 是否为空，或者等于 -1
    return supportedApplicationListener && (delay == null || delay == -1);
}
```

​	暂时忽略 supportedApplicationListener 这个条件，当 delay 为空，或者等于-1时，该方法返回 true，而不是 false。这个方法的返回值让人有点困惑。该方法目前已被重构，详细请参考 [dubbo #2686](https://github.com/apache/dubbo/pull/2686)。

​	3.0找不到该方法。

​	现在解释一下 supportedApplicationListener 变量含义，该变量用于表示当前的 Spring 容器是否支持 ApplicationListener，这个值初始为 false。在 Spring 容器将自己设置到 ServiceBean 中时，ServiceBean 的 setApplicationContext 方法会检测 Spring 容器是否支持 ApplicationListener。若支持，则将 supportedApplicationListener 置为 true。ServiceBean 是 Dubbo 与 Spring 框架进行整合的关键，可以看做是两个框架之间的桥梁。具有同样作用的类还有 ReferenceBean。

​	现在我们知道了 Dubbo 服务导出过程的起点，接下来对服务导出的前置逻辑进行分析。

### 前置工作

​	前置工作主要包含两个部分，分别是配置检查，以及 URL 装配。在导出服务之前，Dubbo 需要检查用户的配置是否合理，或者为用户补充缺省配置。配置检查完成后，接下来需要根据这些配置组装 URL。在 Dubbo 中，URL 的作用十分重要。Dubbo 使用 URL 作为配置载体，所有的拓展点都是通过 URL 获取配置。这一点，官方文档中有所说明。

> 采用 URL 作为配置信息的统一格式，所有扩展点都通过传递 URL 携带配置信息

​	接下来，我们先来分析配置检查部分的源码，随后再来分析 URL 组装部分的源码。

```java
public synchronized void export() {
    if (provider != null) {
        // 获取 export 和 delay 配置
        if (export == null) {
            export = provider.getExport();
        }
        if (delay == null) {
            delay = provider.getDelay();
        }
    }
    // 如果 export 为 false，则不导出服务
    if (export != null && !export) {
        return;
    }

    // delay > 0，延时导出服务
    if (delay != null && delay > 0) {
        delayExportExecutor.schedule(new Runnable() {
            @Override
            public void run() {
                doExport();
            }
        }, delay, TimeUnit.MILLISECONDS);
        
    // 立即导出服务
    } else {
        doExport();
    }
}
```

​	export 方法对两项配置进行了检查，并根据配置执行相应的动作。首先是 export 配置，这个配置决定了是否导出服务。有时候我们只是想本地启动服务进行一些调试工作，我们并不希望把本地启动的服务暴露出去给别人调用。此时，我们可通过配置 export 禁止服务导出，比如：

```xml
<dubbo:provider export="false" />
```

​	delay 配置顾名思义，用于延迟导出服务，这个就不分析了。下面，我们继续分析源码，这次要分析的是 doExport 方法。

```java
protected synchronized void doExport() {
    if (unexported) {
        throw new IllegalStateException("Already unexported!");
    }
    if (exported) {
        return;
    }
    exported = true;
    // 检测 interfaceName 是否合法
    if (interfaceName == null || interfaceName.length() == 0) {
        throw new IllegalStateException("interface not allow null!");
    }
    // 检测 provider 是否为空，为空则新建一个，并通过系统变量为其初始化
    checkDefault();

    // 下面几个 if 语句用于检测 provider、application 等核心配置类对象是否为空，
    // 若为空，则尝试从其他配置类对象中获取相应的实例。
    if (provider != null) {
        if (application == null) {
            application = provider.getApplication();
        }
        if (module == null) {
            module = provider.getModule();
        }
        if (registries == null) {...}
        if (monitor == null) {...}
        if (protocols == null) {...}
    }
    if (module != null) {
        if (registries == null) {
            registries = module.getRegistries();
        }
        if (monitor == null) {...}
    }
    if (application != null) {
        if (registries == null) {
            registries = application.getRegistries();
        }
        if (monitor == null) {...}
    }

    // 检测 ref 是否为泛化服务类型
    if (ref instanceof GenericService) {
        // 设置 interfaceClass 为 GenericService.class
        interfaceClass = GenericService.class;
        if (StringUtils.isEmpty(generic)) {
            // 设置 generic = "true"
            generic = Boolean.TRUE.toString();
        }
        
    // ref 非 GenericService 类型
    } else {
        try {
            interfaceClass = Class.forName(interfaceName, true, Thread.currentThread()
                    .getContextClassLoader());
        } catch (ClassNotFoundException e) {
            throw new IllegalStateException(e.getMessage(), e);
        }
        // 对 interfaceClass，以及 <dubbo:method> 标签中的必要字段进行检查
        checkInterfaceAndMethods(interfaceClass, methods);
        // 对 ref 合法性进行检测
        checkRef();
        // 设置 generic = "false"
        generic = Boolean.FALSE.toString();
    }

    // local 和 stub 在功能应该是一致的，用于配置本地存根
    if (local != null) {
        if ("true".equals(local)) {
            local = interfaceName + "Local";
        }
        Class<?> localClass;
        try {
            // 获取本地存根类
            localClass = ClassHelper.forNameWithThreadContextClassLoader(local);
        } catch (ClassNotFoundException e) {
            throw new IllegalStateException(e.getMessage(), e);
        }
        // 检测本地存根类是否可赋值给接口类，若不可赋值则会抛出异常，提醒使用者本地存根类类型不合法
        if (!interfaceClass.isAssignableFrom(localClass)) {
            throw new IllegalStateException("The local implementation class " + localClass.getName() + " not implement interface " + interfaceName);
        }
    }

    if (stub != null) {
        // 此处的代码和上一个 if 分支的代码基本一致，这里省略
    }

    // 检测各种对象是否为空，为空则新建，或者抛出异常
    checkApplication();
    checkRegistry();
    checkProtocol();
    appendProperties(this);
    checkStubAndMock(interfaceClass);
    if (path == null || path.length() == 0) {
        path = interfaceName;
    }

    // 导出服务
    doExportUrls();

    // ProviderModel 表示服务提供者模型，此对象中存储了与服务提供者相关的信息。
    // 比如服务的配置信息，服务实例等。每个被导出的服务对应一个 ProviderModel。
    // ApplicationModel 持有所有的 ProviderModel。
    ProviderModel providerModel = new ProviderModel(getUniqueServiceName(), this, ref);
    ApplicationModel.initProviderModel(getUniqueServiceName(), providerModel);
}
```

以上就是配置检查的相关分析，代码比较多，需要大家耐心看一下。下面对配置检查的逻辑进行简单的总结，如下：

1. 检测 `<dubbo:service>` 标签的 interface 属性合法性，不合法则抛出异常
2. 检测 ProviderConfig、ApplicationConfig 等核心配置类对象是否为空，若为空，则尝试从其他配置类对象中获取相应的实例。
3. 检测并处理泛化服务和普通服务类
4. 检测本地存根配置，并进行相应的处理
5. 对 ApplicationConfig、RegistryConfig 等配置类进行检测，为空则尝试创建，若无法创建则抛出异常



​	配置检查并非本文重点，因此这里不打算对 doExport 方法所调用的方法进行分析（doExportUrls 方法除外）。在这些方法中，除了 appendProperties 方法稍微复杂一些，其他方法逻辑不是很复杂。因此，大家可自行分析。

#### 多协议多注册中心导出服务

​	Dubbo 允许我们使用不同的协议导出服务，也允许我们向多个注册中心注册服务。Dubbo 在 doExportUrls 方法中对多协议，多注册中心进行了支持。相关代码如下：

```java
private void doExportUrls() {
    // 加载注册中心链接
    List<URL> registryURLs = loadRegistries(true);
    // 遍历 protocols，并在每个协议下导出服务
    for (ProtocolConfig protocolConfig : protocols) {
        doExportUrlsFor1Protocol(protocolConfig, registryURLs);
    }
}
```

​	上面代码首先是通过 loadRegistries 加载注册中心链接，然后再遍历 ProtocolConfig 集合导出每个服务。并在导出服务的过程中，将服务注册到注册中心。下面，我们先来看一下 loadRegistries 方法的逻辑。

```java
protected List<URL> loadRegistries(boolean provider) {
    // 检测是否存在注册中心配置类，不存在则抛出异常
    checkRegistry();
    List<URL> registryList = new ArrayList<URL>();
    if (registries != null && !registries.isEmpty()) {
        for (RegistryConfig config : registries) {
            String address = config.getAddress();
            if (address == null || address.length() == 0) {
                // 若 address 为空，则将其设为 0.0.0.0
                address = Constants.ANYHOST_VALUE;
            }

            // 从系统属性中加载注册中心地址
            String sysaddress = System.getProperty("dubbo.registry.address");
            if (sysaddress != null && sysaddress.length() > 0) {
                address = sysaddress;
            }
            // 检测 address 是否合法
            if (address.length() > 0 && !RegistryConfig.NO_AVAILABLE.equalsIgnoreCase(address)) {
                Map<String, String> map = new HashMap<String, String>();
                // 添加 ApplicationConfig 中的字段信息到 map 中
                appendParameters(map, application);
                // 添加 RegistryConfig 字段信息到 map 中
                appendParameters(map, config);
                
                // 添加 path、pid，protocol 等信息到 map 中
                map.put("path", RegistryService.class.getName());
                map.put("dubbo", Version.getProtocolVersion());
                map.put(Constants.TIMESTAMP_KEY, String.valueOf(System.currentTimeMillis()));
                if (ConfigUtils.getPid() > 0) {
                    map.put(Constants.PID_KEY, String.valueOf(ConfigUtils.getPid()));
                }
                if (!map.containsKey("protocol")) {
                    if (ExtensionLoader.getExtensionLoader(RegistryFactory.class).hasExtension("remote")) {
                        map.put("protocol", "remote");
                    } else {
                        map.put("protocol", "dubbo");
                    }
                }

                // 解析得到 URL 列表，address 可能包含多个注册中心 ip，
                // 因此解析得到的是一个 URL 列表
                List<URL> urls = UrlUtils.parseURLs(address, map);
                for (URL url : urls) {
                    url = url.addParameter(Constants.REGISTRY_KEY, url.getProtocol());
                    // 将 URL 协议头设置为 registry
                    url = url.setProtocol(Constants.REGISTRY_PROTOCOL);
                    // 通过判断条件，决定是否添加 url 到 registryList 中，条件如下：
                    // (服务提供者 && register = true 或 null) 
                    //    || (非服务提供者 && subscribe = true 或 null)
                    if ((provider && url.getParameter(Constants.REGISTER_KEY, true))
                            || (!provider && url.getParameter(Constants.SUBSCRIBE_KEY, true))) {
                        registryList.add(url);
                    }
                }
            }
        }
    }
    return registryList;
}
```

loadRegistries 方法主要包含如下的逻辑：

1. 检测是否存在注册中心配置类，不存在则抛出异常
2. 构建参数映射集合，也就是 map
3. 构建注册中心链接列表
4. 遍历链接列表，并根据条件决定是否将其添加到 registryList 中

关于多协议多注册中心导出服务就先分析到这，代码不是很多，接下来分析 URL 组装过程。

#### 组装 URL

​	配置检查完毕后，紧接着要做的事情是根据配置，以及其他一些信息组装 URL。前面说过，URL 是 Dubbo 配置的载体，通过 URL 可让 Dubbo 的各种配置在各个模块之间传递。URL 之于 Dubbo，犹如水之于鱼，非常重要。大家在阅读 Dubbo 服务导出相关源码的过程中，要注意 URL 内容的变化。既然 URL 如此重要，那么下面我们来了解一下 URL 组装的过程。

```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    String name = protocolConfig.getName();
    // 如果协议名为空，或空串，则将协议名变量设置为 dubbo
    if (name == null || name.length() == 0) {
        name = "dubbo";
    }

    Map<String, String> map = new HashMap<String, String>();
    // 添加 side、版本、时间戳以及进程号等信息到 map 中
    map.put(Constants.SIDE_KEY, Constants.PROVIDER_SIDE);
    map.put(Constants.DUBBO_VERSION_KEY, Version.getProtocolVersion());
    map.put(Constants.TIMESTAMP_KEY, String.valueOf(System.currentTimeMillis()));
    if (ConfigUtils.getPid() > 0) {
        map.put(Constants.PID_KEY, String.valueOf(ConfigUtils.getPid()));
    }

    // 通过反射将对象的字段信息添加到 map 中
    appendParameters(map, application);
    appendParameters(map, module);
    appendParameters(map, provider, Constants.DEFAULT_KEY);
    appendParameters(map, protocolConfig);
    appendParameters(map, this);

    // methods 为 MethodConfig 集合，MethodConfig 中存储了 <dubbo:method> 标签的配置信息
    if (methods != null && !methods.isEmpty()) {
        // 这段代码用于添加 Callback 配置到 map 中，代码太长，待会单独分析
    }

    // 检测 generic 是否为 "true"，并根据检测结果向 map 中添加不同的信息
    if (ProtocolUtils.isGeneric(generic)) {
        map.put(Constants.GENERIC_KEY, generic);
        map.put(Constants.METHODS_KEY, Constants.ANY_VALUE);
    } else {
        String revision = Version.getVersion(interfaceClass, version);
        if (revision != null && revision.length() > 0) {
            map.put("revision", revision);
        }

        // 为接口生成包裹类 Wrapper，Wrapper 中包含了接口的详细信息，比如接口方法名数组，字段信息等
        String[] methods = Wrapper.getWrapper(interfaceClass).getMethodNames();
        // 添加方法名到 map 中，如果包含多个方法名，则用逗号隔开，比如 method = init,destroy
        if (methods.length == 0) {
            logger.warn("NO method found in service interface ...");
            map.put(Constants.METHODS_KEY, Constants.ANY_VALUE);
        } else {
            // 将逗号作为分隔符连接方法名，并将连接后的字符串放入 map 中
            map.put(Constants.METHODS_KEY, StringUtils.join(new HashSet<String>(Arrays.asList(methods)), ","));
        }
    }

    // 添加 token 到 map 中
    if (!ConfigUtils.isEmpty(token)) {
        if (ConfigUtils.isDefault(token)) {
            // 随机生成 token
            map.put(Constants.TOKEN_KEY, UUID.randomUUID().toString());
        } else {
            map.put(Constants.TOKEN_KEY, token);
        }
    }
    // 判断协议名是否为 injvm
    if (Constants.LOCAL_PROTOCOL.equals(protocolConfig.getName())) {
        protocolConfig.setRegister(false);
        map.put("notify", "false");
    }

    // 获取上下文路径
    String contextPath = protocolConfig.getContextpath();
    if ((contextPath == null || contextPath.length() == 0) && provider != null) {
        contextPath = provider.getContextpath();
    }

    // 获取 host 和 port
    String host = this.findConfigedHosts(protocolConfig, registryURLs, map);
    Integer port = this.findConfigedPorts(protocolConfig, name, map);
    // 组装 URL
    URL url = new URL(name, host, port, (contextPath == null || contextPath.length() == 0 ? "" : contextPath + "/") + path, map);
    
    // 省略无关代码
}
```

​	上面的代码首先是将一些信息，比如版本、时间戳、方法名以及各种配置对象的字段信息放入到 map 中，map 中的内容将作为 URL 的查询字符串。构建好 map 后，紧接着是获取上下文路径、主机名以及端口号等信息。最后将 map 和主机名等数据传给 URL 构造方法创建 URL 对象。需要注意的是，这里出现的 URL 并非 java.net.URL，而是 com.alibaba.dubbo.common.URL。

​	上面省略了一段代码，这里简单分析一下。这段代码用于检测 <dubbo:method> 标签中的配置信息，并将相关配置添加到 map 中。代码如下：

```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    // ...

    // methods 为 MethodConfig 集合，MethodConfig 中存储了 <dubbo:method> 标签的配置信息
    if (methods != null && !methods.isEmpty()) {
        for (MethodConfig method : methods) {
            // 添加 MethodConfig 对象的字段信息到 map 中，键 = 方法名.属性名。
            // 比如存储 <dubbo:method name="sayHello" retries="2"> 对应的 MethodConfig，
            // 键 = sayHello.retries，map = {"sayHello.retries": 2, "xxx": "yyy"}
            appendParameters(map, method, method.getName());

            String retryKey = method.getName() + ".retry";
            if (map.containsKey(retryKey)) {
                String retryValue = map.remove(retryKey);
                // 检测 MethodConfig retry 是否为 false，若是，则设置重试次数为0
                if ("false".equals(retryValue)) {
                    map.put(method.getName() + ".retries", "0");
                }
            }
            
            // 获取 ArgumentConfig 列表
            List<ArgumentConfig> arguments = method.getArguments();
            if (arguments != null && !arguments.isEmpty()) {
                for (ArgumentConfig argument : arguments) {
                    // 检测 type 属性是否为空，或者空串（分支1 ⭐️）
                    if (argument.getType() != null && argument.getType().length() > 0) {
                        Method[] methods = interfaceClass.getMethods();
                        if (methods != null && methods.length > 0) {
                            for (int i = 0; i < methods.length; i++) {
                                String methodName = methods[i].getName();
                                // 比对方法名，查找目标方法
                                if (methodName.equals(method.getName())) {
                                    Class<?>[] argtypes = methods[i].getParameterTypes();
                                    if (argument.getIndex() != -1) {
                                        // 检测 ArgumentConfig 中的 type 属性与方法参数列表
                                        // 中的参数名称是否一致，不一致则抛出异常(分支2 ⭐️)
                                        if (argtypes[argument.getIndex()].getName().equals(argument.getType())) {
                                            // 添加 ArgumentConfig 字段信息到 map 中，
                                            // 键前缀 = 方法名.index，比如:
                                            // map = {"sayHello.3": true}
                                            appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                                        } else {
                                            throw new IllegalArgumentException("argument config error: ...");
                                        }
                                    } else {    // 分支3 ⭐️
                                        for (int j = 0; j < argtypes.length; j++) {
                                            Class<?> argclazz = argtypes[j];
                                            // 从参数类型列表中查找类型名称为 argument.type 的参数
                                            if (argclazz.getName().equals(argument.getType())) {
                                                appendParameters(map, argument, method.getName() + "." + j);
                                                if (argument.getIndex() != -1 && argument.getIndex() != j) {
                                                    throw new IllegalArgumentException("argument config error: ...");
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }

                    // 用户未配置 type 属性，但配置了 index 属性，且 index != -1
                    } else if (argument.getIndex() != -1) {    // 分支4 ⭐️
                        // 添加 ArgumentConfig 字段信息到 map 中
                        appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                    } else {
                        throw new IllegalArgumentException("argument config must set index or type");
                    }
                }
            }
        }
    }

    // ...
}
```

​	上面这段代码 for 循环和 if else 分支嵌套太多，导致层次太深，不利于阅读，需要耐心看一下。大家在看这段代码时，注意把几个重要的条件分支找出来。只要理解了这几个分支的意图，就可以弄懂这段代码。请注意上面代码中⭐️符号，这几个符号标识出了4个重要的分支，下面用伪代码解释一下这几个分支的含义。

```java
// 获取 ArgumentConfig 列表
for (遍历 ArgumentConfig 列表) {
    if (type 不为 null，也不为空串) {    // 分支1
        1. 通过反射获取 interfaceClass 的方法列表
        for (遍历方法列表) {
            1. 比对方法名，查找目标方法
        	2. 通过反射获取目标方法的参数类型数组 argtypes
            if (index != -1) {    // 分支2
                1. 从 argtypes 数组中获取下标 index 处的元素 argType
                2. 检测 argType 的名称与 ArgumentConfig 中的 type 属性是否一致
                3. 添加 ArgumentConfig 字段信息到 map 中，或抛出异常
            } else {    // 分支3
                1. 遍历参数类型数组 argtypes，查找 argument.type 类型的参数
                2. 添加 ArgumentConfig 字段信息到 map 中
            }
        }
    } else if (index != -1) {    // 分支4
		1. 添加 ArgumentConfig 字段信息到 map 中
    }
}
```

​	在本节分析的源码中，appendParameters 这个方法出现的次数比较多，该方法用于将对象字段信息添加到 map 中。实现上则是通过反射获取目标对象的 getter 方法，并调用该方法获取属性值。然后再通过 getter 方法名解析出属性名，比如从方法名 getName 中可解析出属性 name。如果用户传入了属性名前缀，此时需要将属性名加入前缀内容。最后将 <属性名，属性值> 键值对存入到 map 中就行了。限于篇幅原因，这里就不分析 appendParameters 方法的源码了，大家请自行分析。

### 导出 Dubbo 服务

​	前置工作做完，接下来就可以进行服务导出了。服务导出分为导出到本地 (JVM)，和导出到远程。在深入分析服务导出的源码前，我们先来从宏观层面上看一下服务导出逻辑。如下：

```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    
    // 省略无关代码
    
    if (ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
            .hasExtension(url.getProtocol())) {
        // 加载 ConfiguratorFactory，并生成 Configurator 实例，然后通过实例配置 url
        url = ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
                .getExtension(url.getProtocol()).getConfigurator(url).configure(url);
    }

    String scope = url.getParameter(Constants.SCOPE_KEY);
    // 如果 scope = none，则什么都不做
    if (!Constants.SCOPE_NONE.toString().equalsIgnoreCase(scope)) {
        // scope != remote，导出到本地
        if (!Constants.SCOPE_REMOTE.toString().equalsIgnoreCase(scope)) {
            exportLocal(url);
        }

        // scope != local，导出到远程
        if (!Constants.SCOPE_LOCAL.toString().equalsIgnoreCase(scope)) {
            if (registryURLs != null && !registryURLs.isEmpty()) {
                for (URL registryURL : registryURLs) {
                    url = url.addParameterIfAbsent(Constants.DYNAMIC_KEY, registryURL.getParameter(Constants.DYNAMIC_KEY));
                    // 加载监视器链接
                    URL monitorUrl = loadMonitor(registryURL);
                    if (monitorUrl != null) {
                        // 将监视器链接作为参数添加到 url 中
                        url = url.addParameterAndEncoded(Constants.MONITOR_KEY, monitorUrl.toFullString());
                    }

                    String proxy = url.getParameter(Constants.PROXY_KEY);
                    if (StringUtils.isNotEmpty(proxy)) {
                        registryURL = registryURL.addParameter(Constants.PROXY_KEY, proxy);
                    }

                    // 为服务提供类(ref)生成 Invoker
                    Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));
                    // DelegateProviderMetaDataInvoker 用于持有 Invoker 和 ServiceConfig
                    DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                    // 导出服务，并生成 Exporter
                    Exporter<?> exporter = protocol.export(wrapperInvoker);
                    exporters.add(exporter);
                }
                
            // 不存在注册中心，仅导出服务
            } else {
                Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, url);
                DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                Exporter<?> exporter = protocol.export(wrapperInvoker);
                exporters.add(exporter);
            }
        }
    }
    this.urls.add(url);
}
```

上面代码根据 url 中的 scope 参数决定服务导出方式，分别如下：

- scope = none，不导出服务
- scope != remote，导出到本地
- scope != local，导出到远程



​	不管是导出到本地，还是远程。进行服务导出之前，均需要先创建 Invoker，这是一个很重要的步骤。因此下面先来分析 Invoker 的创建过程。

###  Invoker 创建过程

​	在 Dubbo 中，Invoker 是一个非常重要的模型。在服务提供端，以及服务引用端均会出现 Invoker。Dubbo 官方文档中对 Invoker 进行了说明，这里引用一下。

> Invoker 是实体域，它是 Dubbo 的核心模型，其它模型都向它靠扰，或转换成它，它代表一个可执行体，可向它发起 invoke 调用，它有可能是一个本地的实现，也可能是一个远程的实现，也可能一个集群实现。

​	既然 Invoker 如此重要，那么我们很有必要搞清楚 Invoker 的用途。Invoker 是由 ProxyFactory 创建而来，Dubbo 默认的 ProxyFactory 实现类是 JavassistProxyFactory。下面我们到 JavassistProxyFactory 代码中，探索 Invoker 的创建过程。如下：

```java
public <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
	// 为目标类创建 Wrapper
    final Wrapper wrapper = Wrapper.getWrapper(proxy.getClass().getName().indexOf('$') < 0 ? proxy.getClass() : type);
    // 创建匿名 Invoker 类对象，并实现 doInvoke 方法。
    return new AbstractProxyInvoker<T>(proxy, type, url) {
        @Override
        protected Object doInvoke(T proxy, String methodName,
                                  Class<?>[] parameterTypes,
                                  Object[] arguments) throws Throwable {
			// 调用 Wrapper 的 invokeMethod 方法，invokeMethod 最终会调用目标方法
            return wrapper.invokeMethod(proxy, methodName, parameterTypes, arguments);
        }
    };
}
```

​	如上，JavassistProxyFactory 创建了一个继承自 AbstractProxyInvoker 类的匿名对象，并覆写了抽象方法 doInvoke。覆写后的 doInvoke 逻辑比较简单，仅是将调用请求转发给了 Wrapper 类的 invokeMethod 方法。Wrapper 用于“包裹”目标类，Wrapper 是一个抽象类，仅可通过 getWrapper(Class) 方法创建子类。在创建 Wrapper 子类的过程中，子类代码生成逻辑会对 getWrapper 方法传入的 Class 对象进行解析，拿到诸如类方法，类成员变量等信息。以及生成 invokeMethod 方法代码和其他一些方法代码。代码生成完毕后，通过 Javassist 生成 Class 对象，最后再通过反射创建 Wrapper 实例。相关的代码如下：

```java
 public static Wrapper getWrapper(Class<?> c) {	
    while (ClassGenerator.isDynamicClass(c))
        c = c.getSuperclass();

    if (c == Object.class)
        return OBJECT_WRAPPER;

    // 从缓存中获取 Wrapper 实例
    Wrapper ret = WRAPPER_MAP.get(c);
    if (ret == null) {
        // 缓存未命中，创建 Wrapper
        ret = makeWrapper(c);
        // 写入缓存
        WRAPPER_MAP.put(c, ret);
    }
    return ret;
}
```

​	getWrapper 方法仅包含一些缓存操作逻辑，不难理解。下面我们看一下 makeWrapper 方法。

```java
private static Wrapper makeWrapper(Class<?> c) {
    // 检测 c 是否为基本类型，若是则抛出异常
    if (c.isPrimitive())
        throw new IllegalArgumentException("Can not create wrapper for primitive type: " + c);

    String name = c.getName();
    ClassLoader cl = ClassHelper.getClassLoader(c);

    // c1 用于存储 setPropertyValue 方法代码
    StringBuilder c1 = new StringBuilder("public void setPropertyValue(Object o, String n, Object v){ ");
    // c2 用于存储 getPropertyValue 方法代码
    StringBuilder c2 = new StringBuilder("public Object getPropertyValue(Object o, String n){ ");
    // c3 用于存储 invokeMethod 方法代码
    StringBuilder c3 = new StringBuilder("public Object invokeMethod(Object o, String n, Class[] p, Object[] v) throws " + InvocationTargetException.class.getName() + "{ ");

    // 生成类型转换代码及异常捕捉代码，比如：
    //   DemoService w; try { w = ((DemoServcie) $1); }}catch(Throwable e){ throw new IllegalArgumentException(e); }
    c1.append(name).append(" w; try{ w = ((").append(name).append(")$1); }catch(Throwable e){ throw new IllegalArgumentException(e); }");
    c2.append(name).append(" w; try{ w = ((").append(name).append(")$1); }catch(Throwable e){ throw new IllegalArgumentException(e); }");
    c3.append(name).append(" w; try{ w = ((").append(name).append(")$1); }catch(Throwable e){ throw new IllegalArgumentException(e); }");

    // pts 用于存储成员变量名和类型
    Map<String, Class<?>> pts = new HashMap<String, Class<?>>();
    // ms 用于存储方法描述信息（可理解为方法签名）及 Method 实例
    Map<String, Method> ms = new LinkedHashMap<String, Method>();
    // mns 为方法名列表
    List<String> mns = new ArrayList<String>();
    // dmns 用于存储“定义在当前类中的方法”的名称
    List<String> dmns = new ArrayList<String>();

    // --------------------------------✨ 分割线1 ✨-------------------------------------

    // 获取 public 访问级别的字段，并为所有字段生成条件判断语句
    for (Field f : c.getFields()) {
        String fn = f.getName();
        Class<?> ft = f.getType();
        if (Modifier.isStatic(f.getModifiers()) || Modifier.isTransient(f.getModifiers()))
            // 忽略关键字 static 或 transient 修饰的变量
            continue;

        // 生成条件判断及赋值语句，比如：
        // if( $2.equals("name") ) { w.name = (java.lang.String) $3; return;}
        // if( $2.equals("age") ) { w.age = ((Number) $3).intValue(); return;}
        c1.append(" if( $2.equals(\"").append(fn).append("\") ){ w.").append(fn).append("=").append(arg(ft, "$3")).append("; return; }");

        // 生成条件判断及返回语句，比如：
        // if( $2.equals("name") ) { return ($w)w.name; }
        c2.append(" if( $2.equals(\"").append(fn).append("\") ){ return ($w)w.").append(fn).append("; }");

        // 存储 <字段名, 字段类型> 键值对到 pts 中
        pts.put(fn, ft);
    }

    // --------------------------------✨ 分割线2 ✨-------------------------------------

    Method[] methods = c.getMethods();
    // 检测 c 中是否包含在当前类中声明的方法
    boolean hasMethod = hasMethods(methods);
    if (hasMethod) {
        c3.append(" try{");
    }
    for (Method m : methods) {
        if (m.getDeclaringClass() == Object.class)
            // 忽略 Object 中定义的方法
            continue;

        String mn = m.getName();
        // 生成方法名判断语句，比如：
        // if ( "sayHello".equals( $2 )
        c3.append(" if( \"").append(mn).append("\".equals( $2 ) ");
        int len = m.getParameterTypes().length;
        // 生成“运行时传入的参数数量与方法参数列表长度”判断语句，比如：
        // && $3.length == 2
        c3.append(" && ").append(" $3.length == ").append(len);

        boolean override = false;
        for (Method m2 : methods) {
            // 检测方法是否存在重载情况，条件为：方法对象不同 && 方法名相同
            if (m != m2 && m.getName().equals(m2.getName())) {
                override = true;
                break;
            }
        }
        // 对重载方法进行处理，考虑下面的方法：
        //    1. void sayHello(Integer, String)
        //    2. void sayHello(Integer, Integer)
        // 方法名相同，参数列表长度也相同，因此不能仅通过这两项判断两个方法是否相等。
        // 需要进一步判断方法的参数类型
        if (override) {
            if (len > 0) {
                for (int l = 0; l < len; l++) {
                    // 生成参数类型进行检测代码，比如：
                    // && $3[0].getName().equals("java.lang.Integer") 
                    //    && $3[1].getName().equals("java.lang.String")
                    c3.append(" && ").append(" $3[").append(l).append("].getName().equals(\"")
                            .append(m.getParameterTypes()[l].getName()).append("\")");
                }
            }
        }

        // 添加 ) {，完成方法判断语句，此时生成的代码可能如下（已格式化）：
        // if ("sayHello".equals($2) 
        //     && $3.length == 2
        //     && $3[0].getName().equals("java.lang.Integer") 
        //     && $3[1].getName().equals("java.lang.String")) {
        c3.append(" ) { ");

        // 根据返回值类型生成目标方法调用语句
        if (m.getReturnType() == Void.TYPE)
            // w.sayHello((java.lang.Integer)$4[0], (java.lang.String)$4[1]); return null;
            c3.append(" w.").append(mn).append('(').append(args(m.getParameterTypes(), "$4")).append(");").append(" return null;");
        else
            // return w.sayHello((java.lang.Integer)$4[0], (java.lang.String)$4[1]);
            c3.append(" return ($w)w.").append(mn).append('(').append(args(m.getParameterTypes(), "$4")).append(");");

        // 添加 }, 生成的代码形如（已格式化）：
        // if ("sayHello".equals($2) 
        //     && $3.length == 2
        //     && $3[0].getName().equals("java.lang.Integer") 
        //     && $3[1].getName().equals("java.lang.String")) {
        //
        //     w.sayHello((java.lang.Integer)$4[0], (java.lang.String)$4[1]); 
        //     return null;
        // }
        c3.append(" }");

        // 添加方法名到 mns 集合中
        mns.add(mn);
        // 检测当前方法是否在 c 中被声明的
        if (m.getDeclaringClass() == c)
            // 若是，则将当前方法名添加到 dmns 中
            dmns.add(mn);
        ms.put(ReflectUtils.getDesc(m), m);
    }
    if (hasMethod) {
        // 添加异常捕捉语句
        c3.append(" } catch(Throwable e) { ");
        c3.append("     throw new java.lang.reflect.InvocationTargetException(e); ");
        c3.append(" }");
    }

    // 添加 NoSuchMethodException 异常抛出代码
    c3.append(" throw new " + NoSuchMethodException.class.getName() + "(\"Not found method \\\"\"+$2+\"\\\" in class " + c.getName() + ".\"); }");

    // --------------------------------✨ 分割线3 ✨-------------------------------------

    Matcher matcher;
    // 处理 get/set 方法
    for (Map.Entry<String, Method> entry : ms.entrySet()) {
        String md = entry.getKey();
        Method method = (Method) entry.getValue();
        // 匹配以 get 开头的方法
        if ((matcher = ReflectUtils.GETTER_METHOD_DESC_PATTERN.matcher(md)).matches()) {
            // 获取属性名
            String pn = propertyName(matcher.group(1));
            // 生成属性判断以及返回语句，示例如下：
            // if( $2.equals("name") ) { return ($w).w.getName(); }
            c2.append(" if( $2.equals(\"").append(pn).append("\") ){ return ($w)w.").append(method.getName()).append("(); }");
            pts.put(pn, method.getReturnType());

        // 匹配以 is/has/can 开头的方法
        } else if ((matcher = ReflectUtils.IS_HAS_CAN_METHOD_DESC_PATTERN.matcher(md)).matches()) {
            String pn = propertyName(matcher.group(1));
            // 生成属性判断以及返回语句，示例如下：
            // if( $2.equals("dream") ) { return ($w).w.hasDream(); }
            c2.append(" if( $2.equals(\"").append(pn).append("\") ){ return ($w)w.").append(method.getName()).append("(); }");
            pts.put(pn, method.getReturnType());

        // 匹配以 set 开头的方法
        } else if ((matcher = ReflectUtils.SETTER_METHOD_DESC_PATTERN.matcher(md)).matches()) {
            Class<?> pt = method.getParameterTypes()[0];
            String pn = propertyName(matcher.group(1));
            // 生成属性判断以及 setter 调用语句，示例如下：
            // if( $2.equals("name") ) { w.setName((java.lang.String)$3); return; }
            c1.append(" if( $2.equals(\"").append(pn).append("\") ){ w.").append(method.getName()).append("(").append(arg(pt, "$3")).append("); return; }");
            pts.put(pn, pt);
        }
    }

    // 添加 NoSuchPropertyException 异常抛出代码
    c1.append(" throw new " + NoSuchPropertyException.class.getName() + "(\"Not found property \\\"\"+$2+\"\\\" filed or setter method in class " + c.getName() + ".\"); }");
    c2.append(" throw new " + NoSuchPropertyException.class.getName() + "(\"Not found property \\\"\"+$2+\"\\\" filed or setter method in class " + c.getName() + ".\"); }");

    // --------------------------------✨ 分割线4 ✨-------------------------------------

    long id = WRAPPER_CLASS_COUNTER.getAndIncrement();
    // 创建类生成器
    ClassGenerator cc = ClassGenerator.newInstance(cl);
    // 设置类名及超类
    cc.setClassName((Modifier.isPublic(c.getModifiers()) ? Wrapper.class.getName() : c.getName() + "$sw") + id);
    cc.setSuperClass(Wrapper.class);

    // 添加默认构造方法
    cc.addDefaultConstructor();

    // 添加字段
    cc.addField("public static String[] pns;");
    cc.addField("public static " + Map.class.getName() + " pts;");
    cc.addField("public static String[] mns;");
    cc.addField("public static String[] dmns;");
    for (int i = 0, len = ms.size(); i < len; i++)
        cc.addField("public static Class[] mts" + i + ";");

    // 添加方法代码
    cc.addMethod("public String[] getPropertyNames(){ return pns; }");
    cc.addMethod("public boolean hasProperty(String n){ return pts.containsKey($1); }");
    cc.addMethod("public Class getPropertyType(String n){ return (Class)pts.get($1); }");
    cc.addMethod("public String[] getMethodNames(){ return mns; }");
    cc.addMethod("public String[] getDeclaredMethodNames(){ return dmns; }");
    cc.addMethod(c1.toString());
    cc.addMethod(c2.toString());
    cc.addMethod(c3.toString());

    try {
        // 生成类
        Class<?> wc = cc.toClass();
        
        // 设置字段值
        wc.getField("pts").set(null, pts);
        wc.getField("pns").set(null, pts.keySet().toArray(new String[0]));
        wc.getField("mns").set(null, mns.toArray(new String[0]));
        wc.getField("dmns").set(null, dmns.toArray(new String[0]));
        int ix = 0;
        for (Method m : ms.values())
            wc.getField("mts" + ix++).set(null, m.getParameterTypes());

        // 创建 Wrapper 实例
        return (Wrapper) wc.newInstance();
    } catch (RuntimeException e) {
        throw e;
    } catch (Throwable e) {
        throw new RuntimeException(e.getMessage(), e);
    } finally {
        cc.release();
        ms.clear();
        mns.clear();
        dmns.clear();
    }
}
```

​	上面代码很长，大家耐心看一下。我们在上面代码中做了大量的注释，并按功能对代码进行了分块，以帮助大家理解代码逻辑。下面对这段代码进行讲解。首先我们把目光移到分割线1之上的代码，这段代码主要用于进行一些初始化操作。比如创建 c1、c2、c3 以及 pts、ms、mns 等变量，以及向 c1、c2、c3 中添加方法定义和类型转换代码。接下来是分割线1到分割线2之间的代码，这段代码用于为 public 级别的字段生成条件判断取值与赋值代码。这段代码不是很难看懂，就不多说了。继续向下看，分割线2和分隔线3之间的代码用于为定义在当前类中的方法生成判断语句，和方法调用语句。因为需要对方法重载进行校验，因此到这这段代码看起来有点复杂。不过耐心看一下，也不是很难理解。接下来是分割线3和分隔线4之间的代码，这段代码用于处理 getter、setter 以及以 is/has/can 开头的方法。处理方式是通过正则表达式获取方法类型（get/set/is/…），以及属性名。之后为属性名生成判断语句，然后为方法生成调用语句。最后我们再来看一下分隔线4以下的代码，这段代码通过 ClassGenerator 为刚刚生成的代码构建 Class 类，并通过反射创建对象。ClassGenerator 是 Dubbo 自己封装的，该类的核心是 toClass() 的重载方法 toClass(ClassLoader, ProtectionDomain)，该方法通过 javassist 构建 Class。这里就不分析 toClass 方法了，大家请自行分析。

​	阅读 Wrapper 类代码需要对 javassist 框架有所了解。关于 javassist，大家如果不熟悉，请自行查阅资料，本节不打算介绍 javassist 相关内容。

​	好了，关于 Wrapper 类生成过程就分析到这。如果大家看的不是很明白，可以单独为 Wrapper 创建单元测试，然后单步调试。并将生成的代码拷贝出来，格式化后再进行观察和理解。

### 导出服务到本地

​	本节我们来看一下服务导出相关的代码，按照代码执行顺序，本节先来分析导出服务到本地的过程。相关代码如下：

```java
private void exportLocal(URL url) {
    // 如果 URL 的协议头等于 injvm，说明已经导出到本地了，无需再次导出
    if (!Constants.LOCAL_PROTOCOL.equalsIgnoreCase(url.getProtocol())) {
        URL local = URL.valueOf(url.toFullString())
            .setProtocol(Constants.LOCAL_PROTOCOL)    // 设置协议头为 injvm
            .setHost(LOCALHOST)
            .setPort(0);
        ServiceClassHolder.getInstance().pushServiceClass(getServiceClass(ref));
        // 创建 Invoker，并导出服务，这里的 protocol 会在运行时调用 InjvmProtocol 的 export 方法
        Exporter<?> exporter = protocol.export(
            proxyFactory.getInvoker(ref, (Class) interfaceClass, local));
        exporters.add(exporter);
    }
}
```

​	exportLocal 方法比较简单，首先根据 URL 协议头决定是否导出服务。若需导出，则创建一个新的 URL 并将协议头、主机名以及端口设置成新的值。然后创建 Invoker，并调用 InjvmProtocol 的 export 方法导出服务。下面我们来看一下 InjvmProtocol 的 export 方法都做了哪些事情。

```java
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    // 创建 InjvmExporter
    return new InjvmExporter<T>(invoker, invoker.getUrl().getServiceKey(), exporterMap);
}
```

​	如上，InjvmProtocol 的 export 方法仅创建了一个 InjvmExporter，无其他逻辑。到此导出服务到本地就分析完了，接下来，我们继续分析导出服务到远程的过程。

###  导出服务到远程

​	与导出服务到本地相比，导出服务到远程的过程要复杂不少，其包含了服务导出与服务注册两个过程。这两个过程涉及到了大量的调用，比较复杂。按照代码执行顺序，本节先来分析服务导出逻辑，服务注册逻辑将在下一节进行分析。下面开始分析，我们把目光移动到 RegistryProtocol 的 export 方法上。

```java
public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
    // 导出服务
    final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker);

    // 获取注册中心 URL，以 zookeeper 注册中心为例，得到的示例 URL 如下：
    // zookeeper://127.0.0.1:2181/com.alibaba.dubbo.registry.RegistryService?application=demo-provider&dubbo=2.0.2&export=dubbo%3A%2F%2F172.17.48.52%3A20880%2Fcom.alibaba.dubbo.demo.DemoService%3Fanyhost%3Dtrue%26application%3Ddemo-provider
    URL registryUrl = getRegistryUrl(originInvoker);

    // 根据 URL 加载 Registry 实现类，比如 ZookeeperRegistry
    final Registry registry = getRegistry(originInvoker);
    
    // 获取已注册的服务提供者 URL，比如：
    // dubbo://172.17.48.52:20880/com.alibaba.dubbo.demo.DemoService?anyhost=true&application=demo-provider&dubbo=2.0.2&generic=false&interface=com.alibaba.dubbo.demo.DemoService&methods=sayHello
    final URL registeredProviderUrl = getRegisteredProviderUrl(originInvoker);

    // 获取 register 参数
    boolean register = registeredProviderUrl.getParameter("register", true);

    // 向服务提供者与消费者注册表中注册服务提供者
    ProviderConsumerRegTable.registerProvider(originInvoker, registryUrl, registeredProviderUrl);

    // 根据 register 的值决定是否注册服务
    if (register) {
        // 向注册中心注册服务
        register(registryUrl, registeredProviderUrl);
        ProviderConsumerRegTable.getProviderWrapper(originInvoker).setReg(true);
    }

    // 获取订阅 URL，比如：
    // provider://172.17.48.52:20880/com.alibaba.dubbo.demo.DemoService?category=configurators&check=false&anyhost=true&application=demo-provider&dubbo=2.0.2&generic=false&interface=com.alibaba.dubbo.demo.DemoService&methods=sayHello
    final URL overrideSubscribeUrl = getSubscribedOverrideUrl(registeredProviderUrl);
    // 创建监听器
    final OverrideListener overrideSubscribeListener = new OverrideListener(overrideSubscribeUrl, originInvoker);
    overrideListeners.put(overrideSubscribeUrl, overrideSubscribeListener);
    // 向注册中心进行订阅 override 数据
    registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);
    // 创建并返回 DestroyableExporter
    return new DestroyableExporter<T>(exporter, originInvoker, overrideSubscribeUrl, registeredProviderUrl);
}
```

上面代码看起来比较复杂，主要做如下一些操作：

1. 调用 doLocalExport 导出服务
2. 向注册中心注册服务
3. 向注册中心进行订阅 override 数据
4. 创建并返回 DestroyableExporter



​	在以上操作中，除了创建并返回 DestroyableExporter 没什么难度外，其他几步操作都不是很简单。这其中，导出服务和注册服务是本章要重点分析的逻辑。 订阅 override 数据并非本文重点内容，后面会简单介绍一下。下面先来分析 doLocalExport 方法的逻辑，如下：

```java
private <T> ExporterChangeableWrapper<T> doLocalExport(final Invoker<T> originInvoker) {
    String key = getCacheKey(originInvoker);
    // 访问缓存
    ExporterChangeableWrapper<T> exporter = (ExporterChangeableWrapper<T>) bounds.get(key);
    if (exporter == null) {
        synchronized (bounds) {
            exporter = (ExporterChangeableWrapper<T>) bounds.get(key);
            if (exporter == null) {
                // 创建 Invoker 为委托类对象
                final Invoker<?> invokerDelegete = new InvokerDelegete<T>(originInvoker, getProviderUrl(originInvoker));
                // 调用 protocol 的 export 方法导出服务
                exporter = new ExporterChangeableWrapper<T>((Exporter<T>) protocol.export(invokerDelegete), originInvoker);
                
                // 写缓存
                bounds.put(key, exporter);
            }
        }
    }
    return exporter;
}
```

​	上面的代码是典型的双重检查锁，大家在阅读 Dubbo 的源码中，会多次见到。接下来，我们把重点放在 Protocol 的 export 方法上。假设运行时协议为 dubbo，此处的 protocol 变量会在运行时加载 DubboProtocol，并调用 DubboProtocol 的 export 方法。所以，接下来我们目光转移到 DubboProtocol 的 export 方法上，相关分析如下：

```java
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    URL url = invoker.getUrl();

    // 获取服务标识，理解成服务坐标也行。由服务组名，服务名，服务版本号以及端口组成。比如：
    // demoGroup/com.alibaba.dubbo.demo.DemoService:1.0.1:20880
    String key = serviceKey(url);
    // 创建 DubboExporter
    DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
    // 将 <key, exporter> 键值对放入缓存中
    exporterMap.put(key, exporter);

    // 本地存根相关代码
    Boolean isStubSupportEvent = url.getParameter(Constants.STUB_EVENT_KEY, Constants.DEFAULT_STUB_EVENT);
    Boolean isCallbackservice = url.getParameter(Constants.IS_CALLBACK_SERVICE, false);
    if (isStubSupportEvent && !isCallbackservice) {
        String stubServiceMethods = url.getParameter(Constants.STUB_EVENT_METHODS_KEY);
        if (stubServiceMethods == null || stubServiceMethods.length() == 0) {
            // 省略日志打印代码
        } else {
            stubServiceMethodsMap.put(url.getServiceKey(), stubServiceMethods);
        }
    }

    // 启动服务器
    openServer(url);
    // 优化序列化
    optimizeSerialization(url);
    return exporter;
}
```

​	如上，我们重点关注 DubboExporter 的创建以及 openServer 方法，其他逻辑看不懂也没关系，不影响理解服务导出过程。另外，DubboExporter 的代码比较简单，就不分析了。下面分析 openServer 方法。

```java
private void openServer(URL url) {
    // 获取 host:port，并将其作为服务器实例的 key，用于标识当前的服务器实例
    String key = url.getAddress();
    boolean isServer = url.getParameter(Constants.IS_SERVER_KEY, true);
    if (isServer) {
        // 访问缓存
        ExchangeServer server = serverMap.get(key);
        if (server == null) {
            // 创建服务器实例
            serverMap.put(key, createServer(url));
        } else {
            // 服务器已创建，则根据 url 中的配置重置服务器
            server.reset(url);
        }
    }
}
```

​	如上，在同一台机器上（单网卡），同一个端口上仅允许启动一个服务器实例。若某个端口上已有服务器实例，此时则调用 reset 方法重置服务器的一些配置。考虑到篇幅问题，关于服务器实例重置的代码就不分析了。接下来分析服务器实例的创建过程。如下：

```java
private ExchangeServer createServer(URL url) {
    url = url.addParameterIfAbsent(Constants.CHANNEL_READONLYEVENT_SENT_KEY,
    // 添加心跳检测配置到 url 中
    url = url.addParameterIfAbsent(Constants.HEARTBEAT_KEY, String.valueOf(Constants.DEFAULT_HEARTBEAT));
	// 获取 server 参数，默认为 netty
    String str = url.getParameter(Constants.SERVER_KEY, Constants.DEFAULT_REMOTING_SERVER);

	// 通过 SPI 检测是否存在 server 参数所代表的 Transporter 拓展，不存在则抛出异常
    if (str != null && str.length() > 0 && !ExtensionLoader.getExtensionLoader(Transporter.class).hasExtension(str))
        throw new RpcException("Unsupported server type: " + str + ", url: " + url);

    // 添加编码解码器参数
    url = url.addParameter(Constants.CODEC_KEY, DubboCodec.NAME);
    ExchangeServer server;
    try {
        // 创建 ExchangeServer
        server = Exchangers.bind(url, requestHandler);
    } catch (RemotingException e) {
        throw new RpcException("Fail to start server...");
    }
                                   
	// 获取 client 参数，可指定 netty，mina
    str = url.getParameter(Constants.CLIENT_KEY);
    if (str != null && str.length() > 0) {
        // 获取所有的 Transporter 实现类名称集合，比如 supportedTypes = [netty, mina]
        Set<String> supportedTypes = ExtensionLoader.getExtensionLoader(Transporter.class).getSupportedExtensions();
        // 检测当前 Dubbo 所支持的 Transporter 实现类名称列表中，
        // 是否包含 client 所表示的 Transporter，若不包含，则抛出异常
        if (!supportedTypes.contains(str)) {
            throw new RpcException("Unsupported client type...");
        }
    }
    return server;
}
```

如上，createServer 包含三个核心的逻辑。

1. 检测是否存在 server 参数所代表的 Transporter 拓展，不存在则抛出异常。

2. 创建服务器实例。

3. 检测是否支持 client 参数所表示的 Transporter 拓展，不存在也是抛出异常。

   

两次检测操作所对应的代码比较直白了，无需多说。但创建服务器的操作目前还不是很清晰，我们继续往下看。



```java
public static ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    if (handler == null) {
        throw new IllegalArgumentException("handler == null");
    }
    url = url.addParameterIfAbsent(Constants.CODEC_KEY, "exchange");
    // 获取 Exchanger，默认为 HeaderExchanger。
    // 紧接着调用 HeaderExchanger 的 bind 方法创建 ExchangeServer 实例
    return getExchanger(url).bind(url, handler);
}
```

​	上面代码比较简单，就不多说了。下面看一下 HeaderExchanger 的 bind 方法。

​	HeaderExchanger 的 bind 方法包含的逻辑比较多，但目前我们仅需关心 Transporters 的 bind 方法逻辑即可。该方法的代码如下：

```java
public static Server bind(URL url, ChannelHandler... handlers) throws RemotingException {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    if (handlers == null || handlers.length == 0) {
        throw new IllegalArgumentException("handlers == null");
    }
    ChannelHandler handler;
    if (handlers.length == 1) {
        handler = handlers[0];
    } else {
    	// 如果 handlers 元素数量大于1，则创建 ChannelHandler 分发器
        handler = new ChannelHandlerDispatcher(handlers);
    }
    // 获取自适应 Transporter 实例，并调用实例方法
    return getTransporter().bind(url, handler);
}
```

​	如上，getTransporter() 方法获取的 Transporter 是在运行时动态创建的，类名为 Transporter$Adaptive，也就是自适应拓展类。Transporter$Adaptive 会在运行时根据传入的 URL 参数决定加载什么类型的 Transporter，默认为 NettyTransporter。下面我们继续跟下去，这次分析的是 NettyTransporter 的 bind 方法。

```java
public Server bind(URL url, ChannelHandler listener) throws RemotingException {
	// 创建 NettyServer
	return new NettyServer(url, listener);
}
```

​	这里仅有一句创建 NettyServer 的代码，无需多说，我们继续向下看。

```java
public class NettyServer extends AbstractServer implements Server {
    public NettyServer(URL url, ChannelHandler handler) throws RemotingException {
        // 调用父类构造方法
        super(url, ChannelHandlers.wrap(handler, ExecutorUtil.setThreadName(url, SERVER_THREAD_POOL_NAME)));
    }
}


public abstract class AbstractServer extends AbstractEndpoint implements Server {
    public AbstractServer(URL url, ChannelHandler handler) throws RemotingException {
        // 调用父类构造方法，这里就不用跟进去了，没什么复杂逻辑
        super(url, handler);
        localAddress = getUrl().toInetSocketAddress();

        // 获取 ip 和端口
        String bindIp = getUrl().getParameter(Constants.BIND_IP_KEY, getUrl().getHost());
        int bindPort = getUrl().getParameter(Constants.BIND_PORT_KEY, getUrl().getPort());
        if (url.getParameter(Constants.ANYHOST_KEY, false) || NetUtils.isInvalidLocalHost(bindIp)) {
            // 设置 ip 为 0.0.0.0
            bindIp = NetUtils.ANYHOST;
        }
        bindAddress = new InetSocketAddress(bindIp, bindPort);
        // 获取最大可接受连接数
        this.accepts = url.getParameter(Constants.ACCEPTS_KEY, Constants.DEFAULT_ACCEPTS);
        this.idleTimeout = url.getParameter(Constants.IDLE_TIMEOUT_KEY, Constants.DEFAULT_IDLE_TIMEOUT);
        try {
            // 调用模板方法 doOpen 启动服务器
            doOpen();
        } catch (Throwable t) {
            throw new RemotingException("Failed to bind ");
        }

        DataStore dataStore = ExtensionLoader.getExtensionLoader(DataStore.class).getDefaultExtension();
        executor = (ExecutorService) dataStore.get(Constants.EXECUTOR_SERVICE_COMPONENT_KEY, Integer.toString(url.getPort()));
    }
    
    protected abstract void doOpen() throws Throwable;

    protected abstract void doClose() throws Throwable;
}
```

​	上面代码多为赋值代码，不需要多讲。我们重点关注 doOpen 抽象方法，该方法需要子类实现。下面回到 NettyServer 中。

```java
protected void doOpen() throws Throwable {
    NettyHelper.setNettyLoggerFactory();
    // 创建 boss 和 worker 线程池
    ExecutorService boss = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerBoss", true));
    ExecutorService worker = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerWorker", true));
    ChannelFactory channelFactory = new NioServerSocketChannelFactory(boss, worker, getUrl().getPositiveParameter(Constants.IO_THREADS_KEY, Constants.DEFAULT_IO_THREADS));
    
    // 创建 ServerBootstrap
    bootstrap = new ServerBootstrap(channelFactory);

    final NettyHandler nettyHandler = new NettyHandler(getUrl(), this);
    channels = nettyHandler.getChannels();
    bootstrap.setOption("child.tcpNoDelay", true);
    // 设置 PipelineFactory
    bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
        @Override
        public ChannelPipeline getPipeline() {
            NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyServer.this);
            ChannelPipeline pipeline = Channels.pipeline();
            pipeline.addLast("decoder", adapter.getDecoder());
            pipeline.addLast("encoder", adapter.getEncoder());
            pipeline.addLast("handler", nettyHandler);
            return pipeline;
        }
    });
    // 绑定到指定的 ip 和端口上
    channel = bootstrap.bind(getBindAddress());
}
```

​	以上就是 NettyServer 创建的过程，dubbo 默认使用的 NettyServer 是基于 netty 3.x 版本实现的，比较老了。因此 Dubbo 另外提供了 netty 4.x 版本的 NettyServer，大家可在使用 Dubbo 的过程中按需进行配置。

​	到此，关于服务导出的过程就分析完了。整个过程比较复杂，大家在分析的过程中耐心一些。并且多写 Demo 进行调试，以便能够更好的理解代码逻辑。

​	本节内容先到这里，接下来分析服务导出的另一块逻辑 — 服务注册。

###  服务注册

​	本节我们来分析服务注册过程，服务注册操作对于 Dubbo 来说不是必需的，通过服务直连的方式就可以绕过注册中心。但通常我们不会这么做，直连方式不利于服务治理，仅推荐在测试服务时使用。对于 Dubbo 来说，注册中心虽不是必需，但却是必要的。因此，关于注册中心以及服务注册相关逻辑，我们也需要搞懂。

​	本节内容以 Zookeeper 注册中心作为分析目标，其他类型注册中心大家可自行分析。下面从服务注册的入口方法开始分析，我们把目光再次移到 RegistryProtocol 的 export 方法上。如下：

```java
public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
    
    // ${导出服务}
    
    // 省略其他代码
    
    boolean register = registeredProviderUrl.getParameter("register", true);
    if (register) {
        // 注册服务
        register(registryUrl, registeredProviderUrl);
        ProviderConsumerRegTable.getProviderWrapper(originInvoker).setReg(true);
    }
    
    final URL overrideSubscribeUrl = getSubscribedOverrideUrl(registeredProviderUrl);
    final OverrideListener overrideSubscribeListener = new OverrideListener(overrideSubscribeUrl, originInvoker);
    overrideListeners.put(overrideSubscribeUrl, overrideSubscribeListener);
    // 订阅 override 数据
    registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);

    // 省略部分代码
}
```

​	RegistryProtocol 的 export 方法包含了服务导出，注册，以及数据订阅等逻辑。其中服务导出逻辑上一节已经分析过了，本节将分析服务注册逻辑，相关代码如下：

```java
public void register(URL registryUrl, URL registedProviderUrl) {
    // 获取 Registry
    Registry registry = registryFactory.getRegistry(registryUrl);
    // 注册服务
    registry.register(registedProviderUrl);
}
```

​	register 方法包含两步操作，第一步是获取注册中心实例，第二步是向注册中心注册服务。接下来分两节内容对这两步操作进行分析。

#### 创建注册中心

​	本节内容以 Zookeeper 注册中心为例进行分析。下面先来看一下 getRegistry 方法的源码，这个方法由 AbstractRegistryFactory 实现。如下：

```java
public Registry getRegistry(URL url) {
    url = url.setPath(RegistryService.class.getName())
            .addParameter(Constants.INTERFACE_KEY, RegistryService.class.getName())
            .removeParameters(Constants.EXPORT_KEY, Constants.REFER_KEY);
    String key = url.toServiceString();
    LOCK.lock();
    try {
    	// 访问缓存
        Registry registry = REGISTRIES.get(key);
        if (registry != null) {
            return registry;
        }
        
        // 缓存未命中，创建 Registry 实例
        registry = createRegistry(url);
        if (registry == null) {
            throw new IllegalStateException("Can not create registry...");
        }
        
        // 写入缓存
        REGISTRIES.put(key, registry);
        return registry;
    } finally {
        LOCK.unlock();
    }
}

protected abstract Registry createRegistry(URL url);
```

​	如上，getRegistry 方法先访问缓存，缓存未命中则调用 createRegistry 创建 Registry，然后写入缓存。这里的 createRegistry 是一个模板方法，由具体的子类实现。因此，下面我们到 ZookeeperRegistryFactory 中探究一番。

```java
public class ZookeeperRegistryFactory extends AbstractRegistryFactory {

    // zookeeperTransporter 由 SPI 在运行时注入，类型为 ZookeeperTransporter$Adaptive
    private ZookeeperTransporter zookeeperTransporter;

    public void setZookeeperTransporter(ZookeeperTransporter zookeeperTransporter) {
        this.zookeeperTransporter = zookeeperTransporter;
    }

    @Override
    public Registry createRegistry(URL url) {
        // 创建 ZookeeperRegistry
        return new ZookeeperRegistry(url, zookeeperTransporter);
    }
}
```

ZookeeperRegistryFactory 的 createRegistry 方法仅包含一句代码，无需解释，继续跟下去。

```java
public ZookeeperRegistry(URL url, ZookeeperTransporter zookeeperTransporter) {
    super(url);
    if (url.isAnyHost()) {
        throw new IllegalStateException("registry address == null");
    }
    
    // 获取组名，默认为 dubbo
    String group = url.getParameter(Constants.GROUP_KEY, DEFAULT_ROOT);
    if (!group.startsWith(Constants.PATH_SEPARATOR)) {
        // group = "/" + group
        group = Constants.PATH_SEPARATOR + group;
    }
    this.root = group;
    // 创建 Zookeeper 客户端，默认为 CuratorZookeeperTransporter
    zkClient = zookeeperTransporter.connect(url);
    // 添加状态监听器
    zkClient.addStateListener(new StateListener() {
        @Override
        public void stateChanged(int state) {
            if (state == RECONNECTED) {
                try {
                    recover();
                } catch (Exception e) {
                    logger.error(e.getMessage(), e);
                }
            }
        }
    });
}
```

​	在上面的代码代码中，我们重点关注 ZookeeperTransporter 的 connect 方法调用，这个方法用于创建 Zookeeper 客户端。创建好 Zookeeper 客户端，意味着注册中心的创建过程就结束了。接下来，再来分析一下 Zookeeper 客户端的创建过程。

​	前面说过，这里的 zookeeperTransporter 类型为自适应拓展类，因此 connect 方法会在被调用时决定加载什么类型的 ZookeeperTransporter 拓展，默认为 CuratorZookeeperTransporter。下面我们到 CuratorZookeeperTransporter 中看一看。

```java
public ZookeeperClient connect(URL url) {
    // 创建 CuratorZookeeperClient
    return new CuratorZookeeperClient(url);
}
```

继续向下看。

```java
public class CuratorZookeeperClient extends AbstractZookeeperClient<CuratorWatcher> {

    private final CuratorFramework client;
    
    public CuratorZookeeperClient(URL url) {
        super(url);
        try {
            // 创建 CuratorFramework 构造器
            CuratorFrameworkFactory.Builder builder = CuratorFrameworkFactory.builder()
                    .connectString(url.getBackupAddress())
                    .retryPolicy(new RetryNTimes(1, 1000))
                    .connectionTimeoutMs(5000);
            String authority = url.getAuthority();
            if (authority != null && authority.length() > 0) {
                builder = builder.authorization("digest", authority.getBytes());
            }
            // 构建 CuratorFramework 实例
            client = builder.build();
            // 添加监听器
            client.getConnectionStateListenable().addListener(new ConnectionStateListener() {
                @Override
                public void stateChanged(CuratorFramework client, ConnectionState state) {
                    if (state == ConnectionState.LOST) {
                        CuratorZookeeperClient.this.stateChanged(StateListener.DISCONNECTED);
                    } else if (state == ConnectionState.CONNECTED) {
                        CuratorZookeeperClient.this.stateChanged(StateListener.CONNECTED);
                    } else if (state == ConnectionState.RECONNECTED) {
                        CuratorZookeeperClient.this.stateChanged(StateListener.RECONNECTED);
                    }
                }
            });
            
            // 启动客户端
            client.start();
        } catch (Exception e) {
            throw new IllegalStateException(e.getMessage(), e);
        }
    }
}
```

CuratorZookeeperClient 构造方法主要用于创建和启动 CuratorFramework 实例。以上基本上都是 Curator 框架的代码，大家如果对 Curator 框架不是很了解，可以参考 Curator 官方文档。

本节分析了 ZookeeperRegistry 实例的创建过程，整个过程并不是很复杂。大家在看完分析后，可以自行调试，以加深理解。现在注册中心实例创建好了，接下来要做的事情是向注册中心注册服务，我们继续往下看。

#### 节点创建

​	以 Zookeeper 为例，所谓的服务注册，本质上是将服务配置数据写入到 Zookeeper 的某个路径的节点下。为了让大家有一个直观的了解，下面我们将 Dubbo 的 demo 跑起来，然后通过 Zookeeper 可视化客户端 [ZooInspector](https://github.com/apache/zookeeper/tree/b79af153d0f98a4f3f3516910ed47234d7b3d74e/src/contrib/zooinspector) 查看节点数据。如下：

![](https://pic.imgdb.cn/item/611bc2b04907e2d39caab4b9.jpg)

从上图中可以看到 com.alibaba.dubbo.demo.DemoService 这个服务对应的配置信息（存储在 URL 中）最终被注册到了 /dubbo/com.alibaba.dubbo.demo.DemoService/providers/ 节点下。搞懂了服务注册的本质，那么接下来我们就可以去阅读服务注册的代码了。服务注册的接口为 register(URL)，这个方法定义在 FailbackRegistry 抽象类中。代码如下：

```java
public void register(URL url) {
    super.register(url);
    failedRegistered.remove(url);
    failedUnregistered.remove(url);
    try {
        // 模板方法，由子类实现
        doRegister(url);
    } catch (Exception e) {
        Throwable t = e;

        // 获取 check 参数，若 check = true 将会直接抛出异常
        boolean check = getUrl().getParameter(Constants.CHECK_KEY, true)
                && url.getParameter(Constants.CHECK_KEY, true)
                && !Constants.CONSUMER_PROTOCOL.equals(url.getProtocol());
        boolean skipFailback = t instanceof SkipFailbackWrapperException;
        if (check || skipFailback) {
            if (skipFailback) {
                t = t.getCause();
            }
            throw new IllegalStateException("Failed to register");
        } else {
            logger.error("Failed to register");
        }

        // 记录注册失败的链接
        failedRegistered.add(url);
    }
}

protected abstract void doRegister(URL url);
```

如上，我们重点关注 doRegister 方法调用即可，其他的代码先忽略。doRegister 方法是一个模板方法，因此我们到 FailbackRegistry 子类 ZookeeperRegistry 中进行分析。如下：

```java
protected void doRegister(URL url) {
    try {
        // 通过 Zookeeper 客户端创建节点，节点路径由 toUrlPath 方法生成，路径格式如下:
        //   /${group}/${serviceInterface}/providers/${url}
        // 比如
        //   /dubbo/org.apache.dubbo.DemoService/providers/dubbo%3A%2F%2F127.0.0.1......
        zkClient.create(toUrlPath(url), url.getParameter(Constants.DYNAMIC_KEY, true));
    } catch (Throwable e) {
        throw new RpcException("Failed to register...");
    }
}
```

​	如上，ZookeeperRegistry 在 doRegister 中调用了 Zookeeper 客户端创建服务节点。节点路径由 toUrlPath 方法生成，该方法逻辑不难理解，就不分析了。接下来分析 create 方法，如下：

```java
public void create(String path, boolean ephemeral) {
    if (!ephemeral) {
        // 如果要创建的节点类型非临时节点，那么这里要检测节点是否存在
        if (checkExists(path)) {
            return;
        }
    }
    int i = path.lastIndexOf('/');
    if (i > 0) {
        // 递归创建上一级路径
        create(path.substring(0, i), false);
    }
    
    // 根据 ephemeral 的值创建临时或持久节点
    if (ephemeral) {
        createEphemeral(path);
    } else {
        createPersistent(path);
    }
}
```

​	上面方法先是通过递归创建当前节点的上一级路径，然后再根据 ephemeral 的值决定创建临时还是持久节点。createEphemeral 和 createPersistent 这两个方法都比较简单，这里简单分析其中的一个。如下：

```java
public void createEphemeral(String path) {
    try {
        // 通过 Curator 框架创建节点
        client.create().withMode(CreateMode.EPHEMERAL).forPath(path);
    } catch (NodeExistsException e) {
    } catch (Exception e) {
        throw new IllegalStateException(e.getMessage(), e);
    }
}
```

### 订阅 override 数据 todo

## 总结

​	本篇文章详细分析了 Dubbo 服务导出过程，包括配置检测，URL 组装，Invoker 创建过程、导出服务以及注册服务等等



## 3.0源码分析

入口函数在DubboBootstrap的exportServices()中。

```java
private void exportServices() {
    for (ServiceConfigBase sc : configManager.getServices()) {
        // TODO, compatible with ServiceConfig.export()
        ServiceConfig<?> serviceConfig = (ServiceConfig<?>) sc;
        serviceConfig.setBootstrap(this);
        if (!serviceConfig.isRefreshed()) {
            serviceConfig.refresh();
        }

        if (sc.shouldExportAsync()) {
            ExecutorService executor = executorRepository.getExportReferExecutor();
            CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
                try {
                    if (!sc.isExported()) {
                        // 异步导出
                        sc.export();
                        exportedServices.add(sc);
                    }
                } catch (Throwable t) {
                    logger.error("export async catch error : " + t.getMessage(), t);
                }
            }, executor);

            asyncExportingFutures.add(future);
        } else {
            if (!sc.isExported()) {
                // 同步导出
                sc.export();
                exportedServices.add(sc);
            }
        }
    }
}
```

重点关注ServiceConfig的export方法

```java
public synchronized void export() {
    // 应该导出但是尚未导出
    if (this.shouldExport() && !this.exported) {
        // 初始化
        this.init();

        // check bootstrap state
        if (!bootstrap.isInitialized()) {
            throw new IllegalStateException("DubboBootstrap is not initialized");
        }
		// 配置属性覆盖
        if (!this.isRefreshed()) {
            this.refresh();
        }

        if (!shouldExport()) {
            return;
        }
		// delay > 0，延时导出服务
        if (shouldDelay()) {
            DELAY_EXPORT_EXECUTOR.schedule(this::doExport, getDelay(), TimeUnit.MILLISECONDS);
        } else {
            // 立即导出服务
            doExport();
        }

        if (this.bootstrap.getTakeoverMode() == BootstrapTakeoverMode.AUTO) {
            this.bootstrap.start();
        }
    }
}
```



```java
protected synchronized void doExport() {
    if (unexported) {
        throw new IllegalStateException("The service " + interfaceClass.getName() + " has already unexported!");
    }
    if (exported) {
        return;
    }

    if (StringUtils.isEmpty(path)) {
        path = interfaceName;
    }
    doExportUrls();
    exported();
}
```



```java
private void doExportUrls() {
    ServiceRepository repository = ApplicationModel.getServiceRepository();
    // 将service、prodivder注册到缓存中
    ServiceDescriptor serviceDescriptor = repository.registerService(getInterfaceClass());
    repository.registerProvider(
            getUniqueServiceName(),
            ref,
            serviceDescriptor,
            this,
            serviceMetadata
    );
	// 获取注册的url
    List<URL> registryURLs = ConfigValidationUtils.loadRegistries(this, true);

    for (ProtocolConfig protocolConfig : protocols) {
        // 暴露给每个协议
        String pathKey = URL.buildKey(getContextPath(protocolConfig)
                .map(p -> p + "/" + path)
                .orElse(path), group, version);
        // In case user specified path, register service one more time to map it to path.
        repository.registerService(pathKey, interfaceClass);
        doExportUrlsFor1Protocol(protocolConfig, registryURLs);
    }
}
```

​	上面代码首先是通过 loadRegistries 加载注册中心链接，然后再遍历 ProtocolConfig 集合导出每个服务。并在导出服务的过程中，将服务注册到注册中心。下面，我们先来看一下 loadRegistries 方法的逻辑。

```java
public static List<URL> loadRegistries(AbstractInterfaceConfig interfaceConfig, boolean provider) {
        // check && override if necessary
        List<URL> registryList = new ArrayList<URL>();
        ApplicationConfig application = interfaceConfig.getApplication();
        List<RegistryConfig> registries = interfaceConfig.getRegistries();
        if (CollectionUtils.isNotEmpty(registries)) {
            for (RegistryConfig config : registries) {
                String address = config.getAddress();
                if (StringUtils.isEmpty(address)) {
                    // 如果address为空，设置为0.0.0.0
                    address = ANYHOST_VALUE;
                }
                // 检测address是否合法
                if (!RegistryConfig.NO_AVAILABLE.equalsIgnoreCase(address)) {
                    Map<String, String> map = new HashMap<String, String>();
                    // 添加 ApplicationConfig 中的字段信息到 map 中
                    AbstractConfig.appendParameters(map, application);
                    // 添加 RegistryConfig 字段信息到 map 中
                    AbstractConfig.appendParameters(map, config);
                    map.put(PATH_KEY, RegistryService.class.getName());
                    AbstractInterfaceConfig.appendRuntimeParameters(map);
                    if (!map.containsKey(PROTOCOL_KEY)) {
                        map.put(PROTOCOL_KEY, DUBBO_PROTOCOL);
                    }
                    // 解析得到 URL 列表，address 可能包含多个注册中心 ip，
                	// 因此解析得到的是一个 URL 列表
                    List<URL> urls = UrlUtils.parseURLs(address, map);

                    for (URL url : urls) {
						// 将 URL 协议头设置为 registry
                        url = URLBuilder.from(url)
                            .addParameter(REGISTRY_KEY, url.getProtocol())
                            .setProtocol(extractRegistryType(url))
                            .build();
                        // 通过判断条件，决定是否添加 url 到 registryList 中，条件如下：
                    	// (服务提供者 && register = true 或 null) 
                    	//    || (非服务提供者 && subscribe = true 或 null)
                        if ((provider && url.getParameter(REGISTER_KEY, true))
                            || (!provider && url.getParameter(SUBSCRIBE_KEY, true))) {
                            registryList.add(url);
                        }
                    }
                }
            }
        }
    	// 生成兼容地址
        return genCompatibleRegistries(registryList, provider);
    }
```

检测是否存在注册中心配置类，不存在则生成兼容的注册中心。
构建参数映射集合，也就是 map
构建注册中心链接列表
遍历链接列表，并根据条件决定是否将其添加到 registryList中
如果没有注册中心，则调用genCompatibleRegistries方法，生成默认的 “service-discovery-registry”

```java
private static List<URL> genCompatibleRegistries(List<URL> registryList, boolean provider) {
    List<URL> result = new ArrayList<>(registryList.size());
    registryList.forEach(registryURL -> {
        if (provider) {
            // 
            // for registries enabled service discovery, automatically register interface compatible addresses.
            String registerMode;
            if (SERVICE_REGISTRY_PROTOCOL.equals(registryURL.getProtocol())) {
                registerMode = registryURL.getParameter(REGISTER_MODE_KEY, ConfigurationUtils.getCachedDynamicProperty(DUBBO_REGISTER_MODE_DEFAULT_KEY, DEFAULT_REGISTER_MODE_INSTANCE));
                if (!isValidRegisterMode(registerMode)) {
                    registerMode = DEFAULT_REGISTER_MODE_INSTANCE;
                }
                result.add(registryURL);
                if (DEFAULT_REGISTER_MODE_ALL.equalsIgnoreCase(registerMode)
                    && registryNotExists(registryURL, registryList, REGISTRY_PROTOCOL)) {
                    URL interfaceCompatibleRegistryURL = URLBuilder.from(registryURL)
                        .setProtocol(REGISTRY_PROTOCOL)
                        .removeParameter(REGISTRY_TYPE_KEY)
                        .build();
                    result.add(interfaceCompatibleRegistryURL);
                }
            } else {
                registerMode = registryURL.getParameter(REGISTER_MODE_KEY, ConfigurationUtils.getCachedDynamicProperty(DUBBO_REGISTER_MODE_DEFAULT_KEY, DEFAULT_REGISTER_MODE_ALL));
                if (!isValidRegisterMode(registerMode)) {
                    registerMode = DEFAULT_REGISTER_MODE_INTERFACE;
                }
                if ((DEFAULT_REGISTER_MODE_INSTANCE.equalsIgnoreCase(registerMode) || DEFAULT_REGISTER_MODE_ALL.equalsIgnoreCase(registerMode))
                    && registryNotExists(registryURL, registryList, SERVICE_REGISTRY_PROTOCOL)) {
                    URL serviceDiscoveryRegistryURL = URLBuilder.from(registryURL)
                        .setProtocol(SERVICE_REGISTRY_PROTOCOL)
                        .removeParameter(REGISTRY_TYPE_KEY)
                        .build();
                    result.add(serviceDiscoveryRegistryURL);
                }

                if (DEFAULT_REGISTER_MODE_INTERFACE.equalsIgnoreCase(registerMode) || DEFAULT_REGISTER_MODE_ALL.equalsIgnoreCase(registerMode)) {
                    result.add(registryURL);
                }
            }

            FrameworkStatusReporter.reportRegistrationStatus(createRegistrationReport(registerMode));
        } else {
            result.add(registryURL);
        }
    });

    return result;
}
```



```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    // 如果协议名为空，或空串，则将协议名变量设置为 dubbo
    String name = protocolConfig.getName();
    if (StringUtils.isEmpty(name)) {
        name = DUBBO;
    }

    Map<String, String> map = new HashMap<String, String>();
    // 添加 side、版本、时间戳以及进程号等信息到 map 中
    map.put(SIDE_KEY, PROVIDER_SIDE);
	
    ServiceConfig.appendRuntimeParameters(map);
    AbstractConfig.appendParameters(map, getMetrics());
    AbstractConfig.appendParameters(map, getApplication());
    AbstractConfig.appendParameters(map, getModule());
    // remove 'default.' prefix for configs from ProviderConfig
    // appendParameters(map, provider, Constants.DEFAULT_KEY);
    // 通过反射将对象的字段信息添加到 map 中
    AbstractConfig.appendParameters(map, provider);
    AbstractConfig.appendParameters(map, protocolConfig);
    AbstractConfig.appendParameters(map, this);
    MetadataReportConfig metadataReportConfig = getMetadataReportConfig();
    if (metadataReportConfig != null && metadataReportConfig.isValid()) {
        map.putIfAbsent(METADATA_KEY, REMOTE_METADATA_STORAGE_TYPE);
    }
    // methods 为 MethodConfig 集合，MethodConfig 中存储了 <dubbo:method> 标签的配置信息
    if (CollectionUtils.isNotEmpty(getMethods())) {
        //TODO Improve method config processing
        // 这段代码用于添加 Callback 配置到 map 中，代码太长，待会单独分析
    }
	 // 检测 generic 是否为 "true"，并根据检测结果向 map 中添加不同的信息
    if (ProtocolUtils.isGeneric(generic)) {
        map.put(GENERIC_KEY, generic);
        map.put(METHODS_KEY, ANY_VALUE);
    } else {
        
        String revision = Version.getVersion(interfaceClass, version);
        if (revision != null && revision.length() > 0) {
            map.put(REVISION_KEY, revision);
        }
		// 为接口生成包裹类 Wrapper，Wrapper 中包含了接口的详细信息，比如接口方法名数组，字段信息等
        String[] methods = Wrapper.getWrapper(interfaceClass).getMethodNames();
        // 添加方法名到 map 中，如果包含多个方法名，则用逗号隔开，比如 method = init,destroy
        if (methods.length == 0) {
            logger.warn("No method found in service interface " + interfaceClass.getName());
            map.put(METHODS_KEY, ANY_VALUE);
        } else {
            // 将逗号作为分隔符连接方法名，并将连接后的字符串放入 map 中
            map.put(METHODS_KEY, StringUtils.join(new HashSet<String>(Arrays.asList(methods)), ","));
        }
    }

    /**
     * Here the token value configured by the provider is used to assign the value to ServiceConfig#token
     */
    // 添加 token 到 map 中
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
    // 获取 host 和 port
    String host = findConfigedHosts(protocolConfig, registryURLs, map);
    Integer port = findConfigedPorts(protocolConfig, name, map);
    // 组装 URL
    URL url = new ServiceConfigURL(name, null, null, host, port, getContextPath(protocolConfig).map(p -> p + "/" + path).orElse(path), map);

    // You can customize Configurator to append extra parameters
    if (ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
            .hasExtension(url.getProtocol())) {
        url = ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
                .getExtension(url.getProtocol()).getConfigurator(url).configure(url);
    }

    String scope = url.getParameter(SCOPE_KEY);
    // don't export when none is configured
    if (!SCOPE_NONE.equalsIgnoreCase(scope)) {

        // export to local if the config is not remote (export to remote only when config is remote)
        if (!SCOPE_REMOTE.equalsIgnoreCase(scope)) {
            exportLocal(url);
        }
        // export to remote if the config is not local (export to local only when config is local)
        if (!SCOPE_LOCAL.equalsIgnoreCase(scope)) {
            if (CollectionUtils.isNotEmpty(registryURLs)) {
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

            MetadataUtils.publishServiceDefinition(url);
        }
    }
    this.urls.add(url);
}
```



```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
[...]
	if (CollectionUtils.isNotEmpty(getMethods())) {
            //TODO Improve method config processing
            for (MethodConfig method : getMethods()) {
                // 添加 MethodConfig 对象的字段信息到 map 中，键 = 方法名.属性名。
            // 比如存储 <dubbo:method name="sayHello" retries="2"> 对应的 MethodConfig，
            // 键 = sayHello.retries，map = {"sayHello.retries": 2, "xxx": "yyy"}
                AbstractConfig.appendParameters(map, method, method.getName());
                String retryKey = method.getName() + ".retry";
                if (map.containsKey(retryKey)) {
                    String retryValue = map.remove(retryKey);
                    if ("false".equals(retryValue)) {
                        map.put(method.getName() + ".retries", "0");
                    }
                }
                
            	// 获取 ArgumentConfig 列表
                List<ArgumentConfig> arguments = method.getArguments();
                if (CollectionUtils.isNotEmpty(arguments)) {
                    for (ArgumentConfig argument : arguments) {
                         // 检测 type 属性是否为空，或者空串（分支1 ⭐️）
                        // convert argument type
                        if (argument.getType() != null && argument.getType().length() > 0) {
                            Method[] methods = interfaceClass.getMethods();
                            // visit all methods
                            if (methods.length > 0) {
                                 
                                for (int i = 0; i < methods.length; i++) {
                                    String methodName = methods[i].getName();
                                    // target the method, and get its signature
                                    // 比对方法名，查找目标方法
                                    if (methodName.equals(method.getName())) {
                                        Class<?>[] argtypes = methods[i].getParameterTypes();
                                        // one callback in the method
                                         // 检测 ArgumentConfig 中的 type 属性与方法参数列表
                                        // 中的参数名称是否一致，不一致则抛出异常(分支2 ⭐️)
                                        if (argument.getIndex() != -1) {
                                            if (argtypes[argument.getIndex()].getName().equals(argument.getType())) {
                                                AbstractConfig.appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                                            } else {
                                                throw new IllegalArgumentException("Argument config error : the index attribute and type attribute not match :index :" + argument.getIndex() + ", type:" + argument.getType());
                                            }
                                        } else {
                                            // 分支3 ⭐️
                                            // multiple callbacks in the method
                                            for (int j = 0; j < argtypes.length; j++) {											// 从参数类型列表中查找类型名称为 argument.type 的参数
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
                             // 用户未配置 type 属性，但配置了 index 属性，且 index != -1 
                            // 添加 ArgumentConfig 字段信息到 map 中
                            AbstractConfig.appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                        } else {
                            throw new IllegalArgumentException("Argument config must set index or type attribute.eg: <dubbo:argument index='0' .../> or <dubbo:argument type=xxx .../>");
                        }

                    }
                }
            } // end of methods for
        }
[...]
}
```

请注意上面代码中⭐️符号，这几个符号标识出了4个重要的分支，下面用伪代码解释一下这几个分支的含义。

```java
// 获取 ArgumentConfig 列表
for (遍历 ArgumentConfig 列表) {
    if (type 不为 null，也不为空串) {    // 分支1
        1. 通过反射获取 interfaceClass 的方法列表
        for (遍历方法列表) {
            1. 比对方法名，查找目标方法
        	2. 通过反射获取目标方法的参数类型数组 argtypes
            if (index != -1) {    // 分支2
                1. 从 argtypes 数组中获取下标 index 处的元素 argType
                2. 检测 argType 的名称与 ArgumentConfig 中的 type 属性是否一致
                3. 添加 ArgumentConfig 字段信息到 map 中，或抛出异常
            } else {    // 分支3
                1. 遍历参数类型数组 argtypes，查找 argument.type 类型的参数
                2. 添加 ArgumentConfig 字段信息到 map 中
            }
        }
    } else if (index != -1) {    // 分支4
		1. 添加 ArgumentConfig 字段信息到 map 中
    }
}
```

### 导出 Dubbo 服务

```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    [...]
    // You can customize Configurator to append extra parameters
        if (ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
                .hasExtension(url.getProtocol())) {
             // 加载 ConfiguratorFactory，并生成 Configurator 实例，然后通过实例配置 url
            url = ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
                    .getExtension(url.getProtocol()).getConfigurator(url).configure(url);
        }

        String scope = url.getParameter(SCOPE_KEY);
        // don't export when none is configured
        if (!SCOPE_NONE.equalsIgnoreCase(scope)) {

            // export to local if the config is not remote (export to remote only when config is remote)
            // 如果 scope = none，则什么都不做
            if (!SCOPE_REMOTE.equalsIgnoreCase(scope)) {
                exportLocal(url);
            }
            // export to remote if the config is not local (export to local only when config is local)
            // scope != local，导出到远程
            if (!SCOPE_LOCAL.equalsIgnoreCase(scope)) {
                if (CollectionUtils.isNotEmpty(registryURLs)) {
                    for (URL registryURL : registryURLs) {
                        //if protocol is only injvm ,not register
                        if (LOCAL_PROTOCOL.equalsIgnoreCase(url.getProtocol())) {
                            continue;
                        }
                        url = url.addParameterIfAbsent(DYNAMIC_KEY, registryURL.getParameter(DYNAMIC_KEY));
                        // 加载监视器链接
                        URL monitorUrl = ConfigValidationUtils.loadMonitor(this, registryURL);
                        if (monitorUrl != null) {
                            // 将监视器链接作为参数添加到 url 中
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
						// 为服务提供类(ref)生成 Invoker
                        Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, registryURL.putAttribute(EXPORT_KEY, url));
                        // DelegateProviderMetaDataInvoker 用于持有 Invoker 和 ServiceConfig
                        DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
						// 导出服务，并生成 Exporter
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

                MetadataUtils.publishServiceDefinition(url);
            }
        }
        this.urls.add(url);
    [...]
}
```

上面代码根据 url 中的 scope 参数决定服务导出方式，分别如下：

- scope = none，不导出服务
- scope != remote，导出到本地
- scope != local，导出到远程

不管是导出到本地，还是远程。进行服务导出之前，均需要先创建 Invoker，这是一个很重要的步骤。因此下面先来分析 Invoker 的创建过程。

### Invoker 创建过程

​	既然 Invoker 如此重要，那么我们很有必要搞清楚 Invoker 的用途。Invoker 是由 ProxyFactory 创建而来，Dubbo 默认的 ProxyFactory 实现类是 JavassistProxyFactory。下面我们到 JavassistProxyFactory 代码中，探索 Invoker 的创建过程。如下：

```java
ublic <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
	// 为目标类创建 Wrapper
    final Wrapper wrapper = Wrapper.getWrapper(proxy.getClass().getName().indexOf('$') < 0 ? proxy.getClass() : type);
    // 创建匿名 Invoker 类对象，并实现 doInvoke 方法。
    return new AbstractProxyInvoker<T>(proxy, type, url) {
        @Override
        protected Object doInvoke(T proxy, String methodName,
                                  Class<?>[] parameterTypes,
                                  Object[] arguments) throws Throwable {
			// 调用 Wrapper 的 invokeMethod 方法，invokeMethod 最终会调用目标方法
            return wrapper.invokeMethod(proxy, methodName, parameterTypes, arguments);
        }
    };
}
```

​	如上，JavassistProxyFactory 创建了一个继承自 AbstractProxyInvoker 类的匿名对象，并覆写了抽象方法 doInvoke。覆写后的 doInvoke 逻辑比较简单，仅是将调用请求转发给了 Wrapper 类的 invokeMethod 方法。Wrapper 用于“包裹”目标类，Wrapper 是一个抽象类，仅可通过 getWrapper(Class) 方法创建子类。在创建 Wrapper 子类的过程中，子类代码生成逻辑会对 getWrapper 方法传入的 Class 对象进行解析，拿到诸如类方法，类成员变量等信息。以及生成 invokeMethod 方法代码和其他一些方法代码。代码生成完毕后，通过 Javassist 生成 Class 对象，最后再通过反射创建 Wrapper 实例。相关的代码如下：

```java
public static Wrapper getWrapper(Class<?> c) {
    while (ClassGenerator.isDynamicClass(c)) // can not wrapper on dynamic class.
    {
        c = c.getSuperclass();
    }

    if (c == Object.class) {
        return OBJECT_WRAPPER;
    }

    return WRAPPER_MAP.computeIfAbsent(c, Wrapper::makeWrapper);
}
```

getWrapper 方法仅包含一些缓存操作逻辑，不难理解。下面我们看一下 makeWrapper 方法。

```java
private static Wrapper makeWrapper(Class<?> c) {
    // 检测 c 是否为基本类型，若是则抛出异常
    if (c.isPrimitive()) {
        throw new IllegalArgumentException("Can not create wrapper for primitive type: " + c);
    }

    String name = c.getName();
    ClassLoader cl = ClassUtils.getClassLoader(c);
	// c1 用于存储 setPropertyValue 方法代码
    StringBuilder c1 = new StringBuilder("public void setPropertyValue(Object o, String n, Object v){ ");
    // c2 用于存储 getPropertyValue 方法代码
    StringBuilder c2 = new StringBuilder("public Object getPropertyValue(Object o, String n){ ");
    // c3 用于存储 invokeMethod 方法代码
    StringBuilder c3 = new StringBuilder("public Object invokeMethod(Object o, String n, Class[] p, Object[] v) throws " + InvocationTargetException.class.getName() + "{ ");
	
    // 生成类型转换代码及异常捕捉代码，比如：
    //   DemoService w; try { w = ((DemoServcie) $1); }}catch(Throwable e){ throw new IllegalArgumentException(e); }
    c1.append(name).append(" w; try{ w = ((").append(name).append(")$1); }catch(Throwable e){ throw new IllegalArgumentException(e); }");
    c2.append(name).append(" w; try{ w = ((").append(name).append(")$1); }catch(Throwable e){ throw new IllegalArgumentException(e); }");
    c3.append(name).append(" w; try{ w = ((").append(name).append(")$1); }catch(Throwable e){ throw new IllegalArgumentException(e); }");

    // pts 用于存储成员变量名和类型
    Map<String, Class<?>> pts = new HashMap<>(); // <property name, property types>
    // ms 用于存储方法描述信息（可理解为方法签名）及 Method 实例
    Map<String, Method> ms = new LinkedHashMap<>(); // <method desc, Method instance>
    // mns 为方法名列表
    List<String> mns = new ArrayList<>(); // method names.
    // dmns 用于存储“定义在当前类中的方法”的名称
    List<String> dmns = new ArrayList<>(); // declaring method names.

    // --------------------------------✨ 分割线1 ✨-------------------------------------
    // get all public field.
    // 获取 public 访问级别的字段，并为所有字段生成条件判断语句
    for (Field f : c.getFields()) {
        String fn = f.getName();
        Class<?> ft = f.getType();
        if (Modifier.isStatic(f.getModifiers()) || Modifier.isTransient(f.getModifiers())) {
            // 忽略关键字 static 或 transient 修饰的变量
            continue;
        }

         // 生成条件判断及赋值语句，比如：
        // if( $2.equals("name") ) { w.name = (java.lang.String) $3; return;}
        // if( $2.equals("age") ) { w.age = ((Number) $3).intValue(); return;}
        c1.append(" if( $2.equals(\"").append(fn).append("\") ){ w.").append(fn).append("=").append(arg(ft, "$3")).append("; return; }");
        // 生成条件判断及返回语句，比如：
        // if( $2.equals("name") ) { return ($w)w.name; }
        c2.append(" if( $2.equals(\"").append(fn).append("\") ){ return ($w)w.").append(fn).append("; }");
        // 存储 <字段名, 字段类型> 键值对到 pts 中
        pts.put(fn, ft);
    }
// --------------------------------✨ 分割线2 ✨-------------------------------------
    final ClassPool classPool = new ClassPool(ClassPool.getDefault());
    classPool.appendClassPath(new CustomizedLoaderClassPath(cl));

    List<String> allMethod = new ArrayList<>();
    try {
        final CtMethod[] ctMethods = classPool.get(c.getName()).getMethods();
        for (CtMethod method : ctMethods) {
            allMethod.add(ReflectUtils.getDesc(method));
        }
    } catch (Exception e) {
        throw new RuntimeException(e);
    }

    Method[] methods = Arrays.stream(c.getMethods())
                             .filter(method -> allMethod.contains(ReflectUtils.getDesc(method)))
                             .collect(Collectors.toList())
                             .toArray(new Method[] {});
    // get all public method.
    // 检测 c 中是否包含在当前类中声明的方法
    boolean hasMethod = hasMethods(methods);
    if (hasMethod) {
        Map<String, Integer> sameNameMethodCount = new HashMap<>((int) (methods.length / 0.75f) + 1);
        for (Method m : methods) {
            sameNameMethodCount.compute(m.getName(),
                    (key, oldValue) -> oldValue == null ? 1 : oldValue + 1);
        }

        c3.append(" try{");
        for (Method m : methods) {
            //ignore Object's method.
            if (m.getDeclaringClass() == Object.class) {
                continue;
            }
			// 生成方法名判断语句，比如：
        	// if ( "sayHello".equals( $2 )
            String mn = m.getName();
            c3.append(" if( \"").append(mn).append("\".equals( $2 ) ");
            int len = m.getParameterTypes().length;
            c3.append(" && ").append(" $3.length == ").append(len);
			// 检测方法是否存在重载情况，条件为：方法对象不同 && 方法名相同
            boolean overload = sameNameMethodCount.get(m.getName()) > 1;
            if (overload) {
                if (len > 0) {
                    // 对重载方法进行处理，考虑下面的方法：
        //    1. void sayHello(Integer, String)
        //    2. void sayHello(Integer, Integer)
        // 方法名相同，参数列表长度也相同，因此不能仅通过这两项判断两个方法是否相等。
        // 需要进一步判断方法的参数类型
                    for (int l = 0; l < len; l++) {
                        c3.append(" && ").append(" $3[").append(l).append("].getName().equals(\"")
                                .append(m.getParameterTypes()[l].getName()).append("\")");
                    }
                }
            }
			// 添加 ) {，完成方法判断语句，此时生成的代码可能如下（已格式化）：
        // if ("sayHello".equals($2) 
        //     && $3.length == 2
        //     && $3[0].getName().equals("java.lang.Integer") 
        //     && $3[1].getName().equals("java.lang.String")) {
            c3.append(" ) { ");
			// 根据返回值类型生成目标方法调用语句
            if (m.getReturnType() == Void.TYPE) {
                // w.sayHello((java.lang.Integer)$4[0], (java.lang.String)$4[1]); return null;
                c3.append(" w.").append(mn).append('(').append(args(m.getParameterTypes(), "$4")).append(");").append(" return null;");
            } else {
                // return w.sayHello((java.lang.Integer)$4[0], (java.lang.String)$4[1]);
                c3.append(" return ($w)w.").append(mn).append('(').append(args(m.getParameterTypes(), "$4")).append(");");
            }
			 // 添加 }, 生成的代码形如（已格式化）：
        // if ("sayHello".equals($2) 
        //     && $3.length == 2
        //     && $3[0].getName().equals("java.lang.Integer") 
        //     && $3[1].getName().equals("java.lang.String")) {
        //
        //     w.sayHello((java.lang.Integer)$4[0], (java.lang.String)$4[1]); 
        //     return null;
        // }
            c3.append(" }");
			// 添加方法名到 mns 集合中
            mns.add(mn);
            // 检测当前方法是否在 c 中被声明的
            if (m.getDeclaringClass() == c) {
                // 若是，则将当前方法名添加到 dmns 中
                dmns.add(mn);
            }
            ms.put(ReflectUtils.getDesc(m), m);
        }
        // 添加异常捕捉语句
        c3.append(" } catch(Throwable e) { ");
        c3.append("     throw new java.lang.reflect.InvocationTargetException(e); ");
        c3.append(" }");
    }
	// 添加 NoSuchMethodException 异常抛出代码
    c3.append(" throw new " + NoSuchMethodException.class.getName() + "(\"Not found method \\\"\"+$2+\"\\\" in class " + c.getName() + ".\"); }");

    // --------------------------------✨ 分割线4 ✨-------------------------------------
    // deal with get/set method.
    // 处理 get/set 方法
    Matcher matcher;
    for (Map.Entry<String, Method> entry : ms.entrySet()) {
        String md = entry.getKey();
        Method method = entry.getValue();
         // 匹配以 get 开头的方法
        if ((matcher = ReflectUtils.GETTER_METHOD_DESC_PATTERN.matcher(md)).matches()) {
            // 获取属性名
            String pn = propertyName(matcher.group(1));
            // 生成属性判断以及返回语句，示例如下：
            // if( $2.equals("name") ) { return ($w).w.getName(); }
            c2.append(" if( $2.equals(\"").append(pn).append("\") ){ return ($w)w.").append(method.getName()).append("(); }");
            pts.put(pn, method.getReturnType());
        } else if ((matcher = ReflectUtils.IS_HAS_CAN_METHOD_DESC_PATTERN.matcher(md)).matches()) {
            // 匹配以 is/has/can 开头的方法
            String pn = propertyName(matcher.group(1));
            // 生成属性判断以及返回语句，示例如下：
            // if( $2.equals("dream") ) { return ($w).w.hasDream(); }
            c2.append(" if( $2.equals(\"").append(pn).append("\") ){ return ($w)w.").append(method.getName()).append("(); }");
            pts.put(pn, method.getReturnType());
        } else if ((matcher = ReflectUtils.SETTER_METHOD_DESC_PATTERN.matcher(md)).matches()) {
            Class<?> pt = method.getParameterTypes()[0];
            String pn = propertyName(matcher.group(1));
            // 匹配以 set 开头的方法
            c1.append(" if( $2.equals(\"").append(pn).append("\") ){ w.").append(method.getName()).append("(").append(arg(pt, "$3")).append("); return; }");
            pts.put(pn, pt);
        }
    }
    // 添加 NoSuchPropertyException 异常抛出代码
    c1.append(" throw new " + NoSuchPropertyException.class.getName() + "(\"Not found property \\\"\"+$2+\"\\\" field or setter method in class " + c.getName() + ".\"); }");
    c2.append(" throw new " + NoSuchPropertyException.class.getName() + "(\"Not found property \\\"\"+$2+\"\\\" field or getter method in class " + c.getName() + ".\"); }");
		
    // --------------------------------✨ 分割线5 ✨-------------------------------------
    // make class
    long id = WRAPPER_CLASS_COUNTER.getAndIncrement();
    // 创建类生成器
    ClassGenerator cc = ClassGenerator.newInstance(cl);
    // 设置类名及超类
    cc.setClassName((Modifier.isPublic(c.getModifiers()) ? Wrapper.class.getName() : c.getName() + "$sw") + id);
    cc.setSuperClass(Wrapper.class);
	// 添加默认构造方法
    cc.addDefaultConstructor();
    
    // 添加字段
    cc.addField("public static String[] pns;"); // property name array.
    cc.addField("public static " + Map.class.getName() + " pts;"); // property type map.
    cc.addField("public static String[] mns;"); // all method name array.
    cc.addField("public static String[] dmns;"); // declared method name array.
    for (int i = 0, len = ms.size(); i < len; i++) {
        cc.addField("public static Class[] mts" + i + ";");
    }
	
    // 添加方法代码
    cc.addMethod("public String[] getPropertyNames(){ return pns; }");
    cc.addMethod("public boolean hasProperty(String n){ return pts.containsKey($1); }");
    cc.addMethod("public Class getPropertyType(String n){ return (Class)pts.get($1); }");
    cc.addMethod("public String[] getMethodNames(){ return mns; }");
    cc.addMethod("public String[] getDeclaredMethodNames(){ return dmns; }");
    cc.addMethod(c1.toString());
    cc.addMethod(c2.toString());
    cc.addMethod(c3.toString());

    try {
        // 生成类
        Class<?> wc = cc.toClass();
        // setup static field.
        // 设置字段值
        wc.getField("pts").set(null, pts);
        wc.getField("pns").set(null, pts.keySet().toArray(new String[0]));
        wc.getField("mns").set(null, mns.toArray(new String[0]));
        wc.getField("dmns").set(null, dmns.toArray(new String[0]));
        int ix = 0;
        for (Method m : ms.values()) {
            wc.getField("mts" + ix++).set(null, m.getParameterTypes());
        }
        // 创建 Wrapper 实例
        return (Wrapper) wc.getDeclaredConstructor().newInstance();
    } catch (RuntimeException e) {
        throw e;
    } catch (Throwable e) {
        throw new RuntimeException(e.getMessage(), e);
    } finally {
        cc.release();
        ms.clear();
        mns.clear();
        dmns.clear();
    }
}
```

### 导出服务到本地

```java
private void exportLocal(URL url) {
    URL local = URLBuilder.from(url)
            .setProtocol(LOCAL_PROTOCOL)
            .setHost(LOCALHOST_VALUE)
            .setPort(0)
            .build();
    // 创建 Invoker，并导出服务，这里的 protocol 会在运行时调用 InjvmProtocol 的 export 方法
    Exporter<?> exporter = PROTOCOL.export(
            PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, local));
    exporters.add(exporter);
    logger.info("Export dubbo service " + interfaceClass.getName() + " to local registry url : " + local);
}
```

​	exportLocal 方法比较简单，首先根据 URL 协议头决定是否导出服务。若需导出，则创建一个新的 URL 并将协议头、主机名以及端口设置成新的值。然后创建 Invoker，并调用 InjvmProtocol 的 export 方法导出服务。下面我们来看一下 InjvmProtocol 的 export 方法都做了哪些事情。

```java
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    // 创建 InjvmExporter
    return new InjvmExporter<T>(invoker, invoker.getUrl().getServiceKey(), exporterMap);
}
```

### 导出服务到远程

​	与导出服务到本地相比，导出服务到远程的过程要复杂不少，其包含了服务导出与服务注册两个过程。这两个过程涉及到了大量的调用，比较复杂。按照代码执行顺序，本节先来分析服务导出逻辑，服务注册逻辑将在下一节进行分析。下面开始分析，我们把目光移动到 RegistryProtocol 的 export 方法上。

```java
public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
    // 获取注册中心 URL，以 zookeeper 注册中心为例，得到的示例 URL 如下：
    // zookeeper://127.0.0.1:2181/com.alibaba.dubbo.registry.RegistryService?application=demo-provider&dubbo=2.0.2&export=dubbo%3A%2F%2F172.17.48.52%3A20880%2Fcom.alibaba.dubbo.demo.DemoService%3Fanyhost%3Dtrue%26application%3Ddemo-provider
    URL registryUrl = getRegistryUrl(originInvoker);
    // url to export locally
    URL providerUrl = getProviderUrl(originInvoker);

    // Subscribe the override data
    // FIXME When the provider subscribes, it will affect the scene : a certain JVM exposes the service and call
    //  the same service. Because the subscribed is cached key with the name of the service, it causes the
    //  subscription information to cover.
    // 获取订阅 URL，比如：
    // provider://172.17.48.52:20880/com.alibaba.dubbo.demo.DemoService?category=configurators&check=false&anyhost=true&application=demo-provider&dubbo=2.0.2&generic=false&interface=com.alibaba.dubbo.demo.DemoService&methods=sayHello
    final URL overrideSubscribeUrl = getSubscribedOverrideUrl(providerUrl);
    // 创建监听器
    final OverrideListener overrideSubscribeListener = new OverrideListener(overrideSubscribeUrl, originInvoker);
    overrideListeners.put(overrideSubscribeUrl, overrideSubscribeListener);

    providerUrl = overrideUrlWithConfig(providerUrl, overrideSubscribeListener);
    //export invoker
    final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);

    // url to registry
    // 根据 URL 加载 Registry 实现类，比如 ZookeeperRegistry
    final Registry registry = getRegistry(registryUrl);
    final URL registeredProviderUrl = getUrlToRegistry(providerUrl, registryUrl);

    // decide if we need to delay publish
     // 根据 register 的值决定是否注册服务
    boolean register = providerUrl.getParameter(REGISTER_KEY, true);
    if (register) {
        register(registry, registeredProviderUrl);
    }

    // register stated url on provider model
    registerStatedUrl(registryUrl, registeredProviderUrl, register);


    exporter.setRegisterUrl(registeredProviderUrl);
    exporter.setSubscribeUrl(overrideSubscribeUrl);

    // Deprecated! Subscribe to override rules in 2.6.x or before.
    // 向注册中心进行订阅 override 数据
    registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);

    notifyExport(exporter);
    //Ensure that a new exporter instance is returned every time export
    // 创建并返回 DestroyableExporter
    return new DestroyableExporter<>(exporter);
}
```

上面代码看起来比较复杂，主要做如下一些操作：

1. 调用 doLocalExport 导出服务
2. 向注册中心注册服务
3. 向注册中心进行订阅 override 数据
4. 创建并返回 DestroyableExporter

```java
private <T> ExporterChangeableWrapper<T> doLocalExport(final Invoker<T> originInvoker, URL providerUrl) {
    String key = getCacheKey(originInvoker);
	// 访问缓存
    return (ExporterChangeableWrapper<T>) bounds.computeIfAbsent(key, s -> {
        Invoker<?> invokerDelegate = new InvokerDelegate<>(originInvoker, providerUrl);
        // 调用 protocol 的 export 方法导出服务
        return new ExporterChangeableWrapper<>((Exporter<T>) protocol.export(invokerDelegate), originInvoker);
    });
}
```

​	上面的代码是典型的双重检查锁，大家在阅读 Dubbo 的源码中，会多次见到。接下来，我们把重点放在 Protocol 的 export 方法上。假设运行时协议为 dubbo，此处的 protocol 变量会在运行时加载 DubboProtocol，并调用 DubboProtocol 的 export 方法。所以，接下来我们目光转移到 DubboProtocol 的 export 方法上，相关分析如下：

```java
@Override
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    URL url = invoker.getUrl();

    // export service.
    // 获取服务标识，理解成服务坐标也行。由服务组名，服务名，服务版本号以及端口组成。比如：
    // demoGroup/com.alibaba.dubbo.demo.DemoService:1.0.1:20880
    String key = serviceKey(url);
    // 创建 DubboExporter
    DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
    // 将 <key, exporter> 键值对放入缓存中
    exporterMap.put(key, exporter);

    
    //export an stub service for dispatching event
     // 本地存根相关代码
    Boolean isStubSupportEvent = url.getParameter(STUB_EVENT_KEY, DEFAULT_STUB_EVENT);
    Boolean isCallbackservice = url.getParameter(IS_CALLBACK_SERVICE, false);
    if (isStubSupportEvent && !isCallbackservice) {
        String stubServiceMethods = url.getParameter(STUB_EVENT_METHODS_KEY);
        if (stubServiceMethods == null || stubServiceMethods.length() == 0) {
            if (logger.isWarnEnabled()) {
                logger.warn(new IllegalStateException("consumer [" + url.getParameter(INTERFACE_KEY) +
                        "], has set stubproxy support event ,but no stub methods founded."));
            }

        }
    }
	// 启动服务器
    openServer(url);
    // 优化序列化
    optimizeSerialization(url);

    return exporter;
}
```

​	如上，我们重点关注 DubboExporter 的创建以及 openServer 方法，其他逻辑看不懂也没关系，不影响理解服务导出过程。另外，DubboExporter 的代码比较简单，就不分析了。下面分析 openServer 方法。

```java
private void openServer(URL url) {
    // find server.
    // 获取 host:port，并将其作为服务器实例的 key，用于标识当前的服务器实例
    String key = url.getAddress();
    //client can export a service which's only for server to invoke
    boolean isServer = url.getParameter(IS_SERVER_KEY, true);
    if (isServer) {
        // 访问缓存
        ProtocolServer server = serverMap.get(key);
        if (server == null) {
            synchronized (this) {
                server = serverMap.get(key);
                if (server == null) {
                    // 创建服务器实例
                    serverMap.put(key, createServer(url));
                }
            }
        } else {
            // server supports reset, use together with override
            // 服务器已创建，则根据 url 中的配置重置服务器
            server.reset(url);
        }
    }
}
```

​	如上，在同一台机器上（单网卡），同一个端口上仅允许启动一个服务器实例。若某个端口上已有服务器实例，此时则调用 reset 方法重置服务器的一些配置。考虑到篇幅问题，关于服务器实例重置的代码就不分析了。接下来分析服务器实例的创建过程。如下：

```java
private ProtocolServer createServer(URL url) {
        url = URLBuilder.from(url)
                // send readonly event when server closes, it's enabled by default
                .addParameterIfAbsent(CHANNEL_READONLYEVENT_SENT_KEY, Boolean.TRUE.toString())
                // enable heartbeat by default
            	// 添加心跳检测配置到 url 中
                .addParameterIfAbsent(HEARTBEAT_KEY, String.valueOf(DEFAULT_HEARTBEAT))
            	// 添加编码解码器参数
                .addParameter(CODEC_KEY, DubboCodec.NAME)
                .build();
    
    	// 获取 server 参数，默认为 netty
        String str = url.getParameter(SERVER_KEY, DEFAULT_REMOTING_SERVER);

    	// 通过 SPI 检测是否存在 server 参数所代表的 Transporter 拓展，不存在则抛出异常
        if (str != null && str.length() > 0 && !ExtensionLoader.getExtensionLoader(Transporter.class).hasExtension(str)) {
            throw new RpcException("Unsupported server type: " + str + ", url: " + url);
        }

        ExchangeServer server;
        try {
            // 创建 ExchangeServer
            server = Exchangers.bind(url, requestHandler);
        } catch (RemotingException e) {
            throw new RpcException("Fail to start server(url: " + url + ") " + e.getMessage(), e);
        }

    	// 获取 client 参数，可指定 netty，mina
        str = url.getParameter(CLIENT_KEY);
        if (str != null && str.length() > 0) {
            // 获取所有的 Transporter 实现类名称集合，比如 supportedTypes = [netty, mina]
            Set<String> supportedTypes = ExtensionLoader.getExtensionLoader(Transporter.class).getSupportedExtensions();
            // 检测当前 Dubbo 所支持的 Transporter 实现类名称列表中，
        	// 是否包含 client 所表示的 Transporter，若不包含，则抛出异常
            if (!supportedTypes.contains(str)) {
                throw new RpcException("Unsupported client type: " + str);
            }
        }

        return new DubboProtocolServer(server);
    }
```

​	如上，createServer 包含三个核心的逻辑。第一是检测是否存在 server 参数所代表的 Transporter 拓展，不存在则抛出异常。第二是创建服务器实例。第三是检测是否支持 client 参数所表示的 Transporter 拓展，不存在也是抛出异常。两次检测操作所对应的代码比较直白了，无需多说。但创建服务器的操作目前还不是很清晰，我们继续往下看。

```java
public static ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    if (handler == null) {
        throw new IllegalArgumentException("handler == null");
    }
    url = url.addParameterIfAbsent(Constants.CODEC_KEY, "exchange");
    // 获取 Exchanger，默认为 HeaderExchanger。
    // 紧接着调用 HeaderExchanger 的 bind 方法创建 ExchangeServer 实
    return getExchanger(url).bind(url, handler);
}
```

HeaderExchanger

```java
public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
    return new HeaderExchangeServer(Transporters.bind(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
}
```

HeaderExchanger 的 bind 方法包含的逻辑比较多，但目前我们仅需关心 Transporters 的 bind 方法逻辑即可。该方法的代码如下：

```java
public static RemotingServer bind(URL url, ChannelHandler... handlers) throws RemotingException {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    if (handlers == null || handlers.length == 0) {
        throw new IllegalArgumentException("handlers == null");
    }
    ChannelHandler handler;
    if (handlers.length == 1) {
        handler = handlers[0];
    } else {
        // 如果 handlers 元素数量大于1，则创建 ChannelHandler 分发器
        handler = new ChannelHandlerDispatcher(handlers);
    }
    return getTransporter().bind(url, handler);
}
```

如上，getTransporter() 方法获取的 Transporter 是在运行时动态创建的，类名为 Transporter$Adaptive，也就是自适应拓展类。Transporter$Adaptive 会在运行时根据传入的 URL 参数决定加载什么类型的 Transporter，默认为 NettyTransporter。下面我们继续跟下去，这次分析的是 NettyTransporter 的 bind 方法。

```java
public RemotingServer bind(URL url, ChannelHandler handler) throws RemotingException {
    return new NettyServer(url, handler);
}
```

这里仅有一句创建 NettyServer 的代码，无需多说，我们继续向下看。

```java
public class NettyServer extends AbstractServer implements RemotingServer {

    public NettyServer(URL url, ChannelHandler handler) throws RemotingException {
        // 调用父类构造方法
        super(ExecutorUtil.setThreadName(url, SERVER_THREAD_POOL_NAME), ChannelHandlers.wrap(handler, url));
    }
}

public abstract class AbstractServer extends AbstractEndpoint implements RemotingServer {
    public AbstractServer(URL url, ChannelHandler handler) throws RemotingException {
        super(url, handler);
        localAddress = getUrl().toInetSocketAddress();

        // 获取 ip 和端口
        String bindIp = getUrl().getParameter(Constants.BIND_IP_KEY, getUrl().getHost());
        int bindPort = getUrl().getParameter(Constants.BIND_PORT_KEY, getUrl().getPort());
        if (url.getParameter(ANYHOST_KEY, false) || NetUtils.isInvalidLocalHost(bindIp)) {
            // 设置 ip 为 0.0.0.0
            bindIp = ANYHOST_VALUE;
        }
        bindAddress = new InetSocketAddress(bindIp, bindPort);
        // 获取最大可接受连接数
        this.accepts = url.getParameter(ACCEPTS_KEY, DEFAULT_ACCEPTS);
        try {
            // 调用模板方法 doOpen 启动服务器
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
    protected abstract void doOpen() throws Throwable;

    protected abstract void doClose() throws Throwable;
}
```

上面代码多为赋值代码，不需要多讲。我们重点关注 doOpen 抽象方法，该方法需要子类实现。下面回到 NettyServer 中

```java
protected void doOpen() throws Throwable {
    NettyHelper.setNettyLoggerFactory();
    // 创建 boss 和 worker 线程池
    ExecutorService boss = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerBoss", true));
    ExecutorService worker = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerWorker", true));
    ChannelFactory channelFactory = new NioServerSocketChannelFactory(boss, worker, getUrl().getPositiveParameter(IO_THREADS_KEY, Constants.DEFAULT_IO_THREADS));
    
    // 创建 ServerBootstrap
    bootstrap = new ServerBootstrap(channelFactory);

    final NettyHandler nettyHandler = new NettyHandler(getUrl(), this);
    channels = nettyHandler.getChannels();
    // https://issues.jboss.org/browse/NETTY-365
    // https://issues.jboss.org/browse/NETTY-379
    // final Timer timer = new HashedWheelTimer(new NamedThreadFactory("NettyIdleTimer", true));
    bootstrap.setOption("child.tcpNoDelay", true);
    bootstrap.setOption("backlog", getUrl().getPositiveParameter(BACKLOG_KEY, Constants.DEFAULT_BACKLOG));
    
    // 设置 PipelineFactory
    bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
        @Override
        public ChannelPipeline getPipeline() {
            NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyServer.this);
            ChannelPipeline pipeline = Channels.pipeline();
            /*int idleTimeout = getIdleTimeout();
            if (idleTimeout > 10000) {
                pipeline.addLast("timer", new IdleStateHandler(timer, idleTimeout / 1000, 0, 0));
            }*/
            pipeline.addLast("decoder", adapter.getDecoder());
            pipeline.addLast("encoder", adapter.getEncoder());
            pipeline.addLast("handler", nettyHandler);
            return pipeline;
        }
    });
    // bind
    // 绑定到指定的 ip 和端口上
    channel = bootstrap.bind(getBindAddress());
}
```

以上就是 NettyServer 创建的过程，dubbo 默认使用的 NettyServer 是基于 netty 3.x 版本实现的，比较老了。因此 Dubbo 另外提供了 netty 4.x 版本的 NettyServer，大家可在使用 Dubbo 的过程中按需进行配置。

到此，关于服务导出的过程就分析完了。整个过程比较复杂，大家在分析的过程中耐心一些。并且多写 Demo 进行调试，以便能够更好的理解代码逻辑。

本节内容先到这里，接下来分析服务导出的另一块逻辑 — 服务注册。

### 服务注册

```java
public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
[...]
    // url to registry
    // 根据 URL 加载 Registry 实现类，比如 ZookeeperRegistry
    final Registry registry = getRegistry(registryUrl);
    final URL registeredProviderUrl = getUrlToRegistry(providerUrl, registryUrl);

    // decide if we need to delay publish
     // 根据 register 的值决定是否注册服务
    boolean register = providerUrl.getParameter(REGISTER_KEY, true);
    if (register) {
        register(registry, registeredProviderUrl);
    }

    // register stated url on provider model
    registerStatedUrl(registryUrl, registeredProviderUrl, register);


    exporter.setRegisterUrl(registeredProviderUrl);
    exporter.setSubscribeUrl(overrideSubscribeUrl);

    // Deprecated! Subscribe to override rules in 2.6.x or before.
    // 向注册中心进行订阅 override 数据
    registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);

    notifyExport(exporter);
    //Ensure that a new exporter instance is returned every time export
    // 创建并返回 DestroyableExporter
    return new DestroyableExporter<>(exporter);
}
```

​	RegistryProtocol 的 export 方法包含了服务导出，注册，以及数据订阅等逻辑。其中服务导出逻辑上一节已经分析过了，本节将分析服务注册逻辑，相关代码如下：

```java
private void register(Registry registry, URL registeredProviderUrl) {
    registry.register(registeredProviderUrl);
}
```

#### 创建注册中心

```java
protected Registry getRegistry(final URL registryUrl) {
    return registryFactory.getRegistry(registryUrl);
}
```

本节内容以 Zookeeper 注册中心为例进行分析。下面先来看一下 getRegistry 方法的源码，这个方法由 AbstractRegistryFactory 实现。如下：

// todo疑问 

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
    LOCK.lock();
    try {
        // double check
        // fix https://github.com/apache/dubbo/issues/7265.
        defaultNopRegistry = getDefaultNopRegistryIfDestroyed();
        if (null != defaultNopRegistry) {
            return defaultNopRegistry;
        }
	   // 访问缓存
        Registry registry = REGISTRIES.get(key);
        if (registry != null) {
            return registry;
        }
        // 缓存未命中，创建 Registry 实例
        //create registry by spi/ioc
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
```

​	如上，getRegistry 方法先访问缓存，缓存未命中则调用 createRegistry 创建 Registry，然后写入缓存。这里的 createRegistry 是一个模板方法，由具体的子类实现。因此，下面我们到 ZookeeperRegistryFactory 中探究一番。

```java
public class ZookeeperRegistryFactory extends AbstractRegistryFactory {

    // zookeeperTransporter 由 SPI 在运行时注入，类型为 ZookeeperTransporter$Adaptive
    private ZookeeperTransporter zookeeperTransporter;

    /**
     * Invisible injection of zookeeper client via IOC/SPI
     * @param zookeeperTransporter
     */
    public void setZookeeperTransporter(ZookeeperTransporter zookeeperTransporter) {
        this.zookeeperTransporter = zookeeperTransporter;
    }

    @Override
    public Registry createRegistry(URL url) {
        // 创建 ZookeeperRegistry
        return new ZookeeperRegistry(url, zookeeperTransporter);
    }

}
```

```java
public ZookeeperRegistry(URL url, ZookeeperTransporter zookeeperTransporter) {
    super(url);
    if (url.isAnyHost()) {
        throw new IllegalStateException("registry address == null");
    }
    // 获取组名，默认为 dubbo
    String group = url.getGroup(DEFAULT_ROOT);
    if (!group.startsWith(PATH_SEPARATOR)) {
        group = PATH_SEPARATOR + group;
    }
    this.root = group;
    // 创建 Zookeeper 客户端，默认为 CuratorZookeeperTransporter
    zkClient = zookeeperTransporter.connect(url);
    
    // 添加状态监听器
    zkClient.addStateListener((state) -> {
        if (state == StateListener.RECONNECTED) {
            logger.warn("Trying to fetch the latest urls, in case there're provider changes during connection loss.\n" +
                    " Since ephemeral ZNode will not get deleted for a connection lose, " +
                    "there's no need to re-register url of this instance.");
            ZookeeperRegistry.this.fetchLatestAddresses();
        } else if (state == StateListener.NEW_SESSION_CREATED) {
            logger.warn("Trying to re-register urls and re-subscribe listeners of this instance to registry...");
            try {
                ZookeeperRegistry.this.recover();
            } catch (Exception e) {
                logger.error(e.getMessage(), e);
            }
        } else if (state == StateListener.SESSION_LOST) {
            logger.warn("Url of this instance will be deleted from registry soon. " +
                    "Dubbo client will try to re-register once a new session is created.");
        } else if (state == StateListener.SUSPENDED) {

        } else if (state == StateListener.CONNECTED) {

        }
    });
}
```

​	在上面的代码代码中，我们重点关注 ZookeeperTransporter 的 connect 方法调用，这个方法用于创建 Zookeeper 客户端。创建好 Zookeeper 客户端，意味着注册中心的创建过程就结束了。接下来，再来分析一下 Zookeeper 客户端的创建过程。

​	前面说过，这里的 zookeeperTransporter 类型为自适应拓展类，因此 connect 方法会在被调用时决定加载什么类型的 ZookeeperTransporter 拓展，默认为 CuratorZookeeperTransporter。下面我们到 CuratorZookeeperTransporter 中看一看。

```java
public ZookeeperClient createZookeeperClient(URL url) {
    return new CuratorZookeeperClient(url);
}
```

```java
public CuratorZookeeperClient(URL url) {
    super(url);
    try {
        int timeout = url.getParameter(TIMEOUT_KEY, DEFAULT_CONNECTION_TIMEOUT_MS);
        int sessionExpireMs = url.getParameter(ZK_SESSION_EXPIRE_KEY, DEFAULT_SESSION_TIMEOUT_MS);
        // 创建 CuratorFramework 构造器
        CuratorFrameworkFactory.Builder builder = CuratorFrameworkFactory.builder()
                .connectString(url.getBackupAddress())
                .retryPolicy(new RetryNTimes(1, 1000))
                .connectionTimeoutMs(timeout)
                .sessionTimeoutMs(sessionExpireMs);
        String authority = url.getAuthority();
        if (authority != null && authority.length() > 0) {
            builder = builder.authorization("digest", authority.getBytes());
        }
        // 构建 CuratorFramework 实例
        client = builder.build();
        // 添加监听器
        client.getConnectionStateListenable().addListener(new CuratorConnectionStateListener(url));
        // 启动客户端
        client.start();
        boolean connected = client.blockUntilConnected(timeout, TimeUnit.MILLISECONDS);
        if (!connected) {
            throw new IllegalStateException("zookeeper not connected");
        }
    } catch (Exception e) {
        throw new IllegalStateException(e.getMessage(), e);
    }
}
```

CuratorZookeeperClient 构造方法主要用于创建和启动 CuratorFramework 实例。以上基本上都是 Curator 框架的代码，大家如果对 Curator 框架不是很了解，可以参考 Curator 官方文档。

本节分析了 ZookeeperRegistry 实例的创建过程，整个过程并不是很复杂。大家在看完分析后，可以自行调试，以加深理解。现在注册中心实例创建好了，接下来要做的事情是向注册中心注册服务，我们继续往下看。

#### 节点创建

​	以 Zookeeper 为例，所谓的服务注册，本质上是将服务配置数据写入到 Zookeeper 的某个路径的节点下。为了让大家有一个直观的了解，下面我们将 Dubbo 的 demo 跑起来，然后通过 Zookeeper 可视化客户端 [ZooInspector](https://github.com/apache/zookeeper/tree/b79af153d0f98a4f3f3516910ed47234d7b3d74e/src/contrib/zooinspector) 查看节点数据。如下：

![](https://pic.imgdb.cn/item/612204134907e2d39ca1ee45.jpg)

​	从上图中可以看到 com.alibaba.dubbo.demo.DemoService 这个服务对应的配置信息（存储在 URL 中）最终被注册到了 /dubbo/com.alibaba.dubbo.demo.DemoService/providers/ 节点下。搞懂了服务注册的本质，那么接下来我们就可以去阅读服务注册的代码了。服务注册的接口为 register(URL)，这个方法定义在 FailbackRegistry 抽象类中。代码如下：

```java
public void register(URL url) {
    if (!acceptable(url)) {
        logger.info("URL " + url + " will not be registered to Registry. Registry " + url + " does not accept service of this protocol type.");
        return;
    }
    super.register(url);
    removeFailedRegistered(url);
    removeFailedUnregistered(url);
    try {
        // Sending a registration request to the server side
        // 模板方法，子类实现
        doRegister(url);
    } catch (Exception e) {
        Throwable t = e;
		// 获取check参数， 如果check为true 直接抛出异常
        // If the startup detection is opened, the Exception is thrown directly.
        boolean check = getUrl().getParameter(Constants.CHECK_KEY, true)
                && url.getParameter(Constants.CHECK_KEY, true)
                && !(url.getPort() == 0);
        boolean skipFailback = t instanceof SkipFailbackWrapperException;
        if (check || skipFailback) {
            if (skipFailback) {
                t = t.getCause();
            }
            throw new IllegalStateException("Failed to register " + url + " to registry " + getUrl().getAddress() + ", cause: " + t.getMessage(), t);
        } else {
            logger.error("Failed to register " + url + ", waiting for retry, cause: " + t.getMessage(), t);
        }

        // Record a failed registration request to a failed list, retry regularly
        // 记录注册失败的链接
        addFailedRegistered(url);
    }
}
```

如上，我们重点关注 doRegister 方法调用即可，其他的代码先忽略。doRegister 方法是一个模板方法，因此我们到 FailbackRegistry 子类 ZookeeperRegistry 中进行分析。如下：

```java
public void doRegister(URL url) {
    try {
        zkClient.create(toUrlPath(url), url.getParameter(DYNAMIC_KEY, true));
    } catch (Throwable e) {
        throw new RpcException("Failed to register " + url + " to zookeeper " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```

​	如上，ZookeeperRegistry 在 doRegister 中调用了 Zookeeper 客户端创建服务节点。节点路径由 toUrlPath 方法生成，该方法逻辑不难理解，就不分析了。接下来分析 create 方法，如下：

```java
public void create(String path, boolean ephemeral) {
    if (!ephemeral) {
        if (persistentExistNodePath.contains(path)) {
            return;
        }
        // 如果要创建的节点类型非临时节点，那么这里要检测节点是否存在
        if (checkExists(path)) {
            persistentExistNodePath.add(path);
            return;
        }
    }
    int i = path.lastIndexOf('/');
    if (i > 0) {
        // 递归创建上一级路径
        create(path.substring(0, i), false);
    }
    // 根据 ephemeral 的值创建临时或持久节点
    if (ephemeral) {
        createEphemeral(path);
    } else {
        createPersistent(path);
        persistentExistNodePath.add(path);
    }
}
```

上面方法先是通过递归创建当前节点的上一级路径，然后再根据 ephemeral 的值决定创建临时还是持久节点。createEphemeral 和 createPersistent 这两个方法都比较简单，这里简单分析其中的一个。如下：

```java
public void createEphemeral(String path) {
    try {
        client.create().withMode(CreateMode.EPHEMERAL).forPath(path);
    } catch (NodeExistsException e) {
        logger.warn("ZNode " + path + " already exists, since we will only try to recreate a node on a session expiration" +
                ", this duplication might be caused by a delete delay from the zk server, which means the old expired session" +
                " may still holds this ZNode and the server just hasn't got time to do the deletion. In this case, " +
                "we can just try to delete and create again.", e);
        deletePath(path);
        createEphemeral(path);
    } catch (Exception e) {
        throw new IllegalStateException(e.getMessage(), e);
    }
}
```

### 订阅 override 数据 todo





# 服务引用



## 简介

​	上一篇文章详细分析了服务导出的过程，本篇文章我们趁热打铁，继续分析服务引用过程。在 Dubbo 中，我们可以通过两种方式引用远程服务。第一种是使用服务直连的方式引用服务，第二种方式是基于注册中心进行引用。服务直连的方式仅适合在调试或测试服务的场景下使用，不适合在线上环境使用。因此，本文我将重点分析通过注册中心引用服务的过程。从注册中心中获取服务配置只是服务引用过程中的一环，除此之外，服务消费者还需要经历 Invoker 创建、代理类创建等步骤。这些步骤，将在后续章节中一一进行分析。

## 服务引用原理

​	Dubbo 服务引用的时机有两个，第一个是在 Spring 容器调用 ReferenceBean 的 afterPropertiesSet 方法时引用服务，第二个是在 ReferenceBean 对应的服务被注入到其他类中时引用。这两个引用服务的时机区别在于，第一个是饿汉式的，第二个是懒汉式的。默认情况下，Dubbo 使用懒汉式引用服务。如果需要使用饿汉式，可通过配置 `<dubbo:reference>` 的 init 属性开启。下面我们按照 Dubbo 默认配置进行分析，整个分析过程从 ReferenceBean 的 getObject 方法开始。当我们的服务被注入到其他类中时，Spring 会第一时间调用 getObject 方法，并由该方法执行服务引用逻辑。按照惯例，在进行具体工作之前，需先进行配置检查与收集工作。接着根据收集到的信息决定服务用的方式，有三种，第一种是引用本地 (JVM) 服务，第二是通过直连方式引用远程服务，第三是通过注册中心引用远程服务。不管是哪种引用方式，最后都会得到一个 Invoker 实例。如果有多个注册中心，多个服务提供者，这个时候会得到一组 Invoker 实例，此时需要通过集群管理类 Cluster 将多个 Invoker 合并成一个实例。合并后的 Invoker 实例已经具备调用本地或远程服务的能力了，但并不能将此实例暴露给用户使用，这会对用户业务代码造成侵入。此时框架还需要通过代理工厂类 (ProxyFactory) 为服务接口生成代理类，并让代理类去调用 Invoker 逻辑。避免了 Dubbo 框架代码对业务代码的侵入，同时也让框架更容易使用。

​	以上就是服务引用的大致原理，下面我们深入到代码中，详细分析服务引用细节。

## 源码分析 2.7

​	服务引用的入口方法为 ReferenceBean 的 getObject 方法，该方法定义在 Spring 的 FactoryBean 接口中，ReferenceBean 实现了这个方法。实现代码如下：

```java
public Object getObject() throws Exception {
    return get();
}

public synchronized T get() {
    if (destroyed) {
        throw new IllegalStateException("Already destroyed!");
    }
    // 检测 ref 是否为空，为空则通过 init 方法创建
    if (ref == null) {
        // init 方法主要用于处理配置，以及调用 createProxy 生成代理类
        init();
    }
    return ref;
}
```

​	以上两个方法的代码比较简短，并不难理解。这里需要特别说明一下，如果你对 2.6.4 及以下版本的 getObject 方法进行调试时，会碰到比较奇怪的的问题。这里假设你使用 IDEA，且保持了 IDEA 的默认配置。当你面调试到 get 方法的`if (ref == null)`时，你会发现 ref 不为空，导致你无法进入到 init 方法中继续调试。导致这个现象的原因是 Dubbo 框架本身有一些小问题。该问题已经在 pull request [#2754](https://github.com/apache/dubbo/pull/2754) 修复了此问题，并跟随 2.6.5 版本发布了。如果你正在学习 2.6.4 及以下版本，可通过修改 IDEA 配置规避这个问题。首先 IDEA 配置弹窗中搜索 toString，然后取消`Enable 'toString' object view`勾选。具体如下：

![](https://pic.imgdb.cn/item/6129dbb844eaada7396de5d4.jpg)

### 处理配置

​	Dubbo 提供了丰富的配置，用于调整和优化框架行为，性能等。Dubbo 在引用或导出服务时，首先会对这些配置进行检查和处理，以保证配置的正确性。配置解析逻辑封装在 ReferenceConfig 的 init 方法中，下面进行分析。

```java
private void init() {
    // 避免重复初始化
    if (initialized) {
        return;
    }
    initialized = true;
    // 检测接口名合法性
    if (interfaceName == null || interfaceName.length() == 0) {
        throw new IllegalStateException("interface not allow null!");
    }

    // 检测 consumer 变量是否为空，为空则创建
    checkDefault();
    appendProperties(this);
    if (getGeneric() == null && getConsumer() != null) {
        // 设置 generic
        setGeneric(getConsumer().getGeneric());
    }

    // 检测是否为泛化接口
    if (ProtocolUtils.isGeneric(getGeneric())) {
        interfaceClass = GenericService.class;
    } else {
        try {
            // 加载类
            interfaceClass = Class.forName(interfaceName, true, Thread.currentThread()
                    .getContextClassLoader());
        } catch (ClassNotFoundException e) {
            throw new IllegalStateException(e.getMessage(), e);
        }
        checkInterfaceAndMethods(interfaceClass, methods);
    }
    
    // -------------------------------✨ 分割线1 ✨------------------------------

    // 从系统变量中获取与接口名对应的属性值
    String resolve = System.getProperty(interfaceName);
    String resolveFile = null;
    if (resolve == null || resolve.length() == 0) {
        // 从系统属性中获取解析文件路径
        resolveFile = System.getProperty("dubbo.resolve.file");
        if (resolveFile == null || resolveFile.length() == 0) {
            // 从指定位置加载配置文件
            File userResolveFile = new File(new File(System.getProperty("user.home")), "dubbo-resolve.properties");
            if (userResolveFile.exists()) {
                // 获取文件绝对路径
                resolveFile = userResolveFile.getAbsolutePath();
            }
        }
        if (resolveFile != null && resolveFile.length() > 0) {
            Properties properties = new Properties();
            FileInputStream fis = null;
            try {
                fis = new FileInputStream(new File(resolveFile));
                // 从文件中加载配置
                properties.load(fis);
            } catch (IOException e) {
                throw new IllegalStateException("Unload ..., cause:...");
            } finally {
                try {
                    if (null != fis) fis.close();
                } catch (IOException e) {
                    logger.warn(e.getMessage(), e);
                }
            }
            // 获取与接口名对应的配置
            resolve = properties.getProperty(interfaceName);
        }
    }
    if (resolve != null && resolve.length() > 0) {
        // 将 resolve 赋值给 url
        url = resolve;
    }
    
    // -------------------------------✨ 分割线2 ✨------------------------------
    if (consumer != null) {
        if (application == null) {
            // 从 consumer 中获取 Application 实例，下同
            application = consumer.getApplication();
        }
        if (module == null) {
            module = consumer.getModule();
        }
        if (registries == null) {
            registries = consumer.getRegistries();
        }
        if (monitor == null) {
            monitor = consumer.getMonitor();
        }
    }
    if (module != null) {
        if (registries == null) {
            registries = module.getRegistries();
        }
        if (monitor == null) {
            monitor = module.getMonitor();
        }
    }
    if (application != null) {
        if (registries == null) {
            registries = application.getRegistries();
        }
        if (monitor == null) {
            monitor = application.getMonitor();
        }
    }
    
    // 检测 Application 合法性
    checkApplication();
    // 检测本地存根配置合法性
    checkStubAndMock(interfaceClass);
    
	// -------------------------------✨ 分割线3 ✨------------------------------
    
    Map<String, String> map = new HashMap<String, String>();
    Map<Object, Object> attributes = new HashMap<Object, Object>();

    // 添加 side、协议版本信息、时间戳和进程号等信息到 map 中
    map.put(Constants.SIDE_KEY, Constants.CONSUMER_SIDE);
    map.put(Constants.DUBBO_VERSION_KEY, Version.getProtocolVersion());
    map.put(Constants.TIMESTAMP_KEY, String.valueOf(System.currentTimeMillis()));
    if (ConfigUtils.getPid() > 0) {
        map.put(Constants.PID_KEY, String.valueOf(ConfigUtils.getPid()));
    }

    // 非泛化服务
    if (!isGeneric()) {
        // 获取版本
        String revision = Version.getVersion(interfaceClass, version);
        if (revision != null && revision.length() > 0) {
            map.put("revision", revision);
        }

        // 获取接口方法列表，并添加到 map 中
        String[] methods = Wrapper.getWrapper(interfaceClass).getMethodNames();
        if (methods.length == 0) {
            map.put("methods", Constants.ANY_VALUE);
        } else {
            map.put("methods", StringUtils.join(new HashSet<String>(Arrays.asList(methods)), ","));
        }
    }
    map.put(Constants.INTERFACE_KEY, interfaceName);
    // 将 ApplicationConfig、ConsumerConfig、ReferenceConfig 等对象的字段信息添加到 map 中
    appendParameters(map, application);
    appendParameters(map, module);
    appendParameters(map, consumer, Constants.DEFAULT_KEY);
    appendParameters(map, this);
    
	// -------------------------------✨ 分割线4 ✨------------------------------
    
    String prefix = StringUtils.getServiceKey(map);
    if (methods != null && !methods.isEmpty()) {
        // 遍历 MethodConfig 列表
        for (MethodConfig method : methods) {
            appendParameters(map, method, method.getName());
            String retryKey = method.getName() + ".retry";
            // 检测 map 是否包含 methodName.retry
            if (map.containsKey(retryKey)) {
                String retryValue = map.remove(retryKey);
                if ("false".equals(retryValue)) {
                    // 添加重试次数配置 methodName.retries
                    map.put(method.getName() + ".retries", "0");
                }
            }
 
            // 添加 MethodConfig 中的“属性”字段到 attributes
            // 比如 onreturn、onthrow、oninvoke 等
            appendAttributes(attributes, method, prefix + "." + method.getName());
            checkAndConvertImplicitConfig(method, map, attributes);
        }
    }
    
	// -------------------------------✨ 分割线5 ✨------------------------------

    // 获取服务消费者 ip 地址
    String hostToRegistry = ConfigUtils.getSystemProperty(Constants.DUBBO_IP_TO_REGISTRY);
    if (hostToRegistry == null || hostToRegistry.length() == 0) {
        hostToRegistry = NetUtils.getLocalHost();
    } else if (isInvalidLocalHost(hostToRegistry)) {
        throw new IllegalArgumentException("Specified invalid registry ip from property..." );
    }
    map.put(Constants.REGISTER_IP_KEY, hostToRegistry);

    // 存储 attributes 到系统上下文中
    StaticContext.getSystemContext().putAll(attributes);

    // 创建代理类
    ref = createProxy(map);

    // 根据服务名，ReferenceConfig，代理类构建 ConsumerModel，
    // 并将 ConsumerModel 存入到 ApplicationModel 中
    ConsumerModel consumerModel = new ConsumerModel(getUniqueServiceName(), this, ref, interfaceClass.getMethods());
    ApplicationModel.initConsumerModel(getUniqueServiceName(), consumerModel);
}
```

上面的代码很长，做的事情比较多。这里根据代码逻辑，对代码进行了分块，下面我们一起来看一下。

首先是方法开始到分割线1之间的代码。这段代码主要用于检测 ConsumerConfig 实例是否存在，如不存在则创建一个新的实例，然后通过系统变量或 dubbo.properties 配置文件填充 ConsumerConfig 的字段。接着是检测泛化配置，并根据配置设置 interfaceClass 的值。接着来看分割线1到分割线2之间的逻辑。这段逻辑用于从系统属性或配置文件中加载与接口名相对应的配置，并将解析结果赋值给 url 字段。url 字段的作用一般是用于点对点调用。继续向下看，分割线2和分割线3之间的代码用于检测几个核心配置类是否为空，为空则尝试从其他配置类中获取。分割线3与分割线4之间的代码主要用于收集各种配置，并将配置存储到 map 中。分割线4和分割线5之间的代码用于处理 MethodConfig 实例。该实例包含了事件通知配置，比如 onreturn、onthrow、oninvoke 等。分割线5到方法结尾的代码主要用于解析服务消费者 ip，以及调用 createProxy 创建代理对象。关于该方法的详细分析，将会在接下来的章节中展开。

### 引用服务

​	本节我们要从 createProxy 开始看起。从字面意思上来看，createProxy 似乎只是用于创建代理对象的。但实际上并非如此，该方法还会调用其他方法构建以及合并 Invoker 实例。具体细节如下。

```java
private T createProxy(Map<String, String> map) {
    URL tmpUrl = new URL("temp", "localhost", 0, map);
    final boolean isJvmRefer;
    if (isInjvm() == null) {
        // url 配置被指定，则不做本地引用
        if (url != null && url.length() > 0) {
            isJvmRefer = false;
        // 根据 url 的协议、scope 以及 injvm 等参数检测是否需要本地引用
        // 比如如果用户显式配置了 scope=local，此时 isInjvmRefer 返回 true
        } else if (InjvmProtocol.getInjvmProtocol().isInjvmRefer(tmpUrl)) {
            isJvmRefer = true;
        } else {
            isJvmRefer = false;
        }
    } else {
        // 获取 injvm 配置值
        isJvmRefer = isInjvm().booleanValue();
    }

    // 本地引用
    if (isJvmRefer) {
        // 生成本地引用 URL，协议为 injvm
        URL url = new URL(Constants.LOCAL_PROTOCOL, NetUtils.LOCALHOST, 0, interfaceClass.getName()).addParameters(map);
        // 调用 refer 方法构建 InjvmInvoker 实例
        invoker = refprotocol.refer(interfaceClass, url);
        
    // 远程引用
    } else {
        // url 不为空，表明用户可能想进行点对点调用
        if (url != null && url.length() > 0) {
            // 当需要配置多个 url 时，可用分号进行分割，这里会进行切分
            String[] us = Constants.SEMICOLON_SPLIT_PATTERN.split(url);
            if (us != null && us.length > 0) {
                for (String u : us) {
                    URL url = URL.valueOf(u);
                    if (url.getPath() == null || url.getPath().length() == 0) {
                        // 设置接口全限定名为 url 路径
                        url = url.setPath(interfaceName);
                    }
                    
                    // 检测 url 协议是否为 registry，若是，表明用户想使用指定的注册中心
                    if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
                        // 将 map 转换为查询字符串，并作为 refer 参数的值添加到 url 中
                        urls.add(url.addParameterAndEncoded(Constants.REFER_KEY, StringUtils.toQueryString(map)));
                    } else {
                        // 合并 url，移除服务提供者的一些配置（这些配置来源于用户配置的 url 属性），
                        // 比如线程池相关配置。并保留服务提供者的部分配置，比如版本，group，时间戳等
                        // 最后将合并后的配置设置为 url 查询字符串中。
                        urls.add(ClusterUtils.mergeUrl(url, map));
                    }
                }
            }
        } else {
            // 加载注册中心 url
            List<URL> us = loadRegistries(false);
            if (us != null && !us.isEmpty()) {
                for (URL u : us) {
                    URL monitorUrl = loadMonitor(u);
                    if (monitorUrl != null) {
                        map.put(Constants.MONITOR_KEY, URL.encode(monitorUrl.toFullString()));
                    }
                    // 添加 refer 参数到 url 中，并将 url 添加到 urls 中
                    urls.add(u.addParameterAndEncoded(Constants.REFER_KEY, StringUtils.toQueryString(map)));
                }
            }

            // 未配置注册中心，抛出异常
            if (urls.isEmpty()) {
                throw new IllegalStateException("No such any registry to reference...");
            }
        }

        // 单个注册中心或服务提供者(服务直连，下同)
        if (urls.size() == 1) {
            // 调用 RegistryProtocol 的 refer 构建 Invoker 实例
            invoker = refprotocol.refer(interfaceClass, urls.get(0));
            
        // 多个注册中心或多个服务提供者，或者两者混合
        } else {
            List<Invoker<?>> invokers = new ArrayList<Invoker<?>>();
            URL registryURL = null;

            // 获取所有的 Invoker
            for (URL url : urls) {
                // 通过 refprotocol 调用 refer 构建 Invoker，refprotocol 会在运行时
                // 根据 url 协议头加载指定的 Protocol 实例，并调用实例的 refer 方法
                invokers.add(refprotocol.refer(interfaceClass, url));
                if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
                    registryURL = url;
                }
            }
            if (registryURL != null) {
                // 如果注册中心链接不为空，则将使用 AvailableCluster
                URL u = registryURL.addParameter(Constants.CLUSTER_KEY, AvailableCluster.NAME);
                // 创建 StaticDirectory 实例，并由 Cluster 对多个 Invoker 进行合并
                invoker = cluster.join(new StaticDirectory(u, invokers));
            } else {
                invoker = cluster.join(new StaticDirectory(invokers));
            }
        }
    }

    Boolean c = check;
    if (c == null && consumer != null) {
        c = consumer.isCheck();
    }
    if (c == null) {
        c = true;
    }
    
    // invoker 可用性检查
    if (c && !invoker.isAvailable()) {
        throw new IllegalStateException("No provider available for the service...");
    }

    // 生成代理类
    return (T) proxyFactory.getProxy(invoker);
}
```

​	上面代码很多，不过逻辑比较清晰。首先根据配置检查是否为本地调用，若是，则调用 InjvmProtocol 的 refer 方法生成 InjvmInvoker 实例。若不是，则读取直连配置项，或注册中心 url，并将读取到的 url 存储到 urls 中。然后根据 urls 元素数量进行后续操作。若 urls 元素数量为1，则直接通过 Protocol 自适应拓展类构建 Invoker 实例接口。若 urls 元素数量大于1，即存在多个注册中心或服务直连 url，此时先根据 url 构建 Invoker。然后再通过 Cluster 合并多个 Invoker，最后调用 ProxyFactory 生成代理类。Invoker 的构建过程以及代理类的过程比较重要，因此接下来将分两小节对这两个过程进行分析

#### 创建 Invoker

​	Invoker 是 Dubbo 的核心模型，代表一个可执行体。在服务提供方，Invoker 用于调用服务提供类。在服务消费方，Invoker 用于执行远程调用。Invoker 是由 Protocol 实现类构建而来。Protocol 实现类有很多，本节会分析最常用的两个，分别是 RegistryProtocol 和 DubboProtocol，其他的大家自行分析。下面先来分析 DubboProtocol 的 refer 方法源码。如下：

```java
public <T> Invoker<T> refer(Class<T> serviceType, URL url) throws RpcException {
    optimizeSerialization(url);
    // 创建 DubboInvoker
    DubboInvoker<T> invoker = new DubboInvoker<T>(serviceType, url, getClients(url), invokers);
    invokers.add(invoker);
    return invoker;
}
```

​	上面方法看起来比较简单，不过这里有一个调用需要我们注意一下，即 getClients。这个方法用于获取客户端实例，实例类型为 ExchangeClient。ExchangeClient 实际上并不具备通信能力，它需要基于更底层的客户端实例进行通信。比如 NettyClient、MinaClient 等，默认情况下，Dubbo 使用 NettyClient 进行通信。接下来，我们简单看一下 getClients 方法的逻辑。

```java
private ExchangeClient[] getClients(URL url) {
    // 是否共享连接
    boolean service_share_connect = false;
  	// 获取连接数，默认为0，表示未配置
    int connections = url.getParameter(Constants.CONNECTIONS_KEY, 0);
    // 如果未配置 connections，则共享连接
    if (connections == 0) {
        service_share_connect = true;
        connections = 1;
    }

    ExchangeClient[] clients = new ExchangeClient[connections];
    for (int i = 0; i < clients.length; i++) {
        if (service_share_connect) {
            // 获取共享客户端
            clients[i] = getSharedClient(url);
        } else {
            // 初始化新的客户端
            clients[i] = initClient(url);
        }
    }
    return clients;
}
```

​	这里根据 connections 数量决定是获取共享客户端还是创建新的客户端实例，默认情况下，使用共享客户端实例。getSharedClient 方法中也会调用 initClient 方法，因此下面我们一起看一下这两个方法。

```java
private ExchangeClient getSharedClient(URL url) {
    String key = url.getAddress();
    // 获取带有“引用计数”功能的 ExchangeClient
    ReferenceCountExchangeClient client = referenceClientMap.get(key);
    if (client != null) {
        if (!client.isClosed()) {
            // 增加引用计数
            client.incrementAndGetCount();
            return client;
        } else {
            referenceClientMap.remove(key);
        }
    }

    locks.putIfAbsent(key, new Object());
    synchronized (locks.get(key)) {
        if (referenceClientMap.containsKey(key)) {
            return referenceClientMap.get(key);
        }

        // 创建 ExchangeClient 客户端
        ExchangeClient exchangeClient = initClient(url);
        // 将 ExchangeClient 实例传给 ReferenceCountExchangeClient，这里使用了装饰模式
        client = new ReferenceCountExchangeClient(exchangeClient, ghostClientMap);
        referenceClientMap.put(key, client);
        ghostClientMap.remove(key);
        locks.remove(key);
        return client;
    }
}
```

​	上面方法先访问缓存，若缓存未命中，则通过 initClient 方法创建新的 ExchangeClient 实例，并将该实例传给 ReferenceCountExchangeClient 构造方法创建一个带有引用计数功能的 ExchangeClient 实例。ReferenceCountExchangeClient 内部实现比较简单，就不分析了。下面我们再来看一下 initClient 方法的代码。

```java
private ExchangeClient initClient(URL url) {

    // 获取客户端类型，默认为 netty
    String str = url.getParameter(Constants.CLIENT_KEY, url.getParameter(Constants.SERVER_KEY, Constants.DEFAULT_REMOTING_CLIENT));

    // 添加编解码和心跳包参数到 url 中
    url = url.addParameter(Constants.CODEC_KEY, DubboCodec.NAME);
    url = url.addParameterIfAbsent(Constants.HEARTBEAT_KEY, String.valueOf(Constants.DEFAULT_HEARTBEAT));

    // 检测客户端类型是否存在，不存在则抛出异常
    if (str != null && str.length() > 0 && !ExtensionLoader.getExtensionLoader(Transporter.class).hasExtension(str)) {
        throw new RpcException("Unsupported client type: ...");
    }

    ExchangeClient client;
    try {
        // 获取 lazy 配置，并根据配置值决定创建的客户端类型
        if (url.getParameter(Constants.LAZY_CONNECT_KEY, false)) {
            // 创建懒加载 ExchangeClient 实例
            client = new LazyConnectExchangeClient(url, requestHandler);
        } else {
            // 创建普通 ExchangeClient 实例
            client = Exchangers.connect(url, requestHandler);
        }
    } catch (RemotingException e) {
        throw new RpcException("Fail to create remoting client for service...");
    }
    return client;
}
```

​	initClient 方法首先获取用户配置的客户端类型，默认为 netty。然后检测用户配置的客户端类型是否存在，不存在则抛出异常。最后根据 lazy 配置决定创建什么类型的客户端。这里的 LazyConnectExchangeClient 代码并不是很复杂，该类会在 request 方法被调用时通过 Exchangers 的 connect 方法创建 ExchangeClient 客户端，该类的代码本节就不分析了。下面我们分析一下 Exchangers 的 connect 方法。

```java
public static ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    if (handler == null) {
        throw new IllegalArgumentException("handler == null");
    }
    url = url.addParameterIfAbsent(Constants.CODEC_KEY, "exchange");
    // 获取 Exchanger 实例，默认为 HeaderExchangeClient
    return getExchanger(url).connect(url, handler);
}
```

​	如上，getExchanger 会通过 SPI 加载 HeaderExchangeClient 实例，这个方法比较简单，大家自己看一下吧。接下来分析 HeaderExchangeClient 的实现。

```java
public ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
    // 这里包含了多个调用，分别如下：
    // 1. 创建 HeaderExchangeHandler 对象
    // 2. 创建 DecodeHandler 对象
    // 3. 通过 Transporters 构建 Client 实例
    // 4. 创建 HeaderExchangeClient 对象
    return new HeaderExchangeClient(Transporters.connect(url, new DecodeHandler(new HeaderExchangeHandler(handler))), true);
}
```

​	这里的调用比较多，我们这里重点看一下 Transporters 的 connect 方法。如下：

```java
public static Client connect(URL url, ChannelHandler... handlers) throws RemotingException {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    ChannelHandler handler;
    if (handlers == null || handlers.length == 0) {
        handler = new ChannelHandlerAdapter();
    } else if (handlers.length == 1) {
        handler = handlers[0];
    } else {
        // 如果 handler 数量大于1，则创建一个 ChannelHandler 分发器
        handler = new ChannelHandlerDispatcher(handlers);
    }
    
    // 获取 Transporter 自适应拓展类，并调用 connect 方法生成 Client 实例
    return getTransporter().connect(url, handler);
}
```

​	如上，getTransporter 方法返回的是自适应拓展类，该类会在运行时根据客户端类型加载指定的 Transporter 实现类。若用户未配置客户端类型，则默认加载 NettyTransporter，并调用该类的 connect 方法。如下：

```java
public Client connect(URL url, ChannelHandler listener) throws RemotingException {
    // 创建 NettyClient 对象
    return new NettyClient(url, listener);
}
```

​	到这里就不继续跟下去了，在往下就是通过 Netty 提供的 API 构建 Netty 客户端了，大家有兴趣自己看看。到这里，关于 DubboProtocol 的 refer 方法就分析完了。接下来，继续分析 RegistryProtocol 的 refer 方法逻辑。

```java
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
    // 取 registry 参数值，并将其设置为协议头
    url = url.setProtocol(url.getParameter(Constants.REGISTRY_KEY, Constants.DEFAULT_REGISTRY)).removeParameter(Constants.REGISTRY_KEY);
    // 获取注册中心实例
    Registry registry = registryFactory.getRegistry(url);
    if (RegistryService.class.equals(type)) {
        return proxyFactory.getInvoker((T) registry, type, url);
    }

    // 将 url 查询字符串转为 Map
    Map<String, String> qs = StringUtils.parseQueryString(url.getParameterAndDecoded(Constants.REFER_KEY));
    // 获取 group 配置
    String group = qs.get(Constants.GROUP_KEY);
    if (group != null && group.length() > 0) {
        if ((Constants.COMMA_SPLIT_PATTERN.split(group)).length > 1
                || "*".equals(group)) {
            // 通过 SPI 加载 MergeableCluster 实例，并调用 doRefer 继续执行服务引用逻辑
            return doRefer(getMergeableCluster(), registry, type, url);
        }
    }
    
    // 调用 doRefer 继续执行服务引用逻辑
    return doRefer(cluster, registry, type, url);
}
```

​	上面代码首先为 url 设置协议头，然后根据 url 参数加载注册中心实例。然后获取 group 配置，根据 group 配置决定 doRefer 第一个参数的类型。这里的重点是 doRefer 方法，如下：

```java
private <T> Invoker<T> doRefer(Cluster cluster, Registry registry, Class<T> type, URL url) {
    // 创建 RegistryDirectory 实例
    RegistryDirectory<T> directory = new RegistryDirectory<T>(type, url);
    // 设置注册中心和协议
    directory.setRegistry(registry);
    directory.setProtocol(protocol);
    Map<String, String> parameters = new HashMap<String, String>(directory.getUrl().getParameters());
    // 生成服务消费者链接
    URL subscribeUrl = new URL(Constants.CONSUMER_PROTOCOL, parameters.remove(Constants.REGISTER_IP_KEY), 0, type.getName(), parameters);

    // 注册服务消费者，在 consumers 目录下新节点
    if (!Constants.ANY_VALUE.equals(url.getServiceInterface())
            && url.getParameter(Constants.REGISTER_KEY, true)) {
        registry.register(subscribeUrl.addParameters(Constants.CATEGORY_KEY, Constants.CONSUMERS_CATEGORY,
                Constants.CHECK_KEY, String.valueOf(false)));
    }

    // 订阅 providers、configurators、routers 等节点数据
    directory.subscribe(subscribeUrl.addParameter(Constants.CATEGORY_KEY,
            Constants.PROVIDERS_CATEGORY
                    + "," + Constants.CONFIGURATORS_CATEGORY
                    + "," + Constants.ROUTERS_CATEGORY));

    // 一个注册中心可能有多个服务提供者，因此这里需要将多个服务提供者合并为一个
    Invoker invoker = cluster.join(directory);
    ProviderConsumerRegTable.registerConsumer(invoker, url, subscribeUrl, directory);
    return invoker;
}
```

#### 创建代理

​	Invoker 创建完毕后，接下来要做的事情是为服务接口生成代理对象。有了代理对象，即可进行远程调用。代理对象生成的入口方法为 ProxyFactory 的 getProxy，接下来进行分析。

```java
public <T> T getProxy(Invoker<T> invoker) throws RpcException {
    // 调用重载方法
    return getProxy(invoker, false);
}

public <T> T getProxy(Invoker<T> invoker, boolean generic) throws RpcException {
    Class<?>[] interfaces = null;
    // 获取接口列表
    String config = invoker.getUrl().getParameter("interfaces");
    if (config != null && config.length() > 0) {
        // 切分接口列表
        String[] types = Constants.COMMA_SPLIT_PATTERN.split(config);
        if (types != null && types.length > 0) {
            interfaces = new Class<?>[types.length + 2];
            // 设置服务接口类和 EchoService.class 到 interfaces 中
            interfaces[0] = invoker.getInterface();
            interfaces[1] = EchoService.class;
            for (int i = 0; i < types.length; i++) {
                // 加载接口类
                interfaces[i + 1] = ReflectUtils.forName(types[i]);
            }
        }
    }
    if (interfaces == null) {
        interfaces = new Class<?>[]{invoker.getInterface(), EchoService.class};
    }

    // 为 http 和 hessian 协议提供泛化调用支持，参考 pull request #1827
    if (!invoker.getInterface().equals(GenericService.class) && generic) {
        int len = interfaces.length;
        Class<?>[] temp = interfaces;
        // 创建新的 interfaces 数组
        interfaces = new Class<?>[len + 1];
        System.arraycopy(temp, 0, interfaces, 0, len);
        // 设置 GenericService.class 到数组中
        interfaces[len] = GenericService.class;
    }

    // 调用重载方法
    return getProxy(invoker, interfaces);
}

public abstract <T> T getProxy(Invoker<T> invoker, Class<?>[] types);
```

​	如上，上面大段代码都是用来获取 interfaces 数组的，我们继续往下看。getProxy(Invoker, Class<?>[]) 这个方法是一个抽象方法，下面我们到 JavassistProxyFactory 类中看一下该方法的实现代码。

```java
public <T> T getProxy(Invoker<T> invoker, Class<?>[] interfaces) {
    // 生成 Proxy 子类（Proxy 是抽象类）。并调用 Proxy 子类的 newInstance 方法创建 Proxy 实例
    return (T) Proxy.getProxy(interfaces).newInstance(new InvokerInvocationHandler(invoker));
}
```

​	上面代码并不多，首先是通过 Proxy 的 getProxy 方法获取 Proxy 子类，然后创建 InvokerInvocationHandler 对象，并将该对象传给 newInstance 生成 Proxy 实例。InvokerInvocationHandler 实现 JDK 的 InvocationHandler 接口，具体的用途是拦截接口类调用。该类逻辑比较简单，这里就不分析了。下面我们重点关注一下 Proxy 的 getProxy 方法，如下。

```java
public static Proxy getProxy(Class<?>... ics) {
    // 调用重载方法
    return getProxy(ClassHelper.getClassLoader(Proxy.class), ics);
}

public static Proxy getProxy(ClassLoader cl, Class<?>... ics) {
    if (ics.length > 65535)
        throw new IllegalArgumentException("interface limit exceeded");

    StringBuilder sb = new StringBuilder();
    // 遍历接口列表
    for (int i = 0; i < ics.length; i++) {
        String itf = ics[i].getName();
        // 检测类型是否为接口
        if (!ics[i].isInterface())
            throw new RuntimeException(itf + " is not a interface.");

        Class<?> tmp = null;
        try {
            // 重新加载接口类
            tmp = Class.forName(itf, false, cl);
        } catch (ClassNotFoundException e) {
        }

        // 检测接口是否相同，这里 tmp 有可能为空
        if (tmp != ics[i])
            throw new IllegalArgumentException(ics[i] + " is not visible from class loader");

        // 拼接接口全限定名，分隔符为 ;
        sb.append(itf).append(';');
    }

    // 使用拼接后的接口名作为 key
    String key = sb.toString();

    Map<String, Object> cache;
    synchronized (ProxyCacheMap) {
        cache = ProxyCacheMap.get(cl);
        if (cache == null) {
            cache = new HashMap<String, Object>();
            ProxyCacheMap.put(cl, cache);
        }
    }

    Proxy proxy = null;
    synchronized (cache) {
        do {
            // 从缓存中获取 Reference<Proxy> 实例
            Object value = cache.get(key);
            if (value instanceof Reference<?>) {
                proxy = (Proxy) ((Reference<?>) value).get();
                if (proxy != null) {
                    return proxy;
                }
            }

            // 并发控制，保证只有一个线程可以进行后续操作
            if (value == PendingGenerationMarker) {
                try {
                    // 其他线程在此处进行等待
                    cache.wait();
                } catch (InterruptedException e) {
                }
            } else {
                // 放置标志位到缓存中，并跳出 while 循环进行后续操作
                cache.put(key, PendingGenerationMarker);
                break;
            }
        }
        while (true);
    }

    long id = PROXY_CLASS_COUNTER.getAndIncrement();
    String pkg = null;
    ClassGenerator ccp = null, ccm = null;
    try {
        // 创建 ClassGenerator 对象
        ccp = ClassGenerator.newInstance(cl);

        Set<String> worked = new HashSet<String>();
        List<Method> methods = new ArrayList<Method>();

        for (int i = 0; i < ics.length; i++) {
            // 检测接口访问级别是否为 protected 或 private
            if (!Modifier.isPublic(ics[i].getModifiers())) {
                // 获取接口包名
                String npkg = ics[i].getPackage().getName();
                if (pkg == null) {
                    pkg = npkg;
                } else {
                    if (!pkg.equals(npkg))
                        // 非 public 级别的接口必须在同一个包下，否者抛出异常
                        throw new IllegalArgumentException("non-public interfaces from different packages");
                }
            }
            
            // 添加接口到 ClassGenerator 中
            ccp.addInterface(ics[i]);

            // 遍历接口方法
            for (Method method : ics[i].getMethods()) {
                // 获取方法描述，可理解为方法签名
                String desc = ReflectUtils.getDesc(method);
                // 如果方法描述字符串已在 worked 中，则忽略。考虑这种情况，
                // A 接口和 B 接口中包含一个完全相同的方法
                if (worked.contains(desc))
                    continue;
                worked.add(desc);

                int ix = methods.size();
                // 获取方法返回值类型
                Class<?> rt = method.getReturnType();
                // 获取参数列表
                Class<?>[] pts = method.getParameterTypes();

                // 生成 Object[] args = new Object[1...N]
                StringBuilder code = new StringBuilder("Object[] args = new Object[").append(pts.length).append("];");
                for (int j = 0; j < pts.length; j++)
                    // 生成 args[1...N] = ($w)$1...N;
                    code.append(" args[").append(j).append("] = ($w)$").append(j + 1).append(";");
                // 生成 InvokerHandler 接口的 invoker 方法调用语句，如下：
                // Object ret = handler.invoke(this, methods[1...N], args);
                code.append(" Object ret = handler.invoke(this, methods[" + ix + "], args);");

                // 返回值不为 void
                if (!Void.TYPE.equals(rt))
                    // 生成返回语句，形如 return (java.lang.String) ret;
                    code.append(" return ").append(asArgument(rt, "ret")).append(";");

                methods.add(method);
                // 添加方法名、访问控制符、参数列表、方法代码等信息到 ClassGenerator 中 
                ccp.addMethod(method.getName(), method.getModifiers(), rt, pts, method.getExceptionTypes(), code.toString());
            }
        }

        if (pkg == null)
            pkg = PACKAGE_NAME;

        // 构建接口代理类名称：pkg + ".proxy" + id，比如 org.apache.dubbo.proxy0
        String pcn = pkg + ".proxy" + id;
        ccp.setClassName(pcn);
        ccp.addField("public static java.lang.reflect.Method[] methods;");
        // 生成 private java.lang.reflect.InvocationHandler handler;
        ccp.addField("private " + InvocationHandler.class.getName() + " handler;");

        // 为接口代理类添加带有 InvocationHandler 参数的构造方法，比如：
        // porxy0(java.lang.reflect.InvocationHandler arg0) {
        //     handler=$1;
    	// }
        ccp.addConstructor(Modifier.PUBLIC, new Class<?>[]{InvocationHandler.class}, new Class<?>[0], "handler=$1;");
        // 为接口代理类添加默认构造方法
        ccp.addDefaultConstructor();
        
        // 生成接口代理类
        Class<?> clazz = ccp.toClass();
        clazz.getField("methods").set(null, methods.toArray(new Method[0]));

        // 构建 Proxy 子类名称，比如 Proxy1，Proxy2 等
        String fcn = Proxy.class.getName() + id;
        ccm = ClassGenerator.newInstance(cl);
        ccm.setClassName(fcn);
        ccm.addDefaultConstructor();
        ccm.setSuperClass(Proxy.class);
        // 为 Proxy 的抽象方法 newInstance 生成实现代码，形如：
        // public Object newInstance(java.lang.reflect.InvocationHandler h) { 
        //     return new org.apache.dubbo.proxy0($1);
        // }
        ccm.addMethod("public Object newInstance(" + InvocationHandler.class.getName() + " h){ return new " + pcn + "($1); }");
        // 生成 Proxy 实现类
        Class<?> pc = ccm.toClass();
        // 通过反射创建 Proxy 实例
        proxy = (Proxy) pc.newInstance();
    } catch (RuntimeException e) {
        throw e;
    } catch (Exception e) {
        throw new RuntimeException(e.getMessage(), e);
    } finally {
        if (ccp != null)
            // 释放资源
            ccp.release();
        if (ccm != null)
            ccm.release();
        synchronized (cache) {
            if (proxy == null)
                cache.remove(key);
            else
                // 写缓存
                cache.put(key, new WeakReference<Proxy>(proxy));
            // 唤醒其他等待线程
            cache.notifyAll();
        }
    }
    return proxy;
}
```

​	上面代码比较复杂，我们写了大量的注释。大家在阅读这段代码时，要搞清楚 ccp 和 ccm 的用途，不然会被搞晕。ccp 用于为服务接口生成代理类，比如我们有一个 DemoService 接口，这个接口代理类就是由 ccp 生成的。ccm 则是用于为 org.apache.dubbo.common.bytecode.Proxy 抽象类生成子类，主要是实现 Proxy 类的抽象方法。下面以 org.apache.dubbo.demo.DemoService 这个接口为例，来看一下该接口代理类代码大致是怎样的（忽略 EchoService 接口）。

```java
package org.apache.dubbo.common.bytecode;

public class proxy0 implements org.apache.dubbo.demo.DemoService {

    public static java.lang.reflect.Method[] methods;

    private java.lang.reflect.InvocationHandler handler;

    public proxy0() {
    }

    public proxy0(java.lang.reflect.InvocationHandler arg0) {
        handler = $1;
    }

    public java.lang.String sayHello(java.lang.String arg0) {
        Object[] args = new Object[1];
        args[0] = ($w) $1;
        Object ret = handler.invoke(this, methods[0], args);
        return (java.lang.String) ret;
    }
}
```

# 服务调用过程

​	在前面的文章中，我们分析了 Dubbo SPI、服务导出与引入、以及集群容错方面的代码。经过前文的铺垫，本篇文章我们终于可以分析服务调用过程了。Dubbo 服务调用过程比较复杂，包含众多步骤，比如发送请求、编解码、服务降级、过滤器链处理、序列化、线程派发以及响应请求等步骤。限于篇幅原因，本篇文章无法对所有的步骤一一进行分析。本篇文章将会重点分析请求的发送与接收、编解码、线程派发以及响应的发送与接收等过程，至于服务降级、过滤器链和序列化大家自行进行分析，也可以将其当成一个黑盒，暂时忽略也没关系。介绍完本篇文章要分析的内容，接下来我们进入正题吧。

## 源码分析 2.7

​	在进行源码分析之前，我们先来通过一张图了解 Dubbo 服务调用过程。

![](https://pic.imgdb.cn/item/612f403644eaada739b958b9.jpg)



​	首先服务消费者通过代理对象 Proxy 发起远程调用，接着通过网络客户端 Client 将编码后的请求发送给服务提供方的网络层上，也就是 Server。Server 在收到请求后，首先要做的事情是对数据包进行解码。然后将解码后的请求发送至分发器 Dispatcher，再由分发器将请求派发到指定的线程池上，最后由线程池调用具体的服务。这就是一个远程调用请求的发送与接收过程。至于响应的发送与接收过程，这张图中没有表现出来。对于这两个过程，我们也会进行详细分析。

### 服务调用方式

​	Dubbo 支持同步和异步两种调用方式，其中异步调用还可细分为“有返回值”的异步调用和“无返回值”的异步调用。所谓“无返回值”异步调用是指服务消费方只管调用，但不关心调用结果，此时 Dubbo 会直接返回一个空的 RpcResult。若要使用异步特性，需要服务消费方手动进行配置。默认情况下，Dubbo 使用同步调用方式。

​	本节以及其他章节将会使用 Dubbo 官方提供的 Demo 分析整个调用过程，下面我们从 DemoService 接口的代理类开始进行分析。Dubbo 默认使用 Javassist 框架为服务接口生成动态代理类，因此我们需要先将代理类进行反编译才能看到源码。这里使用阿里开源 Java 应用诊断工具 [Arthas](https://github.com/alibaba/arthas) 反编译代理类，结果如下：

```java
/**
 * Arthas 反编译步骤：
 * 1. 启动 Arthas
 *    java -jar arthas-boot.jar
 *
 * 2. 输入编号选择进程
 *    Arthas 启动后，会打印 Java 应用进程列表，如下：
 *    [1]: 11232 org.jetbrains.jps.cmdline.Launcher
 *    [2]: 22370 org.jetbrains.jps.cmdline.Launcher
 *    [3]: 22371 com.alibaba.dubbo.demo.consumer.Consumer
 *    [4]: 22362 com.alibaba.dubbo.demo.provider.Provider
 *    [5]: 2074 org.apache.zookeeper.server.quorum.QuorumPeerMain
 * 这里输入编号 3，让 Arthas 关联到启动类为 com.....Consumer 的 Java 进程上
 *
 * 3. 由于 Demo 项目中只有一个服务接口，因此此接口的代理类类名为 proxy0，此时使用 sc 命令搜索这个类名。
 *    $ sc *.proxy0
 *    com.alibaba.dubbo.common.bytecode.proxy0
 *
 * 4. 使用 jad 命令反编译 com.alibaba.dubbo.common.bytecode.proxy0
 *    $ jad com.alibaba.dubbo.common.bytecode.proxy0
 *
 * 更多使用方法请参考 Arthas 官方文档：
 *   https://alibaba.github.io/arthas/quick-start.html
 */
public class proxy0 implements ClassGenerator.DC, EchoService, DemoService {
    // 方法数组
    public static Method[] methods;
    private InvocationHandler handler;

    public proxy0(InvocationHandler invocationHandler) {
        this.handler = invocationHandler;
    }

    public proxy0() {
    }

    public String sayHello(String string) {
        // 将参数存储到 Object 数组中
        Object[] arrobject = new Object[]{string};
        // 调用 InvocationHandler 实现类的 invoke 方法得到调用结果
        Object object = this.handler.invoke(this, methods[0], arrobject);
        // 返回调用结果
        return (String)object;
    }

    /** 回声测试方法 */
    public Object $echo(Object object) {
        Object[] arrobject = new Object[]{object};
        Object object2 = this.handler.invoke(this, methods[1], arrobject);
        return object2;
    }
}
```

​	如上，代理类的逻辑比较简单。首先将运行时参数存储到数组中，然后调用 InvocationHandler 接口实现类的 invoke 方法，得到调用结果，最后将结果转型并返回给调用方。关于代理类的逻辑就说这么多，继续向下分析。

```java
public class InvokerInvocationHandler implements InvocationHandler {

    private final Invoker<?> invoker;

    public InvokerInvocationHandler(Invoker<?> handler) {
        this.invoker = handler;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        Class<?>[] parameterTypes = method.getParameterTypes();
        
        // 拦截定义在 Object 类中的方法（未被子类重写），比如 wait/notify
        if (method.getDeclaringClass() == Object.class) {
            return method.invoke(invoker, args);
        }
        
        // 如果 toString、hashCode 和 equals 等方法被子类重写了，这里也直接调用
        if ("toString".equals(methodName) && parameterTypes.length == 0) {
            return invoker.toString();
        }
        if ("hashCode".equals(methodName) && parameterTypes.length == 0) {
            return invoker.hashCode();
        }
        if ("equals".equals(methodName) && parameterTypes.length == 1) {
            return invoker.equals(args[0]);
        }
        
        // 将 method 和 args 封装到 RpcInvocation 中，并执行后续的调用
        return invoker.invoke(new RpcInvocation(method, args)).recreate();
    }
}
```

​	InvokerInvocationHandler 中的 invoker 成员变量类型为 MockClusterInvoker，MockClusterInvoker 内部封装了服务降级逻辑。下面简单看一下：

```java
public class MockClusterInvoker<T> implements Invoker<T> {
    
    private final Invoker<T> invoker;
    
    public Result invoke(Invocation invocation) throws RpcException {
        Result result = null;

        // 获取 mock 配置值
        String value = directory.getUrl().getMethodParameter(invocation.getMethodName(), Constants.MOCK_KEY, Boolean.FALSE.toString()).trim();
        if (value.length() == 0 || value.equalsIgnoreCase("false")) {
            // 无 mock 逻辑，直接调用其他 Invoker 对象的 invoke 方法，
            // 比如 FailoverClusterInvoker
            result = this.invoker.invoke(invocation);
        } else if (value.startsWith("force")) {
            // force:xxx 直接执行 mock 逻辑，不发起远程调用
            result = doMockInvoke(invocation, null);
        } else {
            // fail:xxx 表示消费方对调用服务失败后，再执行 mock 逻辑，不抛出异常
            try {
                // 调用其他 Invoker 对象的 invoke 方法
                result = this.invoker.invoke(invocation);
            } catch (RpcException e) {
                if (e.isBiz()) {
                    throw e;
                } else {
                    // 调用失败，执行 mock 逻辑
                    result = doMockInvoke(invocation, e);
                }
            }
        }
        return result;
    }
    
    // 省略其他方法
}
```

​	服务降级不是本文重点，因此这里就不分析 doMockInvoke 方法了。考虑到前文已经详细分析过 FailoverClusterInvoker，因此本节略过 FailoverClusterInvoker，直接分析 DubboInvoker。

```java
public abstract class AbstractInvoker<T> implements Invoker<T> {
    
    public Result invoke(Invocation inv) throws RpcException {
        if (destroyed.get()) {
            throw new RpcException("Rpc invoker for service ...");
        }
        RpcInvocation invocation = (RpcInvocation) inv;
        // 设置 Invoker
        invocation.setInvoker(this);
        if (attachment != null && attachment.size() > 0) {
            // 设置 attachment
            invocation.addAttachmentsIfAbsent(attachment);
        }
        Map<String, String> contextAttachments = RpcContext.getContext().getAttachments();
        if (contextAttachments != null && contextAttachments.size() != 0) {
            // 添加 contextAttachments 到 RpcInvocation#attachment 变量中
            invocation.addAttachments(contextAttachments);
        }
        if (getUrl().getMethodParameter(invocation.getMethodName(), Constants.ASYNC_KEY, false)) {
            // 设置异步信息到 RpcInvocation#attachment 中
            invocation.setAttachment(Constants.ASYNC_KEY, Boolean.TRUE.toString());
        }
        RpcUtils.attachInvocationIdIfAsync(getUrl(), invocation);

        try {
            // 抽象方法，由子类实现
            return doInvoke(invocation);
        } catch (InvocationTargetException e) {
            // ...
        } catch (RpcException e) {
            // ...
        } catch (Throwable e) {
            return new RpcResult(e);
        }
    }

    protected abstract Result doInvoke(Invocation invocation) throws Throwable;
    
    // 省略其他方法
}
```

​	上面的代码来自 AbstractInvoker 类，其中大部分代码用于添加信息到 RpcInvocation#attachment 变量中，添加完毕后，调用 doInvoke 执行后续的调用。doInvoke 是一个抽象方法，需要由子类实现，下面到 DubboInvoker 中看一下。

```java
public class DubboInvoker<T> extends AbstractInvoker<T> {
    
    private final ExchangeClient[] clients;
    
    protected Result doInvoke(final Invocation invocation) throws Throwable {
        RpcInvocation inv = (RpcInvocation) invocation;
        final String methodName = RpcUtils.getMethodName(invocation);
        // 设置 path 和 version 到 attachment 中
        inv.setAttachment(Constants.PATH_KEY, getUrl().getPath());
        inv.setAttachment(Constants.VERSION_KEY, version);

        ExchangeClient currentClient;
        if (clients.length == 1) {
            // 从 clients 数组中获取 ExchangeClient
            currentClient = clients[0];
        } else {
            currentClient = clients[index.getAndIncrement() % clients.length];
        }
        try {
            // 获取异步配置
            boolean isAsync = RpcUtils.isAsync(getUrl(), invocation);
            // isOneway 为 true，表示“单向”通信
            boolean isOneway = RpcUtils.isOneway(getUrl(), invocation);
            int timeout = getUrl().getMethodParameter(methodName, Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);

            // 异步无返回值
            if (isOneway) {
                boolean isSent = getUrl().getMethodParameter(methodName, Constants.SENT_KEY, false);
                // 发送请求
                currentClient.send(inv, isSent);
                // 设置上下文中的 future 字段为 null
                RpcContext.getContext().setFuture(null);
                // 返回一个空的 RpcResult
                return new RpcResult();
            } 

            // 异步有返回值
            else if (isAsync) {
                // 发送请求，并得到一个 ResponseFuture 实例
                ResponseFuture future = currentClient.request(inv, timeout);
                // 设置 future 到上下文中
                RpcContext.getContext().setFuture(new FutureAdapter<Object>(future));
                // 暂时返回一个空结果
                return new RpcResult();
            } 

            // 同步调用
            else {
                RpcContext.getContext().setFuture(null);
                // 发送请求，得到一个 ResponseFuture 实例，并调用该实例的 get 方法进行等待
                return (Result) currentClient.request(inv, timeout).get();
            }
        } catch (TimeoutException e) {
            throw new RpcException(..., "Invoke remote method timeout....");
        } catch (RemotingException e) {
            throw new RpcException(..., "Failed to invoke remote method: ...");
        }
    }
    
    // 省略其他方法
}
```

​	上面的代码包含了 Dubbo 对同步和异步调用的处理逻辑，搞懂了上面的代码，会对 Dubbo 的同步和异步调用方式有更深入的了解。Dubbo 实现同步和异步调用比较关键的一点就在于由谁调用 ResponseFuture 的 get 方法。同步调用模式下，由框架自身调用 ResponseFuture 的 get 方法。异步调用模式下，则由用户调用该方法。ResponseFuture 是一个接口，下面我们来看一下它的默认实现类 DefaultFuture 的源码。

```java
public class DefaultFuture implements ResponseFuture {
    
    private static final Map<Long, Channel> CHANNELS = 
        new ConcurrentHashMap<Long, Channel>();

    private static final Map<Long, DefaultFuture> FUTURES = 
        new ConcurrentHashMap<Long, DefaultFuture>();
    
    private final long id;
    private final Channel channel;
    private final Request request;
    private final int timeout;
    private final Lock lock = new ReentrantLock();
    private final Condition done = lock.newCondition();
    private volatile Response response;
    
    public DefaultFuture(Channel channel, Request request, int timeout) {
        this.channel = channel;
        this.request = request;
        
        // 获取请求 id，这个 id 很重要，后面还会见到
        this.id = request.getId();
        this.timeout = timeout > 0 ? timeout : channel.getUrl().getPositiveParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
        // 存储 <requestId, DefaultFuture> 映射关系到 FUTURES 中
        FUTURES.put(id, this);
        CHANNELS.put(id, channel);
    }
    
    @Override
    public Object get() throws RemotingException {
        return get(timeout);
    }

    @Override
    public Object get(int timeout) throws RemotingException {
        if (timeout <= 0) {
            timeout = Constants.DEFAULT_TIMEOUT;
        }
        
        // 检测服务提供方是否成功返回了调用结果
        if (!isDone()) {
            long start = System.currentTimeMillis();
            lock.lock();
            try {
                // 循环检测服务提供方是否成功返回了调用结果
                while (!isDone()) {
                    // 如果调用结果尚未返回，这里等待一段时间
                    done.await(timeout, TimeUnit.MILLISECONDS);
                    // 如果调用结果成功返回，或等待超时，此时跳出 while 循环，执行后续的逻辑
                    if (isDone() || System.currentTimeMillis() - start > timeout) {
                        break;
                    }
                }
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
            
            // 如果调用结果仍未返回，则抛出超时异常
            if (!isDone()) {
                throw new TimeoutException(sent > 0, channel, getTimeoutMessage(false));
            }
        }
        
        // 返回调用结果
        return returnFromResponse();
    }
    
    @Override
    public boolean isDone() {
        // 通过检测 response 字段为空与否，判断是否收到了调用结果
        return response != null;
    }
    
    private Object returnFromResponse() throws RemotingException {
        Response res = response;
        if (res == null) {
            throw new IllegalStateException("response cannot be null");
        }
        
        // 如果调用结果的状态为 Response.OK，则表示调用过程正常，服务提供方成功返回了调用结果
        if (res.getStatus() == Response.OK) {
            return res.getResult();
        }
        
        // 抛出异常
        if (res.getStatus() == Response.CLIENT_TIMEOUT || res.getStatus() == Response.SERVER_TIMEOUT) {
            throw new TimeoutException(res.getStatus() == Response.SERVER_TIMEOUT, channel, res.getErrorMessage());
        }
        throw new RemotingException(channel, res.getErrorMessage());
    }
    
    // 省略其他方法
}
```

​	如上，当服务消费者还未接收到调用结果时，用户线程调用 get 方法会被阻塞住。同步调用模式下，框架获得 DefaultFuture 对象后，会立即调用 get 方法进行等待。而异步模式下则是将该对象封装到 FutureAdapter 实例中，并将 FutureAdapter 实例设置到 RpcContext 中，供用户使用。FutureAdapter 是一个适配器，用于将 Dubbo 中的 ResponseFuture 与 JDK 中的 Future 进行适配。这样当用户线程调用 Future 的 get 方法时，经过 FutureAdapter 适配，最终会调用 ResponseFuture 实现类对象的 get 方法，也就是 DefaultFuture 的 get 方法。

​	到这里关于 Dubbo 几种调用方式的代码逻辑就分析完了，下面来分析请求数据的发送与接收，以及响应数据的发送与接收过程。

### 服务消费方发送请求

#### 发送请求

​	本节我们来看一下同步调用模式下，服务消费方是如何发送调用请求的。在深入分析源码前，我们先来看一张图。

![](https://pic.imgdb.cn/item/612f435a44eaada739be7bf1.jpg)

​	这张图展示了服务消费方发送请求过程的部分调用栈，略为复杂。从上图可以看出，经过多次调用后，才将请求数据送至 Netty NioClientSocketChannel。这样做的原因是通过 Exchange 层为框架引入 Request 和 Response 语义，这一点会在接下来的源码分析过程中会看到。其他的就不多说了，下面开始进行分析。首先分析 ReferenceCountExchangeClient 的源码。

```java
final class ReferenceCountExchangeClient implements ExchangeClient {

    private final URL url;
    private final AtomicInteger referenceCount = new AtomicInteger(0);

    public ReferenceCountExchangeClient(ExchangeClient client, ConcurrentMap<String, LazyConnectExchangeClient> ghostClientMap) {
        this.client = client;
        // 引用计数自增
        referenceCount.incrementAndGet();
        this.url = client.getUrl();
        
        // ...
    }

    @Override
    public ResponseFuture request(Object request) throws RemotingException {
        // 直接调用被装饰对象的同签名方法
        return client.request(request);
    }

    @Override
    public ResponseFuture request(Object request, int timeout) throws RemotingException {
        // 直接调用被装饰对象的同签名方法
        return client.request(request, timeout);
    }

    /** 引用计数自增，该方法由外部调用 */
    public void incrementAndGetCount() {
        // referenceCount 自增
        referenceCount.incrementAndGet();
    }
    
        @Override
    public void close(int timeout) {
        // referenceCount 自减
        if (referenceCount.decrementAndGet() <= 0) {
            if (timeout == 0) {
                client.close();
            } else {
                client.close(timeout);
            }
            client = replaceWithLazyClient();
        }
    }
    
    // 省略部分方法
}
```

​	ReferenceCountExchangeClient 内部定义了一个引用计数变量 referenceCount，每当该对象被引用一次 referenceCount 都会进行自增。每当 close 方法被调用时，referenceCount 进行自减。ReferenceCountExchangeClient 内部仅实现了一个引用计数的功能，其他方法并无复杂逻辑，均是直接调用被装饰对象的相关方法。所以这里就不多说了，继续向下分析，这次是 HeaderExchangeClient。

```java
public class HeaderExchangeClient implements ExchangeClient {

    private static final ScheduledThreadPoolExecutor scheduled = new ScheduledThreadPoolExecutor(2, new NamedThreadFactory("dubbo-remoting-client-heartbeat", true));
    private final Client client;
    private final ExchangeChannel channel;
    private ScheduledFuture<?> heartbeatTimer;
    private int heartbeat;
    private int heartbeatTimeout;

    public HeaderExchangeClient(Client client, boolean needHeartbeat) {
        if (client == null) {
            throw new IllegalArgumentException("client == null");
        }
        this.client = client;
        
        // 创建 HeaderExchangeChannel 对象
        this.channel = new HeaderExchangeChannel(client);
        
        // 以下代码均与心跳检测逻辑有关
        String dubbo = client.getUrl().getParameter(Constants.DUBBO_VERSION_KEY);
        this.heartbeat = client.getUrl().getParameter(Constants.HEARTBEAT_KEY, dubbo != null && dubbo.startsWith("1.0.") ? Constants.DEFAULT_HEARTBEAT : 0);
        this.heartbeatTimeout = client.getUrl().getParameter(Constants.HEARTBEAT_TIMEOUT_KEY, heartbeat * 3);
        if (heartbeatTimeout < heartbeat * 2) {
            throw new IllegalStateException("heartbeatTimeout < heartbeatInterval * 2");
        }
        if (needHeartbeat) {
            // 开启心跳检测定时器
            startHeartbeatTimer();
        }
    }

    @Override
    public ResponseFuture request(Object request) throws RemotingException {
        // 直接 HeaderExchangeChannel 对象的同签名方法
        return channel.request(request);
    }

    @Override
    public ResponseFuture request(Object request, int timeout) throws RemotingException {
        // 直接 HeaderExchangeChannel 对象的同签名方法
        return channel.request(request, timeout);
    }

    @Override
    public void close() {
        doClose();
        channel.close();
    }
    
    private void doClose() {
        // 停止心跳检测定时器
        stopHeartbeatTimer();
    }

    private void startHeartbeatTimer() {
        stopHeartbeatTimer();
        if (heartbeat > 0) {
            heartbeatTimer = scheduled.scheduleWithFixedDelay(
                    new HeartBeatTask(new HeartBeatTask.ChannelProvider() {
                        @Override
                        public Collection<Channel> getChannels() {
                            return Collections.<Channel>singletonList(HeaderExchangeClient.this);
                        }
                    }, heartbeat, heartbeatTimeout),
                    heartbeat, heartbeat, TimeUnit.MILLISECONDS);
        }
    }

    private void stopHeartbeatTimer() {
        if (heartbeatTimer != null && !heartbeatTimer.isCancelled()) {
            try {
                heartbeatTimer.cancel(true);
                scheduled.purge();
            } catch (Throwable e) {
                if (logger.isWarnEnabled()) {
                    logger.warn(e.getMessage(), e);
                }
            }
        }
        heartbeatTimer = null;
    }
    
    // 省略部分方法
}
```

​	HeaderExchangeClient 中很多方法只有一行代码，即调用 HeaderExchangeChannel 对象的同签名方法。那 HeaderExchangeClient 有什么用处呢？答案是封装了一些关于心跳检测的逻辑。心跳检测并非本文所关注的点，因此就不多说了，继续向下看。

```java
final class HeaderExchangeChannel implements ExchangeChannel {
    
    private final Channel channel;
    
    HeaderExchangeChannel(Channel channel) {
        if (channel == null) {
            throw new IllegalArgumentException("channel == null");
        }
        
        // 这里的 channel 指向的是 NettyClient
        this.channel = channel;
    }
    
    @Override
    public ResponseFuture request(Object request) throws RemotingException {
        return request(request, channel.getUrl().getPositiveParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT));
    }

    @Override
    public ResponseFuture request(Object request, int timeout) throws RemotingException {
        if (closed) {
            throw new RemotingException(..., "Failed to send request ...);
        }
        // 创建 Request 对象
        Request req = new Request();
        req.setVersion(Version.getProtocolVersion());
        // 设置双向通信标志为 true
        req.setTwoWay(true);
        // 这里的 request 变量类型为 RpcInvocation
        req.setData(request);
                                        
        // 创建 DefaultFuture 对象
        DefaultFuture future = new DefaultFuture(channel, req, timeout);
        try {
            // 调用 NettyClient 的 send 方法发送请求
            channel.send(req);
        } catch (RemotingException e) {
            future.cancel();
            throw e;
        }
        // 返回 DefaultFuture 对象
        return future;
    }
}
```

​	到这里大家终于看到了 Request 语义了，上面的方法首先定义了一个 Request 对象，然后再将该对象传给 NettyClient 的 send 方法，进行后续的调用。需要说明的是，NettyClient 中并未实现 send 方法，该方法继承自父类 AbstractPeer，下面直接分析 AbstractPeer 的代码。

```java
public abstract class AbstractPeer implements Endpoint, ChannelHandler {
    
    @Override
    public void send(Object message) throws RemotingException {
        // 该方法由 AbstractClient 类实现
        send(message, url.getParameter(Constants.SENT_KEY, false));
    }
    
    // 省略其他方法
}

public abstract class AbstractClient extends AbstractEndpoint implements Client {
    
    @Override
    public void send(Object message, boolean sent) throws RemotingException {
        if (send_reconnect && !isConnected()) {
            connect();
        }
        
        // 获取 Channel，getChannel 是一个抽象方法，具体由子类实现
        Channel channel = getChannel();
        if (channel == null || !channel.isConnected()) {
            throw new RemotingException(this, "message can not send ...");
        }
        
        // 继续向下调用
        channel.send(message, sent);
    }
    
    protected abstract Channel getChannel();
    
    // 省略其他方法
}
```

​	默认情况下，Dubbo 使用 Netty 作为底层的通信框架，因此下面我们到 NettyClient 类中看一下 getChannel 方法的实现逻辑。

```java
public class NettyClient extends AbstractClient {
    
    // 这里的 Channel 全限定名称为 org.jboss.netty.channel.Channel
    private volatile Channel channel;

    @Override
    protected com.alibaba.dubbo.remoting.Channel getChannel() {
        Channel c = channel;
        if (c == null || !c.isConnected())
            return null;
        // 获取一个 NettyChannel 类型对象
        return NettyChannel.getOrAddChannel(c, getUrl(), this);
    }
}

final class NettyChannel extends AbstractChannel {

    private static final ConcurrentMap<org.jboss.netty.channel.Channel, NettyChannel> channelMap = 
        new ConcurrentHashMap<org.jboss.netty.channel.Channel, NettyChannel>();

    private final org.jboss.netty.channel.Channel channel;
    
    /** 私有构造方法 */
    private NettyChannel(org.jboss.netty.channel.Channel channel, URL url, ChannelHandler handler) {
        super(url, handler);
        if (channel == null) {
            throw new IllegalArgumentException("netty channel == null;");
        }
        this.channel = channel;
    }

    static NettyChannel getOrAddChannel(org.jboss.netty.channel.Channel ch, URL url, ChannelHandler handler) {
        if (ch == null) {
            return null;
        }
        
        // 尝试从集合中获取 NettyChannel 实例
        NettyChannel ret = channelMap.get(ch);
        if (ret == null) {
            // 如果 ret = null，则创建一个新的 NettyChannel 实例
            NettyChannel nc = new NettyChannel(ch, url, handler);
            if (ch.isConnected()) {
                // 将 <Channel, NettyChannel> 键值对存入 channelMap 集合中
                ret = channelMap.putIfAbsent(ch, nc);
            }
            if (ret == null) {
                ret = nc;
            }
        }
        return ret;
    }
}
```

​	获取到 NettyChannel 实例后，即可进行后续的调用。下面看一下 NettyChannel 的 send 方法。

```java
public void send(Object message, boolean sent) throws RemotingException {
    super.send(message, sent);

    boolean success = true;
    int timeout = 0;
    try {
        // 发送消息(包含请求和响应消息)
        ChannelFuture future = channel.write(message);
        
        // sent 的值源于 <dubbo:method sent="true/false" /> 中 sent 的配置值，有两种配置值：
        //   1. true: 等待消息发出，消息发送失败将抛出异常
        //   2. false: 不等待消息发出，将消息放入 IO 队列，即刻返回
        // 默认情况下 sent = false；
        if (sent) {
            timeout = getUrl().getPositiveParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
            // 等待消息发出，若在规定时间没能发出，success 会被置为 false
            success = future.await(timeout);
        }
        Throwable cause = future.getCause();
        if (cause != null) {
            throw cause;
        }
    } catch (Throwable e) {
        throw new RemotingException(this, "Failed to send message ...");
    }

    // 若 success 为 false，这里抛出异常
    if (!success) {
        throw new RemotingException(this, "Failed to send message ...");
    }
}
```

经历多次调用，到这里请求数据的发送过程就结束了，过程漫长。为了便于大家阅读代码，这里以 DemoService 为例，将 sayHello 方法的整个调用路径贴出来。

```fallback
proxy0#sayHello(String)
  —> InvokerInvocationHandler#invoke(Object, Method, Object[])
    —> MockClusterInvoker#invoke(Invocation)
      —> AbstractClusterInvoker#invoke(Invocation)
        —> FailoverClusterInvoker#doInvoke(Invocation, List<Invoker<T>>, LoadBalance)
          —> Filter#invoke(Invoker, Invocation)  // 包含多个 Filter 调用
            —> ListenerInvokerWrapper#invoke(Invocation) 
              —> AbstractInvoker#invoke(Invocation) 
                —> DubboInvoker#doInvoke(Invocation)
                  —> ReferenceCountExchangeClient#request(Object, int)
                    —> HeaderExchangeClient#request(Object, int)
                      —> HeaderExchangeChannel#request(Object, int)
                        —> AbstractPeer#send(Object)
                          —> AbstractClient#send(Object, boolean)
                            —> NettyChannel#send(Object, boolean)
                              —> NioClientSocketChannel#write(Object)
```

​	在 Netty 中，出站数据在发出之前还需要进行编码操作，接下来我们来分析一下请求数据的编码逻辑。

#### 请求编码

​	在分析请求编码逻辑之前，我们先来看一下 Dubbo 数据包结构。

![](https://pic.imgdb.cn/item/612f467a44eaada739c3493f.jpg)

​	Dubbo 数据包分为消息头和消息体，消息头用于存储一些元信息，比如魔数（Magic），数据包类型（Request/Response），消息体长度（Data Length）等。消息体中用于存储具体的调用消息，比如方法名称，参数列表等。下面简单列举一下消息头的内容。

| 偏移量(Bit) | 字段         | 取值                                                         |
| ----------- | ------------ | ------------------------------------------------------------ |
| 0 ~ 7       | 魔数高位     | 0xda00                                                       |
| 8 ~ 15      | 魔数低位     | 0xbb                                                         |
| 16          | 数据包类型   | 0 - Response, 1 - Request                                    |
| 17          | 调用方式     | 仅在第16位被设为1的情况下有效，0 - 单向调用，1 - 双向调用    |
| 18          | 事件标识     | 0 - 当前数据包是请求或响应包，1 - 当前数据包是心跳包         |
| 19 ~ 23     | 序列化器编号 | 2 - Hessian2Serialization 3 - JavaSerialization 4 - CompactedJavaSerialization 6 - FastJsonSerialization 7 - NativeJavaSerialization 8 - KryoSerialization 9 - FstSerialization |
| 24 ~ 31     | 状态         | 20 - OK 30 - CLIENT_TIMEOUT 31 - SERVER_TIMEOUT 40 - BAD_REQUEST 50 - BAD_RESPONSE …… |
| 32 ~ 95     | 请求编号     | 共8字节，运行时生成                                          |
| 96 ~ 127    | 消息体长度   | 运行时计算                                                   |

​	了解了 Dubbo 数据包格式，接下来我们就可以探索编码过程了。这次我们开门见山，直接分析编码逻辑所在类。如下

```java
public class ExchangeCodec extends TelnetCodec {

    // 消息头长度
    protected static final int HEADER_LENGTH = 16;
    // 魔数内容
    protected static final short MAGIC = (short) 0xdabb;
    protected static final byte MAGIC_HIGH = Bytes.short2bytes(MAGIC)[0];
    protected static final byte MAGIC_LOW = Bytes.short2bytes(MAGIC)[1];
    protected static final byte FLAG_REQUEST = (byte) 0x80;
    protected static final byte FLAG_TWOWAY = (byte) 0x40;
    protected static final byte FLAG_EVENT = (byte) 0x20;
    protected static final int SERIALIZATION_MASK = 0x1f;
    private static final Logger logger = LoggerFactory.getLogger(ExchangeCodec.class);

    public Short getMagicCode() {
        return MAGIC;
    }

    @Override
    public void encode(Channel channel, ChannelBuffer buffer, Object msg) throws IOException {
        if (msg instanceof Request) {
            // 对 Request 对象进行编码
            encodeRequest(channel, buffer, (Request) msg);
        } else if (msg instanceof Response) {
            // 对 Response 对象进行编码，后面分析
            encodeResponse(channel, buffer, (Response) msg);
        } else {
            super.encode(channel, buffer, msg);
        }
    }

    protected void encodeRequest(Channel channel, ChannelBuffer buffer, Request req) throws IOException {
        Serialization serialization = getSerialization(channel);

        // 创建消息头字节数组，长度为 16
        byte[] header = new byte[HEADER_LENGTH];

        // 设置魔数
        Bytes.short2bytes(MAGIC, header);

        // 设置数据包类型（Request/Response）和序列化器编号
        header[2] = (byte) (FLAG_REQUEST | serialization.getContentTypeId());

        // 设置通信方式(单向/双向)
        if (req.isTwoWay()) {
            header[2] |= FLAG_TWOWAY;
        }
        
        // 设置事件标识
        if (req.isEvent()) {
            header[2] |= FLAG_EVENT;
        }

        // 设置请求编号，8个字节，从第4个字节开始设置
        Bytes.long2bytes(req.getId(), header, 4);

        // 获取 buffer 当前的写位置
        int savedWriteIndex = buffer.writerIndex();
        // 更新 writerIndex，为消息头预留 16 个字节的空间
        buffer.writerIndex(savedWriteIndex + HEADER_LENGTH);
        ChannelBufferOutputStream bos = new ChannelBufferOutputStream(buffer);
        // 创建序列化器，比如 Hessian2ObjectOutput
        ObjectOutput out = serialization.serialize(channel.getUrl(), bos);
        if (req.isEvent()) {
            // 对事件数据进行序列化操作
            encodeEventData(channel, out, req.getData());
        } else {
            // 对请求数据进行序列化操作
            encodeRequestData(channel, out, req.getData(), req.getVersion());
        }
        out.flushBuffer();
        if (out instanceof Cleanable) {
            ((Cleanable) out).cleanup();
        }
        bos.flush();
        bos.close();
        
        // 获取写入的字节数，也就是消息体长度
        int len = bos.writtenBytes();
        checkPayload(channel, len);

        // 将消息体长度写入到消息头中
        Bytes.int2bytes(len, header, 12);

        // 将 buffer 指针移动到 savedWriteIndex，为写消息头做准备
        buffer.writerIndex(savedWriteIndex);
        // 从 savedWriteIndex 下标处写入消息头
        buffer.writeBytes(header);
        // 设置新的 writerIndex，writerIndex = 原写下标 + 消息头长度 + 消息体长度
        buffer.writerIndex(savedWriteIndex + HEADER_LENGTH + len);
    }
    
    // 省略其他方法
}
```

​	以上就是请求对象的编码过程，该过程首先会通过位运算将消息头写入到 header 数组中。然后对 Request 对象的 data 字段执行序列化操作，序列化后的数据最终会存储到 ChannelBuffer 中。序列化操作执行完后，可得到数据序列化后的长度 len，紧接着将 len 写入到 header 指定位置处。最后再将消息头字节数组 header 写入到 ChannelBuffer 中，整个编码过程就结束了。本节的最后，我们再来看一下 Request 对象的 data 字段序列化过程，也就是 encodeRequestData 方法的逻辑，如下：

```java
public class DubboCodec extends ExchangeCodec implements Codec2 {
    
	protected void encodeRequestData(Channel channel, ObjectOutput out, Object data, String version) throws IOException {
        RpcInvocation inv = (RpcInvocation) data;

        // 依次序列化 dubbo version、path、version
        out.writeUTF(version);
        out.writeUTF(inv.getAttachment(Constants.PATH_KEY));
        out.writeUTF(inv.getAttachment(Constants.VERSION_KEY));

        // 序列化调用方法名
        out.writeUTF(inv.getMethodName());
        // 将参数类型转换为字符串，并进行序列化
        out.writeUTF(ReflectUtils.getDesc(inv.getParameterTypes()));
        Object[] args = inv.getArguments();
        if (args != null)
            for (int i = 0; i < args.length; i++) {
                // 对运行时参数进行序列化
                out.writeObject(encodeInvocationArgument(channel, inv, i));
            }
        
        // 序列化 attachments
        out.writeObject(inv.getAttachments());
    }
}
```

​	至此，关于服务消费方发送请求的过程就分析完了，接下来我们来看一下服务提供方是如何接收请求的。

### 服务提供方接收请求

​	前面说过，默认情况下 Dubbo 使用 Netty 作为底层的通信框架。Netty 检测到有数据入站后，首先会通过解码器对数据进行解码，并将解码后的数据传递给下一个入站处理器的指定方法。所以在进行后续的分析之前，我们先来看一下数据解码过程。

#### 请求解码

```java
public class ExchangeCodec extends TelnetCodec {
    
    @Override
    public Object decode(Channel channel, ChannelBuffer buffer) throws IOException {
        int readable = buffer.readableBytes();
        // 创建消息头字节数组
        byte[] header = new byte[Math.min(readable, HEADER_LENGTH)];
        // 读取消息头数据
        buffer.readBytes(header);
        // 调用重载方法进行后续解码工作
        return decode(channel, buffer, readable, header);
    }

    @Override
    protected Object decode(Channel channel, ChannelBuffer buffer, int readable, byte[] header) throws IOException {
        // 检查魔数是否相等
        if (readable > 0 && header[0] != MAGIC_HIGH
                || readable > 1 && header[1] != MAGIC_LOW) {
            int length = header.length;
            if (header.length < readable) {
                header = Bytes.copyOf(header, readable);
                buffer.readBytes(header, length, readable - length);
            }
            for (int i = 1; i < header.length - 1; i++) {
                if (header[i] == MAGIC_HIGH && header[i + 1] == MAGIC_LOW) {
                    buffer.readerIndex(buffer.readerIndex() - header.length + i);
                    header = Bytes.copyOf(header, i);
                    break;
                }
            }
            // 通过 telnet 命令行发送的数据包不包含消息头，所以这里
            // 调用 TelnetCodec 的 decode 方法对数据包进行解码
            return super.decode(channel, buffer, readable, header);
        }
        
        // 检测可读数据量是否少于消息头长度，若小于则立即返回 DecodeResult.NEED_MORE_INPUT
        if (readable < HEADER_LENGTH) {
            return DecodeResult.NEED_MORE_INPUT;
        }

        // 从消息头中获取消息体长度
        int len = Bytes.bytes2int(header, 12);
        // 检测消息体长度是否超出限制，超出则抛出异常
        checkPayload(channel, len);

        int tt = len + HEADER_LENGTH;
        // 检测可读的字节数是否小于实际的字节数
        if (readable < tt) {
            return DecodeResult.NEED_MORE_INPUT;
        }
        
        ChannelBufferInputStream is = new ChannelBufferInputStream(buffer, len);

        try {
            // 继续进行解码工作
            return decodeBody(channel, is, header);
        } finally {
            if (is.available() > 0) {
                try {
                    StreamUtils.skipUnusedStream(is);
                } catch (IOException e) {
                    logger.warn(e.getMessage(), e);
                }
            }
        }
    }
}
```

​	上面方法通过检测消息头中的魔数是否与规定的魔数相等，提前拦截掉非常规数据包，比如通过 telnet 命令行发出的数据包。接着再对消息体长度，以及可读字节数进行检测。最后调用 decodeBody 方法进行后续的解码工作，ExchangeCodec 中实现了 decodeBody 方法，但因其子类 DubboCodec 覆写了该方法，所以在运行时 DubboCodec 中的 decodeBody 方法会被调用。下面我们来看一下该方法的代码。

```java
public class DubboCodec extends ExchangeCodec implements Codec2 {

    @Override
    protected Object decodeBody(Channel channel, InputStream is, byte[] header) throws IOException {
        // 获取消息头中的第三个字节，并通过逻辑与运算得到序列化器编号
        byte flag = header[2], proto = (byte) (flag & SERIALIZATION_MASK);
        Serialization s = CodecSupport.getSerialization(channel.getUrl(), proto);
        // 获取调用编号
        long id = Bytes.bytes2long(header, 4);
        // 通过逻辑与运算得到调用类型，0 - Response，1 - Request
        if ((flag & FLAG_REQUEST) == 0) {
            // 对响应结果进行解码，得到 Response 对象。这个非本节内容，后面再分析
            // ...
        } else {
            // 创建 Request 对象
            Request req = new Request(id);
            req.setVersion(Version.getProtocolVersion());
            // 通过逻辑与运算得到通信方式，并设置到 Request 对象中
            req.setTwoWay((flag & FLAG_TWOWAY) != 0);
            
            // 通过位运算检测数据包是否为事件类型
            if ((flag & FLAG_EVENT) != 0) {
                // 设置心跳事件到 Request 对象中
                req.setEvent(Request.HEARTBEAT_EVENT);
            }
            try {
                Object data;
                if (req.isHeartbeat()) {
                    // 对心跳包进行解码，该方法已被标注为废弃
                    data = decodeHeartbeatData(channel, deserialize(s, channel.getUrl(), is));
                } else if (req.isEvent()) {
                    // 对事件数据进行解码
                    data = decodeEventData(channel, deserialize(s, channel.getUrl(), is));
                } else {
                    DecodeableRpcInvocation inv;
                    // 根据 url 参数判断是否在 IO 线程上对消息体进行解码
                    if (channel.getUrl().getParameter(
                            Constants.DECODE_IN_IO_THREAD_KEY,
                            Constants.DEFAULT_DECODE_IN_IO_THREAD)) {
                        inv = new DecodeableRpcInvocation(channel, req, is, proto);
                        // 在当前线程，也就是 IO 线程上进行后续的解码工作。此工作完成后，可将
                        // 调用方法名、attachment、以及调用参数解析出来
                        inv.decode();
                    } else {
                        // 仅创建 DecodeableRpcInvocation 对象，但不在当前线程上执行解码逻辑
                        inv = new DecodeableRpcInvocation(channel, req,
                                new UnsafeByteArrayInputStream(readMessageData(is)), proto);
                    }
                    data = inv;
                }
                
                // 设置 data 到 Request 对象中
                req.setData(data);
            } catch (Throwable t) {
                // 若解码过程中出现异常，则将 broken 字段设为 true，
                // 并将异常对象设置到 Reqeust 对象中
                req.setBroken(true);
                req.setData(t);
            }
            return req;
        }
    }
}
```

​	如上，decodeBody 对部分字段进行了解码，并将解码得到的字段封装到 Request 中。随后会调用 DecodeableRpcInvocation 的 decode 方法进行后续的解码工作。此工作完成后，可将调用方法名、attachment、以及调用参数解析出来。下面我们来看一下 DecodeableRpcInvocation 的 decode 方法逻辑。

```java
public class DecodeableRpcInvocation extends RpcInvocation implements Codec, Decodeable {
    
	@Override
    public Object decode(Channel channel, InputStream input) throws IOException {
        ObjectInput in = CodecSupport.getSerialization(channel.getUrl(), serializationType)
                .deserialize(channel.getUrl(), input);

        // 通过反序列化得到 dubbo version，并保存到 attachments 变量中
        String dubboVersion = in.readUTF();
        request.setVersion(dubboVersion);
        setAttachment(Constants.DUBBO_VERSION_KEY, dubboVersion);

        // 通过反序列化得到 path，version，并保存到 attachments 变量中
        setAttachment(Constants.PATH_KEY, in.readUTF());
        setAttachment(Constants.VERSION_KEY, in.readUTF());

        // 通过反序列化得到调用方法名
        setMethodName(in.readUTF());
        try {
            Object[] args;
            Class<?>[] pts;
            // 通过反序列化得到参数类型字符串，比如 Ljava/lang/String;
            String desc = in.readUTF();
            if (desc.length() == 0) {
                pts = DubboCodec.EMPTY_CLASS_ARRAY;
                args = DubboCodec.EMPTY_OBJECT_ARRAY;
            } else {
                // 将 desc 解析为参数类型数组
                pts = ReflectUtils.desc2classArray(desc);
                args = new Object[pts.length];
                for (int i = 0; i < args.length; i++) {
                    try {
                        // 解析运行时参数
                        args[i] = in.readObject(pts[i]);
                    } catch (Exception e) {
                        if (log.isWarnEnabled()) {
                            log.warn("Decode argument failed: " + e.getMessage(), e);
                        }
                    }
                }
            }
            
            // 设置参数类型数组
            setParameterTypes(pts);

            // 通过反序列化得到原 attachment 的内容
            Map<String, String> map = (Map<String, String>) in.readObject(Map.class);
            if (map != null && map.size() > 0) {
                Map<String, String> attachment = getAttachments();
                if (attachment == null) {
                    attachment = new HashMap<String, String>();
                }
                // 将 map 与当前对象中的 attachment 集合进行融合
                attachment.putAll(map);
                setAttachments(attachment);
            }
            
            // 对 callback 类型的参数进行处理
            for (int i = 0; i < args.length; i++) {
                args[i] = decodeInvocationArgument(channel, this, pts, i, args[i]);
            }

            // 设置参数列表
            setArguments(args);

        } catch (ClassNotFoundException e) {
            throw new IOException(StringUtils.toString("Read invocation data failed.", e));
        } finally {
            if (in instanceof Cleanable) {
                ((Cleanable) in).cleanup();
            }
        }
        return this;
    }
}
```

上面的方法通过反序列化将诸如 path、version、调用方法名、参数列表等信息依次解析出来，并设置到相应的字段中，最终得到一个具有完整调用信息的 DecodeableRpcInvocation 对象。

到这里，请求数据解码的过程就分析完了。此时我们得到了一个 Request 对象，这个对象会被传送到下一个入站处理器中，我们继续往下看。

#### 调用服务

解码器将数据包解析成 Request 对象后，NettyHandler 的 messageReceived 方法紧接着会收到这个对象，并将这个对象继续向下传递。这期间该对象会被依次传递给 NettyServer、MultiMessageHandler、HeartbeatHandler 以及 AllChannelHandler。最后由 AllChannelHandler 将该对象封装到 Runnable 实现类对象中，并将 Runnable 放入线程池中执行后续的调用逻辑。整个调用栈如下：

```fallback
NettyHandler#messageReceived(ChannelHandlerContext, MessageEvent)
  —> AbstractPeer#received(Channel, Object)
    —> MultiMessageHandler#received(Channel, Object)
      —> HeartbeatHandler#received(Channel, Object)
        —> AllChannelHandler#received(Channel, Object)
          —> ExecutorService#execute(Runnable)    // 由线程池执行后续的调用逻辑
```

考虑到篇幅，以及很多中间调用的逻辑并非十分重要，所以这里就不对调用栈中的每个方法都进行分析了。这里我们直接分析调用栈中的分析第一个和最后一个调用方法逻辑。如下：

```java
@Sharable
public class NettyHandler extends SimpleChannelHandler {
    
    private final Map<String, Channel> channels = new ConcurrentHashMap<String, Channel>();

    private final URL url;

    private final ChannelHandler handler;
    
    public NettyHandler(URL url, ChannelHandler handler) {
        if (url == null) {
            throw new IllegalArgumentException("url == null");
        }
        if (handler == null) {
            throw new IllegalArgumentException("handler == null");
        }
        this.url = url;
        
        // 这里的 handler 类型为 NettyServer
        this.handler = handler;
    }
    
	public void messageReceived(ChannelHandlerContext ctx, MessageEvent e) throws Exception {
        // 获取 NettyChannel
        NettyChannel channel = NettyChannel.getOrAddChannel(ctx.getChannel(), url, handler);
        try {
            // 继续向下调用
            handler.received(channel, e.getMessage());
        } finally {
            NettyChannel.removeChannelIfDisconnected(ctx.getChannel());
        }
    }
}
```

如上，NettyHandler 中的 messageReceived 逻辑比较简单。首先根据一些信息获取 NettyChannel 实例，然后将 NettyChannel 实例以及 Request 对象向下传递。下面再来看看 AllChannelHandler 的逻辑，在详细分析代码之前，我们先来了解一下 Dubbo 中的线程派发模型。

#####  线程派发模型

Dubbo 将底层通信框架中接收请求的线程称为 IO 线程。如果一些事件处理逻辑可以很快执行完，比如只在内存打一个标记，此时直接在 IO 线程上执行该段逻辑即可。但如果事件的处理逻辑比较耗时，比如该段逻辑会发起数据库查询或者 HTTP 请求。此时我们就不应该让事件处理逻辑在 IO 线程上执行，而是应该派发到线程池中去执行。原因也很简单，IO 线程主要用于接收请求，如果 IO 线程被占满，将导致它不能接收新的请求。

以上就是线程派发的背景，下面我们再来通过 Dubbo 调用图，看一下线程派发器所处的位置。

![](https://pic.imgdb.cn/item/612f479c44eaada739c4e6ea.jpg)

​	如上图，红框中的 Dispatcher 就是线程派发器。需要说明的是，Dispatcher 真实的职责创建具有线程派发能力的 ChannelHandler，比如 AllChannelHandler、MessageOnlyChannelHandler 和 ExecutionChannelHandler 等，其本身并不具备线程派发能力。Dubbo 支持 5 种不同的线程派发策略，下面通过一个表格列举一下。

| 策略       | 用途                                                         |
| ---------- | ------------------------------------------------------------ |
| all        | 所有消息都派发到线程池，包括请求，响应，连接事件，断开事件等 |
| direct     | 所有消息都不派发到线程池，全部在 IO 线程上直接执行           |
| message    | 只有**请求**和**响应**消息派发到线程池，其它消息均在 IO 线程上执行 |
| execution  | 只有**请求**消息派发到线程池，不含响应。其它消息均在 IO 线程上执行 |
| connection | 在 IO 线程上，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池 |

​	默认配置下，Dubbo 使用 `all` 派发策略，即将所有的消息都派发到线程池中。下面我们来分析一下 AllChannelHandler 的代码。

```java
public class AllChannelHandler extends WrappedChannelHandler {

    public AllChannelHandler(ChannelHandler handler, URL url) {
        super(handler, url);
    }

    /** 处理连接事件 */
    @Override
    public void connected(Channel channel) throws RemotingException {
        // 获取线程池
        ExecutorService cexecutor = getExecutorService();
        try {
            // 将连接事件派发到线程池中处理
            cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CONNECTED));
        } catch (Throwable t) {
            throw new ExecutionException(..., " error when process connected event .", t);
        }
    }

    /** 处理断开事件 */
    @Override
    public void disconnected(Channel channel) throws RemotingException {
        ExecutorService cexecutor = getExecutorService();
        try {
            cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.DISCONNECTED));
        } catch (Throwable t) {
            throw new ExecutionException(..., "error when process disconnected event .", t);
        }
    }

    /** 处理请求和响应消息，这里的 message 变量类型可能是 Request，也可能是 Response */
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService cexecutor = getExecutorService();
        try {
            // 将请求和响应消息派发到线程池中处理
            cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
            if(message instanceof Request && t instanceof RejectedExecutionException){
                Request request = (Request)message;
                // 如果通信方式为双向通信，此时将 Server side ... threadpool is exhausted 
                // 错误信息封装到 Response 中，并返回给服务消费方。
                if(request.isTwoWay()){
                    String msg = "Server side(" + url.getIp() + "," + url.getPort() 
                        + ") threadpool is exhausted ,detail msg:" + t.getMessage();
                    Response response = new Response(request.getId(), request.getVersion());
                    response.setStatus(Response.SERVER_THREADPOOL_EXHAUSTED_ERROR);
                    response.setErrorMessage(msg);
                    // 返回包含错误信息的 Response 对象
                    channel.send(response);
                    return;
                }
            }
            throw new ExecutionException(..., " error when process received event .", t);
        }
    }

    /** 处理异常信息 */
    @Override
    public void caught(Channel channel, Throwable exception) throws RemotingException {
        ExecutorService cexecutor = getExecutorService();
        try {
            cexecutor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CAUGHT, exception));
        } catch (Throwable t) {
            throw new ExecutionException(..., "error when process caught event ...");
        }
    }
}
```

​	如上，请求对象会被封装 ChannelEventRunnable 中，ChannelEventRunnable 将会是服务调用过程的新起点。所以接下来我们以 ChannelEventRunnable 为起点向下探索。

##### 调用服务

​	本小节，我们从 ChannelEventRunnable 开始分析，该类的主要代码如下：

```java
public class ChannelEventRunnable implements Runnable {
    
    private final ChannelHandler handler;
    private final Channel channel;
    private final ChannelState state;
    private final Throwable exception;
    private final Object message;
    
    @Override
    public void run() {
        // 检测通道状态，对于请求或响应消息，此时 state = RECEIVED
        if (state == ChannelState.RECEIVED) {
            try {
                // 将 channel 和 message 传给 ChannelHandler 对象，进行后续的调用
                handler.received(channel, message);
            } catch (Exception e) {
                logger.warn("... operation error, channel is ... message is ...");
            }
        } 
        
        // 其他消息类型通过 switch 进行处理
        else {
            switch (state) {
            case CONNECTED:
                try {
                    handler.connected(channel);
                } catch (Exception e) {
                    logger.warn("... operation error, channel is ...");
                }
                break;
            case DISCONNECTED:
                // ...
            case SENT:
                // ...
            case CAUGHT:
                // ...
            default:
                logger.warn("unknown state: " + state + ", message is " + message);
            }
        }

    }
}
```

如上，请求和响应消息出现频率明显比其他类型消息高，所以这里对该类型的消息进行了针对性判断。ChannelEventRunnable 仅是一个中转站，它的 run 方法中并不包含具体的调用逻辑，仅用于将参数传给其他 ChannelHandler 对象进行处理，该对象类型为 DecodeHandler。

```java
public class DecodeHandler extends AbstractChannelHandlerDelegate {

    public DecodeHandler(ChannelHandler handler) {
        super(handler);
    }

    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        if (message instanceof Decodeable) {
            // 对 Decodeable 接口实现类对象进行解码
            decode(message);
        }

        if (message instanceof Request) {
            // 对 Request 的 data 字段进行解码
            decode(((Request) message).getData());
        }

        if (message instanceof Response) {
            // 对 Request 的 result 字段进行解码
            decode(((Response) message).getResult());
        }

        // 执行后续逻辑
        handler.received(channel, message);
    }

    private void decode(Object message) {
        // Decodeable 接口目前有两个实现类，
        // 分别为 DecodeableRpcInvocation 和 DecodeableRpcResult
        if (message != null && message instanceof Decodeable) {
            try {
                // 执行解码逻辑
                ((Decodeable) message).decode();
            } catch (Throwable e) {
                if (log.isWarnEnabled()) {
                    log.warn("Call Decodeable.decode failed: " + e.getMessage(), e);
                }
            }
        }
    }
}
```

DecodeHandler 主要是包含了一些解码逻辑。2.2.1 节分析请求解码时说过，请求解码可在 IO 线程上执行，也可在线程池中执行，这个取决于运行时配置。DecodeHandler 存在的意义就是保证请求或响应对象可在线程池中被解码。解码完毕后，完全解码后的 Request 对象会继续向后传递，下一站是 HeaderExchangeHandler。

```java
public class HeaderExchangeHandler implements ChannelHandlerDelegate {

    private final ExchangeHandler handler;

    public HeaderExchangeHandler(ExchangeHandler handler) {
        if (handler == null) {
            throw new IllegalArgumentException("handler == null");
        }
        this.handler = handler;
    }

    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        channel.setAttribute(KEY_READ_TIMESTAMP, System.currentTimeMillis());
        ExchangeChannel exchangeChannel = HeaderExchangeChannel.getOrAddChannel(channel);
        try {
            // 处理请求对象
            if (message instanceof Request) {
                Request request = (Request) message;
                if (request.isEvent()) {
                    // 处理事件
                    handlerEvent(channel, request);
                } 
                // 处理普通的请求
                else {
                    // 双向通信
                    if (request.isTwoWay()) {
                        // 向后调用服务，并得到调用结果
                        Response response = handleRequest(exchangeChannel, request);
                        // 将调用结果返回给服务消费端
                        channel.send(response);
                    } 
                    // 如果是单向通信，仅向后调用指定服务即可，无需返回调用结果
                    else {
                        handler.received(exchangeChannel, request.getData());
                    }
                }
            }      
            // 处理响应对象，服务消费方会执行此处逻辑，后面分析
            else if (message instanceof Response) {
                handleResponse(channel, (Response) message);
            } else if (message instanceof String) {
                // telnet 相关，忽略
            } else {
                handler.received(exchangeChannel, message);
            }
        } finally {
            HeaderExchangeChannel.removeChannelIfDisconnected(channel);
        }
    }

    Response handleRequest(ExchangeChannel channel, Request req) throws RemotingException {
        Response res = new Response(req.getId(), req.getVersion());
        // 检测请求是否合法，不合法则返回状态码为 BAD_REQUEST 的响应
        if (req.isBroken()) {
            Object data = req.getData();

            String msg;
            if (data == null)
                msg = null;
            else if
                (data instanceof Throwable) msg = StringUtils.toString((Throwable) data);
            else
                msg = data.toString();
            res.setErrorMessage("Fail to decode request due to: " + msg);
            // 设置 BAD_REQUEST 状态
            res.setStatus(Response.BAD_REQUEST);

            return res;
        }
        
        // 获取 data 字段值，也就是 RpcInvocation 对象
        Object msg = req.getData();
        try {
            // 继续向下调用
            Object result = handler.reply(channel, msg);
            // 设置 OK 状态码
            res.setStatus(Response.OK);
            // 设置调用结果
            res.setResult(result);
        } catch (Throwable e) {
            // 若调用过程出现异常，则设置 SERVICE_ERROR，表示服务端异常
            res.setStatus(Response.SERVICE_ERROR);
            res.setErrorMessage(StringUtils.toString(e));
        }
        return res;
    }
}
```

到这里，我们看到了比较清晰的请求和响应逻辑。对于双向通信，HeaderExchangeHandler 首先向后进行调用，得到调用结果。然后将调用结果封装到 Response 对象中，最后再将该对象返回给服务消费方。如果请求不合法，或者调用失败，则将错误信息封装到 Response 对象中，并返回给服务消费方。接下来我们继续向后分析，把剩余的调用过程分析完。下面分析定义在 DubboProtocol 类中的匿名类对象逻辑，如下：

```java
public class DubboProtocol extends AbstractProtocol {

    public static final String NAME = "dubbo";
    
    private ExchangeHandler requestHandler = new ExchangeHandlerAdapter() {

        @Override
        public Object reply(ExchangeChannel channel, Object message) throws RemotingException {
            if (message instanceof Invocation) {
                Invocation inv = (Invocation) message;
                // 获取 Invoker 实例
                Invoker<?> invoker = getInvoker(channel, inv);
                if (Boolean.TRUE.toString().equals(inv.getAttachments().get(IS_CALLBACK_SERVICE_INVOKE))) {
                    // 回调相关，忽略
                }
                RpcContext.getContext().setRemoteAddress(channel.getRemoteAddress());
                // 通过 Invoker 调用具体的服务
                return invoker.invoke(inv);
            }
            throw new RemotingException(channel, "Unsupported request: ...");
        }
        
        // 忽略其他方法
    }
    
    Invoker<?> getInvoker(Channel channel, Invocation inv) throws RemotingException {
        // 忽略回调和本地存根相关逻辑
        // ...
        
        int port = channel.getLocalAddress().getPort();
        
        // 计算 service key，格式为 groupName/serviceName:serviceVersion:port。比如：
        //   dubbo/com.alibaba.dubbo.demo.DemoService:1.0.0:20880
        String serviceKey = serviceKey(port, path, inv.getAttachments().get(Constants.VERSION_KEY), inv.getAttachments().get(Constants.GROUP_KEY));

        // 从 exporterMap 查找与 serviceKey 相对应的 DubboExporter 对象，
        // 服务导出过程中会将 <serviceKey, DubboExporter> 映射关系存储到 exporterMap 集合中
        DubboExporter<?> exporter = (DubboExporter<?>) exporterMap.get(serviceKey);

        if (exporter == null)
            throw new RemotingException(channel, "Not found exported service ...");

        // 获取 Invoker 对象，并返回
        return exporter.getInvoker();
    }
    
    // 忽略其他方法
}
```

以上逻辑用于获取与指定服务对应的 Invoker 实例，并通过 Invoker 的 invoke 方法调用服务逻辑。invoke 方法定义在 AbstractProxyInvoker 中，代码如下。

```java
public abstract class AbstractProxyInvoker<T> implements Invoker<T> {

    @Override
    public Result invoke(Invocation invocation) throws RpcException {
        try {
            // 调用 doInvoke 执行后续的调用，并将调用结果封装到 RpcResult 中，并
            return new RpcResult(doInvoke(proxy, invocation.getMethodName(), invocation.getParameterTypes(), invocation.getArguments()));
        } catch (InvocationTargetException e) {
            return new RpcResult(e.getTargetException());
        } catch (Throwable e) {
            throw new RpcException("Failed to invoke remote proxy method ...");
        }
    }
    
    protected abstract Object doInvoke(T proxy, String methodName, Class<?>[] parameterTypes, Object[] arguments) throws Throwable;
}
```

如上，doInvoke 是一个抽象方法，这个需要由具体的 Invoker 实例实现。Invoker 实例是在运行时通过 JavassistProxyFactory 创建的，创建逻辑如下：

```java
public class JavassistProxyFactory extends AbstractProxyFactory {
    
    // 省略其他方法

    @Override
    public <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
        final Wrapper wrapper = Wrapper.getWrapper(proxy.getClass().getName().indexOf('$') < 0 ? proxy.getClass() : type);
        // 创建匿名类对象
        return new AbstractProxyInvoker<T>(proxy, type, url) {
            @Override
            protected Object doInvoke(T proxy, String methodName,
                                      Class<?>[] parameterTypes,
                                      Object[] arguments) throws Throwable {
                // 调用 invokeMethod 方法进行后续的调用
                return wrapper.invokeMethod(proxy, methodName, parameterTypes, arguments);
            }
        };
    }
}
```

Wrapper 是一个抽象类，其中 invokeMethod 是一个抽象方法。Dubbo 会在运行时通过 Javassist 框架为 Wrapper 生成实现类，并实现 invokeMethod 方法，该方法最终会根据调用信息调用具体的服务。以 DemoServiceImpl 为例，Javassist 为其生成的代理类如下。

```java
/** Wrapper0 是在运行时生成的，大家可使用 Arthas 进行反编译 */
public class Wrapper0 extends Wrapper implements ClassGenerator.DC {
    public static String[] pns;
    public static Map pts;
    public static String[] mns;
    public static String[] dmns;
    public static Class[] mts0;

    // 省略其他方法

    public Object invokeMethod(Object object, String string, Class[] arrclass, Object[] arrobject) throws InvocationTargetException {
        DemoService demoService;
        try {
            // 类型转换
            demoService = (DemoService)object;
        }
        catch (Throwable throwable) {
            throw new IllegalArgumentException(throwable);
        }
        try {
            // 根据方法名调用指定的方法
            if ("sayHello".equals(string) && arrclass.length == 1) {
                return demoService.sayHello((String)arrobject[0]);
            }
        }
        catch (Throwable throwable) {
            throw new InvocationTargetException(throwable);
        }
        throw new NoSuchMethodException(new StringBuffer().append("Not found method \"").append(string).append("\" in class com.alibaba.dubbo.demo.DemoService.").toString());
    }
}
```

到这里，整个服务调用过程就分析完了。最后把调用过程贴出来，如下：

```fallback
ChannelEventRunnable#run()
  —> DecodeHandler#received(Channel, Object)
    —> HeaderExchangeHandler#received(Channel, Object)
      —> HeaderExchangeHandler#handleRequest(ExchangeChannel, Request)
        —> DubboProtocol.requestHandler#reply(ExchangeChannel, Object)
          —> Filter#invoke(Invoker, Invocation)
            —> AbstractProxyInvoker#invoke(Invocation)
              —> Wrapper0#invokeMethod(Object, String, Class[], Object[])
                —> DemoServiceImpl#sayHello(String)
```

### 2.4 服务提供方返回调用结果

服务提供方调用指定服务后，会将调用结果封装到 Response 对象中，并将该对象返回给服务消费方。服务提供方也是通过 NettyChannel 的 send 方法将 Response 对象返回，这个方法在 2.2.1 节分析过，这里就不在重复分析了。本节我们仅需关注 Response 对象的编码过程即可，这里仍然省略一些中间调用，直接分析具体的编码逻辑。

```java
public class ExchangeCodec extends TelnetCodec {
	public void encode(Channel channel, ChannelBuffer buffer, Object msg) throws IOException {
        if (msg instanceof Request) {
            encodeRequest(channel, buffer, (Request) msg);
        } else if (msg instanceof Response) {
            // 对响应对象进行编码
            encodeResponse(channel, buffer, (Response) msg);
        } else {
            super.encode(channel, buffer, msg);
        }
    }
    
    protected void encodeResponse(Channel channel, ChannelBuffer buffer, Response res) throws IOException {
        int savedWriteIndex = buffer.writerIndex();
        try {
            Serialization serialization = getSerialization(channel);
            // 创建消息头字节数组
            byte[] header = new byte[HEADER_LENGTH];
            // 设置魔数
            Bytes.short2bytes(MAGIC, header);
            // 设置序列化器编号
            header[2] = serialization.getContentTypeId();
            if (res.isHeartbeat()) header[2] |= FLAG_EVENT;
            // 获取响应状态
            byte status = res.getStatus();
            // 设置响应状态
            header[3] = status;
            // 设置请求编号
            Bytes.long2bytes(res.getId(), header, 4);

            // 更新 writerIndex，为消息头预留 16 个字节的空间
            buffer.writerIndex(savedWriteIndex + HEADER_LENGTH);
            ChannelBufferOutputStream bos = new ChannelBufferOutputStream(buffer);
            ObjectOutput out = serialization.serialize(channel.getUrl(), bos);
           
            if (status == Response.OK) {
                if (res.isHeartbeat()) {
                    // 对心跳响应结果进行序列化，已废弃
                    encodeHeartbeatData(channel, out, res.getResult());
                } else {
                    // 对调用结果进行序列化
                    encodeResponseData(channel, out, res.getResult(), res.getVersion());
                }
            } else { 
                // 对错误信息进行序列化
                out.writeUTF(res.getErrorMessage())
            };
            out.flushBuffer();
            if (out instanceof Cleanable) {
                ((Cleanable) out).cleanup();
            }
            bos.flush();
            bos.close();

            // 获取写入的字节数，也就是消息体长度
            int len = bos.writtenBytes();
            checkPayload(channel, len);
            
            // 将消息体长度写入到消息头中
            Bytes.int2bytes(len, header, 12);
            // 将 buffer 指针移动到 savedWriteIndex，为写消息头做准备
            buffer.writerIndex(savedWriteIndex);
            // 从 savedWriteIndex 下标处写入消息头
            buffer.writeBytes(header); 
            // 设置新的 writerIndex，writerIndex = 原写下标 + 消息头长度 + 消息体长度
            buffer.writerIndex(savedWriteIndex + HEADER_LENGTH + len);
        } catch (Throwable t) {
            // 异常处理逻辑不是很难理解，但是代码略多，这里忽略了
        }
    }
}

public class DubboCodec extends ExchangeCodec implements Codec2 {
    
	protected void encodeResponseData(Channel channel, ObjectOutput out, Object data, String version) throws IOException {
        Result result = (Result) data;
        // 检测当前协议版本是否支持带有 attachment 集合的 Response 对象
        boolean attach = Version.isSupportResponseAttachment(version);
        Throwable th = result.getException();
        
        // 异常信息为空
        if (th == null) {
            Object ret = result.getValue();
            // 调用结果为空
            if (ret == null) {
                // 序列化响应类型
                out.writeByte(attach ? RESPONSE_NULL_VALUE_WITH_ATTACHMENTS : RESPONSE_NULL_VALUE);
            } 
            // 调用结果非空
            else {
                // 序列化响应类型
                out.writeByte(attach ? RESPONSE_VALUE_WITH_ATTACHMENTS : RESPONSE_VALUE);
                // 序列化调用结果
                out.writeObject(ret);
            }
        } 
        // 异常信息非空
        else {
            // 序列化响应类型
            out.writeByte(attach ? RESPONSE_WITH_EXCEPTION_WITH_ATTACHMENTS : RESPONSE_WITH_EXCEPTION);
            // 序列化异常对象
            out.writeObject(th);
        }

        if (attach) {
            // 记录 Dubbo 协议版本
            result.getAttachments().put(Constants.DUBBO_VERSION_KEY, Version.getProtocolVersion());
            // 序列化 attachments 集合
            out.writeObject(result.getAttachments());
        }
    }
}
```

以上就是 Response 对象编码的过程，和前面分析的 Request 对象编码过程很相似。如果大家能看 Request 对象的编码逻辑，那么这里的 Response 对象的编码逻辑也不难理解，就不多说了。接下来我们再来分析双向通信的最后一环 —— 服务消费方接收调用结果。

### 2.5 服务消费方接收调用结果

服务消费方在收到响应数据后，首先要做的事情是对响应数据进行解码，得到 Response 对象。然后再将该对象传递给下一个入站处理器，这个入站处理器就是 NettyHandler。接下来 NettyHandler 会将这个对象继续向下传递，最后 AllChannelHandler 的 received 方法会收到这个对象，并将这个对象派发到线程池中。这个过程和服务提供方接收请求的过程是一样的，因此这里就不重复分析了。本节我们重点分析两个方面的内容，一是响应数据的解码过程，二是 Dubbo 如何将调用结果传递给用户线程的。下面先来分析响应数据的解码过程。

#### 2.5.1 响应数据解码

响应数据解码逻辑主要的逻辑封装在 DubboCodec 中，我们直接分析这个类的代码。如下：

```java
public class DubboCodec extends ExchangeCodec implements Codec2 {

    @Override
    protected Object decodeBody(Channel channel, InputStream is, byte[] header) throws IOException {
        byte flag = header[2], proto = (byte) (flag & SERIALIZATION_MASK);
        Serialization s = CodecSupport.getSerialization(channel.getUrl(), proto);
        // 获取请求编号
        long id = Bytes.bytes2long(header, 4);
        // 检测消息类型，若下面的条件成立，表明消息类型为 Response
        if ((flag & FLAG_REQUEST) == 0) {
            // 创建 Response 对象
            Response res = new Response(id);
            // 检测事件标志位
            if ((flag & FLAG_EVENT) != 0) {
                // 设置心跳事件
                res.setEvent(Response.HEARTBEAT_EVENT);
            }
            // 获取响应状态
            byte status = header[3];
            // 设置响应状态
            res.setStatus(status);
            
            // 如果响应状态为 OK，表明调用过程正常
            if (status == Response.OK) {
                try {
                    Object data;
                    if (res.isHeartbeat()) {
                        // 反序列化心跳数据，已废弃
                        data = decodeHeartbeatData(channel, deserialize(s, channel.getUrl(), is));
                    } else if (res.isEvent()) {
                        // 反序列化事件数据
                        data = decodeEventData(channel, deserialize(s, channel.getUrl(), is));
                    } else {
                        DecodeableRpcResult result;
                        // 根据 url 参数决定是否在 IO 线程上执行解码逻辑
                        if (channel.getUrl().getParameter(
                                Constants.DECODE_IN_IO_THREAD_KEY,
                                Constants.DEFAULT_DECODE_IN_IO_THREAD)) {
                            // 创建 DecodeableRpcResult 对象
                            result = new DecodeableRpcResult(channel, res, is,
                                    (Invocation) getRequestData(id), proto);
                            // 进行后续的解码工作
                            result.decode();
                        } else {
                            // 创建 DecodeableRpcResult 对象
                            result = new DecodeableRpcResult(channel, res,
                                    new UnsafeByteArrayInputStream(readMessageData(is)),
                                    (Invocation) getRequestData(id), proto);
                        }
                        data = result;
                    }
                    
                    // 设置 DecodeableRpcResult 对象到 Response 对象中
                    res.setResult(data);
                } catch (Throwable t) {
                    // 解码过程中出现了错误，此时设置 CLIENT_ERROR 状态码到 Response 对象中
                    res.setStatus(Response.CLIENT_ERROR);
                    res.setErrorMessage(StringUtils.toString(t));
                }
            } 
            // 响应状态非 OK，表明调用过程出现了异常
            else {
                // 反序列化异常信息，并设置到 Response 对象中
                res.setErrorMessage(deserialize(s, channel.getUrl(), is).readUTF());
            }
            return res;
        } else {
            // 对请求数据进行解码，前面已分析过，此处忽略
        }
    }
}
```

以上就是响应数据的解码过程，上面逻辑看起来是不是似曾相识。对的，我们在前面章节分析过 DubboCodec 的 decodeBody 方法中关于请求数据的解码过程，该过程和响应数据的解码过程很相似。下面，我们继续分析调用结果的反序列化过程，如下：

```java
public class DecodeableRpcResult extends RpcResult implements Codec, Decodeable {
    
    private Invocation invocation;
	
    @Override
    public void decode() throws Exception {
        if (!hasDecoded && channel != null && inputStream != null) {
            try {
                // 执行反序列化操作
                decode(channel, inputStream);
            } catch (Throwable e) {
                // 反序列化失败，设置 CLIENT_ERROR 状态到 Response 对象中
                response.setStatus(Response.CLIENT_ERROR);
                // 设置异常信息
                response.setErrorMessage(StringUtils.toString(e));
            } finally {
                hasDecoded = true;
            }
        }
    }
    
    @Override
    public Object decode(Channel channel, InputStream input) throws IOException {
        ObjectInput in = CodecSupport.getSerialization(channel.getUrl(), serializationType)
                .deserialize(channel.getUrl(), input);
        
        // 反序列化响应类型
        byte flag = in.readByte();
        switch (flag) {
            case DubboCodec.RESPONSE_NULL_VALUE:
                break;
            case DubboCodec.RESPONSE_VALUE:
                // ...
                break;
            case DubboCodec.RESPONSE_WITH_EXCEPTION:
                // ...
                break;
                
            // 返回值为空，且携带了 attachments 集合
            case DubboCodec.RESPONSE_NULL_VALUE_WITH_ATTACHMENTS:
                try {
                    // 反序列化 attachments 集合，并存储起来 
                    setAttachments((Map<String, String>) in.readObject(Map.class));
                } catch (ClassNotFoundException e) {
                    throw new IOException(StringUtils.toString("Read response data failed.", e));
                }
                break;
                
            // 返回值不为空，且携带了 attachments 集合
            case DubboCodec.RESPONSE_VALUE_WITH_ATTACHMENTS:
                try {
                    // 获取返回值类型
                    Type[] returnType = RpcUtils.getReturnTypes(invocation);
                    // 反序列化调用结果，并保存起来
                    setValue(returnType == null || returnType.length == 0 ? in.readObject() :
                            (returnType.length == 1 ? in.readObject((Class<?>) returnType[0])
                                    : in.readObject((Class<?>) returnType[0], returnType[1])));
                    // 反序列化 attachments 集合，并存储起来
                    setAttachments((Map<String, String>) in.readObject(Map.class));
                } catch (ClassNotFoundException e) {
                    throw new IOException(StringUtils.toString("Read response data failed.", e));
                }
                break;
                
            // 异常对象不为空，且携带了 attachments 集合
            case DubboCodec.RESPONSE_WITH_EXCEPTION_WITH_ATTACHMENTS:
                try {
                    // 反序列化异常对象
                    Object obj = in.readObject();
                    if (obj instanceof Throwable == false)
                        throw new IOException("Response data error, expect Throwable, but get " + obj);
                    // 设置异常对象
                    setException((Throwable) obj);
                    // 反序列化 attachments 集合，并存储起来
                    setAttachments((Map<String, String>) in.readObject(Map.class));
                } catch (ClassNotFoundException e) {
                    throw new IOException(StringUtils.toString("Read response data failed.", e));
                }
                break;
            default:
                throw new IOException("Unknown result flag, expect '0' '1' '2', get " + flag);
        }
        if (in instanceof Cleanable) {
            ((Cleanable) in).cleanup();
        }
        return this;
    }
}
```

本篇文章所分析的源码版本为 2.6.4，该版本下的 Response 支持 attachments 集合，所以上面仅对部分 case 分支进行了注释。其他 case 分支的逻辑比被注释分支的逻辑更为简单，这里就忽略了。我们所使用的测试服务接口 DemoService 包含了一个具有返回值的方法，正常调用下，线程会进入 RESPONSE_VALUE_WITH_ATTACHMENTS 分支中。然后线程会从 invocation 变量（大家探索一下 invocation 变量的由来）中获取返回值类型，接着对调用结果进行反序列化，并将序列化后的结果存储起来。最后对 attachments 集合进行反序列化，并存到指定字段中。到此，关于响应数据的解码过程就分析完了。接下来，我们再来探索一下响应对象 Response 的去向。

#### 2.5.2 向用户线程传递调用结果

响应数据解码完成后，Dubbo 会将响应对象派发到线程池上。要注意的是，线程池中的线程并非用户的调用线程，所以要想办法将响应对象从线程池线程传递到用户线程上。我们在 2.1 节分析过用户线程在发送完请求后的动作，即调用 DefaultFuture 的 get 方法等待响应对象的到来。当响应对象到来后，用户线程会被唤醒，并通过**调用编号**获取属于自己的响应对象。下面我们来看一下整个过程对应的代码。

```java
public class HeaderExchangeHandler implements ChannelHandlerDelegate {
    
    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        channel.setAttribute(KEY_READ_TIMESTAMP, System.currentTimeMillis());
        ExchangeChannel exchangeChannel = HeaderExchangeChannel.getOrAddChannel(channel);
        try {
            if (message instanceof Request) {
                // 处理请求，前面已分析过，省略
            } else if (message instanceof Response) {
                // 处理响应
                handleResponse(channel, (Response) message);
            } else if (message instanceof String) {
                // telnet 相关，忽略
            } else {
                handler.received(exchangeChannel, message);
            }
        } finally {
            HeaderExchangeChannel.removeChannelIfDisconnected(channel);
        }
    }

    static void handleResponse(Channel channel, Response response) throws RemotingException {
        if (response != null && !response.isHeartbeat()) {
            // 继续向下调用
            DefaultFuture.received(channel, response);
        }
    }
}

public class DefaultFuture implements ResponseFuture {  
    
    private final Lock lock = new ReentrantLock();
    private final Condition done = lock.newCondition();
    private volatile Response response;
    
	public static void received(Channel channel, Response response) {
        try {
            // 根据调用编号从 FUTURES 集合中查找指定的 DefaultFuture 对象
            DefaultFuture future = FUTURES.remove(response.getId());
            if (future != null) {
                // 继续向下调用
                future.doReceived(response);
            } else {
                logger.warn("The timeout response finally returned at ...");
            }
        } finally {
            CHANNELS.remove(response.getId());
        }
    }

	private void doReceived(Response res) {
        lock.lock();
        try {
            // 保存响应对象
            response = res;
            if (done != null) {
                // 唤醒用户线程
                done.signal();
            }
        } finally {
            lock.unlock();
        }
        if (callback != null) {
            invokeCallback(callback);
        }
    }
}
```

以上逻辑是将响应对象保存到相应的 DefaultFuture 实例中，然后再唤醒用户线程，随后用户线程即可从 DefaultFuture 实例中获取到相应结果。

本篇文章在多个地方都强调过调用编号很重要，但一直没有解释原因，这里简单说明一下。一般情况下，服务消费方会并发调用多个服务，每个用户线程发送请求后，会调用不同 DefaultFuture 对象的 get 方法进行等待。 一段时间后，服务消费方的线程池会收到多个响应对象。这个时候要考虑一个问题，如何将每个响应对象传递给相应的 DefaultFuture 对象，且不出错。答案是通过调用编号。DefaultFuture 被创建时，会要求传入一个 Request 对象。此时 DefaultFuture 可从 Request 对象中获取调用编号，并将 <调用编号, DefaultFuture 对象> 映射关系存入到静态 Map 中，即 FUTURES。线程池中的线程在收到 Response 对象后，会根据 Response 对象中的调用编号到 FUTURES 集合中取出相应的 DefaultFuture 对象，然后再将 Response 对象设置到 DefaultFuture 对象中。最后再唤醒用户线程，这样用户线程即可从 DefaultFuture 对象中获取调用结果了。整个过程大致如下图：

![](https://pic.imgdb.cn/item/612f481544eaada739c5934a.jpg)

## 总结

​	本篇文章主要对 Dubbo 中的几种服务调用方式，以及从双向通信的角度对整个通信过程进行了详细的分析。按照通信顺序，通信过程包括服务消费方发送请求，服务提供方接收请求，服务提供方返回响应数据，服务消费方接收响应数据等过程。理解这些过程需要大家对网络编程，尤其是 Netty 有一定的了解。限于篇幅原因，本篇文章无法将服务调用的所有内容都一一进行分析。对于本篇文章未讲到或未详细分析的内容，比如服务降级、过滤器链、以及序列化等。

# 服务目录

## 简介

本篇文章，将开始分析 Dubbo 集群容错方面的源码。集群容错源码包含四个部分，分别是服务目录 Directory、服务路由 Router、集群 Cluster 和负载均衡 LoadBalance。这几个部分的源码逻辑相对比较独立，我们将会分四篇文章进行分析。本篇文章作为集群容错的开篇文章，将和大家一起分析服务目录相关的源码。在进行深入分析之前，我们先来了解一下服务目录是什么。服务目录中存储了一些和服务提供者有关的信息，通过服务目录，服务消费者可获取到服务提供者的信息，比如 ip、端口、服务协议等。通过这些信息，服务消费者就可通过 Netty 等客户端进行远程调用。在一个服务集群中，服务提供者数量并不是一成不变的，如果集群中新增了一台机器，相应地在服务目录中就要新增一条服务提供者记录。或者，如果服务提供者的配置修改了，服务目录中的记录也要做相应的更新。如果这样说，服务目录和注册中心的功能不就雷同了吗？确实如此，这里这么说是为了方便大家理解。实际上服务目录在获取注册中心的服务配置信息后，会为每条配置信息生成一个 Invoker 对象，并把这个 Invoker 对象存储起来，这个 Invoker 才是服务目录最终持有的对象。Invoker 有什么用呢？看名字就知道了，这是一个具有远程调用功能的对象。讲到这大家应该知道了什么是服务目录了，它可以看做是 Invoker 集合，且这个集合中的元素会随注册中心的变化而进行动态调整。

关于服务目录这里就先介绍这些，大家先有个大致印象。接下来我们通过继承体系图来了解一下服务目录的家族成员都有哪些。

## 继承体系

服务目录目前内置的实现有两个，分别为 StaticDirectory 和 RegistryDirectory，它们均是 AbstractDirectory 的子类。AbstractDirectory 实现了 Directory 接口，这个接口包含了一个重要的方法定义，即 list(Invocation)，用于列举 Invoker。下面我们来看一下他们的继承体系图。

![img](http://dubbo.apache.org/imgs/dev/directory-inherit-hierarchy.png)

如上，Directory 继承自 Node 接口，Node 这个接口继承者比较多，像 Registry、Monitor、Invoker 等均继承了这个接口。这个接口包含了一个获取配置信息的方法 getUrl，实现该接口的类可以向外提供配置信息。另外，大家注意看 RegistryDirectory 实现了 NotifyListener 接口，当注册中心节点信息发生变化后，RegistryDirectory 可以通过此接口方法得到变更信息，并根据变更信息动态调整内部 Invoker 列表。

## 源码分析 2.7

本章将分析 AbstractDirectory 和它两个子类的源码。AbstractDirectory 封装了 Invoker 列举流程，具体的列举逻辑则由子类实现，这是典型的模板模式。所以，接下来我们先来看一下 AbstractDirectory 的源码。

```java
public List<Invoker<T>> list(Invocation invocation) throws RpcException {
    if (destroyed) {
        throw new RpcException("Directory already destroyed...");
    }
    
    // 调用 doList 方法列举 Invoker，doList 是模板方法，由子类实现
    List<Invoker<T>> invokers = doList(invocation);
    
    // 获取路由 Router 列表
    List<Router> localRouters = this.routers;
    if (localRouters != null && !localRouters.isEmpty()) {
        for (Router router : localRouters) {
            try {
                // 获取 runtime 参数，并根据参数决定是否进行路由
                if (router.getUrl() == null || router.getUrl().getParameter(Constants.RUNTIME_KEY, false)) {
                    // 进行服务路由
                    invokers = router.route(invokers, getConsumerUrl(), invocation);
                }
            } catch (Throwable t) {
                logger.error("Failed to execute router: ...");
            }
        }
    }
    return invokers;
}

// 模板方法，由子类实现
protected abstract List<Invoker<T>> doList(Invocation invocation) throws RpcException;
```

上面就是 AbstractDirectory 的 list 方法源码，这个方法封装了 Invoker 的列举过程。如下：

1. 调用 doList 获取 Invoker 列表
2. 根据 Router 的 getUrl 返回值为空与否，以及 runtime 参数决定是否进行服务路由

以上步骤中，doList 是模板方法，需由子类实现。Router 的 runtime 参数这里简单说明一下，这个参数决定了是否在每次调用服务时都执行路由规则。如果 runtime 为 true，那么每次调用服务前，都需要进行服务路由。这个对性能造成影响，配置时需要注意。

关于 AbstractDirectory 就分析这么多，下面开始分析子类的源码。

### StaticDirectory

StaticDirectory 即静态服务目录，顾名思义，它内部存放的 Invoker 是不会变动的。所以，理论上它和不可变 List 的功能很相似。下面我们来看一下这个类的实现。

```java
public class StaticDirectory<T> extends AbstractDirectory<T> {

    // Invoker 列表
    private final List<Invoker<T>> invokers;
    
    // 省略构造方法

    @Override
    public Class<T> getInterface() {
        // 获取接口类
        return invokers.get(0).getInterface();
    }
    
    // 检测服务目录是否可用
    @Override
    public boolean isAvailable() {
        if (isDestroyed()) {
            return false;
        }
        for (Invoker<T> invoker : invokers) {
            if (invoker.isAvailable()) {
                // 只要有一个 Invoker 是可用的，就认为当前目录是可用的
                return true;
            }
        }
        return false;
    }

    @Override
    public void destroy() {
        if (isDestroyed()) {
            return;
        }
        // 调用父类销毁逻辑
        super.destroy();
        // 遍历 Invoker 列表，并执行相应的销毁逻辑
        for (Invoker<T> invoker : invokers) {
            invoker.destroy();
        }
        invokers.clear();
    }

    @Override
    protected List<Invoker<T>> doList(Invocation invocation) throws RpcException {
        // 列举 Inovker，也就是直接返回 invokers 成员变量
        return invokers;
    }
}
```

以上就是 StaticDirectory 的代码逻辑，很简单，就不多说了。下面来看看 RegistryDirectory，这个类的逻辑比较复杂。

### RegistryDirectory

RegistryDirectory 是一种动态服务目录，实现了 NotifyListener 接口。当注册中心服务配置发生变化后，RegistryDirectory 可收到与当前服务相关的变化。收到变更通知后，RegistryDirectory 可根据配置变更信息刷新 Invoker 列表。RegistryDirectory 中有几个比较重要的逻辑，第一是 Invoker 的列举逻辑，第二是接收服务配置变更的逻辑，第三是 Invoker 列表的刷新逻辑。接下来按顺序对这三块逻辑。

#### 列举 Invoker

Invoker 列举逻辑封装在 doList 方法中，相关代码如下：

```java
public List<Invoker<T>> doList(Invocation invocation) {
    if (forbidden) {
        // 服务提供者关闭或禁用了服务，此时抛出 No provider 异常
        throw new RpcException(RpcException.FORBIDDEN_EXCEPTION,
            "No provider available from registry ...");
    }
    List<Invoker<T>> invokers = null;
    // 获取 Invoker 本地缓存
    Map<String, List<Invoker<T>>> localMethodInvokerMap = this.methodInvokerMap;
    if (localMethodInvokerMap != null && localMethodInvokerMap.size() > 0) {
        // 获取方法名和参数列表
        String methodName = RpcUtils.getMethodName(invocation);
        Object[] args = RpcUtils.getArguments(invocation);
        // 检测参数列表的第一个参数是否为 String 或 enum 类型
        if (args != null && args.length > 0 && args[0] != null
                && (args[0] instanceof String || args[0].getClass().isEnum())) {
            // 通过 方法名 + 第一个参数名称 查询 Invoker 列表，具体的使用场景暂时没想到
            invokers = localMethodInvokerMap.get(methodName + "." + args[0]);
        }
        if (invokers == null) {
            // 通过方法名获取 Invoker 列表
            invokers = localMethodInvokerMap.get(methodName);
        }
        if (invokers == null) {
            // 通过星号 * 获取 Invoker 列表
            invokers = localMethodInvokerMap.get(Constants.ANY_VALUE);
        }
        
        // 冗余逻辑，pull request #2861 移除了下面的 if 分支代码
        if (invokers == null) {
            Iterator<List<Invoker<T>>> iterator = localMethodInvokerMap.values().iterator();
            if (iterator.hasNext()) {
                invokers = iterator.next();
            }
        }
    }

	// 返回 Invoker 列表
    return invokers == null ? new ArrayList<Invoker<T>>(0) : invokers;
}
```

以上代码进行多次尝试，以期从 localMethodInvokerMap 中获取到 Invoker 列表。一般情况下，普通的调用可通过方法名获取到对应的 Invoker 列表，泛化调用可通过 ***** 获取到 Invoker 列表。localMethodInvokerMap 源自 RegistryDirectory 类的成员变量 methodInvokerMap。doList 方法可以看做是对 methodInvokerMap 变量的读操作，至于对 methodInvokerMap 变量的写操作，下一节进行分析。

#### 接收服务变更通知

RegistryDirectory 是一个动态服务目录，会随注册中心配置的变化进行动态调整。因此 RegistryDirectory 实现了 NotifyListener 接口，通过这个接口获取注册中心变更通知。下面我们来看一下具体的逻辑。

```java
public synchronized void notify(List<URL> urls) {
    // 定义三个集合，分别用于存放服务提供者 url，路由 url，配置器 url
    List<URL> invokerUrls = new ArrayList<URL>();
    List<URL> routerUrls = new ArrayList<URL>();
    List<URL> configuratorUrls = new ArrayList<URL>();
    for (URL url : urls) {
        String protocol = url.getProtocol();
        // 获取 category 参数
        String category = url.getParameter(Constants.CATEGORY_KEY, Constants.DEFAULT_CATEGORY);
        // 根据 category 参数将 url 分别放到不同的列表中
        if (Constants.ROUTERS_CATEGORY.equals(category)
                || Constants.ROUTE_PROTOCOL.equals(protocol)) {
            // 添加路由器 url
            routerUrls.add(url);
        } else if (Constants.CONFIGURATORS_CATEGORY.equals(category)
                || Constants.OVERRIDE_PROTOCOL.equals(protocol)) {
            // 添加配置器 url
            configuratorUrls.add(url);
        } else if (Constants.PROVIDERS_CATEGORY.equals(category)) {
            // 添加服务提供者 url
            invokerUrls.add(url);
        } else {
            // 忽略不支持的 category
            logger.warn("Unsupported category ...");
        }
    }
    if (configuratorUrls != null && !configuratorUrls.isEmpty()) {
        // 将 url 转成 Configurator
        this.configurators = toConfigurators(configuratorUrls);
    }
    if (routerUrls != null && !routerUrls.isEmpty()) {
        // 将 url 转成 Router
        List<Router> routers = toRouters(routerUrls);
        if (routers != null) {
            setRouters(routers);
        }
    }
    List<Configurator> localConfigurators = this.configurators;
    this.overrideDirectoryUrl = directoryUrl;
    if (localConfigurators != null && !localConfigurators.isEmpty()) {
        for (Configurator configurator : localConfigurators) {
            // 配置 overrideDirectoryUrl
            this.overrideDirectoryUrl = configurator.configure(overrideDirectoryUrl);
        }
    }

    // 刷新 Invoker 列表
    refreshInvoker(invokerUrls);
}
```

如上，notify 方法首先是根据 url 的 category 参数对 url 进行分门别类存储，然后通过 toRouters 和 toConfigurators 将 url 列表转成 Router 和 Configurator 列表。最后调用 refreshInvoker 方法刷新 Invoker 列表。这里的 toRouters 和 toConfigurators 方法逻辑不复杂，大家自行分析。接下来，我们把重点放在 refreshInvoker 方法上。

#### 刷新 Invoker 列表

refreshInvoker 方法是保证 RegistryDirectory 随注册中心变化而变化的关键所在。这一块逻辑比较多，接下来一一进行分析。

```java
private void refreshInvoker(List<URL> invokerUrls) {
    // invokerUrls 仅有一个元素，且 url 协议头为 empty，此时表示禁用所有服务
    if (invokerUrls != null && invokerUrls.size() == 1 && invokerUrls.get(0) != null
            && Constants.EMPTY_PROTOCOL.equals(invokerUrls.get(0).getProtocol())) {
        // 设置 forbidden 为 true
        this.forbidden = true;
        this.methodInvokerMap = null;
        // 销毁所有 Invoker
        destroyAllInvokers();
    } else {
        this.forbidden = false;
        Map<String, Invoker<T>> oldUrlInvokerMap = this.urlInvokerMap;
        if (invokerUrls.isEmpty() && this.cachedInvokerUrls != null) {
            // 添加缓存 url 到 invokerUrls 中
            invokerUrls.addAll(this.cachedInvokerUrls);
        } else {
            this.cachedInvokerUrls = new HashSet<URL>();
            // 缓存 invokerUrls
            this.cachedInvokerUrls.addAll(invokerUrls);
        }
        if (invokerUrls.isEmpty()) {
            return;
        }
        // 将 url 转成 Invoker
        Map<String, Invoker<T>> newUrlInvokerMap = toInvokers(invokerUrls);
        // 将 newUrlInvokerMap 转成方法名到 Invoker 列表的映射
        Map<String, List<Invoker<T>>> newMethodInvokerMap = toMethodInvokers(newUrlInvokerMap);
        // 转换出错，直接打印异常，并返回
        if (newUrlInvokerMap == null || newUrlInvokerMap.size() == 0) {
            logger.error(new IllegalStateException("urls to invokers error ..."));
            return;
        }
        // 合并多个组的 Invoker
        this.methodInvokerMap = multiGroup ? toMergeMethodInvokerMap(newMethodInvokerMap) : newMethodInvokerMap;
        this.urlInvokerMap = newUrlInvokerMap;
        try {
            // 销毁无用 Invoker
            destroyUnusedInvokers(oldUrlInvokerMap, newUrlInvokerMap);
        } catch (Exception e) {
            logger.warn("destroyUnusedInvokers error. ", e);
        }
    }
}
```

refreshInvoker 方法首先会根据入参 invokerUrls 的数量和协议头判断是否禁用所有的服务，如果禁用，则将 forbidden 设为 true，并销毁所有的 Invoker。若不禁用，则将 url 转成 Invoker，得到 <url, Invoker> 的映射关系。然后进一步进行转换，得到 <methodName, Invoker 列表> 映射关系。之后进行多组 Invoker 合并操作，并将合并结果赋值给 methodInvokerMap。methodInvokerMap 变量在 doList 方法中会被用到，doList 会对该变量进行读操作，在这里是写操作。当新的 Invoker 列表生成后，还要一个重要的工作要做，就是销毁无用的 Invoker，避免服务消费者调用已下线的服务的服务。

接下来对 refreshInvoker 方法中涉及到的调用一一进行分析。按照顺序，先来分析 url 到 Invoker 的转换过程。

```java
private Map<String, Invoker<T>> toInvokers(List<URL> urls) {
    Map<String, Invoker<T>> newUrlInvokerMap = new HashMap<String, Invoker<T>>();
    if (urls == null || urls.isEmpty()) {
        return newUrlInvokerMap;
    }
    Set<String> keys = new HashSet<String>();
    // 获取服务消费端配置的协议
    String queryProtocols = this.queryMap.get(Constants.PROTOCOL_KEY);
    for (URL providerUrl : urls) {
        if (queryProtocols != null && queryProtocols.length() > 0) {
            boolean accept = false;
            String[] acceptProtocols = queryProtocols.split(",");
            // 检测服务提供者协议是否被服务消费者所支持
            for (String acceptProtocol : acceptProtocols) {
                if (providerUrl.getProtocol().equals(acceptProtocol)) {
                    accept = true;
                    break;
                }
            }
            if (!accept) {
                // 若服务提供者协议头不被消费者所支持，则忽略当前 providerUrl
                continue;
            }
        }
        // 忽略 empty 协议
        if (Constants.EMPTY_PROTOCOL.equals(providerUrl.getProtocol())) {
            continue;
        }
        // 通过 SPI 检测服务端协议是否被消费端支持，不支持则抛出异常
        if (!ExtensionLoader.getExtensionLoader(Protocol.class).hasExtension(providerUrl.getProtocol())) {
            logger.error(new IllegalStateException("Unsupported protocol..."));
            continue;
        }
        
        // 合并 url
        URL url = mergeUrl(providerUrl);

        String key = url.toFullString();
        if (keys.contains(key)) {
            // 忽略重复 url
            continue;
        }
        keys.add(key);
        // 将本地 Invoker 缓存赋值给 localUrlInvokerMap
        Map<String, Invoker<T>> localUrlInvokerMap = this.urlInvokerMap;
        // 获取与 url 对应的 Invoker
        Invoker<T> invoker = localUrlInvokerMap == null ? null : localUrlInvokerMap.get(key);
        // 缓存未命中
        if (invoker == null) {
            try {
                boolean enabled = true;
                if (url.hasParameter(Constants.DISABLED_KEY)) {
                    // 获取 disable 配置，取反，然后赋值给 enable 变量
                    enabled = !url.getParameter(Constants.DISABLED_KEY, false);
                } else {
                    // 获取 enable 配置，并赋值给 enable 变量
                    enabled = url.getParameter(Constants.ENABLED_KEY, true);
                }
                if (enabled) {
                    // 调用 refer 获取 Invoker
                    invoker = new InvokerDelegate<T>(protocol.refer(serviceType, url), url, providerUrl);
                }
            } catch (Throwable t) {
                logger.error("Failed to refer invoker for interface...");
            }
            if (invoker != null) {
                // 缓存 Invoker 实例
                newUrlInvokerMap.put(key, invoker);
            }
            
        // 缓存命中
        } else {
            // 将 invoker 存储到 newUrlInvokerMap 中
            newUrlInvokerMap.put(key, invoker);
        }
    }
    keys.clear();
    return newUrlInvokerMap;
}
```

toInvokers 方法一开始会对服务提供者 url 进行检测，若服务消费端的配置不支持服务端的协议，或服务端 url 协议头为 empty 时，toInvokers 均会忽略服务提供方 url。必要的检测做完后，紧接着是合并 url，然后访问缓存，尝试获取与 url 对应的 invoker。如果缓存命中，直接将 Invoker 存入 newUrlInvokerMap 中即可。如果未命中，则需新建 Invoker。

toInvokers 方法返回的是 <url, Invoker> 映射关系表，接下来还要对这个结果进行进一步处理，得到方法名到 Invoker 列表的映射关系。这个过程由 toMethodInvokers 方法完成，如下：

```java
private Map<String, List<Invoker<T>>> toMethodInvokers(Map<String, Invoker<T>> invokersMap) {
    // 方法名 -> Invoker 列表
    Map<String, List<Invoker<T>>> newMethodInvokerMap = new HashMap<String, List<Invoker<T>>>();
    List<Invoker<T>> invokersList = new ArrayList<Invoker<T>>();
    if (invokersMap != null && invokersMap.size() > 0) {
        for (Invoker<T> invoker : invokersMap.values()) {
            // 获取 methods 参数
            String parameter = invoker.getUrl().getParameter(Constants.METHODS_KEY);
            if (parameter != null && parameter.length() > 0) {
                // 切分 methods 参数值，得到方法名数组
                String[] methods = Constants.COMMA_SPLIT_PATTERN.split(parameter);
                if (methods != null && methods.length > 0) {
                    for (String method : methods) {
                        // 方法名不为 *
                        if (method != null && method.length() > 0
                                && !Constants.ANY_VALUE.equals(method)) {
                            // 根据方法名获取 Invoker 列表
                            List<Invoker<T>> methodInvokers = newMethodInvokerMap.get(method);
                            if (methodInvokers == null) {
                                methodInvokers = new ArrayList<Invoker<T>>();
                                newMethodInvokerMap.put(method, methodInvokers);
                            }
                            // 存储 Invoker 到列表中
                            methodInvokers.add(invoker);
                        }
                    }
                }
            }
            invokersList.add(invoker);
        }
    }
    
    // 进行服务级别路由，参考 pull request #749
    List<Invoker<T>> newInvokersList = route(invokersList, null);
    // 存储 <*, newInvokersList> 映射关系
    newMethodInvokerMap.put(Constants.ANY_VALUE, newInvokersList);
    if (serviceMethods != null && serviceMethods.length > 0) {
        for (String method : serviceMethods) {
            List<Invoker<T>> methodInvokers = newMethodInvokerMap.get(method);
            if (methodInvokers == null || methodInvokers.isEmpty()) {
                methodInvokers = newInvokersList;
            }
            // 进行方法级别路由
            newMethodInvokerMap.put(method, route(methodInvokers, method));
        }
    }
    // 排序，转成不可变列表
    for (String method : new HashSet<String>(newMethodInvokerMap.keySet())) {
        List<Invoker<T>> methodInvokers = newMethodInvokerMap.get(method);
        Collections.sort(methodInvokers, InvokerComparator.getComparator());
        newMethodInvokerMap.put(method, Collections.unmodifiableList(methodInvokers));
    }
    return Collections.unmodifiableMap(newMethodInvokerMap);
}
```

上面方法主要做了三件事情， 第一是对入参进行遍历，然后从 Invoker 的 url 成员变量中获取 methods 参数，并切分成数组。随后以方法名为键，Invoker 列表为值，将映射关系存储到 newMethodInvokerMap 中。第二是分别基于类和方法对 Invoker 列表进行路由操作。第三是对 Invoker 列表进行排序，并转成不可变列表。关于 toMethodInvokers 方法就先分析到这，我们继续向下分析，这次要分析的多组服务的合并逻辑。

```java
private Map<String, List<Invoker<T>>> toMergeMethodInvokerMap(Map<String, List<Invoker<T>>> methodMap) {
    Map<String, List<Invoker<T>>> result = new HashMap<String, List<Invoker<T>>>();
    // 遍历入参
    for (Map.Entry<String, List<Invoker<T>>> entry : methodMap.entrySet()) {
        String method = entry.getKey();
        List<Invoker<T>> invokers = entry.getValue();
        // group -> Invoker 列表
        Map<String, List<Invoker<T>>> groupMap = new HashMap<String, List<Invoker<T>>>();
        // 遍历 Invoker 列表
        for (Invoker<T> invoker : invokers) {
            // 获取分组配置
            String group = invoker.getUrl().getParameter(Constants.GROUP_KEY, "");
            List<Invoker<T>> groupInvokers = groupMap.get(group);
            if (groupInvokers == null) {
                groupInvokers = new ArrayList<Invoker<T>>();
                // 缓存 <group, List<Invoker>> 到 groupMap 中
                groupMap.put(group, groupInvokers);
            }
            // 存储 invoker 到 groupInvokers
            groupInvokers.add(invoker);
        }
        if (groupMap.size() == 1) {
            // 如果 groupMap 中仅包含一组键值对，此时直接取出该键值对的值即可
            result.put(method, groupMap.values().iterator().next());
        
        // groupMap.size() > 1 成立，表示 groupMap 中包含多组键值对，比如：
        // {
        //     "dubbo": [invoker1, invoker2, invoker3, ...],
        //     "hello": [invoker4, invoker5, invoker6, ...]
        // }
        } else if (groupMap.size() > 1) {
            List<Invoker<T>> groupInvokers = new ArrayList<Invoker<T>>();
            for (List<Invoker<T>> groupList : groupMap.values()) {
                // 通过集群类合并每个分组对应的 Invoker 列表
                groupInvokers.add(cluster.join(new StaticDirectory<T>(groupList)));
            }
            // 缓存结果
            result.put(method, groupInvokers);
        } else {
            result.put(method, invokers);
        }
    }
    return result;
}
```

上面方法首先是生成 group 到 Invoker 列表的映射关系表，若关系表中的映射关系数量大于1，表示有多组服务。此时通过集群类合并每组 Invoker，并将合并结果存储到 groupInvokers 中。之后将方法名与 groupInvokers 存到到 result 中，并返回，整个逻辑结束。

接下来我们再来看一下 Invoker 列表刷新逻辑的最后一个动作 — 删除无用 Invoker。如下：

```java
private void destroyUnusedInvokers(Map<String, Invoker<T>> oldUrlInvokerMap, Map<String, Invoker<T>> newUrlInvokerMap) {
    if (newUrlInvokerMap == null || newUrlInvokerMap.size() == 0) {
        destroyAllInvokers();
        return;
    }
   
    List<String> deleted = null;
    if (oldUrlInvokerMap != null) {
        // 获取新生成的 Invoker 列表
        Collection<Invoker<T>> newInvokers = newUrlInvokerMap.values();
        // 遍历老的 <url, Invoker> 映射表
        for (Map.Entry<String, Invoker<T>> entry : oldUrlInvokerMap.entrySet()) {
            // 检测 newInvokers 中是否包含老的 Invoker
            if (!newInvokers.contains(entry.getValue())) {
                if (deleted == null) {
                    deleted = new ArrayList<String>();
                }
                // 若不包含，则将老的 Invoker 对应的 url 存入 deleted 列表中
                deleted.add(entry.getKey());
            }
        }
    }

    if (deleted != null) {
        // 遍历 deleted 集合，并到老的 <url, Invoker> 映射关系表查出 Invoker，销毁之
        for (String url : deleted) {
            if (url != null) {
                // 从 oldUrlInvokerMap 中移除 url 对应的 Invoker
                Invoker<T> invoker = oldUrlInvokerMap.remove(url);
                if (invoker != null) {
                    try {
                        // 销毁 Invoker
                        invoker.destroy();
                    } catch (Exception e) {
                        logger.warn("destroy invoker...");
                    }
                }
            }
        }
    }
}
```

destroyUnusedInvokers 方法的主要逻辑是通过 newUrlInvokerMap 找出待删除 Invoker 对应的 url，并将 url 存入到 deleted 列表中。然后再遍历 deleted 列表，并从 oldUrlInvokerMap 中移除相应的 Invoker，销毁之。整个逻辑大致如此，不是很难理解。

到此关于 Invoker 列表的刷新逻辑就分析了，这里对整个过程进行简单总结。如下：

1. 检测入参是否仅包含一个 url，且 url 协议头为 empty
2. 若第一步检测结果为 true，表示禁用所有服务，此时销毁所有的 Invoker
3. 若第一步检测结果为 false，此时将入参转为 Invoker 列表
4. 对上一步逻辑生成的结果进行进一步处理，得到方法名到 Invoker 的映射关系表
5. 合并多组 Invoker
6. 销毁无用 Invoker

Invoker 的刷新逻辑还是比较复杂的，大家在看的过程中多写点 demo 进行调试，以加深理解。

# 集群

http://dubbo.apache.org/zh/docs/v2.7/dev/source/cluster/

## 简介

​	为了避免单点故障，现在的应用通常至少会部署在两台服务器上。对于一些负载比较高的服务，会部署更多的服务器。这样，在同一环境下的服务提供者数量会大于1。对于服务消费者来说，同一环境下出现了多个服务提供者。这时会出现一个问题，服务消费者需要决定选择哪个服务提供者进行调用。另外服务调用失败时的处理措施也是需要考虑的，是重试呢，还是抛出异常，亦或是只打印异常等。为了处理这些问题，Dubbo 定义了集群接口 Cluster 以及 Cluster Invoker。集群 Cluster 用途是将多个服务提供者合并为一个 Cluster Invoker，并将这个 Invoker 暴露给服务消费者。这样一来，服务消费者只需通过这个 Invoker 进行远程调用即可，至于具体调用哪个服务提供者，以及调用失败后如何处理等问题，现在都交给集群模块去处理。集群模块是服务提供者和服务消费者的中间层，为服务消费者屏蔽了服务提供者的情况，这样服务消费者就可以专心处理远程调用相关事宜。比如发请求，接受服务提供者返回的数据等。这就是集群的作用。

​	Dubbo 提供了多种集群实现，包含但不限于 Failover Cluster、Failfast Cluster 和 Failsafe Cluster 等。每种集群实现类的用途不同，接下来会一一进行分析。

## 集群容错

​	在对集群相关代码进行分析之前，这里有必要先来介绍一下集群容错的所有组件。包含 Cluster、Cluster Invoker、Directory、Router 和 LoadBalance 等。

![](https://pic.imgdb.cn/item/612e208444eaada739ccfbd3.jpg)

​	集群工作过程可分为两个阶段，第一个阶段是在服务消费者初始化期间，集群 Cluster 实现类为服务消费者创建 Cluster Invoker 实例，即上图中的 merge 操作。第二个阶段是在服务消费者进行远程调用时。以 FailoverClusterInvoker 为例，该类型 Cluster Invoker 首先会调用 Directory 的 list 方法列举 Invoker 列表（可将 Invoker 简单理解为服务提供者）。Directory 的用途是保存 Invoker，可简单类比为 `List<Invoker>`。其实现类 RegistryDirectory 是一个动态服务目录，可感知注册中心配置的变化，它所持有的 Invoker 列表会随着注册中心内容的变化而变化。每次变化后，RegistryDirectory 会动态增删 Invoker，并调用 Router 的 route 方法进行路由，过滤掉不符合路由规则的 Invoker。当 FailoverClusterInvoker 拿到 Directory 返回的 Invoker 列表后，它会通过 LoadBalance 从 Invoker 列表中选择一个 Invoker。最后 FailoverClusterInvoker 会将参数传给 LoadBalance 选择出的 Invoker 实例的 invoke 方法，进行真正的远程调用。

​	以上就是集群工作的整个流程，这里并没介绍集群是如何容错的。Dubbo 主要提供了这样几种容错方式：

- Failover Cluster - 失败自动切换
- Failfast Cluster - 快速失败
- Failsafe Cluster - 失败安全
- Failback Cluster - 失败自动恢复
- Forking Cluster - 并行调用多个服务提供者

## 源码分析 2.7

### Cluster 实现类分析

​	我们在上一章看到了两个概念，分别是集群接口 Cluster 和 Cluster Invoker，这两者是不同的。Cluster 是接口，而 Cluster Invoker 是一种 Invoker。服务提供者的选择逻辑，以及远程调用失败后的的处理逻辑均是封装在 Cluster Invoker 中。那么 Cluster 接口和相关实现类有什么用呢？用途比较简单，仅用于生成 Cluster Invoker。下面我们来看一下源码。

```java
public class FailoverCluster implements Cluster {

    public final static String NAME = "failover";

    @Override
    public <T> Invoker<T> join(Directory<T> directory) throws RpcException {
        // 创建并返回 FailoverClusterInvoker 对象
        return new FailoverClusterInvoker<T>(directory);
    }
}
```

​	如上，FailoverCluster 总共就包含这几行代码，用于创建 FailoverClusterInvoker 对象，很简单。下面再看一个。

```java
public class FailbackCluster implements Cluster {

    public final static String NAME = "failback";

    @Override
    public <T> Invoker<T> join(Directory<T> directory) throws RpcException {
        // 创建并返回 FailbackClusterInvoker 对象
        return new FailbackClusterInvoker<T>(directory);
    }

}
```

​	如上，FailbackCluster 的逻辑也是很简单，无需解释了。所以接下来，我们把重点放在各种 Cluster Invoker 上

### Cluster Invoker 分析

​	我们首先从各种 Cluster Invoker 的父类 AbstractClusterInvoker 源码开始说起。前面说过，集群工作过程可分为两个阶段，第一个阶段是在服务消费者初始化期间，这个在[服务引用](http://dubbo.apache.org/zh/docs/v2.7/dev/source/cluster/)那篇文章中分析过，就不赘述。第二个阶段是在服务消费者进行远程调用时，此时 AbstractClusterInvoker 的 invoke 方法会被调用。列举 Invoker，负载均衡等操作均会在此阶段被执行。因此下面先来看一下 invoke 方法的逻辑。

```java
public Result invoke(final Invocation invocation) throws RpcException {
    checkWhetherDestroyed();
    LoadBalance loadbalance = null;

    // 绑定 attachments 到 invocation 中.
    Map<String, String> contextAttachments = RpcContext.getContext().getAttachments();
    if (contextAttachments != null && contextAttachments.size() != 0) {
        ((RpcInvocation) invocation).addAttachments(contextAttachments);
    }

    // 列举 Invoker
    List<Invoker<T>> invokers = list(invocation);
    if (invokers != null && !invokers.isEmpty()) {
        // 加载 LoadBalance
        loadbalance = ExtensionLoader.getExtensionLoader(LoadBalance.class).getExtension(invokers.get(0).getUrl()
                .getMethodParameter(RpcUtils.getMethodName(invocation), Constants.LOADBALANCE_KEY, Constants.DEFAULT_LOADBALANCE));
    }
    RpcUtils.attachInvocationIdIfAsync(getUrl(), invocation);
    
    // 调用 doInvoke 进行后续操作
    return doInvoke(invocation, invokers, loadbalance);
}

// 抽象方法，由子类实现
protected abstract Result doInvoke(Invocation invocation, List<Invoker<T>> invokers,
                                       LoadBalance loadbalance) throws RpcException;
```

​	AbstractClusterInvoker 的 invoke 方法主要用于列举 Invoker，以及加载 LoadBalance。最后再调用模板方法 doInvoke 进行后续操作。下面我们来看一下 Invoker 列举方法 list(Invocation) 的逻辑，如下：

```java
protected List<Invoker<T>> list(Invocation invocation) throws RpcException {
    // 调用 Directory 的 list 方法列举 Invoker
    List<Invoker<T>> invokers = directory.list(invocation);
    return invokers;
}
```

​	如上，AbstractClusterInvoker 中的 list 方法做的事情很简单，只是简单的调用了 Directory 的 list 方法，没有其他更多的逻辑了。Directory 即相关实现类在前文已经分析过，这里就不多说了。接下来，我们把目光转移到 AbstractClusterInvoker 的各种实现类上，来看一下这些实现类是如何实现 doInvoke 方法逻辑的。

#### FailoverClusterInvoker

```java
public class FailoverClusterInvoker<T> extends AbstractClusterInvoker<T> {

    // 省略部分代码

    @Override
    public Result doInvoke(Invocation invocation, final List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        List<Invoker<T>> copyinvokers = invokers;
        checkInvokers(copyinvokers, invocation);
        // 获取重试次数
        int len = getUrl().getMethodParameter(invocation.getMethodName(), Constants.RETRIES_KEY, Constants.DEFAULT_RETRIES) + 1;
        if (len <= 0) {
            len = 1;
        }
        RpcException le = null;
        List<Invoker<T>> invoked = new ArrayList<Invoker<T>>(copyinvokers.size());
        Set<String> providers = new HashSet<String>(len);
        // 循环调用，失败重试
        for (int i = 0; i < len; i++) {
            if (i > 0) {
                checkWhetherDestroyed();
                // 在进行重试前重新列举 Invoker，这样做的好处是，如果某个服务挂了，
                // 通过调用 list 可得到最新可用的 Invoker 列表
                copyinvokers = list(invocation);
                // 对 copyinvokers 进行判空检查
                checkInvokers(copyinvokers, invocation);
            }

            // 通过负载均衡选择 Invoker
            Invoker<T> invoker = select(loadbalance, invocation, copyinvokers, invoked);
            // 添加到 invoker 到 invoked 列表中
            invoked.add(invoker);
            // 设置 invoked 到 RPC 上下文中
            RpcContext.getContext().setInvokers((List) invoked);
            try {
                // 调用目标 Invoker 的 invoke 方法
                Result result = invoker.invoke(invocation);
                return result;
            } catch (RpcException e) {
                if (e.isBiz()) {
                    throw e;
                }
                le = e;
            } catch (Throwable e) {
                le = new RpcException(e.getMessage(), e);
            } finally {
                providers.add(invoker.getUrl().getAddress());
            }
        }
        
        // 若重试失败，则抛出异常
        throw new RpcException(..., "Failed to invoke the method ...");
    }
}
```

​	如上，FailoverClusterInvoker 的 doInvoke 方法首先是获取重试次数，然后根据重试次数进行循环调用，失败后进行重试。在 for 循环内，首先是通过负载均衡组件选择一个 Invoker，然后再通过这个 Invoker 的 invoke 方法进行远程调用。如果失败了，记录下异常，并进行重试。重试时会再次调用父类的 list 方法列举 Invoker。整个流程大致如此，不是很难理解。下面我们看一下 select 方法的逻辑。

```java
protected Invoker<T> select(LoadBalance loadbalance, Invocation invocation, List<Invoker<T>> invokers, List<Invoker<T>> selected) throws RpcException {
    if (invokers == null || invokers.isEmpty())
        return null;
    // 获取调用方法名
    String methodName = invocation == null ? "" : invocation.getMethodName();

    // 获取 sticky 配置，sticky 表示粘滞连接。所谓粘滞连接是指让服务消费者尽可能的
    // 调用同一个服务提供者，除非该提供者挂了再进行切换
    boolean sticky = invokers.get(0).getUrl().getMethodParameter(methodName, Constants.CLUSTER_STICKY_KEY, Constants.DEFAULT_CLUSTER_STICKY);
    {
        // 检测 invokers 列表是否包含 stickyInvoker，如果不包含，
        // 说明 stickyInvoker 代表的服务提供者挂了，此时需要将其置空
        if (stickyInvoker != null && !invokers.contains(stickyInvoker)) {
            stickyInvoker = null;
        }
        
        // 在 sticky 为 true，且 stickyInvoker != null 的情况下。如果 selected 包含 
        // stickyInvoker，表明 stickyInvoker 对应的服务提供者可能因网络原因未能成功提供服务。
        // 但是该提供者并没挂，此时 invokers 列表中仍存在该服务提供者对应的 Invoker。
        if (sticky && stickyInvoker != null && (selected == null || !selected.contains(stickyInvoker))) {
            // availablecheck 表示是否开启了可用性检查，如果开启了，则调用 stickyInvoker 的 
            // isAvailable 方法进行检查，如果检查通过，则直接返回 stickyInvoker。
            if (availablecheck && stickyInvoker.isAvailable()) {
                return stickyInvoker;
            }
        }
    }
    
    // 如果线程走到当前代码处，说明前面的 stickyInvoker 为空，或者不可用。
    // 此时继续调用 doSelect 选择 Invoker
    Invoker<T> invoker = doSelect(loadbalance, invocation, invokers, selected);

    // 如果 sticky 为 true，则将负载均衡组件选出的 Invoker 赋值给 stickyInvoker
    if (sticky) {
        stickyInvoker = invoker;
    }
    return invoker;
}
```

​	如上，select 方法的主要逻辑集中在了对粘滞连接特性的支持上。首先是获取 sticky 配置，然后再检测 invokers 列表中是否包含 stickyInvoker，如果不包含，则认为该 stickyInvoker 不可用，此时将其置空。这里的 invokers 列表可以看做是**存活着的服务提供者**列表，如果这个列表不包含 stickyInvoker，那自然而然的认为 stickyInvoker 挂了，所以置空。如果 stickyInvoker 存在于 invokers 列表中，此时要进行下一项检测 — 检测 selected 中是否包含 stickyInvoker。如果包含的话，说明 stickyInvoker 在此之前没有成功提供服务（但其仍然处于存活状态）。此时我们认为这个服务不可靠，不应该在重试期间内再次被调用，因此这个时候不会返回该 stickyInvoker。如果 selected 不包含 stickyInvoker，此时还需要进行可用性检测，比如检测服务提供者网络连通性等。当可用性检测通过，才可返回 stickyInvoker，否则调用 doSelect 方法选择 Invoker。如果 sticky 为 true，此时会将 doSelect 方法选出的 Invoker 赋值给 stickyInvoker。

​	以上就是 select 方法的逻辑，这段逻辑看起来不是很复杂，但是信息量比较大。不搞懂 invokers 和 selected 两个入参的含义，以及粘滞连接特性，这段代码是不容易看懂的。所以大家在阅读这段代码时，不要忽略了对背景知识的理解。关于 select 方法先分析这么多，继续向下分析。

```java
private Invoker<T> doSelect(LoadBalance loadbalance, Invocation invocation, List<Invoker<T>> invokers, List<Invoker<T>> selected) throws RpcException {
    if (invokers == null || invokers.isEmpty())
        return null;
    if (invokers.size() == 1)
        return invokers.get(0);
    if (loadbalance == null) {
        // 如果 loadbalance 为空，这里通过 SPI 加载 Loadbalance，默认为 RandomLoadBalance
        loadbalance = ExtensionLoader.getExtensionLoader(LoadBalance.class).getExtension(Constants.DEFAULT_LOADBALANCE);
    }
    
    // 通过负载均衡组件选择 Invoker
    Invoker<T> invoker = loadbalance.select(invokers, getUrl(), invocation);

	// 如果 selected 包含负载均衡选择出的 Invoker，或者该 Invoker 无法经过可用性检查，此时进行重选
    if ((selected != null && selected.contains(invoker))
            || (!invoker.isAvailable() && getUrl() != null && availablecheck)) {
        try {
            // 进行重选
            Invoker<T> rinvoker = reselect(loadbalance, invocation, invokers, selected, availablecheck);
            if (rinvoker != null) {
                // 如果 rinvoker 不为空，则将其赋值给 invoker
                invoker = rinvoker;
            } else {
                // rinvoker 为空，定位 invoker 在 invokers 中的位置
                int index = invokers.indexOf(invoker);
                try {
                    // 获取 index + 1 位置处的 Invoker，以下代码等价于：
                    //     invoker = invokers.get((index + 1) % invokers.size());
                    invoker = index < invokers.size() - 1 ? invokers.get(index + 1) : invokers.get(0);
                } catch (Exception e) {
                    logger.warn("... may because invokers list dynamic change, ignore.");
                }
            }
        } catch (Throwable t) {
            logger.error("cluster reselect fail reason is : ...");
        }
    }
    return invoker;
}
```

​	doSelect 主要做了两件事，第一是通过负载均衡组件选择 Invoker。第二是，如果选出来的 Invoker 不稳定，或不可用，此时需要调用 reselect 方法进行重选。若 reselect 选出来的 Invoker 为空，此时定位 invoker 在 invokers 列表中的位置 index，然后获取 index + 1 处的 invoker，这也可以看做是重选逻辑的一部分。下面我们来看一下 reselect 方法的逻辑。

```java
private Invoker<T> reselect(LoadBalance loadbalance, Invocation invocation,
    List<Invoker<T>> invokers, List<Invoker<T>> selected, boolean availablecheck) throws RpcException {

    List<Invoker<T>> reselectInvokers = new ArrayList<Invoker<T>>(invokers.size() > 1 ? (invokers.size() - 1) : invokers.size());

    // 下面的 if-else 分支逻辑有些冗余，pull request #2826 对这段代码进行了简化，可以参考一下
    // 根据 availablecheck 进行不同的处理
    if (availablecheck) {
        // 遍历 invokers 列表
        for (Invoker<T> invoker : invokers) {
            // 检测可用性
            if (invoker.isAvailable()) {
                // 如果 selected 列表不包含当前 invoker，则将其添加到 reselectInvokers 中
                if (selected == null || !selected.contains(invoker)) {
                    reselectInvokers.add(invoker);
                }
            }
        }
        
        // reselectInvokers 不为空，此时通过负载均衡组件进行选择
        if (!reselectInvokers.isEmpty()) {
            return loadbalance.select(reselectInvokers, getUrl(), invocation);
        }

    // 不检查 Invoker 可用性
    } else {
        for (Invoker<T> invoker : invokers) {
            // 如果 selected 列表不包含当前 invoker，则将其添加到 reselectInvokers 中
            if (selected == null || !selected.contains(invoker)) {
                reselectInvokers.add(invoker);
            }
        }
        if (!reselectInvokers.isEmpty()) {
            // 通过负载均衡组件进行选择
            return loadbalance.select(reselectInvokers, getUrl(), invocation);
        }
    }

    {
        // 若线程走到此处，说明 reselectInvokers 集合为空，此时不会调用负载均衡组件进行筛选。
        // 这里从 selected 列表中查找可用的 Invoker，并将其添加到 reselectInvokers 集合中
        if (selected != null) {
            for (Invoker<T> invoker : selected) {
                if ((invoker.isAvailable())
                        && !reselectInvokers.contains(invoker)) {
                    reselectInvokers.add(invoker);
                }
            }
        }
        if (!reselectInvokers.isEmpty()) {
            // 再次进行选择，并返回选择结果
            return loadbalance.select(reselectInvokers, getUrl(), invocation);
        }
    }
    return null;
}
```

​	reselect 方法总结下来其实只做了两件事情，第一是查找可用的 Invoker，并将其添加到 reselectInvokers 集合中。第二，如果 reselectInvokers 不为空，则通过负载均衡组件再次进行选择。其中第一件事情又可进行细分，一开始，reselect 从 invokers 列表中查找有效可用的 Invoker，若未能找到，此时再到 selected 列表中继续查找。关于 reselect 方法就先分析到这，继续分析其他的 Cluster Invoker。

#### FailbackClusterInvoker

​	FailbackClusterInvoker 会在调用失败后，返回一个空结果给服务消费者。并通过定时任务对失败的调用进行重传，适合执行消息通知等操作。下面来看一下它的实现逻辑。

```java
public class FailbackClusterInvoker<T> extends AbstractClusterInvoker<T> {

    private static final long RETRY_FAILED_PERIOD = 5 * 1000;

    private final ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(2,
            new NamedInternalThreadFactory("failback-cluster-timer", true));

    private final ConcurrentMap<Invocation, AbstractClusterInvoker<?>> failed = new ConcurrentHashMap<Invocation, AbstractClusterInvoker<?>>();
    private volatile ScheduledFuture<?> retryFuture;

    @Override
    protected Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        try {
            checkInvokers(invokers, invocation);
            // 选择 Invoker
            Invoker<T> invoker = select(loadbalance, invocation, invokers, null);
            // 进行调用
            return invoker.invoke(invocation);
        } catch (Throwable e) {
            // 如果调用过程中发生异常，此时仅打印错误日志，不抛出异常
            logger.error("Failback to invoke method ...");
            
            // 记录调用信息
            addFailed(invocation, this);
            // 返回一个空结果给服务消费者
            return new RpcResult();
        }
    }

    private void addFailed(Invocation invocation, AbstractClusterInvoker<?> router) {
        if (retryFuture == null) {
            synchronized (this) {
                if (retryFuture == null) {
                    // 创建定时任务，每隔5秒执行一次
                    retryFuture = scheduledExecutorService.scheduleWithFixedDelay(new Runnable() {

                        @Override
                        public void run() {
                            try {
                                // 对失败的调用进行重试
                                retryFailed();
                            } catch (Throwable t) {
                                // 如果发生异常，仅打印异常日志，不抛出
                                logger.error("Unexpected error occur at collect statistic", t);
                            }
                        }
                    }, RETRY_FAILED_PERIOD, RETRY_FAILED_PERIOD, TimeUnit.MILLISECONDS);
                }
            }
        }
        
        // 添加 invocation 和 invoker 到 failed 中
        failed.put(invocation, router);
    }

    void retryFailed() {
        if (failed.size() == 0) {
            return;
        }
        
        // 遍历 failed，对失败的调用进行重试
        for (Map.Entry<Invocation, AbstractClusterInvoker<?>> entry : new HashMap<Invocation, AbstractClusterInvoker<?>>(failed).entrySet()) {
            Invocation invocation = entry.getKey();
            Invoker<?> invoker = entry.getValue();
            try {
                // 再次进行调用
                invoker.invoke(invocation);
                // 调用成功后，从 failed 中移除 invoker
                failed.remove(invocation);
            } catch (Throwable e) {
                // 仅打印异常，不抛出
                logger.error("Failed retry to invoke method ...");
            }
        }
    }
}
```

​	这个类主要由3个方法组成，首先是 doInvoker，该方法负责初次的远程调用。若远程调用失败，则通过 addFailed 方法将调用信息存入到 failed 中，等待定时重试。addFailed 在开始阶段会根据 retryFuture 为空与否，来决定是否开启定时任务。retryFailed 方法则是包含了失败重试的逻辑，该方法会对 failed 进行遍历，然后依次对 Invoker 进行调用。调用成功则将 Invoker 从 failed 中移除，调用失败则忽略失败原因。

​	以上就是 FailbackClusterInvoker 的执行逻辑，不是很复杂，继续往下看。

#### FailfastClusterInvoker

​	FailfastClusterInvoker 只会进行一次调用，失败后立即抛出异常。适用于幂等操作，比如新增记录。源码如下：

```java
public class FailfastClusterInvoker<T> extends AbstractClusterInvoker<T> {

    @Override
    public Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        checkInvokers(invokers, invocation);
        // 选择 Invoker
        Invoker<T> invoker = select(loadbalance, invocation, invokers, null);
        try {
            // 调用 Invoker
            return invoker.invoke(invocation);
        } catch (Throwable e) {
            if (e instanceof RpcException && ((RpcException) e).isBiz()) {
                // 抛出异常
                throw (RpcException) e;
            }
            // 抛出异常
            throw new RpcException(..., "Failfast invoke providers ...");
        }
    }
}
```

​	如上，首先是通过 select 方法选择 Invoker，然后进行远程调用。如果调用失败，则立即抛出异常。FailfastClusterInvoker 就先分析到这，下面分析 FailsafeClusterInvoker。

#### FailsafeClusterInvoker

​	FailsafeClusterInvoker 是一种失败安全的 Cluster Invoker。所谓的失败安全是指，当调用过程中出现异常时，FailsafeClusterInvoker 仅会打印异常，而不会抛出异常。适用于写入审计日志等操作。下面分析源码。

```java
public class FailsafeClusterInvoker<T> extends AbstractClusterInvoker<T> {

    @Override
    public Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        try {
            checkInvokers(invokers, invocation);
            // 选择 Invoker
            Invoker<T> invoker = select(loadbalance, invocation, invokers, null);
            // 进行远程调用
            return invoker.invoke(invocation);
        } catch (Throwable e) {
			// 打印错误日志，但不抛出
            logger.error("Failsafe ignore exception: " + e.getMessage(), e);
            // 返回空结果忽略错误
            return new RpcResult();
        }
    }
}
```

​	FailsafeClusterInvoker 的逻辑和 FailfastClusterInvoker 的逻辑一样简单，无需过多说明。继续向下分析。

#### ForkingClusterInvoker

​	ForkingClusterInvoker 会在运行时通过线程池创建多个线程，并发调用多个服务提供者。只要有一个服务提供者成功返回了结果，doInvoke 方法就会立即结束运行。ForkingClusterInvoker 的应用场景是在一些对实时性要求比较高**读操作**（注意是读操作，并行写操作可能不安全）下使用，但这将会耗费更多的资源。下面来看该类的实现。

```java
public class ForkingClusterInvoker<T> extends AbstractClusterInvoker<T> {
    
    private final ExecutorService executor = Executors.newCachedThreadPool(
            new NamedInternalThreadFactory("forking-cluster-timer", true));

    @Override
    public Result doInvoke(final Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        try {
            checkInvokers(invokers, invocation);
            final List<Invoker<T>> selected;
            // 获取 forks 配置
            final int forks = getUrl().getParameter(Constants.FORKS_KEY, Constants.DEFAULT_FORKS);
            // 获取超时配置
            final int timeout = getUrl().getParameter(Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
            // 如果 forks 配置不合理，则直接将 invokers 赋值给 selected
            if (forks <= 0 || forks >= invokers.size()) {
                selected = invokers;
            } else {
                selected = new ArrayList<Invoker<T>>();
                // 循环选出 forks 个 Invoker，并添加到 selected 中
                for (int i = 0; i < forks; i++) {
                    // 选择 Invoker
                    Invoker<T> invoker = select(loadbalance, invocation, invokers, selected);
                    if (!selected.contains(invoker)) {
                        selected.add(invoker);
                    }
                }
            }
            
            // ----------------------✨ 分割线1 ✨---------------------- //
            
            RpcContext.getContext().setInvokers((List) selected);
            final AtomicInteger count = new AtomicInteger();
            final BlockingQueue<Object> ref = new LinkedBlockingQueue<Object>();
            // 遍历 selected 列表
            for (final Invoker<T> invoker : selected) {
                // 为每个 Invoker 创建一个执行线程
                executor.execute(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            // 进行远程调用
                            Result result = invoker.invoke(invocation);
                            // 将结果存到阻塞队列中
                            ref.offer(result);
                        } catch (Throwable e) {
                            int value = count.incrementAndGet();
                            // 仅在 value 大于等于 selected.size() 时，才将异常对象
                            // 放入阻塞队列中，请大家思考一下为什么要这样做。
                            if (value >= selected.size()) {
                                // 将异常对象存入到阻塞队列中
                                ref.offer(e);
                            }
                        }
                    }
                });
            }
            
            // ----------------------✨ 分割线2 ✨---------------------- //
            
            try {
                // 从阻塞队列中取出远程调用结果
                Object ret = ref.poll(timeout, TimeUnit.MILLISECONDS);
                
                // 如果结果类型为 Throwable，则抛出异常
                if (ret instanceof Throwable) {
                    Throwable e = (Throwable) ret;
                    throw new RpcException(..., "Failed to forking invoke provider ...");
                }
                
                // 返回结果
                return (Result) ret;
            } catch (InterruptedException e) {
                throw new RpcException("Failed to forking invoke provider ...");
            }
        } finally {
            RpcContext.getContext().clearAttachments();
        }
    }
}
```

​	ForkingClusterInvoker 的 doInvoker 方法比较长，这里通过两个分割线将整个方法划分为三个逻辑块。从方法开始到分割线1之间的代码主要是用于选出 forks 个 Invoker，为接下来的并发调用提供输入。分割线1和分割线2之间的逻辑通过线程池并发调用多个 Invoker，并将结果存储在阻塞队列中。分割线2到方法结尾之间的逻辑主要用于从阻塞队列中获取返回结果，并对返回结果类型进行判断。如果为异常类型，则直接抛出，否则返回。

​	以上就是ForkingClusterInvoker 的 doInvoker 方法大致过程。我们在分割线1和分割线2之间的代码上留了一个问题，问题是这样的：为什么要在`value >= selected.size()`的情况下，才将异常对象添加到阻塞队列中？这里来解答一下。原因是这样的，在并行调用多个服务提供者的情况下，只要有一个服务提供者能够成功返回结果，而其他全部失败。此时 ForkingClusterInvoker 仍应该返回成功的结果，而非抛出异常。在`value >= selected.size()`时将异常对象放入阻塞队列中，可以保证异常对象不会出现在正常结果的前面，这样可从阻塞队列中优先取出正常的结果。

​	关于 ForkingClusterInvoker 就先分析到这，接下来分析最后一个 Cluster Invoker。

#### BroadcastClusterInvoker

​	本章的最后，我们再来看一下 BroadcastClusterInvoker。BroadcastClusterInvoker 会逐个调用每个服务提供者，如果其中一台报错，在循环调用结束后，BroadcastClusterInvoker 会抛出异常。该类通常用于通知所有提供者更新缓存或日志等本地资源信息。源码如下。

```java
public class BroadcastClusterInvoker<T> extends AbstractClusterInvoker<T> {

    @Override
    public Result doInvoke(final Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        checkInvokers(invokers, invocation);
        RpcContext.getContext().setInvokers((List) invokers);
        RpcException exception = null;
        Result result = null;
        // 遍历 Invoker 列表，逐个调用
        for (Invoker<T> invoker : invokers) {
            try {
                // 进行远程调用
                result = invoker.invoke(invocation);
            } catch (RpcException e) {
                exception = e;
                logger.warn(e.getMessage(), e);
            } catch (Throwable e) {
                exception = new RpcException(e.getMessage(), e);
                logger.warn(e.getMessage(), e);
            }
        }
        
        // exception 不为空，则抛出异常
        if (exception != null) {
            throw exception;
        }
        return result;
    }
}
```

## 总结

​	

# 负载均衡	

http://dubbo.apache.org/zh/docs/v2.7/dev/source/loadbalance/

## 简介

​	LoadBalance 中文意思为负载均衡，它的职责是将网络请求，或者其他形式的负载“均摊”到不同的机器上。避免集群中部分服务器压力过大，而另一些服务器比较空闲的情况。通过负载均衡，可以让每台服务器获取到适合自己处理能力的负载。在为高负载服务器分流的同时，还可以避免资源浪费，一举两得。负载均衡可分为软件负载均衡和硬件负载均衡。

​	Dubbo 提供了4种负载均衡实现，分别是基于权重随机算法的 RandomLoadBalance、基于最少活跃调用数算法的 LeastActiveLoadBalance、基于 hash 一致性的 ConsistentHashLoadBalance，以及基于加权轮询算法的 RoundRobinLoadBalance。这几个负载均衡算法代码不是很长，但是想看懂也不是很容易，需要大家对这几个算法的原理有一定了解才行。

## 源码分析 2.7

​	在 Dubbo 中，所有负载均衡实现类均继承自 AbstractLoadBalance，该类实现了 LoadBalance 接口，并封装了一些公共的逻辑。

```java
@Override
public <T> Invoker<T> select(List<Invoker<T>> invokers, URL url, Invocation invocation) {
    if (invokers == null || invokers.isEmpty())
        return null;
    // 如果 invokers 列表中仅有一个 Invoker，直接返回即可，无需进行负载均衡
    if (invokers.size() == 1)
        return invokers.get(0);
    
    // 调用 doSelect 方法进行负载均衡，该方法为抽象方法，由子类实现
    return doSelect(invokers, url, invocation);
}

protected abstract <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation);
```

​	AbstractLoadBalance 除了实现了 LoadBalance 接口方法，还封装了一些公共逻辑，比如服务提供者权重计算逻辑。具体实现如下：

```java
	protected int getWeight(Invoker<?> invoker, Invocation invocation) {
    // 从 url 中获取权重 weight 配置值
    int weight = invoker.getUrl().getMethodParameter(invocation.getMethodName(), Constants.WEIGHT_KEY, Constants.DEFAULT_WEIGHT);
    if (weight > 0) {
        // 获取服务提供者启动时间戳
        long timestamp = invoker.getUrl().getParameter(Constants.REMOTE_TIMESTAMP_KEY, 0L);
        if (timestamp > 0L) {
            // 计算服务提供者运行时长
            int uptime = (int) (System.currentTimeMillis() - timestamp);
            // 获取服务预热时间，默认为10分钟
            int warmup = invoker.getUrl().getParameter(Constants.WARMUP_KEY, Constants.DEFAULT_WARMUP);
            // 如果服务运行时间小于预热时间，则重新计算服务权重，即降权
            if (uptime > 0 && uptime < warmup) {
                // 重新计算服务权重
                weight = calculateWarmupWeight(uptime, warmup, weight);
            }
        }
    }
    return weight;
}

static int calculateWarmupWeight(int uptime, int warmup, int weight) {
    // 计算权重，下面代码逻辑上形似于 (uptime / warmup) * weight。
    // 随着服务运行时间 uptime 增大，权重计算值 ww 会慢慢接近配置值 weight
    int ww = (int) ((float) uptime / ((float) warmup / (float) weight));
    return ww < 1 ? 1 : (ww > weight ? weight : ww);
}
```

​	上面是权重的计算过程，该过程主要用于保证当服务运行时长小于服务预热时间时，对服务进行降权，避免让服务在启动之初就处于高负载状态。服务预热是一个优化手段，与此类似的还有 JVM 预热。主要目的是让服务启动后“低功率”运行一段时间，使其效率慢慢提升至最佳状态。

### RandomLoadBalance

​	RandomLoadBalance 是加权随机算法的具体实现，它的算法思想很简单。假设我们有一组服务器 servers = [A, B, C]，他们对应的权重为 weights = [5, 3, 2]，权重总和为10。现在把这些权重值平铺在一维坐标值上，[0, 5) 区间属于服务器 A，[5, 8) 区间属于服务器 B，[8, 10) 区间属于服务器 C。接下来通过随机数生成器生成一个范围在 [0, 10) 之间的随机数，然后计算这个随机数会落到哪个区间上。比如数字3会落到服务器 A 对应的区间上，此时返回服务器 A 即可。权重越大的机器，在坐标轴上对应的区间范围就越大，因此随机数生成器生成的数字就会有更大的概率落到此区间内。只要随机数生成器产生的随机数分布性很好，在经过多次选择后，每个服务器被选中的次数比例接近其权重比例。比如，经过一万次选择后，服务器 A 被选中的次数大约为5000次，服务器 B 被选中的次数约为3000次，服务器 C 被选中的次数约为2000次。

```java
public class RandomLoadBalance extends AbstractLoadBalance {

    public static final String NAME = "random";

    private final Random random = new Random();

    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        int length = invokers.size();
        int totalWeight = 0;
        boolean sameWeight = true;
        // 下面这个循环有两个作用，第一是计算总权重 totalWeight，
        // 第二是检测每个服务提供者的权重是否相同
        for (int i = 0; i < length; i++) {
            int weight = getWeight(invokers.get(i), invocation);
            // 累加权重
            totalWeight += weight;
            // 检测当前服务提供者的权重与上一个服务提供者的权重是否相同，
            // 不相同的话，则将 sameWeight 置为 false。
            if (sameWeight && i > 0
                    && weight != getWeight(invokers.get(i - 1), invocation)) {
                sameWeight = false;
            }
        }
        
        // 下面的 if 分支主要用于获取随机数，并计算随机数落在哪个区间上
        if (totalWeight > 0 && !sameWeight) {
            // 随机获取一个 [0, totalWeight) 区间内的数字
            int offset = random.nextInt(totalWeight);
            // 循环让 offset 数减去服务提供者权重值，当 offset 小于0时，返回相应的 Invoker。
            // 举例说明一下，我们有 servers = [A, B, C]，weights = [5, 3, 2]，offset = 7。
            // 第一次循环，offset - 5 = 2 > 0，即 offset > 5，
            // 表明其不会落在服务器 A 对应的区间上。
            // 第二次循环，offset - 3 = -1 < 0，即 5 < offset < 8，
            // 表明其会落在服务器 B 对应的区间上
            for (int i = 0; i < length; i++) {
                // 让随机值 offset 减去权重值
                offset -= getWeight(invokers.get(i), invocation);
                if (offset < 0) {
                    // 返回相应的 Invoker
                    return invokers.get(i);
                }
            }
        }
        
        // 如果所有服务提供者权重值相同，此时直接随机返回一个即可
        return invokers.get(random.nextInt(length));
    }
}
```

​	RandomLoadBalance 的算法思想比较简单，在经过多次请求后，能够将调用请求按照权重值进行“均匀”分配。当然 RandomLoadBalance 也存在一定的缺点，当调用次数比较少时，Random 产生的随机数可能会比较集中，此时多数请求会落到同一台服务器上。这个缺点并不是很严重，多数情况下可以忽略。RandomLoadBalance 是一个简单，高效的负载均衡实现，因此 Dubbo 选择它作为缺省实现。

###  LeastActiveLoadBalance

​	LeastActiveLoadBalance 翻译过来是最小活跃数负载均衡。活跃调用数越小，表明该服务提供者效率越高，单位时间内可处理更多的请求。此时应优先将请求分配给该服务提供者。在具体实现中，每个服务提供者对应一个活跃数 active。初始情况下，所有服务提供者活跃数均为0。每收到一个请求，活跃数加1，完成请求后则将活跃数减1。在服务运行一段时间后，性能好的服务提供者处理请求的速度更快，因此活跃数下降的也越快，此时这样的服务提供者能够优先获取到新的服务请求、这就是最小活跃数负载均衡算法的基本思想。除了最小活跃数，LeastActiveLoadBalance 在实现上还引入了权重值。所以准确的来说，LeastActiveLoadBalance 是基于加权最小活跃数算法实现的。举个例子说明一下，在一个服务提供者集群中，有两个性能优异的服务提供者。某一时刻它们的活跃数相同，此时 Dubbo 会根据它们的权重去分配请求，权重越大，获取到新请求的概率就越大。如果两个服务提供者权重相同，此时随机选择一个即可。关于 LeastActiveLoadBalance 的背景知识就先介绍到这里，下面开始分析源码。

```java
public class LeastActiveLoadBalance extends AbstractLoadBalance {

    public static final String NAME = "leastactive";

    private final Random random = new Random();

    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        int length = invokers.size();
        // 最小的活跃数
        int leastActive = -1;
        // 具有相同“最小活跃数”的服务者提供者（以下用 Invoker 代称）数量
        int leastCount = 0; 
        // leastIndexs 用于记录具有相同“最小活跃数”的 Invoker 在 invokers 列表中的下标信息
        int[] leastIndexs = new int[length];
        int totalWeight = 0;
        // 第一个最小活跃数的 Invoker 权重值，用于与其他具有相同最小活跃数的 Invoker 的权重进行对比，
        // 以检测是否“所有具有相同最小活跃数的 Invoker 的权重”均相等
        int firstWeight = 0;
        boolean sameWeight = true;

        // 遍历 invokers 列表
        for (int i = 0; i < length; i++) {
            Invoker<T> invoker = invokers.get(i);
            // 获取 Invoker 对应的活跃数
            int active = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName()).getActive();
            // 获取权重 - ⭐️
            int weight = invoker.getUrl().getMethodParameter(invocation.getMethodName(), Constants.WEIGHT_KEY, Constants.DEFAULT_WEIGHT);
            // 发现更小的活跃数，重新开始
            if (leastActive == -1 || active < leastActive) {
            	// 使用当前活跃数 active 更新最小活跃数 leastActive
                leastActive = active;
                // 更新 leastCount 为 1
                leastCount = 1;
                // 记录当前下标值到 leastIndexs 中
                leastIndexs[0] = i;
                totalWeight = weight;
                firstWeight = weight;
                sameWeight = true;

            // 当前 Invoker 的活跃数 active 与最小活跃数 leastActive 相同 
            } else if (active == leastActive) {
            	// 在 leastIndexs 中记录下当前 Invoker 在 invokers 集合中的下标
                leastIndexs[leastCount++] = i;
                // 累加权重
                totalWeight += weight;
                // 检测当前 Invoker 的权重与 firstWeight 是否相等，
                // 不相等则将 sameWeight 置为 false
                if (sameWeight && i > 0
                    && weight != firstWeight) {
                    sameWeight = false;
                }
            }
        }
        
        // 当只有一个 Invoker 具有最小活跃数，此时直接返回该 Invoker 即可
        if (leastCount == 1) {
            return invokers.get(leastIndexs[0]);
        }

        // 有多个 Invoker 具有相同的最小活跃数，但它们之间的权重不同
        if (!sameWeight && totalWeight > 0) {
        	// 随机生成一个 [0, totalWeight) 之间的数字
            int offsetWeight = random.nextInt(totalWeight);
            // 循环让随机数减去具有最小活跃数的 Invoker 的权重值，
            // 当 offset 小于等于0时，返回相应的 Invoker
            for (int i = 0; i < leastCount; i++) {
                int leastIndex = leastIndexs[i];
                // 获取权重值，并让随机数减去权重值 - ⭐️
                offsetWeight -= getWeight(invokers.get(leastIndex), invocation);
                if (offsetWeight <= 0)
                    return invokers.get(leastIndex);
            }
        }
        // 如果权重相同或权重为0时，随机返回一个 Invoker
        return invokers.get(leastIndexs[random.nextInt(leastCount)]);
    }
}
```

​	上面代码的逻辑比较多，我们在代码中写了大量的注释，有帮助大家理解代码逻辑。下面简单总结一下以上代码所做的事情，如下：

1. 遍历 invokers 列表，寻找活跃数最小的 Invoker
2. 如果有多个 Invoker 具有相同的最小活跃数，此时记录下这些 Invoker 在 invokers 集合中的下标，并累加它们的权重，比较它们的权重值是否相等
3. 如果只有一个 Invoker 具有最小的活跃数，此时直接返回该 Invoker 即可
4. 如果有多个 Invoker 具有最小活跃数，且它们的权重不相等，此时处理方式和 RandomLoadBalance 一致
5. 如果有多个 Invoker 具有最小活跃数，但它们的权重相等，此时随机返回一个即可

以上就是 LeastActiveLoadBalance 大致的实现逻辑，大家在阅读的源码的过程中要注意区分活跃数与权重这两个概念，不要混为一谈。

​	以上分析是基于 Dubbo 2.6.4 版本进行的，由于近期 Dubbo 2.6.5 发布了，并对 LeastActiveLoadBalance 进行了一些修改，下面简单来介绍一下修改内容。回到上面的源码中，我们在上面的代码中标注了两个黄色的五角星⭐️。两处标记对应的代码分别如下：

```java
int weight = invoker.getUrl().getMethodParameter(invocation.getMethodName(), Constants.WEIGHT_KEY, Constants.DEFAULT_WEIGHT);
```

```java
offsetWeight -= getWeight(invokers.get(leastIndex), invocation);
```

​	问题出在服务预热阶段，第一行代码直接从 url 中取权重值，未被降权过。第二行代码获取到的是经过降权后的权重。第一行代码获取到的权重值最终会被累加到权重总和 totalWeight 中，这个时候会导致一个问题。offsetWeight 是一个在 [0, totalWeight) 范围内的随机数，而它所减去的是经过降权的权重。很有可能在经过 leastCount 次运算后，offsetWeight 仍然是大于0的，导致无法选中 Invoker。这个问题对应的 issue 为 [#904](https://github.com/apache/dubbo/issues/904)，并在 pull request [#2172](https://github.com/apache/dubbo/pull/2172) 中被修复。具体的修复逻辑是将标注一处的代码修改为：

```java
// afterWarmup 等价于上面的 weight 变量，这样命名是为了强调该变量经过了 warmup 降权处理
int afterWarmup = getWeight(invoker, invocation);
```

​	另外，2.6.4 版本中的 LeastActiveLoadBalance 还有一个缺陷，即当一组 Invoker 具有相同的最小活跃数，且其中一个 Invoker 的权重值为1，此时这个 Invoker 无法被选中。缺陷代码如下：

```java
int offsetWeight = random.nextInt(totalWeight);
for (int i = 0; i < leastCount; i++) {
    int leastIndex = leastIndexs[i];
    offsetWeight -= getWeight(invokers.get(leastIndex), invocation);
    if (offsetWeight <= 0)    // ❌
        return invokers.get(leastIndex);
}
```

​	问题出在了`offsetWeight <= 0`上，举例说明，假设有一组 Invoker 的权重为 5、2、1，offsetWeight 最大值为 7。假设 offsetWeight = 7，你会发现，当 for 循环进行第二次遍历后 offsetWeight = 7 - 5 - 2 = 0，提前返回了。此时，此时权重为1的 Invoker 就没有机会被选中了。该问题在 Dubbo 2.6.5 中被修复了，修改后的代码如下：

```mysql
int offsetWeight = random.nextInt(totalWeight) + 1;
```

​	以上就是 Dubbo 2.6.5 对 LeastActiveLoadBalance 的更新，内容不是很多，先分析到这。接下来分析基于一致性 hash 思想的 ConsistentHashLoadBalance。

### ConsistentHashLoadBalance

​	一致性 hash 算法由麻省理工学院的 Karger 及其合作者于1997年提出的，算法提出之初是用于大规模缓存系统的负载均衡。它的工作过程是这样的，首先根据 ip 或者其他的信息为缓存节点生成一个 hash，并将这个 hash 投射到 [0, 232 - 1] 的圆环上。当有查询或写入请求时，则为缓存项的 key 生成一个 hash 值。然后查找第一个大于或等于该 hash 值的缓存节点，并到这个节点中查询或写入缓存项。如果当前节点挂了，则在下一次查询或写入缓存时，为缓存项查找另一个大于其 hash 值的缓存节点即可。大致效果如下图所示，每个缓存节点在圆环上占据一个位置。如果缓存项的 key 的 hash 值小于缓存节点 hash 值，则到该缓存节点中存储或读取缓存项。比如下面绿色点对应的缓存项将会被存储到 cache-2 节点中。由于 cache-3 挂了，原本应该存到该节点中的缓存项最终会存储到 cache-4 节点中。

![](https://pic.imgdb.cn/item/6126549544eaada739b16761.jpg)

​	这里相同颜色的节点均属于同一个服务提供者，比如 Invoker1-1，Invoker1-2，……, Invoker1-160。这样做的目的是通过引入虚拟节点，让 Invoker 在圆环上分散开来，避免数据倾斜问题。所谓数据倾斜是指，由于节点不够分散，导致大量请求落到了同一个节点上，而其他节点只会接收到了少量请求的情况。比如：

![](https://pic.imgdb.cn/item/612654b244eaada739b1ddb5.jpg)

​	如上，由于 Invoker-1 和 Invoker-2 在圆环上分布不均，导致系统中75%的请求都会落到 Invoker-1 上，只有 25% 的请求会落到 Invoker-2 上。解决这个问题办法是引入虚拟节点，通过虚拟节点均衡各个节点的请求量。

​	到这里背景知识就普及完了，接下来开始分析源码。我们先从 ConsistentHashLoadBalance 的 doSelect 方法开始看起，如下：

```java
public class ConsistentHashLoadBalance extends AbstractLoadBalance {

    private final ConcurrentMap<String, ConsistentHashSelector<?>> selectors = 
        new ConcurrentHashMap<String, ConsistentHashSelector<?>>();

    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        String methodName = RpcUtils.getMethodName(invocation);
        String key = invokers.get(0).getUrl().getServiceKey() + "." + methodName;

        // 获取 invokers 原始的 hashcode
        int identityHashCode = System.identityHashCode(invokers);
        ConsistentHashSelector<T> selector = (ConsistentHashSelector<T>) selectors.get(key);
        // 如果 invokers 是一个新的 List 对象，意味着服务提供者数量发生了变化，可能新增也可能减少了。
        // 此时 selector.identityHashCode != identityHashCode 条件成立
        if (selector == null || selector.identityHashCode != identityHashCode) {
            // 创建新的 ConsistentHashSelector
            selectors.put(key, new ConsistentHashSelector<T>(invokers, methodName, identityHashCode));
            selector = (ConsistentHashSelector<T>) selectors.get(key);
        }

        // 调用 ConsistentHashSelector 的 select 方法选择 Invoker
        return selector.select(invocation);
    }
    
    private static final class ConsistentHashSelector<T> {...}
}
```

​	如上，doSelect 方法主要做了一些前置工作，比如检测 invokers 列表是不是变动过，以及创建 ConsistentHashSelector。这些工作做完后，接下来开始调用 ConsistentHashSelector 的 select 方法执行负载均衡逻辑。在分析 select 方法之前，我们先来看一下一致性 hash 选择器 ConsistentHashSelector 的初始化过程，如下：

```java
private static final class ConsistentHashSelector<T> {

    // 使用 TreeMap 存储 Invoker 虚拟节点
    private final TreeMap<Long, Invoker<T>> virtualInvokers;

    private final int replicaNumber;

    private final int identityHashCode;

    private final int[] argumentIndex;

    ConsistentHashSelector(List<Invoker<T>> invokers, String methodName, int identityHashCode) {
        this.virtualInvokers = new TreeMap<Long, Invoker<T>>();
        this.identityHashCode = identityHashCode;
        URL url = invokers.get(0).getUrl();
        // 获取虚拟节点数，默认为160
        this.replicaNumber = url.getMethodParameter(methodName, "hash.nodes", 160);
        // 获取参与 hash 计算的参数下标值，默认对第一个参数进行 hash 运算
        String[] index = Constants.COMMA_SPLIT_PATTERN.split(url.getMethodParameter(methodName, "hash.arguments", "0"));
        argumentIndex = new int[index.length];
        for (int i = 0; i < index.length; i++) {
            argumentIndex[i] = Integer.parseInt(index[i]);
        }
        for (Invoker<T> invoker : invokers) {
            String address = invoker.getUrl().getAddress();
            for (int i = 0; i < replicaNumber / 4; i++) {
                // 对 address + i 进行 md5 运算，得到一个长度为16的字节数组
                byte[] digest = md5(address + i);
                // 对 digest 部分字节进行4次 hash 运算，得到四个不同的 long 型正整数
                for (int h = 0; h < 4; h++) {
                    // h = 0 时，取 digest 中下标为 0 ~ 3 的4个字节进行位运算
                    // h = 1 时，取 digest 中下标为 4 ~ 7 的4个字节进行位运算
                    // h = 2, h = 3 时过程同上
                    long m = hash(digest, h);
                    // 将 hash 到 invoker 的映射关系存储到 virtualInvokers 中，
                    // virtualInvokers 需要提供高效的查询操作，因此选用 TreeMap 作为存储结构
                    virtualInvokers.put(m, invoker);
                }
            }
        }
    }
}
```

​	ConsistentHashSelector 的构造方法执行了一系列的初始化逻辑，比如从配置中获取虚拟节点数以及参与 hash 计算的参数下标，默认情况下只使用第一个参数进行 hash。需要特别说明的是，ConsistentHashLoadBalance 的负载均衡逻辑只受参数值影响，具有相同参数值的请求将会被分配给同一个服务提供者。ConsistentHashLoadBalance 不 关系权重，因此使用时需要注意一下。

​	在获取虚拟节点数和参数下标配置后，接下来要做的事情是计算虚拟节点 hash 值，并将虚拟节点存储到 TreeMap 中。到此，ConsistentHashSelector 初始化工作就完成了。接下来，我们来看看 select 方法的逻辑。

```java
public Invoker<T> select(Invocation invocation) {
    // 将参数转为 key
    String key = toKey(invocation.getArguments());
    // 对参数 key 进行 md5 运算
    byte[] digest = md5(key);
    // 取 digest 数组的前四个字节进行 hash 运算，再将 hash 值传给 selectForKey 方法，
    // 寻找合适的 Invoker
    return selectForKey(hash(digest, 0));
}

private Invoker<T> selectForKey(long hash) {
    // 到 TreeMap 中查找第一个节点值大于或等于当前 hash 的 Invoker
    Map.Entry<Long, Invoker<T>> entry = virtualInvokers.tailMap(hash, true).firstEntry();
    // 如果 hash 大于 Invoker 在圆环上最大的位置，此时 entry = null，
    // 需要将 TreeMap 的头节点赋值给 entry
    if (entry == null) {
        entry = virtualInvokers.firstEntry();
    }

    // 返回 Invoker
    return entry.getValue();
}
```

​	如上，选择的过程相对比较简单了。首先是对参数进行 md5 以及 hash 运算，得到一个 hash 值。然后再拿这个值到 TreeMap 中查找目标 Invoker 即可。

​	到此关于 ConsistentHashLoadBalance 就分析完了。在阅读 ConsistentHashLoadBalance 源码之前，大家一定要先补充背景知识，不然很难看懂代码逻辑。

### RoundRobinLoadBalance

​	本节，我们来看一下 Dubbo 中加权轮询负载均衡的实现 RoundRobinLoadBalance。在详细分析源码前，我们先来了解一下什么是加权轮询。这里从最简单的轮询开始讲起，所谓轮询是指将请求轮流分配给每台服务器。举个例子，我们有三台服务器 A、B、C。我们将第一个请求分配给服务器 A，第二个请求分配给服务器 B，第三个请求分配给服务器 C，第四个请求再次分配给服务器 A。这个过程就叫做轮询。轮询是一种无状态负载均衡算法，实现简单，适用于每台服务器性能相近的场景下。但现实情况下，我们并不能保证每台服务器性能均相近。如果我们将等量的请求分配给性能较差的服务器，这显然是不合理的。因此，这个时候我们需要对轮询过程进行加权，以调控每台服务器的负载。经过加权后，每台服务器能够得到的请求数比例，接近或等于他们的权重比。比如服务器 A、B、C 权重比为 5:2:1。那么在8次请求中，服务器 A 将收到其中的5次请求，服务器 B 会收到其中的2次请求，服务器 C 则收到其中的1次请求。

​	以上就是加权轮询的算法思想，搞懂了这个思想，接下来我们就可以分析源码了。我们先来看一下 2.6.4 版本的 RoundRobinLoadBalance。

```java
public class RoundRobinLoadBalance extends AbstractLoadBalance {

    public static final String NAME = "roundrobin";

    private final ConcurrentMap<String, AtomicPositiveInteger> sequences = 
        new ConcurrentHashMap<String, AtomicPositiveInteger>();

    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        // key = 全限定类名 + "." + 方法名，比如 com.xxx.DemoService.sayHello
        String key = invokers.get(0).getUrl().getServiceKey() + "." + invocation.getMethodName();
        int length = invokers.size();
        // 最大权重
        int maxWeight = 0;
        // 最小权重
        int minWeight = Integer.MAX_VALUE;
        final LinkedHashMap<Invoker<T>, IntegerWrapper> invokerToWeightMap = new LinkedHashMap<Invoker<T>, IntegerWrapper>();
        // 权重总和
        int weightSum = 0;

        // 下面这个循环主要用于查找最大和最小权重，计算权重总和等
        for (int i = 0; i < length; i++) {
            int weight = getWeight(invokers.get(i), invocation);
            // 获取最大和最小权重
            maxWeight = Math.max(maxWeight, weight);
            minWeight = Math.min(minWeight, weight);
            if (weight > 0) {
                // 将 weight 封装到 IntegerWrapper 中
                invokerToWeightMap.put(invokers.get(i), new IntegerWrapper(weight));
                // 累加权重
                weightSum += weight;
            }
        }

        // 查找 key 对应的对应 AtomicPositiveInteger 实例，为空则创建。
        // 这里可以把 AtomicPositiveInteger 看成一个黑盒，大家只要知道
        // AtomicPositiveInteger 用于记录服务的调用编号即可。至于细节，
        // 大家如果感兴趣，可以自行分析
        AtomicPositiveInteger sequence = sequences.get(key);
        if (sequence == null) {
            sequences.putIfAbsent(key, new AtomicPositiveInteger());
            sequence = sequences.get(key);
        }

        // 获取当前的调用编号
        int currentSequence = sequence.getAndIncrement();
        // 如果最小权重小于最大权重，表明服务提供者之间的权重是不相等的
        if (maxWeight > 0 && minWeight < maxWeight) {
            // 使用调用编号对权重总和进行取余操作
            int mod = currentSequence % weightSum;
            // 进行 maxWeight 次遍历
            for (int i = 0; i < maxWeight; i++) {
                // 遍历 invokerToWeightMap
                for (Map.Entry<Invoker<T>, IntegerWrapper> each : invokerToWeightMap.entrySet()) {
					// 获取 Invoker
                    final Invoker<T> k = each.getKey();
                    // 获取权重包装类 IntegerWrapper
                    final IntegerWrapper v = each.getValue();
                    
                    // 如果 mod = 0，且权重大于0，此时返回相应的 Invoker
                    if (mod == 0 && v.getValue() > 0) {
                        return k;
                    }
                    
                    // mod != 0，且权重大于0，此时对权重和 mod 分别进行自减操作
                    if (v.getValue() > 0) {
                        v.decrement();
                        mod--;
                    }
                }
            }
        }
        
        // 服务提供者之间的权重相等，此时通过轮询选择 Invoker
        return invokers.get(currentSequence % length);
    }

    // IntegerWrapper 是一个 int 包装类，主要包含了一个自减方法。
    private static final class IntegerWrapper {
        private int value;

        public void decrement() {
            this.value--;
        }
        
        // 省略部分代码
    }
}
```

如上，RoundRobinLoadBalance 的每行代码都不是很难理解，但是将它们组合在一起之后，就不是很好理解了。所以下面我们举例进行说明，假设我们有三台服务器 servers = [A, B, C]，对应的权重为 weights = [2, 5, 1]。接下来对上面的逻辑进行简单的模拟。

mod = 0：满足条件，此时直接返回服务器 A

mod = 1：需要进行一次递减操作才能满足条件，此时返回服务器 B

mod = 2：需要进行两次递减操作才能满足条件，此时返回服务器 C

mod = 3：需要进行三次递减操作才能满足条件，经过递减后，服务器权重为 [1, 4, 0]，此时返回服务器 A

mod = 4：需要进行四次递减操作才能满足条件，经过递减后，服务器权重为 [0, 4, 0]，此时返回服务器 B

mod = 5：需要进行五次递减操作才能满足条件，经过递减后，服务器权重为 [0, 3, 0]，此时返回服务器 B

mod = 6：需要进行六次递减操作才能满足条件，经过递减后，服务器权重为 [0, 2, 0]，此时返回服务器 B

mod = 7：需要进行七次递减操作才能满足条件，经过递减后，服务器权重为 [0, 1, 0]，此时返回服务器 B

经过8次调用后，我们得到的负载均衡结果为 [A, B, C, A, B, B, B, B]，次数比 A:B:C = 2:5:1，等于权重比。当 sequence = 8 时，mod = 0，此时重头再来。从上面的模拟过程可以看出，当 mod >= 3 后，服务器 C 就不会被选中了，因为它的权重被减为0了。当 mod >= 4 后，服务器 A 的权重被减为0，此后 A 就不会再被选中。

​	以上是 2.6.4 版本的 RoundRobinLoadBalance 分析过程，2.6.4 版本的 RoundRobinLoadBalance 在某些情况下存在着比较严重的性能问题，该问题最初是在 [issue #2578](https://github.com/apache/dubbo/issues/2578) 中被反馈出来。问题出在了 Invoker 的返回时机上，RoundRobinLoadBalance 需要在`mod == 0 && v.getValue() > 0` 条件成立的情况下才会被返回相应的 Invoker。假如 mod 很大，比如 10000，50000，甚至更大时，doSelect 方法需要进行很多次计算才能将 mod 减为0。由此可知，doSelect 的效率与 mod 有关，时间复杂度为 O(mod)。mod 又受最大权重 maxWeight 的影响，因此当某个服务提供者配置了非常大的权重，此时 RoundRobinLoadBalance 会产生比较严重的性能问题。这个问题被反馈后，社区很快做了回应。并对 RoundRobinLoadBalance 的代码进行了重构，将时间复杂度优化至了常量级别。这个优化可以说很好了，下面我们来学习一下优化后的代码。

```java
public class RoundRobinLoadBalance extends AbstractLoadBalance {

    public static final String NAME = "roundrobin";

    private final ConcurrentMap<String, AtomicPositiveInteger> sequences = new ConcurrentHashMap<String, AtomicPositiveInteger>();

    private final ConcurrentMap<String, AtomicPositiveInteger> indexSeqs = new ConcurrentHashMap<String, AtomicPositiveInteger>();

    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        String key = invokers.get(0).getUrl().getServiceKey() + "." + invocation.getMethodName();
        int length = invokers.size();
        int maxWeight = 0;
        int minWeight = Integer.MAX_VALUE;
        final List<Invoker<T>> invokerToWeightList = new ArrayList<>();
        
        // 查找最大和最小权重
        for (int i = 0; i < length; i++) {
            int weight = getWeight(invokers.get(i), invocation);
            maxWeight = Math.max(maxWeight, weight);
            minWeight = Math.min(minWeight, weight);
            if (weight > 0) {
                invokerToWeightList.add(invokers.get(i));
            }
        }
        
        // 获取当前服务对应的调用序列对象 AtomicPositiveInteger
        AtomicPositiveInteger sequence = sequences.get(key);
        if (sequence == null) {
            // 创建 AtomicPositiveInteger，默认值为0
            sequences.putIfAbsent(key, new AtomicPositiveInteger());
            sequence = sequences.get(key);
        }
        
        // 获取下标序列对象 AtomicPositiveInteger
        AtomicPositiveInteger indexSeq = indexSeqs.get(key);
        if (indexSeq == null) {
            // 创建 AtomicPositiveInteger，默认值为 -1
            indexSeqs.putIfAbsent(key, new AtomicPositiveInteger(-1));
            indexSeq = indexSeqs.get(key);
        }

        if (maxWeight > 0 && minWeight < maxWeight) {
            length = invokerToWeightList.size();
            while (true) {
                int index = indexSeq.incrementAndGet() % length;
                int currentWeight = sequence.get() % maxWeight;

                // 每循环一轮（index = 0），重新计算 currentWeight
                if (index == 0) {
                    currentWeight = sequence.incrementAndGet() % maxWeight;
                }
                
                // 检测 Invoker 的权重是否大于 currentWeight，大于则返回
                if (getWeight(invokerToWeightList.get(index), invocation) > currentWeight) {
                    return invokerToWeightList.get(index);
                }
            }
        }
        
        // 所有 Invoker 权重相等，此时进行普通的轮询即可
        return invokers.get(sequence.incrementAndGet() % length);
    }
}
```

上面代码的逻辑是这样的，每进行一轮循环，重新计算 currentWeight。如果当前 Invoker 权重大于 currentWeight，则返回该 Invoker。下面举例说明，假设服务器 [A, B, C] 对应权重 [5, 2, 1]。

第一轮循环，currentWeight = 1，可返回 A 和 B

第二轮循环，currentWeight = 2，返回 A

第三轮循环，currentWeight = 3，返回 A

第四轮循环，currentWeight = 4，返回 A

第五轮循环，currentWeight = 0，返回 A, B, C

​	如上，这里的一轮循环是指 index 再次变为0所经历过的循环，这里可以把 index = 0 看做是一轮循环的开始。每一轮循环的次数与 Invoker 的数量有关，Invoker 数量通常不会太多，所以我们可以认为上面代码的时间复杂度为常数级。

​	重构后的 RoundRobinLoadBalance 看起来已经很不错了，但是在代码更新不久后，很快又被重构了。这次重构原因是新的 RoundRobinLoadBalance 在某些情况下选出的服务器序列不够均匀。比如，服务器 [A, B, C] 对应权重 [5, 1, 1]。进行7次负载均衡后，选择出来的序列为 [A, A, A, A, A, B, C]。前5个请求全部都落在了服务器 A上，这将会使服务器 A 短时间内接收大量的请求，压力陡增。而 B 和 C 此时无请求，处于空闲状态。而我们期望的结果是这样的 [A, A, B, A, C, A, A]，不同服务器可以穿插获取请求。为了增加负载均衡结果的平滑性，社区再次对 RoundRobinLoadBalance 的实现进行了重构，这次重构参考自 Nginx 的平滑加权轮询负载均衡。每个服务器对应两个权重，分别为 weight 和 currentWeight。其中 weight 是固定的，currentWeight 会动态调整，初始值为0。当有新的请求进来时，遍历服务器列表，让它的 currentWeight 加上自身权重。遍历完成后，找到最大的 currentWeight，并将其减去权重总和，然后返回相应的服务器即可。

​	上面描述不是很好理解，下面还是举例进行说明。这里仍然使用服务器 [A, B, C] 对应权重 [5, 1, 1] 的例子说明，现在有7个请求依次进入负载均衡逻辑，选择过程如下：

| 请求编号 | currentWeight 数组 | 选择结果 | 减去权重总和后的 currentWeight 数组 |
| -------- | ------------------ | -------- | ----------------------------------- |
| 1        | [5, 1, 1]          | A        | [-2, 1, 1]                          |
| 2        | [3, 2, 2]          | A        | [-4, 2, 2]                          |
| 3        | [1, 3, 3]          | B        | [1, -4, 3]                          |
| 4        | [6, -3, 4]         | A        | [-1, -3, 4]                         |
| 5        | [4, -2, 5]         | C        | [4, -2, -2]                         |
| 6        | [9, -1, -1]        | A        | [2, -1, -1]                         |
| 7        | [7, 0, 0]          | A        | [0, 0, 0]                           |

​	如上，经过平滑性处理后，得到的服务器序列为 [A, A, B, A, C, A, A]，相比之前的序列 [A, A, A, A, A, B, C]，分布性要好一些。初始情况下 currentWeight = [0, 0, 0]，第7个请求处理完后，currentWeight 再次变为 [0, 0, 0]。

```java
public class RoundRobinLoadBalance extends AbstractLoadBalance {
    public static final String NAME = "roundrobin";
    
    private static int RECYCLE_PERIOD = 60000;
    
    protected static class WeightedRoundRobin {
        // 服务提供者权重
        private int weight;
        // 当前权重
        private AtomicLong current = new AtomicLong(0);
        // 最后一次更新时间
        private long lastUpdate;
        
        public void setWeight(int weight) {
            this.weight = weight;
            // 初始情况下，current = 0
            current.set(0);
        }
        public long increaseCurrent() {
            // current = current + weight；
            return current.addAndGet(weight);
        }
        public void sel(int total) {
            // current = current - total;
            current.addAndGet(-1 * total);
        }
    }

    // 嵌套 Map 结构，存储的数据结构示例如下：
    // {
    //     "UserService.query": {
    //         "url1": WeightedRoundRobin@123, 
    //         "url2": WeightedRoundRobin@456, 
    //     },
    //     "UserService.update": {
    //         "url1": WeightedRoundRobin@123, 
    //         "url2": WeightedRoundRobin@456,
    //     }
    // }
    // 最外层为服务类名 + 方法名，第二层为 url 到 WeightedRoundRobin 的映射关系。
    // 这里我们可以将 url 看成是服务提供者的 id
    private ConcurrentMap<String, ConcurrentMap<String, WeightedRoundRobin>> methodWeightMap = new ConcurrentHashMap<String, ConcurrentMap<String, WeightedRoundRobin>>();
    
    // 原子更新锁
    private AtomicBoolean updateLock = new AtomicBoolean();
    
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        String key = invokers.get(0).getUrl().getServiceKey() + "." + invocation.getMethodName();
        // 获取 url 到 WeightedRoundRobin 映射表，如果为空，则创建一个新的
        ConcurrentMap<String, WeightedRoundRobin> map = methodWeightMap.get(key);
        if (map == null) {
            methodWeightMap.putIfAbsent(key, new ConcurrentHashMap<String, WeightedRoundRobin>());
            map = methodWeightMap.get(key);
        }
        int totalWeight = 0;
        long maxCurrent = Long.MIN_VALUE;
        
        // 获取当前时间
        long now = System.currentTimeMillis();
        Invoker<T> selectedInvoker = null;
        WeightedRoundRobin selectedWRR = null;

        // 下面这个循环主要做了这样几件事情：
        //   1. 遍历 Invoker 列表，检测当前 Invoker 是否有
        //      相应的 WeightedRoundRobin，没有则创建
        //   2. 检测 Invoker 权重是否发生了变化，若变化了，
        //      则更新 WeightedRoundRobin 的 weight 字段
        //   3. 让 current 字段加上自身权重，等价于 current += weight
        //   4. 设置 lastUpdate 字段，即 lastUpdate = now
        //   5. 寻找具有最大 current 的 Invoker，以及 Invoker 对应的 WeightedRoundRobin，
        //      暂存起来，留作后用
        //   6. 计算权重总和
        for (Invoker<T> invoker : invokers) {
            String identifyString = invoker.getUrl().toIdentityString();
            WeightedRoundRobin weightedRoundRobin = map.get(identifyString);
            int weight = getWeight(invoker, invocation);
            if (weight < 0) {
                weight = 0;
            }
            
            // 检测当前 Invoker 是否有对应的 WeightedRoundRobin，没有则创建
            if (weightedRoundRobin == null) {
                weightedRoundRobin = new WeightedRoundRobin();
                // 设置 Invoker 权重
                weightedRoundRobin.setWeight(weight);
                // 存储 url 唯一标识 identifyString 到 weightedRoundRobin 的映射关系
                map.putIfAbsent(identifyString, weightedRoundRobin);
                weightedRoundRobin = map.get(identifyString);
            }
            // Invoker 权重不等于 WeightedRoundRobin 中保存的权重，说明权重变化了，此时进行更新
            if (weight != weightedRoundRobin.getWeight()) {
                weightedRoundRobin.setWeight(weight);
            }
            
            // 让 current 加上自身权重，等价于 current += weight
            long cur = weightedRoundRobin.increaseCurrent();
            // 设置 lastUpdate，表示近期更新过
            weightedRoundRobin.setLastUpdate(now);
            // 找出最大的 current 
            if (cur > maxCurrent) {
                maxCurrent = cur;
                // 将具有最大 current 权重的 Invoker 赋值给 selectedInvoker
                selectedInvoker = invoker;
                // 将 Invoker 对应的 weightedRoundRobin 赋值给 selectedWRR，留作后用
                selectedWRR = weightedRoundRobin;
            }
            
            // 计算权重总和
            totalWeight += weight;
        }

        // 对 <identifyString, WeightedRoundRobin> 进行检查，过滤掉长时间未被更新的节点。
        // 该节点可能挂了，invokers 中不包含该节点，所以该节点的 lastUpdate 长时间无法被更新。
        // 若未更新时长超过阈值后，就会被移除掉，默认阈值为60秒。
        if (!updateLock.get() && invokers.size() != map.size()) {
            if (updateLock.compareAndSet(false, true)) {
                try {
                    ConcurrentMap<String, WeightedRoundRobin> newMap = new ConcurrentHashMap<String, WeightedRoundRobin>();
                    // 拷贝
                    newMap.putAll(map);
                    
                    // 遍历修改，即移除过期记录
                    Iterator<Entry<String, WeightedRoundRobin>> it = newMap.entrySet().iterator();
                    while (it.hasNext()) {
                        Entry<String, WeightedRoundRobin> item = it.next();
                        if (now - item.getValue().getLastUpdate() > RECYCLE_PERIOD) {
                            it.remove();
                        }
                    }
                    
                    // 更新引用
                    methodWeightMap.put(key, newMap);
                } finally {
                    updateLock.set(false);
                }
            }
        }

        if (selectedInvoker != null) {
            // 让 current 减去权重总和，等价于 current -= totalWeight
            selectedWRR.sel(totalWeight);
            // 返回具有最大 current 的 Invoker
            return selectedInvoker;
        }
        
        // should not happen here
        return invokers.get(0);
    }
}
```

