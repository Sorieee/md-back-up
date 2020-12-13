---
title: Centos虚拟机安装
date: 2020-12-13 12:48:57
tags: [Linux, CentOs]
---

​	操作环境：Win10 + VirtualBox + CentOs8

​	安装过程略，主要是一些命令。

# 防火墙

## systemctl

```sh
systemctl unmask firewalld                    #执行命令，即可实现取消服务的锁定
systemctl mask firewalld                    # 下次需要锁定该服务时执行
systemctl start firewalld.service               #启动防火墙  
systemctl stop firewalld.service                #停止防火墙  
systemctl reloadt firewalld.service             #重载配置
systemctl restart firewalld.service             #重启服务
systemctl status firewalld.service              #显示服务的状态
systemctl enable firewalld.service              #在开机时启用服务
systemctl disable firewalld.service             #在开机时禁用服务
systemctl is-enabled firewalld.service          #查看服务是否开机启动
systemctl list-unit-files|grep enabled          #查看已启动的服务列表
systemctl --failed                                     #查看启动失败的服务列表            
```

## firewall-cmd

```sh
firewall-cmd --state                         #查看防火墙状态  
firewall-cmd --reload                        #更新防火墙规则  
firewall-cmd --state                         #查看防火墙状态  
firewall-cmd --reload                        #重载防火墙规则  
firewall-cmd --list-ports                    #查看所有打开的端口  
firewall-cmd --list-services                 #查看所有允许的服务  
firewall-cmd --get-services                  #获取所有支持的服务  

#区域相关
firewall-cmd --list-all-zones                    #查看所有区域信息  
firewall-cmd --get-active-zones                  #查看活动区域信息  
firewall-cmd --set-default-zone=public           #设置public为默认区域  
firewall-cmd --get-default-zone                  #查看默认区域信息  
firewall-cmd --zone=public --add-interface=eth0  #将接口eth0加入区域public

#接口相关
firewall-cmd --zone=public --remove-interface=eth0       #从区域public中删除接口eth0  
firewall-cmd --zone=default --change-interface=eth0      #修改接口eth0所属区域为default  
firewall-cmd --get-zone-of-interface=eth0                #查看接口eth0所属区域  

#端口控制
firewall-cmd --add-port=80/tcp --permanent               #永久添加80端口例外(全局)
firewall-cmd --remove-port=80/tcp --permanent            #永久删除80端口例外(全局)
firewall-cmd --add-port=65001-65010/tcp --permanent      #永久增加65001-65010例外(全局)  
firewall-cmd  --zone=public --add-port=80/tcp --permanent            #永久添加80端口例外(区域public)
firewall-cmd  --zone=public --remove-port=80/tcp --permanent         #永久删除80端口例外(区域public)
firewall-cmd  --zone=public --add-port=65001-65010/tcp --permanent   #永久增加65001-65010例外(区域public) 
firewall-cmd --query-port=8080/tcp    # 查询端口是否开放
firewall-cmd --permanent --add-port=80/tcp    # 开放80端口
firewall-cmd --permanent --remove-port=8080/tcp    # 移除端口
firewall-cmd --reload    #重启防火墙(修改配置后要重启防火墙)
```

## 改用iptables.service

```sh
yum install iptables-services           #安装iptables  
systemctl stop firewalld.service        #停止firewalld  
systemctl mask firewalld.service        #禁止自动和手动启动firewalld  
systemctl start iptables.service        #启动iptables
systemctl start ip6tables.service       #启动ip6tables  
systemctl enable iptables.service       #设置iptables自启动  
systemctl enable ip6tables.service      #设置ip6tables自启动  
```

# 无法联网问题

​	在`/etc/sysconfig/network-scripts`目录下存放着网卡的配置文件，文件名称是`ifcfg- 网卡名称`。

## 1.修改配置文件

​	设置网络时首先打开配置文件，配置文件默认如下所示，如果使用dhcp自动获取ip，只需将`ONBOOT=no`修改为`ONBOOT=yes`即可。

```sh
# 网卡配置文件按默认配置
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=e4987998-a4ce-4cef-96f5-a3106a97f5bf
DEVICE=ens33
ONBOOT=no  #如果使用dhcp分配ip的话，只需要将这里no改为yes，然后重启网络服务就行
```

​	如果需要配置静态ip，则按照以下修改方法修改。IPADDR的前三位和宿主机前三位一致，网关地址和宿主机一致。(注意是wifi还是网线的网卡)。

```sh
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static   #将dhcp修改为stati表示使用静态ip
DEFROUTE=yes
IPADDR=192.168.128.129   #设置IP地址
NETMASK=255.255.255.0    #设置子网掩码
GATEWAY=192.168.128.1    #设置网关
DNS1=114.114.114.114     #设置dns
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=e4987998-a4ce-4cef-96f5-a3106a97f5bf
DEVICE=ens33
ONBOOT=yes  #将no改为yes
```

## 2. 重启网络服务

​	使用**`nmcli c reload`**命令重启网络服务。有时候需要重启虚拟机。



# 安装后又回到安装界面

​	正常关机回到VirtualBox管理器，点击设置。

​	将系统下的启动顺序调整一下，**硬盘**上移到第一个，此时即可正常启动centos8。

# 图形界面

开机以命令模式启动，执行：

```
systemctl set-default multi-user.target
```

开机以图形界面启动，执行：

```
systemctl set-default graphical.target
```



# 上传下载文件

```sh
#安装软件
yum -y install lrzsz
#下载
sz xx.txt
#上传
rz
```

