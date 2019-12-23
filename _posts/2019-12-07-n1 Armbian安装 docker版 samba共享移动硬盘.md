---
layout:     post
title:      n1 Armbian安装Docker版Samba共享移动硬盘
subtitle:   n1 Armbian Docker Samba
date:       2019-12-06 18:00:00
author:     "jkdigger"
header-img: 
catalog: true
tags:
    - n1刷机
    - docker
---

## 说明

在使用docker版openwrt的情况下的，通过安装docker版samba共享移动硬盘。

## 共享移动硬盘

### 01 取消openwrt下的自动挂载

> 如果是小钢炮或其他安装了samba服务的系统需要关闭自带samba服务，否则端口占用导致创建容器失败.

- 系统→挂载点→全部取消勾选
- 保存并应用

### 02 安装samab

#### ①安装方法一

host模式

```
docker run -d \
  --name samba \
  --restart=always  \
  --net=host \
  -v /docker/samb/conf:/etc/samba \
  -v /docker/samb/share:/mnt/share \
  -v /docker/samb/data:/mnt/data \
  lstcml/samba
```

#### ②使用samba

安装命令结束，即可在电脑上访问samab。可以看到data和share目录

```
data目录：需要输入用户名和密码才能访问，拥有读写删权限
share目录：不需要输入用户名和密码就能访问，拥有读写删权限
```

```
默认用户账户密码：admin/admin
```

docker配置： docker/samb/conf

#### ②给予权限

>  如果在电脑上创建文件时，提示访问没权限，说明映射到宿主机的目录没有权限

- ssh登入armbian

```
chmod 777 /docker/samb/data
chmod 777 /docker/samb/share
```

### 03 Armbian下挂载移动硬盘

- ssh连接Armbian系统

- 查看移动硬盘硬盘

```
blkid
```

>  ```
>  /dev/sda1: LABEL="usb" UUID="589A8FB59A8F8E66" TYPE="ntfs" PARTUUID="9e23dbc0-01"
>  /dev/sdb1: SEC_TYPE="msdos" LABEL="BOOT" UUID="86FB-1779" TYPE="vfat" PARTUUID="ed3d6e03-01"
>  /dev/sdb2: LABEL="ROOTFS" UUID="3f6dd295-ceca-4982-abe3-2aa3ec766e28" TYPE="ext4" PARTUUID="ed3d6e03-02"
>  ```
>
>  可以看出 sdb1 和 sdb2 是属于U盘，而 sda1 是属于移动硬盘 

- 建立挂载点

> 该目录已存在可以不用创建
>
> ```
> mkdir /docker/samb/data
> ```

- 把 `/dev/sdb1` 挂载到 `/docker/samb/data` 上面 

```
mount -t ntfs /dev/sda1 /docker/samb/data
```

- 修改`/etc/fstab`,追加一行,实现开机自动挂载

> mount命令会在重启服务器后失效，所以要将分区信息写到/etc/fstab文件中让它永久挂载

```
vi /etc/fstab
```

- 按i进入编辑模式，

```
UUID="589A8FB59A8F8E66" /docker/samb/data ntfs defaults 0 1
```

>  ```
>  具体说明，以挂载/dev/sdb1为例：
>  <fs spec>：分区定位，可以给UUID或LABEL，例如：UUID=6E9ADAC29ADA85CD或LABEL=software
>  <fs file>：具体挂载点的位置，例如：/data
>  <fs vfstype>：挂载磁盘类型，linux分区一般为ext4，windows分区一般为ntfs
>  <fs mntops>：挂载参数，一般为defaults
>  <fs freq>：磁盘检查，默认为0
>  <fs passno>：磁盘检查，默认为0，不需要检查
>  ```

- 按`Esc`，输入`:wq` 回车确认

```
:wq
```

- 验证一下配置是否正确

```
mount -a
```

- 重启系统

```
reboot
```

## 参考资料

- [Linux配置硬盘自动挂载](https://www.jianshu.com/p/336758411dbf)
- [ 30多个N1可用docker镜像](https://www.right.com.cn/forum/thread-911375-1-1.html)
- [Docker安装samba共享目录](https://dylanyang.top/post/2019/05/22/docker%E5%AE%89%E8%A3%85samba%E5%85%B1%E4%BA%AB%E7%9B%AE%E5%BD%95/)

