---
title: centOs图形界面开启和关闭
date: 2020-04-04 15:20:50
tags: centOs
---

开机以命令模式启动，执行：

```
systemctl set-default multi-user.target
```

开机以图形界面启动，执行：

```
systemctl set-default graphical.target
```

