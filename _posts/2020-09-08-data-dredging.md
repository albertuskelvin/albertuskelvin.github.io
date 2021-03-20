---
title: 'Data Dredging (p-hacking)'
date: 2020-09-08
permalink: /posts/2020/09/data-dredging/
tags:
  - statistics
  - hypothesis testing
  - data dredging
  - p-hacking
---

"If you torture the data long enough, it will confess to anything" - Ronald Coase.

Data dredging / p-hacking for the sake of a published study.

Let's take a look at an example.

A group of scientists are asked for investigating whether a new developed drug improves the recovery time from a virus.

They perform a statistical hypothesis testing which starts from stating that there's no difference between the recovery time for people without & with the drug (null hypothesis). Let's say the used confidence level is 95%.

Twenty experiments are performed. Basically, here's how each experiment is performed.

(A) Several people who recovered without the drug are asked to participate in the experiment<br/>
(B) Several people who recovered without the drug are asked to participate in the experiment & are then given the drug

The mean of (A) and (B) are then compared to check whether there's a statistically significant difference between them.

Suppose that out of 20 experiments, only 1 experiment that reports a statistically significant result. This experiment is then brought to the surface and the scientists conclude that their new drug has improved the recovery time from the virus.

Performing experiments multiple times until getting the wanted result (without reporting lots of unwanted results) is called data dredging or p-hacking.

The top reason behind this act is because researches with statistically significant results are more likely to be accepted.

<b>Here's the why behind.</b>

The sampling distribution of the mean difference has zero mean and 95% of the time we'd get the mean differences that are not statistically significant.

On the other hand, there's a 5% chance that the mean difference is considered significant but not in reality (false positive / type I error).

To ensure that the difference really exists, we expect that the mean between two group of people recovered without the drug doesn't differ too much (for the sake of the same starting point).

Since the people recovered without the drug for both groups were taken from the same population, there's a high chance that their means should be close enough (fit for the starting point) every time we collect them.

If the null hypothesis is true, then it's expected that 19 out of 20 experiments the scientists performed should have non-statistically significant difference. Additionally, it's also expected that 1 out of 20 experiments is a false positive.

The above is what happens in our case example.

This false positive obviously can't be considered as a statistically significant result. Simply put, even though both groups of people without hte drug were taken from the same population (expected to have a close mean difference), there's still a little chance that both groups yield a quite big mean difference.

Therefore, even though we get one "significant" result out of 20 experiments, such a result should be considered carefully. Such a "significant" difference might not be caused by the drug, yet only due to a rare chance of selecting two groups that yield a big mean difference (making the scientists satisfied that they found the result they wanted but wrong in reality).
