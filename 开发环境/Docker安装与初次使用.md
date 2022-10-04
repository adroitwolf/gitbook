# Docker的安装与初次使用

这几天一直在搞爬虫，偶尔发现一篇帖子发现scrapy可以在Docker里面运行，太好了！正好让我们的spring程序和爬虫相辅相成一起运行，然后让自己的网站变得更好\[幻想ing....]

好了，废话不多说了，本来这个就很简单的，一会还要写另一篇爬虫文章。

说一下我的Linux环境是centos7 docker的内置环境是centos6.5

## 安装命令

首先安装yum-utils

```bash
yum install -y yum-utils
```

然后添加阿里云镜像

```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

最后下载docker-ce

```bash
yum install docker-ce
```

安装完成！

开启开机自启

```bash
systemctl start docker
```

## nginx的使用

```bash
# 细心的你一定会发现，凡是下载命令一定pull，运行命令一定是run
docker pull nginx #下载nginx
#  -d 后台 -p 指定端口
docker run -d -p 80:80 nginx
```

访问本机端口 localhost:8080，看到nginx的欢迎界面，安装完成！

## tomcat的使用

由于我的项目是jar包格式，所以这里的tomcat只是凑字数的，目的只是看一下怎么样下载镜像和启动

```bash
docker pull tomcat
# 以my_tomcat的名称启动
docker run -d -p 8080:8080 --name my_tomcat tomcat
```

访问本机端口 localhost:8080，看到tomcat的欢迎界面，安装完成！

## mysql的使用

这里要说一下由于本人用的是mysql5.6的，而最新的mysql是8.x。我的tomcat会出现连接不上的状况，这是因为8.x的连接算法做出了更改。 所以为了方便，我的服务器下载的是mysql5.6的

```bash

docker pull  mysql:5.6

# 这里是密码为xxx的root的mysql开启方法
docker run -d -p 3306:3306 - v /usr/local/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=xxx mysql
```

下面介绍一下-v 和 -e 的作用，首先你可以想象这些容器启动后都是缓存到一个内存里面{这里只是方便大家理解才说是内存中，大家不要误解}，当你关闭之后或者重启里面的数据都会消失。说的高大上一点就是无所做数据持久化，那个-v这个参数就出来了，他是以':'分隔，其实只要记住':',所有参数:左边的都是值得你的物理主机\[也就是宿主机]，而右边的就是你的容器。上面的意思就是将本层目录下面的data文件夹来存mysql数据表，启动完是这样的。

![](https://s2.ax1x.com/2019/11/23/MbgQxK.png)

## redis的使用

当你的访问量上去的时候，缓存也是必须要上的，不然给数据库的压力是相当的大。

```bash
docker pull redis
```

这里需要说一下,docker的容器运行必须是要有一个前台程序的，不能所有的进程都是守护态。 因为我没有搞懂这个，所以一开始我的redis是一直起来就停止了。

如果对于redis不太了解的话可以看一下这里[centos下安装redis](https://whoami1231.github.io/2019/04/30/centos7%E5%AE%89%E8%A3%85redis/)

## docker常用的命令

看到这里你可能对镜像\[image]和容器\[container]有了一定的理解，镜像你可以想象成安装包，而容器就是安装包安装完的软件。如果我没有记错的话，一个安装包是可以安装很多此软件的。 同样的容器和镜像也是这样，镜像只能有一个版本的(但是不同版本的可以共存)。而容器就可以启动很多了，只要你的端口不冲突，随便起。

而且docker也有一个特点就是，每一个容器都是一个冲突域\[如果你学了计算机网络，那么冲突域一定不陌生]。这个意思就是说，即使你们容器内部端口一样，只要你们的宿主机端口不一致就好。

那么，我们下面开始讲解一下docker的常用命令。

### 下载镜像

```bash
docker run xxx:xxx
```

### 查看所有镜像

```bash

docker image ls
```

### 删除镜像

```bash
docker rmi xxx
```

### 删除无用镜像

> 为什么会有这个呢，是因为当你不小心删除了一个有容器且正在运行的镜像的时候，会留下一个名字为的镜像保证当前的容器的正常运行。可是有的时候我们还需要删掉这些来减轻我们的强迫症和空间

```bash

docker image prune
```

### 启动容器

```bash
docker run -d -p xxx:xxx -e xx  -v xx:xx --name xxx
```

### 停止容器

```bash
docker stop xxx
```

### 删除容器

```bash
docker rm xxx
```

### 查看当前docker进程

```bash

docker ps -a
```

### 进入容器内部

```bash

docker exec -it xxx
```
