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
   String类是我们平常项目中使用频率非常高的一种对象类型，jvm为了提升性能和减少内存开销，避免字符的重复创建，其维护了一块特殊的内存空间，即字符串池，字符串池一般在**方法区**中创建。当需要使用字符串时，先去字符串池中查看该字符串是否已经存在，如果存在，则可以直接使用，如果不存在，初始化，并将该字符串放入字符创常量池中。  
   String类是final类，这意味着不允许任何人定义String的子类。使用 new String 的话可能至少会创建一个对象,即使是里面的值相同，但是 使用==比较的是内存地址，会输出false.      
     
   
 (String：字符串常量池)[https://segmentfault.com/a/1190000009888357]  
### String、StringBuffer和 StringBuilder
 -  String是字符串常量,StringBuffer和StringBuilder 是字符串变量,使用 + 的话会创建一个新的字符串对象。   
 -  为了实现修改字符序列的目的，StringBuffer和StringBuilder底层都是利用可修改的（char，JDK 9以后是byte）数组，构建时初始字符串长度为16。区别在于是否加入了snychronized关键字
 -  String长度不可变，StringBuffer和StringBuilder都是可变的。  
 -  StringBuffer是线程安全的，StringBuilder是非线程安全的。这是因为Stringbuffer中方法大都采用了synchronized的关键字修饰。  
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
可以看到有一个IntegerCache，默认IntegerCache.low 是-128，Integer.high是127，如果参数中的i在这个范围之内，则返回一个数组中的内容，
如果不在这个范围，则new一个新的Integer对象并返回。  

两个自动装箱的变量，但是装箱传递的值大于127，比如 Integer a=200,b=200; a==b; 那么a==b 就会返回false。因为值不在-128-127这个返回，如果在这个范围那么返回的是true.
自动拆箱是一个整型和一个Integer对象进行比较如： int a=200;Integer b=200; a==b; 那么b就会自动拆箱调用**intValue**方法，得到对象里的值再进行比较。  
当一个整型传入Integer对象的equals方法里，也会发生自动装箱，java会自动将这个值打包装箱为Integer类。  

因此可以得出：  
 1). Integer类里的equals方法是重写过的，比较的是值  
 2). 发生自动装箱，那么要根据传入的值进行判断，如果不在[-128,127]这个范围，就会new一个Integer对象.  

