---
title: 'Speeding Up Window Function by Repartitioning the Dataframe First'
date: 2019-05-17
permalink: /posts/2019/05/window-function-and-repartitioning/
tags:
  - spark
  - big data
  - repartition
  - dataframe
  - window function
---

The concept of window function in Spark is pretty interesting. One of its primary usage is calculating cumulative values. Here's a simple example.

<pre>
COL_A   |   COL_B   |   COL_C   |   CUMULATIVE_SUM(COL_B)
---------------------------------------------------------
CA      |   1       |   testC   |   1
CA      |   3       |   testC   |   4
CA      |   5       |   testC   |   9
CA      |   7       |   testC   |   16
CA      |   9       |   testC   |   25
CA      |   11      |   testC   |   36
CB      |   2       |   testC   |   2
CB      |   4       |   testC   |   6
CB      |   6       |   testC   |   12
CB      |   8       |   testC   |   20
CB      |   10      |   testC   |   30
CB      |   12      |   testC   |   42
CC      |   100     |   testC   |   100
CC      |   200     |   testC   |   300
CC      |   300     |   testC   |   600
</pre>

To create this CUMULATIVE_SUM(COL_B) column, the code is really simple.

<pre>
w = Window.partitionBy('COL_A').rowsBetween(-sys.maxsize, 0)

df = df.withColumn('CUMULATIVE_SUM(COL_B)', F.sum(df['COL_B']).over(w))
</pre>

Based on the code snippet above, <i>Window.partitionBy('COL_A')</i> simply means that we divide the dataframe into several categories based on <i>COL_A</i>. Simply put, if <i>COL_A</i> has N distinct elements, than we'll end up with having 3 categories, each consists of the corresponding element of <i>COL_A</i>. After doing this partitioning process, the computation is done for each partition (in this case, we sum up the current element with the sum of all the previous elements within the same partition).

However, when repartitioning a dataframe, Spark will use the default value of the number of partitions if you don't specify one. In this case, the default value is 200. Since we only have 3 partitions, the other 197 partitions are empty. Processing empty partitions might take a little time which correlates positively with the number of empty partitions. Therefore, one of the solutions to speed up the computation process is by repartitioning the dataframe first into the relevant number of partitions. Here's the updated code snippet.

<pre>
num_of_distinct_elements_col_a = df.select('COL_A').distinct().count()
df = df.repartition(num_of_distinct_elements_col_a, 'COL_A')

w = Window.partitionBy('COL_A').rowsBetween(-sys.maxsize, 0)

df = df.withColumn('CUMULATIVE_SUM(COL_B)', F.sum(df['COL_B']).over(w))
</pre>

Voila, the time needed to complete the process was reduced significantly. This is because Spark only needs to apply the computation on the exact number of partitions (no less and no more).

Just FYI, here's what I got from my experiments.

<pre>
total rows: 90
num of distinct elements COL_A: 3 -> each has 30 instances
[1] repartition first: 0.5387139320373535 secs
[2] no repartition first: 2.091965913772583 secs

total rows: 450
num of distinct elements COL_A: 3 -> each has 150 instances
[1] repartition first: 0.546422004699707 secs
[2] no repartition first: 2.1635990142822266 secs

total rows: 9000
num of distinct elements COL_A: 3 -> each has 3000 instances
[1] repartition: 0.6950509548187256 secs
[2] no repartition first: 2.1719470024108887 secs

total rows: 36000
num of distinct elements COL_A: 3 -> each has 12000 instances
[1] repartition first: 1.0497028827667236 secs
[2] no repartition first: 2.456840991973877 secs

total rows: 90000
num of distinct elements COL_A: 3 -> each has 30000 instances
[1] repartition first: 1.5045750141143799 secs
[2] no repartition first: 2.773104190826416 secs

rows: 810000
num of distinct elements COL_A: 3 -> each has 270000 elements
[1] repartition first: 4.972311973571777 secs
[2] no repartition first: 6.299668312072754 secs
</pre>

Wow, there's an interesting clue here. As you can see, the time needed by [1] appears to be close to [2]. From experiments 1 - 3, the time needed by [1] is about 3 - 4 times faster than the time needed by [2]. However, experiment 4 and 5 shows that [1] takes about 1,8 - 2,5 times faster than [2]. Meanwhile, the last experiment shows that [1] takes about 1,27 times faster than [2]. My hypothesis would be as the number of elements within each partition grows, the time needed by both approaches might be close to each other. However, both approaches might not need the exact same processing time (or with very little difference) since the 2nd approach (without repartitioning first) will have to deal with empty partitions. The data distribution across the partitions is not balanced.

And here are the physical plans for both approaches.

<pre>
WITH REPARTITIONING FIRST

== Physical Plan ==
Window [sum(COL_B#1L) windowspecdefinition(COL_A#0, specifiedwindowframe(RowFrame, unboundedpreceding$(), currentrow$())) AS CUMULATIVE_SUM(COL_B)18L], [COL_A#0]
+- *(1) Sort [COL_A#0 ASC NULLS FIRST], false, 0
   +- Exchange hashpartitioning(COL_A#0, 3)
      +- Scan ExistingRDD[COL_A#0,COL_B#1,COL_C#2L]

WITHOUT REPARTITIONING FIRST

== Physical Plan ==
Window [sum(COL_B#1L) windowspecdefinition(COL_A#0, specifiedwindowframe(RowFrame, unboundedpreceding$(), currentrow$())) AS CUMULATIVE_SUM(COL_B)18L], [COL_A#0]
+- *(1) Sort [COL_A#0 ASC NULLS FIRST], false, 0
   +- Exchange hashpartitioning(COL_A#0, 200)
      +- Scan ExistingRDD[COL_A#0,COL_B#1,COL_C#2L]
</pre>

Thanks for reading.
