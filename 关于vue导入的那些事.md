---
title: 关于vue导入的那些事
date: 2019-07-15 22:22:28
categories:
- vue
tags:
- 前后端分离
- vue
---

# 关于vue中import的那些事



再写vue的项目的时候，import导入其他组件和export的组合是最常用的了。

那么就那我的项目为例子，我的项目导入了iview
```javascript
import { Button, Table, Message, Input } from 'iview'
import 'iview/dist/styles/iview.css';

```
可以看到 这个组件是从根路径出发的，而且这个路径是从node_moudle这个文件夹内找的，如果不熟悉node_moudle，可以看我之前的vue入门博客。


而且，我们可以看到，导入的组件是拿{}包裹这的，我们要知道，一般的组件像是vue官网上的那些，我们都是不拿{}包裹的。

> 那么，这两种方式有区别么？


答案是有的，{}导入的组件，需要用export default {xxx,xxx,xxx}导出，而不需要{}的则需要用export default 'xxx'导出，而且 default 不可以导出多个，也就是说，我们可以因为某些逻辑将一些组件写在一个js文件中，然后用{}导出
