<!-- TOC -->
- [1. 关于String](#1-关于String)   
- [2. 关于==和equals](#2-关于==和equals)  
- [3. 关于Integer](#3-关于Integer)
- [4. 关于hashMap](#4-关于hashMap)
  - [4.1 hashMap的结构](#41-hashMap的结构)  
  - [4.2 怎么确定桶的索引](#42-怎么确定桶的索引)  
  - [4.3 hashMap的扩容机制](#43-hashMap的扩容机制)  
  - [4.4 使用ConcurrentHashMap](#44-使用ConcurrentHashMap)  
- [5. 关于ArrayList和LinkedList](#4-关于ArrayList和LinkedList)  
- [6. java抽象类和接口](#5-java抽象类和接口)  
- [7. jdk1.8相对于1.7有什么变化](#6-jdk1.8相对于1.7有什么变化)  
- [8. java内部类](#7-java内部类)  
- [9. BIO、NIO和AIO的区别](#8-BIO、NIO和AIO的区别)

<!-- /TOC -->
## 1. 关于String  
   String类是我们平常项目中使用频率非常高的一种对象类型，jvm为了提升性能和减少内存开销，避免字符的重复创建，其维护了一块特殊的内存空间，即字符串池，当需要使用字符串时，先去字符串池中查看该字符串是否已经存在，如果存在，则可以直接使用，如果不存在，初始化，并将该字符串放入字符创常量池中。  
   String类是final类，这意味着不允许任何人定义String的子类。使用 new String 的话可能至少会创建一个对象,即使是里面的值相同，但是 使用==比较的是内存地址，会输出false.  
   
### String、StringBuffer和 StringBuilder
   String是字符串常量,StringBuffer和StringBuilder 是字符串变量,使用 + 的话会创建一个新的字符串对象。 
   String长度不可变，StringBuffer和StringBuilder都是可变的。
   StringBuffer是线程安全的，StringBuilder是非线程安全的。这是因为Stringbuffer中方法大都采用了synchronized的关键字修饰。
## 2. 关于==和equals
   ==比较的是变量的内存地址，用来判断两个变量是否为同一对象，有相同的内存地址。equals一般比较对象中的内容是否相同，可以通俗的理解我们是否赋值的相同。  
   默认情况下，当一个类从Object类继承而来，如果没有重写equals，那么equals和“==”的作用是相同的，都是比较内存地址。因此我们可以按照我们自己的需求，来重写equals。
#### 重写equals为什么要重写hashCode()方法？
   重写equals我们一般是希望判断对象里面的内容是否相同，两个对象如果equals返回true，我们认为即对象相等，2个对象的equals()方法返回true的话，其hashCode()必须返回相同的值。否则对于HashSet, HashMap, HashTable等基于hash值的类就会出现问题。
## 3. 关于Integer
关于Integer这个类，它是重写了equals方法的，比较的是两个Intger对象的值是否相同。  
如果将一个整型直接赋值给一个Integer对象，会发生自动装箱，比如： Integer a=200。它相当于 Integer a=Integer.valueOf(200).下面valueOf方法的源代码：  
```
public static Integer valueOf(int i) {  
  assert IntegerCache.high >= 127;  
  if (i >= IntegerCache.low && i <= IntegerCache.high)  
    return IntegerCache.cache[i + (-IntegerCache.low)];  
  return new Integer(i);  
}
```
可以看到有一个IntegerCache，默认IntegerCache.low 是-127，Integer.high是128，如果参数中的i在这个范围之内，则返回一个数组中的内容，
如果不在这个范围，则new一个新的Integer对象并返回。  

两个自动装箱的变量，但是装箱传递的值大于127，比如 Integer a=200,b=200; a==b; 那么a==b 就会返回false。因为值不在-128-127这个返回，如果在这个范围那么返回的是true.
自动拆箱是一个整型和一个Integer对象进行比较如： int a=200;Integer b=200; a==b; 那么b就会自动拆箱调用**intValue**方法，得到对象里的值再进行比较。  
当一个整型传入Integer对象的equals方法里，也会发生自动装箱，java会自动将这个值打包装箱为Integer类。  

因此可以得出：  
 1). Integer类里的equals方法是重写过的，比较的是值  
 2). 发生自动装箱，那么要根据传入的值进行判断，如果不在[-128,127]这个范围，就会new一个Integer对象.  

[java基础中Integer值用==和equals判断相等问题解析](https://blog.csdn.net/w112736112736/article/details/77986283)
## 4. 关于hashMap
### 4.1 hashMap的结构
   从存储结构来讲，hashMap内部使用Node实现，Node是HashMap的一个内部类，实现了Map.Entry接口，本质是就是一个映射(键值对)。一个Node对象有四个属性：hash值，key，value，还有链表的下一个Node.  
   
   HashMap结构是数组+链表+红黑树（JDK1.8增加了红黑树部分),也就是解决hash冲突中的链地址法。简单来说就是数组加链表的结构，当key的到对应的hash值后，就得到数组的下表(哈希桶下标)，然后把数据放到对应数组的链表中。
   
   hashMap初始桶的大小为16，当链表的长度大于8时，转化为红黑树。负载因子是0.75,阀值threshold=length*0.75是hashMap所能容纳的最大数量，当hashMap的size大于阀值时，就会扩容，将length变为原来的2倍。
### 4.2 怎么确定桶的索引
   确定桶的索引即是hashMap的hash算法：
   1.  获取key的hash值
   2.  将hash值的高16位与低16位进行异或运算，得到h
   3.  将 h&(length-1) 得到桶的索引
   
   为什么这么计算？ 
   1.  将高位与低位异或，这么做可以在length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，是桶的索引的分散更加均匀，减少hash碰撞同时不会有太大的开销。
   2.  h&(length-1) 当length总是2的n次方时，h& (length-1)运算等价于对length取模，也更加的高效，同时也解释为什么length总是2的幂次方。
   
### 4.3 hashMap的扩容机制
   扩容(resize)即当前hashMap的size大于阀值，就需要扩大通道额数量。长度变为原来的2倍。
   为什么长度要变为原来的两倍？首先是上面hash值的计算，然后当我们扩容为原来的两倍时，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。
   我们不需要再重新像jdk1.7那样重新计算hash值，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+原来的hashMpa的长度”。  
   
   ![Alt](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/4d8022db.png)
   ![Alt](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/d773f86e.png)
   
#### 为什么hashMap是线程不安全的
   HashMap是线程不安全的，它的不安全就体现在resize的时候，多线程的情况下，可能会形成环形链表，导致下一次读取的时候可能会出现死循环。
   
### 4.4 使用ConcurrentHashMap
   
   ConcurrentHashMap在java1.8中，抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性。
   结构同hashMap一样，内部都是Node节点存储数据，但是node节点中的val 和 next都使用了volatile保证可见性。
   在put的时候会利用 synchronized 锁写入数据。
## 5. 关于ArrayList和LinkedList
   ArrayList为动态数组,随机访问元素效率高，删除元素或者向数组总添加元素效率更低。
   LinkedList是链表数组，数据添加，删除效率高，只需要改变指针即可，但访问元素效率低。
   
### 5.1 关于ArrayList
   ArrayList默认大小为10，当加入元素个数大于10时，会扩容到原来大小的1.5倍。扩容，是重新定义一个原来大小1.5倍大小的数组，将原来的数据复制到新数组中，再把对象指向新的数组地址即可。  
   ArrayList生成对象时，不会分配给空间，此时调用size()会得到0值。只有当第一次调用add时，才会分配空间，调用size会得到相应的元素个数。
   
## 6. java抽象类和接口

抽象类是对事物的抽象,而接口是对行为的抽象，抽象类一般是为继承而存在的,继承这个抽象类是为了使用这个类的属性及方法,实现接口是使用它规范的某一行为。在语法层次上，它们有如下区别:
1)  抽象类中成员变量可以为各种类型，接口中成员变量在**JDK1.8**中可以有public 和protected的静态变量。
2)  接口中在**JDK1.7**不能含有静态代码块及静态方法，但是**JDK1.8**中接口可以有默认方法，就是一个在接口里面有了一个实现的方法,使用**default**修饰还可以有静态方法。
3)  一个类只能继承一个抽象类，但是可以实现多个接口。
## 7. jdk1.8相对于1.7有什么变化
1.  **lambda表达式**：可以将函数作为参数传递进方法中，使代码更加简洁，可以简化匿名内部类的书写，但是这个匿名内部类必须是个函数接口。比如线程的创建runnable
2.  **方法引用**： 通过类名称::方法的名字来指向一个方法,使用双冒号 :: .比如 System.out:println
3.  **默认方法**:  默认方法就是一个在接口里面有了一个实现的方法,使用关键字default,还可以有静态变量及方法。
4.  **函数接口**： 是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。可以转化为lambda表达式
5.  **引入Stream**: 函数式编程的核心，类似于spark RDD。有中间操作和结束操作 类似于spark的transformer 和 action.

## 8. java内部类
将一个类定义在类里面或者是方法里,这个定义的类就被称为内部类。使用内部类最吸引人的原因是：**每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。**    

内部类是个编译时的概念，一旦编译成功后，它就与外围类属于两个完全不同的类（当然他们之间还是有联系的）。对于一个名为OuterClass的外围类和一个名为InnerClass的内部类，在编译成功后，会出现这样两个class文:OuterClass.class和OuterClass$InnerClass.class。

####  成员内部类:
定义于类的内部，可以看做是类的一个成员。可以访问外部类的所有属性和方法，尽管是private修饰的，外部类要访问内部类需要先创建内部类的对象，才能进行访问。需要注意的是：

1.  成员内部类中不能存在任何static的变量和方法。
2.  成员内部类是依附于外围类的，所以只有先创建了外围类才能够创建内部类。  
创建内部类对象时，需要使用下面格式:
```
OuterClass.InnerClass innerClass = outerClass.new InnerClass();
```
####  局部内部类
定义在一个方法内或者作用域内，只能在这个作用域内使用

####  匿名内部类
匿名内部类没有构造函数,一般用来继承其他类或者实现接口。在日常编程中，监听器，比较器，线程创建都可以使用匿名内部类。
####  静态内部类
相当于静态成员。可以不依靠外部类创建，直接使用```new InnerClass1()```。静态内部类只能访问外围类的静态成员变量和方法,不能访问外围类的非静态成员变量和方法。

## 9. BIO、NIO和AIO的区别

#### 同步和异步
当你同步执行某项任务时，你需要等待其完成才能继续执行其他任务。当你异步执行某些操作时，你可以在完成另一个任务之前继续进行。  

**同步**两个同步任务相互依赖，并且一个任务必须以依赖于另一任务的某种方式执行  
**异步**两个异步的任务完全独立的，一方的执行不需要等待另外一方的执行。

#### 阻塞和非阻塞
**阻塞** 阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才能继续。  
**非阻塞** 非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先去干其他事情。  
  
 
同步/异步是从行为角度描述事物的，而阻塞和非阻塞描述的当前事物的状态
  
**BIO**:同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。
**NIO**:同步非阻塞的I/O模型
**AIO**:异步非阻塞IO模型

[如何理解BIO、NIO、AIO的区别](https://juejin.im/post/5dbba5df6fb9a0204a08ae55#heading-6)  
[BIO,NIO,AIO 总结](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/BIO-NIO-AIO.md#bionioaio-%E6%80%BB%E7%BB%93)
