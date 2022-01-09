---
title: java多线程通信
date: 2019-03-31T20:58:04.000Z
categories:
  - java
tags:
  - java架构
  - 多线程
---

# java 生产者与消费者的模型分析

* 技术需求： 当生产者更新一条数据后，会立即通知消费者。
* 原理分析图
* ![原理图](https://s2.ax1x.com/2019/03/31/Ar7fN4.png)
* 这种需求可以看成一种消息队列 我们可以利用多线程来开启两个队列，一个是生产者，另一个是消费者。

## 利用synchronized 对象锁来实现线程原子性。

```java
package com.Thread.Test;



//消息模型
class Msg{
    public String data1;
    public String data2;

}

//生产者线程
class Producer extends Thread{

    private Msg msg;
    private int count = 0;

    public Producer(Msg msg) {
        this.msg = msg;
    }

    @Override
    public void run() {
        while (true){
            synchronized (msg){
                if(count == 0){
                    msg.data1 = "消息1";
                    msg.data2 = "状态1";
                }else{
                    msg.data1 = "消息2";
                    msg.data2 = "状态2";
                }

                count = (count +1)%2;
            }
        }
    }
}


//消费者线程
class Consumer extends Thread{


    private Msg msg;

    public Consumer(Msg msg) {
        this.msg = msg;
    }

    @Override
    public void run() {

        while (true){
            synchronized (msg){
                System.out.println("data1:"+msg.data1+";data2:"+msg.data2);
            }
        }

    }
}


public class Main {

    public static void main(String[] args) throws InterruptedException {


        Msg msg = new Msg();

        Producer producer = new Producer(msg);

        Consumer consumer = new Consumer(msg);

        producer.start();
        Thread.sleep(300);
        consumer.start();

    }
}
```

这里有两个关键点

* **为什么要使用对象锁？**
* 因为如果不利用对象锁的话，这两个线程是不安全的，因为JMM的原因使得线程不可见。
* 并且对象锁一定是一致的，不然数据不会同步。
* **这个解决方案是否可行？**
* 并不可以，因为两个线程存在抢占资源锁的情况，所以有可能生产者更新几次资源，但是消费者只显示一次，或者消费者重复显示几次的情况，并不符合我们的预期。
* 基于以上两点，我们采用线程通信技术。

## 多线程通信常用函数

1. wait()函数 该函数基于Object对象，他的作用是，暂时休眠该线程，并且**释放锁资源**
2. notify()函数 他的作用是唤醒线程池其他线程
3. interrupt()函数 将当前正在等待的线程【可以是wait的线程】，直接抛出异常，用来停止线程。

> 这两个函数通常都是配套使用，并且一定用在synchronized锁对象的情况下

***

利用上面两个函数进行线程通信

```java
package com.Thread.Test;



//消息模型
class Msg{
    public String data1;
    public String data2;
    //假定flag= false 的时候 生产者激活，反之消费者激活
    public boolean flag = false;
}

//生产者线程
class Producer extends Thread{

    private Msg msg;
    private int count = 0;

    public Producer(Msg msg) {
        this.msg = msg;
    }

    @Override
    public void run() {
        while (true){
            synchronized (msg){
//                此时生产者线程应该休眠
                if(msg.flag){
                    try {
                        Thread.sleep(1000);
                        msg.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                if(count == 0){
                    msg.data1 = "消息1";
                    msg.data2 = "状态1";
                }else{
                    msg.data1 = "消息2";
                    msg.data2 = "状态2";
                }

                count = (count +1)%2;

                msg.flag = true;
                //通知其他线程
                msg.notify();
            }
        }
    }
}

class Consumer extends Thread{


    private Msg msg;

    public Consumer(Msg msg) {
        this.msg = msg;
    }

    @Override
    public void run() {

        while (true){
            synchronized (msg){
                if(!msg.flag){

                    try {
                        Thread.sleep(1000);
                        msg.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                }
                System.out.println("data1:"+msg.data1+";data2:"+msg.data2);
                msg.flag = false;
                msg.notify();
            }
        }

    }
}


public class Main {

    public static void main(String[] args) throws InterruptedException {


        Msg msg = new Msg();

        Producer producer = new Producer(msg);

        Consumer consumer = new Consumer(msg);

        producer.start();
        consumer.start();

    }
}
```

## 利用Lock锁实现通信

Lock锁与synchronized的不同之处在于，wait和notify函数对于lock是没有用的

不多说，上代码

```java
package com.Thread.Test;


import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

//消息模型
class Msg2{
    public String data1;
    public String data2;
    //假定flag= false 的时候 生产者激活，反之消费者激活
    public boolean flag = false;
    public Lock lock = new ReentrantLock();
}

//生产者线程
class Producer2 extends Thread {

    private Msg2 msg;
    private int count = 0;
    private Condition condition;


    public Producer2(Msg2 msg, Condition condition) {
        this.msg = msg;
        this.condition = condition;
    }

    @Override
    public void run() {
        while (true) {
            try {
                msg.lock.lock();
//                此时生产者线程应该休眠
                if (msg.flag) {
                    try {
                        Thread.sleep(1000);
                        condition.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                if (count == 0) {
                    msg.data1 = "消息1";
                    msg.data2 = "状态1";
                } else {
                    msg.data1 = "消息2";
                    msg.data2 = "状态2";
                }

                count = (count + 1) % 2;

                msg.flag = true;
                //通知其他线程
                condition.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                msg.lock.unlock();
            }
        }
    }
}

class Consumer2 extends Thread{


    private Msg2 msg;

    private Condition condition;

    public Consumer2(Msg2 msg, Condition condition) {
        this.msg = msg;
        this.condition = condition;
    }

    @Override
    public void run() {

        while (true){
                try {
                    msg.lock.lock();

                    if(!msg.flag){

                        try {
                            Thread.sleep(1000);
                            condition.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                    }
                    System.out.println("data1:"+msg.data1+";data2:"+msg.data2);
                    msg.flag = false;
                    condition.signal();
                }catch (Exception e) {
                    e.printStackTrace();
                }finally {
                    msg.lock.unlock();
                }


            }
        }

}



public class LockTest {

    public static void main(String[] args) throws InterruptedException {


        Msg2 msg = new Msg2();
        //利用Condition来限定通信
        Condition newCondition = msg.lock.newCondition();

        Producer2 producer = new Producer2(msg,newCondition);

        Consumer2 consumer = new Consumer2(msg,newCondition);

        producer.start();
        consumer.start();

    }
}
```

## 如何优雅的停止线程（补充）

首先，放弃Thread.stop()函数 要知道，一些线程都是一些while循环的，即可能是while(true)格式的，这样停止他可以用这样的思路： 设置一个boolean的flag，当flag = true的时候正常运行，flag = false的时候停止线程。

> 这时应该考虑线程可见问题。需要将flag修改为 voliate格式。

但是，如果当前的线程是synchronized锁，并且在wait状态下，flag修改对本线程没有什么影响。因为现在线程已经休眠了。 那么可以利用interrupt函数让他抛出异常，然后在catch的代码块上面将flag修改。
