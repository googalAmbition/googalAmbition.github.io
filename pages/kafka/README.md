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
- Consumer：消费者，消息接收方，使用pull模式从broker拉取消息，并保存offset，负责业务处理
- Broker：服务代理节点。可以看作是Kafka实例
- Topic：主题 消息以主题为单位进行归类，生产的每条消息都需要指定Topic。逻辑上的概念，一个主题可以跨多个broker
- Partition：分区 一个分区只属于一个主题，物理上可以看作一个可以追加日志的文件
- offset：消息在追加到分区日志文件时都会分配一个特定的偏移量。是消息在分区中的唯一标识. 不能跨分区，可以保证主题的分区有序
- Replica：副本 同一分区不同副本保存的是相同的消息
  - leader：负责处理读写请求
  - follower：只负责与leader副本同步，与leader处于不同的broker，leader失效时从follower中重新选举leader分区副本
- AR（Assigned Replicas）：分区中所有的副本 AR = ISR + OSR；leader副本负责维护ISR
- ISR（In Sync Replicas）：所有与leader副本保持一定程度同步的副本（包括leader副本）
- OSR（Out Sync Replicas）：与leader副本滞后过多的副本
- HW（High Water）：高水位，一个特殊的offset，消费者只能读取到这个offset之前的消息
- LEO（Log End Offset）：日志文件中下一条待写入消息的offset，ISR副本都会维护LEO，min（offset）为HW
- 
## 3. 分章节重点

## 4. 关联其他知识点

## 5. 源码验证

<img src="/pages/kafka/04_KafkaProducer源码分析%20(10).jpg" alt="" >
