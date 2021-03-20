---
title: 'Drawing ROC Curve Without Applying the Formulas'
date: 2019-12-24
permalink: /posts/2019/12/how-to-draw-roc-curve/
tags:
  - machine learning
  - roc curve
  - true positive rate
  - false positive rate
---

One of the evaluation metrics that is often optimised is ROC-AUC. In this post, we're going to discuss how an ROC curve is created.

The required information are the true label and prediction probability for the positive class. So let's take a look at the following example.

```
Label	| Probability
=======================
1	|	0.75
1	|	0.9
0	|	0.85
0	|	0.95
0	|	0.55
1	|	0.98
1	|	0.97
=======================
```

Here's the basic way on how the ROC curve is created.

<b>(A)</b> Sort the instances based on the prediction probability (descending order). The following is the new dataframe after being sorted.

```
Label	| Probability
=======================
1	|	0.98
1	|	0.97
0	|	0.95
1	|	0.9
0	|	0.85
1	|	0.75
0	|	0.55
=======================
```

<b>(B)</b> Consider the prediction probability from each row as the threshold. In other words, predict all instances whose probability >= threshold as positive class. The remaining rows are predicted as negative class.

For the sake of clarity, let's take a look at how this rule applies on the previous dataframe.

<b><I>Trial A</I></b>

```
Label	|	Probability	|	Label Hat
==================================================
1	|	0.98 (threshold)|	1
1	|	0.97		|	0
0	|	0.95		|	0
1	|	0.9		|	0
0	|	0.85		|	0
1	|	0.75		|	0
0	|	0.55		|	0
==================================================
```

<b><I>Trial B</I></b>

```
Label	|	Probability	|	Label Hat
==================================================
1	|	0.98		|	1
1	|	0.97 (threshold)|	1
0	|	0.95		|	0
1	|	0.9		|	0
0	|	0.85		|	0
1	|	0.75		|	0
0	|	0.55		|	0
==================================================
```

<b><I>Trial C</I></b>

```
Label	|	Probability	|	Label Hat
==================================================
1	|	0.98		|	1
1	|	0.97            |	1
0	|	0.95 (threshold)|	1
1	|	0.9		|	0
0	|	0.85		|	0
1	|	0.75		|	0
0	|	0.55		|	0
==================================================
```

<b><I>Trial D</I></b>

```
Label	|	Probability	|	Label Hat
==================================================
1	|	0.98		|	1
1	|	0.97            |       1
0	|	0.95		|       1
1	|	0.9 (threshold) |       1
0	|	0.85		|	0
1	|	0.75		|	0
0	|	0.55		|	0
==================================================
```

<b><I>Trial E</I></b>

```
Label	|	Probability	|	Label Hat
==================================================
1	|	0.98		|	1
1	|	0.97            |	1
0	|	0.95            |	1
1	|	0.9		|	1
0	|	0.85 (threshold)|	1
1	|	0.75		|	0
0	|	0.55		|	0
==================================================
```

<b><I>Trial F</I></b>

```
Label	|	Probability	|	Label Hat
==================================================
1	|	0.98		|	1
1	|	0.97            |	1
0	|	0.95            |	1
1	|	0.9		|	1
0	|	0.85            |	1
1	|	0.75 (threshold)|	1
0	|	0.55		|	0
==================================================
```

<b><I>Trial G</I></b>

```
Label	|	Probability	|	Label Hat
==================================================
1	|	0.98		|	1
1	|	0.97            |	1
0	|	0.95            |	1
1	|	0.9		|	1
0	|	0.85            |	1
1	|	0.75		|	1
0	|	0.55 (threshold)|	1
==================================================
```

<b>(C)</b> Draw the ROC curve.

To draw the curve, we need to know the TPR and FPR. In the ROC curve, TPR acts as the y-axis, while FPR acts as the x-axis.

For TPR, we partition the y-axis into N ranges. In this case, N denotes the number of positive class (in label). Since in this case we have 4 positive classes, the ranges would be computed as the following.

```
Scale of the ranges: 1 / 4 = 0.25

Range A: 0.0 - 0.25
Range B: 0.25 - 0.5
Range C: 0.5 - 0.75
Range D: 0.75 - 1.0
```

We perform the same approach for the x-axis (FPR). Since in this example we have 3 negative classes, we would then have 3 ranges with the following ranges calculation.

```
Scale of the ranges: 1 / 3 = 0.33

Range A: 0.0 - 0.33
Range B: 0.33 - 0.66
Range C: 0.66 - 0.99
```

The above ranges information will be combined with the previous sliding threshold results. Here's what I mean precisely.

```
For each TP, draw a vertical line (y-axis) that covers a single range.
For each FP, draw a horizontal line (x-axis) that covers a single range.
```

We, off course, start from Trial A. Let's take a look at what this statement actually means.

<b>Trial A - We got 1 TP & 0 FP - Draw a vertical line (0.0 to 0.25)</b>

<b>Trial B - We got 2 TP & 0 FP - Draw a vertical line (0.25 to 0.5)</b>

<b>Trial C - We got 2 TP & 1 FP - Draw a horizontal line (0.0 to 0.33)</b>

<b>Trial D - We got 3 TP & 1 FP - Draw a vertical line (0.5 to 0.75)</b>

<b>Trial E - We got 3 TP & 2 FP - Draw a horizontal line (0.33 to 0.66)</b>

<b>Trial F - We got 4 TP & 2 FP - Draw a vertical line (0.75 to 1.0)</b>

<b>Trial G - We got 4 TP & 3 FP - Draw a horizontal line (0.66 to 1.0)</b>

Finally, here's the final ROC curve.

```
1.00          -------
              |
0.75    ------
        |
0.5 ----
    |	
0.25 
    |
0.0   0.33  0.66  0.99
```

=== === ===

Easy, heh?

It's not surprising that the following example (different from the previous one) gives the best ROC curve.

```
Label	|	Probability
===========================
1	|	0.98
1	|	0.97
1	|	0.95
0	|	0.5
0	|	0.4
0	|	0.3
===========================
```

Applying the sliding threshold like the preceding example yields the following results.

```
Threshold 0.98 - 1 TP & 0 FP
Threshold 0.97 - 2 TP & 0 FP
Threshold 0.95 - 3 TP & 0 FP
Threshold 0.5 - 3 TP & 1 FP
Threshold 0.4 - 3 TP & 2 FP
Threshold 0.3 - 3 TP & 3 FP
```

Here's the generated ROC curve.

```
1.00 --------------------
    |
0.66	
    |	
0.33 
    |
0.0   0.33    0.66    0.99
```

=== === ===

Thank you for reading.
