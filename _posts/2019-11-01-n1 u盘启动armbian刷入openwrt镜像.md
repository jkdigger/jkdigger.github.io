---
layout:     post
title:     n1 u盘启动armbian，刷入openwrt镜像
subtitle:   
date:        2019-11-01 18:26:30
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
---

## 准备工作

- 已恢复安卓官改系统的的n1盒子
- 已刷入XQ7大神的armbian 5.6的 u盘
- openwrt固件

## 刷入openwrt

### 01 查看盒子ip

- n1连接网线到主路由lan口，通电
- 获取ip

> 方法一：登入路由器查看
>
> 方法二：连接显示器和键盘，登入电视界面查看ip
>
> 方法三：n1通电，电脑上打开scanport工具，即可获取n1的ip
>
> ```
> scanport设定
> 起始ip 192.168.5.1（路由器网关）
> 结束ip 192.168.5.255
> 端口 22
> 超时 200
> 线程 10
> ```

### 02 设置n1从u盘启动

- 电脑上打开cmd，输入如下命令

```
adb connect N1的IP地址
adb shell reboot update
```

- 在系统关机的过程中，**插入写好的系统U盘**，此时连接显示器会看到四只企鹅。

### 03 刷入openwrt固件

-  通过 [MobaXterm](https://mobaxterm.mobatek.net/) 软件连接 N1 

```
点击session->点击SSH->输入ip和用户名
```

-   执行脚本，将 EMMC 分成两个分区，并将 Armbian 写入到 EMMC 中： 

```
/boot/create-mbr-linux.sh
./install.sh
```

- 在**MobaXterm**能看到 root 目录的文件，把 Op固件拖入到 root 目录中 
-  上传完成后，输入下列命令

```
创建一个emmc2文件夹
mkdir /emmc2

将Armbian所在的分区挂载到emmc2文件夹
mount /dev/mmcblk1p2 /emmc2

删除Armbian的所有文件
rm -rf /emmc2/*

挂载Openwrt镜像文件，名字以你自己镜像文件为准
losetup -P -f --show N1_openwrt_clash_wifi_20190527.img

挂载指定的文件夹
mount /dev/loop0p2 /media

将Openwrt的所有文件复制到Armbian分区文件夹emmc2
cp -R /media/* /emmc2

卸载所有的挂载
umount /media
losetup -d /dev/loop0
umount /emmc2

关机
poweroff
```

- 先拔掉电源，再拔掉 U 盘，n1重新通电

> 此时如果n1已经杀入openwrt系统

## 参考资料

- [斐讯N1 eMMC 刷入OpenWRT系统教程笔记]( https://www.maxlicheng.com/embedded/36.html )