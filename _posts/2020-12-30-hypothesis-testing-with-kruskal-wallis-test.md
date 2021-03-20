---
title: 'Hypothesis Testing with the Kruskal-Wallis Test'
date: 2020-12-30
permalink: /posts/2020/12/hypothesis-testing-kruskal-wallis-test/
tags:
  - statistics
  - data science
  - hypothesis testing
  - kruskal wallis
  - non-parametric
---

The Kruskal-Wallis test is a non-parametric statistical test that is used to evaluate whether the medians of two or more groups are different. Since the test is non-parametric, it doesn't assume that the data comes from a particular distribution.

The test statistic calculated in this test is called the <b>H statistic</b>.

---

## Hypotheses

- H0: population medians are equal
- H1: population medians are unequal

---

## Assumptions

- Since this test is an extension of Mann-Whitney U Test, this test is commonly leveraged to evaluate differences between three or more groups
- The observations to evaluate should be in ordinal scale, ratio scale or interval scale
- The observations should be independent where there should be no relationship between the members in each group or between groups
- All groups should have the same shape distributions (number of peaks, symmetry, skewness)

---

## Test statistic formula

The general formula of the H statistic is the following (<b>Source</b>: Wikipedia - Kruskal–Wallis one-way analysis of variance):

```
H = (N - 1) *     SUM(i=1 to g) n_i * (r_i_hat - r_hat)^2
              -----------------------------------------------
              SUM(i=1 to g) SUM(j=1 to n_i) (r_i_j - r_hat)^2
```

Where:
- `n_i`: the number of observations in group `i`
- `r_i_j`: the rank (among all observations) of observation `j` from group `i`
- `N`: the total number of observations across all groups
- `r_i_hat`: the average rank of all observations in group `i` which is given by `(1/n_i) * (SUM(j=1 to n_i) r_i_j)`
- `r_hat`: the average of all the `r_i_j` which is given by `0.5 * (N + 1)`

In addition, you might see that if the combined observations doesn't consist of the same values, then the test statistic could be expressed alternatively as follow:

```
H = [[12 / (N(N + 1))] * SUM(i=1 to g) n_i * (r_i_hat)^2] - 3 * (N + 1)
```

---

### Notice

I mentioned about how to prove the above formula (H statistic when there are no tied values) in this <a href="https://albertuskelvin.github.io/posts/2020/12/kruskal-wallis-test-no-tied-values-formula/">post</a>.

---

## How to run the test

For the sake of clarity, let's use the following example data for the demonstration:

```
Group A: 30, 40, 50, 60
Group B: 10, 20, 70, 80
Group C: 100, 200, 300, 400
```

<b>Step 1.</b> Combine the observations from all the groups and sort in ascending order

```
Combined observations: 30, 40, 50, 60, 10, 20, 70, 80, 100, 200, 300, 400
Sorted observations: 10, 20, 30, 40, 50, 60, 70, 80, 100, 200, 300, 400
```

<b>Step 2.</b> Assign ranks to the sorted observations

```
Sorted observations: 10, 20, 30, 40, 50, 60, 70, 80, 100, 200, 300, 400
Ranks: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12
```

In case there are duplicate values, the rank is assigned by the following way:
- Assign normal ranks like the one in the above example
- Take the average of the ranks for the duplicate values

Here's a simple example.

```
Sorted observations: 10, 10, 10, 30, 40, 40, 50
Normal ranks: 1, 2, 3, 4, 5, 6, 7
Averaged ranks: 2, 2, 2, 4, 5.5, 5.5, 7
```

<b>Step 3.</b> Calculate the H statistic

Recall that the H statistic is given by the following:

```
H = (N - 1) *     SUM(i=1 to g) n_i * (r_i_hat - r_hat)^2
              -----------------------------------------------
              SUM(i=1 to g) SUM(j=1 to n_i) (r_i_j - r_hat)^2
```

In this case, `g` is the number of groups which is 3 (Group A, B, and C).

Before executing the formula, let's calculate the expressions within the formula first.

The first one is `r_i_hat` which is the average rank of all observations in group `i`. It equals to `(1/n_i) * (SUM(j=1 to n_i) r_i_j)`.

Applying the formula to our data yields the following.

```
r_i_j: the rank (among all observations) of observation j from group i

Group A
r_1_hat = (1/4) * (r_1_1 + r_1_2 + r_1_3 + r_1_4)
r_1_hat = (1/4) * (3 + 4 + 5 + 6) = 1/4 * 18 = 4.5

Group B
r_2_hat = (1/4) * (r_2_1 + r_2_2 + r_2_3 + r_2_4)
r_2_hat = (1/4) * (1 + 2 + 7 + 8) = 1/4 * 18 = 4.5

Group C
r_3_hat = (1/4) * (r_3_1 + r_3_2 + r_3_3 + r_3_4)
r_3_hat = (1/4) * (9 + 10 + 11 + 12) = 1/4 * 42 = 10.5
```

The second one is `r_hat` which is the average of all the `r_i_j`. It equals to `0.5 * (N + 1)`.

Applying the formula to our data yields the following.

```
r_hat = 0.5 * (12 + 1) = 0.5 * 13 = 6.5
```

With all the above results, let's compute the H statistic.

```
H = (N - 1) *     SUM(i=1 to g) n_i * (r_i_hat - r_hat)^2
              -----------------------------------------------
              SUM(i=1 to g) SUM(j=1 to n_i) (r_i_j - r_hat)^2

H = (12 - 1) * [(4 * (4.5 - 6.5)^2) + (4 * (4.5 - 6.5)^2) + (4 * (10.5 - 6.5)^2)]
               ------------------------------------------------------------------
                 [(3 - 6.5)^2 + (4 - 6.5)^2 + ... + (11 - 6.5)^2 + (12 - 6.5)^2]

H = 11 *                             [(4 * 4) + (4 * 4) + (4 * 16)]
         -----------------------------------------------------------------------------------------
         [12.25 + 6.25 + 2.25 + 0.25 + 30.25 + 20.25 + 0.25 + 2.25 + 6.25 + 12.25 + 20.25 + 30.25]

H = 11 * (96 / 143)

H = 7.3843
```

<b>Step 4.</b> State the conclusion.

After computing the H statistic, we compare the value to a critical value of chi-squared with `g - 1` degrees of freedom (`g` is the number of groups. In our example above, `g` is 3) and an alpha level. This critical value could be retrieved from the chi-squared probability distribution's table. Let's denote his critical value as `Hc`.

If `H` is bigger than `Hc`, then reject the null hypothesis. Otherwise, there is no evidence that the population medians are not equal.

---

## References

- Wikipedia: Kruskal–Wallis one-way analysis of variance
- statisticshowto: Kruskal Wallis H Test: Definition, Examples & Assumptions
