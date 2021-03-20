---
title: 'Two-sample Kolmogorov-Smirnov Test for Empirical Distributions in Spark'
date: 2020-07-15
permalink: /posts/2020/07/two-sample-kolmogorov-smirnov-test/
tags:
  - statistic
  - machine learning
  - data distribution
  - kolmogorov smirnov test
  - empirical distribution
  - cumulative distribution function
---

Kolmogorov-Smirnov (KS) test is a non-parametric test for the equality of probability distributions.

Basically, KS test is categorised into two types, namely one-sample KS test and two-sample KS test. The former let us test whether a sample comes from a known distribution (e.g. normal, uniform, binomial, etc.). Meanwhile, the latter lets us test whether two samples come from the same distribution.

A real use case for this is when we want to evaluate whether our test data has different distribution with the train data. Another use case is when we want to evaluate our ML model performance.

In this post, we’re going to discuss about two-sample KS test.

A few days ago I’ve created an implementation of two-sample KS test for comparing two empirical distributions in Spark.

You can find the Github repo <a href="https://github.com/albertuskelvin/distribution-shift">here</a>.

Let’s take a look at how two-sample KS test works.

Suppose we’d like to compare the following two samples.

```
Sample A
========
3
4
4
5
9
10
10
25
30

Sample B
========
4
5
5
7
18
25
25
25
25
```

### Computing Cumulative Count

We sort both samples in ascending way and compute the cumulative count, such as the followings.

```
Sample A  |   Cum Count
=======================
3         |   1
4         |   3
4         |   3
5         |   4
9         |   5
10        |   7
10        |   7
25        |   8
30        |   9

Sample B  |   Cum Count
=======================
4         |   1
5         |   3
5         |   3
7         |   4
18        |   5
25        |   9
25        |   9
25        |   9
25        |   9
```

The following is the implementation code in Spark (Scala).

```scala
private def computeCumulativeSum(df: DataFrame): DataFrame = {
    val window = Window.orderBy(DistributionGeneralConstants.DSHIFT_COMPARED_COL)
    df.withColumn(
      KSTestConstants.DSHIFT_CUMSUM,
      F.count(DistributionGeneralConstants.DSHIFT_COMPARED_COL).over(window))
}
```

### Computing Empirical Cumulative Distribution Function

Next, we compute the empirical CDF (ECDF). As a refresher, CDF is the probability of getting value that is less than or equal to a certain value.

Mathematically, we can write CDF as the following.

```
CDF(x) = N(value <= x) / total observations
```

Using the above example, below is the calculated ECDF.

```
Sample A  |   Cum Count   |   ECDF A
====================================
3         |   1           |   1/9
4         |   3           |   3/9
4         |   3           |   3/9
5         |   4           |   4/9
9         |   5           |   5/9
10        |   7           |   7/9
10        |   7           |   7/9
25        |   8           |   8/9
30        |   9           |   9/9

Sample B  |   Cum Count   |   ECDF B
====================================
4         |   1           |   1/9
5         |   3           |   3/9
5         |   3           |   3/9
7         |   4           |   4/9
18        |   5           |   5/9
25        |   9           |   9/9
25        |   9           |   9/9
25        |   9           |   9/9
25        |   9           |   9/9
```

The following is the implementation code in Spark (Scala).

```scala
private def computeEmpiricalCDF(df: DataFrame, renamedECDF: String): DataFrame = {
    val totalObservations = df.agg(F.max(KSTestConstants.DSHIFT_CUMSUM)).head.get(0)
    df.withColumn(renamedECDF, F.col(KSTestConstants.DSHIFT_CUMSUM) / F.lit(totalObservations))
      .select(DistributionGeneralConstants.DSHIFT_COMPARED_COL, renamedECDF)
}
```

### Computing the Absolute ECDF Differences

The calculation of ECDF difference is performed pair-wise (the same sample’s value).

The problem is that there might be several values in <b>Sample A</b> that don’t exist in <b>Sample B</b> (vice versa). Since the difference calculation must be applied on the pair of the same values from each sample, we need to take a bit work-around.

One of the simplest solution is to add a new column for the ECDF of the other sample with null as the value, such as the following.

