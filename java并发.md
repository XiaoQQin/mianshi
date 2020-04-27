<!-- TOC -->
- [1. 进程和线程的区别](#1-进程和线程的区别)  
- [2. java实现多线程的方式](#2-java实现多线程的方式)  
- [3. 关于线程安全](#3-关于线程安全)
- [4. 关于线程池](#4-关于线程池)
   - [4.1 使用线程池的好处](#41-使用线程池的好处)
   - [4.2 Executor框架](#42-Executor框架)
   - [4.3 线程池参数](#43-线程池参数)  
   - [4.4 线程池执行流程](#44-线程池执行流程)
<!-- /TOC -->
## 1. 进程和线程的区别
  
**进程**  
  
进程是程序的一次执行过程,是动态的,是程序执行过程中分配和管理资源的基本单位。系统运行一个程序即是一个进程从创建，运行到消亡的过程。  
  
**线程**  
  
线程与进程相似，但线程是一个比进程更小的执行单位，线程也被称为轻量级进程。它是cpu调度和执行的基本单位。  
  
**线程和进程的区别**  
一个进程可以有多个进程，多个线程共享进程的**堆**和**方法区**，每个线程都有自己私有的**虚拟机栈**，**本地方法栈**和**程序计数器**。线程和进程的最大区别是基本上各进程是独立的，而线程不一样，同属于一个进程的线程极大可能受影响。线程执行开销小，不利于资源的管理和保护，进程则相反。  

## 2. java实现多线程的方式
  1.  **继承Thread类**  
  2.  **实现Runnable接口**  
  3.  **实现Callable接口**  
     与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装，这是最主要的区别。  
  4.  **使用线程池**  
    
    
  **继承和实现接口的区别**    
    
    
  实现接口只能当做一个可以在线程中运行的任务，不是真正意义上的线程。实现接口更好一些  
  java类不能多继承，继承Thread类就不能继承其他类。  
  类可能要求可执行即可，继承Thread类开销过大。
  
 ## 3. 关于线程安全  
 **线程安全**可以简单理解为一个方法或一个实例在多线程中使用不会发生问题。同一程序中执行多线程不会发生问题，问题在于多个线程访问了相同的资源。一或多个线程向这些资源做了写操作时就会发生**线程安全**问题，只要资源没有发生变化,多个线程读取相同的资源就是安全的。如果一个资源的创建，使用，销毁都在同一个线程内发生，且不会脱离线程，那么资源的使用就是线程安全的。    
  
 当两个线程竞争同一资源时，如果对资源的访问顺序敏感，就称存在**竞态条件**。导致竞态条件发送的代码叫做**临界区**。  
 
 **实现线程安全的方法**  
   
   1. 使用synchorized关键字
   2. 使用java.util.concurrent.locks 包中的锁  
   3. 使用线程安全的集合,比如concurrentHashMap等  
   4. 使用volatile关键字，保证变量的可见性。  
     
     
 [什么是线程安全](https://blog.csdn.net/suifeng3051/article/details/52164267)

## 4. 关于线程池
  
### 4.1 使用线程池的好处
  
  1. **降低资源消耗**：通过重复利用已创建的线程降低线程创建和销毁造成的消耗  
  2. **提高响应速度**: 当任务到达时，不需要创建线程就能立即执行  
  3. **提高线程的可管理性**：线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，也会降低系统的稳定性,使用线程池可以进行统一的分配，调控，管理。       
### 4.2 Executor框架
  
**Executor框架结构(三大部分组成)**  
  
  - **1.任务(Runnable/Callable)**  
     执行的任务需要实现**Runnable**/**Callable**结构，这两个接口的实现类都可以被**ThreadPoolExecutor**执行  
       
       
  - **2.任务的执行者(Executor)**
     主要关注**ThreadPoolExecutor**这个类，它实现了继承自**Executor**接口的**ExecutorService**接口，在日常的线程池使用过程中出现的最频繁  
       
  - **3.异步计算的结构(Future)**  
    
      **Future** 接口以及该接口的实现类**FutureTask**类都可以代表异步计算的结果。当我们把实现**Runnable**或**Callable**的类提交给**ThreadPoolExecutor**执行(使用**submit**方法提交就会返回一个**FutureTask**对象)。
        
  
### 4.3 线程池参数  
   线程池实现类**ThreadPoolExecutor**是**Executor**框架的最核心的实现类。**ThreadPoolExecutor**有4个构造函数，有一个参数最多(这也是面试中问的所在)，其他都是在这个构造函数基础上实现的。  
   ```
     /**
     * 用给定的初始参数创建一个新的ThreadPoolExecutor。
     */
    public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
                              int maximumPoolSize,//线程池的最大线程数
                              long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
                              ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
                              RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                               ) {
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
   可以看出，线程池参数如下，具体上如代码所示：  
      - **线程池核心线程数**  
      - **线程池最大线程数**  
      - 当线程数大于核心线程数，多余线程存活时间  
      - 时间单位  
      - **任务队列**  
      - 线程工厂  
      - **拒绝策略**  
     
     
   **ThreadPoolExecutor饱和策略(拒绝策略)**  
     
   当线程池中的线程数已达到最大线程数，并且任务队列已满时，线程池就会采用**饱和策略(拒绝策略)**  
     
   - **ThreadPoolExecutor.AbortPolicy**：抛出 RejectedExecutionException异常来拒绝新任务的处理。  
   - **ThreadPoolExecutor.CallerRunsPolicy**：调用执行自己的线程运行任务，也就是直接在调用execute方法的线程中运行(run)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。另外，这个策略喜欢增加队列容量。如果您的应用程序可以承受此延迟并且你不能任务丢弃任何一个任务请求的话，你可以选择这个策略。  
   - **ThreadPoolExecutor.DiscardPolicy**： 不处理新任务，直接丢弃掉  
   - **ThreadPoolExecutor.DiscardOldestPolicy**： 此策略将丢弃最早的未处理的任务请求
   
### 4.4 线程池执行流程  
     
线程池会按照以下流程执行任务：
  
- 当线程池中线程数小于核心线程数，会创建新线程  
- 当线程数大于等于核心线程数，任务队列未满，将任务放入任务队列  
- 当线程数大于等于核心线程数，且任务队列已满，则会有两种情况：  
      - 若当前线程数小于最大线程数，则创建新线程  
      - 若当前线程数等于最大线程数，则采取拒绝策略  
  
![图解线程池实现原理](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/图解线程池实现原理.png)  
  
更多详细内容，强烈推荐[java线程池学习总结](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93.md)
