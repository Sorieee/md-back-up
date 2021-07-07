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

# 语雀

https://item.jd.com/12545055.html

https://www.yuque.com/apache-dubbo/dubbo3/pyrcyr

https://www.yuque.com/apache-dubbo