[java基础中Integer值用==和equals判断相等问题解析](https://blog.csdn.net/w112736112736/article/details/77986283)
## java基本概念
#### Java⾯向对象编程三⼤特性
- **封装**：即将事物进行封装
- **继承**：子类继续父类
- **多态**：所谓多态就是指程序中定义的引⽤变量所指向的具体类型和通过该引⽤变量发出的⽅法调⽤在编程时并不确定，⽽是在程序运⾏期间才确定，即⼀个引⽤变量到底会指向哪个类的实例对象，该引⽤变量发出的⽅法调⽤到底是哪个类中实现的⽅法，必须在由程序运⾏期间才能决定。
#### 重载和重写
- **重载**：发生在同一类中，**方法名相同，其他可以不同**，参数类型不同、个数不同、顺序不同，⽅法返回值和访问修饰符可以不同。
- **重写**：重写发⽣在运⾏期，是⼦类对⽗类的允许访问的⽅法的实现过程进⾏重新编写。
#### java平台的理解
Java本身是一种面向对象的语言，最显著的特性有两个方面，一是所谓的“书写一次，到处运行”（Write once, run anywhere），能够非常容易地获得跨平台能力；另外就是垃圾收集（GC, Garbage Collection），Java通过垃圾收集器（Garbage Collector）回收分配内存，大部分情况下，程序员不需要自己操心内存的分配和回收。  
![20bc6a900fc0b829c2f0e723df050732.png](https://i.loli.net/2020/08/18/flGs2yUA5kQm6Tc.png)  
我们日常会接触到JRE（Java Runtime Environment）或者JDK（Java Development Kit）。 JRE，也就是Java运行环境，包含了JVM和Java类库，以及一些模块等。而JDK可以看作是JRE的一个超集，提供了更多工具，比如编译器、各种诊断工具等。
#### java是解释运行还是编译运行
我们开发的Java的源代码，首先通过**Javac编译成为字节码（bytecode)**，然后，在运行时，通过 Java虚拟机（JVM）内嵌的解释器将字节码转换成为最终的机器码。但是常见的JVM，比如我们大多数情况使用的Oracle JDK提供的Hotspot JVM，都提供了JIT（Just-In-Time）编译器，也就是通常所说的动态编译器，JIT能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就属于编译执行，而不是解释执行了。
## Exception和Error的区别  

Exception和Error都是继承了Throwable类，在Java中只有Throwable类型的实例才可以被抛出（throw）或者捕获（catch），它是异常处理机制的基本组成类型。    
  
Exception和Error体现了Java平台设计者对不同异常情况的分类。Exception是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应处理。  
  
Error是指在正常情况下，不大可能出现的情况，绝大部分的Error都会导致程序（比如JVM自身）处于非正常的、不可恢复状态。既然是非正常情况，所以不便于也不需要捕获，常见的比如OutOfMemoryError之类，都是Error的子类。  
  
Exception又分为可检查（checked）异常和不检查（unchecked）异常，可检查异常在源代码里必须显式地进行捕获处理，这是编译期检查的一部分。前面我介绍的不可查的Error，是Throwable不是Exception。
  
不检查异常就是所谓的运行时异常，类似 NullPointerException、ArrayIndexOutOfBoundsException之类，通常是可以编码避免的逻辑错误，具体根据需要来判断是否需要捕获，并不会在编译期强制要求。
#### 捕捉异常的开销
- try-catch代码段会产生额外的性能开销，或者换个角度说，它往往会影响JVM对代码进行优化，所以建议仅捕获有必要的代码段，尽量不要一个大的try包住整段的代码；与此同时，利用异常控制代码流程，也不是一个好主意，远比我们通常意义上的条件语句（if/else、switch）要低效。  
- Java每实例化一个Exception，都会对当时的栈进行快照，这是一个相对比较重的操作。如果发生的非常频繁，这个开销可就不能被忽略了
#### 捕捉异常的实践
- **尽量不要捕捉类似Exception的通用异常，让程序更加的明了，可读性更高**。如下所示
  ```java
  try {
  // 业务代码
  Thread.sleep(1000L);
  } catch (Exception e) { //这里应该捕捉 Thread.sleep()抛出的InterruptedException
  //处理
  }
  ```
- **不要生吞异常，即直接输出异常**，如下所示
  ```java
  try {
   // 业务代码
  } catch (IOException e) {
    e.printStackTrace(); //不要直接输入异常，应该输出到日志文件中
  }
  ```
- **Throw early,catch late原则**
  ```java
  public void readFile(String filename){
     Objects.requireNonNull(filname)  //throw early
     // some operations
     InputStream int=new FileInputStream(filename);
     // some operations
  }
  ```
  如果fileName是null，那么程序就会抛出NullPointerException，但是由于没有第一时间暴露出问题，堆栈信息可能非常令人费解，往往需要相对复杂的定位。所以一开始在方法里就判断是否为空
#### NoClassDefFoundError和ClassNotFoundException
NoClassDefFoundError是一个**错误(Error)**，而 ClassNOtFoundException 是**一个异常**。  
- **ClassNotFoundException**产生原因:
   -  Java支持使用 Class.forName 方法来动态地加载类，任意一个类的类名如果被作为参数传，递给这个方法都将导致该类被加载到 JVM 中。如果这个类在类路径中没有被找到，那么此时就会在运行时抛出 ClassNotFoundException 异常.
   -  当一个类已经某个类加载器加载到内存中了，此时另一个类加载器又尝试着动态地从同一个包中加载这个类。
- **NoClassDefFoundError 产生的原因**:
   -  当 Java 虚拟机 或 ClassLoader 实例试图在类的定义中加载（作为通常方法调用的一部分，或者是使用 new 来创建新的对象）时，却找不到类的定义（要查找的类在编译的时候是存在的，运行的时候却找不到了），抛出此异常。即当前执行的类被编译时，所搜索的类定义存在，但无法再找到该定义。 这个错误往往是你使用 new 操作符来创建一个新的对象，但却找不到该对象对应的类。这个时候就会导致NoClassDefFoundError
   
#### NoClassDefFoundError和ClassNotFoundException的区别
-   ClassNotFoundException 发生在装入阶段。当应用程序试图通过类的字符串名称，使用常规的三种方法装入类，但却找不到指定名称的类定义时就抛出该异常。
-   NoClassDefFoundError 当目前执行的类已经编译，但是找不到它的定义时。也就是说你如果编译了一个类B，在类A中调用，编译完成以后，你又删除掉B，运行A的时候那么就会出现这个错误  
-   加载时从外存储器找不到需要的 Class 就出现 ClassNotFoundException   
-   连接时从内存找不到需要的 class 就出现 NoClassDefFoundError
[ClassNotFoundException 和 NoClassDefFoundError 的区别](https://cloud.tencent.com/developer/article/1153789)  
## Java反射中获取Class对象三种方式的区别
- Class.forName("java.util.String"):获取对象的Class对象，根本不会调用对象中任何的代码块或代码   
- Object.class:调用静态代码块的内容
- new Object().getClass：要先实例化对象
## final、finally、 finalize的区别
- final可以用来修饰类、方法、变量，分别有不同的意义，final修饰的class代表不可以继承扩展，final的变量是不可以修改的，而final的方法也是不可以重写的（override）。在并发编程中，将变量设置为final可以保证程序的只读性。
- finally则是Java保证重点代码一定要被执行的一种机制。我们可以使用try-finally或者try-catch-finally来进行类似关闭JDBC连接、保证unlock锁等动作。
  ```java
  try {
  // do something
  System.exit(1);
  } finally{
    System.out.println(“Print from finally”); //这里的finally是不会输出的
  }
  ```
- finalize是基础类java.lang.Object的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize机制现在已经不推荐使用。
## 4. 关于hashMap
自己实现的MyHashMap[MyHashMap](./MyHashMap.md)
### 4.1 hashMap的结构
   从存储结构来讲，hashMap内部使用Node实现，Node是HashMap的一个内部类，实现了Map.Entry接口，本质是就是一个映射(键值对)。一个Node对象有四个属性：hash值，key，value，还有链表的下一个Node.  
   
   HashMap结构是数组+链表+红黑树（JDK1.8增加了红黑树部分),也就是解决hash冲突中的链地址法。简单来说就是数组加链表的结构，当key的到对应的hash值后，就得到数组的下表(哈希桶下标)，然后把数据放到对应数组的链表中。
   
   hashMap初始桶的大小为16，当链表的长度大于8时，转化为红黑树。负载因子是0.75,阀值threshold=length*0.75是hashMap所能容纳的最大数量，当hashMap的size大于阀值时，就会扩容，将length变为原来的2倍。  
   在jdk8以后，hashmap中的put是采用尾插法的，头插法会在多线程下产生循环链表。
### 4.2 怎么确定桶的索引
   确定桶的索引即是hashMap的hash算法：
   1.  获取key的hash值
   2.  将hash值的高16位与低16位进行异或运算，得到h
   3.  将 h&(length-1) 得到桶的索引
   ```java
   方法一
   static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    方法二：
    static int indexFor(int h, int length) {  //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
       return h & (length-1);  //第三步 取模运算
    }
   ```
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
   是线程不安全的。分别需要从jdk7和jdk8进行分析
   - jdk1.8  
   ```java
   final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    //map的数组
    Node<K,V>[] tab; Node<K,V> p; 
    int n, i;//n为数组的长度，i是当前要进入put的元素所在的数组的index
    if ((tab = table) == null || (n = tab.length) == 0)//数组为null或者长度为0.进行懒加载
        //初始化的resize()扩容
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        //tab中没有Node，也就是没有hash冲突，则new一个node,并且复制到数组，此时就会发生线程安全问题
        tab[i] = newNode(hash, key, value, null);
    else{
        //发生hash冲突
    }
   ```
   
   在put方法中，图中标识处，当两个线程同时到达此处，发现tab[i]都是null，则会new一个Node，此时如果一个线程挂起了，另一个线程完成了tab[i]的赋值，另一个线程此时重新执行，由于不会检查，认为此时还是null，则会进行覆盖，那么之前的数据就丢失了。  
   ```java
     //当容量大于阀值，就发生扩容
     // java8中 先插入值后再进行扩容，java7中如果大于阀值就先扩容再插入
     if (++size > threshold)
      resize();
   ```
   此外size不是volatile修饰，也没有进行同步，那么多线程情况就下，++size就会出现线程不安全。  
   **总结**：因为是尾插，不会产生7那种闭环死循环情况，jdk1.8中的hashMap线程不安全是在多线程并发情况下会产生数据丢失的问题.
   - jdk1.7中HashMap的put方法采用的是头插法，这会在resize也就是扩容的时候，有一个tranfor方法将作用旧数组上的数据转移到新table中，从而完成扩容，这时候在多线程环境下会产生循环链表
   ```java
    /**
   * 作用：将旧数组上的数据转移到新table中，从而完成扩容
   * 过程：按旧链表的正序遍历链表、在新链表的头部依次插入
   */
    void transfer(Entry[] newTable) {
          Entry[] src = table;              
          int newCapacity = newTable.length;
          // 通过遍历旧数组，将旧数组上的数据转移到新数组中
          for (int j = 0; j < src.length; j++) {  
              Entry e = src[j];          
              if (e != null) {
                  src[j] = null;
                  do {
                      // 注：转移链表时，因是单链表，故要保存下1个结点，否则转移后链表会断开
                      Entry next = e.next;
                     int i = indexFor(e.hash, newCapacity);
                     // 扩容后，出现逆序：1->2->3 => 因为头插法：3->2->1
                     e.next = newTable[i];
                     //这里会出现线程不安全，当一个线程在这里挂起
                     newTable[i] = e;
                     // 访问下1个Entry链上的元素，如此不断循环，直到遍历完该链表上的所有节点
                     e = next;            
                 } while (e != null);
             }
         }
     }
   ```
   就是当线程A执行到面线程不安全的地方，失去cpu时间片，当线程B完成了扩容，此时A获取到cpu时间片了，继续执行代码，这时候会出现两个Node的next互相指向，形成了闭环，也就是死循环，还有造成数据丢失。  
   **总结**：HashMap因此采用头插法，在多线程情况下，resize时会导致node的next相互指引，产生循环链表
#### 为啥链表为8就转为红黑树?为什么负载因子为0.75？
   在随机hash的情况下，进每个桶的概率遵循**泊松分布**，当这个桶内已经有8个node节点时，后续进入这个桶的概率接近于0
   ```
     * 0:    0.60653066
     * 1:    0.30326533
     * 2:    0.07581633
     * 3:    0.01263606
     * 4:    0.00157952
     * 5:    0.00015795
     * 6:    0.00001316
     * 7:    0.00000094
     * 8:    0.00000006
   ```
   负载因子为0.75是出于时间和空间的考虑，负载因子过大就会造成过多的hash冲突，过小就会引起空间的浪费，因此设为0.75
### 4.4 使用ConcurrentHashMap
   
   ConcurrentHashMap在java1.8中，抛弃了原有的 Segment 分段锁，而采用了 CAS + 变量volatile+putValue方法synchronized 来保证并发安全性。
   结构同hashMap一样，内部都是Node节点存储数据，但是node节点中的val 和 next都使用了volatile保证可见性。
   在put的时候会利用 synchronized 锁写入数据。

## 5. 关于ArrayList和LinkedList和Vector
-  Vector是Java早期提供的线程安全的动态数组，如果不需要线程安全，并不建议选择，毕竟同步是有额外开销的。Vector内部是使用对象数组来保存数据，可以根据需要自动的增加容量，当数组已满时，会创建新的数组，并拷贝原有数组数据
-  ArrayList是应用更加广泛的动态数组实现，它本身不是线程安全的，所以性能要好很多。与Vector近似，ArrayList也是可以根据需要调整容量，不过两者的调整逻辑有所区别，Vector在扩容时会提高1倍，而ArrayList则是增加50%。  
-  LinkedList顾名思义是Java提供的双向链表，所以它不需要像上面两种那样调整容量，它也不是线程安全的。
#### 区别
   ArrayList为动态数组,随机访问元素效率高，插入和删除效率低。
   LinkedList是链表数组，节点插入、删除却要高效得多，但是随机访问性能则要比动态数组慢。  
   - **是否保证线程安全**：ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全。如果ArrayList要使用线程安全的，需要使用**Vector**。    
   - **底层数据结构**：ArrayList底层使用**Object数组**，LinkedList使用的是**双向链表**数据结构
   
### 5.1 关于ArrayList
   ArrayList默认大小为10，当加入元素个数大于10时，会扩容到原来大小的1.5倍。扩容，是重新定义一个原来大小1.5倍大小的数组，将原来的数据复制到新数组中，再把对象指向新的数组地址即可。  
   ArrayList生成对象时，不会分配给空间，此时调用size()会得到0值。只有当第一次调用add时，才会分配空间，调用size会得到相应的元素个数。
   
## 6. java抽象类和接口

抽象类是对事物的抽象,而接口是对行为的抽象，抽象类一般是为继承而存在的,继承这个抽象类是为了使用这个类的属性及方法,实现接口是使用它规范的某一行为。在语法层次上，它们有如下区别:
1)  抽象类中成员变量可以为各种类型，接口中成员变量会被隐式的指定为 public static final 变量 
```java
    public interface testInterface {
    int a=1; // 被隐式的指定为 public static final 变量 
    static void showForInterface(){
        System.out.println(a);
    }
    
    default void testDefault(){
        //直接在implements的类里直接调用，testDefault(),不需要testInterface.
        System.out.println(a);
    }
    void funInInterface();
   }
```
2)  接口中在**JDK1.7**不能含有静态代码块及静态方法，但是**JDK1.8**中接口可以有默认方法，就是一个在接口里面有了一个实现的方法,使用**default**修饰还可以有静态方法。**静态方法需要调用接口名，而默认方法不需要，直接在实现接口的类中调用方法即可**。  
3)  一个类只能继承一个抽象类，但是可以实现多个接口。
## 7. jdk1.8相对于1.7有什么变化
1.  **lambda表达式**：可以将函数作为参数传递进方法中，使代码更加简洁，可以简化匿名内部类的书写，但是这个匿名内部类必须是个函数接口。比如线程的创建runnable
```java
//使用lamda表达式实现Comparator接口
ArrayList<Integer> list=new ArrayList<>();
Collections.sort(list,(a,b)->b-a);

Integer[] s=new Integer[7];
Arrays.sort(s,(a,b)->b-a);
//使用lamda表达式实现runnable接口
new Thread(()->{System.out.println("thread start");}).start();
```  
  
