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

# 3. 使用Docker镜像

# 3.1 获取镜像

​	可以使用docker [image] pull命令直接从Docker Hub镜像源来下载镜像。该命令的格式为docker [image] pull NAME[:TAG]。其中，NAME是镜像仓库名称（用来区分镜像）, TAG是镜像的标签（往往用来表示版本信息）。通常情况下，描述一个镜像需要包括“名称+标签”信息。

```sh
$ docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
```

​	下载过程中可以看出，镜像文件一般由若干层（layer）组成，6c953ac5d795这样的串是层的唯一id（实际上完整的id包括256比特，64个十六进制字符组成）。使用docker pull命令下载中会获取并输出镜像的各层信息。当不同的镜像包括相同的层时，本地仅存储了层的一份内容，减小了存储空间。

​	严格地讲，镜像的仓库名称中还应该添加仓库地址（即registry，注册服务器）作为前缀，只是默认使用的是官方Docker Hub服务，该前缀可以忽略。例如，docker pull ubuntu:18.04命令相当于docker pullregistry.hub.docker.com/ubuntu:18.04命令，即从默认的注册服务器DockerHub Registry中的ubuntu仓库来下载标记为18.04的镜像。

pull子命令支持的选项主要包括：

* -a, --all-tags=true|false：是否获取仓库中的所有镜像，默认为否；

* --disable-content-trust：取消镜像的内容校验，默认为真。



​	下载镜像到本地后，即可随时使用该镜像了，例如利用该镜像创建一个容器，在其中运行bash应用，执行打印“Hello World”命令：

```
docker run -it ubuntu: 18.04 bash
```

## 3.2 查看镜像信息

**1．使用images命令列出镜像**

```
docker images
```

在列出信息中，可以看到几个字段信息：

* 来自于哪个仓库，比如ubuntu表示ubuntu系列的基础镜像；
* 镜像的标签信息，比如18.04、latest表示不同的版本信息。标签只是标记，并不能标识镜像内容；
* 镜像的ID（唯一标识镜像），如果两个镜像的ID相同，说明它们实际上指向了同一个镜像，只是具有不同标签名称而已；
* 创建时间，说明镜像最后的更新时间；
* 镜像大小，优秀的镜像往往体积都较小。



images子命令主要支持如下选项，用户可以自行进行尝试：

* -a, --all=true|false：列出所有（包括临时文件）镜像文件，默认为否；
* --digests=true|false：列出镜像的数字摘要值，默认为否；
* -f, --filter=[]：过滤列出的镜像，如dangling=true只显示没有被使用的镜像；也可指定带有特定标注的镜像等；
* --format="TEMPLATE"：控制输出格式，如．ID代表ID信息，.Repository代表仓库信息等；
* --no-trunc=true|false：对输出结果中太长的部分是否进行截断，如镜像的ID信息，默认为是；
* -q, --quiet=true|false：仅输出ID信息，默认为否。

**2．使用tag命令添加镜像标签**

​	为了方便在后续工作中使用特定镜像，还可以使用docker tag命令来为本地镜像任意添加新的标签。例如，添加一个新的myubuntu:latest镜像标签：

```
docker tag ubuntu: latest myubuntu: latest
```

**3．使用inspect命令查看详细信息**

​	使用docker[image]inspect命令可以获取该镜像的详细信息，包括制作者、适应架构、各层的数字摘要等：

```
docker [images] inspect mysql:latest
[
    {
        "Id": "sha256:0716d6ebcc1a61c5a296fcb187e71f93531e510d4e4400267e2e502103d0194c",
        "RepoTags": [
            "mysql:latest"
        ],
        "RepoDigests": [
            "mysql@sha256:99e0989e7e3797cfbdb8d51a19d32c8d286dd8862794d01a547651a896bcf00c"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2021-09-03T07:24:57.3195538Z",
...
]
```

​	上面代码返回的是一个JSON格式的消息，如果我们只要其中一项内容时，可以使用-f来指定，例如，获取镜像的Architecture：

```
docker [images] inspect -f {{".Architecture"}} mysql:latest
```

**4．使用history命令查看镜像历史**

​	既然镜像文件由多个层组成，那么怎么知道各个层的内容具体是什么呢？这时候可以使用history子命令，该命令将列出各层的创建信息。

