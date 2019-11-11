# Spark Sql

## Hive on Mapreduce

* Step 1：UI(user interface) 调用 executeQuery 接口，发送 HQL 查询语句给 Driver

* Step 2：Driver 为查询语句创建会话句柄，并将查询语句发送给 Compiler， 等待其进行语句解析并生成执行计划

* Step 3 and 4：Compiler 从 metastore 获取相关的元数据

* Step 5：元数据用于对查询树中的表达式进行类型检查，以及基于查询谓词调整分区，生成计划

* Step 6 (6.1，6.2，6.3)：由 Compiler 生成的执行计划是阶段性的 DAG，每个阶段都可能会涉及到 Map/Reduce job、元数据的操作、HDFS 文件的操作，Execution Engine 将各个阶段的 DAG 提交给对应的组件执行。

* Step 7, 8 and 9：在每个任务（mapper / reducer）中，查询结果会以临时文件的方式存储在 HDFS 中。保存查询结果的临时文件由 Execution Engine 直接从 HDFS 读取，作为从 Driver Fetch API 的返回内容。

hive 是 hadoop MapReduce 的封装，会将查询 sql 转换为复杂的 MapReduce 程序。hive对数据的操作**只支持覆盖原数据和追加数据**。

## Shark

MapReduce计算过程中大量的中间磁盘落地过程消耗了大量的I/O，降低的运行效率，为了提高SQL-on-Hadoop的效率，Shark应运而生，但又因为Shark对于Hive的太多依赖**（如采用Hive的语法解析器、查询优化器等等)**,2014年spark团队停止对Shark的开发，将所有资源放SparkSQL项目上。

SparkSQL作为Spark生态的一员继续发展，**而不再受限于Hive，只是兼容Hive；而Hive on Spark是一个Hive的发展计划，该计划将Spark作为Hive的底层引擎之一，也就是说，Hive将不再受限于一个引擎，可以采用Map-Reduce、Tez、Spark等引擎。**



## Hive on Spark

 [hive](https://link.jianshu.com?t=http://lib.csdn.net/base/hive) on Spark是由Cloudera发起，由Intel、MapR等公司共同参与的开源项目，<font color='red'>其目的是把Spark作为Hive的一个计算引擎，将Hive的查询作为Spark的任务提交到Spark集群上进行计算。</font>通过该项目，可以提高Hive查询的性能，同时为已经部署了Hive或者Spark的用户提供了更加灵活的选择，从而进一步提高Hive和Spark的普及率。

## SparkSQL

类似于关系型数据库，SparkSQL也是语句也是由Projection（a1，a2，a3）、Data Source（tableA）、Filter（condition）组成，分别对应sql查询过程中的Result、Data Source、Operation，也就是说SQL语句按Operation–>Data Source–>Result的次序来描述的。

![2671133-3972ba4fa19a51e7](https://tva1.sinaimg.cn/large/006y8mN6ly1g8of8rof4bg30dc087q36.gif)



## Hive on Spark与SparkSql的区别

**hive on spark大体与SparkSQL结构类似，只是SQL引擎不同，但是计算引擎都是spark！**敲黑板！这才是重点！
结构上Hive On Spark和SparkSQL都是一个翻译层，把一个SQL翻译成分布式可执行的Spark程序。

SparkSQL和Hive On Spark都是在Spark上实现SQL的解决方案。Spark早先有Shark项目用来实现SQL层，不过后来推翻重做了，就变成了SparkSQL。这是Spark官方Databricks的项目，Spark项目本身主推的SQL实现。Hive On Spark比SparkSQL稍晚。Hive原本是没有很好支持MapReduce之外的引擎的，而Hive On Tez项目让Hive得以支持和Spark近似的Planning结构（非MapReduce的DAG）。所以在此基础上，Cloudera主导启动了Hive On Spark。这个项目得到了IBM，Intel和MapR的支持（但是没有Databricks）。




## DataFrame

```python

# RDD 是分布式数据集合，但是内部结构是未知的
# DataFrame 是以 RDD 为基础的分布式数据集，提供详细的结构信息

# Spark 使用 SparkSession 接口实现对数据加载、转换、处理的功能
# 在 pyspark 中，pyspark 提供了一个 SparkContext 对象(sc)和一个 SparkSession 对象(spark)

# 加载文件
df = spark.read.text('people.txt')
df = spark.read.json('people.json')

# 保存文件
df.write.text('people.txt')
df.write.json('people.json')

# DataFrame 常用操作
df.printSchema() # 打印信息
df.select() # 选择
df.filter() # 筛选
df.groupBy()
df.sort()

# 从 RDD 转换为 DataFrame
# people.txt
# Michael, 29
# Andy, 30
# Justin, 19
# 1.利用反射机制推断
from pyspark.sql import Row
rdd = sc.textFile('people.txt')
people = rdd.map(lambda x:x.split(',')).map(lambda x:Row(name=x[0], age=int(x[1])))
schemaPeople = spark.createDataFrame(people)
# 2.使用编程方式定义 RDD
# 生成表头、拼加数据
from spark.sql.types import *
from spark.sql import Row
schemaString = "name age"
field = [StructField(f_name, StringType(), True) for f_name in schemaString.split(' ')]
schema = StructField(field)
rdd = sc.textFile('/usr/local/bin/spark/examples/src/main/resources/people.txt')
people = rdd.map(lambda x:x.split(',')).map(lambda x:Row(x[0], x[1]))
people_df = spark.createDataFrame(people, schema)


```



## Spark sql 连接 mysql

```python
# 下载 mysql jdbc 驱动
# wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.48.tar.gz
# 将解压出来的 jar 文件 cp 到 $SPARK_HOME/jars/
jdbcDF = spark.read.format('jdbc').option('driver', 'com.mysql.jdbc.Driver')\
					.option('url', 'jdbc:mysql://localhost:3306/mydb')\
					.option('dbtable', 'mytable') \
					.option('user', 'myuser') \
					.option('password', 'mypassword') \
					.load()

 # 读取 mysql 数据
jdbcDF.show()

# 写入 mysql 数据
# 先设置表头、即 schema
# 再通过 spark.createDataFrame() 创建 DataFrame
prop = {'user': myuser, 'password', mypassword, 'driver': 'com.mysql.jdbc.driver'}
df.write.jdbc('jdbc:mysql://localhost:3306/mydb', 'mytable', 'append', prop)
```



*参考：*

* https://www.jianshu.com/p/a38215b6395c




