---
title: 'Kruskal-Wallis Test Statistic Formula Derivation When No Tied Values Exist'
date: 2020-12-30
permalink: /posts/2020/12/kruskal-wallis-test-no-tied-values-formula/
tags:
  - statistics
  - data science
  - hypothesis testing
  - kruskal wallis
  - non-parametric
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/12/hypothesis-testing-kruskal-wallis-test/">post</a>, I mentioned about the general formula of the H statistic is the following (<b>Source</b>: Wikipedia - Kruskal–Wallis one-way analysis of variance):

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
H = [[12 / (N * (N + 1))] * SUM(i=1 to g) n_i * (r_i_hat)^2] - 3 * (N + 1)    --> We're going to prove this
```

In this post, we're going to look at how to derive the above formula (H statistic without tied values).

---

## Start from the denominator

We'll start from the denominator. Let's take a look at what the denominator would be if there're no tied values.

Expanding the denominator yields the following.

```
SUM(i=1 to g) SUM(j=1 to n_i) (r_i_j - r_hat)^2

SUM(i=1 to g) SUM(j=1 to n_i) (r_i_j)^2 - 2*(r_i_j)*(r_hat) + (r_hat)^2        (A)
```

---

## Re-structure the formula for rank average

Next, we'll leverage the formula of `r_hat` which is `(N + 1) / 2`. It becomes the following.

```
(N + 1) / 2 = [SUM(i=1 to g) SUM(j=1 to n_i) r_i_j] / N

N * (N + 1) / 2 = SUM(i=1 to g) SUM(j=1 to n_i) r_i_j            (B)
```

---

## Express (A) in form of (B)

Next, we'll express each term in `(A)` in form of `(B)`.

```
SUM(i=1 to g) SUM(j=1 to n_i) (r_i_j)^2 - 2*(r_i_j)*(r_hat) + (r_hat)^2

[SUM(i=1 to g) SUM(j=1 to n_i) (r_i_j)^2] \
- [SUM(i=1 to g) SUM(j=1 to n_i) 2 * (r_i_j) * (r_hat)] \
+ [SUM(i=1 to g) SUM(j=1 to n_i) (r_hat)^2]                     (C)
```

The term `[SUM(i=1 to g) SUM(j=1 to n_i) (r_hat)^2]` simply states that `(r_hat)^2` appears `N` times. Therefore, the term becomes `N * (r_hat)^2` or `N * (N + 1)^2 / 4`.

Meanwhile, the term `[SUM(i=1 to g) SUM(j=1 to n_i) 2 * (r_i_j) * (r_hat)]` can be replaced by `2 * r_hat * N * (N + 1) / 2` or `0.5 * N * (N + 1)^2`.

Last but not least, the term `[SUM(i=1 to g) SUM(j=1 to n_i) (r_i_j)^2]` is basically in the form of `1^2 + 2^2 + 3^2 + ... + n^2`. Recall that such a sum of squared can be expressed as `n * (n + 1) * (2n + 1) / 6` where `n = N`.

Therefore, `(C)` can be expressed as the following.

```
[N * (N + 1) * (2N + 1) / 6] - [0.5 * N * (N + 1)^2] + [N * (N + 1)^2 / 4]
```

And expanding the above yields the following.

```
{ [2 * N * (N + 1) * (2N + 1)] - [6 * N * (N + 1)^2] + [3 * N * (N + 1)^2] } / 12

{ [N + 1] * [2N(2N+1) - 6N(N+1) + 3N(N+1)] } / 12

{ [N + 1] * [N^2 - N] } / 12

{ [N + 1] * N * [N - 1] } / 12           (D)
```

To conclude all the process above, the denominator can be expressed by `(D)`.

---

## Plug in (D) to the H statistic formula

Let's plug in `(D)` to the H statistic formula.

```
H = (N - 1) * SUM(i=1 to g) n_i * (r_i_hat - r_hat)^2
              ---------------------------------------
                  { [N + 1] * N * [N - 1] } / 12

H = 12 * SUM(i=1 to g) n_i * (r_i_hat - r_hat)^2
         ---------------------------------------
                    { [N + 1] * N }
```

Continuing the process yields the following.

```
H = [12 / (N(N+1))] * [SUM(i=1 to g) n_i * (r_i_hat - r_hat)^2]

H = [12 / (N(N+1))] * [SUM(i=1 to g) n_i * {(r_i_hat)^2 - 2*r_i_hat*r_hat + (r_hat)^2}]

H = [12 / (N(N+1))] * [SUM(i=1 to g) n_i * (r_i_hat)^2 - (n_i * 2 * r_i_hat * r_hat) + n_i * (r_hat)^2]

H = [12 / (N(N+1))] * [{SUM(i=1 to g) n_i * (r_i_hat)^2} - {SUM(i=1 to g) n_i * 2 * r_i_hat * r_hat} + {SUM(i=1 to g) n_i * (r_hat)^2}]           (E)
```

---

## Express (E) in form of N (the total number of observations across all groups)

Recall the followings before proceeding to the next step:<br/>

(X) The formula of `r_i_hat` is `(1/n_i) * (SUM(j=1 to n_i) r_i_j)`<br/>
(Y) The formula of `r_hat` is `(N + 1) / 2`<br/>
(Z) The formula from `(B)` is `N * (N + 1) / 2 = SUM(i=1 to g) SUM(j=1 to n_i) r_i_j`<br/><br/>

We'll expand the following terms so that it's in the form of `N`:<br/>

(P) `SUM(i=1 to g) n_i * 2 * r_i_hat * r_hat`<br/>
(Q) `SUM(i=1 to g) n_i * (r_hat)^2`<br/><br/>

Now let's take a look at `(P)` first.

```
SUM(i=1 to g) n_i * 2 * r_i_hat * r_hat

Using (X) and (Y) to replace r_i_hat and r_hat yields the following:

SUM(i=1 to g) n_i * 2 * (1/n_i) * (SUM(j=1 to n_i) r_i_j) * (N + 1) / 2

SUM(i=1 to g) (SUM(j=1 to n_i) r_i_j) * (N + 1)

(N + 1) * SUM(i=1 to g) (SUM(j=1 to n_i) r_i_j)

Using (Z), the above can be expressed with the following:

N * (N + 1)^2 / 2           (F)
```

Next, let's take a look at `(Q)`.

```
SUM(i=1 to g) n_i * (r_hat)^2

Using (Y) to replace r_hat yields the following:

SUM(i=1 to g) n_i * (N + 1)^2 / 4

[(N + 1)^2 / 4] * SUM(i=1 to g) n_i

Know that SUM(i=1 to g) n_i is simply N or the total number of observations (combined groups).

Therefore, we get:

N * (N + 1)^2 / 4          (G)
```

---

## Plug in (F) and (G) to (E)

Finally, plugging in `(F)` and `(G)` to `(E)`.

```
H = [12 / (N(N+1))] * [{SUM(i=1 to g) n_i * (r_i_hat)^2} - {N * (N + 1)^2 / 2} + {N * (N + 1)^2 / 4}]

H = [12 / (N(N+1))] * [{SUM(i=1 to g) n_i * (r_i_hat)^2} - {N * (N + 1)^2 / 4}]

H = [12 / (N(N+1))] * [{SUM(i=1 to g) n_i * (r_i_hat)^2}] - [12 / (N * (N + 1))] * [{N * (N + 1)^2 / 4}]

H = [[12 / (N * (N + 1))] * [{SUM(i=1 to g) n_i * (r_i_hat)^2}]] - [3 * (N + 1)]
```

Done.
