---
layout:     post
title:      openwrt安装clash插件与clash使用
subtitle:   
date:      2019-11-06 23:46:47
author:     "jkdigger"
header-img: 
catalog: true
tags:
    -  n1刷机
---

### 1. 下载ipk文件

- [下载clash]( https://github.com/frainzy1477/clash/releases/tag/v0.16.3 )：n1对应的版本是 [clash_0.16.3_sunix_aarch64_cortex-a53.ipk](https://github.com/frainzy1477/clash/releases/download/v0.16.3/clash_0.16.3_sunix_aarch64_cortex-a53.ipk)
- [下载  luci-app-clash]( https://github.com/frainzy1477/luci-app-clash/releases/tag/v1.2.7 )：选择原版openwrt或  lean 编译openwrt

### 2. 安装ipk文件

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

### 3. clash设置

- 打开浏览器，输入openwrt ip
- 服务-clash（自动出现）

#### 01 添加节点

- 点击[订阅&服务器]
- 在[订阅URL] 处，输入订阅地址
- 点击更新

#### 02 启用clash

- 点击[客户端client]
- 勾选 [启用]，选择内核clash，选择配置类型
- 点击[保存&应用]

#### 03 操作clash

- 点击[总览]，如果一片绿表示正常
- 点击[打开外部控制]，弹出clash面板
- 输入host、端口、密钥（在openwrt[总览]可以查到）
- 点击[代理]->选择规则

### 参考资料

-  [frainzy1477/clash](https://github.com/frainzy1477/clash)