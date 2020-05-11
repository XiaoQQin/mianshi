<!-- TOC -->
[1. Spark连接kafka的方式](#1-Spark连接kafka的方式)  

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
