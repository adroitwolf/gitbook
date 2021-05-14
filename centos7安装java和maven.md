---
title: Centos7安装java和maven
date: 2019-11-18 20:57:23
categories:
- linux
tags:
- 安装教程
- java
- maven
---

# Centos7安装java和maven

## java教程
> 由于之前的笔记都是存到了网易云笔记上面，现在都懒得打开那个软件了，现在干脆直接把上面那些有的没的直接搬到这上面来，省时省力。

好了，正片开始


首先将我们的jdk包[这里我用的是jdk1.8.0的]上传到服务器上面去，之后一步一步敲命令就好了

```bash
mkdir -p /usr/share/java

mv jar包 /usr/share/java

tar -zxvf jar包

# 修改环境变量

vi /etc/profile
```


其实关于上面的修改环境变量我更倾向于用xftp,因为他可以直接调用你的物理主机修改服务器上面的文件，I love it....

在上面的文件的后面加上

```bash
# java环境变量配置
JAVA_HOME=/usr/share/java/jar包名称
CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar
PATH=$PATH:$JAVA_HOME/bin
export JAVA_HOME CLASSPATH PATH
```


保存文件后更新一下配置文件

```bash

source /etc/profile

```

验证java是否安装成功

```bash

java -version
```

出现java版本后则代表安装成功

## maven教程

我们直接在清华镜像里面下载maven压缩包
```bash
wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```


解压到user/local/ 下面

```bash

tar -zxvf apache-maven-3.6.3-bin.tar.gz -C /usr/local/maven

# 修改环境变量

vi /etc/profile

# 生效配置文件
source /etc/profile
```


在配置文件的后面添加
```
export PATH=$PATH:/usr/local/maven/bin
```
出现下面的情况则代表安装成功
[![sn5M26.png](https://s3.ax1x.com/2021/01/08/sn5M26.png)](https://imgchr.com/i/sn5M26)
  