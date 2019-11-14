# 流计算

### 流数据特点

* 持续快速到达，大小可能是无穷无尽的
* 来源多，格式复杂
* 数据量大，但是不关注存储
* 注重整体价值，不过分关注个别数据
* 顺序可能是不正确的



### 流计算概念

* 高性能
* 海量式
* 实时性
* 分布式
* 易用、可靠



## Storm

Storm 可以方便地与数据库系统进行整合，从而开发出强大的实时计算系统。

主要术语有：

* Streams：流数据
* Tuple：Storm 将 Streams 描述为一个无界的 Tuple 序列，每个 Tuple 是一堆值的列表
* Spouts：每个 Stream 的源头抽象为一个 Spout，Spout 会从外部读取数据，封装成一个 Tuple 发送到 Stream，包括一个 nextTuple 函数，不停调用该函数，产生数据流
* Bolts：Bolt 是 Streams 的转换过程，既可以处理 Tuple，也可以将处理后的 Tuple 作为新的 Stream 发送给下一个 Bolt。Bolt 可以执行过滤、函数、join 等操作，接口有一个 execute 方法，收到消息会调用此函数。
* Topology：Storm 将 Spout、Bolt 组成的网络抽象为Topology（拓扑），里面的每个组件都是并行执行的。
* Groupings：用于告知 Topology 如何在两个组件中进行 Tuple 的传送

### Streams Grouping

* Shuffle Grouping：随机分组随机分发 Stream 中的 Tuple，保证每个 Bolt 的 Task 收到的 Tuple 数量大概一致
* FieldsGrouping：按照字段分组，保证相同字段的 Tuple 分配到一个 Task
* AllGrouping：广播发送，每个 Task 都会收到所有的 Tuple
* GlobalGrouping：全局分组，所有 Tuple 会发送到同一个 Task
* NonGrouping：不分组，类似于 ShuffleGrouping，当前 Task 的执行会和他的被订阅者在同一个线程中执行
* DirectGrouping：直接分组，直接指定由某个 Task 来执行 Tuple 的处理

### Storm 框架设计

Storm 运行 Topoplgy，类似于 Hadoop 的 MapReduce。但是两者的任务不同，Hadoop 的 MR 作业会完成计算并结束运行，而 Topology 持续处理消息知道人为终止。

|          | **Hadoop**  | **Storm**  |
| :------: | :---------: | :--------: |
| 应用名称 |     Job     |  Topology  |
| 系统角色 | JobTracker  |   Nimbus   |
|          | TaskTracker | Supervisor |
| 组件接口 | Map/Reduce  | Spout/Bolt |

节点：

* Master：运行名为 Nimbus 的后台程序，负责在集群内分发代码，为 worker 分配任务。
* Worker：运行 Supervisor 的后台程序，监听分配给他的工作，一个 Worker 节点同时运行多个 Worker 进程。
* Zookeeper：Storm 使用 ZK 做协调，负责 Nimbus 和 Supervisor 的协调工作。

![截屏2019-11-14下午3.18.09](https://tva1.sinaimg.cn/large/006y8mN6ly1g8xlktkl6sj30bv09674s.jpg)

1. 每台 Supervisor 上运行多个 worker，每个 worker 对 Topology 中的每个组件运行一个或者多个 executor
2. 每个 executor 是产生于 worker 内部的线程，执行同一个组件的多个 Task
3. 实际的数据处理有 Task 来完成


