---
title: 'The Number of Partitions After Unioning Two or More Dataframes'
date: 2019-06-14
permalink: /posts/2019/06/number-partitions-after-union/
tags:
  - spark
  - dataframe
  - union
---

An intriguing question popped into my mind. After unioning several dataframes, how many partitions the resulting dataframe will have?

I investigated such a behavior using few configurations, such as the number of unioned dataframes, the number of distinct elements in each dataframe, and way of repartitioning the dataframe. The code I used is shown below.

```python
def create_df(on_cols, amount_of_each_on_elements):
	ret_list = []
	for index, on_elmt in enumerate(on_cols):
		amount = amount_of_each_on_elements[index]
		for i in range(amount):
			ret_list.append((on_elmt, 'feature'+str(i)))

	return ret_list

def show_partitions(df):
	print('Num of partitions: {}'.format(df.rdd.getNumPartitions()))
	for index, partition in enumerate(df.rdd.glom().collect()):
		print('Partition {}'.format(index))
		print(partition)
	print('\n')


on_cols = ['A','B', 'C', 'D', 'E']
amount_of_each_on_elements = [5, 3, 3, 2, 3]

ret_list = create_df(on_cols, amount_of_each_on_elements)
df_0 = spark.createDataFrame(ret_list, ['ON', 'OTHER_FEATURES'])
df_0 = df_0.repartition(5, 'ON')

on_cols = ['X','Y','Z']
amount_of_each_on_elements = [10, 3, 15]

ret_list = create_df(on_cols, amount_of_each_on_elements)
df_1 = spark.createDataFrame(ret_list, ['ON', 'OTHER_FEATURES'])
df_1 = df_1.repartition(3, 'ON')

on_cols = ['P','Q','R']
amount_of_each_on_elements = [5, 3, 5]

ret_list = create_df(on_cols, amount_of_each_on_elements)
df_2 = spark.createDataFrame(ret_list, ['ON', 'OTHER_FEATURES'])
df_2 = df_2.repartition(3, 'ON')

on_cols = ['A','B','C', 'X', 'Y', 'Z']
amount_of_each_on_elements = [5, 5, 5, 5, 5, 5]

ret_list = create_df(on_cols, amount_of_each_on_elements)
df_3 = spark.createDataFrame(ret_list, ['ON', 'OTHER_FEATURES'])
df_3 = df_3.repartition(6, 'ON')

union_df = df_0.union(df_1)
union_df = union_df.union(df_2)
union_df = union_df.union(df_3)

show_partitions(df_0)
show_partitions(df_1)
show_partitions(df_2)
show_partitions(df_3)
show_partitions(union_df)
```

The investigation results show that the number of partitions of the unioned dataframe follows the following formula:

<pre>
num_partitions(U) = num_partitions(df_0) + num_partitions(df_1) + ... + num_partitions(df_n).
</pre>

In addition, the order of partitions in the unioned dataframe follows the original order in each dataframe being unioned. For the sake of clarity, here's a simple example.

<pre>
Dataframe A
+++++++++++
Partition 0
<i>list of Rows</i>
Partition 1
<i>list of Rows</i>

Dataframe B
+++++++++++
Partition 0
<i>list of Rows</i>
Partition 1
<i>list of Rows</i>
Partition 2
<i>list of Rows</i>

Dataframe C
+++++++++++
Partition 0
<i>list of Rows</i>
Partition 1
<i>list of Rows</i>
Partition 2
<i>list of Rows</i>
Partition 3
<i>list of Rows</i>

Then we union these 3 dataframes: <b>unioned_df = dfA.union(dfB).union(dfC)</b>

Here's the result.

Unioned dataframe
+++++++++++++++++
Partition 0 (from Dataframe A)
<i>list of Rows</i>
Partition 1 (from Dataframe A)
<i>list of Rows</i>
Partition 2 (from Dataframe B)
<i>list of Rows</i>
Partition 3 (from Dataframe B)
<i>list of Rows</i>
Partition 4 (from Dataframe B)
<i>list of Rows</i>
Partition 5 (from Dataframe C)
<i>list of Rows</i>
Partition 6 (from Dataframe C)
<i>list of Rows</i>
Partition 7 (from Dataframe C)
<i>list of Rows</i>
Partition 8 (from Dataframe C)
<i>list of Rows</i>
</pre>

