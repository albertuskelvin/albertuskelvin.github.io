---
title: 'Data Quality with Apache Griffin Overview'
date: 2020-05-13
permalink: /posts/2020/05/apache-griffin-high-level-overview/
tags:
  - data quality
  - griffin
  - machine learning
---

A few days back I was exploring a big data quality tool called Griffin. There are lots of DQ tools out there, such as Deequ, Target’s data validator, Tensorflow data validator, PySpark Owl, and Great Expectation. There’s another one called Cerberus. It doesn’t natively support large-scale data however.

This post is going to highlight Griffin only.

If you browse on the internet, Griffin was originally built at eBay and now has been donated as an Apache project. Please visit the Github <a href="https://github.com/apache/griffin">repo</a> for more details.

On its repo, we can see that Griffin has provided several docker images already. It should be a quick start to experiment with this tool. The images are `apachegriffin/griffin_spark2`, `apachegriffin/kafka`, and `apachegriffin/elasticseach`. In my case, I only needed the first one since it already covers the core components to run Griffin for batch validation. Another thing is that the Kafka image is purposed for streaming validation.

Just create the container from `apachegriffin/griffin_spark2`.

```
docker run -it <IMAGE_ID>
```

We should be in the working directory (root) and few interest sub-directories are available, such as `data`, `measure`, and `json`. To put it simply, here are the primary objectives of each sub-directory.

```
data: stores the local datasets
json: store the local configuration (environment & DQ job) files
measure: stores the griffin jar and scripts for spark-submit
```

We can also store the data and configurations in HDFS. To search for the data in HDFS, just use the following command.

```
hdfs dfs -ls -R /griffin 
```

The above should return the `griffin` directory along with its sub-directories.

Let’s take a look at the `measure` directory. There exists the Griffin JAR file and several scripts for spark job submission. Take a look at one of the scripts.

```
cat batch-accu.sh
```

It should return the following.

```
spark-submit --class org.apache.griffin.measure.Application --master yarn --deploy-mode client --queue default \
--driver-memory 1g --executor-memory 1g --num-executors 3 \
--conf spark.yarn.executor.memoryOverhead=512 \
griffin-measure.jar \
hdfs:///griffin/json/env.json ~/json/batch-accu-config.json
```

After adjusting the `env.json` and `batch-accu-config.json`, we can execute Griffin just by running the above script (or custom one according to your needs). To run the script, simply use the following command.

```
./batch-accu.sh
```

If anything goes well, the computation logs should be outputted in the Terminal and the metrics result should be available in the specified sinks.

## The Configs

We have seen how to run the Griffin measure module. Turns out it’s pretty simple and what we just need to do is adjust the configuration files.

There are two primary configurations in Griffin. The first one is for the environment configuration, while the second one is for the data quality job configuration.

### Environment Configuration

Basically, this config is used to specify all things related to the parameters of the Griffin’s components. Several examples, such as the spark configurations, sink locations, and zookeeper configurations.

Below is the simple example of this configuration.

```
{
  "spark": {
    "log.level": "WARN",
    "checkpoint.dir": "hdfs:///griffin/streaming/cp",
    "batch.interval": "1m",
    "process.interval": "5m",
    "config": {
      "spark.default.parallelism": 5,
      "spark.task.maxFailures": 5,
      "spark.streaming.kafkaMaxRatePerPartition": 1000,
      "spark.streaming.concurrentJobs": 4,
      "spark.yarn.maxAppAttempts": 5,
      "spark.yarn.am.attemptFailuresValidityInterval": "1h",
      "spark.yarn.max.executor.failures": 120,
      "spark.yarn.executor.failuresValidityInterval": "1h",
      "spark.hadoop.fs.hdfs.impl.disable.cache": true
    }
  },

  "sinks": [
    {
      "type": "console",
      "config": {
        "max.log.lines": 100
      }
    }, {
      "type": "hdfs",
      "config": {
        "path": "hdfs:///griffin/streaming/persist",
        "max.lines.per.file": 10000
      }
    }
  ],

  "griffin.checkpoint": [
    {
      "type": "zk",
      "config": {
        "hosts": "<zookeeper host ip>:2181",
        "namespace": "griffin/infocache",
        "lock.path": "lock",
        "mode": "persist",
        "init.clear": true,
        "close.clear": false
      }
    }
  ]
}
```

### Data Quality Configuration

This is the primary medium in which we specify all the data validation related things. Several of them including the validation mode (batch or streaming), data sources, constraints evaluation, and selected sink locations.

Below is the simple example of this configuration.

```
{
  "name": "accu_batch",
  "process.type": "BATCH",
  "data.sources": [
    {
      "name": "src",
      "connector": {
        "type": "file",
        "config": {
          "format": "parquet",
          "paths": []
        }
      }
    },
    {
      "name": "tgt",
      "connector": {
        "type": "file",
        "config": {
          "format": "parquet",
          "paths": []
        }
      }
    }
  ],
  "evaluate.rule": {
    "rules": [
      {
        "dsl.type": "griffin-dsl",
        "dq.type": "ACCURACY",
        "out.dataframe.name": "accu",
        "rule": "source.user_id = target.user_id AND upper(source.first_name) = upper(target.first_name) AND source.last_name = target.last_name AND source.address = target.address AND source.email = target.email AND source.phone = target.phone AND source.post_code = target.post_code",
        "details": {
          "source": "source",
          "target": "target",
          "miss": "miss_count",
          "total": "total_count",
          "matched": "matched_count"
        },
        "out": [
          {
            "type": "metric",
            "name": "accu"
          },
          {
            "type": "record"
          }
        ]
      }
    ]
  },
  "sinks": [
    "CONSOLE",
    "HDFS"
  ]
}
```

## Notable Points

The above only tells you how to use Griffin using its pre-built jar file in a dockerized environment. This also means that all the required components are already properly set up.

However, I observed some notable pluses & minuses when investigating this tool. To make this article short, I’ll continue this point in the next post.
