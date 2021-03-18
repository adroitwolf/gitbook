---
title: html自适应界面
date: 2019-05-19 14:44:19
categories:
- 前端
tags:
- 自适应
---

# html自适应界面
-----

学习过bootStarp之后，一直最自适应布局很感兴趣，尤其是BootStarp的栅格系统，所以做了一个html小demo，来记录一下自适应的过程


## 重要的css属性 弹性盒子

* display:flex;
* flex-wrap:wrap;
* justify-content:center;


接下来看一段代码

```html
<!-- html -->
<div class="container">
  <div class="service-box">
      <div class="service-icon">
          <i class="fa fa-window-restore"></i>
        </div>
      <div class="service-title">
          <span>网站商品管理</span>
      </div>
      <div class="service-desc">
          <span>这是管理网站所有商品功能，在这里可以管理用户上架的所有商品</span>
      </div>
  </div>
</div>
<!-- css -->
<style>
.container{
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.service-box{
  max-width: 33.33%;
  padding: 10px;
  text-align: center;
  cursor: pointer;
}
</style>
```
这是没有加弹性盒子的效果
![](https://s2.ax1x.com/2019/05/19/EjiQln.png)

这是加了弹性盒子的效果 [注意:这里截图的时候屏幕尺寸小于960px]
![](https://s2.ax1x.com/2019/05/19/EjiiQI.png)

**分析原因**
在我看来,这个弹性盒子其实就是让子类可以像是block一样排列起来,并且子类的max-width的百分比决定着他们一排能排几个


## 重要的属性 @media and screen(max-width:960px)


这个语句的意思是 如果屏幕尺寸小于960px那么就执行下面的代码

> 这个语句由一个前提,那就是这一定是个html:5文件

```html
<!-- html5 必须的属性 -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

下面看一下实例

承接上面的css代码
```css
@media screen and (max-width:960px) {
  .service-box{
    max-width: 45%;
  }
}
```

因为盒子最大是占据45%,也就是一行只能两行,就能看到上面的图的情况,正常来说,如果用电脑打开那个网页的话,一行会出现三个盒子的