```
Sample A  |   Cum Count   |   ECDF A  |   ECDF B
=================================================
3         |   1           |   1/9     |   null
4         |   3           |   3/9     |   null
4         |   3           |   3/9     |   null
5         |   4           |   4/9     |   null
9         |   5           |   5/9     |   null
10        |   7           |   7/9     |   null
10        |   7           |   7/9     |   null
25        |   8           |   8/9     |   null
30        |   9           |   9/9     |   null

Sample B  |   Cum Count   |   ECDF A  |   ECDF B
=================================================
4         |   1           |   null    |   1/9
5         |   3           |   null    |   3/9
5         |   3           |   null    |   3/9
7         |   4           |   null    |   4/9
18        |   5           |   null    |   5/9
25        |   9           |   null    |   9/9
25        |   9           |   null    |   9/9
25        |   9           |   null    |   9/9
25        |   9           |   null    |   9/9
```

Afterwards, we union both samples such as the followings.

```
=======================================
Sample Value  |   ECDF A    |   ECDF B
=======================================
3             |	  1/9       |     null
4             |	  3/9       |     null
4             |	  3/9       |	  null
5             |	  4/9       |	  null
9             |	  5/9       |	  null
10            |	  7/9       |	  null
10            |	  7/9       |	  null
25            |	  8/9       |	  null
30            |	  9/9       |	  null
4             |	  null      |	  1/9
5             |	  null      |	  3/9
5             |	  null      |	  3/9
7             |	  null      |	  4/9
18            |	  null      |	  5/9
25            |	  null      |	  9/9
25            |	  null      |	  9/9
25            |	  null      |	  9/9
25            |	  null      |	  9/9
=====================================
```

Here’s the code in Spark.

```scala
val sampleOneWithECDFSampleTwo = ecdfSampleOne.withColumn(KSTestConstants.DSHIFT_ECDF_SAMPLE_TWO, F.lit(null))
val sampleTwoWithECDFSampleOne = ecdfSampleTwo.withColumn(KSTestConstants.DSHIFT_ECDF_SAMPLE_ONE, F.lit(null))

val unionedSamples = sampleOneWithECDFSampleTwo.unionByName(sampleTwoWithECDFSampleOne)
```

Next, our task is only to fill in the nulls with the appropriate value. Technically, we fill in the nulls for each pair of `Sample Value` and `ECDF`.

Let’s take a look at the examples.

We first fill in the nulls in `ECDF A`.

Here’s how we fill in the nulls in `ECDF A`.
- Sort the rows with the pair of <b>Sample Value</b> and <b>ECDF A</b> as the key
- The sorting is performed in ascending way in which null values in <b>ECDF A</b> are grouped together in the last group. For instance, [null, 5, 3, null, 4, null] will be sorted to [3, 4, 5, null, null, null]
- In <b>ECDF A</b> column, create a window starting from the first row up to the current row. Fill in the null by the last non-null value in the window
- Fill in the remaining nulls with 0

Here’s the result.

```
===========================================================
Sample Value  | ECDF A  | Last Non-null Value in the Window
===========================================================
3             | 1/9     |           1/9
4             |	3/9     |           3/9
4             |	3/9     |           3/9
4             |	null    |           3/9
5             |	4/9     |           4/9
5             |	null    |           4/9
5             |	null    |           4/9
7             |	null    |           4/9
9             |	5/9     |           5/9
10            |	7/9     |           7/9
10            |	7/9     |           7/9
18            |	null    |           7/9
25            |	8/9     |           8/9
25            |	null    |           8/9
25            |	null    |           8/9
25            |	null    |           8/9
25            |	null    |           8/9
30            |	9/9     |           9/9
===========================================================
```

The same algorithm can be applied to fill in null values in `ECDF B`.

Here’s the result.

```
===========================================================
Sample Value  | ECDF B  | Last Non-null Value in the Window
===========================================================
3             | null    |            0
4             | 1/9     |           1/9
4             | null    |           1/9
4             | null    |           1/9
5             | 3/9     |           3/9
5             | 3/9     |           3/9
5             | null    |           3/9
7             | 4/9     |           4/9
9             | null    |           4/9
10            | null    |           4/9
10            | null    |           4/9
18            | 5/9     |           5/9
25            | 9/9     |           9/9
25            | 9/9     |           9/9
25            | 9/9     |           9/9
25            | 9/9     |           9/9
25            | null    |           9/9
30            | null    |           9/9
===========================================================
```

