---
layout:     post
title:      n1 Armbian安装Docker版aria2和aria2 webui
subtitle:   安装aria2和aria2 webui
date:       2019-12-31 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
    - docker
---

## 安装Docker版aria2和aria2-webui

### 01 下载文件

- [下载地址](https://github.com/tonynii/alpine-aria2-webui)
- 下载方法1

```
点击clone or download→点击download zip
```

- 方法方法2

```
git clone git@github.com:tonynii/alpine-aria2-webui.git
```

###  02 编译映像 

- moba连接n1
- 上传下载下来的整个文件夹`alpine-aria2-webui`到root目录
- 切换路径到`alpine-aria2-webui`

```
cd `alpine-aria2-webui/
```

- 编译镜像

```
docker build -t="leafney/alpine-aria2-webui" .
```

> 编译时间较长，注意末尾有一个点。

### 03 运行aria2

- moba中输入如下命令

```
docker run --name aria2ui --restart always -p 6800:6800 -p 6801:6801 -p 51413:51413 -p 51415:51415 -d -v /boxvalume/aria2pan:/aria2down leafney/alpine-aria2-webui
```

> 注意：
>
> - “/boxvalume/aria2pan”为N1上实际存放下载文件的路径
> - 51413为bt监听端口，如果之前使用过这个端口需要替换成其他端口，并且在  `aria2.conf`中修改。  
>
> ```
> WEBUI_PORT 6801
> RPC_LISTEN_PORT 6800
> BT_LISTEN_PORT 51413
> DHT_LISTEN_PORT 51415
> ```

- 此时已经可以打开浏览器，输入n1的ip:6801 访问到aria2 web ui

```
比如输入 192.168.2.225:6801
```

## aria2初始化设置

### 01 设置登录secret

- 登入aria2 web ui
- 设置→连接设置→ 密码令牌（可选）处输入

```
aria2.secret
```

### 02 设置bttracker

- moba登入n1
- 打开之前上传的`root`目录下的`alpine-aria2-webui`文件夹
- 双击打开`etc/aria.conf`文件

- 拉到最后添加如下内容

```
# bt-tracker=BT服务器（多个服务器之间用,分开）
bt-tracker=udp://tracker.coppersurfer.tk:6969/announce,udp://tracker.leechers-paradise.org:6969/announce,udp://tracker.opentrackr.org:1337/announce,udp://tracker.internetwarriors.net:1337/announce,udp://p4p.arenabg.com:1337/announce,udp://9.rarbg.to:2710/announce,udp://9.rarbg.me:2710/announce,udp://tracker.openbittorrent.com:80/announce,udp://exodus.desync.com:6969/announce,udp://tracker.tiny-vps.com:6969/announce,udp://retracker.lanta-net.ru:2710/announce,udp://tracker.torrent.eu.org:451/announce,udp://tracker.moeking.me:6969/announce,udp://tracker.cyberia.is:6969/announce,udp://open.stealth.si:80/announce,udp://denis.stalker.upeer.me:6969/announce,udp://ipv4.tracker.harry.lu:80/announce,udp://open.demonii.si:1337/announce,udp://explodie.org:6969/announce,udp://zephir.monocul.us:6969/announce
```

> 注意`bt-tracker=`后的内容来自于 [trackers_best (推荐)](https://tk.sleele.com/)

- 点击保存→点击弹出对话中的yes
- 重启容器

### 03 修改pt相关设置

>  注意：如果需要下载pt需要进行额外的设置

- 打开之前上传的`root`目录下的`alpine-aria2-webui`文件夹
- 双击打开`etc/aria.conf`文件

- 修改与pt有关的选项

```
## BT/PT下载相关 ##

# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
follow-torrent=true
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=51413
# 打开DHT功能, PT需要禁用, 默认:true==========
enable-dht=true
# DHT网络监听端口, 默认:6881-6999
dht-listen-port=51415
# 本地节点查找, PT需要禁用, 默认:false=========
bt-enable-lpd=true
# 种子交换, PT需要禁用, 默认:true============
enable-peer-exchange=true
# 每个种子限速, 对少种的PT很有用, 默认:50K
#bt-request-peer-speed-limit=50K
# 客户端伪装, PT需要
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
#seed-ratio=0
# BT校验相关, 默认:true
#bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
bt-save-metadata=true
```

> ============是需要修改的地方

- 保存→点击yes确认替换

## 通过aria2+pandownload下载百度云

### 01 设置远程主机

- [pandownload下载地址](https://pandownload.com/index.html)
- 打开pandownload→更多功能→设置→远程

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191231210351.png)

```
主机： n1的ip，比如 192.168.2.225
端口： 6800
token： aria2.secret
```

- 检查连接

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191231210425.png)

### 02 添加下载连接到aria2

- 右键要下载的文件
- 选择远程主机

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191231210545.png)

> 此时已经可以去aria2 web ui中查看到刚刚添加的下载链接。

## 参考资料

- [tonynii/alpine-aria2-webu](https://github.com/tonynii/alpine-aria2-webui)
- [N1 Coreelec 利用Docker搭建aria2和webUI ](https://www.right.com.cn/forum/thread-810527-9-1.html)
- [解决ARIA2 BT下载速度慢没速度的问题:TRACKER和DHT](https://www.tjflora.com/archives/359)
- [pandownload远程下载](https://pandownload.com/document/remote.html)