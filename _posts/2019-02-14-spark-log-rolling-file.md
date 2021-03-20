---
title: 'Rolling File Appender in PySpark Logging'
date: 2019-02-14
permalink: /posts/2019/02/spark-log-rolling-file/
tags:
  - spark
  - logging
  - rolling file appender
---

This article is about a brief overview of how to store several of the most recent log files in PySpark logging.

# Introduction

Appender in Spark logging is used to show the log messages in a specified destination, such as console or file. Special case occurs when the log messages are directed to output destinations that might increase the storage size i.e. log files. For example, an error would occur in your streaming application after several hours for the log files' size grew and occupied most of the storage space.

Basically, to debug the application, we only need to analyze several of the most recent log files. Since it's impossible and not a good practice to remove the old log files manually, we do need another way to clean up the storage space from irrelevant log files. In Spark, we can use **Rolling File Appender** method to resolve such an issue.

Here are the two main properties to consider when using Rolling File Appender:
<ol>
  <li><b>maxfilesize</b>: maximum size for your log file</li>
  <li><b>maxbackupsize</b>: maximum number of files to be kept</li>
</ol>

# How it works?

<ol>
  <li>
    If the size of your log file (ex. logfile.log) exceeds the <b>maxfilesize</b>, the content of the logfile.log will be moved to logfile.log.1. Then, the content of logfile.log will be truncated
  </li>
  <li>
    If the size of logfile.log exceeds for the 2nd time, the content of logfile.log.1 will be moved to logfile.log.2. The content of logfile.log will be moved to logfile.log.1. Same with the 1st point above, the content of logfile.log will be truncated
  </li>
  <li>
    If the number of backup files exceeds the <b>maxbackupsize</b>, the file with the biggest index will be removed. In this case, if maxbackupsize = 3, then logfile.log.2 will be removed
  </li>
</ol>
