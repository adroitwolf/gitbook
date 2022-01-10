---
title: ssh免密
date: 2021-01-08T17:51:55.000Z
tags:
  - linux
  - ssh
categories:
  - linux
---

# ssh免密

最近由于经常用linux，尤其是git这一方面，需要经常的push项目，而没有ssh的时候经常会需要登陆密码登陆，很麻烦。

查了一堆文章后，写了下面最适合我的这篇文章

首先我们要有我们的ssh命令，在windows里面生成ssh的rsa密钥，具体命令如下：

```bash
ssh-keygen -t rsa -C ""
# 引号里面是邮箱，外国人有毛病，什么玩意都要弄个邮箱，我就直接为null了
```

生成的ssh文件在 /c/Users/用户名/.ssh/里面，其中带.pub的为公钥，不带的为私钥

之后我们要在我们的服务器上，创建一个想要本地服务器登陆的账号：

```bash
usradd git #由于我这里是为了用Git,所以直接创建的git

passwd git #root用户创建密码
```

创建成功之后打开我们的Windows里面的git工具，然后输入下面的命令，将本地的公钥上传到服务器上面

```bash
ssh-copy-id git@192.168.3.99
# 用户名@ip地址
```

[![sKA00O.png](https://s3.ax1x.com/2021/01/08/sKA00O.png)](https://imgchr.com/i/sKA00O)

输入密码后，成功

> 这个命令的原理是 将 /c/Users/用户名/.ssh/id\_rsa.pub 里面的内容复制到 远程服务器git用户的 /home/git/.ssh/authorized\_keys文件里面最后一行中,如果不想用命令直接手动操作也可以。

## 测试ssh连接命令

```bash

ssh -v git@192.168.3.99
```

[![sKEaCj.png](https://s3.ax1x.com/2021/01/08/sKEaCj.png)](https://imgchr.com/i/sKEaCj)

上面图片可以看出我们正在密钥校验，成功之后可直接登入服务器。
