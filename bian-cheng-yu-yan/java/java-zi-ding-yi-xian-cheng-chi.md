---
title: java自定义线程池
date: 2019-04-08T15:29:04.000Z
categories:
  - java
tags:
  - java架构
  - 线程池
---

# java线程池

## java初始定义线程

![](https://s2.ax1x.com/2019/04/08/A4XAkF.png)

## java线程池源码分析

![](https://s2.ax1x.com/2019/04/08/A5iM28.png)

```java
/**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters and default rejected execution handler.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @param threadFactory the factory to use when the executor
     *        creates a new thread
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code threadFactory} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }
```

1. corePoolSize 表示常驻核心线程数。如果等于0，则任务执行完成后，没有任何请求进入时销毁线程池的线程；如果大于0，即使本地任务执行完毕，核心线程也不会被销毁。这个值的设置非常关键，设置过大会浪费资源，设置的过小会导致线程频繁地创建或销毁。再考虑到keepAliveTime和allowCoreThreadTimeOut超时参数的影响，所以没有任务需要执行的时候，线程池的大小不一定是corePoolSize。
2. maximumPoolSize 线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；当阻塞队列是无界队列, 则maximumPoolSize则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入workQueue.

## 合理配置线程数

***

### 分析需求

根据需求的不同，可以将技术分成IO密集和CPU密集两种类型。

IO密集：

线程中存在大量的阻塞，例如，请求，数据库连接，。。。线程会有大量的等待，这样会有等待时间，那么这个线程就是IO密集型，一般这种类型，配置核心线程数的规则是 core = 2\*CPU内核

CPU密集：

线程中代码非常多，而且计算的内容非常多，阻塞少，这个就是CPU密集，配置规则是：core = CPU内核。
