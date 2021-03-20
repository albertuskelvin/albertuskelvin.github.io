---
title: 'Effects of Shuffling on RDDs and Dataframes Partitioning'
date: 2019-05-21
permalink: /posts/2019/05/shuffle-and-partitions/
tags:
  - spark
  - dataframe
  - rdd
  - shuffle
  - partitions
---

In Spark, data shuffling simply means data movement. In a single machine with multiple partitions, data shuffling means that data move from one partition to another partition. Meanwhile, in multiple machines, data shuffling can have two kinds of work. The first one is data move from one partition (A) to another partition (B) within the same machine (M1), while the second one is data move from partition B to another partition (C) within different machine (M2). Data in partition C might be moved to another partition within different machine again (M3). 

Recently I was doing a simple exploration to examine the effects of data shuffling on RDDs and Dataframes partitioning behavior. Let's jump into the code I used in my exploration.

```python
df = spark.createDataFrame([('A0',0,'C0'), ('A0',1,'C1'), ('A0',2,'C2'), ('A0',3,'C3'),
                            ('A1',4,'C4'), ('A1',5,'C5'), ('A1',6,'C6'), ('A1',7,'C7'),
                            ('A2',4,'C4'), ('A2',5,'C5'), ('A2',6,'C6'), ('A2',7,'C7'),
                            ('A3',4,'C4'), ('A3',5,'C5'), ('A3',6,'C6'), ('A3',7,'C7'),
                            ('A4',4,'C4'), ('A4',5,'C5'), ('A4',6,'C6'), ('A4',7,'C7'),
                            ('A5',4,'C4'), ('A5',5,'C5'), ('A5',6,'C6'), ('A5',7,'C7'),
                            ('A6',4,'C4'), ('A6',5,'C5'), ('A6',6,'C6'), ('A6',7,'C7'),
                            ('A7',4,'C4'), ('A7',5,'C5'), ('A7',6,'C6'), ('A7',7,'C7'),
                            ('A8',4,'C4'), ('A8',5,'C5'), ('A8',6,'C6'), ('A8',7,'C7'),
                            ('A9',4,'C4'), ('A9',5,'C5'), ('A9',6,'C6'), ('A9',7,'C7'),
                            ('A10',4,'C4'), ('A10',5,'C5'), ('A10',6,'C6'), ('A10',7,'C7'),
                            ('A11',4,'C4'), ('A11',5,'C5'), ('A11',6,'C6'), ('A11',7,'C7')], 
                            ['COL_A', 'COL_B', 'COL_C'])

num_of_distinct_col_a = df.select('COL_A').distinct().count()
df = df.repartition(num_of_distinct_col_a)

# RDD
paired_rdd = df.rdd.map(lambda x: (1, x)) 
print('NUM OF RDD PARTITIONS BEFORE SHUFFLING: {}\n'.format(str(paired_rdd.getNumPartitions())))
for index, partition in enumerate(paired_rdd.glom().collect()):
  print('Partition {}'.format(index))
  print(partition)

grouped_rdd = paired_rdd.groupByKey()

print('\n')
print('NUM OF RDD PARTITIONS AFTER SHUFFLING: {}\n'.format(str(grouped_rdd.getNumPartitions())))
for index, partition in enumerate(grouped_rdd.glom().collect()):
  print('Partition {}'.format(index))
  print(partition)


# DATAFRAME
print('\n')
print('NUM OF DATAFRAME PARTITIONS BEFORE SHUFFLING: {}'.format(df.rdd.getNumPartitions()))
for index, partition in enumerate(df.rdd.glom().collect()):
  print('Partition {}'.format(index))
  print(partition)

grouped_df = df.groupby('COL_A').agg(F.sum('COL_B'))

print('\n')
print('NUM OF DATAFRAME PARTITIONS AFTER SHUFFLING: {}'.format(grouped_df.rdd.getNumPartitions()))
for index, partition in enumerate(grouped_df.rdd.glom().collect()):
  print('Partition {}'.format(index))
  print(partition)
```

