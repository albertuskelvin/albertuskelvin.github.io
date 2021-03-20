---
title: 'Riemann Hypothesis and One Question in My Mind'
date: 2019-07-10
permalink: /posts/2019/07/riemann-hypothesis/
tags:
  - maths
  - riemann hypothesis
  - zeta function
  - prime numbers counting
---

Yesterday I came across an interesting Math paper discussing about the Riemann hypothesis. Regarding the concept itself, there's lots of maths but I think I enjoyed the reading. Frankly speaking, although mathematics is one of my favourite subjects, I’ve been rarely playing with it (esp. pure maths) since I got acquainted with AI and big data engineering world. Now I think it’s just fine to play with it again. Just for fun.

The motivation behind the reading was because I was investigating the Ramanujan’s infinite sum series. Simply put, the infinite series shows us that the infinite sum of all positive integers converges to <b>-1/12</b>. To put it more mathematically, the value of <b>1 + 2 + 3 + 4 + … = -1/12</b>. Just wondering though, back then, how could the sum of series eminently diverging to infinity has a negative fractional value?

Well, it turned out that the infinite sum series appeared from what’s called as the Euler zeta function. Since mathematicians love to map a concept to a name (CS people also do this, pretty sure), I would call this Euler zeta function as <b>Z(x)</b>. Now, what is <b>Z(x)</b>? Mathematically, <b>Z(x) = 1 + 1/(2^x) + 1/(3^x) + 1/(4^x) + …</b>. If you observe the equation, you can tell yourself that for all real numbers greater than 1, the <b>Z(x)</b> converges. Otherwise, it simply diverges to infinity.

Finally Riemann zeta function (call it <b>R(x)</b>) came up to the surface. This <b>R(x)</b> applies a concept known as analytic continuation. By applying this concept, we simply extend the definition of the original function.

The Euler zeta function is defined (the resulting value converges) for real numbers greater than 1. Since real numbers are also complex numbers (only keeps the real part and omit the imaginary part), we can regard it as a complex function and then apply analytic continuation to get a new function. This new function is known as the Riemann zeta function which is defined for the whole complex plan, yet yields the same result with the Euler zeta function when the input number is greater than 1.

Now let’s go to the main topic.

Frankly, I was just curious about this Riemann zeta function. That’s why I did a quick investigation on this topic. Managed to find a maths topic on the Riemann hypothesis. I found that the hypothesis related closely to the previous Riemann zeta function. As we know, the Riemann zeta function was created to fix the limitation of the Euler zeta function (which only deals with real numbers greater than 1). However, from the paper I read, the Riemann zeta function (<b>ζ(s)</b>) is really hideous. In case you’re curious about the function, just browse the internet with this keyword: “A Friendly Introduction to The Riemann Hypothesis by Thomas Wright”.

A quick note we can get from the function is that for any <b>s</b> that doesn't make the Euler zeta function <b>Z(s)</b> blow up, <b>ζ(s) = Z(s)</b>. This <b>ζ(s)</b> also safe to use as it doesn’t diverge when <b>s</b> is less or equal to 1. Moreover, it also handles complex numbers making it a complex function. However, there’s still a limitation, namely the <b>ζ(1)</b> is undefined. 

Interesting things happen when we ask, when does <b>ζ(s)</b> result in zero? In other words, could you list all values of <b>s</b> making <b>ζ(s)</b> equals to zero?

The answer to such a question is not easy, anyway. Turns out that <b>ζ(s)</b> has two kinds of zero condition. The first one is trivial zero condition, while the other one is non-trivial zero condition. The former states that <b>ζ(s) = 0</b> happens for all negative even numbers, such as -2, -4, -6, -8, and so forth. Meanwhile, the latter states that <b>ζ(s) = 0</b> happens when <b>s</b> is considered as a complex number. Several examples, such as <b>ζ(1/2 + 14.134725142i) = 0, ζ(1/2 + 21.022039639i) = 0, ζ(1/2 + 25.010857580i) = 0</b>, and <b>ζ(1/2 + 30.424876126i) = 0</b>.

Now the question is, is the format of the complex number always includes 1/2 as the real part? In other words, is s always in the form of <b>1/2 + yi</b>, where <b>y</b> equals to real numbers? Riemann himself said that the answer to this question might be yes. Therefore, his hypothesis can be summarised as "If <b>ζ(s) = 0</b> and <b>s</b> is not a negative even integer than <b>s = 1/2 + it</b> for some real number <b>t</b>".

Up till now it’s still an unsolved problem in mathematics. In addition, this Riemann hypothesis is one of the most important as well as difficult concepts in mathematics.

In addition, there’s another fact that this Riemann hypothesis is applied to prime numbers counting. For those who don’t know about prime numbers counting theory, the task is to count how many prime numbers less than a certain value. Mathematically, <b>P(x)</b> states the number of prime numbers less than <b>x</b>.

Finding such a function is not an easy task. Fortunately, a guy named Carl Friedrich Gauss has found a reasonable estimation for <b>P(x)</b>. This estimation is known as <b>Li(x)</b> or logarithmic integral function. In case you’re curious, <b>Li(x)</b> is defined as <b>integral[2 to x] dt / ln(t)</b>. This resulted in the Prime Number Theorem stating that the number of primes less than <b>x</b> is approximately <b>Li(x)</b>.

However, with estimation comes what’s called as error / residual value. This error is sometimes called as the error term in the Prime Number Theorem. Based on the paper mentioned previously, <b>Li(x)</b> and <b>P(x)</b> can differ by as much as <b>sqrt(x)·ln(x)</b> because they’ve seen it in actual data. The hypothesis that the maximum difference is <b>sqrt(x)·ln(x)</b> because it’s pretty small relative to the number things we’re counting. In other words, if this hypothesis is true then <b>Li(x)</b> is a quite good approximation of <b>P(x)</b>.

So, the maximum error between <b>Li(x)</b> and <b>P(x)</b> is expected to be <b>sqrt(x) . ln(x)</b>. In order for the estimation hypothesis to be true, the Riemann hypothesis must be true as well. Here’s the complete statement based on the paper: “If the Riemann Hypothesis is true then <b>Li(x)</b> and <b>P(x)</b> never differ by more than about <b>sqrt(x) · ln(x)</b>”.

Using the above estimation hypothesis, we can re-state the hypothesis with the following: “If <b>Li(x)</b> and <b>P(x)</b> never differ by more than about <b>sqrt(x) · ln(x)</b> then the Riemann Hypothesis is true”.

In other words, if we could prove one of them, the other is true as well.

A question came up in my mind was how in earth the Riemann hypothesis might relate to the error term in prime number counting? Proving that the non-trivial zero in zeta function to be <b>1/2 + ti</b> (for some real numbers <b>t</b>) might prove that the error difference between the estimated and the actual number of primes is less or equal to about <b>sqrt(x) . ln(x)</b>?

I’ve searched for the answer to the above question. Got lots of maths showing such relations. I think I’ll leave that for another day.
