---
title: 'Finding the Best Threshold that Maximizes Accuracy from ROC & PR Curve'
date: 2019-12-23
permalink: /posts/2019/12/best-threshold-maximize-accuracy-from-roc-pr-curve/
tags:
  - machine learning
  - threshold
  - precision recall curve
  - roc curve
  - accuracy
---

The problem is simple. How to find the best threshold from an ROC and PR curve that maximise a certain binary classification metric?

To make it clearer, let's take the approach that is commonly used in scikit-learn. Scikit-learn provides a module called `metrics` that consists of some evaluation metrics, such as `accuracy_score`, `roc_curve`, `precision_recall_curve`, and so forth.

To get the ROC and PR curve, we can apply `roc_curve` and `precision_recall_curve` respectively. Both methods require two main parameters, such as the true label and the prediction probability. Take a look at the following code snippet.

```python
from sklearn.metrics import roc_curve, precision_recall_curve

fpr, tpr, thresholds = roc_curve(true_label, pred_proba)
precision, recall, thresholds = precision_recall_curve(true_label, pred_proba)
```

As you can see, both methods return the parameters needed for the curve creation. Based on the return values, how would one determine the best threshold that maximize a certain evaluation metric? In this post, I'm going to use <b>accuracy</b> as the metric. I think the approach could be adjusted in accordance with the formula of other metrics.

The solution should be simple.

The following shows one of the threshold finding approaches for ROC curve.

```python
def get_metric_and_best_threshold_from_roc_curve(
    tpr, fpr, thresholds, num_pos_class, num_neg_class
):
    tp = tpr * num_pos_class
    tn = (1 - fpr) * num_neg_class
    acc = (tp + tn) / (num_pos_class + num_neg_class)

    best_threshold = thresholds[np.argmax(acc)]
    return np.amax(acc), best_threshold
```

Below are the steps used to achieve the above formula. Starts from the below basic formulas.

```
TPR = recall = TP / TP + FN
FPR = FP / FP + TN
Accuracy = TP + TN / TP + TN + FP + FN
```

The denominator of the accuracy should be the total number of data. So it's simply `num_pos_class + num_neg_class`.

Now, we only need to find the value of `TP` and `TN`.

From the `TPR` formula, we can easily find `TP = TPR * (TP + FN)`. Since `TP + FN` equals to the number of positive class, then the formula becomes `TP = TPR * num_pos_class`.

Finding the value of `TN` is more challenging. We might apply the following approach:

```
TN = (FP - (FPR * FP)) / FPR
```

However, the above approach requires us to know the value of `FP` which is pretty troublesome to calculate.

Alternatively, we could use its neighbour called `TNR` which equals to `TNR = TN / (FP + TN)`. In such a case, we can calculate `TN` by `TNR * (FP + TN)` or `TNR * num_neg_class`. To make the formula complete, just replace `TNR` with `1 - FPR`.

Knowing all the required parameters, we can now compute the accuracy by `accuracy = (TP + TN) / (num_pos_class + num_neg_class)`.

One thing to note here is that all the variables are in the form of numpy array. Please check the <a href="https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_curve.html">documentation</a> for more details.

To find the best threshold that maximises accuracy, we just need to find the index of the maximum accuracy, then use that index to locate the corresponding threshold.

One curve down. Next, the precision-recall curve.

PR curve consists of precision and recall as its parameters. Below is the code I used to search for the best threshold.

```python
def get_metric_and_best_threshold_from_pr_curve(
    precision, recall, thresholds, num_pos_class, num_neg_class
):
    tp = recall * num_pos_class
    fp = (tp / precision) - tp
    tn = num_neg_class - fp
    acc = (tp + tn) / (num_pos_class + num_neg_class)

    best_threshold = thresholds[np.argmax(acc)]
    return np.amax(acc), best_threshold
```

Only needs a few step to reach the final formula. Here's how it’s built.

Using the following basic formulas,

```
Precision = TP / TP + FP
Recall = TP / TP + FN
Accuracy = TP + TN / TP + TN + FP + TN
```

Finding `TP` is easy. We can just use the recall's formula, which results in `TP = recall * (TP + FN)` or `TP = recall * num_pos_class`.

To find `TN`, we need to apply a bit tricks since the above precision and recall formulas don't include `TN`.

Recall that `TN` basically equals to `num_neg_class - FP`. In this case, we just need to find `FP`. It should become easier since `FP` can be calculated by leveraging the precision formula. So it becomes `FP = (TP / precision) - TP`.

We can then use the calculated variables to find the accuracy. The best threshold is generated using the previous approach.
