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
- [7. 线程之间的协作](#7-线程之间的协作)
   - [7.1 sleep和wait方法](#71-sleep和wait方法)  
   - [7.2 notify和notifyAll方法](#72-notify和notifyAll方法)
   - [7.3 join方法](#73-join方法)
   
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
   
如果存在两个线程同时对一个主内存中的实例对象的变量进行操作就有可能诱发线程安全问题。
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
     
   - **ThreadPoolExecutor.AbortPolicy**：当任务添加到线程池中被拒绝时，它将抛出 RejectedExecutionException 异常。 
   - **ThreadPoolExecutor.CallerRunsPolicy**：当任务添加到线程池中被拒绝时，会在线程池当前正在运行的Thread线程池中处理被拒绝的任务。  
   - **ThreadPoolExecutor.DiscardPolicy**： 当任务添加到线程池中被拒绝时，线程池将丢弃被拒绝的任务 
   - **ThreadPoolExecutor.DiscardOldestPolicy**： 当任务添加到线程池中被拒绝时，线程池会放弃等待队列中最旧的未处理任务，然后将被拒绝的任务添加到等待队列中。
   
### 4.4 线程池执行流程  
     
线程池会按照以下流程执行任务：
  
- 当线程池中线程数小于核心线程数，会创建新线程  
- 当线程数大于等于核心线程数，任务队列未满，将任务放入任务队列  
- 当线程数大于等于核心线程数，且任务队列已满，则会有两种情况：  
      - 若当前线程数小于最大线程数，则创建新线程  
      - 若当前线程数等于最大线程数，则采取拒绝策略  
  
![图解线程池实现原理](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/图解线程池实现原理.png)  
  
更多详细内容，强烈推荐[java线程池学习总结](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93.md)

### submit和execute方法
**把一个任务加入线程池可以使用submit和execute方法**，submit方法有返回future。
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
### volatile为什么不能保证原子性  
  
Java中只有对基本类型变量的赋值和读取是原子操作，如i = 1的赋值操作，但是像j = i或者i++这样的操作都不是原子操作，因为他们都进行了多次原子操作，比如先读取i的值，再将i的值赋值给j，两个原子操作加起来就不是原子操作了。  

内存屏障是线程安全的,但是内存屏障之前的指令并不是.在某一时刻线程1将i的值load取出来，放置到cpu缓存中，然后再将此值放置到寄存器A中，然后A中的值自增1（寄存器A中保存的是中间值，没有直接修改i，因此其他线程并不会获取到这个自增1的值）。如果在此时线程2也执行同样的操作，获取值i==10,自增1变为11，然后马上刷入主内存。此时由于线程2修改了i的值，实时的线程1中的i==10的值缓存失效，重新从主内存中读取，变为11。接下来线程1恢复。将自增过后的A寄存器值11赋值给cpu缓存i。这样就出现了线程安全问题。

[volatile为什么不能保证原子性？ - 千帆的回答 - 知乎](https://www.zhihu.com/question/329746124/answer/1205806238)    
[Java并发编程：volatile关键字解析](https://www.cnblogs.com/dolphin0520/p/3920373.html)    
[全面理解Java内存模型(JMM)及volatile关键字](https://blog.csdn.net/javazejian/article/details/72772461)
## 6. synchronized关键字和lock的区别  
  1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现  
  2. synchronized发生异常时会自动释放线程占有的锁，防止死锁的发生；而Lock发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用finally块中释放锁  
  3. Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断  
  4. 通过Lock的tryLock()可以知道有没有成功获得锁，而synchronized 无法办到     
      
### 6.1 synchronized的低层实现  
synchroized可以用于**方法和代码块**。
```java
public class SynchronizedDemo {
//同步方法
    public synchronized void doSth(){
        System.out.println("Hello World");
    }

//同步代码块
    public void doSth1(){
        synchronized (SynchronizedDemo.class){
            System.out.println("Hello World");
        }
    }
}
```
**重量级锁也就是通常说synchronized的对象锁**，锁标识位为10，其中指针指向的是monitor对象（也称为管程或监视器锁）的起始地址。每个对象都存在着一个 monitor 与之关联，Monitor是基于C++实现。  
   
   
- synchronied用于方法时：同步方法的常量池中会有一个**ACC_SYNCHRONIZED标志**。当某个线程要访问某个方法的时候，会检查是否有ACC_SYNCHRONIZED，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。     
  
  
- synchronized用于代码块时：同步代码块使用monitorenter和monitorexit两个指令实现。可以把执行monitorenter指令理解为加锁，执行monitorexit理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行monitorenter）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行monitorexit指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。
 
synchronized锁释放有两种机制，一种就是执行完释放；另外一种就是发送异常，虚拟机释放。所以一个synchronized关键字会有两个**monitorexit**，其中一个会在程序异常时执行来释放锁。  
  
### 6.2 Lock低层实现  
lock是一个接口，主要实现为 ```ReentrantLock``` 。ReentrantLock是Java并发包中提供的一个可重入的互斥锁，所谓可重入性：**可以支持一个线程对锁的重复获取**。ReentrantLock有**公平和非公平两种方式**。
- **公平锁**：顾名思义，意指锁的获取策略相对公平，当多个线程在获取同一个锁时，必须按照锁的申请时间来依次获得锁，排排队，不能插队
- **非公平锁**：当锁被释放时，等待中的线程均有机会获得锁
   
synchronized是非公平锁，**ReentrantLock默认也是非公平的**，但是可以通过带boolean参数的构造方法指定使用公平锁，**但非公平锁的性能一般要优于公平锁**。  

ReentrantLock是一种可重入的，可实现公平性的互斥锁，它的设计基于AQS框架，可重入和公平性的实现逻辑都不难理解，每重入一次，state就加1，当然在释放的时候，也得一层一层释放。至于公平性，在尝试获取锁的时候多了一个判断：是否有比自己申请早的线程在同步队列中等待，若有，去等待；若没有，才允许去抢占。　　

#### ReentrantLock的实现原理
**ReentrantLock是基于AQS的，AQS是Java并发包中众多同步组件的构建基础，它通过一个int类型的状态变量state和一个FIFO队列来完成共享资源的获取，线程的排队等待等。**  
ReentrantLock其内部定义了三个重要的静态内部类，**Sync，NonFairSync，FairSync**。Sync作为ReentrantLock中公用的同步组件，继承了AQS（要利用AQS复杂的顶层逻辑嘛，线程排队，阻塞，唤醒等等）；
  
**NonFairSync和FairSync则都继承Sync，调用Sync的公用逻辑，然后再在各自内部完成自己特定的逻辑（公平或非公平）。**  

#### NonFairSync（非公平可重入锁）
1. 先获取state值，若为0，意味着此时没有线程获取到资源，CAS将其设置为1，设置成功则代表获取到排他锁了
2. 若state大于0，肯定有线程已经抢占到资源了，此时再去判断是否就是自己抢占的，是的话，state累加，返回true，重入成功，state的值即是线程重入的次数；
3. 其他情况，则获取锁失败。

#### FairSync(公平可重入锁)
1. 公平锁的大致逻辑与非公平锁是一致的，不同的地方在于有了!hasQueuedPredecessors()这个判断逻辑，即便state为0，也不能贸然直接去获取，要先去看有没有还在排队的线程，若没有，才能尝试去获取，做后面的处理。反之，返回false，获取失败。

### 6.3 synchronized在JDK1.6以后的优化  
Java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统的帮忙，这就要从用户态转换到核心态，因此状态转换需要花费很多的处理器时间，对于代码简单的同步块（如被synchronized修饰的get 或set方法）状态转换消耗的时间有可能比用户代码执行的时间还要长，所以说synchronized是java语言中一个重量级的操纵。所以在JDK1.6中出现对锁进行了很多的优化，进而出现轻量级锁，偏向锁，锁消除，适应性自旋锁，锁粗化(自旋锁在1.4就有 只不过默认的是关闭的，jdk1.6是默认开启的)，这些操作都是为了在线程之间更高效的共享数据，解决竞争问题。
  
  
锁的状态总共有四种，**无锁状态、偏向锁、轻量级锁和重量级锁**。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁，但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级，  
#### 线程自旋和适用性自旋  
  
 java线程其实是映射在内核之上的，线程的挂起和恢复会极大的影响开销。许多线程在等待锁的时候，在很短一段时间就获得了锁，所以它们在线程等待的时候，并不需要把线程挂起，而是让他无目的的循环，一般设置10次。这样就避免了线程切换的开销，极大的提升了性能。  
   
 
自旋等待不能代替阻塞，且先不说对处理器数量的要求，自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的，因此，如果锁被占用的时间很短，自旋等待的效果就会非常好，反之，如果锁被占用的时间很长，那么自旋的线程只会白白消耗处理器资源，而不会做任何有用的工作，反而会带来性能上的浪费。因此，自旋等待的时间必须要有一定的限度，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程了。自旋次数的默认值是 10 次
  
**自适应意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定**。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将允许自旋等待持续相对更长的时间，比如100个循环。另外，如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能省略掉自旋过程，以避免浪费处理部资源。有了自适应自旋，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测就会越来越准确。

   
#### 锁清除  
  
  
**锁消除是指虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除**。锁消除的主要判定依据来源于**逃逸分析的数据支持**，如果判断在一段代码中，堆上的所有数据都不会逃逸出去从而被其他线程访问到，那就可以把它们当做栈上数据对待，认为它们是线程私有的，同步加锁自然就无须进行。


3. 锁粗化：
4. 轻量级锁  
5. 偏向锁  
   
[Java中synchronized与lock的区别与使用场景](https://my.oschina.net/u/4045381/blog/3082532)  
[详解synchronized与Lock的区别与使用](https://blog.csdn.net/u012403290/article/details/64910926)  
[深入理解Java并发之synchronized实现原理](https://blog.csdn.net/javazejian/article/details/72828483)  
## 乐观锁和悲观锁
### 何为乐观锁和悲观锁
#### 悲观锁
总是假设最坏的情况，每次操作数据都会加上锁，这样其他想操作这个数据就会阻塞直到它拿到锁。**共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程**。传统的关系数据库里就用到很多这种锁机制，比如表锁、行锁、读锁，写锁等，都是在做操作之前先上锁。Java中的```synchronized和ReentrantLock```都是悲观锁。
#### 乐观锁
总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制和CAS算法实现。Java中的```java.util.concurrent.atomic```包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。

#### 使用场景
- **乐观锁适合于多读少写的场景**
- **悲观锁适合多写的场景**
#### 乐观锁的两种实现方式
- **版本号机制**
  一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功，也就是提交版本必须大于记录当前版本才能执行更新。
- **CAS算法**  
  即**compare and swap(比较与交换)**，是一种有名的无锁算法。无锁编程，即不使用锁的情况下实现多线程之间的变量同步，也就是在没有线程被阻塞的情况下实现变量的同步，所以也叫非阻塞同步（Non-blocking Synchronization)。CAS算法涉及到三个操作数：
     - 需要读写的内存值V
     - 需要比较的值A
     - 拟写入的新值B
   当且仅当V的值等于A时，CAS通过原子方式用新值B来更新V，否则不会执行任何操作(比较和替换时一个原子操作)
#### 乐观锁的缺点
- **ABA问题**
  如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个问题被称为CAS操作的ABA问题。
- **循环时间长开销大**
  自旋CAS（也就是不成功就一直循环执行直到成功）如果长时间不成功，会给CPU带来非常大的执行开销

  
[面试必备之乐观锁与悲观锁](https://juejin.im/post/6844903639207641096)
## 7. 线程之间的协作   
### 7.1 sleep和wait方法
  - sleep()是线程类Thread的方法，而wait是Object类的方法  
  - sleep不会释放当前对象的锁，到时间后会继续执行，wait()方法则是指当前线程让自己暂时退让出同步资源锁，以便其他正在等待该资源的线程得到该资源进而运行，只有调用了notify()/notifyAll()方法，之前调用wait()的线程才会解除wait状态，可以去参与竞争同步资源锁，进而得到执行  
  - 在线程中，调用sleep（0）可以释放cpu时间，让线程马上重新回到就绪队列而非等待队列，sleep(0)释放当前线程所剩余的时间片（如果有剩余的话），这样可以让操作系统切换其他线程来执行，提升效率
### 7.2 notify和notifyAll方法
    
  这两个方法都是Object对象的方法，不需要继承Thread类  
  - 如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁。  
  - 当有线程调用了对象的 notifyAll()方法（唤醒所有 wait 线程）或 notify()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争。  
  - notify可能会导致死锁，而notifyAll则不会。  
    
  [java中的notify和notifyAll有什么区别](java中的notify和notifyAll有什么区别？ - 知乎用户的回答 - 知乎
https://www.zhihu.com/question/37601861/answer/145545371)  
  
### 7.3 join方法  
在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。
## 死锁
锁是一种特定的程序状态，在实体之间，由于循环依赖导致彼此一直处于等待之中，没有任何个体可以继续前进。死锁不仅仅是在线程之间会发生，存在资源独占的进程之间同样也可能出现死锁。通常来说，我们大多是聚焦在多线程场景中的死锁，指两个或多个线程之间，由于互相持有对方需要的锁，而永久处于阻塞的状态。  
  
![ea88719ec112dead21334034c9ef8a6c.png](https://i.loli.net/2020/08/31/VzB3OCLGxPpoUHy.png)
### java定位死锁的过程
- 首先，可以使用**jps或者系统的ps命令**、任务管理器等工具，确定进程ID
- 其次，调用**jstack**获取线程栈
- 最后，结合代码分析线程栈信息

## 线程打印问题
### 两个线程交替打印奇数和偶数
-  使用synchronized
   ```java
   public class useSynchronized2  implements Runnable{
       //static 保证变量的可见性
       static int value=1;
       @Override
       public void run() {
           while(value<=100){
               //锁住的是class
               synchronized (useSynchronized2.class){
                   System.out.println(Thread.currentThread().getName()+":"+value++);
                   //唤醒其他线程
                   useSynchronized2.class.notify();
                   //增加判断，如果当前value>100，唤醒wait的线程，结束程序
                   try {
                       if(value>100){
                           useSynchronized2.class.notifyAll();
                       }else{
                           useSynchronized2.class.wait();
                       }
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
               }
           }
       }

       public static void main(String[] args) {

           new Thread(new useSynchronized2(),"奇数").start();
           new Thread(new useSynchronized2(),"偶数").start();
       }
   }
   ```
-  使用lock
   ```java
   public class useLock  implements Runnable{
    //通过lock锁来保证线程的访问的互斥
    static Lock lock=new ReentrantLock();
    static int value=1;

    @Override
    public void run() {
        while (value<=100){
            try {
                lock.lock();
                System.out.println(Thread.currentThread().getName()+":"+value++);
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        new Thread(new useLock(),"奇数").start();
        new Thread(new useLock(),"偶数").start();
    }
   }
   ```
   
### 线程交替打印ABC
- 使用synchornized
   ```java
   public class printABC {
       static class PrintThread implements Runnable{
           String name;
           Object pre;
           Object self;
           PrintThread(String name,Object pre,Object self){
               this.name=name;
               this.pre=pre;
               this.self=self;
           }
           @Override
           public void run() {
               int count=10;
               while (count>0){
                   synchronized (pre){
                       synchronized (self){
                           System.out.print(name);
                           count--;
                           //唤醒其他线程竞争self
                           self.notifyAll();
                       }
                       try {
                           //当前线程阻塞
                           if(count==0){
                               //如果count==0,表示这是最后一次打印操作，通过notifyAll操作释放对象锁。
                               pre.notifyAll();
                           }else
                           pre.wait();
                       } catch (InterruptedException e) {
                           e.printStackTrace();
                       }
                   }
               }
           }
       }

       public static void main(String[] args) throws InterruptedException {
           Object a=new Object();
           Object b=new Object();
           Object c=new Object();

           PrintThread pta=new PrintThread("A",c,a);
           PrintThread ptb=new PrintThread("B",a,b);
           PrintThread ptc=new PrintThread("C",b,c);

           new Thread(pta).start();
           Thread.sleep(10);

           new Thread(ptb).start();
           Thread.sleep(10);

           new Thread(ptc).start();
           Thread.sleep(10);
       }
   }
   ```
