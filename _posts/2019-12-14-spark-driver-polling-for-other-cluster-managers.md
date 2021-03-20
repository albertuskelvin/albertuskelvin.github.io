---
title: 'Airflow Feature Improvement: Spark Driver Status Polling Support for YARN, Mesos & K8S'
date: 2019-12-14
permalink: /posts/2019/12/airflow-spark-submit-hook-for-other-cluster-managers/
tags:
  - airflow
  - spark
  - cluster managers
---

According to the <a href="https://github.com/apache/airflow/blob/master/airflow/contrib/hooks/spark_submit_hook.py#L162">code base</a>, the driver status tracking feature is only implemented for standalone cluster manager. However, based on this <a href="https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L543">reference</a>, we could also poll the driver status for mesos and kubernetes (cluster deploy mode). Additionally, such a feature is also possible for YARN.

I decided to raise this future improvement on JIRA. Here's how the probable improvement.

There are two primary modules that need to be considered.

The first one is originally coded like <a href="https://github.com/apache/airflow/blob/master/airflow/contrib/hooks/spark_submit_hook.py#L162">this</a>. The probable modificiation would be like the following.

```python
def _resolve_should_track_driver_status(self):
        """
        Determines whether or not this hook should poll the spark driver status through
        subsequent spark-submit status requests after the initial spark-submit request
        :return: if the driver status should be tracked
        """
        if self._connection['deploy_mode'] == 'cluster':
              cluster_manager_schemes = ('spark://', 'mesos://', 'k8s://https://', 'yarn')
              if self._connection['master'].startswith(cluster_manager_schemes):
                    return True
```

The second one is originally coded like <a href="https://github.com/apache/airflow/blob/master/airflow/contrib/hooks/spark_submit_hook.py#L309">this</a>. The possible modification would be like the following.

```python
def _build_track_driver_status_command(self):
        # The driver id so we can poll for its status
        if not self._driver_id:
            raise AirflowException(
                "Invalid status: attempted to poll driver " +
                "status but no driver id is known. Giving up.")

        cluster_manager_schemes = ("spark://", "mesos://", "k8s://https://")
        if self._connection['master'].startswith(cluster_manager_schemes): 
            # standalone, mesos, kubernetes
            connection_cmd = self._get_spark_binary_path()
            connection_cmd += ["--master", self._connection['master']]
            connection_cmd += ["--status", self._driver_id]
        else:
            # yarn
            connection_cmd = ["yarn application -status"]
            connection_cmd += [self._driver_id]

        self.log.debug("Poll driver status cmd: %s", connection_cmd)

        return connection_cmd
```

Have any thought or different opinion? Really appreciate it.

Thank you for reading.
