# 线程池总结

[原文地址](https://mp.weixin.qq.com/s/_ibq9m98XYOu9VIMRnRtgg)

## 1. corePoolSize=0会怎么样

这是一个争议点。我发现大部分博文，不论是国内的还是国外的，都是这样回答这个问题的：

* 提交任务后，先判断当前池中线程数是否小于corePoolSize, 如果小于直接创建新线程。
* 否则，判断等待队列是否已满，如果没有满，就插入到等待队列中。
* 否则，判断当前线程池中线程数量是否大于maxmumPoolSize, 如果大于则拒绝。
* 否则，创建一个新的线程执行这个任务。

上述问题需区分JDK版本。在1.6版本之后，如果`corePoolSize=0`，提交任务时如果线程池为空，则会立即创建一个线程来执行任务（先排队再获取）；如果提交任务的时候，线程池不为空，则先在等待队列中排队，只有队列满了才会创建新线程。

所以，优化在于，在队列没有满的这段时间内，会有一个线程在消费提交的任务；1.6之前的实现是，必须等队列满了之后，才开始消费。

## 2. 线程池创建之后，会立即创建核心线程么

不会。从上面的源码可以看出，在刚刚创建ThreadPoolExecutor的时候，线程并不会立即启动，而是要等到有任务提交时才会启动，除非调用了prestartCoreThread/prestartAllCoreThreads事先启动核心线程。

## 3. 核心线程永远不会销毁么

> 首先我们要明确一下概念，虽然在JavaDoc中也使用了“core/non-core threads”这样的描述，但其实这是一个动态的概念，JDK并没有给一部分线程打上“core”的标记，做什么特殊化的处理。这个问题我认为想要探讨的是闲置线程终结策略的问题。
>
> 在JDK1.6之前，线程池会尽量保持corePoolSize个核心线程，即使这些线程闲置了很长时间。这一点曾被开发者诟病，所以从JDK1.6开始，提供了方法allowsCoreThreadTimeOut，如果传参为true，则允许闲置的核心线程被终止。

请注意这种策略和corePoolSize=0的区别。我总结的区别是：

- corePoolSize=0：在一般情况下只使用一个线程消费任务，只有当并发请求特别多、等待队列都满了之后，才开始用多线程。

- allowsCoreThreadTimeOut=true && corePoolSize>1：在一般情况下就开始使用多线程（corePoolSize个），当并发请求特别多，等待队列都满了之后，继续加大线程数。但是当请求没有的时候，允许核心线程也终止。

所以corePoolSize=0的效果，基本等同于allowsCoreThreadTimeOut=true && corePoolSize=1，但实现细节其实不同。

**答**

在JDK1.6之后，如果allowsCoreThreadTimeOut=true，核心线程也可以被终止。

## 4. 如何保证线程不被销毁

Q: 线程池中的线程是啥？

A: 其实就是**Worker**。

Q: 等待队列中的元素是啥？

A: 等待队列中的元素，就是我们提交的`Runnable`任务。

每一个`Worker`在创建出来的时候，会调用它本身的`run()`方法，实现是`runWorker(this)`，这个实现的核心是一个`while循环`，这个循环不结束，`Worker`线程就不会终止，就是这个基本逻辑。

- 在这个`while条件`中，有个`getTask()`方法是核心中的核心，它所做的事情就是从等待队列中取出任务来执行：

- 如果没有达到`corePoolSize`，则创建的`Worker`在执行完它承接的任务后，会用`workQueue.take()`取任务、注意，这个接口是**阻塞接口**，如果取不到任务，`Worker线程`一直阻塞。

- 如果超过了`corePoolSize`，或者`allowCoreThreadTimeOut`，一个`Worker`在空闲了之后，会用`workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS)`取任务。注意，这个接口只阻塞等待`keepAliveTime`时间，超过这个时间返回`null`，则`Worker`的`while循环`执行结束，则被终止了。

**答**

实现方式非常巧妙，核心线程（Worker）即使一直空闲也不终止，是通过`workQueue.take()`实现的，它会一直阻塞到从等待队列中取到新的任务。非核心线程空闲指定时间后终止是通过`workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS)`实现的，一个空闲的`Worker`只等待`keepAliveTime`，如果还没有取到任务则循环终止，线程也就运行结束了。

**引申思考**

Worker本身就是个线程，它再调用我们传入的Runnable.run()，会启动一个子线程么？如果你还没有答案，再回想一下Runnable和Thread的关系。

## 5. 空闲线程过多会有什么问题

首先，比较普通的一部分，一个线程的内存模型：

- 虚拟机栈
- 本地方法栈
- 程序计数器

JVM使用TLAB来避免多线程冲突，在给对象分配内存时，每个线程使用自己的TLAB，这样可以避免线程同步，提高了对象分配的效率。
TLAB本身占用eEden区空间，在开启TLAB的情况下，虚拟机会为**每个Java线程分配一块TLAB空间**。参数-XX:+UseTLAB开启TLAB，默认是开启的。TLAB空间的内存非常小，缺省情况下仅占有整个Eden空间的1%，当然可以通过选项-XX:TLABWasteTargetPercent设置TLAB空间所占用Eden空间的百分比大小。

我想额外强调是下面这几个内存占用，需要小心：

- ThreadLocal：业务代码是否使用了ThreadLocal？就算没有，Spring框架中也大量使用了ThreadLocal，你所在公司的框架可能也是一样。

- 局部变量：线程处于阻塞状态，肯定还有栈帧没有出栈，栈帧中有局部变量表，凡是被局部变量表引用的内存都不能回收。所以如果这个线程创建了比较大的局部变量，那么这一部分内存无法GC。

- TLAB机制：如果你的应用线程数处于高位，那么新的线程初始化可能因为Eden没有足够的空间分配TLAB而触发YoungGC。

**答**

- 线程池保持空闲的核心线程是它的默认配置，一般来讲是没有问题的，因为它占用的内存一般不大。怕的就是业务代码中使用ThreadLocal缓存的数据过大又不清理。

- 如果你的应用线程数处于高位，那么需要观察一下YoungGC的情况，估算一下Eden大小是否足够。如果不够的话，可能要谨慎地创建新线程，并且让空闲的线程终止；必要的时候，可能需要对JVM进行调参。

## 6. keepAliveTime=0会怎么样

在JDK1.8中，`keepAliveTime=0`表示非核心线程执行完立刻终止。

默认情况下，`keepAliveTime`小于0，初始化的时候才会报错；但如果`allowsCoreThreadTimeOut`，`keepAliveTime`必须大于0，不然初始化报错。

## 7. 怎么进行异常处理

- 如果我们使用execute()提交任务，我们一般要在Runable任务的代码加上try-catch进行异常处理。

- 如果我们使用submit()提交任务，我们一般要在主线程中，对Future.get()进行try-catch进行异常处理。

## 8. 线程池需不需要关闭

一般来讲，线程池的生命周期跟随服务的生命周期。如果一个服务（Service）停止服务了，那么需要调用shutdown方法进行关闭。所以ExecutorService.shutdown在Java以及一些中间件的源码中，是封装在Service的shutdown方法内的。

如果是Server端不重启就不停止提供服务，我认为是不需要特殊处理的。

## 9.  shutdown和shutdownNow的区别

- `shutdown` => 平缓关闭，等待所有已添加到线程池中的任务执行完再关闭。

- `shutdownNow` => 立刻关闭，停止正在执行的任务，并返回队列中未执行的任务。





## **最佳实践总结**

- 【强制】使用ThreadPoolExecutor的构造函数声明线程池，避免使用Executors类的 newFixedThreadPool和newCachedThreadPool。

- 【强制】 创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。即threadFactory参数要构造好。

- 【建议】建议不同类别的业务用不同的线程池。

- 【建议】CPU密集型任务(N+1)：这种任务消耗的主要是CPU资源，可以将线程数设置为N（CPU核心数）+1，比CPU核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用CPU的空闲时间。

- 【建议】I/O密集型任务(2N)：这种任务应用起来，系统会用大部分的时间来处理I/O交互，而线程在处理I/O的时间段内不会占用CPU来处理，这时就可以将CPU交出给其它线程使用。因此在I/O密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是2N。

- 【建议】workQueue不要使用无界队列，尽量使用有界队列。避免大量任务等待，造成OOM。

- 【建议】如果是资源紧张的应用，使用allowsCoreThreadTimeOut可以提高资源利用率。

- 【建议】虽然使用线程池有多种异常处理的方式，但在任务代码中，使用try-catch最通用，也能给不同任务的异常处理做精细化。

- 【建议】对于资源紧张的应用，如果担心线程池资源使用不当，可以利用ThreadPoolExecutor的API实现简单的监控，然后进行分析和优化。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naJ90Ks7UHT87Q3foSu1rY6y8e4XGCibgibia2R8iaYGogDHTAEVwb3O9eOUBuWMRhyL2iaNbx5ZvzplIbQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



线程池初始化示例：

```java
    private static final ThreadPoolExecutor pool;
    static {        ThreadFactory threadFactory = new ThreadFactoryBuilder().setNameFormat("po-detail-pool-%d").build();        pool = new ThreadPoolExecutor(4, 8, 60L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>(512),            threadFactory, new ThreadPoolExecutor.AbortPolicy());        pool.allowCoreThreadTimeOut(true);    }
```



- `threadFactory`：给出带业务语义的线程命名。

- `corePoolSize`：快速启动4个线程处理该业务，是足够的。

- `maximumPoolSize`：IO密集型业务，我的服务器是4C8G的，所以4*2=8。

- `keepAliveTime`：服务器资源紧张，让空闲的线程快速释放。

- `pool.allowCoreThreadTimeOut(true)`：也是为了在可以的时候，让线程释放，释放资源。

- `workQueue`：一个任务的执行时长在100~300ms，业务高峰期8个线程，按照10s超时（已经很高了）。10s钟，8个线程，可以处理10 * 1000ms / 200ms * 8 = 400个任务左右，往上再取一点，512已经很多了。

- handler：极端情况下，一些任务只能丢弃，保护服务端。

------

