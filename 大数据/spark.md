<!-- TOC -->
- [1. RDD的基本原理](#1-RDD的基本原理)  
- [2. spark的部署方式](#2-spark的部署方式)  
- [3. 创建RDD的方式](#3-创建RDD的方式)  
- [4. RDD的依赖关系](#4-RDD的依赖关系)  
- [5. RDD的持久化](#5-RDD的持久化)  
- [6. Spark的Checkpoint机制](#6-Spark的Checkpoint机制)  
- [7. job和Stage和Task](#7-job和Stage和Task)  
- [8. Spark的运行流程](#8-Spark的运行流程)  
- [9. 数据倾斜](#9-数据倾斜)  
    - [9.1 什么是数据倾斜](#91-什么是数据倾斜)  
    - [9.2 数据倾斜的解决方案](#92-数据倾斜的解决方案)
<!-- /TOC -->
## 1. RDD的基本原理  
RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据抽象，它代表一个不可变、可分区、里面的元素可并行计算的集合。  
RDD具有数据流模型的特点：自动容错、位置感知性调度和可伸缩性。它并不是真正存储数据的结构，而是一种抽象的概念,我们可以认为 RDD 就是一个代理，
我们操作这个代理就像操作本地集合一样，不需去关心任务调度、容错等问题。  
  
### RDD的属性
RDD具有属性，它的属性用来描述当前数据集的状态，我们可以简单的理解为RDD里的包括什么  
  
- **一组分片(Partition)**: 即数据集的基本组成单位。对于RDD来说，每个分片都会被一个计算任务处理，并决定并行计算的粒度。
用户可以在创建RDD 的时候指定RDD的分片个数，如果没有指定，那么就会采用默认值。默认值就是程序所分配到的CPU Cores 的数目。  
  
- **一个计算每个分区的函数**：。Spark中RDD的计算是以分片为单位的，每个RDD都会实现compute函数以达到这个目的。
compute函数会对迭代器进行复合，不需要保存每次计算的结果  
  

- **一个Partitioner**：即RDD的分片函数。当前Spark中实现了两种类型的分片函数，一个是基于哈希的HashPartitioner，另外一个是基于范围的RangePartitioner。
只有对于于key-value的RDD，才会有Partitioner，非key-value的RDD的Parititioner的值是None。
Partitioner函数不但决定了RDD本身的分片数量，也决定了parent RDD Shuffle输出时的分片数量。  
  
- **RDD之间的依赖关系**：RDD的每次转换都会生成一个新的RDD，所以RDD之间就会形成类似于流水线一样的前后依赖关系。
在部分分区数据丢失时，Spark可以通过这个依赖关系重新计算丢失的分区数据，而不是对RDD的所有分区进行重新计算  
  
- **一个列表，存储存取每个Partition的优先位置（preferred location）**。对于一个HDFS文件来说，这个列表保存的就是每个Partition所在的块的位置。
按照“移动数据不如移动计算”的理念，Spark在进行任务调度的时候，会尽可能地将计算任务分配到其所要处理数据块的存储位置。
  
## 2. spark的部署方式  
  
1) local本地模式 :常用于本地开发测试，如在eclipse，idea中写程序测试等,本地还分为local单线程和local-cluster多线程。  

2) standalone模式: 集群模式,Spark自带的一个资源调度框架，支持完全分布式。  
3) spark on yarn ：集群模式，运行在yarn资源管理器框架之上，由yarn负责资源管理，Spark负责任务调度和计算 
4) spark on mesos： 运行在mesos资源管理器框架之上，由mesos负责资源管理，Spark负责任务调度和计算
  
## 3. 创建RDD的方式  
  
- **由一个存在的 Scala 集合进行创建**  
  
```
var rdd = sc.parallelize(List(1,2,3,4,5,6,7,8,9))
```  
- **由外部的存储系统的数据集创建，包括本地的文件系统，还有所有 Hadoop 支持的数据集，比如 HDFS、Cassandra、Hbase**  
  
```
var rdd1 = sc.textFile("/root/words.txt")
var rdd2 = sc.textFile("hdfs:192.168.80.131:9000/words.text")
```
  
- **调用一个已经存在了的RDD 的 Transformation，会生成一个新的 RDD**
  
## 4. RDD的依赖关系  
  
1) **窄依赖**：每个父RDD的一个partition最多被子RDD的一个partition所使用，例如map(独生子女)  
2) **宽依赖**: 一个父RDD的partition会被多个子RDD的partirion所使用，例如reduceByKey(非独生子女)  
  
## 5. RDD的持久化  
Spark非常重要的一个功能特性就是可以将RDD持久化在内存中。当对RDD执行持久化操作时，每个节点都会将自己操作的RDD的partition持久化到内存中，并且在之后对该RDD的反复使用中，直接使用内存缓存的partition。RDD持久化是使用persist和cache两个函数。  
  
- **cache**:内部调用persist，持久化到内存  
- **persist**: 自由设置存储级别，默认为内存

## 6. Spark的Checkpoint机制  
Checkpoint 是 Spark 提供的一种缓存机制，checkpoint的意思就是建立检查点,类似于快照。当需要计算依赖链非常长又想避免重新计算之前的 RDD 时，可以对 RDD 做 Checkpoint 处理，检查 RDD 是否被物化或计算，并将结果持久化到磁盘或 HDFS 内。  
  
#### checkpoint与cache对比  
  
cache 缓存数据由 executor 管理，若 executor 消失，它的数据将被清除，RDD 需要重新计算；而 checkpoint 将数据保存到磁盘或 HDFS 内，job 可以从 checkpoint 点继续计算  
  
#### checkpoint与persist对比  
Spark 提供了 rdd.persist(StorageLevel.DISK_ONLY) 这样的方法，相当于 cache 到磁盘上，这样可以使 RDD 第一次被计算得到时就存储到磁盘上。  
  
 
-  persist 虽然可以将 RDD 的 partition 持久化到磁盘，但一旦作业执行结束，被 cache 到磁盘上的 RDD 会被清空  
-  checkpoint 将 RDD 持久化到 HDFS 或本地文件夹，如果不被手动 remove 掉，是一直存在的。
  
## 7. job和Stage和Task  
  
**当遇到一个action算子时，就会提交一个job**，执行前面的一系列操作，通常一个任务会有多个job，job之间是按照串行的方式执行的。  
一个job包括一个或者多个stage，stage是根据是否发生shuffle划分的。  
一个Stage由多个task组成，task的数量是stage的并行度，RDD计算时，每个分区都会生成一个Task
  
## 8. Spark的运行流程  
  
1) 构建Spark Applcation的运行环境(启动SparkContext)。SparkContext向资源管理器(可以是standalone获yarn)注册并申请运行Executor资源。  
2) 资源管理器分配资源给Executor，并启动守护进程StandaloneExecutorBackend,Execuor运行情况将随心跳发送到资源管理器上。  
3) SparkContext构建DAG图，将DAG图分解为Stage，并把Taskset(Stage)发送给Task Scheduler。Executor向SparkContext申请Task  
4) Task Scheduler将Task发送给Executor发送给Executor运行，同时SparkContext将应用程序代码发送给Executor。  
5) Task在Executor上运行，完毕后释放资源。 
  
