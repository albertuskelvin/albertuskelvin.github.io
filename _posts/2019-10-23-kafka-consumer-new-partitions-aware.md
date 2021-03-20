---
title: 'Kafka Consumer Awareness of New Topic Partitions'
date: 2019-10-23
permalink: /posts/2019/10/kafka-consumer-awares-of-new-partitions/
tags:
  - kafka
  - python
  - topic
  - partition
  - producer
  - consumer
---

Just wanted to confirm whether the Kafka consumers were aware of new topic’s partitions.

I’m going to set up a simple messaging scenario with a broker and a topic with one partition at first. This setting is done in local mode.

Just FYI, I used Kafka with version 2.1.0 and kafka-python with version 1.4.6.

Since setting up Kafka cluster in local might be cumbersome, I’ll be using GNU screen so that I don’t need to open up many Terminal windows.

In addition, I’ll leave all the configs with their default settings. Please adjust in accordance with your needs.

Open up the Terminal, go to the Kafka’s installation directory and do the followings.

<h2>Run the Zookeeper</h2>

```
# create a screen for the zookeeper
screen -S zookeeper

# run the zookeeper inside the screen
./bin/zookeeper-server-start.sh config/zookeeper.properties

# detach from the screen by pressing Ctrl-A + D
```

<h2>Run the Kafka broker</h2>

```
# create a screen for the broker
screen -S broker

# run the broker inside the screen
./bin/kafka-server-start.sh config/server.properties

# detach from the screen by pressing Ctrl-A + D
```

<h2>Create a topic</h2>

```
# create a screen for the topic creation
screen -S topic_creation

# run the topic creation inside the screen
./bin/kafka-topics.sh --create \
--zookeeper zoo_kpr_host:zoo_kpr_port \
--replication-factor 1 \
--partitions 1 \
--topic test

# detach from the screen by pressing Ctrl-A + D
```

Please note that the option <b>--zookeeper</b> should follow the location of the zookeeper which is set in the <b>config/zookeeper.properties</b>.

The above code simply tells Kafka to create a topic called <b>test</b> with one partition which is replicated once. The broker on which the partition is located will be determined by the zookeeper residing on <b>zoo_kpr_host</b> (port <b>zoo_kpr_port</b>).

To verify that the topic has been created with the specified requirements (number of partitions, replication factor, etc.), we can do the following on the same screen.

```
# get inside the screen
screen -r topic_creation

# describe the topic
./bin/kafka-topics.sh \
--describe \
--zookeeper zoo_kpr_host:zoo_kpr_port \
--topic test

# detach from the screen by pressing Ctrl-A + D
```

By now, our Kafka cluster has been set up completely. We can now publish some messages and consume them.

<h2>Create a producer</h2>

```
# create a screen for the producer
screen -S producer

# run the producer
python path/to/the/producer_py_file

# detach from the screen by pressing Ctrl-A + D
```

Here’s an example of a simple producer in Python. Please make sure that <b>kafka-python</b> has been installed.

```python
import json

from kafka import KafkaProducer


producer = KafkaProducer(bootstrap_servers=['host:port'],
                         key_serializer=lambda m: m.encode('utf8'),
                         value_serializer=lambda m: json.dumps(m).encode('utf8'))


def loop_produce(n: int = 100):
	for i in range(n):
		if i % 2 == 0:
			producer.send('test', key='0', value=dict(hello_a=i))
		else:
			if i % 3 == 0:
				producer.send('test', key='1', value=dict(hello_b=i))
			else:
				producer.send('test', key='2', value=dict(hello_c=i))
				
			
	producer.flush()

loop_produce()
```

The above code simply does the followings:

<ul>
<li>create a Kafka producer object. This producer should pass its message to a Kafka cluster whose location is specified by <b>bootstrap_servers</b>. This parameter lists all of your brokers in the cluster</li>
<li>the producer applies a serializer for key and value. The key-value pair should be in the form of <b>{key: value}</b> in which <b>key</b> is a string and <b>value</b> is a JSON</li>
<li>execute the producer method called <I>loop_producer</i>. The method simply sends out <b>n</b> messages to topic <b>test</b>. Each message will be located on a partition specified by the <b>key</b>. Simply put, by applying hash partitioning, messages with key X will be located on a partition <b>hash(X) modulo num_of_partitions</b></li>
</ul>

<h2>Create a consumer</h2>

```
# create a screen for the consumer
screen -S consumer

# run the consumer
python path/to/the/consumer_py_file

# detach from the screen by pressing Ctrl-A + D
```

Here’s an example of a simple consumer in Python. Please make sure that <b>kafka-python</b> has been installed.

```python
import json

from kafka import KafkaConsumer
from kafka.structs import OffsetAndMetadata, TopicPartition


consumer = KafkaConsumer(bootstrap_servers=['host:port'],
                         key_deserializer=lambda m: m.decode('utf8'),
                         value_deserializer=lambda m: json.loads(m.decode('utf8')),
                         auto_offset_reset="earliest",
                         group_id='1')

consumer.subscribe(['test'])

for msg in consumer:
	print(‘Consumer Record’)
	print(msg)

	tp = TopicPartition(msg.topic, msg.partition)
	offsets = {tp: OffsetAndMetadata(msg.offset, None)}
	consumer.commit(offsets=offsets)
```

The above code simply does the followings:

<ul>
<li>creates a consumer object that will be subscribing topics from a Kafka cluster whose location is specified by <b>bootstrap_servers</b></li>
<li>applies a deserializer for key and value</li>
<li>assigns the consumer object to a group with ID ‘1’</li>
<li>subscribes to a topic called <b>test</b></li>
</ul>

The subscription is performed on long-running basis. When a new message is fetched by the consumer, the consumer record is printed out. Basically, the consumer record consists of several information, such as the topic, partition, key, and value.

Afterwards, the consumer simply commits the consumed message. This commit is performed to tell Kafka that the corresponding messages have been read. Usually, this commit is called after all the processing on the message is done.

By committing the messages, when an exception occurs and makes the consumer stop working, the new consumer won’t read the messages that has already been processed.

I think we’re done with the setup. Go back to the main question. I’d like to confirm whether the <b>Kafka consumers are aware of new topic’s partitions</b>.

To verify it, let’s add another partition to the topic <b>test</b>.

Go back to the <b>topic_creation</b> screen and add one more partition.

```
# go inside topic_creation screen
screen -r topic_creation

# add one more partition
./bin/kafka-topics.sh \
--zookeeper zoo_kpr_host:zoo_kpr_port \
--alter \
--topic test \
--partitions 2

# detach from the screen by pressing Ctrl-A + D
```

Kafka will print out a confirmation message denoting that the number of partitions has been modified.

Let's re-run the producer.

```
# go inside producer screen
screen -r producer

# re-run the producer
python path/to/the/producer_py_file

# detach from the screen by pressing Ctrl-A + D
```

Wait for a second. The consumer will then fetch messages from <b>both</b> partitions. You can try to add more partitions. The result should be the same that the consumer would still retrieve data from all partitions.

This means that <b>at least for this Kafka & kafka-python version and by using the specific configuration (one consumer in a group consuming more than one partitions), the consumer is aware of new partitions existence</b>.

Additional thing to investigate is how a consumer fetches messages coming from more than one partitions. Does it read all the messages from a certain partition first before doing the same for other partitions?

I'll leave it for another day.

Thank you for reading.
