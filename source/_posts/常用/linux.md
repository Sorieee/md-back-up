# 开启ssh端口号

1. 进入/etc/[ssh](https://so.csdn.net/so/search?q=ssh&spm=1001.2101.3001.7020)/sshd_config目录，修改以下内容增加端口号33

```
Port 22
Port 1024
```

2. 重启ssh服务： service sshd restart