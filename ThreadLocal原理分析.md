---
title: ThreadLocal原理分析
date: 2019-04-02 09:41:55
categories:
- java
tags:
- java架构
- 多线程
---

# ThreadLocal原理分析

需求：如果我们引入一个全局变量，但是他的值在每个线程需要互相不影响。

> 解决办法1:可以根据线程的次数new 几次实体类 ，然后分别装进每个线程中，但是如果线程数量很大并且不确定，这个方法不符合实际。

>解决办法2：将实体类的变量设置为ThreadLocal类型

<!-- more -->



## ThreadLocal案例引入


```java
package com.Thread.Test;

class Local{
//一定要初始化变量的值
    ThreadLocal<Integer> count = new ThreadLocal<Integer>(){
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };

    public void set(){
        this.count.set(this.count.get()+1);
    }

    public Integer get(){
        return count.get();
    }
}


class ThreadTest implements Runnable{

    Local local;

    public ThreadTest(Local local) {
        this.local = local;
    }

    @Override
    public void run() {
        for (int i=0;i<3;i++){
            local.set();
            System.out.println(Thread.currentThread().getName()+":"+local.get());
        }
    }
}


public class Main {

    public static void main(String[] args) throws InterruptedException {

        Local local = new Local();

        Thread thread1 = new Thread(new ThreadTest(local));
        Thread thread2 = new Thread(new ThreadTest(local));
        Thread thread3 = new Thread(new ThreadTest(local));
        thread1.start();
        thread2.start();
        thread3.start();
    }
}

```



## ThreadLocal源码分析

* 将变量设置成一个Map类型，存的时候，将线程的名字和操作一起存入Map进去，取出来的时候，根据自己的线程名字来取



set方法源码：

```java
  public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```


get方法源码：
```java
  public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```
