---
title: 'Kafka Producer Awareness of Topic Metadata (Partitions) Updates'
date: 2019-11-04
permalink: /posts/2019/11/kafka-producer-awareness-of-metadata-updates/
tags:
  - kafka
  - producer
  - metadata
  - partition
---

In the previous article about <a href="https://albertuskelvin.github.io/posts/2019/10/kafka-consumer-awares-of-new-partitions/">Kafka Consumer Awareness of New Topic Partitions</a>, I wrote about partitions balancing by Kafka consumers. In other words, I’d like to see whether Kafka consumers are aware of new topic partitions.

This article will take up the same topic but in the context of Kafka producer.

Just FYI, I’ll be using Kafka-python v.1.4.7 as the client.

Let’s imagine the following scenario. An application produces some messages to a Kafka topic with X partitions. One day, the server admin decided to add one more partition to the topic making the total number of partitions is X + 1. For now, let’s ignore the type of partitioner used by the admin and presume that the producer doesn’t know the partition that the message should be stored in.

The question then would be, is the producer aware of the new partitions?

To answer such a question, I chose to read the codebase of <a href="https://github.com/dpkp/kafka-python">kafka-python</a> since source code is the source of truth of a software. Precisely, the answer should be located on the following modules, <a href="https://github.com/dpkp/kafka-python/blob/master/kafka/producer/kafka.py">Kafka producer</a> and <a href="https://github.com/dpkp/kafka-python/blob/master/kafka/partitioner/default.py">Kafka partitioner</a>.

The <b>send</b> method in Kafka producer module clearly states that the producer will request for cluster metadata update before sending the messages. To be precise, this procedure is done by calling <b>wait_on_metadata</b> for a certain topic. By doing so, the producer expects to receive the most recent information regarding the topic (in this case, the topic's partitions) it would like to send the message to.

Below is the code snippet showing what initially happens in the <b>send</b> method.

```python
# request for metadata updates
self._wait_on_metadata(topic, self.config['max_block_ms'] / 1000.0)

key_bytes = self._serialize(
	self.config['key_serializer'],
	topic, key)
value_bytes = self._serialize(
	self.config['value_serializer'],
	topic, value)

assert type(key_bytes) in (bytes, bytearray, memoryview, type(None))
assert type(value_bytes) in (bytes, bytearray, memoryview, type(None))

# retrieves information regarding the partition to which the message should be sent
partition = self._partition(topic, partition, key, value,
			key_bytes, value_bytes)
```

Upon receiving the metadata, the producer then do a sort of serialization to the key and message.

And here’s the last part. The producer then calls the partitioner to store the message. This is done by calling the <b>_partition</b> method. Below is the code snippet of the <b>_partition</b> method.

```python
def _partition(self, topic, partition, key, value, serialized_key, serialized_value):
	if partition is not None:
		assert partition >= 0
		assert partition in self._metadata.partitions_for_topic(topic), 'Unrecognized partition'
		return partition

	all_partitions = sorted(self._metadata.partitions_for_topic(topic))
	available = list(self._metadata.available_partitions_for_topic(topic))
	return self.config[‘partitioner’](serialized_key, all_partitions, available)
```

The above code shows that both <I>all_partitions</I> and <I>available</I> variables store the most recent partitions information for the corresponding topic. This is accomplished by calling <b>_metadata.partitions_for_topic</b> and <b>_metadata.available_partitions_for_topic</b>. Since the producer already refreshed the metadata, then both methods should return the most recent values.

Hence, the producer should be aware of new topic partitions.
