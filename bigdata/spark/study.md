
```
textFile = spark.read.text("hdfs://localhost:9000//user/hive/warehouse/README.md")

textFile.first()

textFile.filter(textFile.value.contains('Spark')).count()

from pyspark.sql.functions import *
textFile.select(size(split(textFile.value, "\s+")).name("numWords")).agg(max(col("numWords"))).collect()

textFile.select(explode(split(textFile.value, "\s+")).alias("word")).groupBy("word").count().colllect()
[Row(word=u'online', count=1), Row(word=u'graphs', count=1),
```


```
from pyspark import SparkContext, SparkConf
conf = SparkConf().setAppName('test').setMaster()
```

//rdd
```
lines = sc.textFile("/Users/rick/src/hadoop/spark-2.2.1-bin-hadoop2.7/data/graphx/data.txt")
pairs = lines.map(lambda s: (s, 1))
counts = pairs.reduceByKey(lambda a, b: a + b)
```

//SQL, dataFrames DataSets


//Creating DataFrames

```
# spark is an existing SparkSession
df = spark.read.json("examples/src/main/resources/people.json")
# Displays the content of the DataFrame to stdout
df.show()
# +----+-------+
# | age|   name|
# +----+-------+
# |null|Michael|
# |  30|   Andy|
# |  19| Justin|
# +----+-------+

```

```
//Untyped Dataset Operations (aka DataFrame Operations)
# spark, df are from the previous example
# Print the schema in a tree format
df.printSchema()
# root
# |-- age: long (nullable = true)
# |-- name: string (nullable = true)

# Select only the "name" column
df.select("name").show()
# +-------+
# |   name|
# +-------+
# |Michael|
# |   Andy|
# | Justin|
# +-------+

# Select everybody, but increment the age by 1
df.select(df['name'], df['age'] + 1).show()
# +-------+---------+
# |   name|(age + 1)|
# +-------+---------+
# |Michael|     null|
# |   Andy|       31|
# | Justin|       20|
# +-------+---------+

# Select people older than 21
df.filter(df['age'] > 21).show()
# +---+----+
# |age|name|
# +---+----+
# | 30|Andy|
# +---+----+

# Count people by age
df.groupBy("age").count().show()
# +----+-----+
# | age|count|
# +----+-----+
# |  19|    1|
# |null|    1|
# |  30|    1|
# +----+-----+
```

```
//Running SQL Queries Programmatically
# Register the DataFrame as a SQL temporary view
df.createOrReplaceTempView("people")

sqlDF = spark.sql("SELECT * FROM people")
sqlDF.show()
# +----+-------+
# | age|   name|
# +----+-------+
# |null|Michael|
# |  30|   Andy|
# |  19| Justin|
# +----+-------+
```

```
//global temporay view
# Register the DataFrame as a global temporary view
df.createGlobalTempView("people")

# Global temporary view is tied to a system preserved database `global_temp`
spark.sql("SELECT * FROM global_temp.people").show()
# +----+-------+
# | age|   name|
# +----+-------+
# |null|Michael|
# |  30|   Andy|
# |  19| Justin|
# +----+-------+

# Global temporary view is cross-session
spark.newSession().sql("SELECT * FROM global_temp.people").show()
# +----+-------+
# | age|   name|
# +----+-------+
# |null|Michael|
# |  30|   Andy|
# |  19| Justin|
# +----+-------+
```

```
from pyspark.sql import Row

sc = spark.sparkContext

# Load a text file and convert each line to a Row.
lines = sc.textFile("examples/src/main/resources/people.txt")
parts = lines.map(lambda l: l.split(","))
people = parts.map(lambda p: Row(name=p[0], age=int(p[1])))

# Infer the schema, and register the DataFrame as a table.
schemaPeople = spark.createDataFrame(people)
schemaPeople.createOrReplaceTempView("people")

# SQL can be run over DataFrames that have been registered as a table.
teenagers = spark.sql("SELECT name FROM people WHERE age >= 13 AND age <= 19")

# The results of SQL queries are Dataframe objects.
# rdd returns the content as an :class:`pyspark.RDD` of :class:`Row`.
teenNames = teenagers.rdd.map(lambda p: "Name: " + p.name).collect()
for name in teenNames:
    print(name)
```

```
Programmatically Specifying the Schema
# Import data types
from pyspark.sql.types import *

sc = spark.sparkContext

# Load a text file and convert each line to a Row.
lines = sc.textFile("examples/src/main/resources/people.txt")
parts = lines.map(lambda l: l.split(","))
# Each line is converted to a tuple.
people = parts.map(lambda p: (p[0], p[1].strip()))

# The schema is encoded in a string.
schemaString = "name age"

fields = [StructField(field_name, StringType(), True) for field_name in schemaString.split()]
schema = StructType(fields)

# Apply the schema to the RDD.
schemaPeople = spark.createDataFrame(people, schema)

# Creates a temporary view using the DataFrame
schemaPeople.createOrReplaceTempView("people")

# SQL can be run over DataFrames that have been registered as a table.
results = spark.sql("SELECT name FROM people")

results.show()
# +-------+
# |   name|
# +-------+
# |Michael|
# |   Andy|
# | Justin|
# +-------+
```