However, a little problem appears when each dataframe has lots of empty partitions. This means that the unioned dataframe will contain much more empty partitions (because of the sum formula). It's because the Dataframes use <i>HashPartitioner</i> as the default partitioner. Therefore, we might want to create a custom partitioner such that all the partitions in each dataframe are not empty. Here's how we can do it.

```python
df_0 = spark.createDataFrame(ret_list, ['ON', 'OTHER_FEATURES'])
rdd_key = df_0.rdd.keyBy(lambda row: row['ON'])
df_0 = rdd_key.partitionBy(NUM_OF_DISTINCT_ELEMENTS, lambda k: on_cols.index(k)).map(lambda kv: kv[1]).toDF(['ON', 'OTHER_FEATURES'])

df_1 = spark.createDataFrame(ret_list, ['ON', 'OTHER_FEATURES'])
rdd_key = df_1.rdd.keyBy(lambda row: row['ON'])
df_1 = rdd_key.partitionBy(NUM_OF_DISTINCT_ELEMENTS, lambda k: on_cols.index(k)).map(lambda kv: kv[1]).toDF(['ON', 'OTHER_FEATURES'])

df_2 = spark.createDataFrame(ret_list, ['ON', 'OTHER_FEATURES'])
rdd_key = df_2.rdd.keyBy(lambda row: row['ON'])
df_2 = rdd_key.partitionBy(NUM_OF_DISTINCT_ELEMENTS, lambda k: on_cols.index(k)).map(lambda kv: kv[1]).toDF(['ON', 'OTHER_FEATURES'])

df_3 = spark.createDataFrame(ret_list, ['ON', 'OTHER_FEATURES'])
rdd_key = df_3.rdd.keyBy(lambda row: row['ON'])
df_3 = rdd_key.partitionBy(NUM_OF_DISTINCT_ELEMENTS, lambda k: on_cols.index(k)).map(lambda kv: kv[1]).toDF(['ON', 'OTHER_FEATURES'])
```

As the result of using a custom partitioner, we might finally end up with the following results.

<pre>
DEFAULT PARTITIONER
===================
Dataframe A
+++++++++++
Num of partitions: 5
Partition 0                                                                     
<i>empty</i>
Partition 1
Row(ON='A', OTHER_FEATURES='feature0'), Row(ON='A', OTHER_FEATURES='feature1'), Row(ON='A', OTHER_FEATURES='feature2'), Row(ON='A', OTHER_FEATURES='feature3'), Row(ON='A', OTHER_FEATURES='feature4')
Partition 2
<i>empty</i>
Partition 3
Row(ON='D', OTHER_FEATURES='feature0'), Row(ON='D', OTHER_FEATURES='feature1'), Row(ON='E', OTHER_FEATURES='feature0'), Row(ON='E', OTHER_FEATURES='feature1'), Row(ON='E', OTHER_FEATURES='feature2')
Partition 4
Row(ON='B', OTHER_FEATURES='feature0'), Row(ON='B', OTHER_FEATURES='feature1'), Row(ON='B', OTHER_FEATURES='feature2'), Row(ON='C', OTHER_FEATURES='feature0'), Row(ON='C', OTHER_FEATURES='feature1'), Row(ON='C', OTHER_FEATURES='feature2')

Dataframe B
+++++++++++
Num of partitions: 3
Partition 0
Row(ON='X', OTHER_FEATURES='feature0'), Row(ON='X', OTHER_FEATURES='feature1'), Row(ON='X', OTHER_FEATURES='feature2'), Row(ON='X', OTHER_FEATURES='feature3'), Row(ON='X', OTHER_FEATURES='feature4'), Row(ON='X', OTHER_FEATURES='feature5'), Row(ON='X', OTHER_FEATURES='feature6'), Row(ON='X', OTHER_FEATURES='feature7'), Row(ON='X', OTHER_FEATURES='feature8'), Row(ON='X', OTHER_FEATURES='feature9'), Row(ON='Y', OTHER_FEATURES='feature0'), Row(ON='Y', OTHER_FEATURES='feature1'), Row(ON='Y', OTHER_FEATURES='feature2')
Partition 1
Row(ON='Z', OTHER_FEATURES='feature0'), Row(ON='Z', OTHER_FEATURES='feature1'), Row(ON='Z', OTHER_FEATURES='feature2'), Row(ON='Z', OTHER_FEATURES='feature3'), Row(ON='Z', OTHER_FEATURES='feature4'), Row(ON='Z', OTHER_FEATURES='feature5'), Row(ON='Z', OTHER_FEATURES='feature6'), Row(ON='Z', OTHER_FEATURES='feature7'), Row(ON='Z', OTHER_FEATURES='feature8'), Row(ON='Z', OTHER_FEATURES='feature9'), Row(ON='Z', OTHER_FEATURES='feature10'), Row(ON='Z', OTHER_FEATURES='feature11'), Row(ON='Z', OTHER_FEATURES='feature12'), Row(ON='Z', OTHER_FEATURES='feature13'), Row(ON='Z', OTHER_FEATURES='feature14')
Partition 2
<i>empty</i>

