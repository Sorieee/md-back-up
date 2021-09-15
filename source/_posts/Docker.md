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

# 4. 操作Docker容器

​	容器是Docker的另一个核心概念。简单来说，容器是镜像的一个运行实例。所不同的是，镜像是静态的只读文件，而容器带有运行时需要的可写文件层，同时，容器中的应用进程处于运行状态。

## 4.1 创建容器

**1．新建容器**

​	可以使用docker [container] create命令新建一个容器，例如：

![](https://pic.imgdb.cn/item/6138525d44eaada739aa64cb.jpg)

​	使用docker [container] create命令新建的容器处于停止状态，可以使用docker[container] start命令来启动它。

​	由于容器是整个Docker技术栈的核心，create命令和后续的run命令支持的选项都十分复杂，需要读者在实践中不断体会。

​	选项主要包括如下几大类：与容器运行模式相关、与容器环境配置相关、与容器资源限制和安全保护相关，参见表4-1～表4-3。

**表4-1 create命令与容器运行模式相关的选项**

![](https://pic.imgdb.cn/item/6138555744eaada739af82f8.jpg)

**表4-2 create命令与容器环境和配置相关的选项**

![](https://pic.imgdb.cn/item/6138559f44eaada739affe29.jpg)

**表4-3 create命令与容器资源限制和安全保护相关的选项**

![](https://pic.imgdb.cn/item/6138561a44eaada739b0e333.jpg)

**其他选项还包括：**

* -l, --label=[]：以键值对方式指定容器的标签信息；
* --label-file=[]：从文件中读取标签信息。

**2．启动容器**

​	使用docker [container] start命令来启动一个已经创建的容器。例如，启动刚创建的ubuntu容器：

![](https://pic.imgdb.cn/item/613856a344eaada739b22db0.jpg)

**3．新建并启动容器**

​	除了创建容器后通过start命令来启动，也可以直接新建并启动容器。

​	所需要的命令主要为docker [container] run，等价于先执行docker [container]create命令，再执行docker [container] start命令。

​	例如，下面的命令输出一个“Hello World”，之后容器自动终止：

![](https://pic.imgdb.cn/item/6138576a44eaada739b38513.jpg)

​	这跟在本地直接执行/bin/echo 'hello world’相比几乎感觉不出任何区别。

​	当利用docker [container] run来创建并启动容器时，Docker在后台运行的标准操作包括：

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载；
* 利用镜像创建一个容器，并启动该容器；
* 分配一个文件系统给容器，并在只读的镜像层外面挂载一层可读写层；
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；
* 从网桥的地址池配置一个IP地址给容器；
* 执行用户指定的应用程序；
* 执行完毕后容器被自动终止。

下面的命令启动一个bash终端，允许用户进行交互：

![](https://pic.imgdb.cn/item/6138580144eaada739b4983d.jpg)

​	其中，-t选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上，-i则让容器的标准输入保持打开。更多的命令选项可以通过man docker-run命令来查看。

​	在交互模式下，用户可以通过所创建的终端来输入命令，例如：

![](https://pic.imgdb.cn/item/6138585d44eaada739b538ae.jpg)

​	在容器内用ps命令查看进程，可以看到，只运行了bash应用，并没有运行其他无关的进程。

​	用户可以按Ctrl+d或输入exit命令来退出容器：

![](https://pic.imgdb.cn/item/6138587244eaada739b55f81.jpg)

​	对于所创建的bash容器，当用户使用exit命令退出bash进程之后，容器也会自动退出。这是因为对于容器来说，当其中的应用退出后，容器的使命完成，也就没有继续运行的必要了。

​	可以使用docker container wait CONTAINER [CONTAINER...]子命令来等待容器退出，并打印退出返回结果。

​	某些时候，执行docker [container] run时候因为命令无法正常执行容器会出错直接退出，此时可以查看退出的错误代码。

​	默认情况下，常见错误代码包括：

* 125:Docker daemon执行出错，例如指定了不支持的Docker命令参数；
* 126：所指定命令无法执行，例如权限出错；
* 127：容器内命令无法找到。



​	命令执行后出错，会默认返回命令的退出错误码。

**4．守护态运行**

​	更多的时候，需要让Docker容器在后台以守护态（Daemonized）形式运行。此时，可以通过添加-d参数来实现。

![](https://pic.imgdb.cn/item/6138599944eaada739b7cefb.jpg)

**5．查看容器输出**

​	要获取容器的输出信息，可以通过docker [container] logs命令。

​	该命令支持的选项包括：

* -details：打印详细信息；
* -f, -follow：持续保持输出；
* -since string：输出从某个时间开始的日志；
* -tail string：输出最近的若干日志；
* -t, -timestamps：显示时间戳信息；
* -until string：输出某个时间之前的日志。

![](https://pic.imgdb.cn/item/613859e044eaada739b84e2e.jpg)

## 4.2 停止容器

​	本节主要介绍Docker容器的pause/unpause、stop和prune子命令。

**1．暂停容器**

​	可以使用docker [container] pause CONTAINER [CONTAINER...]命令来暂停一个运行中的容器。

​	例如，启动一个容器，并将其暂停：

![](https://pic.imgdb.cn/item/61385a2344eaada739b8be28.jpg)

**2．终止容器**

​	可以使用docker [container] stop来终止一个运行中的容器。该命令的格式为docker [container] stop [-t|--time[=10]] [CONTAINER...]。

​	该命令会首先向容器发送SIGTERM信号，等待一段超时时间后（默认为10秒），再发送SIGKILL信号来终止容器：

![](https://pic.imgdb.cn/item/61385a5444eaada739b912c1.jpg)

​	此时，执行docker container prune命令，会自动清除掉所有处于停止状态的容器。

​	此外，还可以通过docker [container] kill直接发送SIGKILL信号来强行终止容器。

​	当Docker容器中指定的应用终结时，容器也会自动终止。例如，对于上一章节中只启动了一个终端的容器，用户通过exit命令或Ctrl+d来退出终端时，所创建的容器立刻终止，处于stopped状态。

​	可以用docker ps -qa命令看到所有容器的ID。例如：

![](https://pic.imgdb.cn/item/61385a8a44eaada739b97343.jpg)

​	docker [container] restart命令会将一个运行态的容器先终止，然后再重新启动：

![](https://pic.imgdb.cn/item/61385aa644eaada739b9a07e.jpg)

## 4.3 进入容器

​	在使用-d参数时，容器启动后会进入后台，用户无法看到容器中的信息，也无法进行操作。

​	这个时候如果需要进入容器进行操作，推荐使用官方的attach或exec命令。

**1. attach命令**

​	attach是Docker自带的命令，命令格式为：

![](https://pic.imgdb.cn/item/61385af444eaada739ba1f5d.jpg)

这个命令支持三个主要选项：

* --detach-keys[=[]]：指定退出attach模式的快捷键序列，默认是CTRL-pCTRL-q；
* --no-stdin=true|false：是否关闭标准输入，默认是保持打开；
* --sig-proxy=true|false：是否代理收到的系统信号给应用进程，默认为true。

下面示例如何使用该命令：

![](https://pic.imgdb.cn/item/61385b3144eaada739ba86db.jpg)

​	然而使用attach命令有时候并不方便。当多个窗口同时attach到同一个容器的时候，所有窗口都会同步显示；当某个窗口因命令阻塞时，其他窗口也无法执行操作了。

**2. exec命令**

​	从Docker的1.3.0版本起，Docker提供了一个更加方便的工具exec命令，可以在运行中容器内直接执行任意命令。

​	该命令的基本格式为：

![](https://pic.imgdb.cn/item/61385bc844eaada739bb7cb5.jpg)

​	比较重要的参数有：

* -d, --detach：在容器中后台执行命令；
* --detach-keys=""：指定将容器切回后台的按键；
* -e, --env=[]：指定环境变量列表；
* -i, --interactive=true|false：打开标准输入接受用户输入命令，默认值为false；
* --privileged=true|false：是否给执行命令以高权限，默认值为false；
* -t, --tty=true|false：分配伪终端，默认值为false；
* -u, --user=""：执行命令的用户名或ID。



​	例如，进入到刚创建的容器中，并启动一个bash：

![](https://pic.imgdb.cn/item/61385c7644eaada739bc9f0b.jpg)

​	可以看到会打开一个新的bash终端，在不影响容器内其他应用的前提下，用户可以与容器进行交互。

> **注意**
>
> ​	通过指定-it参数来保持标准输入打开，并且分配一个伪终端。通过exec命令对容器执行操作是最为推荐的方式。



​	进一步地，可以在容器中查看容器中的用户和进程信息：

![](https://pic.imgdb.cn/item/61385cab44eaada739bd2787.jpg)

## 4.4 删除容器

​	可以使用docker [container] rm命令来删除处于终止或退出状态的容器，命令格式为docker [container] rm [-f|--force] [-l|--link] [-v|--volumes] CONTAINER[CONTAINER...]。

​	主要支持的选项包括：

* -f, --force=false：是否强行终止并删除一个运行中的容器；
* -l, --link=false：删除容器的连接，但保留容器；
* -v, --volumes=false：删除容器挂载的数据卷。

![](https://pic.imgdb.cn/item/61385d8d44eaada739beeb03.jpg)

​	默认情况下，docker rm命令只能删除已经处于终止或退出状态的容器，并不能删除还处于运行状态的容器。

​	如果要直接删除一个运行中的容器，可以添加-f参数。Docker会先发送SIGKILL信号给容器，终止其中的应用，之后强行删除：

![](https://pic.imgdb.cn/item/61385dfb44eaada739bf9f2b.jpg)

## 4.5 导入和导出容器

​	某些时候，需要将容器从一个系统迁移到另外一个系统，此时可以使用Docker的导入和导出功能，这也是Docker自身提供的一个重要特性。

**1．导出容器**

​	导出容器是指，导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态。可以使用docker [container] export命令，该命令格式为：

![](https://pic.imgdb.cn/item/61385e5944eaada739c04790.jpg)

​	其中，可以通过-o选项来指定导出的tar文件名，也可以直接通过重定向来实现。

​	首先，查看所有的容器，如下所示：

![](https://pic.imgdb.cn/item/61385ee444eaada739c13e06.jpg)

​	分别导出ce554267d7a4容器和e812617b41f6容器到文件test_for_run.tar文件和test_for_stop.tar文件：

![](https://pic.imgdb.cn/item/61385f3c44eaada739c1db1e.jpg)

​	之后，可将导出的tar文件传输到其他机器上，然后再通过导入命令导入到系统中，实现容器的迁移。

**2．导入容器**

​	导出的文件又可以使用docker [container] import命令导入变成镜像，该命令格式为：

![](https://pic.imgdb.cn/item/61385ffa44eaada739c33d02.jpg)

​	用户可以通过-c, --change=[]选项在导入的同时执行对容器进行修改的Dockerfile指令（可参考后续相关章节）。

​	下面将导出的test_for_run.tar文件导入到系统中：

![](https://pic.imgdb.cn/item/6138612f44eaada739c56148.jpg)

​	之前的镜像章节（第3章）中，笔者曾介绍过使用docker load命令来导入一个镜像文件，与docker [container] import命令十分类似。

​	实际上，既可以使用docker load命令来导入镜像存储文件到本地镜像库，也可以使用docker [container] import命令来导入一个容器快照到本地镜像库。这两者的区别在于：容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积更大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。

## 4.6 查看容器

**1．查看容器详情**

​	查看容器详情可以使用docker container inspect [OPTIONS] CONTAINER[CONTAINER...]子命令。

​	例如，查看某容器的具体信息，会以json格式返回包括容器Id、创建时间、路径、状态、镜像、配置等在内的各项信息：

![](https://pic.imgdb.cn/item/6138617f44eaada739c5ec9b.jpg)

**2．查看容器内进程**

​	查看容器内进程可以使用docker [container] top [OPTIONS] CONTAINER[CONTAINER...]子命令。

​	这个子命令类似于Linux系统中的top命令，会打印出容器内的进程信息，包括PID、用户、时间、命令等。例如，查看某容器内的进程信息，命令如下：

![](https://pic.imgdb.cn/item/613861af44eaada739c63d25.jpg)

**3．查看统计信息**

​	查看统计信息可以使用docker [container] stats [OPTIONS] [CONTAINER...]子命令，会显示CPU、内存、存储、网络等使用情况的统计信息。

​	支持选项包括：

* -a, -all：输出所有容器统计信息，默认仅在运行中；
* -format string：格式化输出信息；
* -no-stream：不持续输出，默认会自动更新持续实时结果；
* -no-trunc：不截断输出信息。



​	例如，查看当前运行中容器的系统资源使用统计：

![](https://pic.imgdb.cn/item/613861f444eaada739c6b771.jpg)

## 4.7 其他容器命令

**1．复制文件**

​	container cp命令支持在容器和主机之间复制文件。命令格式为docker[container] cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-。支持的选项包括：

* -a, -archive：打包模式，复制文件会带有原始的uid/gid信息；
* -L, -follow-link：跟随软连接。当原路径为软连接时，默认只复制链接信息，使用该选项会复制链接的目标内容。



​	例如，将本地的路径data复制到test容器的/tmp路径下：

![](https://pic.imgdb.cn/item/6138648a44eaada739cb4956.jpg)

**2．查看变更**

​	container diff查看容器内文件系统的变更。命令格式为docker [container] diffCONTAINER。

​	例如，查看test容器内的数据修改：

![](https://pic.imgdb.cn/item/613864f744eaada739cc0101.jpg)

**3．查看端口映射**

​	container port命令可以查看容器的端口映射情况。命令格式为docker containerport CONTAINER [PRIVATE_PORT[/PROTO]]。例如，查看test容器的端口映射情况：

![](https://pic.imgdb.cn/item/6138650d44eaada739cc3773.jpg)

**4．更新配置**

​	container update命令可以更新容器的一些运行时配置，主要是一些资源限制份额。命令格式为docker [container] update [OPTIONS] CONTAINER[CONTAINER...]。

​	支持的选项包括：

* -blkio-weight uint16：更新块IO限制，10～1000，默认值为0，代表着无限制；
* -cpu-period int：限制CPU调度器CFS（Completely Fair Scheduler）使用时间，单位为微秒，最小1000；
* -cpu-quota int：限制CPU调度器CFS配额，单位为微秒，最小1000；
* -cpu-rt-period int：限制CPU调度器的实时周期，单位为微秒；
* -cpu-rt-runtime int：限制CPU调度器的实时运行时，单位为微秒；
*  -c, -cpu-shares int：限制CPU使用份额；
* -cpus decimal：限制CPU个数；
* -cpuset-cpus string：允许使用的CPU核，如0-3,0,1；
* -cpuset-mems string：允许使用的内存块，如0-3,0,1；
* -kernel-memory bytes：限制使用的内核内存；
* -m, -memory bytes：限制使用的内存；
* -memory-reservation bytes：内存软限制；
* -memory-swap bytes：内存加上缓存区的限制，-1表示为对缓冲区无限制；
* -restart string：容器退出后的重启策略。



例如，限制总配额为1秒，容器test所占用时间为10%，代码如下所示：

![](https://pic.imgdb.cn/item/613865bd44eaada739cd669b.jpg)

# 5. 访问Docker仓库

​	仓库（Repository）是集中存放镜像的地方，又分公共仓库和私有仓库。

​	有时候容易把仓库与注册服务器（Registry）混淆。实际上注册服务器是存放仓库的具体服务器，一个注册服务器上可以有多个仓库，而每个仓库下面可以有多个镜像。从这方面来说，仓库可以被认为是一个具体的项目或目录。例如对于仓库地址private-docker. com/ubuntu来说，private-docker.com是注册服务器地址，ubuntu是仓库名。

## 5.1 Docker Hub公共镜像市场

​	Docker Hub是Docker官方提供的最大的公共镜像仓库，目前包括了超过100000的镜像，地址为https://hub.docker.com。大部分对镜像的需求，都可以通过在Docker Hub中直接下载镜像来实现，如图5-1所示。

**1．登录**

​	可以通过命令行执行docker login命令来输入用户名、密码和邮箱来完成注册和登录。注册成功后，本地用户目录下会自动创建．docker/config.json文件，保存用户的认证信息。

​	登录成功的用户可以上传个人制作的镜像到Docker Hub。

**2．基本操作**

​	用户无须登录即可通过docker search命令来查找官方仓库中的镜像，并利用docker [image] pull命令来将它下载到本地。

​	根据是否为官方提供，可将这些镜像资源分为两类：

* 一种是类似于centos这样的基础镜像，也称为根镜像。这些镜像是由Docker公司创建、验证、支持、提供，这样的镜像往往使用单个单词作为名字；
* 另一种类型的镜像，比如ansible/centos7-ansible镜像，是由Docker用户ansible创建并维护的，带有用户名称为前缀，表明是某用户下的某仓库。可以通过用户名称前缀“user_name/镜像名”来指定使用某个用户提供的镜像。

**3．自动创建**

​	自动创建（Automated Builds）是Docker Hub提供的自动化服务，这一功能可以自动跟随项目代码的变更而重新构建镜像。

​	例如，用户构建了某应用镜像，如果应用发布新版本，用户需要手动更新镜像。而自动创建则允许用户通过Docker Hub指定跟踪一个目标网站（目前支持GitHub或BitBucket）上的项目，一旦项目发生新的提交，则自动执行创建。

要配置自动创建，包括如下的步骤：

​	1）创建并登录Docker Hub，以及目标网站如Github；

​	2）在目标网站中允许Docker Hub访问服务；

​	3）在Docker Hub中配置一个“自动创建”类型的项目；

​	4）选取一个目标网站中的项目（需要含Dockerfile）和分支；

​	5）指定Dockerfile的位置，并提交创建。之后，可以在Docker Hub的“自动创建”页面中跟踪每次创建的状态。

## 5.2 第三方镜像市场

​	国内不少云服务商都提供了Docker镜像市场，包括腾讯云、网易云、阿里云等。下面以时速云为例，介绍如何使用这些市场，如图5-2所示。

![](https://pic.imgdb.cn/item/61419b902ab3f51d91c8dabf.jpg)

**1．查看镜像**

​	访问https://hub.tenxcloud.com，即可看到已存在的仓库和存储的镜像，包括Ubuntu、Java、Mongo、MySQL、Nginx等热门仓库和镜像。时速云官方仓库中的镜像会保持与DockerHub中官方镜像的同步。

​	以MongoDB仓库为例，其中包括了2.6、3.0和3.2等镜像。

**2．下载镜像**

​	下载镜像也是使用docker pull命令，但是要在镜像名称前添加注册服务器的具体地址。格式为`index.tenxcloud.com/<namespace>/<repository>:<tag>`。

​	例如，要下载Docker官方仓库中的node:latest镜像，可以使用如下命令：

```
docker pull index.tenxcloud.com/docker_library/node:latest
```

​	下载后，可以更新镜像的标签，与官方标签保持一致，方便使用：

![](https://pic.imgdb.cn/item/61419c2e2ab3f51d91c95ffb.jpg)

## 5.3 搭建本地私有仓库

**1．使用registry镜像创建私有仓库**

​	安装Docker后，可以通过官方提供的registry镜像来简单搭建一套本地私有仓库环境：

```sh
docker run -d -p 5000:5000 registry:2
```

​	这将自动下载并启动一个registry容器，创建本地的私有仓库服务。

​	默认情况下，仓库会被创建在容器的/var/lib/registry目录下。可以通过-v参数来将镜像文件存放在本地的指定路径。例如下面的例子将上传的镜像放到/opt/data/registry目录：

```sh
$ docker run -d -p 5000:5000-v /opt/data/registry:/var/1ib/registry registry:2
```

**2．管理私有仓库**

​	首先在本书环境的笔记本上（Linux Mint）搭建私有仓库，查看其地址为10.0.2.2:5000，然后在虚拟机系统（Ubuntu 18.04）里测试上传和下载镜像。

​	在Ubuntu 18.04系统查看已有的镜像：

![](https://pic.imgdb.cn/item/61419d632ab3f51d91ca735d.jpg)

使用docker push上传标记的镜像：

```sh
docker push 10.10.2.2:5000/test
```

用curl查看仓库10.0.2.2:5000中的镜像：

![](https://pic.imgdb.cn/item/61419d982ab3f51d91caa43e.jpg)

​	比较新的Docker版本对安全性要求较高，会要求仓库支持SSL/TLS证书。对于内部使用的私有仓库，可以自行配置证书或关闭对仓库的安全性检查。

​	首先，修改Docker daemon的启动参数，添加如下参数，表示信任这个私有仓库，不进行安全证书检查：

```sh
DOCKER_OPTS='--insecure-registry 10.0.2.2:5000”
```

​	之后重启Docker服务，并从私有仓库中下载镜像到本地：

![](https://pic.imgdb.cn/item/61419e042ab3f51d91cb07e1.jpg)

​	下载后，还可以添加一个更通用的标签ubuntu:18.04，方便后续使用：

![](https://pic.imgdb.cn/item/61419e202ab3f51d91cb1e14.jpg)

# 6. Docker数据管理

​	在生产环境中使用Docker，往往需要对数据进行持久化，或者需要在多个容器之间进行数据共享，这必然涉及容器的数据管理操作。

​	容器中的管理数据主要有两种方式：

* 数据卷（Data Volumes）：容器内数据直接映射到本地主机环境；
* 数据卷容器（Data Volume Containers）：使用特定容器维护数据卷。

## 6.1 数据卷

​	数据卷（Data Volumes）是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于Linux中的mount行为。

​	数据卷可以提供很多有用的特性：

* 数据卷可以在容器之间共享和重用，容器间传递数据将变得高效与方便；
* 对数据卷内数据的修改会立马生效，无论是容器内操作还是本地操作；
* 对数据卷的更新不会影响镜像，解耦开应用和数据；
* 卷会一直存在，直到没有容器使用，可以安全地卸载它。

**1．创建数据卷**

```sh
docker volume create -d local test
```

​	此时，查看/var/lib/docker/volumes路径下，会发现所创建的数据卷位置：

![](https://pic.imgdb.cn/item/61419f0e2ab3f51d91cbf526.jpg)

​	除了create子命令外，docker volume还支持inspect（查看详细信息）、ls（列出已有数据卷）、prune（清理无用数据卷）、rm（删除数据卷）等，读者可以自行实践。

**2．绑定数据卷**

​	除了使用volume子命令来管理数据卷外，还可以在创建容器时将主机本地的任意路径挂载到容器内作为数据卷，这种形式创建的数据卷称为绑定数据卷。

​	在用docker [container] run命令的时候，可以使用-mount选项来使用数据卷。

​	-mount选项支持三种类型的数据卷，包括：

* volume：普通数据卷，映射到主机/var/lib/docker/volumes路径下；
* bind：绑定数据卷，映射到主机指定路径下；
* tmpfs：临时数据卷，只存在于内存中。



​	下面使用training/webapp镜像创建一个Web容器，并创建一个数据卷挂载到容器的/opt/webapp目录：

![](https://pic.imgdb.cn/item/61419f9e2ab3f51d91cc665c.jpg)

​	这个功能在进行应用测试的时候十分方便，比如用户可以放置一些程序或数据到本地目录中实时进行更新，然后在容器内运行和使用。

​	另外，本地目录的路径必须是绝对路径，容器内路径可以为相对路径。如果目录不存在，Docker会自动创建。

​	Docker挂载数据卷的默认权限是读写（rw），用户也可以通过ro指定为只读：

![](https://pic.imgdb.cn/item/6141a00f2ab3f51d91ccbd9b.jpg)

​	加了：ro之后，容器内对所挂载数据卷内的数据就无法修改了。

​	如果直接挂载一个文件到容器，使用文件编辑工具，包括vi或者sed --in-place的时候，可能会造成文件inode的改变。从Docker 1.1.0起，这会导致报错误信息。所以推荐的方式是直接挂载文件所在的目录到容器内。

## 6.2 数据卷容器

​	如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。数据卷容器也是一个容器，但是它的目的是专门提供数据卷给其他容器挂载。

​	首先，创建一个数据卷容器dbdata，并在其中创建一个数据卷挂载到/dbdata：

![](https://pic.imgdb.cn/item/6141a1502ab3f51d91cdb99e.jpg)

![](https://pic.imgdb.cn/item/6141a18c2ab3f51d91cde7b7.jpg)

>  使用--volumes-from参数所挂载数据卷的容器自身并不需要保持在运行状态。

​	如果删除了挂载的容器（包括dbdata、db1和db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时显式使用docker rm -v命令来指定同时删除关联的容器。

## 6.3 利用数据卷容器来迁移数据

​	可以利用数据卷容器对其中的数据卷进行备份、恢复，以实现数据的迁移。

**1．备份**

![](https://pic.imgdb.cn/item/6141a23e2ab3f51d91ce6b9f.jpg)

​	首先利用ubuntu镜像创建了一个容器worker。使用--volumes-from dbdata参数来让worker容器挂载dbdata容器的数据卷（即dbdata数据卷）；使用-v$(pwd):/backup参数来挂载本地的当前目录到worker容器的/backup目录。

​	worker容器启动后，使用tar cvf /backup/backup.tar /dbdata命令将/dbdata下内容备份为容器内的/backup/backup.tar，即宿主主机当前目录下的backup.tar。

**2．恢复**

​	如果要恢复数据到一个容器，可以按照下面的操作。

​	首先创建一个带有数据卷的容器dbdata2：

![](https://pic.imgdb.cn/item/6141a2a62ab3f51d91cec0ae.jpg)

# 7. 端口映射与容器互联

​	docker除了通过网络访问外，还提供了两个很方便的功能来满足服务访问的基本需求：一个是允许映射容器内应用的服务端口到本地宿主主机；另一个是互联机制实现多个容器间通过容器名来快速访问。本章将分别讲解这两个很实用的功能。

## 7.1 端口映射实现容器访问

**1．从外部访问容器应用**

​	在启动容器的时候，如果不指定对应参数，在容器外部是无法通过网络来访问容器内的网络应用和服务的。

​	当容器中运行一些网络应用，要让外部访问这些应用时，可以通过-P或-p参数来指定端口映射。当使用-P（大写的）标记时，Docker会随机映射一个49000～49900的端口到内部容器开放的网络端口：

![](https://pic.imgdb.cn/item/6141aa9f2ab3f51d91d5477d.jpg)

​	-p（小写的）则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有IP:HostPort:ContainerPort | IP::ContainerPort |HostPort:ContainerPort。

**2．映射所有接口地址**

​	使用HostPort:ContainerPort格式本地的5000端口映射到容器的5000端口，可以执行如下命令：

```
docker run -d -p 5000:5000 training/webapp python app.py
```

**3．映射到指定地址的指定端口**

​	可以使用IP:HostPort:ContainerPort格式指定映射使用一个特定地址，比如localhost地址127.0.0.1：

```sh
docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

**4．映射到指定地址的任意端口**

​	使用IP::ContainerPort绑定localhost的任意端口到容器的5000端口，本地主机会自动分配一个端口：

![](https://pic.imgdb.cn/item/6141af722ab3f51d91d9ac21.jpg)

**5．查看映射端口配置**

​	使用docker port来查看当前映射的端口配置，也可以查看到绑定的地址：

![](https://pic.imgdb.cn/item/6141affb2ab3f51d91da1a0e.jpg)

## 7.2 互联机制实现便捷互访

​	容器的互联（linking）是一种让多个容器中的应用进行快速交互的方式。它会在源和接收容器之间创建连接关系，接收容器可以通过容器名快速访问到源容器，而不用指定具体的IP地址。

**1．自定义容器命名**

​	连接系统依据容器的名称来执行。因此，首先需要自定义一个好记的容器命名。虽然当创建容器的时候，系统默认会分配一个名字，但自定义命名容器有两个好处：

* 自定义的命名，比较好记，比如一个Web应用容器我们可以给它起名叫web，一目了然；
* 当要连接其他容器时候（即便重启），也可以使用容器名而不用改变，比如连接web容器到db容器。

使用--name标记可以为容器自定义命名：

![](https://pic.imgdb.cn/item/6141b0712ab3f51d91da7d65.jpg)

> 容器的名称是唯一的。如果已经命名了一个叫web的容器，当你要再次使用web这个名称的时候，需要先用docker rm命令删除之前创建的同名容器。

​	在执行docker [container] run的时候如果添加--rm标记，则容器在终止后会立刻删除。注意，--rm和-d参数不能同时使用。

**2．容器互联**

​	使用--link参数可以让容器之间安全地进行交互。

​	下面先创建一个新的数据库容器：

![](https://pic.imgdb.cn/item/6141b1302ab3f51d91db390b.jpg)

![](https://pic.imgdb.cn/item/6141b1482ab3f51d91db4f96.jpg)

​	可以看到自定义命名的容器：db和web, db容器的names列有db也有web/db。这表示web容器链接到db容器，web容器将被允许访问db容器的信息。

​	Docker相当于在两个互联的容器之间创建了一个虚机通道，而且不用映射它们的端口到宿主主机上。在启动db容器的时候并没有使用-p和-P标记，从而避免了暴露数据库服务端口到外部网络上。

​	Docker通过两种方式为容器公开连接信息：

* 更新环境变量；
* 更新/etc/hosts文件。



​	使用env命令来查看web容器的环境变量：

![](https://pic.imgdb.cn/item/6141b1e42ab3f51d91dbd3de.jpg)

​	其中DB_开头的环境变量是供web容器连接db容器使用，前缀采用大写的连接别名。

​	除了环境变量，Docker还添加host信息到父容器的/etc/hosts的文件。下面是父容器web的hosts文件：

![](https://pic.imgdb.cn/item/6141b2032ab3f51d91dbeb4f.jpg)

​	这里有2个hosts信息，第一个是web容器，web容器用自己的id作为默认主机名，第二个是db容器的IP和主机名。

​	可以在web容器中安装ping命令来测试跟db容器的连通：

![](https://pic.imgdb.cn/item/6141b2282ab3f51d91dc08db.jpg)

# 8. 使用Dockerfile创建镜像

​	Dockerfile是一个文本格式的配置文件，用户可以使用Dockerfile来快速创建自定义的镜像。

## 8.1 基本结构

​	一般而言，Dockerfile主体内容分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

![](https://pic.imgdb.cn/item/6141b39e2ab3f51d91dd3655.jpg)

​	首行可以通过注释来指定解析器命令，后续通过注释说明镜像的相关信息。主体部分首先使用FROM指令指明所基于的镜像名称，接下来一般是使用LABEL指令说明维护者信息。后面则是镜像操作指令，例如RUN指令将对镜像执行跟随的命令。每运行一条RUN指令，镜像添加新的一层，并提交。最后是CMD指令，来指定运行容器时的操作命令。

​	下面是Docker Hub上两个热门镜像nginx和Go的Dockerfile的例子，通过这两个例子。读者可以对Dockerfile结构有个基本的感知。

​	第一个是在debian:jessie基础镜像基础上安装Nginx环境，从而创建一个新的nginx镜像：

![](https://pic.imgdb.cn/item/6141b3ef2ab3f51d91dd8604.jpg)

​	第二个是基于buildpack-deps:jessie-scm基础镜像，安装Golang相关环境，制作一个Go语言的运行环境镜像：

![](https://pic.imgdb.cn/item/6141b40d2ab3f51d91dda111.jpg)

## 8.2 指令说明

​	Dockerfile中指令的一般格式为INSTRUCTION arguments，包括“配置指令”（配置镜像信息）和“操作指令”（具体执行操作），参见表8-1。

![](https://pic.imgdb.cn/item/6141b4622ab3f51d91ddeaf3.jpg)

### 8.2.1 配置指令

**1. ARG**

​	定义创建镜像过程中使用的变量。

​	格式为`ARG <name>[=<default value>]`。

​	在执行docker build时，可以通过-build-arg[=]来为变量赋值。当镜像编译成功后，ARG指定的变量将不再存在（ENV指定的变量将在镜像中保留）。

​	Docker内置了一些镜像创建变量，用户可以直接使用而无须声明，包括（不区分大小写）HTTP_PROXY、HTTPS_PROXY、FTP_PROXY、NO_PROXY。

**2. FROM**

​	格式为`FROM <image> [AS <name>]或FROM <image>:<tag> [AS <name>]或FROM <image>@<digest> [AS <name>]`。

​	任何Dockerfile中第一条指令必须为FROM指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令（每个镜像一次）。

​	为了保证镜像精简，可以选用体积较小的镜像如Alpine或Debian作为基础镜像。例如：

![](https://pic.imgdb.cn/item/6141b4e72ab3f51d91de5893.jpg)

**3. LABEL**

​	LABEL指令可以为生成的镜像添加元数据标签信息。这些信息可以用来辅助过滤出特定镜像。

​	格式为`LABEL <key>=<value> <key>=<value> <key>=<value> ...`。

![](https://pic.imgdb.cn/item/6141b52b2ab3f51d91de9855.jpg)

**4. EXPOSE**

​	声明镜像内服务监听的端口。

​	格式为`EXPOSE <port> [<port>/<protocol>...]`。

例如：

![](https://pic.imgdb.cn/item/6141b5502ab3f51d91debe34.jpg)

​	注意该指令只是起到声明作用，并不会自动完成端口映射。

​	如果要映射端口出来，在启动容器时可以使用-P参数（Docker主机会自动分配一个宿主机的临时端口）或-p HOST_PORT:CONTAINER_PORT参数（具体指定所映射的本地端口）。

**5. ENV**

​	指定环境变量，在镜像生成过程中会被后续RUN指令使用，在镜像启动的容器中也会存在。

​	格式为`ENV <key> <value>`或`ENV <key>=<value> ...`。

![](https://pic.imgdb.cn/item/6141b5852ab3f51d91deea10.jpg)

**6.ENTRYPOINT**

​	指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执行，所有传入值作为该命令的参数。

​	支持两种格式：

* ENTRYPOINT ["executable", "param1", "param2"]:exec调用执行；
*  ENTRYPOINT command param1 param2:shell中执行。



​	此时，CMD指令指定值将作为根命令的参数。

​	每个Dockerfile中只能有一个ENTRYPOINT，当指定多个时，只有最后一个起效。

​	在运行时，可以被--entrypoint参数覆盖掉，如docker run --entrypoint。

**7. VOLUME**

​	创建一个数据卷挂载点。

​	格式为VOLUME ["/data"]。

​	运行容器时可以从本地主机或其他容器挂载数据卷，一般用来存放数据库和需要保持的数据等。

**8. USER**

​	指定运行容器时的用户名或UID，后续的RUN等指令也会使用指定的用户身份。

​	格式为USER daemon。

​	当服务不需要管理员权限时，可以通过该命令指定运行用户，并且可以在Dockerfile中创建所需要的用户。例如：

![](https://pic.imgdb.cn/item/6141b8552ab3f51d91e3f324.jpg)

**9. WORKDIR**

​	为后续的RUN、CMD、ENTRYPOINT指令配置工作目录。

​	格式为WORKDIR /path/to/workdir。

​	可以使用多个WORKDIR指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如：

![](https://pic.imgdb.cn/item/6141b8772ab3f51d91e45284.jpg)

**10. ONBUILD**

​	指定当基于所生成镜像创建子镜像时，自动执行的操作指令。

​	格式为ONBUILD [INSTRUCTION]。

​	例如，使用如下的Dockerfile创建父镜像ParentImage，指定ONBUILD指令：

![](https://pic.imgdb.cn/item/6141b8962ab3f51d91e4b0c9.jpg)

​	使用docker build命令创建子镜像ChildImage时（FROM ParentImage），会首先执行ParentImage中配置的ONBUILD指令：

![](https://pic.imgdb.cn/item/6141b8e62ab3f51d91e593f6.jpg)

**11. STOPSIGNAL**

​	指定所创建镜像启动的容器接收退出的信号值：

![](https://pic.imgdb.cn/item/6141b91e2ab3f51d91e6382d.jpg)

**12. HEALTHCHECK**

​	配置所启动容器如何进行健康检查（如何判断健康与否），自Docker 1.12开始支持。

​	格式有两种：

* HEALTHCHECK [OPTIONS] CMD command：根据所执行命令返回值是否为0来判断；
*  HEALTHCHECK NONE：禁止基础镜像中的健康检查。



​	OPTION支持如下参数：

* -interval=DURATION (default: 30s)：过多久检查一次；
* -timeout=DURATION (default: 30s)：每次检查等待结果的超时；
* -retries=N (default: 3)：如果失败了，重试几次才最终确定失败。

**13. SHELL**

​	指定其他命令使用shell时的默认shell类型：

![](https://pic.imgdb.cn/item/6141b9842ab3f51d91e75152.jpg)

### 8.2.2 操作指令

**1. RUN**

​	运行指定命令。

​	格式为`RUN <command>`或RUN ["executable", "param1", "param2"]。注意后者指令会被解析为JSON数组，因此必须用双引号。前者默认将在shell终端中运行命令，即/bin/sh -c；后者则使用exec执行，不会启动shell环境。

​	指定使用其他终端类型可以通过第二种方式实现，例如RUN ["/bin/bash", "-c","echo hello"]。

​	每条RUN指令将在当前镜像基础上执行指定命令，并提交为新的镜像层。当命令较长时可以使用\来换行。例如：

![](https://pic.imgdb.cn/item/6141ba0a2ab3f51d91e8d1e0.jpg)

**2. CMD**

​	CMD指令用来指定启动容器时默认执行的命令。

​	支持三种格式：

* CMD ["executable", "param1", "param2"]：相当于执行executableparam1 param2，推荐方式；
* CMD command param1 param2：在默认的Shell中执行，提供给需要交互的应用；
* CMD ["param1", "param2"]：提供给ENTRYPOINT的默认参数。



​	每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。

​	如果用户启动容器时候手动指定了运行的命令（作为run命令的参数），则会覆盖掉CMD指定的命令。

**3. ADD**

​	添加内容到镜像。

​	格式为`ADD <src> <dest>`。

​	该命令将复制指定的`<src>`路径下内容到容器中的`<dest>`路径下。

​	其中`<src>`可以是Dockerfile所在目录的一个相对路径（文件或目录）；也可以是一个URL；还可以是一个tar文件（自动解压为目录）`<dest>`可以是镜像内绝对路径，或者相对于工作目录（WORKDIR）的相对路径。

​	路径支持正则格式，例如：

![](https://pic.imgdb.cn/item/6141bb102ab3f51d91ebc34f.jpg)

**4. COPY**

​	复制内容到镜像。

​	格式为`COPY <src> <dest>`。复制本地主机的`<src>`（为Dockerfile所在目录的相对路径，文件或目录）下内容到镜像中的`<dest>`。目标路径不存在时，会自动创建。路径同样支持正则格式。COPY与ADD指令功能类似，当使用本地目录为源目录时，推荐使用COPY。

## 8.3 创建镜像

​	编写完成Dockerfile之后，可以通过docker [image] build命令来创建镜像。

​	基本的格式为docker build [OPTIONS] PATH | URL | -。

​	该命令将读取指定路径下（包括子目录）的Dockerfile，并将该路径下所有数据作为上下文（Context）发送给Docker服务端。Docker服务端在校验Dockerfile格式通过后，逐条执行其中定义的指令，碰到ADD、COPY和RUN指令会生成一层新的镜像。最终如果创建镜像成功，会返回最终镜像的ID。

​	如果上下文过大，会导致发送大量数据给服务端，延缓创建过程。因此除非是生成镜像所必需的文件，不然不要放到上下文路径下。如果使用非上下文路径下的Dockerfile，可以通过-f选项来指定其路径。

​	要指定生成镜像的标签信息，可以通过-t选项。该选项可以重复使用多次为镜像一次添加多个名称。

​	例如，上下文路径为/tmp/docker_builder/，并且希望生成镜像标签为builder/first_image:1.0.0，可以使用下面的命令：

![](https://pic.imgdb.cn/item/6141bbb32ab3f51d91edace4.jpg)

### 8.3.1 命令选项

​	docker [image] build命令支持一系列的选项，可以调整创建镜像过程的行为，参见表8-2。

![](https://pic.imgdb.cn/item/6141bbd82ab3f51d91ee16ec.jpg)

### 8.3.2 选择父镜像

​	大部分情况下，生成新的镜像都需要通过FROM指令来指定父镜像。父镜像是生成镜像的基础，会直接影响到所生成镜像的大小和功能。

​	用户可以选择两种镜像作为父镜像，一种是所谓的基础镜像（baseimage），另外一种是普通的镜像（往往由第三方创建，基于基础镜像）。

​	基础镜像比较特殊，其Dockerfile中往往不存在FROM指令，或者基于scratch镜像（FROM scratch），这意味着其在整个镜像树中处于根的位置。

​	下面的Dockerfile定义了一个简单的基础镜像，将用户提前编译好的二进制可执行文件binary复制到镜像中，运行容器时执行binary命令：

![](https://pic.imgdb.cn/item/6141bc3c2ab3f51d91ef43d3.jpg)

### 8.3.3 使用.dockerignore文件

​	可以通过．dockerignore文件（每一行添加一条匹配模式）来让Docker忽略匹配路径或文件，在创建镜像时候不将无关数据发送到服务端。

​	例如下面的例子中包括了6行忽略的模式（第一行为注释）：

![](https://pic.imgdb.cn/item/6141bcac2ab3f51d91f0a038.jpg)

* dockerignore文件中模式语法支持Golang风格的路径正则格式：
* “*”表示任意多个字符；

* “? ”代表单个字符；
* “! ”表示不匹配（即不忽略指定的路径或文件）。

### 8.3.4 多步骤创建

​	自17.05版本开始，Docker支持多步骤镜像创建（Multi-stage build）特性，可以精简最终生成的镜像大小。

​	对于需要编译的应用（如C、Go或Java语言等）来说，通常情况下至少需要准备两个环境的Docker镜像：

* 编译环境镜像：包括完整的编译引擎、依赖库等，往往比较庞大。作用是编译应用为二进制文件；
* 运行环境镜像：利用编译好的二进制文件，运行应用，由于不需要编译环境，体积比较小。



​	使用多步骤创建，可以在保证最终生成的运行环境镜像保持精简的情况下，使用单一的Dockerfile，降低维护复杂度。

​	以Go语言应用为例。创建干净目录，进入到目录中，创建main.go文件，内容为：

![](https://pic.imgdb.cn/item/6141bd362ab3f51d91f2314f.jpg)

​	创建Dockerfile，使用golang:1.9镜像编译应用二进制文件为app，使用精简的镜像alpine:latest作为运行环境。Dockerfile完整内容为：

![](https://pic.imgdb.cn/item/6141bd492ab3f51d91f2688b.jpg)

![](https://pic.imgdb.cn/item/6141bda52ab3f51d91f37aca.jpg)

## 8.4 最佳实践

​	所谓最佳实践，就是从需求出发，来定制适合自己、高效方便的镜像。

​	首先，要尽量吃透每个指令的含义和执行效果，多编写一些简单的例子进行测试，弄清楚了再撰写正式的Dockerfile。此外，Docker Hub官方仓库中提供了大量的优秀镜像和对应的Dockefile，可以通过阅读它们来学习如何撰写高效的Dockerfile。

​	笔者在应用过程中，也总结了一些实践经验。建议读者在生成镜像过程中，尝试从如下角度进行思考，完善所生成镜像：

* 精简镜像用途：尽量让每个镜像的用途都比较集中单一，避免构造大而复杂、多功能的镜像；
* 选用合适的基础镜像：容器的核心是应用。选择过大的父镜像（如Ubuntu系统镜像）会造成最终生成应用镜像的臃肿，推荐选用瘦身过的应用镜像（如node:slim），或者较为小巧的系统镜像（如alpine、busybox或debian）；
* 提供注释和维护者信息：Dockerfile也是一种代码，需要考虑方便后续的扩展和他人的使用；
* 正确使用版本号：使用明确的版本号信息，如1.0,2.0，而非依赖于默认的latest。通过版本号可以避免环境不一致导致的问题；
* 减少镜像层数：如果希望所生成镜像的层数尽量少，则要尽量合并RUN、ADD和COPY指令。通常情况下，多个RUN指令可以合并为一条RUN指令；
* 恰当使用多步骤创建（17.05+版本支持）：通过多步骤创建，可以将编译和运行等过程分开，保证最终生成的镜像只包括运行应用所需要的最小化环境。当然，用户也可以通过分别构造编译镜像和运行镜像来达到类似的结果，但这种方式需要维护多个Dockerfile。
* 使用.dockerignore文件：使用它可以标记在执行docker build时忽略的路径和文件，避免发送不必要的数据内容，从而加快整个镜像创建过程。
* 及时删除临时文件和缓存文件：特别是在执行apt-get指令后，/var/cache/apt下面会缓存了一些安装包；
* 提高生成速度：如合理使用cache，减少内容目录下的文件，或使用.dockerignore文件指定等；
* 调整合理的指令顺序：在开启cache的情况下，内容不变的指令尽量放在前面，这样可以尽量复用；
* 减少外部源的干扰：如果确实要从外部引入数据，需要指定持久的地址，并带版本信息等，让他人可以复用而不出错。

