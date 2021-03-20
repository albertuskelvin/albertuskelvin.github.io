---
title: 'Monotonic Binning for Weight of Evidence (WoE) Encoding'
date: 2020-04-25
permalink: /posts/2020/04/monotonic-binning-algorithm-weight-of-evidence/
tags:
  - machine learning
  - weight of evidence
  - monotonic binning
---

I was experimenting with the weight of evidence (WoE) encoding for continuous data. The preparation is quite different from categorical data in terms of binning characteristics.

Applying WoE encoding on continuous data requires the data to be binned in monotonic way (bins & WoE). Take a look at the following example.

```
Covariate: age
Bins: [-inf, 10], (10, 20], (20, 30], (30, 40], (40, inf]
Possible WoE: 0, 0.5, 0.8, 0.9, 0.95
Possible WoE: 0.95, 0.9, 0.8, 0.5, 0
```

Provided below a simple algorithm to perform monotonic binning on a certain covariate. The `qcut` from pandas is leveraged to bucketize the data since I think it's more established to do the thing compared with Spark's bucketizer.

One more thing. I've provided a simple python package for WoE. You can find the code <a href="https://github.com/albertuskelvin/weight-of-evidence-spark">here</a>.

<h2>Fit the data for monotonic binning</h2>

```python
import pandas as pd
import numpy as np
import scipy.stats.stats as stats

from pyspark.sql import SparkSession
from pyspark.sql import functions as F

from woe_iv.woe import WOE_IV


def fit_mono_bin(X, Y, n = 20):
    col_to_bucketed = X.columns[0]
    label_col = Y.columns[0]
    Ypd = Y.toPandas().values.reshape(-1)
    Xpd = X.toPandas().values.reshape(-1)
    r = 0
    while np.abs(r) < 1:
        d1 = pd.DataFrame({col_to_bucketed: Xpd, 
                           label_col: Ypd, 
                           "bucket": pd.qcut(Xpd, n, duplicates='drop')})
        d1['bucket_label'] = d1.groupby('bucket').ngroup()
        
        # compute woe for the bucket
        woe_df = compute_woe(df=spark.createDataFrame(d1.drop('bucket', 1)), 
                             cols_to_woe=["bucket_label"], 
                             label_col=label_col, 
                             good_label='yes')
        d1["bucket_label_woe"] = woe_df.select("bucket_label_woe").toPandas()
        
        r, p = stats.spearmanr(d1['bucket_label'], d1['bucket_label_woe'])
        n = n - 1
        
        if n < 1:
            print('could not perform monotonic bin for column {}'.format(col_to_bucketed))
            return None

    print('found monotonic bin for column {} with n_bin = {}'.format(col_to_bucketed, d1["bucket"].nunique()))
    return d1['bucket'].unique()

    
def compute_woe(df, cols_to_woe, label_col, good_label):
    woe = WOE_IV(df, cols_to_woe, label_col, good_label)
    woe.fit()
    return woe.transform(df)
```

The above fit method only returns the intervals of each bin. We need to fetch the minimum and maximum number for each bin. This information will be applied when the corresponding covariate is transformed. The `retrieve_bucket_min_max` method performs this step.

In addition, the `retrieve_bucket_min_max` method also adds two bins other than the fitted bins, namely `[-inf, minimum_value_of_the_first_bin]` and `(maximum_value_of_the_last_bin, inf]`. This is just simply to accomodate the possible case of differences in variance between train (where the bins are fitted), validation, and test data.

```python
def retrieve_bucket_min_max(bucket_df):
    lower_bounds = pd.IntervalIndex(bucket_df).left
    upper_bounds = pd.IntervalIndex(bucket_df).right
    
    tmp_dict = {}
    # from the fitted bins
    for bound in zip(lower_bounds, upper_bounds):
        key_name = '(' + str(bound[0]) + ', ' + str(bound[1]) + ']'
        tmp_dict.update({
            key_name: {
                'min': bound[0],
                'max': bound[1]
            }
        })

    # other than the fitted bins
    key_name_inf_to_lower_bound = '[-inf, ' + str(min(lower_bounds)) + ']'
    tmp_dict.update({
        key_name_inf_to_lower_bound: {
            'min': -float("inf"),
            'max': min(lower_bounds)
        }
    })
    
    key_name_upper_bound_to_inf = '(' + str(max(upper_bounds)) + ', inf]'
    tmp_dict.update({
        key_name_upper_bound_to_inf: {
            'min': max(upper_bounds),
            'max': float("inf")
        }
    })
    return tmp_dict
```

Last but not least, below is the starting point of monotonic bin fitting. In this case, `train_df` is a pyspark dataframe.

```python
# fit binning
bin_data = {}
for col, var_type in train_df.dtypes:
    if var_type != 'string':
        bucket_df = fit_mono_bin(train_df.select(col), train_df.select(TARGET_COL))
        if bucket_df is not None:
            bucket_min_max = retrieve_bucket_min_max(bucket_df)
```

<h2>Transform the data</h2>

Simply put, the `transform_mono_bin` method transform each continuous covariate into the corresponding bin.

In this case, `df` is a pyspark dataframe to be transformed and `bin_data` is the bin intervals information. The below method creates a new column storing the bins for each continuous variable.

```python    
def transform_mono_bin(df, bin_data):
    for col, bin_info in bin_data.items():
        df = df.withColumn(col + '_bin', 
                           F.coalesce(
                               *[F.when((F.col(col) > min_max['min']) & (F.col(col) <= min_max['max']), 
                                         F.lit(bucket)) 
                                 for bucket, min_max in bin_info.items()]
                           ))
    return df
```
