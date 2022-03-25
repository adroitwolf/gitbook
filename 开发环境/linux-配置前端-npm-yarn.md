---
title: linux 配置前端 npm yarn 
date: 2021-05-15 09:29:12
categories:
- Linux
tags:
- linux
---

# linux 配置前端 npm yarn

这几天更新了windows系统，安装一下wsl,试一下之前就想用的zsh，感觉还不错。之后就直接用Linux接管windows命令。

现在安装一下前端常用的工具  npm hexo


## 安装ndoe

这里建议下载v12或者以前的版本，新版本在笔者写的这段时间内很不稳定，容易出问题。


```bash

wget https://js.org/dist/v12.18.1/-v12.18.1-linux-x64.tar.xz # 我这里下载到了/usr/local路径下

tar -vxf -v12.18.1-linux-x64.tar.xz

cd -v12.18.1-linux-x64
```

> 因为我现在用的是zsh shell，所以我不能在**/etc/profile**这个全局配置文件里面写的全局命令，我需要在~/.profile里面写路径


如果用的是正常的shell，直接在/etc/profile里面写，然后更新一下就ok

```bash
export PATH=$PATH:/user/local/-v12.18.1-linux-x64/bin
```


然后我们去~/.zshbrc里面添加以下porfile文件

```bash
source ~/.profile
```


更新一下后验证是否可以使用

```bash
ndoe -v #  出现版本号则证明配置完成
```


### 安装npm


因为我们安装之后，会自动安装npm依赖，所以我们直接用就行。

但是有一点，如果我们使用sudo npm 的时候，会直接提示不存在npm。

这是什么原因呢？

我们之前安装不是/usr/local路径下了么，这个是普通用户的bin目录
而sudo执行的是/usr/bin目录，这是root用户的目录

所以使用sudo命令是识别不到这个命令的，我们可以使用以下方法来处理这个问题
我们在/usr/bin目录下面创建一个软连接


```bash

sudo ln -s /usr/local/-v12.18.1-linux-x64/bin/ /usr/bin/

sudo ln -s /usr/local/-v12.18.1-linux-x64/bin/npm /usr/bin/npm
```

问题解决。


## 安装yarn

一定要用npm安装，不能用apt或者yum工具安装