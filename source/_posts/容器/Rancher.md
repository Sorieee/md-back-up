# 安装

通用配置

```sh
yum update -y

#安装Docker version 19.03.6
yum install -y yum-utils

# 阿里云地址（国内地址，相对更快）
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sudo yum remove -y docker-ce docker-ce-cli containerd.io
sudo yum install -y docker-ce-19.03.6 docker-ce-cli-19.03.6 containerd.io
docker --version #查看版本

#  docker 服务
systemctl start docker && systemctl enable docker
systemctl status docker		
 
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "registry-mirrors": ["https://mr63yffu.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload

#系统准备
systemctl stop firewalld && systemctl disable firewalld
sed -i 's/enforcing/disabled/g' /etc/selinux/config; setenforce 0


# 若都显示 0 则表示关闭成功
vim /etc/sysctl.conf                           # 永久生效
#  修改 vm.swappiness 的修改为 0
vm.swappiness=0

sysctl -p                                            # 使配置生效
swapoff -a
free -m
#enable br_netfilter
sudo modprobe br_netfilter
lsmod | grep br_netfilter

# setting iptables
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    br_netfilter
EOF
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
 
```

rancher安装

```sh
#安装rancher
sudo docker run --privileged -d --restart=unless-stopped -p 80:80 -p 443:443 -v /root/rancher:/var/lib/rancher/ rancher/rancher:v2.0.0
```

## 修改host

rancher机器需要修改host

比如

192.168.160.211 node1

其余node节点需要修改hostname, 比如node1

https://yanue.net/post-142.html
