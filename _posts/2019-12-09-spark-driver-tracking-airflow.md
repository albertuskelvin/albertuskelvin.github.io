---
title: 'Bug on Airflow When Polling Spark Job Status Deployed with Cluster Mode'
date: 2019-12-09
permalink: /posts/2019/12/airflow-tracks-spark-driver-status-cluster-deployment/
tags:
  - airflow
  - spark
  - cluster
  - driver status
  - status tracking
  - scheduling
---

I was thinking of the following case.

Suppose we schedule Airflow to submit a Spark job to a cluster. We use cluster deploy mode meaning that the driver program lives in one of the cluster machines. Since we haven't defined any communication mechanism between the Airflow machine and Spark cluster, how the scheduler knows whether it should re-submit the job in case the job fails?

Well, the conventional approach where a `BashOperator` is defined to execute a `spark-submit` command won't work. In this case, the concern would be on the communication between two different clusters, one for Airflow and the other one for Spark.

Turns out that there's a dedicated operator for such a case, that is `SparkSubmitOperator`. Please refer to its <a href="https://airflow.readthedocs.io/en/stable/_modules/airflow/contrib/operators/spark_submit_operator.html">code</a>.

This operator accepts several parameters needed for `spark-submit` execution, such as the job file, address of spark master, location of `spark-submit`, arguments of `spark-submit` (<I>py-files</I>, <I>archives</I>, <I>files</I>, etc.), and so forth.

When the operator is executed, it calls a hook called `SparkSubmitHook`. This hook is simply used as a bridge for communication between two different machines. You can find the code for the hook <a href="https://airflow.readthedocs.io/en/stable/_modules/airflow/contrib/hooks/spark_submit_hook.html">here</a>.

This hook initialises several parameters before submitting the job, such as creating connection info (spark master, location of `spark-submit`, location of spark installation, deploy mode, etc.). Please refer to the <a href="https://github.com/apache/airflow/blob/master/airflow/contrib/hooks/spark_submit_hook.py#L171">code</a> for more details.

One thing to note here is that the `_conn_id` attribute (<a href="https://github.com/apache/airflow/blob/master/airflow/contrib/hooks/spark_submit_hook.py#L183">here</a>) will be used to denote the location of spark master. As you can see from the code, the default value for this attribute is <I>spark-default</I>. When you investigate the `get_connection` method (<a href="https://github.com/apache/airflow/blob/master/airflow/hooks/base_hook.py#L79">here</a>), you can see that this `_conn_id` is used as part of the connection URI (<a href="https://github.com/apache/airflow/blob/master/airflow/hooks/base_hook.py#L56">here</a>). Therefore, if you use a standalone cluster, then the value of the connection URI should be similar to `spark://host:port`.

<b>Now, let's get into the problem.</b>

I decided to give this `SparkSubmitOperator` a try using standalone cluster first. Only to check whether the initial setup was proper.

Unfortunately, the `spark-submit` task was failed. The following exception occurred.

```
airflow.exceptions.AirflowException: Cannot execute: [path/to/spark-submit, '--master', host:port, job_file.py]
```

The first thing that came up into my mind was why the master address excluded the `spark://` prefix. So it should be like `--master spark://host:port`.

I then decided to run the `spark-submit` via Terminal as usual (without Airflow). The command used was the same as the following.

```
path/to/spark-submit --master host:port job_file.py
```

An exception still occurred. The error log seemed to validate my initial hypothesis. It said something like this (might be different).

```
Master should start with 'local', 'spark://', 'mesos://', 'k8s://', or 'yarn’
```

According to the above log, it is definitely clear that the master address (standalone mode in this case) should include the `spark://` (scheme).

The next step should be obvious. I performed a quick check to the source code and found that such a thing hadn’t been handled. Please take a look at the following code snippet (<a href="https://github.com/apache/airflow/blob/master/airflow/contrib/hooks/spark_submit_hook.py#L171">source</a>).

```python
conn = self.get_connection(self._conn_id)
if conn.port:
	conn_data['master'] = "{}:{}".format(conn.host, conn.port)
else:
	conn_data['master'] = conn.host
```

After reviewing the subsequent method callings, it turned out that the driver status tracking feature won't be utilised at all because of the above bug. Look at the following code snippet (<a href="https://github.com/apache/airflow/blob/master/airflow/contrib/hooks/spark_submit_hook.py#L162">source</a>).

```python
def _resolve_should_track_driver_status(self):
	"""
	Determines whether to not this hook should poll the spark driver status through subsequent spark-submit status requests after the initial spark-submit request
	:return: if the driver status should be tracked
	"""
	return ('spark://' in self._connection['master'] and self._connection['deploy_mode'] == 'cluster')
```

The above method will always return <b>False</b> as the spark master's address doesn't start with the scheme, such as `spark://`. The method is used by this <a href="https://github.com/apache/airflow/blob/master/airflow/contrib/hooks/spark_submit_hook.py#L366">part</a> of the job submission.

<h3>Further Investigation</h3>

