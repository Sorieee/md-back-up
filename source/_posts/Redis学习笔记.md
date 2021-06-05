---
REPLCONF ACK命令做确认title: Redis学习笔记
date: 2020-03-29 00:08:27
tags: redis
---

学习视频:https://www.bilibili.com/video/BV1CJ411m7Gc

# 背景

奥运订票，春运订票，淘宝访问量比较大等网站与服务。

问题现象:

* 海量用户
* 高并发

罪魁祸首--关系型数据库

* 性能瓶颈：磁盘IO性能低下
* 扩展瓶颈：数据关系复杂，扩展性差，不便于大规模集群

解决思路：

* 降低磁盘IO次数，越低越好 —— 内存存储
* 去除数据间关系，越简单越好 —— 不存储关系，仅存储数据

# NoSql

Nosql: 即Not-OnlySQL(泛指非关系型数据库)，作为关系数据库的补充。

作用：应对基于海量用户和海量数据前提下的数据处理问题。

特征：

* 可扩容，可伸缩
* 大数据量下高性能
* 灵活的数据模型
* 高可用

常见的NoSql数据库：

* Redis
* memcahe
* HBase
* MongoDB

解决方案（电商常见):

1. 商品基本信息(MySql)

2. 商品附加信息(MongoDB)

3. 图片信息(分布式文件系统)

4. 搜索关键字(ES、Lucene、Solr)

5. 热点信息

   * 高频

   * 波段性

     —> Redis, memche,tair

   ![](https://pic.imgdb.cn/item/5edb0099c2a9a83be5b9e04b.jpg)


# Redis介绍

概念: Redis（Remote Dictionary Server)是用C语言开发的一个开源的高性能键值对(key-value)数据库。
## Redis的应用

* 为热点信息加速查询
* 任务队列，如秒杀，抢购，排队等
* 即使信息查询，如各排行榜、各类网站访问统计、公交到站信息、在线人数信息、设备信息等
* 时效性信息控制，如验证码控制，投票控制等
* 分布式数据共享，如分部是集群架构中的session分离
* 消息队列
* 分布式锁

# Redis下载

Linux版(适用于企业开发)

 * Redis高级开始使用
* 以4.0版本作为主版本

Windows版本(适合0基础学习)
* Redis入门使用
* 以3.2版本作为主版本

   官网:https://redis.io/

# 服务启动

window环境下

**启动服务端**

在redis文件下执行以下命令

```
redis-server.exe redis.windows.conf
```

**启动客户端**

```
redis-cli
```



# Redis基本操作

* 功能性指令
* 清除屏幕指令
* 帮助信息查阅
* 退出指令

**信息添加**

功能: 设置key value的数据

命令

```
set key value
//eg: set name sorie
```

**信息查询**

功能: 根据key查询对应的value，如果不存在，返回空(nil)

命令

```
get key
//get name
```

**清屏**

```
clear
```

**帮助信息**

```
help
//或者
help <command>
```

**退出**

```
quit
exit
<ESC>
```

# Redis数据类型

业务数据的特殊性

**作为缓存使用**

1. 原始业务功能
   * 秒杀
   * 618活动
   * 双11活动
   * 排队购票
2. 运营平台监控到的突发高频访问数据
   * 突发时政要问，被强势关注围观
3. 高频，复杂的统计数据
   * 在线人数
   * 投票排行榜

**附加功能**

系统功能优化或升级

* 单服务器升级集群
* Session管理
* Token管理

## 数据类型(5种常用)

* string   String
* hash    HashMap
* list     LinkedList
* set     HashSet
* sorted_set  TreeSet

# Redis数据存储格式

* redis自身是一个Map，其中所有的数据都是采用key:value的形式存储
* 数据类型指的是存储的数据的类型，也就是value部分的类型，key部分永远都是字符串

## string 类型

* 存储的数据：单个数据，最简单的数据存储类型，也是最常用的数据存储类型。
* 存储的格式：一个存储空间保存一个数据
* 存储内容：通常使用字符串，如果字符串以整数形式展示，可以作为数据操作使用

###  基本操作

```
// 添加修改数据
set key value

// 获取数据
get key

// 删除数据
del key

// 添加修改多个数据
mset key1 value1 key2 value2

// 获取多个数据
mget key1 key2

// 获取数据字符个数(字符长度)
strlen key

// 追加信息到原始信息后(如果存在追加，不存在则新建)
append key value
```

### 单数据操作VS多数据操作

* 多数据操作可以节省数数据传输时间
* 需要考虑数据量，比如50条可能mset好，如果是1亿条，就要重新考虑，可能需要分组执行

### string 类型数据的扩展操作

**业务场景**

大型企业应用中，分表是基本操作，使用多张表存储同类型数据，但是对于的主键id必须保证统一性，不能重复。Oracle数据库具有sequence设定，可以解决该问题，但是MySQL并不具有类似的机制，那么如何解决？

**解决方案**

* 设置数值数据增加/减少指定范围的值

```
incr key
incrby key increment
incrbyfloat key increment
decr key
decrby key increment
```

**string作为数值操作**

* string在redis内部存储默认就是一个字符串，当遇到增减类操作incr, decr时会转成数值型进行计算

* redis所有操作都是原子性的，采用单线程处理所有业务，命令是一个一个执行的，因此无需考虑并发带来的数据影响

* 注意：按数字进行操作时，如果原始数据不能转换为数值或者超宇了redis上限范围，将报错9223372036854775807（java Long最大值,Long.MAX_VALUE)


Tips 1:
* redis用于控制数据库主键id，为数据库表主键提供生成策略，保证数据库表的主键唯一性

* 此方案适用于所有数据库，且支持数据库集群

**业务场景2**

  “最强女生”启动海选投票，只能通过微信投票，每个微信号每4小时只能投1票

电商商家开启热门商品推荐，热门商品不能一直处于热门期，没中商品热门期维持3天，3天后自动取消热门。

新闻网站出现热点新闻，热点新闻最大特征是时效性，如何自动控制热点新闻的时效性。

**解决方案**

设置数据具有制定的生命周期

```
setex key seconds vlaue
psetex key milliseonds value
```



Tips 2：

* redis控制数据的生命周期，通过数据是否失效控制业务行为，适用于所有具有时效性限定控制的操作。

### string 类型数据操作注意事项

* 数据操作不成功的反馈与数据正常操作之前的差异
	1. 表示运行结果是否成功
		* (integer)0 -> false 失败
		* (interger)1 -> true 成功
	2. 表示运行结果值
		* (integer) 3 -> 3 3个
		* (integer)1 -> 1 1个
* 数据未获取到
	(nil) 等同于null
* 数据量最大存储量
	512MB
* 数据计算最大范围(java中long的最大值)
	9223372036854775807
### string 类型应用场景

**业务场景**

* 主页高频访问信息显示控制，例如新浪微博大V主页显示粉丝数与微博数量

```
user:id:123:fans -> 122128
user:id:123:blogs -> 6164
user:id:123:focuss -> 83
```

* 在redis以json个数存储打V用户数据，定时刷新(也可以使用hash类型)

Tips 3：

* redis应用于各种结构型和非结构型高热度数据访问加速

## hash 类型

**存储的困惑**

对象类数据的存储如果具有较频繁的更新需求会显得笨重

* 新的存储需求：对一系列存储的数据进行编组，方便管理，典型应用存储对象信息
* 需要的存储结构：一个存储空间保存多个键值对数据
* has类型：底层使用哈希表结构实现数据存储

key -> field -> value

**hash存储结构优化**

* 如果field数量较少，存储结构优化为类结构
* 如果field数量较多，存储结构使用HashMap结构

### hash类型数据的基本操作

```
// 添加修改数据
hset key field value

//获取数据
hget key field
hgetall key

//删除数据
hdel key field1 field2

// 添加修改多个
hmset key field1 value1 field2 value2

//获取多个数据
hmget key field1 field2

//获取hash表中字段的数量
hlen key

//获取hash表中是否存在指定字段
hexists key field
```

