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

## 基本操作

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

## string 类型数据的扩展操作

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

  

  

  