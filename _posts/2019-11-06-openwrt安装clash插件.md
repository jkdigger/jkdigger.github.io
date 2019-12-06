---
layout:     post
title:      openwrt安装clash插件
subtitle:   openwrt安装ipk文件
date:      2019-11-06 23:46:47
author:     "jkdigger"
header-img: 
catalog: true
tags:
    -  n1刷机
	- openwrt
---

## 安装clash
### 01 下载ipk文件

- [下载clash]( https://github.com/frainzy1477/clash/releases/tag/v0.16.3 )：n1对应的版本是 [clash_0.16.3_sunix_aarch64_cortex-a53.ipk](https://github.com/frainzy1477/clash/releases/download/v0.16.3/clash_0.16.3_sunix_aarch64_cortex-a53.ipk)
- [下载  luci-app-clash]( https://github.com/frainzy1477/luci-app-clash/releases/tag/v1.2.7 )：选择原版openwrt或  lean 编译openwrt

### 02 安装ipk文件

- 用MobaXterm连接openwrt

```
ip： openwrt登入地址
用户名： root
密码默认：password
```

- 把ipk文件上传道openwrt的`/tmp`目录下
- MobaXterm输入如下命令

```
opkg update  #必要步骤
opkg install clash_0.16.3_sunix_aarch64_cortex-a53.ipk
opkg install luci-app-clash_1.2.7-2_all.ipk
```

- 退出 

```
exit
```

## 解决安装clash出错

> 安装clash openwrt报错信息如下: 
>
> incompatible with the architectures configured...
>
> 报错原因：架构不匹配

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

## clash设置

- 打开浏览器，输入openwrt ip
- 服务-clash（自动出现）

### 01 添加节点

- 点击[订阅&服务器]
- 在[订阅URL] 处，输入订阅地址
- 点击更新

### 02 启用clash

- 点击[客户端client]
- 勾选 [启用]，选择内核clash，选择配置类型
- 点击[保存&应用]

### 03 操作clash

- 点击[总览]，如果一片绿表示正常
- 点击[打开外部控制]，弹出clash面板
- 输入host、端口、密钥（在openwrt[总览]可以查到）
- 点击[代理]->选择规则

## 参考资料

-  [frainzy1477/clash](https://github.com/frainzy1477/clash)