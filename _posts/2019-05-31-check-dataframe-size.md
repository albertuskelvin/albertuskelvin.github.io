---
title: 'How to Check the Size of a Dataframe?'
date: 2019-05-31
permalink: /posts/2019/05/check-dataframe-size/
tags:
  - spark
  - dataframe
  - size in memory
  - size in disk
---

Have you ever wondered how the size of a dataframe can be discovered? Perhaps it sounds not so fancy thing to know, yet I think there are certain cases requiring us to have pre-knowledge of the size of our dataframe. One of them is when we want to apply broadcast operation. As you might've already knownn, broadcasting requires the dataframe to be small enough to fit in memory in each executor. This implicitly means that we should know about the size of the dataframe beforehand in order for broadcasting to be applied successfully. Just FYI, broadcasting enables us to configure the maximum size of a dataframe that can be pushed into each executor. Precisely, this maximum size can be configured via <i>spark.conf.set("spark.sql.autoBroadcastJoinThreshold", MAX_SIZE)</i>.

Now, how to check the size of a dataframe? Specifically in Python (pyspark), you can use this code.

```python
import pyspark

df.persist(pyspark.StorageLevel.MEMORY_AND_DISK)
df.count()

# make a forever-loop condition so that we can inspect the Spark UI
i = 0
while True:
	i += 1
```

As you can see from the code above, I'm using a method called <i>persist</i> to keep the dataframe in memory and disk (for partitions that don't fit in memory). There are some parameters you can use for <i>persist</i> as described <a href="https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#rdd-persistence">here</a>. Afterwards, we call an action to execute the <i>persist</i> operation. Just FYI, according to this <a href="https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#rdd-persistence">article</a>, when an action is applied on the dataframe for the first time, the resulting dataframe will be kept in memory (depends on the parameter of <i>persist</i>). When there's another action applied on this dataframe, Spark will use the persisted dataframe, therefore makes the operation faster.

Another thing to note from the code above is the forever-loop condition. Basically, you can do anything here as long as the Spark application doesn't stop working. Up till this forever-loop point, you can go to the Spark UI which can be accessed via:

<pre>HOST_ADDRESS:SPARK_UI_PORT</pre>.

After you're in the Spark UI, go to the <b>Storage</b> tab and you'll see the size of your dataframe. As simple as that.