Dataframe C
+++++++++++
Num of partitions: 3
Partition 0
<i>empty</i>
Partition 1
<i>empty</i>
Partition 2
Row(ON='P', OTHER_FEATURES='feature0'), Row(ON='P', OTHER_FEATURES='feature1'), Row(ON='P', OTHER_FEATURES='feature2'), Row(ON='P', OTHER_FEATURES='feature3'), Row(ON='P', OTHER_FEATURES='feature4'), Row(ON='Q', OTHER_FEATURES='feature0'), Row(ON='Q', OTHER_FEATURES='feature1'), Row(ON='Q', OTHER_FEATURES='feature2'), Row(ON='R', OTHER_FEATURES='feature0'), Row(ON='R', OTHER_FEATURES='feature1'), Row(ON='R', OTHER_FEATURES='feature2'), Row(ON='R', OTHER_FEATURES='feature3'), Row(ON='R', OTHER_FEATURES='feature4')

Dataframe D
+++++++++++
Num of partitions: 6
Partition 0
<i>empty</i>
Partition 1
[Row(ON='C', OTHER_FEATURES='feature0'), Row(ON='C', OTHER_FEATURES='feature1'), Row(ON='C', OTHER_FEATURES='feature2'), Row(ON='C', OTHER_FEATURES='feature3'), Row(ON='C', OTHER_FEATURES='feature4')]
Partition 2
[Row(ON='A', OTHER_FEATURES='feature0'), Row(ON='A', OTHER_FEATURES='feature1'), Row(ON='A', OTHER_FEATURES='feature2'), Row(ON='A', OTHER_FEATURES='feature3'), Row(ON='A', OTHER_FEATURES='feature4')]
Partition 3
[Row(ON='B', OTHER_FEATURES='feature0'), Row(ON='B', OTHER_FEATURES='feature1'), Row(ON='B', OTHER_FEATURES='feature2'), Row(ON='B', OTHER_FEATURES='feature3'), Row(ON='B', OTHER_FEATURES='feature4'), Row(ON='X', OTHER_FEATURES='feature0'), Row(ON='X', OTHER_FEATURES='feature1'), Row(ON='X', OTHER_FEATURES='feature2'), Row(ON='X', OTHER_FEATURES='feature3'), Row(ON='X', OTHER_FEATURES='feature4'), Row(ON='Y', OTHER_FEATURES='feature0'), Row(ON='Y', OTHER_FEATURES='feature1'), Row(ON='Y', OTHER_FEATURES='feature2'), Row(ON='Y', OTHER_FEATURES='feature3'), Row(ON='Y', OTHER_FEATURES='feature4')]
Partition 4
[Row(ON='Z', OTHER_FEATURES='feature0'), Row(ON='Z', OTHER_FEATURES='feature1'), Row(ON='Z', OTHER_FEATURES='feature2'), Row(ON='Z', OTHER_FEATURES='feature3'), Row(ON='Z', OTHER_FEATURES='feature4')]
Partition 5
<i>empty</i>

