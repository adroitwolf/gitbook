---
title: maven导入本地jar包
date: 2019-06-22T10:24:32.000Z
categories:
  - maven
tags:
  - maven
---

# maven导入本地jar包的方法

我们有的时候会找不到maven仓库的地址，只能先下载jar包，然后保存到maven仓库中。或者自己开发的jar包，然后自己导入到maven仓库中。

比如 我们现在在D盘由一个jar包，我们在命令行中输入

```bash
# Dfile 文件路径

# DgroupId DartifactId jar包信息

# Dpackaging 打包方式

mvn install:install-file -Dfile=D:\QRCode-3.0.0.jar -DgroupId=QRCode -DartifactId=QRCode -Dversion=3.0.0 -Dpackaging=jar
```
