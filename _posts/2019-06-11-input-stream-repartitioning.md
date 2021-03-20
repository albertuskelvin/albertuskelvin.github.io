---
title: 'Repartitioning Input Data Stream'
date: 2019-06-11
permalink: /posts/2019/06/input-stream-repartitioning/
tags:
  - spark streaming
  - input data stream
  - repartitioning
---

Recently I played with a simple Spark Streaming application. Precisely, I investigated the behavior of repartitioning on different level of input data streams. For instance, we have two input data streams, such as <i>linesDStream</i> and <i>wordsDStream</i>. The question is, is the repartitioning result different if I repartition after <i>linesDStream</i> and after <i>wordsDStream</i>? 

Here's the code I used.

```python
from pyspark import SparkContext
from pyspark.sql import Row, SparkSession
from pyspark.streaming import StreamingContext


def getSparkSessionInstance():
  # this code was taken from https://github.com/apache/spark/blob/v2.4.3/examples/src/main/python/streaming/sql_network_wordcount.py
  if ('sparkSessionSingletonInstance' not in globals()):
	  globals()['sparkSessionSingletonInstance'] = SparkSession.builder.getOrCreate()
  return globals()['sparkSessionSingletonInstance']

sc = SparkContext("local[*]", "RepartitioningInputDataStream")
ssc = StreamingContext(sc, 20)    # Uses 20 second as the batch interval

def process(time, rdd):
	print("========= %s =========" % str(time))
  
	try:
		print(rdd.collect())
		print('Num of partitions: {}'.format(rdd.getNumPartitions()))
		for index, partition in enumerate(rdd.glom().collect()):
			print('Partition {}'.format(index))
			print(partition)
	except:
		pass

lines = ssc.socketTextStream("localhost", 9999)
lines = lines.repartition(8)

words = lines.flatMap(lambda line: line.split(" "))
#words = words.repartition(8)

pairs = words.map(lambda word: (word, 1))
#pairs = pairs.repartition(8)

pairs.foreachRDD(process)

ssc.start()                       # Start the computation
ssc.awaitTermination()            # Wait for the computation to terminate
```

The code above does a very simple thing. It receives input stream at localhost:9999. Then, each line is splitted to words using a whitespace as delimiter. Finally, each word is mapped to an integer. As the final process, we calculate the number and content of partitions in each RDD. Additionally, after each transformation, a repartitioning operation is done.

To execute the code, open 2 Terminals and type the following commands:

<pre>
Terminal 1
nc -lk 9999

Terminal 2
(path_to_the_spark_submit) (path_to_the_python_code) localhost 9999
</pre>

Here's the input data stream that I used (type these on <i>Terminal 1</i>).

<pre>
ab cd ef gh ij kl
mn op qr st
a
b
c
x
y
z
</pre>

Here's the result I got.

<pre>
Repartitioning after <b>ssc.socketTextStream</b>

Num of partitions: 8
Partition 0
('y', 1)
Partition 1
('z', 1)
Partition 2
('mn', 1), ('op', 1), ('qr', 1), ('st', 1), ('x', 1)
Partition 3
('c', 1)
Partition 4
<i>empty</i>
Partition 5
<i>empty</i>
Partition 6
('ab', 1), ('cd', 1), ('ef', 1), ('gh', 1), ('ij', 1), ('kl', 1)
Partition 7
('a', 1), ('b', 1)

============

Repartitioning after <b>lines.flatMap(lambda line: line.split(" "))</b>

Num of partitions: 8
Partition 0
('y', 1)
Partition 1
('z', 1)
Partition 2
('mn', 1), ('op', 1), ('qr', 1), ('st', 1), ('x', 1)
Partition 3
('c', 1)
Partition 4
<i>empty</i>
Partition 5
<i>empty</i>
Partition 6
('ab', 1), ('cd', 1), ('ef', 1), ('gh', 1), ('ij', 1), ('kl', 1)
Partition 7
('a', 1), ('b', 1)

============

Repartitioning after <b>words.map(lambda word: (word, 1))</b>

Num of partitions: 8
Partition 0
('y', 1)
Partition 1
('z', 1)
Partition 2
('mn', 1), ('op', 1), ('qr', 1), ('st', 1), ('x', 1)
Partition 3
('c', 1)
Partition 4
<i>empty</i>
Partition 5
<i>empty</i>
Partition 6
('ab', 1), ('cd', 1), ('ef', 1), ('gh', 1), ('ij', 1), ('kl', 1)
Partition 7
('a', 1), ('b', 1)
</pre>

Based on the result above, the repartitioning result is the same. From all the repartitioning positions, the same data is located to the same partition.

Furthermore, all the elements that was actually in the same line before being splitted to words are located to the same partition. For instance, <b>ab cd ef gh ij kl</b> is splitted to <b>ab, cd, ef, gh, ij, kl</b>. At first I thought these elements would be located to different partitions since they were already splitted (became different rows). But the result shows that Spark makes them reside in the same partition (Partition 6). However, this doesn't mean that a single partition will only be occupied by elements splitted from a line. We can see that in <i>Partition 2</i> we have another element, namely <i>x</i>.

Thanks for reading.
