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

## 什么是容器

Simply put, a container is a sandboxed process on your machine that is isolated from all other processes on the host machine. That isolation leverages [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504), features that have been in Linux for a long time. Docker has worked to make these capabilities approachable and easy to use. To summarize, a container:

* is a runnable instance of an image. You can create, start, stop, move, or delete a container using the DockerAPI or CLI.
* can be run on local machines, virtual machines or deployed to the cloud.
* is portable (can be run on any OS)
* Containers are isolated from each other and run their own software, binaries, and configurations.

## 什么是容器镜像

When running a container, it uses an isolated filesystem. This custom filesystem is provided by a **container image**. Since the image contains the container’s filesystem, it must contain everything needed to run an application - all dependencies, configuration, scripts, binaries, etc. The image also contains other configuration for the container, such as environment variables, a default command to run, and other metadata.



# 常用命令

```sh
# 运行
docker run -dp 80:80 docker/getting-started

# 构建镜像
docker build -t getting-started .
```

