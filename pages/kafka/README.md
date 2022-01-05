# kafka
<img width="160" height="190" src="/pages/kafka/img.png" alt="ddd"/>

> 主要参考《深入理解Kafka核心设计与实战原理》、网络课程、kafka源码和本地调试
> 书中源码地址 https://github.com/hiddenzzh/kafka_book_demo
## 1. 思维导图
## 2. 涉及的名词
- Producer：生产者，负责创建消息
  - 三种发送模
    - fire-and-forget：发后即忘
    - Sync：同步
    - Async：异步
  - 发送过程经过组件
    - 拦截器
    - 序列化器
    - 分区器
- Consumer：消费者，消息接收方，使用pull模式从broker拉取消息，并保存offset，负责业务处理
  - Consumer Group：消费者组（逻辑上的概念），每个消费者都有一个对应的消费者组，每个分区只能被一个消费者组中的一个消费者所消费
- Broker：服务代理节点。可以看作是Kafka实例
- Topic：主题 消息以主题为单位进行归类，生产的每条消息都需要指定Topic。逻辑上的概念，一个主题可以跨多个broker
- Partition：分区 一个分区只属于一个主题，物理上可以看作一个可以追加日志（Log）的文件
- offset：消息在追加到分区日志文件时都会分配一个特定的偏移量。是消息在分区中的唯一标识. 不能跨分区，可以保证主题的分区有序
- Replica：副本 同一分区不同副本保存的是相同的消息
  - leader：负责处理读写请求
  - follower：只负责与leader副本同步，与leader处于不同的broker，leader失效时从follower中重新选举leader分区副本
- AR（Assigned Replicas）：分区中所有的副本 AR = ISR + OSR；leader副本负责维护ISR
- ISR（In Sync Replicas）：所有与leader副本保持一定程度同步的副本（包括leader副本）
- OSR（Out Sync Replicas）：与leader副本滞后过多的副本
- HW（High Water）：高水位，一个特殊的offset，消费者只能读取到这个offset之前的消息
- LEO（Log End Offset）：日志文件中下一条待写入消息的offset，ISR副本都会维护LEO，min（offset）为HW
- Log：一个分区对应一个Log（对应一个文件夹）
- LogSegment：为了防止Log过大而引入（对应一个日志文件和两个索引文件，以及可能的事务文件）
  - 日志文件：.log后缀
  - 偏移量索引文件：.index后缀
  - 时间戳索引：.timeindex后缀
- 控制器：集群中会有一个或多个Broker，其中一个Broker会被选举为控制器。负责管理集群中所有分区和副本状态。使用单线程基于队列的模型实现
  - 分区Leader选举：按照AR集合顺序查找第一个存活且在ISR集合中的副本

## 3. 重要知识点
Consumer：
- 消费支持正则表达式
- 位移提交：旧版存储在Zookeeper中，新版存储在内部主题_consumer_offsets
- 再均衡：分区所属权从一个消费者转移到另一个消费者的行为。在此期间消费者无法读到消息

- 命令：
- 主题管理
- 分区管理
- KafkaAdminClient

- 日志存储
- 日志删除
  - 基于时间
  - 基于日志大小
  - 基于日志起始偏移量
- 日志压缩
- 磁盘存储
  - 采用文件追加的方式写入消息，即顺序写盘操作
  - 页缓存：操作系统实现的一种主要的磁盘缓存，以减少对磁盘的I/O操作
  - 零拷贝：数据直接从磁盘文件复制到网卡设备中，而不需经过应用程序之手，依赖于 Linux下的mmap(Memory Map),sendfile，Java下是FileChannal.transferTo()， 可以参考[多路复用](https://zhuanlan.zhihu.com/p/258513662)
    - DMA（Direct Memory Access）：将文件内容复制到内核模式下的Read Buffer

服务端
- 时间轮：[时间轮解析](https://www.cnblogs.com/zhongwencool/p/timing_wheel.html)
## 4. 关联其他知识点

## 5. 源码阅读
生产者
重点类
RecordAccumulator

<img src="/pages/kafka/04_KafkaProducer源码分析%20(10).jpg" alt="" >

## 6. 常用配置参数
