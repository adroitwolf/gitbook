# idea热更新手段

由于在调试或者开发程序的时候，经常会遇到一个小小的bug改半天的情况，所以要一遍又一遍的重启项目，小一点的项目还好，稍微那种起一次就得花一分钟的程序改起bug来简直就是要命。还好，idea有一个收费插件"JRebel",可以让我们热更新

## 下载JRelbel

在setting=> plugins路径下，搜索 JRelbel，安装
> 上面的方法，在老版本的idea已经不适用，所以在使用老版本的IDE时，需要下载相应的插件版本
* 卸载jrelbel，并删除~/路径下的jrel缓存文件
* 去[官网](https://plugins.jetbrains.com/plugin/4441-jrebel-and-xrebel/versions)下载老版本的安装包
* 将安装包解压到idea的plugin文件夹下


## 破解

打开https://jrebel.qekang.com/，按照操作激活即可。

## 使用

通过JRebel的方式启动，当需要更新项目的时候，按下ctrl + shift + F9即可