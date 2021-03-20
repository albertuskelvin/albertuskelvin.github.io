---
title: 'Handling Dot Character in Spark Dataframe Column Name (Partial Solution)'
date: 2020-01-02
permalink: /posts/2020/01/handling-dot-character-spark-dataframe-column-name/
tags:
  - spark
  - dataframe
  - dot character
---

A few days ago I came across a case where I needed to define a dataframe's column name with a special character, that is a dot ('.'). Take a look at thee following schema example.

```
root
 |-- p.x: string (nullable = true)
 |-- p.y: string (nullable = true)
```

However, the following exception occurred when executing few kinds of operations, such as `select`, `filter`, and `groupby`.

```
pyspark.sql.utils.AnalysisException: "cannot resolve '`p.x`' given input columns: [p.x, p.y]"
```

If you're aware of dataframe creation, this `dot (.)` character is used as the reference to the sub-columns contained within a nested column. There might be a possibility that using `dot (.)` in a non-nested column makes Spark looks for the sub-column (specified after the dot). I've written an article about how to create nested columns in PySpark. You can find it <a href="https://albertuskelvin.github.io/posts/2020/01/create-nested-columns-spark/">here</a>.

After searching for the solutions, turns out that we only need to wrap the column name with backticks ('\`\`'). I tried this solution and it worked only for several operations, such as `select`, `filter`, and `groupby` (<b>as far as I've tried</b>). Using backticks for other operations, such as `drop` doesn’t work.

```
=== With backticks ===

df.drop('`p.x`').printSchema()

Resulting schema:
root
 |-- p.x: string (nullable = true)
 |-- p.y: string (nullable = true)
 

=== Without backticks ===

df.drop('p.x').printSchema()

Resulting schema:
root
 |-- p.y: string (nullable = true)
```

To conclude, I think this solution (backticks wrapping) does not cover all the dataframe transformations. Applying it on the real environment might be cumbersome since we might need to list all the transformations for which this solution can work properly.

Do you have other suggestions? I'd love to know your thoughts.
