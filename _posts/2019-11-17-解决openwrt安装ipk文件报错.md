---
layout:     post
title:      解决openwrt安装ipk文件报错
subtitle:   原因是架构不匹配
date:      2019-11-17 13:28:47
author:     "jkdigger"
header-img: 
catalog: true
tags:
    -  n1刷机
---

## 前言

> 安装clash openwrt报错信息如下: 
>
> incompatible with the architectures configured...

### 01 查看openwrt已安装的软件架构

- ssh登入openwrt输入如下命令

```
opkg info | grep Architecture
```

- 显示如下

```
Architecture: all
Architecture: aarch64_generic
```

>  表示有两种架构 `all`和 `aarch64_generic`

### 02 编辑架构文件

```
vi /etc/opkg.conf
```

- 显示如下

```
dest root /
dest ram /tmp
lists_dir ext /var/opkg-lists
option overlay_root /overlay
option check_signature
```

### 03 添加架构信息

- 按`i`进入编辑模式，追加如下信息

```
arch all 100
arch aarch64_cortex-a53 200
arch aarch64_generic 300

```

- 追加后完成信息如下

```
dest root /
dest ram /tmp
lists_dir ext /var/opkg-lists
option overlay_root /overlay
option check_signature

arch all 100
arch aarch64_generic 200
arch aarch64_cortex-a53 300
```

- 按`Esc`，输入 `:wq` 保存并退出

注意:

- 后面的数字（例如`100、200、300`）表示多种架构的软件同时存在时安装采用的优先级，数字越小优先级越高。
- `aarch64_cortex-a53`是我要安装的 [clash ipk软件]( https://github.com/frainzy1477/clash/releases/tag/v0.16.3 ) 架构

### 04 正常安装ipk软件

- 定位到ipk文件所在目录

```
cd /tmp
```

- 更新软件包

```
opkg update
```

- 安装ipk文件

```
opkg install clashclash_0.16.3_sunix_aarch64_cortex-a53.ipk
opkg install luci-app-clash_1.2.8-2_all.ipk
```

> 如果遇到 But that file is already provided by package * libubox ，在安装命令后加上
>
> ```
> opkg install luci-app-clash_1.2.8-2_all.ipk --force-overwrite
> ```



 