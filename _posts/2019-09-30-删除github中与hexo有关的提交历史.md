---
layout:     post
title:      如何删除github中与hexo有关的提交历史？
subtitle:   
date:       2019-09-30 07:26:34
author:     jkdigger
header-img:  
catalog: 	 true
tags:
    - 博客教程
---

- 第一步：备份一份 ``.deply_git` `文件夹
- 第二步：删除``.deply_git` `文件夹和 `public`文件夹
- 第三步：进入博客安装目录，比如`C:\APPing\jkreading\blog`，右键选择`git bash here`
- 第四步：依次输入 `hexo clean` 、`hexo g`和`hexo d`命令

此时，去github仓库中查看，只剩下两个commit，一个是第一次创建时的commit，另一个是本次提交的commit。