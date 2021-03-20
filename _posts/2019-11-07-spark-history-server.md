---
title: 'Spark History Server: Setting Up & How It Works'
date: 2019-11-07
permalink: /posts/2019/11/setting-up-spark-history-server/
tags:
  - spark
  - history server
  - performance debugging
  - monitoring
---

Application monitoring is critically important, especially when we encounter performance issues. In Spark, one way to monitor a Spark application is via Spark UI. The problem is, this Spark UI can only be accessed when the application is running.

Since the UI is only available when the job is running, we may not be able to debug the performance in case the application stopped because of an error.

Fortunately, Spark comes with a history server, a tool that supports monitoring and performance issue debugging. This history server can be accessed even though the application isn’t running.

The way this history server works is actually simple. It basically just records the application’s event logs and shows them via its dedicated page.

Setting up a history server only requires a few steps.

a) Add the followings to the <b>spark-defaults.conf</b> file

```
spark.eventLog.enabled			true
spark.eventLog.dir			path/to/event/log
spark.history.fs.logDirectory		path/to/event/log
```

b) Start the history server with the following command:

```
- Go to the 'sbin' directory
- Execute: ./start-history-server.sh
```

c) Go to the history server UI located on <b>http://localhost:18080</b>

d) To stop the history server, use the following command:

```
- Go to the 'sbin' directory
- Execute: ./stop-history-server.sh
```

As you can see, we added three additional parameters to the spark’s config files. Let’s take a look at the brief overview of those parameters.

<ul>
<li><b>spark.eventLog.enabled</b>: denotes that the Spark app should output its event logs</li>
<li><b>spark.eventLog.dir</b>: specifies the location in which the Spark app should output its event logs</li>
<li><b>spark.history.fs.logDirectory</b>: specifies the location of the event logs that will be loaded by the history server</li>
</ul>

The definition of the above parameters should be clear enough to explain how the history server works in general.

<ul>
<li>Imagine that event logs are the dumped version of the Spark UI information</li>
<li>The Spark app dumps its progress status to a file which location is specified by <b>spark.eventLog.dir</b></li>
<li>Within a certain period of time, the history server will check the directory specified by <b>spark.history.fs.logDirectory</b>. This directory simply contains all the event logs from one or more Spark apps.
<br/>
<br/>
Upon loading the event log data, the history server builds up the UI page describing the event log data which is the same as what is shown in the Spark UI when the application is still running. That’s why the value of <b>spark.history.fs.logDirectory</b> should be the same as <b>spark.eventLog.dir</b>.
</li>
</ul>
