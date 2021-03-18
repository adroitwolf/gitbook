---
title: Centos7安装nacos
date: 2021-01-08 10:48:42
categories:
- linux
tags:
- 安装教程
- linux
---

# Centos7安装nacos
打开nacos[官网首页](https://nacos.io/zh-cn/docs/quick-start.html)

在这里我们不使用git,而是直接用wget命令获取最新版本,所以我们需要先下载启动环境

> 请直接参考我的博客项目 Centos7安装java和maven

安装完环境之后我们需要下载并安装nacos
[![snBwi4.png](https://s3.ax1x.com/2021/01/08/snBwi4.png)](https://imgchr.com/i/snBwi4)

去github下载你想要下载的版本，这里我下载的是1.4.0版本
```bash
wget https://github.com/alibaba/nacos/releases/download/1.4.0/nacos-server-1.4.0.tar.gz
```
**解压**

```bash
tar -zxvf nacos-server-1.4.0.tar.gz -C /usr/local/
```

## 启动

```bash
cd /usr/local/nacos/bin

./startup.sh -m standalone

# 以单机模式启动
```


## 运行相关


账号密码： 

> nacos/nacos

网址：

IP:8848/nacos
