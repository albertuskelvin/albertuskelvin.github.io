---
title: 'Retrieving Rows with Duplicate Values on the Columns of Interest in Spark'
date: 2020-06-06
permalink: /posts/2020/06/spark-retrieves-duplicates.md/
tags:
  - spark
  - scala
  - duplicates
  - groupby
  - window
---

There are several ways of removing duplicate rows in Spark. Two of them are by using `distinct()` and `dropDuplicates()`. The former lets us to remove rows with the same values on all the columns. Meanwhile, the latter lets us to remove rows with the same values on multiple selected columns.

But how would you identify which rows with the same values on the columns of interest?

Two of the simplest ways are by leveraging the `groupBy()` and `Window` approaches.

For the sake of clarity, the below dataframe will be our illustration.

```
Name: df

=======================================
    A    |    B    |    C    |    D
=======================================
    p    |    q    |    r    |    s
    p    |    q    |    r    |    s
    q    |    p    |    r    |    s
    r    |    s    |    s    |    r
    q    |    p    |    p    |    q
    q    |    p    |    s    |    r
    p    |    s    |    p    |    s
    s    |    r    |    s    |    r
    s    |    r    |    p    |    p
=======================================
```

With the above dataframe, let's retrieve all rows with the same values on column <b>A</b> and <b>B</b>. These columns are our columns of interest.

## groupBy()

The idea is pretty simple.

We could just group the dataframe based on the columns of interest, then count the occurrence for each of the distinct values on the columns of interest.

Here's the code snippet in Scala.

```scala
val colsOfInterestOccurenceDf = df
           .groupBy("A", "B")
           .count()
```

The above will result in the following.

```
=================================
    A    |    B    |    count    
=================================
    p    |    q    |      2
    q    |    p    |      3
    r    |    s    |      1
    p    |    s    |      1
    s    |    r    |      2
=================================
```

Let's retrieve the values of `(A, B)` that occur more than once.

```scala
val duplicateColsOfInterestDf = colsOfInterestOccurenceDf.filter(F.col("count") > 1)
```

We'll get the following.

```
=================================
    A    |    B    |    count    
=================================
    p    |    q    |      2
    q    |    p    |      3
    s    |    r    |      2
=================================
```

Up til now, we've known which values of the columns of interest that are duplicates. How would we retrieve the original dataframe with all the columns (A, B, C, and D)?

The simplest approach is just joining the `duplicateColsOfInterestDf` with the original dataframe. In this case, inner join will be suitable since we might not want to add unnecessary task of filtering out null values if we use left join.

```scala
val duplicateRowsDf = df
        .join(duplicateColsOfInterestDf)
        .drop("count")
```

We should get back a dataframe with duplicate rows of the columns of interest such as the following.

```
=======================================
    A    |    B    |    C    |    D
=======================================
    p    |    q    |    r    |    s
    p    |    q    |    r    |    s
    q    |    p    |    r    |    s
    q    |    p    |    p    |    q
    q    |    p    |    s    |    r
    s    |    r    |    s    |    r
    s    |    r    |    p    |    p
=======================================
```

## Window

The idea is quite similar to the previous approach. We group each value of the columns of interest into the corresponding category and then calculate the number of elements within the category.

Take a look at the code below.

```scala
val dataPartitioner = Window.partitionBy("A", "B")

val appendedNumOfOccurenceDf = df
        .withColumn("count", 
                    F.count("*").over(dataPartitioner))
```

Simply put, the `dataPartitioner` specifies which columns that are used to partition the dataframe. In this case, we use the columns of interest <b>A</b> and <b>B</b>. Therefore, the dataframe would look like this in runtime.

```
Partition A
=======================================
    A    |    B    |    C    |    D    
=======================================
    p    |    q    |    r    |    s    
    p    |    q    |    r    |    s    
=======================================

Partition B
=======================================
    A    |    B    |    C    |    D    
=======================================
    q    |    p    |    r    |    s  
    q    |    p    |    p    |    q   
    q    |    p    |    s    |    r   
=======================================

Partition C
=======================================
    A    |    B    |    C    |    D  
=======================================
    r    |    s    |    s    |    r  
=======================================

Partition D
=======================================
    A    |    B    |    C    |    D    
=======================================
    p    |    s    |    p    |    s    
=======================================

Partition E
=======================================
    A    |    B    |    C    |    D    
=======================================
    s    |    r    |    s    |    r    
    s    |    r    |    p    |    p   
=======================================
```

Afterwards, we can proceed by counting the number of rows for each partition (`F.count("*").over(dataPartitioner)`).

The `appendedNumOfOccurenceDf` will result in the following.

```
=====================================================
    A    |    B    |    C    |    D    |    count
=====================================================
    p    |    q    |    r    |    s    |      2
    p    |    q    |    r    |    s    |      2
    q    |    p    |    r    |    s    |      3
    r    |    s    |    s    |    r    |      1
    q    |    p    |    p    |    q    |      3
    q    |    p    |    s    |    r    |      3
    p    |    s    |    p    |    s    |      1
    s    |    r    |    s    |    r    |      2
    s    |    r    |    p    |    p    |      2
=====================================================
```

As you can see, we directly got the original dataframe with a new appended column <i>count</i>. This <i>count</i> column simply denotes the number of occurrence for the corresponding column.

To retrieve the duplicates, we can just filter out the rows with `count = 1`.

```scala
val duplicateRowsDf = appendedNumOfOccurenceDf
       .filter(F.col("count") > 1)
       .drop("count")
```

We'll get the same result as one in the previous approach.

Notice that in this approach, we don't need to perform join to get all the columns. Using the previous approach requires us to perform shuffle twice, namely `groupBy()` and `join()`. Meanwhile, in this approach, we only perform shuffle once, namely when we partition the data according to the columns of interest (`Window.partitionBy()`).
