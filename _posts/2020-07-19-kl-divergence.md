---
title: 'Kullback-Leibler Divergence for Empirical Probability Distributions in Spark'
date: 2020-07-19
permalink: /posts/2020/07/kullback-leibler-divergence-spark/
tags:
  - statistics
  - machine learning
  - distribution
  - kl divergence
  - kullback-leibler
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/07/two-sample-kolmogorov-smirnov-test/">post</a>, I mentioned about the basic concept of two-sample Kolmogorov-Smirnov (KS) test and its implementation in Spark (Scala API).

The implementation can be found on this Github <a href="https://github.com/albertuskelvin/distribution-shift">repo</a>. Just for quick information, the repo provides a collection of methods to compare two empirical distributions. Currently, there are only two methods available, namely two-sample KS test and Kullback-Leibler (KL) divergence.

In this post, we're going to look at how to implement KL divergence in Spark (Scala API) to compare two empirical probability distributions. 

As a refresher, KL divergence basically informs how much information loss when approaching distribution `P` with distribution `Q`. The KL divergence formula for discrete probability distributions `P` and `Q` which are defined in the same probability space `PS` is stated in the following form.

```
Dkl(P || Q) = SUM(x E PS) P(x) log(P(x) / Q(x))
```

Since we're going to apply KL divergence on empirical probability distributions, we can simply use the above formula.

Suppose we're going to compare the following samples (how much information loss when approaching `P` with `Q`).

```
Sample P
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

Sample Q
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

## Smoothing

Notice that it's possible for an observation exists in `P` but doesn't exist in `Q`. This might become an issue since we'll get the following state when applying the formula.

```
P(x) log(P(x) / Q(x))

P(x) log(P(x) / 0)

P(x) log(inf) which approaches to inf.
```

So it basically means that when an observation exists in `P` but not in `Q`, then simply those two empirical distributions are absolutely different. Because when approaching `P` with `Q`, we'll lose so much information.

However, since what we have are only <b>samples</b>, there might be a chance that we haven't included the unobserved data in our calculation. In addition, I think this unobserved data should be considered especially for the case of random sampling.

Think of it with the following case.

Suppose a probability distribution A (right skewed) has an intersection area with a probability distribution B (left skewed) in their tails. By random sampling, suppose that we didn't retrieve samples from the tails, therefore, makes our sample A and sample B don't have common observations. In this case, KL divergence will state that both distributions are completely different. However, we know that A and B have an intersection area. Theoretically, both distributions shouldn't be considered as completely different since there are common values in the intersection area.

Therefore, one of the simplest solutions to this issue is by considering the unobserved events in our calculation. Generally, including the unobserved events to the calculation is called as smoothing.

Various approaches are valid though for smoothing. One of the approaches for smoothing is called as <i>absolute discounting</i>. You can find more details about the approach <a href="https://www.cs.bgu.ac.il/~elhadad/nlp09/KL.html">here</a>.

However, in my repo, I use a simple approach, that is by assigning the frequency of unobserved events with a very small value (e.g. 0.0001). This assignment basically assumes that the unobserved events are very rare.

Here's the Spark code for smoothing step.

```scala
private def smoothSample(targetDf: DataFrame, complementDf: DataFrame): DataFrame = {
    val unObservedTargetSampleDf = complementDf
      .join(targetDf, Seq(DistributionGeneralConstants.DSHIFT_COMPARED_COL), "left_anti")
      .distinct()

    val unObservedTargetSampleCountDf =
      unObservedTargetSampleDf.withColumn(
        KLDivergenceConstants.DSHIFT_KLDIV_SAMPLE_FREQUENCY,
        F.lit(KLDivergenceConstants.DSHIFT_KLDIV_UNOBSERVED_SAMPLE_FREQUENCY))

    val observedTargetSampleCountDf = targetDf
      .groupBy(DistributionGeneralConstants.DSHIFT_COMPARED_COL)
      .count()
      .withColumnRenamed("count", KLDivergenceConstants.DSHIFT_KLDIV_SAMPLE_FREQUENCY)

    val columns = observedTargetSampleCountDf.columns
    unObservedTargetSampleCountDf
      .select(columns.head, columns.tail: _*)
      .union(observedTargetSampleCountDf)
}
```

From the above method, `targetDf` is the sample that will be smoothed. Meanwhile, `complementDf` is the sample from which the unobserved events are taken.

Here's what each variable stores.
- <i>unObservedTargetSampleDf</i> => events that exist on the <i>complementDf</i> but not in <i>targetDf</i>
- <i>unObservedTargetSampleCountDf</i> => frequency of unobserved events (assigned with a very small value)
- <i>observedTargetSampleCountDf</i> => frequency of observed events in <i>targetDf</i>

Lastly, we combine the observed and unobserved events for <i>targetDf</i> along with their frequencies.

Alright, using the above samples example, here's what we got after smoothing.

```
Event	|   Frequency
=========================
7	|    0.0001
18	|    0.0001
3	|       1
4	|       2
5	|       1
9	|       1
10	|       2
25	|       1
30	|       1
=========================


