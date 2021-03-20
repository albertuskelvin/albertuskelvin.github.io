---
title: 'Multicollinearity - A Bit of Maths Behind Why It is a Problem (Part 1)'
date: 2020-04-15
permalink: /posts/2020/04/maths-of-multicollinearity-part-1/
tags:
  - multicollinearity
  - machine learning
  - regression
  - maths
---

In simple terms, we could define collinearity as a condition where two variables are highly correlated (positively / negatively). When there are more than two variables, it’s sometimes referred as multicollinearity.

In regression settings, this multicollinearity is not necessarily an issue when we only care about the model’s prediction. However, it could introduce a critical issue when we’re trying to draw insights (based on the estimated coefficients) from the model.

The next question would be in what context this multicollinearity could introduce a problem? I’m going to discuss this in its maths point of view.

```
Notes

X' denotes the transpose of matrix X
inv(X) denotes the inverse of matrix X
```

<h2>Derivation of the Betas Estimating Equation</h2>

Recall that we can leverage the following estimating equation to compute the coefficients (weights). Let’s refer the coefficient as `beta`.

`beta = inv(X' X) X' Y`

`X: matrix of variables`<br/>
`Y: matrix of actual response`

I’m not going to write down how to derive the above formula in a detail manner. However, let’s take a look at the basic idea of it.

Suppose that we have a train data consisting of two regressors (`P` and `Q`) and a target value (`y`). Our goal is to fit the model that yields the best estimate of the beta coefficients (`beta_0, beta_1, beta_2`).

`y = beta_0 + beta_1 * P + beta_2 * Q + error`

Let’s create the matrix form for each element of the above multiple regression formula.

```
Y
[y0]
[y1]
[..]
[yn]

BETA
[beta_0]
[beta_1]
[beta_2]

X
[1	P0	Q0]
[1	P1	Q1]
[.	.	 .]
[1	Pn	Qn]
```

The first column of the variables vector `X` is consists of all one because we got the intercept (`beta_0`) here. If we multiple `X` and `BETA`, it should yield the following.

```
X . BETA
[beta_0 + P0 * beta_1 + Q0 * beta_2]
[beta_0 + P1 * beta_1 + Q1 * beta_2]
[. . . . . . . . . . .  . . . . . .]
[beta_0 + Pn * beta_1 + Qn * beta_2]
```

We’re going to leverage the ordinary least square to search for the optimal beta coefficient. Simply put, we’re going to minimise the sum of squared error (actual response - predicted response) across the whole data points.

Since `e = Y - (X BETA)` and `MSE = 1/n * SUM[(e)^2]`, the matrix form of the MSE would be look like the following.

`MSE = 1/n * ( e' e ) where e is a matrix`

Using the above MSE matrix form, we may expand it by replacing <b>e</b> with `Y - (X BETA)`. You can try to do it by yourself.

We should come to the following result afterwards.

`MSE = 1/n * (Y' Y - 2 BETA' X' Y + BETA' X' (X BETA))`

The final task would be to minimise the above MSE. We could accomplish this by finding the estimated beta coefficients that minimise the above MSE.

It’s pretty simple, however. We just need to find the zero gradient of the MSE with respect to `BETA` (first derivation of the MSE with respect to `BETA`).

If you calculate it carefully, the derivative of the MSE should be `2/n * (X' (X BETA) - X' Y)`.

Setting the gradient to zero yields `BETA = inv(X' X) X' Y`.

The above formula <b>estimates</b> the beta coefficients.

<h2>What’s the Problem?</h2>

Based on `BETA = inv(X' X) X' Y`, we know that it would be a problem if the square matrix `(X' X)` is non-invertible (singular matrix) which means that it doesn’t have the inverse matrix.

How do we know whether a square matrix is invertible? There are several characteristics that we can use to state whether a square matrix is invertible.

<ul>
<li>The determinant is 0</li>
<li>The square matrix is rank deficient. It means that one or more of its rows (or columns) are perfectly correlated</li>
</ul>

The second characteristic is what we call as <b>collinearity</b> and it can be proved easily. Turns out that when a square matrix is rank deficient, then its determinant would be zero. This obviously refers to the first characteristic.

Let’s take a look at the simple proof here using our `(X' X)`.

```
Suppose that we have a square matrix X which stores the variables. 
Here, our multiple regression consists of two variables (p and q) and is fitted on two instances. 
To simplify that we only want to compute the determinant of a two-dimensional matrix, we omit the intercept (beta_0).
Therefore, our matrix X does not have a column with all ones.

X
[p0	q0]
[p1	q1]

X'
[p0	p1]
[q0	q1]

X'X
[p0^2 + p1^2		p0q0 + p1q1]
[q0p0 + q1p1		q0^2 + q1^2]

Assume that p is perfectly correlated with q with the equation of p = 2q.
In this case, the matrix X is column-rank deficient. 
Well, basically, it’s also row-rank deficient in nature. 

Let’s compute the determinant of X.

D = (p0^2 + p1^2)(q0^2 + q1^2) - (q0p0 + q1p1)(p0q0 + p1q1)
D = (4q0^2 + 4q1^2)(q0^2 + q1^2) - (2q0^2 + 2q1^2)(2q0^2 + 2q1^2)
D = 4q0^4 + 4q0^2q1^2 + 4q1^2q0^2 + 4q1^4 - 4q0^4 - 4q0^2q1^2 - 4q1^2q0^2 - 4q1^4

D = 0

The determinant is 0. This concludes that the square matrix X is non-invertible.
In addition, we can say that collinearity plays a role in square matrix inversion.
```

Well, some people might argue that we don’t really need to compute the inverse of `(X' X)` in order to be able to retrieve the estimated beta coefficients. Computing the inverse of a matrix is costly.

If we look at the estimating equation again (`BETA = inv(X' X) X' Y`), seems that we could just multiple the `(X' X)` with `BETA` and we’ll get a new matrix with beta coefficients as the elements. On the RHS, we’ll simply get a matrix consisting of constants. At the end of the day, what we need to do is to solve a linear equation with beta coefficients as the variables.

Well, let’s take a look in a more formal way.

```
Suppose that we don’t want to compute the inverse matrix.

X'X  B = X' Y

[p0^2 + p1^2		p0q0 + p1q1]	[b0] 	=   [p0y0 + p1y1]
[q0p0 + q1p1		q0^2 + q1^2]	[b1]        [q0y0 + q1y1]

(p0^2 + p1^2)b0 + (p0q0 + p1q1)b1 = p0y0 + p1y1
(q0p0 + q1p1)b0 + (q0^2 + q1^2)b1 = q0y0 + q1y1

Assume that p is perfectly correlated with q with the equation of p = 2q.

(4q0^2 + 4q1^2)b0 + (2q0^2 + 2q1^2)b1 = 2q0y0 + 2q1y1
(2q0^2 + 2q1^2)b0 + (q0^2 + q1^2)b1 = q0y0 + q1y1

The above equations can not be solved.
```

Turns out that we can’t find the beta coefficients with the above approach when there is collinearity.

The next question. What if the variables are not perfectly correlated but the linear relationship is very close to one (or minus one)? With this context, the `(X' X)` will be invertible and we could compute the estimated betas. What could be the problem then? This question will be answered in the next part.
