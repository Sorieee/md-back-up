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

### 2.2.1 安装kubeadm 

​	首先配置yum源，官方yum源配置文件`/etc/yum.repos.d/kubernetes.repo`的内容如下：

```sh
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes Repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
enabled=1
EOF

yum install -y kubelet kubeadm kubectl

systemctl start kubelet
systemctl enable kubelet
systemctl enable kubelet.service
```

​	kubeadm将使用kubelet服务以容器方式部署和启动Kubernetes的主要服务，所以需要先启动kubelet服务。运行systemctl start命令启动kubelet服务，并设置为开机自启动：

**交换区**

​	kubeadm还需要关闭Linux的swap系统交换区，这可以通过`swapoff -a` 命令实现：

```sh
swapoff -a ; sed -i '/swap/d' /etc/fstab
```

**允许路由转发，不对bridge的数据进行处理**

```sh
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

**修改host文件**

```sh
vim /etc/hosts
192.168.0.200 node1
192.168.0.201 node2
192.168.0.202 node3
```

### 2.2.2 修改kubeadm的默认配置

kubeadm config子命令提供了对这组功能的支持。

* kubeadm config print init-defaults：输出kubeadm init命令默认参数的内容。
* kubeadm config print join-defaults：输出kubeadm join命令默认参数的内容。
* kubeadm config migrate：在新旧版本之间进行配置转换。
* kubeadm config images list：列出所需的镜像列表。
* kubeadm config images pull：拉取镜像到本地。

例如，运行kubeadm config print init-defaults命令，可以获得默认的初始化参数文件：

```sh
kubeadm config print init-defaults > init.default.yaml
```

​	对生成的文件进行编辑，可以按需生成合适的配置。例如，若需要自定义镜像的仓库地址、需要安装的Kubernetes版本号及Pod的IP地址范围，则可以将默认配置修改如下：

![](https://pic.imgdb.cn/item/629d20dc09475431298eba02.jpg)

### 2.2.3 下载Kubernetes的相关镜像

https://www.jianshu.com/p/a4001280e264

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: docker.io/dustise
kind: ClusterConfiguration
kubernetesVersion: 1.24.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 192.168.0.0/16
scheduler: {}
```

将上面的内容保存为init-config.yaml备用

### 2.2.3 下载Kubernetes的相关镜像

​	为了加快kubeadm创建集群的过程，可以预先将所需镜像下载完成。可以通过kubeadm config images list命令查看镜像列表，例如：

```sh
k8s.gcr.io/kube-apiserver:v1.24.1
k8s.gcr.io/kube-controller-manager:v1.24.1
k8s.gcr.io/kube-scheduler:v1.24.1
k8s.gcr.io/kube-proxy:v1.24.1
k8s.gcr.io/pause:3.7
k8s.gcr.io/etcd:3.5.3-0
k8s.gcr.io/coredns/coredns:v1.8.6
```

如果无法访问k8s.gcr.io，则可以使用国内镜像托管站点进行下载，
例如https://1nj0zren.mirror.aliyuncs.com，这可以通过修改Docker服务
的配置文件（默认为/etc/docker/daemon.json）进行设置。

然后，使用kubeadm config images pull命令或者docker pull命令下载上述镜像，例如：

```sh
kubeadm config images pull --config=init-config.yaml
```

unknown service runtime.v1alpha2.ImageService

https://blog.csdn.net/weixin_40668374/article/details/124849090

### 2.2.4 运行kubeadm init命令安装Master节点

​	至此，准备工作已经就绪，运行kubeadm init命令即可一键安装Kubernetes的Master节点，也称之为Kubernetes控制平面（Control Plane）。

​	在开始之前需要注意：kubeadm的安装过程不涉及网络插件（CNI）的初始化，因此kubeadm初步安装完成的集群不具备网络功能，任何Pod（包括自带的CoreDNS）都无法正常工作。而网络插件的安装往往对kubeadm init命令的参数有一定要求。例如，安装Calico插件时需要指定--pod-network-cidr=192.168.0.0/16。关于安装CNI网络插件的更多内容，可参考官方文档的说明。

​	kubeadm init命令在执行具体的安装操作之前，会执行一系列被称为pre-flight checks的系统预检查，以确保主机环境符合安装要求，如果检查失败就直接终止，不再进行init操作。用户可以通过kubeadm init phase preflight命令执行预检查操作，确保系统就绪后再执行init操作。
如果不希望执行预检查，则也可以为kubeadm init命令添加--ignorepreflight-errors参数进行关闭。如表2.3所示是kubeadm检查的系统配置，对不符合要求的检查项以warning或error级别的信息给出提示。