### hash类型数据的扩展操作

```
//获取hash表中的字段名或者字段值
hkeys key
hvals key
//设置制定字段数值数据增加制定范围的值
hincrby key field increment
hincrbyfloat key field increment
//存在不设置 不存在就设置
hsetnx key field value
```

### hash 类型数据操作的注意事项

* hash类型下的value只能存储字符串，不允许存储其他数据类型，不存在嵌套现象，如果数据未获取到，对应值为(nil)
* 每个hash可以存储2^32 -1 
* hash类型是非贴近对象的数据存储格式，并且可以灵活添加删除对象属性。但hash设计初衷不是为了存储大量对象而设计的，切记不可滥用，更不可以将hash对象列表使用
* hgetall操作可以获取全部属性，但是内部field过多，遍历整体数据效率就会很低，有可能成为数据访问的瓶颈

### hash类型应用场景

**购物车**

* 每条购物车中的商品信息保存成两条field

* field1专用于保存购买数量

  * 命名格式:商品id:nums
  * 保存数据:数值

* field2专用于保存购物车中的信息

  * 命名格式:商品id:info
  * 保存数据:json

  

其中field2可以把商品信息单独存储

**双11**

* 商家key作为value
* 将参与抢购的商品id作为field
* 将参与抢购的商品数量作为value
* 抢购时使用降值的方式控制产品数量

Tips 5:

* redis 应用于抢购，限购类，限量发放优惠券，激活码等业务数据存储设计

### string存对象和hash存对象

* string整存整取
* hash比较分离



## list 类型

* 数据存储需求：存储多个数据，并对数据进入存储空间的数据进行区分
* 需要的存储结构：一个存储空间保存多个数据，且通过数据可以体现进入顺序
* list类型：保存多个数据，底层使用双向链表存储结构实现。

### list 类型数据基本操作

```
//添加修改数据
lpush key value1 [value2].....
rpush key value1 [value2].....

//获取数据
lrange key start stop
lindex key index
llen key

//获取并移除数据
lpop key
rpop key
```

### list 类型数据扩展操作

```
//规定时间内获取并移除数据
blpop key1 [key2] timeout
brpop key1 [key2] timeout
```

### 业务场景1

微信朋友圈点赞，要求按点赞顺序显示点赞好友信息。

如果取消点赞，移除对应好友信息。

```
//移除指定数据
lrem key count value
```

Tips 6:

* redis应用于具有操作先后顺序的数据控制

### list 类型数据操作注意事项

* list中保存的数据都是string类型的，数据总容量是优先的，最多2^32-1个(4284967295)
* list具有索引的概念，但是操作数据时通常以队列的形式进行入队出队操作，或者以栈的形式进行出栈入栈操作
* 获取全部数据操作结束索引设置为-1
* list可以对数据进行分页操作，通常第一个的信息来自于list，第二页及更多的信息通过数据库的形式加载

### 业务场景 2

Twitter、微博中个人用户的关注列表需要按用户的关注顺序进行展示，粉丝列表需要将最新关注的粉丝列在前面

新闻

日志等

**解决方案**

* 依赖list的数据具有顺序的特征对信息进行管理
* 使用队列模型解决多路信息汇总合并的问题
* 使用栈模型解决最新消息的问题

Tips 7

* redis应用于最新的消息展示



## set类型

* 新的存储需求：存储大量数据，在查询方面提供更高的效率。
* 需要的存储结构：能够保存大量数据，高效的内部存储机制，便于查询

