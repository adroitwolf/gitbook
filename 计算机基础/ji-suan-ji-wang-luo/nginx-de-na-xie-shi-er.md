---
title: nginx的那些事儿
date: 2021-01-05T21:23:47.000Z
tags:
  - nginx
categories:
  - 运维
---

# nginx的那些事儿

## 配置文件

> 下面的是我的nginx里面的一些重点的配置

```
http {
	upstream eblogserver{
		server eblog_server_1:8099 weight=1;
		server eblog_server_2:8098 weight=2;
	}

    server {
        listen       80;
        server_name  localhost;

        location / {
            try_files $uri $uri/ /index.html last;
            root /usr/share/nginx/html;
            index  index.html;

        }
		location ^~ /server/ {
        # 保证服务器访问的是真实ip地址
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
			  proxy_pass http://eblogserver/;
		}

    }
}
```

```
我们要知道nginx是一个反向代理(说是这样说，其实就是代理一下服务器，正想代理就是代理客户端相当于我们的vpn)
```

我们要清楚下面几个重点的关键字

* http{} 用来嵌套多个server ：即我们的http请求，和配置代理缓存等等的web服务
* server{} 用来处理http请求
  * listen 监听端口
  * upstream 里面用来写反向代理服务器的地址和权重
  * location 后面跟正则表达式，用来处理用户请求的网址
    * proxy\_pass 反向代理的入口和上面的upstream一起用

配置文件写好了，如何看写的正不正确，用下面的命令

```bash

./nginx -t (测试文件) -c(指定配置文件路径) ../conf/redis.conf
```

如果显示successful，则证明nginx配置正确
