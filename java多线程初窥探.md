---
title: java多线程初窥探
date: 2019-03-27 21:00:23
categories:
- java
tags:
- java架构
- 多线程

---
# java多线程

多线程的作用就是提高应用程序的运行效率，提高用户的体验。那么，和多线程相近的进程又是什么意思呢？这两个有什么作用呢？

<!-- more -->

## 进程

* 打开计算机的任务管理器可以看到里面有很多的应用程序正在运行，那么这些程序就是许许多多的进程。
* 进程可以看成一个线程的集合【List】,许许多多的线程合成了一个进程，也就是我们的应用程序。
### 计算机如何进行许多应用程序的？
* 通过CPU对不同的应用程序进行不停的切换，也就是轮询？给用户了一种很多程序同时进行的假象，但是你打开一个程序很长时间不用，突然打开会有一瞬间的卡顿，这样就能看出他其实并不是和你当前的程序有着相同的地位的。
* ![多线程实现原理](http://pp1zup2fb.bkt.clouddn.com/java-1.png)

------------------------

## 线程

* **多线程**
* 开辟多线程是为了提高应用程序的运行效率。
* 开辟多线程之后，代码将不会从上至下进行。
* * **主线程和子线程**
* 子线程一般都是用类自己定义然后开启，他们的行为将受限于主线程。
### 线程的执行方法
* 继承Thread方法，并且重写Run方法，然后启动start服务
* 实现Runable方法，重写Run方法，并且将线程类交给Thread类去构建。然后start服务
* 利用匿名类
* [开发中常用]利用线程池开启服务

-----------------

### 多线程的具体实现
1. 继承Thread

```java
/**
 * 
 * @classDesc: 功能描述:(创建多线程例子-Thread类 重写run方法)
 * @author: WHOAMI
 * @version: v1.0
 */
class CreateThread extends Thread {
	// run方法中编写 多线程需要执行的代码
	public void run() {
		for (int i = 0; i< 10; i++) {
			System.out.println("i:" + i);
		}
	}
}
public class ThreadDemo {

	public static void main(String[] args) {
		System.out.println("-----多线程创建开始-----");
		// 1.创建一个线程
		CreateThread createThread = new CreateThread();
		// 2.开始执行线程 注意 开启线程不是调用run方法，而是start方法
		//3. 如果调用run方法，那么这个类就和普通的类没有区别
		System.out.println("-----多线程创建启动-----");
		createThread.start();
		System.out.println("-----多线程创建结束-----");
	}

}
```

2. 实现Runable接口

```java
/**
 * 
 * @classDesc: 功能描述:(创建多线程例子-Thread类 重写run方法)
 * @author: WHOAMI
 * @version: v1.0
 */
class CreateRunnable implements Runnable {

	@Override
	publicvoid run() {
		for (inti = 0; i< 10; i++) {
			System.out.println("i:" + i);
		}
	}

}

/**
 * 
 * @classDesc: 功能描述:(实现Runnable接口,重写run方法)
 * @author: WHOAMI
 * @version: v1.0
 */
public class ThreadDemo2 {
	public static void main(String[] args) {
		System.out.println("-----多线程创建开始-----");
		// 1.创建一个线程
		CreateRunnable createThread = new CreateRunnable();
		// 2.开始执行线程 注意 开启线程不是调用run方法，而是start方法
		System.out.println("-----多线程创建启动-----");
		Thread thread = new Thread(createThread);
		thread.start();
		System.out.println("-----多线程创建结束-----");
	}
}
```

3. 使用匿名内部类

```java
	//在main函数里面直接写子线程方法
	 System.out.println("-----多线程创建开始-----");
		 Thread thread = new Thread(new Runnable() {
			public void run() {
				for (int i = 0; i< 10; i++) {
					System.out.println("i:" + i);
				}
			}
		});
		 thread.start();
		 System.out.println("-----多线程创建结束-----");
```


#### Q&A
* Q:**那么相对于前面两种方法，那种方法比较好?**
>  A:使用接口比较好，因为java只能实现单继承，但是可以实现多种接口。而且公司开发大多都是面向接口编程。
#### 多线程常用API

|常用API||
|:----|:----|
| start()|启动线程|
| CurrentThread() |当前线程 |
| getID()|获取当前线程ID      Thread-编号  该编号从0开始|
|getName()|获取当前线程名称|
|sleep(long mill)|设置线程执行时间(ms)|

|常用线程构造函数||
| :-----| :-----|
|Thread()|分配一个新的Thread对象|
| Thread（String name）|分配一个新的Thread 对象，并且指定线程名称|
| Thread(Runable r)|分配一个新的Thread对象|
|Thread(Runable r,String name)|分配一个新的Thread对象|



