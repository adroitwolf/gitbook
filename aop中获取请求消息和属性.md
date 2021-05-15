---
title: aop中获取请求消息和属性
date: 2019-07-23 18:25:26
categories:
- springboot
tags:
- springboot
- 前后端分离
---

# aop中获取请求消息和属性

----


这篇博客仅仅提供代码。

由于我们利用@interface并且搭配上Aspect里面的@annotation可以做到切面的效果，所以我在项目上运用了这项技术，他们的参数是ProceedingJoinPoint joinPoint,并没有HttpServletRequest,但是我们又需要请求头的信息，所以代码如下：


```java
ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
       HttpServletRequest servletRequest = attributes.getRequest();
```
