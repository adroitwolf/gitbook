---
title: centos7 安装mariadb 并配置主从复制
date: 2019-05-12 17:05:21
summary: 在高并发下，一个数据库肯定是不得行的，那么主从复制是必要的！
categories:
- linux
tags:
- 安装教程
- linux
- mysql
---

# centos7下安装mariadb

## 卸载已经存在的mariadb

-------

检查是否已经安装mariadb

```
rpm -qa|grep mariadb
```

如果存在先卸载掉

```
rpm -e --nodeps xxxx
```

<!-- more -->

## 安装mariadb

------

```
yum -y install mariadb mariadb-server #安装
systemctl start mariadb # 启动mariadb服务
systemctl enable mariadb #设置开机启动

```



## mariadb简单配置

```
mysql_secure_installation #开始简单配置
首先是设置密码，会提示输入密码
Enter current password for root (enter for none):  -->初次设置直接回车
Set root password? [Y/n] -->y
New password:
Re-enter new password:
其他设置
Remove anonymous users? [Y/n] -->是否删除匿名用户 回车
Disallow root login remotely? [Y/n] -->是否禁止远程用户 n
Remove test database and access to it? [Y/n] -->删除test数据库 回车
Reload privilege tables now? [Y/n]  -->是否重新加载权限表 回车

```

## 配置mariadb编码

```
vi /etc/my.cnf
#在[mysqld]下面添加
init_connect='SET collation_connection = utf_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake

vi etc/my.cnf.d/clent.cnf
# 在[client]下添加
default-character-set=utf8

vi etc/my.cnf.d/mysql-clients.cnf
#在[mysqld]下添加
default-character-set=utf8
```

```
配置完成 重启服务
systemctl restart mariadb
查看mariadb字符集
show variables like "%character%';
show variables like "%collation%";

```



## 添加外网用户

------

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY '123' WITH GRANT OPTION; #配置用户并授权
flush privileges; #刷新权限
```



## 主从复制

### 主服务器配置

```
vi /etc/my.conf
# 添加下面两个字段
server-id=137 #这个数字随便写

log-bin=master-log #我尝试过放到另一个文件夹下，但是由于权限问题，始终启动错误，现在这个二进制文件是在你配置的datadir下面的
innodb_file_per_table=on  #跳过主机名解析。在CentOS 6自带的mysql后面的=on不用写
skip_name_resolve=on #innodb的每个表是用单独的文件
```

启动服务
```
systemctl start mariadb      #启动数据库
GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'guest'@'%' IDENTIFIED BY '123'; #创建从服务器用来连接的账号
FLUSH PRIVILEGES; #刷新服务器
```

查询当前日志情况

```
show master status;
```

这两个参数供从服务器连接并复制

![](https://s2.ax1x.com/2019/05/12/E4phUe.png)


### 从服务器配置

```
server_id=2
relay_log=relay-log         #启用中继日志。在数据目录下有一个relay-kog.info里面保存了当前的中继日志和位置会主节点二进制文件的名字和位置。
read_only=on                #禁止用户写入数据，这一项的管理员和复制重放无效。
```

启动服务

```

mysql> FLUSH TABLES WITH READ LOCK;     #添加全局读锁,只允许读
mysql> reset slave;
mysql> CHANGE MASTER TO MASTER_HOST='192.168.3.137',MASTER_USER='guest',MASTER_PASSWORD='123',MASTER_LOG_FILE='master-log.000003',MASTER_LOG_POS=245; #远程连接，将上文查出来的日志文件和位置填写在上面
mysql> START SLAVE; #开始线程
```

```
#查询启动情况
mysql> SHOW SLAVE STATUS\G;

```

![](https://s2.ax1x.com/2019/05/12/E4p7vt.png)

>启动成功!