```
Aggregations

```

###  Spark基本原理以及核心概念  Spark基本工作原理 
1. Client客户端：我们在本地编写了spark程序，打成jar包，或python脚本,通过spark submit命令提交到Spark集群； 
2. 只有Spark程序在Spark集群上运行才能拿到Spark资源，来读取数据源的数据进入到内存里； 
3. 客户端就在Spark分布式内存中并行迭代地处理数据，注意每个处理过程都是在内存中并行迭代完成；注意：每一批节点上的每一批数据，实际上就是一个RDD！！！一个RDD是分布式的，所以数据都散落在一批节点上了，每个节点都存储了RDD的部分partition。 
4. Spark与MapReduce最大的不同在于，迭代式计算模型：MapReduce，分为两个阶段，map和reduce，两个阶段完了，就结束了，所以我们在一个job里能做的处理很有限； Spark，计算模型，可以分为n个阶段，因为它是内存迭代式的。我们在处理完一个阶段以后，可以继续往下处理很多个阶段，而不只是两个阶段。所以，Spark相较于MapReduce来说，计算模型可以提供更强大的功能。

### RDD以及其特性 
1. RDD是Spark提供的核心抽象，全称为Resillient Distributed Dataset，即弹性分布式数据集; 
2. RDD在抽象上来说是一种元素集合，包含了数据。它是被分区的，分为多个分区，每个分区分布在集群中的不同节点上，从而让RDD中的数据可以被并行操作。（分布式数据集）; 
3. RDD通常通过Hadoop上的文件，即HDFS文件或者Hive表，来进行创建；有时也可以通过应用程序中的集合来创建; 
4. RDD最重要的特性就是，提供了容错性，可以自动从节点失败中恢复过来。即如果某个节点上的RDD partition，因为节点故障，导致数据丢了，那么RDD会自动通过自己的数据来源重新计算该partition。这一切对使用者是透明的; 
5. RDD的数据默认情况下存放在内存中的，但是在内存资源不足时，Spark会自动将RDD数据写入磁盘。（弹性）。

### 问题：RDD分布式是什么意思？ 
* 一个RDD，在逻辑上，抽象地代表了一个HDFS文件；但是，它实际上是被分为多个分区；多个分区散落在Spark集群中，不同的节点上。比如说，RDD有900万数据。分为9个partition，9个分区。
### 问题：RDD弹性是什么意思，体现在哪一方面？ 
* RDD的每个partition，在spark节点上存储时，默认都是放在内存中的。但是如果说内存放不下这么多数据时，比如每个节点最多放50万数据，结果你每个partition是100万数据。那么就会把partition中的部分数据写入磁盘上，进行保存。 
而上述这一切，对于用户来说，都是完全透明的。也就是说，你不用去管RDD的数据存储在哪里，内存，还是磁盘。只要关注，你针对RDD来进行计算，和处理，等等操作即可。所以说，RDD的这种自动进行内存和磁盘之间权衡和切换的机制，就是RDD的弹性的特点所在。

### 问题：RDD容错性体现在哪方面？ 
* 现在，节点9出了些故障，导致partition9的数据丢失了。那么此时Spark会脆弱到直接报错，直接挂掉吗？不可能！！ 
RDD是有很强的容错性的，当它发现自己的数据丢失了以后，会自动从自己来源的数据进行重计算，重新获取自己这份数据，这一切对用户，都是完全透明的。

### 什么是Spark开发 
1. 核心开发：离线批处理 / 延迟性的交互式数据处理 
Spark的核心编程是什么？其实，就是： 
首先，第一，定义初始的RDD，就是说，你要定义第一个RDD是从哪里，读取数据，hdfs、linux本地文件、程序中的集合。 
第二，定义对RDD的计算操作，这个在spark里称之为算子，map、reduce、flatMap、groupByKey，比mapreduce提供的map和reduce强大的太多太多了。 
第三，其实就是循环往复的过程，第一个计算完了以后，数据可能就会到了新的一批节点上，也就是变成一个新的RDD。然后再次反复，针对新的RDD定义计算操作。。。。 
第四，最后，就是获得最终的数据，将数据保存起来。 
2. SQL查询：底层都是RDD和计算操作 
3. 实时计算：底层都是RDD和计算操作

### 两种方式实现word_count
```
//1.
textFile = spark.read.textFile("../readme.md")
val words = textFile.flatMap({line => line.split(" ")}).groupByKey(identity).count()
varl wordCounts = words.collect()

//2.
textFile = sctextFile("../readme.md")
val words = textFile.flatMat(line => line.split(" "))
val wordsCount = words.map(line => (line, 1)).reduceByKey((x, y) => x+ y)
```
