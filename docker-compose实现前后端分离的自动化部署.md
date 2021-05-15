---
title: docker-compose实现前后端分离的自动化部署
date: 2019-11-21 14:17:26
categories:
- linux
tags:
- docker
- docker-compose
- 安装教程
---

# docker 实现前后端分离的自动化部署

时隔一个学年，我再一次租起了阿里云服务器(哎呀真香啊)，其实一开始我也是拒绝的，但是由于我的电脑配置日渐落后，连启动一个虚拟机都要等上半天。我就想说：买！咱不受那个窝囊气。

具体配置是这样的

![](https://s2.ax1x.com/2019/11/18/McPwsf.png)


好了，废话不多说，这篇文章是搭建web环境的，看过之前的文章都知道，技术选型肯定就是docker + springboot + vue了


## 下载环境

这里我默认大家docker已经装好了，如果没有装好，请参考我之前的文章
我们现在需要下载的是docker-compose
```bash
# 下载和安装
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

# 赋予权限
chmod +x /usr/local/bin/docker-compose

# 检查版本

docker-compose -vesion
```

## 项目架构

![](https://s2.ax1x.com/2019/11/23/Mbfslq.png)

这是我的项目的所有的工程文件，下面讲解一下这些都是些啥玩意。

首先介绍一下前后端分别都是我的github项目，地址是 [前端](https://github.com/whoami1231/blog-vue)|[后端](https://github.com/whoami1231/blog-java).


## 前端准备
先介绍一下前端文件，也就是web文件夹
我们的前端是通过vue打包后部署到nginx，因为要做后端的负载均衡。
### 修改源代码
首先我们需要修改文件，因为我们要部署到docker里面。容器与容器之间会有冲突域产生，localhost是绝对访问不到自己的项目的，所以我们需要将我们请求后端的axios的bashUrl修改一下改成这样-> http://example.com/server

> 中间的域名可以换成你的服务器ip地址，后面的server是一个声明，好让nginx反向代理到后端

### 打包项目文件

然后我们要给我们的前端打包，打开项目根目录然后输入下面的命令

```bash

npm run build
```

我们会得到一个dist文件夹，也就是我们的前端文件[我还没有搞cdn加速，太卑微了]

### 编写nginx配置文件 nginx.conf
具体文件如下:

```text

worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    client_max_body_size 4m;

	upstream eblogserver{
		server eblog_server_1:8099 weight=1;
		server eblog_server_2:8098 weight=2;
	}

    sendfile        on;
    keepalive_timeout  65;
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/vnd.ms-fontobject font/ttf font/opentype font/x-woff image/svg+xml;
    gzip_vary off;
    gzip_disable "MSIE [1-6]\.";
    server {
        listen       80;
        server_name  localhost;
        location / {
            try_files $uri $uri/ /index.html last;
            root /usr/share/nginx/html;
            index  index.html;

        }
		location ^~ /server/ {
			proxy_pass http://eblogserver/;
		}

    }
}

```

> 上面的upstream是加权轮转进行负载均衡的意思

### 编写Dockerfile

具体文件如下

```bash
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
COPY ./dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
```


## 后端

### 修改源代码
因为我这里使用的idea的IDE，所以我直接在这里面打包，但是因为有冲突域的存在，所以我们通过127.0.0.1是访问不到我们的mysql的，所以我们需要修改我们的配置文件，而这里又因为打包的时候需要检查所有的test文件，包括数据库是否可以连接。所以这里就会产生一个很大的冲突，经过度娘得知，我们可以跳过安全检查。

打开我们的pom.xml，修改里面的properties

```xml

<properties>
    <java.version>1.8</java.version>
    <skipTests>true</skipTests><!--添加 -->
</properties>
```


然后我们修改我们的数据库连接路径

url: jdbc:mysql://docker_mysql:3306/eblog?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
> 这里面的docker_mysql是我们一会启动的mysql的服务


### 打包jar

具体操作如下
![](https://s2.ax1x.com/2019/11/23/Mboco8.png)

### 编写Dockerfile

文件代码如下


```bash
# 继承java 1.8
FROM java:8
# 容器的操作者
MAINTAINER whoami
# 数据卷
VOLUME /temp
# 修改jar包名称
ADD app-0.0.1-SNAPSHOT.jar eblog-server.jar
RUN bash -c 'touch /eblog-server.jar'
ENTRYPOINT ["java","-jar","eblog-server.jar"]
```

### 构建镜像

```bash
docker build -t eblog-server .
```

> 其实这里不用编成镜像也行，只是为了展示不同的构建方法才这么写


## docker-compose.yml

具体代码如下

```yml
version: '3'
services:
    nginx:
        restart: always
        container_name: nginx
        build:
            context: ./web
            dockerfile: ./Dockerfile
        ports:
            - 80:80
        depends_on:
            - eblog_server_1
            - eblog_server_2
        volumes:
            - ./web/log:/var/log/nginx
    eblog_server_1:
        container_name: eblog_server_1
        image: eblog-server
        ports:
            - 8099:8099
        depends_on:
            - docker_mysql
            - docker_redis
        volumes:
          - ./server/img:/img
    eblog_server_2:
        container_name: eblog_server_2
        image: eblog-server
        ports:
            - 8098:8098
        depends_on:
            - docker_mysql
            - docker_redis
        volumes:
          - ./server/img:/img
    docker_mysql:
        container_name: docker_mysql
        image: mysql:5.6
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: 123
            MYSQL_ROOT_HOST: '%'
        ports:
            - 3306:3306
        volumes:
            - ./mysql/data:/var/lib/mysql
    docker_redis:
      container_name: docker_redis
      image: redis
      restart: always
      command: redis-server /usr/local/etc/redis/redis.conf
      ports:
        - 6379:6379
      volumes:
        - ./redis/data:/data
        - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
```
下面讲解一下文件配置
- version -> 没什么好说的 版本而已
- services -> 需要运行的服务
- nginx,eblog_server_1 .. 标识符
- depends_on -> 等里面的服务启动完成后才会启动
- container_name -> 容器名称
- image -> 使用docker里面的镜像

- build

这个是重点

如果使用Dockerfile创建镜像的话，比如我们的前端，我们需要利用build先构建镜像，然后再创建容器。
而那个context就是构建的目录,我们进入web目录下才会后我们的dockerfile，并且在web目录下才会由我们的nginx.conf文件



## 启动服务

打包到服务器之后输入下面的命令


```bash
# 后台运行
docker-compose up -d
```
```bash
# 前台运行
docker-compose up
```

运行成功后，打开运行中的服务:

![](https://s2.ax1x.com/2019/11/23/MbHMRg.png)

发现运行成功

## 停止服务

```bash
dcoker-compose down
```
