---
title: 'Making mapPartitions Accepts Partition Functions with More Than One Arguments'
date: 2019-10-16
permalink: /posts/2019/10/map-partitions-with-more-than-one-parameters/
tags:
  - spark
  - python
  - rdd
  - mappartitions
---

There might be a case where we need to perform a certain operation on each data partition. One of the most common examples is the use of <b>mapPartitions</b>. Sometimes, such an operation probably requires a more complicated procedure. This, in the end, makes the method executing the operation needs more than one parameter.

However, according to the <a href="https://spark.apache.org/docs/latest/rdd-programming-guide.html#transformations">documentation</a>, such a transformer only accepts a function with <I>iterator</I> as the single parameter.

Therefore, the question is simply, how to make the function accepts more than just one parameter?

I came across such a challenge recently. I thought it was one of the limitations in Spark that couldn’t be engineered. Until I’ve found a solution that seems extremely simple.

To make it brief, just imagine that we have a dataframe that has been repartitioned into a certain number of partitions.

To apply an operation on each partition, we pass the corresponding function as the parameter of <b>mapPartitions</b>.

```python
def func(param_a, param_b):
	def partition_func(iterator):
		total = 0
		for row in iterator:
			total += (row[‘a’] + row[‘b’]) * param_a
		
		total = total - param_b
		return total

	return partition_func

# compute the total value for each partition
total = df.rdd.mapPartitions(func(param_a, param_b))
```

The above code will return the value of <b>total</b> for each partition. To get more insight on what the return value looks like, take a look at the below code snippet.

```python
total.glom().collect()
```

The above code will return the following.

```
[
	[total_for_partition_0],
	[total_for_partition_1],
	[total_for_partition_2],
]
```

Thank you for reading.