UNIONED DATAFRAME
+++++++++++++++++
Num of partitions: 17
Partition 0
<i>empty</i>
Partition 1
[Row(ON='A', OTHER_FEATURES='feature0'), Row(ON='A', OTHER_FEATURES='feature1'), Row(ON='A', OTHER_FEATURES='feature2'), Row(ON='A', OTHER_FEATURES='feature3'), Row(ON='A', OTHER_FEATURES='feature4')
Partition 2
<i>empty</i>
Partition 3
Row(ON='D', OTHER_FEATURES='feature0'), Row(ON='D', OTHER_FEATURES='feature1'), Row(ON='E', OTHER_FEATURES='feature0'), Row(ON='E', OTHER_FEATURES='feature1'), Row(ON='E', OTHER_FEATURES='feature2')
Partition 4
Row(ON='B', OTHER_FEATURES='feature0'), Row(ON='B', OTHER_FEATURES='feature1'), Row(ON='B', OTHER_FEATURES='feature2'), Row(ON='C', OTHER_FEATURES='feature0'), Row(ON='C', OTHER_FEATURES='feature1'), Row(ON='C', OTHER_FEATURES='feature2')
Partition 5
Row(ON='X', OTHER_FEATURES='feature0'), Row(ON='X', OTHER_FEATURES='feature1'), Row(ON='X', OTHER_FEATURES='feature2'), Row(ON='X', OTHER_FEATURES='feature3'), Row(ON='X', OTHER_FEATURES='feature4'), Row(ON='X', OTHER_FEATURES='feature5'), Row(ON='X', OTHER_FEATURES='feature6'), Row(ON='X', OTHER_FEATURES='feature7'), Row(ON='X', OTHER_FEATURES='feature8'), Row(ON='X', OTHER_FEATURES='feature9'), Row(ON='Y', OTHER_FEATURES='feature0'), Row(ON='Y', OTHER_FEATURES='feature1'), Row(ON='Y', OTHER_FEATURES='feature2')
Partition 6
Row(ON='Z', OTHER_FEATURES='feature0'), Row(ON='Z', OTHER_FEATURES='feature1'), Row(ON='Z', OTHER_FEATURES='feature2'), Row(ON='Z', OTHER_FEATURES='feature3'), Row(ON='Z', OTHER_FEATURES='feature4'), Row(ON='Z', OTHER_FEATURES='feature5'), Row(ON='Z', OTHER_FEATURES='feature6'), Row(ON='Z', OTHER_FEATURES='feature7'), Row(ON='Z', OTHER_FEATURES='feature8'), Row(ON='Z', OTHER_FEATURES='feature9'), Row(ON='Z', OTHER_FEATURES='feature10'), Row(ON='Z', OTHER_FEATURES='feature11'), Row(ON='Z', OTHER_FEATURES='feature12'), Row(ON='Z', OTHER_FEATURES='feature13'), Row(ON='Z', OTHER_FEATURES='feature14')
Partition 7
<i>empty</i>
Partition 8
<i>empty</i>
Partition 9
<i>empty</i>
Partition 10
Row(ON='P', OTHER_FEATURES='feature0'), Row(ON='P', OTHER_FEATURES='feature1'), Row(ON='P', OTHER_FEATURES='feature2'), Row(ON='P', OTHER_FEATURES='feature3'), Row(ON='P', OTHER_FEATURES='feature4'), Row(ON='Q', OTHER_FEATURES='feature0'), Row(ON='Q', OTHER_FEATURES='feature1'), Row(ON='Q', OTHER_FEATURES='feature2'), Row(ON='R', OTHER_FEATURES='feature0'), Row(ON='R', OTHER_FEATURES='feature1'), Row(ON='R', OTHER_FEATURES='feature2'), Row(ON='R', OTHER_FEATURES='feature3'), Row(ON='R', OTHER_FEATURES='feature4')
Partition 11
<i>empty</i>
Partition 12
Row(ON='C', OTHER_FEATURES='feature0'), Row(ON='C', OTHER_FEATURES='feature1'), Row(ON='C', OTHER_FEATURES='feature2'), Row(ON='C', OTHER_FEATURES='feature3'), Row(ON='C', OTHER_FEATURES='feature4')
Partition 13
Row(ON='A', OTHER_FEATURES='feature0'), Row(ON='A', OTHER_FEATURES='feature1'), Row(ON='A', OTHER_FEATURES='feature2'), Row(ON='A', OTHER_FEATURES='feature3'), Row(ON='A', OTHER_FEATURES='feature4')
Partition 14
Row(ON='B', OTHER_FEATURES='feature0'), Row(ON='B', OTHER_FEATURES='feature1'), Row(ON='B', OTHER_FEATURES='feature2'), Row(ON='B', OTHER_FEATURES='feature3'), Row(ON='B', OTHER_FEATURES='feature4'), Row(ON='X', OTHER_FEATURES='feature0'), Row(ON='X', OTHER_FEATURES='feature1'), Row(ON='X', OTHER_FEATURES='feature2'), Row(ON='X', OTHER_FEATURES='feature3'), Row(ON='X', OTHER_FEATURES='feature4'), Row(ON='Y', OTHER_FEATURES='feature0'), Row(ON='Y', OTHER_FEATURES='feature1'), Row(ON='Y', OTHER_FEATURES='feature2'), Row(ON='Y', OTHER_FEATURES='feature3'), Row(ON='Y', OTHER_FEATURES='feature4')
Partition 15
Row(ON='Z', OTHER_FEATURES='feature0'), Row(ON='Z', OTHER_FEATURES='feature1'), Row(ON='Z', OTHER_FEATURES='feature2'), Row(ON='Z', OTHER_FEATURES='feature3'), Row(ON='Z', OTHER_FEATURES='feature4')
Partition 16
<i>empty</i>


CUSTOM PARTITIONER
==================
Dataframe A
+++++++++++
Num of partitions: 5                                                            
Partition 0
Row(ON='A', OTHER_FEATURES='feature0'), Row(ON='A', OTHER_FEATURES='feature1'), Row(ON='A', OTHER_FEATURES='feature2'), Row(ON='A', OTHER_FEATURES='feature3'), Row(ON='A', OTHER_FEATURES='feature4')
Partition 1
Row(ON='B', OTHER_FEATURES='feature0'), Row(ON='B', OTHER_FEATURES='feature1'), Row(ON='B', OTHER_FEATURES='feature2')
Partition 2
Row(ON='C', OTHER_FEATURES='feature0'), Row(ON='C', OTHER_FEATURES='feature1'), Row(ON='C', OTHER_FEATURES='feature2')
Partition 3
Row(ON='D', OTHER_FEATURES='feature0'), Row(ON='D', OTHER_FEATURES='feature1')
Partition 4
Row(ON='E', OTHER_FEATURES='feature0'), Row(ON='E', OTHER_FEATURES='feature1'), Row(ON='E', OTHER_FEATURES='feature2')

Dataframe B
+++++++++++
Num of partitions: 3
Partition 0
Row(ON='X', OTHER_FEATURES='feature0'), Row(ON='X', OTHER_FEATURES='feature1'), Row(ON='X', OTHER_FEATURES='feature2'), Row(ON='X', OTHER_FEATURES='feature3'), Row(ON='X', OTHER_FEATURES='feature4'), Row(ON='X', OTHER_FEATURES='feature5'), Row(ON='X', OTHER_FEATURES='feature6'), Row(ON='X', OTHER_FEATURES='feature7'), Row(ON='X', OTHER_FEATURES='feature8'), Row(ON='X', OTHER_FEATURES='feature9')
Partition 1
Row(ON='Y', OTHER_FEATURES='feature0'), Row(ON='Y', OTHER_FEATURES='feature1'), Row(ON='Y', OTHER_FEATURES='feature2')
Partition 2
Row(ON='Z', OTHER_FEATURES='feature0'), Row(ON='Z', OTHER_FEATURES='feature1'), Row(ON='Z', OTHER_FEATURES='feature2'), Row(ON='Z', OTHER_FEATURES='feature3'), Row(ON='Z', OTHER_FEATURES='feature4'), Row(ON='Z', OTHER_FEATURES='feature5'), Row(ON='Z', OTHER_FEATURES='feature6'), Row(ON='Z', OTHER_FEATURES='feature7'), Row(ON='Z', OTHER_FEATURES='feature8'), Row(ON='Z', OTHER_FEATURES='feature9'), Row(ON='Z', OTHER_FEATURES='feature10'), Row(ON='Z', OTHER_FEATURES='feature11'), Row(ON='Z', OTHER_FEATURES='feature12'), Row(ON='Z', OTHER_FEATURES='feature13'), Row(ON='Z', OTHER_FEATURES='feature14')

Dataframe C
+++++++++++
Num of partitions: 3
Partition 0
Row(ON='P', OTHER_FEATURES='feature0'), Row(ON='P', OTHER_FEATURES='feature1'), Row(ON='P', OTHER_FEATURES='feature2'), Row(ON='P', OTHER_FEATURES='feature3'), Row(ON='P', OTHER_FEATURES='feature4')
Partition 1
Row(ON='Q', OTHER_FEATURES='feature0'), Row(ON='Q', OTHER_FEATURES='feature1'), Row(ON='Q', OTHER_FEATURES='feature2')
Partition 2
Row(ON='R', OTHER_FEATURES='feature0'), Row(ON='R', OTHER_FEATURES='feature1'), Row(ON='R', OTHER_FEATURES='feature2'), Row(ON='R', OTHER_FEATURES='feature3'), Row(ON='R', OTHER_FEATURES='feature4')

Dataframe D
+++++++++++
Num of partitions: 6
Partition 0
Row(ON='A', OTHER_FEATURES='feature0'), Row(ON='A', OTHER_FEATURES='feature1'), Row(ON='A', OTHER_FEATURES='feature2'), Row(ON='A', OTHER_FEATURES='feature3'), Row(ON='A', OTHER_FEATURES='feature4')
Partition 1
Row(ON='B', OTHER_FEATURES='feature0'), Row(ON='B', OTHER_FEATURES='feature1'), Row(ON='B', OTHER_FEATURES='feature2'), Row(ON='B', OTHER_FEATURES='feature3'), Row(ON='B', OTHER_FEATURES='feature4')
Partition 2
Row(ON='C', OTHER_FEATURES='feature0'), Row(ON='C', OTHER_FEATURES='feature1'), Row(ON='C', OTHER_FEATURES='feature2'), Row(ON='C', OTHER_FEATURES='feature3'), Row(ON='C', OTHER_FEATURES='feature4')
Partition 3
Row(ON='X', OTHER_FEATURES='feature0'), Row(ON='X', OTHER_FEATURES='feature1'), Row(ON='X', OTHER_FEATURES='feature2'), Row(ON='X', OTHER_FEATURES='feature3'), Row(ON='X', OTHER_FEATURES='feature4')
Partition 4
Row(ON='Y', OTHER_FEATURES='feature0'), Row(ON='Y', OTHER_FEATURES='feature1'), Row(ON='Y', OTHER_FEATURES='feature2'), Row(ON='Y', OTHER_FEATURES='feature3'), Row(ON='Y', OTHER_FEATURES='feature4')
Partition 5
Row(ON='Z', OTHER_FEATURES='feature0'), Row(ON='Z', OTHER_FEATURES='feature1'), Row(ON='Z', OTHER_FEATURES='feature2'), Row(ON='Z', OTHER_FEATURES='feature3'), Row(ON='Z', OTHER_FEATURES='feature4')

UNIONED DATAFRAME
+++++++++++++++++
Num of partitions: 17
Partition 0
Row(ON='A', OTHER_FEATURES='feature0'), Row(ON='A', OTHER_FEATURES='feature1'), Row(ON='A', OTHER_FEATURES='feature2'), Row(ON='A', OTHER_FEATURES='feature3'), Row(ON='A', OTHER_FEATURES='feature4')
Partition 1
Row(ON='B', OTHER_FEATURES='feature0'), Row(ON='B', OTHER_FEATURES='feature1'), Row(ON='B', OTHER_FEATURES='feature2')
Partition 2
Row(ON='C', OTHER_FEATURES='feature0'), Row(ON='C', OTHER_FEATURES='feature1'), Row(ON='C', OTHER_FEATURES='feature2')
Partition 3
Row(ON='D', OTHER_FEATURES='feature0'), Row(ON='D', OTHER_FEATURES='feature1')
Partition 4
Row(ON='E', OTHER_FEATURES='feature0'), Row(ON='E', OTHER_FEATURES='feature1'), Row(ON='E', OTHER_FEATURES='feature2')
Partition 5
Row(ON='X', OTHER_FEATURES='feature0'), Row(ON='X', OTHER_FEATURES='feature1'), Row(ON='X', OTHER_FEATURES='feature2'), Row(ON='X', OTHER_FEATURES='feature3'), Row(ON='X', OTHER_FEATURES='feature4'), Row(ON='X', OTHER_FEATURES='feature5'), Row(ON='X', OTHER_FEATURES='feature6'), Row(ON='X', OTHER_FEATURES='feature7'), Row(ON='X', OTHER_FEATURES='feature8'), Row(ON='X', OTHER_FEATURES='feature9')
Partition 6
Row(ON='Y', OTHER_FEATURES='feature0'), Row(ON='Y', OTHER_FEATURES='feature1'), Row(ON='Y', OTHER_FEATURES='feature2')
Partition 7
Row(ON='Z', OTHER_FEATURES='feature0'), Row(ON='Z', OTHER_FEATURES='feature1'), Row(ON='Z', OTHER_FEATURES='feature2'), Row(ON='Z', OTHER_FEATURES='feature3'), Row(ON='Z', OTHER_FEATURES='feature4'), Row(ON='Z', OTHER_FEATURES='feature5'), Row(ON='Z', OTHER_FEATURES='feature6'), Row(ON='Z', OTHER_FEATURES='feature7'), Row(ON='Z', OTHER_FEATURES='feature8'), Row(ON='Z', OTHER_FEATURES='feature9'), Row(ON='Z', OTHER_FEATURES='feature10'), Row(ON='Z', OTHER_FEATURES='feature11'), Row(ON='Z', OTHER_FEATURES='feature12'), Row(ON='Z', OTHER_FEATURES='feature13'), Row(ON='Z', OTHER_FEATURES='feature14')
Partition 8
Row(ON='P', OTHER_FEATURES='feature0'), Row(ON='P', OTHER_FEATURES='feature1'), Row(ON='P', OTHER_FEATURES='feature2'), Row(ON='P', OTHER_FEATURES='feature3'), Row(ON='P', OTHER_FEATURES='feature4')
Partition 9
Row(ON='Q', OTHER_FEATURES='feature0'), Row(ON='Q', OTHER_FEATURES='feature1'), Row(ON='Q', OTHER_FEATURES='feature2')
Partition 10
Row(ON='R', OTHER_FEATURES='feature0'), Row(ON='R', OTHER_FEATURES='feature1'), Row(ON='R', OTHER_FEATURES='feature2'), Row(ON='R', OTHER_FEATURES='feature3'), Row(ON='R', OTHER_FEATURES='feature4')
Partition 11
Row(ON='A', OTHER_FEATURES='feature0'), Row(ON='A', OTHER_FEATURES='feature1'), Row(ON='A', OTHER_FEATURES='feature2'), Row(ON='A', OTHER_FEATURES='feature3'), Row(ON='A', OTHER_FEATURES='feature4')
Partition 12
Row(ON='B', OTHER_FEATURES='feature0'), Row(ON='B', OTHER_FEATURES='feature1'), Row(ON='B', OTHER_FEATURES='feature2'), Row(ON='B', OTHER_FEATURES='feature3'), Row(ON='B', OTHER_FEATURES='feature4')
Partition 13
Row(ON='C', OTHER_FEATURES='feature0'), Row(ON='C', OTHER_FEATURES='feature1'), Row(ON='C', OTHER_FEATURES='feature2'), Row(ON='C', OTHER_FEATURES='feature3'), Row(ON='C', OTHER_FEATURES='feature4')
Partition 14
Row(ON='X', OTHER_FEATURES='feature0'), Row(ON='X', OTHER_FEATURES='feature1'), Row(ON='X', OTHER_FEATURES='feature2'), Row(ON='X', OTHER_FEATURES='feature3'), Row(ON='X', OTHER_FEATURES='feature4')
Partition 15
Row(ON='Y', OTHER_FEATURES='feature0'), Row(ON='Y', OTHER_FEATURES='feature1'), Row(ON='Y', OTHER_FEATURES='feature2'), Row(ON='Y', OTHER_FEATURES='feature3'), Row(ON='Y', OTHER_FEATURES='feature4')
Partition 16
Row(ON='Z', OTHER_FEATURES='feature0'), Row(ON='Z', OTHER_FEATURES='feature1'), Row(ON='Z', OTHER_FEATURES='feature2'), Row(ON='Z', OTHER_FEATURES='feature3'), Row(ON='Z', OTHER_FEATURES='feature4')
</pre>

Thank you for reading.
