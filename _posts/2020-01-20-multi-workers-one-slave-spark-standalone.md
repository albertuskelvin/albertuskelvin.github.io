---
title: 'Multiple Workers in a Single Node Configuration for Spark Standalone Cluster'
date: 2020-01-20
permalink: /posts/2020/01/spark-standalone-multi-workers-single-node/
tags:
  - spark
  - standalone mode
  - workers
---

A few days back I tried to set up a spark standalone cluster in my own machine with the following specification: two workers (balanced cores) within a single node.

Here's how you'd do it.

<b>Step 1.</b> Start the master instance via `<path_to_sbin>/start-master.sh`

<b>Step 2.</b> Configure the spark environment properties.

Go to the `conf` directory and add the following lines to `spark-env.sh`.

```
export SPARK_WORKER_INSTANCES=<number_of_workers>
export SPARK_WORKER_CORES=<total_cores_for_the_workers>
```

The property names should be intuitive. We specified the number of worker instances and the total number of cores for the workers.

<b>Step 3.</b> Start the slave node via `<path_to_sbin>/start-slave.sh <path_to_master>`

<b>Step 4.</b> To check the configuration results, just go to `<your_host_address>:<spark_master_webui_port>`.
