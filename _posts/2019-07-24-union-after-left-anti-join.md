---
title: 'Union Operation After Left-anti Join Might Result in Inconsistent Attributes Data'
date: 2019-07-24
permalink: /posts/2019/07/union-left-anti/
tags:
  - spark
  - dataframe
  - union
  - left anti join
---

Unioning two dataframes after joining them with <i>left_anti</i>? Well, seems like a straightforward approach. However, recently I encountered a case where join operation might shift the location of the join key in the resulting dataframe. This, unfortunately, makes the dataframe’s merging result inconsistent in terms of the data in each attribute.

For the sake of clarity, here’s what I found in my observation.

Let’s create two simple dataframes called <b>df</b> and <b>df1</b>.

```python
df = spark.createDataFrame([('a0','b0','rowx','c0'), ('a1','b1','rowx','c1'), ('a2','b2','rowy','c2'),('a3','b3','rowz','c3'),('a4','b4','rowy','c4'),('a5','b5','rowy','c5'),('a6','b6','rowx','c6'),('a7','b7','rowz','c7'),('a8','b8','rowy','c8'),('a9','b9','rowx','c9')],['col_a','col_b','id','col_c'])

df1 = spark.createDataFrame([('a0','rowx','b0','c0'), ('a1','rowy','b1','c1'), ('a2','rowx','b2','c2'),('a3','rowx','b3','c3'),('a4','rowx','b4','c4'),('a5','rowy','b5','c5')],['col_a','id','col_b','col_c'])
```

After that, join the dataframes on the column <b>id</b> using <I>left_anti</I> approach.

```python
joined_df = df.join(df1, ‘id’, how=‘left_anti’)
```

Here’s the resulting dataframe.

<pre>
+----+-----+-----+-----+
|  id|col_a|col_b|col_c|
+----+-----+-----+-----+
|rowz|   a3|   b3|   c3|
|rowz|   a7|   b7|   c7|
+----+-----+-----+-----+
</pre>

As you can see, the location of column <b>id</b> (join key) in <b>joined_df</b> differs from the one in <b>df</b>. The order of the rest of the attributes stays the same.

With this condition, let’s union <b>df</b> and <b>joined_df</b> using all the possibilities.

```python
# case 1
unioned_df = df.union(joined_df)

# case 2
unioned_df_2 = joined_df.union(df)
```

Here are the results.

<pre>
# case 1: df.union(joined_df)
+-----+-----+----+-----+
|col_a|col_b|  id|col_c|
+-----+-----+----+-----+
|   a0|   b0|rowx|   c0|
|   a1|   b1|rowx|   c1|
|   a2|   b2|rowy|   c2|
|   a3|   b3|rowz|   c3|
|   a4|   b4|rowy|   c4|
|   a5|   b5|rowy|   c5|
|   a6|   b6|rowx|   c6|
|   a7|   b7|rowz|   c7|
|   a8|   b8|rowy|   c8|
|   a9|   b9|rowx|   c9|
| rowz|   a3|  b3|   c3|
| rowz|   a7|  b7|   c7|
+-----+-----+----+-----+

# case 2: joined_df.union(df)
+----+-----+-----+-----+
|  id|col_a|col_b|col_c|
+----+-----+-----+-----+
|rowz|   a3|   b3|   c3|
|rowz|   a7|   b7|   c7|
|  a0|   b0| rowx|   c0|
|  a1|   b1| rowx|   c1|
|  a2|   b2| rowy|   c2|
|  a3|   b3| rowz|   c3|
|  a4|   b4| rowy|   c4|
|  a5|   b5| rowy|   c5|
|  a6|   b6| rowx|   c6|
|  a7|   b7| rowz|   c7|
|  a8|   b8| rowy|   c8|
|  a9|   b9| rowx|   c9|
+----+-----+-----+-----+
</pre>

In the first case, the column’s order is the same as what appears in <b>df</b>. Meanwhile, in the second case, the column’s order follows the order in <b>joined_df</b>.

However, as you can see, the union result is definitely wrong. To resolve this issue, let’s add a simple line of code like the following.

```python
# case 1
unioned_df = df.union(joined_df.select(*df.columns))

# case 2
unioned_df_2 = joined_df.union(df.select(*joined_df.columns))
```

And here are the results.

<pre>
# case 1: df.union(joined_df.select(*df.columns))
+-----+-----+----+-----+
|col_a|col_b|  id|col_c|
+-----+-----+----+-----+
|   a0|   b0|rowx|   c0|
|   a1|   b1|rowx|   c1|
|   a2|   b2|rowy|   c2|
|   a3|   b3|rowz|   c3|
|   a4|   b4|rowy|   c4|
|   a5|   b5|rowy|   c5|
|   a6|   b6|rowx|   c6|
|   a7|   b7|rowz|   c7|
|   a8|   b8|rowy|   c8|
|   a9|   b9|rowx|   c9|
|   a3|   b3|rowz|   c3|
|   a7|   b7|rowz|   c7|
+-----+-----+----+-----+

# case 2: joined_df.union(df.select(*joined_df.columns))
+----+-----+-----+-----+
|  id|col_a|col_b|col_c|
+----+-----+-----+-----+
|rowz|   a3|   b3|   c3|
|rowz|   a7|   b7|   c7|
|rowx|   a0|   b0|   c0|
|rowx|   a1|   b1|   c1|
|rowy|   a2|   b2|   c2|
|rowz|   a3|   b3|   c3|
|rowy|   a4|   b4|   c4|
|rowy|   a5|   b5|   c5|
|rowx|   a6|   b6|   c6|
|rowz|   a7|   b7|   c7|
|rowy|   a8|   b8|   c8|
|rowx|   a9|   b9|   c9|
+----+-----+-----+-----+
</pre>

So, the solution is simple. Just add the <b>select</b> command to the 2nd dataframe (inside the <b>union</b> method). By doing so, the column’s order in the 2nd dataframe will follow the column’s order in the 1st dataframe (outside the <b>union</b> method). Feel free to clarify this :)