I investigated the <b>Connection</b> module (<i>airflow.models.connection</i>) further and found that if we provide the URI (ex: spark://host:port), then the attributes of the <b>Connection</b> object will be derived via URI parsing.

When parsing the host (<a href="https://github.com/apache/airflow/blob/master/airflow/models/connection.py#L137">code</a>), the resulting value was only the hostname without the scheme.

Therefore, the `conn.host` in the following code will only store the hostname.

```python
conn = self.get_connection(self._conn_id)
if conn.port:
	conn_data['master'] = "{}:{}".format(conn.host, conn.port)
else:
	conn_data['master'] = conn.host
```

Frankly, I was a bit confused when dove into how the connection was resolved. If you read the connection data retriever (<a href="https://github.com/apache/airflow/blob/master/airflow/hooks/base_hook.py#L79">code</a>), you may find that there are two ways of specifying the connection data. The first one is via database, while the second one is via environment variables.

Unfortunately, specifying the connection data via environment variables is more complicated since we are only allowed to pass the connection URI. The attributes, such as host, port, scheme, and so forth will be derived via parsing the URI. This somewhat makes it harder for several kinds of Spark masters, such as `local`, `yarn`, and `k8s://https://host:port`. Unlike standalone and mesos whose URI's pattern is common (scheme://host:port), parsing these three kinds of URI doesn't work properly since the attributes of `urlparse` will store irrelevant values.

Another point that I want to note is that by storing the connection data in the database, we can somehow engineer the attributes. For instance, the `host` column in the <b>Connection</b> table could store `spark://host` or `spark://host:port`. Although the table already has a dedicated column for the scheme, we're still able to store "irrelevant" patterns to other columns. I came across such an approach on the <a href="https://github.com/apache/airflow/blob/master/tests/contrib/hooks/test_spark_submit_hook.py">unit test module</a>.

Now let's say the `host` column stores `spark://host:port` (standalone), `mesos://host:port` (mesos), `k8s://https://host:port` (kubernetes), `local` (local mode), and `yarn` (yarn cluster). The following code will work as expected.

```python
conn = self.get_connection(self._conn_id)
if conn.port:
	conn_data['master'] = "{}:{}".format(conn.host, conn.port)
else:
	conn_data['master'] = conn.host
```

On the other hand, if we store the data via environment variables, we will have to pass the URI parsing which causes we can't use kubernetes, local, and yarn mode (`conn_data['master']` will store irrelevant value).

Last but not least, if you observe the above code snippet, even though we store the connection data in a database, the connection resolve won't work when the `host` doesn't contain the scheme. In other words, the above code will only work for database mode only when the `host` consists of the scheme.

Therefore, I think the way of resolving the connection to the Spark cluster only works when the connection data is stored in database with an exception that the host includes the scheme. Storing the data as an environment variable will involve URI parsing which might result in irrelevant results.

Since this might be a critical and annoying bug (I'm pretty sure it is), I decided to report it as an issue on JIRA.

<h3>What's the Probable Solution?</h3>

I think we don't really need the whole URI. I mean, when we store the connection data as an environment variable, we could just specify the URI parts in form of JSON. This approach is mainly used to tackle the URI parsing problem.

In this case, the `conn_id` will still be preserved.

Take a look at the following example (`conn_id` = "spark_default"). For simplicity, let's presume that `extra` is in JSON form.

```
AIRFLOW_CONN_SPARK_DEFAULT='{"conn_type": <conn_type>, "host":<host>, "port":<port>, "schema":<schema>, "extra":<extra>}'
```

Then we might want to modify this <a href="https://github.com/apache/airflow/blob/master/airflow/hooks/base_hook.py#L56">part</a> into the following. 

```python
@classmethod
def _get_connection_from_env(cls, conn_id):
	environment_uri = os.environ.get(CONN_ENV_PREFIX + conn_id.upper())
	conn = None
	if environment_uri:
	    obj = json.loads(environment_uri)
	    conn_args = {
	    	'conn_type': obj.get('conn_type', None),
		'host': obj.get('host', None),
		'login': obj.get('login', None),
		'password': obj.get('password', None),
		'schema': obj.get('schema', None),
		'port': obj.get('port', None),
		'extra': obj.get('extra', None)
	    }
	    conn = Connection(conn_id=conn_id, **conn_args)
	    
	return conn
```

And then the <a href="https://github.com/apache/airflow/blob/master/airflow/contrib/hooks/spark_submit_hook.py#L171">code part</a> that calls the above method.

```python
def _resolve_connection(self):
        # Build from connection master or default to yarn if not available
        conn_data = {'master': 'yarn',
                     'queue': None,
                     'deploy_mode': None,
                     'spark_home': None,
                     'spark_binary': self._spark_binary or "spark-submit",
                     'namespace': None}

        try:
            # Master can be local, yarn, spark://HOST:PORT, mesos://HOST:PORT and
            # k8s://https://<HOST>:<PORT>
            conn = self.get_connection(self._conn_id)
	    if conn.conn_type in ['spark', 'mesos']:
	    	# standalone and mesos
	    	conn_data['master'] = "{}://{}:{}".format(conn.conn_type, conn.host, conn.port)
	    elif conn.conn_type == 'k8s':
	    	# kubernetes
	    	conn_data['master'] = "{}://https://{}:{}".format(conn.conn_type, conn.host, conn.port)
	    else:
	    	# local and yarn
	    	conn_data['master'] = conn_type
	    
            # Determine optional yarn queue from the extra field
            extra = conn.extra_dejson
            conn_data['queue'] = extra.get('queue', None)
            conn_data['deploy_mode'] = extra.get('deploy-mode', None)
            conn_data['spark_home'] = extra.get('spark-home', None)
            conn_data['spark_binary'] = self._spark_binary or  \
                extra.get('spark-binary', "spark-submit")
            conn_data['namespace'] = extra.get('namespace')
        except AirflowException:
            self.log.info(
                "Could not load connection string %s, defaulting to %s",
                self._conn_id, conn_data['master']
            )

        return conn_data
```

The primary point is the above. The rest of the code can be adjusted accordingly.

Even though this solution could reduce the false result returned by URI parsing, one need to strictly ensure that each attribute (host, port, scheme, etc.) should store the relevant value. I think it's much easier than creating a correct URI parser. Moreover, applying such a technique makes the whole connection data builder for both database & environment variable mode have the same pattern (both use a structured data specification).

Do you have any thought on this? Or other better solutions? I'd love to hear that.

Thank you for reading.
