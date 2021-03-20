---
title: 'Resolving Reference Column Ambiguity After Self-Joining by Deep Copying the Dataframes'
date: 2019-06-28
permalink: /posts/2019/06/self-join-duplicate-columns/
tags:
  - spark
  - selfjoin
  - dataframe
  - deep copy
---

I encountered an intriguing result when joining a dataframe with itself (self-join). As you might have already known, one of the problems occurred when doing a self-join relates to duplicated column names. Because of this duplication, there's an ambiguity when we do operations requiring us to provide the column names.

In this post I'm going to share what I did on my investigation. I used Spark in local mode.

Let's start with creating a simple dataframe called <b>df</b>.

```python
df = spark.createDataFrame([('a0', 'b0'), ('a1', 'b1'), ('a1', 'b2'), ('a1', 'b3'), ('a2', 'b2'), ('a2', 'b3'), ('a3', 'b3')], ['A','B'])
```

Several experiments were conducted after this step. Here we go.

<h2>Experiment 01</h2>

Let's create two aliases (<b>df0</b> and <b>df1</b>) for the previous dataframe.

```python
df0 = df.alias('df0')
df1 = df.alias('df1')
```

I did a quick check both data frames with the following code.

<pre>
df0 == df1
</pre>

Which resulted in <b>False</b>.

Next, I joined both dataframes.

```python
joined_df = df0.join(df1, 'A', how='left')
```

Here's the result.

<pre>
+---+---+---+
|  A|  B|  B|
+---+---+---+
| a3| b3| b3|
| a2| b2| b2|
| a2| b2| b3|
| a2| b3| b2|
| a2| b3| b3|
| a1| b1| b1|
| a1| b1| b2|
| a1| b1| b3|
| a1| b2| b1|
| a1| b2| b2|
| a1| b2| b3|
| a1| b3| b1|
| a1| b3| b2|
| a1| b3| b3|
| a0| b0| b0|
+---+---+---+
</pre>

Now suppose I want to select one of the columns named <b>B</b>.

```python
joined_df.select('B')
```

And certainly got the following error message.

<pre>
pyspark.sql.utils.AnalysisException: "Reference 'B' is ambiguous, could be: df0.B, df1.B.;"
</pre>

Now let's try to obey the error message by providing the dataframe's name holding the column.

```python
col_B_df0 = joined_df.select(df0.B)
col_B_df1 = joined_df.select(df1.B)
```

Both <b>col_B_df0</b> and <b>col_B_df1</b> outputted the following dataframe.

<pre>
+---+
|  B|
+---+
| b3|
| b2|
| b2|
| b3|
| b3|
| b1|
| b1|
| b1|
| b2|
| b2|
| b2|
| b3|
| b3|
| b3|
| b0|
+---+
</pre>

The resulting dataframe is the same when we use <b>df0.B</b> and <b>df1.B</b>. However, when we use <b>df1.B</b>, we expect the following dataframe.

<pre>
+---+
|  B|
+---+
| b3|
| b2|
| b3|
| b2|
| b3|
| b1|
| b2|
| b3|
| b1|
| b2|
| b3|
| b1|
| b2|
| b3|
| b0|
+---+
</pre>

<h2>Experiment 02</h2>

The code was pretty similar, yet in this experiment we'll use <b>=</b> operator when assigning a dataframe to a variable.

```python
df0 = df
df1 = df
```

Did a quick check.

<pre>
df0 == df1
</pre>

And resulted in <b>True</b>.

Joined the two dataframes.

```python
joined_df = df0.join(df1, 'A', how='left')
```

The resulting joined dataframe was the same as the 1st experiment. When selecting column <b>B</b>, I got the same result as well.

<h2>Experiment 03</h2>

In this experiment, the only difference was in the approach of making two dataframes that were the same. Precisely, I used a method called _deep copy_.

Here's the code.

```python
import copy

copied_schema = copy.deepcopy(df.schema)

df0 = df.rdd.zipWithIndex().map(lambda r: r[0]).toDF(copied_schema)
df1 = df.rdd.zipWithIndex().map(lambda r: r[0]).toDF(copied_schema)
```

And performed a quick check.

<pre>
df0 == df1
</pre>

I got <b>False</b> for the above command.

Next, I joined <b>df0</b> and <b>df1</b> and got the same results as the previous experiments. However, when I selected column <b>B</b>, the results were different. Take a look!

<pre>
joined_df.select(df0.B).show()
------------------------------
+---+
|  B|
+---+
| b3|
| b2|
| b2|
| b3|
| b3|
| b1|
| b1|
| b1|
| b2|
| b2|
| b2|
| b3|
| b3|
| b3|
| b0|
+---+


joined_df.select(df1.B).show()
------------------------------
+---+
|  B|
+---+
| b3|
| b2|
| b3|
| b2|
| b3|
| b1|
| b2|
| b3|
| b1|
| b2|
| b3|
| b1|
| b2|
| b3|
| b0|
+---+
</pre>

As I was curious and wanted to clarify the result further, I modified the dataframes by adding a new column called <b>C</b>.

```python
from pyspark.sql import functions as F

# add a new column for df0
df0 = df0.withColumn('C', F.lit('c'))

# add a new column for df1
df1 = df1.withColumn('C', F.lit('CC'))
```

After being joined, the result looked like the following.

<pre>
+---+---+---+---+---+
|  A|  B|  C|  B|  C|
+---+---+---+---+---+
| a3| b3|  c| b3| CC|
| a2| b2|  c| b2| CC|
| a2| b2|  c| b3| CC|
| a2| b3|  c| b2| CC|
| a2| b3|  c| b3| CC|
| a1| b1|  c| b1| CC|
| a1| b1|  c| b2| CC|
| a1| b1|  c| b3| CC|
| a1| b2|  c| b1| CC|
| a1| b2|  c| b2| CC|
| a1| b2|  c| b3| CC|
| a1| b3|  c| b1| CC|
| a1| b3|  c| b2| CC|
| a1| b3|  c| b3| CC|
| a0| b0|  c| b0| CC|
+---+---+---+---+---+
</pre>

I selected column <b>C</b> and it returned the same error message (_Reference is ambiguous_) when I didn't specify the dataframe's name.

However, when I specified the dataframe's name, here's what I got.

<pre>
joined_df.select(df0.C).show()
------------------------------
+---+
|  C|
+---+
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
|  c|
+---+

joined_df.select(df1.C).show()
------------------------------
+---+
|  C|
+---+
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
| CC|
+---+
</pre>

---

Well, it seems that we need to make the schema of the two dataframes independent (two different schemas even though the schema's content is the same). I decided to use a deep copy based on a thought that two dataframes created from scratch (using _spark.createDataFrame()_) would solve such an issue. And one way to achieve that was by applying a deep copy mechanism (the resulting dataframe won't refer to the dataframe from which it was copied).

Thank you for reading.
