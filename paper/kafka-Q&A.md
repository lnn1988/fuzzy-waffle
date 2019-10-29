# kafka 问题汇总

### Q1：什么是Apache Kafka?

Apache Kafka是一个发布 - 订阅开源消息代理应用程序。这个消息传递应用程序是用“scala”编码的。基本上，这个项目是由Apache软件启动的。Kafka的设计模式主要基于事务日志设计。



### Q1：kafka 的几个概念

* 主题（topic）：Kafka主题是一堆或一组消息。
* 生产者（producer）：在Kafka，生产者发布通信以及向Kafka主题发布消息。
* 消费者（consumer）：Kafka消费者订阅了一个主题，并且还从主题中读取和处理消息。
* 经纪人（broker）：在管理主题中的消息存储时，我们使用Kafka Brokers。

### Q2： Kafka 的设计时什么样的呢？

Kafka 将消息以 topic 为单位进行归纳

将向 Kafka topic 发布消息的程序成为 producers。

将预订 topics 并消费消息的程序成为 consumer。

Kafka 以集群的方式运行，可以由一个或多个服务组成，每个服务叫做一个 broker。

producers 通过网络将消息发送到 Kafka 集群，集群向消费者提供消息。

### Q3：偏移（offset）的作用

给分区中的消息提供了一个顺序ID号，我们称之为偏移量。因此，为了唯一地识别分区中的每条消息，我们使用这些偏移量。

### Q4：消费者组（Consumer Group）？

每个Kafka消费群体都由一个或多个共同消费一组订阅主题的消费者组成。Group 中的所有 Consumer 共同消费同一 Topic 下的所有数据。

### Q5：Zookeeper 扮演的角色？

Zookeerer 它为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册。

在 Kafka 中，zookeeper 负责存储元数据信息：包括consumerGroup/consumer、broker、Topic等。

kafka使用zookeeper来实现动态的集群扩展，不需要更改客户端（producer和consumer）的配置。broker会在zookeeper注册并保持相关的元数据（topic，partition信息等）更新。而客户端会在zookeeper上注册相关的watcher。一旦zookeeper发生变化，客户端能及时感知并作出相应调整。这样就保证了添加或去除broker时，各broker间仍能自动实现负载均衡。

zookeeper 在kafka中还用来选举controller 和 检测broker是否存活等等。

### Q6：Kafka 的主要优点

* 高吞吐量
* 低延迟
* 容错：Kafka能够抵抗集群中的节点/机器故障。
* 易扩展：不需要通过停机就可以添加额外的节点。

###  Q7：数据传输的事物定义有哪三种？

数据传输的事务定义通常有以下三种级别：

1. 最多一次: 消息不会被重复发送，最多被传输一次，但也有可能一次不传输

2. 最少一次: 消息不会被漏发送，最少被传输一次，但也有可能被重复传输.

3. 精确的一次（Exactly once）: 不会漏传输也不会重复传输,每个消息都传输被一次而且仅仅被传输一次，这是大家所期望的

### Q8：Kafka 存储在硬盘上的消息格式是什么？

消息由一个固定长度的头部和可变长度的字节数组组成。头部包含了一个版本号和 CRC32校验码。

* 消息长度: 4 bytes (value: 1+4+n)
* 版本号：1
* CRC校验码：4
* 具体消息：n

### Q9：Kafka 高效文件存储设计特点：

* 把一个 topic 下的 partition 文件，分成多个小的分段文件，容易删除清理。
* 建立文件的稀疏索引index 文件，存在 memory 中，可以快速定位文件位置。

### Q10：kafka 的生产者 ack 机制

request.required.acks 有三个值 0 1 -1

* 0：不会等待 ack，延迟最低，但是可能丢数据
* 1：等待leader 确认
* -1：等待所有的 follow 确认消息

### Q11：为什么要使用 kafka，为什么要使用消息队列

* 缓冲、削峰：避免上游数据发送太快，下游可能扛不住，kafka 起一个缓冲的作用，将数据暂存在 kafka 中，下游服务仍按照自己的速度慢慢消费。
* 冗余：可以采用一对多的方式，一个生产者发布消息，可以被多个订阅topic的服务消费到，供多个毫无关联的业务使用。
* 异步通信：很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。



### Q12： ISR

分布式系统保证数据可靠性的一个常用手段就是增加副本个数，ISR就是建立在这个手段上。

ISR全称”In-Sync Replicas”，是保证HA和一致性的重要机制。副本数对Kafka的吞吐率是有一定的影响，但极大的增强了可用性。一般2-3个为宜。

副本有两个要素，一个是数量要够多，一个是不要落在同一个实例上。ISR是针对与Partition的，每个分区都有一个同步列表。N个replicas中，其中一个replica为leader，其他都为follower, **leader处理partition的所有读写请求**，其他的都是备份。与此同时，follower会被动定期地去复制leader上的数据。

如果一个flower比一个leader落后太多，或者超过一定时间未发起数据复制请求，则leader将其重ISR中移除。

当ISR中所有Replica都向Leader发送ACK时，leader才commit。

Kafka的ISR的管理最终都会反馈到Zookeeper节点上。具体位置为：/brokers/topics/[topic]/partitions/[partition]/state。当Leader节点失效，也会依赖Zk进行新的Leader选举。Offset转移到Kafka内部的Topic以后，KAFKA对ZK的依赖就越来越小了。