[函数式接口和Lambda表达式深入理解](https://www.jianshu.com/p/40f833bf2c48)  
  
[关于Java Lambda表达式看这一篇就够了](https://objcoding.com/2019/03/04/lambda/#lambda-and-anonymous-classesii)  
  

2.  **方法引用**： 通过类名称::方法的名字来指向一个静态方法或者构造函数引用,比如 System.out::println。  
```java
List<String> demoList = Arrays.asList("小明", "Zing", "阿三", "小红", "赵日天");
demoList.forEach(System.out::println);
```
3.  **默认方法**:  默认方法就是一个在接口里面有了一个实现的方法,使用关键字default,还可以有静态变量及方法。  
```java
public interface testInterface {
    int a=1;
    default void testDefault(){
        //直接在implements的类里直接调用，testDefault(),不需要testInterface.
        System.out.println(a);
    }
}
```
4.  **函数接口**： 是一个接口中**有且仅有一个抽象方法**，但是可以有多个非抽象方法的接口。可以转化为lambda表达式.可以使用**@FunctionalInterface**标记。  
```java
@FunctionalInterface //只有一个抽象方法
public interface testInterface {
    int a=1;
    //静态方法，非抽象
    static void showForInterface(){

        System.out.println(a);
    }
    //默认方法,非抽象
    default void testDefault(){
        //直接在implements的类里直接调用，testDefault(),不需要testInterface.
        System.out.println(a);
    }
    //唯一的一个抽象方法
    void funInInterface();
}
```
5.  **引入Stream**: 函数式编程的核心，类似于spark RDD。有中间操作和结束操作 类似于spark的transformer 和 action.  
6.  **HashMap性能优化**：引入了红黑树，当链表超过8时，链表就转换为红黑树。
## 8. java内部类
将一个类定义在类里面或者是方法里,这个定义的类就被称为内部类。使用内部类最吸引人的原因是：**每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。**    

内部类是个编译时的概念，一旦编译成功后，它就与外围类属于两个完全不同的类（当然他们之间还是有联系的）。对于一个名为OuterClass的外围类和一个名为InnerClass的内部类，在编译成功后，会出现这样两个class文:OuterClass.class和OuterClass$InnerClass.class。

####  成员内部类:
定义于类的内部，可以看做是类的一个成员。可以访问外部类的所有属性和方法，尽管是private修饰的，外部类要访问内部类需要先创建内部类的对象，才能进行访问。需要注意的是：

1.  成员内部类中不能存在任何static的变量和方法。
2.  成员内部类是依附于外围类的，所以只有先创建了外围类才能够创建内部类。  
创建内部类对象时，需要使用下面格式:
```java
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

## 10. java泛型   
泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型。
泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法。  
  
### 泛型类
泛型类定义时只需要在类名后面加上类型参数即可.  

```java
class DataHolder<T>{
    T item;
    
    public void setData(T t) {
    	this.item=t;
    }
    
    public T getData() {
    	return this.item;
    }
}
```  
### 泛型方法  
泛型方法既可以存在于泛型类中，也可以存在于普通的类中。如果使用泛型方法可以解决问题，那么应该尽量使用泛型方法。所有泛型方法声明都有一个类型参数声明部分（由尖括号分隔），该类型参数声明部分在方法返回类型之前。  
  
```java
public <E> void PrinterInfo(E e) {
    	System.out.println(e);
}
```
泛型方法里面的类型参数E和泛型类里面的类型参数(如果类是一个泛型类)是不一样的类型，既然如果泛型类声明为 ```class testClass<E>```，并且声明的该类的一个对象```testClass<String> tc=new testClass<>();```,声明对象里的为String类型，那么PrinterInfo接受的参数可以为其他类型。
  
### 泛型接口
Java泛型接口的定义和Java泛型类基本相同,在声明类的时候，需将泛型的声明也一起加到类中。
```java
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}

/* 即：class DataHolder implements Generator<T>{
 * 如果不声明泛型，如：class DataHolder implements Generator<T>，编译器会报错："Unknown class"
 */
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}

