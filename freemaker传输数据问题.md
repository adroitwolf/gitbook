---
title: freemaker传输数据问题
date: 2019-04-13 19:56:01
categories:
- ssm错误笔记
tags:
- 错误笔记
- ssm
- freemaker
---

# FreeMaker传输数据问题

-----

* 正常输出方程式 ${XXX}
* 如果页面需要字符串，那么我们需要 在变量两边加引号 '${XXX}'
* 如果传输的是数字，在超出100的时候，会自行加逗号，例如1,400,解决办法： ${XXX?c}

<!-- more -->

* 如果你的字段是null的话，freemaker直接引用是会报错的
  ![](https://s2.ax1x.com/2019/04/13/AL5JCF.png)

  这样需要你在可能出现错误的字段上加一个判断他是否为空，$(XXX!"啊，这是null")



* 其实在判断字符串的时候 例如 <#if sex= “男”> 和 <#if sex== “男”> 是一样的