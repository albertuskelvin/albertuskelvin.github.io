---
title: 'Weight of Evidence & Information Value for Attributes Relevance Analysis with PySpark'
date: 2020-04-09
permalink: /posts/2020/04/woe-iv-attributes-relevance-analysis-with-pyspark/
tags:
  - weight of evidence
  - information value
  - attribute relevance analysis
  - machine learning
  - spark
---

Woe & information value (IV) are used as a framework for attribute relevance analysis. WoE and IV can be utilised independently since each of them play different roles.

WoE describes the relationship between a predictor and a binary dependent variable. Meanwhile, IV is the measurement of that relationship’s power. Based on its role, IV can be used as a base for attributes selection.

I’m more interested in WoE this time since I’ve been experimenting with it recently. Although the variables in the WoE’s formula tell a lot about credit scoring world (it was built based on this domain back then), it’s applicable for other domains as its main objective is to describe the pattern of the dependent variable on each group (bin) in a predictor.

Let’s say we have a predictor <b>age</b> that stores information about the age of loan’s applicants. Suppose, after applying a proper monotonic binning algorithm to create the bins, we managed to calculate the WoE for each bin. By monotonic, it means that, ideally, the spearman’s correlation between the grouping and WoE values should be one (perfect monotonic relationship).

To make it clearer, let’s take a look at a simple illustration.

```
Age bin [20, 30) with WoE -0.5
Age bin [30, 40) with WoE 0.5
Age bin [40, 50) with WoE 0.8

WoE < 0 means that the bad distribution is higher than good distribution
WoE = 0 means that bad and good has the same distribution
WoE > 0 means that the good distribution is higher than bad distribution
```

From the above, we may conclude that younger people has more default risk than older ones.

Lots of WoE python implementation are available, actually. However, I was struggling enough to do the computation on a large dataset. I decided to implement the WoE and IV computation with PySpark.

You can find the repo <a href="https://github.com/albertuskelvin/weight-of-evidence-spark">here</a>.
