title: Java 基础 —— 多线程（读书笔记）「三」
date: 2016-09-29 16:02:31
---

前面两章已经了解了很多线程相关的知识，但是当我们需要频繁地创建多个线程进行耗时操作时，每次通过 `new Thread` 并不是很好的实现方式。
- 新建和销毁对象性能较差
- 线程缺乏统一管理
- 可能无限制新建线程，相互竞争，死锁
- 缺少定时执行，定期执行，线程中断等等

<!-- more -->
#### 目录
- [目录](#目录)
- [简介](#简介)
- [使用](#使用)
  - [ThreadPoolExecutor](#ThreadPoolExecutor)
    - [构造函数](#构造函数)
    - [WorkQueue](#WorkQueue)
    - [RejectedExecutionHandler](#RejectedExecutionHandler)


#### 简介

线程池简单来说：创建多个线程并且进行管理，提交给线程的任务会被线程池指派给其中的线程执行，通过线程池的统一调度，管理使得多线程的使用更简单，高效。

大致流程图：
![](http://ww1.sinaimg.cn/large/65e4f1e6gw1f8m936f9a5j21540x0ada.jpg)


线程池都实现了 ExecutorService 接口，它的实现有 ThreadPoolExecutor 和 ScheduledExecutorService 。ThreadPoolExecutor 也就是我们运用最多的线程池实现，而 ScheduledExecutorService 通过名字我们就可以知道用于__周期性地执行任务__。

通常我们都不会使用 `new` 的形式来创建线程池，而使用 JDK 给我们封装好了的 Executors 工厂类来简化这个过程。

#### 使用

##### ThreadPoolExecutor

ThreadPoolExecutor 是线程池的实现之一，它的功能是启动指定数量的线程以及将任务添加到一个队列中，并且将任务分发给空闲线程。

ExecutorService 的生命周期包括三种状态：运行，关闭，终止。
创建后便进入运行状态，当调用了 `shutdown()` 方法时，便进入关闭状态，此时意味着 ExecutorService 不再接受新的任务，但它还在执行已经提交了的任务。当所有已经提交的任务执行完毕后，就变成终止状态。

做了一个图，直观的展示了状态的转换：
![](http://ww1.sinaimg.cn/large/65e4f1e6gw1f8me14yaucj212a0l2jse.jpg)

###### 构造函数
```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```
我们主要对构造函数的参数进行简要说明：

- corePoolSize 线程池中所保存的核心线程数。
- maximumPoolSize 线程池允许创建的最大线程数。
- keepAliveTime 当前线程池线程总数大于核心线程数时，终止多余的空闲线程的时间。
- Unit keepAliveTime 参数的时间单位，可选值有毫秒，秒，分等。
- workQueue 任务队列，如果当前线程池达到核心线程数 corePoolSize ，且当前线程都处于活动状态时，则将新加入的任务放到此队列。
- threadFactory 线程工厂，让用户可以定制线程的创建过程，通常不需要设置。
- RejectedExecutionHandler 拒绝策略，当线程池与 workQueue 队列都满了的情况下，对新加任务采取的处理策略。

###### WorkQueue

其中 workQueue 有下列几个常用的实现：

1. ArrayBlockingQueue： 基于数组结构的有界队列，此队列按 FIFO 原则对任务进行排序。如果队列满了还有任务进来，则调用拒绝策略。
2. LinkedBlockingQueue：基于链表结构的无界队列，此队列按 FIFO 原则对任务进行排序。因为它是无界的，根本不会满，所以采用此队列后线程池将忽略拒绝策略 (handler) 参数；同时还将忽略最大线程数 maximumPoolSize 。
3. SynchronousQueue：直接将任务提交给线程而不是将它加入到队列，实际上此队列是空的。每个插入的操作必须等到另一个调用移除的操作；如果新任务来了线程池没有任何可用线程处理的话，则调用拒绝策略。
4. PriorityBlockingQueue：具有优先级基于数组结构的有界队列，可以自定义优先级，默认是按照自然排序。


###### RejectedExecutionHandler

当线程池与 workQueue 队列都满了的情况下，对新加任务采取的处理策略也有几个默认实现：

1. AbortPolicy：拒绝任务，抛出 RejectedExecutionException 异常。线程池默认策略。
2. CallerRunsPolicy：拒绝新任务进入，如果该线程池还没有被关闭，那么将这个新任务执行在调用线程中。
3. DiscardOldestPolicy：如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序。
4. DiscardPolicy： 加不进的任务都被抛弃了，同时没有异常抛出。


###### 实践

Executors 可以构建几种常见的线程池，我们通过实例对比来加深对他们的印象。


- __newCachedThreadPool__

当线程池中的线程空闲时间超过 60s 则会自动回收该线程，当任务超过线程池的线程数则创建新线程。线程池的大小上限为 Integer.MAX_VALUE，可看做是无限大。

```java
public class ThreadPoolTest {

    public static void main(String[] args) {
        cachedThreadPoolRun();
    }
    
    private static void produceTasks(ExecutorService service) {
        for (int i = 0; i < 10; i++) {
            final int finalI = i;
            service.submit(new Runnable() {
                @Override
                public void run() {
                    System.out.println("This Thread is: " + Thread.currentThread().getName() + " >>> Task:" + finalI);
                }
            });

            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private static void cachedThreadPoolRun() {
        ExecutorService service = Executors.newCachedThreadPool();
        produceTasks(service);
        service.shutdown();
    }
}
```
运行后控制台输出：
> This Thread is: pool-1-thread-1 >>> Task:0
> This Thread is: pool-1-thread-1 >>> Task:1
> This Thread is: pool-1-thread-1 >>> Task:2
> This Thread is: pool-1-thread-1 >>> Task:3
> This Thread is: pool-1-thread-1 >>> Task:4
> This Thread is: pool-1-thread-1 >>> Task:5
> This Thread is: pool-1-thread-1 >>> Task:6
> This Thread is: pool-1-thread-1 >>> Task:7
> This Thread is: pool-1-thread-1 >>> Task:8
> This Thread is: pool-1-thread-1 >>> Task:9

从结果可以看出，整个过程都在 pool-1-thread-1 中执行，后续任务一直在复用之前线程。

- **newFixedThreadPool**





#### 总结和参考




- [ThreadPoolExecutor 源码学习笔记](http://extremej.itscoder.com/threadpoolexecutor_source/)
- [Java 线程池分析](http://gityuan.com/2016/01/16/thread-pool/)