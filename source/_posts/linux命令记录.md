---
title: linux命令记录
date: 2020-04-05 23:30:01
tags:
---

centOs8

```
# 重载所有ifcfg或route到connection（不会立即生效）
nmcli c reload
```

打包、解压

```
　　.tar 
　　解包：tar xvf FileName.tar 
　　打包：tar cvf FileName.tar DirName 
　　（注：tar是打包，不是压缩！） 
　　——————————————— 
　　.gz 
　　解压 1：gunzip FileName.gz 
　　解压2：gzip -d FileName.gz 
　　压缩：gzip FileName 
　　.tar.gz 和 .tgz 
　　解压：tar zxvf FileName.tar.gz 
　　压缩：tar zcvf FileName.tar.gz DirName 
```

重命名

```
mv filenameA filenameB
```

ip

```
ifconfig -a
```

上传

从官网下载，用lrzsz上传到/opt文件夹，并解压

```
sudo apt-get install lrzsz
rz
```

防火墙

```
# 查看防火墙是否开启
firewall-cmd --state
systemctl stop firewalld.service
```

virtualBox扩容

https://www.cnblogs.com/forbeat/p/5001240.html

# 重启网络

```
# 重载所有ifcfg或route到connection（不会立即生效）
nmcli c reload ifcfg-xxx
# 重载指定ifcfg或route到connection（不会立即生效）
nmcli c load /etc/sysconfig/network-scripts/ifcfg-ethX
nmcli c load /etc/sysconfig/network-scripts/route-ethX
# 立即生效connection，有3种方法
nmcli c up ethX
nmcli d reapply ethX
nmcli d connect ethX


```