```sh
docker history mysql:latest
IMAGE          CREATED      CREATED BY                                      SIZE      COMMENT
0716d6ebcc1a   3 days ago   /bin/sh -c #(nop)  CMD ["mysqld"]               0B
<missing>      3 days ago   /bin/sh -c #(nop)  EXPOSE 3306 33060            0B
<missing>      3 days ago   /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B
<missing>      3 days ago   /bin/sh -c ln -s usr/local/bin/docker-entryp…   34B
<missing>      3 days ago   /bin/sh -c #(nop) COPY file:345a22fe55d3e678…   14.5kB
<missing>      3 days ago   /bin/sh -c #(nop) COPY dir:2e040acc386ebd23b…   1.12kB
<missing>      3 days ago   /bin/sh -c #(nop)  VOLUME [/var/lib/mysql]      0B
<missing>      3 days ago   /bin/sh -c {   echo mysql-community-server m…   378MB
<missing>      3 days ago   /bin/sh -c echo 'deb http://repo.mysql.com/a…   55B
<missing>      3 days ago   /bin/sh -c #(nop)  ENV MYSQL_VERSION=8.0.26-…   0B
<missing>      3 days ago   /bin/sh -c #(nop)  ENV MYSQL_MAJOR=8.0          0B
<missing>      3 days ago   /bin/sh -c set -ex;  key='A4A9406876FCBD3C45…   1.84kB
<missing>      3 days ago   /bin/sh -c apt-get update && apt-get install…   52.2MB
<missing>      3 days ago   /bin/sh -c mkdir /docker-entrypoint-initdb.d    0B
<missing>      3 days ago   /bin/sh -c set -eux;  savedAptMark="$(apt-ma…   4.17MB
<missing>      3 days ago   /bin/sh -c #(nop)  ENV GOSU_VERSION=1.12        0B
<missing>      3 days ago   /bin/sh -c apt-get update && apt-get install…   9.34MB
<missing>      3 days ago   /bin/sh -c groupadd -r mysql && useradd -r -…   329kB
<missing>      3 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      3 days ago   /bin/sh -c #(nop) ADD file:4ff85d9f6aa246746…   69.3MB

```

​	注意，过长的命令被自动截断了，可以使用前面提到的--no-trunc选项来输出完整命令。

## 3.3 搜寻镜像

​	本节主要介绍Docker镜像的search子命令。使用docker search命令可以搜索Docker Hub官方仓库中的镜像。语法为docker search [option] keyword。支持的命令选项主要包括：

* -f, --filter filter：过滤输出内容；
* --format string：格式化输出内容；
* --limit int：限制输出结果个数，默认为25个；
* --no-trunc：不截断输出结果。

例如，搜索官方提供的带nginx关键字的镜像，如下所示：

```
$ docker search --filter=is-official=true nginx
```

再比如，搜索所有收藏数超过4的关键词包括tensorflow的镜像：

```
docker search --filter=stars=4 nginx
```

## 3.4 删除和清理镜像

**1．使用标签删除镜像**

​	使用docker rmi或docker image rm命令可以删除镜像，命令格式为docker rmiIMAGE [IMAGE...]，其中IMAGE可以为标签或ID。

​	支持选项包括：

* -f, -force：强制删除镜像，即使有容器依赖它；
* -no-prune：不要清理未带标签的父镜像。



​	例如，要删除掉myubuntu:latest镜像，可以使用如下命令：

```
docker rmi myubuntu:last
```

​	读者可能会想到，本地的ubuntu:latest镜像是否会受到此命令的影响。无须担心，当同一个镜像拥有多个标签的时候，docker rmi命令只是删除了该镜像多个标签中的指定标签而已，并不影响镜像文件。因此上述操作相当于只是删除了镜像0458a4468cbc的一个标签副本而已。

​	但当镜像只剩下一个标签的时候就要小心了，此时再使用docker rmi命令会彻底删除镜像。

