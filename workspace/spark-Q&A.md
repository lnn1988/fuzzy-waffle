# Spark 常见问题

## spark 特点
1. 速度快：基于内存
2. 易使用：提供 java、py、scala 的 API
3. 通用性：提供统一的方案，包括 SparkSQL、Spark Streaming、MMLib 和 GraphX
4. 可融合：可以继承 hdfs 以及 YARN

## spark 和 mapreduce 的关系
1. hadoop 的一个作业称为一个job，包括 map task 和 reduce task，每个 task 都在自己的进程中执行。当 task 结束，进程也会结束。
2. spark 提交的程序称为一个 application，对应一个 spark context，存在多个 job，每次触发 action 操作就会产生一个job。每个 job 存在多个stage，stage 是 shuffle 过程中 DAGSchaduler 通过各个 RDD 的依赖关系来划分 job 的。每个 stage 有多个 task，组成 taskSet 由 TaskSchaduler 分发到各个 executor。executor 的执行周期和 app 一样，即使没有job 运行也会存在，所以 task 可以快速启动读取内存进行计算。
3. hadoop 的 job 只有 map 和 reduce，会不断落盘和读取，产生大量 IO 开销。而 Spark 的迭代计算都是在内存进行，通过 DAG 可以实现很好的容错性。所谓容错性，就是处理错误的情况，由于 DAG 存储了 RDD 之间的依赖关系，假设某个 RDD 数据丢失，只需要从其父 RDD 再次进行转换操作就可以得到。

## RDD 的宽依赖和窄依赖
RDD和它依赖的parent RDD(s)的关系有两种不同的类型，即窄依赖（narrow dependency）和宽依赖（wide dependency）。
1. 窄依赖指的是每一个parent RDD的Partition最多被子RDD的一个Partition使用
2. 宽依赖指的是多个子RDD的Partition会依赖同一个parent RDD的Partition

## spark 部署模式
* 本地模式
    * local：只启动一个 executor
    * local[k]：启动 K 个 exeecutor
* standalone 模式
    * 分布式部署集群，自带完整服务
* Spark on Yarn 
    * 资源调度交给 Yarn 来做

## Spark 有哪些组件
* Spark Core：spark 内核，其他组件的基础，包括 DAG、RDD 等
* Spark SQL：可以连接 mysql、hive 等关系型数据库，处理关系表
* Spark Streaming：流计算、将数据流分成一个个小段。可以对接多种数据源，比如文件、socket 套接字、kafka 
* Spark MLLib：机器学习

## Spark 的各个节点
* client：用户提交程序入口
* master：管理集群节点
* worker：计算节点
* Driver：