![](https://pic.imgdb.cn/item/5eca6548c2a9a83be50f0684.jpg)

### 基本操作

```
# 添加数据
sadd key member1 [member2]
# 获取全部数据
smembers key
# 删除数据
srem key member1 [member2]
# 获取数据总量
scard key
# 判断集合中是否包含指定数据
sismember key member
```

### 扩展操作

**业务场景**

每位用户首次使用今日头条会设置3项爱好，但是后期为了增加用户活跃度、兴趣点，必须让用户对其他信息类别逐渐产生兴趣，增加客户留存度。

**业务分析**

* 系统分析各个分类最新或者最热点信息条目并组织set集合
* 随机挑选其中部分消息
* 配合用户关注信息分类中的热点信息组织成展示的全信息集合

**解决方案**

```
# 随机获取元素集合中制定数量的数据
srandmember key [count]
# 随机获取集合中的某个数据并将该数据移出集合
spop key [count]
```

**业务场景**

共同关注，共同好友

```
# 求2两个集合的交、并、差集
sinter key1 [key2]
sunion key1 [key2]
sdiff key1 [key2]

# 求两个集合的交、并、差集并存储到指定集合中
sinterstore destination key1 [key2]
sunionstore destination key1 [key2]
sdiffstore destionation key1 [key2]

# 将制定数据从原始集合异动到目标集合
smove source destination member
```

### 注意事项

* 数据不重复
* set和hash结构相似，但是无法启用hash中存储值的空间

### 应用场景

* 权限判断
* 网站访问统计
* 黑白名单

## sorted_set

* 新的存储需求：数据排序有利于数据的有效展示，需要提供一种可以根据自身特征进行排序的方式
* 需要的结存结构：新的存储模型，可以保存可排序的数据
* sorted_set类型：在set的存储结构基础上添加可排序的字段

![](https://pic.imgdb.cn/item/5eca6570c2a9a83be50f3332.jpg)

### 基础操作

```
# 添加数据
zadd key score1 member1 [score2 member2]
# 获取全部数据
zrange key start stop [withscores]
zrevrange key start stop [withscores]
# 删除数据
zrem key member [member...]
# 按条件获取数据
zrangebyscore key min max [withscores] [limit]
zrevrangebyscore key min max [withscores] [limit]
# 条件删除数据
zremrangebyrank ke start stop
zremrangebyscore key min max

# 获取集合数据量
zcard key
zcount key min max
# 集合交并
zinterstore destination numkeys key [key ..]
zunionstore destination numkeys key [key ..]
```

### 扩展

```
# 获取数据对应的索引（排名）
zrank key member
zrevrank key member

# score值的获取与修改
zscore key member
zincrby key increment member
```

### 注意事项

* score保存的数据存储空间是64位，如果是整数范围是-9007199254740992~9007199254740992
* score保存的数据也可以使一个双精度double值，基于双精度浮点数的特征，可能会丢失经度，使用时要慎重
* sorted_set底层存储还是基于set结构的，因此数据不能重复，如果重复添加相同数据，score将被覆盖，保存最后一次修改的结果

```
# 当前系统时间
time
```

# 数据类型实践案例

## 业务背景

按此结算的服务控制

**解决方案**

* 设计计数器，记录调用次数，用于控制业务执行次数。以用户id作为key，使用次数作为value
* 在调用前获取次数，判断是否超过限定次数
  * 不超过的情况下，每次调用技术+1
  * 业务调用失败，计数-1
* 为计数器设置生命周期，例如1秒/分钟，自动清空周期内使用次数

**改良方案**

* 取消最大值判定，利用incr操作超过最大值抛出异常的形式
* 判断是否为nil
  * 如果是，设置Max-次数
  * 如果不是，计数+1
  * 调用失败，计数-1

* 遇到异常即+操作超过上限，视为使用达到上限。

Tips 16：

* redis应用于限时按次结算的服务控制

## 业务背景

微信接收消息顺序控制

**解决方案**

* 依赖list的数据具有顺序的特征对消息进行管理，将list作为栈使用

* 对置顶与普通会话分别创建独立的list分别管理

* 当某个list中接收到用户消息后，将消息发送方的id从list的一侧加入list（此处设定为左侧）

* 多个相同id发出消息反复入栈会出现问题，在入栈之前无论是否具有当前id的消息，先删除id

* 推送消息时，先推送置顶会话list，再推送普通会话list，推送完的list清除所有信息=69780-3245`1 12234780- 56=

  消息的数量，也就是微信用户对话数量采用计数器的思想另行记录，伴随list操作同步更新
  
  ![](https://pic.imgdb.cn/item/5eca6611c2a9a83be50fd750.jpg)

# 通用命令

## key基本操作

* key是一个字符串，通过key获取redis中保存的数据

```
# 删除key
del key
# 获取key是否存在
exists key
# 获取key类型
```

## key的扩展-时效性

```
# 为指定key设置有效期
expire key seconds
pexpire key milliseconds
expireat key timestamp
pexpireat key milliseconds-timestamp
# 获取key的有效时间
ttl key
pttl key
# 切换key从时效性转换为永久性
persist key
```

## key的扩展操作-查询

```
# 查询key
key parttern
```

查询模式规则

\* 匹配任意数量的字符      ？ 匹配任意一个字符    []匹配一个自定字符

keys * 查询所有

keys it* 查询所有以it开头

keys *heima 查询所有以heima结尾

keys ??heima 查询所有前面两个字符任意，后面以黑马结尾

keys user:? 查询任何以user:开头，最后一个字符结尾

keys u[st]er:1 查询所有以u开头，以er:1结尾，中间包含一个字符（s或t)

## key的其他操作

```
# 改名
rename key newkey  //会覆盖
renamenx key newkey //不会改
# 对所有key排序
sort 
# 其他key通用操作
help @generic
```

## db基本操作

key重复问题

* key由程序员定义
* redis在使用过程中，伴随着操作数据量的增加，会出现大量的数据以及对应的key
* 数据不区分种类，类别混杂在一起，极易出现重复或冲突

**解决方案**

* redis为每个服务提供有16个数据库，编号从0到15
* 每个数据库之间的数据相互独立

```
# 切换数据库
select index
# 其他操作
quit
ping
echo message
```

## db其他操作

```
# 数据异动
move key db
# 数据清除
dbsize
flushab
flushall
```



# Redis规范

https://blog.csdn.net/glx490676405/article/details/79580748

# Jedis简介

一个Java语言连接Redis工具、通道。

```java
@Test
public void testJedis() {
    //连接redis
    Jedis jedis = new Jedis("127.0.0.1", 6379);
    //操作redis
    jedis.set("name", "sorie");
    String name = jedis.get("name");
    System.out.println(name);
    //关闭连接
    jedis.close();
}
```

# Jedis常规操作

```java
@Test
public void testList() {
    Jedis jedis = new Jedis("127.0.0.1", 6379);
    jedis.lpush("list1", "a", "b", "c");
    jedis.rpush("list1", "x");
    List<String> list = jedis.lrange("list1", 0, -1);
    list.forEach(System.out::println);
    jedis.close();
}

@Test
public void testHash() {
    Jedis jedis = new Jedis("127.0.0.1", 6379);
    jedis.hset("hash1", "a1", "1");
    jedis.hset("hash1", "a2", "2");
    jedis.hset("hash1", "a3", "3");
    Map<String, String> map = jedis.hgetAll("hash1");
    for(Map.Entry<String, String> entry : map.entrySet()) {
        System.out.println(entry.getKey() + " " + entry.getValue());
    }
    jedis.close();
}
```

# Linux安装

如果make不成功可能是没有安装gcc 参考：

https://blog.csdn.net/yudianxiaoxiao/article/details/102525099

```
wget http://download.redis.io/releases/redis-5.0.8.tar.gz
tar zxvf redis-5.0.8.tar.gz
make
make install
```

## 指定端口运行

```
redis-server --port 6380
redis-server -p 6380
```

## 指定配置文件启动

```
redis-server redis-6379.conf
# 查看进程
ps -ef | grep redis-
# 杀死进程
kill -s 9 pid
```

# 持久化

## 持久化简介

​	利用永久性存储介质将数据进行保存，在特定时间将保存的数据进行恢复的工作机制称为持久化。

**为什么要进行持久化？**

​	防止数据的意外丢失，确保数据安全性。

**持久化过程保存什么？**

* 数据快照（RDB)
* 数据操作过程/日志(AOF)

## RDB

**命令**

```
save
```

**作用**

​	手动执行一次保存，会在设定的数据文件目录保存一个rdb数据

### save指令相关配置

**dbfilename dump.rdb**

​	说明：设置本地数据库文件名，默认值为dump.rdb

​	经验：通常设置为dump-端口号.rdb

**dir**

​	说明：设置存储.rdb的路径

​	经验：通常设置成存储空间较大的目录中，目录名称data

**rdbcompression yes**

​	说明：设置存储至本地数据库时是否压缩数据，默认为yes，采用LZF压缩

​	经验：通常默认为开启状态，如果设置为no，可以节省CPU运行时间，但会使存储的文件变大（巨大）

**rdbchecksum yes**

​	说明：设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程中进行。

​	经验：通常默认为开启状态，如果设置为no，可以节约读写性过程约10%时间消耗，但存储一定的数据损坏风险。

### 恢复

​	启动时恢复

### save指令工作原理

​	save执行时间很长怎么办？

​	注意：save指令的执行会阻塞当前redis服务器，直到当前RDB过程完成为止，有可能会造成长时间阻塞，线上环境不建议使用。

### 后台保存-bgsave

```
bgsave
```

作用

​	手动启动后台保存操作，但不是立即执行

**bgsave工作原理**

![](https://pic.imgdb.cn/item/5ea2fd18c2a9a83be5da6152.jpg)

注意：bgsave命令是针对save阻塞问题做的优化。Redis内部所有涉及到RDB操作都采用bgsave的方式。save命令可以放弃使用。

**bgsave指令相关配置**

**stop-writes-on-bgsave-error yes**

​	说明：后台存储过程中如果出现错误现象，是否停止保存操作

​	经验：通常默认为开启状态。

### 自动执行save

redis服务器基于条件执行，满足条件就保存。

**配置**

```
save second changes
```

**作用**

​	满足限定时间范围key的变化数量达到制定数量即进行持久化

**参数**

​	seond：监控时间范围

​	changes: 监控key的变化量

**位置**

​	在conf文件中进行配置

**范例**

​	save 900 1

​	save 300 10

​	save 60 10000

注意：

​	save配置要根据实际业务情况进行配置，频度过高或者过低都会出现性能问题，结果可能是灾难性的。

​	save配置中对second和changes设置通常具有互补对应关系，尽量不要设置包含性关系

​	save配置启动后执行的是bgsave



### 区别

![](https://pic.imgdb.cn/item/5ea30232c2a9a83be5de7ba8.jpg)

### rdb特殊启动形式

* 全量复制
* 服务器运行过程中重启

```
debug reload
```

* 关闭服务器时制定保存数据

```
shutdown save
```

**rdb优点**

* RDB是一个紧凑压缩的二进制文件，存储效率高
* RDB内部存储的是redis在某个时间点的快照，非常适合于数据备份，全量复制等场景。
* RDB恢复数据的速度要比AOF快很多
* 应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程服务机器中，用于灾难恢复。

**rdb缺点**

* RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据。
* bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能。
* Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象。

## AOF

* 不写全数据，仅记录部分数据

* 改记录数据为记录操作过程
* 对所有操作均进行记录，排除丢失数据的风险

### 概念

AOF(append only file)持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令，达到恢复数据的目的。与RDB相比可以简单描述为改记录数据为记录数据产生的过程。

AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式。

**写数据过程**

![](https://pic.imgdb.cn/item/5ec62803c2a9a83be57d5c0a.jpg)



**写数据策略**

* always（每次）

  ​	每次写入操作均同步到AOF文件中，数据零误差，性能较低

* everysec（每秒）

  ​	每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高

  ​	在系统突然宕机的情况下丢失1秒的数据

* no（系统控制）

  ​	由操作系统控制每次同步到AOF文件的周期，整体过程不可控

### AOF功能开启

* 配置

```
appendonly yes|no
```

* 作用

  是否开启AOF持久化功能，默认为不开启

* 配置

```
appendfsync always|everysec|no
```

* 作用

  AOF写数据策略

* 配置

```
appendfilename filename
```

* 作用

  AOF之持久化文件名，默认文件名为appendonly.aof, 建议配置为appendonly-端口号.aof

* 配置

```
appendfsync always|everysec|no
```

* 作用

  AOF写数据策略

### AOF重写

随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入了AOF重写机制压缩文件体积。AOF文件重写是将Redis进程的数据转化为写命令同步到新AOF文件的过程。简单说就是将同一个数据的若干个命令执行结果转化成最终结果数据对应的指令进行记录。

### AOF重写规则

* 进程内已超时的数据不再写入文件

* 忽略无效指令，重写时使用进程数据直接生成，这样新的AOF文件只保留最终数据写入命令

* 对同一条数据的多条写命令合并为一条命令

  为防止数据过大造成客户端缓冲区溢出，对list、set、hash、zset等数据，每条指令最多写入64个元素。

### AOF重写方式

* 手动重写

  ```
  bgrewriteaof
  ```



![](https://pic.imgdb.cn/item/5ec634c0c2a9a83be58fd552.jpg)

* 自动重写

```
auto-aof-rewrite-min-size size
auto-aof-rewrite-percentage percentage
```

* 自动重写触发比对参数(运行指令info Persistence获取具体信息)

```
aof_current_size
aof_base_size
```

* 自动重写触发条件

```
aof_current_size > auto-aof-rewrite-min-size
(aof_current_size - aof_base_size) / aof_base_size >= auto-aof-rewrite-percentage
```

**自动重写工作原理自动重写**

![](https://pic.imgdb.cn/item/5ec636f6c2a9a83be592e697.jpg)

## AOF VS RDB

![](https://pic.imgdb.cn/item/5ec6372ac2a9a83be59339bd.jpg)



![](https://pic.imgdb.cn/item/5ec6375fc2a9a83be5939011.jpg)

## 应用场景

![](https://pic.imgdb.cn/item/5ec638c1c2a9a83be5958728.jpg)

#  事务

Redis执行指令过程中，多条连续执行的指令被干扰，打断，插队。

按照添加顺序依次执行，中间不会被打断。



## 基本操作

开启事务

```
multi
```

作用

设定事务的开启位置，此指令执行后，后续所有的指令均加入到事务中



执行事务

```
exec
```

作用

设定事务结束位置，同时执行事务。与multi成对出现，成对使用。



取消事务

```
discard
```

作用

终止当前事务的定义，发送在multi之后，exec之前。

## 工作流程

![](https://pic.imgdb.cn/item/5ec63d3ac2a9a83be59c3b9b.jpg)



**注意事项**

语法错误

​	如果定义的事务中所包含的指令存在语法错误，整体事务所有命令均不执行，包括那些正确的指令



执行错误

​	正确运行的指令会执行，运行错误的指令不会被执行。

**手动进行事务回滚**

* 记录操作过程中被影响的数据之前的状态
  * 单数据：string
  * 多数据：hash、list、set、zset
* 设置指令恢复所有的被修改的项
  * 单数据：直接set（注意周边属性、时效性）
  * 多数据：修改对应值或整体克隆复制。

## 锁

* 对key添加监视锁，在执行exec前如果key发生了变化，终止事务的执行

```
watch key1 [key2]
```

* 取消对所有key的监视

```
unwatch
```

## 分布式锁

超卖问题

使用setnx 设置一个公共锁

```
setnx lock-key value
```

## 死锁

给锁添加时效性

# 删除策略

数据特征

Redis是一种内存级数据库，所有数据均存放在内存中，内存中的数据可以通过TTL指令获取其状态

* XX： 具有时效性的数据
* -1：永久有效的数据
* -2：已经过期的数据 被删除的数据 或未定义的数据

已经过期的数据真的删除了吗？

## 数据删除策略

* 定时删除
* 惰性删除
* 定期删除

**时效性数据存储格式**

![](https://pic.imgdb.cn/item/5ec7b8a8c2a9a83be5e5766f.jpg)



**数据删除策略的目标**

在内存占用和CPU占用之间寻找一种平衡，顾此失彼都会造成redis性能的下降，甚至引发服务宕机或内存泄露。

## 定时删除

* 创建一个定时器，当key设置有过期时间，且过期时间到达时，由定时任务立即执行对键的删除。

* 优点：节约内存，到时就删除，快速释放不必要的内存占用。
* 缺点：CPU压力很大，无论CPU此时负载量多高，均占用CPU，会影响redis服务器响应时间和指令吞吐量。



* 总结：用处理器性能换存储空间。

## 惰性删除

* 数据到达过期时间，不做处理。等下次访问该数据时删除。
* 优点：节约CPU性能，发现必须删除的时候才删除
* 缺点：内存压力很大，出现长期占用内存的数据。

## 定期删除

* Redis启动服务器初始化时，读取配置server.hz的值，默认为10
* 每秒钟执行server.hz次serverCron()
  - databasesCron()
    - activeExpireCycle()
* activeExpireCycle()对每个expires[*]逐一进行检测，每次执行250ms/server.hz
* 对某个expires[*]检测时，随机挑选W个key检测
  * key超时删除
  * 如果一轮中删除的key数量> w * 25%,循环该过程
  * 否则检查下一个expires[*],0-15循环。
  * W取值=ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP属性值
* 参数current_db用于记录activeExpireCycle()进入哪个expires[*]执行



* 周期性轮询redis库中时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度
* 特点1：CPU性能占用设置有峰值，检测频度可自定义设置。
* 特点2：内存压力不是很大，长期占用的冷数据会被持续处理。
* 总结：周期性抽查存储空间（随机抽查，重点抽查）

## 逐出算法

新数据进入redis，内存不足怎么办？

* Redis使用内存存储数据，在执行每一个命令前，会调用freeMemoryIfNeeded()检测内存是否充足。如果内存不满足新加入数据最低存储要求，redis要求临时删除一些数据为当前指令清理存储空间。清理数据的策略称为逐出算法。
* 注意：逐出策略的过程不是100%能够清理出足够的可用空间，如果不成功则反复执行。当对所有数据尝试完毕后，如果不能达到内存清理要求，将出现错误信息。

* 配置最大可用内存

```
maxmemory
```

占用物理内存的比例，默认值为0，表示不限制。生产环境中根据需求设定，通常在50%。

* 配置每次选取待删除的数据个数

```
maxmeory-samples
```

选取数据时并不会全库扫描，导致严重的性能消耗，降低读写性能，因此采用随机获取数据的方式作为待检测删除数据。

* 删除策略

```
maxmemory-policy
```

达到最大内存后，对被挑选出来的数据进行删除的策略。

* 检测易失数据（可能会过期的数据集server.db[i].expires)
  1. volatile-lru: 挑选最长时间没有使用的数据淘汰
  2. volatile-lfu: 挑选最近使用次数最少的数据进行淘汰
  3. volatile-ttl: 挑选刚要过期的数据淘汰
  4. volatile-random: 任意选择数据淘汰
* 检测全库数据（所有数据集server.db[i].dict)
  5. allkeys-lru: 挑选最长时间没有使用过的数据淘汰
  6. allkeys-lfu: 挑选最近使用次数最少的数据进行淘汰
  7. allkeys-random: 任意选择数据淘汰
* 放弃数据驱逐
  8. no-enviction(驱逐): 禁止驱逐(redis4.0中默认策略)，会引发OOM



可以使用info命令输出监控信息，查询缓存hit和miss次数，根据业务需求调优redis配置。



# 服务器配置

```
# 设置服务器以守护进程的方式运行
daemonize yes

# 绑定主机地址
bind 127.0.0.1

# 设置端口号
port 6379

# 设置数据库数量
databases 16

# 设置服务器以指定日志记录级别
loglevel debug|verbose|notice|warning

# 日志记录文件名 注意日志级别开发库设置为verbose，生产库设置为notice，简化日志输出量，降低写日志的IO频度
logfile 端口号.log

# 设置同一时间最大客户端连接数，默认无限制。当客户端连接达到上限，Redis会关闭新的连接
maxclients 0

# 客户端闲置等待最大时长，达到最大值后关闭连接，如需关闭该功能，设置为0
timeout 300

#导入并加载指定配置文件信息，用于快速创建redis公共配置较多的redis实例配置文件，便于维护
include /path/server-端口号.conf
```

# 高级数据结构

## BitMaps

问题：年度浏览量最低，月度浏览量最低，周浏览量最低

存储需求：

非黑即白的存储

其实就是string的二进制操作api

```
#获取指定key对应偏移量上的bit值
getbit key offset

# 获取指定key对应偏移量上的bit值,value只能是1,或者0
setbit key offset value

# 对指定key按位进行交、并、非、异或操作，并将结果保存到destKey中
bittop op destKey key1 [key2 ....]
	and, or, not, xor
# 统计key中1的数量
bitcount key [start end]
```

## HyperLogLog

统计独立UV

* 原始方案：set

  ​	* 存储每个用户的id

* 改进方案：BitMaps

   * 存储每个用户的状态bit

* 全新方案HyperLogLog

**基数**

* 基数是数据集去重后元素个数
* HyperLogLog是用来做基数统计的，运用了LogLog算法

LogLog算法

```
# 添加数据
pfadd key element [element2...]
# 统计数据
pfcount key [key...]
# 合并数据
pfmerge destkety sourcekey [sourcekey]
```

相关说明

* 用于进行基数统计，不是集合，不保存数据，只记录数量而不是具体数据
* 核心是基数估算算法，最终数值存在一定误差
* 误差范围：基数估计的结果是一个带有0.81%标准错误的近似值
* 耗空间极小，每个hyperloglog key占用12k的内存用于标记基数
* pdadd命令不是一次性分配12k内存使用，会随着基数的增加内存逐渐增大
* pfmerge命令合并后占用的存储空间为12K，无论合并之前的数据量多少

## GEO

附近的人

就是位置关系

```
# 添加坐标点
geoadd key longitude latitude member [longitude latitude member...]
# 获取坐标
geopos key member [member ...]
# 计算坐标点距离
geodist key member1 member2 [unit]
# 根据坐标求范围内的数据
georadius key longitude latitude radius m|km|ft|mi [withcoord][withdist][withhash][count count]
# 根据点求范围内数据
georadiusbymember key member radius m|km|ft|mi [withcoord][withdist][withhash][count count]
# 获取指定点对应坐标的hash值
geohash key member [member...]
```

# 集群

## 主从复制

**互联网三高架构**

* 高并发
* 高性能
* 高可用（追求99.999%的高可用)



单机redis的风险与问题

* 问题1：机器故障

  * 现象：硬盘故障、系统崩溃
  * 本质：数据丢失，很可能对业务造成灾难性打击
  * 结论：基本上会放弃使用redis

* 问题2：容量瓶颈

  * 现象：内存不足，从16G升级到64G，从64G升级到128G，无限升级内存
  * 本质：穷，硬件条件跟不上
  * 结论：放弃使用redis

* 结论

  ​	为了避免单点Redis服务器故障，准备多台服务器，相互连通。将数据复制多个副本保存在不同的服务器上，连接在一起，并保证数据是同步的。即使其中有一台服务器宕机，其他服务器依然可以继续提供服务，实现Redis高可用，同时实现数据冗余备份。



### 主从复制简介

![](https://pic.imgdb.cn/item/5ed3a554c2a9a83be5796588.jpg)

主从复制即将master中数据即时、有效地复制到slave中。

特征：一个master可以拥有多个slave，一个slave只对应一个master

职责：

* master
  * 写数据
  * 执行写操作时，将出现变化的数据自动同步到slave
  * 读数据（可忽略）
* slave
  * 读数据
  * 写数据（禁止）

### 主从复制作用

* 读写分离：master写，slave读，提高服务器的读写负载能力
* 负载均衡：基于主从结构，配合读写分离，由slave分担master负载，并根据需求的变化，改变slave的数量，通过多个从节点分担数据读取负载，大大提高Redis服务器并发量与数据吞吐量。
* 故障恢复：当master出现问题时，由slave提供服务，实现快速的故障恢复。
* 数据冗余：实现数据热备份，是持久化之外的一种数据冗余方式。
* 高可用基石：基于主从复制，构建哨兵模式与集群，实现Redis的高可用方案。



### 主从复制工作流程

3个阶段：

* 建立连接阶段
* 数据同步阶段
* 命令传播阶段

![](https://pic.imgdb.cn/item/5ed3a9d6c2a9a83be582e5f0.jpg)



### 阶段1：建立连接阶段

* 建立slave到master的连接，使master能够识别slave，并保存slave端口号

  1. slave给master发送指令：slaveof ip port

  2. master接收命令，响应对方

  3. slave保存master的IP与端口

     masterhost

     masterport

  4. slave根据保存的信息创建连接master的socket

  5. salve周期性发送命令：ping

  6. master响应pong

  7. slave发送指令：auth password

  8. master授权验证

  9. slave发送命令： replconf listening-port <port-number>

  10. master保存slave的端口号

  ![](https://pic.imgdb.cn/item/5ed3ab4cc2a9a83be585fd38.jpg)



### 搭建主从结构

* 方式1：客户端发送命令

  ```
  slaveof <masterip> <masterport>
  ```

* 方式2：启动服务器参数

  ```
  redis-server --slaveof <masterip> <masterport>
  ```

* 方式3： 服务器配置

  ```
  slaveof <masterip> <masterport>
  ```

  

* slave信息
  * master_link_down_since_seconds
  * masterhost
  * masterport
* master信息
  
  * slave_listening_port（多个）



断开连接

```
slaveof no one
```



#### 授权访问

```
#master设置密码
requirepass <password>

#master客户端发送命令设置密码
config set requirepass <password>
config get requirepass

#slave客户端发送命令设置密码
auth <password>

#客户端配置文件设置密码
masterauth <password>

#启动客户端设置密码
redis-cli -a <password>
```



### 阶段2： 数据同步阶段

* 在slave初连接master后，复制master中所有数据到slave
* 将slave的数据库状态更新成master当前的数据库状态

#### 工作流程

![](https://pic.imgdb.cn/item/5eda6329c2a9a83be5c76ef8.jpg)

#### 数据同步阶段master说明

1. 如果master数据量巨大，数据同步阶段应避免流量高峰期，避免造成master阻塞，影响业务正常执行

2. 复制缓冲区大小设置不合理，会导致数据溢出，如果全量复制周期太长，进行部分复制时发现数据已经存在丢失的情况，必须进行第二次全量复制，致使slave陷入死循环状态

```sh
repl-backlog-size 1mb
```

3. master单机内存占用主机内存的比例不应过大，建议使用50%-70%的内存，留下30%-50%内存用于执行bgsave命令和创建缓冲区。

#### 数据同步阶段slave说明

1. 为避免slave进行全量复制、部分复制时服务器响应阻塞或数据不同步，建议关闭此期间对外服务

```sh
slave-server-stale-data yes|no
```

2. 数据同步阶段，master发送slave信息可以理解master是slave的一个客户端，主动向slave发送命令
3. 多个slave同时对master请求数据同步，master发送的RDB文件增多，会对带宽造成巨大冲击，如果master带宽不足，那么数据同步需要根据业务需求，适量错峰。
4. slave过多时，建议调整为拓扑结构，从一主多从变为树状结构，中间节点既是master，也是slave，注意使用树状结构时，由于层级深度，导致深度越高的slave与最顶层master间数据同步延迟较大，数据一致性变差，应谨慎选择。

### 阶段3：命令传播阶段

* 当master数据库状态被修改后，导致主从服务器库状态不一致，此时需要让主从同步到一致的状态，同步的动作称为命令传播。
* master将接收到的数据变更命令发给slave，slave接收命令后执行命令。

#### 命令传播阶段的部分复制

* 命令传播阶段出现了断网现象

  * 网络闪断闪连  忽略
  * 长时间网络中断   全量复制
  * 短时间网络中断 部分复制
* 部分复制的三个核心要素
  * 服务器的运行id(runid)
  * 主服务器的复制积压缓冲区
  * 主从服务器的复制偏移量

#### 服务器运行ID(runid)

* 概念：服务器运行ID是每一台服务器每次运行的身份识别码，一台服务器多次运行可以生成多个运行id。

* 组成：运行id由40位字符组成，是一个随机的十六进制字符。

* 作用：运行id被用于在服务器间进行传输，识别身份。

  ​	如果想两次操作均对同一台服务器进行，必须每次操作携带对应的运行id，用于对方识别。

* 实现方式：运行id在每台服务器启动时自动生成的，master在首次连接slave时，会将自己的运行ID发送给slave，slave保存此ID，通过info server命令，可以查看节点runid。

### 复制缓冲区

* 概念：复制缓冲区，又名复制积压缓冲区，是一个先进先出的(FIFO)的队列，用于存储服务器执行过的命令，每次传播命令，master都会将传播的命令记录下来，并存储在复制缓冲区。
* 复制缓冲区默认数据存储空间大小是1M，由于存储空间大小是固定的，当入队的数量大于队列长度时，最先入队的元素会被弹出，而新元素会被放入队列。
* 由来：每台服务器启动时，如果开启AOF或被连接成为master节点，即创建复制缓冲区。
* 作用：用于保存master收到的所有指令（仅影响数据变更的指令，例如set，select)
* 数据来源：当master接收到主客户端指令时，除了将指令执行，会将该指令存储到缓冲区中。

![image-20200606101537760](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200606101537760.png)



#### 复制缓冲区内部工作原理

* 组成
  * 偏移量
  * 字节值
* 工作原理
  * 通过offset区分不同slave当前数据传播的差异。
  * slave和master都记录offset

![](https://pic.imgdb.cn/item/5edafd63c2a9a83be5b335c5.jpg)



​	

#### 主从服务器复制偏移量（offset）

* 概念：一个数字，描述复制缓冲区中的指令字节位置
* 分类
  * master复制偏移量：记录发送给所有slave的指令字节对应的偏移量（多个）
  * slave复制偏移量：记录slave接收master发送过来的指令字节对应的位置（一个）
* 数据来源
  * master端：发送一次记录一次
  * slave端：接收节次记录一次
* 作用：同步信息，比对master与slave的差异，当slave断线后，恢复数据使用

### 数据同步+命令传播阶段工作流程

![image-20200606103802566](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200606103802566.png)

### 心跳机制

* 进入命令传播阶段，master与slave间需要进行信息交换，使用心跳机制进行维护，实现双方连接保持在线。
* master心跳：
  * 指令：PING
  * 周期：由repl-ping-slave-period决定，默认10秒
  * 作用：判断slave是否在线
  * 查询：INFO replication             获取slave最后一次连接时间间隔，lag项维持在0或1为正常
* slave心跳任务：
  * 指令：REPLCONF ACK {offset}
  * 周期：1秒
  * 作用1：汇报slave自己的复制偏移量，获取最新的数据变更指令
  * 作用2：判断master是否在线

#### 心跳阶段的注意事项

* 当slave多数掉线或延迟过高时，master为保障数据稳定性，将拒绝所有信息同步操作

```sh
min-slaves-to-write 2
min-slaves-max-lag 8
```

slave数量少于2个，或者所有slave的延迟都大于等于8秒时，强制关闭master写功能，停止数据同步。

* slave数量由slave发送 REPLCONF ACK命令做确认
* slave延迟由slave发送REPLCONF ACK命令做确认

![](https://pic.imgdb.cn/item/5edb052fc2a9a83be5c34457.jpg)

### 主从复制常见问题

#### 频繁的全量复制（1）

伴随系统的进行，master的数据量会越来越大，一旦master重启，runid会发生变化，会导致全部slave的全量复制操作。



内部优化方案：

1. master内部创建master_replid变量，使用runid相同的策略生成，长度41位，并发送给所有slave

2. 在master关闭时指令指令，shutdown save，进行RDB持久化，将runid与offset保存到RDB文件中

* repl-id, repl-offset
* 通过redis-check_rdb命令可以查看该信息

3. master重启后加载RDB文件，恢复数据
4. 重启后，将RDB文件中保存repl-id与repl-offeset加载到内存中。
   * master_repl_id = repl      master_repl_offset = repl-offset
   * 通过info命令可以查看该信息

作用：

​	本机保存上次runid，重启后恢复该值，使所有slave认为还是之前的master



#### 频繁的全量复制（2）

* 问题现象
  * 网络环境不佳，出现网络中断，slave不提供服务
* 问题原因
  * 复制缓冲区过小，断网后slave的offset越界，触发全量复制
* 最终结果
  * slave反复进行全量复制
* 解决方案
  * 修改复制缓冲区大小

```sh
repl-backlog-size
```

* 建议设置如下：
  1. 测算从master到slave重连平均时长second
  2. 获取master每秒产生的写数据总量write_size_per_second
  3. 最优复制缓冲区空间 = 2 * second * write_size_per_second

#### 频繁的网络中断（1）

* 问题现象
  * master的CPU占用过高或 slave频繁断开连接
* 问题原因
  * slave每1秒发送REPLCONF ACK命令到master
  * 当slave接到慢查询(keys *, hgetall等)，会大量占用CPU性能
  * master每1秒调用复制定时函数replicationCron()，比对slave发现长时间没有进行响应。
* 最终结果
  * master各种资源（输出缓冲区、带宽、连接等）被严重占用
* 解决方案
  * 通过设置合理的超时时间，确认是否释放slave(默认60秒)

```
repl-timeout
```

该参数定义了超时时间的阈值（默认60）,超过该值，释放slave

#### 频繁的网络中断（2）

* 问题现象
  * slave与master连接断开
* 问题原因
  * master发送ping命令频度低
  * master设定超时时间较短
  * ping指令在网络中存在丢包
* 解决方案
  * 提高ping指令发送的频度

```
repl-ping-slave-period
```

超时时间repl-time的时间至少是ping指令频度的5-10倍，否则slave很容易被判定超时。

#### 数据不一致

* 问题现象

  * 多个slave获取相同数据不同步

* 问题原因

  * 网络信息不同步，数据发送有延迟

* 解决方案

  * 优化主从间网络环境，通常放置在同一个机房部署，如使用阿里云等云服务器要注意此现象

  * 监控主从节点延迟（通过offset）判断，如果slave延迟过大，暂时屏蔽程序对该slave的数据访问

    ```
    slave-serve-stale-data yes|no
    ```

    开启后仅响应info，slaveof等少数命令（慎用，除非对数据一致性要求很高。



## 哨兵

主机宕机

![](https://pic.imgdb.cn/item/5edb1cd6c2a9a83be5f46c3d.jpg)

* 将宕机的master下线
* 找一个slave作为master
* 通识所有的slave连接新的master
* 启动新的master和slave
* 全量复制xN+ 部分复制xN
* 谁来确认master宕机了？
* 找一个master？怎么找？
* 修改配置后，原始的master恢复了怎么办？

#### 哨兵简介

哨兵（sentinel）是一个分布式系统，用于对主从结构的没台服务器进行监控，当出现故障通过投票机制选择新的master并将所有slave连接到新的master。

![](https://pic.imgdb.cn/item/5edb1e1cc2a9a83be5f7dc43.jpg)

#### 哨兵的作用

* 监控

  ​	不断地检查master和slave是否正常进行

  ​	master存活检测、master与slave运行情况检测

* 通知（提醒）

  ​	当被监控的服务器出现问题时，向其他（哨兵间，客户端）发送通知

* 自动故障转移

  ​	断开master与slave连接，选取一个slave作为master，将其他slave连接到新的master，并告知客户端新的服务器地址。

注意：

​	哨兵也是一台redis服务器，只是不提供数据服务

​	通常哨兵配置数量为奇数（竞选活动可能打平）

#### 配置哨兵

* 配置一拖二的主从结构

* 配置三个哨兵（配置相同，接口不同）

  ​	参看sentinel.conf

* 启动哨兵

```
redis-sentinel setinel-端口号.conf
```



sentinel.conf

输入cat sentinel.conf | grep -v '#' | grep -v "^$"

```
daemonize no
pidfile /var/run/redis-sentinel.pid
logfile ""
dir /tmp
sentinel monitor mymaster 127.0.0.1 6379 2  # 最后一个参数是多少个哨兵认为这个master挂了，那么就挂了 一般设置为哨兵的一半+1
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1  # master挂了 一次有多少台数据复制
sentinel failover-timeout mymaster 180000  # 超时
sentinel deny-scripts-reconfig yes
```

复制一份

```sh
sed 's/26379/26381/g' sentinel-26379.conf > sentinel-26381.conf
```

启动后配置文件会发生变化

哨兵之间可以相互感知

关掉master后，哨兵投票推举一个slave出来当master

![](https://pic.imgdb.cn/item/5edb2610c2a9a83be50e7c48.jpg)



### 哨兵工作原理

#### 主从切换

* 哨兵在进行主从切换过程中经历了三个阶段
  * 监控
  * 通知
  * 故障转移

#### 阶段1：监控阶段

* 用于同步各个节点的状态信息

  * 获取各个sentinel的状态（是否在线）

  * 获取master的状态

    * master属性
      * runid
      * role：master
    * 各个slave的详细信息

  * 获取所有slave的状态（根据master中的slave信息）

    * slave属性
      * runid
      * role：slave
      * master_host，master_port
      * offset
      * .......

    ![](https://pic.imgdb.cn/item/5edb2763c2a9a83be5129274.jpg)



![](https://pic.imgdb.cn/item/5edb27ccc2a9a83be513c7f4.jpg)



#### 阶段2：通知阶段

一个哨兵获取到master的信息会传递给其他哨兵

![image-20200606132238835](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200606132238835.png)



#### 阶段3：故障转移阶段

发现故障

一个哨兵发现故障，

![](https://pic.imgdb.cn/item/5edb28adc2a9a83be5165723.jpg)

选举哨兵

![image-20200606132715357](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200606132715357.png)

* 服务器列表中挑选备选master

  * 在线的
  * 响应慢的pass掉
  * 与原来的master断开时间久的
  * 优先原则
    * 优先级
    * offset
    * runid（runid比较小的）
  * 发送指令（sentinel）
    * 向新的master发送 slaveof no one
    * 向其他slave发送salveof 新masterIP端口

  ![image-20200606132923564](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200606132923564.png)



#### 总结

* 监控
	* 同步信息
* 通知
  * 保持连通
* 故障转移
  * 发现问题
  * 竞选负责人
  * 优选新master
  * 新master上任，其他slave切换master，原master作为slave故障回复后连接

![](https://pic.imgdb.cn/item/5edb2b42c2a9a83be51e15c4.jpg)



## 集群

**业务发展过程中遇到的峰值瓶颈**

* redis提供的服务OPS可以达到10万/秒，当前OPS已经达到20万/秒
* 内存单击容量达到256G,当前业务需求内存容量1T
* 使用集群的方式可以快速解决上述问题



#### 集群架构

* 集群就是使用网络将若干台计算机联通起来，并提供统一的管理方式，使其对外呈现单机的服务效果。

#### 集群作用

* 分散单台服务器的访问压力，实现负载均衡
* 分散单台服务器的存储压力，实现可扩展性
* 降级单台服务器宕机带来的业务灾难

![](https://pic.imgdb.cn/item/5edb2d6ec2a9a83be5246230.jpg)



#### 集群结构设计

**数据结构设计**

* 通过算法设计，计算出key应该保存的位置
* 将所有的存储空间计划切割成16384份，每台主机保存一部分
  * 每份代表一个存储空间，不是一个key的保存空间
* 按key按照计算出的结果放到对应的存储空间

![](https://pic.imgdb.cn/item/5edb2e7fc2a9a83be5275694.jpg)



* 增强可扩展性

![](https://pic.imgdb.cn/item/5edb2e5cc2a9a83be526f4b3.jpg)

#### 集群内部通讯设计

* 各个数据库相互通信，保存各个库中槽的编号数据
* 一次命中，直接返回
* 一次未命中，告知具体位置

![](https://pic.imgdb.cn/item/5edb2f06c2a9a83be528bb53.jpg)



#### cluster集群搭建

cluster配置redis-conf

```sh
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout  10000 #企业用的时候时间会比较长
```

然后复制出来很多个

启动三主三从

要基于ruby和rubygem

```sh
./redis-trib.rb create --replicas 1 127.0.0.1:6379 # replicas后一个master连n个slave的标记
```

![](https://pic.imgdb.cn/item/5edb3136c2a9a83be52ece6e.jpg)

![](https://pic.imgdb.cn/item/5edb3177c2a9a83be52f80d6.jpg)



#### cluster设置与获取数据

![](https://pic.imgdb.cn/item/5edb3217c2a9a83be53144c1.jpg)

![](https://pic.imgdb.cn/item/5edb3233c2a9a83be5319da5.jpg)



#### 主从下线与主从切换

从离线直接离线，

主离线标记离线，从slave推举一个主出来。

如果原主又上线，只能变成slave



#### cluster配置

```sh
#设置加入cluster，成为其中的节点
cluster-enabled yes
#cluster配置文件名，该文件属于自动生成，仅用于快速查找文件并查找文件内容
cluster-config-file nodes-6379.conf
# 节点服务响应超时时间，用于判定该节点是否下线或切换为从节点
cluster-node-timeout  10000 #企业用的时候时间会比较长
# master连接slave最小数量
cluster-migration-barrier <count>
```

#### cluster节点操作命令

```sh
# 查看集群节点信息
cluster nodes
# 进入一个从节点redis，切换其主节点
cluster replicate <master-id>
# 发现一个新节点，新增主节点
cluster meet ip:port
# 忽略一个没有slot的节点
cluster forget <id>
# 手动故障转移
cluster failover
```



## 缓存预热


"宕机"

1. 请求数据较高
2. 主从之间吞吐量较大，数据同步操作频度高



**解决方案**

前置准备工作：

1.日常例行统计数据访问记录，统计访问频度比较高的热点数据

2.利用LRU数据删除策略，构建数据留存队列

​		例如:storm和kafka配合

准备工作:

3.将统计结果中的数据分类，根据级别，redis优先加载级别较高的热点数据

4.利用分布式多服务器同时进行数据读取，提高数据加载过程

实施：

1. 使用脚本程序固定触发缓存预热过程
2. 如果条件允许，使用CDN，效果会更好

总结：

缓存预热就是系统启动前，提前将相关的缓存数据直接加载到缓存系统。避免在用户请求的时候，先查询数据库，然后在将数据缓存的问题。用户直接查询的就是缓存的数据。

## 缓存雪崩

数据库服务器崩溃(1)

1. 忽然数据库连接量激增
2. 应用服务器无法及时处理请求
3. 大量408，500错误页面出现
4. 客户反复刷新页面获取数据
5. 数据库崩溃
6. 应用服务器崩溃
7. 重启服务器无效
8. Redis服务器崩溃
9. Redis集群崩溃
10. 重启数据库后再次被瞬间流量放到

问题排查

1. 较短时间内，缓存中较多的key集中过期
2. 此周期请求访问过期的数据，redis未命中，redis向数据库获取数据
3. 数据库同时接收到大量的请求无法及时处理
4. Redis大量请求被积压，开始出现超时现象
5. 数据库流量激增，数据库崩溃
6. 重启后仍然面对缓存中无数据可用的问题
7. Redis服务器资源被严重占用，Redis数据库服务器崩溃
8. Redis集群呈现崩溃，集群瓦解
9. 应用服务器无法及时得到数据响应请求，来自客户端的请求数量越来越多，应用服务器崩溃
10. 应用服务器，redis，数据库全部重启，效果不理想。

解决方案（道）

1. 更多的页面静态化处理

2. 构建多级缓存架构

   Nginx缓存+redis缓存+ehcache缓存

3. 检测mysql严重耗时业务进行优化

   对数据库的瓶颈排查：例如超时查询、耗时较高的业务等

4. 灾难预警机制

   监控redis服务器指标性能

   * CPU占用、CPU使用率
   * 内存容量
   * 查询平均响应时间
   * 线程数

5. 限流、降级

   短时间内牺牲一些客户体验，限制一部分请求访问，降低应用服务器压力，待业务低速运转后，再逐步开放访问

解决方案（术）

1. LRU和LFU切换

2. 数据有效策略调整

   * 根据业务数据有效期进行分类错峰，A类90分钟，B类80分钟，C类70分钟
   * 过期时间使用固定时间+随机值的形式，稀释集中到期的key的数量

3. 超热数据使用永久key

4. 定期维护（自动+人工）

   ​	对即将过期的数据做访量分析，确认是否延时，配合访问量统计，做热点数据的延时

5. 加锁

   ​	慎用！

总结

缓存雪崩就是瞬间过期数据量太大，导致对数据库服务器造成压力。如能够有效避免，过期时间集中，可以有效解决雪崩现象出现(约40%)，配合其他策略一起使用，并监控数据库运行数据，根据运行记录进行快速调整。

## 缓存击穿

数据库崩溃2

1. 系统平稳运行中
2. 数据库连接量瞬间激增
3. Redis服务器无大量key过期
4. Redis内存平稳，无波动
5. Redis服务器CPU正常
6. 数据库崩溃

问题排查

1. Redis中某个key过期，该key访问量巨大
2. 多个数据请求从服务器直接到Redis中，均未命中
3. Redis在短时间内发起了大量对数据库中同一数据的访问

问题

* 单个key高热数据
* key过期

解决方案（术)

1. 预先设定

   以电商为例，每个商家根据店铺等级，指定若干款主打产品，在购物节期间，加大此类信息的key的过期时长。

   注意：购物节不仅仅指当天，以及后续若干天，访问峰值呈现逐渐降低的趋势

2. 现场调整

   监控访问量，对自然流量激增的数据延长过期时间或者设置为永久key

3. 后台刷新数据

   启动定时任务，高峰期来临之前，刷新数据有效期，确保不丢失

4. 二级缓存

   设置不同失效时间，保障不会被同时淘汰就行

5. 加锁

   分布式锁，防止被击穿，但是要注意也是性能瓶颈，慎重。

总结

缓存击穿就是单个高热数据过期的瞬间，数据访问量较大，未命中redis后，发起了大量对同一数据的数据访问，导致对数据库服务器造成压力。应对策略应该在业务数据分析与预防方面进行，配合运行监控测试与即时调整策略，毕竟单个key 的过期监控难度较高，配合雪崩处理策略即可。

## 缓存穿透

数据库崩溃3

1. 系统平稳运行中
2. 应用服务器流量随时间增量较大
3. Redis服务器命中率随时间逐步降低
4. Redis内存平稳，内存无压力
5. Redis服务器CPU占用激增
6. 数据库服务器压力激增
7. 数据库崩溃

问题排查

1.Redis大面积出现未命中

2.出现非正常URL的访问

问题分析

* 获取的数据在数据库中也不存在，数据库查询未得到对应数据
* Redis获取到null数据未进行持久化，直接返回
* 下次此类数据到达重复上述过程
* 出现黑客攻击服务器

解决方案（术）

1. 缓存null

   ​	对查询结果为null的数据进行缓存（长期使用，定期清理），设定短时限，例如30-60秒，最高5分钟

2. 白名单策略

   * 提前预热各种分类数据id对应的bitmaps，id作为bitmaps的offset，相当于设置了数据白名单。当加载正常数据时，放行，加载异常数据时直接拦截（效率偏低）
   * 使用布隆过滤器（有关布隆过滤器命中问题对当前状况可以忽略）

3. 实施监控

   实时监控redis命中率（业务正常范围时，通常会有一个波动值）与null数据的占比。

   * 非活动时段波动：通常检测3-5倍，超过5倍纳入重点排查对象
   * 活动时段波动：通常检测10-50倍，超过50倍纳入重点排查对象

   根据倍数不同，启动不同的排查流程，然后使用黑名单进行防控（运营）

4. key加密

   ​	问题出现后，临时启动防灾业务key，对key进行业务层传输加密服务，设定校验程序，过来的key校验

   ​	例如每天随机分配60个加密串，挑选2-3个，混淆在页面数据id中，发现key不满足规则，驳回数据访问。

总结：

缓存穿透访问了不存在的数据，跳过了合法数据的redis数据缓存阶段，每次访问数据库，导致对数据库服务器造成压力。通常此类数据出现量是一个较低的值，当出现此类情况以毒攻毒，并及时报警。应对策略应该在临时预案防范方面多做文章。



无论是黑名单还是白名单，都是对整体系统的压力，警报解除后尽快移除。

## 性能指标

* 性能指标

| Name                    | Description              |
| ----------------------- | ------------------------ |
| latency                 | Redis响应一个请求的时间  |
| instanceous_ops_per_sec | 平均每秒处理的请求总数   |
| hit rate(calculated)    | 缓存命中率（计算出来的） |

* 内存指标

| Name                    | Description                                    |
| ----------------------- | ---------------------------------------------- |
| used_memory             | 已使用内存                                     |
| mem_fragmentation_ratio | 内存碎片率                                     |
| evicted_keys            | 由于最大内存限制被移除的key的数量              |
| blocked_clients         | 由于BLPOP,BRPOP，or BRPOPLPUSH而被阻塞的客户端 |

* 基本活动指标：Basic activity

| Name                       | Description                |
| -------------------------- | -------------------------- |
| connected_clients          | 客户端连接数               |
| connected_slaves           | Slave数量                  |
| master_last_io_seconds_ago | 最近一次主从交互之后的秒数 |
| keyspace                   | 数据库中key值的总数        |

* 持久化指标

| Name                       | Description                        |
| -------------------------- | ---------------------------------- |
| rdb_last_save_time         | 最后一次持久化保存到磁盘的时间戳   |
| rdb_chages_since_last_save | 自最后一次持久化以来数据库的更改数 |

* 错误指标

| Name                           | Description                      |
| ------------------------------ | -------------------------------- |
| rejected_connections           | 由于达到maxClients限制而被拒绝   |
| keyspace_misses                | Key值查找失败的次数（没有命中）  |
| master_link_down_since_seconds | 主从断开的持续时间（以秒为单位） |

## 性能监控

建议开发一些

* Cloud Insight Redis
* Prometheus
* Redis-stat
* Redis-faina
* RedsiLive
* zabbix

* 命令
  * benchmark
  * redis cli
    * monitor
    * slowlog



* 命令

```
redis-benchmark [-h] [-p] [-c] [-n<requests>] [-k]
```

* 范例

```
redis-benchmark
```

​	说明：50个连接，10000次请求对应的性能

* 范例2

```
redis-benchmark -c 100 -n 5000
```

说明：100个连接，5000次请求对应的性能

### monitor

* 命令

```
monitor
```

打印服务器调试信息

### slowlog

* 命令

```
slowlog [operator]
```

* get:获取慢查询日志
* len:获取慢查询日志条目数
* reset:重置慢查询日志



* 相关配置

```
slowlog-log-slower-than 1000 # 设置慢查询日志下限，单位：微秒
slowlog-max-len 100 # 设置慢查询命令对应的日志显示长度，单位：命令数
```



# 待补充内容

* redis6新特性介绍
* redis lua脚本使用
* redission
* redis配置文件
* pipeline

https://redis.io/topics/config