Basically, what I did was examine whether processes that require data shuffling affected the number of partitions of the RDDs or Dataframes. Here's what I got.

<pre>
NUM OF RDD PARTITIONS BEFORE SHUFFLING: 12

Partition 0
[(1, Row(COL_A='A6', COL_B=6, COL_C='C6')), (1, Row(COL_A='A4', COL_B=6, COL_C='C6')), (1, Row(COL_A='A10', COL_B=4, COL_C='C4')), (1, Row(COL_A='A0', COL_B=3, COL_C='C3'))]
Partition 1
[(1, Row(COL_A='A5', COL_B=4, COL_C='C4')), (1, Row(COL_A='A9', COL_B=5, COL_C='C5')), (1, Row(COL_A='A9', COL_B=4, COL_C='C4')), (1, Row(COL_A='A1', COL_B=4, COL_C='C4'))]
Partition 2
[(1, Row(COL_A='A1', COL_B=6, COL_C='C6')), (1, Row(COL_A='A4', COL_B=5, COL_C='C5')), (1, Row(COL_A='A1', COL_B=7, COL_C='C7')), (1, Row(COL_A='A3', COL_B=5, COL_C='C5'))]
Partition 3
[(1, Row(COL_A='A7', COL_B=7, COL_C='C7')), (1, Row(COL_A='A7', COL_B=4, COL_C='C4')), (1, Row(COL_A='A2', COL_B=7, COL_C='C7')), (1, Row(COL_A='A5', COL_B=7, COL_C='C7'))]
Partition 4
[(1, Row(COL_A='A11', COL_B=5, COL_C='C5')), (1, Row(COL_A='A1', COL_B=5, COL_C='C5')), (1, Row(COL_A='A5', COL_B=6, COL_C='C6')), (1, Row(COL_A='A5', COL_B=5, COL_C='C5'))]
Partition 5
[(1, Row(COL_A='A8', COL_B=6, COL_C='C6')), (1, Row(COL_A='A3', COL_B=6, COL_C='C6')), (1, Row(COL_A='A4', COL_B=4, COL_C='C4')), (1, Row(COL_A='A4', COL_B=7, COL_C='C7'))]
Partition 6
[(1, Row(COL_A='A8', COL_B=7, COL_C='C7')), (1, Row(COL_A='A0', COL_B=1, COL_C='C1')), (1, Row(COL_A='A8', COL_B=4, COL_C='C4')), (1, Row(COL_A='A7', COL_B=6, COL_C='C6'))]
Partition 7
[(1, Row(COL_A='A10', COL_B=7, COL_C='C7')), (1, Row(COL_A='A6', COL_B=4, COL_C='C4')), (1, Row(COL_A='A6', COL_B=7, COL_C='C7')), (1, Row(COL_A='A3', COL_B=7, COL_C='C7'))]
Partition 8
[(1, Row(COL_A='A10', COL_B=5, COL_C='C5')), (1, Row(COL_A='A9', COL_B=7, COL_C='C7')), (1, Row(COL_A='A2', COL_B=5, COL_C='C5')), (1, Row(COL_A='A7', COL_B=5, COL_C='C5'))]
Partition 9
[(1, Row(COL_A='A11', COL_B=4, COL_C='C4')), (1, Row(COL_A='A10', COL_B=6, COL_C='C6')), (1, Row(COL_A='A11', COL_B=7, COL_C='C7')), (1, Row(COL_A='A0', COL_B=2, COL_C='C2'))]
Partition 10
[(1, Row(COL_A='A2', COL_B=6, COL_C='C6')), (1, Row(COL_A='A8', COL_B=5, COL_C='C5')), (1, Row(COL_A='A9', COL_B=6, COL_C='C6')), (1, Row(COL_A='A6', COL_B=5, COL_C='C5'))]
Partition 11
[(1, Row(COL_A='A3', COL_B=4, COL_C='C4')), (1, Row(COL_A='A2', COL_B=4, COL_C='C4')), (1, Row(COL_A='A0', COL_B=0, COL_C='C0')), (1, Row(COL_A='A11', COL_B=6, COL_C='C6'))]


