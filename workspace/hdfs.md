# 分布式文件系统 HDFS

***Hadoop Distributed File System***

## HDFS 特性

HDFS 默认一个块 64Mb，一个大文件会被分为多个块。

### 主要实现目标

* 兼容廉价硬件
* 流数据读写
* 大数据集
* 简单的文件模型

### 局限性

* 不支持低延迟数据访问
* 无法高效存储小文件
* 不适合任意修改文件

## HDFS 主要组件

1. NameNode
    * 存储元数据（block、datanode 之间的关系）
    * 元数据存在内存

2. DataNode
    * 存储文件内容的实际数据
    * 内容写在磁盘

### NameNode

* FsImage：维护文件系统树文件的元数据
* EditLog：操作日志文件，记录针对文件的创建、删除、重命名等操作

在名称节点启动的时候，会将 FsImage 的内容加载到内存中，再执行 EditLog 的各项操作。在内存中成功建立文件系统元数据之后，创建一个新的 FsImage 和一个空的 EditLog，以后再有针对文件的创建、删除等操作，就记录在新创建的 EditLog 之中。

**Why：**FsImage 一般会比较大，操作日志直接记录在 FsImage 会导致系统运行缓慢，所以记录在 EditLog。

**EditLog 文件变大怎么办：**使用 SecondaryNameNode 进行备份，第二名称节点定时从主节点拉取 FsImage 和 EditLog，此时名称节点之后的操作写在新的 EditLog.new 中，在第二名称节点将 FsImage 和 EditLog 合并之后，再将新生成的 FsImage 发回给主节点，进行替换即可。

### DataNode

* •数据节点是分布式文件系统HDFS的工作节点，负责数据的存储和读取，会根据客户端或者是名称节点的调度来进行数据的存储和检索，并且向名称节点定期发送自己所存储的块的列表
* 每个数据节点中的数据会被保存在各自节点的本地Linux文件系统中

## HDFS 存储原理

### 冗余数据保存

* 每个数据块都会冗余保存（默认3）
    * 加快数据传输（不同应用可以从不同机器取数据、并行记性）
    * 容易检查数据错误（一旦某个数据块出现错误，会再备份至冗余保存数目）
    * 保证数据的可靠性

### 数据存取策略

1. 数据存放

    * 放在上传文件的机器数据节点；如果集群外提交，随机挑选一台磁盘不太满，CPU 不太忙的节点
    * 放在第一个机架不同的机器上
    * 放在第一个机架相同的机器上

2. 数据读取

    就近选择

### 数据错误、回复

1. NameNode 出现错误

    HDFS设置了备份机制，把这些核心文件同步复制到备份服务器SecondaryNameNode上。当名称节点出错时，就可以根据备份服务器SecondaryNameNode中的FsImage和Editlog数据进行恢复。

2. DataNode
    * 每个数据节点会定期向 NameNode 发送心跳信息
    * NameNode 检测不到 DataNode 时，标记其为“宕机”，将其他机器上与此 DataNode 相同的数据再启动冗余复制，进行备份
    *  DataNode 读取 block 的时候，它会计算 checksum。如果计算后的 checksum，与 block 创建时值不一样，说明该 block 已经损坏。  Client 读取其它 DataNode上的 block 并且标记该块损坏。

## HDFS 数据读写过程

### 读数据

1. 打开分布式文件
2. 寻址请求，从 DataNode 获得 DataNode 的地址
3. 连接 DataNode，获取数据
4. 读取另外的 DataNode 直到读取完成
5. 关闭连接