![](https://pic.imgdb.cn/item/6136af9d44eaada739d76393.jpg)

**2．使用镜像ID来删除镜像**

​	当使用docker rmi命令，并且后面跟上镜像的ID（也可以是能进行区分的部分ID串前缀）时，会先尝试删除所有指向该镜像的标签，然后删除该镜像文件本身。

​	注意，当有该镜像创建的容器存在时，镜像文件默认是无法被删除的，例如：先利用ubuntu:18.04镜像创建一个简单的容器来输出一段话：

![](https://pic.imgdb.cn/item/6136b03d44eaada739d802bc.jpg)

​	试图删除该镜像，Docker会提示有容器正在运行，无法删除：

![](https://pic.imgdb.cn/item/6136b07044eaada739d8345a.jpg)

​	注意，通常并不推荐使用-f参数来强制删除一个存在容器依赖的镜像。正确的做法是，先删除依赖该镜像的所有容器，再来删除镜像。

**3．清理镜像**

​	使用Docker一段时间后，系统中可能会遗留一些临时的镜像文件，以及一些没有被使用的镜像，可以通过docker image prune命令来进行清理。

​	支持选项包括：

* -a, -all：删除所有无用镜像，不光是临时镜像；
* -filter filter：只清理符合给定过滤器的镜像；
* -f, -force：强制删除镜像，而不进行提示确认。



​	例如，如下命令会自动清理临时的遗留镜像文件层，最后会提示释放的存储空间：

![](https://pic.imgdb.cn/item/6136b0cf44eaada739d894b1.jpg)

## 3.5 创建镜像

​	本节主要介绍Docker的commit、import和build子命令。

**1．基于已有容器创建**

​	该方法主要是使用docker [container] commit命令。

​	命令格式为docker [container] commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]，主要选项包括：

*  -a, --author=""：作者信息；
* -c, --change=[]：提交的时候执行Dockerfile指令，包括CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR等；
* -m, --message=""：提交消息；
* -p, --pause=true：提交时暂停容器运行。



​	首先，启动一个镜像，并在其中进行修改操作。例如，创建一个test文件，之后退出，代码如下：

![](https://pic.imgdb.cn/item/61372e1444eaada739af5b6d.jpg)

​	记住容器的ID为a925cb40b3f0。

​	此时该容器与原ubuntu:18.04镜像相比，已经发生了改变，可以使用docker[container] commit命令来提交为一个新的镜像。提交时可以使用ID或名称来指定容器：

![](https://pic.imgdb.cn/item/61372e2f44eaada739af89fd.jpg)

​	顺利的话，会返回新创建镜像的ID信息，例如9e9c814023bcffc3e67e892a235afe61b02f66a947d2747f724bd317dda02f27。

​	此时查看本地镜像列表，会发现新创建的镜像已经存在了：

![](https://pic.imgdb.cn/item/61372e4644eaada739afb2b5.jpg)

​	顺利的话，会返回新创建镜像的ID信息，例如9e9c814023bcffc3e67e892a235afe61b02f66a947d2747f724bd317dda02f27。

​	此时查看本地镜像列表，会发现新创建的镜像已经存在了：

![](https://pic.imgdb.cn/item/61372ee944eaada739b0ccd9.jpg)

**2．基于本地模板导入**

​	用户也可以直接从一个操作系统模板文件导入一个镜像，主要使用docker[container] import命令。命令格式为docker [image] import [OPTIONS]file|URL|-[REPOSITORY [:TAG]]

​	要直接导入一个镜像，可以使用OpenVZ提供的模板来创建，或者用其他已导出的镜像模板来创建。OPENVZ模板的下载地址为http://openvz.org/Download/templates/precreated。

​	例如，下载了ubuntu-18.04的模板压缩包，之后使用以下命令导入即可：

![](https://pic.imgdb.cn/item/61372f1d44eaada739b12e01.jpg)

**3．基于Dockerfile创建**

​	基于Dockerfile创建是最常见的方式。Dockerfile是一个文本文件，利用给定的指令描述基于某个父镜像创建新镜像的过程。

​	下面给出Dockerfile的一个简单示例，基于debian:stretch-slim镜像安装Python3环境，构成一个新的python:3镜像：

![](https://pic.imgdb.cn/item/61372ff944eaada739b2b66f.jpg)

​	创建镜像的过程可以使用docker [image] build命令，编译成功后本地将多出一个python:3镜像：

![](https://pic.imgdb.cn/item/6137309144eaada739b3b042.jpg)

## 3.6 存出和载入镜像

​	本节主要介绍Docker镜像的save和load子命令。用户可以使用docker [image]save和docker [image] load命令来存出和载入镜像。

**1．存出镜像**

​	如果要导出镜像到本地文件，可以使用docker [image] save命令。该命令支持-o、-output string参数，导出镜像到指定的文件中。

![](https://pic.imgdb.cn/item/613731f144eaada739b5e41b.jpg)

**2．载入镜像**

​	可以使用docker [image] load将导出的tar文件再导入到本地镜像库。支持-i、-input string选项，从指定文件中读入镜像内容。例如，从文件ubuntu_18.04.tar导入镜像到本地镜像列表，如下所示：

![](https://pic.imgdb.cn/item/6137338344eaada739b8704a.jpg)

**docker load和docker import区别**

​	实际上，既可以使用docker load命令来导入镜像库存储文件到本地镜像库，也可以使用docker import命令来导入一个容器快照到本地镜像库。

​	两者的区别在于容器快照将会丢弃所有的历史记录和元数据信息，而镜像存储文件将保存完整记录，体积也会更大。此外从容器快照文件导入时，也可以重新指定标签等元数据

## 3.7 上传镜像

​	本节主要介绍Docker镜像的push子命令。可以使用docker [image] push命令上传镜像到仓库，默认上传到Docker Hub官方仓库（需要登录）。命令格式为docker [image] push NAME[:TAG] |[REGISTRY_HOST[:REGISTRY_PORT]/]NAME[:TAG]。

​	用户在Docker Hub网站注册后可以上传自制的镜像。例如，用户user上传本地的test:latest镜像，可以先添加新的标签user/test:latest，然后用docker [image] push命令上传镜像：

![](https://pic.imgdb.cn/item/6137341744eaada739b96e6e.jpg)

