# Spark Streaming



**静态数据>>批量计算**

**流数据>>实时计算**



### 流数据的特征：

* 持续到达
* 数据来源多、格式复杂
* 数据量大
* 注重整体价值
* 顺序不完整



### 流计算框架的需求

* 高性能
* 海量式：支持 TB、PB 规模数据
* 实时性：延迟低
* 分布式：容易扩展
* 易用、可靠



## Spark Streaming 数据源

![截屏2019-11-06下午5.10.00](https://tva1.sinaimg.cn/large/006y8mN6ly1g8ofnabi53j30j207g74r.jpg)

spark strraming 最主要的抽象是 DStream (Discretized Stream， 离散化数据流)，表示连续不断的数据流。

在内部实现，输入流按照时间分成一个个小段（通常是 1 秒），每一段数据转换为 Spark RDD。这些分段就是 DStrean。

![截屏2019-11-06下午5.12.38](https://tva1.sinaimg.cn/large/006y8mN6ly1g8ofqf0pq0j30i107vwff.jpg)

## DStream

在 Spark Streaming 中，会有一个 Receiver，作为一个长期运行的 task 跑在一个 Executor 上。每个 Receiver 会负责一个 input DStream（比如从文件读取文件流、负责套接字、读取 Kafka 一个 topic）等。

* 创建输入流对象 DStream 定义输入源
* 通过对 DStream 应用转换和动作操作处理数据
* 用 streamingContext.start() 来开始接收数据
* 用 streamingContext.awaitTermination()来等待数据处理结束
* 用 streamingContext.stop() 手动结束流计算进程

### Spark Streaming 读取文件流

```python
# pyspark
# 创建 streamingContext 对象
from pyspark.streaming import StreamingContext
ssc = StreamingContext(sc, 1)
# batchDuration:1 每次 1 秒触发计算

# .py
from pyspark import SparkContext, SparkConf 
from pyspark.streaming import StreamingContext 
conf = SparkConf() conf.setAppName('TestDStream') 
conf.setMaster('local[2]') 
sc = SparkContext(conf = conf) 
# 定义 SparkStreaming 对象
ssc = StreamingContext(sc, 5) 
lines = ssc.textFileStream('file:///root/logfile') 
words = lines.flatMap(lambda line: line.split(' ')) 
wordCounts = words.map(lambda x : (x,1)).reduceByKey(lambda a,b:a+b) 
wordCounts.pprint() 
ssc.start() 
ssc.awaitTermination()

```



### Spark Streaming 读取套接字

### Spark Streaming 读取 RDD 数据源

```python
from pyspark import SparkContext
from pyspark.streaning import StreamingContext
import time
sc = SparkContext(appName="PythonStreamingQueueStream") 
ssc = StreamingContext(sc, 2) #创建一个队列，通过该队列可以把RDD推给一个RDD队列流 
rddQueue = [] 
for i in range(5): 
    rddQueue += [ssc.sparkContext.parallelize([j for j in range(1, 1001)], 10)] 
    time.sleep(1) #创建一个RDD队列流 
inputStream = ssc.queueStream(rddQueue) 
mappedStream = inputStream.map(lambda x: (x % 10, 1)) 
reducedStream = mappedStream.reduceByKey(lambda a, b: a + b) 
reducedStream.pprint() 
ssc.start() 
ssc.stop(stopSparkContext=True, stopGraceFully=True)
```

### Spark Streaming 读取 kafka Topic

```python
from __future__ import print_function 
import sys 
from pyspark import SparkContext 
from pyspark.streaming import StreamingContext 
from pyspark.streaming.kafka import KafkaUtils

if __name__ == "__main__": 
    if len(sys.argv) != 3: 
        print("Usage: KafkaWordCount.py <zk> <topic>", file=sys.stderr) exit(-1) 
    sc = SparkContext(appName="PythonStreamingKafkaWordCount") 
    ssc = StreamingContext(sc, 1) 
    zkQuorum, topic = sys.argv[1:] 
    kvs = KafkaUtils.createStream(ssc, zkQuorum, "spark-streaming-consumer", {topic: 1}) 
    lines = kvs.map(lambda x: x[1]) 
    counts = lines.flatMap(lambda line: line.split(" ")).map(lambda word: (word, 1)).reduceByKey(lambda a, b: a+b) 
    counts.pprint() 
    ssc.start() 
    ssc.awaitTermination()
```



## DStream 转换操作

### 无状态转换

只对当前批次进行处理，而不关注历史数据。

* map(func)
* flatMap(func)
* filter(func)
* repartition(numPartitions): 创建分区改变 DStream 的并行程度
* reduce(func)
* count()
* union(otherDstream)
* countByValue()
* reduceByKey(func)
* join(otherStream)
* cogroup(otherStream)
* transform(func):

### 有状态转换

#### 1. 滑动窗口转换操作

reduceByKeyAndWindow(func, invFunc, windowLengthm slideInterval)

**invFunc**作用：

对于离开滑动窗口的数据，使用 invFunc，减少计算量。

否则，需要对窗口内所有数据使用 func 计算。

#### 2. updateStateByKey 操作

可以再跨批次之间维护状态。

```python
def updateFunc(new_values, last_sum):
    return sum(new_values) + (last_sum or 0)

ssc = StreamingContext(sc, 1)
ssc.checkpoint("file:///root/stateful/")
initialStateRDD = sc.parallelize([(u'hello', 1), (u'world', 1)])
lines = ssc.socketTextStream(sys.argv[1], int(sys.argv[2])) 
running_counts = lines.flatMap(lambda line: line.split(" "))\
                .map(lambda word: (word, 1))\
                .updateStateByKey(updateFunc, initialRDD=initialStateRDD)
running_counts.saveAsTextFiles("file:///root/outp")
running_counts.pprint()
ssc.start()
ssc.awaitTermination()
```


