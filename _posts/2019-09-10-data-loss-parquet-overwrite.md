---
title: 'Failure When Overwriting A Parquet File Might Result in Data Loss'
date: 2019-09-10
permalink: /posts/2019/09/data-loss-parquet-overwrite/
tags:
  - pyspark
  - parquet
  - durability
---

There are several critical issues that present when using Spark. One of them relates to data loss when a failure occurs.

Recently I came across such an issue when overwriting a parquet file. Let me simulate the process in a simplified way. I used Spark in local mode.

Suppose we have a simple dataframe <b>df</b>.

```python
df_elements = [
	(‘row_a’, ‘row_b’, ‘row_c’),
] * 100000

df = spark.createDataFrame(df_elements, [‘a’, ‘b’, ‘c’])
```

Now let’s store the dataframe to a parquet file.

```python
df.write.mode(‘overwrite’).parquet(‘path_to_the_parquet_files’)
```

You should see that there are several partition files created when the saving process finishes.

Let’s make the overwriting process fails in the middle.

```python
df.write.mode(‘overwrite’).parquet(‘path_to_the_parquet_files’)
```

When the above code is running, just press <b>Ctrl + C</b> to stop it.

Go back to <I>path_to_the_parquet_files</I> and you should find that all the previous files (before the second parquet write) has been removed.

I browsed the internet to investigate more about this issue, and found a YouTube video titled <a href="https://www.youtube.com/watch?v=0GhFAzN4qs4">Delta Lake for Apache Spark - Why do we need Delta Lake for Spark?</a>. Please watch it in case you want to know more.
