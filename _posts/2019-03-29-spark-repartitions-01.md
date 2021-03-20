---
title: 'A Little Experiment on Dataframe Repartitioning'
date: 2019-03-29
permalink: /posts/2019/03/dataframe-repartition-experiment
tags:
  - spark
  - repartitioning
  - dataframe
---

Spark has two types of partitioning. The first one is coalesce, while the second one is repartition.

In coalesce, Spark only reduces the number of partition. The reduction is performed by moving elements from some partitions to the **existing** partitions. The values in some partitions are not moved at all. Therefore, it's really obvious that we can't increase the number of partitions. However, coalesce algorithm is fast since it doesn't move the data in some partitions at all (reduces data movement).

Meanwhile, in repartition, Spark enables us to create a brand new partition (and also reduces the number of partition). It shuffles the data in the partitions which means that the concept of <b>"values in some partitions are not moved at all"</b> like in coalesce doesn't apply to repartition. However, repartitioning strives to make each partition has similar number of data. However, this algorithm might be slower than coalesce since it doesn't try to reduce data movement.

Alright, I think the introduction is enough. Let's take a dive into the story.

So I was experimenting with Spark's repartitioning. I did it by using dataframe's columns. In other words, what I did was repartitioning the dataframe based on column (similar to group by concept). For example, let's use the following dataframe:

<pre>
    ----------------------
    COL_A   COL_B    COL_C
    ----------------------
      1   |   1   |  test0
      1   |   0   |  test1
      1   |   1   |  test2
      0   |   0   |  test3
      1   |   0   |  test4
      0   |   0   |  test5
      0   |   1   |  test6
    ----------------------
</pre>

Let's say that I want to repartition the dataframe based on columns **COL_A** and **COL_B**. Therefore, the repartitioning syntax will be (**df** is our dataframe):

<pre>
    [1] df = df.repartition('COL_A', 'COL_B')
    [2] num_partitions = df.rdd.getNumPartitions()
    [3] partitioning_distribution = df.rdd.glom().map(len).collect()
</pre>

As we can see, line [1] denotes that we'd like to repartition our dataframe based on **COL_A** and **COL_B**. At this point, the result should be something like the following:

<pre>

    PARTITIONING DISTRIBUTION
    +++++++++++++++++++++++++
    
    <b>-- PARTITION 01 --</b>
    ----------------------
    COL_A   COL_B    COL_C
    ----------------------
      1   |   1   |  test0
      1   |   1   |  test2
    ----------------------
      
    <b>-- PARTITION 02 --</b>
    ----------------------
    COL_A   COL_B    COL_C
    ----------------------
      1   |   0   |  test1
      1   |   0   |  test4
    ----------------------
   
    <b>-- PARTITION 03 --</b>
    ----------------------
    COL_A   COL_B    COL_C
    ----------------------
      0   |   1   |  test6
    ----------------------
    
    <b>-- PARTITION 04 --</b>
    ----------------------
    COL_A   COL_B    COL_C
    ----------------------
      0   |   0   |  test3
      0   |   0   |  test5
    ----------------------
</pre>

And line [2] should show us that the dataframe has been split into **4** partitions. Line [3] should show the data in each partition like the above **PARTITIONING DISTRIBUTION**.

But the reality was different from my expectation. Line [2] gave **200** as the number of partitions. Line [3] showed that **4** partitions had data (non-empty), while the other **196** partitions didn't have data (empty).

A question popped up in my mind. Why the number of partitions was 200?

I browsed the internet and found this great article: <a href="https://medium.com/@mrpowers/managing-spark-partitions-with-coalesce-and-repartition-4050c57ad5c4">Managing Spark Partitions with Coalesce and Repartition</a>.

And I found that when we do repartitioning by columns, Spark will create **200** partitions by default. Off course it's greatly inefficient to process 200 partitions where most of them are empty - like a sparse vector :D.

To resolve such an issue, we only need to specify the number of partition we'd like Spark to create. In this case, since the combination of **COL_A** and **COL_B** gives a total of 4 possible values (11, 10, 01, and 00), the number of partitions should be 4. We can replace the above code (line [1]) with the following:

<pre>
    [1] df = df.repartition(4, 'COL_A', 'COL_B')
</pre>

Or if the number of possible values is too many and it's difficult to calculate them manually, we may do this simple trick:

<pre>
    [1] num_of_distinct_values = df.select('COL_A', 'COL_B').distinct().count()
    [2] df = df.repartition(num_of_distinct_values, 'COL_A', 'COL_B')
</pre>

Problem solved.

Thanks for reading.
