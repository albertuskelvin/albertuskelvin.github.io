---
title: 'XGBoost Algorithm for Classification Problem'
date: 2021-02-11
permalink: /posts/2021/02/xgboost-classification/
tags:
  - machine learning
  - extreme gradient boosting
  - xgboost
  - classification
---

Let’s use a simple example data to demonstrate how XGBoost algorithm works on a classification problem.

```
X	|	Y
======================
1	|	false
2	|	true
3	|	false
4	|	true
```

From the above dataset, we have one independent variable (`X`) and one dependent variable (`Y`). The task is to classify whether an input `X` belongs to class `false` or `true`.

In addition, XGBoost has several hyperparameters to tune. For the sake of this demonstration, we’re going to set these hyperparameters:
- `n_estimators`: 2
- `max_depth`: 2
- `learning_rate`: 1
- `gamma`: 0.5
- `min_child_weight`: 0
- `reg_lambda`: 0

---

## Step 1. Set an initial prediction

Let’s set the initial prediction (`F0(x)`) to be 0.5. In other words, regardless of the value of `X`, the predicted `Y` will be 0.5.

With the above dataset, we can see that the probability of being `true` for `X` equals to 2 and 4 is one. Meanwhile, the probability of being `true` for `X` equals to 1 and 3 is zero.

---

## Step 2. Calculate the residuals

In this step we calculate the difference between the actual value (`Y`) and the predicted value (`F0(x)`).

Note that we may represent `Y` in terms of probability of being classified as `true`.

Here’s the result.

```
X	|	Y	|	F0(x)		|	res_0
==============================================================
1	|	0	|	0.5		|	-0.5
2	|	1	|	0.5		|	0.5
3	|	0	|	0.5		|	-0.5
4	|	1	|	0.5		|	0.5
```

---

## Step 3. Train model on the residuals

Here, we build one or more tree-based models that are trained on [`X`, `res`].

The following shows how a tree model is constructed.

### A) Generate split candidates

In this step we find one or more split candidates. The formula is with `N` rows we would have `N - 1` split candidates.

With our dataset example, here’s the result.

```
===== Candidate Split 1 =====

X	|	Y	|	F0(x)		|	res_0
==============================================================
1	|	0	|	0.5		|	-0.5
————————————————————————————————————————————————————————————————————————> threshold = (1 + 2) / 2 = 1.5
2	|	1	|	0.5		|	0.5
3	|	0	|	0.5		|	-0.5
4	|	1	|	0.5		|	0.5

===== Candidate Split 2 =====

X	|	Y	|	F0(x)		|	res_0
==============================================================
1	|	0	|	0.5		|	-0.5
2	|	1	|	0.5		|	0.5
————————————————————————————————————————————————————————————————————————> threshold = (2 + 3) / 2 = 2.5
3	|	0	|	0.5		|	-0.5
4	|	1	|	0.5		|	0.5

===== Candidate Split 3 =====

X	|	Y	|	F0(x)		|	res_0
==============================================================
1	|	0	|	0.5		|	-0.5
2	|	1	|	0.5		|	0.5
3	|	0	|	0.5		|	-0.5
————————————————————————————————————————————————————————————————————————> threshold = (3 + 4) / 2 = 3.5
4	|	1	|	0.5		|	0.5
```

The `threshold` will be used to create two child nodes. For instance, `Candidate Split 1` with threshold 1.5 will have a root node of `X < 1.5`.

Here’s the complete parent-child nodes for each candidate.

```
===== Candidate Split 1 =====

Parent node -> X < 1.5
Left child node -> [-0.5]
Right child node -> [0.5, -0.5, -0.5]

===== Candidate Split 2 =====

Parent node -> X < 2.5
Left child node -> [-0.5, 0.5]
Right child node -> [-0.5, 0.5]

===== Candidate Split 3 =====

Parent node -> X < 3.5
Left child node -> [-0.5, 0.5, -0.5]
Right child node -> [0.5]
```

### B) Select the best split

From step <b>(A)</b> we generate one or more split candidates. The question is how to select the best one?

In XGBoost, we select the best split by leveraging what are called as Similarity Score and Gain. Gain is calculated for the parent node, while Similarity Score is calculated for the child nodes. We select a candidate with the maximum Gain.

Here are the formulas.

