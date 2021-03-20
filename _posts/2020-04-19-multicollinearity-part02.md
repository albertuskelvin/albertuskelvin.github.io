---
title: 'Multicollinearity - Large Estimating Betas Variance (Part 2)'
date: 2020-04-19
permalink: /posts/2020/04/maths-behind-why-multicollinearity-is-problem-part02/
tags:
  - machine learning
  - multicollinearity
  - regression
  - variance
  - maths
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/04/maths-of-multicollinearity-part-1/">post</a>, I mentioned about how collinearity affects the computation of the beta estimators.

To summarize briefly, when there is a perfect linear combination between two or more covariates, the `X'X` will not be invertible (singular matrix). Consequently, we could not calculate the estimated betas.

The previous post also showed the proof of how a perfect collinearity yields a zero determinant making a matrix does not have an inverse. However, how if the collinearity is not perfect (the relationship between covariates is close to 1 or -1)?

Well, we can easily prove that in that context, the `X'X` will be invertible. However, the value of `inv(X'X)` will be large. Let's see how it affects the covariates interpretation.

```
Notes:
X' denotes the transpose matrix of X
inv(X) denotes the inverse matrix of X
```

## Deriving the Betas Variance

We're gonna derive the formula of betas variance.

Recall that `Var(X) = E(X^2) - E(X)^2`. Therefore, we simply perform the followings.

```
BETAS = inv(X' X) X' Y
Y = X TRUE_BETAS + ERROR

Var(BETAS) = Var(inv(X' X) X' Y)

Var(inv(X' X) X' Y) = E((inv(X' X) X' Y)^2) - E(inv(X' X) X' Y)^2

Let's solve E((inv(X' X) X' Y)^2) first.

K = inv(X' X) X' Y
K = inv(X' X) X' (X TRUE_BETAS + ERROR)
K = inv(X' X) X' X TRUE_BETAS + inv(X' X) X' ERROR
K = TRUE_BETAS + inv(X' X) X' ERROR

K^2 = TRUE_BETAS^2 + (inv(X' X) X' ERROR)^2 + 2 * (TRUE_BETAS inv(X' X) X' ERROR)

E(K^2) = TRUE_BETAS^2 + E((inv(X' X) X' ERROR)^2) + E(2 * (TRUE_BETAS inv(X' X) X' ERROR))

Since the errors are presumed to be normally distributed with mean equals to 0 (E(ERROR) = 0), 
then we conclude that E(2 * (TRUE_BETAS inv(X' X) X' ERROR)) = 2 * TRUE_BETAS inv(X' X) X' E(ERROR) = 0.

Hence,

E(K^2) = TRUE_BETAS^2 + (inv(X' X) X')^2 E(ERROR^2)

Since Var(ERROR) = E(ERROR^2) - E(ERROR)^2 and E(ERROR) = 0, we got E(ERROR^2) = Var(ERROR).

Finally, E(K^2) = TRUE_BETAS^2 + (inv(X' X) X')^2 Var(ERROR).

===============

Let's solve E(inv(X' X) X' Y)^2.

Since BETAS are the estimators of TRUE_BETAS, we could say that presumming E(BETAS) = TRUE_BETAS results in an unbiased estimator for each beta.

Therefore, E(inv(X' X) X' Y)^2 = TRUE_BETAS^2.

===============

As the last step,

Var(inv(X' X) X' Y) = E((inv(X' X) X' Y)^2) - E(inv(X' X) X' Y)^2
Var(inv(X' X) X' Y) = TRUE_BETAS^2 + (inv(X' X) X')^2 Var(ERROR) - TRUE_BETAS^2
Var(inv(X' X) X' Y) = (inv(X' X) X')^2 Var(ERROR)
Var(inv(X' X) X' Y) = (inv(X' X) X')^2 Var(ERROR)

Since (inv(X' X) X')^2 = inv(X' X), then

Var(inv(X' X) X' Y) = Var(ERROR) inv(X' X)
Var(BETAS) = Var(ERROR) inv(X' X)
```

Last but not least, we might also want to expand the value of `inv(X' X)`. Here's how we're gonna do it.

```
Recall that X is a matrix consisting of the covariates. Suppose that we have two covariates p and q. 
To simplify the inverse calculation, we'll ignore the intercept (beta_0).

X
[p0	q0]
[p1	q1]
[.	.]
[pn	qn]

The transpose of X would be the following.

X'
[p0	p1	p2	...	pn]
[q0	q1	q2	...	qn]

Then, the value of X'X and inv(X'X) would be:

X'X
[p0^2+p1^2+...+pn^2	p0q0+p1q1+...+pnqn]
[q0p0+q1p1+...+qnpn	q0^2+q1^2+...+qn^2]

Suppose the determinant of X'X = D which equals to 
(p0^2+p1^2+...+pn^2) * (q0^2+q1^2+...+qn^2) - (q0p0+q1p1+...+qnpn)*(p0q0+p1q1+...+pnqn).

inv(X'X)
[(q0^2+q1^2+...+qn^2) / D	-(p0q0+p1q1+...+pnqn)]
[-(q0p0+q1p1+...+qnpn)		(p0^2+p1^2+...+pn^2) / D]

Or we can rewrite it in a more compact form.

inv(X'X)
[SUM(q(i)^2) / D	-SUM(p(i)q(i))]
[-SUM(q(i)p(i))		SUM(p(i)^2) / D]
```

## Then What?

With the above matrix form of `inv(X'X)`, let's build the variance-covariance matrix.

```
Var(BETAS) = Var(ERROR) inv(X' X)

Var(BETAS)
[Var(ERROR) * SUM(q(i)^2) / D		- Var(ERROR) * SUM(p(i)q(i))]
[-Var(ERROR) * SUM(q(i)p(i))		Var(ERROR) * SUM(p(i)^2) / D]
```

Since our example has two covariates (`p` and `q`), then our multiple regression equation should be look like the following.

```
y = beta_1 * p + beta_2 * q + error
```

According to the variance-covariance matrix above, the variance for each of the estimated beta can be interpreted as the followings.

```
Var(beta_1) = Var(ERROR) * SUM(q(i)^2) / D
Var(beta_2) = Var(ERROR) * SUM(p(i)^2) / D
```

It should be obvious what would happen when there's a near-perfect correlation between covariates. The determinant `D` would be very small (close to zero) and consequently, the variances will be large.

This large variances might introduce an issue when we're trying to draw insights from the model. We can't be so sure how much a covariate affects the target variable.
