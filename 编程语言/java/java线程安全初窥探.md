---
title: java线程安全初窥探
date: 2019-03-29T10:08:37.000Z
categories:
  - java
tags:
  - java架构
  - 线程安全
---

# java线程安全

* 知识点： 线程同步 线程并发
* 问题描述：在当处理全局变量的时候，当两个或者以上的线程处理同一个\*\* 全局 \*\*变量的时候，可能会出现冲突问题。

***

## java 同步函数

首先看一下问题场景

```java
package com.Thread.Test;

/**
 * 抢票问题的一个案例分析
 */

class ThreadTrain implements Runnable{

    private int TrainCount = 100;


    @Override
    public void run() {
            while (TrainCount>0){
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                sale();
            }
    }

    private void sale(){
        System.out.println(Thread.currentThread().getName()+"：正在出售第"+(100-TrainCount+1)+"张票");
        TrainCount--;
    }
}


public class Main {

    public static void main(String[] args) {
    //开辟两个线程
        ThreadTrain threadTrain = new ThreadTrain();
        Thread t1 = new Thread(threadTrain, "窗口一");
        Thread t2 = new Thread(threadTrain, "窗口二");

        t1.start();

        t2.start();

    }
}
```

> ![Console ：](https://s2.ax1x.com/2019/03/29/A0Gqk6.png) ![Console :](https://s2.ax1x.com/2019/03/29/A0GHTx.png) 可以看到 上图 会出现两个线程同时贩卖一张票的情况，而且最后会出现贩卖101张票的时候

* 那么为什么会产生这样的问题呢？
* 原因分析： 是因为两个线程当时同时处于运行状态，那么他们接收的全局变量的value是相等的，那么就会出现贩卖同一张票的情况，这样就会产生线程不安全的情况！
* 解决方案分析：就像是购票的原理一样，会对数据库进行锁表，来实现数据同步，java也有锁这种东西
*
  * synchronize ---- >自动锁
*
  * lock --> jdk1.5 手动锁

### synchronize 解决代码：

```java
package com.Thread.Test;

/**
 * 抢票问题的一个案例分析
 */

class ThreadTrain implements Runnable{

    private int TrainCount = 100;


    @Override
    public void run() {
        while (TrainCount>0){
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            sale();
        }
    }

    //synchronized 分成 函数 和标识两个  
    private synchronized void sale(){ //this锁
//        synchronized (this){
        //加一个判断 判断最后一张票的两个线程情况
        if(TrainCount>0){
            System.out.println(Thread.currentThread().getName()+"：正在出售第"+(100-TrainCount+1)+"张票");
            TrainCount--;
        }

//        }
    }
}


public class Main {

    public static void main(String[] args) {
        ThreadTrain threadTrain = new ThreadTrain();
        Thread t1 = new Thread(threadTrain, "窗口一");
        Thread t2 = new Thread(threadTrain, "窗口二");

        t1.start();

        t2.start();

    }
}
```

> synchronize 方法 比较方便，但是拓展性不高，资源占用大

### lock锁的解决办法

```java
package com.Thread.Test;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class ThreadTrain implements Runnable{

    private int TrainCount = 100;
    private Lock lock = new ReentrantLock();

    @Override
    public void run() {

        while (TrainCount>0){
            try {
                Thread.sleep(50);
                sale();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    //synchronized 分成 函数 和标识两个
    private  void sale(){ //this锁

        try {
            lock.lock();
            if(TrainCount>0){
                System.out.println(Thread.currentThread().getName()+"：正在出售第"+(100-TrainCount+1)+"张票");
                TrainCount--;
            }
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }

    }
}


public class LockTest {
    public static void main(String args[]) throws InterruptedException {
        ThreadTrain threadTrain = new ThreadTrain();
        Thread t1 = new Thread(threadTrain, "窗口一");
        Thread t2 = new Thread(threadTrain, "窗口二");

        t1.start();
        Thread.sleep(40);
        t2.start();
    }
}
```

***

![result](https://s2.ax1x.com/2019/03/29/A0RwV0.png)

* 值得注意的是 如果将函数标识成synchronized锁的话，这个函数只是一个this锁，但是如果使用synchronized函数的话，函数的变量可以定义任何Object类型
* 如果是用lock锁的话，如果代码在加锁的过程中，程序崩溃报错，那么这个锁就一直会在锁定状态，所以应该用try catch的时候，在finally加上unlock保证锁的正常运行
* 通过锁来实现数据同步，来解决一个像是抢票的并发问题。

***

## java静态同步函数

* 如果将synchroized锁函数名前面加上static 标识限制的时候，那么这个函数不再是一个this锁，而是锁本类的java对象 例如

```java
private static synchronized void sale(){ //this锁
//        synchronized (this){
        //加一个判断 判断最后一张票的两个线程情况
        if(TrainCount>0){
            System.out.println(Thread.currentThread().getName()+"：正在出售第"+(100-TrainCount+1)+"张票");
            TrainCount--;
        }

//        }
    }
```

是和下面的功能是一样的

```java
 private  void sale(){ //this锁
        synchronized (ThreadTrain.class){
        //加一个判断 判断最后一张票的两个线程情况
        if(TrainCount>0){
            System.out.println(Thread.currentThread().getName()+"：正在出售第"+(100-TrainCount+1)+"张票");
            TrainCount--;
        }

        }
    }
```

## 线程死锁

* 千万不要在数据同步的时候在嵌套一个数据锁，这样可能产生一个线程死锁
* 具体代码如下

```java
package com.Thread.Test;

/**
 * 抢票问题的一个案例分析
 */

class ThreadTrain implements Runnable{

    private int TrainCount = 100;

    private Object oj = new Object();

    public boolean flag = true;

    @Override
    public void run() {
        if(flag){
            while (TrainCount>0){

                synchronized (oj){
                    //加一个判断 判断最后一张票的两个线程情况
                    sale();

                }
            }
        }else{
            while(TrainCount>0){
                sale();
            }

        }

    }

    private synchronized void sale(){
        synchronized (oj){
        //加一个判断 判断最后一张票的两个线程情况
        if(TrainCount>0){
            try {
                Thread.sleep(40);
            } catch (Exception e) {

            }

            System.out.println(Thread.currentThread().getName()+"：正在出售第"+(100-TrainCount+1)+"张票");
            TrainCount--;
        }

        }
    }
}

public class Main {

    public static void main(String[] args) throws InterruptedException {
        ThreadTrain threadTrain = new ThreadTrain();
        Thread t1 = new Thread(threadTrain, "窗口一");
        Thread t2 = new Thread(threadTrain, "窗口二");

        t1.start();
        Thread.sleep(40);
        threadTrain.flag = false;
        t2.start();

    }
}
```

![运行结果](https://s2.ax1x.com/2019/03/30/AB6zyn.png)

> 产生原因： 一个线程已经占用了Object锁之后，打算进入this锁。但是第二个线程从flag = false那里的代码块直接占用this锁，从而第一个线程进不去sale()方法，而第二个方法执行sale()方法需要解开Object锁，导致死锁的产生。 这就好比是两个好友分别有对方的密码盒，并且都有自己钥匙，但是都不会把钥匙给对方，从而会产生一个谁也打不就开密码盒的尴尬情况。