## 9. 数据倾斜  
  
### 9.1 什么是数据倾斜  
 在进行shuffle时，必须将各个节点上相同key拉取到某一个节点上的一个task来进行，此时key对应的数据量特别大，就会发生数据倾斜。比如大部分key对应10条数据，但是个别key却对应了100万条数据，那么大部分task可能就只会分配到10条数据，然后1秒钟就运行完了；但是个别task可能分配到了100万数据，要运行一两个小时。因此，整个Spark作业的运行进度是由运行时间最长的那个task决定的。  
   
出现数据倾斜的时候，Spark作业看起来会运行得非常缓慢，甚至可能因为某个task处理的数据量过大导致内存溢出。  
  
### 9.2 数据倾斜的解决方案  
-  **过滤少数导致倾斜的key**:导致倾斜的key就少数几个，并且对结算结果影响不大的话，就可以采用filter等算子进行过滤，适用场景不多。  
-  **提高shuffle操作的并行度**: 对RDD执行shuffle算子时，给shuffle算子传入一个参数，比如reduceByKey(1000)，该参数就设置了这个shuffle算子执行时shuffle read task的数量。  
  
- **两阶段聚合（局部聚合+全局聚合）**：将原本相同的key通过附加随机前缀的方式，变成多个不同的key，就可以让原本被一个task处理的数据分散到多个task上去做局部聚合，进而解决单个task处理数据量过多的问题。接着去除掉随机前缀，再次进行全局聚合，就可以得到最终的结果。  
  
- **将reduce join转为map join**：普通的join是会走shuffle过程的，而一旦shuffle，就相当于会将相同key的数据拉取到一个shuffle read task中再进行join，此时就是reduce join。但是如果一个RDD是比较小的，则可以采用广播小RDD全量数据+map算子来实现与join同样的效果，也就是map join，此时就不会发生shuffle操作，也就不会发生数据倾斜  
  
- **采样倾斜key并分拆join操作**：对于join导致的数据倾斜，如果只是某几个key导致了倾斜，可以将少数几个key分拆成独立RDD，并附加随机前缀打散成n份去进行join，此时这几个key对应的数据就不会集中在少数几个task上，而是分散到多个task进行join了。  
  
- **使用随机前缀和扩容RDD进行join**：将原先一样的key通过附加随机前缀变成不一样的key，然后就可以将这些处理后的“不同key”分散到多个task中去处理，而不是让一个task处理大量的相同key
     
 [Spark（十）Spark之数据倾斜调优](https://www.cnblogs.com/frankdeng/p/9301695.html)