```  
  
[深入理解Java泛型](https://juejin.im/post/5b614848e51d45355d51f792#heading-4)
## 11. java反射
Java 反射机制在程序**运行时**，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性。这种**动态的获取信息**以及**动态调用对象的方法**的功能称为 **java的反射机制**。  
  
### 使用反射获取类的信息    
  

1.  获取类的所有变量信息  
```java
//1.获取并输出类的名称
Class mClass = SonClass.class;
System.out.println("类的名称：" + mClass.getName());

//2.1 获取所有 public 访问权限的变量
// 包括本类声明的和从父类继承的
Field[] fields = mClass.getFields();
```  
  
2. 获取类的所有方法信息  
```java
//1.获取并输出类的名称
Class mClass = SonClass.class;
System.out.println("类的名称：" + mClass.getName());

//2.1 获取所有 public 访问权限的方法
//包括自己声明和从父类继承的
Method[] mMethods = mClass.getMethods();

```
  
### 访问或操作类的私有变量和方法  
  
1.  访问私有方法  
  
```java
/**
 * 访问对象的私有方法
 * 为简洁代码，在方法上抛出总的异常，实际开发别这样
 */
private static void getPrivateMethod() throws Exception{
    //1. 获取 Class 类实例
    TestClass testClass = new TestClass();
    Class mClass = testClass.getClass();
    
    //2. 获取私有方法
    //第一个参数为要获取的私有方法的名称
    //第二个为要获取方法的参数的类型，参数为 Class...，没有参数就是null
    //方法参数也可这么写 ：new Class[]{String.class , int.class}
    Method privateMethod =
            mClass.getDeclaredMethod("privateMethod", String.class, int.class);
            
    //3. 开始操作方法
    if (privateMethod != null) {
        //获取私有方法的访问权
        //只是获取访问权，并不是修改实际权限
        privateMethod.setAccessible(true);
        
        //使用 invoke 反射调用私有方法
        //privateMethod 是获取到的私有方法
        //testClass 要操作的对象
        //后面两个参数传实参
        privateMethod.invoke(testClass, "Java Reflect ", 666);
    }
}

