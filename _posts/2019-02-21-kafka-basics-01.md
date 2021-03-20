---
title: 'WTF is Kafka? A High-level Overview'
date: 2019-02-21
permalink: /posts/2019/02/kafka-high-level-overview/
tags:
  - kafka
  - stream processing
  - message broker
---

Basically, you can presume Kafka as a messaging system. When an application sends a message to another application, one thing they need to do is to specify **how** to send the message. The most obvious use case in using a messaging system, in my opinion, is when we're dealing with big data. For instance, a sender application shares a large amount of data that need to be processed by a receiver application. However, the processing rate by the receiver is lower than the sending rate. Consequently, the receiver might be overloaded since it's unable to receive messages anymore while the processing is running. Although we're using distributed receivers, we still have to tell the sender about which receiver node it should send the message to.

Messaging system comes to rescue. Using a messaging system, the applications don't need to worry about how to share the data. They can only focus on the data itself.

# How Kafka Works? Step by step...

Let's start with a story of an application sharing it log messages to another application. In Kafka, a sender is called as a **producer**, while a receiver is called as a **consumer**.

## What is Producer?

A producer simply means an application that sends messages to a receiver. That's it.

When a **producer** generates message, it will specify the category of the message. In Kafka, such a category is called as a **topic** and every message should be classified into a topic. For instance, since in this story our **producer** generates log messages, then those log messages will be classified into, let's say, **LOG MESSAGE** topic.

## What is Kafka Topic?

A **Kafka Topic** is divided into one (at least) or more **partitions**. Think of **partition** as an array of messages. The messages are ordered and a new message is appended to the array (becomes the last element).

Let's continue the story...

After the **producer** classified a message into a **topic**, the message will be sent to a **Kafka cluster**. In Kafka, **Kafka cluster** simply means a collection of Kafka servers or brokers. These brokers manage and store the data sent by the **producer**.

## What is Kafka Broker?

A **Kafka Broker** stores zero or more **partitions**. In case you're wondering, a **topic** does not need to have all of its **partitions** in the same **Kafka Broker**. In case of messages, a **producer** may send a sequence of messages to different **partitions** within the same **topic**. However, in certain cases, we would want to send all the messages with the same key to a single **partition** only.

Another thing you need to know regarding **Kafka Broker** is that each partition can be replicated. We define the replication factor by ourselves. For instance, a replication factor 3 specifies that a partition will be replicated 3 times.

Let's take a look at an example to understand the **partition** replication better.

Suppose a **topic** named _mytopic_ has 1 **partition** only (P0) with replication factor 3. We also have 3 **brokers** (B0, B1, and B2). Therefore, the followings are the structure of **partition** division accross the provided **brokers**:

<ul>
  <li><b>P0</b> resides in <b>B0</b> and acts as the <i>leader</i></li>
  <li><b>P0 replica 1</b> resides in <b>B0</b></li>
  <li><b>P0 replica 2</b> resides in <b>B1</b></li>
  <li><b>P0 replica 3</b> resides in <b>B2</b></li>
</ul>

We can see the partitions state in _mytopic_ using the following command on Terminal:

<pre>
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic mytopic
Topic:mytopic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: mytopic  Partition: 0    Leader: 0   Replicas: 1,2,0 Isr: 1,2,0
</pre>

As you can see from the output above, Kafka describes _mytopic_ along with its **partitions**. The first line shows the properties of the topic:
<ul>
  <li><b>PartitionCount</b> states that <i>mytopic</i> has 1 <b>partition</b></li>
  <li><b>ReplicationFactor</b> states that each partition will be replicated 3 times</li>
</ul>

Meanwhile, the next line shows the partitions configurations. There are several properties, such as:
<ul>
  <li><b>Partition</b> which shows the <b>partition</b> ID (should be incremental)</li>
  <li><b>Leader</b> which shows the broker leader for this particular partition</li>
  <li><b>Replicas</b> which shows all the brokers in which the replicated partitions reside in</li>
  <li><b>Isr</b> which shows all the brokers containing the replicated partitions that are still alive (can be elected as the new leader later)</li>
</ul>

Based on this configuration, when **B0** (leader) fails, Kafka will elect a new leader from **B1** and **B2**. This replication feature makes Kafka able to handle server failure.

Now let's look at the partitions state again.

<pre>
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic mytopic
Topic:mytopic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: mytopic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 2,1
</pre>

As you can see, **B1** now becomes the leader for **P0**. In addition, **B0** is not included in **Isr** anymore since it is not active.

We've taken a deeper look at **producer**, **topic**, and **broker**. Now let's dive into the last element - **consumer**.

## What is Consumer?

A **consumer** simply means an application that receives messages from **producers**. That's it.

Ok, let's go deeper for **consumer**. Basically, Kafka uses a concept called _publish-subscribe_. Using such a concept, a **producer** can send messages related to a certain **topic** to the Kafka cluster. Any **consumer** subscribed to the topic will receive the messages. Well, I think this paragraph should be clear enough :)

Now, here comes the challenge.

What if the rate of publishing by a **producer** is much higher than the rate of message processing by a **consumer**? The most obvious consequence is that the final output by the **consumer** will be delayed.

Introducing **consumer group**.

Basically, a receiver application can be consisted of one or more consumer machines. When it has more than one consumer machines, then the collection of those machines is called as a **consumer group**.

Each **consumer** in a **consumer group** pulls messages from a certain **partition** (within the same topic off course). The processing result by each **consumer** will be aggregated later to form the final output.

Using **consumer group**, an application might give the output faster since the payload reading is distributed accross the **consumers**. However, if the number of **consumers** is more than the number of **partitions**, then there is a possibility that one or more **consumers** won't process any message. For instance, if we have 3 **partitions** and 5 **consumers**, then 2 **consumers** won't pull any message.

Here's another case. When we have 3 **partitions** and 2 **consumers** in a **consumer group**, then one **consumer** will pull messages from 2 **partitions** and another **consumer** will pull messages from 1 **partition**.

## Closing

Alright, we've reached the end of this post. Since this article only covers the high-level overview of Apache Kafka, there are several concepts that have not been explained in detail. Moreover, there are some advanced concepts like **message offset**, **partition rebalancing**, **zookeeper**, and so forth that have not been covered in this article. I hope I can write about them in the next articles.

Thank you for reading. Have a nice day!
