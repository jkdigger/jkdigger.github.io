---
layout:     post
title:      n1 Docker版Openwrt更新方法
subtitle:   Armbian+Docker+Openwrt
date:       2019-12-02 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机	
    - docker
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

#### 本地导入（方法一）

> ```
> docker import 镜像 自定义标签名
> docker run --restart always -d --network macnet --privileged 自定义标签名 /sbin/init
> ```
>
> `openwrt:R9.10.1` 为 自定义标签名

- 把tar.gz的镜像拖入到root文件夹
- 导入镜像

```
docker import openwrt-armvirt-64-default-rootfs.tar.gz openwrt:R9.10.1
```

- 部署镜像

```
docker run --restart always -d --network macnet --privileged openwrt:R9.10.1 /sbin/init
```

#### 拉取镜像（方法二）

> ```
> docker pull 作者/镜像名:标签
> docker run --restart always -d --network macnet --privileged 作者/镜像名:标签 /sbin/init
> ```
>
> `kanshudj/n1-openwrtgateway:r9.10.1` 为 `作者/镜像名:标签`

- 拉取镜像

```
docker pull kanshudj/n1-openwrtgateway:r9.10.1
```

- 部署镜像

```
docker run --restart always -d --network macnet --privileged kanshudj/n1-openwrtgateway:r9.10.1 /sbin/init
```



> 说明：如果刷入的是梁非凡的固件，此时已经可以输入192.168.2.3登入openwrt，不需要进行下面的步骤。
>
> 梁非凡固件说明：如果你的路由器网段是192.168.2.x，可直接浏览器输入ip 192.168.2.3 登入管理页，否则就按教程修改ip，密码password

#### 推荐的op镜像

- [breakersun/openwrt](https://hub.docker.com/r/breakersun/openwrt)

> breakersun/openwrt:aarch64
> 按照原计划维护，含openclash以及较多组件
>
> ![breakersun/openwrt:aarch64](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191212120122.png)
>
> breakersun/openwrt:pigroup
> 新增：群主老大最新发布的固件（精简，仅含hello world）

- 梁非凡docker版op
- [wky0211/pin1-openwrt:R91215](https://hub.docker.com/r/wky0211/pin1-openwrt)

### 04 配置 openwrt

#### 方法一（推荐）

- 查看镜像

```
docker ps
```

- 进入镜像

```
docker exec -it ‘container id’ sh
```

> ‘container id’ 是一串数字

- 编辑网络

```
vi /etc/config/network
```

- 输入i进入编辑，同样将**x**改为你主路由的网段

```
config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.2.2'
        option netmask '255.255.255.0'
        option gateway '192.168.2.1'
        option dns '114.114.114.114 223.5.5.5'
```

> ```
> config interface 'loopback'
>         option ifname 'lo'
>         option proto 'static'
>         option ipaddr '127.0.0.1'
>         option netmask '255.0.0.0'
> 
> config globals 'globals'
>         option ula_prefix 'fd2f:ea21:0e02::/48'
> 
> config interface 'lan'
>         option type 'bridge'
>         option ifname 'eth0'
>         option proto 'static'
>         option ipaddr '192.168.2.2'
>         option netmask '255.255.255.0'
>         option gateway '192.168.2.1'
>         option dns '114.114.114.114 223.5.5.5'
> 
> config interface 'vpn0'
>         option ifname 'tun0'
>         option proto 'none'
> 
> ```

- 按`Esc`，输入`:wq!`保存并退出编辑
- 重启网络

```
/etc/init.d/network restart
```

- 退出docker

```
exit
```

- 重启

```
reboot
```

#### 方法二

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
        option ipaddr '192.168.2.2'
        option netmask '255.255.255.0'
        option gateway '192.168.2.1'
        option dns '114.114.114.114 223.5.5.5'
```

> ```
> config interface 'loopback'
>         option ifname 'lo'
>         option proto 'static'
>         option ipaddr '127.0.0.1'
>         option netmask '255.0.0.0'
> 
> config globals 'globals'
>         option ula_prefix 'fddd:594f:f602::/48'
> 
> config interface 'lan'
>         option type 'bridge'
>         option ifname 'eth0'
>         option proto 'static'
>         option ipaddr '192.168.2.3'
>         option netmask '255.255.255.0'
>         option ip6assign '60'
>         option gateway '192.168.2.1'
>         option dns '114.114.114.114 223.5.5.5'
> 
> config interface 'vpn0'
>         option ifname 'tun0'
>         option proto 'none'
> ```

- 按`Esc`，输入`:wq!`保存并退出编辑
- 重启网络

```
/etc/init.d/network restart
```

> 重启网络也可以用这种方法：
>
> - 点击 [disconnect]
> - 在 containers处：勾选op→点击 [restart]

- 退出openwrt shell

```
exit
```

- 等一会浏览器中输入1.2.168.2.2 即可进入op

### 05 登入openwrt

- 浏览器输入 `192.168.x.2` 即可进入openwrt

> 用户名 root
>
> 密码 password

#### ①添加防火墙规则

```
iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
```

#### ②旁路由设置

> n1只负责网关

```
DHCP服务器 勾选 忽略此接口
关闭ipv6
```

![n1只负责网关](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191206200607.png)

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191212122439.png)

- 主路由不做任何更改

> 需要科学上网的设备：ip自定义+dns和网关设为192.168.2.2，网段前缀24

### 06 安装clash（可选）

> [openwrt安装clash插件](http://jkdigger.me/2019/11/06/openwrt%E5%AE%89%E8%A3%85clash%E6%8F%92%E4%BB%B6/)

### 参考资料

-  [梁非凡n1玩法](https://github.com/real-pin1group/3000web/wiki/playerdev_n1) 

