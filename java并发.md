<!-- TOC -->
- [1. 进程和线程的区别](#1-进程和线程的区别)  
- [2. java实现多线程的方式](#2-java实现多线程的方式)  
- [3. 关于线程安全](#3-关于线程安全)  
   - [3.1 java内存模型(JMM)](#31-java内存模型(JMM))  
   - [3.2 实现线程安全的方法](#32-实现线程安全的方法)
- [4. 关于线程池](#4-关于线程池)
   - [4.1 使用线程池的好处](#41-使用线程池的好处)
   - [4.2 Executor框架](#42-Executor框架)
   - [4.3 线程池参数](#43-线程池参数)  
   - [4.4 线程池执行流程](#44-线程池执行流程)
- [5. volatile和synchronized的区别](#5-volatile和synchronized的区别)  
- [6. synchronized关键字和lock的区别](#6-synchronized关键字和lock的区别)  
   - [6.1 synchronized的低层实现](#61-synchronized的低层实现)  
   - [6.2 Lock低层实现](#62-Lock低层实现)  
   - [6.3 synchronized在JDK1.6以后的优化](#63-synchronized在JDK1.6以后的优化)
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
 ### 3.1 java内存模型(JMM)
 java内存模型(即Java Memory Model，简称JMM)本身是一种抽象的概念，并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式，用来屏蔽不同硬件和操作系统的内存访问差异，让Java程序在各自平台下都达到一致的内存访问效果。由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存(有些地方称为栈空间)，用于存储线程私有的数据，而Java内存模型中规定所有变量都存储在主内存，主内存是共享内存区域，所有线程都可以访问，但线程对变量的操作(读取赋值等)必须在工作内存中进行，首先要将变量从主内存拷贝的自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，工作内存中存储着主内存中的变量副本拷贝。  
![java内存模型示意图](https://img-blog.csdn.net/20170608221857890?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
### 3.2 实现线程安全的方法  
   
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

## 5. volatile和synchronized的区别    
  
  
在Java虚拟机规范中试图定义一种Java内存模型(**JMM**),来屏蔽各个硬件平台和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。  
Java内存模型规定所有的变量都是存在主存当中（类似于前面说的物理内存），每个线程都有自己的工作内存（类似于前面的高速缓存）。线程对变量的所有操作都必须在工作内存中进行，而不能直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。  
如果一个程序想要并发的执行，那么必须保证原子性、可见性、一致性。   
  
  
**原子性：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。  
可见性：可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。  
有序性：有序性：即程序执行的顺序按照代码的先后顺序执行**  

加入volatile关键字时，会多出一个lock前缀指令：
lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：  
   1)  它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成  
   2)  它会强制将对缓存的修改操作立即写入内存  
   3)  如果是写操作，它会导致其他CPU中对应的缓存行无效。    
   
synchronized则会阻止其它线程获取当前对象的监控锁，这样就使得当前对象中被synchronized关键字保护的代码块无法被其它线程访问，也就无法并发执行。
关于volatile和synchronized的区别：
1)  volatile本质是在告诉当前线程中的工作内存中的值是不确定的，需要从主存中读取； synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
2)  volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的
3)  volatilebn能实现变量的修改可见性和有序性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性,有序性
4)  volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞

[Java并发编程：volatile关键字解析](https://www.cnblogs.com/dolphin0520/p/3920373.html)    
[全面理解Java内存模型(JMM)及volatile关键字](https://blog.csdn.net/javazejian/article/details/72772461)
## 6. synchronized关键字和lock的区别  
  1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现  
  2. synchronized发生异常时会自动释放线程占有的锁，防止死锁的发生；而Lock发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用finally块中释放锁  
  3. Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断  
  4. 通过Lock的tryLock()可以知道有没有成功获得锁，而synchronized 无法办到     
      
### 6.1 synchronized的低层实现  
java是用字节码指令来控制程序，在字节指令中，存在synchronized关键字的所包含的代码块会形成2段流程的执行。  
synchronized映射成字节码指令就是增加来两个指令：**monitorenter**和**monitorexit**。当一个线程执行遇到**monitorenter**指令时，他会尝试获得锁，如果获得锁，那么锁计数+1（为什么会加一呢，因为它是一个可重入锁，所以需要用这个锁计数判断锁的情况），如果没有获得锁，那么阻塞。当线程遇到**monitorexit**时，锁计数-1,当计数器为0，那么就会释放锁。  
synchronized锁释放有两种机制，一种就是执行完释放；另外一种就是发送异常，虚拟机释放。所以一个synchronized关键字会有两个**monitorexit**，其中一个会在程序异常时执行来释放锁。  
  
### 6.2 Lock低层实现  
**Lock**实现和**synchronized**不一样，后者是一种悲观锁，而**Lock**是采用CAS乐观锁，主要靠volatile和CAS低层实现。  
  
尽可能去使用synchronized而不要去使用LOCK。在Java1.5中，synchronize是性能低效的。因为这是一个重量级操作，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用Java提供的Lock对象，性能更高一些。但是到了Java1.6，发生了变化，官方也表示，他们也更支持synchronize，在未来的版本中还有优化余地。  
  
### 6.3 synchronized在JDK1.6以后的优化  
  
1. 线程自旋和适用性自旋  
  
 java线程其实是映射在内核之上的，线程的挂起和恢复会极大的影响开销。许多线程在等待锁的时候，在很短一段时间就获得了锁，所以它们在线程等待的时候，并不需要把线程挂起，而是让他无目的的循环，一般设置10次。这样就避免了线程切换的开销，极大的提升了性能。  
 **适用性自旋**，是赋予自旋一种学习能力，并不固定自旋10下，而是根据前面线程的自旋情况，调整自己的自旋。  
   
2. 锁清除：把不必要的同步在编译阶段进行移除  
3. 锁粗化：
4. 轻量级锁  
5. 偏向锁  
   
参考链接：[Java中synchronized与lock的区别与使用场景](https://my.oschina.net/u/4045381/blog/3082532)  
         [详解synchronized与Lock的区别与使用](https://blog.csdn.net/u012403290/article/details/64910926)
