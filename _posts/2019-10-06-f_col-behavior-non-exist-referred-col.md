---
title: 'F.col() Behavior With Non-Existing Referred Columns on Dataframe Operations'
date: 2019-10-06
permalink: /posts/2019/10/column-function-behavior-on-non-exist-referred-columns/
tags:
  - pyspark
  - column
  - dataframe
---

I came across an odd use case when applying **F.col()** on certain dataframe operations on PySpark v.2.4.0. Please note that the context of this issue is on Oct 6, 2019. Such an issue might have been solved in the future.

Suppose a dataframe **df** has three columns, namely column **A, B**, and **C**. Consider the following operation.

```
selected_df = df.select(*['A', 'B'])
filtered_df = selected_df.filter(F.col('C') == 'x')
```

The above operations simply do the followings:

<ul>
<li>Select column <b>A</b> and <b>B</b> from the original dataframe (<b>df</b>), </li>
<li>Store the resulting dataframe to <b>selected_df</b>,</li>
<li>Retrieve rows from <b>selected_df</b> where the value of column <b>C</b> equals to 'x',</li>
<li>Store the resulting dataframe to <b>filtered_df</b></li>
</ul>

Such operations **run successfully** and the **resulting dataframe is correct** as well even though the <b>selected_df</b> does not have column <b>C</b> anymore.

However, if we replace <b>F.col()</b> with another type of column referencing, an error is thrown.

```
filtered_df = selected_df.filter(selected_df['C'] == 'x')
```

The above operation throws the following exception: <i>"pyspark.sql.utils.AnalysisException: 'Cannot resolve column name "C" among (A, B);'"</i>.

Okay. Let's move on to another dataframe operation, such as **withColumn** combined with a conditional check.

```
new_df = selected_df.withColumn('D', F.when(F.col('C') == 'x', F.lit('YES')).otherwise(F.lit('NO')))
```

Such an operation failed with an exception denoting that Spark could not resolve column **C** provided input columns **A** and **B**. Replacing **F.col()** with **selected_df[column]** also throws the same error.

Let's try another operation.

```
new_df = selected_df.withColumn('D', F.col('C'))

AND

new_df = selected_df.withColumn('D', selected_df['C'])
```

The result? Also failed with the same exception as before.

Now the question should be why using <b>F.col()</b> with non-existing referred columns (in this case, column **C**) makes the <b>filter</b> operation run successfully, while it fails on other operations?

Have you ever experienced the same use case? I'd love to hear your thoughts on this.

Thank you for reading.