Here’s the Spark code for filling in the nulls.

```scala
private def getWindowFillers: Seq[WindowSpec] = {
    val windowFillerSampleOne = Window
      .orderBy(
        Seq(
          F.col(DistributionGeneralConstants.DSHIFT_COMPARED_COL),
          F.col(KSTestConstants.DSHIFT_ECDF_SAMPLE_ONE).asc_nulls_last): _*)
      .rowsBetween(Window.unboundedPreceding, Window.currentRow)

    val windowFillerSampleTwo = Window
      .orderBy(
        Seq(
          F.col(DistributionGeneralConstants.DSHIFT_COMPARED_COL),
          F.col(KSTestConstants.DSHIFT_ECDF_SAMPLE_TWO).asc_nulls_last): _*)
      .rowsBetween(Window.unboundedPreceding, Window.currentRow)

    Seq(windowFillerSampleOne, windowFillerSampleTwo)
}

private def fillNullInUnionedSamples(df: DataFrame, windowFillers: Seq[WindowSpec]): DataFrame = {
    val windowFillerSampleOne = windowFillers.head
    val windowFillerSampleTwo = windowFillers.tail.head

    df.withColumn(
        KSTestConstants.DSHIFT_ECDF_SAMPLE_ONE,
        F.last(KSTestConstants.DSHIFT_ECDF_SAMPLE_ONE, ignoreNulls = true)
          .over(windowFillerSampleOne)
      )
      .withColumn(
        KSTestConstants.DSHIFT_ECDF_SAMPLE_TWO,
        F.last(KSTestConstants.DSHIFT_ECDF_SAMPLE_TWO, ignoreNulls = true)
          .over(windowFillerSampleTwo)
      )
      .na
      .fill(0.0)
}

// CALLER
val sampleOneWithECDFSampleTwo = ecdfSampleOne.withColumn(KSTestConstants.DSHIFT_ECDF_SAMPLE_TWO, F.lit(null))
val sampleTwoWithECDFSampleOne = ecdfSampleTwo.withColumn(KSTestConstants.DSHIFT_ECDF_SAMPLE_ONE, F.lit(null))

val unionedSamples = sampleOneWithECDFSampleTwo.unionByName(sampleTwoWithECDFSampleOne)

val windowFillers: Seq[WindowSpec] = getWindowFillers

val filledUnionedSamples = fillNullInUnionedSamples(unionedSamples, windowFillers)
```

Last but not least, let’s compute the absolute ECDF difference.

```
========================================================
Sample Value  | ECDF A  | ECDF B  |   | ECDF A - ECDF B|
========================================================
3             | 1/9     | 0       |           1/9
4             | 3/9     | 1/9     |           2/9
4             | 3/9     | 1/9     |           2/9
4             | 3/9     | 1/9     |           2/9
5             | 4/9     | 3/9     |           1/9
5             | 4/9     | 3/9     |           1/9
5             | 4/9     | 3/9     |           1/9
7             | 4/9     | 4/9     |           0
9             | 5/9     | 4/9     |           1/9
10            | 7/9     | 4/9     |           3/9
10            | 7/9     | 4/9     |           3/9
18            | 7/9     | 5/9     |           2/9
25            | 8/9     | 9/9     |           1/9
25            | 8/9     | 9/9     |           1/9
25            | 8/9     | 9/9     |           1/9
25            | 8/9     | 9/9     |           1/9
25            | 8/9     | 9/9     |           1/9
30            | 9/9     | 9/9     |           0
========================================================
```

### Finding the Maximum ECDF Difference

This step should be the easiest one.

From `| ECDF A - ECDF B |` column, we know that the maximum one is 3/9 or 1/3.

Therefore, the D-statistic for our samples is 1/3.

Here’s the Spark code.

```scala
private def getMaxECDFDifference(df: DataFrame): Double =
    df.agg(F.max(KSTestConstants.DSHIFT_ECDF_DIFFERENCE)).head.get(0).asInstanceOf[Double]
}
```