```
Gain = Left_similarity_score + Right_similarity_score - Root_similarity_score

Similarity score = SUM(residual_i)^2 / {SUM(prev_proba_i x (1 - prev_proba_i)) + lambda}
```

For this demonstration, let’s set `lambda` to zero.

With the above formulas, let’s apply them on our split candidates. Note that the `Root_similarity_score` is zero for the root node.

```
===== Candidate Split 1 =====

Parent node -> X < 1.5
Left child node -> [-0.5]
Right child node -> [0.5, -0.5, -0.5]

Left_similarity_score = (-0.5)^2 / [(0.5 * (1 - 0.5)) + 0] = 1

Right_similarity_score = (0.5 - 0.5 - 0.5)^2 / [(0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + 0] = 0.33

Gain = 1 + 0.33 - 0 = 1.33

===== Candidate Split 2 =====

Parent node -> X < 2.5
Left child node -> [-0.5, 0.5]
Right child node -> [-0.5, 0.5]

Left_similarity_score = (-0.5 + 0.5)^2 / [(0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + 0] = 0

Right_similarity_score = (-0.5 + 0.5)^2 / [(0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + 0] = 0

Gain = 0 + 0 - 0 = 0

===== Candidate Split 3 =====

Parent node -> X < 3.5
Left child node -> [-0.5, 0.5, -0.5]
Right child node -> [0.5]

Left_similarity_score = (-0.5 + 0.5 - 0.5)^2 / [(0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + 0] = 0.33

Right_similarity_score = (0.5)^2 / [(0.5 * (1 - 0.5)) + 0] = 1

Gain = 0.33 + 1 - 0 = 1.33
```

According to the above calculation, the maximum Gain is 1.33 and being acquired by `Candidate Split 1` and `Candidate Split 3`. Let’s choose `Candidate Split 3`.

### C) Continue searching the best split until max_depth is reached

Since `max_depth` is set to 2, this basically means that our tree model should have maximum two levels. Therefore, let’s perform the same task like <b>(B)</b>.

We’ll start from `Candidate Split 3` since we chose it in the previous step.

Here’s the `Candidate Split 3`. Recall that with `N` rows we’d have `N - 1` split candidates. Since the right child node consists of only 1 row, there’ll be no split candidates. On the other hand, we’ll get two candidates from the left child node.

```
===== Candidate Split 3 =====

X	|	Y	|	F0(x)		|	res_0
==============================================================
1	|	0	|	0.5		|	-0.5
2	|	1	|	0.5		|	0.5
3	|	0	|	0.5		|	-0.5
————————————————————————————————————————————————————————————————————————> threshold = (3 + 4) / 2 = 3.5
4	|	1	|	0.5		|	0.5

Parent node -> X < 3.5
Left child node -> [-0.5, 0.5, -0.5]
Right child node -> [0.5]

Left_similarity_score = (-0.5 + 0.5 - 0.5)^2 / [(0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + 0] = 0.33

Right_similarity_score = (0.5)^2 / [(0.5 * (1 - 0.5)) + 0] = 1

Gain = 0.33 + 1 - 0 = 1.33
```

So, let’s grow the tree from the left child node! We start by generating the split candidates.

```
===== Candidate Split 3_1 =====

X	|	Y	|	F0(x)		|	res_0
==============================================================
1	|	0	|	0.5		|	-0.5
————————————————————————————————————————————————————————————————————————> threshold = (1 + 2) / 2 = 1.5
2	|	1	|	0.5		|	0.5
3	|	0	|	0.5		|	-0.5

===== Candidate Split 3_2 =====

X	|	Y	|	F0(x)		|	res_0
==============================================================
1	|	0	|	0.5		|	-0.5
2	|	1	|	0.5		|	0.5
————————————————————————————————————————————————————————————————————————> threshold = (2 + 3) / 2 = 2.5
3	|	0	|	0.5		|	-0.5
```

Next, similar to the previous step, we select the best split candidate by calculating the Gain and Similarity Score.

Note that when calculating Gain, the `Root_similarity_score` is simply the same as `Left_similarity_score` of `Candidate Split 3` which is 0.33.

