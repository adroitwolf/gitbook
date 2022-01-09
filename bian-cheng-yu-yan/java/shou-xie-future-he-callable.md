---
title: 手写Future和Callable
date: 2019-04-21T13:32:22.000Z
categories:
  - java
tags:
  - java架构
---

# java Future和Callable模式

***

* 我们之前使用的多线程需要在run方法中执行多线程代码，但是run方法有一个致命缺点，就是他的返回值是空。但是，如果我们像要我们如何知道子线程执行完毕呢？没有返回值告诉我们，所以我们需要上面的两种模式。

## Callable模式

***

其实，Callable就相当于一个有返回值的线程，而Future是一个可以接收Callable的返回值。

Callable的源代码如下：

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

他和Runable接口源代码是很相似的。他和Runable的唯一的不同是返回值V是泛型,而Runable是void.

_**Future常用方法**_

**V get()** \*\*：\*\*获取异步执行的结果，如果没有结果可用，此方法会阻塞直到异步计算完成。

**V get(Long timeout , TimeUnit unit)** ：获取异步执行结果，如果没有结果可用，此方法会阻塞，但是会有时间限制，如果阻塞时间超过设定的timeout时间，该方法将抛出异常。

**boolean isDone()** \*\*：\*\*如果任务执行结束，无论是正常结束或是中途取消还是发生异常，都返回true。

**boolean isCanceller()** \*\*：\*\*如果任务完成前被取消，则返回true。

**boolean cancel(boolean mayInterruptRunning)** \*\*：\*\*如果任务还没开始，执行cancel(...)方法将返回false；如果任务已经启动，执行cancel(true)方法将以中断执行此任务线程的方式来试图停止任务，如果停止成功，返回true；当任务已经启动，执行cancel(false)方法将不会对正在执行的任务线程产生影响(让线程正常执行到完成)，此时返回false；当任务已经完成，执行cancel(...)方法将返回false。mayInterruptRunning参数表示是否中断执行中的线程。

通过方法分析我们也知道实际上Future提供了3种功能：（1）能够中断执行中的任务（2）判断任务是否执行完成（3）获取任务执行完成后额结果。

***

实例:

```java
public class TestMain {
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		
        ExecutorService executor = Executors.newCachedThreadPool();
		Future<Integer> future = executor.submit(new AddNumberTask());
		System.out.println(Thread.currentThread().getName() + "线程执行其他任务");
		Integer integer = future.get();
		System.out.println(integer);
		// 关闭线程池
		if (executor != null)
			executor.shutdown();
	}

}

class AddNumberTask implements Callable<Integer> {

	public AddNumberTask() {

	}

	@Override
	public Integer call() throws Exception {
		System.out.println("####AddNumberTask###call()");
		Thread.sleep(5000);
		return 5000;
	}
}
```

Future的好处就是先传给你一个结果,然后等资源全部解析完成之后,再传给你值,相当于ajax的异步.

![](https://s2.ax1x.com/2019/04/21/EF0tSO.png)

## 手写Callable

***

仔细一想，这相当于生产者消费者模型。可以用wait和notify来解决。

核心代码：

```java
package com.Future;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2019-2019/4/21-13:00
 * Description: :/TODO_
 */
public class FutureCore {
	//请求参数
    private String arg;
	//标识变量
    private volatile  boolean FLAG = false;
	//返回结果
    private String result;

    public FutureCore(String arg) {

        this.arg = arg;
    }

	//这里可能会有阻塞，模拟下载
    public synchronized void setRequest(){
        if(FLAG){
            return;
        }
        System.out.println("接收网络参数" + arg+";数据请求中...");

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        this.result = "结果";
        FLAG = true;
        notify();
    }
	
    //返回结果
    public synchronized String getResult(){

        while (!FLAG){
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        return result;
    }

}
```

这里是封装future的类

```java
package com.Future;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2019-2019/4/21-13:05
 * Description: :/TODO_
 */
public class FutureUtil {

    public static FutureCore submit(String arg){
        FutureCore futureCore = new FutureCore(arg);
		//子线程开始下载资源
        new Thread(new Runnable() {
            @Override
            public void run() {
                futureCore.setRequest();
            }
        }).start();
		//直接返回结果
        return futureCore;
    }
}
```

测试主函数如下:

```java
package com.Future;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2019-2019/4/21-13:09
 * Description: :/TODO_
 */
public class ThreadTest {

    public static void main(String args[]){
        System.out.println("开始future");
        FutureCore core = FutureUtil.submit("测试");

        System.out.println("主线程继续执行");

        System.out.println(core.getResult());

    }
}
```

结果如下：

![](https://s2.ax1x.com/2019/04/21/EF0fmj.png)
