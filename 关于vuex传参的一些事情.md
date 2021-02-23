---
title: 关于vuex传参的一些事情
date: 2019-07-28 10:34:08
categories:
- vue
tags:
- vuex
- 前端
---
# 关于vuex传参的一些事情

之前没有说vuex，这里说一下它的传参问题吧。对于我来说,vuex其实就可以看成一个静态的类，类里面分成四个模块

state =>  存放data的

getters => get函数，获取变量的

mutataions => set函数，设置变量的


actions => 具体函数实现，比如利用axios之后将数据利用mutations提交的，也算是一个变相的set的函数吧

## 数据取不到？ 一直undefined?
本以为按照上面的做法去弄来着，但是当我在做完登陆功能，将token存到state里面，然后在axios封装的js文件将token取出来并且当作header的时候，数据一直是undefined,机智的我发现事情并不这么简单。吓得我马上开始了百度。


-------
> 不出意料的是，百度总会在将你想要查询的东西替换成你最不想要的结果

查了一顿，啥也没查出来，其实一堆文章都阐述了这个问题，但是他们大多都是将什么token存入到localStorage里面。是的，这也是一个解决办法，不过我以后的getter不还是取不到数据么！


无奈下，打开gayhub,随便down一个大佬的vue项目，偷偷观察下(大佬不愧是大佬，代码结构清晰明了)。我发现大佬们的vuex代码无非是这种结构

![](https://s2.ax1x.com/2019/10/11/uHU6zV.png)

moudles ->  存放我们需要的一些数据，但是和上面我们想的不同的是，这些moudles里面并没有getters。而是单独抽出来了一个getters文件
getters -> 抽象出来的getters对象

index -> 注册moudles和getters组件

> 需要主要的是，这个getters组件是全局的，而我们在moudles里面写的state的局部的，所以仍然还是取不到任何东西，因此我们需要指明moudle名字
> 例如： avatarId: state => state.user.avatarId  和别的不一样的是，它在state后面又指明了user作用域


## mutataions只接受一个参数

这个问题其实困扰了我一会，因为如果你在actions里面写这样的话

commit("xxx", 变量1，变量2)

并且那个xxx的函数是(state,变量1，变量2)的时候，是不会报错的！而且只接收变量1，其余的自动undefined，一开始我还以为是我的变量不对呢，后来才发现是这的原因，不过这样解决办法也很明显了，我们可以弄一个复杂json过去，就像这样

commit("xxx",{1:xx,2:xx})

然后接收的时候正常接收就行，注意！一定用一个变量名接收，血与泪的教训啊。。。。
