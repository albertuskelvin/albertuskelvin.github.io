---
title: "Sigma Operation in Spark's Dataframe"
date: 2019-08-08
permalink: /posts/2019/08/sigma-dataframe-spark/
tags:
  - spark
  - python
  - sigma operation
  - dataframe
  - functions
---

Have you ever encountered a case where you need to compute the sum of a certain one-item operation? Consider the following example.

<pre>
sum_{i=1}^{5} (X_i * 100) + Y_i
</pre>

How would you implement such an operation using Spark’s dataframe APIs?

Good news is it’s really simple. You can just play with the columns to specify the operation, use a few methods from Spark’s SQL functions to compute the sigma, and use a dataframe’s API to derive the final result.

Let’s dive into the code.

Suppose you have the following dataframe.

<pre>
df = spark.createDataFrame([(100, 200, 300), (200,300,150), (150,150,150), (100,100,100), (200,300,200), (300,200,100), (300,300,300), (100,300,100)], [‘COL_A’, ‘COL_B’, ‘COL_C’])

+-----+-----+-----+
|COL_A|COL_B|COL_C|
+-----+-----+-----+
|  100|  200|  300|
|  200|  300|  150|
|  150|  150|  150|
|  100|  100|  100|
|  200|  300|  200|
|  300|  200|  100|
|  300|  300|  300|
|  100|  300|  100|
+-----+-----+-----+
</pre>

Next, let’s say that you would like to compute a division operation with the following numerator and denumerator.

<pre>
numerator = sum_{i=1}^{N} COL_A(i) + (COL_B * 30)
denumerator = 30 * sum_{i=1}^{N} COL_B + COL_C
</pre>

Let’s transform the above operation into code.

```python
from pyspark.sql import functions as F

# numerator
numerator = F.sum(F.col(‘COL_A’) + F.col(‘COL_B’) * 30)

# denumerator
denumerator = F.sum(F.col(‘COL_C’) + F.col(‘COL_B’)) * 30
```

Afterwards, let’s combine them to get the final result.

```python
df.select(numerator / denumerator).show()
```

We get the following dataframe.

<pre>
+-----------------------------------------------------------+
|(sum((COL_A + (COL_B * 30))) / (sum((COL_C + COL_B)) * 30))|
+-----------------------------------------------------------+
|                                         0.5841025641025641|
+-----------------------------------------------------------+
</pre>

In case you’re curious, you might want to check what would be the result if we just pass either the numerator or denumerator.

```python
df.select(numerator).show()
```

Which will give the following result.

<pre>
+---------------------------+
|sum((COL_A + (COL_B * 30)))|
+---------------------------+
|                      56950|
+---------------------------+
</pre>

Please try by yourself for the denumerator :)

Last but not least, as you can see from the resulting dataframe, the column name is not tidy. How would you transform those column names into understandable words?

Fortunately, we can use alias operation to modify the column name.

```python
operation = (numerator / denumerator).alias(‘division_operation’)

df.select(operation).show()
```

You’ll get the following result.

<pre>
+------------------+
|division_operation|
+------------------+
|0.5841025641025641|
+------------------+
</pre>

Thanks for reading.
