---
title: 'Kafka Partitioning Consistency After Topic Metadata Updates'
date: 2019-11-05
permalink: /posts/2019/11/kafka-partitioning-consistency-after-metadata-updates/
tags:
  - kafka
  - partition
  - custom partitioner
  - metadata
  - hash
---

I used kafka-python v.1.4.7 as the client.

Last time I was experimenting with effects of Kafka’s metadata updates on producer & consumer side. Specifically, the metadata includes the number of partitions of a topic.

In my opinion, changing the number of partitions might probably disrupt the future stored data. Let’s imagine a simple scenario. A producer sends a message by specifying the key without any custom partitioner. Doing so should make Kafka applies the default partitioner. This default partitioner leverages murmur hash algorithm at its core. The formula used by the default partitioner is similar to `murmurhash(key) % num_of_partitions`.

We might use several simple test cases to check the shift of partitioning result. Suppose that the initial number of partitions is <b>N</b>.
<ul>
<li>The first message with key <b>K_a</b> goes to partition <b>p_x</b></li>
<li>The second message with key <b>K_b</b> goes to partition <b>p_y</b></li>
<li>The third message with key <b>K_c</b> goes to partition <b>p_z</b></li>
</ul>

An admin did a bit modification to the number of partitions, making it to <b>N+m</b> where <b>N+m</b> > <b>N</b>. In this case, the result of the previous partitioning formula would be different for each key.
<ul>
<li>The fourth message with key <b>K_a</b> might go to partition <b>p_y</b></li>
<li>The fifth message with key <b>K_b</b> might go to partition <b>p_x</b></li>
<li>The sixth message with key <b>K_c</b> might go to the new partition <b>p_w</b></li>
</ul>

The above might also happen when the number of partitions is reduced.

So, the question would be, is there a way of ensuring that the same key will always go to the same partition?

Well, when the partitioner only purely leverages the key, there’s a high chance that the same key will always go to the same partition. This implicitly states that we need to create a custom partitioner to accomplish such a condition.

Here’s an example of such a customised partitioner considering that we don’t need to do any transformation on the key, such as converting the key (which is a string) to an integer.

```
partition_id = key
```

However, there might be several drawbacks of using the above approach. One of them is the producer doesn’t have the control of the topic’s metadata. In case of the number of partitions is modified, the producer doesn’t know anything about it unless the admin informs the application developer.

So, is there another approach?

I think when the hash result is always less than the number of partitions (<b>Assumption A</b>), the partitioning might be consistent. Let’s take a look.

```
partition_id = hash(key) % num_of_partitions
```

The above is pretty similar to the formula used by the default partitioner. Instead of using murmurhash partitioner, we use the hash method from Python. The background of this hash method is pretty simple. We can use `sys.hash_info` to get a list of hashing information.

One of the information we get from `sys.hash_info` is <b>modulus</b>. To explain what this variable is, let’s take a look at the following code snippet.

```
for i = -(modulus - 1) to (modulus - 1)
	print hash(i)
```

The above code snippet will print a set of numbers ranging from -(modulus - 1) to (modulus - 1). This means that `hash(i) = i` for the corresponding range.

However, if the number is outside of the range, then the hash method will return `number % modulus`. Consequently, `hash(i) != i` if `i` doesn’t lie between -(modulus - 1) and (modulus - 1).

I think the behaviour of this hash method might be relevant for partitioning consistency. When the Murmurhash returns a large number for a small key (`key << murmurhash(key)`), this hash function returns the same number as the key when the range assumption holds.

Since the hash function always returns the same value as the key (`hash(key) = key`), we can now apply <b>Assumption A</b>. As a reminder, here’s what the assumption says: <I>"when the hash result is always less than the number of partitions, the partitioning might be consistent because the result of `hash(key) % num_of_partitions` always equals to `hash(key)`"</I>.
