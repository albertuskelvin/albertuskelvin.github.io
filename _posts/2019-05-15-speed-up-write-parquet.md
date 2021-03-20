---
title: 'Speeding Up Parquet Write'
date: 2019-05-15
permalink: /posts/2019/05/speed-up-write-parquet/
tags:
  - spark
  - parquet
  - dataframe
  - partitionby
---

Parquet is a file format with columnar style. Columnar style means that we don't store the content of each row of the data. Here's a simple example.

<pre>
Suppose we want to store this dataframe to a parquet file.

COL_A  |  COL_B  |  COL_C
-------------------------
tempA1 |  tempB1 |  tempC1
tempA2 |  tempB2 |  tempC2
tempA3 |  tempB3 |  tempC3
tempA4 |  tempB4 |  tempC4
tempA5 |  tempB5 |  tempC5
tempA6 |  tempB6 |  tempC6

Here's what the content of the parquet might look like.

COL_A
-----
tempA1
tempA2
tempA3
tempA4
tempA5
tempA6

COL_B
-----
tempB1
tempB2
tempB3
tempB4
tempB5
tempB6

COL_C
-----
tempC1
tempC2
tempC3
tempC4
tempC5
tempC6
</pre>

You can find more detail information from its official website <a href="https://parquet.apache.org/">here</a>.

However, one of the problems when writing data to any storage file is writing time. Bigger dataframes might take longer time to be written than smaller dataframes.

Recently I dealt with this issue when writing a dataframe to a parquet file. Based on my investigation, there's a <b>positive correlation</b> between the number of dataframe partitions and the time needed to write the dataframe to the parquet file. In my humble opinion, one of the reasons for this correlation is because when we store a partitioned dataframe, we ask parquet to create one directory for each partition. For instance, when a dataframe has 5 partitions, then parquet will create 5 directories, each containing dataframe in the corresponding partition. Let's take a look at the below example.

<pre>
Partition 0
===========
COL_A   |   COL_B   |   COL_C
-----------------------------
monday  |   testB1  |   testC1
monday  |   testB2  |   testC2
monday  |   testB3  |   testC3

Partition 1
===========
COL_A   |   COL_B   |   COL_C
-----------------------------
tuesday |   testB4  |   testC4
tuesday |   testB5  |   testC5
tuesday |   testB6  |   testC6

Partition 2
===========
COL_A   |   COL_B   |   COL_C
-----------------------------
friday  |   testB7  |   testC7
friday  |   testB8  |   testC8
friday  |   testB9  |   testC9


PARQUET FILES IF THE NUMBER OF PARTITIONS = 1
=============================================
path/to/the/parquet/files/my_parquet.parquet
    + _SUCCESS
    + part-000-...-snappy.parquet

PARQUET FILES IF THE NUMBER OF PARTITIONS = 3
=============================================
path/to/the/parquet/files/my_parquet.parquet
    + _SUCCESS
    + COL_A=monday
          - part-000-...-snappy.parquet
    + COL_A=tuesday
          - part-000-...-snappy.parquet
    + COL_A=friday
          - part-000-...-snappy.parquet
</pre>

As we can see from the above example, if the number of partitions is 1, we'll get a single parquet file containing the whole dataframe. Meanwhile, if the number of partitions is 3 (or > 1), we'll get 3 directories, each contains dataframe with the corresponding column value.

Below is the result of my experiments.

<pre>
NUM_OF_DISTINCT_VALUES_IN_COL_A = 1000
NUM_OF_ELEMENTS_FOR_EACH_DISTINCT_VALUE_IN_COL_A = 1000
TIME NEEDED BY USING 1 PARTITION = 5.375618934631348 secs
TIME NEEDED BY USING 1000 PARTITIONS = 19.109354972839355 secs

NUM_OF_DISTINCT_VALUES_IN_COL_A = 900
NUM_OF_ELEMENTS_FOR_EACH_DISTINCT_VALUE_IN_COL_A = 1000
TIME NEEDED BY USING 1 PARTITION = 4.966969966888428 secs
TIME NEEDED BY USING 900 PARTITIONS = 14.826158046722412 secs

NUM_OF_DISTINCT_VALUES_IN_COL_A = 500
NUM_OF_ELEMENTS_FOR_EACH_DISTINCT_VALUE_IN_COL_A = 1000
TIME NEEDED BY USING 1 PARTITION = 4.055495262145996 secs
TIME NEEDED BY USING 500 PARTITIONS = 10.482619762420654 secs

NUM_OF_DISTINCT_VALUES_IN_COL_A = 100
NUM_OF_ELEMENTS_FOR_EACH_DISTINCT_VALUE_IN_COL_A = 1000
TIME NEEDED BY USING 1 PARTITION = 2.8042681217193604 secs
TIME NEEDED BY USING 100 PARTITIONS = 4.308538913726807 secs
</pre>

In case you're wondering, here's the code I used to conduct the experiment. You've to comment the other block when doing an experiment on a certain block. For instance, comment BLOCK 1 when you're experimenting on BLOCK 0, and vice versa.

<pre>
'''BLOCK 0'''
start = time.time()
# after creating the dataframe, I checked the num of partitions (df.rdd.getNumPartitions()) and I got 1
df.write.mode('overwrite').parquet(path_to_parquet_files)
print('TIME: ' + str(time.time() - start))

'''BLOCK 1'''
start = time.time()
df.write.partitionBy('COL_A').mode('overwrite').parquet(path_to_parquet_files)
print('TIME: ' + str(time.time() - start))
</pre>

Thanks for reading.
