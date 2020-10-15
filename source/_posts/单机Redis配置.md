---
title: Redis配置
date: 2020-08-01 20:09:13
tags: [Redis]
---

# 内存分配控制

## vm.overcommit_memory

![](https://pic.imgdb.cn/item/5f255c4214195aa59422fe50.jpg)



Redis建议将vm.overcommit_memory设置为1

获取

```
# cat /proc/sys/vm/overcommit_memory
0
```

设置：

```
echo "vm.overcommit_memory=1" >> /etc/sysctl.conf
sysctl vm.overcommit_memory=1
```

**最佳实践**

* Redis设置合理的maxmemory，保证机器有20%~30%的限制内存

conf文件中设置(根据实际情况设置)

注意，在64bit系统下，maxmemory设置为0表示不限制Redis内存使用，在32bit系统下，maxmemory不能超过3GB；

```
# maxmemory <bytes>
maxmemory 268435456
```

* 设置vm.overcommit_memory=1，防止极端情况下会造成fork失败

# **持久化**

## RDB



```
save 900 10
```

**RBD 15分钟如果有10条数据修改就同步一次**





```
stop-writes-on-bgsave-error yes
```

说明：后台存储过程中如果出现错误现象，是否停止保存操作

经验：通常默认为开启状态。

```
dbfilename dump.rdb
	说明：设置本地数据库文件名，默认值为dump.rdb
	经验：通常设置为dump-端口号.rdb
dir
	说明：设置存储.rdb的路径
	经验：通常设置成存储空间较大的目录中，目录名称data
rdbcompression yes
	说明：设置存储至本地数据库时是否压缩数据，默认为yes，采用LZF压缩
	经验：通常默认为开启状态，如果设置为no，可以节省CPU运行时间，但会使存储的文件变大（巨大）
rdbchecksum yes
	说明：设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程中进行。
	经验：通常默认为开启状态，如果设置为no，可以节约读写性过程约10%时间消耗，但存储一定的数据损坏风险。
```



## AOF

```
#开启AOF功能
appendonly yes

#每一秒写入aof文件，并完成磁盘同步
appendfsync everysec
#AOF之持久化文件名，默认文件名为appendonly.aof, 建议配置为appendonly-端口号.aof
appendfilename filename


#如果该参数设置为no，是最安全的方式，不会丢失数据，但是要忍受阻塞的问题。如果设#置为yes呢？这就相当于将appendfsync设置为no，这说明并没有执行磁盘操作，只是写入了缓冲区，因此这样并不会造成阻塞（因为没有竞争磁盘），但是如果这个时候redis挂掉，就会丢失数据。丢失多少数据呢？在linux的操作系统的默认设置下，最多会丢失30s的数据
no-appendfsync-on-rewrite yes	

#比如说上一次AOF rewrite 是100mb，然后就会接着100mb继续写AOF的日志，如果发现增长的比例，超过了之前的100% 也就是200mb，就可能会去触发一次rewrite；
但是此时还要去跟min-size，64mb去比较，200mb > 64mb，才会去触发rewrite。

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

```



# Redis安全

* 密码要足够复杂64字节以上
* 不使用默认端口（可以一定程度上降低被攻击概率）
* 使用非root启动



# 删除策略

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

采用方案（待讨论）

挑选最近使用次数最少的数据进行淘汰

```
maxmemory-policy volatile-lfu
```



# 其他配置

```
# 设置服务器以守护进程的方式运行
daemonize yes

# 设置端口号
port 6374

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

