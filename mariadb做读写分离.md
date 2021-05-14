---
title: mariadb做读写分离
date: 2019-05-13 12:50:08
categories:
- mysql
tags:
- mysql优化
---

# mariadb读写分离

上一次文章我在centos7上安装了mariadb并开启了主从复制模式，这一篇打算利用mycat做读写分离

mycat现在可以相称像是nginx一样的反向代理，他可以不暴露数据库的ip

环境：

| IP地址        | 作用                  |
| ------------- | --------------------- |
| 192.168.3.137 | mycat服务器，主数据库 |
| 192.168.3.136|从数据库|

## 安装mycat
------
* 打开mycat[官网](http://www.mycat.io/),选择版本下载，我这里是1.6.6

* 将tar包解压到centos下的/usr/local/下

```
添加环境变量
vi /etc/profile
export MYCAT_HOME=/usr/local/mycat
# 退出
source /etc/profile #使之生效
cd /usr/local/mycat/bin
./mycat start #启动
./mycat status #查看是否启动
```

![](https://s2.ax1x.com/2019/05/13/E4b7VA.png)

一般没什么问题

## 配置环境

-----

首先备份一下两个重要文件，如果弄坏了还可以还原

```
cd /usr/local/mycat/conf
cp ./server.xml ./server.xml.bak
cp ./schema.xml ./schema.xml.bak
```



配置用户供远程登陆 **server.xml**

```
vi /usr/local/conf/server.xml 

```

![](https://s2.ax1x.com/2019/05/13/E4qP5q.png)

拉到最后，修改里面的用户就行了



配置需要管理的表 **schema.xml**

![](https://s2.ax1x.com/2019/05/13/E4LkTA.png)

## 测试连接
------
重启一下mycat,查看一下状态是否运行
```
./mycat restart
./mycat status
```
打开window上的navicat 选择连接 注意 这里一定是mysql 不然会报错
> 注意 mycat的默认端口是8066

![](https://s2.ax1x.com/2019/05/13/E4Llwj.png)

交易成功！