---
title: 'Spark Accumulator'
date: 2019-03-23
permalink: /posts/2019/03/spark-accumulator/
tags:
  - spark
  - accumulator
---

A few days ago I conducted a little experiment on Spark's RDD operations. One of them was **foreach** operation (included as an action). Simply, this operation is applied to each rows in the RDD and the kind of operation applied is specified via a certain function. Here's a simple example:

<pre>
    [1] info = {
    [2]     'total_point': 0
    [3] }
    [4]
    [5] def fn(rdd_row):
    [6]     global info
    [7]     if rdd_row['score'] <= 50:
    [8]         info['total_point'] += 10
    [9]     else:
    [10]        info['total_point'] += 50
    [11]
    [12] fore = my_rdd.foreach(fn)
    [13] 
    [14] print(info)
</pre>

Simple, isn't it? Well, the problem was the value of <i>info['total_point']</i> after executing **foreach** was still 0.

I browsed the internet and found that the primary problem was related to task's closure. Basically, the final value of <i>total_point</i> was still zero because each executor only referred to the copies of _info_ variable. Since the variables were only copies, the executors didn't really update the value of the _info_ resided within the driver node. You can find more details <a href="https://spark.apache.org/docs/latest/rdd-programming-guide.html#local-vs-cluster-modes">here</a>.

The solution was pretty simple. I used what is called with **Accumulator**. It ensures well-defined behavior in such an operation scenario. You can find more details about **Accumulator** <a href="https://spark.apache.org/docs/latest/rdd-programming-guide.html#accumulators">here</a>.

Oh ya, here's the updated version of the previous code snippet:

<pre>
    [1] info = {
    [2]     'total_point': sc.accumulator(0)    # 'sc' is our Spark Context
    [3] }
    [4]
    [5] def fn(rdd_row):
    [6]     if rdd_row['score'] <= 50:
    [7]         info['total_point'].add(10)
    [8]     else:
    [9]         info['total_point'].add(50)
    [10]
    [11] fore = my_rdd.foreach(fn)
    [12] 
    [13] print(info)
</pre>

Problem solved.

Well, I think I'd love to take a deeper dive into this **Accumulator** topic in the future.

Thanks for reading.
