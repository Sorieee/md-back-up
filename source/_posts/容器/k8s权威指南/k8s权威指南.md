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
