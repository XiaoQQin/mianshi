<!-- TOC -->
[1. Spark连接kafka的方式](#1-Spark连接kafka的方式)  
[2. kafka如何实现数据的可靠性](#2-kafka如何实现数据的可靠性)  
[3. kafka维持数据的一致性](#3-kafka维持数据的一致性)  
[4. kafka消费者组](#4-kafka消费者组)  
[5. kafka如何保证高吞吐](#5-kafka如何保证高吞吐)  
[6. kafka如果一个broker宕机会发生什么](#6-kafka如果一个broker宕机会发生什么)
<!-- /TOC -->

## 1. Spark连接kafka的方式  
  
#### Receiver方式  
从Kafka通过Receiver接收的数据存储在Spark Executor的内存中，然后由Spark Streaming启动的job来处理数据。这种方式可能会因为底层的失败而丢失数据。
如果要启用高可靠机制，确保零数据丢失，要启用Spark Streaming的**预写日志机制(Write Ahead Log)**。
该机制会同步地将接收到的Kafka数据保存到分布式文件系统（比如HDFS）上的预写日志中，以便底层节点在发生故障时也可以使用预写日志中的数据进行恢复。  
   
#### Direct方式  
Direct方式没有receiver这一层，其会周期性的获取Kafka中每个topic的每个partition中的最新offsets，之后根据设定的maxRatePerPartition来处理每个batch  
  
  
#### Direct方式的优点  
    
- **简化并行读取**: spark会创建和kafka partition一样多的RDD partition,并行从kafka中读取数据，两者有一一对应的关系。  
  
- **高性能**: 保证数据的零丢失，在基于receiver方式中要开启WAL(日志预写)机制，这种方式效率低下，数据实际复制两份。而基于direct方式不需要开启WAL机制。  
  
- **执行一次**：基于receiver的方式是使用zookeeper保存消费国的offset的，这种方式配合WAL机制可以保证数据零丢失，但是无法保证数据被处理一次且仅一次。spark和
zookeeper之间是不同步的。  
    基于direct方式的，spark streaming自己负责追踪消费的offset.
      
## 2. kafka如何实现数据的可靠性  
  
    
- **分区多副本**:通过分区副本引入数据冗余，一个分区的副本有一个leader和多个follower，所有的读写操作都是通过leader进行的。  
  
- **消息确认机制**: 即生产者向集群上发送消息时，可以设置acks参数。acks参数为0时表示消息发送即可；为1时，leader接受到消息并写入到分区数据中；为all，需要所有的
的副本都同步完。  
    
  
## 3. kafka维持数据的一致性  
  
维护一个**High water mask**值，这个值取决于**ISR(副本同步队列)**列表里偏移量最小的分区，类似于木桶原理，只有High water mask以上的消息才支持被消费者消费。
  
## 4. kafka消费者组 
  
**每个消费者都属于消费者组**。一个消费者组内的所有消费者一起来消费订阅主题的所有分区，每个分区只能由同一消费者组内的一个消费者进行消费。  
  
决定组内哪个消费者消费哪个分区有默认的两种分区分配策略：
  
-  range: 分区数除以总的消费者数，消费者按照名称进行排序
-  roundRobin:即轮询  
    
## 5. kafka如何保证高吞吐  
  
1) 顺序读写：生产者写入数据时，kafka持久化到磁盘，消费者通过控制偏移来进行读取数据  
2) 零拷贝  
3) 分区：kafka的topic内容可以分为多个partition存在，每个partition又分为多个片段segment。所以每次操作都是针对一部分数据进行操作。  
4) 批量发送：kafka允许批量发送消息，生产者发送消息时，可以将消息缓存到本地，等到固定条件时再发送到kafka  
5) 数据压缩
    
      
## 6. kafka如果一个broker宕机会发生什么
当kafka的一个broker,是不会将宕机掉的broker上的数据复制到其他broker上。通常情况下，如果broker没有彻底坏，请快速启动，复制数据是很浪费的。在更罕见的情况下，可以重新启动一个broker,并且将它的broker id设置为宕机broker的id,那么新的broker将会自动复制缺失的数据。  
