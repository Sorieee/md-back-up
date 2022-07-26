参考书籍《精通Nginx》

# 1. 安装Nginx及第三方模块

​	在我们开始探索如何配置Nginx之前，首先我们要安装它。这一章将详细讲述如何安装Nginx，以及如何获取正确的模块并安装与配置它们。Nginx是模块化设计的，并且有非常丰富的第三方模块开发者社区。它们的设计者通过创建这些模块为核心Nginx服务器增添了功能，我们可以在编译安装Nginx时将它们添加到Nginx服务器。

## 1.1 使用包管理器安装Nginx

```sh
# deb
sudo apt-get install nginx
# rpm
sudo yum install nginx
# FreeBSD
sudp pkg_install -r nginx
```

​	通过上述命令，Nginx将会安装到操作系统的标准位置下。如果使用操作系统的安装包安装Nginx，那么通过上面的命令来安装是最佳方式。

​	Nginx核心团队也提供了稳定的二进制版本，可以从http://nginx.org/en/download.html页面下载可用的版本。未发布nginx安装包的系统用户（例如，CentOS），可以使用下面的指导来安装预测试、预编译二进制版本。

### 1.1.1 在Centos上安装Nginx

​	通过创建下面的文件，在系统中添加Nginx仓库的yum配置：

```sh
sudo vi /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

然后通过命令安装

```sh
sudo yum install nginx
```

## 1.2 从源代码安装Nginx

略

## 1.5 查找并安装第三方模块

安装第三方模块的过程相当简单，步骤如下。

1．定位你想要使用的模块（在https://github.com或者是http://wiki.nginx.org/3rdPartyModules查找）。2．下载该模块。

3．解压缩源代码安装包。

4．如果有README文件，那么阅读README文件，查看在安装中是否有依赖安装。

5．通过`./configure-add-module=<path>`选项配置使用该模块。这个过程会给你的nginx二进制文件与模块附加这个功能。需要注意的是，很多第三方模块是实验性质的。因此，在将这些模块用于生产系统之前，首先要测试使用这些模块。另外请记住，Nginx的开发版本中可能会有API的变化，会导致第三方模块出现问题。

## 1.6 添加对Lua的支持

​	特别应该提到的是ngx_lua这个第三方模块，ngx_lua模块提供了启用Lua的功能，而不是像Perl一样在配置时嵌入式脚本语言。该模块对于perl模块来说最大的优点就是它的无阻塞性，并与其他第三方模块紧密集成。对于它的安装说明的完整描述详见：https://github.com/openresty/lua-nginx-module#installation

# 2. 配置指南

通过这一章的讨论话题，帮助你达到如下目标。

◆ 基本配置格式。

◆ Nginx全局配置参数。

◆ 使用include文件。

◆ HTTP的server部分。

◆ 虚拟服务器部分。

◆ location——where, when, how。

◆ mail的server部分。

◆ 完整的示例配置文件。

## 2.1 基本配置格式

基本的Nginx配置文件由若干个部分组成，每一个部分都是通过下列方法定义的。

```
<section> {
	<directive> <parameters>;
}
```

## 2.2 Nginx全局配置参数

全局配置部分被用于配置对整个server都有效的参数和前一个章节中的例外格式。全局部分可能包含配置指令，例如，user和worker_processes，也包括“节、部分（section）”。例如，events，这里没有大括号（{}）包围全局部分。在全局部分中，最重要的配置指令都在表2-1中，这些配置指令将会是你处理的最重要部分。

![](https://pic.imgdb.cn/item/62df9b67f54cd3f9374a749d.jpg)

下面是一个使用这些指令的简短示例：

```
# we want nginx to run as user 'www'
user www;

# the load is CPU-bound and we have 12 cores
worker_processes 12;

