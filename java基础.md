#  关于String  
   String类是我们平常项目中使用频率非常高的一种对象类型，jvm为了提升性能和减少内存开销，避免字符的重复创建，其维护了一块特殊的内存空间，即字符串池，当需要使用字符串时，先去字符串池中查看该字符串是否已经存在，如果存在，则可以直接使用，如果不存在，初始化，并将该字符串放入字符创常量池中。  
   String类是final类，这意味着不允许任何人定义String的子类。使用 new String 的话可能至少会创建一个对象,即使是里面的值相同，但是 使用==比较的是内存地址，会输出false.  
   
### String、StringBuffer和 StringBuilder
   String是字符串常量,StringBuffer和StringBuilder 是字符串变量,使用 + 的话会创建一个新的字符串对象。 
   String长度不可变，StringBuffer和StringBuilder都是可变的。
   StringBuffer是线程安全的，StringBuilder是非线程安全的。这是因为Stringbuffer中方法大都采用了synchronized的关键字修饰。
#  关于==和equals
   ==比较的是变量的内存地址，用来判断两个变量是否为同一对象，有相同的内存地址。equals一般比较对象中的内容是否相同，可以通俗的理解我们是否赋值的相同。  
   默认情况下，当一个类从Object类继承而来，如果没有重写equals，那么equals和“==”的作用是相同的，都是比较内存地址。因此我们可以按照我们自己的需求，来重写equals。
   
   ### 重写equals为什么要重写hashCode()方法？
   重写equals我们一般是希望判断对象里面的内容是否相同，两个对象如果equals返回true，我们认为即对象相等，2个对象的equals()方法返回true的话，其hashCode()必须返回相同的值。否则对于HashSet, HashMap, HashTable等基于hash值的类就会出现问题。
#  关于hashMap
   ### hashMap的结构
   从存储结构来讲，hashMap内部使用Node实现，Node是HashMap的一个内部类，实现了Map.Entry接口，本质是就是一个映射(键值对)。一个Node对象有四个属性：hash值，key，value，还有链表的下一个Node.  
   
   HashMap结构是数组+链表+红黑树（JDK1.8增加了红黑树部分),也就是解决hash冲突中的链地址法。简单来说就是数组加链表的结构，当key的到对应的hash值后，就得到数组的下表(哈希桶下标)，然后把数据放到对应数组的链表中。
#  关于ArrayList和LinkedList
#  java抽象类和接口
#  jdk 1.8相对于1.7有什么变化
