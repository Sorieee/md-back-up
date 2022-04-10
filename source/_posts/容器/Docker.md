# 安装

参考

http://doc.ruoyi.vip/ruoyi-cloud/cloud/dokcer.html#%E5%9F%BA%E6%9C%AC%E4%BB%8B%E7%BB%8D

```sh
yum install -y yum-utils
# 官方地址（比较慢）
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
	
# 阿里云地址（国内地址，相对更快）
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
yum install docker-ce docker-ce-cli containerd.io


docker version # 查看Docker版本信息

systemctl start docker		# 启动 docker 服务:
systemctl enable docker		# 开机启动 docker 服务:
systemctl status docker		# 查看 docker 服务状态
# 配置镜像
touch /etc/docker/daemon.json
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "registry-mirrors": ["https://mr63yffu.mirror.aliyuncs.com"]
}
EOF

systemctl daemon-reload
```

# 架构概念

​	通过下图可以得知，`Docker`在运行时分为`Docker引擎（服务端守护进程）`和`客户端工具`，我们日常使用各种`docker命令`，其实就是在使用`客户端工具`与`Docker`引擎进行交互。

![](https://oscimg.oschina.net/oscnet/up-216ccca6be8f28927f914b667ad9b2dad74.JPEG)