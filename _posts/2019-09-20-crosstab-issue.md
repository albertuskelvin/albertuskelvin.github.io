---
title: 'Crosstab Does Not Yield the Same Result for Different Column Data Types'
date: 2019-09-20
permalink: /posts/2019/09/crosstab-data-loss/
tags:
  - pyspark
  - crosstab
  - dataframe
  - groupby
  - pivot
---

I encountered an issue when applying crosstab function in PySpark to a <b>pretty big</b> data. And I think this should be considered as a pretty big issue. Please note that the context of this issue is on Sep 20, 2019. Such an issue might have been solved in the future.

Suppose we have a dataframe <b>df</b> with two columns, <b>A</b> and <b>B</b>. Column <b>A</b> consists of lots of unique values, while column <b>B</b> consists of two unique values, namely 0 and 1. We would like in this case, to calculate the number of each unique value of column <b>B</b> for each unique value of column <b>A</b>.

Let’s take a look at an example for the sake of clarity.

<pre>
DATAFRAME df

=============================
	A	|	B
=============================
unique_val_a	|	0
unique_val_a	|	0
unique_val_a	|	0
unique_val_b	|	1
unique_val_c	|	1
unique_val_c	|	1
unique_val_c	|	1
unique_val_d	|	0
unique_val_d	|	0
unique_val_d	|	0
unique_val_d	|	1
=============================


EXPECTED OUTPUT

============================================
	A_B	|	0	|	1
============================================
unique_val_a	|	3	|	0
unique_val_b	|	0	|	1
unique_val_c	|	0	|	3
unique_val_d	|	3	|	1
============================================
</pre>

Accomplishing such a result is extremely simple though thanks to PySpark’s statistical function, such as <I>crosstab</I>. Let’s see how we can get the result.

```python
df = df.crosstab(‘A’, ‘B’)
```

<h2>THE PROBLEM</h2>

The function seems perfectly fit to our needs from the outside. However, I came across an odd result when playing with the data type of column <b>B</b>. At that time I was experimenting with <I>integer</I> and <I>string</I> data type only. Let’s take a look at the code.

```python
from pyspark.sql.types import IntegerType, StringType

int_df = df.withColumn(‘B’, F.col(‘B’).cast(IntegerType()))
int_df = int_df.crosstab(‘A’, ‘B’)
print(int_df.count())

str_df = df.withColumn(‘B’, F.col(‘B’).cast(StringType()))
str_df = str_df.crosstab(‘A’, ‘B’)
print(str_df.count())
```

The focus was that I wanted to check whether different data type yielded the same number of rows. I thought that there was no need to check the dataframes’ content (<I>int_df</I> and <I>str_df</I>) when the number of rows was different.

The result? Surprisingly it <b>didn’t yield the same result</b> for <I>integer</I> and <I>string</I> data type. The difference was pretty small, however. But the point here is that the function (<I>crosstab</I>) doesn’t yield the same output for different data types. I’m not sure about the rationales since there is no any information regarding data types usage in the documentation. Perhaps I need to look at the code base.

What made the case worse was that each dataframe (<I>int_df</I> and <I>str_df</I>) seemed to have different data. We can check it by computing the set difference of column <b>A</b> between <I>int_df</I> and <I>str_df</I>. Let’s take a look.

```python
set(int_df.select(‘A_B’).collect()) - set(str_df.select(‘A_B’).collect())
```

As I said before, the above code <b>didn’t yield an empty set</b>. In my case, both sets differed by many elements. Obviously, this should not be a good news as it means that this <I>crosstab</I> depends on the used data type. Since there’s no any documentation on data type use when using <I>crosstab</I>, I think the developers might use the function inappropriately.

Since I was curious, I created a <b>small</b> dataframe (the one explained before was pretty big) and applied the same scenario (two different data types for column <b>B</b>) just to investigate the behaviour. I expected that the result should be different as shown by the big dataframe. My expectation was wrong since both dataframe yielded the same result.

Based on the quick investigation using small and pretty big data, up to this point I haven’t known the limit of dataframe size in order for <I>crosstab</I> to result in the same output for different data types.

<h2>THE SOLUTION</h2>

But let’s leave this issue for a while. I decided to search for a better solution.

The principle was since <I>crosstab</I> utilises the concept of <I>groupby</I>, I thought that <I>groupby</I> with a few engineering tricks might be the solution. Let’s take a look at the code.

```python
from pyspark.sql.types import IntegerType, StringType

int_df = df.withColumn(‘B’, F.col(‘B’).cast(IntegerType()))
int_df = int_df.groupby(‘A’).pivot(‘B’, [0, 1]).count().fillna(0)
print(int_df.count())

str_df = df.withColumn(‘B’, F.col(‘B’).cast(StringType()))
str_df = str_df.groupby(‘A’).pivot(‘B’, [‘0’, ‘1’]).count().fillna(0)
print(str_df.count())
```

As you can see, the engineering trick is implemented by <I>pivot</I>. Simply, <I>pivot</I> creates a tabular data with several main columns first followed by elements provided in the list (second argument of <I>pivot</I> method). We apply a <I>count</I> method to calculate the number of each unique value of column <b>B</b>. And then <i>fillna</i> to replace all <I>null</I> values with zero (it seems that the <I>count</I> method only returns values more than 0).

Here comes the good news. Using this <I>groupby-pivot</I> approach yields the <b>same</b> result for both <I>integer</I> and <I>string</I> data type.

Now, let’s execute the set difference computation.

```python
set(int_df.select(‘A’).collect()) - set(str_df.select(‘A’).collect())
```

Well, it resulted in an <b>empty set</b>, which is good.

<h2>OVERALL CHECK & OBSERVATION</h2>

I compared the result returned by <I>crosstab</I> and <I>groupby_pivot</I> with <I>groupby_only</I> approach. For the sake of clarity, let’s take a look at the code.

```python
gb_only = df.groupby(‘A’).count()
print(gb_only.count())
```

The above code yielded the same result as what was returned by using <I>groupby_pivot</I> approach. <b>This should clarify that <I>groupby_pivot</I> really works correctly</b>.

Last but not least, I encountered that the number of rows (by <I>count()</I> method) returned by <I>crosstab</I> and <I>groupby_pivot</I> differed by factor of two (approximately). This means that the number of rows returned by <I>groupby_pivot</I> was approx. two times more than the number of rows returned by <I>crosstab</I>.

What does this mean?

Well, according to me, this means that <I>crosstab<I> function might cause what’s called as <b>data loss</b>. Using the above case, we can consider that this function didn’t retain the other 50% (approximately) of the data.

<h2>WHAT DO YOU THINK?</h2>

So, do you have any idea about this PySpark’s <I>crosstab</I> issue? I would love to know your thoughts.
