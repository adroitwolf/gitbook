# Centos7安装nginx

首先去官网下载想要的版本然后解压到linux上面

解压之后是这样的

[![sF6dHJ.png](https://s3.ax1x.com/2021/01/05/sF6dHJ.png)](https://imgchr.com/i/sF6dHJ)

> 这里面configure 就相当于我们windows上面的exe安装包

* 我们需要知道的是，nginx是编译安装的，不像tomcat那样，解压之后直接可以使用。
* nginx是在c语言编译的，所以我们开始先安装所需要的环境

> centos版本

```bash
yum install -y gcc gcc-c++ pcre pcre-devel zlib  zlib-devel openssl-devel
```

> debain版本

```bash
sudo apt-get install build-essential

sudo apt-get install libtool

sudo apt-get install libpcre3 libpcre3-dev

sudo apt-get install zlib1g-dev

sudo apt-get install openssl

apt-get install libssl-dev
```

接下来 安装nginx

具体命令为：

```bash

./configure --prefix=/usr/local/nginx
# prefix指的是安装路径
```

然后nginx会检查环境，完全通过后

```bash

make && make install 
#先编译，然后安装
```

## nginx 的常用命令

* 启动命令： ./sbin/nginx
* 重启命令： ./sbin/nginx -s reload
* 停止命令: ./sbin/nginx -s stop
