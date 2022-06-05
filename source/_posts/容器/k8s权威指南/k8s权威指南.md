# 1. K8s入门

## 1.1　了解Kubernetes

​	Kubernetes是一个完备的分布式系统支撑平台。

​	Kubernetes提供了完善的管理工具，这些工具涵盖了包括开发、部署测试、运维监控在内的各个环节。因此，Kubernetes是一个全新的基于容器技术的分布式架构解决方案，并且是一个一站式的完备的分布式系统开发和支撑平台。

​	在Kubernetes中，Service是分布式集群架构的核心。一个Service对象拥有如下关键特征。

* 拥有唯一指定的名称（比如mysql-server）。
* 拥有一个虚拟IP地址（ClusterIP地址）和端口号。
* 能够提供某种远程服务能力。
* 能够将客户端对服务的访问请求转发到一组容器应用上。



​	Kubernetes设计了Pod对象，将每个服务进程都包装到相应的Pod中，使其成为在Pod中运行的一个容器（Container）。为了建立Service和Pod间的关联关系，Kubernetes首先给
每个Pod都贴上一个标签（Label），比如给运行MySQL的Pod贴上name=mysql标签，给运行PHP的Pod贴上name=php标签，然后给相应的Service定义标签选择器（Label Selector），例如，MySQL Service的标签选择器的选择条件为name=mysql，意为该Service要作用于所有包含name=mysql标签的Pod。这样一来，就巧妙解决了Service与Pod的关联问题。

​	这里先简单介绍Pod的概念。首先，Pod运行在一个被称为节点（Node）的环境中，这个节点既可以是物理机，也可以是私有云或者公有云中的一个虚拟机，在一个节点上能够运行多个Pod；其次，在每个Pod中都运行着一个特殊的被称为Pause的容器，其他容器则为业务容器，这些业务容器共享Pause容器的网络栈和Volume挂载卷，因此它们之间的通信和数据交换更为高效，在设计时我们可以充分利用这一特性将一组密切相关的服务进程放入同一个Pod中；最后，需要注意的是，并不是每个Pod和它里面运行的容器都能被映射到一个Service上，只有提供服务（无论是对内还是对外）的那组Pod才会被映射为一个服务。

​	在集群管理方面，Kubernetes将集群中的机器划分为一个Master和一些Node。在Master上运行着集群管理相关的一些进程：kubeapiserver、kube-controller-manager和kube-scheduler，这些进程实现了整个集群的资源管理、Pod调度、弹性伸缩、安全控制、系统监控和纠错等管理功能，并且都是自动完成的。

​	Node作为集群中的工作节点，其上运行着真正的应用程序。在Node上，Kubernetes管理的最小运行单元是Pod。在Node上运行着Kubernetes的kubelet、kube-proxy服务进程，这些服务进程负责Pod的创建、启动、监控、重启、销毁，以及实现软件模式的负载均衡器。

​	在Kubernetes集群中，只需为需要扩容的Service关联的Pod创建一个Deployment对象，服务扩容以至服务升级等令人头疼的问题就都迎刃而解了。在一个Deployment定义文件中包括以下3个关键信息。

* 目标Pod的定义。
* 目标Pod需要运行的副本数量（Replicas）。
* 要监控的目标Pod的标签。



​	在创建好Deployment之后，Kubernetes会根据这一定义创建符合要求的Pod，并且通过在Deployment中定义的Label筛选出对应的Pod实例并实时监控其状态和数量。如果实例数量少于定义的副本数量，则会根据在Deployment对象中定义的Pod模板创建一个新的Pod，然后将此Pod调度到合适的Node上启动运行，直到Pod实例的数量达到预定目标。这个过程完全是自动化的，无须人工干预。有了Deployment，服务扩容就变成一个纯粹的简单数字游戏了，只需修改Deployment中的副本数量即可。后续的服务升级也将通过修改Deployment来自动完成。

## 1.2 为什么要用Kubernetes

​	使用Kubernetes的理由很多，最重要的理由是，IT行业从来都是由新技术驱动的。Kubernetes是软件领域近几年来最具创新的容器技术，涵盖了架构、研发、部署、运维等全系列软件开发流程，不仅对互联网公司的产品产生了极大影响，也对传统行业的IT技术产生了越来越强的冲击。

**使用Kubernetes会收获哪些好处呢？**

