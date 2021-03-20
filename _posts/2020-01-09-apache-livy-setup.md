---
title: 'Submitting and Polling Spark Job Status with Apache Livy'
date: 2020-01-09
permalink: /posts/2020/01/apache-livy-setup-sessions-batches/
tags:
  - livy
  - spark
  - sessions
  - batches
---

Livy offers a REST interface that is used to interact with Spark cluster. It provides two general approaches for job submission and monitoring.

<ul>
<li><b>Session / interactive mode:</b> creates a REPL session that can be used for Spark codes execution. Think of it like when using <I>spark shell</I> to retrieve the computation results interactively</li>
<li><b>Batch mode:</b> submits the job to a Spark cluster. Think of it like when submitting a Spark job file to cluster for execution. This mode obviously does not support interactive session</li>
</ul>

In this post, we're going to look at how to set up Livy and leverage some of its features in <b>local mode</b>.

<h2>Setting up</h2>

Setting up Livy is pretty straightforward. You can go to its official <a href="https://livy.incubator.apache.org/">site</a> to download the zip file.

After extracting the zip file, you need to set few variables in order for Livy to operate. The first one is `SPARK_HOME` which refers to your Spark's installation directory. Since we're using local mode for this post, let's set the value via environment variables. I'm using Mac OS here, please adjust accordingly.

```
> nano ~/.bash_profile

Add the following line:
export SPARK_HOME = path/to/spark/installation/directory

> source ~/.bash_profile
```

The next step would be creating a directory for Livy's logs. By default, Livy outputs its logs into `$LIVY_HOME/logs` directory. You might have to create the `logs` folder manually in case it doesn't exist.

This configuration should be enough for our case. Just FYI, if you want to use standalone mode, you will need to specify the master address via the configuration file. Here's how to do it.

```
# go to the conf directory
> cd $LIVY_HOME/conf
> nano livy.conf.template

# fill in the following variable
livy.spark.master = <master_address>

# save the conf file and rename it to livy.conf
```

Go back to our main point.

Having all the configs set up, let's start the Livy server first.

```
.$LIVY_HOME/bin/livy-server start
```

Go to `host_address:8998` to check out the Livy UI.

<h2>Sessions</h2>

To experiment with <b>Session</b>, we are going to use the following code.

```python
from pyspark.sql import functions as F

df = spark.createDataFrame([('row_p_a', 'row_p_b'), ('row_q_a', 'row_q_b'), ('row_r_a', 'row_r_b')],['col_a','col_b'])

print('Rows: {}'.format(df.count()))

df = df.filter(F.col('col_a').contains('p'))
df.show()
```

Note that we don't need to import and create the spark session as it's already provided as `spark` by the spark shell.

First thing to do is to create a session to execute the above code. We do it by sending a POST request to the Livy server. For more examples, please visit the <a href="https://livy.incubator.apache.org/docs/latest/rest-api.html">documentation</a>.

```
curl -X POST -d '{"kind": "pyspark"}' -H "Content-Type: application/json" <host_address>:8998/sessions
```

The above simply shows that we send a POST request to the Livy server run on `<host_address>:8998`. The request data is in the form of JSON which tells the server that our code is in python. In case you use Scala or R, just set it as `spark` or `sparkr` respectively.

Go back to the UI and you should see there's a session with its ID. To be able to run our code, we need to wait until the status of the session returns `idle`.

Presuming that the session is available to use, let's submit our code.

```
curl -X POST -d '{"code": "from pyspark.sql import functions as F\n df = spark.createDataFrame([('row_p_a', 'row_p_b'), ('row_q_a', 'row_q_b'), ('row_r_a', 'row_r_b')],['col_a','col_b’])\n print('Rows: {}'.format(df.count()))\n df = df.filter(F.col('col_a').contains(‘p’))\n df.show()"}' -H "Content-Type: application/json" <host_address>:8998/sessions/{sessionID}/statements
```

Just fill the `{sessionID}` with the session ID on which the code will be run. The above request should return a response stating the information regarding the job submission and execution, such as statement ID, code, state, and output.

To retrieve the status of the submitted Spark job, we can use the following command.

```
# {statementID} refers to the Spark job ID
curl <host_address>:8998/sessions/{sessionID}/statements/{statementID}
```

The above should return the following information: statement ID, code, state, and output.

In addition, you can use the same session for multiple job submissions.

<h2>Batches</h2>

This mode offers you to submit a job file to be executed. Actually you can think of this mode like when submitting a job via `spark-submit` directly. In other words, we still need to specify the `spark-submit` parameters, such as python files, spark configuration properties, driver memory, application parameters, and so on.

Let’s use the following code as the example.

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = SparkSession.builder.getOrCreate()

df = spark.createDataFrame([('row_p_a', 'row_p_b'), ('row_q_a', 'row_q_b'), ('row_r_a', 'row_r_b')],['col_a','col_b'])

print('Rows: {}'.format(df.count()))

df = df.filter(F.col('col_a').contains('p'))
df.show()
```

Note that in this mode we need to create the spark session by ourselves since this mode treats the job submission similarly to the way of submission when using `spark-submit` directly.

Use the following command to submit the above code to the Livy server.

```
curl -X POST -d '{"file": path/to/job/file}' -H "Content-Type: application/json" <host_address>:8998/batches
```

The above instruction will create a new batch every time it's executed. You can see the batch logs for the information related to submission and execution.

To retrieve the batch status, use the following command.

```
# {batchID} refers to the submitted batch whose status will be returned
curl <host_address>:8998/batches/{batchID}
```

The above command should return several information about the submitted batch, such as the batch ID, application ID, application info, log lines, and batch state.

To retrieve the batch state only, use the below command.

```
curl <host_address>:8998/batches/{batchID}/state
```

The above will return information about the batch ID and batch state.

---

Hope it helps.

Thank you for reading.
