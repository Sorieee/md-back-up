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
sudo yum updatesudo yum install -y yum-utils \	device-mapper-persistent-data \	lvm2
```

添加Docker稳定版本的yum软件源：

```sh
sudo yum-config-manager \	--add-repo \	https://download.docker.com/linux/centos/docker-ce.repo 
```

之后更新yum软件源缓存，并安装Docker：

```sh
sudo yum updatesudo yum install -y docker-ce
```

最后确认Docker服务启动正常

```sh
sudo systemctl start dockersudo systemctl status docker
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
{	"debug" : true,	"hosts' : ["tcp://127.0.0.1:2376" ]}    
```

​	对于CentOS、RedHat等系统，服务通过systemd来管理，配置文件路径为/etc/systemd/system/docker.service.d/docker.conf。更新配置后需要通过systemctl命令来管理Docker服务：

```
sudo systemctl daemon-reloadsudo systemctl start docker.service 
```

​	此外，如果服务工作不正常，可以通过查看Docker服务的日志信息来确定问题，例如在RedHat系统上日志文件可能为/var/log/messages，在Ubuntu或CentOS系统上可以执行命令journalctl -u docker.service。

​	每次重启Docker服务后，可以通过查看Docker信息（docker info命令），确保服务已经正常运行。

# 3. 使用Docker镜像

# 3.1 获取镜像

​	可以使用docker [image] pull命令直接从Docker Hub镜像源来下载镜像。该命令的格式为docker [image] pull NAME[:TAG]。其中，NAME是镜像仓库名称（用来区分镜像）, TAG是镜像的标签（往往用来表示版本信息）。通常情况下，描述一个镜像需要包括“名称+标签”信息。

```sh
$ docker pull ubuntu:18.0418.04: Pulling from library/ubuntu
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
docker [images] inspect mysql:latest[    {        "Id": "sha256:0716d6ebcc1a61c5a296fcb187e71f93531e510d4e4400267e2e502103d0194c",        "RepoTags": [            "mysql:latest"        ],        "RepoDigests": [            "mysql@sha256:99e0989e7e3797cfbdb8d51a19d32c8d286dd8862794d01a547651a896bcf00c"        ],        "Parent": "",        "Comment": "",        "Created": "2021-09-03T07:24:57.3195538Z",...]
```

​	上面代码返回的是一个JSON格式的消息，如果我们只要其中一项内容时，可以使用-f来指定，例如，获取镜像的Architecture：

```
docker [images] inspect -f {{".Architecture"}} mysql:latest
```

**4．使用history命令查看镜像历史**

​	既然镜像文件由多个层组成，那么怎么知道各个层的内容具体是什么呢？这时候可以使用history子命令，该命令将列出各层的创建信息。

```sh
docker history mysql:latestIMAGE          CREATED      CREATED BY                                      SIZE      COMMENT0716d6ebcc1a   3 days ago   /bin/sh -c #(nop)  CMD ["mysqld"]               0B<missing>      3 days ago   /bin/sh -c #(nop)  EXPOSE 3306 33060            0B<missing>      3 days ago   /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B<missing>      3 days ago   /bin/sh -c ln -s usr/local/bin/docker-entryp…   34B<missing>      3 days ago   /bin/sh -c #(nop) COPY file:345a22fe55d3e678…   14.5kB<missing>      3 days ago   /bin/sh -c #(nop) COPY dir:2e040acc386ebd23b…   1.12kB<missing>      3 days ago   /bin/sh -c #(nop)  VOLUME [/var/lib/mysql]      0B<missing>      3 days ago   /bin/sh -c {   echo mysql-community-server m…   378MB<missing>      3 days ago   /bin/sh -c echo 'deb http://repo.mysql.com/a…   55B<missing>      3 days ago   /bin/sh -c #(nop)  ENV MYSQL_VERSION=8.0.26-…   0B<missing>      3 days ago   /bin/sh -c #(nop)  ENV MYSQL_MAJOR=8.0          0B<missing>      3 days ago   /bin/sh -c set -ex;  key='A4A9406876FCBD3C45…   1.84kB<missing>      3 days ago   /bin/sh -c apt-get update && apt-get install…   52.2MB<missing>      3 days ago   /bin/sh -c mkdir /docker-entrypoint-initdb.d    0B<missing>      3 days ago   /bin/sh -c set -eux;  savedAptMark="$(apt-ma…   4.17MB<missing>      3 days ago   /bin/sh -c #(nop)  ENV GOSU_VERSION=1.12        0B<missing>      3 days ago   /bin/sh -c apt-get update && apt-get install…   9.34MB<missing>      3 days ago   /bin/sh -c groupadd -r mysql && useradd -r -…   329kB<missing>      3 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B<missing>      3 days ago   /bin/sh -c #(nop) ADD file:4ff85d9f6aa246746…   69.3MB
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
*  -c, --change=[]：提交的时候执行Dockerfile指令，包括CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR等；
*  -m, --message=""：提交消息；
*  -p, --pause=true：提交时暂停容器运行。



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
* -c, -cpu-shares int：限制CPU使用份额；
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
* ENTRYPOINT command param1 param2:shell中执行。



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
* HEALTHCHECK NONE：禁止基础镜像中的健康检查。



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

# 9. 操作系统

## 9.1 Busybox

​	BusyBox是一个集成了一百多个最常用Linux命令（如cat、echo、grep、mount、telnet等）的精简工具箱，它只有不到2 MB大小，被誉为“Linux系统的瑞士军刀”。BusyBox可运行于多款POSIX环境的操作系统中，如Linux（包括Android）、Hurd、FreeBSD等。

**获取官方镜像并运行**

```sh
docker search busyboxdocker pull busybix:lastestdocker run -it busybox
```

**相关资源**

* BusyBox官网：https://busybox.net/
* BusyBox官方仓库：https://git.busybox.net/busybox/
* BusyBox官方镜像：https://hub.docker.com/_/busybox/
* BusyBox官方镜像仓库：https://github.com/docker-library/busybox

## 9.2 Alpine

**简介**

​	Alpine操作系统是一个面向安全的轻型Linux发行版，关注安全，性能和资源效能。不同于其他发行版，Alpine采用了musl libc和BusyBox以减小系统的体积和运行时资源消耗，比BusyBox功能上更完善。在保持瘦身的同时，Alpine还提供了包管理工具apk查询和安装软件包。

**获取并使用官方镜像**

​	由于镜像很小，下载时间几乎可以忽略，读者可以使用docker [container] run指令直接运行一个Alpine容器，并指定运行的指令，例如：

![](https://pic.imgdb.cn/item/614b35002ab3f51d91f3a804.jpg)

**迁移至Alpine基础镜像**

​	目前，大部分Docker官方镜像都已经支持Alpine作为基础镜像，可以很容易进行迁移。例如：

* ubuntu/debian -> alpine
* python:2.7-> python:3.6-alpine
* ruby:2.6-> ruby:2.6-alpine



​	另外，如果使用Alpine镜像，安装软件包时可以使用apk工具，则如：

![](https://pic.imgdb.cn/item/614b35292ab3f51d91f3ec1a.jpg)

**相关资源**

* Apline官网：http://alpinelinux.org/
* Apline官方仓库：https://github.com/alpinelinux
* Apline官方镜像：https://hub.docker.com/_/alpine/
* Apline官方镜像仓库：https://github.com/gliderlabs/docker-alpine

## 9.3 Debian/Ubuntu

​	Debian和Ubuntu都是目前较为流行的Debian系的服务器操作系统，十分适合研发场景。Docker Hub上提供了它们的官方镜像，国内各大容器云服务都提供了完整的支持。

**Debian系统简介及官方镜像使用**

```sh
docker search debiandocker run -it debian bash
```

![](https://pic.imgdb.cn/item/614b35292ab3f51d91f3ec1a.jpg)

**ubuntu**

​	略

**相关资源**

Debian的相关资源如下：

* Debian官网：https://www.debian.org/
* Debian官方镜像：https://hub.docker.com/_/debian/

Ubuntu的相关资源

* Ubuntu官网：http://www.ubuntu.org.cn/global
* Ubuntu官方镜像：https://hub.docker.com/_/ubuntu/

## 9.4 CentOS/Fedora

​	略

# 10. 为镜像添加SSH服务

​	在第一部分中介绍了一些进入容器的办法，比如用attach、exec等命令，但是这些命令都无法解决远程管理容器的问题。因此，当读者需要远程登录到容器内进行一些操作的时候，就需要SSH的支持了。

​	本章将具体介绍如何自行创建一个带有SSH服务的镜像，并详细介绍了两种创建容器的方法：基于docker commit命令创建和基于Dockerfile创建。

## 10.1 基于commit命令创建

**1．准备工作**

​	首先，获取ubuntu:18.04镜像，并创建一个容器：

```sh
docker pull ubuntu:18.04docker run -it ubuntu:18.04 bash
```

**2．配置软件源**

​	检查软件源，并使用apt-get update命令来更新软件源信息：

```sh
apt-get update
```

​	如果默认的官方源速度慢的话，也可以替换为国内163、sohu等镜像的源。以163源为例，在容器内创建/etc/apt/sources.list.d/163.list文件：

![](https://pic.imgdb.cn/item/614b37092ab3f51d91f75e1b.jpg)

**3．安装和配置SSH服务**

​	更新软件包缓存后可以安装SSH服务了，选择主流的openssh-server作为服务端。可以看到需要下载安装众多的依赖软件包：

```sh
apt-get install openssh-server
```

​	如果需要正常启动SSH服务，则目录 /var/run/sshd必须存在。下面手动创建它，并启动SSH服务：

```sh
mkdir -p /var/run/sshd/usr/bin/sshd -D &
```

![](https://pic.imgdb.cn/item/614b37522ab3f51d91f7e108.jpg)

![](https://pic.imgdb.cn/item/614b37622ab3f51d91f803f0.jpg)

**4．保存镜像**

​	将所退出的容器用docker commit命令保存为一个新的sshd:ubuntu镜像。

![](https://pic.imgdb.cn/item/614b37772ab3f51d91f82f65.jpg)

**5.使用镜像**

![](https://pic.imgdb.cn/item/614b37e82ab3f51d91f8fd41.jpg)

![](https://pic.imgdb.cn/item/614b37f52ab3f51d91f91979.jpg)

## 10.2 使用Dockerfile创建

​	在第一部分中笔者曾介绍过Dockerfile的基础知识，下面将介绍如何使用Dockerfile来创建一个支持SSH服务的镜像。

**1．创建工作目录**

首先，创建一个sshd_ubuntu工作目录：

```sh
mkdir sshd_ubuntucd sshd_ubuntu/touch Dockerfile run.sh
```

**2．编写run.sh脚本和authorized_keys文件**

![](https://pic.imgdb.cn/item/614b385e2ab3f51d91f9db0e.jpg)

**3．编写Dockerfile**

​	下面是Dockerfile的内容及各部分的注释，可以对比上一节中利用docker commit命令创建镜像过程，所进行的操作基本一致：

![](https://pic.imgdb.cn/item/614b38762ab3f51d91fa0cdb.jpg)

![](https://pic.imgdb.cn/item/614b38832ab3f51d91fa2339.jpg)

**4．创建镜像**

​	在sshd_ubuntu目录下，使用docker build命令来创建镜像。这里用户需要注意在最后还有一个“.”，表示使用当前目录中的Dockerfile：

![](https://pic.imgdb.cn/item/614b38a12ab3f51d91fa5a38.jpg)

![](https://pic.imgdb.cn/item/614b38ab2ab3f51d91fa6beb.jpg)

**5．测试镜像，运行容器**

![](https://pic.imgdb.cn/item/614b38be2ab3f51d91fa8eba.jpg)

​	在Docker社区中，对于是否需要为Docker容器启用SSH服务一直有争论。	

​	一方的观点是：Docker的理念是一个容器只运行一个服务。因此，如果每个容器都运行一个额外的SSH服务，就违背了这个理念。而且认为根本没有从远程主机进入容器进行维护的必要。

​	笔者认为，这两种说法各有道理，其实是在讨论不同的容器场景：作为应用容器，还是作为系统容器。应用容器行为围绕应用生命周期，较为简单，不需要人工的额外干预；而系统容器则需要支持管理员的登录操作，这个时候，对SSH服务的支持就变得十分必要了。

# 11.Web服务与应用

## 11.1 Apache

**1．使用DockerHub镜像**

![](https://pic.imgdb.cn/item/615186d82ab3f51d9173643c.jpg)

也可以不创建自定义镜像，直接通过映射目录方式运行Apache容器：

![](https://pic.imgdb.cn/item/615188832ab3f51d9175339c.jpg)

**2．使用自定义镜像**

​	首先，创建一个apache_ubuntu工作目录，在其中创建Dockerfile文件、run.sh文件和sample目录：

![](https://pic.imgdb.cn/item/6151890a2ab3f51d9175b19c.jpg)

![](https://pic.imgdb.cn/item/6151897c2ab3f51d91763a0b.jpg)

![](https://pic.imgdb.cn/item/615189892ab3f51d917645a4.jpg)

​	此sample站点的内容为输出Hello Docker!。下面用户在sample目录下创建index.html文件，内容为：

​	run.sh脚本内容也很简单，只是启动apache服务：

![](https://pic.imgdb.cn/item/615189b12ab3f51d91766dfb.jpg)

下面，开始创建apache:ubuntu镜像。使用docker build命令创建apache:ubuntu镜像，注意命令最后的“.”：

![](https://pic.imgdb.cn/item/615189ce2ab3f51d917688f7.jpg)

![](https://pic.imgdb.cn/item/615189d92ab3f51d917692ad.jpg)

​	下面，用户看看Dockerfile创建的镜像拥有继承的特性。不知道有没有细心的读者发现，在apache镜像的Dockerfile中只用EXPOSE定义了对外开放的80端口，而在docker ps-a命令的返回中，却看到新启动的容器映射了2个端口：22和80。

​	但是实际上，当尝试使用SSH登录到容器时，会发现无法登录。这是因为在run.sh脚本中并未启动SSH服务。这说明在使用Dockerfile创建镜像时，会继承父镜像的开放端口，但却不会继承启动命令。因此，需要在run.sh脚本中添加启动sshd的服务的命令：

![](https://pic.imgdb.cn/item/61518a092ab3f51d9176c651.jpg)

![](https://pic.imgdb.cn/item/61518a182ab3f51d9176d4ec.jpg)



**3．相关资源**

​	Apache的相关资源如下：

* Apache官网：https://httpd.apache.org/
* Apache官方仓库：https://github.com/apache/httpd

## 11.2 Nginx

Nginx特性如下：

* 热部署：采用master管理进程与worker工作进程的分离设计，支持热部署。在不间断服务的前提下，可以直接升级版本。也可以在不停止服务的情况下修改配置文件，更换日志文件等。
* 高并发连接：Nginx可以轻松支持超过100K的并发，理论上支持的并发连接上限取决于机器内存。
* 低内存消耗：在一般的情况下，10K个非活跃的HTTP Keep-Alive连接在Nginx中仅消耗2.5 MB的内存，这也是Nginx支持高并发连接的基础。
* 响应快：在正常的情况下，单次请求会得到更快的响应。在高峰期，Nginx可以比其他的Web服务器更快地响应请求。
* 高可靠性：Nginx是一个高可靠性的Web服务器，这也是用户为什么选择Nginx的基本条件，现在很多的网站都在使用Nginx，足以说明Nginx的可靠性。高可靠性来自其核心框架代码的优秀设计和实现。

**1．使用DockerHub镜像**

![](https://pic.imgdb.cn/item/61518aa62ab3f51d91776eb2.jpg)

​	1.9.8版本后的镜像支持debug模式，镜像包含nginx-debug，可以支持更丰富的log信息：

![](https://pic.imgdb.cn/item/61518ac12ab3f51d91778b87.jpg)

**2．自定义Web页面![](https://pic.imgdb.cn/item/61518ad82ab3f51d9177a273.jpg)**

![](https://pic.imgdb.cn/item/61518b2c2ab3f51d9177f667.jpg)

**（1）使用自定义Dockerfile**

略

**3．参数优化**

​	为了能充分发挥Nginx的性能，用户可对系统内核参数做一些调整。下面是一份常见的适合运行Nginx服务器的内核优化参数：

![](https://pic.imgdb.cn/item/61518ba12ab3f51d91786154.jpg)

![](https://pic.imgdb.cn/item/61518bab2ab3f51d91786b2a.jpg)

## 11.3 Tomcat

​	略。

## 11.4 Jetty

​	略

## 11.5 LAMP

略

## 11.6 持续开发与管理

​	持续集成的特点包括：

* 鼓励自动化的周期性的过程，从检出代码、编译构建、运行测试、结果记录、测试统计等都是自动完成的，减少人工干预；
* 需要有持续集成系统的支持，包括代码托管机制支持，以及集成服务器等。

**1. Jenkins及官方镜像**

​	Jenkis自2.0版本推出了“Pipeline as Code”，帮助Jenkins实现对CI和CD更好的支持。通过Pipeline，将原本独立运行的多个任务连接起来，可以实现十分复杂的发布流程，如图11-9所示。

![](https://pic.imgdb.cn/item/61518fba2ab3f51d917ce7ed.jpg)

​	Jenkins官方在DockerHub上提供了全功能的基于官方发布版的Docker镜像。可以方便地使用docker [container] run指令一键部署Jenkins服务：

![](https://pic.imgdb.cn/item/61518fc82ab3f51d917cf6d3.jpg)

​	目前运行的容器中，数据会存储在工作目录 /var/jenkins_home中，这包括Jenkins中所有的数据，如插件和配置信息等。如果需要数据持久化，读者可以使用数据卷机制：

![](https://pic.imgdb.cn/item/615190022ab3f51d917d38c7.jpg)

**2.GitLab及其官方镜像**

​	GitLab官方提供了社区版本（GitLab CE）的DockerHub镜像，可以直接使用docker run指令运行：

![](https://pic.imgdb.cn/item/615190202ab3f51d917d5bc6.jpg)

# 12. 数据库应用

略

# 13. 分布式处理与大数据平台

略

# 14. 编程开发

略

# 15. 容器与云服务

略。

# 16. 容器实战思考

## 16.1 Docker为什么会成功

​	Docker提供了一种统一的实践方法，每个服务（或应用）维护一个Dockerfile文件。即便使用编排工具如Docker Compose，一个服务（或应用）也只需维护一个docker-compose. yml文件。应用程序及其运行时环境全部打包到一个简单易读的Dockerfile或Compose文件中，开发团队和运维团队都可以透明地合作维护这个文件，极大地降低了沟通成本与部署成本，满足了研发团队与DevOps团队、运维团队之间的沟通需求，清晰划分了责任边界。

## 16.2 研发人员该如何看待容器

**1．快速上手新技术**

​		通过Docker的使用，用户可以将精力和注意力都尽快地放在语言本身的学习上，而无须折腾系统环境的各种配置。Docker官网的口号就包含了以上含义：Build,Ship and Run Any App, Anywhere，即“任何应用都可以自动构建、发布、运行于任何环境”，将环境的影响因素降至最低，全面掌控应用整个生命周期。

**2．容器化的代码仓库提升开发速度**

​	在技术团队中，为何行业新人和资深工程师之间的生产力可以有几十倍的差距呢？暂且不论基础技能和经验的差距，同样是做一件任务，新人首先面对的就是工具的选择，然后需要解决工程实践中的各种“坑”。而资深工程师接手后，可以快速规划所需要的资源，并在最短时间内利用积累的模块搭建起系统，从而可以快速完成任务。

**3．面向业务编程**

​	笔者根据Docker的特性，给出一个可行方案：使用Docker快速掌握新技术要点并完成适当的技术储备。下面，举一个简单的例子，假定读者是Python技术栈的后端工程师，熟悉常规网站的后台建设，那么如何快速实现移动应用的Restful APISever呢？可以去Docker Hub搜索适合做API服务器的Python快速开发框架，根据自身业务需求修改Dockerfile，定制符合要求的镜像，然后快速启动一套能满足相关API的系统。

**4．使用Docker Hub发布开源项目**

​	技术人员从社区借鉴和学习各种好用的工具和技能时，也需要积极反馈社区，共同营造一个良好的生态环境。

## 16.3 容器化开发模式

​	传统开发模式会涉及多种环境和团队。开发团队在开发环境中完成软件开发，本地完成单元测试，测试通过，则可提交到代码版本管理库；测试团队打包进行进一步测试。运维团队把应用部署到测试环境，开发团队或测试团队再次进行测试，通过后通知部署人员发布到生产环境。

​	在上述过程中涉及的三个环境（开发、测试和生产）以及三个团队（开发、测试、运维），彼此之间需要进行大量人工交互，很容易出现由于环境不一致而导致出错的情况，浪费不必要的人力物力。

​	在容器化开发模式中，应用是以容器的形式存在，所有和该应用相关的依赖都会在容器中，因此移植非常方便，避免了因为环境不一致而出错的风险。

![](https://pic.imgdb.cn/item/6151949a2ab3f51d91831c35.jpg)

**1．操作流程**

​	在容器化的应用中，项目架构师和开发人员的作用贯穿整个开发、测试、生产三个环节。

​	项目伊始，架构师根据项目预期创建好基础的base镜像，如Nginx、Tomcat、MySQL镜像，或者将Dockerfile分发给所有开发人员。开发人员根据Dockerfile创建的容器或者从内部仓库下载的镜像来进行开发，达到开发环境的充分一致。若开发过程中需要添加新的软件，只需要向架构师申请修改基础的base镜像的Dockerfile即可。

​	开发任务结束后，架构师调整Dockerfile或者Docker镜像，然后分发给测试部门，测试部门马上就可以进行测试，消除了部署困难等难缠的问题。

**2．场景示例**

![](https://pic.imgdb.cn/item/615194e82ab3f51d91837a0b.jpg)

![](https://pic.imgdb.cn/item/615194f42ab3f51d91838537.jpg)

## 16.4 容器与生产环境

​	对于生产环境，不同的产品技术团队可能有不同的解读。在这里，生产环境是指企业运行其商业应用的IT环境，是相对于开发环境、预发布环境和测试环境而言的。

​	在生产环境中，容器既可以作为API后端服务器，也可以作为业务应用的Web服务器，还可以作为微服务中的服务节点。但是不管用户将容器用于哪种场景，在生产环境中运行容器与其他环境相比，对安全性与稳定性等方面都有更高的要求。

​	Docker算是IT生产环境与基础设施的新成员。近些年，Docker在DevOps和基础设施领域中快速风靡起来。Google、IBM、Amazon、Microsoft，以及几乎所有云计算供应商都宣布支持Docker。很多容器领域中的创业公司都在2014年或2015年初获得了风险投资。同时，Docker公司在2015年的估值也达到了10亿美元。

​	尽管Docker获得广大公有云厂商的大力支持，但是目前容器技术生态中已经存在许多分支与分歧，如rkt项目。为了解决容器生态中的差异化问题，为了从根本上解决生产环境中运用Docker的风险，Google、Intel、Microsoft、IBM、Amazon、VMware、Oracle、HPE、Facebook等IT巨头于2015年6月共同宣布成立OCI（Open Container Initiative）组织。[插图]OCI组织的目标在于建立通用的容器技术标准。除了保障与延续既有容器服务的生命周期外，还通过不断推出标准的、创新的容器解决方案赋能开发者。而OCI成员企业也会秉持开放、安全、弹性等核心价值观来发展容器生态。客观而言，OCI组织的出现确立了容器技术的标准，避免容器技术被单一厂商垄断。统一技术标准后，广大企业不用担心未来新兴的容器技术不兼容Docker。

​	现在越来越多的企业正在生产环境中使用Docker。2016年，DockerHub镜像下载量超过100亿次。最近某容器服务的研究显示，八成的IT从业者了解和接触过Docker，四成的组织目前正在生产环境中使用Docker，预计这个比例会在未来两年内还会继续上升。

在生产环境中使用容器，这里提供一些基本建议供大家参考：

* 如果Docker出现不可控的风险，是否考虑了备选的解决方案；
* 是否需要对Docker容器做资源限制，以及如何限制，如CPU、内存、网络、磁盘等；
* 目前，Docker对容器的安全管理做得不够完善，在应用到生产环境之前可以使用第三方工具来加强容器的安全管理，如使用apparmor对容器的能力进行限制，使用更加严格的iptable规则，禁止root用户登录，限制普通用户权限以及做好系统日志的记录；
* 公司内部私有仓库的管理、镜像的管理问题是否解决。目前官方提供的私有仓库管理工具功能并不十分完善，若在生产环境中使用还需要更多的完善措施。

# 17. 核心实现技术

## 17.1 基本架构

​	Docker目前采用了标准的C/S架构，包括客户端、服务端两大核心组件，同时通过镜像仓库来存储镜像。客户端和服务端既可以运行在一个机器上，也可通过socket或者RESTful API来进行通信，如图17-1所示。

![](https://pic.imgdb.cn/item/615af5662ab3f51d91fd7db8.jpg)

图17-1 Docker基本架构

**1．服务端**

​	Docker服务端一般在宿主主机后台运行，dockerd作为服务端接受来自客户的请求，并通过containerd具体处理与容器相关的请求，包括创建、运行、删除容器等。服务端主要包括四个组件：

* dockerd：为客户端提供RESTful API，响应来自客户端的请求，采用模块化的架构，通过专门的Engine模块来分发管理各个来自客户端的任务。可以单独升级；

* docker-proxy：是dockerd的子进程，当需要进行容器端口映射时，docker-proxy完成网络映射配置；
* containerd：是dockerd的子进程，提供gRPC接口响应来自dockerd的请求，对下管理runC镜像和容器环境。可以单独升级；
* containerd-shim：是containerd的子进程，为runC容器提供支持，同时作为容器内进程的根进程。



​	runC是从Docker公司开源的libcontainer项目演化而来的，目前作为一种具体的开放容器标准实现加入Open Containers Initiative（OCI）。runC已经支持了Linux系统中容器相关技术栈，同时正在实现对其他操作系统的兼容。用户也可以通过使用docker-runc命令来直接使用OCI规范的容器。dockerd默认监听本地的unix:///var/run/docker.sock套接字，只允许本地的root用户或docker用户组成员访问。可以通过-H选项来修改监听的方式。例如，让dockerd监听本地的TCP连接1234端口，代码如下：

![](https://pic.imgdb.cn/item/615af64c2ab3f51d91feced5.jpg)

​	此外，Docker还支持通过TLS认证方式来验证访问。docker-proxy只有当启动容器并且使用端口映射时候才会执行，负责配置容器的端口映射规则：

![](https://pic.imgdb.cn/item/615af7772ab3f51d91007e58.jpg)

**2．客户端**

​	Docker客户端为用户提供一系列可执行命令，使用这些命令可实现与Docker服务端交互。

​	用户使用的Docker可执行命令即为客户端程序。与Docker服务端保持运行方式不同，客户端发送命令后，等待服务端返回；一旦收到返回后，客户端立刻执行结束并退出。用户执行新的命令，需要再次调用客户端命令。

​	客户端默认通过本地的unix:///var/run/docker.sock套接字向服务端发送命令。如果服务端没有监听在默认的地址，则需要客户端在执行命令的时候显式地指定服务端地址。例如，假定服务端监听在本地的TCP连接1234端口为tcp://127.0.0.1:1234，只有通过-H参数指定了正确的地址信息才能连接到服务端：

![](https://pic.imgdb.cn/item/615b13932ab3f51d912efe7f.jpg)

**3．镜像仓库**

​	镜像是使用容器的基础，Docker使用镜像仓库（Registry）在大规模场景下存储和分发Docker镜像。镜像仓库提供了对不同存储后端的支持，存放镜像文件，并且支持RESTful API，接收来自dockerd的命令，包括拉取、上传镜像等。

​	用户从镜像仓库拉取的镜像文件会存储在本地使用；用户同时也可以上传镜像到仓库，方便其他人获取。使用镜像仓库可以极大地简化镜像管理和分发的流程。镜像仓库目前作为Docker分发项目，已经开源在Github（https://github.com/docker/distribution），目前支持API版本为2.0。

## 17.2 命名空间

​	命名空间（namespace）是Linux内核的一个强大特性，为容器虚拟化的实现带来极大便利。利用这一特性，每个容器都可以拥有自己单独的命名空间，运行在其中的应用都像是在独立的操作系统环境中一样。命名空间机制保证了容器之间彼此互不影响。

​	在操作系统中，包括内核、文件系统、网络、进程号（ProcessID, PID）、用户号（User ID, UID）、进程间通信（InterProcess Communication, IPC）等资源，所有的资源都是应用进程直接共享的。要想实现虚拟化，除了要实现对内存、CPU、网络IO、硬盘IO、存储空间等的限制外，还要实现文件系统、网络、PID、UID、IPC等的相互隔离。前者相对容易实现一些，后者则需要宿主主机系统的深入支持。

​	随着Linux系统对于命名空间功能的逐步完善，现在已经可以实现这些需求，让进程在彼此隔离的命名空间中运行。虽然这些进程仍在共用同一个内核和某些运行时环境（runtime，例如一些系统命令和系统库），但是彼此是不可见的，并且认为自己是独占系统的。

​	Docker容器每次启动时候，通过调用funcsetNamespaces(daemon *Daemon, s *specs. Spec, c*container.Container) error方法来完成对各个命名空间的配置。

**1．进程命名空间**

​	Linux通过进程命名空间管理进程号，对于同一进程（同一个task_struct），在不同的命名空间中，看到的进程号不相同。每个进程命名空间有一套自己的进程号管理方法。进程命名空间是一个父子关系的结构，子空间中的进程对于父空间是可见的。新fork出的一个进程，在父命名空间和子命名空间将分别对应不同的进程号。例如，查看Docker服务主进程（dockerd）的进程号是3393，它作为父进程启动了docker-containerd进程，进程号为3398，代码如下所示：

![](https://pic.imgdb.cn/item/615b14952ab3f51d9130ba78.jpg)

​	新建一个Ubuntu容器，执行sleep命令。此时，docker-containerd进程作为父进程，会为每个容器启动一个docker-containerd-shim进程，作为该容器内所有进程的根进程：

![](https://pic.imgdb.cn/item/615b1e832ab3f51d91417945.jpg)

​	从宿主机上查看新建容器的进程的父进程，正是docker-containerd-shim进程：

![](https://pic.imgdb.cn/item/615b1f2c2ab3f51d91428828.jpg)

​	而在容器内的进程空间中，则把docker-containerd-shim进程作为0号根进程（类似宿主系统中0号根进程idle）, while进程的进程号则变为1（类似宿主系统中1号初始化进程/sbin/init）。容器内只能看到docker-containerd-shim进程往下的子进程空间，而无法获知宿主机上的进程信息：

![](https://pic.imgdb.cn/item/615b1f892ab3f51d914329eb.jpg)

![](https://pic.imgdb.cn/item/615b1f942ab3f51d91433c6a.jpg)

​	一般情况下，启动多个容器时，宿主机与容器内进程空间的关系如图17-2所示。

![](https://pic.imgdb.cn/item/615b1fa52ab3f51d914359c4.jpg)

图17-2 宿主机与容器内进程空间的关系

**2. IPC命名空间**

​	容器中的进程交互还是采用了Linux常见的进程间交互方法（Interprocess Communication, IPC），包括信号量、消息队列和共享内存等方式。PID命名空间和IPC命名空间可以组合起来一起使用，同一个IPC命名空间内的进程可以彼此可见，允许进行交互；不同空间的进程则无法交互。

**3．网络命名空间**

​	有了进程命名空间后，不同命名空间中的进程号可以相互隔离，但是网络端口还是共享本地系统的端口。

​	通过网络命名空间，可以实现网络隔离。一个网络命名空间为进程提供了一个完全独立的网络协议栈的视图。包括网络设备接口、IPv4和IPv6协议栈、IP路由表、防火墙规则、sockets等，这样每个容器的网络就能隔离开来。

​	Docker采用虚拟网络设备（Virtual Network Device, VND）的方式，将不同命名空间的网络设备连接到一起。默认情况下，Docker在宿主机上创建多个虚机网桥（如默认的网桥docker0），容器中的虚拟网卡通过网桥进行连接，如图17-3所示。

![](https://pic.imgdb.cn/item/615b1fdf2ab3f51d9143b7ae.jpg)

​	使用docker network ls命令可以查看到当前系统中的网桥：

![](https://pic.imgdb.cn/item/615b1ff22ab3f51d9143d633.jpg)

​	使用brctl工具（需要安装bridge-utils工具包），还可以看到连接到网桥上的虚拟网口的信息。每个容器默认分配一个网桥上的虚拟网口，并将docker0的IP地址设置为默认的网关，容器发起的网络流量通过宿主机的iptables规则进行转发：

![](https://pic.imgdb.cn/item/615b1ffe2ab3f51d9143eb34.jpg)

**4．挂载命名空间**

​	

​	类似于chroot，挂载（Mount, MNT）命名空间可以将一个进程的根文件系统限制到一个特定的目录下。挂载命名空间允许不同命名空间的进程看到的本地文件位于宿主机中不同路径下，每个命名空间中的进程所看到的文件目录彼此是隔离的。例如，不同命名空间中的进程，都认为自己独占了一个完整的根文件系统（rootfs），但实际上，不同命名空间中的文件彼此隔离，不会造成相互影响，同时也无法影响宿主机文件系统中的其他路径。

**5. UTS命名空间**

​	UTS（UNIX Time-sharing System）命名空间允许每个容器拥有独立的主机名和域名，从而可以虚拟出一个有独立主机名和网络空间的环境，就跟网络上一台独立的主机一样。

​	如果没有手动指定主机名称，Docker容器的主机名就是返回的容器ID的前6字节前缀，否则为指定的用户名：

![](https://pic.imgdb.cn/item/615b20542ab3f51d914477f6.jpg)

**6．用户命名空间**

​	每个容器可以有不同的用户和组id，也就是说，可以在容器内使用特定的内部用户执行程序，而非本地系统上存在的用户。

​	每个容器内部都可以有最高权限的root帐号，但跟宿主主机不在一个命名空间。通过使用隔离的用户命名空间，可以提高安全性，避免容器内的进程获取到额外的权限；同时通过使用不同用户也可以进一步在容器内控制权限。

​	例如，下面的命令在容器内创建了test用户，只有普通权限，无法访问更高权限的资源：

![](https://pic.imgdb.cn/item/615b20882ab3f51d9144cfc0.jpg)

## 17.3 控制组

​	控制组（CGroups）是Linux内核的一个特性，主要用来对共享资源进行隔离、限制、审计等。只有将分配到容器的资源进行控制，才能避免多个容器同时运行时对宿主机系统的资源竞争。每个控制组是一组对资源的限制，支持层级化结构。

​	控制组技术最早是由Google的程序员在2006年提出的，Linux内核自2.6.24开始原生支持，可以提供对容器的内存、CPU、磁盘IO等资源进行限制和计费管理。最初的设计目标是为不同的应用情况提供统一的接口，从控制单一进程（比如nice工具）到系统级虚拟化（包括OpenVZ, Linux-VServer, LXC等）。

​	具体来看，控制组提供如下功能：

* 资源限制（resource limiting）：可将组设置一定的内存限制。比如：内存子系统可以为进程组设定一个内存使用上限，一旦进程组使用的内存达到限额再申请内存，就会出发Out of Memory警告。
* 优先级（prioritization）：通过优先级让一些组优先得到更多的CPU等资源。
* 资源审计（accounting）：用来统计系统实际上把多少资源用到适合的目的上，可以使用cpuacct子系统记录某个进程组使用的CPU时间。
* 隔离（isolation）：为组隔离命名空间，这样使得一个组不会看到另一个组的进程、网络连接和文件系统。
* 控制（control）：执行挂起、恢复和重启动等操作。



​	Docker容器每次启动时候，通过调用func setCapabilities(s ＊specs.Spec, c ＊container.Container) error方法来完成对各个命名空间的配置。安装Docker后，用户可以在/sys/fs/cgroup/memory/docker/目录下看到对Docker组应用的各种限制项，包括全局限制和位于子目录中对于某个容器的单独限制：

![](https://pic.imgdb.cn/item/615bb9f12ab3f51d91cd6afe.jpg)

![](https://pic.imgdb.cn/item/615bba052ab3f51d91cd7efc.jpg)

​	用户可以通过修改这些文件值来控制组，从而限制Docker应用资源。例如，通过下面的命令可限制Docker组中的所有进程使用的物理内存总量不超过100 MB：

![](https://pic.imgdb.cn/item/615bbb062ab3f51d91ce814c.jpg)

​	同时，可以在创建或启动容器时为每个容器指定资源的限制，例如使用-c|--cpu-shares[=0]参数可调整容器使用CPU的权重；使用-m|--memory[=MEMORY]参数可调整容器最多使用内存的大小。

## 17.4 联合文件系统

​	联合文件系统（UnionFS）是一种轻量级的高性能分层文件系统，它支持将文件系统中的修改信息作为一次提交，并层层叠加，同时可以将不同目录挂载到同一个虚拟文件系统下，应用看到的是挂载的最终结果。联合文件系统是实现Docker镜像的技术基础。

​	Docker镜像可以通过分层来进行继承。例如，用户基于基础镜像（用来生成其他镜像的基础，往往没有父镜像）来制作各种不同的应用镜像。这些镜像共享同一个基础镜像层，提高了存储效率。此外，当用户改变了一个Docker镜像（比如升级程序到新的版本），则会创建一个新的层（layer）。因此，用户不用替换整个原镜像或者重新建立，只需要添加新层即可。用户分发镜像的时候，也只需要分发被改动的新层内容（增量部分）。这让Docker的镜像管理变得十分轻量和快速。

**1. Docker存储原理**

​	Docker目前通过插件化方式支持多种文件系统后端。Debian/Ubuntu上成熟的AUFS（Another Union FileSystem，或v2版本往后的Advanced multi layeredUnification File System），就是一种联合文件系统实现。AUFS支持为每一个成员目录（类似Git的分支）设定只读（readonly）、读写（readwrite）或写出（whiteout-able）权限，同时AUFS里有一个类似分层的概念，对只读权限的分支可以逻辑上进行增量地修改（不影响只读部分的）。

​	Docker镜像自身就是由多个文件层组成，每一层有基于内容的唯一的编号（层ID）。可以通过docker history查看一个镜像由哪些层组成。例如查看ubuntu:16.04镜像由6层组成，每层执行了不同的命令，如下所示：

![](https://pic.imgdb.cn/item/615bbc712ab3f51d91d00491.jpg)

​	对于Docker镜像来说，这些层的内容都是不可修改的、只读的。而当Docker利用镜像启动一个容器时，将在镜像文件系统的最顶端再挂载一个新的可读写的层给容器。容器中的内容更新将会发生在可读写层。当所操作对象位于较深的某层时，需要先复制到最上层的可读写层。当数据对象较大时，往往意味着较差的IO性能。因此，对于IO敏感型应用，一般推荐将容器修改的数据通过volume方式挂载，而不是直接修改镜像内数据。

​	另外，对于频繁启停Docker容器的场景下，文件系统的IO性能也将十分关键。

**2. Docker存储结构**

​	所有的镜像和容器都存储都在Docker指定的存储目录下，以Ubuntu宿主系统为例，默认路径是/var/lib/docker。在这个目录下面，存储由Docker镜像和容器运行相关的文件和目录，可能包括builder、containerd、containers、image、network、aufs/overlay2、plugins、runtimes、swarm、tmp、trust、volumes等。

​	其中，如果使用AUFS存储后端，则最关键的就是aufs目录，保存Docker镜像和容器相关数据和信息。包括layers、diff和mnt三个子目录。1.9版本和之前的版本中，命名跟镜像层的ID是匹配的；而自1.10开始，层数据相关的文件和目录名与层ID不再匹配。

​	layers子目录包含层属性文件，用来保存各个镜像层的元数据：某镜像的某层下面包括哪些层。例如：某镜像由5层组成，则文件内容应该如下：

![](https://pic.imgdb.cn/item/615bbcd52ab3f51d91d07193.jpg)

​		mnt子目录下面的子目录是各个容器最终的挂载点，所有相关的AUFS层在这里挂载到一起，形成最终效果。一个运行中容器的根文件系统就挂载在这下面的子目录上。同样，1.10版本之前的Docker中，子目录名和容器ID是一致的。其中，还包括容器的元数据、配置文件和运行日志等。

**3．多种文件系统比较**

​	Docker目前支持的联合文件系统种类包括AUFS、btrfs、Device Mapper、overlay、over-lay2、vfs、zfs等。多种文件系统目前的支持情况总结如下：

*  AUFS：最早支持的文件系统，对Debian/Ubuntu支持好，虽然没有合并到Linux内核中，但成熟度很高；
*  btrfs：参考zfs等特性设计的文件系统，由Linux社区开发，试图未来取代Device Mapper，成熟度有待提高；
*  Device Mapper:RedHat公司和Docker团队一起开发用于支持RHEL的文件系统，内核支持，性能略慢，成熟度高；
*  overlay：类似于AUFS的层次化文件系统，性能更好，从Linux 3.18开始已经合并到内核，但成熟度有待提高；
*  overlay 2:Docker 1.12后推出，原生支持128层，效率比OverlayFS高，较新版本的Docker支持，要求内核大于4.0；
*  vfs：基于普通文件系统（ext、nfs等）的中间层抽象，性能差，比较占用空间，成熟度也一般。
*  zfs：最初设计为Solaris 10上的写时文件系统，拥有不少好的特性，但对Linux支持还不够成熟。



​	目前，AUFS应用最为广泛，支持也相对成熟，推荐生产环境考虑。对于比较新的内核，可以尝试overlay2，作为Docker最新推荐使用的文件系统，将具有更多的特性和潜力。

## 17.5 Linux网络虚拟化

​	Docker的本地网络实现其实就是利用了Linux上的网络命名空间和虚拟网络设备（特别是veth pair）。熟悉这两部分的基本概念有助于理解Docker网络的实现过程。

**1．基本原理**

​	直观上看，要实现网络通信，机器需要至少一个网络接口（物理接口或虚拟接口）与外界相通，并可以收发数据包；此外，如果不同子网之间要进行通信，还需要额外的路由机制。

​	Docker中的网络接口默认都是虚拟接口。虚拟接口的最大优势就是转发效率极高。这是因为Linux通过在内核中进行数据复制来实现虚拟接口之间的数据转发，即发送接口的发送缓存中的数据包将被直接复制到接收接口的接收缓存中，而无须通过外部物理网络设备进行交换。对于本地系统和容器内系统来看，虚拟接口跟一个正常的以太网卡相比并无区别，只是它的速度要快得多。

​	Docker容器网络就很好地利用了Linux虚拟网络技术，它在本地主机和容器内分别创建一个虚拟接口veth，并连通（这样的一对虚拟接口叫做veth pair），如图17-4所示。

![](https://pic.imgdb.cn/item/615bbe7f2ab3f51d91d23337.jpg)

**2．网络创建过程**

​	一般情况下，Docker创建一个容器的时候，会具体执行如下操作：

1）创建一对虚拟接口，分别放到本地主机和新容器的命名空间中；

2）本地主机一端的虚拟接口连接到默认的docker0网桥或指定网桥上，并具有一个以veth开头的唯一名字，如veth1234；

3）器一端的虚拟接口将放到新创建的容器中，并修改名字作为eth0。这个接口只在容器的命名空间可见；

4）从网桥可用地址段中获取一个空闲地址分配给容器的eth0（例如172.17.0.2/16），并配置默认路由网关为docker0网卡的内部接口docker0的IP地址（例如172.17.42.1/16）。

​	完成这些之后，容器就可以使用它所能看到的eth0虚拟网卡来连接其他容器和访问外部网络。

​	用户也可以通过docker network命令来手动管理网络，这将在后续章节中介绍。

​	在使用docker [container] run命令启动容器的时候，可以通过--net参数来指定容器的网络配置。有5个可选值bridge、none、container、host和用户定义的网络：

*  --net=bridge：默认值，在Docker网桥docker0上为容器创建新的网络栈；
*  --net=none：让Docker将新容器放到隔离的网络栈中，但是不进行网络配置。之后，用户可以自行配置；
*  --net=container:NAME_or_ID：让Docker将新建容器的进程放到一个已存在容器的网络栈中，新容器进程有自己的文件系统、进程列表和资源限制，但会和已存在的容器共享IP地址和端口等网络资源，两者进程可以直接通过lo环回接口通信；
*  --net=host：告诉Docker不要将容器网络放到隔离的命名空间中，即不要容器化容器内的网络。此时容器使用本地主机的网络，它拥有完全的本地主机接口访问权限。容器进程跟主机其他root进程一样可以打开低范围的端口，可以访问本地网络服务（比如D-bus），还可以让容器做一些影响整个主机系统的事情，比如重启主机。因此使用这个选项的时候要非常小心。如果进一步使用--privileged=true参数，容器甚至会被允许直接配置主机的网络栈；
*  --net=user_defined_network：用户自行用network相关命令创建一个网络，之后将容器连接到指定的已创建网络上去。

**3．手动配置网络**

​	用户使用--net=none后，Docker将不对容器网络进行配置。下面，介绍手动完成配置网络的整个过程。通过这个过程，可以了解到Docker配置网络的更多细节。

​	首先，启动一个ubuntu:16.04容器，指定--net=none参数：

![](https://pic.imgdb.cn/item/615bbf1d2ab3f51d91d2dedd.jpg)

![](https://pic.imgdb.cn/item/615bbf2c2ab3f51d91d2f161.jpg)

​	当容器终止后，Docker会清空容器，容器内的网络接口会随网络命名空间一起被清除，A接口也被自动从docker0卸载并清除。此外，在删除/var/run/netns/下的内容之前，用户可以使用ip netns exec命令在指定网络命名空间中进行配置，从而更新容器内的网络配置。

# 18. 配置私有仓库

## 18.1 安装Docker Registry

​	新版本的Registry基于Golang进行了重构，提供更好的性能和扩展性，并且支持Docker 1.6+的API，非常适合用来构建私有的镜像注册服务器。官方仓库中也提供了Registry的镜像，因此用户可以通过容器运行和源码安装两种方式来使用Registry。

**1．基于容器安装运行**

​	基于容器的运行方式十分简单，只需要一条命令：

![](https://pic.imgdb.cn/item/615bbfef2ab3f51d91d3ccc7.jpg)

​	启动后，服务监听在本地的5000端口，可以通过访问http://localhost:5000/v2/测试启动成功。

​	Registry比较关键的参数是配置文件和仓库存储路径。默认的配置文件为/etc/docker/registry/config.yml，因此，通过如下命令，可以指定使用本地主机上的配置文件（如/home/user/registry-conf）：

![](https://pic.imgdb.cn/item/615bc0032ab3f51d91d3e4f4.jpg)

​	默认的存储位置为/var/lib/registry，可以通过-v参数来映射本地的路径到容器内。

​	例如，下面将镜像存储到本地/opt/data/registry目录：

![](https://pic.imgdb.cn/item/615bc0142ab3f51d91d3f9a0.jpg)

**2．本地安装运行**

​	有时候需要本地运行仓库服务，可以通过源码方式进行安装。

​	首先安装Golang环境支持，以Ubuntu为例，可以执行如下命令：

![](https://pic.imgdb.cn/item/615bc0292ab3f51d91d410aa.jpg)

​	确认Golang环境安装成功，并配置$GOPATH环境变量，例如/go。

​	创建$GOPATH/src/github.com/docker/目录，并获取源码：

![](https://pic.imgdb.cn/item/615bc0452ab3f51d91d42dd0.jpg)

​	将自带的模板配置文件复制到/etc/docker/registry/路径下，创建存储目录/var/lib/registry：

![](https://pic.imgdb.cn/item/615bc05f2ab3f51d91d44b64.jpg)

​	编译成功后，可以通过下面的命令来启动：

![](https://pic.imgdb.cn/item/615bc0712ab3f51d91d45ce2.jpg)

## 18.2 配置TLS证书

​	当本地主机运行Registry服务后，所有能访问到该主机的Docker Host都可以把它作为私有仓库使用，只需要在镜像名称前面添加上具体的服务器地址即可。

​	例如将本地的ubuntu:latest镜像上传到私有仓库myrepo.com：

![](https://pic.imgdb.cn/item/615bc0902ab3f51d91d47dbb.jpg)

​	私有仓库需要启用TLS认证，否则会报错。在第一部分中，我们介绍了通过添加DOCKER_OPTS="--insecure-registrymyrepo.com:5000来避免这个问题。在这里将介绍如何获取和生成TLS证书。

**1．自行生成证书**

​	![](https://pic.imgdb.cn/item/615bc0a42ab3f51d91d49401.jpg)

**2．从代理商申请证书**

​	如果Registry服务需要对外公开，需要申请大家都认可的证书。知名的代理商包括SSLs.com、GoDaddy.com、LetsEncrypt.org、GlobalSign.com等，用户可以自行选择权威的证书提供商。

**3．启用证书**

​	当拥有秘钥文件和证书文件后，可以配置Registry启用证书支持，主要通过使用REGI-STRY_HTTP_TLS_CERTIFICATE和REGISTRY_HTTP_TLS_KEY参数：

![](https://pic.imgdb.cn/item/615bc0bf2ab3f51d91d4b0cd.jpg)

## 18.3 管理访问权限

​	![](https://pic.imgdb.cn/item/615bc0d82ab3f51d91d4cb61.jpg)

**1. Docker Registry v2的认证模式**

​	Docker Registry v2的认证模式和v1有了较大的变化，降低了系统的复杂度、减少了服务之间的交互次数，其基本工作模式如图18-1所示。

具体交互过程包括如下步骤：

1）Docker Daemon或者其他客户端尝试访问Registry服务器，比如pull、push或者访问manifiest文件；

2）在Registry服务器开启了认证服务模式时，就会直接返回401 Unauthorized错误，并通知调用方如何获得授权；

3）调用方按照要求，向Authorization Service发送请求，并携带Authorization Service需要的信息，比如用户名、密码；

4）如果授权成功，则可以拿到合法的Bearer token，来标识该请求方可以获得的权限；

5）请求方将拿到Bearer token加到请求的Authorizationheader中，再次尝试步骤1中的请求；

6）Registry服务通过验证Bearer token以及JWT格式的授权数据，来决定用户是否有权限进行请求的操作。

当启用认证服务时，需要注意以下两个地方：

*  对于Authentication Service, Docker官方目前并没有放出对应的实现方案，需要自行实现对应的服务接口；
*  Registry服务和Authentication服务之间通过证书进行Bearer token的生成和认证，所以要保证两个服务之间证书的匹配。



​	除了使用第三方实现的认证服务（如docker_auth、SUSEPortus等）外，还可以通过Nginx代理方式来配置基于用户名和密码的认证。

**2．配置Nginx代理**

​	使用Nginx来代理registry服务的原理十分简单，在上一节中，我们让Registry服务监听在127.0.0.1:5000，这意味着只允许本机才能通过5000端口访问到，其他主机是无法访问到的。为了让其他主机访问到，可以通过Nginx监听在对外地址的15000端口，当外部访问请求到达15000端口时，内部再将请求转发到本地的5000端口。具体操作如下。

​	首先，安装Nginx：

![](https://pic.imgdb.cn/item/615bc1432ab3f51d91d53904.jpg)

![](https://pic.imgdb.cn/item/615bc1582ab3f51d91d55056.jpg)

![](https://pic.imgdb.cn/item/615bc16d2ab3f51d91d56808.jpg)

![](https://pic.imgdb.cn/item/615bc17c2ab3f51d91d576f1.jpg)

​	建立配置文件软连接，放到/etc/nginx/sites-enabled/下面，让Nginx启用它，最后重启Nginx服务：

![](https://pic.imgdb.cn/item/615bc18a2ab3f51d91d585f7.jpg)

![](https://pic.imgdb.cn/item/615bc1952ab3f51d91d590ce.jpg)

**3．添加用户认证**

​	公共仓库DockerHub是通过注册索引（index）服务来实现的。由于index服务并没有完善的开源实现，在这里介绍基于Nginx代理的用户访问管理方案。Nginx支持基于用户名和密码的访问管理。

​	首先，在配置文件的location / 字段中添加两行：

![](https://pic.imgdb.cn/item/615bc1ab2ab3f51d91d5a836.jpg)

​	其中，auth_basic行说明启用认证服务，不通过的请求将无法转发。auth_basic_user_file docker-registry-htpasswd；行指定了验证的用户名和密码存储文件为本地（/etc/nginx/下）的docker-registry-htpasswd文件。

​	docker-registry-htpasswd文件中存储用户名和密码的格式为每行放一个用户名、密码对。例如：

![](https://pic.imgdb.cn/item/615bc1c32ab3f51d91d5c060.jpg)

​	创建用户user1，并添加密码。

​	例如，如下的操作会创建/etc/nginx/docker-registry-htpasswd文件来保存用户名和加密后的密码信息，并创建user1和对应密码：

![](https://pic.imgdb.cn/item/615bc1d72ab3f51d91d5d3d8.jpg)

​	添加更多用户，可以重复上面的命令（密码文件存在后，不需要再使用-c选项来新创建）。

​	最后，重新启动Nginx服务：

![](https://pic.imgdb.cn/item/615bc1e72ab3f51d91d5e46f.jpg)

​	通过命令行访问，需要在地址前面带上用户名和密码才能正常返回：

![](https://pic.imgdb.cn/item/615bc1f72ab3f51d91d5f5b0.jpg)

**4．用Compose启动Registry**

​	一般情况下，用户使用Registry需要的配置包括存储路径、TLS证书和用户认证。这里提供一个基于Docker Compose的快速启动Registry的模板：

![](https://pic.imgdb.cn/item/615bc20c2ab3f51d91d60dae.jpg)

## 18.4 配置Registry

​	Docker Registry利用提供了一些样例配置，用户可以直接使用它们进行开发或生产部署。

**1．示例配置**

![](https://pic.imgdb.cn/item/615bc2a92ab3f51d91d6b865.jpg)

![](https://pic.imgdb.cn/item/615bc2ba2ab3f51d91d6ca13.jpg)

![](https://pic.imgdb.cn/item/615bc2c62ab3f51d91d6d5fe.jpg)

![](https://pic.imgdb.cn/item/615bc2cf2ab3f51d91d6e08f.jpg)

**2．选项**

​	这些选项以yaml文件格式提供，用户可以直接进行修改，也可以添加自定义的模板段。默认情况下变量可以从环境变量中读取，例如log.level:debug可以配置为：

![](https://pic.imgdb.cn/item/615bc2e12ab3f51d91d6f2c4.jpg)

​	比较重要的选项包括版本信息、log选项、hooks选项、存储选项、认证选项、HTTP选项、通知选项、redis选项、健康监控选项、代理选项和验证选项等。下面分别介绍。

![](https://pic.imgdb.cn/item/615bc2ef2ab3f51d91d70283.jpg)

其中：

* level：字符串类型，标注输出调试信息的级别，包括debug、info、warn、error；
* fomatter：字符串类型，日志输出的格式，包括text、json、logstash等；
* fields：增加到日志输出消息中的键值对，可以用于过滤日志。

**（3）hooks选项**

​	配置当仓库发生异常时，通过邮件发送日志时的参数，代码如下：

![](https://pic.imgdb.cn/item/615bc31e2ab3f51d91d73b3f.jpg)

**（4）存储选项**

​	storage选项将配置存储的引擎，默认支持包括本地文件系统、Google云存储、AWS S3云存储和OpenStack Swift分布式存储等，代码如下：

![](https://pic.imgdb.cn/item/615bc3352ab3f51d91d75575.jpg)

![](https://pic.imgdb.cn/item/615bc3462ab3f51d91d7672f.jpg)

![](https://pic.imgdb.cn/item/615bc3542ab3f51d91d7775b.jpg)

![](https://pic.imgdb.cn/item/615bc35f2ab3f51d91d7844d.jpg)

比较重要的选项如下：

* maintenance：配置维护相关的功能，包括对孤立旧文件的清理、开启只读模式等；
* delete：是否允许删除镜像功能，默认关闭；
* cache：开启对镜像层元数据的缓存功能，默认开启；

**（5）认证选项**

![](https://pic.imgdb.cn/item/615bc3822ab3f51d91d7abb1.jpg)

**其中：**

* silly：仅供测试使用，只要请求头带有认证域即可，不做内容检查；
* token：基于token的用户认证，适用于生产环境，需要额外的token服务来支持；
* htpasswd：基于Apache htpasswd密码文件的权限检查。



**（6）HTTP选项**

![](https://pic.imgdb.cn/item/615bc3a52ab3f51d91d7d2d7.jpg)

![](https://pic.imgdb.cn/item/615bc3af2ab3f51d91d7de74.jpg)

**其中：**

* addr：必选，服务监听地址；
* secret：必选，与安全相关的随机字符串，用户可以自己定义；
* tls：证书相关的文件路径信息；
* http2：是否开启http2支持，默认关闭。

**（7）通知选项**

![](https://pic.imgdb.cn/item/615bc3d82ab3f51d91d80b89.jpg)

**（8）redis选项**

​	Registry可以用Redis来缓存文件块，这里可以配置相关选项：

![](https://pic.imgdb.cn/item/615bc3ea2ab3f51d91d81f8e.jpg)

**（9）健康监控选项**

​	与健康监控相关，主要是对配置服务进行检测判断系统状态，代码如下：

![](https://pic.imgdb.cn/item/615bc4042ab3f51d91d84892.jpg)

![](https://pic.imgdb.cn/item/615bc4152ab3f51d91d85ac1.jpg)

默认并未启用。

（10）代理选项

​	配置Registry作为一个pull代理，从远端（目前仅支持官方仓库）下拉Docker镜像，代码如下：

![](https://pic.imgdb.cn/item/615bc42b2ab3f51d91d8731d.jpg)

**（11）验证选项**

![](https://pic.imgdb.cn/item/615bc4392ab3f51d91d882d9.jpg)

## 18.5 批量管理镜像

​	在之前章节中，笔者介绍了如何对单个镜像进行上传、下载的操作。有时候，本地镜像很多，逐个打标记进行操作将十分浪费时间。这里将以批量上传镜像为例，介绍如何利用脚本实现对镜像的批量化处理。

**1．批量上传指定镜像**

​	可以使用下面的push_images.sh脚本，批量上传本地的镜像到注册服务器中，默认是本地注册服务器127.0.0.1:5000，用户可以通过修改registry=127.0.0.1:5000这行来指定目标注册服务器：

![](https://pic.imgdb.cn/item/615bc45c2ab3f51d91d8a95c.jpg)

![](https://pic.imgdb.cn/item/615bc4672ab3f51d91d8b826.jpg)

![](https://pic.imgdb.cn/item/615bc4742ab3f51d91d8c822.jpg)

​	建议把脚本存放到本地可执行路径下，例如放在/usr/local/bin/下面。然后添加可执行权限，就可以使用该脚本了：

![](https://pic.imgdb.cn/item/615bc4832ab3f51d91d8d9ac.jpg)

![](https://pic.imgdb.cn/item/615bc4972ab3f51d91d8f172.jpg)

​	上传后，查看本地镜像，会发现上传中创建的临时标签也同时被清理了。

**2．上传本地所有镜像**

​	在push_images工具的基础上，还可以进一步地创建push_all工具，来上传本地所有镜像：

![](https://pic.imgdb.cn/item/615bc4b72ab3f51d91d91b44.jpg)

​	另外，推荐读者把它放在/usr/local/bin/下面，并添加可执行权限。这样就可以通过push_all命令来同步本地所有镜像到本地私有仓库了。

​	同样的，读者可以试着修改脚本，实现批量化下载镜像、删除镜像、更新镜像标签等更多的操作。

## 18.6 使用通知系统

​	Docker Registry v2还内置提供了Notification功能，提供了非常方便、快捷地集成接口，避免了v1中需要用户自己实现的麻烦。

​	Notification功能其实就是Registry在有事件发生的时候，向用户自己定义的地址发送webhook通知。目前的事件包括镜像manifest的push、pull，镜像层的push、pull。这些动作会被序列化成webhook事件的payload，为集成服务提供事件详情，并通过Registry v2的内置广播系统发送到用户定义的服务接口，Registry v2将这些用户服务接口称为Endpoints。

​	Registry服务器的事件会通过HTTP协议发送到用户定义的所有Endpoints上，而且每个Registry实例的每个Endpoint都有自己独立的队列、重试选项以及HTTP的目的地址。当一个动作发生时，会被转换成对应的事件并放置到一个内存队列中。镜像服务器会依次处理队列中的事件，并向用户定义的Endpoint发送请求。事件发送处理是串行的，但是Registry服务器并不会保证其到达顺序。

**1．相关配置**

![](https://pic.imgdb.cn/item/615bc4ef2ab3f51d91d9584b.jpg)

​	上面的配置会在pull或者push发生时向http://cd-service-host/api/v1/cd-service发送事件，并在HTTP请求的header中传入认证信息，可以是Basic、token、Bearer等模式，主要用于接收事件方进行身份认证。更新配置后，需要重启Registry服务器，如果配置正确，会在日志中看到对应的提示信息，比如：

![](https://pic.imgdb.cn/item/615bc4ff2ab3f51d91d96df3.jpg)

​	此时，用户再通过docker客户端进行push、pull，或者查询一些manifiest信息时，就会有相应的事件发送到定义的Endpoint上。

​	接下来看一下事件的格式及其主要属性：

![](https://pic.imgdb.cn/item/615bc5172ab3f51d91d9884f.jpg)

![](https://pic.imgdb.cn/item/615bc52a2ab3f51d91d9a109.jpg)

![](https://pic.imgdb.cn/item/615bc5372ab3f51d91d9b1b4.jpg)

​	每个事件的payload，都是一个定义好的JSON格式的数据。通知系统的主要属性主要包括action、target.mediaType、target.repository、target.url、request. method、request.useragent、actor.name等，参见表18-1。

![](https://pic.imgdb.cn/item/615bc5492ab3f51d91d9c669.jpg)

**2．通知系统的使用场景**

​	理解了如何配置Docker Registry v2的Notification、Endpoint以及接收的Event数据格式，我们就可以很方便地实现一些个性化的需求。这里简单列举两个场景：一个是如何统计镜像的上传、下载次数，方便了解镜像的使用情况；另一个是对服务的持续部署，方便管理镜像，参见图18-2。

![](https://pic.imgdb.cn/item/615bc55d2ab3f51d91d9db2f.jpg)

**（1）镜像上传、下载计数**

​	很常见的一个场景是根据镜像下载次数，向用户推荐使用最多的镜像，或者统计镜像更新的频率，以便了解用户对镜像的维护程度。

​	用户可以利用Notification功能定义自己的计数服务，并在Docker Registry上配置对应的Endpoint。在有pull、push动作发生时，对对应镜像的下载或者上传次数进行累加，达到计数效果。然后添加一个查询接口，供用户查看用户镜像的上传、下载次数，或者提供排行榜等扩展服务。

**（2）实现应用的自动部署**

​	在这个场景下，可以在新的镜像push到DockerRegistry服务器时候，自动创建或者更新对应的服务，这样可以快速查看新镜像的运行效果或者进行集成测试。用户还可以根据事件中的相应属性，比如用户信息、镜像名称等，调用对应的服务部署接口进行自动化部署操作。

​	另外，镜像的命名规则是namespace/repository:tag，但在上面的事件payload示例中，并没有看到tag的属性。如果需要tag信息，需要使用Docker Registry v2.4.0及以上的版本，在这个版本中对应的manifest事件中将会携带tag的属性，来标识该动作涉及的镜像版本信息。

# 19. 安全防护与配置

​	Docker的安全性在生产环境中是十分关键的衡量因素。Docker容器的安全性，在很大程度上依赖于Linux系统自身。目前，在评估Docker的安全性时，主要考虑下面几个方面：

* Linux内核的命名空间机制提供的容器隔离安全；
* Linux控制组机制对容器资源的控制能力安全；
* Linux内核的能力机制所带来的操作权限安全；
* Docker程序（特别是服务端）本身的抗攻击性；
* 其他安全增强机制（包括AppArmor、SELinux等）对容器安全性的影响；
* 通过第三方工具（如Docker Bench工具）对Docker环境的安全性进行评估。

## 19.1 命名空间隔离的安全

​	Docker容器和LXC容器在实现上很相似，所提供的安全特性也基本一致。当用docker [container]run命令启动一个容器时，Docker将在后台为容器创建一个独立的命名空间。命名空间提供了最基础也是最直接的隔离，在容器中运行的进程不会被运行在本地主机上的进程和其他容器通过正常渠道发现和影响。

​	从网络架构的角度来看，所有的容器实际上是通过本地主机的网桥接口（docker0）进行相互通信，就像物理机器通过物理交换机通信一样。

​	那么，Linux内核中实现命名空间（特别是网络命名空间）的机制是否足够成熟呢？Linux内核从2.6.15版本（2008年7月发布）开始引入命名空间，至今经历了数年的演化和改进，并应用于诸多大型生产系统中。实际上，命名空间的想法和设计提出的时间要更早，最初是OpenVZ项目的重要特性。OpenVZ项目早在2005年就已经正式发布，其设计和实现更加成熟。

​	当然，与虚拟机方式相比，通过命名空间来实现的隔离并不是那么绝对。运行在容器中的应用可以直接访问系统内核和部分系统文件。因此，用户必须保证容器中应用是安全可信的（这跟保证运行在系统中的软件是可信的一个道理），否则本地系统将可能受到威胁，即必须保证镜像的来源和自身可靠。

​	Docker自1.3.0版本起对镜像管理引入了签名系统，加强了对镜像安全性的防护，用户可以通过签名来验证镜像的完整性和正确性。

## 19.2 控制组资源控制的安全

​	控制组是Linux容器机制中的另外一个关键组件，它负责实现资源的审计和限制。

​	控制组提供了很多有用的特性。它可以确保各个容器公平地分享主机的内存、CPU、磁盘IO等资源；当然，更重要的是，通过控制组可以限制容器对资源的占用，确保了当某个容器对资源消耗过大时，不会影响到本地主机系统和其他容器。

​	尽管控制组不负责隔离容器之间相互访问、处理数据和进程，但是它在防止恶意攻击特别是拒绝服务攻击（DDoS）方面是十分有效的。

​	对于支持多用户的服务平台（比如公有的各种PaaS、容器云）上，控制组尤其重要。例如，当个别应用容器出现异常的时候，可以保证本地系统和其他容器正常运行而不受影响，从而避免引发“雪崩”灾难。

## 19.3 内核能力机制

​	能力机制（capability）是Linux内核一个强大的特性，可以提供细粒度的权限访问控制。传统的Unix系统对进程权限只有根权限（用户id为0，即为root用户）和非根权限（用户非root用户）两种粗粒度的区别。

​	Linux内核自2.2版本起支持能力机制，将权限划分为更加细粒度的操作能力，既可以作用在进程上，也可以作用在文件上。例如，一个Web服务进程只需要绑定一个低于1024端口的权限，并不需要完整的root权限，那么给它授权net_bind_service能力即可。此外，还可以赋予很多其他类似能力来避免进程获取root权限。

​	默认情况下，Docker启动的容器有严格限制，只允许使用内核的一部分能力，包括chown、dac_override、fowner、kill、setgid、setuid、setpcap、net_bind_service、net_raw、sys_chroot、mknod、setfcap、audit_write，等等。

​	使用能力机制对加强Docker容器的安全性有很多好处。通常，在服务器上会运行一堆特权进程，包括ssh、cron、syslogd、硬件管理工具模块（例如负载模块）、网络配置工具等。容器与这些进程是不同的，因为几乎所有的特权进程都由容器以外的支持系统来进行管理。例如：

* ssh访问由宿主主机上的ssh服务来管理；
* cron通常应该作为用户进程执行，权限交给使用它服务的应用来处理；
* 日志系统可由Docker或第三方服务管理；
* 硬件管理无关紧要，容器中也就无须执行udevd以及类似服务；
* 网络管理也都在主机上设置，除非特殊需求，容器不需要对网络进行配置。



​	从上面的例子可以看出，大部分情况下，容器并不需要“真正的”root权限，容器只需要少数的能力即可。为了加强安全，容器可以禁用一些没必要的权限，包括：

* 完全禁止任何文件挂载操作；
* 禁止直接访问本地主机的套接字；
* 禁止访问一些文件系统的操作，比如创建新的设备、修改文件属性等；
* 禁止模块加载。



​	这样，就算攻击者在容器中取得了root权限，也不能获得本地主机的较高权限，能进行的破坏也有限。

​	不恰当地给容器分配了内核能力，会导致容器内应用获取破坏本地系统的权限。例如，早期的Docker版本曾经不恰当地继承CAP_DAC_READ_SEARCH能力，导致容器内进程可以通过系统调用访问到本地系统的任意文件目录。

​	默认情况下，Docker采用白名单机制，禁用了必需的一些能力之外的其他权限，目前支持CAP_CHOWN、CAP_DAC_OVERRIDE、CAP_FSETID、CAP_FOWNER、CAP_MKNOD、CAP_NET_RAW、CAP_SETGID、CAP_SETUID、CAP_SETFCAP、CAP_SETPCAP、CAP_NET_BIND_SERVICE、CAP_SYS_CHROOT、CAP_KILL、CAP_AUDIT_WRITE等。

​	当然，用户也可以根据自身需求为Docker容器启用额外的权限。

## 19.4 Docker服务端的防护

​	使用Docker容器的核心是Docker服务端。Docker服务的运行目前还需要root权限的支持，因此服务端的安全性十分关键。

​	首先，必须确保只有可信的用户才能访问到Docker服务。Docker允许用户在主机和容器间共享文件夹，同时不需要限制容器的访问权限，这就容易让容器突破资源限制。例如，恶意用户启动容器的时候将主机的根目录/映射到容器的/host目录中，那么容器理论上就可以对主机的文件系统进行任意修改了。事实上，几乎所有虚拟化系统都允许类似的资源共享，而没法阻止恶意用户共享主机根文件系统到虚拟机系统。

​	这将会造成很严重的安全后果。因此，当提供容器创建服务时（例如通过一个Web服务器），要更加注意进行参数的安全检查，防止恶意用户用特定参数来创建一些破坏性的容器。

​	为了加强对服务端的保护，Docker的REST API（客户端用来与服务端通信的接口）在0.5.2之后使用本地的Unix套接字机制替代了原先绑定在127.0.0.1上的TCP套接字，因为后者容易遭受跨站脚本攻击。现在用户使用Unix权限检查来加强套接字的访问安全。

​	用户仍可以利用HTTP提供REST API访问。建议使用安全机制，确保只有可信的网络或VPN网络，或证书保护机制（例如受保护的stunnel和ssl认证）下的访问可以进行。此外，还可以使用TLS证书来加强保护，可以进一步参考dockerd的tls相关参数。

​	最近改进的Linux命名空间机制将可以实现使用非root用户来运行全功能的容器。这将从根本上解决了容器和主机之间共享文件系统而引起的安全问题。

​	目前，Docker自身改进安全防护的目标是实现以下两个重要安全特性：

* 将容器的root用户映射到本地主机上的非root用户，减轻容器和主机之间因权限提升而引起的安全问题；
* 允许Docker服务端在非root权限下运行，利用安全可靠的子进程来代理执行需要特权权限的操作。这些子进程将只允许在限定范围内进行操作，例如仅仅负责虚拟网络设定或文件系统管理、配置操作等。

## 19.5 更多安全特性的使用

​	除了默认启用的能力机制之外，还可以利用一些现有的安全软件或机制来增强Docker的安全性，例如GRSEC、AppArmor、SELinux等：

* 在内核中启用GRSEC和PAX，这将增加更多的编译和运行时的安全检查；并且通过地址随机化机制来避免恶意探测等。启用该特性不需要Docker进行任何配置；
* 使用一些增强安全特性的容器模板，比如带AppArmor的模板和RedHat带SELinux策略的模板。这些模板提供了额外的安全特性；
* 用户可以自定义更加严格的访问控制机制来定制安全策略。



​	此外，在将文件系统挂载到容器内部时候，可以通过配置只读（read-only）模式来避免容器内的应用通过文件系统破坏外部环境，特别是一些系统运行状态相关的目录，包括但不限于/proc/sys、/proc/irq、/proc/bus等。这样，容器内应用进程可以获取所需要的系统信息，但无法对它们进行修改。

​	同时，对于应用容器场景下，Docker内启动应用的用户都应为非特权用户（可以进一步禁用用户权限，如访问Shell），避免出现故障时对容器内其他资源造成损害。

## 19.6 使用第三方检测工具

​	前面笔者介绍了大量增强Docker安全性的手段。要逐一去检查会比较繁琐，好在已经有了一些进行自动化检查的开源工具，比较出名的有DockerBench和clair。

### 19.6.1 Docker Bench

​	Docker Bench是一个开源项目，代码托管在https://github.com/docker/docker-bench-secu-rity。该项目按照互联网安全中心（Centerfor Internet Security, CIS）对于Docker1.13.0+的安全规范进行一系列环境检查，可发现当前Docker部署在配置、安全等方面的潜在问题。CIS Docker规范在主机配置、Docker引擎、配置文件权限、镜像管理、容器运行时环境、安全项等六大方面都进行了相关的约束和规定，推荐大家在生产环境中使用Docker时，采用该规范作为部署的安全标准。

​	Docker Bench自身也提供了Docker镜像，采用如下命令，可以快速对本地环境进行安全检查：

![](https://pic.imgdb.cn/item/615c51252ab3f51d91a3ed02.jpg)

![](https://pic.imgdb.cn/item/615c51452ab3f51d91a41ebf.jpg)

![](https://pic.imgdb.cn/item/615c51592ab3f51d91a442a4.jpg)

​	输出结果中，带有不同的级别，说明问题的严重程度，最后会给出整体检查结果和评分。一般要尽量避免出现WARN或以上的问题。

​	用户也可以通过获取最新开源代码方式启动检测：

![](https://pic.imgdb.cn/item/615c51782ab3f51d91a47722.jpg)

### 19.6.2 clair

​	除了Docker Bench外，还有CoreOS团队推出的clair，它基于Go语言实现，支持对容器（支持appc和Docker）的文件层进行静态扫描发现潜在漏洞。项目地址为https://github.com/coreos/clair。读者可以使用Docker或Docker-compose方式快速进行体验。

​	使用Docker方式启动clair，如下所示：

![](https://pic.imgdb.cn/item/615c526c2ab3f51d91a5eb4b.jpg)

![](https://pic.imgdb.cn/item/615c527e2ab3f51d91a605c3.jpg)

在使用Docker的过程中，尤其需要注意如下几方面：

* 首先要牢记容器自身所提供的隔离性只是相对的，并没有虚拟机那样完善。因此，必须对容器内应用进行严格的安全审查。同时从容器层面来看，容器即应用，原先保障应用安全的各种手段，都可以合理地借鉴利用；
* 采用专用的服务器来运行Docker服务端和相关的管理服务（比如ssh监控和进程监控、管理工具nrpe、collectd等），并对该服务器启用最高级别的安全机制。而把其他业务服务都放到容器中去运行，确保即便个别容器出现问题，也不会影响到其他容器资源；
* 将运行Docker容器的机器划分为不同的组，互相信任的机器放到同一个组内；组之间进行资源隔离；同时进行定期的安全检查；
* 大规模运营场景下，需要考虑在容器网络上进行必备的安全防护，避免诸如DDoS、ARP攻击、规则表攻击等网络安全威胁，这也是生产环境需要关注的重要问题。

# 20. 高级网络功能

## 20.1 启动与配置参数

**1．网络启动过程**

​	Docker服务启动时会首先在主机上自动创建一个docker0虚拟网桥，实际上是一个Linux网桥。网桥可以理解为一个软件交换机，负责挂载其上的接口之间进行包转发。

​	同时，Docker随机分配一个本地未占用的私有网段（在RFC1918中定义）中的一个地址给docker0接口。比如典型的172.17.0.0/16网段，掩码为255.255.0.0。此后启动的容器内的网口也会自动分配一个该网段的地址。

​	当创建一个Docker容器的时候，同时会创建了一对veth pair互联接口。当向任一个接口发送包时，另外一个接口自动收到相同的包。互联接口的一端位于容器内，即eth0；另一端在本地并被挂载到docker0网桥，名称以veth开头（例如vethAQI2QT）。通过这种方式，主机可以与容器通信，容器之间也可以相互通信。如此一来，Docker就创建了在主机和所有容器之间一个虚拟共享网络，如图20-1所示。

**2．网络相关参数**

​	下面是与Docker网络相关的命令参数。其中部分命令选项只有在Docker服务启动的时候才能配置，修改后重启生效，包括：

* -b BRIDGE or --bridge=BRIDGE：指定容器挂载的网桥；
* --bip=CIDR：定制docker0的掩码；
* -H SOCKET... or --host=SOCKET...:Docker服务端接收命令的通道；

![](https://pic.imgdb.cn/item/615c6dde2ab3f51d91d581a5.jpg)

* --icc=true|false：是否支持容器之间进行通信；
* --ip-forward=true|false：启用net.ipv4.ip_forward，即打开转发功能；
* --iptables=true|false：禁止Docker添加iptables规则；
* --mtu=BYTES：容器网络中的MTU。



​	下面的命令选项既可以在启动服务时指定，也可以Docker容器启动（使用docker [con-tainer] run命令）时候指定。在Docker服务启动的时候指定则会成为默认值，后续执行该命令时可以覆盖设置的默认值：

* --dns=IP_ADDRESS：使用指定的DNS服务器；
* --dns-opt=""：指定DNS选项；
* --dns-search=DOMAIN：指定DNS搜索域。



​	还有些选项只能在docker [container] run命令执行时使用，因为它针对容器的配置：

* -h HOSTNAME or --hostname=HOSTNAME：配置容器主机名；
* -ip=""：指定容器内接口的IP地址；
* --link=CONTAINER_NAME:ALIAS：添加到另一个容器的连接；
* --net=bridge|none|container:NAME_or_ID|host|user_defined_network：配置容器的桥接模式；
* --network-alias：容器在网络中的别名；
* -p SPEC or --publish=SPEC：映射容器端口到宿主主机；
* -P or --publish-all=true|false：映射容器所有端口到宿主主机。

其中，--net选项支持以下五种模式：

* --net=bridge：默认配置。为容器创建独立的网络命名空间，分配网卡、IP地址等网络配置，并通过veth接口对将容器挂载到一个虚拟网桥（默认为docker0）上；
* --net=none：为容器创建独立的网络命名空间，但不进行网络配置，即容器内没有创建网卡、IP地址等；
* --net=container:NAME_or_ID：新创建的容器共享指定的已存在容器的网络命名空间，两个容器内的网络配置共享，但其他资源（如进程空间、文件系统等）还是相互隔离的；
* --net=host：不为容器创建独立的网络命名空间，容器内看到的网络配置（网卡信息、路由表、Iptables规则等）均与主机上的保持一致。注意其他资源还是与主机隔离的；
* --net=user_defined_network：用户自行用network相关命令创建一个网络，同一个网络内的容器彼此可见，可以采用更多类型的网络插件。

## 20.2 配置容器DNS和主机名

​	Docker服务启动后会默认启用一个内嵌的DNS服务，来自动解析同一个网络中的容器主机名和地址，如果无法解析，则通过容器内的DNS相关配置进行解析。用户可以通过命令选项自定义容器的主机名和DNS配置，下面分别介绍。

**1．相关配置文件**

​	容器中主机名和DNS配置信息可以通过三个系统配置文件来管理：/etc/resolv.conf、/etc/hostname和/etc/hosts。

​	启动一个容器，在容器中使用mount命令可以看到这三个文件挂载信息：

![](https://pic.imgdb.cn/item/615ec2782ab3f51d91f14fba.jpg)

​	Docker启动容器时，会从宿主机上复制/etc/resolv.conf文件，并删除掉其中无法连接到的DNS服务器：

![](https://pic.imgdb.cn/item/615ec2942ab3f51d91f1789d.jpg)

​	/etc/hosts文件中默认只记录了容器自身的地址和名称：

![](https://pic.imgdb.cn/item/615ec2a02ab3f51d91f18a89.jpg)

​	/etc/hostname文件则记录了容器的主机名：

![](https://pic.imgdb.cn/item/615ec2ac2ab3f51d91f19bef.jpg)

**2．容器内修改配置文件**

​	容器运行时，可以在运行中的容器里直接编辑/etc/hosts、/etc/hostname和/etc/resolve.conf文件。但是这些修改是临时的，只在运行的容器中保留，容器终止或重启后并不会被保存下来，也不会被docker commit提交。

**3．通过参数指定**

​	如果用户想要自定义容器的配置，可以在创建或启动容器时利用下面的参数指定，注意一般不推荐与-net=host一起使用，会破坏宿主机上的配置信息：

* 指定主机名-h HOSTNAME或者--hostname=HOSTNAME：设定容器的主机名。容器主机名会被写到容器内的/etc/hostname和/etc/hosts。但这个主机名只有容器内能中看到，在容器外部则看不到，既不会在docker ps中显示，也不会在其他容器的/etc/hosts中看到；
* --link=CONTAINER_NAME:ALIAS：记录其他容器主机名。在创建容器的时候，添加一个所连接容器的主机名到容器内/etc/hosts文件中。这样，新建容器可以直接使用主机名与所连接容器通信；
* --dns=IP_ADDRESS：指定DNS服务器。添加DNS服务器到容器的/etc/resolv. conf中，容器会用指定的服务器来解析所有不在/etc/hosts中的主机名；
* --dns-option list：指定DNS相关的选项；
* --dns-search=DOMAIN：指定DNS搜索域。设定容器的搜索域，当设定搜索域为．example.com时，在搜索一个名为host的主机时，DNS不仅搜索host，还会搜索host.example.com。

## 20.3 容器访问控制

​	容器的访问控制主要通过Linux上的iptables防火墙软件来进行管理和实现。iptables是Linux系统流行的防火墙软件，在大部分发行版中都自带。

**1．容器访问外部网络**

​	从前面的描述中，我们知道容器默认指定了网关为docker0网桥上的docker0内部接口。docker0内部接口同时也是宿主机的一个本地接口。因此，容器默认情况下可以访问到宿主机本地网络。如果容器要想通过宿主机访问到外部网络，则需要宿主机进行辅助转发。

​	在宿主机Linux系统中，检查转发是否打开，代码如下：

![](https://pic.imgdb.cn/item/615ec3472ab3f51d91f272e5.jpg)

**2．容器之间访问**

​	容器之间相互访问需要两方面的支持：

* 网络拓扑是否已经连通。默认情况下，所有容器都会连接到docker0网桥上，这意味着默认情况下拓扑是互通的；
* 本地系统的防火墙软件iptables是否允许访问通过。这取决于防火墙的默认规则是允许（大部分情况）还是禁止。



下面分两种情况介绍容器间的访问。

（1）访问所有端口

​	当启动Docker服务时候，默认会添加一条“允许”转发策略到iptables的FORWARD链上。通过配置--icc=true|false（默认值为true）参数可以控制默认的策略。

​	为了安全考虑，可以在Docker配置文件中配置DOCKER_OPTS=--icc=false来默认禁止容器之间的相互访问。

​	同时，如果启动Docker服务时手动指定--iptables=false参数，则不会修改宿主机系统上的iptables规则。
（2）访问指定端口

​	在通过-icc=false禁止容器间相互访问后，仍可以通过--link=CONTAINER_NAME:ALIAS选项来允许访问指定容器的开放端口。

​	例如，在启动Docker服务时，可以同时使用icc=false --iptables=true参数来配置容器间禁止访问，并允许Docker自动修改系统中的iptables规则。此时，系统中的iptables规则可能是类似如下规则，禁止所有转发流量：

![](https://pic.imgdb.cn/item/615ec9c12ab3f51d91fc2d16.jpg)

​	之后，启动容器（docker [container] run）时使用--link=CONTAINER_NAME:ALIAS选项。Docker会在iptable中为两个互联容器分别添加一条ACCEPT规则，允许相互访问开放的端口（取决于Dockerfile中的EXPOSE行）。

​	此时，iptables的规则可能是类似如下规则：

![](https://pic.imgdb.cn/item/615ec9d32ab3f51d91fc4852.jpg)

## 20.4 映射容器端口到宿主主机的实现

​	默认情况下，容器可以主动访问到外部网络的连接，但是外部网络无法访问到容器。

**1．容器访问外部实现**

​	假设容器内部的网络地址为172.17.0.2，本地网络地址为10.0.2.2。容器要能访问外部网络，源地址不能为172.17.0.2，需要进行源地址映射（Source NAT, SNAT），修改为本地系统的IP地址10.0.2.2。

​	映射是通过iptables的源地址伪装操作实现的。查看主机nat表上POSTROUTING链的规则。该链负责网包要离开主机前，改写其源地址：

![](https://pic.imgdb.cn/item/615ecc362ab3f51d91fff651.jpg)

**2．外部访问容器实现**

​	容器允许外部访问，可以在docker [container] run时候通过-p或-P参数来启用。

​	不管用哪种办法，其实也是在本地的iptable的nat表中添加相应的规则，将访问外部IP地址的包进行目标地址DNAT，将目标地址修改为容器的IP地址。

​	以一个开放80端口的Web容器为例，使用-P时，会自动映射本地49000～49900范围内的随机端口到容器的80端口：

![](https://pic.imgdb.cn/item/615ecc552ab3f51d910029e6.jpg)

​	可以看到，nat表中涉及两条链：PREROUTING链负责包到达网络接口时，改写其目的地址，其中规则将所有流量都转发到DOCKER链；而DOCKER链将所有不是从docker0进来的包（意味着不是本地主机产生），同时目标端口为49153的修改其目标地址为172.17.0.2，目标端口修改为80。

​	使用-p 80:80时，与上面类似，只是本地端口也为80：

![](https://pic.imgdb.cn/item/615ecc6d2ab3f51d9100526b.jpg)

这里有两点需要注意：

* 规则映射地址为0.0.0.0，意味着将接受主机来自所有网络接口上的流量。用户可以通过-pIP:host_port:container_port或-p IP::port来指定绑定的外部网络接口，以制定更严格的访问规则；
* 如果希望映射绑定到某个固定的宿主机IP地址，可以在Docker配置文件中指定DOCKER_OPTS="--ip=IP_ADDRESS"，之后重启Docker服务即可生效。

## 20.5 配置容器网桥

​	Docker服务默认会创建一个名称为docker0的Linux网桥（其上有一个docker0内部接口），它在内核层连通了其他的物理或虚拟网卡，这就将所有容器和本地主机都放到同一个物理网络。用户使用Docker创建多个自定义网络时可能会出现多个容器网桥。

​	Docker默认指定了docker0接口的IP地址和子网掩码，让主机和容器之间可以通过网桥相互通信，它还给出了MTU（接口允许接收的最大传输单元），通常是1500 B，或宿主主机网络路由上支持的默认值。这些值都可以在服务启动的时候进行配置：

* --bip=CIDR:IP地址加掩码格式，例如192.168.1.5/24；
* --mtu=BYTES：覆盖默认的Docker mtu配置。



​	也可以在配置文件中配置DOCKER_OPTS，然后重启服务。由于目前Docker网桥是Linux网桥，用户可以使用brctl show来查看网桥和端口连接信息：

![](https://pic.imgdb.cn/item/615ed15c2ab3f51d910821b7.jpg)

​	每次创建一个新容器的时候，Docker从可用的地址段中选择一个空闲的IP地址分配给容器的eth0端口，并且使用本地主机上docker0接口的IP作为容器的默认网关：

![](https://pic.imgdb.cn/item/615ed2362ab3f51d910993ce.jpg)

​	目前，Docker不支持在启动容器时候指定IP地址。

> 注意
>
> 容器默认使用Linux网桥，用户也可以替换为OpenvSwitch等功能更强大的网桥实现，支持更多的软件定义网络特性。

## 20.6 自定义网桥

​	除了默认的docker0网桥，用户也可以指定其他网桥来连接各个容器。在启动Docker服务的时候，可使用-b BRIDGE或--bridge=BRIDGE来指定使用的网桥。

​	如果服务已经运行，就需要先停止服务，并删除旧的网桥：

![](https://pic.imgdb.cn/item/615ed3332ab3f51d910b1580.jpg)

![](https://pic.imgdb.cn/item/615ed3e32ab3f51d910c6c1c.jpg)

## 20.7 使用OpenvSwitch网桥

​	Docker默认使用的是Linux自带的网桥实现，可以替换为使用功能更强大的Openv-Switch虚拟交换机实现。

**1．环境**

​	在debian:stable系统中进行测试。操作流程也适用于RedHat/CentOS系列系统，但少数命令和配置文件可能略有差异。

**2．安装Docker**

​	安装最近版本的Docker并启动服务。默认情况下，Docker服务会创建一个名为docker0的Linux网桥，作为连接容器的本地网桥。

​	可以通过如下命令查看：

![](https://pic.imgdb.cn/item/615ed41f2ab3f51d910ce122.jpg)

**3．安装OpenvSwitch**

![](https://pic.imgdb.cn/item/615ed42e2ab3f51d910d004e.jpg)

**4．配置容器连接到OpenvSwitch网桥**

​	目前OpenvSwitch网桥还不能直接支持挂载容器，需要手动在OpenvSwitch网桥上创建虚拟网口并挂载到容器中。操作方法如下。

（1）创建无网口容器

​	启动一个容器，并指定不创建网络，后面我们手动添加网络。较新版本的Docker默认不允许在容器内修改网络配置，需要在run的时候指定参数-privileged=true：

![](https://pic.imgdb.cn/item/615ed4492ab3f51d910d30f6.jpg)

（2）手动为容器添加网络

![](https://pic.imgdb.cn/item/615ed4582ab3f51d910d4ccc.jpg)

![](https://pic.imgdb.cn/item/615ed4682ab3f51d910d6e29.jpg)

![](https://pic.imgdb.cn/item/615ed4ec2ab3f51d910e61d3.jpg)

![](https://pic.imgdb.cn/item/615ed4f72ab3f51d910e7518.jpg)

## 20.8 创建一个点到点连接

​	在默认情况下，Docker会将所有容器连接到由docker0提供的虚拟网络中。

​	用户有时候需要两个容器之间可以直连通信，而不用通过主机网桥进行桥接。解决办法很简单：创建一对peer接口，分别放到两个容器中，配置成点到点链路类型即可。

​	下面笔者将通过手动操作完成Docker配置容器网络的过程。

![](https://pic.imgdb.cn/item/615ed5222ab3f51d910ebb82.jpg)

![](https://pic.imgdb.cn/item/615ed52d2ab3f51d910ecc41.jpg)

# 21. libnetwork插件化网络功能

## 21.1 容器网络模型

​	libnetwork中容器网络模型（Container Networking Model,CNM）十分简洁和抽象，可以让其上层使用网络功能的容器最大程度地忽略底层具体实现。

​	容器网络模型的结构如图21-1所示。

![](https://pic.imgdb.cn/item/6166d0dd2ab3f51d91be71e0.jpg)

​	容器网络模型包括三种基本元素：

* 沙盒（Sandbox）：代表一个容器（准确地说，是其网络命名空间）；
* 接入点（Endpoint）：代表网络上可以挂载容器的接口，会分配IP地址；
* 网络（Network）：可以连通多个接入点的一个子网。



​	可见，对于使用CNM的容器管理系统来说，具体底下网络如何实现，不同子网彼此怎么隔离，有没有QoS，都不关心。只要插件能提供网络和接入点，只需把容器给接上或者拔下，剩下的都是插件驱动自己去实现，这样就解耦了容器和网络功能，十分灵活。

​	CNM的典型生命周期如图21-2所示：首先，驱动注册自己到网络控制器，网络控制器使用驱动类型，来创建网络；然后在创建的网络上创建接口；最后把容器连接到接口上即可。销毁过程则正好相反，先把容器从接入口上卸载，然后删除接入口和网络即可。

![](https://pic.imgdb.cn/item/6166d1492ab3f51d91bf20e5.jpg)

​	目前CNM支持的驱动类型有四种：Null、Bridge、Overlay、Remote，简单介绍如下：

* Null：不提供网络服务，容器启动后无网络连接；
* Bridge：就是Docker传统上默认用Linux网桥和Iptables实现的单机网络；
* Overlay：是用vxlan隧道实现的跨主机容器网络；
* Remote：扩展类型，预留给其他外部实现的方案，比如有一套第三方的SDN方案（如OpenStack Neutron）就可以接进来。



​	从位置上看，libnetwork往上提供容器支持，往下隐藏实现差异，自身处于十分关键的中间层。读者如果熟悉计算机网络协议模型的话，libnetwork可以类比为最核心的TCP/IP层。

​	目前，已有大量的网络方案开始支持libnetwork。包括OpenStack Kuryr项目，使用libnetwork，让Docker可以直接使用Neutron提供的网络功能。Calico等团队也编写了插件支持libnetwork，可以无缝地支持Docker高级网络功能。

## 21.2 Docker网络命令

​	在libnetwork支持下，Docker网络相关操作都作为network的子命令出现。

​	围绕着CNM生命周期的管理，主要包括以下命令：

* create：创建一个网络；
* connect：将容器接入到网络；
* disconnect：把容器从网络上断开；
* inspect：查看网络的详细信息。
* ls：列出所有的网络；
*  rm：删除一个网络。

**1．创建网络**

​	creat命令用于创建一个新的容器网络。Docker内置了bridge（默认使用）和overlay两种驱动，分别支持单主机和多主机场景。Docker服务在启动后，会默认创建一个bridge类型的网桥bridge。不同网络之间默认相互隔离。

​	创建网络命令格式为docker network create [OPTIONS]NETWORK。

​	支持参数包括：

* -attachable[=false]：支持手动容器挂载；
* -aux-address=map[]：辅助的IP地址；
* -config-from=""：从某个网络复制配置数据；
* -config-only[=false]：启用仅可配置模式；
*  -d, -driver="bridge"：网络驱动类型，如bridge或overlay；
* -gateway=[]：网关地址；
* -ingress[=false]：创建一个Swarm可路由的网状网络用于负载均衡，可将对某个服务的请求自动转发给一个合适的副本；
* -internal[=false]：内部模式，禁止外部对所创建网络的访问；
* -ip-range=[]：指定分配IP地址范围；
*  -ipam-driver="default":IP地址管理的插件类型；
* -ipam-opt=map[]:IP地址管理插件的选项；
* -ipv6[=false]：支持IPv6地址；
* -label value：为网络添加元标签信息；
* -o, -opt=map[]：网络驱动所支持的选项；
* -scope=""：指定网络范围；
* -subnet=[]：网络地址段，CIDR格式，如172.17.0.0/16。

**2．接入网络**

​	connect命令将一个容器连接到一个已存在的网络上。连接到网络上的容器可以跟同一网络中其他容器互通，同一个容器可以同时接入多个网络。也可以在执行docker run命令时候通过-net参数指定容器启动后自动接入的网络。

​	接入网络命令格式为docker network connect [OPTIONS]NETWORK CONTAINER。

​	支持参数包括：

* -alias=[]：为容器添加一个别名，此别名仅在所添加网络上可见；
* -ip=""：指定IP地址，需要注意不能跟已接入的容器地址冲突；
* -ip6=""：指定IPv6地址；
* -link value：添加链接到另外一个容器；
* -link-local-ip=[]：为容器添加一个链接地址。

**3．断开网络**

​	disconnect命令将一个连接到网络上的容器从网络上断开连接。

​	命令格式为docker network disconnect [OPTIONS]NETWORK CONTAINER。

​	支持参数包括-f, -force：强制把容器从网络上移除。

**4．查看网络信息**

​	inspect命令用于查看一个网络的具体信息（JSON格式），包括接入的容器、网络配置信息等。

​	命令格式为docker network inspect [OPTIONS] NETWORK[NETWORK...]。

​	支持参数包括：

* -f, -format=""：给定一个Golang模板字符串，对输出结果进行格式化，如只查看地址配置可以用-f'{{.IPAM.Config}}'；
* -v, -verbose[=false]：输出调试信息。

**5．列出网络**

​	ls命令用于列出网络。命令格式为docker network ls[OPTIONS]，其中支持的选项主要有：

*  -f, -filter=""：指定输出过滤器，如driver=bridge；
* -format=""：给定一个golang模板字符串，对输出结果进行格式化；
* -no-trunc[=false]：不截断地输出内容；
* -q, -quiet[=false]：安静模式，只打印网络的ID。



​	实际上，在不执行额外网络命令的情况下，用户执行dockernetwork ls命令，一般情况下可以看到已创建的三个网络：

![](https://pic.imgdb.cn/item/6166d5e02ab3f51d91c61369.jpg)

**6．清理无用网络**

​	prune命令用于清理已经没有容器使用的网络。

​	命令格式为docker network prune [OPTIONS] [flags]，支持参数包括：

* -filter=""：指定选择过滤器；
* -f, -force：强制清理资源。

**7．删除网络**

​	rm命令用于删除指定的网络。当网络上没有容器连接上时，才会成功删除。

​	命令格式为docker network rm NETWORK [NETWORK...]。

## 21.3 构建跨主机容器网络

​	在这里，笔者将演示使用libnetwork自带的Overlay类型驱动来轻松实现跨主机的网络通信。Overlay驱动默认采用VXLAN协议，在IP地址可以互相访问的多个主机之间搭建隧道，让容器可以互相访问。

**1．配置网络信息管理数据库**

​	我们知道，在现实世界中，要连通不同的主机，需要交换机或路由器（跨子网时需要）这样的互联设备。这些设备一方面是在物理上起到连接作用，但更重要的是起到了网络管理的功能。例如，主机位置在什么地方，地址是多少等信息，都需要网络管理平面来维护。

​	在libnetwork的网络方案中，要实现跨主机容器网络，也需要类似的一个网络信息管理机制，只不过这个机制简单得多，只是一个键值数据库而已，如Consul、Etcd、ZooKeeper等工具都可以满足需求。

​	以Consul为例，启动一个progrium/consul容器，并映射服务到本地的8500端口，代码如下：

![](https://pic.imgdb.cn/item/6166d6652ab3f51d91c6cc66.jpg)

**2．配置Docker主机**

​	启动两台Docker主机n1和n2，分别安装好最新的Docker-engine（1.7.0+）。确保这两台主机之间可以通过IP地址互相访问，另外，都能访问到数据库节点的8500端口。

​	配置主机的Docker服务启动选项如下：

![](https://pic.imgdb.cn/item/6166d68c2ab3f51d91c70038.jpg)

**3．创建网络**

​	分别在n1和n2上查看现有的Docker网络，包括三个默认网络：分别为bridge、host和none类型：

![](https://pic.imgdb.cn/item/6166d6a22ab3f51d91c71e9e.jpg)

![](https://pic.imgdb.cn/item/6166d6b02ab3f51d91c7344d.jpg)

![](https://pic.imgdb.cn/item/6166d6c02ab3f51d91c74a94.jpg)

**4．测试网络**

​	在n1上启动一个容器c1，通过--net选项指定连接到multi网络上。

​	查看网络信息，其中一个接口eth0已经连接到了multi网络上：

![](https://pic.imgdb.cn/item/6166d6e22ab3f51d91c77a0e.jpg)

​	在n2上启动一个容器c2，同样连接到multi网络上。

​	通过ping c1进行测试，可以访问到另外一台主机n1上的容器c1：

![](https://pic.imgdb.cn/item/6166d6ff2ab3f51d91c7a462.jpg)

## 21.4 小结

​	略。

