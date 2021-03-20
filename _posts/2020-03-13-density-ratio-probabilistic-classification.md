---
title: 'Density Ratio Estimation with Probabilistic Classification for Handling Covariate Shift'
date: 2020-03-13
permalink: /posts/2020/03/density-ratio-estimation-probabilistic-classification/
tags:
  - machine learning
  - covariate shift
  - density ratio
  - probabilistic classification
  - importance weighting
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/03/covariate-shift-detection-machine-learning-based-approach/">post</a> I shared about how to detect covariate shift with a simple technique--model based approach. After knowing that the data distribution changes, what can we do to address such an issue?

Various techniques are available. However, in this post I'm going to highlight a technique called importance weighting. It's basically tells the model about a way of restructuring the train distribution so that it moves towards the test distribution.

The train distribution is restructured via a factor called as the density ratio between the test and train data. Formally, it can be written as `ratio = p(x) / q(x)` or `p(x) = ratio * q(x)` for a random variable `x`. In other words, to make the train distribution becomes almost similar to the test distribution, we need a correction factor of `ratio`. 

The problem is that we don't always have direct access to the true distribution of both train and test data. What we have are the distribution of samples from the train and test population. Therefore, what we're going to do is to estimate the density ratio given that we're only able to play with the sample distributions.

Interestingly, there are lots of approaches to estimate the density ratio. Some of them estimate the ratio without computing the individual density. However, I always have a place for the classic ones. It's basically reduces the density ratio estimation problem to the probabilistic classification problem.

Suppose `p(x)` denotes the probability density function of the test distribution, while `q(x)` denotes the probability density function of the train distribution for a random variable `x`.

We collect `Np` samples from `p(x)` and label them with `test`. We also collect `Nq` samples from `q(x)` and label them with `train`. Then, we merged all the data.

Let's take a look at the following formula derivation.

<p>
<h3>
ratio = p(x) / q(x)
</h3>
</p>

<p>
<h3>
p(x) ~ P(x | y = test) = [ P(y = test | x) . P(x) ] / P(y = test)<br/>
q(x) ~ P(x | y = train) = [ P(y = train | x) . P(x) ] / P(y = train)
</h3>
</p>

<p>
<h3>
p(x) / q(x) ~ [ ( P(y = test | x) . P(x) ) / P(y = test) ] / [ ( P(y = train | x) . P(x) ) / P(y = train) ]
</h3>
</p>

<p>
<h3>
p(x) / q(x) ~ [ P(y = test | x) / P(y = train | x) ] . [ P(y = train) / P(y = test) ]
</h3>
</p>

<p>
<h3>
P(y = train) = Nq / (Np + Nq)<br/>
P(y = test) = Np / (Np + Nq)
</h3>
</p>

<p>
<h3>
p(x) / q(x) ~ [ P(y = test | x) / P(y = train | x) ] . ( Nq / Np )<br/><br/>
p(x) / q(x) ~ [ P(y = test | x) / ( 1 - P(y = test | x) ) ] . ( Nq / Np )
</h3>
</p>

Based on the last line of the above equations, the density ratio could be approximated just by computing the probability of a random variable `x` belongs to the test distribution. Computing the `P(y = test | x)` is the same as building a classifier that gives the prediction probability of a random variable `x` belongs to class test.

Generally, this estimation technique works as follows:

<ul>
<li>Add a new column called <b>origin</b> to both train and test set. For train set, the <b>origin</b> column should be <i>train</i>, while it should be <i>test</i> for test set</li>
<li>Combine the train and test set into a new dataset. The <b>origin</b> column acts as the new label</li>
<li>Build a classifier that predicts whether an instance belongs to test or train origin</li>
<li>Retrieve the probability that an instance belongs to test origin</li>
<li>Apply the resulted probability on the previous formula</li>
<li>Use the result from the formula as the weight for the instance</li>
<li>Repeat step 4 - 6 for the other instances</li>
</ul>