NUM OF RDD PARTITIONS AFTER SHUFFLING: 12

Partition 0
[]
Partition 1
[(1, <pyspark.resultiterable.ResultIterable object at 0x11b1e84a8>)]
Partition 2
[]
Partition 3
[]
Partition 4
[]
Partition 5
[]
Partition 6
[]
Partition 7
[]
Partition 8
[]
Partition 9
[]
Partition 10
[]
Partition 11
[]


NUM OF DATAFRAME PARTITIONS BEFORE SHUFFLING: 12

Partition 0
[Row(COL_A='A6', COL_B=6, COL_C='C6'), Row(COL_A='A4', COL_B=6, COL_C='C6'), Row(COL_A='A10', COL_B=4, COL_C='C4'), Row(COL_A='A0', COL_B=3, COL_C='C3')]
Partition 1
[Row(COL_A='A5', COL_B=4, COL_C='C4'), Row(COL_A='A9', COL_B=5, COL_C='C5'), Row(COL_A='A9', COL_B=4, COL_C='C4'), Row(COL_A='A1', COL_B=4, COL_C='C4')]
Partition 2
[Row(COL_A='A1', COL_B=6, COL_C='C6'), Row(COL_A='A4', COL_B=5, COL_C='C5'), Row(COL_A='A1', COL_B=7, COL_C='C7'), Row(COL_A='A3', COL_B=5, COL_C='C5')]
Partition 3
[Row(COL_A='A7', COL_B=7, COL_C='C7'), Row(COL_A='A7', COL_B=4, COL_C='C4'), Row(COL_A='A2', COL_B=7, COL_C='C7'), Row(COL_A='A5', COL_B=7, COL_C='C7')]
Partition 4
[Row(COL_A='A11', COL_B=5, COL_C='C5'), Row(COL_A='A1', COL_B=5, COL_C='C5'), Row(COL_A='A5', COL_B=6, COL_C='C6'), Row(COL_A='A5', COL_B=5, COL_C='C5')]
Partition 5
[Row(COL_A='A8', COL_B=6, COL_C='C6'), Row(COL_A='A3', COL_B=6, COL_C='C6'), Row(COL_A='A4', COL_B=4, COL_C='C4'), Row(COL_A='A4', COL_B=7, COL_C='C7')]
Partition 6
[Row(COL_A='A8', COL_B=7, COL_C='C7'), Row(COL_A='A0', COL_B=1, COL_C='C1'), Row(COL_A='A8', COL_B=4, COL_C='C4'), Row(COL_A='A7', COL_B=6, COL_C='C6')]
Partition 7
[Row(COL_A='A10', COL_B=7, COL_C='C7'), Row(COL_A='A6', COL_B=4, COL_C='C4'), Row(COL_A='A6', COL_B=7, COL_C='C7'), Row(COL_A='A3', COL_B=7, COL_C='C7')]
Partition 8
[Row(COL_A='A10', COL_B=5, COL_C='C5'), Row(COL_A='A9', COL_B=7, COL_C='C7'), Row(COL_A='A2', COL_B=5, COL_C='C5'), Row(COL_A='A7', COL_B=5, COL_C='C5')]
Partition 9
[Row(COL_A='A11', COL_B=4, COL_C='C4'), Row(COL_A='A10', COL_B=6, COL_C='C6'), Row(COL_A='A11', COL_B=7, COL_C='C7'), Row(COL_A='A0', COL_B=2, COL_C='C2')]
Partition 10
[Row(COL_A='A2', COL_B=6, COL_C='C6'), Row(COL_A='A8', COL_B=5, COL_C='C5'), Row(COL_A='A9', COL_B=6, COL_C='C6'), Row(COL_A='A6', COL_B=5, COL_C='C5')]
Partition 11
[Row(COL_A='A3', COL_B=4, COL_C='C4'), Row(COL_A='A2', COL_B=4, COL_C='C4'), Row(COL_A='A0', COL_B=0, COL_C='C0'), Row(COL_A='A11', COL_B=6, COL_C='C6')]


