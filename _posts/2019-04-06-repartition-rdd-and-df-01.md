---
title: 'Custom Partitioner for Repartitioning in Spark'
date: 2019-04-06
permalink: /posts/2019/04/repartition-rdd-and-df-01/
tags:
  - spark
  - rdd
  - dataframe
  - repartitioning
---

A statement I encountered a few days ago: "Avoid to use Resilient Distributed Datasets (RDDs) and use Dataframes/Datasets (DFs/DTs) instead, especially in production stage".

Using RDDs means that we're telling Spark "how to get a certain task done". Meanwhile, DFs/DTs enable us to express "what should be done" and let Spark handles the rest. That's why implementing the same operation using DFs/DTs is way simpler and more efficient (well, you've to be familiar with SQL first).

However, I think we can't just ignore RDDs usage at least until Spark has already had a nearly similar flexibility (like what exist on RDDs) applied to DFs/DTs. Let me show you a simple case. Suppose you're going to repartition a DF based on a column as well as sort the DF within each partition. In Spark, you can use a method called **repartition** to repartition the DF and **sortWithinPartitions** to sort the DF within each partition. However, sometimes, **repartition** doesn't give you the expected result. In other words, it's not guaranteed that each partition created for each column will have the same value. Different values from the same column might be stored in the same partition. Well, I've experienced this before. I investigated such an issue and realised that Spark uses a **HashPartitioner** to do the repartitioning. Simply put, an example of a **HashPartitioner** is <i>hash(X) % M</i> where <i>X</i> is the key and <i>M</i> is the number of partitions.

Let's say we have <i>X = 1, 2, 3, 4, 5, 8</i>. In this case we've 6 distinct values (<i>M</i>). We can see that the previous hash function gives the same result (2) for <i>X = 2</i> and <i>X = 8</i>. Therefore, these two <i>X</i> values will go to partition 2, which is wrong.

Unfortunately, a feature which can be used to customize the partitioner function hasn't been released for DF/DT. So the behavior of the <b>repartition</b> method will not be well-defined until we can modify the partitioner. I'd like to write more about this behavior in my upcoming tech articles.

However, we've another option to deal with this issue. We may use a similar method provided by low-level APIs (in this case RDDs), namely <b>repartitionAndSortWithinPartitions</b>. The only difference between this method and the previous one is that it repartitions and sorts the RDDs simultaneously. FYI, it's much more efficient than repartitioning first and then sorting the data.

Using this RDDs' method, we can customize the partitioner and make sure that it locates each data to the relevant partition. Yes, we can use any algorithm other than Hash. Greatly flexible, right?

For the sake of clarity, let's get our hands dirty.

Suppose we have a dataframe **df** consisting of several columns, such as **COL_A, COL_B,** and **COL_C**. We're going to repartition **df** based on **COL_A** and sort it using **COL_C**.

<pre>
Original Dataframe (df)

COL_A   COL_B   COL_C
---------------------
  1     test1     2
  1     test2     1
  2     test3     5
  3     test4     0
  3     test5     7
  8     test6     6
  9     test7     8
  9     test8     3
  9     test9     9
  13    test10    10
</pre>

Now let's see the final result using different approaches mentioned previously.

<pre>
<b>API level: Dataframe</b>

<b>Method:</b> <b>repartition</b> first then <b>sortWithinPartitions</b>

<b>Partitioner:</b> hash(x) % n; where <b>x</b> is the data and <b>n</b> is the number of partitions we'd like to create
</pre>

<b>CODE:</b>

```python
num_partitions = df.select('COL_A').distinct().count()
df = df.repartition(num_partitions, 'COL_A')
df = df.sortWithinPartitions('COL_C')
```

<pre>
<b>Result:</b>

COL_A value = 1 and partitioner = hash(1) % 6 = 1 then stored in <b>partition 1</b> -> sort based on COL_C
COL_A value = 2 and partitioner = hash(2) % 6 = 2 then stored in <b>partition 2</b> -> sort based on COL_C
COL_A value = 3 and partitioner = hash(3) % 6 = 3 then stored in <b>partition 3</b> -> sort based on COL_C
COL_A value = 8 and partitioner = hash(8) % 6 = 2 then stored in <b>partition 2</b> -> sort based on COL_C
COL_A value = 9 and partitioner = hash(9) % 6 = 3 then stored in <b>partition 3</b> -> sort based on COL_C
COL_A value = 13 and partitioner = hash(13) % 6 = 1 then stored in <b>partition 1</b> -> sort based on COL_C

-- PARTITION 0 --

Empty

-- PARTITION 1 --

COL_A   COL_B   COL_C
---------------------
  1     test2     1
  1     test1     2
  13    test10    10
  
-- PARTITION 2 --

COL_A   COL_B   COL_C
---------------------
  2     test3     5
  8     test6     6
  
-- PARTITION 3 --

COL_A   COL_B   COL_C
---------------------
  3     test4     0
  3     test5     7
  9     test8     3
  9     test7     8
  9     test9     9
  
-- PARTITION 4 --

Empty

-- PARTITION 5 --

Empty
</pre>

As you can see, the result is not what we expected before. We expected that each partition should contain the same value for <b>COL_A</b>.

Now let's see if we're using the 2nd approach.

<pre>
<b>API level: RDD</b>

<b>Method:</b> <b>repartitionAndsortWithinPartitions</b>

<b>Partitioner:</b> customized. See the code section below.
</pre>

<b>CODE:</b>

```python

distinct_col_a_data_df = df.select('COL_A').distinct()
num_partitions = distinct_col_a_data_df.count()
distinct_col_a_data = [row['COL_A'] for row in distinct_col_a_data_df.collect()]

# create paired RDDs
df_rdd = df.rdd.keyBy(lambda row: (row['COL_A'], row['COL_C']))

def customized_partitioner(distinct_col_a_data: [int]) -> int:
	def partitioner_(rdd_key: (int, int)) -> int:
		return distinct_col_a_data.index(rdd_key[0])
	return partitioner_

df_rdd = df_rdd.repartitionAndSortWithinPartitions(numPartitions=num_partitions,
						   partitionFunc=customized_partitioner(distinct_col_a_data),
						   ascending=True, keyfunc=lambda rdd_key: rdd_key[1])
```

<pre>
<b>Result:</b>

distinct_col_a_data = [1, 2, 3, 8, 9, 13]

COL_A value = 1 and partitioner = distinct_col_a_data.index(1) = 0 then stored in <b>partition 0</b> -> sort based on COL_C
COL_A value = 2 and partitioner = distinct_col_a_data.index(2) = 1 then stored in <b>partition 1</b> -> sort based on COL_C
COL_A value = 3 and partitioner = distinct_col_a_data.index(3) = 2 then stored in <b>partition 2</b> -> sort based on COL_C
COL_A value = 8 and partitioner = distinct_col_a_data.index(8) = 3 then stored in <b>partition 3</b> -> sort based on COL_C
COL_A value = 9 and partitioner = distinct_col_a_data.index(9) = 4 then stored in <b>partition 4</b> -> sort based on COL_C
COL_A value = 13 and partitioner = distinct_col_a_data.index(13) = 5 then stored in <b>partition 5</b> -> sort based on COL_C

-- PARTITION 0 --

COL_A   COL_B   COL_C
---------------------
  1     test2     1
  1     test1     2
  
-- PARTITION 1 --

COL_A   COL_B   COL_C
---------------------
  2     test3     5
  
-- PARTITION 2 --

COL_A   COL_B   COL_C
---------------------
  3     test4     0
  3     test5     7
 
-- PARTITION 3 --

COL_A   COL_B   COL_C
---------------------
  8     test6     6
  
-- PARTITION 4 --

COL_A   COL_B   COL_C
---------------------
  9     test8     3
  9     test7     8
  9     test9     9
  
-- PARTITION 5 --

COL_A   COL_B   COL_C
---------------------
  13    test10    10
</pre>

Well yeah, that's the result we expected!

Several remarks here. The primary objective in using customized partitioner is that we want to make sure that each partition contains the same data from <b>COL_A</b>. Therefore, one way to do that is by using the data index from <b>distinct_col_a_data</b>. You can use another approach but I think the one I'm using here is the most straightforward and the easiest to implement.

Another remark is, as you can see, I created paired RDDs which means that our RDDs is paired with keys. In this case, the keys are <b>COL_A</b> and <b>COL_C</b>. Precisely, the resulting paired RDDs should be something like this:

<ul>
    <li>(1, 2), (Row(COL_A)=1, Row(COL_B)=test_1, Row(COL_C)=2)</li>
    <li>(1, 1), (Row(COL_A)=1, Row(COL_B)=test_2, Row(COL_C)=1)</li>
    <li>(2, 5), (Row(COL_A)=2, Row(COL_B)=test_3, Row(COL_C)=5)</li>
    <li>(3, 0), (Row(COL_A)=3, Row(COL_B)=test_4, Row(COL_C)=0)</li>
    <li>(3, 7), (Row(COL_A)=3, Row(COL_B)=test_5, Row(COL_C)=7)</li>
    <li>(8, 6), (Row(COL_A)=8, Row(COL_B)=test_6, Row(COL_C)=6)</li>
    <li>(9, 3), (Row(COL_A)=9, Row(COL_B)=test_8, Row(COL_C)=3)</li>
    <li>(9, 8), (Row(COL_A)=9, Row(COL_B)=test_7, Row(COL_C)=8)</li>
    <li>(9, 9), (Row(COL_A)=9, Row(COL_B)=test_9, Row(COL_C)=9)</li>
    <li>(13, 10), (Row(COL_A)=13, Row(COL_B)=test_10, Row(COL_C)=10)</li>
</ul>

To sort the dataframe, we provide Spark with the key on which the sorting will be based. In this case, we specify the key through <i>keyfunc=lambda rdd_key: rdd_key[1]</i> stating that we're using <b>COL_C</b> as the sorting key.

Yeah, I think using this 2nd approach needs more time than using the 1st approach as we need to specify everything so that Spark knows how to get the task done. But at least we got what we want, right?

Personally I was wondering why such a feature (customizing partitioner) hasn't been implemented in DF/DT. Tag <b>Matei Zaharia and friends</b>, the original authors of Spark. Or <b>awesome engineers at Databricks</b> :).

Perhaps they want to keep the basic principles stating that the developers don't need to specify the steps to get a task done when using DF/DT. FYI, customizing a partitioner function means that we're telling Spark how to do the repartitioning using our own way which I think might violate the principle.

Lots of words in a single post. Thank you for reading :)
