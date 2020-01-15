# MapReduce



## 简介

* MapReduce 将复杂的、运行于大规模集群上的并行计算过程高度抽象为两个函数：**Map**和 **Reduce**
* **分而治之**，存在分布式文件系统的文件，会被分成多个分片，这些分片可以被多个 Map 任务并行处理
* **计算向数据靠拢**，因为文件的移动需要耗费大量的网络传输开销

## MapReduce 体系结构

* Client：用户提交的 MR 程序通过 client 提交到 JobTracker
* JobTracker
    * 监控资源和作业调度
    * 监测其他 TaskTracker 和 job 的状态，一旦某个节点失败，将相应的任务转移到其他节点
    * 跟踪任务的执行和资源情况，汇报给 TaskScheduler（资源调度器），调度器还在资源出现空闲时，选择合适的 job 去使用这些资源
* TaskTracker
    * 周期性通过 heartbeat 将自己的情况发给 JobTracker，同时接受 JobTracker 发过来的命令执行操作
    * 使用 slot（槽）等量划分自己节点上的资源（CPU，内存）。只有空闲 slot 时才会被分配 task
* Task：分为 Map Task、Reduce Task

![image-20191113174101757](https://tva1.sinaimg.cn/large/006y8mN6ly1g8wjvj725kj30gw09st9j.jpg)



## MapReduce 工作流程

在多个分片存储的多个机器上启动多个 Map Task，每个 Map 任务得到一系列 (key, value) 的键值对，之后进行 Shuffle 将得到的键值对发到 Reduce 任务所在机器，由 Reduce 任务进行合并。

HDFS 以固定的大小 block 座位存储的基本单位，但是 Map 函数的个数由 split 个数决定。RecordReader 会读取特定的 split，传递键值对给 Map 函数。

split 只是逻辑分片并非物理分片，只需要记录文件起始位置、结束位置等元数据，RecordReader 从 block 读取特定的 split。split 是由用户自己控制的，理想的 split 大小是和 block 大小一致。



## Shuffle 过程



![截屏2019-11-13下午6.01.52](https://tva1.sinaimg.cn/large/006y8mN6ly1g8wkl3on0xj30gt07cwfk.jpg)

Map 任务运行用户处理逻辑，生成键值对，写入缓存，缓存满了之后发生溢写，将文件写到磁盘文件。然后将磁盘文件进行排序、分区等归并。最终得到分区的，经过排序的键值对文件。之后 JobTracker 通知各个 reduce 任务来取走属于该分区的文件。

### Map Shuffle

* 读取文件
* 将键值对写入缓存（默认 100MB）
* 溢写（默认比例 0.8）
* 文件归并、分区（默认哈希分区，有几个 Reduce 就分几个区）
* JobTracker 通知Reduce 任务来领取数据

### Reduce Shuffle

* Reduce 接收到通知后，去 Map 机器上属于自己的分区的数据拉回数据。
* 领取数据写到缓存，进行归并、合并
* 如果数据太大就进行溢写到硬盘
* 生成的整理好的文件交给 Reduce 任务进行处理



