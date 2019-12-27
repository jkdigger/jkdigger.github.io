---
layout:     post
title:      n1 Armbian安装Docker版transmission
subtitle:   安装Docker版transmission
date:       2019-12-27 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
    - docker
---

## 安装Docker版transmission

### 01 ssh拉取镜像

- ssh连接n1
- 拉取镜像

```
docker pull linuxserver/transmission:arm64v8-latest
```

### 02 输入名字和镜像地址

- 浏览器登入docker管理页面
- Name输入

```
transmission
```

- Image输入

```
linuxserver/transmission:arm64v8-latest
```

### 03 设置Volumes

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191227200453.png)

- 点击Volumes
- **第一次**点击map additional volume
- container输入

```
/config
```

- 点击bind
- host输入

```
/etc/docker_transmission/config
```

- 点击writable

> ssh登入n1，在etc目录创建docker_transmission文件夹，给予权限 `chmod 777 /etc/docker_transmission/config`

- 第二次点击map additional volume
- container输入

```
/downloads
```

- 点击bind
- host输入

```
/docker/samb/data/tr
```

- 点击writable

> `/docker/samb/data`为映射的移动硬盘目录，tr为设定的下载文件夹



- 第三次点击map additional volume
- container输入

```
/watch
```

- 点击bind
- host输入

```
/etc/docker_transmission/watch
```

- 点击writable

> `/docker/samb/data`为映射的移动硬盘目录，存放watch文件夹下的种子会自动添加到tr下载。

### 04 设置env

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191227200615.png)

- 点击env
- 点击add environment variable
- 输入5组name和value

```
PUID → 1007   
PGID → 100  
TZ → Asia/Shanghai  
WEBUI_PORT → 9091
TRANSMISSION_WEB_HOME → /transmission-web-control/
```

> TRANSMISSION_WEB_HOME为ui界面，有三种 `/combustion-release/`, `/transmission-web-control/`, and `/kettu/` 。

### 05 设置Network ports configuration

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191227200843.png)

- 点击publish a new network port
- 输入host和container点击TCP或UDP

```
9091/9091 Tcp
9091/9091 Udp
51413/51413 Tcp
51413/51413 Udp
```

### 06 部署容器

![img](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191220201800.png)

- 点击 Deploy the container
- 之后去浏览器输入n1 的ip:9091即可登入tr，比如

```
192.168.2.225:9091
```

## 其他设置

### 01 端口映射

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191227203034.png)

>  *tr的默认下载端口是51413，需要在路由器进行端口转发* 

- 登入k2p 老毛子系统
- 外部网络（WAN）→端口转发（UPnP）→端口转发列表

```
服务名称 tr
源地址 *.*.*.*
端口范围 51413
内网ip  192.168.2.225（n1的ip）
本地端口 51413
协议 both
```

- 点击 加号+ 添加
- 应用本页面设置

### 02 公网ip

> pt下载需要有公网ip，并设置端口映射，才有上传速度。

查看是否有公网ip

- 打开百度，搜索ip地址，查看显示的ip地址1
- 登入路由器，查看WAN IPv4地址 2
- 如果二者一样，说明有公网ip，如果不一样说明没有公网ip

打电话给电信客服要求开通公网ip

- 电信 10000
- 联通 10010
- 移动 10086

> 要公网ip的理由装监控，如果不给就说要去工信部投诉。一般都会给，然后重启光猫，再查看公网ip是否设置成功。

### 03 检测端口是否开启

通过网站检测

-  [端口检测工具](http://tool.chinaz.com/port/)
- 输入查看到的公网ip和端口（51413）

通过transmission检测

- 齿轮标志→网络传输→测试端口

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191227202737.png)

## 参考资料

- [linuxserver/transmission](https://hub.docker.com/r/linuxserver/transmission/)