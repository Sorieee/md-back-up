---
title: zookeeper学习笔记
date: 2020-12-13 14:38:16
tags: [zookeeper]
---

# 《Zookeeper: 分布式过程协同技术详解》

# 1. 简介

## 1.1 Zookeeper的使命

​	关于ZooKeeper这样的系统功能的讨论都围绕着一条主线：它可以在分布式系统中协作多个任务。

​	让我们看一些ZooKeeper的使用实例，以便更直观地理解其用处：

**Apache HBase**

​	HBase是一个通常与Hadoop一起使用的数据存储仓库。在HBase中，ZooKeeper用于选举一个集群内的主节点，以便跟踪可用的服务器，并保存集群的元数据。

**Apache Kafka**

​	Kafka是一个基于发布-订阅（pub-sub）模型的消息系统。其中ZooKeeper用于检测崩溃，实现主题（topic）的发现，并保持主题的生产和消费状态。

**Apache Solr**

Solr是一个企业级的搜索平台。Solr的分布式版本命名为SolrCloud，它使用ZooKeeper来存储集群的元数据，并协作更新这些元数据。

**Yahoo！Fetching Service**

​	Yahoo！Fetching Service是爬虫实现的一部分，通过缓存内容的方式高效地获取网页信息，同时确保满足网页服务器的管理规则（比如robots.txt文件）。该服务采用ZooKeeper实现主节点选举、崩溃检测和元数据存储。

**Facebook Messages**

​	Facebook推出的这个应用（http://on.fb.me/1a7uViK）集成了email、短信、Facebook聊天和Facebook收件箱等通信通道。该应用将ZooKeeper作为控制器，用来实现数据分片、故障恢复和服务发现等功能。



Zookeep的客户端API功能强大，其中包括：

* 保障强一致性、有序性和持久性。
* 实现通用的同步原语的能力。
* 在实际分布式系统中，并发往往导致不正确的行为。ZooKeeper提供了一种简单的并发处理机制。

### ZooKeeper改变了什么

​	使用ZooKeeper是否意味着需要以全新的方式进行应用程序开发？事实并非如此，ZooKeeper实际上简化了开发流程，提供了更加敏捷健壮的方案。

​	ZooKeeper之前的其他一些系统采用分布式锁管理器或者分布式数据库来实现协作。实际上，ZooKeeper也从这些系统中借鉴了很多概念。但是，ZooKeeper的设计更专注于任务协作，并不提供任何锁的接口或通用存储数据接口。同时，ZooKeeper没有给开发人员强加任何特殊的同步原语，使用起来非常灵活。

### ZooKeeper不适用的场景

​	整个ZooKeeper的服务器集群管理着应用协作的关键数据。ZooKeeper不适合用作海量数据存储。对于需要存储海量的应用数据的情况，我们有很多备选方案，比如说数据库和分布式文件系统等。因为不同的应用有不同的需求，如对一致性和持久性的不同需求，所以在设计应用时，最佳实践还是应该将应用数据和协同数据独立开。

### 通过ZooKeeper构建分布式系统

​	分布式系统中的进程通信有两种选择：直接通过网络进行信息交换，或读写某些共享存储。ZooKeeper使用共享存储模型来实现应用间的协作和同步原语。对于共享存储本身，又需要在进程和存储间进行网络通信。我们强调网络通信的重要性，因为它是分布式系统中并发设计的基础。

​	在真实的系统中，我们需要特别注意以下问题：

**消息延迟**

​	操作系统的调度和超载也可能导致消息处理的任意延迟。当一个进程向另一个进程发送消息时，整个消息的延时时间约等于发送端消耗的时间、传输时间、接收端的处理时间的总和。如果发送或接收过程需要调度时间进行处理，消息延时会更高。

**处理器性能**

​	操作系统的调度和超载也可能导致消息处理的任意延迟。当一个进程向另一个进程发送消息时，整个消息的延时时间约等于发送端消耗的时间、传输时间、接收端的处理时间的总和。如果发送或接收过程需要调度时间进行处理，消息延时会更高。

**时钟偏移**

​	使用时间概念的系统并不少见，比如，确定某一时间系统中发生了哪些事件。处理器时钟并不可靠，它们之间也会发生任意的偏移。因此，依赖处理器时钟也许会导致错误的决策。

## 1.2 示例：主-从应用

