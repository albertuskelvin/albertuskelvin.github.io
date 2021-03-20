---
title: 'The Investigation of Skewness & Kurtosis in Spark (Scala)'
date: 2020-08-18
permalink: /posts/2020/08/spark-sample-skewness-kurtosis/
tags:
  - statistics
  - maths
  - central moments
  - skewness
  - kurtosis
  - spark
  - scala
---

Applying central moment functions in Spark might be tricky, especially for skewness and kurtosis.

A few weeks ago I was experimenting with these functions. Fortunately, the IDE shows all the relevant functions which might fit my needs. For instance, the IDE suggests `var_pop` and `var_samp` for variance calculation. Such suggestions might make us aware of our initial goal of leveraging variance functions (at least for me). The same goes for standard deviation which has `stddev_pop` and `stddev_samp`.

However, seems that such kind of feature isn’t available for skewness & kurtosis.

The available function is just `skewness` for skewness and `kurtosis` for kurtosis. The problem is that I don’t know whether such functions are for population (parameters) or sample (statistics) calculation.

I decided, once again, to take a dive into the Spark’s code itself.

## Locating the Appropriate Modules

So I started from the most obvious location. Since all these central moment functions are included as Spark SQL functions, we basically summon them by the following.

```scala
import org.apache.spark.sql.functions

val mean = df.agg(avg(columnName)).head.get(0).asInstanceOf[Double]
val stddev_pop = df.agg(stddev_pop(columnName)).head.get(0).asInstanceOf[Double]
val stddev_samp = df.agg(stddev_samp(columnName)).head.get(0).asInstanceOf[Double]
val var_pop = df.agg(var_pop(columnName)).head.get(0).asInstanceOf[Double]
val var_samp = df.agg(var_samp(columnName)).head.get(0).asInstanceOf[Double]
```

Therefore, we can start from `functions.scala` module which is located <a href="https://github.com/apache/spark/blob/v2.4.4/sql/core/src/main/scala/org/apache/spark/sql/functions.scala">here</a>.

However, there is no any implementation for each function.

Looking further at the imported modules, seems that there might be a chance that the implementation are included there. The followings might be the best possible ones.

```scala
import org.apache.spark.sql.catalyst.expressions._
import org.apache.spark.sql.catalyst.expressions.aggregate._
```

Long story short, after searching for the correct location of `sql.catalyst.expressions`, I managed to find it <a href="https://github.com/apache/spark/tree/v2.4.4/sql/catalyst/src/main/scala/org/apache/spark/sql/catalyst/expressions">here</a>.

There is no any implementation for the central moment functions there. However, recall that there is also an import for `sql.catalyst.expressions.aggregate._` there. And yes, there’s indeed a package called `aggregate` which stores all the statistical modules.

Since our need is specified for central moments, just click on the `CentralMomentAgg.scala` module.

From there, the investigation began.

## The Investigation

As you might have seen, this module provides the central moment functions, such as standard deviation, variance, skewness, and kurtosis. As expected, both of the standard deviation and variance have two separated classes, one is for parameter while the other is for statistic.

Back to our original question, how about skewness and kurtosis?

Well, as you can see, the class names don’t inform us whether each of them acts as a parameter or statistic. Just take a deeper look at the applied formula.

Starting from skewness, recall that the population skewness formula is given by the following.

```
skewness_pop = E [((X - mu_pop) / stddev_pop) ^ 3]

X: the random variable
mu_pop: population mean
stddev_pop: population standard deviation
```

The implemented formula in `CentralMomentAgg.scala` is given as the following.

```
sqrt(n) * m3 / sqrt(m2 * m2 * m2)
``` 

where if `m` refers to `(X - mu)`, then `m2` refers to `(X - mu)^2` and `m3` refers to `(X - mu)^3`.

If you take a closer look, you might find that after replacing `stddev_pop` in `skewness_pop` with its formula and simplifying the `skewness_pop` equation, the implemented formula is the simplified version of `skewness_pop`.

To support the argument, recall that the sample skewness formula is given by the following.

```
skewness_samp = SUM(i=1 to n) [(xi - mu_samp) ^ 3] / n
                --------------------------------------
                           stddev_samp ^ 3
```

After replacing `stddev_samp` with the appropriate value (recall it has `n-1` factor) and simplifying the above equation, we don’t end up with the implemented formula.

Therefore, we may conclude that the implemented skewness in `CentralMomentAgg.scala` refers to the population skewness.

I performed the same step for kurtosis.

Recall that the population excess kurtosis is given as the following.

```
kurtosis_pop = E [((X - mu_pop) / stddev_pop) ^ 4]

excess_kurtosis_pop = kurtosis_pop - 3.0

X: the random variable
mu_pop: population_mean
stddev_pop: population standard deviation
```

The implemented formula in `CentralMomentAgg.scala` is given as the following.

```
n * m4 / (m2 * m2) - 3.0
``` 

where if `m` refers to `(X - mu)`, then `m2` refers to `(X - mu)^2` and `m4` refers to `(X - mu)^4`.

If you take a closer look, you might find that after replacing `stddev_pop` in `kurtosis_pop` with its formula and simplifying the `kurtosis_pop` equation, the implemented formula is the simplified version of `excess_kurtosis_pop`.

To support the argument, recall that the sample kurtosis formula is given by the following.

```
kurtosis_samp =   SUM(i=1 to n) [(xi - mu_samp) ^ 4] / n
                  --------------------------------------
                             stddev_samp ^ 4

excess_kurtosis_samp = kurtosis_samp - 3.0
```

After replacing `stddev_samp` with the appropriate value (recall it has `n-1` factor) and simplifying the above equation, we don’t end up with the implemented formula.

Therefore, we may conclude that the implemented kurtosis in `CentralMomentAgg.scala` refers to the population kurtosis, specifically the population excess kurtosis.

## The Implementation in Spark (Scala)

Knowing that both of skewness and kurtosis are for population, I decided to implement the sample version for both of them in Spark (Scala).

With this, I presume that `calculateMean` and `calculateStdDevSamp` are already implemented.

### Sample Skewness

```scala
def calculateSkewnessSamp(df: DataFrame, column: String): Double = {
  val observedDf = df.filter(!F.isnull(F.col(column)) && !F.isnan(F.col(column)))

  val observationsMean = calculateMean(observedDf, column)
  val observationsStdDev = calculateStdDevSamp(observedDf, column)

  val totalObservations = observedDf.count()

  val thirdCentralSampleMoment =
    observedDf
      .agg(F.sum(F.pow(F.col(column) - observationsMean, 3) / totalObservations))
      .head
      .get(0)
      .asInstanceOf[Double]
  val thirdPowerOfSampleStdDev = scala.math.pow(observationsStdDev, 3)

  thirdCentralSampleMoment / thirdPowerOfSampleStdDev
}
```

### Sample Excess Kurtosis

```scala
def calculateExcessKurtosisSamp(df: DataFrame, column: String): Double = {
  val observedDf = df.filter(!F.isnull(F.col(column)) && !F.isnan(F.col(column)))

  val observationsMean = calculateMean(observedDf, column)
  val observationsStdDev = calculateStdDevSamp(observedDf, column)

  val totalObservations = observedDf.count()

  val fourthCentralSampleMoment =
    observedDf
      .agg(F.sum(F.pow(F.col(column) - observationsMean, 4) / totalObservations))
      .head
      .get(0)
      .asInstanceOf[Double]
  val fourthPowerOfSampleStdDev = scala.math.pow(observationsStdDev, 4)

  (fourthCentralSampleMoment / fourthPowerOfSampleStdDev) - 3.0
}
```