Event	|   Frequency
=========================
3	|    0.0001
9	|    0.0001
10	|    0.0001
30	|    0.0001
4	|       1
5	|       2
7	|       1
18	|       1
25	|       4
=========================
```

## Computing Probability Distributions

After getting the events' frequency for both distribution `P` and `Q`, we proceed by computing the probability of each event.

Here's the Spark code.

```scala
private def computeProbaDistr(df: DataFrame, probaDistrColName: String): DataFrame = {
    val totalObservations =
      df.agg(F.sum(F.col(KLDivergenceConstants.DSHIFT_KLDIV_SAMPLE_FREQUENCY))).first.get(0)

    df.withColumn(
        probaDistrColName,
        F.col(KLDivergenceConstants.DSHIFT_KLDIV_SAMPLE_FREQUENCY) / F.lit(totalObservations))
      .drop(KLDivergenceConstants.DSHIFT_KLDIV_SAMPLE_FREQUENCY)
}
```

Simply, the above method first compute the number of observations (`totalObservations`) by summing up the events' frequency.

Afterwards, a new column is created for storing the events' probability. This is simply achieved by calculating the ratio of event frequency and the total observations.

Here's what we got for our example.

```
Event	|   Probability
=========================
7	| 0.0001 / 9.0002
18	| 0.0001 / 9.0002
3	|   1 / 9.0002
4	|   2 / 9.0002
5	|   1 / 9.0002
9	|   1 / 9.0002
10	|   2 / 9.0002
25	|   1 / 9.0002
30	|   1 / 9.0002
=========================


Event	|   Frequency
=========================
3	| 0.0001 / 9.0004
9	| 0.0001 / 9.0004
10	| 0.0001 / 9.0004
30	| 0.0001 / 9.0004
4	|   1 / 9.0004
5	|   2 / 9.0004
7	|   1 / 9.0004
18	|   1 / 9.0004
25	|   4 / 9.0004
=========================
```

## Computing KL divergence statistic

As the final step, we can just calculate the statistic of KL divergence by applying its formula.

Here's the Spark code.

```scala
private def computeKLDivStatistic(
    originSampleProbaDistrDf: DataFrame,
    currentSampleProbaDistrDf: DataFrame): Double = {
    val pairOfProbaDistrDf = originSampleProbaDistrDf
      .join(
        currentSampleProbaDistrDf,
        Seq(DistributionGeneralConstants.DSHIFT_COMPARED_COL),
        "inner")

    pairOfProbaDistrDf
      .withColumn(
        KLDivergenceConstants.DSHIFT_KLDIV_STATISTIC,
        F.col(KLDivergenceConstants.DSHIFT_KLDIV_ORIGIN_PROBA_DISTR) * F.log(
          F.col(KLDivergenceConstants.DSHIFT_KLDIV_ORIGIN_PROBA_DISTR) / F.col(
            KLDivergenceConstants.DSHIFT_KLDIV_CURRENT_PROBA_DISTR))
      )
      .drop(
        KLDivergenceConstants.DSHIFT_KLDIV_ORIGIN_PROBA_DISTR,
        KLDivergenceConstants.DSHIFT_KLDIV_CURRENT_PROBA_DISTR)
      .agg(F.sum(F.col(KLDivergenceConstants.DSHIFT_KLDIV_STATISTIC)))
      .first
      .get(0)
      .asInstanceOf[Double]
}
```

From the above method, we can see that `pairOfProbaDistrDf` stores the pairs of event probability.

The formula is then applied on each pair.

Lastly, we sum up the result of applying the formula. This sum is the statistic returned by KL divergence.

Here's what we got for our example.

```
Event	|   Probability P  |   Probability Q   |     Applied formula
=========================================================================
7	| 0.0001 / 9.0002  |     1 / 9.0004    |  P(7) log(P(7) / Q(7))
18	| 0.0001 / 9.0002  |     1 / 9.0004    |  P(18) log(P(18) / Q(18))
3	|   1 / 9.0002     |  0.0001 / 9.0004  |  P(3) log(P(3) / Q(3))
4	|   2 / 9.0002     |     1 / 9.0004    |  P(4) log(P(4) / Q(4))
5	|   1 / 9.0002     |     2 / 9.0004    |  P(5) log(P(5) / Q(5))
9	|   1 / 9.0002     |  0.0001 / 9.0004  |  P(9) log(P(9) / Q(9))
10	|   2 / 9.0002     |  0.0001 / 9.0004  |  P(10) log(P(10) / Q(10))
25	|   1 / 9.0002     |     4 / 9.0004    |  P(25) log(P(25) / Q(25))
30	|   1 / 9.0002     |  0.0001 / 9.0004  |  P(30) log(P(30) / Q(30))
=========================================================================

Statistic = SUM(Applied formula)
```
