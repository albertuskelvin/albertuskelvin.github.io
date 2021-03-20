---
title: 'Spark Structured Streaming with Parquet Stream Source & Multiple Stream Queries'
date: 2019-11-15
permalink: /posts/2019/11/spark-structured-streaming-multiple-queries/
tags:
  - spark
  - structured streaming
  - parquet
  - multiple queries
---

Whenever we call `dataframe.writeStream.start()` in structured streaming, Spark creates a new stream that reads from a data source (specified by `dataframe.readStream`). The data passed through the stream is then processed (if needed) and sinked to a certain location.

It got me thinking. What would happen if we define multiple writestream that read data from the same stream source? To be specific, this article uses parquet files as the stream source.

Let’s simulate it with the following simple code.

```python
spark = SparkSession \
    .builder \
    .appName("MultipleStreamQueries") \
    .getOrCreate()


def process_batch_query_a(df, epoch_id):
    print('Query A: {}'.format(datetime.fromtimestamp(time.time())))
    df = df.withColumn('QUERY', F.lit('a'))
    df.show()

def process_batch_query_b(df, epoch_id):
    print('Query B: {}'.format(datetime.fromtimestamp(time.time())))
    df = df.withColumn('QUERY', F.lit(‘b’))
    df.show()


parquet_schema = StructType([
    StructField('word', StringType())
])

df = spark \
    .readStream \
    .schema(parquet_schema) \
    .option('maxFilesPerTrigger', 4) \
    .parquet("streamedParquets/")

df_query_a = df \
    .writeStream \
    .outputMode('append') \
    .trigger(processingTime='1 seconds') \
    .foreachBatch(lambda df, epoch_id: process_batch_query_a(df, epoch_id)) \
    .start()

# wait until it is terminated
df_query_a.awaitTermination()

df_query_b = df \
    .writeStream \
    .outputMode('append') \
    .trigger(processingTime='1 seconds') \
    .foreachBatch(lambda df, epoch_id: process_batch_query_b(df, epoch_id)) \
    .start()

# wait until it is terminated
df_query_b.awaitTermination()
```

The above code simply tells Spark to read four parquet files in each micro batch (trigerred every 1 second). Afterwards, we create two writestream which each of them has different batch processor.

However, the above code failed when I tried to execute it. Only one writestream that was executed (`df_sink_a`).

Turned out that there’s indeed a way to execute multiple stream queries (writestream) in a single Spark session. Here’s what is written in the <a href="https://spark.apache.org/docs/2.3.0/structured-streaming-programming-guide.html#managing-streaming-queries">documentation</a> itself:

```
You can start any number of queries in a single SparkSession. 
They will all be running concurrently sharing the cluster resources. 
You can use sparkSession.streams() to get the StreamingQueryManager (Scala/Java/Python docs) that can be used to manage the currently active queries.
```

And the way to enable this multiple streaming queries is by applying `sparkSession.streams.awaitAnyTermination()` instead of `dataframe.awaitTermination()` applied to each writestream. Since we want these multiple streaming queries to run concurrently, we use <b>StreamingQueryManager</b> class instead of <b>StreamingQuery</b> class.

So here’s the updated solution.

```python
df_query_a = df \
    .writeStream \
    .outputMode('append') \
    .trigger(processingTime='1 seconds') \
    .foreachBatch(lambda df, epoch_id: process_batch_query_a(df, epoch_id)) \
    .start()

df_query_b = df \
    .writeStream \
    .outputMode('append') \
    .trigger(processingTime='1 seconds') \
    .foreachBatch(lambda df, epoch_id: process_batch_query_b(df, epoch_id)) \
    .start()

# long-running until one of the queries is terminated
spark.streams.awaitAnyTermination()
```

However, although `sparkSession.streams.awaitAnyTermination()` enables all the queries run concurrently, the whole streaming queries will be terminated once a query stops. I think this behaviour might not be suitable for certain cases where each streaming query is independent from each other.

In addition, creating two `writeStream` (streaming queries) like we did above means that the stream source will be read twice (one for each streaming query).

So how to make the whole streaming process still run even though there’s a failure in one of the queries?

I think we could leverage `foreachBatch` approach as the work around. However, we should use this approach with certain assumptions, such as we don’t care about the trigger time, we use the same output mode, and so forth.

Here’s the code snippet.

```python
def process_batch(df, epoch_id):
    # sink to the first location
    …
    # sink to the second location
    …


df = df \
    .writeStream \
    .outputMode('append') \
    .trigger(processingTime=‘1 seconds') \
    .foreachBatch(lambda df, epoch_id: process_batch(df, epoch_id)) \
    .start()

df.awaitTermination()
```

Using the above approach, we only create one streaming query for multiple sinks. Additionally, we don’t need to re-read the same stream source all over again.

One thing left is to evaluate the performance of both approaches.
