---
title: hexo部署云服务器
date: 2021-01-08T19:39:19.000Z
tags:
  - Hexo
  - Linux
  - Git
categories:
  - Hexo
---

# hexo部署云服务器

之前我的hexo是直接部署到git上面的，但是随着我的VPN到期，访问博客的速度是越来越慢，最后打算直接部署到linux服务器上。

## 项目环境准备

### git

服务器上面需要安装git，创建一个空仓库，然后利用钩子监控，每次有push过来直接clone到nginx的文件夹里面。

```bash
yum install -y git 
```

### git用户

再有就是我们不能直接利用root用户来远程push，这样权限太大了，我们需要直接创建用户，这里我直接创建git用户

```bash
usradd git
passwd git
```

这里的用户都是git组里面的，具体查询可以根据id命令或者groups命令查询。

我们还需要SSH连接，这时候可以参考我的博文：SSH免密这篇文章

### nginx

nginx用来代理服务器，下载安装nginx的文章我其他的文章有写

我们这里指定/var/blog文件夹为Hexo部署之后的文件夹

我们用git用户远程部署，所以需要将此文件夹设置成git用户

> 这一步非常重要

```bash
chown -R git:git /var/blog
        #组:用户
```

## 准备上传

我们需要登陆我们的git用户

```bash
su git 

cd /home/git
```

### 创建一个裸git项目

```bash
git init -bare  blog.git
cd blog.git
```

[![sKasUg.png](https://s3.ax1x.com/2021/01/08/sKasUg.png)](https://imgchr.com/i/sKasUg)

### 创建钩子

```bash
vi ./hooks/post-receive

# 下面为文件内容，可以直接粘贴,注意下面的注释后面一定要跟回车

#!/bin/bash

git  --work-tree=/var/blog --git-dir=/home/git/blog.git checkout -f


# 保存退出
```

### 本地Hexo配置文件修改

先备份一份本地\_config配置文件，然后打开，拉到最底

里面有这些项：

```
deploy:
  # 类型
  type: git
  # 仓库
  repo: 用户@服务器IP或域名地址:服务器上面创建的仓库绝对地址
  # 分支
  branch: master
```

### nginx配置

打开服务器的nginx的conf里面的nginx.conf文件

创建一个server

[![sKdfWd.png](https://s3.ax1x.com/2021/01/08/sKdfWd.png)](https://imgchr.com/i/sKdfWd)

重启

```bash

nginx -s reload
```

## 上传

上传命令

本地Hexo

```bash
hexo g -d
```

查看/var/blog里面出现文件,配置成功
