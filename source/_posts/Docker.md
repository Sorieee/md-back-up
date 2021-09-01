《Docker技术入门与实战》第3版

# 核心概念与安装配置

## 核心概念

### Docker镜像

​	Docker镜像类似于虚拟机镜像，可以将它理解为一个只读的模板。

​	例如，一个镜像可以包含一个基本的操作系统环境，里面仅安装了Apache应用程序（或用户需要的其他软件）。可以把它称为一个Apache镜像。镜像是创建Docker容器的基础。

​	通过版本管理和增量的文件系统，Docker提供了一套十分简单的机制来创建和更新现有的镜像，用户甚至可以从网上下载一个已经做好的应用镜像，并直接使用。

### Docker容器

​	Docker容器类似于一个轻量级的沙箱，Docker利用容器来运行和隔离应用。

​	容器是从镜像创建的应用运行实例。它可以启动、开始、停止、删除，而这些容器都是彼此相互隔离、互不可见的。

​	可以把容器看作一个简易版的Linux系统环境（包括root用户权限、进程空间、用户空间和网络空间等）以及运行在其中的应用程序打包而成的盒子。

> 镜像自身是只读的。容器从镜像启动的时候，会在镜像的最上层创建一个可写层。

### Docker仓库

​	Docker仓库类似于代码仓库，是Docker集中存放镜像文件的场所。

​	有时候我们会将Docker仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器是存放仓库的地方，其上往往存放着多个仓库。每个仓库集中存放某一类镜像，往往包括多个镜像文件，通过不同的标签（tag）来进行区分。例如存放Ubuntu操作系统镜像的仓库，被称为Ubuntu仓库，其中可能包括16.04、18.04等不同版本的镜像。仓库注册服务器的示例如图2-1所示。

![](https://pic.imgdb.cn/item/612ca6c144eaada739055338.jpg)



​		根据所存储的镜像公开分享与否，Docker仓库可以分为公开仓库（Public）和私有仓库（Private）两种形式。

​	目前，最大的公开仓库是官方提供的Docker Hub，其中存放着数量庞大的镜像供用户下载。国内不少云服务提供商（如腾讯云、阿里云等）也提供了仓库的本地源，可以提供稳定的国内访问。

​	当然，用户如果不希望公开分享自己的镜像文件，Docker也支持用户在本地网络内创建一个只能自己访问的私有仓库。

> 可以看出，Docker利用仓库管理镜像的设计理念与Git代码仓库的概念非常相似，实际上Docker设计上借鉴了Git的很多优秀思想。

## 安装Docker引擎

​	Docker引擎是使用Docker容器的核心组件，可以在主流的操作系统和云平台上使用，包括Linux操作系统（如Ubuntu、Debian、CentOS、Redhat等）, macOS和Windows操作系统，以及IBM、亚马逊、微软等知名云平台。

​	用户可以访问Docker官网的Get Docker（https://www.docker.com/get-docker）页面，查看获取Docker的方式，以及Docker支持的平台类型，如图2-2所示。

![](https://pic.imgdb.cn/item/612ca72744eaada73905fa6e.jpg)

​	目前Docker支持Docker引擎、Docker Hub、Docker Cloud等多种服务。

* Docker引擎：包括支持在桌面系统或云平台安装Docker，以及为企业提供简单安全弹性的容器集群编排和管理；
* DockerHub：官方提供的云托管服务，可以提供公有或私有的镜像仓库；
* DockerCloud：官方提供的容器云服务，可以完成容器的部署与管理，可以完整地支持容器化项目，还有CI、CD功能。



​	Docker引擎目前分为两个版本：社区版本（Community Edition, CE）和企业版本（Enterprise Edition, EE）。社区版本包括大部分的核心功能，企业版本则通过付费形式提供认证支持、镜像管理、容器托管、安全扫描等高级服务。通常情况下，用户使用社区版本可以满足大部分需求；若有更苛刻的需求，可以购买企业版本服务。社区版本每个月会发布一次尝鲜（Edge）版本，每个季度（3、6、9、12月）会发行一次稳定（Stable）版本。版本号命名格式为“年份．月份”，如2018年6月发布的版本号为v18.06。

### CentOS环境下安装Docker

​	Docker目前支持CentOS 7及以后的版本。系统的要求跟Ubuntu情况类似，64位操作系统，内核版本至少为3.10。

​	首先，为了方便添加软件源，以及支持devicemapper存储类型，安装如下软件包：

```sh
sudo yum update
sudo yum install -y yum-utils \
	device-mapper-persistent-data \
	lvm2
```

添加Docker稳定版本的yum软件源：

```sh
sudo yum-config-manager \
	--add-repo \
	https://download.docker.com/linux/centos/docker-ce.repo 
```

之后更新yum软件源缓存，并安装Docker：

```sh
sudo yum update
sudo yum install -y docker-ce
```

最后确认Docker服务启动正常

```sh
sudo systemctl start docker
sudo systemctl status docker
```

## 配置Docker服务

​	为了避免每次使用Docker命令时都需要切换到特权身份，可以将当前用户加入安装中自动创建的docker用户组，代码如下：

```
sudo usermod -aG docker USER_NAME
```

​	用户更新组信息，退出并重新登录后即可生效。

​	Docker服务启动时实际上是调用了dockerd命令，支持多种启动参数。因此，用户可以直接通过执行dockerd命令来启动Docker服务，如下面的命令启动Docker服务，开启Debug模式，并监听在本地的2376端口：

```
dockerd -D -H tcp://127.0.0.1:2376
```

​	这些选项可以写入/etc/docker/路径下的daemon.json文件中，由dockerd服务启动时读取：

```json
{
	"debug" : true,
	"hosts' : ["tcp://127.0.0.1:2376" ]
}    
```

​	对于CentOS、RedHat等系统，服务通过systemd来管理，配置文件路径为/etc/systemd/system/docker.service.d/docker.conf。更新配置后需要通过systemctl命令来管理Docker服务：

```
sudo systemctl daemon-reload
sudo systemctl start docker.service 
```

​	此外，如果服务工作不正常，可以通过查看Docker服务的日志信息来确定问题，例如在RedHat系统上日志文件可能为/var/log/messages，在Ubuntu或CentOS系统上可以执行命令journalctl -u docker.service。

​	每次重启Docker服务后，可以通过查看Docker信息（docker info命令），确保服务已经正常运行。

# 使用Docker镜像

