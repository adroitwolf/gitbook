# Linux设置固定IP

> 一般我们想要拿本地的linux当做服务器用来测试，但是我们的ip设置规则时DHCP,即自动分配ip原则，所以为了方便，这里记录了如何固定ip的方法

我们要设置ip，要知道VM的网络适配器有三种原则

* 桥接模式
* 主机模式
* NAT模式

我们的真实主机有两块对应的网卡
* VMnet0
* VMnet8
<!-- more -->

由于我们这里只固定ip，所以别的我们现在先不了解。

------
## 首先设置网关，打开VM的虚拟网络适配器

------

![网络适配器](https://s2.ax1x.com/2019/04/03/Ag94FH.png)

修改我们的子网ip和子网掩码，注意！上面的框只有里面的3你可以自行修改！

## 修改主机的VM网卡 VMnet8

--------

![](https://s2.ax1x.com/2019/04/03/Ag9pqO.png)

## 按照下图操作
-----

![操作](https://s2.ax1x.com/2019/04/03/Ag9bOf.png)

其中也是建议只有3更改

------

##  注意linux选择NAT链接后，开始修改IP！
----------
```
ip addr
#记住画圈的这个文件名，之后会用到
```
![](https://s2.ax1x.com/2019/04/03/AgClnK.png)


## 打开下面的文件
----------

我们首先要知道这里文件夹只要存储的是网卡的相关信息**/etc/sysconfig/network-scripts/**

```
 cd /etc/sysconfig/network-scripts
```



![](https://s2.ax1x.com/2019/04/03/AgPGrV.png)

```
vim ifcfg-ens33
# 修改下面的两处为
#BOOTPROTO=static
#ONBOOT=yes
ONBOOT=no #修改为自动连接

#并在下面添加网址等信息
IPADDR=192.168.3.135#静态IP
GATEWAY=192.168.3.2 #默认网关
NETMASK=255.255.255.0 #子网掩码
```

![](https://s2.ax1x.com/2019/04/03/AgPTqf.png)

## 重启网络服务
---------
```
service network restart
```

> 重启后如果无法正常上网，可以在配置文件上面写上DNS1=8.8.8.8这一项


