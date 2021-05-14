---
title: docker运维优化
date: 2019-11-27 16:25:24
categories:
- linux
tags:
- 运维
- springboot
- nginx
- docker
---

# docker运维优化

## springboot访问静态资源

部署完我的项目之后，在经过一系列的坑之后终于看似正常了。
但是，就在我更改完我的头像之后，我的图片突然爆出来了500错误，而且在我的面板上也爆出了这个错误。
![](https://s2.ax1x.com/2019/11/27/QC1awj.png)

经过测试得知,我的jar包是运行在 / 目录下的，而且我在写代码的时候，我的静态资源都是存到 ${user.dir}/img 里面。这就导致了我的img目录变成了'//img'。但是就算是这样，我的图片还是可以写入的，所以问题不是出现在这里，经过多方查阅发现，是我的资源映射写错了。
我直接写的是static-locations:file:图片上传地址,这个在windows和linux运行都正确，因为你的运行路径又不是，但是在docker上面就不一样了。

正确写法应该是:
```yaml
resources:
    static-locations: file:/${web.upload-path},classpath:/META-INF/resources/,classpath:/resources/, classpath:/static/, classpath:/public/
```

加一个/就行，翻了一下资料，大概是说加上/表示的是绝对路径。


## nginx刷新网页出现404现象

之所以出现这个现象，是因为我在匹配路径的时候没有加上这样的代码:
  try_files $uri $uri/ /index.html last;
上面的代码的意思是  查看本地磁盘文件有没有上面的网页的意思。
