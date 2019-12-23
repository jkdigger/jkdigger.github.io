---
layout:     post
title:      n1 Armbian安装Docker版filebrowser
subtitle:   Docker版filebrowser
date:       2019-12-22 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
    - docker
---


## 前言
根据灯大小钢炮filebrowser修改而来，感谢灯大的开发。
## 安装Docker版filebrowser

### 01 ssh拉取镜像

- ssh连接n1
- 拉取镜像

```
docker pull 80x86/filebrowser:2.8.8-arm64
```

### 02 输入名字和镜像地址

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191223115904.jpg)

-  浏览器登入docker管理页面 
-  Name输入 

```
filebrowser
```

-  Image输入 

```
linuxserver/qbittorrent:latest
```

### 03 设置Volumes

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191223120015.jpg)

- 点击Volumes
- 第一次点击map additional volume
- container输入

````
/config
````

-  点击bind 
- host输入

```
/etc/docker_filebrowser
```

- 点击writable

> ssh登入n1，在etc目录创建docker_filebrowser文件夹，给予权限 `chmod 777 /etc/docker_filebrowser`



- 第二次点击map additional volume
- container输入

```
/myfiles
```

- 点击bind 
- host输入

```
/docker/samb/data
```

- 点击writable

> `/docker/samb/data`为映射的移动硬盘目录

### 04 设置env

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191223120141.jpg)

- 点击env
- 点击add environment variable
- 输入5组name和value

```
PUID → 1007   
PGID → 100  
TZ → Asia/Shanghai  
WEBUI_PORT → 8082
```

### 05 设置Network ports configuration

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191223120318.jpg)

- 点击publish a new network port
- 输入host和container点击TCP或UDP

```
8082/8082 TCP
8082/8082 UDP
```

### 06 部署容器

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191220201800.png)

- 点击 Deploy the container 

## filebrowser初始化

### 01 禁用2FA

```
docker exec -ti filebrowser sh -c 's6-svc -d /etc/services.d/fb && su -s /bin/sh -c "filebrowser users update admin --otp.disable" app && s6-svc -u /etc/services.d/fb'
```

> admin是默认的用户名和密码

### 02 登入filebrowser

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191223121455.png)

- 浏览器输入n1的ip:8082

```
比如 192.168.2.225:8082
```

- 用户名/密码

```
admin
admin
```

### 03 修改密码（可选）

```
docker exec -ti filebrowser sh -c 's6-svc -d /etc/services.d/fb && su -s /bin/sh -c "filebrowser users update 用户名 --password 新密码" app && s6-svc -u /etc/services.d/fb'
```

### 04 查看被封印的IP（可选）

```
docker exec -ti filebrowser sh -c 's6-svc -d /etc/services.d/fb && su -s /bin/sh -c "filebrowser logs ls" app && s6-svc -u /etc/services.d/fb'
```

### 05 解除封印（可选）

```
docker exec -ti filebrowser sh -c 's6-svc -d /etc/services.d/fb && su -s /bin/sh -c "filebrowser logs rm 被封禁的IP" app && s6-svc -u /etc/services.d/fb'
```

## 参考资料

- [File Browser Enhanced v2.8.8 更新](https://mp.weixin.qq.com/s/rIxat1fiN5a8mpKlMfrMMw)

