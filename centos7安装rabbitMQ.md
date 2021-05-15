---
title: centos7安装rabbitMQ
date: 2019-04-07 11:17:44
categories:
- linux
tags:
- 安装教程
- linux
---

# centos安装RabbitMQ并配置环境

之前我在网上搜罗了很多教程，但是大多都很繁琐，要么就是版本太老。

这里我提供了一个非常简单的方法

<!-- more -->

## 下载安装包

-----

打开[官网地址]: (https://www.rabbitmq.com/releases/)  

如下图：

![](https://s2.ax1x.com/2019/04/14/AOb5kR.png)



+ 这里我们需要erlang和rabbitmq-server
	- 打开对应网址下载最新的rpm安装包

## 安装
---

> 将安装包利用xftp传输到linux，注意不要直接拖拽，安装包容易出问题。

+ 在断网的时候安装erlang，因为我的centos7 的yum源会自动安装低版本的erlang。

```
yum install xxx  #安装软件
yum list installed #查看利用yum安装的所有软件
yum remove xxx #卸载软件
rpm -ql [rabbitmq-server] #查看软件安装在那里
```

+ 在联机的情况下安装rabbitmq，因为他需要安装socat依赖。

## 启动

-----

由于我这里是用来做javaweb的消息中间件，所以我需要他可以http访问，所以需要添加插件

  ```
  rabbitmq-plugins enable rabbitmq_management
  ```

启动rabbitmq程序

```
rabbitmq-server start
```

其他命令

```
rabbitmq-server restart #重启
rabbitmqctl stop   #停止
rabbitmqctl status  #状态
```

> 这里插播一条linux命令
>
> ```
> ps -ef|grep xxx  #查询所有xxx的进程
> kill -9 xxx #杀死进程端口号为xxx的进程
> ```

* 注意！默认guest guest账户已经不能允许在远程登陆，所以我们需要创建账号

## 配置rabbitmq
-----

下面我们创建账号，注意 这时候rabbitmq必须是启动状态

### 命令行

----

```
rabbitmqctl add_user root 111111 #创建账号
rabbitmqctl set_user_tags root administrator #设置root为管理员
rabbitmqctl list_users #查询所有用户
```

### ui模式

---

![操作](https://s2.ax1x.com/2019/04/07/Afjlg1.png)