​	首先，可以“轻装上阵”地开发复杂系统。以前需要很多人（其中不乏技术达人）一起分工协作才能设计、实现和运维的分布式系统，在采用Kubernetes解决方案之后，只需一个精悍的小团队就能轻松应对。在这个团队里，只需一名架构师负责系统中服务组件的架构设计，几名开发工程师负责业务代码的开发，一名系统兼运维工程师负责Kubernetes的部署和运维，因为Kubernetes已经帮我们做了很多。

​	其次，可以全面拥抱以微服务架构为核心思想的新一代容器技术的领先架构，包括基础的微服务架构，以及增强的微服务架构（如服务网格、无服务器架构等）。微服务架构的核心是将一个巨大的单体应用分解为很多小的相互连接的微服务，一个微服务可能由多个实例副本支撑，副本的数量可以随着系统的负荷变化进行调整。微服务架构使得每个服务都可以独立开发、升级和扩展，因此系统具备很高的稳定性和快速迭代能力，开发者也可以自由选择开发技术。谷歌、亚马逊、eBay、Netflix等大型互联网公司都采用了微服务架构，谷歌更是将微服务架构的基础设施直接打包到Kubernetes解决方案中，让我们可以直接应用微服务架构解决复杂业务系统的架构问题。

​	再次，可以随时随地将系统整体“搬迁”到公有云上。Kubernetes最初的设计目标就是让用户的应用运行在谷歌自家的公有云GCE中，华为云（CCE）、阿里云（ACK）和腾讯云（TKE）全部支持Kubernetes集群，未来会有更多的公有云及私有云支持Kubernetes。

​	然后，Kubernetes内建的服务弹性扩容机制可以让我们轻松应对突发流量。

​	最后，Kubernetes系统架构超强的横向扩容能力可以让我们的竞争力大大提升。

## 1.3　从一个简单的例子开始

### 1.3.1　环境准备

​	这里先安装Kubernetes和下载相关镜像，本书建议采用VirtualBox或者VMware Workstation在本机中虚拟一个64位的CentOS 7虚拟机作为学习环境。虚拟机采用NAT的网络模式以便连接外网，然后使用kubeadm快速安装一个Kubernetes集群（安装步骤详见2.2节的说明），之后就可以在这个Kubernetes集群中进行练习了。

# 第2章 Kubernetes安装配置指南

## 2.1 系统要求

![](https://pic.imgdb.cn/item/629c693a0947543129bf13a5.jpg)

​	Kubernetes需要容器运行时（Container Runtime Interface，CRI）的支持，目前官方支持的容器运行时包括：Docker、Containerd、CRI-O和frakti等。容器运行时的原理详见5.4.5节的说明。本节以Docker作为容器运行环境，推荐的版本为Docker CE 19.03。

​	安装docker：

```sh
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce--19.03.13-3.el8 docker-ce-cli-19.03.13-3.el8
 \ containerd.io
 
systemctl start docker	
systemctl enable docker	

cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "registry-mirrors": ["https://mr63yffu.mirror.aliyuncs.com"]
}
EOF

systemctl daemon-reload
```



​	宿主机操作系统以CentOS 7为例，使用Systemd系统完成对Kubernetes服务的配置。其他	Linux发行版的服务配置请参考相关的系统管理手册。为了便于管理，常见的做法是将Kubernetes服务程序配置为Linux系统开机自启动的服务。

​	需要注意的是，CentOS 7 默认启动了防火墙服务（firewalld.service），而Kubernetes的Master与工作Node之间会有大量的网络通信。安全的做法是在防火墙上配置各组件需要相互通信的端口号，具体要配置的端口号如表2.2所示。

![](https://pic.imgdb.cn/item/629c6a760947543129c0cb95.jpg)

​	其他组件可能还需要开通某些端口号，例如CNI网络插件calico需要179端口号；镜像库需要5000端口号等，需要根据系统要求逐个在防火墙服务上配置网络策略。

​	在安全的网络环境中，可以简单地关闭防火墙服务：

```sh
systemctl disable firewalld
systemctl stop firewalld
```

​	另外，建议在主机上禁用SELinux（修改文件/etc/sysconfig/selinux，将SELINUX=enforcing修改为SELINUX=disabled），让容器可以读取主机文件系统。随着Kubernetes对SELinux支持的增强，可以逐步启用SELinux机制，并通过Kubernetes设置容器的安全机制。

https://blog.csdn.net/weixin_38889300/article/details/122852715

## 2.2 使用kubeadm工具快速安装Kubernetes集群

​	Kubernetes从1.4版本开始引入了命令行工具kubeadm，致力于简化集群的安装过程，到Kubernetes 1.13版本时，kubeadm工具达到GA阶段。本节讲解基于kubeadm的安装过程，操作系统以CentOS 7为例。
