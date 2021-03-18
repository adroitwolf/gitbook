---
title: mybatis的走过的坑
date: 2019-07-26 08:16:47
categories:
- mybatis
tags:
- mysql
- mybatis
---

# mybatis的走过的坑

## mybatis字段映射
这里说到的字段映射指的是在没有任何配置的时候mybatis和tkmybatis是不能将数据库中的下划线转成java中驼峰命名的
我们需要开启设置：
```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

## 关于MyBatis自增主键那些事

有于当初利用SSM框架编写Mybatis的时候，mybatis的mapper文件等都是逆向生成的，所以用的时候非常方便，但是除了一点，就是插入数据的时候，主键是必须要写的，但是为了安全和方便(其实懒占主要因素)，否则就会爆出sql语句错误，最后解决办法如下:

1. mysql表中要把自增主键的选项打开


其实大部分人只要打开自增主键就已经OK了，但是有的时候电脑抽风还是报错，那么就需要更改逆向出来的源码了

找到你想插入的地方<insert id='xxx'>

然后添加这么一个关键字:


```xml
 <insert id="insertSelective"  
 useGeneratedKeys="true" parameterType="xxx">
```
这个useGeneratedKeys就是自增主键的意思
