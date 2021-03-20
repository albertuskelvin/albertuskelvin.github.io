---
title: 'Structured Streaming Checkpointing with Parquet Stream Source'
date: 2019-10-12
permalink: /posts/2019/10/structured-streaming-with-parquet-stream-source/
tags:
  - spark
  - parquet
  - structured streaming
---

I was curious about how checkpoint files in Spark <a href="https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html">structured streaming</a> looked like. To introduce the basic concept, checkpointing simply denotes the progress information of streaming process. This checkpoint files are usually used for failure recovery. More detail explanation can be found <a href="https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#recovering-from-failures-with-checkpointing">here</a>.

The structure of the checkpoint directory might be different for various streaming sources. For instance, socket and parquet streaming source yield different checkpoint directory structures. Socket stream source does not generate a directory called <b>sources/0/</b>, while parquet stream source generates it.

In this article I'll use parquet as the streaming source.

Here's the scenario. We're going to read streaming data from a dedicated sink source, let's call it <b>streamedParquets</b>. This <b>streamedParquets</b> stores a collection of parquet files sunk by another streaming process. Suppose the followings are the parquet files.

```
\ streamedParquets
    -- a.parquet
    -- b.parquet
    -- c.parquet
    -- d.parquet
    -- e.parquet
    -- f.parquet
    -- g.parquet
    -- h.parquet
```

In addition, let's presume that each parquet file contains an extremely simple dataframe, such as the following.

```
+---------+
+   word  +
+---------+
+ hello_a +
+ world_a +
+---------+
```

Our streaming process will then read the parquet files according to the below code.

```python
import time
from datetime import datetime

from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.types import StringType, StructField, StructType

spark = SparkSession \
    .builder \
    .appName("structuredStreamingParquet") \
    .getOrCreate()

# Define a schema for the parquet files
parquet_schema = StructType([
    StructField('word', StringType())
])

df = spark \
    .readStream \
    .schema(parquet_schema) \
    .option('maxFilesPerTrigger', 4) \
    .parquet("streamedParquets/")
```

The above code simply works like the followings:

<ul>
<li>Define a schema for the parquet files. In this case, the dataframe in each parquet file should have a column called <b>word</b> with string as the data type. Any parquet that does not follow this rule will not be processed</li>
<li>Create a stream reader for reading the stream data (does not start the streaming process). This reader consists of several information, such as:
  <ul>
    <li>valid schema</li>
    <li>number of parquets to be processed within a single batch (4 parquet files in this case)</li>
    <li>parquet stream source (<b>streamedParquets/</b>)</li>
  </ul>
</li>
</ul>

After defining the stream reader, we then can proceed to start the streaming process. Below shows the code snippet.

```python
def process_batch(df, epoch_id):
	print("Batch {}: {}".format(epoch_id, datetime.fromtimestamp(time.time())))
	df = df.withColumn('TEST', F.lit('abc'))
	df.show()


df = df \
    .writeStream \
    .trigger(processingTime='5 seconds') \
    .foreachBatch(lambda df, epoch_id: process_batch(df, epoch_id)) \
    .option('checkpointLocation', "checkpoints/") \
    .start()

df.awaitTermination()
```

The above code simply does the followings:

<ul>
<li>For every 5 seconds, create a new batch by the following processes:
  <ul>
    <li>retrieve <b>maxFilesPerTrigger</b> parquet data</li>
    <li>append the data to an unbounded (input) table</li>
    <li>process the data using the defined batch processor (in this case, a method called <b>process_batch</b>)</li>
  </ul>
</li>
<li>Write the checkpoint files to a dedicated location, that is <b>checkpoints</b> in this case</li>
</ul>

After the streaming is executed, Spark will create several directories in <b>checkpoints</b> directory. One of them is <b>sources/0/</b>.

Inside the <b>sources/0/</b> is simply a collection of files named incrementally (starts from 0). This file name denotes the batch ID.

Here's the example for <b>batch 0</b>.

```
v1
{"path":"streamedParquets/a.parquet","timestamp":<timestamp_p>,"batchId":0}
{"path":"streamedParquets/b.parquet","timestamp":<timestamp_q>,"batchId":0}
{"path":"streamedParquets/c.parquet","timestamp":<timestamp_r>,"batchId":0}
{"path":"streamedParquets/d.parquet","timestamp":<timestamp_s>,"batchId":0}
```

And here's the example for <b>batch 1</b>.

```
v1
{"path":"streamedParquets/e.parquet","timestamp":<timestamp_k>,"batchId":1}
{"path":"streamedParquets/f.parquet","timestamp":<timestamp_l>,"batchId":1}
{"path":"streamedParquets/g.parquet","timestamp":<timestamp_m>,"batchId":1}
{"path":"streamedParquets/h.parquet","timestamp":<timestamp_n>,"batchId":1}
```

Please note that the timestamp for each parquet file within the same batch <b>might be different</b>.

After knowing the content of the checkpoint files, let's take a look at the output of the batch processor on Terminal.

```
Batch 0: datetime_batch_0
+---------+
+   word  +
+---------+
+ hello_a +
+ world_a +
+ hello_b +
+ world_b +
+ hello_c +
+ world_c +
+ hello_d +
+ world_d +
+---------+

Batch 1: datetime_batch_0 + 5 seconds (processing time)
+---------+
+   word  +
+---------+
+ hello_e +
+ world_e +
+ hello_f +
+ world_f +
+ hello_g +
+ world_g +
+ hello_h +
+ world_h +
+---------+
```

Thank you for reading.
