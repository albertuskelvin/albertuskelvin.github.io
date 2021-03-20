---
title: 'A Brief Overview of PySpark Logging'
date: 2019-02-08
permalink: /posts/2019/02/pyspark-logging/
tags:
  - pyspark
  - logging
---

This article is about a brief overview of how to write log messages using PySpark logging.<br/>

# Log Properties Configuration

## I. Go to the **conf** folder located in PySpark directory.<br/>

<p>
  <i>
$ cd spark-2.4.0-bin-hadoop2.7/conf
  </i>
</p>

## II. Modify the **log4j.properties.template** by appending these lines:<br/>

```
# Define the root logger with Appender file
log4j.rootLogger=WARN, console

# Define the file appender
log4j.appender.FILE=org.apache.log4j.DailyRollingFileAppender

# Name of the log file
log4j.appender.FILE.File=/tmp/logfile.out

# Set immediate flush to true
log4j.appender.FILE.ImmediateFlush=true 

# Set the threshold to DEBUG mode
log4j.appender.FILE.Threshold=debug

# Set File append to true
log4j.appender.FILE.Append=true

# Set the Default Date pattern
log4j.appender.FILE.DatePattern='.' yyyy-MM-dd

# Default layout for the appender
log4j.appender.FILE.layout=org.apache.log4j.PatternLayout
log4j.appender.FILE.layout.conversionPattern=%m%n
```

---

<p>
Then, save the configuration file (log4j.properties.template) and give it a new name: <b>log4j.properties</b>
</p>

<p>
You can adjust the configuration file in accordance with your specific needs. Please visit log4j documentation for more information. However, the above configuration is quiet enough for a simple logging task.
</p>

---

<h1>Use Logging in Your PySpark Code</h1>

Execute this simple code:<br/>

```python
from pyspark import SparkContext, SparkConf

conf = SparkConf().setAppName("pyspark-logging")
sc = SparkContext(conf=conf)
log4jLogger = sc._jvm.org.apache.log4j

log = log4jLogger.LogManager.getLogger(__name__)
log.trace("Trace Message!")
log.debug("Debug Message!")
log.info("Info Message!")
log.warn("Warn Message!")
log.error("Error Message!")
log.fatal("Fatal Message!")
```

---

If you want to write the log messages into a file, you can modify these property config lines:<br/>

```
log4j.rootLogger=WARN, FILE
log4j.appender.FILE.File=path_to_the_log_file
```

FYI, you can use any name for the appender. In this case, we're using 'FILE' as the appender's name.
