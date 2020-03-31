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

