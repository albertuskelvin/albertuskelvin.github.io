---
title: 'A Brief Report on GroupBy Operation After Dataframe Repartitioning'
date: 2019-04-19
permalink: /posts/2019/04/spark-groupby-consistency-01/
tags:
  - spark
  - groupby
  - dataframe
  - data sorting
  - repartitioning
---

A few days ago I did a little exploration on Spark's groupBy behavior. Precisely, I wanted to see whether the order of the data was still preserved when applying groupBy on a repartitioned dataframe.

Suppose we have a dataframe (df).

<pre>  
+---+-------+-----+-------+
| ID|COL_A  |COL_B|COL_C  |
+---+-------+-----+-------+
|  0|      A|  1|        0|
|  1|      A|  1|        1|
|  2|      A|  1|        2|
|  3|      B|  1|        3|
|  4|      A|  1|        4|
|  5|      B|  1|        5|
|  6|      C|  1|        6|
|  7|      C|  1|        7|
|  8|      D|  1|        8|
|  9|      D|  1|        9|
| 10|      B|  2|       10|
| 11|      A|  2|       11|
| 12|      B|  2|       12|
| 13|      C|  2|       13|
| 14|      X|  3|       14|
| 15|      X|  3|       15|
| 16|      X|  3|       16|
| 17|      X|  3|       17|
| 18|      Z|  3|       18|
| 19|      Z|  3|       19|
| 20|      Y|  3|       20|
| 21|      Y|  3|       21|
| 22|      Y|  3|       22|
| 23|      Y|  3|       23|
| 24|      A|  1|       11|
| 25|      C|  4|       11|
| 26|      X|  2|       17|
| 27|      C|  3|       11|
| 28|      A|  3|       44|
| 29|      C|  3|       11|
| 30|      X|  1|       13|
| 31|      C|  3|       11|
| 32|      B|  1|       29|
| 33|      C|  5|       11|
| 34|      B|  3|       30|
| 35|      Y|  5|       21|
| 36|      X|  1|       21|
| 37|      C|  3|       22|
| 38|      Z|  2|       10|
| 39|      C|  2|       15|
| 40|      B|  2|       19|
| 41|      C|  2|       14|
| 42|      Z|  1|       14|
| 43|      X|  3|       13|
| 44|      X|  3|       19|
| 45|      A|  3|       14|
| 46|      Z|  1|       18|
| 47|      C|  3|       12|
| 48|      Y|  1|       20|
| 49|      Y|  3|       11|
| 50|      Y|  3|       22|
| 51|      A|  3|       25|
+---+-------+---+---------+
</pre>

FYI, according to this <a href="https://bzhangusc.wordpress.com/2015/05/28/groupby-on-dataframe-is-not-the-groupby-on-rdd/">post</a>, groupBy on DataFrame is not the same as the groupBy on RDD. groupBy on DataFrame preserves the order of the data in the original dataframe, while groupBY on RDD doesn't.

