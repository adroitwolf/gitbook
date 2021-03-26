---
title: 关于vue传参问题
date: 2019-07-27 06:54:30
categories:
- vue
tags:
- vue
- 前后端分离
---


# 关于vue传参问题


传参的时候我比较喜欢利用params方式传参


比如

> 标签式传参

```html
<router-link :to="{name:'xx',params:{xxx:xxx}}"></router-link>
```



> 编程式传参

```javaScript
this.$router.push({
  name:'xxx',
  params:{
    xx:xx,
    xx:xx
  }
  })
```


跳转过去的网页利用this.$route.params.xx接收。

但是一定要注意的一点是,this.$router和this.$route的区别！！！

* this.$router

router 实例。

* this.$route

当前激活的路由信息对象。这个属性是只读的，里面的属性是 immutable (不可变) 的，不过你可以 watch (监测变化) 它。

