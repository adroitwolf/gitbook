---
title: centos7安装redis
date: 2019-04-30 08:06:48
categories:
- linux
tags:
- 安装教程
- redis
---

# centos7安装redis

我的安装包版本是redis5.6

<!-- more -->

## 准备安装

---

安装redis需要gcc环境

```
yum install gcc
```

准备redis5.6的tar包，我这里离线下载的

```
# 这里我解压到了桌面 /root/桌面
tar -zxvf redis...   #解压
```

## 编译安装

---

```
make MALLOC=libc
make install  # 这里会将启动程序安装到 /usr/local/bin下面
cd /usr/local/bin
./redis-server #测试运行
```


如果出现下面的图片则证明启动成功。![](https://s2.ax1x.com/2019/04/30/E8A5h6.png)

## 修改配置

-----

在/path/下 我这里则是/root/桌面/redis5.6/redis.conf修改下面的变量

```
vi redis.conf
#下面是修改项 yes的意思就是可以守护态运行 no则是必须前台运行
daemonize yes
requirepass 123  #这个是密码 可以自定义
#注释下列选项 这样就可以远程连接
bind 127.0.0.1
```

移动redis.conf到/usr/local/bin下

```
cp redis.conf /usr/local/bin
```

## 启动

```
cd /usr/local/bin
./redis-server redis.conf & 
# &代表后台启动
```

利用本地电脑的redis可视化工具测试连接