Using the above dataframe (df), if we do a groupBy operation based on **COL_A** and only take rows with <i>COL_A = 'A'</i>, then the result (let's ignore the aggregation for now) should be similar to:

<pre>
+---+-------+-----+-------+
| ID|COL_A  |COL_B|COL_C  |
+---+-------+-----+-------+
|  0|      A|  1|        0|
|  1|      A|  1|        1|
|  2|      A|  1|        2|
|  4|      A|  1|        4|
| 11|      A|  2|       11|
| 24|      A|  1|       11|
| 28|      A|  3|       44|
| 45|      A|  3|       14|
| 51|      A|  3|       25|
+---+-------+---+---------+
</pre>

As you can see, the order of the data is preserved which means that the rows were collected from the first till the last row sequentially (row's index was incremented by 1). Based on this result, I'm pretty sure that if we sort a certain column first, let's say **COL_C**, then do a groupBy operation using **COL_A**, the resulting group would still be sorted according to **COL_C**. Please confirm if you found different result.

Now let's see what happens if we repartition and sort the dataframe simultaneously first, then execute the groupBy. Here's the code.

<pre>
def _partitioner(distinct_col_b: [str]) -> int:
    def partitioner_(rdd_key: (str, int)) -> int:
        return distinct_col_b.index(rdd_key[0])
    return partitioner_

# repartition and sort the dataframe
rdd = df.rdd.keyBy(lambda row: (row['COL_B'], row['COL_C']))
rdd = rdd.repartitionAndSortWithinPartitions(numPartitions=num_of_distinct_col_b,
                                             partitionFunc=_partitioner(distinct_col_b),
                                             ascending=True, keyfunc=lambda rdd_key: rdd_key[1])

# group by COL_A
df = rdd.map(lambda row: row[1]).toDF(['ID', 'COL_A', 'COL_B', 'COL_C'])

df_gb = df.groupBy('COL_A').agg(
                                F.sort_array(F.collect_list('COL_C')).alias('sorted_COL_C'),
                                F.collect_list('COL_C').alias('list_COL_C'),
                                F.first('ID'),
                                F.last('ID')
                            )
df_gb.show(100)
</pre>

And show several outputs using this code.

<pre>
# show the partitions' content
for index, partition in enumerate(rdd.glom().collect()):
    print('Partition {}'.format(index))
    
    for i in partition:
        print(i)

# show the columns' value of df_gb
sorted_col_c = df_gb.select('sorted_COL_C').collect()
print('sorted_COL_C')
for i in sorted_col_c:
    print(i)

list_col_c = df_gb.select('list_COL_C').collect()
print('list_COL_C')
for i in list_col_c:
    print(i)
</pre>

The repartitioning code gives the result as shown below.

<pre>
Partition 0
(('3', 11), Row(ID=27, COL_A='C', COL_B='3', COL_C=11))
(('3', 11), Row(ID=29, COL_A='C', COL_B='3', COL_C=11))
(('3', 11), Row(ID=31, COL_A='C', COL_B='3', COL_C=11))
(('3', 11), Row(ID=49, COL_A='Y', COL_B='3', COL_C=11))
(('3', 12), Row(ID=47, COL_A='C', COL_B='3', COL_C=12))
(('3', 13), Row(ID=43, COL_A='X', COL_B='3', COL_C=13))
(('3', 14), Row(ID=14, COL_A='X', COL_B='3', COL_C=14))
(('3', 14), Row(ID=45, COL_A='A', COL_B='3', COL_C=14))
(('3', 15), Row(ID=15, COL_A='X', COL_B='3', COL_C=15))
(('3', 16), Row(ID=16, COL_A='X', COL_B='3', COL_C=16))
(('3', 17), Row(ID=17, COL_A='X', COL_B='3', COL_C=17))
(('3', 18), Row(ID=18, COL_A='Z', COL_B='3', COL_C=18))
(('3', 19), Row(ID=19, COL_A='Z', COL_B='3', COL_C=19))
(('3', 19), Row(ID=44, COL_A='X', COL_B='3', COL_C=19))
(('3', 20), Row(ID=20, COL_A='Y', COL_B='3', COL_C=20))
(('3', 21), Row(ID=21, COL_A='Y', COL_B='3', COL_C=21))
(('3', 22), Row(ID=22, COL_A='Y', COL_B='3', COL_C=22))
(('3', 22), Row(ID=37, COL_A='C', COL_B='3', COL_C=22))
(('3', 22), Row(ID=50, COL_A='Y', COL_B='3', COL_C=22))
(('3', 23), Row(ID=23, COL_A='Y', COL_B='3', COL_C=23))
(('3', 25), Row(ID=51, COL_A='A', COL_B='3', COL_C=25))
(('3', 30), Row(ID=34, COL_A='B', COL_B='3', COL_C=30))
(('3', 44), Row(ID=28, COL_A='A', COL_B='3', COL_C=44))

Partition 1
(('5', 11), Row(ID=33, COL_A='C', COL_B='5', COL_C=11))
(('5', 21), Row(ID=35, COL_A='Y', COL_B='5', COL_C=21))

Partition 2
(('1', 0), Row(ID=0, COL_A='A', COL_B='1', COL_C=0))
(('1', 1), Row(ID=1, COL_A='A', COL_B='1', COL_C=1))
(('1', 2), Row(ID=2, COL_A='A', COL_B='1', COL_C=2))
(('1', 3), Row(ID=3, COL_A='B', COL_B='1', COL_C=3))
(('1', 4), Row(ID=4, COL_A='A', COL_B='1', COL_C=4))
(('1', 5), Row(ID=5, COL_A='B', COL_B='1', COL_C=5))
(('1', 6), Row(ID=6, COL_A='C', COL_B='1', COL_C=6))
(('1', 7), Row(ID=7, COL_A='C', COL_B='1', COL_C=7))
(('1', 8), Row(ID=8, COL_A='D', COL_B='1', COL_C=8))
(('1', 9), Row(ID=9, COL_A='D', COL_B='1', COL_C=9))
(('1', 11), Row(ID=24, COL_A='A', COL_B='1', COL_C=11))
(('1', 13), Row(ID=30, COL_A='X', COL_B='1', COL_C=13))
(('1', 14), Row(ID=42, COL_A='Z', COL_B='1', COL_C=14))
(('1', 18), Row(ID=46, COL_A='Z', COL_B='1', COL_C=18))
(('1', 20), Row(ID=48, COL_A='Y', COL_B='1', COL_C=20))
(('1', 21), Row(ID=36, COL_A='X', COL_B='1', COL_C=21))
(('1', 29), Row(ID=32, COL_A='B', COL_B='1', COL_C=29))

Partition 3
(('4', 11), Row(ID=25, COL_A='C', COL_B='4', COL_C=11))

Partition 4
(('2', 10), Row(ID=10, COL_A='B', COL_B='2', COL_C=10))
(('2', 10), Row(ID=38, COL_A='Z', COL_B='2', COL_C=10))
(('2', 11), Row(ID=11, COL_A='A', COL_B='2', COL_C=11))
(('2', 12), Row(ID=12, COL_A='B', COL_B='2', COL_C=12))
(('2', 13), Row(ID=13, COL_A='C', COL_B='2', COL_C=13))
(('2', 14), Row(ID=41, COL_A='C', COL_B='2', COL_C=14))
(('2', 15), Row(ID=39, COL_A='C', COL_B='2', COL_C=15))
(('2', 17), Row(ID=26, COL_A='X', COL_B='2', COL_C=17))
(('2', 19), Row(ID=40, COL_A='B', COL_B='2', COL_C=19))
</pre>

The groupBy and aggregation code give the result as shown below.

<pre>
+-------+--------------------+----------------------+---------------+--------------+
|COL_A  |  sorted_COL_C      |list_COL_C            |first(ID, true)|last(ID, true)|
+-------+--------------------+----------------------+---------------+--------------+
|      B|[3, 5, 10, 12, 19...|  [30, 3, 5, 29, 10...|             34|            40|
|      Y|[11, 20, 20, 21, ...|  [11, 20, 21, 22, ...|             49|            48|
|      D|              [8, 9]|                [8, 9]|              8|             9|
|      C|[6, 7, 11, 11, 11...|  [11, 11, 11, 12, ...|             27|            39|
|      Z|[10, 14, 18, 18, 19]|  [18, 19, 14, 18, 10]|             18|            38|
|      A|[0, 1, 2, 4, 11, ...|  [14, 25, 44, 0, 1...|             45|            11|
|      X|[13, 13, 14, 15, ...|  [13, 14, 15, 16, ...|             43|            26|
+-------+--------------------+----------------------+---------------+--------------+

sorted_COL_C
Row(sorted_COL_C=[3, 5, 10, 12, 19, 29, 30])
Row(sorted_COL_C=[11, 20, 20, 21, 21, 22, 22, 23])
Row(sorted_COL_C=[8, 9])
Row(sorted_COL_C=[6, 7, 11, 11, 11, 11, 11, 12, 13, 14, 15, 22])
Row(sorted_COL_C=[10, 14, 18, 18, 19])
Row(sorted_COL_C=[0, 1, 2, 4, 11, 11, 14, 25, 44])
Row(sorted_COL_C=[13, 13, 14, 15, 16, 17, 17, 19, 21])

list_COL_C
Row(list_COL_C=[30, 3, 5, 29, 10, 12, 19])
Row(list_COL_C=[11, 20, 21, 22, 22, 23, 21, 20])
Row(list_COL_C=[8, 9])
Row(list_COL_C=[11, 11, 11, 12, 22, 11, 6, 7, 11, 13, 14, 15])
Row(list_COL_C=[18, 19, 14, 18, 10])
Row(list_COL_C=[14, 25, 44, 0, 1, 2, 4, 11, 11])
Row(list_COL_C=[13, 14, 15, 16, 17, 19, 13, 21, 17])
</pre>

For each element in <b>COL_A</b>, we can see that the elements in <b>COL_C</b> are not ordered. In addition, the groupBy operation doesn't preserve the order from the original dataframe. At first I thought that when the dataframe in each partition was sorted according to <b>COL_C</b>, then groupBy operation should collect the rows with the same value for <b>COL_A</b> as well as maintain the order of the rows using <b>COL_C</b> as the sorting key. In other words, the result of <i>list_COL_C</i> should be the same as <i>sorted_COL_C</i>. At this first hypothesis I was wrong.

Moreover, I also thought that if the first hypothesis failed, then the second hypothesis would be that the elements of the aggregated column (list_COL_C) should have the same order as the original dataframe (preserved order). In other words, if the order of <b>COL_C</b> for <i>COL_A = 'B'</i> in the original dataframe is [3, 5, 10, 12, 29, 30, 19], then the result of <i>list_COL_C</i> should be [3, 5, 10, 12, 29, 30, 19] too. While in fact, it returned [30, 3, 5, 29, 10, 12, 19]. At this second hypothesis I was wrong again.

The same thing also applies to <b>F.first</b> and <b>F.last</b> operation. I thought that each function would return the first and last element in the group respectively where those two elements position is the same as their position in the original dataframe. For instance, <i>COL_A = 'Z'</i> should return 18 and 46 as the first and last ID respectively. However, after being repartitioned, it returned 18 (from partition 0) and 38 (from partition 4) as the first and last ID respectively.

Based on the above brief explanation, I think the root cause of this problem lies on the Spark's behavior that seems to retrieve elements starting from the first up to the last partition sequentially. For instance, when Spark does a groupBy operation based on <b>COL_A</b> and converts the rows of <b>COL_C</b> to a list, we may find that Spark searches for <b>COL_A</b>'s elements from partition 0 till 4 with the same value and then takes the <b>COL_C</b> elements. Presuming that Spark is searching for <i>COL_A = 'B'</i>, then the followings are what Spark retrieved at the end of the process:
<ul>
  <li>Partition 0 -> (('3', 30), Row(ID=34, COL_A='B', COL_B='3', COL_C=30))</li>
  <li>Partition 2 -> (('1', 3), Row(ID=3, COL_A='B', COL_B='1', COL_C=3)), (('1', 5), Row(ID=5, COL_A='B', COL_B='1', COL_C=5)), (('1', 29), Row(ID=32, COL_A='B', COL_B='1', COL_C=29))</li>
  <li>Partition 4 -> (('2', 10), Row(ID=10, COL_A='B', COL_B='2', COL_C=10)), (('2', 12), Row(ID=12, COL_A='B', COL_B='2', COL_C=12)), (('2', 19), Row(ID=40, COL_A='B', COL_B='2', COL_C=19))</li>
</ul>

Therefore, since Spark refers to the dataframe in each partition and knowing that there's a possibility that the result of <i>new_partition 0 + new_partition 1 + new_partition 2 + ... + new_partition n</i> might be different from <i>original_partition_0 + original_partition_1 + ... + original_partition_n</i>, we can say that doing a groupBy operation after repartitioning <b>doesn't preserve</b> the order of the data in the original dataframe.

NB: <i>new_partitions</i> refers to the partitions created after the <b>repartition</b> method is called. Meanwhile, <i>original_partitions</i> refers to the default partitions created by Spark when we define a dataframe.

So, be careful when playing with this chain of operations.

Thanks for reading.