NUM OF DATAFRAME PARTITIONS AFTER SHUFFLING: 200
Partition 0 - 4
[]
Partition 5
[Row(COL_A='A9', sum(COL_B)=22)]
Partition 6 - 15
[]
Partition 16
[Row(COL_A='A6', sum(COL_B)=22)]
Partition 17 - 31
[]
Partition 32
[Row(COL_A='A2', sum(COL_B)=22), Row(COL_A='A8', sum(COL_B)=22)]
Partition 33 - 64
[]
Partition 65
[Row(COL_A='A11', sum(COL_B)=22)]
Partition 66 - 104
[]
Partition 105
[Row(COL_A='A10', sum(COL_B)=22)]
Partition 106 - 113
[]
Partition 114
[Row(COL_A='A4', sum(COL_B)=22)]
Partition 115 - 135
[]
Partition 136
[Row(COL_A='A0', sum(COL_B)=6)]
Partition 137 - 142
[]
Partition 143
[Row(COL_A='A7', sum(COL_B)=22)]
Partition 144 - 156
[]
Partition 157
[Row(COL_A='A5', sum(COL_B)=22)]
Partition 158 - 187
[]
Partition 188
[Row(COL_A='A3', sum(COL_B)=22)]
Partition 189 - 196
[]
Partition 197
[Row(COL_A='A1', sum(COL_B)=22)]
Partition 198 - 199
[]
</pre>

As you can see, the number of partitions in `paired_rdd` and `grouped_rdd` is the same. It means that in RDDs, shuffling doesn't affect its partitioning. Meanwhile, after applying data shuffling to a dataframe, we can see that the number of partitions is different from the original dataframe (changes from 12 to 200 partitions). It seems that this data shuffling creates a new repartitioning process based on certain columns, something like `new_df = df.repartition(COLUMNS)`. Just FYI, when repartitioning a dataframe based on columns, Spark will create partitions using the default number of partitions, which is 200. That's why the resulting dataframe has 200 partitions after a data shuffling has been aplied to it.

However, we can prevent such a behavior by simply avoiding data shuffling. Precisely, we can still use processes that require data shuffling like we did above but the data only move within its partition. So there'll be no data moving from a partition to another partition. To accomplish that, we simply modify the dataframe repartitioning code. Here's a simple example.

```python
df = df.repartition(num_of_distinct_col_a, 'COL_A')
```

As you can see, I specify two parameters for the repartitioning method. By doing so, we require Spark to repartition the dataframe based on `COL_A` with an exact number of partitions (`num_of_distinct_col_a`) instead of the default number of partitions.

Here's what I got.

<pre>
NUM OF DATAFRAME PARTITIONS BEFORE SHUFFLING: 12