![](https://pic.imgdb.cn/item/6130976c44eaada739e85c58.jpg)

​	要实现主-从模式的系统，我们必须解决以下三个关键问题：

**主节点崩溃**

​	如果主节点发送错误并失效，系统将无法分配新的任务或重新分配已失败的任务。

**从节点崩溃**

​	如果从节点崩溃，已分配的任务将无法完成。

**通信故障**

如果主节点和从节点之间无法进行信息交换，从节点将无法得知新任务分配给它。

### 主节点失效

​	主节点失效时，我们需要有一个备份主节点（backup master）。当主要主节点（primary master）崩溃时，备份主节点接管主要主节点的角色，进行故障转移，然而，这并不是简单开始处理进入主节点的请求。新的主要主节点需要能够恢复到旧的主要主节点崩溃时的状态。对于主节点状态的可恢复性，我们不能依靠从已经崩溃的主节点来获取这些信息，而需要从其他地方获取，也就是通过ZooKeeper来获取。

​	状态恢复并不是唯一的重要问题。假如主节点有效，备份主节点却认为主节点已经崩溃。这种错误的假设可能发生在以下情况，例如主节点负载很高，导致消息任意延迟（关于这部分内容请参见1.1.4节），备份主节点将会接管成为主节点的角色，执行所有必需的程序，最终可能以主节点的角色开始执行，成为第二个主要主节点。更糟的是，如果一些从节点无法与主要主节点通信，如由于网络分区（network partition）错误导致，这些从节点可能会停止与主要主节点的通信，而与第二个主要主节点建立主-从关系。针对这个场景中导致的问题，我们一般称之为脑裂（split-brain）：系统中两个或者多个部分开始独立工作，导致整体行为不一致性。我们需要找出一种方法来处理主节点失效的情况，关键是我们需要避免发生脑裂的情况。

### 从节点失效

​	客户端向主节点提交任务，之后主节点将任务派发到有效的从节点中。从节点接收到派发的任务，执行完这些任务后会向主节点报告执行状态。主节点下一步会将执行结果通知给客户端。

​	如果从节点崩溃了，所有已派发给这个从节点且尚未完成的任务需要重新派发。其中首要需求是让主节点具有检测从节点的崩溃的能力。主节点必须能够检测到从节点的崩溃，并确定哪些从节点是否有效以便派发崩溃节点的任务。一个从节点崩溃时，从节点也许执行了部分任务，也许全部执行完，但没有报告结果。如果整个运算过程产生了其他作用，我们还有必要执行某些恢复过程来清除之前的状态。

### 通信故障

​	如果一个从节点与主节点的网络连接断开，比如网络分区（network partition）导致，重新分配一个任务可能会导致两个从节点执行相同的任务。如果一个任务允许多次执行，我们在进行任务再分配时可以不用验证第一个从节点是否完成了该任务。如果一个任务不允许，那么我们的应用需要适应多个从节点执行相同任务的可能性。

> **关于“仅一次”和“最多一次”的语义**
>
> 对任务加锁并不能保证一个任务执行多次，比如以下场景中描述的情况：
>
> 1.主节点M1派发任务T1给从节点W1。
>
> 2.W1为任务T1获取锁，执行任务，然后释放锁。
>
> 3.M1怀疑W1已经崩溃，所以再次派发任务T1给从节点W2。
>
> 4.W2为任务T1获取锁，执行任务，然后释放锁。

​	在这里，T1的锁并没有阻止任务被执行两次，因为两个从节点间运行任务时没有步骤交错。处理类似情况就需要“仅一次”和“最多一次”的语义学，而这又依赖于应用的特定处理机制。例如，如果应用数据使用了时间戳数据，而假定任务会修改应用数据，那么该任务的执行成功就取决于这个任务所取得的这个时间戳的值。如果改变应用状态的操作不是原子性操作，那么应用还需要具有局部变更的回退能力，否则最终将导致应用的非一致性。

​	通信故障导致的另一个重要问题是对锁等同步原语的影响。因为节点可能崩溃，而系统也可能网络分区（network partition），锁机制也会阻止任务的继续执行。因此ZooKeeper也需要实现处理这些情况的机制。首先，客户端可以告诉ZooKeeper某些数据的状态是临时状态（ephemeral）；其次，同时ZooKeeper需要客户端定时发送是否存活的通知，如果一个客户端未能及时发送通知，那么所有从属于这个客户端的临时状态的数据将全部被删除。通过这两个机制，在崩溃或通信故障发生时，我们就可以预防客户端独立运行而发生的应用宕机。

### 任务总结

**主节点选举**

​	这是关键的一步，使得主节点可以给从节点分配任务。

**崩溃检测**

​	主节点必须具有检测从节点崩溃或失去连接的能力。

**组成员关系管理**

​	主节点必须具有知道哪一个从节点可以执行任务的能力。

**元数据管理**

​	主节点和从节点必须具有通过某种可靠的方式来保存分配状态和执行状态的能力。

## 1.3 分布式协作的难点

​	当开发分布式应用时，其复杂性会立即突显出来。例如，当我们的应用启动后，所有不同的进程通过某种方法，需要知道应用的配置信息，一段时间之后，配置信息也许发生了变化，我们可以停止所有进程，重新分发配置信息的文件，然后重新启动，但是重新配置就会延长应用的停机时间。

​	与配置信息问题相关的是组成员关系的问题，当负载变化时，我们希望增加或减少新机器和进程。

​	当你自己实现分布式应用时，这个问题仅仅被描述为功能性问题，你可以设计解决方案，部署前你测试了你的解决方案，并非常确定地认为你已经正确解决了问题。当你在开发分布式应用时，你就会遇到真正困难的问题，你就不得不面对故障，如崩溃、通信故障等各种情况。这些问题会在任何可能的点突然出现，甚至无法列举需要处理的所有的情况。

**注意：拜占庭将军问题**

​	拜占庭将军问题（Byzantine Faults）是指可能导致一个组件发生任意行为（常常是意料之外的）的故障。这个故障的组件可能会破坏应用的状态，甚至是恶意行为。系统是建立在假设会发生这些故障，需要更高程度的复制并使用安全原语的基础上。尽管我们从学术文献中知道，针对拜占庭将军问题技术发展已经取得了巨大进步，我们还是觉得没有必要在ZooKeeper中采用这些技术，因此，我们也避免代码库中引入额外的复杂性。

​	在独立主机上运行的应用与分布式应用发生的故障存在显著的区别：在分布式应用中，可能会发生局部故障，当独立主机崩溃，这个主机上运行的所有进程都会失败，如果是独立主机上运行多个进程，一个进程执行的失败，其他进程可以通过操作系统获得这个故障，操作系统提供了健壮的多进程消息通信的保障。在分布式环境中这一切发生了改变：如果一个主机或进程发生故障，其他主机继续运行，并会接管发生故障的进程，为了能够处理故障进程，这些仍在运行的进程必须能够检测到这个故障，无论是消息丢失或发生了时间偏移。

​	这个例子原本是一个在分布式计算领域非常著名的定律，被称为FLP（由其作者命名：Fischer，Lynch，Patterson），这个结论证明了在异步通信的分布式系统中，进程崩溃，所有进程可能无法在这个比特位的配置上达成一致[[1\]](part0009.html#ch1_back)。类似的定律称为CAP，表示一致性（Consistency）、可用性（Availability）和分区容错性（Partition-tolerance），该定律指出，当设计一个分布式系统时，我们希望这三种属性全部满足，但没有系统可以同时满足这三种属性[[2\]](part0009.html#ch2_back)。因此ZooKeeper的设计尽可能满足一致性和可用性，当然，在发生网络分区时ZooKeeper也提供了只读能力。

### ZooKeeper的成功和注意事项

​	不得不指出，完美的解决方案是不存在的，我们重申ZooKeeper无法解决分布式应用开发者面对的所有问题，而是为开发者提供了一个优雅的框架来处理这些问题。多年以来，ZooKeeper在分布式计算领域进行了大量的工作。Paxos算法[[1\]](part0010.html#ch1_back)和虚拟同步技术（virtual synchrony）[[2\]](part0010.html#ch2_back)给ZooKeeper的设计带来了很大影响，通过这些技术可以无缝地处理所发生的某些变化或情况，并提供给开发者一个框架，来应对无法自动处理的某些情况。

# 2. 了解ZooKeeper

## 2.1 ZooKeeper基础

​	很多用于协作的原语常常在很多应用之间共享，因此，设计一个用于协作需求的服务的方法往往是提供原语列表，暴露出每个原语的实例化调用方法，并直接控制这些实例。比如，我们可以说分布式锁机制组成了一个重要的原语，同时暴露出创建（create）、获取（acquire）和释放（release）三个调用方法。

​	这种设计存在一些重大的缺陷：首先，我们要么预先提出一份详尽的原语列表，要么提供API的扩展，以便引入新的原语；其次，以这种方式实现原语的服务使得应用丧失了灵活性。

​	因此，在ZooKeeper中我们另辟蹊径。ZooKeeper并不直接暴露原语，取而代之，它暴露了由一小部分调用方法组成的类似文件系统的API，以便允许应用实现自己的原语。我们通常使用菜谱（recipes）来表示这些原语的实现。菜谱包括ZooKeeper操作和维护一个小型的数据节点，这些节点被称为znode，采用类似于文件系统的层级树状结构进行管理。图2-1描述了一个znode树的结构，根节点包含4个子节点，其中三个子节点拥有下一级节点，叶子节点存储了数据信息。

![](https://pic.imgdb.cn/item/6130d0d744eaada7395ab9d8.jpg)

·/workers节点作为父节点，其下每个znode子节点保存了系统中一个可用从节点信息。如图2-1所示，有一个从节点（foot.com：2181）。

·/tasks节点作为父节点，其下每个znode子节点保存了所有已经创建并等待从节点执行的任务的信息，主-从模式的应用的客户端在/tasks下添加一个znode子节点，用来表示一个新任务，并等待任务状态的znode节点。

·/assign节点作为父节点，其下每个znode子节点保存了分配到某个从节点的一个任务信息，当主节点为某个从节点分配了一个任务，就会在/assign下增加一个子节点。

### 2.1.1 API概述

​	znode节点可能含有数据，也可能没有。如果一个znode节点包含任何数据，那么数据存储为字节数组（byte array）。字节数组的具体格式特定于每个应用的实现，ZooKeeper并不直接提供解析的支持。我们可以使用如Protocol Buffers、Thrift、Avro或MessagePack等序列化（Serialization）包来方便地处理保存于znode节点的数据格式，不过有些时候，以UTF-8或ASCII编码的字符串已经够用了。

​	ZooKeeper的API暴露了以下方法：

**create/path data**

​	创建一个名为/path的znode节点，并包含数据data。

**delete/path**

​	删除名为/path的znode。

**exists/path**

​	检查是否存在名为/path的节点。

**setData/path data**

​	设置名为/path的znode的数据为data。

**getData/path**

​	返回名为/path节点的数据信息。

**getChildren/path**

​	返回所有/path节点的所有子节点列表。



​	需要注意的是，ZooKeeper并不允许局部写入或读取znode节点的数据。当设置一个znode节点的数据或读取时，znode节点的内容会被整个替换或全部读取进来。

​	ZooKeeper客户端连接到ZooKeeper服务，通过API调用来建立会话（session）。如果你对如何使用ZooKeeper非常感兴趣，请跳转到后面的“会话”一节，在那一节会讲解如何通过命令行的方式来运行ZooKeeper指令。

### 2.1.2 znode的不同类型

​	当新建znode时，还需要指定该节点的类型（mode），不同的类型决定了znode节点的行为方式。

**持久节点和临时节点**

​	znode节点可以是持久（persistent）节点，还可以是临时（ephemeral）节点。持久的znode，如/path，只能通过调用delete来进行删除。临时的znode与之相反，当创建该节点的客户端崩溃或关闭了与ZooKeeper的连接时，这个节点就会被删除。

​	持久znode是一种非常有用的znode，可以通过持久类型的znode为应用保存一些数据，即使znode的创建者不再属于应用系统时，数据也可以保存下来而不丢失。例如，在主-从模式例子中，需要保存从节点的任务分配情况，即使分配任务的主节点已经崩溃了。

​	临时znode传达了应用某些方面的信息，仅当创建者的会话有效时这些信息必须有效保存。例如，在主从模式的例子中，当主节点创建的znode为临时节点时，该节点的存在意味着现在有一个主节点，且主节点状态处于正常运行中。如果主znode消失后，该znode节点仍然存在，那么系统将无法监测到主节点崩溃。这样就可以阻止系统继续进行，因此这个znode需要和主节点一起消失。我们也在从节点中使用临时的znode，如果一个从节点失效，那么会话将会过期，之后znode/workers也将自动消失。

​	一个临时znode，在以下两种情况下将会被删除：

​	1.当创建该znode的客户端的会话因超时或主动关闭而中止时。

​	2.当某个客户端（不一定是创建者）主动删除该节点时。

**有序节点**

​	一个znode还可以设置为有序（sequential）节点。一个有序znode节点被分配唯一个单调递增的整数。当创建有序节点时，一个序号会被追加到路径之后。例如，如果一个客户端创建了一个有序znode节点，其路径为/tasks/task-，那么ZooKeeper将会分配一个序号，如1，并将这个数字追加到路径之后，最后该znode节点为/tasks/task-1。有序znode通过提供了创建具有唯一名称的znode的简单方式。同时也通过这种方式可以直观地查看znode的创建顺序。

​	总之，znode一共有4种类型：持久的（persistent）、临时的（ephemeral）、持久有序的（persistent_sequential）和临时有序的（ephemeral_sequential）。

### 2.1.3 监视与通知

​	ZooKeeper通常以远程服务的方式被访问，如果每次访问znode时，客户端都需要获得节点中的内容，这样的代价就非常大。因为这样会导致更高的延迟，而且ZooKeeper需要做更多的操作。考虑图2-2中的例子，第二次调用getChildren/tasks返回了相同的值，一个空的集合，其实是没有必要的。

![](https://pic.imgdb.cn/item/6130e0ce44eaada7398601a4.jpg)

​	这是一个常见的轮询问题。为了替换客户端的轮询，我们选择了基于通知（notification）的机制：客户端向ZooKeeper注册需要接收通知的znode，通过对znode设置监视点（watch）来接收通知。监视点是一个单次触发的操作，意即监视点会触发一个通知。为了接收多个通知，客户端必须在每次通知后设置一个新的监视点。在图2-3阐述的情况下，当节点/tasks发生变化时，客户端会收到一个通知，并从ZooKeeper读取一个新值。

![](https://pic.imgdb.cn/item/6130e0f244eaada7398661c5.jpg)

​	通知机制的一个重要保障是，对同一个znode的操作，先向客户端传送通知，然后再对该节点进行变更。如果客户端对一个znode设置了监视点，而该znode发生了两个连续更新。第一次更新后，客户端在观察第二次变化前就接收到了通知，然后读取znode中的数据。我们认为主要特性在于通知机制阻止了客户端所观察的更新顺序。虽然ZooKeeper的状态变化传播给某些客户端时更慢，但我们保障客户端以全局的顺序来观察ZooKeeper的状态。

​	ZooKeeper可以定义不同类型的通知，这依赖于设置监视点对应的通知类型。客户端可以设置多种监视点，如监控znode的数据变化、监控znode子节点的变化、监控znode的创建或删除。为了设置监视点，可以使用任何API中的调用来读取ZooKeeper的状态，在调用这些API时，传入一个watcher对象或使用默认的watcher。本章后续（主从模式的实现）及第4章会以主从模式的例子来展开讨论，我们将深入研究如何使用该机制。

**注意：谁来管理我的缓存**

​	如果不让客户端来管理其拥有的ZooKeeper数据的缓存，我们不得不让ZooKeeper来管理这些应用程序的缓存。但是，这样会导致ZooKeeper的设计更加复杂。事实上，如果让ZooKeeper管理缓存失效，可能会导致ZooKeeper在运行时，停滞在等待客户端确认一个缓存失效的请求上，因为在进行所有的写操作前，需要确认所有的缓存数据是否已经失效。

#### 2.1.4 版本

​	每一个znode都有一个版本号，它随着每次数据变化而自增。两个API操作可以有条件地执行：setData和delete。这两个调用以版本号作为转入参数，只有当转入参数的版本号与服务器上的版本号一致时调用才会成功。当多个ZooKeeper客户端对同一个znode进行操作时，版本的使用就会显得尤为重要。例如，假设客户端c1对znode/config写入了一些配置信息，如果另一个客户端c2同时更新了这个znode，此时c1的版本号已经过期，c1调用setData一定不会成功。使用版本机制有效避免了以上情况。在这个例子中，c1在写入数据时使用的版本无法匹配，使得操作失败，图2-4描述了这个情况。

![](https://pic.imgdb.cn/item/6130e18644eaada7398805fd.jpg)

## 2.2 ZooKeeper架构

​	ZooKeeper服务器端运行于两种模式下：独立模式（standalone）和仲裁模式（quorum）。独立模式几乎与其术语所描述的一样：有一个单独的服务器，ZooKeeper状态无法复制。在仲裁模式下，具有一组ZooKeeper服务器，我们称为ZooKeeper集合（ZooKeeper ensemble），它们之前可以进行状态的复制，并同时为服务于客户端的请求。从这个角度出发，我们使用术语“ZooKeeper集合”来表示一个服务器设施，这一设施可以由独立模式的一个服务器组成，也可以仲裁模式下的多个服务器组成。

![](https://pic.imgdb.cn/item/6130e1a844eaada73988646b.jpg)

#### 2.2.1 ZooKeeper仲裁

​	在仲裁模式下，ZooKeeper复制集群中的所有服务器的数据树。但如果让一个客户端等待每个服务器完成数据保存后再继续，延迟问题将无法接受。在公共管理领域，法定人数是指进行一项投票所需的立法者的最小数量。而在ZooKeeper中，则是指为了使ZooKeeper工作必须有效运行的服务器的最小数量。这个数字也是服务器告知客户端安全保存数据前，需要保存客户端数据的服务器的最小个数。例如，我们一共有5个ZooKeeper服务器，但法定人数为3个，这样，只要任何3个服务器保存了数据，客户端就可以继续，而其他两个服务器最终也将捕获到数据，并保存数据。

​	选择法定人数准确的大小是一个非常重要的事。法定人数的数量需要保证不管系统发生延迟或崩溃，服务主动确认的任何更新请求需要保持下去，直到另一个请求代替它。

​	为了明白这到底是什么意思，让我们先来通过一个例子来看看，如果法定人数太小，会如何出错。假设有5个服务器并设置法定人数为2，现在服务器s1和s2确认它们需要对一个请求创建的znode/z进行复制，服务返回客户端，指出znode创建完成。现在假设在复制新的znode到其他服务器之前，服务器s1和s2与其他服务器和客户端发生了长时间的分区隔离，整个服务的状态仍然正常，因为基于我们的假设设定法定人数为2，而现在还有3个服务器，但这3个服务器将无法发现新的znode/z。因此，对创建节点/z的请求是非持久化的。

​	这就是第1章中讲述的脑裂场景的例子。为了避免这个问题，这个例子中，法定人数的大小必须至少为3，即集合中5个服务器的多数原则。为了能正常工作，集合中至少要有3个有效的服务器。为了确认一个请求对状态的更新是否成功完成，这个集合同时需要至少3个服务器确认已经完成了数据的复制操作。因此，如果要保证集合可以正常工作，对任何更新操作的成功完成，我们至少要有1个有效的服务器来保存更新的副本（即至少在一个节点上合理的法定人数存在交集）。

​	这就是第1章中讲述的脑裂场景的例子。为了避免这个问题，这个例子中，法定人数的大小必须至少为3，即集合中5个服务器的多数原则。为了能正常工作，集合中至少要有3个有效的服务器。为了确认一个请求对状态的更新是否成功完成，这个集合同时需要至少3个服务器确认已经完成了数据的复制操作。因此，如果要保证集合可以正常工作，对任何更新操作的成功完成，我们至少要有1个有效的服务器来保存更新的副本（即至少在一个节点上合理的法定人数存在交集）。

### 2.2.2 会话

​	在对ZooKeeper集合执行任何请求前，一个客户端必须先与服务建立会话。会话的概念非常重要，对ZooKeeper的运行也非常关键。客户端提交给ZooKeeper的所有操作均关联在一个会话上。当一个会话因某种原因而中止时，在这个会话期间创建的临时节点将会消失。

​	当客户端通过某一个特定语言套件来创建一个ZooKeeper句柄时，它就会通过服务建立一个会话。客户端初始连接到集合中某一个服务器或一个独立的服务器。客户端通过TCP协议与服务器进行连接并通信，但当会话无法与当前连接的服务器继续通信时，会话就可能转移到另一个服务器上。ZooKeeper客户端库透明地转移一个会话到不同的服务器。

​	会话提供了顺序保障，这就意味着同一个会话中的请求会以FIFO（先进先出）顺序执行。通常，一个客户端只打开一个会话，因此客户端请求将全部以FIFO顺序执行。如果客户端拥有多个并发的会话，FIFO顺序在多个会话之间未必能够保持。而即使一个客户端中连贯的会话并不重叠，也未必能够保证FIFO顺序。下面的情况说明如何发生这种问题：

* 客户端建立了一个会话，并通过两个连续的异步调用来创建/tasks和/workers。
* 第一个会话过期。
* 客户端创建另一个会话，并通过异步调用创建/assign。



​	在这个调用顺序中，可能只有/tasks和/assign成功创建了，因为第一个会话保持了FIFO顺序，但在跨会话时就违反了FIFO顺序。

### 2.3 开始使用ZooKeeper

​	开始之前，需要下载ZooKeeper发行包。ZooKeeper作为Apache项目托管到http://zookeeper.apache.org。通过下载链接，你会下载到一个名字类似zookeepe-3.4.5.tar.gz的压缩TAR格式文件。在Linux、Mac OS X或任何其他类UNIX系统上，可以通过一下命令解压缩发行包：

```sh
tar -xvzf zookeeper-3.4.5.tar.gz
```

 	在发行包（distribution）的目录中，你会发现在bin目录中有启动ZooKeeper的脚本。以.sh结尾的脚本运行于UNIX平台（Linux、Mac OS X等），以.cmd结尾的脚本则用于Windows。在conf目录中保存配置文件。lib目录包括Java的JAR文件，它们是运行ZooKeeper所需要的第三方文件。稍后我们需要引用ZooKeeper解压缩的目录。我们以{PATH_TO_ZK}方式来引用该目录

### 2.3.1 第一个ZooKeeper会话

​	首先我们以独立模式运行ZooKeeper并创建一个会话。要做到这一点，使用ZooKeeper发行包中bin/目录下的zkServer和zkCli工具。有经验的管理员常常使用这两个工具来进行调试和管理，同时也非常适合初学者熟悉和了解ZooKeeper。

​	假设你已经下载并解压了ZooKeeper发行包，进入shell，变更目录（cd）到项目根目录下，重命名配置文件：

```sh
mv conf/zoo_sample.cfg conf/zoo.cfg
```

​	虽然是可选的，最好还是把data目录移出/tmp目录，以防止ZooKeeper填满了根分区（root partition）。可以在zoo.cfg文件中修改这个目录的位置。

```
dataDir=/users/me/zookeeper
```

​	最后，为了启动服务器，执行以下命令：

```sh
bin/zkServer.sh start
JMX enabled by default
Using config: ../conf/zoo.cfg
Starting zookeeper ... STARTED
```

​	这个服务器命令使得ZooKeeper服务器在后台中运行。如果在前台中运行以便查看服务器的输出，可以通过以下命令运行：

```sh
bin/zkServer.sh start-foreground
```

​	这个选项提供了大量详细信息的输出，以便允许查看服务器发生了什么。

​	现在我们准备启动客户端。在另一个shell中进入项目根目录，运行以下命令：

```sh
bin/zkCli.sh
.
.
.
<some omitted output>
.
.
.
2012-12-06 12:07:23,545 [myid:] - INFO  [main:ZooKeeper@438] -①
Initiating client connection, connectString=localhost:2181
sessionTimeout=30000 watcher=org.apache.zookeeper.
ZooKeeperMain$MyWatcher@2c641e9a
Welcome to ZooKeeper!
2012-12-06 12:07:23,702 [myid:] - INFO  [main-SendThread②
(localhost:2181):ClientCnxn$SendThread@966] - Opening
socket connection to server localhost/127.0.0.1:2181.
Will not attempt to authenticate using SASL (Unable to
locate a login configuration)
JLine support is enabled
2012-12-06 12:07:23,717 [myid:] - INFO  [main-SendThread③
(localhost:2181):ClientCnxn$SendThread@849] - Socket
connection established to localhost/127.0.0.1:2181, initiating
session [zk: localhost:2181(CONNECTING) 0]
2012-12-06 12:07:23,987 [myid:] - INFO  [main-SendThread④
(localhost:2181):ClientCnxn$SendThread@1207] - Session
establishment complete on server localhost/127.0.0.1:2181,
sessionid = 0x13b6fe376cd0000, negotiated timeout = 30000
WATCHER::
WatchedEvent state:SyncConnected type:None path:null⑤
```

①客户端启动程序来建立一个会话。

②客户端尝试连接到localhost/127.0.0.1：2181。

③客户端连接成功，服务器开始初始化这个新会话。

④会话初始化成功完成。

⑤服务器向客户端发送一个SyncConnected事件。

​	在输出的结尾，我们看到会话建立的日志消息。第一处提到“Initiating client connection.”。消息本身说明到底发生了什么，而额外的重要细节说明了客户端尝试连接到客户端发送的连接串localhost/127.0.0.1：2181中的一个服务器。这个例子中，字符串只包含了localhost，因此指明了具体连接的地址。之后我们看到关于SASL的消息，我们暂时忽略这个消息，随后一个确认信息说明客户端与本地的ZooKeeper服务器建立了TCP连接。后面的日志信息确认了会话的建立，并告诉我们会话ID为：0x13b6fe376cd0000。最后客户端库通过SyncConncted事件通知了应用。应用需要实现Watcher对象来处理这个事件。下一节将详细说明事件。

​	为了更加了解ZooKeeper，让我们列出根（root）下的所有znode，然后创建一个znode。首先我们要确认此刻znode树为空，除了节点/zookeeper之外，该节点内标记了ZooKeeper服务所需的元数据树。

```sh
WATCHER::
WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]
```

现在发生了什么？我们执行ls/后看到这里只有/zookeeper节点。现在我们创建一个名为/workers的znode，确保如下所示：

```sh
WATCHER::
WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0]
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 1] create /workers ""
Created /workers
[zk: localhost:2181(CONNECTED) 2] ls /
[workers, zookeeper]
[zk: localhost:2181(CONNECTED) 3]
```

**注意：Znode数据**

​	当创建/workers节点后，我们指定了一个空字符串（""），说明我们此刻不希望在这个znode中保存数据。然而，该接口中的这个参数可以使我们保存任何字符串到ZooKeeper的节点中。比如，可以替换""为"workers"。

​	为了完成这个练习，删除znode，然后退出：

------

```sh
[zk: localhost:2181(CONNECTED) 3] delete /workers
[zk: localhost:2181(CONNECTED) 4] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 5] quit
Quitting...
2012-12-06 12:28:18,200 [myid:] - INFO  [main-EventThread:ClientCnxn$
EventThread@509] - EventThread shut down
2012-12-06 12:28:18,200 [myid:] - INFO  [main:ZooKeeper@684] - Session:
0x13b6fe376cd0000 closed
```

​	观察到znode/workers已经被删除，并且会话现在也关闭。为了完成最后的清理，退出ZooKeeper服务器：

```sh
bin/zkServer.sh stop
JMX enabled by default
Using config: ../conf/zoo.cfg
Stopping zookeeper ... STOPPED
#
```

### 2.3.2 会话的状态和声明周期

​	会话的生命周期（lifetime）是指会话从创建到结束的时期，无论会话正常关闭还是因超时而导致过期。为了讨论在会话中发生了什么，我们需要考虑会话可能的状态，以及可能导致会话状态改变的事件。

​	一个会话的主要可能状态大多是简单明了的：CONNECTING、CONNECTED、CLOSED和NOT_CONNECTED。状态的转换依赖于发生在客户端与服务之间的各种事件（见图2-6）。

![](https://pic.imgdb.cn/item/6131e9df44eaada739843f8b.jpg)

​	一个会话从NOT_CONNECTED状态开始，当ZooKeeper客户端初始化后转换到CONNECTING状态（图2-6中的箭头1）。正常情况下，成功与ZooKeeper服务器建立连接后，会话转换到CONNECTED状态（箭头2）。当客户端与ZooKeeper服务器断开连接或者无法收到服务器的响应时，它就会转换回CONNECTING状态（箭头3）并尝试发现其他ZooKeeper服务器。如果可以发现另一个服务器或重连到原来的服务器，当服务器确认会话有效后，状态又会转换回CONNECTED状态。否则，它将会声明会话过期，然后转换到CLOSED状态（箭头4）。应用也可以显式地关闭会话（箭头4和箭头5）。

**注意：发生网络分区时等待CONNECTING**

​	如果一个客户端与服务器因超时而断开连接，客户端仍然保持CONNECTING状态。如果因网络分区问题导致客户端与ZooKeeper集合被隔离而发生连接断开，那么其状态将会一直保持，直到显式地关闭这个会话，或者分区问题修复后，客户端能够获悉ZooKeeper服务器发送的会话已经过期。发生这种行为是因为ZooKeeper集合对声明会话超时负责，而不是客户端负责。直到客户端获悉ZooKeeper会话过期，否则客户端不能声明自己的会话过期。然而，客户端可以选择关闭会话。

​	创建一个会话时，你需要设置会话超时这个重要的参数，这个参数设置了ZooKeeper服务允许会话被声明为超时之前存在的时间。如果经过时间t之后服务接收不到这个会话的任何消息，服务就会声明会话过期。而在客户端侧，如果经过t/3的时间未收到任何消息，客户端将向服务器发送心跳消息。在经过2t/3时间后，ZooKeeper客户端开始寻找其他的服务器，而此时它还有t/3时间去寻找。

**注意：客户端会尝试连接哪一个服务器？**

​	在仲裁模式下，客户端有多个服务器可以连接，而在独立模式下，客户端只能尝试重新连接单个服务器。在仲裁模式中，应用需要传递可用的服务器列表给客户端，告知客户端可以连接的服务器信息并选择一个进行连接。

​	当尝试连接到一个不同的服务器时，非常重要的是，这个服务器的ZooKeeper状态要与最后连接的服务器的ZooKeeper状态保持最新。客户端不能连接到这样的服务器：它未发现更新而客户端却已经发现的更新。ZooKeeper通过在服务中排序更新操作来决定状态是否最新。ZooKeeper确保每一个变化相对于所有其他已执行的更新是完全有序的。因此，如果一个客户端在位置i观察到一个更新，它就不能连接到只观察到i'<i的服务器上。在ZooKeeper实现中，系统根据每一个更新建立的顺序来分配给事务标识符。

​	图2-7描述了在重连情况下事务标识符（zkid）的使用。当客户端因超时与s1断开连接后，客户端开始尝试连接s2，但s2延迟于客户端所知的变化。然而，s3对这个变化的情况与客户端保持一致，所以s3可以安全连接。

![](https://pic.imgdb.cn/item/6131eaca44eaada73985e742.jpg)

### 2.3.3 ZooKeeper与仲裁模式

​	到目前为止，我们一直基于独立模式配置的服务器端。如果服务器启动，服务就启动了，但如果服务器故障，整个服务也因此而关闭。这非常不符合可靠的协作服务的承诺。出于可靠性，我们需要运行多个服务器。

​	幸运的是，我们可以在一台机器上运行多个服务器。我们仅仅需要做的便是配置一个更复杂的配置文件。

​	为了让服务器之间可以通信，服务器间需要一些联系信息。理论上，服务器可以使用多播来发现彼此，但我们想让ZooKeeper集合支持跨多个网络而不是单个网络，这样就可以支持多个集合的情况。

```conf
tickTime=2000
initLimit=10
syncLimit=5
dataDir=./data
clientPort=2181
server.1=127.0.0.1:2222:2223
server.2=127.0.0.1:3333:3334
server.3=127.0.0.1:4444:4445
```

​	每一个server.n项指定了编号为n的ZooKeeper服务器使用的地址和端口号。每个server.n项通过冒号分隔为三部分，第一部分为服务器n的IP地址或主机名（hostname），第二部分和第三部分为TCP端口号，分别用于仲裁通信和群首选举。因为我们在同一个机器上运行三个服务器进程，所以我们需要在每一项中使用不同的端口号。通常，我们在不同的服务器上运行每个服务器进程，因此每个服务器项的配置可以使用相同的端口号。

​	我们还需要分别设置data目录，我们可以在命令行中通过以下命令来操作：

```sh
mkdir z1
mkdir z1/data
mkdir z2
mkdir z2/data
mkdir z3
mkdir z3/data
```

​	当启动一个服务器时，我们需要知道启动的是哪个服务器。一个服务器通过读取data目录下一个名为myid的文件来获取服务器ID信息。可以通过以下命令来创建这些文件：

​	当服务器启动时，服务器通过配置文件中的dataDir参数来查找data目录的配置。它通过mydata获得服务器ID，之后使用配置文件中server.n对应的项来设置端口并监听。当在不同的机器上运行ZooKeeper服务器进程时，它们可以使用相同的客户端端口和相同的配置文件。但对于这个例子，在一台服务器上运行，我们需要自定义每个服务器的客户端端口。

​	因此，首先使用本章之前讨论的配置文件，创建z1/z1.cfg。之后通过分别改变客户端端口号为2182和2183，创建配置文件z2/z2.cfg和z3/z3.cfg。

​	现在可以启动服务器，让我们从z1开始：

```sh
cd z1
{PATH_TO_ZK}/bin/zkServer.sh start ./z1.cfg
```

​	服务器的日志记录为zookeeper.out。因为我们只启动了三个ZooKeeper服务器中的一个，所以整个服务还无法运行。在日志中我们将会看到以下形式的记录：

```
... [myid:1] - INFO  [QuorumPeer[myid=1]/...:2181:QuorumPeer@670] - LOOKING
... [myid:1] - INFO  [QuorumPeer[myid=1]/...:2181:FastLeaderElection@740] -
New election. My id =  1, proposed zxid=0x0
... [myid:1] - INFO  [WorkerReceiver[myid=1]:FastLeaderElection@542] -
Notification: 1 ..., LOOKING (my state)
... [myid:1] - WARN  [WorkerSender[myid=1]:QuorumCnxManager@368] - Cannot
open channel to 2 at election address /127.0.0.1:3334
       Java.net.ConnectException: Connection refused
           at java.net.PlainSocketImpl.socketConnect(Native Method)
           at java.net.PlainSocketImpl.doConnect(PlainSocketImpl.java:351)
```

​	这个服务器疯狂地尝试连接到其他服务器，然后失败，如果我们启动另一个服务器，我们可以构成仲裁的法定人数：

```sh
cd z2
{PATH_TO_ZK}/bin/zkServer.sh start ./z2.cfg
```

​	如果我们观察第二个服务器的日志记录zookeeper.out，我们将会看到：

```
... [myid:2] - INFO  [QuorumPeer[myid=2]/...:2182:Leader@345] - LEADING
- LEADER ELECTION TOOK - 279
... [myid:2] - INFO  [QuorumPeer[myid=2]/...:2182:FileTxnSnapLog@240] -
Snapshotting: 0x0 to ./data/version-2/snapshot.0
```

​	该日志指出服务器2已经被选举为群首。如果我们现在看看服务器1的日志，我们会看到：

```
... [myid:1] - INFO  [QuorumPeer[myid=1]/...:2181:QuorumPeer@738] -
FOLLOWING
... [myid:1] - INFO  [QuorumPeer[myid=1]/...:2181:ZooKeeperServer@162] -
Created server ...
... [myid:1] - INFO  [QuorumPeer[myid=1]/...:2181:Follower@63] - FOLLOWING
- LEADER ELECTION TOOK - 212
```

​	服务器1作为服务器2的追随者被激活。我们现在具有了符合法定仲裁（三分之二）的可用服务器。

​	在此刻服务开始可用。我们现在需要配置客户端来连接到服务上。连接字符串需要列出所有组成服务的服务器host：port对。对于这个例子，连接串为"127.0.0.1：2181，127.0.0.1：2182，127.0.0.1：2183"（我们包含第三个服务器的信息，即使我们永远不启动它，因为这可以说明ZooKeeper一些有用的属性）。

​	我们使用zkCli.sh来访问集群：

```
{PATH_TO_ZK}/bin/zkCli.sh -server 
127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
```

​	当连接到服务器后，我们会看到以下形式的消息：

​	

```
[myid:] - INFO  [...] - Session establishment
complete on server localhost/127.0.0.1:2182 ...
```

​	注意日志消息中的端口号，在本例中的2182。如果通过Ctrl-C来停止客户端并重启多次它，我们将会看到端口号在218102182之间来回变化。我们也许还会注意到尝试2183端口后连接失败的消息，之后为成功连接到某一个服务器端口的消息。

**注意：简单的负载均衡**

​	客户端以随机顺序连接到连接串中的服务器。这样可以用ZooKeeper来实现一个简单的负载均衡。不过，客户端无法指定优先选择的服务器来进行连接。例如，如果我们有5个ZooKeeper服务器的一个集合，其中3个在美国西海岸，另外两个在美国东海岸，为了确保客户端只连接到本地服务器上，我们可以使在东海岸客户端的连接串中只出现东海岸的服务器，在西海岸客户端的连接串中只有西海岸的服务器。

​	这个连接尝试说明如何通过运行多个服务器来达到可靠性（当然，在生产环境中，你需要在不同的主机上进行这些操作）。对于本书大部分，包括后续几章，我们一直以独立模式的服务器进行开发，因为启动和管理多个服务器非常简单，实现这个例子也非常简单。除了连接串外，客户端不用关心ZooKeeper服务由多少个服务器组成，这也是ZooKeeper的优点之一。

### 2.3.4 实现一个原语：通过ZooKeeper实现锁

​	关于ZooKeeper的功能，一个简单的例子就是通过锁来实现临界区域。我们知道有很多形式的锁（如：读/写锁、全局锁），通过ZooKeeper来实现锁也有多种方式。这里讨论一个简单的方式来说明应用中如何使用ZooKeeper，我们不再考虑其他形式的锁。

​	假设有一个应用由n个进程组成，这些进程尝试获取一个锁。再次强调，ZooKeeper并未直接暴露原语，因此我们使用ZooKeeper的接口来管理znode，以此来实现锁。为了获得一个锁，每个进程p尝试创建znode，名为/lock。如果进程p成功创建了znode，就表示它获得了锁并可以继续执行其临界区域的代码。不过一个潜在的问题是进程p可能崩溃，导致这个锁永远无法释放。在这种情况下，没有任何其他进程可以再次获得这个锁，整个系统可能因死锁而失灵。为了避免这种情况，我们不得不在创建这个节点时指定/lock为临时节点。

​	其他进程因znode存在而创建/lock失败。因此，进程监听/lock的变化，并在检测到/lock删除时再次尝试创建节点来获得锁。当收到/lock删除的通知时，如果进程p还需要继续获取锁，它就继续尝试创建/lock的步骤，如果其他进程已经创建了，就继续监听节点。

## 2.4 一个主-从模式例子的实现

主-从模式的模型中包括三个角色：

**主节点**

​	主节点负责监视新的从节点和任务，分配任务给可用的从节点。

**从节点**

​	从节点会通过系统注册自己，以确保主节点看到它们可以执行任务，然后开始监视新任务。

**客户端**

​	客户端创建新任务并等待系统的响应。

​	现在探讨这些不同的角色以及每个角色需要执行的确切步骤。

### 2.4.1 主节点角色

​	因为只有一个进程会成为主节点，所以一个进程成为ZooKeeper的主节点后必须锁定管理权。为此，进程需要创建一个临时znode，名为/master：

```sh
[zk: localhost:2181(CONNECTED) 0] create -e /master "master1.example.com:2223"①
Created /master
[zk: localhost:2181(CONNECTED) 1] ls /②
[master, zookeeper]
[zk: localhost:2181(CONNECTED) 2] get /master③
"master1.example.com:2223"
cZxid = 0x67
ctime = Tue Dec 11 10:06:19 CET 2012
mZxid = 0x67
mtime = Tue Dec 11 10:06:19 CET 2012
pZxid = 0x67
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x13b891d4c9e0005
dataLength = 26
numChildren = 0
[zk: localhost:2181(CONNECTED) 3]
```

①创建主节点的znode，以便获得管理权。使用-e标志来表示创建的znode为临时性的。

②列出ZooKeeper树的根。

③获取/master znode的元数据和数据。

​	刚刚发生了什么？首先创建一个临时znode/master。我们在znode中添加了主机信息，以便ZooKeeper外部的其他进程需要与它通信。添加主机信息并不是必需的，但这样做仅仅是为了说明我们可以在需要时添加数据。为了设置znode为临时性的，需要添加-e标志。记得，一个临时节点会在会话过期或关闭时自动被删除。

​	现在让我们看下我们使用两个进程来获得主节点角色的情况，尽管在任何时刻最多只能有一个活动的主节点，其他进程将成为备份主节点。假如其他进程不知道已经有一个主节点被选举出来，并尝试创建一个/master节点。让我们看看会发生什么：

```sh
[zk: localhost:2181(CONNECTED) 0] create -e /master 
"master2.example.com:2223"
Node already exists: /master
[zk: localhost:2181(CONNECTED) 1]
```

​	ZooKeeper告诉我们一个/master节点已经存在。这样，第二个进程就知道已经存在一个主节点。然而，一个活动的主节点可能会崩溃，备份主节点需要接替活动主节点的角色。为了检测到这些，需要在/master节点上设置一个监视点，操作如下：

```sh
[zk: localhost:2181(CONNECTED) 0] create -e /master "master2.example.com:2223"
Node already exists: /master
[zk: localhost:2181(CONNECTED) 1] stat /master true
cZxid = 0x67
ctime = Tue Dec 11 10:06:19 CET 2012
mZxid = 0x67
mtime = Tue Dec 11 10:06:19 CET 2012
pZxid = 0x67
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x13b891d4c9e0005
dataLength = 26
numChildren = 0
[zk: localhost:2181(CONNECTED) 2]
```

​	stat命令可以得到一个znode节点的属性，并允许我们在已经存在的znode节点上设置监视点。通过在路径后面设置参数true来添加监视点。当活动的主节点崩溃时，我们会观察到以下情况：

```sh
[zk: localhost:2181(CONNECTED) 0] create -e /master "master2.example.com:2223"
Node already exists: /master
[zk: localhost:2181(CONNECTED) 1] stat /master true
cZxid = 0x67
ctime = Tue Dec 11 10:06:19 CET 2012
mZxid = 0x67
mtime = Tue Dec 11 10:06:19 CET 2012
pZxid = 0x67
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x13b891d4c9e0005
dataLength = 26
numChildren = 0
[zk: localhost:2181(CONNECTED) 2]
WATCHER::
WatchedEvent state:SyncConnected type:NodeDeleted path:/master
[zk: localhost:2181(CONNECTED) 2] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 3]
```

​	在输出的最后，我们注意到NodeDeleted事件。这个事件指出活动主节点的会话已经关闭或过期。同时注意，/master节点已经不存在了。现在备份主节点通过再次创建/master节点来成为活动主节点。

​	因为备份主节点成功创建了/master节点，所以现在客户端开始成为活动主节点。

#### 2.4.2 从节点、任务和分配

​	在我们讨论从节点和客户端所采取的步骤之前，让我们先创建三个重要的父znode，/workers、/tasks和/assign：

```
[zk: localhost:2181(CONNECTED) 0] create /workers ""
Created /workers
[zk: localhost:2181(CONNECTED) 1] create /tasks ""
Created /tasks
[zk: localhost:2181(CONNECTED) 2] create /assign ""
Created /assign
[zk: localhost:2181(CONNECTED) 3] ls /
[assign, tasks, workers, master, zookeeper]
[zk: localhost:2181(CONNECTED) 4]
```

​	这三个新的znode为持久性节点，且不包含任何数据。本例中，通过使用这些znode可以告诉我们哪个从节点当前有效，还告诉我们当前有任务需要分配，并向从节点分配任务。

​	在真实的应用中，这些znode可能由主进程在分配任务前创建，也可能由一个引导程序创建，不管这些节点是如何创建的，一旦这些节点存在了，主节点就需要监视/workers和/tasks的子节点的变化情况：

```
[zk: localhost:2181(CONNECTED) 4] ls /workers true
[]
[zk: localhost:2181(CONNECTED) 5] ls /tasks true
[]
[zk: localhost:2181(CONNECTED) 6]
```

​	请注意，在主节点上调用stat命令前，我们使用可选的true参数调用ls命令。通过true这个参数，可以设置对应znode的子节点变化的监视点。

#### 2.4.3 从节点角色

​	从节点首先要通知主节点，告知从节点可以执行任务。从节点通过在/workers子节点下创建临时性的znode来进行通知，并在子节点中使用主机名来标识自己：

```
[zk: localhost:2181(CONNECTED) 0] create -e 
/workers/worker1.example.com
                                            "worker1.example.com:2224"
Created /workers/worker1.example.com
[zk: localhost:2181(CONNECTED) 1]
```

​	注意，输出中，ZooKeeper确认znode已经创建。之前主节点已经监视了/workers的子节点变化情况。一旦从节点在/workers下创建了一个znode，主节点就会观察到以下通知信息：

```
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged 
path:/workers
```

​	下一步，从节点需要创建一个父znode/assing/worker1.example.com来接收任务分配，并通过第二个参数为true的ls命令来监视这个节点的变化，以便等待新的任务

```sh
[zk: localhost:2181(CONNECTED) 0] create -e /workers/worker1.example.com
                                            "worker1.example.com:2224"
Created /workers/worker1.example.com
[zk: localhost:2181(CONNECTED) 1] create /assign/worker1.example.com ""
Created /assign/worker1.example.com
[zk: localhost:2181(CONNECTED) 2] ls /assign/worker1.example.com true
[]
[zk: localhost:2181(CONNECTED) 3]
```

​	从节点现在已经准备就绪，可以接收任务分配。之后，我们通过讨论客户端角色来看一下任务分配的问题。

### 2.4.4 客户端角色

​	客户端向系统中添加任务。在本示例中具体任务是什么并不重要，我们假设客户端请求主从系统来运行cmd命令。为了向系统添加一个任务，客户端执行以下操作：

```sh
[zk: localhost:2181(CONNECTED) 0] create -s /tasks/task- "cmd"
Created /tasks/task-0000000000
```

​	我们需要按照任务添加的顺序来添加znode，其本质上为一个队列。客户端现在必须等待任务执行完毕。执行任务的从节点将任务执行完毕后，会创建一个znode来表示任务状态。客户端通过查看任务状态的znode是否创建来确定任务是否执行完毕，因此客户端需要监视状态znode的创建事件：

```
[zk: localhost:2181(CONNECTED) 1] ls /tasks/task-0000000000 true
[]
[zk: localhost:2181(CONNECTED) 2]
```

​	执行任务的从节点会在/tasks/task-0000000000节点下创建状态znode节点，所以我们需要用ls命令来监视/tasks/task-0000000000的子节点。

​	一旦创建任务的znode，主节点会观察到以下事件：

```sh
[zk: localhost:2181(CONNECTED) 6]
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/tasks
```

​	主节点之后会检查这个新的任务，获取可用的从节点列表，之后分配这个任务给worker1.example.com：

```sh
[zk: 6] ls /tasks
[task-0000000000]
[zk: 7] ls /workers
[worker1.example.com]
[zk: 8] create /assign/worker1.example.com/task-0000000000 ""
Created /assign/worker1.example.com/task-0000000000
[zk: 9]
```

​	从节点接收到新任务分配的通知：

```sh
[zk: localhost:2181(CONNECTED) 3]
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged
path:/assign/worker1.example.com
```

​	从节点之后便开始检查新任务，并确认该任务是否分配给自己：

```sh
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged
path:/assign/worker1.example.com
[zk: localhost:2181(CONNECTED) 3] ls /assign/worker1.example.com
[task-0000000000]
[zk: localhost:2181(CONNECTED) 4]
```

​	一旦从节点完成任务的执行，它就会在/tasks中添加一个状态znode：

```sh
[zk: localhost:2181(CONNECTED) 4] create /tasks/task-0000000000/status "done"
Created /tasks/task-0000000000/status
[zk: localhost:2181(CONNECTED) 5]
```

​	之后，客户端接收到通知，并检查执行结果：

```sh
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged
path:/tasks/task-0000000000
[zk: localhost:2181(CONNECTED) 2] get /tasks/task-0000000000
"cmd"
cZxid = 0x7c
ctime = Tue Dec 11 10:30:18 CET 2012
mZxid = 0x7c
mtime = Tue Dec 11 10:30:18 CET 2012
pZxid = 0x7e
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 1
[zk: localhost:2181(CONNECTED) 3] get /tasks/task-0000000000/status
"done"
cZxid = 0x7e
ctime = Tue Dec 11 10:42:41 CET 2012
mZxid = 0x7e
mtime = Tue Dec 11 10:42:41 CET 2012
pZxid = 0x7e
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 8
numChildren = 0
[zk: localhost:2181(CONNECTED) 4]
```

​	客户端检查状态znode的信息，并确认任务的执行结果。本例中，我们看到任务成功执行，其状态为“done”。当然任务也可能非常复杂，甚至涉及另一个分布式系统。最终不管是什么样的任务，执行任务的机制与通过ZooKeeper来传递结果，本质上都是一样的。

# 3. 开始使用ZooKeeper的API

## 3.1 设置ZooKeeper的CLASSPATH

​	我们需要设置正确的classpath，以便运行或编译ZooKeeper的Java代码。除了ZooKeeper的JAR包外，ZooKeeper使用了大量的第三方库。为了简化输入和方便阅读，我们使用环境变量CLASSPATH来表示所有必需的库。ZooKeeper发行包中bin目录下的zkEnv.sh脚本会为我们设置该环境变量。我们需要使用以下方式来编码：

```
ZOOBINDIR="<path_to_distro>/bin"
. "$ZOOBINDIR"/zkEnv.sh
```

​	一旦运行这个脚本，环境变量CLASSPATH就会正确设置。我们在编译和运行Java程序时用到它。

## 3.2 建立ZooKeeper会话

​	ooKeeper的API围绕ZooKeeper的句柄（handle）而构建，每个API调用都需要传递这个句柄。这个句柄代表与ZooKeeper之间的一个会话。在图3-1中，与ZooKeeper服务器已经建立的一个会话如果断开，这个会话就会迁移到另一台ZooKeeper服务器上。只要会话还存活着，这个句柄就仍然有效，ZooKeeper客户端库会持续保持这个活跃连接，以保证与ZooKeeper服务器之间的会话存活。如果句柄关闭，ZooKeeper客户端库会告知ZooKeeper服务器终止这个会话。如果ZooKeeper发现客户端已经死掉，就会使这个会话无效。如果客户端之后尝试重新连接到ZooKeeper服务器，使用之前无效会话对应的那个句柄进行连接，那么ZooKeeper服务器会通知客户端库，这个会话已失效，使用这个句柄进行的任何操作都会返回错误。

![](https://pic.imgdb.cn/item/6135ab2544eaada7394746c0.jpg)

​	创建ZooKeeper句柄的构造函数如下所示：

```java
ZooKeeper(
    String connectString,
    int sessionTimeout,
    Watcher watcher)
```

其中：

**connectString**

包含主机名和ZooKeeper服务器的端口。我们之前通过zkCli连接ZooKeeper服务时，已经列出过这些服务器。

**sessionTimeout**

​	以毫秒为单位，表示ZooKeeper等待客户端通信的最长时间，之后会声明会话已死亡。目前我们使用15000，即15秒。这就是说如果ZooKeeper与客户端有15秒的时间无法进行通信，ZooKeeper就会终止客户端的会话。需要注意，这个值比较高，但对于我们后续的实验会非常有用。ZooKeeper会话一般设置超时时间为5~10秒。

**watcher**

​	用于接收会话事件的一个对象，这个对象需要我们自己创建。因为Wacher定义为接口，所以我们需要自己实现一个类，然后初始化这个类的实例并传入ZooKeeper的构造函数中。客户端使用Watcher接口来监控与ZooKeeper之间会话的健康情况。与ZooKeeper服务器之间建立或失去连接时就会产生事件。它们同样还能用于监控ZooKeeper数据的变化。最终，如果与ZooKeeper的会话过期，也会通过Watcher接口传递事件来通知客户端的应用。

### 3.2.1 实现一个Watcher

为了从ZooKeeper接收通知，我们需要实现监视点。首先让我们进一步了解Watcher接口，该接口的定义如下：

------

```java
public interface Watcher {
    void process(WatchedEvent event);
}
```

------

这个接口没有多少内容，我们不得不自己实现，但现在我们只是简单地输出事件。所以，让我们从一个名为Master的类开始实现示例：

------

```java
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.Watcher;
public class Master implements Watcher {
    ZooKeeper zk;
    String hostPort;
    Master(String hostPort) {
        this.hostPort = hostPort;①
    }
    void startZK() {
        zk = new ZooKeeper(hostPort, 15000, this);②
    }
    public void process(WatchedEvent e) {
        System.out.println(e);
    }
    public static void main(String args[])
        throws Exception {
        Master m = new Master(args[0]);
        m.startZK();
        // wait for a bit
        Thread.sleep(60000);
    }
}
```

​	①在构造函数中，我们并未实例化ZooKeeper对象，而是先保存hostPort留着后面使用。Java最佳实践告诉我们，一个对象的构造函数没有完成前不要调用这个对象的其他方法。因为这个对象实现了Watcher，并且当我们实例化ZooKeeper对象时，其Watcher的回调函数就会被调用，所以我们需要Master的构造函数返回后再调用ZooKeeper的构造函数。

​	②使用Master对象来构造ZooKeeper对象，以便添加Watcher的回调函数。

​	③这个简单的示例没有提供复杂的事件处理逻辑，而只是将我们收到的事件进行简单的输出。

​	④我们连接到ZooKeeper后，后台就会有一个线程来维护这个ZooKeeper会话。该线程为守护线程，也就是说线程即使处于活跃状态，程序也可以退出。因此我们在程序退出前休眠一段时间，以便我们可以看到事件的发生。

​	①在构造函数中，我们并未实例化ZooKeeper对象，而是先保存hostPort留着后面使用。Java最佳实践告诉我们，一个对象的构造函数没有完成前不要调用这个对象的其他方法。因为这个对象实现了Watcher，并且当我们实例化ZooKeeper对象时，其Watcher的回调函数就会被调用，所以我们需要Master的构造函数返回后再调用ZooKeeper的构造函数。

​	②使用Master对象来构造ZooKeeper对象，以便添加Watcher的回调函数。

​	③这个简单的示例没有提供复杂的事件处理逻辑，而只是将我们收到的事件进行简单的输出。

​	④我们连接到ZooKeeper后，后台就会有一个线程来维护这个ZooKeeper会话。该线程为守护线程，也就是说线程即使处于活跃状态，程序也可以退出。因此我们在程序退出前休眠一段时间，以便我们可以看到事件的发生。

通过以下方式就可以编译这个简单的例子：

------

```
$ javac  -cp $CLASSPATH Master.java
```

------

当我们编译完Master.java这个文件后，运行它并查看结果：

------

```
$ java  -cp $CLASSPATH Master 127.0.0.1:2181
... - INFO  [...] - Client environment:zookeeper.version=3.4.5-1392090, ...①
...
... - INFO  [...] - Initiating client connection,
connectString=127.0.0.1:2181 ...②
... - INFO  [...] - Opening socket connection to server
localhost/127.0.0.1:2181. ...
... - INFO  [...] - Socket connection established to localhost/127.0.0.1:2181,
initiating session
... - INFO  [...] - Session establishment complete on server
localhost/127.0.0.1:2181, ...③
WatchedEvent state:SyncConnected type:None path:null④
```

------

ZooKeeper客户端API产生很多日志消息，使用户可以了解发生了什么。日志非常详细，可以通过配置文件来禁用这么详细的日志，不过在开发时，这些消息非常有用，甚至在正式部署后，发生了某些我们不希望发生的事情，这些日志消息也是非常有价值的。

①前面几行日志消息描述了ZooKeeper客户端的实现和环境。

②当客户端初始化一个到ZooKeeper服务器的连接时，无论是最初的连接还是随后的重连接，都会产生这些日志消息。

③这个消息展示了连接建立之后，该连接的信息，其中包括客户端所连接的主机和端口信息，以及这个会话与服务器协商的超时时间。如果服务器发现请求的会话超时时间太短或太长，服务器会调整会话超时时间。

④最后这行并不是ZooKeeper库所输出，而是我们实现的Watcher.process（WatchedEvent e）函数中输出的WatchEvent对象。

这个例子中，假设设运行时所有必需的库均在lib子目录下，同时假设log4j.conf文件在conf子目录中。你可以在你所使用的ZooKeeper发行包中找到这两个目录。如果你看到以下信息：

------

```
log4j:WARN No appenders could be found for logger
      (org.apache.zookeeper.ZooKeeper).
log4j:WARN Please initialize the log4j system properly.
```

------

表示你还没有将log4j.conf放到classpath下。

### 3.2.2　运行Watcher的示例

​	如果我们不启动ZooKeeper服务就启动主节点，这样会发生什么呢？我们可以试一下。停止服务，然后运行Master，看到了什么？在之前输出中的最后一行，即WatchedEvent的数据，现在并没有出现。因为ZooKeeper库无法连接ZooKeeper服务器，所以我们看不到这些信息了。

​	现在我们启动服务器，然后运行Master，之后停止服务器并保持Master继续运行。你会看到在SyncConnected事件之后发生了Disconnected事件。

​	当开发者看到Disconnected事件时，有些人认为需要创建一个新的ZooKeeper句柄来重新连接服务。不要这么做！当你启动服务器，然后启动Master，再重启服务器时看一下发生了什么。你看到SyncConnected事件之后为Disconnected事件，然后又是一个SyncConnected事件。ZooKeeper客户端库负责为你重新连接服务。当不幸遇到网络中断或服务器故障时，ZooKeeper可以处理这些故障问题。

​	我们需要知道ZooKeeper本身也可能发生这些故障问题。一个ZooKeeper服务器也许会故障或失去网络连接，类似我们停止主节点后所模拟的场景。如果ZooKeeper服务至少由三台服务器组成，那么一个服务器的故障并不会导致服务中断。而客户端也会很快收到Disconnected事件，之后便为SyncConnected事件。

**注意：ZooKeeper管理连接**

​	请不要自己试着去管理ZooKeeper客户端连接。ZooKeeper客户端库会监控与服务之间的连接，客户端库不仅告诉我们连接发生问题，还会主动尝试重新建立通信。一般客户端开发库会很快重建会话，以便最小化应用的影响。所以不要关闭会话后再启动一个新的会话，这样会增加系统负载，并导致更长时间的中断。

​	客户端就像除了休眠外什么都没做一样，而我们通过发生的事件可以看到后台到底发生了什么。我们还可以看看ZooKeeper服务端都发生了什么。ZooKeeper有两种管理接口：JMX和四字母组成的命令。第10章会深入讨论这些接口，现在我们通过stat和dump这两个四字母命令来看看服务器上发生了什么。

​	要使用这些命令，需要先通过telnet连接到客户端端口2181，然后输入这些命令（在命令后输入Enter键）。例如，如果启动Master这个程序后，使用stat命令，我们会看到以下输出信息：

------

```
$ telnet 127.0.0.1 2181
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
stat
ZooKeeper version: 3.4.5-1392090, built on 09/30/2012 17:52 GMT
Clients:
 /127.0.0.1:39470[1](queued=0,recved=3,sent=3)
 /127.0.0.1:39471[0](queued=0,recved=1,sent=0)
Latency min/avg/max: 0/5/48
Received: 34
Sent: 33
Connections: 2
Outstanding: 0
Zxid: 0x17
Mode: standalone
Node count: 4
Connection closed by foreign host.
```

------

​	我们从输出信息看到有两个客户端连接到ZooKeeper服务器。一个是Master程序，另一个为Telnet连接。

​	如果我们启动Master程序后，使用dump命令，我们会看到以下输出信息：

------

```
$ telnet 127.0.0.1 2181
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
dump
SessionTracker dump:
Session Sets (3):
0 expire at Wed Nov 28 20:34:00 PST 2012:
0 expire at Wed Nov 28 20:34:02 PST 2012:
1 expire at Wed Nov 28 20:34:04 PST 2012:
        0x13b4a4d22070006
ephemeral nodes dump:
Sessions with Ephemerals (0):
Connection closed by foreign host.
```

------

我们从输出信息中看到有一个活动的会话，这个会话属于Master程序。我们还能看到这个会话还有多长时间会过期。会话超时的过期时间取决于我们创建ZooKeeper对象时所指定的值。

让我结束Master程序，再次使用dump命令来看一下活动的会话信息。你会注意到会话过一段时间后才消失。这是因为直到会话超时时间过了以后，服务器才会结束这个会话。当然，客户端会不断延续与ZooKeeper服务器的活动连接的到期时间。

当Master结束时，最好的方式是使会话立即消失。这可以通过ZooKeeper.close（）方法来结束。一旦调用close方法后，ZooKeeper对象实例所表示的会话就会被销毁。

让我们在示例程序中加入close调用：

------

```
    void stopZK() throws Exception { zk.close(); }
    public static void main(String args[]) throws Exception {
        Master m = new Master(args[0]);
        m.startZK();
        // wait for a bit
        Thread.sleep(60000);
        m.stopZK();
    }
```

------

​	现在我们可以再次运行Master程序，运行dump命令来看一下会话是否还存活。因为Master程序中显示关闭了会话，所以ZooKeeper关闭会话前就不需要等待会话超时了。

## 3.3  获取管理权

​	现在我们有了会话，我们的Master程序需要获得管理权，虽然现在我们只有一个主节点，但我们还是要小心仔细。我们需要运行多个进程，以便在活动主节点发生故障后，可以有进程接替主节点。

​	为了确保同一时间只有一个主节点进程出于活动状态，我们使用ZooKeeper来实现简单的群首选举算法（在2.4.1节中所描述的）。这个算法中，所有潜在的主节点进程尝试创建/master节点，但只有一个成功，这个成功的进程成为主节点。

​	常量ZooDefs.Ids.OPEN_ACL_UNSAFE为所有人提供了所有权限（正如其名所显示的，这个ACL策略在不可信的环境下使用是非常不安全的）。

​	ZooKeeper通过插件式的认证方法提供了每个节点的ACL策略功能，因此，如果我们需要，就可以限制某个用户对某个znode节点的哪些权限，但对于这个简单的例子，我们继续使用OPEN_ACL_UNSAFE策略。当然，我们希望在主节点死掉后/master节点会消失。正如我们在2.1.2节中所提到的持久性和临时性znode节点，我们可以使用ZooKeeper的临时性znode节点来达到我们的目的。我们将定义一个EPHEMERAL的znode节点，当创建它的会话关闭或无效时，ZooKeeper会自动检测到，并删除这个节点。

​	因此，我们将会在我们的程序中添加以下代码：

------

```java
    String serverId = Integer.toHexString(random.nextInt());
    void runForMaster() {
        zk.create("/master",①
        	serverId.getBytes(),②
        	OPEN_ACL_UNSAFE,③
        	CreateMode.EPHEMERAL);④
    }
```

------

​	①我们试着创建znode节点/master。如果这个znode节点存在，create就会失败。同时我们想在/master节点的数据字段保存对应这个服务器的唯一ID。

​	②数据字段只能存储字节数组类型的数据，所以我们将int型转换为一个字节数组。

​	③如之前所提到的，我们使用开放的ACL策略。

​	④我们创建的节点类型为EPHEMERAL。

​	然而，我们这样做还不够，create方法会抛出两种异常：KeeperException和InterruptedException。我们需要确保我们处理了这两种异常，特别是ConnectionLossException（KeeperException异常的子类）和InterruptedException。对于其他异常，我们可以忽略并继续执行，但对于这两种异常，create方法可能已经成功了，所以如果我们作为主节点就需要捕获并处理它们。

​	ConnectionLossException异常发生于客户端与ZooKeeper服务端失去连接时。一般常常由于网络原因导致，如网络分区或ZooKeeper服务器故障。当这个异常发生时，客户端并不知道是在ZooKeeper服务器处理前丢失了请求消息，还是在处理后客户端未收到响应消息。如我们之前所描述的，ZooKeeper的客户端库将会为后续请求重新建立连接，但进程必须知道一个未决请求是否已经处理了还是需要再次发送请求。

​	InterruptedException异常源于客户端线程调用了Thread.interrupt，通常这是因为应用程序部分关闭，但还在被其他相关应用的方法使用。从字面来看这个异常，进程会中断本地客户端的请求处理的过程，并使该请求处于未知状态。

​	这两种请求都会导致正常请求处理过程的中断，开发者不能假设处理过程中的请求的状态。当我们处理这些异常时，开发者在处理前必须知道系统的状态。如果发生群首选举，在我们没有确认情况之前，我们不希望确定主节点。如果create执行成功了，活动主节点死掉以前，没有任何进程能够成为主节点，如果活动主节点还不知道自己已经获得了管理权，不会有任何进程成为主节点进程。

​	当处理ConnectionLossException异常时，我们需要找出那个进程创建的/master节点，如果进程是自己，就开始成为群首角色。我们通过getData方法来处理：

------

```java
byte[] getData(
    String path,
    bool watch,
    Stat stat)
```

------

其中：

**path**

​	类似其他ZooKeeper方法一样，第一个参数为我们想要获取数据的znode节点路径。

**watch**

​	表示我们是否想要监听后续的数据变更。如果设置为true，我们就可以通过我们创建ZooKeeper句柄时所设置的Watcher对象得到事件，同时另一个版本的方法提供了以Watcher对象为入参，通过这个传入的对象来接收变更的事件。我们在后续章节再讨论如何监视变更情况，现在我们设置这个参数为false，因为我们现在我们只想知道当前的数据是什么。

**stat**

​	最后一个参数类型Stat结构，getData方法会填充znode节点的元数据信息。

**返回值**

​	方法返回成功（没有抛出异常），就会得到znode节点数据的字节数组。

让我们按以下代码段来修改代码，在runForMaster方法中引入异常处理：

------

```java
    String serverId = Integer.toString(Random.nextLong());
    boolean isLeader = false;
    // returns true if there is a master
    boolean checkMaster() {
        while (true) {
            try {
                Stat stat = new Stat();
                byte data[] = zk.getData("/master", false, stat);①
                isLeader = new String(data).equals(serverId));②
                return true;
            } catch (NoNodeException e) {
                // no master, so try create again
                return false;
            } catch (ConnectionLossException e) {
            }
        }
    }
    void runForMaster() throws InterruptedException {③
        while (true) {
            try {④
                zk.create("/master", serverId.getBytes(),
                          OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);⑤
                isLeader = true;
                break;
            } catch (NodeExistsException e) {
                isLeader = false;
                break;
            } catch (ConnectionLossException e) {⑥
            }
            if (checkMaster()) break;⑦
        }
    }
```

------

​	①通过获取/master节点的数据来检查活动主节点。

​	②该行展示了为什么我们需要使用在创建/master节点时保存的数据：如果/master存在，我们使用/master中的数据来确定谁是群首。如果一个进程捕获到ConnectionLossException，这个进程可能就是主节点，因create操作实际上已经处理完，但响应消息却丢失了。

​	③我们将InterruptedException异常简单地传递给调用者。

​	④我们将zk.create方法包在try块之中，以便我们捕获并处理ConnectionLossException异常。

​	⑤这里为create请求，如果成功执行将会成为主节点。

​	⑥处理ConnectionLossException异常的catch块的代码为空，因为我们并不想中止函数，这样就可以使处理过程继续向下执行。

​	⑦检查活动主节点是否存在，如果不存在就重试。

​	在这个例子中，我们简单地传递InterruptedException给调用者，即向上传递异常。不过，在Java中没有明确的指导方针告诉我们如何处理线程中断，甚至没有告诉我们这个中断代表什么。有些时候，中断用于通知线程现在要退出了，需要进行清理操作，另外的情况，中断用于获得一个线程的控制权，应用的执行还将继续。

​	InterruptedException异常的处理依赖于程序的上下文环境，如果向上抛出InterruptedException异常，最终关闭zk句柄，我们可以抛出异常到调用栈顶，当句柄关闭时就可以清理所有一切。如果zk句柄未关闭，在重新抛出异常前，我们需要弄清楚自己是不是主节点，或者继续异步执行后续操作。后者情况非常棘手，需要我们仔细设计并妥善处理。

​	现在，我们看一下Master的main主函数：

------

```java
    public static void main(String args[]) throws Exception {
        Master m = new Master(args[0]);
        m.startZK();
        m.runForMaster();①
        if (isLeader) {
            System.out.println("I'm the leader");②
            // wait for a bit
            Thread.sleep(60000);
        } else {
            System.out.println("Someone else is the leader");
        }
        m.stopZK();
    }
```

------

​	①调用我们之前实现的runForMaster函数，当前进程成为主节点或另一进程成为主节点后返回。

​	②当我们开发主节点的应用逻辑时，我们在此处开始执行这些逻辑，现在我们仅仅输出我们成为主节点的信息，然后等待60秒后退出main函数。

​	因为我们并没有直接处理InterruptedException异常，如果发生该异常，我们的进程将会简单地退出，主节点也不会在退出前进行其他操作。在下一章，主节点开始管理系统队列中的任务，但现在我们先继续完善其他组件。

### 3.3.1 异步获取管理权

​	ZooKeeper中，所有同步调用方法都有对应的异步调用方法。通过异步调用，我们可以在单线程中同时进行多个调用，同时也可以简化我们的实现方式。让我们回顾管理权的例子，修改为异步调用的方式。

​	以下为create方法的异步调用版本：

------

```
void create(String path,
    byte[] data,
    List<ACL> acl,
    CreateMode createMode,
    AsyncCallback.StringCallback cb,①
    Object ctx)②
```

------

​	create方法的异步方法与同步方法非常相似，仅仅多了两个参数：

​	①提供回调方法的对象。

​	②用户指定上下文信息（回调方法调用是传入的对象实例）。

​	该方法调用后通常在create请求发送到服务端之前就会立即返回。回调对象通过传入的上下文参数来获取数据，当从服务器接收到create请求的结果时，上下文参数就会通过回调对象提供给应用程序。

​	注意，该create方法不会抛出异常，我们可以简化处理，因为调用返回前并不会等待create命令完成，所以我们无需关心InterruptedException异常；同时因请求的所有错误信息通过回调对象会第一个返回，所以我们也无需关心KeeperException异常。

​	回调对象实现只有一个方法的StringCallback接口：

------

```java
void processResult(int rc, String path, Object ctx, String name)
```

------

​	异步方法调用会简单化队列对ZooKeeper服务器的请求，并在另一个线程中传输请求。当接收到响应信息，这些请求就会在一个专用回调线程中被处理。为了保持顺序，只会有一个单独的线程按照接收顺序处理响应包。

​	processResult各个参数的含义如下：

**rc**

​	返回调用的结构，返回OK或与KeeperException异常对应的编码值。

**path**

​	我们传给create的path参数值。

**ctx**

​	我们传给create的上下文参数。

**name**

​	创建的znode节点名称。

​	目前，调用成功后，path和name的值一样，但是，如果采用CreateMode.SEQUENTIAL模式，这两个参数值就不会相等。

**注意：回调函数处理**

​	因为只有一个单独的线程处理所有回调调用，如果回调函数阻塞，所有后续回调调用都会被阻塞，也就是说，一般不要在回调函数中集中操作或阻塞操作。有时，在回调函数中调用同步方法是合法的，但一般还是避免这样做，以便后续回调调用可以快速被处理。

​	让我们继续完成我们的主节点的功能，我们创建了masterCreateCallback对象，用于接收create命令的结果：

------

```java
    static boolean isLeader;
    static StringCallback masterCreateCallback = new StringCallback() {
        void processResult(int rc, String path, Object ctx, String name) {
            switch(Code.get(rc)) {①
            case CONNECTIONLOSS:②
                checkMaster();
                return;
            case OK:③
                isLeader = true;
                break;
            default:④
                isLeader = false;
            }
            System.out.println("I'm " + (isLeader ? "" : "not ") +
                               "the leader");
        }
    };
    void runForMaster() {
        zk.create("/master", serverId.getBytes(), OPEN_ACL_UNSAFE,
                  CreateMode.EPHEMERAL, masterCreateCallback, null);⑤
    }
```

------

​	①我们从rc参数中获得create请求的结果，并将其转换为Code枚举类型。rc如果不为0，则对应KeeperException异常。

​	②如果因连接丢失导致create请求失败，我们会得到CONNECTIONLOSS编码的结果，而不是ConnectionLossException异常。当连接丢失时，我们需要检查系统当前的状态，并判断我们需要如何恢复，我们将会在我们后面实现的checkMaster方法中进行处理。

​	③我们现在成为群首，我们先简单地赋值isLeader为true。

​	④其他情况，我们并未成为群首。

​	⑤在runForMaster方法中，我们将masterCreateCallback传给create方法，传入null作为上下文对象参数，因为在runForMaster方法中，我们现在不需要向masterCreateCallback.processResult方法传入任何信息。

​	我们现在需要实现checkMaster方法，这个方法与之前的同步情况不太一样，我们通过回调方法实现处理逻辑，因此在checkMaster函数中不会看到一系列的事件，而只有getData方法。getData调用完成后，后续处理将会在DataCallback对象中继续：

------

```java
    DataCallback masterCheckCallback = new DataCallback() {
        void processResult(int rc, String path, Object ctx, byte[] data,
                           Stat stat) {
            switch(Code.get(rc)) {
            case CONNECTIONLOSS:
                checkMaster();
                return;
            case NONODE:
                runForMaster();
                return;
            }
        }
    }
    void checkMaster() {
        zk.getData("/master", false, masterCheckCallback, null);
    }
```

------

​	同步方法和异步方法的处理逻辑是一样的，只是异步方法中，我们没有使用while循环，而是通过异步操作在回调函数中进行错误处理。

​	此时，同步的版本看起来比异步版本实现起来更简单，但在下一章我们会看到，应用程序常常由异步变化通知所驱动，因此最终以异步方式构建系统，反而使代码更简单。同时，异步调用不会阻塞应用程序，这样其他事务可以继续进行，甚至是提交新的ZooKeeper操作。

### 3.3.2 设置元数据

​	我们将使用异步API方法来设置元数据路径。我们的主从模型设计依赖三个目录：/tasks、/assign和/workers，我们可以在系统启动前通过某些系统配置来创建所有目录，或者通过在主节点程序每次启动时都创建这些目录。以下代码段会创建这些路径，例子中除了连接丢失错误的处理外没有其他错误处理：

------

```java
public void bootstrap() {
    createParent("/workers", new byte[0]);①
    createParent("/assign", new byte[0]);
    createParent("/tasks", new byte[0]);
    createParent("/status", new byte[0]);
}
void createParent(String path, byte[] data) {
    zk.create(path,
              data,
              Ids.OPEN_ACL_UNSAFE,
              CreateMode.PERSISTENT,
              createParentCallback,
              data);②
}
StringCallback createParentCallback = new StringCallback() {
    public void processResult(int rc, String path, Object ctx, String name) {
        switch (Code.get(rc)) {
        case CONNECTIONLOSS:
            createParent(path, (byte[]) ctx);③
            break;
        case OK:
            LOG.info("Parent created");
            break;
        case NODEEXISTS:
            LOG.warn("Parent already registered: " + path);
            break;
        default:
            LOG.error("Something went wrong: ",
                      KeeperException.create(Code.get(rc), path));
        }
    }
};
```

------

​	①我们没有数据存入这些znode节点，所以只传入空的字节数组。

​	②因为如此，我们不用关心去跟踪每个znode节点对应的数据，但是往往每个路径都具有独特的数据，所以我们通过回调上下文参数对create操作进行跟踪数据。在create函数的第二个和第四个参数均传入的data对象，也许看起来有些奇怪，但第二个参数传入的data表示要保存到znode节点的数据，而第四个参数传入的data，我们可以在createParentCallback回调函数中继续使用。

​	③如果回调函数中得到CONNECTIONLOSS返回码，我们通过调用createPath方法来对create操作进行重试，然而调用createPath我们需要知道之前的create调用中的data参数，因此我们通过create的第四个参数传入data，就可以将数据通过ctx对象传给回调函数。因为上下文对象与回调对象不同，我们可以使所有create操作使用同一个回调对象。

​	从本例中，你会注意到znode节点与文件（一个包含数据的znode节点）和目录（含有子节点的znode节点）没有什么区别，每个znode节点可以具备以上两个特点。

## 3.4  注册从节点

​	现在我们已经有了主节点，我们需要配置从节点，以便主节点可以发号施令。根据我们的设计，每个从节点会在/workers下创建一个临时性的znode节点，很简单，我们通过以下代码就可以实现。我们将使用znode节点中的数据，来指示从节点的状态：

------

```java
import java.util.*;
import org.apache.zookeeper.AsyncCallback.DataCallback;
import org.apache.zookeeper.AsyncCallback.StringCallback;
import org.apache.zookeeper.AsyncCallback.VoidCallback;
import org.apache.zookeeper.*;
import org.apache.zookeeper.ZooDefs.Ids;
import org.apache.zookeeper.AsyncCallback.ChildrenCallback;
import org.apache.zookeeper.KeeperException.Code;
import org.apache.zookeeper.data.Stat;
import org.slf4j.*;
public class Worker implements Watcher {
    private static final Logger LOG = LoggerFactory.getLogger(Worker.class);
    ZooKeeper zk;
    String hostPort;
    String serverId = Integer.toHexString(random.nextInt());
    Worker(String hostPort) {
        this.hostPort = hostPort;
    }
    void startZK() throws IOException {
        zk = new ZooKeeper(hostPort, 15000, this);
    }
    public void process(WatchedEvent e) {
        LOG.info(e.toString() + ", " + hostPort);
    }
    void register() {
        zk.create("/workers/worker-" + serverId,
                  "Idle".getBytes(),①
                  Ids.OPEN_ACL_UNSAFE,
                  CreateMode.EPHEMERAL,②
                  createWorkerCallback, null);
    }
    StringCallback createWorkerCallback = new StringCallback() {
        public void processResult(int rc, String path, Object ctx,
                                  String name) {
            switch (Code.get(rc)) {
            case CONNECTIONLOSS:
                register();③
                break;
            case OK:
                LOG.info("Registered successfully: " + serverId);
                break;
            case NODEEXISTS:
                LOG.warn("Already registered: " + serverId);
                break;
            default:
                LOG.error("Something went wrong: "
                + KeeperException.create(Code.get(rc), path));
            }
        }woz
    };
    public static void main(String args[]) throws Exception {
        Worker w = new Worker(args[0]);
        w.startZK();
        w.register();
        Thread.sleep(30000);
    }
}
```

------

​	①我们将从节点的状态信息存入代表从节点的znode节点中。

​	②如果进程死掉，我们希望代表从节点的znode节点得到清理，所以我们使用了EPHEMERAL标志，这意味着，我们简单地关注/workers就可以得到有效从节点的列表。

​	③因为这个进程是唯一创建表示该进程的临时性znode节点的进程，如果创建节点时连接丢失，进程会简单地重试创建过程。

​	正如我们之前所看到的，因为我们注册了一个临时性节点，如果从节点死掉，表示这个从节点znode节点也会消失，所以这是我们在从节点组成管理上所有需要做的事情。

​	我们将从节点状态信息存入了代表从节点的znode节点，这样我们就可以通过查询ZooKeeper来获得从节点的状态。当前，我们只有初始化和空闲状态，但是，一旦从节点开始处理某些事情，我们还需要设置其他状态信息。

​	以下为setStatus的实现，这个方法与之前我们看到的方法有些不同，我们希望异步方式来设置状态，以便不会延迟常规流程的操作：

------

```java
    StatCallback statusUpdateCallback = new StatCallback() {
        public void processResult(int rc, String path, Object ctx, Stat stat) {
            switch(Code.get(rc)) {
            case CONNECTIONLOSS:
                updateStatus((String)ctx);①
                return;
            }
        }
    };
    synchronized private void updateStatus(String status) {
        if (status == this.status) {②
            zk.setData("/workers/" + name, status.getBytes(), -1,
                       statusUpdateCallback, status);③
        }
    }
    public void setStatus(String status) {
        this.status = status;④
        updateStatus(status);⑤
    }
```

------

​	④我们将状态信息保存到本地变量中，万一更新失败，我们需要重试。

​	⑤我们并未在setStatus进行更新，而是新建了一个updateStatus方法，我们在setStatus中使用它，并且可以在重试逻辑中使用。

​	②重新处理异步请求连接丢失时有个小问题：处理流程可能变得无序，因为ZooKeeper对请求和响应都会很好地保持顺序，但如果连接丢失，我们又再发起一个新的请求，就会导致整个时序中出现空隙。因此，我们进行一个状态更新请求前，需要先获得当前状态，否则就要放弃更新。我们通过同步方式进行检查和重试操作。

​	③我们执行无条件更新（第三个参数值为-1，表示禁止版本号检查），通过上下文对象参数传递状态。

​	①如果我们收到连接丢失的事件，我们需要用我们想要更新的状态再次调用updateStatus方法（通过setData的上下文参数传递参数），因为在updateStatus方法中进行了竞态条件的检查，所以我们在这里就不需要再次检查。

​	为了更多地理解连接丢失时补发操作的问题，考虑以下场景：

​	1.从节点开始执行任务task-1，因此设置其状态为working on task-1。

​	2.客户端库尝试通过setData来实现，但此时遇到了网络问题。

​	3.客户端库确定与ZooKeeper的连接已经丢失，同时在statusUpdateCallback调用前，从节点完成了任务task-1并处于空闲状态。

​	4.从节点调用客户端库，使用setData方法置状态为Idle。

​	5.之后客户端处理连接丢失的事件，如果updateStatus方法未检查当前状态，setData调用还是会设置状态为working on task-1。

​	6.当与ZooKeeper连接重新建立时，客户端库会按顺序如实地调用这两个setData操作，这就意味着最终状态为working on task-1。

​	在updateStatus方法中，在补发setData之前，先检查当前状态，这样我们就可以避免以上场景。

​	注意：顺序和ConnectionLossException异常

​	ZooKeeper会严格地维护执行顺序，并提供了强有力的有序保障，然而，在多线程下还是需要小心面对顺序问题。多线程下，当回调函数中包括重试逻辑的代码时，一些常见的场景都可能导致错误发生。当遇到ConnectionLossException异常而补发一个请求时，新建立的请求可能排序在其他线程中的请求之后，而实际上其他线程中的请求应该在原来请求之后。

## 3.5 任务队列化

系统最后的组件为Client应用程序队列化新任务，以便从节点执行这些任务，我们会在/tasks节点下添加子节点来表示需要从节点需要执行的命令。我们将会使用有序节点，这样做有两个好处，第一，序列号指定了任务被队列化的顺序；第二，可以通过很少的工作为任务创建基于序列号的唯一路径。我们的Client代码如下：

------

```java
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.Watcher;
public class Client implements Watcher {
    ZooKeeper zk;
    String hostPort;
    Client(String hostPort) { this.hostPort = hostPort; }
    void startZK() throws Exception {
        zk = new ZooKeeper(hostPort, 15000, this);
    }
    String queueCommand(String command) throws KeeperException {
        while (true) {
            try {
                String name = zk.create("/tasks/task-",①
                                        command.getBytes(), OPEN_ACL_UNSAFE,
                                        CreateMode.SEQUENTIAL);②
                return name;③
                break;
            } catch (NodeExistsException e) {
                throw new Exception(name + " already appears to be running");
            } catch (ConnectionLossException e) {④
            }
        }
    public void process(WatchedEvent e) { System.out.println(e); }
    public static void main(String args[]) throws Exception {
        Client c = new Client(args[0]);
        c.start();
        String name = c.queueCommand(args[1]);
        System.out.println("Created " + name);
    }
}
```

​	①我们在/tasks节点下创建znode节点来标识一个任务，节点名称前缀为task-。

​	②因为我们使用的是CreateMode.SEQUENTIAL模式的节点，task-后面会跟随一个单调递增的数字，这样就可以保证为每个任务创建的znode节点的名称是唯一的，同时ZooKeeper会确定任务的顺序。

​	③我们无法确定使用CreateMode.SEQUENTIAL调用create的序列号，create方法会返回新建节点的名称。

​	④如果我们在执行create时遇到连接丢失，我们需要重试create操作。因为多次执行创建操作，也许会为一个任务建立多个znode节点，对于大多数至少执行一次（execute-at-least-once）策略的应用程序，也没什么问题。对于某些最多执行一次（execute-at-most-once）策略的应用程序，我们就需要多一些额外工作：我们需要为每一个任务指定一个唯一的ID（如会话ID），并将其编码到znode节点名中，在遇到连接丢失的异常时，我们只有在/tasks下不存在以这个会话ID命名的节点时才重试命令。

​	当我们运行Client应用程序并发送一个命令时，/tasks节点下就会创建一个新的znode节点，该节点并不是临时性节点，因此即使Client程序结束了，这个节点依然会存在。

## 3.6 管理客户端

​	最后，我们将会写一个简单的AdminClient，通过该程序来展示系统的运行状态。ZooKeeper优点之一是我们可以通过zkCli工具来查看系统的状态，但是通常你希望编写你自己的管理客户端，以便更快更简单地管理系统。在本例中，我们通过getData和getChildren方法来获得主从系统的运行状态。

​	这些方法的使用非常简单，因为这些方法不会改变系统的运行状态，我们仅需要简单地传播我们遇到的错误，而不需要进行任何清理操作。

​	该示例使用了同步调用的方法，这些方法还有一个watch参数，我们置为false值，因为我们不需要监视变化情况，只是想获得系统当前的运行状态。在下一章中我们将会看到如何使用这个参数来跟踪系统的变化情况。现在，让我们看一下AdminClient的代码：

------

```java
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.Watcher;
public class AdminClient implements Watcher {
    ZooKeeper zk;
    String hostPort;
    AdminClient(String hostPort) { this.hostPort = hostPort; }
    void start() throws Exception {
        zk = new ZooKeeper(hostPort, 15000, this);
    }
    void listState() throws KeeperException {
        try {
            Stat stat = new Stat();
            byte masterData[] = zk.getData("/master", false, stat);①
            Date startDate = new Date(stat.getCtime());②
            System.out.println("Master: " + new String(masterData) +
                               " since " + startDate);
        } catch (NoNodeException e) {
            System.out.println("No Master");
        }
        System.out.println("Workers:");
        for (String w: zk.getChildren("/workers", false)) {
            byte data[] = zk.getData("/workers/" + w, false, null);③
            String state = new String(data);
            System.out.println("\t" + w + ": " + state);
        }
        System.out.println("Tasks:");
        for (String t: zk.getChildren("/assign", false)) {
            System.out.println("\t" + t);
        }
    }
    public void process(WatchedEvent e) { System.out.println(e); }
    public static void main(String args[]) throws Exception {
        AdminClient c = new AdminClient(args[0]);
        c.start();
        c.listState();
    }
}
```

------

​	①我们在/master节点中保存了主节点名称信息，因此我们从/master中获取数据，以获得当前主节点的名称。因为我们并不关心变化情况，所以我们将第二个参数置为false。

​	②我们通过Stat结构，可以获得当前主节点成为主节点的时间信息。ctime为该znode节点建立时的秒数（系统纪元以来的秒数，即自1970年1月1日00：00：00UTC的描述），详细信息请查看java.lang.System.currentTimeMillis（）。

​	③临时节点含有两个信息：指示当前从节点正在运行；其数据表示从节点的状态。

​	AdminClient非常简单，该程序简单地通过数据结构获得在主从示例的信息。我们试着可以启停Master程序、Worker程序，多次运行Client来队列化一些任务，AdminClient会展示系统的这些变化的状态。

​	你也许想知道如果在AdminClient中使用异步API是否更好。ZooKeeper有一个流水线的实现，用于处理成千上万的并发请求，对于系统需要面对的各种各样的延迟问题是非常重要的，而最大的延迟在于硬盘和网络。异步和同步的组件均通过队列方式来有效利用吞吐量。getData方法并不会进行任何硬盘的访问，但却需要依赖网络传输，同时我们使得同步方法的请求流水线管道化。如果我们的AdminClient程序运行于仅有几十个从节点、几百个任务的小型系统中，也许不会是大问题，但如果系统包含的从节点、任务再上升一个数量级，延迟问题可能就变得重要了。

​	考虑以下场景，请求的传输往返时延时间为1毫秒，如果进程需要读取10000个znode节点，我们就要在网络延迟上耗费10秒的时间，而通过异步API，我们可以缩短整个时间到接近1秒的时间。

​	以上面的Master、Worker、Client这些的基本实现带领我们进入了主从系统的开端，但到目前为止还没有实际调度起来。当一个任务加入队列，主节点需要唤醒并分配任务给一个从节点，从节点需要找出分配给自己的任务，任务完成时，客户端需要及时知道，如果主节点故障，另一个等待中的主节点需要接管主节点工作。如果从节点故障，分配给这个从节点的任务需要分配给其他从节点，在下一章中，我们将会讨论这些必要功能的实现。

## 3.7 小结

​	我们在zkCli工具中使用的命令与我们通过ZooKeeper编程所使用的API非常接近，因此，zkCli工具在初期调研时非常有用，我们可以通过该工具尝试不同的应用数据的组织方式。API与zkCli的命令非常接近，通过zkCli工具调研后，我们就可以快速写出与zkCli命令对应的应用程序。然而还有一些注意事项。首先，我们常常在一个稳定环境中使用zkCli工具，而不会发生某些未知故障，而对于需要部署的代码，我们需要处理异常情况使得我们的代码非常复杂，尤其是ConnectionLossException异常，开发者需要检查系统状态并合理恢复（ZooKeeper用于协助管理分布式状态，提供了故障处理的框架，但遗憾的是，它并不能让故障消失）。其次，适应异步API的开发非常有用，异步API提供了巨大的性能优势，简化了错误恢复工作。

# 4. 处理状态变化

​	在应用程序中，需要知道ZooKeeper集合的状态，这种情况并不少见。例如，在第1章的例子中，备份主节点需要知道主要主节点已经崩溃，从节点需要知道任务分配给了自己，甚至ZooKeeper的客户端会定时轮询ZooKeeper集合，检查系统状态是否发生了变化。然而轮询方式并非高效的方式，尤其是在期望的变化发生频率很低时。

​	举例说明，在主要主节点崩溃时，备份主节点需要知道这一情况，以便它们可以进行故障处理。为了减少主节点崩溃后的恢复时间，我们需要频繁轮询，如每50毫秒，只是为了用示例说明积极轮询的情况。在这种情况下，每个备份主节点每秒会产生20个请求，如果有多个备份主节点，备份主节点的数量乘上这个频率，就得到了为得到主要主节点状态而轮询ZooKeeper所消耗的全部请求量，即使像ZooKeeper这样的系统处理这个数量请求也非常简单，但主要主节点崩溃的情况很少发生，因此这些请求其实是多余的。假设我们通过增加主节点状态查询的轮询周期来减少对ZooKeeper的查询数量，如1秒，周期时间的增加则会导致主节点崩溃时恢复时间的增加。

​	我们完全可以通过ZooKeeper通知客户端感兴趣的具体事件来避免轮询的调优和轮询流量。ZooKeeper提供了处理变化的重要机制——监视点（watch）。通过监视点，客户端可以对指定的znode节点注册一个通知请求，在发生变化时就会收到一个单次的通知。例如，我们的主要主节点创建了一个临时性的znode节点来标识主节点锁，而备份主节点注册一个监视点来监视这个主节点锁是否存在，如果主节点崩溃，主节点锁自动被删除，并通知所有备份主节点。一旦备份主节点收到通知，它们就可以开始进行主节点选举，如3.3节中所述，通过尝试创建一个临时的znode节点来标识主节点锁。

​	监视点和通知形成了一个通用机制，使客户端可以观察变化情况，而不用不断地轮询ZooKeeper。我们已经通过主节点的例子进行了说明，但该通用机制还适用于很多情况。

## 4.1 单次触发器

​	在深入讨论监视点之前，我们先了解一些术语。我们所说的事件（event）表示一个znode节点执行了更新操作。而一个监视点（watch）表示一个与之关联的znode节点和事件类型组成的单次触发器（例如，znode节点的数据被赋值，或znode节点被删除）。当一个监视点被一个事件触发时，就会产生一个通知（notification）。通知是注册了监视点的应用客户端收到的事件报告的消息。

​	当应用程序注册了一个监视点来接收通知，匹配该监视点条件的第一个事件会触发监视点的通知，并且最多只触发一次。例如，当znode节点/z被删除，客户端需要知道该变化（例如，表示备份主节点），客户端在/z节点执行exists操作并设置监视点标志位，等待通知，客户端会以回调函数的形式收到通知。

​	客户端设置的每个监视点与会话关联，如果会话过期，等待中的监视点将会被删除。不过监视点可以跨越不同服务端的连接而保持，例如，当一个ZooKeeper客户端与一个ZooKeeper服务端的连接断开后连接到集合中的另一个服务端，客户端会发送未触发的监视点列表，在注册监视点时，服务端将要检查已监视的znode节点在之前注册监视点之后是否已经变化，如果znode节点已经发生变化，一个监视点的事件就会被发送给客户端，否则在新的服务端上注册监视点。

**单次触发是否会丢失事件**

​	答案是肯定的。一个应用在接收到通知后，注册另一个监视点时，可能会丢失事件，不过，这个问题需要再深入讨论。丢失事件通常并不是问题，因为任何在接收通知与注册新监视点之间的变化情况，均可以通过读取ZooKeeper的状态信息来获得。

​	假设一个从节点接收到一个新任务分配给它的通知。为了接收新任务，从节点读取任务列表，如果在通知接收后，又给这个从节点分配了更多的任务，在通过getChildren调用获取任务列表时会返回所有的任务。同时调用getChildren时也可以设置新的监视点，从而保证从节点不会丢失任务。

​	实际上，将多个事件分摊到一个通知上具有积极的作用，比如，应用进行高频率的更新操作时，这种通知机制比每个事件都发送通知更加轻量化。举个例子，如果每个通知平均捕获两个事件，我们为每个事件只产生了0.5个通知，而不是每个事件1个通知。

## 4.2 如何设置监视点

​	ZooKeeper的API中的所有读操作：getData、getChildren和exists，均可以选择在读取的znode节点上设置监视点。使用监视点机制，我们需要实现Watcher接口类，实现其中的process方法：

------

```
public void process(WatchedEvent event);
```

------

​	WatchedEvent数据结构包括以下信息：

* ZooKeeper会话状态（KeeperState）：Disconnected、SyncConnected、AuthFailed、ConnectedReadOnly、SaslAuthenticated和Expired。

* 事件类型（EventType）：NodeCreated、NodeDeleted、NodeDataChanged、NodeChildrenChanged和None。

* 如果事件类型不是None时，返回一个znode路径。



​	其中前三个事件类型只涉及单个znode节点，第四个事件类型涉及监视的znode节点的子节点。我们使用None表示无事件发生，而是ZooKeeper的会话状态发生了变化。

​	监视点有两种类型：数据监视点和子节点监视点。创建、删除或设置一个znode节点的数据都会触发数据监视点，exists和getData这两个操作可以设置数据监视点。只有getChildren操作可以设置子节点监视点，这种监视点只有在znode子节点创建或删除时才被触发。对于每种事件类型，我们通过以下调用设置监视点：

**NodeCreated**

​	通过exists调用设置一个监视点。

**NodeDeleted**

​	通过exists或getData调用设置监视点。

**NodeDataChanged**

​	通过exists或getData调用设置监视点。

**NodeChildrenChanged**

​	通过getChildren调用设置监视点。

​	当创建一个ZooKeeper对象（见第3章），我们需要传递一个默认的Watcher对象，ZooKeeper客户端使用这个监视点来通知应用ZooKeeper状态的变化情况，如会话状态的变化。对于ZooKeeper节点的事件的通知，你可以使用默认的监视点，也可以单独实现一个。例如，getData调用有两种方式设置监视点：

------

```java
public byte[] getData(final String path, Watcher watcher, Stat stat);
public byte[] getData(String path, boolean watch, Stat stat);
```

------

​	两个方法第一个参数均为znode节点，第一个方法传递一个新的Watcher对象（我们已经创建完毕），第二个方法则告诉客户端使用默认的监视点，我们只需要在调用时将第二个参数传递true。

​	stat入参为Stat类型的实例化对象，ZooKeeper使用该对象返回指定的path参数的znode节点信息。Stat结构包括znode节点的属性信息，如该znode节点的上次更新（zxid）的时间戳，以及该znode节点的子节点数。

​	对于监视点的一个重要问题是，一旦设置监视点就无法移除。要想移除一个监视点，只有两个方法，一是触发这个监视点，二是使其会话被关闭或过期。在未来版本中可能会改变这个特性，这是因为开发社区致力于在版本3.5.0中提供该功能。

**注意：关于一些重载**

​	在ZooKeeper的会话状态和znode节点的变化事件中，我们使用了相同的监视机制来处理应用程序的相关事件的通知。虽然会话状态的变化和znode状态的变化组成了两个独立的事件集合，为简单其见，我们使用了相同的机制传送这些事件。

## 4.3 普遍模型

我们进入主-从模式例子的章节之前，先看一些ZooKeeper的应用中使用的通用代码的模型：

​	1.进行调用异步。

​	2.实现回调对象，并传入异步调用函数中。

​	3.如果操作需要设置监视点，实现一个Watcher对象，并传入异步调用函数中。

​	以下为exists的异步调用的示例代码：

```java
zk.exists("/myZnode",①
          myWatcher,
          existsCallback,
          null);
Watcher myWatcher = new Watcher() {②
    public void process(WatchedEvent e) {
        // Process the watch event
    }
}
StatCallback existsCallback = new StatCallback() {③
    public void processResult(int rc, String path, Object ctx, Stat stat) {
        // Process the result of the exists call
    }
};
```

------

①ZooKeeper的exists调用，注意该调用为异步方式。

②Watcher的实现。

③exists的回调对象。

之后我们会看到，该框架使用非常广泛。

## 4.4 主-从模式的例子

现在，我们通过主-从模式的例子来看看如何处理状态的变化。以下为一个任务列表，一个组件需要等待处理的变化情况：

* 管理权变化。

* 主节点等待从节点列表的变化。

* 主节点等待新任务进行分配。

* 从节点等待分配新任务。

* 客户端等待任务的执行结果。



​	我们之后会通过一些代码片段来说明如何在ZooKeeper环境中处理这些任务。

### 4.4.1 管理权变化

​	回忆在3.3节讨论的内容，应用客户端通过创建/master节点来推选自己为主节点（我们称为“主节点竞选”），如果znode节点已经存在，应用客户端确认自己不是主要主节点并返回，然而，这种实现方式无法容忍主要主节点的崩溃。如果主要主节点崩溃，备份主节点并不知道，因此我们需要在/master上设置监视点，在节点删除时（无论是显式关闭还是因为主要主节点的会话过期），ZooKeeper会通知客户端。

​	为了设置监视点，我们新建一个新的监视点对象，命名为masterExistsWatcher，并传入exists方法中。一旦/master删除就会发出通知，就会调用在masterExistsWatcher定义的process函数，并调用runForMaster方法。

------

```java
StringCallback masterCreateCallback = new StringCallback() {
    public void processResult(int rc, String path, Object ctx, String name) {
        switch (Code.get(rc)) {
        case CONNECTIONLOSS:
            checkMaster();①
            break;
        case OK:
            state = MasterStates.ELECTED;
            takeLeadership();②
            break;
        case NODEEXISTS:
            state = MasterStates.NOTELECTED;
            masterExists();③
            break;
        default:
            state = MasterStates.NOTELECTED;
            LOG.error("Something went wrong when running for master.",④
                      KeeperException.create(Code.get(rc), path));
        }
    }
};
void masterExists() {
    zk.exists("/master",⑤
              masterExistsWatcher,
              masterExistsCallback,
              null);
}
Watcher masterExistsWatcher = new Watcher() {
    public void process(WatchedEvent e) {
        if(e.getType() == EventType.NodeDeleted) {
            assert "/master".equals( e.getPath() );
            runForMaster();⑥
        }
    }
};
```

------

​	①在连接丢失事件发生的情况下，客户端检查/master节点是否存在，因为客户端并不知道是否能够创建这个节点。

​	②如果返回OK，那么开始行使领导权。

​	③如果其他进程已经创建了这个znode节点，客户端需要监视该节点。

​	④如果发生了某些意外情况，就会记录错误日志，而不再做其他事情。

​	⑤通过exists调用在/master节点上设置了监视点。

​	⑥如果/master节点删除了，那么再次竞选主节点。

​	下面继续采用我们在3.3.1节中所讨论的异步方式，我们同样需要为exists调用创建一个回调函数，以便在回调函数中关注某些情况。首先，在发生连接丢失的事件时，因为需要在/master节点上设置监视点，所以需要再次调用exists操作；其次，在create的回调方法执行和exists操作执行之间发生了/master节点被删除的情况，因此在exists返回操作成功后（返回OK），我们需要检查返回的stat对象是否为空，因为当节点不存在时，stat为null；最后，如果返回的结果不是OK或CONNECTIONLOSS，我们通过获取节点数据来检查/master节点。加入客户端的会话过期，在这种情况下，获得/master数据的回调方法会记录一个错误信息并退出。以下为我们的exists回调方法的代码：

------

```java
StatCallback masterExistsCallback = new StatCallback() {
    public void processResult(int rc, String path, Object ctx, Stat stat) {
        switch (Code.get(rc)) {
        case CONNECTIONLOSS:
            masterExists();①
            break;
        case OK:
            if(stat == null) {
                state = MasterStates.RUNNING;
                runForMaster();②
            }
            break;
        default:
            checkMaster();③
            break;
        }
    }
};
```

------

​	①连接丢失的情况下重试。

​	②如果返回OK，判断znode节点是否存在，不存在就竞选主节点。

​	③如果发生意外情况，通过获取节点数据来检查/master是否存在。

​	对/master节点执行exists操作，返回结果也许是该znode节点已经被删除，这时因为无法保证监视点的设置是在znode节点删除前，所以客户端需要再次竞选/master节点。如果再次尝试成为主节点失败，那么客户端就知道有其他客户端成功了，之后该客户端就需要再次为/master节点添加监视点。如果收到的是/master节点创建的通知，而不是删除通知，客户端就不再竞选/master节点，同时，对应的exists操作（设置监视点的操作）会返回/master节点不存在，然后触发exists回调方法，进行/master节点竞选操作。

​	我们注意到，只要客户端程序运行中，且没有成为主要主节点，客户端竞选主节点并执行exists来设置监视点，这一模式就会一直运行下去。如果客户端成为主要主节点，但却崩溃了，客户端重启还继续重新执行这些代码。

​	图4-1直观地展示了这些交错的操作。如果竞选主节点成功（图中a），create操作执行完成，应用客户端不需要做其他事情。如果create操作失败，则意味着该节点已经存在，客户端就会执行exists操作来设置/master节点的监视点（图中b）。在竞选主节点和执行exists操作之间，也许/master节点已经删除了，这时，如果exists调用返回该节点依然存在，客户端只需要等待通知的到来，否则就需要再次尝试创建/master进行竞选主节点操作。如果创建/master节点成功，监视点就会被触发，表示znode节点发生了变化（图中c），不过，这个通知没有什么意义，因为这是客户端自己引起的变化。如果再次执行create操作失败，我们就会通过执行exists设置监视点来重新执行这一流程（图中d）。

![](https://pic.imgdb.cn/item/6135be4c44eaada7396c946a.jpg)

### 4.4.2 主节点等待从节点列表的变化

​	系统中任何时候都可能发生新的从节点加入进来，或旧的从节点退役的情况，从节点执行分配给它的任务前也许会崩溃。为了确认某个时间点可用的从节点信息，我们通过在ZooKeeper中的/workers下添加子节点来注册新的从节点。当一个从节点崩溃或从系统中被移除，如会话过期等情况，需要自动将对应的znode节点删除。优雅实现的从节点会显式地关闭其会话，而不需要ZooKeeper等待会话过期。

​	主要主节点使用getChildren来获取有效的从节点列表，同时还会监控这个列表的变化。以下为获取列表并监视变化的示例代码：

------

```java
Watcher workersChangeWatcher = new Watcher() {①
    public void process(WatchedEvent e) {
        if(e.getType() == EventType.NodeChildrenChanged) {
            assert "/workers".equals( e.getPath() );
            getWorkers();
        }
    }
};
void getWorkers() {
    zk.getChildren("/workers",
                    workersChangeWatcher,
                    workersGetChildrenCallback,
                    null);
}
ChildrenCallback workersGetChildrenCallback = new ChildrenCallback() {
    public void processResult(int rc, String path, Object ctx,
                              List<String> children) {
        switch (Code.get(rc)) {
        case CONNECTIONLOSS:
            getWorkerList();②
            break;
        case OK:
            LOG.info("Succesfully got a list of workers: "
                     + children.size()
                     + " workers");
            reassignAndSet(children);③
            break;
        default:
            LOG.error("getChildren failed",
                      KeeperException.create(Code.get(rc), path));
        }
    }
};
```

------

​	①workersChangeWatcher为从节点列表的监视点对象。

​	②当CONNECTIONLOSS事件发生时，我们需要重新获取子节点并设置监视点的操作。

​	③重新分配崩溃从节点的任务，并重新设置新的从节点列表。

​	我们从getWorkerList方法开始执行，通过异步方式执行getChildren方法，传入workersGetChildrenCallback参数用于处理操作结果。如果客户端失去与服务端的连接（CONNECTIONLOSS事件），监视点不会被添加，我们也不会得到从节点的列表，我们再次执行getWorkerList来设置监视点并获取从节点列表，如果执行getChildren成功，我们就会调用reassignAndSet方法，该方法的代码如下：

------

```java
ChildrenCache workersCache;①
void reassignAndSet(List<String> children) {
    List<String> toProcess;
        if(workersCache == null) {
            workersCache = new ChildrenCache(children);②
            toProcess = null;③
        } else {
            LOG.info( "Removing and setting" );
            toProcess = workersCache.removedAndSet( children );④
        }
        if(toProcess != null) {
            for(String worker : toProcess) {
                getAbsentWorkerTasks(worker);⑤
            }
        }
    }
```

------

​	①该变量用于保存上次获得的从节点列表的本地缓存。

​	②如果第一次使用本地缓存这个变量，那么初始化该变量。

​	③第一次获得所有从节点时，不需要做什么其他事。

​	④如果不是第一次，那么需要检查是否有从节点已经被移除了。

​	⑤如果有从节点被移除了，我们就需要重新分配任务。

​	我们需要保存之前获得的信息，因此使用本地缓存。假设在我们第一次获得从节点列表后，当收到从节点列表更新的通知时，如果我们没有保存旧的信息，即使我们再次读取信息也不知道具体变化的信息是什么。本例中的缓存类简单地保存主节点上次读取的列表信息，并实现检查变化信息的一些方法。

**注意：基于CONNECTIONLOSS事件的监视**

​	监视点的操作执行成功后就会为一个znode节点设置一个监视点，如果ZooKeeper的操作因为客户端连接断开而失败，应用需要再次执行这些调用。

### 4.4.3 主节点等待新任务进行分配

​	与等待从节点列表变化类似，主要主节点等待添加到/tasks节点中的新任务。主节点首先获得当前的任务集，并设置变化情况的监视点。在ZooKeeper中，/tasks的子节点表示任务集，每个子节点对应一个任务，一旦主节点获得还未分配的任务信息，主节点会随机选择一个从节点，将这个任务分配给从节点。以下assignTasks方法为任务分配的实现：

------

```java
Watcher tasksChangeWatcher = new Watcher() {①
    public void process(WatchedEvent e) {
        if(e.getType() == EventType.NodeChildrenChanged) {
            assert "/tasks".equals( e.getPath() );
            getTasks();
        }
    }
};
void getTasks() {
    zk.getChildren("/tasks",
                   tasksChangeWatcher,
                   tasksGetChildrenCallback,
                   null);②
}
ChildrenCallback tasksGetChildrenCallback = new ChildrenCallback() {
    public void processResult(int rc,
                              String path,
                              Object ctx,
                              List<String> children) {
        switch(Code.get(rc)) {
        case CONNECTIONLOSS:
            getTasks();
            break;
        case OK:
            if(children != null) {
                assignTasks(children);③
            }
            break;
        default:
            LOG.error("getChildren failed.",
                      KeeperException.create(Code.get(rc), path));
        }
    }
};
```

------

​	①在任务列表变化时，处理通知的监视点实现。

​	②获得任务列表。

​	③分配列表中的任务。

​	现在我们实现assignTasks方法，该方法简单地分配在/tasks子节点表示的列表中的每个任务。在建立任务分配的znode节点前，我们首先通过getData方法获得任务信息：

------

```java
void assignTasks(List<String> tasks) {
    for(String task : tasks) {
        getTaskData(task);
    }
}
void getTaskData(String task) {
    zk.getData("/tasks/" + task,
               false,
               taskDataCallback,
               task);①
}
DataCallback taskDataCallback = new DataCallback() {
    public void processResult(int rc,
                              String path,
                              Object ctx,
                              byte[] data,
                              Stat stat)  {
        switch(Code.get(rc)) {
        case CONNECTIONLOSS:
            getTaskData((String) ctx);
            break;
        case OK:
            /*
             * Choose worker at random.
             */
            int worker = rand.nextInt(workerList.size());
            String designatedWorker = workerList.get(worker);
            /*
             * Assign task to randomly chosen worker.
             */
            String assignmentPath = "/assign/" + designatedWorker + "/" +
                                     (String) ctx;
            createAssignment(assignmentPath, data);②
            break;
        default:
            LOG.error("Error when trying to get task data.",
                    KeeperException.create(Code.get(rc), path));
        }
    }
};
```

------

​	①获得任务信息。

​	②随机选择一个从节点，分配任务给这个从节点。

​	首先我们需要获得任务的信息，因为分配任务后，我们会删除/tasks下的这个子节点，这样，主节点就不需要记住分配了哪些任务。让我们看一下分配一个任务的代码：

------

```java
void createAssignment(String path, byte[] data) {
    zk.create(path,
            data, Ids.OPEN_ACL_UNSAFE,
            CreateMode.PERSISTENT,
            assignTaskCallback,
            data);①
}
StringCallback assignTaskCallback = new StringCallback() {
    public void processResult(int rc, String path, Object ctx, String name) {
        switch(Code.get(rc)) {
        case CONNECTIONLOSS:
            createAssignment(path, (byte[]) ctx);
            break;
        case OK:
            LOG.info("Task assigned correctly: " + name);
            deleteTask(name.substring( name.lastIndexOf("/") + 1 ));②
            break;
        case NODEEXISTS:
            LOG.warn("Task already assigned");
            break;
        default:
            LOG.error("Error when trying to assign task.",
                      KeeperException.create(Code.get(rc), path));
        }
    }
};
```

------

​	①创建分配节点，路径形式为/assign/worker-id/task-num。

​	②删除/tasks下对应的任务节点。

​	对于新任务，主节点选择一个从节点分配任务之后，主节点就会在/assign/work-id节点下创建一个新的znode节点，其中id为从节点标识符，之后主节点从任务列表中删除该任务节点。上面的例子中，其中删除znode节点的代码采用了我们之前讲到的模式的代码。

​	当主节点为某个标识符为id的从节点创建任务分配节点时，假设从节点在任务分配节点（/assign/work-id）上注册了监视点，ZooKeeper会向从节点发送一个通知。

​	注意，主节点在成功分配任务后，会删除/tasks节点下对应的任务。这种方式简化了主节点角色接收新任务并分配的设计，如果任务列表中混合的已分配和未分配的任务，主节点还需要区分这些任务。

### 4.4.4  从节点等待分配新任务

从节点第一步需要先向ZooKeeper注册自己，如我们之前已经讨论过的，在/workers节点下创建一个子节点：

------

```java
void register() {
        zk.create("/workers/worker-" + serverId,
                  new byte[0],
                  Ids.OPEN_ACL_UNSAFE,
                  CreateMode.EPHEMERAL,
                  createWorkerCallback, null);①
}
StringCallback createWorkerCallback = new StringCallback() {
    public void processResult(int rc, String path, Object ctx, String name) {
        switch (Code.get(rc)) {
        case CONNECTIONLOSS:
            register(); ②|
            break;
        case OK:
            LOG.info("Registered successfully: " + serverId);
            break;
        case NODEEXISTS:
            LOG.warn("Already registered: " + serverId);
            break;
        default:
            LOG.error("Something went wrong: " +
                      KeeperException.create(Code.get(rc), path));
        }
    }
};
```

------

​	①通过创建一个znode节点来注册从节点。

​	②重试，注意再次注册不会有问题，因为如果znode节点已经存在，我们会收到NODEEXISTS事件。

​	添加该znode节点会通知主节点，这个从节点的状态是活跃的，且已准备好处理任务。注意我们并未使用第3章中介绍的空闲/忙碌（idle/busy）状态，只是为了该示例的简化。

​	同样，我们还创建了/assign/work-id节点，这样，主节点可以为这个从节点分配任务。如果我们在创建/assign/worker-id节点之前创建了/workers/worker-id节点，我们可能会陷入以下情况，主节点尝试分配任务，因分配节点的父节点还没有创建，导致主节点分配失败。为了避免这种情况，我们需要先创建/assign/worker-id节点，而且从节点需要在/assign/worker-id节点上设置监视点来接收新任务分配的通知。

​	一旦有任务列表分配给从节点，从节点就会从/assign/worker-id获取任务信息并执行任务。从节点从本地列表中获取每个任务的信息并验证任务是否还在待执行的队列中，从节点保存一个本地待执行任务的列表就是为了这个目的。注意，为了释放回调方法的线程，我们在单独的线程对从节点的已分配任务进行循环，否则，我们会阻塞其他的回调方法的执行。示例中，我们使用了Java的ThreadPoolExecutor类分配一个线程，该线程进行任务的循环操作：

------

```java
Watcher newTaskWatcher = new Watcher() {
    public void process(WatchedEvent e) {
        if(e.getType() == EventType.NodeChildrenChanged) {
            assert new String("/assign/worker-"+ serverId).equals( e.getPath() );
            getTasks();①
        }
    }
};
void getTasks() {
    zk.getChildren("/assign/worker-" + serverId,
                   newTaskWatcher,
                   tasksGetChildrenCallback,
                   null);
}
ChildrenCallback tasksGetChildrenCallback = new ChildrenCallback() {
    public void processResult(int rc,
                              String path,
                              Object ctx,
                              List<String> children) {
        switch(Code.get(rc)) {
        case CONNECTIONLOSS:
            getTasks();
            break;
        case OK:
            if(children != null) {
                executor.execute(new Runnable() {②
                    List<String> children;
                    DataCallback cb;
                    public Runnable init (List<String> children,
                                          DataCallback cb) {
                        this.children = children;
                        this.cb = cb;
                        return this;
                    }
                    public void run() {
                        LOG.info("Looping into tasks");
                        synchronized(onGoingTasks) {
                            for(String task : children) {③
                                if(!onGoingTasks.contains( task )) {
                                    LOG.trace("New task: {}", task);
                                    zk.getData("/assign/worker-" +
                                               serverId + "/" + task,
                                               false,
                                               cb,
                                               task);④
                                    onGoingTasks.add( task );⑤
                                }
                            }
                        }
                    }
                }
                .init(children, taskDataCallback));
            }
            break;
        default:
            System.out.println("getChildren failed: " +
                               KeeperException.create(Code.get(rc), path));
        }
    }
};
```

------

​	①当收到子节点变化的通知后，获得子节点的列表。

​	②单独线程中执行。

​	③循环子节点列表。

​	④获得任务信息并执行任务。

​	⑤将正在执行的任务添加到执行中列表，防止多次执行。

**注意：会话事件和监视点**

​	当我们与服务端的连接断开时（例如，服务端崩溃时），直到连接重新建立前，不会传送任何监视点。因此，会话事件CONNECTIONLOSS会发送给所有已知的监视点进行处理。一般来说，应用使用会话事件进入安全模式：ZooKeeper客户端在失去连接后不会接收任何事件，因此客户端需要继续保持这种状态。在我们的主从应用的例子中，处理提交任务外，其他所有动作都是被动的，所以如果主节点或从节点发生连接断开时，不会触发任何动作。而且在连接断开时，主从应用中的客户端也无法提交新任务以及接收任务状态的通知。

### 4.4.5  客户端等待任务的执行结果

​	假设应用客户端已经提交了一个任务，现在客户端需要知道该任务何时被执行，以及任务状态。回忆之前所讨论的，从节点执行执行一个任务时，会在/status下创建一个znode节点。让我们先看下提交任务执行的代码：

------

```java
void submitTask(String task, TaskObject taskCtx) {
    taskCtx.setTask(task);
    zk.create("/tasks/task-",
              task.getBytes(),
              Ids.OPEN_ACL_UNSAFE,
              CreateMode.PERSISTENT_SEQUENTIAL,
              createTaskCallback,
              taskCtx);①
}
StringCallback createTaskCallback = new StringCallback() {
    public void processResult(int rc, String path, Object ctx, String name) {
        switch (Code.get(rc)) {
        case CONNECTIONLOSS:
            submitTask(((TaskObject) ctx).getTask(),
                         (TaskObject) ctx);②
            break;
        case OK:
            LOG.info("My created task name: " + name);
            ((TaskObject) ctx).setTaskName(name);
            watchStatus("/status/" + name.replace("/tasks/", ""),
                         ctx);③
            break;
        default:
            LOG.error("Something went wrong" +
                       KeeperException.create(Code.get(rc), path));
        }
    }
};
```

------

①与之前的ZooKeeper调用不同，我们传递了一个上下文对象，该对象为我们实现的Task类的实例。

②连接丢失时，再次提交任务，注意重新提交任务可能会导致任务重复。

③为这个任务的znode节点设置一个监视点。

**注意：有序节点是否创建成功？**

​	在创建有序节点时发生CONNECTIONLOSS事件，处理这种情况比较棘手，因为ZooKeeper每次分配一个序列号，对于连接断开的客户端，无法确认这个节点是否创建成功，尤其是在其他客户端一同并发请求时（我们提到的并发请求是指多个客户端进行相同的请求）。为了解决这个问题，我们需要添加一些提示信息来标记这个znode节点的创建者，比如，在任务名称中加入服务器ID等信息，通过这个方法，我们就可以通过获取所有任务列表来确认任务是否添加成功。

​	我们检查状态节点是否已经存在（也许任务很快处理完成），并设置监视点。我们提供了一个收到znode节点创建的通知时进行处理的监视点的实现和一个exists方法的回调实现：

------

```java
ConcurrentHashMap<String, Object> ctxMap =
    new ConcurrentHashMap<String, Object>();
void watchStatus(String path, Object ctx) {
    ctxMap.put(path, ctx);
    zk.exists(path,
              statusWatcher,
              existsCallback,
              ctx);①
}
Watcher statusWatcher = new Watcher() {
    public void process(WatchedEvent e) {
        if(e.getType() == EventType.NodeCreated) {
            assert e.getPath().contains("/status/task-");
            zk.getData(e.getPath(),
                       false,
                       getDataCallback,
                       ctxMap.get(e.getPath()));
        }
    }
};
StatCallback existsCallback = new StatCallback() {
    public void processResult(int rc, String path, Object ctx, Stat stat) {
        switch (Code.get(rc)) {
        case CONNECTIONLOSS:
            watchStatus(path, ctx);
            break;
        case OK:
            if(stat != null) {
                zk.getData(path, false, getDataCallback, null);②
            }
            break;
        case NONODE:
            break;③
        default:
            LOG.error("Something went wrong when " +
                          "checking if the status node exists: " +
                      KeeperException.create(Code.get(rc), path));
            break;
        }
    }
};
```

------

​	①客户端通过该方法传递上下对象，当收到状态节点的通知时，就可以修改这个表示任务的对象（TaskObject）。

​	②状态节点已经存在，因此客户端获取这个节点信息。

​	③如果状态节点不存在，这是常见情况，客户端不进行任何操作。

## 4.5  另一种调用方式：Multiop

​	Multiop并非ZooKeeper的原始设计，该特性在3.4.0版本中被添加进来。Multiop可以原子性地执行多个ZooKeeper的操作，执行过程为原子性，即在multiop代码块中的所有操作要不全部成功，要不全部失败。例如，我们在一个multiop块中删除一个父节点以及其子节点，执行结果只可能是这两个操作都成功或都失败，而不可能是父节点被删除而子节点还存在，或子节点被删除而父节点还存在。

使用multiop特性：

​	1.创建一个Op对象，该对象表示你想通过multiop方法执行的每个ZooKeeper操作，ZooKeeper提供了每个改变状态操作的Op对象的实现：create、delete和setData。

​	2.通过Op对象中提供的一个静态方法调用进行操作。

​	3.将Op对象添加到Java的Iterable类型对象中，如列表（List）。

​	4.使用列表对象调用multi方法。

​	以下示例说明这个过程：

------

```java
    Op deleteZnode(String z) {①
        return Op.delete(z, -1);②
    }
    ...
    List<OpResult> results = zk.multi(Arrays.asList(deleteZnode("/a/b"),
                                      deleteZnode("/a"));③
```

------

​	①为delete方法创建Op对象。

​	②通过对应的Op方法返回对象。

​	③以列表方式传入每个delete操作的元素执行multi方法。

​	调用multi方法返回一个OpResult对象的列表，每个对象对应每个操作。例如，对于delete操作，我们使用DeleteResult类，该类继承自OpResult，通过每种操作类型对应的结果对象暴露方法和数据。DeleteResult对象仅提供了equals和hashCode方法，而CreateResult对象暴露出操作的路径（path）和Stat对象。对于错误处理，ZooKeeper返回一个包含错误码的ErrorResult类的实例。

​	multi方法同样也有异步版本，以下为同步方法和异步方法的定义：

------

```
public List<OpResult> multi(Iterable<Op> ops) throws InterruptedException,
                                                     KeeperException;
public void multi(Iterable<Op> ops, MultiCallback cb, Object ctx);
```

------

​	Transaction封装了multi方法，提供了简单的接口。我们可以创建Transaction对象的实例，添加操作，提交事务。使用Transaction重写上一示例的代码如下：

------

```
Transaction t = new Transaction();
t.delete("/a/b", -1);
t.delete("/a", -1);
List<OpResult> results = t.commit();
```

------

​	commit方法同样也有一个异步版本的方法，该方法以MultiCallback对象和上下文对象为输入：

------

```
public void commit(MultiCallback cb, Object ctx);
```

------

​	multiop可以简化不止一处的主从模式的实现，当分配一个任务，在之前的例子中，主节点会创建任务分配节点，然后删除/tasks下对应的任务节点。如果在删除/tasks下的节点时，主节点崩溃，就会导致一个已分配的任务还在/tasks下。使用multiop，我们可以原子化创建任务分配节点和删除/tasks下对应的任务节点这两个操作。使用这个方式，我们可以保证没有已分配的任务还在/tasks节点下，如果备份节点接管了主节点角色，就不用再区分/tasks下的任务是不是没有分配的。

​	multiop提供的另一个功能是检查一个znode节点的版本，通过multiop可以同时读取的多个节点的ZooKeeper状态并回写数据——如回写某些读取到的数据信息。当被检查的znode版本号没有变化时，就可以通过multiop调用来检查没有被修改的znode节点的版本号，这个功能非常有用，如在检查一个或多个znode节点的版本号取决于另外一个znode节点的版本号时。在我们的主从模式的示例中，主节点需要让客户端在主节点指定的路径下添加新任务，例如，主节点要求客户端在/task-mid的子节点中添加新任务节点，其中mid为主节点的标识符，主节点在/master-path节点中保存这个路径的数据，客户端在添加新任务前，需要先读取/master-path的数据，并通过Stat获取这个节点的版本号信息，然后，客户端通过multiop的部分调用方式在/task-mid节点下添加新任务节点，同时会检查/master-path的版本号是否与之前读取的相匹配。

​	check方法的定义与setData方法相似，只是没有data参数：

------

```
public static Op check(String path, int version);
```

------

​	如果输入的path的znode节点的版本号不匹配，multi调用会失败。通过以下简单的示例代码，我们来说明如何实现上面所讨论的场景：

------

```
byte[] masterData = zk.getData("/master-path", false, stat);①
String parent = new String(masterData);②
...
zk.multi(Arrays.asList(Op.check("/master-path", stat.getVersion()),
                       Op.create(, modify(z1Data),-1),③
```

------

​	①获取/master节点的数据。

​	②从/master节点获得路径信息。

​	③两个操作的multi调用。

注意，如果我们在/master节点中以主节点ID来保存路径信息，以上方式就无法正常运行，因为新主节点每次都会创建/master，从而导致/master的版本号始终为1。

## 4.6 通过监视点代替显式缓存管理

​	从应用的角度来看，客户端每次都是通过访问ZooKeeper来获取给定znode节点的数据、一个znode节点的子节点列表或其他相关的ZooKeeper状态，这种方式并不可取。反而更高效的方式为客户端本地缓存数据，并在需要时使用这些数据，一旦这些数据发生变化，你让ZooKeeper通知客户端，客户端就可以更新缓存的数据。这些通知与我们之前所讨论的一样，应用的客户端通过注册监视点来接收这些通知消息。总之，监视点可以让客户端在本地缓存一个版本的数据（比如，一个znode节点数据或节点的子节点列表信息），并在数据发生变化时接收到通知来进行更新。

​	ZooKeeper的设计者还可以采用另一种方式，客户端透明地缓存客户端访问的所有ZooKeeper状态，并在更新缓存数据时将这些数据置为无效。实现这种缓存一致性的方案代价非常大，因为客户端也许并不需要缓存所有它们所访问的ZooKeeper状态，而且服务端需要将缓存状态置为无效，为了实现失效机制，服务端不得不关注每个客户端中缓存的信息，并广播失效请求。客户端数量很大时，这两方面的代价都非常大，而且我们认为这样也是不可取的。

​	不管是哪部分负责管理客户端缓存，ZooKeeper直接管理或ZooKeeper应用来管理，都可以通过同步或异步方式进行更新操作的客户端通知。同步方式使所有持有该状态拷贝的客户端中的状态无效，这种方式效率很低，因为客户端往往以不同的速度运行中，因此缓慢的客户端会强制其他客户端进行等待，随着客户端越来越多，这种差异就会更加频繁发生。

​	设计者选择的通知方式可视为一种在客户端一侧使ZooKeeper状态失效的异步方式，ZooKeeper将给客户端的通知消息队列化，这些通知会以异步的方式进行消费。这种失效方案也是可选方案，应用程序中需要解决，对于任何给定的客户端，哪些部分ZooKeeper状态需要置为无效。这种设计的选择更适合ZooKeeper的应用场景。

## 4.7 顺序的保障

​	在通过ZooKeeper实现我们的应用时，我们还要牢记一些很重要的涉及顺序性的事项。

### 4.7.1 写操作的顺序

​	ZooKeeper状态会在所有服务端所组成的全部安装中进行复制。服务端对状态变化的顺序达成一致，并使用相同的顺序执行状态的更新。例如，如果一个ZooKeeper的服务端执行了先建立一个/z节点的状态变化之后再删除/z节点的状态变化这个顺序的操作，所有的在集合中的服务端均需以相同的顺序执行这些变化。

​	所有服务端并不需要同时执行这些更新，而且事实上也很少这样操作。服务端更可能在不同时间执行状态变化，因为它们以不同的速度运行，即使它们运行在同种硬件下。有很多原因会导致这种时滞发生，如操作系统的调度、后台任务等。

​	对于应用程序来说，在不同时间点执行状态更新并不是问题，因为它们会感知到相同的更新顺序。应用程序也可能感知这一顺序，但如果ZooKeeper状态通过隐藏通道进行通信时，我们将在后续章节进行讨论。

### 4.7.2 读操作的顺序

​	ZooKeeper客户端总是会观察到相同的更新顺序，即使它们连接到不同的服务端上。但是客户端可能是在不同时间观察到了更新，如果他们还在ZooKeeper以外通信，这种差异就会更加明显。

​	让我们考虑以下场景：

* 客户端c1更新了/z节点的数据，并收到应答。

* 客户端c1通过TCP的直接连接告知客户端c2，/z节点状态发生了变化。

* 客户端c2读取/z节点的状态，但是在c1更新之前就观察到了这个状态。



​	我们称之为隐藏通道（hidden channel），因为ZooKeeper并不知道客户端之间额外的通信。现在c2获得了过期数据，图4-2描述了这种情况。

![](https://pic.imgdb.cn/item/6135cafa44eaada73986ab00.jpg)

​	为了避免读取到过去的数据，我们建议应用程序使用ZooKeeper进行所有涉及ZooKeeper状态的通信。例如，为了避免刚刚描述的场景，c2可以在/z节点设置监视点来代替从c1直接接收消息，通过监视点，c2就可以知道/z节点的变化，从而消除隐藏通道的问题。

### 4.7.3 通知的顺序

​	ZooKeeper对通知的排序涉及其他通知和异步响应，以及对系统状态更新的顺序。如ZooKeeper对两个状态更新进行排序，u和u'，u'紧随u之后，如果u和u'分别修改了/a节点和/b节点，其中客户端c在/a节点设置了监视点，c只能观察到u'的更新，即接收到u所对应通知后读取/b节点。

​	这种顺序可以使应用通过监视点实现安全的参数配置。假设一个znode节点/z被创建或删除表示在ZooKeeper中保存的一些配置信息变为无效的。在对这个配置进行任何实际更新之前，将创建或删除的通知发给客户端，这一保障非常重要，可以确保客户端不会读取到任何无效配置。

​	更具体一些，假如我们有一个znode节点/config，其子节点包含应用配置元数据：/config/m1，/config/m2，，/config/m_n。目的只是为了说明这个例子，不管这些znode节点的实际内容是什么。假如主节点应用进程通过setData更新每个znode节点，且不能让客户端只读取到部分更新，一个解决方案就是在开始更新这些配置前主节点先创建一个/config/invalid节点，其他需要读取这一状态的客户端会监视/config/invalid节点，如果该节点存在就不会读取配置状态，当该节点被删除，就意味着有一个新的有效的配置节点集合可用，客户端可以进行读取该集合的操作。

​	对于这个具体的例子，我们还可以使用multiop来对/config/m[1-n]这些节点原子地执行所有setData操作，而不是使用一个znode节点来标识部分修改的状态。在例子中的原子性问题，我们可以使用multiop代替对额外znode节点或通知的依赖，不过通知机制非常通用，而且并未约束为原子性的。

​	因为ZooKeeper根据触发通知的状态更新对通知消息进行排序，客户端就可以通过这些通知感知到真正的状态变化的顺序。

​	注意：活性与安全性

​	在本章中，因活性广泛使用了通知机制。活性（liveness）会确保系统最终取得进展。新任务和新的从节点的通知只是关于活性的事件的例子。如果主节点没有对新任务进行通知，这个任务就永远不会被执行，至少从提交任务的客户端的视角来看，已提交的任务没有执行会导致活性缺失。

​	原子更新一组配置节点的例子中，情况不太一样：这个例子涉及安全性，而不是活性。在更新中读取znode节点可能会导致客户端到非一致性配置信息，而invalid节点可以确保只有当合法配置信息有效时，客户端才读取正确状态。

​	在我们看到的关于活性的例子中，通知的传送顺序并不是特别重要，只要最终客户端最终获知这些事件就可以继续取得进展。不过为了安全性，不按顺序接收通知也许会导致不正确的行为。

### 4.8 监视点的羊群效应和可扩展性

​	有一个问题需要注意，当变化发生时，ZooKeeper会触发一个特定的znode节点的变化导致的所有监视点的集合。如果有1000个客户端通过exists操作监视这个znode节点，那么当znode节点创建后就会发送1000个通知，因而被监视的znode节点的一个变化会产生一个尖峰的通知，该尖峰可能带来影响，例如，在尖峰时刻提交的操作延迟。可能的话，我们建议在使用ZooKeeper时，避免在一个特定节点设置大量的监视点，最好是每次在特定的znode节点上，只有少量的客户端设置监视点，理想情况下最多只设置一个。

​	解决该问题的方法并不适用于所有的情况，但在以下情况下可能很有用。假设有n个客户端争相获取一个锁（例如，主节点锁）。为了获取锁，一个进程试着创建/lock节点，如果znode节点存在了，客户端就会监视这个znode节点的删除事件。当/lock被删除时，所有监视/lock节点的客户端收到通知。另一个不同的方法，让客户端创建一个有序的节点/lock/lock-，回忆之前讨论的有序节点，ZooKeeper在这个znode节点上自动添加一个序列号，成为/lock/lock-xxx，其中xxx为序列号。我们可以使用这个序列号来确定哪个客户端获得锁，通过判断/lock下的所有创建的子节点的最小序列号。在该方案中，客户端通过/getChildren方法来获取所有/lock下的子节点，并判断自己创建的节点是否是最小的序列号。如果客户端创建的节点不是最小序列号，就根据序列号确定序列，并在前一个节点上设置监视点。例如：假设我们有三个节点：/lock/lock-001、/lock/lock-002和/lock/lock-003，在这个例子中情况如下：

* 创建/lock/lock-001的客户端获得锁。

* 创建/lock/lock-002的客户端监视/lock/lock-001节点。

* 创建/lock/lock-003的客户端监视/lock/lock-002节点。



​	这样，每个节点上设置的监视点只有最多一个客户端。

​	另一方面需要注意的问题，就是当服务端一侧通过监视点产生的状态变化。设置一个监视点需要在服务端创建一个Watcher对象，根据YourKit（http://www.yourkit.com/）的分析工具所分析，设置一个监视点会使服务端的监视点管理器的内存消耗上增加大约250到300个字节，设置非常多的监视点意味着监视点管理器会消耗大量的服务器内存。例如，如果存在一百万个监视点，估计会消耗0.3GB的内存，因此，开发者必须时刻注意设置的监视点数量。