# explicitly specifying the path to the mandatory error log
error_log /var/log/nginx/error.log
# sets up a new configuration context for the 'events' module
events {
	use /dev/poll;
	worker_connections 2048;
}
```

## 2.3 使用include文件

​	在Nginx的配置文件中，include文件可以在任何地方，以便增强配置文件的可读性，并且能够使得部分配置文件重新使用。使用include文件，要确保被包含的文件自身有正确的Nginx语法，即配置指令和块（blocks），然后指定这些文件的路径。

![](https://pic.imgdb.cn/item/62df9c59f54cd3f9374f6e26.jpg)

## 2.4 HTTP的server部分

### 2.4.1 客户端指令

如表2-2所示，这一组指令用于处理客户端连接本身的各个方面以及不同类型的客户端。

![](https://pic.imgdb.cn/item/62df9c8bf54cd3f937508ece.jpg)

### 2.4.2 文件I/O 指令

![](https://pic.imgdb.cn/item/62df9d14f54cd3f937538e77.jpg)

### 2.4.3 Hash指令

![](https://pic.imgdb.cn/item/62df9d3ef54cd3f93754816d.jpg)

### 2.4.4 Socket指令

![](https://pic.imgdb.cn/item/62df9d4cf54cd3f93754cded.jpg)

### 2.4.5 示例配置文件

下面是一个http配置部分的例子。

![](https://pic.imgdb.cn/item/62df9d64f54cd3f937555ad0.jpg)

## 2.5 虚拟服务器

​	任何由关键字server开始的部分都被称作“虚拟服务器”部分。它描述的是一组根据不同的server_name指令逻辑分割的资源，这些虚拟服务器响应HTTP请求，因此它们都包含在http部分中。

​	一个虚拟服务器由listen和server_name指令组合定义，listen指令定义了一个IP地址/端口组合或者是UNIX域套接字路径。

![](https://pic.imgdb.cn/item/62df9db3f54cd3f937572606.jpg)

​	server_name指令是相当简单的，但可以用来解决一些配置问题。它的默认值为""，这意味着server部分没有server_name指令，对于没有设置Host头字段的请求，它将会匹配该server处理。这种情况可用于如丢弃这种缺乏Host头的请求。

![](https://pic.imgdb.cn/item/62df9dd7f54cd3f93757eb64.jpg)

​	在这个例子中，使用的HTTP非标准代码444将会使得Nginx立即关闭一个连接。

​	除了普通的字符串之外，Nginx也接受通配符作为server_name指令的参数。

* 在这个例子中，使用的HTTP非标准代码444将会使得Nginx立即关闭一个连接。除了普通的字符串之外，Nginx也接受通配符作为server_name指令的参数。
* 通配符可以替代部分顶级域：www.example.*。
  * 一种特殊形式将匹配子域或域本身：.example.com（匹配*.example.com也包括example.com）。

![](https://pic.imgdb.cn/item/62df9e0df54cd3f937592762.jpg)

对于一个特定的请求，确定哪些虚拟服务器提供该请求的服务时，Nginx应该遵循下面的逻辑。

1．匹配IP地址和listen指令指定的端口。

2．将Host头字段作为一个字符串匹配server_name指令。

3．将Host头字段与server_name指令值字符串的开始部分做匹配。

4．将Host头字段与server_name指令值字符串的结尾部分做匹配。

5．将Host头字段与server_name指令值进行正则表达式匹配。

6．如果所有Host头匹配失败，那么将会转向listen指令标记的default_server。

7．如果所有的Host头匹配失败，并且没有default_server，那么将会转向第一个server的listen指令，以满足第1步。

![](https://pic.imgdb.cn/item/62df9e38f54cd3f9375a1a5b.jpg)

​	参数default_server被用于处理其他没有被处理的请求。因此，总是明确地推荐设置default_server，以便这些没有被处理的请求通过这种定义的方式处理。

​	除了这个用法外，default_server也可以使用同样的listen指令配置若干个虚拟服务器。这里设置的任何指令都将会在匹配的server区段有效。

## 2.6 Locations—where, when, how

​	location指令可以用在虚拟服务器server部分，并且意味着提供来自客户端的URI或者内部重定向访问。除少数情况外，location也可以被嵌套使用，它们被作为特定的配置尽可能地处理请求。

![](https://pic.imgdb.cn/item/62df9ec5f54cd3f9375d38d0.jpg)

当一个请求进入时，URI将会被检测匹配一个最佳的location。

* 没有正则表达式的location被作为最佳的匹配，独立于含有正则表达式的location顺序。
* 在配置文件中按照查找顺序进行正则表达式匹配。在查找到第一个正则表达式匹配之后结束查找。由这个最佳的location提供请求处理。



这里比较匹配描述的是解码URI，例如，在URI中的"%20"，将会匹配location中的" "（空格）。

![](https://pic.imgdb.cn/item/62df9f24f54cd3f9375f49f3.jpg)

​	指令try_files在这里也值得一提，它也可以用在server部分，但是最常见的还是在location部分中。try_files指令将会按照给定它的参数列出顺序进行尝试，第一个被匹配的将会被使用。它经常被用于从一个变量去匹配一个可能的文件，然后将处理传递到一个命名location，如下面的示例所示。

![](https://pic.imgdb.cn/item/62df9f62f54cd3f93760a7d0.jpg)

​	在这里有一个隐含的目录索引，如果给定的URI作为一个文件没有被找到，那么处理将会通过代理被传递到appserver。我们将会在本书的其他部分讨论如何最好地使用location、try_files和proxy_pass来解决特定的问题。

​	除以下前缀外，locations可以被嵌套。

​	◆ 具有"=" 前缀。

​	◆ 命名location。最佳实践表明正则表达式location被嵌套在基于字符串的location内，如下面的示例所示。

![](https://pic.imgdb.cn/item/62df9f62f54cd3f93760a7d0.jpg)

## 2.7 完整的示例配置文件

下面是一个示例配置文件，它包括了本章讨论的各个不同方面。请注意，不要复制粘贴该示例配置文件。因为它很可能不是你需要的配置，该代码只是显示了一个完整配置文件的架构而已。

![](https://pic.imgdb.cn/item/62df9fc4f54cd3f93762b6f7.jpg)

![](https://pic.imgdb.cn/item/62df9fd7f54cd3f9376329f1.jpg)

![](https://pic.imgdb.cn/item/62df9feaf54cd3f937638ffc.jpg)

# 3. 使用mail模块

​	Nginx设计为不但能够提供Web服务，而且还提供了邮件代理服务。在本章中，你将会学习到如何将Nginx配置为一个代理POP3、IMAP和SMTP的服务器。mail模块对需要接受大量连接的用户很有用，然而，后端邮箱基础架构无法处理负载或者需要防止直接接入到Internet。本章还包括一般用途的服务，如认证服务、缓存和解释日志文件等主题，即使电子邮件服务不能满足你的需求。

# 4.  Nginx作为反向代理

## 4.1 反向代理简介

​	Nginx能够作为一个反向代理来终结来自于客户端的请求，并且向上游服务器打开一个新请求。在这个处理的过程中，为了更好地响应客户端请求，该请求可以根据它的URI、客户机参数或者一些其他的逻辑进行拆分。通过代理服务器，请求的原始URL中的任何部分都能够以这种方式进行转换。

![](https://pic.imgdb.cn/item/62dfa252f54cd3f93770f0d2.jpg)

​	第二个例外的情况，如果在location内有rewrite规则改变了URI，那么Nginx使用这个URI处理请求，不再发生转换。在这例子中，URI传递到上游服务器的将会是`/index.php? page=<match>`，这个`<match>`会是来自于括号中捕获的参数，而不是预期的/index，如proxy_pass指令指示的URI部分。

![](https://pic.imgdb.cn/item/62dfa269f54cd3f9377170f8.jpg)

## 4.2 代理模块

表4-1总结了一些在代理模块中常用的指令。

![](https://pic.imgdb.cn/item/62dfa296f54cd3f937726a9a.jpg)

## 4.4 upstream模块

​	与proxy模块紧密搭配的是upstream模块。upstream模块将会启用一个新的配置区段，在该区段定义了一组上游服务器。这些服务器可能被设置了不同的权重（权重越高的上游服务器将会被Nginx传递越多的连接），也可能是不同的类型（TCP与UNIX域），也可能出于需要对服务器进行维护，故而标记为down。

![](https://pic.imgdb.cn/item/62dfa363f54cd3f93776c7a2.jpg)

​	keepalive指令特别值得一提，Nginx服务器将会为每一个worker进程保持同上游服务器的连接。在Nginx需要同上游服务器持续保持一定数量的打开连接时，连接缓存非常有用。如果上游服务器通过HTTP进行“对话”，那么Nginx将会使用HTTP/1.1协议的持久连接机制维护这些打开的连接。

![](https://pic.imgdb.cn/item/62dfa380f54cd3f937776655.jpg)

## 4.6 上游服务器的类型

​	上游服务器是Nginx代理连接的一个服务器，它可以是不同的物理机器，也可以是虚拟机，但是并不是必须如此。上游服务器可以是一个在本地机器上监听UNIX域套接字的机器，也可能是TCP监听的众多不同机器中的其中一员。它可能是拥有处理不同请求的多种模块的Apache服务器，或者是一个Rack中间件服务器，为Ruby应用程序提供HTTP接口，Nginx可以为它们配置代理。

## 4.8 多个上游服务器

​	也可能需要配置Nginx将请求传递到多个上游服务器，这可以通过upstream来声明，定义多个server可以参考upstream中的proxy_pass指令。

![](https://pic.imgdb.cn/item/62dfa416f54cd3f9377a8933.jpg)

​	使用这个配置，Nginx将会通过轮询的方式将连续的请求传递给3个上游服务器。在一个应用程序仅处理一个请求时，这个配置很有用。你希望Nginx使用这种方式处理客户端通信，以便应用程序不会过载。图4-1将阐释这个配置。

![](https://pic.imgdb.cn/item/62dfa42ff54cd3f9377b0583.jpg)

## 4.9 非HTTP型上游服务器

​	到目前为止，我们讨论的都是与上游服务器通过HTTP进行通信的情况，对于这种情况，我们使用了proxy_pass指令。正如本章前面暗示的，在保持活动连接部分，Nginx能够将请求代理到不同类型的上游服务器，它们每一个都有相应的*_pass指令。

### 4.9.1 Memcached上游服务器

​	在Nginx中，memcached模块（默认启用）负责与memcached守护进程通信。因此，客户端和memcached守护进程之间没有直接通信，也就是说，在这种情况下，Nginx不是充当反向代理。memcached模块使得Nginx使用memcached协议会话，因此，key的查询能够在请求传递到应用程序服务器之前完成。

![](https://pic.imgdb.cn/item/62dfa472f54cd3f9377c80da.jpg)

​	memcached_pass指令使用$memcached_key变量实现key的查找。如果没有响应值（error_page 404），我们将该请求传递到localhost，可能在该服务器上运行着用于处理这种请求的服务，并且会在memcached实例中插入键/值对。

## 4.10 负载均衡

​	我们已经在我们的上游服务器讨论中展示了一些使用负载均衡的示例。除了作为客户端从互联网作为反向代理的终止点，Nginx还提供负载均衡器的功能。它可以通过扩展它代理的连接来保护你的上游服务器免于过载。取决于你使用的环境，你可以选择三种负载均衡算法中的一种。

**负载均衡算法**

​	upstream模块能够使用3种负载均衡：轮询（round-robin）、IP哈希（IP hash）和最少连接数（Least Connection），你可以使用其中的一种来选择哪一个上游服务器将会在下一步中被连接。在默认情况下，使用轮询算法，它不需要配置指令来激活。该算法选择下一个服务器，基于先前选择，在配置文件中哪一个是下一个服务器，以及每一个服务器的负载权重。轮询算法是基于在队列中谁是下一个的原理确保将访问量均匀地分配给每一个上游服务器的。

## 4.11 将if配置转换为一个更现代的解释

​	在location内使用if指令，在某些情况下，真的只是考虑有效性。if可能与last或者break标签实现返回和重定向，但是在一般情况下避免使用。这是由于在部分的事实中，if可能产生一些意想不到的结果。考虑一下下面示例的配置。

![](https://pic.imgdb.cn/item/62dfa651f54cd3f93786a38c.jpg)

![](https://pic.imgdb.cn/item/62dfa6f2f54cd3f93789f2ae.jpg)

## 4.12 使用错误文件处理上游服务器问题

略

## 4.13 确定客户端真实的IP地址

![](https://pic.imgdb.cn/item/62dfaa42f54cd3f9379af68b.jpg)