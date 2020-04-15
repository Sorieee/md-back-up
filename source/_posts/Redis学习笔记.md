---
title: Redis学习笔记
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

   ![image-20200331210328872](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200331210328872.png)


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
//移除制定数据
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

![image-20200411142137761](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200411142137761.png)

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

![image-20200411144452744](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20200411144452744.png)

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

![](https://raw.githubusercontent.com/Ewasong/picBed/master/20200414211346.png)

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



# Redis规范

https://blog.csdn.net/glx490676405/article/details/79580748

