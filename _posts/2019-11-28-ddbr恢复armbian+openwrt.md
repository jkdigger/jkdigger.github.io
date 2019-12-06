---
layout:     post
title:      ddbr恢复armbian系统
subtitle:  	armbian+openwrt
date:       2019-11-28 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
	- 恢复系统
---



## 准备工作

- 恢复官改系统的u盘
- 恢复路由系统的u盘

> 其实只要一个u盘即可，为了方便准备两个。区别在于ddbr目录中的用来恢复的备份文件不同。

## 1. ddbr恢复官改安卓系统

- n1插上u盘→上电
- 查看路由器上n1的ip，比如 `192.168.2.100`
- ssh登入n1

```
ddbr

r

yes

回车
```

> 等待恢复完成

- 断电

```
poweroff
```

- 拔电→拔u盘
- n1上电开机，此时n1已经恢复官改安卓系统



具体参考

- [ddbr法](http://jkdigger.me/2019/10/27/n1%E6%81%A2%E5%A4%8DEMMC-ddbr%E6%B3%95/)

- 也可以采用[线刷法](http://jkdigger.me/2019/11/06/n1%E6%81%A2%E5%A4%8Demmc-%E7%BA%BF%E5%88%B7%E6%B3%95/)

## 2. ddbr恢复盘路由系统

- 通过路由器查看n1的ip，比如 `192.168.2.100`
- 连接显示器
- 设置u盘启动

```
adb connect 192.168.2.100
adb shell reboot update
```

> 在黑屏的一瞬间，插上已经准备好的u盘。

- 输入ddbr命令

```
ddbr

r

yes

回车
```

> 等待恢复完成

- 输入 `poweroff`
- 拔电→拔u盘
- n1上电开机，此时n1已经恢复armbian+docker+openwrt的系统

## 3. 修改ip和dns

- ssh连接n1

> 用户名 root
>
> 密码 修改后的密码

### 01 修改原配置中n1的ip

```
vi /etc/network/interfaces
```

> 按`i`进入编辑模式
>
> 改address为n1的ip，比如 `192.168.2.100`
>
> 改gateway为主路由地址，比如`192.168.2.1`
>
> 按`Esc`→输入`:wq!`回车

### 02 修改原配置中的dns

```
vi /etc/resolv.conf
```

> 按`i`进入编辑模式
>
> 添加 `search lan`
>
> 修改 nameserver为主路由地址，比如`192.168.2.1`
>
> 按`Esc`→输入`:wq!`回车

- 重启n1

```
reboot
```

### 03 查看ip和dns

- ssh连接n1
- 查看ip

```
route -n
```

- 查看dns

```
nslookup www.baidu.com
```

- 重启n1

```
reboot
```

## 4. 登入docker图形界面

### 01 配置 Openwrt

- 浏览器中输入 `n1的ip：9000`

> 用户名：admin
>
> 密码：修改后的密码

- 点击[local]→[containers]
- 点击之前导入的[openwrt]的`>_`
- 点击[connect]
- 输入如下命令修改网关信息

```
vi /etc/config/network
```

- 按`i`进入编辑模式

> ip改成`x.*`，比如`x.2`，`x.3`
>
> - x是主路由ip
> - *是主路由网段内的一个数字，推荐2或3
>
> - `x.1`是主路由网段

- 按`Esc`，输入`:wq!`保存并退出编辑 

```
exit
```

-  在 containers处：勾选op→点击 [restart] 

### 02 登入openwrt

- 浏览器输入 `192.168.x.2` 即可进入openwrt

> 用户名 root
>
> 密码 password



## 5. 旁路由设置

恢复的固件默认全局模式

### 01 n1设置

> 此时op不需要更改。查看openclash插件是否生效。

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191127230010.jpg)

### 02 修改主路由的网关和DNS为op地址

- 登入主路由
- 老毛子系统：内部网络（Lan）→内网设置

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191127225617.jpg)

- 内部网络（Lan）→DHCP服务器

> 默认网关：op的地址
>
> DNS服务器：op的地址

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191127225434.jpg)

### 03 重启路由器和n1

### 04 防火墙规则

- 做旁路由，网站上不了或慢速添加的规则：在网络→防火墙→自定义规则加上 

```
 iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
```

