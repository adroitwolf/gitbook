---
title: vue全家桶搭建项目和入门理解
date: 2019-07-07T17:37:20.000Z
categories:
  - vue
tags:
  - 前后端分离
  - vue
---

# vue全家桶搭建项目和入门理解

***

由于目前需要做一个博客项目，所以需要先搭建前端，springboot + vue是一个不错的选择，所以这篇博客写的是我目前对vue的理解。

## vue脚手架

其实这里没有网上说的这么复杂。完全可以这么理解，他们说的一些vue知识属于框架,类似于bootstrap那种的(其实并不是，不过你可以先这么理解)，需要你去理解一些标签啊，语法啊那种东西，然后这个脚手架是你搭建的一个**前端**项目，这里你可以理解成java后端的那种搭环境，比如spring啊什么的。

## 初始化vue脚手架(vue-cli 2.x)

在保证自己电脑上安装了node.js前提下，我们需要安装vue环境和webpack.

```bash
npm install vue
npm install -g vue-cli #安装脚手架
```

这里的webpack其实就是一个工具将你写的vue打包成html,js,css文件

* 安装一个vue-cli项目

```bash
vue  init webpack [ProjectName]
```

这里会下载一大堆东西，然后出现一些让你确认的东西，比如说是项目名啊，作者啊，路由啊(这个一会讲)，一直敲回车就ok。

* 项目目录结构

安装完成后，我们的项目结构是这个样子的 ![](https://s2.ax1x.com/2019/07/07/ZBb8zt.png)

package.json -> 这个是项目必须的，每次安装插件什么的都需要这个json，你可以把它想象成gradle或者maven

index.html -> 网页的门面，后面的初始化Vue对象也是初始在了这个地方

test -> 单元测试，相当于Junit

static -> 这个是在webpack打包后直接将里面的东西直接转移到了dist/static

src -> 整个项目基本上你都要在里面写东西

src-> assets -> 项目的静态资源，存放一些js,css,图片什么的。

node\_modules -> 你下载的一些插件都在里面，比如路由,jQuery

config ->配置文件包

build ->项目启动的一些包，比如端口什么的

* 项目启动

```bash
npm run dev # 启动项目

npm run build # 将你写的项目打包成html,一般都是在项目写完的时候这么干
```

## 初始化vue脚本(@vue/cli 3.x)

到我目前开始修改之前vue的博客的时候 vue cli 已经升级到4.x,但是因为4.x是基于Vue 3.x版本的，和我之前使用的Vue2.x有很大的区别。所以现在暂时先使用3.x。

## 安装

```bash
  npm install @vue/cli 3.11.0
  vue -V # 只要不是什么4.x就ok
```

### 创建项目

```bash
  vue create [name]
```

## 安装必要插件

* 首先是路由和vuex,由于一开始创建项目的时候提示我们是否安装路由了，所以这里我不需要安装，直接安装vuex.

```bash
npm install vuex --save # save 是保存到本地的意思
```

> 上面都是Vue Cli 2.x 必须安装的，VueCli3 在创建项目时可以自定义。

### vue路由器入门

我们要清楚，vue里面切换页面是用vue路由来实现的，我们初始创建的项目里面是存在src/router/router.js这个文件，这里面的文件是我们所有配置路由的文件，目前我的里面随便写了一个页面，具体代码如下:

首先是我们在compnents/下面随便创建一个页面

```html
<template>
    <Input v-model="value" placeholder="Enter something..." style="width: 300px" />
</template>
<script>
    export default {
        data () {
            return {
                value: ''
            }
        }
    }
</script>
```

上面的代码是抄自iview官网的一个input标签

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'

Vue.use(Router)

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: ()  => import('@/components/HelloWorld')
    },
    {
      path: '/login',
      name: 'login',
      component: () => import('@/components/login')
    }
  ]

})
```

mode:'history'是将网页上的#删除掉。