```  

2.  修改私有变量  
    
```java
/**
 * 修改对象私有变量的值
 * 为简洁代码，在方法上抛出总的异常
 */
private static void modifyPrivateFiled() throws Exception {
    //1. 获取 Class 类实例
    TestClass testClass = new TestClass();
    Class mClass = testClass.getClass();
    
    //2. 获取私有变量
    Field privateField = mClass.getDeclaredField("MSG");
    
    //3. 操作私有变量
    if (privateField != null) {
        //获取私有变量的访问权
        privateField.setAccessible(true);
        
        //修改私有变量，并输出以测试
        System.out.println("Before Modify：MSG = " + testClass.getMsg());
        
        //调用 set(object , value) 修改变量的值
        //privateField 是获取到的私有变量
        //testClass 要操作的对象
        //"Modified" 为要修改成的值
        privateField.set(testClass, "Modified");
        System.out.println("After Modify：MSG = " + testClass.getMsg());
    }
}

```
## 12 JavaIO流
按照数据流的方向可以分为：输入流和输出流。按照处理数据单位不同可以分为：字节流和字符流。  
- 字节流：操作的基本单元为**字节**，一次读入或读出是8位二进制。
- 字符流：操作的基本单位为**Unicode码元**，一次读入或读出是16位二进制  
Jdk提供的流继承了四大类：**InputStream(字节输入流)，OutputStream（字节输出流），Reader（字符输入流），Writer（字符输出流）**。一般使用**FileInputStream（字节输入流），FileOutputStream（字节输出流），FileReader（字符输入流），FileWriter（字符输出流）**进行操作
### 12.1 字节流与字符流有什么区别   
字节流的操作不会经过缓冲区（内存）而是直接操作文本本身的，而字符流的操作会先经过缓冲区（内存）然后通过缓冲区再操作文件  
- 字节流操作的基本单元为字节；字符流操作的基本单元为Unicode码元  
- 字节流默认不使用缓冲区；字符流使用缓冲区。  
- 字节流通常用于处理二进制数据，实际上它可以处理任意类型的数据，但它不支持直接写入或读取Unicode码元；字符流通常处理文本数据，它支持写入及读取Unicode码元。
在选择字节流和字符流时，要根据实际情况选择不同的流，当需要频繁的**IO操作字符串**时应当使用字符流，操作磁盘文件，比如说图片的时候，我们都是使用字节流的。
## 缓存淘汰机制
最常见的就是FIFO、FRU、LFU：
- FIFO（First In First out）：先见先出，淘汰最先近来的页面，新进来的页面最迟被淘汰，完全符合队列。
  按照算法先进先出的原理淘汰数据，实现步骤如下：
  - 新访问的数据插入FIFO队列尾部，数据在FIFO队列中顺序移动；
  - 淘汰FIFO队列头部的数据
- LRU（Least recently used）:最近最少使用，淘汰最近不使用的页面
  核心思想为：如果数据最近被访问过，那么将来被访问的几率也更高。实现步骤原理如下：
  - 新数据插入到链表头部
  - 每当缓存命中（即缓存数据被访问），则将数据移到链表头部；
  - 当链表满的时候，将链表尾部的数据丢弃。
- LFU（Least frequently used）: 最近使用次数最少， 淘汰使用次数最少的页面
  如果数据过去被访问多次，那么将来被访问的频率也更高，LFU的每个数据块都有一个引用计数，所有数据块按照引用计数排序，具有相同引用计数的数据块则按照时间排序。实现步骤如下：
  - 新加入数据插入到队列尾部（因为引用计数为1）；
  - 队列中的数据被访问后，引用计数增加，队列重新排序；
  - 当需要淘汰数据时，将已经排序的列表最后的数据块删除。
