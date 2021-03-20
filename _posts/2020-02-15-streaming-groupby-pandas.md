---
title: 'Streaming GroupBy for Large Datasets with Pandas'
date: 2020-02-15
permalink: /posts/2020/02/streaming-groupby-pandas-big-data/
tags:
  - streaming
  - groupby
  - pandas
  - big data
---

I came across an article about how to perform `groupBy` operation for large dataset. Long story short, the author proposes an approach called <i>streaming groupBy</i> where the dataset is divided into chunks and the `groupBy` operation is applied to each chunk. This approach is implemented with pandas.

For those who are curious about the article, you can find it <a href="https://maxhalford.github.io/blog/streaming-groupbys-in-pandas-for-big-datasets/">here</a> (<i>Streaming Groupbys In Pandas For Big Datasets</i>). It's authored by Max Halford.

However, the operation is not merely applied to the whole chunk. There are several assumptions needed in order for this approach can work. One of the assumptions is that the dataset must consist of a column which acts as the grouping key, such as user ID. Another assumption is that this grouping key should be sorted first before the `groupBy` operation is performed.

Here's a sample data taken from the author's blog.

```
object_id   passband  flux    mjd
===================================
615         uu        52.91   59750
615         gg        381.95  59750
615         gg        384.18  59751
615         uu        153.49  59751
615         yy        -111.06 59750
713         yy        -180.23 59751
713         uu        61.06   59753
713         uu        107.64  59754
713         yy        -133.42 59752
713         uu        118.74  59755
```

In the above sample, `object_id` is the grouping key that has been sorted.

Below is the algorithm used to perform the streaming groupby:

<ul>
<li>Load the next <i>k</i> rows from a dataset</li>
<li>Identify the last group from the <i>k</i> rows</li>
<li>Put the <i>m</i> rows corresponding to the last group aside (called <i>orphans</i>)</li>
<li>Perform the groupBy on the remaining <i>k - m</i> rows</li>
<li>Repeat from step 1, and add the <i>orphan</i> rows at the top of the next chunk</li>
</ul>

Let's simulate the above algorithm with the previous sample data.

Suppose that we'd like to compute the mean of the flux for each pair of `object_id` and `passband`. In pandas, we can achieve this by simply using `df.groupby(['object_id', 'passband'])['flux'].mean()`. However, since this instruction will load all the data into memory, it surely will be an issue when the size of the data is large.

How about applying the streaming groupby approach?

Let's use `k = 7` to simulate the algorithm.

<b>i)</b> Load `k` rows from the dataset.

```
object_id   passband  flux    mjd
===================================
615         uu        52.91   59750
615         gg        381.95  59750
615         gg        384.18  59751
615         uu        153.49  59751
615         yy        -111.06 59750
713         yy        -180.23 59751
713         uu        61.06   59753
```

<b>ii)</b> Identify the last group from the `k` rows. Basically, we just take the group ID of the last row. In this case, the last group ID is 713.

<b>iii)</b> Put the `m` rows corresponding to the last group aside (called as orphans). In this case, the value of `m` is 2 since there are two rows whose group ID is 713.

<b>iv)</b> Perform the `groupBy` on the remaining `k - m` rows. In this case, we perform the `groupBy` operation on the first 5 rows. The last 2 rows is discarded for the next chunk. For the sake of completeness, here's the result of the `groupBy` operation.

```
group_id    passband    flux_mean
=================================
615         uu          103.2
615         gg          383.065
615         yy          -111.06
```

<b>v)</b> Repeat from step (i), and add the <i>orphan</i> rows at the top of the next chunk. The previous last two columns (group id 713) are added at the top of the next seven rows.

```
object_id   passband    flux      mjd
=======================================
713         yy          -180.23   59751	--> from the previous chunk
713         uu          61.06     59753	--> from the previous chunk
713         uu          107.64    59754
713         yy          -133.42   59752
713         uu          118.74    59755
```

The algorithm is quite simple, though. However, in my humble opinion, the used assumptions make this approach limited to particular cases. Additionally, requiring the dataset to be sorted first before applying the algorithm might be a critical issue when the data size is extremely large.

Apart from the assumptions, I was thinking of an extreme case which might introduce a critical risk for this approach.

Consider the following data.

```
row_id    object_id   passband    flux
======================================
1         100         uu          50
2         100         uu          50
3         100         vv          30
4         100         vv          30
...
50        100         ww          90
51        100         ww          90
52        100         ww          90
53        500         aa          10
54        500         bb          10
...
100       500         cc          30
101       500         cc          30
102       900         pp          10
103       900         qq          30
...
150       900         pp          10
151       900         qq          30
```

Based on the above sample data, we know that `object_id = 100` occupies 52 rows, `object_id = 500` occupies 49 rows, and `object_id = 900` occupies 50 rows.

Suppose that we use `k = 10` for this case.

Let's execute the algorithm.

<b>i)</b> Take `k` rows from the dataset. This should result in the first 10 rows of the data.

<b>ii)</b> Identify the last group from the `k` rows. Basically, we just take the group ID of the last row. In this case, the last group ID is 100.

<b>iii)</b> Put the `m` rows corresponding to the last group aside (called as orphans). In this case, the value of `m` is 10 since there are ten rows whose group ID is 100.

<b>iv)</b> Perform the `groupBy` on the remaining `k - m` rows. In this case, we perform the `groupBy` operation on 0 rows. All 10 rows are left out for the next chunk.

The `groupBy` operation was not performed at all for the first chunk since all the rows are left out for the second chunk.

Here's what the second chunk looks like.

```
row_id    object_id   passband    flux
======================================
1         100         uu          50
...
10        100         vv          50

--> The first 10 rows are from the first chunk

--> The next 10 rows are the actual data for the second chunk

11        100         ww          90
...
20        100         ww          90
```

When the algorithm is applied, we can see that there would be no `groupBy` operation performed on the second chunk since the third step discards all the rows again.

Consequently, we have to add this 20 rows at the top of the third chunk. Long story short, the third chunk now should consists of 30 rows with the same group id. Again, the `groupBy` operation won't be performed on this chunk.

To make it short, we should be able to execute the `groupBy` operation on the sixth chunk. This sixth chunk should consists of the additional 50 rows from the first till fifth chunk. Since the next 10 rows have different group id (500), the `groupBy` operation can now be applied.

However, we need to remember that the `groupBy` operation will only be applied on rows with group id 100 since all the rows with group id 500 will be put aside for the next chunk.

The problem might occur when the discarded rows grow in large quantity that causes the data can't be loaded into memory. Moreover, applying the algorithm over and over (which requires quite lots of checks) might introduce unnecessary performance issues.
