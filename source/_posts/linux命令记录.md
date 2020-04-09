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

