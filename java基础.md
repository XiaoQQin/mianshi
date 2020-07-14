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
   ```
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
   HashMap是线程不安全的，它的不安全就体现在resize的时候，多线程的情况下，可能会形成环形链表，导致下一次读取的时候可能会出现死循环。
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
1)  抽象类中成员变量可以为各种类型，接口中成员变量会被隐式的指定为 public static final 变量 
```
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
```
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
```
List<String> demoList = Arrays.asList("小明", "Zing", "阿三", "小红", "赵日天");
demoList.forEach(System.out::println);
```
3.  **默认方法**:  默认方法就是一个在接口里面有了一个实现的方法,使用关键字default,还可以有静态变量及方法。  
```
public interface testInterface {
    int a=1;
    default void testDefault(){
        //直接在implements的类里直接调用，testDefault(),不需要testInterface.
        System.out.println(a);
    }
}
```
4.  **函数接口**： 是一个接口中**有且仅有一个抽象方法**，但是可以有多个非抽象方法的接口。可以转化为lambda表达式.可以使用**@FunctionalInterface**标记。  
```
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

## 10. java泛型   
泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型。
泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法。  
  
### 泛型类
泛型类定义时只需要在类名后面加上类型参数即可.  

```
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
  
```
public <E> void PrinterInfo(E e) {
    	System.out.println(e);
}
```
泛型方法里面的类型参数E和泛型类里面的类型参数(如果类是一个泛型类)是不一样的类型，既然如果泛型类声明为 ```class testClass<E>```，并且声明的该类的一个对象```testClass<String> tc=new testClass<>();```,声明对象里的为String类型，那么PrinterInfo接受的参数可以为其他类型。
  
### 泛型接口
Java泛型接口的定义和Java泛型类基本相同,在声明类的时候，需将泛型的声明也一起加到类中。
```
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
```
//1.获取并输出类的名称
Class mClass = SonClass.class;
System.out.println("类的名称：" + mClass.getName());

//2.1 获取所有 public 访问权限的变量
// 包括本类声明的和从父类继承的
Field[] fields = mClass.getFields();
```  
  
2. 获取类的所有方法信息  
```
//1.获取并输出类的名称
Class mClass = SonClass.class;
System.out.println("类的名称：" + mClass.getName());

//2.1 获取所有 public 访问权限的方法
//包括自己声明和从父类继承的
Method[] mMethods = mClass.getMethods();

```
  
### 访问或操作类的私有变量和方法  
  
1.  访问私有方法  
  
```
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
    
```
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
