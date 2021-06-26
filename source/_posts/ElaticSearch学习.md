# 官方文档(start)

https://www.elastic.co/guide/en/elasticsearch/reference/7.x/targz.html

# Set up ES

## 下载

https://www.elastic.co/guide/en/elasticsearch/reference/7.x/install-elasticsearch.html

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.2-linux-x86_64.tar.gz
```

### Enable系统索引自动创建

​	一些商业特性自动用es创建索引。默认情况下，es需要配置去允许自动创建

​	Ff you have disabled automatic index creation in Elasticsearch, you must configure [`action.auto_create_index`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docs-index_.html#index-creation) in `elasticsearch.yml` to allow the commercial features to create the following indices:

```
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```

> If you are using [Logstash](https://www.elastic.co/products/logstash) or [Beats](https://www.elastic.co/products/beats) then you will most likely require additional index names in your `action.auto_create_index` setting, and the exact value will depend on your local configuration. If you are unsure of the correct value for your environment, you may consider setting the value to `*` which will allow automatic creation of all indices.

### 命令行运行es

```
./bin/elasticsearch
```

​	If you have password-protected the Elasticsearch keystore, you will be prompted to enter the keystore’s password. See [Secure settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html) for more details.

​	By default, Elasticsearch runs in the foreground, prints its logs to the standard output (`stdout`), and can be stopped by pressing `Ctrl-C`.

>不能用root账号运行
>
>group add es
>
>useradd es -g es
>
>passwd es
>
>es3957979

**报错解决**

https://blog.csdn.net/weixin_42844971/article/details/93719571

https://blog.csdn.net/qq_43655835/article/details/104637625

**外网访问**

https://blog.csdn.net/lin_will/article/details/798347

### 检查ES是否运行

```
curl -X GET "localhost:9200/?pretty"
```

output

```
{
  "name" : "localhost.localdomain",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "mg7ltsxkT0uhx6bEbaGeew",
  "version" : {
    "number" : "7.13.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "4d960a0733be83dd2543ca018aa4ddc42e956800",
    "build_date" : "2021-06-10T21:01:55.251515791Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```

### Running as a daemon

​	To run Elasticsearch as a daemon, specify `-d` on the command line, and record the process ID in a file using the `-p` option:

```
./bin/elasticsearch -d -p pid
```

​	Log messages can be found in the `$ES_HOME/logs/` directory.

​	To shut down Elasticsearch, kill the process ID recorded in the `pid` file:

```
pkill -F pid
```

> The Elasticsearch `.tar.gz` package does not include the `systemd` module. To manage Elasticsearch as a service, use the [Debian](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/starting-elasticsearch.html#start-deb) or [RPM](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/starting-elasticsearch.html#start-rpm) package instead.

### 带参数运行

​	Elasticsearch loads its configuration from the `$ES_HOME/config/elasticsearch.yml` file by default. The format of this config file is explained in [*Configuring Elasticsearch*](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/settings.html).

​	Any settings that can be specified in the config file can also be specified on the command line, using the `-E` syntax as follows:

```
./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1
```

> Typically, any cluster-wide settings (like `cluster.name`) should be added to the `elasticsearch.yml` config file, while any node-specific settings such as `node.name` could be specified on the command line.

### Directory layout of archives

​	The archive distributions are entirely self-contained. All files and directories are, by default, contained within `$ES_HOME` — the directory created when unpacking the archive.

| Type        | Description                                                  | Default Location                           | Setting        |
| ----------- | ------------------------------------------------------------ | ------------------------------------------ | -------------- |
| **home**    | Elasticsearch home directory or `$ES_HOME`                   | Directory created by unpacking the archive |                |
| **bin**     | Binary scripts including `elasticsearch` to start a node and `elasticsearch-plugin` to install plugins | `$ES_HOME/bin`                             |                |
| **conf**    | Configuration files including `elasticsearch.yml`            | `$ES_HOME/config`                          | `ES_PATH_CONF` |
| **data**    | The location of the data files of each index / shard allocated on the node. | `$ES_HOME/data`                            | `path.data`    |
| **logs**    | Log files location.                                          | `$ES_HOME/logs`                            | `path.logs`    |
| **plugins** | Plugin files location. Each plugin will be contained in a subdirectory. | `$ES_HOME/plugins`                         |                |
| **repo**    | Shared file system repository locations. Can hold multiple locations. A file system repository can be placed in to any subdirectory of any directory specified here. | Not configured                             | `path.repo`    |

## Configuring Elasticsearch

​	略

## Important System Configuration

### Configuring system setting

​	Where to configure systems settings depends on which package you have used to install Elasticsearch, and which operating system you are using.

​	When using the `.zip` or `.tar.gz` packages, system settings can be configured:

- temporarily with [`ulimit`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setting-system-settings.html#ulimit), or
- permanently in [`/etc/security/limits.conf`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setting-system-settings.html#limits.conf).



​	When using the RPM or Debian packages, most system settings are set in the [system configuration file](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setting-system-settings.html#sysconfig). However, systems which use systemd require that system limits are specified in a [systemd configuration file](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setting-system-settings.html#systemd).



**ulimit**

​	On Linux systems, `ulimit` can be used to change resource limits on a temporary basis. Limits usually need to be set as `root` before switching to the user that will run Elasticsearch. For example, to set the number of open file handles (`ulimit -n`) to 65,536, you can do the following:

```
sudo su  
ulimit -n 65535 
su elasticsearch 
```

* Become root

* Change the max number of open files

* Become the `elasticsearch` user in order to start Elasticsearch.

  

  The new limit is only applied during the current session.

  You can consult all currently applied limits with `ulimit -a`.

**/etc/security/limits.conf**

​	On Linux systems, persistent limits can be set for a particular user by editing the `/etc/security/limits.conf` file. To set the maximum number of open files for the `elasticsearch` user to 65,535, add the following line to the `limits.conf` file:

```conf
elasticsearch  -  nofile  65535
```

​	This change will only take effect the next time the `elasticsearch` user opens a new session.

> ### Ubuntu and `limits.conf`
>
> Ubuntu ignores the `limits.conf` file for processes started by `init.d`. To enable the `limits.conf` file, edit `/etc/pam.d/su` and uncomment the following line:
>
> ```
> # session    required   pam_limits.so
> ```

#### Sysconfig file

​	When using the RPM or Debian packages, system settings and environment variables can be specified in the system configuration file, which is located in:

|        |                                |
| ------ | ------------------------------ |
| RPM    | `/etc/sysconfig/elasticsearch` |
| Debian | `/etc/default/elasticsearch`   |

​	However, for systems which uses `systemd`, system limits need to be specified via [systemd](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setting-system-settings.html#systemd).

#### System

​	When using the RPM or Debian packages on systems that use [systemd](https://en.wikipedia.org/wiki/Systemd), system limits must be specified via systemd.

The systemd service file (`/usr/lib/systemd/system/elasticsearch.service`) contains the limits that are applied by default.

To override them, add a file called `/etc/systemd/system/elasticsearch.service.d/override.conf` (alternatively, you may run `sudo systemctl edit elasticsearch` which opens the file automatically inside your default editor). Set any changes in this file, such as:

```sh
[Service]
LimitMEMLOCK=infinity
```



Once finished, run the following command to reload units:

```sh
sudo systemctl daemon-reload
```

### Disable swapping

​	Most operating systems try to use as much memory as possible for file system caches and eagerly swap out unused application memory. This can result in parts of the JVM heap or even its executable pages being swapped out to disk.

### Disable all swap files

​	Usually Elasticsearch is the only service running on a box, and its memory usage is controlled by the JVM options. There should be no need to have swap enabled.

​	On Linux systems, you can disable swap temporarily by running:

```sh
sudo swapoff -a
```

​	This doesn’t require a restart of Elasticsearch.

​	To disable it permanently, you will need to edit the `/etc/fstab` file and comment out any lines that contain the word `swap`.

​	On Windows, the equivalent can be achieved by disabling the paging file entirely via `System Properties → Advanced → Performance → Advanced → Virtual memory`.

#### Configure `swappiness`

​	Another option available on Linux systems is to ensure that the sysctl value `vm.swappiness` is set to `1`. This reduces the kernel’s tendency to swap and should not lead to swapping under normal circumstances, while still allowing the whole system to swap in emergency conditions.

#### Enable `bootstrap.memory_lock`

​	Another option is to use [mlockall](http://opengroup.org/onlinepubs/007908799/xsh/mlockall.html) on Linux/Unix systems, or [VirtualLock](https://msdn.microsoft.com/en-us/library/windows/desktop/aa366895(v=vs.85).aspx) on Windows, to try to lock the process address space into RAM, preventing any Elasticsearch heap memory from being swapped out.

>Some platforms still swap off-heap memory when using a memory lock. To prevent off-heap memory swaps, [disable all swap files](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setup-configuration-memory.html#disable-swap-files) instead.

​	To enable a memory lock, set `bootstrap.memory_lock` to `true` in `elasticsearch.yml`:

```yml
bootstrap.memory_lock: true
```

> `mlockall` might cause the JVM or shell session to exit if it tries to allocate more memory than is available!



​	After starting Elasticsearch, you can see whether this setting was applied successfully by checking the value of `mlockall` in the output from this request:

```
curl -X GET "localhost:9200/_nodes?filter_path=**.mlockall&pretty"
```

​	If you see that `mlockall` is `false`, then it means that the `mlockall` request has failed. You will also see a line with more information in the logs with the words `Unable to lock JVM Memory`.

​	The most probable reason, on Linux/Unix systems, is that the user running Elasticsearch doesn’t have permission to lock memory. This can be granted as follows:

**`.zip` and `.tar.gz`**

​	Set [`ulimit -l unlimited`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setting-system-settings.html#ulimit) as root before starting Elasticsearch. Alternatively, set `memlock` to `unlimited` in `/etc/security/limits.conf`:

```
# allow user 'elasticsearch' mlockall
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited
```

**RPM and Debian**

Set `MAX_LOCKED_MEMORY` to `unlimited` in the [system configuration file](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setting-system-settings.html#sysconfig) (or see below for systems using `systemd`).

**Systems using `systemd`**

​	Set `LimitMEMLOCK` to `infinity` in the [systemd configuration](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setting-system-settings.html#systemd).

​	Another possible reason why `mlockall` can fail is that [the JNA temporary directory (usually a sub-directory of `/tmp`) is mounted with the `noexec` option](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/executable-jna-tmpdir.html). This can be solved by specifying a new temporary directory for JNA using the `ES_JAVA_OPTS` environment variable:

```
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djna.tmpdir=<path>"
./bin/elasticsearch
```

​	or setting this JVM flag in the jvm.options configuration file.

# 黑马(start)

https://www.bilibili.com/video/BV1QE411w747?from=search&seid=5172679768015955203

# 简介

​	是一个开源的高扩展的分布式全文检索引擎。基于Lucene。

## 使用案例

* 2013年初，Github抛弃了Solr，采用ES来做PB级的搜索。
* 维基百科。
* SpringCloud
* 百度。
* 新浪
* 阿里。

## ES对其Solr

* Solr利用ZK进行分布式管理，而ES自带分布式协调管理功能。
* Solr支持更多格式的数据，ES只支持Json文件格式。
* Solr官方提供的功能更多，ES更注重核心功能，高级功能有第三方插件提供。
* Solr在传统的搜索应用中表现好于ES，但在实时搜索效率明显低于ES。

# 安装启动

略 参考官方文档

# head插件安装及配置

​	https://github.com/mobz/elasticsearch-head

**安装nodejs&cnpm**

​	https://nodejs.org/zh-cn/download/

​	https://blog.csdn.net/a1104258464/article/details/52273774



​	在elasticsearch-head下

```
cnpm install
cnpm install -g grunt server
grunt server
```

​	如果不能成功连接到es，修改eslaticsearch.yml 增加以下两条配置

```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

![](https://pic.imgdb.cn/item/60d14fc2844ef46bb2e04419.jpg)

# ES相关概念

## 概述

​	ES是面向文档的，意味着可以存储整个对象或文档，然而，它不仅是存储，还会索引每个文档的内容使其可以被搜索。在ES中，可以对文档进行索引、搜索、排序、过滤。ES比传统关系型数据库如下:

![](https://pic.imgdb.cn/item/60d1505a844ef46bb2e3cb63.jpg)

## 核心概念

### 索引 index

​	一个索引就是拥有几分相似特征文档的集合。一个索引由一个名字大来表示(小写字母)，并且要对于这个索引中的稳定进行索引，搜索，更新、删除的时候都要用到这个名字。集群中可以定义任意多的索引。

### 类型 type

​	索引中可以定义一个到多个类型。类型是索引的逻辑分类/分区，语义由自己来定。

​	通常，会为具有一组相同字段的文档定义一个类型。

### 字段Field

​	相当于是数据库的字段，对文档数据根据不同属性进行分类标识

### 映射 mapping

​	是处理数据的方式和规则方面做一些限制。比如某个字段的数据类型、默认值、r分析器、是否被索引等等，这些都是映射里面可以设置的，其它就是处理es里面数据的一些规则设置也叫做映射，按着最优规则处理数据对性能提升很大，因此才需要建立映射，并且需要如何建立映射才能对性能更好。

### 文档 document

​	是一个可被索引的基础信息单元。文档以Json格式来表示，而Json是一个到处存在的互联网数据交互格式。

​	在一个index/type下可以存储任意多的文档。注意，尽管一个文档，物理上存在于一个索引中，文档必须被索引/赋予一个索引的type

### 接近实时 NRT

​	ES是一个接近实时的平台，这意味着，从索引一个文档到这个文档能够被搜索有一个轻微的延迟(通常是1秒内)。

### 集群 cluster

​	集群共同持有整个的数据，病一起提供索引和搜索功能。一个集群由一个唯一的名字表示，默认是`elasticsearch`。名字非常重要，借点只能通过指定某个集群的名字，来加入这个集群。

### 节点 node

​	一个节点是集群中的一个服务器，作为集群的一部分，可以存储数据，参与集群的索引和搜索功能。一个节点由一个名字来表示，默认情况下，是一个随机的漫威漫画角色名字，名字会在启动时赋予节点。

​	一个节点可以通过配置集群名称来加入一个制定的集群。如果网络中启动了若干个节点，并且假定它们能够互相发现彼此，它们将会自定地形成并加入到集群中。

### 分配和复制shards&replicas

![](https://pic.imgdb.cn/item/60d1549d844ef46bb2fe58d8.jpg)

# 使用postman操作

## postman创建索引

* 简单创建

![](https://pic.imgdb.cn/item/60d15a94844ef46bb225d762.jpg)

* 设置请求体

es7 type默认取消了type

```json
{
  "mappings": {
    "properties": {
      "id": {
        "type": "long",
        "store": true,
        "index": false
      },
      "title": {
        "type": "text",
        "store": true,
        "index": true,
        "analyzer": "standard"
      },
      "content": {
        "type": "text",
        "store": true,
        "index": true,
        "analyzer": "standard"
      }
    }
  }
}
```

需要话

在 PUT请求中的链接中添加**?include_type_name=true**即可。

```
{
  "mappings": {
    "article": {
      "properties": {
        "id": {
          "type": "long",
          "store": true,
          "index": false
        },
        "title": {
          "type": "text",
          "store": true,
          "index": true,
          "analyzer": "standard"
        },
        "content": {
          "type": "text",
          "store": true,
          "index": true,
          "analyzer": "standard"
        }
      }
    }
  }
}
```



![](https://pic.imgdb.cn/item/60d15e86844ef46bb240a55e.jpg)

## 使用postman设置mapping映射

![](https://pic.imgdb.cn/item/60d15e5e844ef46bb23fa04a.jpg)

## 使用postman删除索引库

![](https://pic.imgdb.cn/item/60d15ef2844ef46bb243630a.jpg)

## 创建文档document

```
POST http://192.168.170.200:9200/blog/article/1
```

```json
{
    "id": 1,
    "title": "ES简介",
    "content": "是一个开源的高扩展的分布式全文检索引擎。基于Lucene。"
}
```

建议id和主键一致

![](https://pic.imgdb.cn/item/60d174a3844ef46bb2e0cd8f.jpg)

## 删除文档

```
DELETE http://192.168.170.200:9200/blog/article/1
```

![](https://pic.imgdb.cn/item/60d17580844ef46bb2e89e06.jpg)

## 修改文档

​	更新原理是先删除再增加，所以根据id直接执行添加操作即可

```
POST http://192.168.170.200:9200/blog/article/1
```

* 第一次body

```json
{
    "id": 1,
    "title": "ES简介",
    "content": "是一个开源的高扩展的分布式全文检索引擎。基于Lucene。"
}
```

* 第二次body

```json
{
    "id": 1,
    "title": "新添加",
    "content": "新添加"
}
```

## 根据关键词查询

```
POST http://192.168.170.200:9200/blog/article/_search
```



```
{
    "query": {
        "term": {
            "title": "新"
        }
    }
}
```

标准分析器分词中文是一个一个字

```
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 0.6931471,
        "hits": [
            {
                "_index": "blog",
                "_type": "article",
                "_id": "1",
                "_score": 0.6931471,
                "_source": {
                    "id": 1,
                    "title": "新添加",
                    "content": "新添加"
                }
            }
        ]
    }
}
```

## 查询文档-querystring查询

```
POST http://192.168.170.200:9200/blog/article/_search
```

```json
{
    "query": {
        "query_string": {
            "default_field": "title",
            "query": "添加"
        }
    }
}
```

# IK分词器和ElasticSearch集成使用

## 上述查询存在问题分析

​	在进行字符串查询时，我们发现搜索"搜索服务器"和"钢索"都可以搜索到数据:

​	而在进行词条查询时, 我们搜索"搜索"却没有搜索到数据:

​	究其原因是ElasticSearch的标准分词器导致的，当我们创建索引时，字符使用的是标准分词器:

```json
{
  "mappings": {
    "properties": {
      "id": {
        "type": "long",
        "store": true,
        "index": false
      },
      "title": {
        "type": "text",
        "store": true,
        "index": true,
        "analyzer": "standard"
      },
      "content": {
        "type": "text",
        "store": true,
        "index": true,
        "analyzer": "standard"
      }
    }
  }
}
```

查询分析结果

```
POST http://192.168.170.200:9200/_analyze
```

请求体

```json
{
    "analyzer": "standard",
    "text": "variously spelled wildcard or wild-card, also known as at-large berth&pretty=true"
}
```

分词结果

```json
{
    "tokens": [
        {
            "token": "variously",
            "start_offset": 0,
            "end_offset": 9,
            "type": "<ALPHANUM>",
            "position": 0
        },
        {
            "token": "spelled",
            "start_offset": 10,
            "end_offset": 17,
            "type": "<ALPHANUM>",
            "position": 1
        },
        {
            "token": "wildcard",
            "start_offset": 18,
            "end_offset": 26,
            "type": "<ALPHANUM>",
            "position": 2
        },
        {
            "token": "or",
            "start_offset": 27,
            "end_offset": 29,
            "type": "<ALPHANUM>",
            "position": 3
        },
        {
            "token": "wild",
            "start_offset": 30,
            "end_offset": 34,
            "type": "<ALPHANUM>",
            "position": 4
        },
        {
            "token": "card",
            "start_offset": 35,
            "end_offset": 39,
            "type": "<ALPHANUM>",
            "position": 5
        },
        {
            "token": "also",
            "start_offset": 41,
            "end_offset": 45,
            "type": "<ALPHANUM>",
            "position": 6
        },
        {
            "token": "known",
            "start_offset": 46,
            "end_offset": 51,
            "type": "<ALPHANUM>",
            "position": 7
        },
        {
            "token": "as",
            "start_offset": 52,
            "end_offset": 54,
            "type": "<ALPHANUM>",
            "position": 8
        },
        {
            "token": "at",
            "start_offset": 55,
            "end_offset": 57,
            "type": "<ALPHANUM>",
            "position": 9
        },
        {
            "token": "large",
            "start_offset": 58,
            "end_offset": 63,
            "type": "<ALPHANUM>",
            "position": 10
        },
        {
            "token": "berth",
            "start_offset": 64,
            "end_offset": 69,
            "type": "<ALPHANUM>",
            "position": 11
        }
    ]
}
```

## IK分词器简介

​	IKAnalyzer是一个开源，基于Java语言开发的轻量级的中文分词工具包。最初，以Lucene为应用主体的，结合分词和文法分析算法的中文分词组件。新版本的IKAnalyzer3.0发展为面向Java的公用分词组件，独立Lucene项目，同时提供了对Lucene的默认优化实现。

​	IK分词器3.0的特性如下:

1. 采用了特有的"正向迭代最细粒度切分算法"，具有60万词/秒的高速处理能力。
2. 采用了多子处理器分析模式, 支持英文字母，数字，中文词汇等分词处理。
3. 对中英文联合支持不是很好，在这方面的处理比较麻烦，需再做一次查询，同时是支持个人词条的优化的词典存储，更小的内存占用。
4. 支持用户词典扩展定义。
5. 针对Lucene全文检索优化查询分析器IKQueryAnalyzer; 采用其一分析算法优化查询关键字的搜索排列组合，能极大的提高Lucene检索的命中率。

## ES集成IK分词器

### 安装

https://github.com/medcl/elasticsearch-analysis-ik/releases

下载后解压jar包到es的plugins目录下的ik-analyzer

启动时会扫描并启动plugins

### IK分词器测试

​	提供了两个分词算法ik_smart和ik_max_wrod

​	smart为最少切分，ik_max_word为最细粒度划分

1) 最小切分

```
POST http://192.168.170.200:9200/_analyze
{
    "analyzer": "ik_smart",
    "text": "我们是共产主义接班人"
}
```

返回值

```java
{
    "tokens": [
        {
            "token": "我们",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "是",
            "start_offset": 2,
            "end_offset": 3,
            "type": "CN_CHAR",
            "position": 1
        },
        {
            "token": "共产主义",
            "start_offset": 3,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 2
        },
        {
            "token": "接班人",
            "start_offset": 7,
            "end_offset": 10,
            "type": "CN_WORD",
            "position": 3
        }
    ]
}
```

2）最细粒度划分

```
POST http://192.168.170.200:9200/_analyze
{
    "analyzer": "ik_max_word",
    "text": "我们是共产主义接班人"
}
```

```json
{
    "tokens": [
        {
            "token": "我们",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "是",
            "start_offset": 2,
            "end_offset": 3,
            "type": "CN_CHAR",
            "position": 1
        },
        {
            "token": "共产主义",
            "start_offset": 3,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 2
        },
        {
            "token": "共产",
            "start_offset": 3,
            "end_offset": 5,
            "type": "CN_WORD",
            "position": 3
        },
        {
            "token": "主义",
            "start_offset": 5,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 4
        },
        {
            "token": "接班人",
            "start_offset": 7,
            "end_offset": 10,
            "type": "CN_WORD",
            "position": 5
        },
        {
            "token": "接班",
            "start_offset": 7,
            "end_offset": 9,
            "type": "CN_WORD",
            "position": 6
        },
        {
            "token": "人",
            "start_offset": 9,
            "end_offset": 10,
            "type": "CN_CHAR",
            "position": 7
        }
    ]
}
```

# ES集群

暂略

# Java集成ES

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-create-index.html

## 创建索引库

1. 创建Java工程
2. 添加jar包

```xml
<dependencies>
        <!-- https://mvnrepository.com/artifact/org.elasticsearch/elasticsearch -->
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.13.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.elasticsearch.client/elasticsearch-rest-high-level-client -->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.13.2</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
            <scope>provided</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.8.0-M1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

3. 编写测试方法实现创建索引库
   * 创建Settings对象，主要配置集群的名称
   * 创建一个客户端Client对象
   * 使用client对象创建一个索引库
   * 关闭client

```java
@Test
    public void createIndexTest() throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
                new HttpHost("192.168.170.200", 9200, "http")));
        // 创建request
        CreateIndexRequest request = new CreateIndexRequest("twitter");
        // index设置
        request.settings(Settings.builder()
                .put("index.number_of_shards", 3)
                .put("index.number_of_replicas", 2)
        );
        // index mapping
        request.mapping(
                "{\n" +
                        "  \"properties\": {\n" +
                        "    \"message\": {\n" +
                        "      \"type\": \"text\"\n" +
                        "    }\n" +
                        "  }\n" +
                        "}",
                XContentType.JSON);
        // 另外一种形式的设置mapping
//        Map<String, Object> message = new HashMap<>();
//        message.put("type", "text");
//        Map<String, Object> properties = new HashMap<>();
//        properties.put("message", message);
//        Map<String, Object> mapping = new HashMap<>();
//        mapping.put("properties", properties);
//        request.mapping(mapping);
//        XContentBuilder builder = XContentFactory.jsonBuilder();
//        builder.startObject();
//        {
//            builder.startObject("properties");
//            {
//                builder.startObject("message");
//                {
//                    builder.field("type", "text");
//                }
//                builder.endObject();
//            }
//            builder.endObject();
//        }
//        builder.endObject();
//        request.mapping(builder);
        // 别名
        request.alias(new Alias("twitter_alias").filter(QueryBuilders.termQuery("user", "kimchy")));

        // 以上配置等同于
//        request.source("{\n" +
//                "    \"settings\" : {\n" +
//                "        \"number_of_shards\" : 1,\n" +
//                "        \"number_of_replicas\" : 0\n" +
//                "    },\n" +
//                "    \"mappings\" : {\n" +
//                "        \"properties\" : {\n" +
//                "            \"message\" : { \"type\" : \"text\" }\n" +
//                "        }\n" +
//                "    },\n" +
//                "    \"aliases\" : {\n" +
//                "        \"twitter_alias\" : {}\n" +
//                "    }\n" +
//                "}", XContentType.JSON);
        // 可选参数
        // 等待所有节点返回的过期时间
        request.setTimeout(TimeValue.timeValueMinutes(2));
        // 主节点的过期时间
        request.setMasterTimeout(TimeValue.timeValueMinutes(1));
        // The number of active shard copies to wait for before the create index API returns a response, as an int
        request.waitForActiveShards(ActiveShardCount.from(2));
        // The number of active shard copies to wait for before the create index API returns a response,
        // as an ActiveShardCount
        request.waitForActiveShards(ActiveShardCount.DEFAULT);
        // 同步请求
        CreateIndexResponse createIndexResponse = client.indices().create(request, RequestOptions.DEFAULT);
        // 异步请求
//        ActionListener<CreateIndexResponse> listener =
//                new ActionListener<CreateIndexResponse>() {
//
//                    @Override
//                    public void onResponse(CreateIndexResponse createIndexResponse) {
//
//                    }
//
//                    @Override
//                    public void onFailure(Exception e) {
//
//                    }
//                };
//        client.indices().createAsync(request, RequestOptions.DEFAULT, listener);
        // 获取结果
        boolean acknowledged = createIndexResponse.isAcknowledged();
        boolean shardsAcknowledged = createIndexResponse.isShardsAcknowledged();
        client.close();
    }
```

## 更新Mapping

```java
@Test
    public void updateMapping() throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
                new HttpHost("192.168.170.200", 9200, "http")));

        PutMappingRequest request = new PutMappingRequest("twitter");
        // 设置mapping source
        request.source(
                "{\n" +
                        "  \"properties\": {\n" +
                        "    \"message\": {\n" +
                        "      \"type\": \"text\"\n" +
                        "    }\n" +
                        "  }\n" +
                        "}",
                XContentType.JSON);
        // 其他形式设置mapping 略

        // 同步请求
        AcknowledgedResponse putMappingResponse = client.indices().putMapping(request, RequestOptions.DEFAULT);
        // 异步请求
        ActionListener<AcknowledgedResponse> listener =
                new ActionListener<AcknowledgedResponse>() {
                    @Override
                    public void onResponse(AcknowledgedResponse putMappingResponse) {

                    }

                    @Override
                    public void onFailure(Exception e) {

                    }
                };
        client.indices().putMappingAsync(request, RequestOptions.DEFAULT, listener);
        boolean acknowledged = putMappingResponse.isAcknowledged();
    }
```

## 更新文档

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-document-update.html

支持多种更新方式，在这里只展示一种简单的

```java
   @Test
    public void updateDoc() throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
                new HttpHost("192.168.170.200", 9200, "http")));
        // index
        // doc id
        UpdateRequest request = new UpdateRequest(
                "blog",
                "3");
        String jsonString = "{\n" +
                "  \n" +
                "    \"id\": 3,\n" +
                "    \"title\": \"新添加\",\n" +
                "    \"content\": \"新添加\"\n" +
                "  \n" +
                "}\n";
        request.doc(jsonString, XContentType.JSON);
        Map<String, Object> parameters = new HashMap<>();
//
        // 如果不存在就插入就 写一个upsert
//        request.upsert(jsonString, XContentType.JSON);
//        Map<String, Object> jsonMap = new HashMap<>();
//        jsonMap.put("id", 1);
//        jsonMap.put("title", "新添加");
//        jsonMap.put("content", "新添加");

//        UpdateRequest request = new UpdateRequest("blog/article", "1")
//                .doc(jsonMap);
//        XContentBuilder builder = XContentFactory.jsonBuilder();
//        builder.startObject();
//        {
//            builder.timeField("updated", new Date());
//            builder.field("reason", "daily update");
//        }
//        builder.endObject();
//        UpdateRequest request = new UpdateRequest("posts", "1")
//                .doc(builder);
//        UpdateRequest request = new UpdateRequest("posts", "1")
//                .doc("updated", new Date(),
//                        "reason", "daily update");

        // 可选参数 略

        // 同步请求
        UpdateResponse updateResponse = client.update(
                request, RequestOptions.DEFAULT);

        // 异步请求
//        ActionListener listener = new ActionListener<UpdateResponse>() {
//            @Override
//            public void onResponse(UpdateResponse updateResponse) {
//
//            }
//
//            @Override
//            public void onFailure(Exception e) {
//
//            }
//        };
//        client.updateAsync(request, RequestOptions.DEFAULT, listener);

        // 结果
//        String index = updateResponse.getIndex();
//        String id = updateResponse.getId();
//        long version = updateResponse.getVersion();
//        if (updateResponse.getResult() == DocWriteResponse.Result.CREATED) {
//
//        } else if (updateResponse.getResult() == DocWriteResponse.Result.UPDATED) {
//
//        } else if (updateResponse.getResult() == DocWriteResponse.Result.DELETED) {
//
//        } else if (updateResponse.getResult() == DocWriteResponse.Result.NOOP) {
//
//        }
    }
```

## 多条件搜索(简单)

```java
	@Test
    public void SearchTest() throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
                new HttpHost("192.168.170.200", 9200, "http")));
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));
        SearchRequest searchRequest = new SearchRequest("test-time");
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.filter(QueryBuilders.rangeQuery("authDate").
                from("2018-02-05 14:25:00").to("2021-06-23 17:27:00"));
//        boolQueryBuilder.filter(QueryBuilders.matchQuery("id", 3));
        sourceBuilder.query(boolQueryBuilder);
        searchRequest.source(sourceBuilder);
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
        System.out.println(Arrays.toString(searchResponse.getHits().getHits()));
        client.close();
    }
```



# SpringDataES

