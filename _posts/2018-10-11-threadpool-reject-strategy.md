---
layout: post
title: 围绕线程池的拒绝策略展开的
category: tech
---

拒绝策略：

拒绝策略其实就是这几种，但我们不看答案，如果让你设计拒绝策略，该如何设计呢？



首先要明确，设计的是拒绝策略，为什么要拒绝，资源不够，说明只有一部分的提交的任务会被执行

计算机是个图灵机、状态机，那么，我们是怎么知道从服务接收态转向了服务拒绝态呢？这个临界点在哪里？

显然这个临界点是压垮骆驼的最后一根稻草，是最后那个大于任务队列长度的新提交任务

因此，拒绝策略就要围绕这个任务展开了，我们叫它 **临界任务** ，针对临界任务，我们可以直接抛弃

但是抛弃的过程中，我们可以让他默默的抛弃，也可以然它给我们一点提示

我们也可以给他开个后门，让他执行，但这时候问题又来了，这时候是资源不够的状态，需要占用资源了，所以我们可以踢掉队列中的一个任务，让新任务上位；或者可以直接执行这个任务

我们梳理一下思路：

* 临界任务出现，面临资源不足的问题
 * 必须执行临界任务
   * 在线程池中执行，抛弃线程池中的老任务 ----> D
   * 由于是一个Runnable，可以让他独立执行 ---> B
 * 抛弃临界任务，保证线程池的内部稳定
   * 沉默的抛弃（do nothing） --------------> C
   * 有声音的抛弃 （exception）-------------> A

对应的就是下面这四种策略了，通常会把两种Discard放一起，但这完全是两种策略，分析问题要从 **状态转换** 来进行分析。

```
A. taskPool.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
B. taskPool.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
C. taskPool.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardPolicy());
D. taskPool.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardOldestPolicy());
```

其中B策略是在线程池未shutdown的前提下，执行任务，这样确保了任务执行依托于线程池的原则；

这就够了么？当然不是

通过源码发现，这些策略只不过是实现了 **setRejectedExecutionHandler** 而已，因此，拒绝策略的本源是根据我们对问题的分析，来实现自己需要的处理策略，也就是ThreadPoolExecutor的第六个参数：

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          RejectedExecutionHandler handler) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), handler);
}
```

结合一下开发中遇到的问题，队列满的时候，我希望提交任务的线程能够阻塞，那么这时候该如何来做呢？一样的，抓住 **临界任务** 这个突破口，往高层次说了是状态转换的美丽瞬间

定制RejectedExecutionHandler，只要在队列满的时候采用阻塞式方式添加任务就好，这就涉及到了阻塞队列添加任务的方式了。

我们看到，线程池提交任务时，使用的是offer方法

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
        if (runState == RUNNING && workQueue.offer(command)) {
            if (runState != RUNNING || poolSize == 0)
                ensureQueuedTaskHandled(command);
        }
        else if (!addIfUnderMaximumPoolSize(command))
            reject(command); // is shutdown or saturated
    }
}
```

再过一次阻塞队列的方法吧：

boolean add(E e); -> 非阻塞，成功返回true,失败抛异常

boolean offer(E e); -> 非阻塞，成功返回true,失败返回false，相比于add更友好；add和offer采用链表尾插法，必须加锁

boolean offer(E e, long timeout, TimeUnitunit)throws InterruptedException; -> 非阻塞，增加了等待时间的

void put(E e) throws InterruptedException; -> 唯一的阻塞

因此，使用put代替offer就是问题的解决方法了

```
ThreadPoolExecutor redoPool = new ThreadPoolExecutor(threadCount, threadCount,
                60L, TimeUnit.SECONDS,
                new LinkedBlockingQueue<Runnable>(1000),
                new RejectedExecutionHandler() {
                    @Override
                    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                        if (!executor.isShutdown()) {
                            try {
                                // 阻塞队列满时采用阻塞型put方法
                                executor.getQueue().put(r);
                            } catch (InterruptedException e) {
                                // do nothing
                            }
                        }
                    }
                });
```

就到这里吧，问题还是挺简单的，趁还没有那么多的烦心事，一定要严谨，要一丝不苟