Partition 0
[Row(COL_A='A0', COL_B=0, COL_C='C0'), Row(COL_A='A0', COL_B=1, COL_C='C1'), Row(COL_A='A0', COL_B=2, COL_C='C2'), Row(COL_A='A0', COL_B=3, COL_C='C3')]
Partition 1
[Row(COL_A='A1', COL_B=4, COL_C='C4'), Row(COL_A='A1', COL_B=5, COL_C='C5'), Row(COL_A='A1', COL_B=6, COL_C='C6'), Row(COL_A='A1', COL_B=7, COL_C='C7'), Row(COL_A='A5', COL_B=4, COL_C='C4'), Row(COL_A='A5', COL_B=5, COL_C='C5'), Row(COL_A='A5', COL_B=6, COL_C='C6'), Row(COL_A='A5', COL_B=7, COL_C='C7')]
Partition 2
[Row(COL_A='A4', COL_B=4, COL_C='C4'), Row(COL_A='A4', COL_B=5, COL_C='C5'), Row(COL_A='A4', COL_B=6, COL_C='C6'), Row(COL_A='A4', COL_B=7, COL_C='C7')]
Partition 3
[Row(COL_A='A7', COL_B=4, COL_C='C4'), Row(COL_A='A7', COL_B=5, COL_C='C5'), Row(COL_A='A7', COL_B=6, COL_C='C6'), Row(COL_A='A7', COL_B=7, COL_C='C7')]
Partition 4
[Row(COL_A='A3', COL_B=4, COL_C='C4'), Row(COL_A='A3', COL_B=5, COL_C='C5'), Row(COL_A='A3', COL_B=6, COL_C='C6'), Row(COL_A='A3', COL_B=7, COL_C='C7'), Row(COL_A='A6', COL_B=4, COL_C='C4'), Row(COL_A='A6', COL_B=5, COL_C='C5'), Row(COL_A='A6', COL_B=6, COL_C='C6'), Row(COL_A='A6', COL_B=7, COL_C='C7')]
Partition 5
[Row(COL_A='A9', COL_B=4, COL_C='C4'), Row(COL_A='A9', COL_B=5, COL_C='C5'), Row(COL_A='A9', COL_B=6, COL_C='C6'), Row(COL_A='A9', COL_B=7, COL_C='C7'), Row(COL_A='A10', COL_B=4, COL_C='C4'), Row(COL_A='A10', COL_B=5, COL_C='C5'), Row(COL_A='A10', COL_B=6, COL_C='C6'), Row(COL_A='A10', COL_B=7, COL_C='C7')]
Partition 6
[]
Partition 7
[]
Partition 8
[Row(COL_A='A2', COL_B=4, COL_C='C4'), Row(COL_A='A2', COL_B=5, COL_C='C5'), Row(COL_A='A2', COL_B=6, COL_C='C6'), Row(COL_A='A2', COL_B=7, COL_C='C7'), Row(COL_A='A8', COL_B=4, COL_C='C4'), Row(COL_A='A8', COL_B=5, COL_C='C5'), Row(COL_A='A8', COL_B=6, COL_C='C6'), Row(COL_A='A8', COL_B=7, COL_C='C7')]
Partition 9
[Row(COL_A='A11', COL_B=4, COL_C='C4'), Row(COL_A='A11', COL_B=5, COL_C='C5'), Row(COL_A='A11', COL_B=6, COL_C='C6'), Row(COL_A='A11', COL_B=7, COL_C='C7')]
Partition 10
[]
Partition 11
[]


NUM OF DATAFRAME PARTITIONS AFTER SHUFFLING: 12
Partition 0
[Row(COL_A='A0', sum(COL_B)=6)]
Partition 1
[Row(COL_A='A1', sum(COL_B)=22), Row(COL_A='A5', sum(COL_B)=22)]
Partition 2
[Row(COL_A='A4', sum(COL_B)=22)]
Partition 3
[Row(COL_A='A7', sum(COL_B)=22)]
Partition 4
[Row(COL_A='A3', sum(COL_B)=22), Row(COL_A='A6', sum(COL_B)=22)]
Partition 5
[Row(COL_A='A9', sum(COL_B)=22), Row(COL_A='A10', sum(COL_B)=22)]
Partition 6
[]
Partition 7
[]
Partition 8
[Row(COL_A='A2', sum(COL_B)=22), Row(COL_A='A8', sum(COL_B)=22)]
Partition 9
[Row(COL_A='A11', sum(COL_B)=22)]
Partition 10
[]
Partition 11
[]
</pre>

As we can see, before data shuffling, all `COL_A`'s elements with the same value are stored in the same partition. Let's use a simple example for `COL_A = A0`. When the `groupby` transformation is executed, Spark doesn't need to move `A0` from a partition to another partition. It already resides within <b>Partition 0</b>. The same rule applies to other elements of `COL_A`. In my opinion, since there's no data movement between different partitions, Spark doesn't need to re-allocate partitions for the resulting aggregation action (`agg(F.sum('COL_B'))`).

Thanks for reading.
