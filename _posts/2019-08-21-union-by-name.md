---
title: 'Resolving Attributes Data Inconsistency with Union By Name'
date: 2019-08-21
permalink: /posts/2019/08/union-by-name/
tags:
  - pyspark
  - dataframe
  - union by name
---

If you read my previous article titled <a href="https://albertuskelvin.github.io/posts/2019/07/union-left-anti/">Union Operation After Left-anti Join Might Result in Inconsistent Attributes Data</a>, it was shown that the attributes data was inconsistent when combining two data frames after inner-join. According to the article, the solution is really simple. We just need to reorder the attributes order by using <b>select</b> command. Here’s a simple example.

<pre>
unioned_df = joined_df.union(df.select(*joined_df.columns))
</pre>

However, recently I did a little investigation on PySpark’s Github repo. I jumped into the dataframe’s module code and found a method called `unionByName `. There’s a short statement explaining the use of the method: `The difference between this function and :fun:’union’ is that this function resolves columns by name (not by position)`.

Let’s take a look at a simple example (taken from the Spark’s Github repo):

<pre>
>>> df1 = spark.createDataFrame([[1, 2, 3]], ["col0", "col1", "col2"])
>>> df2 = spark.createDataFrame([[4, 5, 6]], ["col1", "col2", "col0"])

>>> df1.unionByName(df2).show()

+----+----+----+
|col0|col1|col2|
+----+----+----+
|   1|   2|   3|
|   6|   4|   5|
+----+----+----+
</pre>

What does it mean? Well, we have two solutions here, either using the <b>select</b> approach (as mentioned in the previous article) or just simply using this `unionByName` method.

Thank you for reading.