![](https://pic.imgdb.cn/item/629d2558094754312990a491.jpg)

![](https://pic.imgdb.cn/item/629d256b094754312990ab89.jpg)

​	另外，Kubernetes默认设置cgroup驱动（cgroupdriver）为“systemd”，而Docker服务的cgroup驱动默认值为“cgroupfs”，建议将其修改为“systemd”，与Kubernetes保持一致。这可以通过修改Docker服务的配置文件（默认为/etc/docker/daemon.json）进行设置：

```sh
{
  "registry-mirrors": ["https://mr63yffu.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}

systemctl daemon-reload
```

​	准备工作就绪之后，就可以运行kubeadm init命令，使用之前创建的配置文件一键安装Master节点（控制平面）了：

```sh
dnf install -y iproute-tc
kubeadm init --config=init-config.yaml
```

​	看到“Your Kubernetes control-plane has initialized successfully！”的提示，就说明Master节点（控制平面）已经安装成功了。

​	接下来就可以通过kubectl命令行工具访问集群进行操作了。由于kubeadm默认使用CA证书，所以需要为kubectl配置证书才能访问Master。

​	按照安装成功的提示，非root用户可以将admin.conf配置文件复制到HOME目录的.kube子目录下，命令如下：

![](https://pic.imgdb.cn/item/629d27cc094754312991adbe.jpg)

```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```

​	然后就可以使用kubectl命令行工具对Kubernetes集群进行访问和操作了。

​	例如查看命名空间kube-system中的ConfigMap列表：

```sh
kubectl -n kube-system get configmap
```

​	到此，Kubernetes的Master节点已经可以工作了，但在集群内还是没有可用的Worker Node，并缺乏容器网络的配置。

​	接下来安装Worker Node，需要用到kubeadm init命令运行完成后的最后几行提示信息，其中包含将节点加入集群的命令（kubeadm join）和所需的Token。

**"Error getting node" err="node \"node1\" not found**

https://blog.csdn.net/huotongwangbs/article/details/121159429

**The connection to the server localhost:8080 was refused**

https://blog.csdn.net/CEVERY/article/details/108753379

**kubectl get pod卡住的问题**

https://blog.csdn.net/hawk2014bj/article/details/109807413

journalctl -xefu kubelet



https://blog.csdn.net/m0_37944592/article/details/122824531

## 2.3 以二进制文件方式安装Kubernetes安全高可用集群

​	通过kubeadm能够快速部署一个Kubernetes集群，但是如果需要精细调整Kubernetes各组件服务的参数及安全设置、高可用模式等，管理员就可以使用Kubernetes二进制文件进行部署。

### 2.3.1 Master高可用部署架构

​	在Kubernetes系统中，Master节点扮演着总控中心的角色，通过不间断地与各个工作节点（Node）通信来维护整个集群的健康工作状态，集群中各资源对象的状态则被保存在etcd数据库中。如果Master不能正常工作，各Node就会处于不可管理状态，用户就无法管理在各Node上运行的Pod，其重要性不言而喻。同时，如果Master以不安全方式提供服务（例如通过HTTP的8080端口号），则任何能够访问Master的客户端都可以通过API操作集群中的数据，可能导致对数据的非法访问或篡改。

在正式环境中应确保Master的高可用，并启用安全访问机制，至少包括以下几方面。

* Master的kube-apiserver、kube-controller-mansger和kubescheduler服务至少以3个节点的多实例方式部署。
* Master启用基于CA认证的HTTPS安全机制。
* etcd至少以3个节点的集群模式部署。
* etcd集群启用基于CA认证的HTTPS安全机制。
* Master启用RBAC授权模式（详见6.2节的说明）。

![](https://pic.imgdb.cn/item/62a1f9f10947543129fe7ff0.jpg)

​	在Master的3个节点之前，应通过一个负载均衡器提供对客户端的唯一访问入口地址，负载均衡器可以选择硬件或者软件进行搭建。软件负载均衡器可以选择的方案较多，本文以HAProxy搭配Keepalived为例进行说明。主流硬件负载均衡器有F5、A10等，需要额外采购，其负载均衡配置规则与软件负载均衡器的配置类似，本文不再赘述。

​	本例中3台主机的IP地址分别为192.168.18.3、192.168.18.4、192.168.18.5，负载均衡器使用的VIP为192.168.18.100。

​	下面分别对etcd、负载均衡器、Master、Node等组件如何进行高可用部署、关键配置、CA证书配置等进行详细说明。

### 2.3.2 创建CA根证书

​	为etcd和Kubernetes服务启用基于CA认证的安全机制，需要CA证书进行配置。如果组织能够提供统一的CA认证中心，则直接使用组织颁发的CA证书即可。如果没有统一的CA认证中心，则可以通过颁发自签名的CA证书来完成安全配置。

​	etcd和Kubernetes在制作CA证书时，均需要基于CA根证书，本文以为Kubernetes和etcd使用同一套CA根证书为例，对CA证书的制作进行说明。

​	CA证书的制作可以使用openssl、easyrsa、cfssl等工具完成，本文以openssl为例进行说明。下面是创建CA根证书的命令，包括私钥文件ca.key和证书文件ca.crt：

```sh
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=192.168.0.200" -days 36500 -out ca.crt
mkdir -p /etc/kubernetes/pki
cp ca.key /etc/kubernetes/pk
cp ca.crt /etc/kubernetes/pki
```

主要参数如下。

* -subj：“/CN”的值为Master主机名或IP地址。
* -days：设置证书的有效期。

将生成的ca.key和ca.crt文件保存在/etc/kubernetes/pki目录下。

### 2.3.3　部署安全的etcd高可用集群

​	etcd作为Kubernetes集群的主数据库，在安装Kubernetes各服务之前需要首先安装和启动。

**1.下载etcd二进制文件，配置systemd服务**

​	从GitHub官网下载etcd二进制文件，例如etcd-v3.4.13-linuxamd64.tar.gz，如图2.2所示。

![](https://pic.imgdb.cn/item/62a1fbcb0947543129013a6d.jpg)

​	解压缩后得到etcd和etcdctl文件，将它们复制到/usr/bin目录下。

​	然后将其部署为一个systemd的服务，创建systemd服务配置文件`/usr/lib/systemd/system/etcd.service`，内容示例如下：

```sh
[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network.target

[Service]
EnviromentFile=/etc/etcd/etcd.conf
ExecStart=/usr/bin/etcd
Restart=always

[Install]
WantedBy=multi-user.target
```

​	其中，EnvironmentFile指定配置文件的全路径，例如`/etc/etcd/etcd.conf`，其中的参数以环境变量的格式进行配置。

​	接下来先对etcd需要的CA证书配置进行说明。对于配置文件`/etc/etcd/etcd.conf`中的完整配置参数，将在创建完CA证书后统一说明。

**2.创建etcd的CA证书**

​	先创建一个x509 v3配置文件etcd_ssl.cnf，其中subjectAltName参数（alt_names）包括所有etcd主机的IP地址，例如：

```sh
[req]
req_extensions=v3_req
distinguished_name=req_distinguished_name

[req_distinguished_name]

[v3_req]
basicConstraints=CA:FALSE
keyUsage=nonRepudiation,digitalSignature,keyEncipherment
subjectAltName=@alt_names

[alt_names]
IP.1=192.168.0.200
IP.2=192.168.0.201
IP.3=192.168.0.203
```

然后使用openssl命令创建etcd的服务端CA证书，包括`etcd_server.key`和`etcd_server.crt`文件，将其保存到`/etc/etcd/pki`目录下：

```sh
openssl genrsa -out etcd_server.key 2048
openssl req -new -key etcd_server.key -config etcd_ssl.cnf -subj "/CN=etcd-server" -out etcd_server.crt
openssl x509 -req -in etcd_server.scr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 36500 -extension v3_req -extfile etcd_ssl.conf -out etcd_server.crt
```

​	再创建客户端使用的CA证书，包括etcd_client.key和etcd_client.crt文
件，也将其保存到`/etc/etcd/pki`目录下，后续供kube-apiserver连接etcd时使用：

```sh
openssl genrsa -out etcd_client.key 2048
openssl req -new -key etcd_client.key -config etcd_ssl.cnf -subj "/CN=etcd-client" -out etcd_client.crt
openssl x509 -req -in etcd_client.scr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 36500 -extension v3_req -extfile etcd_ssl.conf -out etcd_client.crt
```

**3.etcd参数配置说明**

​	接下来对3个etcd节点进行配置。etcd节点的配置方式包括启动参数、环境变量、配置文件等，本例使用环境变量方式将其配置到`/etc/etcd/etcd.conf`文件中，供systemd服务读取。

​	3个etcd节点将被部署在192.168.0.200、192.168.0.201和192.168.0.203 3台主机上，配置文件`/etc/etcd/etcd.conf`的内容示例如下：

```properties
#节点1的配置
ETCD_NAME=etcd1
ETCD_DATA_DIR=/etc/etcd/data

ETCD_CERT_FILE=/etc/etcd/pki/etcd_server.crt
ETCD_KEY_FILE=/etc/etcd/pki/etcd_server.key
ETCD_TRUSTED_CA_FILE=/etc/kubernetes/pki/ca.crt
ETCD_CLIENT_CERT_AUTH=true
ETCD_LISTEN_CLIENT_URLS=https://192.168.0.200:2379
ETCD_ADVERTISE_CLIENT_URLS=https://192.168.0.200:2379

ETCD_PEER_CERT_FILE=/etc/etcd/pki/etcd_server.crt
ETCD_PEER_KEY_FILE=/etc/etcd/pki/etcd_server.key
ETCD_PEER_TRUSTED_CA_FILE=/etc/kubernetes/pki/ca.crt
ETCD_LISTEN_PEER_URLS=https://192.168.0.200:2380
ETCD_INITIAL_ADVERTISE_PEER_URLs=https://192.168.0.200:2380

ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
ETCD_INITIAL_CLUSTER="etcd1=https://192.168.0.200:2380,etcd2=https://192.168.0.201:2380,etcd3=https://192.168.0.203:2380"
ETCD_INITIAL_CLUSTER_STATE=new
```

```properties
#节点2的配置
ETCD_NAME=etcd2
ETCD_DATA_DIR=/etc/etcd/data

ETCD_CERT_FILE=/etc/etcd/pki/etcd_server.crt
ETCD_KEY_FILE=/etc/etcd/pki/etcd_server.key
ETCD_TRUSTED_CA_FILE=/etc/kubernetes/pki/ca.crt
ETCD_CLIENT_CERT_AUTH=true
ETCD_LISTEN_CLIENT_URLS=https://192.168.0.201:2379
ETCD_ADVERTISE_CLIENT_URLS=https://192.168.0.201:2379

ETCD_PEER_CERT_FILE=/etc/etcd/pki/etcd_server.crt
ETCD_PEER_KEY_FILE=/etc/etcd/pki/etcd_server.key
ETCD_PEER_TRUSTED_CA_FILE=/etc/kubernetes/pki/ca.crt
ETCD_LISTEN_PEER_URLS=https://192.168.0.201:2380
ETCD_INITIAL_ADVERTISE_PEER_URLs=https://192.168.0.201:2380

ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
ETCD_INITIAL_CLUSTER="etcd1=https://192.168.0.200:2380,etcd2=https://192.168.0.201:2380,etcd3=https://192.168.0.203:2380"
ETCD_INITIAL_CLUSTER_STATE=new
```

```sh
# 节点3的配置
ETCD_NAME=etcd3
ETCD_DATA_DIR=/etc/etcd/data

ETCD_CERT_FILE=/etc/etcd/pki/etcd_server.crt
ETCD_KEY_FILE=/etc/etcd/pki/etcd_server.key
ETCD_TRUSTED_CA_FILE=/etc/kubernetes/pki/ca.crt
ETCD_CLIENT_CERT_AUTH=true
ETCD_LISTEN_CLIENT_URLS=https://192.168.0.203:2379
ETCD_ADVERTISE_CLIENT_URLS=https://192.168.0.203:2379

ETCD_PEER_CERT_FILE=/etc/etcd/pki/etcd_server.crt
ETCD_PEER_KEY_FILE=/etc/etcd/pki/etcd_server.key
ETCD_PEER_TRUSTED_CA_FILE=/etc/kubernetes/pki/ca.crt
ETCD_LISTEN_PEER_URLS=https://192.168.0.203:2380
ETCD_INITIAL_ADVERTISE_PEER_URLs=https://192.168.0.203:2380

ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
ETCD_INITIAL_CLUSTER="etcd1=https://192.168.0.200:2380,etcd2=https://192.168.0.201:2380,etcd3=https://192.168.0.203:2380"
ETCD_INITIAL_CLUSTER_STATE=new
```

主要配置参数包括为客户端和集群其他节点配置的各监听URL地址
（均为HTTPS URL地址），并配置相应的CA证书参数。

​	etcd服务相关的参数如下。

* ETCD_NAME：etcd节点名称，每个节点都应不同，例如etcd1、etcd2、etcd3。
* ETCD_DATA_DIR：etcd数据存储目录，例如`/etc/etcd/data/etcd1`。

* ETCD_LISTEN_CLIENT_URLS和
  ETCD_ADVERTISE_CLIENT_URLS：为客户端提供的服务监听URL地址，例如``https//192.168.18.32379`。

* ETCD_LISTEN_PEER_URLS和
  ETCD_INITIAL_ADVERTISE_PEER_URLS：为本集群其他节点提供的
  服务监听URL地址，例如`https://192.168.18.3:2380`。

* ETCD_INITIAL_CLUSTER_TOKEN：集群名称，例如etcdcluster
* ETCD_INITIAL_CLUSTER：集群各节点的endpoint列表，例
  如`"etcd1=https://192.168.18.3:2380,etcd2=https://192.168.18.4:2380,etcd3=https://192.168.18.5:2380"`

* ETCD_INITIAL_CLUSTER_STATE：初始集群状态，新建集群时设置为“new”，集群已存在时设置为“existing”。
* CA证书相关的配置参数如下。

* ETCD_CERT_FILE：etcd服务端CA证书-crt文件全路径，例如/etc/etcd/pki/etcd_server.crt。
* ETCD_KEY_FILE：etcd服务端CA证书-key文件全路径，例如/etc/etcd/pki/etcd_server.key。
* ETCD_TRUSTED_CA_FILE：CA根证书文件全路径，例如/etc/kubernetes/pki/ca.crt。
* ETCD_CLIENT_CERT_AUTH：是否启用客户端证书认证。
* ETCD_PEER_CERT_FILE：集群各节点相互认证使用的CA证书-crt文件全路径，例如/etc/etcd/pki/etcd_server.crt。
* ETCD_PEER_KEY_FILE：集群各节点相互认证使用的CA证书-key文件全路径，例如/etc/etcd/pki/etcd_server.key。
* ETCD_PEER_TRUSTED_CA_FILE：CA根证书文件全路径，例如/etc/kubernetes/pki/ca.crt。

**4.启动etcd集群**

基于systemd的配置，在3台主机上分别启动etcd服务，并设置为开机自启动：

```sh
systemctl restart etcd & systemctl enable etcd
```

​	然后用etcdctl客户端命令行工具携带客户端CA证书，运行`etcdctl endpoint health`命令访问etcd集群，验证集群状态是否正常，命令如下：

```sh
etcdctl --cacert=/etc/kubernetes/pki/ca.crt --cert=/etc/etcd/pki/etcd_client.crt --key=/etc/etcd/pki/etcd_client.key --endpoints=https://192.168.0.200:2379,https://192.168.0.201:2379,https://192.168.0.203:2379 endpoint health
```

结果显示各节点状态均为“healthy”，说明集群正常运行。

至此，一个启用了HTTPS的3节点etcd集群就部署完成了，更多的配置参数请参考etcd官方文档的说明。

### 2.3.4 部署安全的Kubernetes Master高可用集群

**1.下载Kubernetes服务的二进制文件**

​	首先，从Kubernetes的官方GitHub代码库页面下载各组件的二进制文件，在Releases页面找到需要下载的版本号，单击CHANGELOG链接，跳转到已编译好的Server端二进制（Server Binaries）文件的下载页面进行下载，如图2.3和图2.4所示。

![](https://pic.imgdb.cn/item/62a2035309475431290c38d4.jpg)

​	在压缩包kubernetes.tar.gz内包含了Kubernetes的全部服务二进制文件和容器镜像文件，也可以分别下载Server Binaries和Node Binaries二进制文件。在Server Binaries中包含不同系统架构的服务端可执行文件，例如kubernetes-server-linux-amd64.tar.gz文件包含了x86架构下Kubernetes需
要运行的全部服务程序文件；Node Binaries则包含了不同系统架构、不同操作系统的Node需要运行的服务程序文件，包括Linux版和Windows版等。

![](https://pic.imgdb.cn/item/62a2039909475431290ca05e.jpg)

​	在Kubernetes的Master节点上需要部署的服务包括etcd、kubeapiserver、kube-controller-manager和kube-scheduler。

​	在工作节点（Worker Node）上需要部署的服务包括docker、kubelet和kube-proxy。

​	将Kubernetes的二进制可执行文件复制到/usr/bin目录下，然后在/usr/lib/systemd/system目录下为各服务创建systemd服务配置文件（完整的systemd系统知识请参考Linux的相关手册），这样就完成了软件的安装。

**2.部署kube-apiserver服务**

（1）设置kube-apiserver服务需要的CA相关证书。准备master_ssl.cnf文件用于生成x509 v3版本的证书，示例如下：

```
[req]
req_extensions=v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]

[ v3_req ]
basicContraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
DNS.5 = k8s-1
DNS.6 = k8s-2
DNS.7 = k8s-3
IP.1 = 169.169.0.1
IP.2 = 192.168.0.200
IP.3 = 192.168.0.201
IP.4 = 192.168.0.203
IP.5 = 192.168.0.100
```

​	在该文件中主要需要在subjectAltName字段（[alt_names]）设置Master服务的全部域名和IP地址，包括：

* DNS主机名，例如k8s-1、k8s-2、k8s-3等；
* Master Service虚拟服务名称，例如kubernetes.default等；
* IP地址，包括各kube-apiserver所在主机的IP地址和负载均衡器的IP地址，例如192.168.18.3、192.168.18.4、192.168.18.5和192.168.18.100；
* Master Service虚拟服务的ClusterIP地址，例如169.169.0.1。



​	然后使用openssl命令创建kube-apiserver的服务端CA证书，包括apiserver.key和apiserver.crt文件，将其保存到/etc/kubernetes/pki目录下：

```sh
openssl genrsa -out apiserver.key 2048
openssl req -new -key apiserver.key -config master_ssl.cnf -subj "/CN=192.168.0.200" -out apiserver.crt
openssl x509 -req -in apiserver.scr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 36500 -extension v3_req -extfile master_ssl.conf -out apiserver.crt
```

（2）为kube-apiserver服务创建systemd服务配置文件/usr/lib/systemd/system/kube-apiserver.service，内容如下：

```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=/etc/kubernetes/apiserver
ExecStart=/usr/bin/kube-apiserver $KUBE_API_ARGS
Restart=always

[Install]
WantedBy=multi-user.target
```

（3）配置文件/etc/kubernetes/apiserver的内容通过环境变量KUBE_API_ARGS设置kube-apiserver的全部启动参数，包含CA安全配置的启动参数示例如下：

```
KUBE_API_ARGS="--insecure-port=0\
--secure-port=6443 \
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt \
--tls-private-key-file=/etc/kubernetes/pki/apiserver.key \
--client-ca-file=/etc/kubernetes/pki/ca.crt \
--apiserver-count=3 --endpoint-reconciler-type=master-count \
--etcd-servers=https://192.168.0.200:2379,https://192.168.0.201:2379,https://192.168.0.203:2379 \
--etcd-cafile=/etc/kubernetes/pki/ca.crt \
--etcd-certfile=/etc/etcd/pki/etcd_client.crt \
--etcd-keyfile=/etc/etcd/pki/etcd_client.key \
--service-cluster-ip-range=169.169.0.0/16 \
--service-node-port-range=30000-32767 \
--allow-privileged=true \
--logtostderr=false \
--log-dir=/var/log/kubernetes \
--v=0"
```

（4）在配置文件准备完毕后，在3台主机上分别启动kube-apiserver服务，并设置为开机自启动：

```sh
systemctl start kube-apiserver && systemctl enable kube-apiserver
```

**3.创建客户端CA证书**

​	kube-controller-manager、kube-scheduler、kubelet和kube-proxy服务作为客户端连接kube-apiserver服务，需要为它们创建客户端CA证书进行访问。这里以对这几个服务统一创建一个证书作为示例。

（1）通过openssl工具创建CA证书和私钥文件，命令如下：

```sh
openssl genrsa -out client.key 2048
openssl req -new -key client.key -subj "/CN=admin" -out client.crt
openssl x509 -req -in client.scr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 36500 -out client.crt
```

​	其中，-subj参数中“/CN”的名称可以被设置为“admin”，用于标识连接kube-apiserver的客户端用户的名称。

（2）将生成的client.key和client.crt文件保存在/etc/kubernetes/pki目录下。

**4.创建客户端连接kube-apiserver服务所需的kubeconfig配置文件**

​	本节为kube-controller-manager、kube-scheduler、kubelet和kubeproxy服务统一创建一个kubeconfig文件作为连接kube-apiserver服务的配置文件，后续也作为kubectl命令行工具连接kube-apiserver服务的配置文件。

​	在Kubeconfig文件中主要设置访问kube-apiserver的URL地址及所需CA证书等的相关参数，示例如下：

```yml
apiVersion: v1
kind: Config
clusters:
- name: default
  cluster:
    server: https://192.168.0.100:9443
    certificate-authority: /etc/kubernetes/pki/ca.crt
users:
- name: admin
  user:
    client-certificate: /etc/kubernetes/pki/client.crt
    client-key: /etc/kubernetes/pki/client.key
contexts:
- context:
    cluster: default
    user: admin
  name: default
current-context: default
```

其中的关键配置参数如下。

* server URL地址：配置为负载均衡器（HAProxy）使用的VIP地
  址（如192.168.18.100）和HAProxy监听的端口号（如9443）。
* client-certificate：配置为客户端证书文件（client.crt）全路径。
* client-key：配置为客户端私钥文件（client.key）全路径。
* certificate-authority：配置为CA根证书（ca.crt）全路径。
* users中的user name和context中的user：连接API Server的用户名，设置为与客户端证书中的“/CN”名称保持一致，例如“admin”。

将kubeconfig文件保存到/etc/kubernetes目录下。

**5.部署kube-controller-manager服务**

（1）为kube-controller-manager服务创建systemd服务配置文件/usr/lib/systemd/system/kube-controller-manager.service，内容如下：

```
# /usr/lib/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=/etc/kubernetes/controller-manager
ExecStart=/usr/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_ARGS
Restart=always

[Install]
WantedBy=multi-user.target
```

（2）配置文件/etc/kubernetes/controller-manager的内容为通过环境变量KUBE_CONTROLLER_MANAGER_ARGS设置的kube-controllermanager的全部启动参数，包含CA安全配置的启动参数示例如下：

```sh
# /etc/kubernetes/controller-manager
KUBE_CONTROLLER_MANAGER_ARGS="--kubeconfig=/etc/kubernetes/kubeconfig \
--leader-elect=true \
--service-cluster-ip-range=169.169.0.0/16 \
--service-account-private-key-file=/etc/kubernetes/pki/apiserver.key \
--root-ca-file=/etc/kubernetes/pki/ca.crt \
--log-dir=/var/log/kubernetes --logtostderr=false --v=0"
```



systemctl start kube-controller-manager && systemctl enable kube-controller-manager

对主要参数说明如下。

* --kubeconfig：与API Server连接的相关配置。

* --leader-elect：启用选举机制，在3个节点的环境中应被设置为
  true。

* --service-account-private-key-file：为ServiceAccount自动颁发
  token使用的私钥文件全路径，例如/etc/kubernetes/pki/apiserver.key。

* --root-ca-file：CA根证书全路径，例
  如/etc/kubernetes/pki/ca.crt。

* --service-cluster-ip-range：Service虚拟IP地址范围，以CIDR格式表示，例如169.169.0.0/16，与kube-apiserver服务中的配置保持一致。

  

（3）配置文件准备完毕后，在3台主机上分别启动kube-controllermanager服务，并设置为开机自启动：

```sh
systemctl start kube-controller-manager && systemctl enable kube-controller-manager
```

**6.部署kube-scheduler服务**

（1）为kube-scheduler服务创建systemd服务配置文件/usr/lib/systemd/system/kube-scheduler.service，内容如下：

```sh
# /usr/lib/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=/etc/kubernetes/scheduler
ExecStart=/usr/bin/kube-scheduler $KUBE_SCHEDULER_ARGS
Restart=always

[Install]
WantedBy=multi-user.target
```

（2）配置文件/etc/kubernetes/scheduler的内容为通过环境变量KUBE_SCHEDULER_ARGS设置的kube-scheduler的全部启动参数，示例如下：

```sh
# /etc/kubernetes/scheduler
KUBE_SCHEDULER_ARGS="--kubeconfig=/etc/kubernetes/kubeconfig \
--leader-elect=true \
--logtostderr=false --log-dir=/var/log/kubernetes --v=0"
```

对主要参数说明如下。

* --kubeconfig：与API Server连接的相关配置。
* --leader-elect：启用选举机制，在3个节点的环境中应被设置为
  true。

（3）在配置文件准备完毕后，在3台主机上分别启动kube-scheduler服务，并设置为开机自启动：

```sh
systemctl start kube-scheduler && systemctl enable kube-scheduler
```

​	通过systemctl status <service_name>验证服务的启动状态，状态为running并且没有报错日志表示启动成功，例如：（略)

**7.使用HAProxy和keepalived部署高可用负载均衡器**

​	接下来，在3个kube-apiserver服务的前端部署HAProxy和keepalived，使用VIP 192.168.18.100作为Master的唯一入口地址，供客户端访问。

​	将HAProxy和keepalived均部署为至少有两个实例的高可用架构，以避免单点故障。下面以在192.168.18.3和192.168.18.4两台服务器上部署为例进行说明。HAProxy负责将客户端请求转发到后端的3个kubeapiserver实例上，keepalived负责维护VIP 192.168.18.100的高可用。HAProxy和keepalived的部署架构如图2.5所示。

![](https://pic.imgdb.cn/item/62a3fc8809475431292ea48f.jpg)

接下来对部署HAProxy和keepalived组件进行说明。
1）部署两个HAProxy实例
准备HAProxy的配置文件haproxy.cfg，内容示例如下：

```cfg
# haproxy.cfg
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4096
    user        haproxy
    group       haproxy
    daemon
    stats socket /var/lib/haproxy/stats

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option                  http-server-close
    option                  forwardfor    except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
frontend  kube-apiserver
    mode                 tcp
    bind                 *:9443
    option               tcplog
    default_backend      kube-apiserver

listen stats
    mode                 http
    bind                 *:8888
    stats auth           admin:password
    stats refresh        5s
    stats realm          HAProxy\ Statistics
    stats uri            /stats
    log                  127.0.0.1 local3 err

backend kube-apiserver
    mode        tcp
    balance     roundrobin
    server  k8s-master1 192.168.0.200:6443 check
    server  k8s-master2 192.168.0.201:6443 check
    server  k8s-master3 192.168.0.203:6443 check
```

对主要参数说明如下。

* frontend：HAProxy的监听协议和端口号，使用TCP，端口号为
  9443。
* backend：后端3个kube-apiserver的地址，以IP：Port方式表
  示，例如192.168.18.3：6443、192.168.18.4：6443和192.168.18.5：
  6443；mode字段用于设置协议，此处为tcp；balance字段用于设置负载
  均衡策略，例如roundrobin为轮询模式。
* listen stats：状态监控的服务配置，其中，bind用于设置监听端口号为8888；stats auth用于配置访问账号；stats uri用于配置访问URL路径，例如/stats。

下面以Docker容器方式运行HAProxy且镜像使用haproxytech/haproxy-debian为例进行说明。

​	在两台服务器192.168.18.3和192.168.18.4上启动HAProxy，将配置文件haproxy.cfg挂载到容器的/usr/local/etc/haproxy目录下，启动命令如下：

```sh
docker run -d --name k8s-haproxy \
  --net=host \
  --restart=always \
  -v ${PWD}/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro \
  haproxytech/haproxy-debian:2.3
```

在一切正常的情况下，通过浏览器访问http://192.168.0.200:8888/stats地址即可访问HAProxy的管理页面，登录后查看到的主页界面如图2.6所示。

![](https://pic.imgdb.cn/item/62a3fea40947543129310a45.jpg)

​	这里主要关注最后一个表格，其内容为haproxy.cfg配置文件中backend配置的3个kube-apiserver地址，它们的状态均为“UP”，表示与3个kube-apiserver服务成功建立连接，说明HAProxy工作正常。

**2）部署两个keepalived实例**

​	Keepalived用于维护VIP地址的高可用，同样在192.168.0.200和192.168.0.201两台服务器上进行部署。主要需要配置keepalived监控HAProxy的运行状态，当某个HAProxy实例不可用时，自动将VIP地址切换到另一台主机上。下面对keepalived的配置和启动进行说明。

​	在第1台服务器192.168.0.200上创建配置文件keepalived.conf，内容如下：

```sh
# keepalived.conf - master 1
! Configuration File for keepalived

global_defs {
   router_id LVS_1
}

vrrp_script checkhaproxy
{
    script "/usr/bin/check-haproxy.sh"
    interval 2
    weight -30
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1

    virtual_ipaddress {
        192.168.0.100/24 dev ens33
    }

    authentication {
        auth_type PASS
        auth_pass password
    }

    track_script {
        checkhaproxy
    }
}

```

主要参数在vrrp_instance段中进行设置，说明如下。

* vrrp_instance VI_1：设置keepalived虚拟路由器VRRP的名称。
* state：设置为“MASTER”，将其他keepalived均设置
  为“BACKUP”。
* interface：待设置VIP地址的网卡名称。
* virtual_router_id：例如51。
* priority：优先级，例如100。
* virtual_ipaddress：VIP地址，例如192.168.0.100/24。
* authentication：访问keepalived服务的鉴权信息。
* track_script：HAProxy健康检查脚本。

Keepalived需要持续监控HAProxy的运行状态，在某个HAProxy实例运行不正常时，自动切换到运行正常的HAProxy实例上。需要创建一个HAProxy健康检查脚本，定期运行该脚本进行监控，例如新建脚本check-haproxy.sh并将其保存到/usr/bin目录下，内容示例如下：

```sh
# check-haproxy.sh
#!/bin/bash

count=`netstat -apn | grep 9443 | wc -l`

if [ $count -gt 0 ]; then
    exit 0
else
    exit 1
fi
```

​	若检查成功，则应返回0；若检查失败，则返回非0值。Keepalived根据上面的配置，会每隔2s检查一次HAProxy的运行状态。例如，如果在192.168.0.200 上检查失败，keepalived就会将VIP地址切换到正常运行HAProxy的192.168.0.201服务器上，保证VIP 192.168.0.100地址的高可用。

​	在第2台服务器192.168.0.201上创建配置文件keepalived.conf，内容示例如下：

```
# keepalived.conf - master 2
! Configuration File for keepalived

global_defs {
   router_id LVS_2
}

vrrp_script checkhaproxy
{
    script "/usr/bin/check-haproxy.sh"
    interval 2
    weight -30
}

vrrp_instance VI_1 {

    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1

    virtual_ipaddress {
        192.168.0.100/24 dev ens33
    }

    authentication {
        auth_type PASS
        auth_pass password
    }

    track_script {
        checkhaproxy
    }
}

```

这里与第1个keepalived配置的主要差异如下。

* vrrp_instance中的state被设置为“BACKUP”，这是因为在整个
  keepalived集群中只能有一个被设置为“MASTER”。如果keepalived集群
  不止2个实例，那么除了MASTER，其他都应被设置为“BACKUP”。
* vrrp_instance的值“VI_1”需要与MASTER的配置相同，表示它
  们属于同一个虚拟路由器组（VRRP），当MASTER不可用时，同组的
  其他BACKUP实例会自动选举出一个新的MASTER。
* HAProxy健康检查脚本check-haproxy.sh与第1个keepalived的相同。

​	下面以Docker容器方式运行HAProxy且镜像使用osixia/keepalived为例进行说明。在两台服务器192.168.18.3和192.168.18.4上启动HAProxy，将配置文件keepalived.conf挂载到容器的/container/service/keepalived/assets目录下，启动命令如下：

```sh
docker run -d --name k8s-keepalived \
  --restart=always \
  --net=host \
  --cap-add=NET_ADMIN --cap-add=NET_BROADCAST --cap-add=NET_RAW \
  -v ${PWD}/keepalived.conf:/container/service/keepalived/assets/keepalived.conf \
  -v ${PWD}/check-haproxy.sh:/usr/bin/check-haproxy.sh \
  osixia/keepalived:2.0.20 --copy-service
```

​	在运行正常的情况下，keepalived会在服务器192.168.0.200的网卡ens33上设置192.168.0.100的IP地址，同样在服务器192.168.0.201上运行的HAProxy将在该IP地址上监听9443端口号，对需要访问Kubernetes Master的客户端提供负载均衡器的入口地址，即192.168.0.100:9443。

​	通过ip addr命令查看服务器192.168.0.200的IP地址信息，可以看到在ens33网卡上新增了192.168.0.100地址：

![](https://pic.imgdb.cn/item/62a3fff30947543129327f0c.jpg)

​	使用curl命令即可验证通过HAProxy的192.168.0.200:9443地址是否可以访问到kube-apiserver服务：

![](https://pic.imgdb.cn/item/62a40013094754312932a5d7.jpg)

可以看到TCP/IP连接创建成功，得到响应码为401的应答，说明通过VIP地址192.168.0.100成功访问到了后端的kube-apiserver服务。至此，Master上所需的3个服务就全部启动完成了。接下来就可以部署
Node的服务了。

### 2.3.5 部署Node的服务

​	在Node上需要部署Docker、kubelet、kube-proxy，在成功加入Kubernetes集群后，还需要部署CNI网络插件、DNS插件等管理组件。Docker的安装和启动详见Docker官网的说明文档。本节主要对如何部署
kubelet和kube-proxy进行说明。CNI网络插件的安装部署详见7.7节的说明，DNS插件的安装部署详见4.3节的说明。

​	本节以将192.168.0.200、192.168.0.201和192.168.0.203三台主机部署为Node为例进行说明，由于这三台主机都是Master节点，所以最终部署结果为一个包含三个Node节点的Kubernetes集群。

**1.部署kubelet服务**
（1）为kubelet服务创建systemd服务配置文件/usr/lib/systemd/system/kubelet.service，内容如下：

```sh
# /usr/lib/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/kubernetes/kubernetes
After=docker.target

[Service]
EnvironmentFile=/etc/kubernetes/kubelet
ExecStart=/usr/bin/kubelet $KUBELET_ARGS
Restart=always

[Install]
WantedBy=multi-user.target
```

（2）配置文件/etc/kubernetes/kubelet的内容为通过环境变量KUBELET_ARGS设置的kubelet的全部启动参数，示例如下：

```sh
# /etc/kubernetes/kubelet
KUBELET_ARGS="--kubeconfig=/etc/kubernetes/kubeconfig --config=/etc/kubernetes/kubelet.config \
--hostname-override=192.168.0.200 \
--network-plugin=cni \
--logtostderr=false --log-dir=/var/log/kubernetes --v=0"
```

对主要参数说明如下。

* --kubeconfig：设置与API Server连接的相关配置，可以与kubecontroller-
  manager使用的kubeconfig文件相同。需要将相关客户端证书文
  件从Master主机复制到Node主机的/etc/kubernetes/pki目录下，例如
  ca.crt、client.key、client.crt文件。
* --config：kubelet配置文件，从Kubernetes 1.10版本开始引入，
  设置可以让多个Node共享的配置参数，例如address、port、
  cgroupDriver、clusterDNS、clusterDomain等。关于kubelet.config文件中
  可以设置的参数内容和详细说明，请参见官方文档的说明。
* --hostname-override：设置本Node在集群中的名称，默认值为
  主机名，应将各Node设置为本机IP或域名。
* --network-plugin：网络插件类型，建议使用CNI网络插件。

配置文件kubelet.config的内容示例如下：

```sh
# /etc/kubernetes/kubelet.config
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
port: 10250
cgroupDriver: cgroupfs
clusterDNS: ["169.169.0.100"]
clusterDomain: cluster.local
authentication:
  anonymous:
    enabled: true
```

在本例中设置的kubelet参数如下。

* address：服务监听IP地址。
* port：服务监听端口号，默认值为10250。
* cgroupDriver：设置为cgroupDriver驱动，默认值为cgroupfs，
  可选项包括systemd。
* clusterDNS：集群DNS服务的IP地址，例如169.169.0.100。
* clusterDomain：服务DNS域名后缀，例如cluster.local。
* authentication：设置是否允许匿名访问或者是否使用webhook
  进行鉴权。

（3）在配置文件准备完毕后，在各Node主机上启动kubelet服务并
设置为开机自启动：

```sh
systemctl start kubelet && system enable kubelet
```

**2.部署kube-proxy服务**

（1）为kube-proxy服务创建systemd服务配置文件/usr/lib/systemd/system/kube-proxy.service，内容如下：

```
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
EnvironmentFile=/etc/kubernetes/proxy
ExecStart=/usr/bin/kube-proxy $KUBE_PROXY_ARGS
Restart=always

[Install]
WantedBy=multi-user.target
```

（2）配置文件/etc/kubernetes/proxy的内容为通过环境变量KUBE_PROXY_ARGS设置的kube-proxy的全部启动参数，示例如下：

```sh
# /etc/kubernetes/proxy
KUBE_PROXY_ARGS="--kubeconfig=/etc/kubernetes/kubeconfig \
--hostname-override=192.168.18.3 \
--proxy-mode=iptables \
--logtostderr=false --log-dir=/var/log/kubernetes --v=0"
```

对主要参数说明如下。

* --kubeconfig：设置与API Server连接的相关配置，可以与kubelet使用的kubeconfig文件相同。相关客户端CA证书使用部署kubelet服务时从Master主机复制到Node主机的/etc/kubernetes/pki目录下的文件，包括ca.crt、client.key和client.crt。
* --hostname-override：设置本Node在集群中的名称，默认值为
  主机名，各Node应被设置为本机IP或域名。
* --proxy-mode：代理模式，包括iptables、ipvs、kernelspace（Windows节点使用）等。

（3）在配置文件准备完毕后，在各Node主机上启动kube-proxy服务，并设置为开机自启动：

```sh
systemctl start kube-proxy && systemctl enable kube-proxy
```

**3.在Master上通过kubectl验证Node信息**

​	在各个Node的kubelet和kube-proxy服务正常启动之后，会将本Node自动注册到Master上，然后就可以到Master主机上通过kubectl查询自动注册到Kubernetes集群的Node的信息了。

​	由于Master开启了HTTPS认证，所以kubectl也需要使用客户端CA证书连接Master，可以直接使用kube-controller-manager的kubeconfig文件，命令如下：

![](https://pic.imgdb.cn/item/62a4018c09475431293441ef.jpg)

​	我们可以看到各Node的状态为“NotReady”，这是因为还没有部署CNI网络插件，无法设置容器网络。

​	类似于通过kubeadm创建Kubernetes集群，例如选择Calico CNI插件运行下面的命令一键完成CNI网络插件的部署：

```sh
kubectl apply -f "https://docs.projectcalico.org/manifests/calico.yaml"
```

​	在CNI网络插件成功运行之后，Node的状态会更新为“Ready”：

​	为了使Kubernetes集群正常工作，我们还需要部署DNS服务，建议使用CoreDNS进行部署，请参见4.3节的说明。

​	至此，一个有三个Master节点的高可用Kubernetes集群就部署完成了，接下来用户就可以创建Pod、Deployment、Service等资源对象来部署、管理容器应用和微服务了。

​	本节对Kubernetes各服务启动进程的关键配置参数进行了简要说明，实际上Kubernetes的每个服务都提供了许多可配置的参数。这些参数涉及安全性、性能优化及功能扩展等方方面面。全面理解和掌握这些参数的含义和配置，对Kubernetes的生产部署及日常运维都有很大帮助。对各服务配置参数的详细说明参见附录A。

### 2.3.6 kube-apiserver基于token的认证机制 

​	Kubernetes除了提供了基于CA证书的认证方式，也提供了基于HTTP Token的简单认证方式。各客户端组件与API Server之间的通信方式仍然采用HTTPS，但不采用CA数字证书。这种认证机制与CA证书相比，安全性很低，在生产环境不建议使用。

​	采用基于HTTP Token的简单认证方式时，API Server对外暴露HTTPS端口，客户端携带Token来完成认证过程。需要说明的是，kubectl命令行工具比较特殊，它同时支持CA证书和简单认证两种方式
与API Server通信，其他客户端组件只能配置基于CA证书的认证方式或者非安全方式与API Server通信。

基于Token认证的配置过程如下。

（1）创建包括用户名、密码和UID的文件token_auth_file，将其放置在合适的目录下，例如/etc/kuberntes目录。需要注意的是，这是一个纯文本文件，用户名、密码都是明文。

![](https://pic.imgdb.cn/item/62a402c40947543129359a02.jpg)

（3）用curl客户端工具通过token访问API Server：

![](https://pic.imgdb.cn/item/62a402db094754312935b42b.jpg)

## 2.4 使用私有镜像库的相关配置

​	在Kubernetes集群中，容器应用都是基于镜像启动的，在私有云环境中建议搭建私有镜像库对镜像进行统一管理，在公有云环境中可以直接使用云服务商提供的镜像库。

私有镜像库有两种选择。

（1）Docker提供的Registry镜像库，详细说明请参考官网的说明

（2）Harbor镜像仓库，详细说明请参考官网的说明或者Harbor项目维护者及贡献者编写的《Harbor权威指南》一书。

此外，Kubernetes对于创建Pod需要使用一个名为“pause”的镜像，

​	tag名为“k8s.gcr.io/pause：3.2”，默认从镜像库k8s.gcr.io下载，在私有云环境中可以将其上传到私有镜像库，并修改kubelet的启动参数--podinfra-container-image，将其设置为使用镜像库的镜像名称，例如：

![](https://pic.imgdb.cn/item/62a4032709475431293608da.jpg)

## 2.5　Kubernetes的版本升级

### 2.5.1　二进制文件升级

​	在进行Kubernetes的版本升级之前，需要考虑不中断正在运行的业务容器的灰度升级方案。常见的做法是：先更新Master上Kubernetes服务的版本，再逐个或批量更新集群中的Node上Kubernetes服务的版本。更新Node的Kubernetes服务的步骤通常包括：先隔离一个或多个Node的业务流量，等待这些Node上运行的Pod将当前任务全部执行完成后，停掉业务应用（Pod），再更新这些Node上的kubelet和kube-proxy版本，更新完成后重启业务应用（Pod），并将业务流量导入新启动的这些Node上，再隔离剩余的Node，逐步完成Node的版本升级，最终完成整个集群的Kubernetes版本升级。

​	同时，应该考虑高版本的Master对低版本的Node的兼容性问题。高版本的Master通常可以管理低版本的Node，但版本差异不应过大，以免某些功能或API版本被弃用后，低版本的Node无法运行。

* 通过官网获取最新版本的二进制包kubernetes.tar.gz，解压后提取服务的二进制文件。
* 更新Master的kube-apiserver、kube-controller-manager、kubescheduler服务的二进制文件和相关配置（在需要修改时更新）并重启服务。
* 逐个或批量隔离Node，等待其上运行的全部容器工作完成后停掉Pod，更新kubelet、kube-proxy服务文件和相关配置（在需要修改时更新），然后重启这两个服务。

### 2.5.2 使用kubeadm进行集群升级

kubeadm提供了upgrade命令用于对kubeadm安装的Kubernetes集群进行升级。这一功能提供了从1.10到1.11、从1.11到1.12、从1.12到1.13及从1.13到1.14升级的能力，本节以从1.13到1.14升级为例进行说明。

升级之前需要注意：

* 虽然kubeadm的升级不会触及工作负载，但还是要在升级之前
  做好备份；
* 升级过程中可能会因为Pod的变化而造成容器重启。继续以CentOS 7环境为例，首先需要升级的是kubeadm：

```sh
yum install -y kubeadm-1.14.0 --disableexcludes=kubernetes
```

查看kubeadm的版本：

![](https://pic.imgdb.cn/item/62a403bd094754312936af93.jpg)

接下来查看kubeadm的升级计划：

```sh
kubeadm upgrade plan
```

![](https://pic.imgdb.cn/item/62a403d8094754312936d259.jpg)

按照任务指引进行升级

![](https://pic.imgdb.cn/item/62a403e5094754312936e5d5.jpg)

输入“y”，确认后开始升级。
运行完成之后，再次查询版本：

![](https://pic.imgdb.cn/item/62a403f5094754312936f749.jpg)

然后可以对节点配置进行升级：

```sh
kubeadm upgrade node config --kubelet-version 1.14.0
```

接下来，直接下载新版本的kubectl二进制文件，用其覆盖旧版本的文件来完成kubectl的升级，这样就完成了集群的整体升级：

![](https://pic.imgdb.cn/item/62a404140947543129371950.jpg)

## 2.6　CRI（容器运行时接口）详解

​	归根结底，Kubernetes Node（kubelet）的主要功能就是启动和停止容器的组件，我们称之为容器运行时（Container Runtime），其中最知名的就是Docker了。为了更具扩展性，Kubernetes从1.5版本开始就加入了容器运行时插件API，即Container Runtime Interface，简称CRI。

### 2.6.1　CRI概述

​	每个容器运行时都有特点，因此不少用户希望Kubernetes能够支持更多的容器运行时。Kubernetes从1.5版本开始引入了CRI接口规范，通过插件接口模式，Kubernetes无须重新编译就可以使用更多的容器运行时。CRI包含Protocol Buffers、gRPC API、运行库支持及开发中的标准规范和工具。Docker的CRI实现在Kubernetes 1.6中被更新为Beta版本，并在kubelet启动时默认启动。

​	可替代的容器运行时支持是Kubernetes中的新概念。在Kubernetes1.3发布时，rktnetes项目同时发布，让rkt容器引擎成为除Docker外的又一选择。然而，不管是Docker还是rkt，都用到了kubelet的内部接口，同kubelet源码纠缠不清。这种程度的集成需要对kubelet的内部机制有非常深入的了解，还会给社区带来管理压力，这就给新生代容器运行时造成了难以跨越的集成壁垒。CRI接口规范尝试用定义清晰的抽象层清除这一壁垒，让开发者能够专注于容器运行时本身。

### 2.6.2　CRI的主要组件

​	kubelet使用gRPC框架通过UNIX Socket与容器运行时（或CRI代理）进行通信。在这个过程中kubelet是客户端，CRI代理（shim）是服务端，如图2.7所示。

![](https://pic.imgdb.cn/item/62a404b2094754312937c854.jpg)

### 2.6.3 Pod和容器的生命周期管理

​	Pod由一组应用容器组成，其中包含共有的环境和资源约束。在CRI里，这个环境被称为PodSandbox。Kubernetes有意为容器运行时留下一些发挥空间，它们可以根据自己的内部实现来解释PodSandbox。对于Hypervisor类的运行时，PodSandbox会具体化为一个虚拟机。其他例如Docker，会是一个Linux命名空间。在v1alpha1 API中，kubelet会创建Pod级别的cgroup传递给容器运行时，并以此运行所有进程来满足PodSandbox对Pod的资源保障。

​	在启动Pod之前，kubelet调用RuntimeService.RunPodSandbox来创建环境。这一过程包括为Pod设置网络资源（分配IP等操作）。PodSandbox被激活之后，就可以独立地创建、启动、停止和删除不同的容器了。kubelet会在停止和删除PodSandbox之前首先停止和删除其中的容器。

​	kubelet的职责在于通过RPC管理容器的生命周期，实现容器生命周期的钩子、存活和健康监测，以及执行Pod的重启策略等。

​	RuntimeService服务包括对Sandbox和Container操作的方法，下面的伪代码展示了主要的RPC方法：

![](https://pic.imgdb.cn/item/62a405260947543129384abd.jpg)

![](https://pic.imgdb.cn/item/62a405330947543129385d33.jpg)

### 2.6.4　面向容器级别的设计思路

​	众所周知，Kubernetes的最小调度单元是Pod，它曾经可能采用的一个CRI设计就是复用Pod对象，使得容器运行时可以自行实现控制逻辑和状态转换，这样一来，就能极大地简化API，让CRI能够更广泛地适用于多种容器运行时。但是经过深入讨论之后，Kubernetes放弃了这一想法。

​	首先，kubelet有很多Pod级别的功能和机制（例如crash-loop backoff机制），如果交给容器运行时去实现，则会造成很重的负担；然后，Pod标准还在快速演进。很多新功能（如初始化容器）都是由kubelet完成管理的，无须交给容器运行时实现。

​	CRI选择了在容器级别进行实现，使得容器运行时能够共享这些通用特性，以获得更快的开发速度。这并不意味着设计哲学的改变—kubelet要负责、保证容器应用的实际状态和声明状态的一致性。

​	Kubernetes为用户提供了与Pod及其中的容器进行交互的功能（kubectl exec/attach/port-forward）。kubelet目前提供了两种方式来支持这些功能：①调用容器的本地方法；②使用Node上的工具（例如nsenter及socat）。

​	因为多数工具都假设Pod用Linux namespace做了隔离，因此使用Node上的工具并不是一种容易移植的方案。在CRI中显式定义了这些调用方法，让容器运行时进行具体实现。下面的伪代码显示了Exec、Attach、PortForward这几个调用需要实现的RuntimeService方法：

![](https://pic.imgdb.cn/item/62a40571094754312938ae53.jpg)

​	目前还有一个潜在的问题是，kubelet处理所有的请求连接，使其有成为Node通信瓶颈的可能。在设计CRI时，要让容器运行时能够跳过中间过程。容器运行时可以启动一个单独的流式服务来处理请求（还能对Pod的资源使用情况进行记录），并将服务地址返回给kubelet。这样kubelet就能反馈信息给API Server，使之可以直接连接到容器运行时提供的服务，并连接到客户端。

### 2.6.5 尝试使用新的Docker-CRI来创建容器

​	要尝试新的kubelet-CRI-Docker集成，只需为kubelet启动参数加上--enable-cri=true开关来启动CRI。这个选项从Kubernetes 1.6开始已经作为kubelet的默认选项了。如果不希望使用CRI，则可以设置--enablecri=false来关闭这个功能。

​	查看kubelet的日志，可以看到启用CRI和创建gRPC Server的日志：

![](https://pic.imgdb.cn/item/62a405a0094754312938dc89.jpg)

查看Pod的详细信息，可以看到将会创建沙箱（Sandbox）的Event：

![](https://pic.imgdb.cn/item/62a405b0094754312938ecd8.jpg)

### 2.6.6　CRI的进展

​	目前已经有多款开源CRI项目可用于Kubernetes：Docker、CRI-O、Containerd、frakti（基于Hypervisor的容器运行时），各CRI运行时的安装手册可参考官网的说明。

## 2.7 kubectl命令行工具用法详解 

​	kubectl作为客户端CLI工具，可以让用户通过命令行对Kubernetes集群进行操作。本节对kubectl的子命令和用法进行详细说明。

### 2.7.1　kubectl用法概述

kubectl命令行的语法如下：

![](https://pic.imgdb.cn/item/62a40678094754312939d88c.jpg)

其中，command、TYPE、NAME、flags的含义如下。
（1）command：子命令，用于操作资源对象，例如create、get、describe、delete等。
（2）TYPE：资源对象的类型，区分大小写，能以单数、复数或者简写形式表示。例如以下3种TYPE是等价的。

![](https://pic.imgdb.cn/item/62a4068c094754312939ebc9.jpg)

（3）NAME：资源对象的名称，区分大小写。如果不指定名称，系统则将返回属于TYPE的全部对象的列表，例如运行kubectl get pods命令后将返回所有Pod的列表。

在一个命令行中也可以同时对多个资源对象进行操作，以多个TYPE和NAME的组合表示，示例如下。

* 获取多个相同类型资源的信息，以TYPE1 name1 name2 name<#>格式表示：

![](https://pic.imgdb.cn/item/62a406ac09475431293a10fd.jpg)

* 获取多种不同类型对象的信息，以TYPE1/name1 TYPE1/name2
  TYPE2/name3 TYPE<#>/name<#>格式表示：

![](https://pic.imgdb.cn/item/62a406c509475431293a2ff4.jpg)

（4）flags：kubectl子命令的可选参数，例如使用-s或--server设置API Server的URL地址，而不使用默认值。

### 2.7.2 kubectl子命令详解 

​	kubectl的子命令非常丰富，涵盖了对Kubernetes集群的主要操作，包括资源对象的创建、删除、查看、修改、配置、运行等。详细的子命令如表2.5所示。

![](https://pic.imgdb.cn/item/62a406f809475431293a6902.jpg)

![](https://pic.imgdb.cn/item/62a4071409475431293a8841.jpg)

![](https://pic.imgdb.cn/item/62a4072209475431293a9792.jpg)

### 2.7.3 kubectl可操作的资源对象详解

​	kubectl可操作的资源对象列表如表2.6所示，可以通过kubectl apiresources命令进行查看。

![](https://pic.imgdb.cn/item/62a4077009475431293aef9d.jpg)

![](https://pic.imgdb.cn/item/62a4078109475431293b0615.jpg)

![](https://pic.imgdb.cn/item/62a407a209475431293b2bb4.jpg)

### 2.7.4　kubectl的公共参数说明

![](https://pic.imgdb.cn/item/62a407b209475431293b3c4e.jpg)

![](https://pic.imgdb.cn/item/62a407c009475431293b4ba4.jpg)

​	每个子命令（如create、delete、get等）还有其特定的命令行参数，可以通过$ kubectl [command]--help命令进行查看。

### 2.7.5　kubectl格式化输出

kubectl命令可以对结果进行多种格式化显示，输出的格式通过-o参数指定：

![](https://pic.imgdb.cn/item/62a407f109475431293b7ebe.jpg)

常用的输出格式示例如下。
（1）显示Pod的更多信息，例如Node IP等：

![](https://pic.imgdb.cn/item/62a4080909475431293b94eb.jpg)

![](https://pic.imgdb.cn/item/62a4285c0947543129615fee.jpg)

（5）关闭服务端列名。在默认情况下，Kubernetes服务端会将资源对象的某些特定信息显示为列，这可以通过设置--server-print=false参数进行关闭，例如：

![](https://pic.imgdb.cn/item/62a4288509475431296195f2.jpg)

### 2.7.6　kubectl常用操作示例

![](https://pic.imgdb.cn/item/62a4289d094754312961b1df.jpg)

![](https://pic.imgdb.cn/item/62a428ba094754312961d48e.jpg)

![](https://pic.imgdb.cn/item/62a428dd094754312961f9bd.jpg)

![](https://pic.imgdb.cn/item/62a428f30947543129621204.jpg)

![](https://pic.imgdb.cn/item/62a429170947543129623d12.jpg)

![](https://pic.imgdb.cn/item/62a438c8094754312976ba42.jpg)

![](https://pic.imgdb.cn/item/62a438d7094754312976d900.jpg)

![](https://pic.imgdb.cn/item/62a439020947543129770f0b.jpg)

![](https://pic.imgdb.cn/item/62a43910094754312977200c.jpg)

# 第3章　深入掌握Pod

## 3.1　Pod定义详解

YAML格式的Pod定义文件的完整内容如下：

![](https://pic.imgdb.cn/item/62a4398d094754312977b6fa.jpg)

![](https://pic.imgdb.cn/item/62a439aa094754312977d982.jpg)

![](https://pic.imgdb.cn/item/62a439cf09475431297809b5.jpg)

![](https://pic.imgdb.cn/item/62a439e3094754312978236f.jpg)

![](https://pic.imgdb.cn/item/62a43a050947543129785495.jpg)

![](https://pic.imgdb.cn/item/62a43a3f094754312978a19a.jpg)

![](https://pic.imgdb.cn/item/62a43a51094754312978bbac.jpg)

## 3.2　Pod的基本用法

​	在使用Docker时，可以使用docker run命令创建并启动一个容器。而在Kubernetes系统中对长时间运行容器的要求是：其主程序需要一直在前台运行。如果我们创建的Docker镜像的启动命令是后台执行程序，例如Linux脚本：

```sh
nohup ./start.sh &
```

​	则在kubelet创建包含这个容器的Pod之后运行完该命令，即认为Pod执行结束，将立刻销毁该Pod。如果为该Pod定义了ReplicationController，则系统会监控到该Pod已经终止，之后根据RC定
义中Pod的replicas副本数量生成一个新的Pod。一旦创建新的Pod，就在运行完启动命令后陷入无限循环的过程中。这就是Kubernetes需要我们自己创建Docker镜像并以一个前台命令作为启动命令的原因。

​	对于无法改造为前台执行的应用，也可以使用开源工具Supervisor辅助进行前台运行的功能。Supervisor提供了一种可以同时启动多个后台应用，并保持Supervisor自身在前台执行的机制，可以满足Kubernetes对容器的启动要求。关于Supervisor的安装和使用，请参考官网的文档说明。

​	Pod可以由1个或多个容器组合而成。在上一节Guestbook的例子中，名为frontend的Pod只由一个容器组成：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  containers:
  - name: frontend
    image: kubeguide/guestbook-php-frontend
    env:
    - name: GET_HOSTS_FROM
      value: env
    ports:
    - containerPort: 80
```

​	这个frontend Pod在成功启动之后，将启动1个Docker容器。

​	另一种场景是，当frontend和redis两个容器应用为紧耦合的关系，并组合成一个整体对外提供服务时，应将这两个容器打包为一个Pod，如图3.1所示

![](https://pic.imgdb.cn/item/62a443f4094754312985d4a7.jpg)
