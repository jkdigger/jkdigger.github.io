---
layout:     post
title:      n1 Armbian安装Docker版qbittorrent
subtitle:   安装Docker版qbittorrent
date:       2019-12-20 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
---

## 安装Docker版qbittorrent

### 01 ssh连接n1拉取镜像

```
docker pull linuxserver/qbittorrent:latest
```

### 02 输入名字和镜像地址

- 浏览器登入docker管理页面

![输入名字和镜像地址](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191220200343.png)

- Name输入

```
qbittorrent 
```

- Image输入

```
linuxserver/qbittorrent:latest
```

### 03 设置Volumes

![设置qb目录](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191220200255.png)

- 点击Volumes
- 第一次点击map additional volume
- container输入

```
/config
```

>  这个是配置文件存放目录 

- host 输入

```
/etc/docker_qbittorrent
/etc/docker_transmission
```

> 提前把 /etc/docker_qbittorrent 文件夹创建好或者写其他目录也可以。

- 点击bind



- 第二次点击map additional volume
- container输入

```
/downloads 
```

> /downloads 为下载目录，**注意在软件中不要更改**!

- 点击bind
- host

```
/docker/samb/data/qb
```

> 输入你挂载的磁盘下载目录

###  04 设置env 

![设置env](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191220201106.png)

- 点击env 
- 点击add environment variable
- 输入5组name和value

```
PUID → 1007   
PGID → 100  
TZ → Asia/Shanghai  
UMASK_SET → 022  
WEBUI_PORT → 8084
```

> WEBUI_PORT是写你访问qb时，再ip后输入的端口，可以写其他端口，比如8084

###  05 设置Network ports configuration 

![设置端口](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191220201626.png)

- 点击publish a new network port
- 输入host和container点击TCP或UDP

```
6881/6881 TCP
6881/6881 UDP
8084/8084 TCP
8084/8084 UDP
```

> 上面那个6881是跟qb里面的监听端口同步
> 下面那个8084是跟WEBUI_PORT的端口同步的

### 06 部署容器

![Deploy the container](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191220201800.png)

- 点击  Deploy the container

## qb的初始化设置

- 浏览中输入 n1的ip: 8084

```
192.168.2.103:8084
账号admin 密码adminadmin
```

### 01 设置中文

- option→web ui→language：选择中文
- 保存save

### 02 添加torrents

> [torrents地址](https://github.com/ngosang/trackerslist)

- 工具→选项→BitTorrent
- 勾选 【自动添加以下 trackers 到新的 torrents】
- 粘贴tracker地址

### 03 修改qBittorrent配置文件 

- moba登入n1
-  文件位置/etc/docker_qbittorrent/qBittorrent  打开qBittorrent.conf 

```
在 [Preferences]下面添加Advanced\AnnounceToAllTrackers=true
```

- 在容器中重启qb

### 04 设置端口转发

- 在路由器中设置端口转发

![设置端口转发](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191220204935.png)

> 192.168.2.103是n1的ip

 **注意**：

- qb里面的默认保存路径不要修改!!

## 参考资料

- [使用Docker创建最新qBittorrent v4.2.0 适用于n1 贝壳云 我家云等amr64 arm x86](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=1325243&extra=page%3D3%26filter%3Dtypeid%26typeid%3D21&page=1)