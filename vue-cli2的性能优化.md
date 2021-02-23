---
title: vue-cli2的性能优化
date: 2019-11-25 19:25:19
categories:
- vue
tags:
- 性能优化
- vue
---

# vue-cli2的性能优化

前几天终于将自己的项目打包到了服务器上，但是由于本人买的阿里云的学生服务器，所有网速慢点。但是最不能忍的是css加载速度直接到了18s!所以才有了这篇文章
![](https://s2.ax1x.com/2019/11/25/Mvw9rF.png)


## 文件分析
我们首先在项目上运行
```bash
 npm run build --report
```

运行成功后自动打开 http://127.0.0.1:8888
我们会看到下面的界面
![](https://s2.ax1x.com/2019/11/25/Mv07kj.png)


## gzip


由于大部分的游览器已经支持gzip了，所以我的第一步优化就是从gzip入手。

当我们查看刚才打开的页面并且点击gzip按钮的时候，我们看到数据压缩的大小

![](https://s2.ax1x.com/2019/11/25/MvBS74.png)

打开我们的项目，查看我们的config/index.js

会发现我们里面有productionGzip这个选项，它默认是false的，所以我们需要将这个设置成true,然后执行打包命令。

> 注意如果提示你少插件安装即可，因为我的版本是vue-cli2，所以插件版本不能过高，这里我推荐一个插件。

```bash
npm install compression-webpack-plugin@1.1.12
```

好的，打包之后发布我们可以看到我们页面加载速度已经稍微便快一些了。
![](https://s2.ax1x.com/2019/11/25/Mvhmbn.png)


由于我是部署在nginx上面的，所以我们需要nginx开启一下gzip选项：

```text
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/vnd.ms-fontobject font/ttf font/opentype font/x-woff image/svg+xml;
    gzip_vary off;
    gzip_disable "MSIE [1-6]\.";
```

gzip选项可以写在http、server、location下面

## cdn


一谈到cdn我就头疼，因为我的项目是vue-cli2的，而且说实话哈，我webpack根本就没学。当时认为直接搭框架干就完了呗，现在想想看，当时的自己真是蠢，早晚都得学。


好了，我当初再往上看了好多相关的教程。哎呀，这里我总结一下吧。

首先我们要清楚的一点是,vue如果是将你下载的第三方插件,比如说jquery啥的利用cdn优化的时候，一定要只将那些在vue首页加载的插件弄上cdn，其他的插件弄上cdn后，反而会变慢。

为啥嘞？
问的好！我们搞cdn的时候是将那些cdn链接弄到index.html里面引入的，就像这样:

```html

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.4.1/dist/jquery.min.js"></script>
    <title>逝痕</title>
</head>

```

如果在首页塞上太多的资源链接的话，就会弄得很慢。

> 还有一件事~，就是按需引入的插件不要用cdn

由于本人的框架用的iview,所以如果利用cdn的话会产生很多bug。
所以我把眼光放到了vue、vue-router、vuex这样的插件。


所以，我们需要先找到对应自己版本的cdn,这里我推荐几个网站：[jsdelivr](https://www.jsdelivr.com/)、[unpkg](https://unpkg.com)

我对照上面的项目状况找到了我的版本，index.html代码如下:


```html

<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>逝痕</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vuex@3.1.2/dist/vuex.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue-router@3.1.3/dist/vue-router.min.js"></script>

</head>

<body>
    <div id="app"></div>
</body>
</html>
```

vue-cli2需要改一下webpack配置，我们打开/build/webpack.base.conf

改动如下:

```javascript
module.exports = {
    externals: {
        'vue': 'Vue',
        'vue-router': 'VueRouter',
        'vuex': 'Vuex'
    },
// ...
}

```
webpack的配置我推荐你看这个网站:[外部拓展](https://webpack.docschina.org/configuration/externals/)

运行我们的打包命令

```bash

npm run build --report
```

经过cdn我们发现我们的项目是这样的

![](https://s2.ax1x.com/2019/11/26/QSedxI.png)


## 图片压缩

cdn搞完了，但是说实话提升效果不是很好。打开我的控制台发现我的网站图片加载需要10s,所以我又将我的重心转到了图片压缩上面。

这里推荐两个网站:

http://www.bejson.com/ui/compress_img/

https://tinypng.com/


压缩之后替换图片即可


## 图片渐进式加载

## 关闭map资源

我们打包之后看到我们有很多的map后缀的js,其实这些东西都是可以不要的。

打开 /config/index.js

将productionSourceMap设置成false即可
