---
title: java基础知识(一)
date: 2021-02-01 23:25:08
tags:
- java
categories:
- java
---

# java基础知识(一)

## java数据类型
八大基础数据类型： byte,char,short,int,long,float,double,boolean

> 包装类型是类/对象，默认是Null,而int这种的基础数据类型默认为0(牛客中出现过)

## String 详解

### 内存问题

```java

A:{
    String a = "a";
    String b = "a";
}

B:{
    String a = new String("a");
    String b = new String("a");
}

```


上面两个代码块，有区别没？
答案肯定是有的，A代码块的执行原理是这样的

首先在自己线程的栈帧中生成一个引用型变量a,然后再到堆里面的常量池中寻找有没有a的对象，如果有就把地址直接转向那里，如果没有，新建；b也是同理。

> 注意，java中只要是那种包装数据类型都是创建一个相当于指针的东西。



**提示**这个是包装数据类型，如果是char这种的呢？则是直接存在栈中


B代码块你就考虑只要是new，那肯定是直接再heap里面直接创建一个新的对象啦。


## java编译

java编译命令: javac xxx.java

编译后会生成class文件，生成的文件个数和源程序中类的**个数一致**。

## java值传递和引用传递

网上的教程大多都是参差不齐，这里整理一下我自己的思路。

值传递： 基本数据类型

引用传递： String等各种对象，数组。

## Java中Stream的使用

### forEach

这个我们肯定特别熟悉，遍历嘛。但是这里有一个注意的点，Collection包下面的集合都已经实现了forEach方法，也就是说，我们不需要将List对象先转换成stream对象，再用forEach方法

### map
map对象，他可以做一个对数据的处理，最后return出来的那个东西变成了一个steam流,举个例子

```java

List<User> users;
 // 这里是个有好多数据的user列表，我们想吧里面的属性转出两个来怎么办？直接用map

// 假设UserChange里面就一个id和name,其他的都删掉了
 List<UserChange> lists = users.stream().map(item->{
     xxxxx //去掉那些属性
     return A;
 }).collect(Collections.toList());

```



### filter

这个是筛选出来符合filter里面条件的list，这里不举例了