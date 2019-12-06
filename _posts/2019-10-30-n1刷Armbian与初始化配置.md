---
layout:     post
title:      n1刷Armbian与初始化配置
subtitle:   官改安卓系统→Armbian→BBR→换源→安装软件→修改时区
date:       2019-10-30 19:49:56
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
	- Armbian
---



## 注意

在刷Armbian之前必须确认n1的emmc没有写入其他固件。如果已经写入了其他固件，比如op系统、小钢炮系统等，必须先恢复官改安卓系统，之后才能刷Armbian。

## 准备工作

- Armbian5.77固件
- Armbian5.77修改后的的dtb文件
- 写入镜像软件Usb Image Tool

## 写入Armbian到n1

### 01 把镜像写入u盘

注意：已有Armbian系统的u盘可忽略这一步

1.将镜像写入U盘

- 打开usb image tool，此时左侧出现u盘，单击u盘
- 点击【恢复】，选择要写入的armbian img文件，等待写入完成

> 写入完成后会提示格式化，原因是windows系统无法识别linux系统，点击取消。

 2.修改镜像中的文件

- 复制 `meson-gxl-s905d-phicomm-n1.dtb` 到U盘的 `boot/dtb`目录下
- 修改u盘的`boot/`目录下的uEnv.ini文件：将meson-gxl-s905x-khadas-vim.dtb替换为meson-gxl-s905d-phicomm-n1.dtb

```
dtb_name=/dtb/meson-gxl-s905d-phicomm-n1.dtb
```

3.关闭软件，拔下u盘

### 02 设置从u盘启动

- 电脑上cmd
- 输入如下命令

```
adb connect 192.168.x.x    //注意：这里的IP地址为你N1设备的地址
adb shell reboot update
```

- 在系统关机的过程中，**插入写好的系统U盘**

> 如果是刚写入的系统，会要求你登入。
>
> 输入root，密码1234.
>
> 登入完，第一个是让输入默认密码1234，第二是输入新密码，不能太短也不能与默认密码接近(可输入password)，第三个是确认新密码(password)。提示创建新用户名，不创建，ctrl+c取消即可。
>
> 用新密码重新连接。

- 此时连接显示器，出现进入armbian界面

### 03 把Armbian写入n1

- 使用scanport工具获取n1的ip（没有显示器的情况可以这种操作，目的是获取n1的ip）

```
scanport设定
起始ip 192.168.5.1（路由器网关）
结束ip 192.168.5.255
端口 80,22
超时 200
线程 10
```

- 安装到emmc

```
./install.sh
```

>  提示`Complete copy OS to eMMC`即写入完成。 

```
shutdown now
```

- 关机后断电，拔下u盘。

> 此时让n1通电开机，会自动进入Armbian系统

## Armbian初始化配置

### 01 开启BBR加速

- 查询是否已开启

```
 lsmod | grep bbr
```

> tcp_bbr 20480 1表明已开启bbr

- 没有开启则输入如下

```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

- 重启

```
reboot
```

- 查询是否已开启

```
 lsmod | grep bbr
```

### 02 更换国内源

#### 方法一（推荐）

- 输入

```
armbian-config
```

- 回车选择`personal`
- 回车选择 `Mirror`
- 回车选择`tsinghua`
- 按`ESC`退出

#### 方法二

- 输入如下命令 

```
nano /etc/apt/sources.list
```

- 在原先的源前面加#号注释掉（有4条需要注释）
- 单击右键粘贴**更新源5.6+5.77 (debian 9):**

```
deb http://mirrors.tuna.tsinghua.edu.cn/debian stretch main contrib non-free
deb http://mirrors.tuna.tsinghua.edu.cn/debian stretch-updates main contrib non-free
deb http://mirrors.tuna.tsinghua.edu.cn/debian-security stretch/updates main contrib non-free
deb http://mirrors.tuna.tsinghua.edu.cn/debian stretch-backports main
```

> [来XQ7大神给的自清华源](https://www.right.com.cn/forum/thread-430903-1-1.html)

-  `ctrl+x` 退出编辑
-  按`y`→回车保存 
-  更新

```
apt-get update
```

 至此，软件源更换完毕。 

### 03 安装常用软件包

- 5.6/5.77需要安装

```
apt install ipset tcpdump pppoe pppoeconf net-tools git dnsmasq isc-dhcp-server cifs-utils tcptraceroute iftop telnet -y
```

> 5.91 5.94或后续新版：可以不用执行上面的命令，因为新版的一般都自带了。

### 04 修改时区

- ssh连接n1后

#### 方法一（推荐）

- 输入

```
armbian-config
```

- 回车选择`personal`
- 回车选择 `timezone`
- 回车选择`Aisa`
- 回车选择`chongqing`
- 按`ESC`退出
- 验证

```
date -R
```

- 重启

```
reboot
```

#### 方法二

```
tzselect
```

- 依次输入如下

```
4
9
1
```

- 复制文件到/etc目录下

```
cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
```

- 更新时间

```
ntpdate time.windows.com #有时候会提示 ntpdate: command not found，则输入如下

ntpd time.windows.com
```

- 验证时间

```
date -R
```

- 重启

```
reboot
```

### 05 右键粘贴

> 默认debian的vim右键没法粘贴内容，需要改一下模式：

```
vim /usr/share/vim/vim80/defaults.vim
```

> 查找 set mouse
> if has('mouse')
>   set mouse=a
> endif
>
> 将值从"a"改成"r"
> if has('mouse')
>   set mouse=r
> endif

## 参考资料

- [斐讯N1 – 完美刷机Armbian教程](https://yuerblog.cc/2019/10/23/%e6%96%90%e8%ae%afn1-%e5%ae%8c%e7%be%8e%e5%88%b7%e6%9c%baarmbian%e6%95%99%e7%a8%8b/)