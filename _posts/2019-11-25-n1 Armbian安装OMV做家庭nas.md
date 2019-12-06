---
layout:     post
title:      n1 Armbian安装OMV做家庭nas
subtitle:   OMV即OpenMediaVault
date:       2019-11-25 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
---



## 前言

小钢炮的samba总是出问题，电脑无法连接，因此换成omv做家庭nas。

## 准备工作

- 已写入Armbian到系统emmc的n1一台。

## 1. 安装OMV

### 01 方法1：自动安装OMV

- ssh登入n1
- -输入如下命令

```
armbian-config
```

- 方向键移到Software⇨回车
- 方向键移到Softy⇨回车
- 方向键移到OMV⇨ 空格键选择⇨回车确定
- 等待安装完成

### 02 方法2：手动安装

> 手动安装的原因是自动安装容易出错，下载进度缓慢。

- 添加 OMV 软件源

```
echo "deb http://packages.openmediavault.org/public arrakis main" > /etc/apt/sources.list.d/openmediavault.list

apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 7E7A6C592EF35D13 24863F0C716B980B
```

- 设置环境变量

```
export LANG=C
export DEBIAN_FRONTEND=noninteractive
export APT_LISTCHANGES_FRONTEND=none
```

- 开始安装 OMV

```
apt update
apt install openmediavault-keyring postfix
apt install openmediavault
```

### 03 修复安装错误

- 安装出现错误后，输入如下命令即可安装成功

```
apt --fix-broken install
```

- 浏览器输入n1 ip即可进入omv系统

```
默认用户名 admin 
密码为：openmediavault
```

## 2. OMV必要设置

- 浏览器输入n1 ip

```
默认用户名 admin 
密码为：openmediavault
```

### 01 修改默认密码

- 常规设置→web管理员密码
- 输入密码和确认密码，比如

```
admin
admin
```

- 保存并应用

### 02 修改日期与时间

- 日期和时间->设置
- 时区输入，然后点击选择[Asia/Shanghai]

```
Asia/Shanghai 
```

- 勾选 [使用NTP服务器]
- 保存并应用

### 03 修改自动登出时间

- 系统→Web管理员→常规设置

> 自动登出 改为30分钟

- 保存并应用

## 3. 共享移动硬盘

> 注意： NFS 只是一小众的文件共亨协议，应用的不多，但苹果系统下应用很广，建议还是用SMB协议吧。通用性较强些。 

###  01 挂载移动硬盘

- 插上移动硬盘接到n1的usb口

>  OMV不支持EXT2，3，extFat，所以最好用ext4或NTFS 

-  存储器→文件系统
-  选择移动硬盘→点击[挂载]→保存并应用

### 02 添加移动硬盘中需要共享的文件夹

-  访问权限管理→点击[共享文件夹]
-  点击[添加]

> 名称：share
>
> 设备：选择移动硬盘
>
> 路径：点击选择，/表示共享整个移动硬盘
>

- 点击[保存]

  ![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126133124.png)



### 03 新建用户

- 访问权限管理->用户->点[添加]

```
名称 注意不能为root
密码
确认密码 

用户组 勾选users
```

### 04 设置新建用户对文件夹的权限

- 点击新建的用户
- 点击[特权]→勾选[读和写]
- 保存并应用

### 05 通过smb把共享文件夹共享出去

-  [服务]→[SMB/CIFS] 
- [设置]→常规设置→勾选[启用]→保存并应用
- [共享]→点击[添加]

> 共享文件夹：选择 share
>

- 保存并应用

![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126133146.png) 

>  之后就可以去[我的电脑]->[网络]中看到 名为“AML”的磁盘，输入刚创建的用户名和密码就可以登入

### 06 设置移动硬盘待机

-  存储器→磁盘
- 选择 移动硬盘→点击[编辑]

> 高级电源管理：1-待机状态时最低功耗
>
> 自动声音管理：禁用
>
> 停转时间：20分钟
>
> 写入缓存：勾选 [开启写入缓存]

 ![](https://raw.githubusercontent.com/jkdigger/picForBlog/master/images/20191126133226.png) 

## 4. 开启transmission

### 01 安装插件

- 系统→插件→点击[检查]
- 等待检查完成
- 输入trans找到 `openmediavault-transmissionbt`
- 勾选 [ openmediavault-transmissionbt]->点击安装→等待安装完成

### 02 开启插件

- 服务→BitTorrent
- Files and Locations

> 1.下载
>
> 共享文件夹： 移动硬盘
>
> 目录：Downloads
>
> 2.未完成-勾选[启用]
>
> 共享文件夹： 移动硬盘
>
> 目录：incomplete
>
> 3.Watch-勾选[启用]
>
> 共享文件夹： 移动硬盘
>
> 目录：watch
>
> **注意**：在移动硬盘自动创建Downloads、incomplete、watch文件夹
>
> 4.保存并应用

- RPC

> 勾选[启用]
>
> **修改登入密码**
>
> 保存并应用

- 设置

> 勾选[启用]
>
> 保存并应用

### 03  transmission汉化

- ssh登入n1
- 输入如下命令

```
wget https://github.com/ronggang/transmission-web-control/raw/master/release/install-tr-control-cn.sh

chmod +x install-tr-control-cn.sh

bash install-tr-control-cn.sh
```

- 此时出现中文件界面，按照提示输入数字

```
1
```

- 此时刷新页面`n1的ip:9091`即可看到新界面

> 如果没有看到说明浏览器缓存了，强制刷新Ctrl + F5或清除缓存

### 04 自动更新tracker

[参考](https://github.com/AndrewMarchukov/tracker-add)

- moba登入n1
- 在 opt下新建bin文件夹
- 下载两个脚本

```
wget --no-check-certificate -O /opt/bin/add-trackers-auto.sh https://raw.githubusercontent.com/AndrewMarchukov/tracker-add/master/tracker-add-auto.sh
```

```
wget --no-check-certificate -O /etc/systemd/system/transmission-tracker-add.service https://raw.githubusercontent.com/AndrewMarchukov/tracker-add/master/transmission-tracker-add.service
```

- 给执行权限

```
chmod +x /opt/bin/add-trackers-auto.sh
```

- 编辑`opt`目录下的`add-trackers-auto.sh`

> 设置文件中用户名密码，如果没有修改为“：”

- 自启动设置

```
systemctl daemon-reload
systemctl enable transmission-tracker-add.service
systemctl start transmission-tracker-add.service
```

- 查看状态

```
systemctl status transmission-tracker-add.service
```

- ctrl+c退出

> 此时，添加种子就会自动更新 tracker



### 05 启用随机端口

- 浏览器中输入n1的ip:9091即可进入transmission

```
192.168.2.111:9091
```

- 启用随机端口

> 设置→网络传输→勾选[启用随机端口]

## 参考资料

- [N1 Armbian 安装 OpenMediaVault](https://www.cnblogs.com/HintLee/p/9899471.html)
- [phicomm N1 armbian环境下安装功能丰富的开源NAS系统 OpenMediaVault](https://www.right.com.cn/forum/thread-342164-1-1.html)