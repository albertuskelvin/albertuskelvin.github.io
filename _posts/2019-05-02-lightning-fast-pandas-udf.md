---
title: 'Lightning Fast Pandas UDF'
date: 2019-05-02
permalink: /posts/2019/05/lightning-fast-pandas-udf/
tags:
  - spark
  - pandas udf
  - distributed computing
---

Spark functions (UDFs) are simply functions created to overcome speed performance problem when you want to process a dataframe. It'd be useful when your Python functions were so slow in processing a dataframe in large scale. When you use a Python function, it will process the dataframe with one-row-at-a-time manner, meaning that the process would be executed sequentially. Meanwhile, if you use a Spark UDF, Spark will distribute the dataframe and the Spark UDF to the provided executors. Hence, the dataframe processing would be executed in parallel. For more information about Spark UDF, please take a look at this <a href="https://changhsinlee.com/pyspark-udf/">post</a>.

Distributing Spark UDFs to several executors doesn't always mean that the processing time will be faster. One thing to note here is that Spark UDFs apply a <b>one-row-at-a-time</b> approach. This obviously means that the UDFs will be applied to a single row at one time and the next row will be processed afterwards. Consequently, the processing time would take a long time to complete.

Fortunately, PySpark has already included <b>Pandas UDFs</b>. Its concept is quite similar to regular Spark UDF. The difference lies on how the computation is done. If Spark UDFs process one row at a time, then Pandas UDFs process multiple rows at a time. Therefore, Pandas UDFs are definitely much faster than Spark UDFs.

Now let's dive into the code.

Suppose you have ensembled three classifiers and you're going to do a prediction with this ensembled classifier.

<pre>
# suppose that we've defined our own random forest class
class MyRandomForest(object):
    def __init__(self, feature_columns):
        self._model = None
        self._feature_columns = feature_columns

    def predict(self, df):
        # spark udf
        def _rf_predict(*cols):
            return float(self._model.predict_proba((cols,))[:, 1])

        # pandas udf
        def _rf_predict_pandas(*cols):
            X = pd.concat(cols, axis=1)
            return pd.Series(self._model.predict_proba(X.values)[:, 1])

        predict_udf = F.pandas_udf(_rf_predict_pandas, DoubleType())
        return predict_udf(*self._feature_columns)


# ensembled model's prediction
# feature_columns is a list consisting of features that are used to train the classifier
rf1 = MyRandomForest(feature_columns)
rf2 = MyRandomForest(feature_columns)
rf3 = MyRandomForest(feature_columns)

classifiers = [rf1, rf2, rf3]
for index, model in enumerate(classifiers):
    col_name = 'POSITIVE_PROB_' + str(index)
    x = model.predict(df)
    df = df.withColumn(col_name, x)
</pre>

As we can see, our <i>predict</i> method consists of <i>spark udf</i> and <i>pandas udf</i>. In <i>spark udf</i>, the input parameter is a one-dimensional array consisting of the value of each column, while the output is a float number. Such an input-output format applies as <b>Spark UDFs</b> processes one row at a time, gives the output for the corresponding row, and then combines all prediction results. Meanwhile, <i>pandas udf</i> receives a <b>pandas series</b> as both the input and output. Since the input is just a series of feature columns, then we need to merge those columns so that we got instances to predict.

Based on this little experiment, I found that the speed performance of <b>Pandas UDFs</b> was much faster than <b>Spark UDFs</b>. To get a clear overview of the processing time, you can simply add these lines after the <i>for-loop</i>.

<pre>
start = time.time()
df.collect()
print('Processing time: ' + str(time.time() - start))
</pre>

Just FYI, I tested the code using 6800 data points and by using <b>Pandas UDF</b>, the process needed approximately 1.64s (in average) to complete. Meanwhile, <b>Spark UDF</b> needed approximately 31.13s (in average) to complete.

Since I was curious, I increased the number of instances to 680000. Using <b>Pandas UDF</b>, the process took approximately 7.81s (in average) to complete. Meanwhile, <b>Spark UDF</b> took a very long time to complete and I decided to abort the process.

Finally, here's the execution plan (optimized and physical) for both scenario.

<pre>
<b>Spark UDF</b>

== Optimized Logical Plan ==
Project [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6, (pythonUDF0#142) AS POSITIVE_PROB_0#101, (pythonUDF1#143) AS POSITIVE_PROB_1#114, (pythonUDF2#144) AS POSITIVE_PROB_2#128]
+- !BatchEvalPython [_rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L), _rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L), _rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L)], [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6, pythonUDF0#142, pythonUDF1#143, pythonUDF2#144]
   +- LogicalRDD [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6], false

== Physical Plan ==
*(1) Project [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6, (pythonUDF0#142) AS POSITIVE_PROB_0#101, (pythonUDF1#143) AS POSITIVE_PROB_1#114, (pythonUDF2#144) AS POSITIVE_PROB_2#128]
+- !BatchEvalPython [_rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L), _rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L), _rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L)], [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6, pythonUDF0#142, pythonUDF1#143, pythonUDF2#144]
   +- Scan ExistingRDD[F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6]

<b>Pandas UDF</b>

== Optimized Logical Plan ==
Project [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6, (pythonUDF0#142) AS POSITIVE_PROB_0#101, (pythonUDF1#143) AS POSITIVE_PROB_1#114, (pythonUDF2#144) AS POSITIVE_PROB_2#128]
+- !ArrowEvalPython [_rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L), _rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L), _rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L)], [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6, pythonUDF0#142, pythonUDF1#143, pythonUDF2#144]
   +- LogicalRDD [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6], false

== Physical Plan ==
*(1) Project [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6, (pythonUDF0#142) AS POSITIVE_PROB_0#101, (pythonUDF1#143) AS POSITIVE_PROB_1#114, (pythonUDF2#144) AS POSITIVE_PROB_2#128]
+- !ArrowEvalPython [_rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L), _rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L), _rf_predict(F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L)], [F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6, pythonUDF0#142, pythonUDF1#143, pythonUDF2#144]
   +- Scan ExistingRDD[F1#0L, F2#1L, F3#2L, F4#3L, F5#4L, F6#5L, LABEL#6]
</pre>

Thanks for reading.
