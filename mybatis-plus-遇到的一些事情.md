---
title: mybatis plus 遇到的一些事情
date: 2021-02-12 14:00:20
tags:
- java
- mybatis
- spring boot
categories:
- mybatis
---


# mybatis plus 遇到的一些事情

最近由于学习微服务，学到数据库的时候，由于之前都是用的正常版的mybatis,这一次想用一下听说特别叼的mybatis-plus

## 个人感觉mybatis和plus的区别

其实我选择Mybatis而不是h2和JPA的最主要原因还是mybatis它可以自己写xml的sql文件，jpa的那种虽然看着可以减少代码编写啥的。我总感觉哪里有点不对劲，就比如说多表联查啥的，h2和jpa的处理方式就是有点花里胡哨。

mybatis plus(后面简称mp)的出现让困扰我多年的变成规范问题给解决了。那就是当我们完成基本的CRUD的时候，其实代码都差不多，也就是表名或者字段不一样嘛，表一多起来，就感觉那些代码有点臃肿了。我之前的博客项目的时候有想过用泛型去解决这个问题，但是后面由于自己技术不过关，弄了一个四不像，但是mp的BaseMapper<T>的出现真的惊艳到我了，并且mp也支持xml文件，我能不吹爆么？


## mp 的字段映射问题

我之前没有经历过真正上线的项目，所以好多都是学生思维的代码，而且我的数据库特比喜欢用驼峰命名，所以这次我用mp的时候不小心爆出来这样的一个错,字段映射错误.



[![yDgLPP.png](https://s3.ax1x.com/2021/02/12/yDgLPP.png)](https://imgchr.com/i/yDgLPP)

实体编写没有问题,但是一运行就显示这样的错

[![yDgzrQ.png](https://s3.ax1x.com/2021/02/12/yDgzrQ.png)](https://imgchr.com/i/yDgzrQ)

注意上面的字符是name_zh,经过查询就能看出来,mybatis plus 是直接讲驼峰命名自动转成下划线命名了.

### 解决方法

1. 修改数据库字段命名方式(个人感觉最好的,有种编程规范的感觉)
2. @TableField("nameZh") 利用注解将字段和属性对应起来
3. 关闭自动映射(最不推荐) mybatis-plus.configuration.map-underscore-to-camel-case=false