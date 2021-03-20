---
title: 'CAP Theorem'
date: 2019-10-25
permalink: /posts/2019/10/cap-theorem/
tags:
  - cap theorem
  - distributed system
---

"Consistency, Availability, and Partition Tolerance" - choose <b>two</b>.

To explain those terms clearly, let’s presume that we have a cluster with two nodes, <b>A</b> and <b>B</b>. Also, a client <b>C</b> that will communicate to the cluster.

Node <b>A</b> acts as a master node, while node <b>B</b> acts as a slave node. When the master node receives a message, it replicates the message on the slave node. Let’s suppose that initially both nodes store a value <b>Va</b>.

<h2>Consistency</h2>

When the client <b>C</b> sends a “read” request to either node <b>A</b> or node <b>B</b>, both nodes have to return the <b>same</b> value.

An inconsistent event occurs when the client successfully sends a message <b>Vb</b> to node <b>A</b>, yet the client still receives <b>Va</b> when sending a “read” request to node <b>B</b>.

<h2>Availability</h2>

Simply, all nodes in the cluster must response to client’s requests anytime. Ideally, the response must be returned with an extremely small latency.

Suppose that the client sends a message <b>Vb</b> to the master node. The master node sends back an "ACK" message to the client. The master node then replicates the message <b>Vb</b> to the slave node.

Now, the client sends a "read" request to the slave node. The problem is that the replication process has not been completed yet. This introduces a kind of latency when reading a message from the slave node. In other words, this latency makes the slave node "unavailable" for a certain amount of time.

<h2>Partition Tolerance</h2>

Basically, we want all the nodes in the cluster can communicate with each other smoothly. However, there's an occasion when the cluster runs into network partition. Having such an issue, the master node might not be able to replicate the messages to the slave node.

Turns out that each node won't be in a consistent state.

---

Now, the more interesting question would be how we prove this CAP theorem?

I think the easiest way to prove the theorem is by applying contradiction approach.

For our initial assumption, let’s assume that we have a distributed system that is consistent, available, and tolerant to network partition.

Our cluster runs into network intrusion making the nodes can’t send messages to each other.

With such a condition, the client may not receive consistent output as the master node couldn’t replicate the messages to the slave node.

This condition contradicts with the initial assumption. Hence, such a system doesn’t exist.

---

When designing a real-world distributed system, network partition always occurs. This obviously means that partition tolerance should be included to our list. This also means that now we only need to choose one out of two (consistency or availability).

Let’s dig deeper into every possible choice.

<b>Consistent-Available</b>

In my opinion, we might don’t want to opt this since the probability for network partition to happen is really high.

Additionally, how the system could achieve a consistent state where each node couldn’t communicate with each other?

Do you think this option is possible?

<b>Consistent-Partition Tolerance</b>

In this option we sacrifice availability. Although the system is consistent, the client might receive messages with a certain amount of latency.

This is caused by the process of replicating messages across the cluster which takes a certain amount of time.

<b>Available-Partition Tolerance</b>

In this option we sacrifice consistency. The client can receive messages with a small amount of latency, yet the messages received might not be the most updated ones.

This happens because the replication process needs to be performed successfully across the cluster in order to achieve consistency.

Thank you for reading.
