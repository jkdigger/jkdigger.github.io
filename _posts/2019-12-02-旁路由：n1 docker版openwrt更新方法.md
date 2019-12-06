---
layout:     post
title:      n1 docker版openwrt更新方法
subtitle:   Armbian+Docker+Openwrt
date:       2019-12-02 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机	
	- openwrt
---

### 说明

以下教程适用于docker下安装openwrt。

### 01 移除旧版op

- 登录docker容器管理页

> n1 ip:9000

- 移除旧版op容器

> 在containers处：先点kill→再点remove

- 移除旧版op镜像

> 在image处：点击remove

### 02 重启设备

- ssh连接n1

```
reboot
```

### 03 部署op

- 把tar.gz的镜像拖入到root文件夹
- 导入镜像

```
docker import openwrt-armvirt-64-default-rootfs.tar.gz openwrt:R9.10.1
```

> docker import+镜像+自定义标签名

- 部署镜像

```
docker run --restart always -d --network macnet --privileged openwrt:R9.10.1 /sbin/init
```

> 如果之前没有部署过op，则需要输入完整的部署命令为：
>
> ```
> docker import openwrt-armvirt-64-default-rootfs.tar.gz openwrt:R9.10.1
> ip link set eth0 promisc on
> modprobe pppoe
> docker network create -d macvlan --subnet=192.168.2.0/24 --gateway=192.168.2.1 -o parent=eth0 macnet
> docker run --restart always -d --network macnet --privileged openwrt:R9.10.1 /sbin/init
> ```



> 说明：如果刷入的是梁非凡的固件，此时已经可以输入192.168.2.3登入openwrt，不需要进行下面的步骤。
>
> 梁非凡固件说明：如果你的路由器网段是192.168.2.x，可直接浏览器输入ip 192.168.2.3 登入管理页，否则就按教程修改ip，密码password

### 04 配置 openwrt

> chrome进入vi可能存在问题，可以换edge浏览器解决。

- 登入docker图形管理界面

```
n1的ip:9000
```

- 点[container]
- 选择刚导入的镜像→ 点击 `>_` → 点击 [Connect]

![img](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126235631.png)

- 按i改网关信息

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

- 按`Esc`，输入`:wq!`保存并退出编辑
- 点击 [disconnect]
- 在 containers处：勾选op→点击 [restart]

### 05 登入openwrt

- 浏览器输入 `192.168.x.2` 即可进入openwrt

> 用户名 root
>
> 密码 password

### 参考资料

-  [梁非凡n1玩法](https://github.com/real-pin1group/3000web/wiki/playerdev_n1) 