```
===== Candidate Split 3_1 =====

Parent node -> X < 1.5
Left child node -> [-0.5]
Right child node -> [0.5, -0.5]

Left_similarity_score = (-0.5)^2 / [(0.5 * (1 - 0.5)) + 0] = 1

Right_similarity_score = (0.5 - 0.5)^2 / [(0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + 0] = 0

Gain = 1 + 0 - 0.33 = 0.67

===== Candidate Split 3_2 =====

Parent node -> X < 2.5
Left child node -> [-0.5, 0.5]
Right child node -> [-0.5]

Left_similarity_score = (-0.5 + 0.5)^2 / [(0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + 0] = 0

Right_similarity_score = (-0.5)^2 / [(0.5 * (1 - 0.5)) + 0] = 1

Gain = 0 + 1 - 0.33 = 0.67
```

We got the same number. Well, let’s just select `Candidate Split 3_1`.

Since we have reached the `max_depth`, we’ll stop building the tree.

Our final tree is shown by the following.

```
IF X < 3.5 THEN
	IF X < 1.5 THEN
		[-0.5]
	ELSE
		[0.5, -0.5]
ELSE
	[0.5]
```

### D) Tree pruning

In this step, we reduce the size of decision tree by removing sections that has little power to classify data points.

Below is the rule of pruning.

```
Start from low level parent node

IF (Gain - gamma) < 0 THEN
	Remove that parent node & Check the same for upper parent node
ELSE
	Don’t remove the parent node & Stop pruning
```

Using our decision tree, here’s how the pruning is performed.

```
The lowest parent node has Gain of 0.67 with Similarity Score of 0.33.
Calculating Gain - gamma gives 0.67 - 0.5 = 0.17.
Since 0.17 > 0, we don’t remove the node and stop pruning.
```

### E) Calculate Output Values

Output Values is calculated for all leaves of the decision tree.

The formula is given by the following:

```
Output Value = SUM(residual_i) / [SUM(prev_proba_i x (1 - prev_proba_i)) + lambda]
```

Recall that our previous tree has leaves with value of `[0.5, -0.5]`, `[0.5]` and `[-0.5]`.

```
Output value for [0.5, -0.5] => (0.5 - 0.5) / [(0.5 * (1 - 0.5)) + (0.5 * (1 - 0.5)) + 0] = 0

Output value for [-0.5] => (-0.5) / [(0.5 * (1 - 0.5)) + 0] = -2

Output value for [0.5] => (0.5) / [(0.5 * (1 - 0.5)) + 0] = 2
```

This step simply replaces the leaf values (which may have multiple values of residuals) with the Output Values.

To conclude, here’s our first decision tree (<b>DT_0</b>) that is constructed by fitting it on [`X`, `res_0`].

```
IF X < 3.5 THEN
	IF X < 1.5 THEN
		-2
	ELSE
		0
ELSE
	2
```

### F) Predict the new values using DT_0

Before predicting with <b>DT_0</b>, we need to convert the initial probability (which is 0.5) to the log(odds). This can be achieved by taking the log of `(p / (1 - p))` where `p` is the initial prediction. Therefore, the `log(odds)` of the initial prediction is `log(1)` or zero.

To predict the new values, we use the following formula.

```
log(odds)_F1(x) = log(odds)_F0(x) + learning_rate * (DT_0(x))

F1(x) is retrieved by computing e^log(odds)_F1(x) / [1 + e^log(odds)_F1(x)]
```

Let’s apply this step to our dataset. Assume that `learning_rate` is set to 1.

```
X	|	Y	|	F0(x)		|	res_0		|	F1(x)		|    res_1
=============================================================================================================
1	|	0	|	0.5		|	-0.5		|  e^-2 / [1 + e^-2]	|  0 - F1(x)
2	|	1	|	0.5		|	0.5		|  e^0 / [1 + e^0]	|  1 - F1(x)
3	|	0	|	0.5		|	-0.5		|  e^0 / [1 + e^0]	|  0 - F1(x)
4	|	1	|	0.5		|	0.5		|  e^2 / [1 + e^2]	|  1 - F1(x)
```

---

## Step 4. Repeat Step 3 until n_estimators is reached

After we get `res_1`, we can build another decision tree that is fitted on [`X`, `res_1`] until `n_estimators` (number of decision trees fitted on residuals) is reached.

To conclude, the final XGBoost model is given by the following.

```
log(odds)_Fn(x) = log(odds)_F0(x) + [learning_rate * (DT_0(x))] + [learning_rate * (DT_1(x))] + [learning_rate * (DT_2(x))] + . . .

Fn(x) = e^log(odds)_Fn(x) / [1 + e^log(odds)_Fn(x)]
```
