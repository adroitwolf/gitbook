---
title: vue走过的那些坑
date: 2021-03-25T11:27:02.000Z
tags:
  - vue
categories:
  - vue
---

# vue走过的那些坑

## vue修改数组

当我们想修改原数组的某一个元素的值得时候，并不会使页面动态渲染。

像是这样：

```html
<!-- 截取重要字段 -->
<div v-for=(item ,index) in data>{{item}}</div>
<button @click="change">修改<button>
<script>
data(){
    return {
        data:[1,2,2,3]
    }
}
methods:{
    change(){
        this.data[2] = 5
    }
}
</script>
```

array\[index] = value，vue扫描不到数组的变化。 下面提供了三个解决办法：

1. **this.$set(arrat,index,value)** （个人不是很喜欢）
2. array = newArray (之前用axios做过，但是现在还不是很确定是否可以用)
3. array.splice(index,1,newValue) (个人推荐)

## 关于vue中import的那些事

再写vue的项目的时候，import导入其他组件和export的组合是最常用的了。

那么就那我的项目为例子，我的项目导入了iview

```javascript
import { Button, Table, Message, Input } from 'iview'
import 'iview/dist/styles/iview.css';
```

可以看到 这个组件是从根路径出发的，而且这个路径是从node\_moudle这个文件夹内找的，如果不熟悉node\_moudle，可以看我之前的vue入门博客。

而且，我们可以看到，导入的组件是拿{}包裹这的，我们要知道，一般的组件像是vue官网上的那些，我们都是不拿{}包裹的。

> 那么，这两种方式有区别么？

答案是有的，{}导入的组件，需要用export {xxx,xxx,xxx}导出，而不需要{}的则需要用export default 'xxx'导出，而且 default 不可以导出多个，也就是说，我们可以因为某些逻辑将一些组件写在一个js文件中，然后用{}导出

## 关于vuex传参的一些事情

之前没有说vuex，这里说一下它的传参问题吧。对于我来说,vuex其实就可以看成一个静态的类，类里面分成四个模块

state => 存放data的

getters => get函数，获取变量的

mutataions => set函数，设置变量的

actions => 具体函数实现，比如利用axios之后将数据利用mutations提交的，也算是一个变相的set的函数吧

### 数据取不到？ 一直undefined?

本以为按照上面的做法去弄来着，但是当我在做完登陆功能，将token存到state里面，然后在axios封装的js文件将token取出来并且当作header的时候，数据一直是undefined,机智的我发现事情并不这么简单。吓得我马上开始了百度。

***

> 不出意料的是，百度总会在将你想要查询的东西替换成你最不想要的结果

查了一顿，啥也没查出来，其实一堆文章都阐述了这个问题，但是他们大多都是将什么token存入到localStorage里面。是的，这也是一个解决办法，不过我以后的getter不还是取不到数据么！

无奈下，打开gayhub,随便down一个大佬的vue项目，偷偷观察下(大佬不愧是大佬，代码结构清晰明了)。我发现大佬们的vuex代码无非是这种结构

![](https://s2.ax1x.com/2019/10/11/uHU6zV.png)

moudles -> 存放我们需要的一些数据，但是和上面我们想的不同的是，这些moudles里面并没有getters。而是单独抽出来了一个getters文件 getters -> 抽象出来的getters对象

index -> 注册moudles和getters组件

> 需要主要的是，这个getters组件是全局的，而我们在moudles里面写的state的局部的，所以仍然还是取不到任何东西，因此我们需要指明moudle名字 例如： avatarId: state => state.user.avatarId 和别的不一样的是，它在state后面又指明了user作用域

### mutataions只接受一个参数

这个问题其实困扰了我一会，因为如果你在actions里面写这样的话

commit("xxx", 变量1，变量2)

并且那个xxx的函数是(state,变量1，变量2)的时候，是不会报错的！而且只接收变量1，其余的自动undefined，一开始我还以为是我的变量不对呢，后来才发现是这的原因，不过这样解决办法也很明显了，我们可以弄一个复杂json过去，就像这样

commit("xxx",{1:xx,2:xx})

然后接收的时候正常接收就行，注意！一定用一个变量名接收，血与泪的教训啊。。。。
