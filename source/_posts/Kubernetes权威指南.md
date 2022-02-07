# 1. Kubernetes入门

​	它是一个全新的基于容器技术的分布式架构领先方案。

​	在Kubernetes中，Service是分布式集群架构的核心，一个Service对象拥有如下关键特征。

* 拥有唯一指定的名称（比如mysql-server）。
* 拥有一个虚拟IP（Cluster IP、Service IP或VIP）和端口号。
* 能够提供某种远程服务能力。
* 被映射到提供这种服务能力的一组容器应用上。



​	这里先简单介绍Pod的概念。首先，Pod运行在一个被称为节点（Node）的环境中，这个节点既可以是物理机，也可以是私有云或者公有云中的一个虚拟机，通常在一个节点上运行几百个Pod；其次，在每个Pod中都运行着一个特殊的被称为Pause的容器，其他容器则为业务容器，这些业务容器共享Pause容器的网络栈和Volume挂载卷，因此它们之间的通信和数据交换更为高效，在设计时我们可以充分利用这一特性将一组密切相关的服务进程放入同一个Pod中；最后，需要注意的是，并不是每个Pod和它里面运行的容器都能被映射到一个Service上，只有提供服务（无论是对内还是对外）的那组Pod才会被映射为一个服务。

​	在Kubernetes集群中，只需为需要扩容的Service关联的Pod创建一个RC（Replication Controller），服务扩容以至服务升级等令人头疼的问题都迎刃而解。在一个RC定义文件中包括以下3个关键信息。

* 目标Pod的定义。
* 目标Pod需要运行的副本数量（Replicas）。
* 要监控的目标Pod的标签。

## 1.3 从一个简单的例子开始

略

## 1.4 Kubernetes的基本概念和术语

​	Kubernetes平台采用了“核心+外围扩展”的设计思路，在保持平台核心稳定的同时具备持续演进升级的优势。Kubernetes大部分常见的核心资源对象都归属于v1这个核心API，比如Node、Pod、Service、Endpoints、Namespace、RC、PersistentVolume等。

​	随着Kubernetes版本的持续升级，一些资源对象会不断引入新的属性。为了在不影响当前功能的情况下引入对新特性的支持，我们通常会采用下面两种典型方法。

* 方法1，在设计数据库表的时候，在每个表中都增加一个很长的备注字段，之后扩展的数据以某种格式（如XML、JSON、简单字符串拼接等）放入备注字段。因为数据库表的结构没有发生变化，所以此时程序的改动范围是最小的，风险也更小，但看起来不太美观。
* 方法2，直接修改数据库表，增加一个或多个新的列，此时程序的改动范围较大，风险更大，但看起来比较美观。



​	显然，两种方法都不完美。更加优雅的做法是，先采用方法1实现这个新特性，经过几个版本的迭代，等新特性变得稳定成熟了以后，可以在后续版本中采用方法2升级到正式版。为此，Kubernetes为每个资源对象都增加了类似数据库表里备注字段的通用属性Annotations，以实现方法1的升级。

### 1.4.1 Master

​	Kubernetes里的Master指的是集群控制节点，在每个Kubernetes集群里都需要有一个Master来负责整个集群的管理和控制，基本上Kubernetes的所有控制命令都发给它，它负责具体的执行过程，我们后面执行的所有命令基本都是在Master上运行的。Master通常会占据一个独立的服务器（高可用部署建议用3台服务器），主要原因是它太重要了，是整个集群的“首脑”，如果它宕机或者不可用，那么对集群内容器应用的管理都将失效。

​	在Master上运行着以下关键进程。

* Kubernetes API Server（kube-apiserver）：提供了HTTP Rest接口的关键服务进程，是Kubernetes里所有资源的增、删、改、查等操作的唯一入口，也是集群控制的入口进程。
* Kubernetes Controller Manager（kube-controller-manager）：Kubernetes里所有资源对象的自动化控制中心，可以将其理解为资源对象的“大总管”。
* Kubernetes Scheduler（kube-scheduler）：负责资源调度（Pod调度）的进程，相当于公交公司的“调度室”

### 1.4.2 Node

​	除了Master，Kubernetes集群中的其他机器被称为Node，在较早的版本中也被称为Minion。与Master一样，Node可以是一台物理主机，也可以是一台虚拟机。Node是Kubernetes集群中的工作负载节点，每个Node都会被Master分配一些工作负载（Docker容器），当某个Node宕机时，其上的工作负载会被Master自动转移到其他节点上。

在每个Node上都运行着以下关键进程。

* kubelet：负责Pod对应的容器的创建、启停等任务，同时与Master密切协作，实现集群管理的基本功能。
*  kube-proxy：实现Kubernetes Service的通信与负载均衡机制的重要组件。
* Docker Engine（docker）：Docker引擎，负责本机的容器创建和管理工作。

# 2. Kubernetes安装配置指南

## 2.2使用kubeadm工具快速安装Kubernetes集群

/etc/yum.repos.d/kubernetes.repo

```properties
[kubernetes]
name=Kubernetes Repository
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
```

然后运行yum install命令安装kubeadm和相关工具:

```sh
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

​	运行下面的命令，启动Docker服务（如果已安装Docker，则无须再次启动）和kubelet服务，并设置为开机自动启动：

