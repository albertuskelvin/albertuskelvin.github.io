---
title: 'Too Lazy to Process the Whole Dataframe'
date: 2019-05-09
permalink: /posts/2019/05/lazy-evaluation-01/
tags:
  - spark
  - lazy evaluation
  - dataframe
---

One of the characteristics of Spark that makes me interested to explore this framework further is its lazy evaluation approach. Simply put, Spark won't execute the transformation until an action is called. I think it's logical since when we only specify the transformation plan and don't ask it to execute the plan, why it needs to force itself to do the computation on the data? In addition, by implementing this lazy evaluation approach, Spark might be able to optimize the logical plan. The task of making the query to be more efficient manually might be reduced significantly. Cool, right?

However, there was one simple thing that made me surprised about this lazy behavior. Consider this case. You defined a transformation plan on a dataframe. Then, you called an action using a show() method. By default, Spark will only show 20 rows. You might want to retrieve more rows and you did that by passing an integer N as a parameter to the show() method where N > 20. You did the same thing again but this time the new integer M is larger than N. The question is, would the execution time needed by all the three cases be pretty similar or even the same? An intriguing question. Let's write some code.

<pre>
import time

from pyspark import SparkContext, SparkConf, SQLContext

conf = SparkConf().setAppName("test")
sc = SparkContext(conf=conf)
sqlContext = SQLContext(sc)

columns = ['F0', 'F1', 'F2', 'F3', 'F4', 'F5', 'F6', 'F7', 'F8', 'F9', 'F10']

# L is the list of tuples representing rows in the dataframe
rdd = sc.parallelize(L)
df = rdd.toDF(columns)

start = time.time()

df = df.withColumn('NEW_COL', df['F0'] * df['F1'] * df['F2'])

# shows the first N elements
df.show(20)

print('TIME: ' + str(time.time() - start))
</pre>

You can play with the number of N to answer the previous question. I set N to several values, such as 20, 100, 500, 2500, 12500, 62500, and 312500. Here's what I got.

<pre>
Number of rows: time needed to execute the plan (in secs)

20 rows: 0.888638973236084 secs
100 rows: 0.8791308403015137 secs
500 rows: 0.9497220516204834 secs
2500 rows: 1.0902130603790283 secs
12500 rows: 1.3850088119506836 secs
62500 rows: 2.189028739929199 secs
312500 rows: 8.4181969165802 secs
</pre>

As you can see from the above result, as I increased the number of rows, the time needed to execute the plan increased as well. This might conclude that Spark only applies the transformation to N rows. We specify the value of N by ourselves. Even though the resulting dataframe actually has M rows (where M > N), Spark only process N rows since we asked it to do so. The remaining (M - N) rows won't be processed. Well, I think it's efficient since we don't need to wait for Spark to process the whole dataframe (M rows) only to retrieve N rows (where N <= M).
