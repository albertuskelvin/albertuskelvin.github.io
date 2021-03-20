---
title: 'Incremental Query for Large Streaming Data Operation'
date: 2020-02-22
permalink: /posts/2020/02/incremental-query-big-data-stream-operations/
tags:
  - big data
  - incremental query
  - streaming
---

In the previous post, I wrote about how to perform pandas groupBy operation on a large dataset in streaming way. The main problem being addressed is optimum memory consumption since the data size might be extremely large.

However, the previous post also lists a few drawbacks of the approach. One of them is the assumption that rows with the same grouping key must reside in the same chunk. Another one is related to an example of extreme case that might result in a not enough memory exception. Please visit the previous <a href="https://albertuskelvin.github.io/posts/2020/02/streaming-groupby-pandas-big-data/">post</a> for a more detailed explanation.

I was thinking of another approach to attack this big dataset transformation problem. The initial goal was to eliminate the previous assumptions, especially the first one since it somehow limits the possible cases on which the approach applied.

Basically, the applied approach can be presumed like what's so called as incremental query. To make it short, here's the algorithm overview.

a) Read the data in chunks<br/>
b) Append the chunk to an **Input Table** (unbounded table)<br/>
c) Apply the data transformations on the **Input Table**. This step returns a **Result Table** that will be updated eventually<br/>
d) Repeat step b till c for the other chunks<br/>

Take a look at the following example for a better understanding.

Suppose that we decided to divide the data into chunks with two rows each.

Initially, the <b>Input Table</b> and <b>Result Table</b> are empty.

```
Original data
-------------
... Row 0 ...
... Row 1 ...
... ..... ...
... ..... ...
... Row 8 ...
... Row 9 ...
-------------

a) Read the first two chunks

... Row 0 ...
... Row 1 ...

b) Append the chunks to the Input Table

 Input Table
-------------
... Row 0 ...
... Row 1 ...
-------------

c) Apply the transformations on the Input Table. We'll get the Result Table

      Result Table
-----------------------
Transformed Input Table
-----------------------

d) Repeat the above steps for the other chunks. Read the next two chunks.

... Row 2 ...
... Row 3 ...

e) Append the chunks to the Input Table

 Input Table
-------------
... Row 0 ...
... Row 1 ...
... Row 2 ...
... Row 3 ...
-------------

f) Apply the transformations on the Input Table. We'll get the Result Table

      Result Table
-----------------------
Transformed Input Table
-----------------------

g) Repeat the above steps for the other chunks
```

This approach surely eliminates the drawbacks introduced at the beginning of this article, such as:

<b>a)</b> The assumption that rows with the same grouping key must reside in the same chunk.<br/><br/>

Here's an example.

```
TABLE A

col_a   |   col_b   |   col_c
-----------------------------
row_ax  |   row_bx  |   ...
row_ay  |   row_by  |   ...
row_az  |   row_bz  |   ...
row_ax  |   row_bx  |   ...
row_ay  |   row_by  |   ...
row_az  |   row_bz  |   ...
```

The previous approach requires the above dataset to be transformed to the following.

```
TABLE B

col_a   |   col_b   |   col_c
-----------------------------
row_ax  |   row_bx  |   ...
row_ax  |   row_bx  |   ...
row_ay  |   row_by  |   ...
row_ay  |   row_by  |   ...
row_az  |   row_bz  |   ...
row_az  |   row_bz  |   ...
```

With the above transformed dataset, we can now apply the previous approach.

The current approach (incremental query) enables each data instance arrive in random order. In other words, we can just execute the operation (groupby, etc.) on <b>TABLE A</b>.

<b>b)</b> An extreme case that might introduce the not enough memory exception. Please refer to the previous <a href="https://albertuskelvin.github.io/posts/2020/02/streaming-groupby-pandas-big-data/">post</a> for a more detailed explanation on this point.<br/><br/>

To address this possible issue, in my opinion, we need a more robust approach, such as distributed computing. Unless the resources of the single machine is supportive enough for large dataset, I think it's obvious to leverage big data tools.

One of the most obvious reasons to use distributed approach is that since this incremental query builds up the <b>Input Table</b> continuously (unbounded table), this table surely will occupy lots of memory spaces. It should be partitioned and stored in multiple machines. Hence, this approach should be more robust towards the not enough memory exception. 

---

In principal, this incremental query approach has been introduced already in Spark's structured streaming. Please visit this <a href="https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html">page</a> for more details.

---

What do you think? Please provide your suggestion if you have better approaches.
