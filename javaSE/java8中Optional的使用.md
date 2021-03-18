---
title: java8中Optional的使用
date: 2019-06-02 21:08:25
categories:
- java
tags:
- java
---

# java8中Optional的使用
-------


我们在之前的业务开发中，有时候会遇到搜查结果为空的情况，但是返回Null的时候，我们就必须要判断一下是否为null,如果是的话要么throw一个异常或者别的什么操作，例如账号登陆失败的时候。

这样显得代码非常臃肿，而java8得Optional完美解决了这个问题(主要是空指针问题)。

## 创建一个Optional实例

```java
Optional<String> name2 = Optional.empty();
Optional<String> name = Optional.ofNullable(null);
Optional<String> name3 = Optional.of("111");

```

这三个其实区别就是 第一个只是创建一个Optional实例，相当于Null。第二个允许传入一个Null的对象，如果用第三个Optional.of(null)的话，直接会爆出空指针异常。


## 获取Optional的值


```java
User user = new User("john@gmail.com","1234");
User user2 = new User("anna@gmail.com", "1234");


User result = Optional.ofNullable(user).orElse(new User);
User result2 = Optional.of(user).orElseGet(() -> new User);

User result3 = Optional.ofNullable(user)
                .orElseThrow(() -> new RuntimeExcetion("运行出错了"));

User result4 = Optional.ofNullable(user).Get();
```


下面来讲一下这几个方法的异同


orElse:有值就返回值，不管有没有值都会执行后面的操作(new 一个对象等等)，如果没有值才会返回结果。

orElseGet:有值就返回值，没有值才进行后面的表达式操作


> 如果传入的值都是Null的情况下，两个函数没有区别，但是不是空的情况下，orElseGet显然效率比较高

orElseThrow:有值就返回值,没有值抛用户定义的异常


Get:有值就返回值,没有值NoSuchElementException。

## 需要注意的点

如果对得到的Optional的函数不提前利用isPresent()函数判断里面是否有值的话，就Get函数，**一定**会爆出NoSuchElementException这个异常，不管里面是不是有值，对代码的逻辑产生一定的影响，所以最好还是先判断一下。
