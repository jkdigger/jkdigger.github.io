---
layout:     post
title:      n1 OMV通过docker安装openwrt
subtitle:   导入op镜像+docker镜像管理
date:       2019-11-26 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
---



## 前言

n1安装了omv开启samba作为nas使用，omv作为开源的nas解决方案稳定可靠。再通过docker安装openwrt作为旁路由使用。

## 1. 在omv中开启docker

### 01 启用docker服务

- 系统→ OMV-Extras 
- 勾选 [Docker CE]

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126132212.webp)

 

- 点击[编辑]→[启用]→点击[保存]

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126132308.webp)

> 稍等片刻，安装完成，即可看见已启动一栏变绿 

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126132330.webp)

### 02 安装docker-gui插件

- 系统→插件
- 点击 [检查] ，等待插件列表出现
- 搜索 docker→勾选 docker-gui→点击安装

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126132348.webp)

>  之后在服务下就会多一个Docker标签 

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126132612.webp)

### 03 开启docker-gui

- 服务→Docker
- 勾选 [enable the plugin]


>  docker镜像的默认位置是`/var/lib/docker`下.

## 2. 安装openwrt

### 01 拉取op镜像

- ssh登入n1

- 拉取openwrt镜像

```
docker pull kanshudj/n1-openwrtgateway:r9.10.1
```

- 设置网卡：开启网卡混杂模式

```
ip link set eth0 promisc on
modprobe pppoe
```

- 设置网关：新建好一个与主路由网段一样的给旁路由用的网络

```
docker network create -d macvlan --subnet=192.168.2.0/24 --gateway=192.168.2.1 -o parent=eth0 macnet
```

>  其中2.0,2.1是你主路由的网段。
>

- 运行docker

```
docker run --restart always -d --network macnet --privileged kanshudj/n1-openwrtgateway:r9.10.1 /sbin/init
```

###  02 配置 Openwrt 

- 查看容器id

```
docker ps
```

-  `95e8335a2034`为容器id，就可以进入容器的命令行 

```
docker exec -it 95e8335a2034 sh 
```

- 编辑LAN

```
vi /etc/config/network
```

- 找到3处包含192.168.X.X的地方，输入i进入编辑，同样将**x**改为你主路由的网段

```
config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.X.2'
        option netmask '255.255.255.0'
        option gateway '192.168.X.1'
        option dns '114.114.114.114 223.5.5.5'
```

> 网关改成自定义的

- 按`Esc`，输入`:wq!`保存并退出编辑
-  应用网络更改

```
/etc/init.d/network restart
```

-  退出容器命令行 

```
exit
```

### 03 登入openwrt

- 浏览器输入 `192.168.x.2` 即可进入openwrt

> 用户名 root
>
> 密码 password

## 3. 更新openwrt

### 01 手动上传op镜像

- ssh连接n1
- 手动上传`.gz`格式的镜像到`/root/`目录

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126162353.PNG)

- 导入镜像

```
docker import openwrt-armvirt-64-default-rootfs.tar.gz openwrt:R9
```

>  **docker import+文件名称+镜像名称:标签，标签可以自定义修改** 

- 运行docker

```
docker run --restart always -d --network macnet --privileged openwrt:R9 /sbin/init
```

> 由于之前部署过openwrt，因此无需设置网卡和设置网关，直接运行docker即可。

###  02 配置 Openwrt 

- 查看容器id

```
docker ps
```

-  95e8335a2034为容器id，就可以进入容器的命令行 

```
docker exec -it 95e8335a2034 sh 
```

- 编辑LAN

```
vi /etc/config/network
```

- 找到3处包含192.168.X.X的地方，输入i进入编辑，同样将**x**改为你主路由的网段

```
config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.X.2'
        option netmask '255.255.255.0'
        option gateway '192.168.X.1'
        option dns '192.168.X.1 114.114.114.114'
```

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126162504.PNG)

- 按`Esc`，输入`:wq!`保存并退出编辑
- 应用更改

```
/etc/init.d/network restart
```

-  退出容器命令行 

```
exit
```

### 03 登入openwrt

- 浏览器输入 `192.168.x.2` 即可进入openwrt

> 用户名 root
>
> 密码 password

## 4. Docker镜像管理

- 登入OMV
- 服务→容器
- [Docker容器]：点击选中不用的容器→点击[停止]→点击[删除]

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126163359.PNG)

- [Docker镜像]：点击选中不用的镜像→点击[删除]

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126163531.PNG)

## 参考资料

- [OpenMediaVault(OMV)配置Docker](https://www.jianshu.com/p/5b0eacc61527)
- [梁非凡n1玩法](https://github.com/real-pin1group/3000web/wiki/playerdev_n1)