---
title: 'Apache Cassandra: Begins with Docker'
date: 2019-08-17
permalink: /posts/2019/08/install-cassandra-docker/
tags:
  - cassandra
  - nosql
  - big data
  - docker
---

This article is about how to install Cassandra and play with several of its query languages. To accomplish that, I'm going to utilize Docker.

Since the topic covered in this article is really specific, I'll make it short. For those who want to know more about Cassandra, please visit its official page <a href="http://cassandra.apache.org/">here</a>.

**A. Open up your Terminal**<br/><br/>
**B. Create a network for our Cassandra container. Let's call it "cassandra-net"**<br/>

<pre>
docker network create cassandra-net
</pre>

**C. Pull Cassandra image from Docker hub, then create and run the container using the following command**<br/>

<pre>
docker run --name my-cassandra --network cassandra-net -d cassandra:latest
</pre>

<pre>
The above command does the followings:
<ul>
<li>Pull a Cassandra image with the latest version,</li>
<li>Create a container from it named "my-cassandra",</li>
<li>Put the container in a network called "cassandra-net",</li>
<li>Run the container in background (daemon process)</li>
</ul>
</pre>

**D. Let's create another Cassandra container. To make it simpler, we'll place both containers within the same node (local machine)**<br/>

<pre>
docker run --name my-cassandra-1 --network cassandra-net -d -e CASSANDRA_SEEDS=my-cassandra cassandra:latest
</pre>

<pre>
The above command does the followings:
<ul>
<li>Pull a Cassandra image with the latest version,</li>
<li>Create a container from it named "my-cassandra",</li>
<li>Put the container in a network called "cassandra-net",</li>
<li>Run the container in background (daemon process),</li>
<li>Connect the container to the first container (my-cassandra) via CASSANDRA_SEEDS. Since the containers reside within the same node, we can just use the container's name. In the case they are placed within different nodes, just use the node's address</li>
</ul>
</pre>

**E. After the above steps, we've already installed and run Cassandra. Now, let's do some interactions using several query commands**<br/><br/>
**F. Create another container dedicated for executing Cassandra Query Language Shell (CQLSH)**<br/>

<pre>
docker run -it --rm --network cassandra-net cassandra:latest cqlsh my-cassandra
</pre>

<pre>
The above command does the followings:
<ul>
<li>Pull a Cassandra image with the latest version,</li>
<li>Put the container in a network called "cassandra-net",</li>
<li>Remove the container automatically after it stops (after we exit the cqlsh),</li>
<li>Execute `cqlsh my-cassandra` command instead of the default one when the container starts</li>
</ul>
</pre>

**G. The previous `cqlsh` command enables us to execute shell commands**<br/><br/>
**H. In Cassandra, we need to create a keyspace before creating tables. Perhaps, you might presume a keyspace as a database**<br/>

<pre>
# create a keyspace called 'my-cassandra-test' using 'SimpleStrategy' class for data replication
CREATE KEYSPACE my-cassandra-test WITH REPLICATION={'class': 'SimpleStrategy', 'replication_factor': 1};

# list all existing keyspaces
SELECT * FROM system_schema.keyspaces;

# use the keyspace to create tables
USE my-cassandra-test;

# create a table
CREATE TABLE test-table (
    col0 varchar,
    col1 varchar,
    col2 varchar,
    col3 varchar,
    PRIMARY KEY ((col0), col1)
);

# insert some data into the table
INSERT INTO test-table (col0, col1, col2, col3) values ('dummy0_col0', 'dummy0_col1', 'dummy0_col2', 'dummy0_col3');
INSERT INTO test-table (col0, col1, col2, col3) values ('dummy1_col0', 'dummy1_col1', 'dummy1_col2', 'dummy1_col3');

# show the data
SELECT * FROM test-table;
</pre>

**I. Please try other query commands**<br/>

---

Thanks for reading.
