---
title: 'The Monty Hall Problem Using Conditional Probability'
date: 2019-08-03
permalink: /posts/2019/08/monty-hall-problem/
tags:
  - monty hall
  - probability
  - bayes
  - conditional probability
---

The Monty Hall Problem can be stated as the following:

<pre>
1. There are 3 closed doors (A, B, and C). There's a car behind a door and goats behind the other two
2. You choose one door
3. The host opens a door with goats behind it, leaving you with the door you chose and the other one that's still closed
4. The host asks you whether you want to switch to the other closed door
</pre>

The question is, should you stay with your initial choice or switch to the other door?

In this article, we're gonna try to solve this problem using Bayes' Theorem (conditional probability). As a refresher, here's the formula:

<pre>
P(X | Y) = P(X intersects Y) / P(Y)

or

P(X | Y) = P(Y | X) * P(X) / P(Y)
</pre>

<b>Assume that we chose door A and the host shows us door B</b>. Then, let's assign X and Y to the corresponding events.

<pre>
X = probability that the car is behind door A
Y = probability that door B is shown
</pre>

Obviously, `P(X)` equals to 1/3. `P(Y | X)` would be equal to 1/2 since there are two doors left (B and C) to be chosen by the host and each has a 50% chance to be selected.

Now, how about `P(Y)`? Basically, `P(Y)` should be 1/3. However, we need to assign a weight to each case. Precisely, we should consider several cases here, such as the followings:

<ul>
<li>If the car is behind door A, then the host will show door B 50% of the time (weight 1/2)</li>
<li>If the car is behind door B, then the host will never show door B (weight 0)</li>
<li>If the car is behind door C, then the host will always show door B (weight 1)</li>
</ul>

We've got all the variables' value. Let's insert them into the formula.

<pre>
P(X | Y) = P(Y | X) * P(X) / P(Y)
P(Y | X) = 1/2
P(X) = 1/3
P(Y) = (1/3*1/2) + (1/3*0) + (1/3*1) = 1/2

Therefore, P(X | Y) = 1/3
</pre>

So, based on Bayes' Theorem, the probabiliy that the car is behind door A given that the host shows door B with goats behind it is 1/3. Since the car will either be behind door A or door C, the probability that the car is behind door C is simply (1 - 1/3) = 2/3.

Therefore, <b>the probability is higher if you switch to the other door.</b>
