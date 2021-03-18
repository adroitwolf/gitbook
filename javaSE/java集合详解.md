---
title: java集合详解
date: 2019-05-03 08:29:36
categories:
- java
tags:
- 数据结构
- javase
---

# java集合详解

>这里我们分析Collection和Map
[大图查看详情](https://img-blog.csdn.net/20160124221843905)



# javaCollection的分析

Collection是在java.util里面

![Java队列](https://s2.ax1x.com/2019/04/02/AywLb6.png)

> 集合里面是一定是存放对象的，不能存放基本数据类型，像是int,也是先转换成Integer然后在放在集合中。

## List

<!-- more -->

* List是一个接口，而下面有ArrayList和LinkList两个实现类，关于Array和Linkd的区别，详见数据结构。
*  List是有顺序的！
* Vector(同步) 非常类似ArrayList，但是Vector是同步的

## Set

* Set也是一个接口，下面有HashSet和TreeSet两个实现类
* HashSet: HashSet类按照哈希算法来存取集合中的对象，存取速度比较快。 里面的对象可能会被打乱，但是效率更高。
* TreeSet: TreeSet类实现了SortedSet接口，能够对集合中的对象进行排序。 保证对象的有序性
* Set里面没有重复数据  这样可以去重
| 重要方法 | 作用  |
| ----------- | ------------- |
| iterator( ) | 主要用于递归集合，返回一个Iterator()对象，利用Iterator()去接收 |

> 《后期添加》注意：这个Iterator是Collection下面的 。

## Deque && Queue
* 其实Deque是Queue的一个接口，是父子关系
* 注意他的子接口Deque下面有两个实现类ArrayDeque...

    ![](https://s2.ax1x.com/2019/04/28/EQg5N9.png)


* Queue是一个普通队列，详情见数据结构

* Deque是Queue的子接口，他是一个双端队列，可以支持FIFO(First Input First Output)和LIFO原则。


### 队列基础
* BlockingQueue阻塞队列:当添加队列的时候，可以去设置等待时间。即当队列满的时候，可以等待，超过等待时间返回false。取队列的时候也可以等待。
* ConcurrentLinkedQueue非阻塞队列，添加和取队列的时候不去等待。
* 队列原则
* * 先进先出（FIFO）：先插入的队列的元素也最先出队列，类似于排队的功能。从某种程度上来说这种队列也体现了一种公平性。
* * 后进先出（LIFO）：后插入队列的元素最先出队列，这种队列优先处理最近发生的事件。　　
> Tips: 非阻塞队列比阻塞队列效率要好，但是确实不安全的

--------------------

### 非阻塞队列

#### ConcurrentLinkedQueue<E>()

| 方法|作用|
|:---|:----|
| offer|将Object添加到队列的尾部|
|poll|从队列头取Object并且将其在队列中删除|
|peek|从队列头取Object并且不将其在队列中删除|


###  阻塞队列

#### BlockingQueue<E>(Size)

![常用队列](https://s2.ax1x.com/2019/04/08/A4Oysx.png)

* ArrayBlockingQueue 数组型队列，必须要指定size

*  linkedBlockingQueue  链表型无界队列，可以指定size

| 方法|作用|
|:---|:----|
| offer()|将Object添加到队列的尾部<非阻塞式>|
| offer(Object,time,timeUtil)|将Object添加到队列的尾部<阻塞式>,设定队满的时候等待时间|
|poll|从队列头取Object并且将其在队列中删除<非阻塞式>|
| poll(Object,time,timeUtil)|队列头取Object并且将其在队列中删除<阻塞式>,设定队空的时候等待时间|
|peek|从队列头取Object并且不将其在队列中删除

> 这里要注意： add()方法和offer方法的区别，add方法在队列满的情况下会抛出一个异常。而offer并不会。|
