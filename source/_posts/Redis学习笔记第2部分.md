---
title: Redis学习笔记第2部分
date: 2020-04-15 23:00:30
tags: redis

---

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

