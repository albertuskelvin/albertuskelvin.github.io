---
title: 'Airflow Executor & Friends: How Actually Does the Executor Run the Task?'
date: 2019-11-29
permalink: /posts/2019/11/how-airflow-executes-tasks/
tags:
  - airflow
  - workflow management
  - scheduler
  - executor
  - task instance
---

A few days ago I did a small experiment with Airflow. To be precise, scheduling Airflow to run a Spark job via `spark-submit` to a standalone cluster. I have actually mentioned briefly about how to create a DAG and Operators in the previous <a href="https://albertuskelvin.github.io/posts/2019/11/setting-up-airflow-local-mode/">post</a>.

Here's a brief scenario used in the experiment.

<ul>
<li>Submit a PySpark Job to a standalone cluster with <b>client</b> deploy mode</li>
<li>Submit a PySpark Job to a standalone cluster with <b>cluster</b> deploy mode</li>
</ul>

Setting up a Spark standalone cluster in your machine is simple. You can find how to do that in the <a href="https://spark.apache.org/docs/2.2.0/spark-standalone.html">documentation</a>. If you want to check the Spark master's dashboard (served in port 8080 by default), you should use different port as Airflow webserver already uses that port. Just add the parameter `--webui-port <port>` after `./sbin/start-master.sh`. 

If you noticed, currently Spark job in Python doesn't support <b>cluster</b> deploy mode. Passing this mode as a `spark-submit` parameter will result in below exception.

```
org.apache.spark.SparkException: Cluster deploy mode is currently not supported for python applications on standalone clusters.
```

According to above fact, the second experiment should be a failed task.

After completely running both tasks, the following notification was returned for both tasks.

```
INFO - Executor reports execution of <DAG_ID>.<Task_ID> execution_date=<execution_date> exited with status success for try_number <try_num>
```

What made me confused was that the fact that the execution of both tasks returned <b>success</b> status. What differred both of them was that the first experiment has <b>SUCCESS</b> status for the `TaskInstance`, while the second experiment has <b>UP_FOR_RETRY</b> status. This <b>UP_FOR_RETRY</b> requests the `Scheduler` for another retry for this second experiment.

Even though the status of `TaskInstance` for both experiments are <b>different</b>, the `Executor` reports the <b>same</b> status --that is <b>SUCCESS</b>-- for both experiments.

So I wondered, what did this report by the `Executor` mean?

I decided to find out the answer by performing the most boring and challenging task in software engineering - reading the code base.

You can find the Airflow's code base <a href="https://github.com/apache/airflow">here</a>.

I'm going to make this post brief actually. Here's what I found from my quick investigation.

<b>NB:</b> I used <b>Sequential Executor</b> when performing this investigation.

<h1>How Airflow Executes a Task?</h1>

<h2>(A) The Scheduler Talks to the Executor</h2>

The `Scheduler` basically does the following after finding out what `Tasks` can be executed.
<ul>
  <li>Sends out the information regarding the task to the <b>Executor</b> (DAG ID, Task ID, execution date, try number, priority for execution, and queue mode)</li>
<li>Generates the command to run the task</li>
<li>Specifies the priority to run the task & queue mode</li>
  <li>Asks the <b>Executor</b> to add the generated command, task priority, queue mode, and simple task instance to a queue (let's call this as <b>TaskInfo</b>)</li>
</ul>

You can find an example of the above work by inspecting the `Scheduler` log.

```
{scheduler_job.py:1148} INFO - Sending (DAG_ID, Task_ID, execution_date, try_number) to executor with priority <priority> and queue <queue_mode>
```

You can read the code <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/jobs/scheduler_job.py#L1072">here</a>.

<h2>(B) The Executor Performs the Scheduler's Request</h2>

The `Executor` adds the <b>Task Info</b> from the `Scheduler` to a queue.

You can find the above work on the `Scheduler`'s log.

```
{base_executor.py:59} INFO - Adding to queue: ['airflow', 'run', DAG_ID, Task_ID, execution_date, '--local', '--pool', 'default_pool', '-sd', path/to/dag/file]
```

You can find the method used by the `Executor` to add the <b>Task Info</b> to the queue <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/executors/base_executor.py#L54">here</a>.

<h2>(C) The Scheduler Asks the Executor to Run the Task</h2>

The `Scheduler` sends heartbeat to the `Executor`. You can find the code <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/executors/base_executor.py#L111">here</a>.

The heartbeat makes the `Executor` triggered to run the task. Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/executors/base_executor.py#L135">code</a>.

Before running the task, the `Executor` sorts the queued <b>Task Info</b> according to the task priority in descending order. I think bigger value should denote more priority in this case. Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/executors/base_executor.py#L142">code</a>.

The `Executor` adds all the queued <b>Task Info</b> to a list. However, based on the source code, only key & command to run the task are added. The key should has the following form: <i>dag_id, task_id, execution_date, try_number</i>. Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/executors/base_executor.py#L146">code</a>.

<h2>(D) The Executor Starts to Run the Task</h2>

The `Executor` calls the <b>sync</b> method to execute the command used to run the task. Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/executors/sequential_executor.py#L42">code</a>.

The `Scheduler` waits for notification from the `Executor` regarding whether the task has finished.

<h2>(E) The Executor Asks Other Layers For Help</h2>

The `Executor` executes the task by calling the `Local Task Job`.

The `Local Task Job` retrieves the `Task Runner` specified in the configuration file (airflow.cfg). Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/jobs/local_task_job.py#L70">code</a>. Below is the example of the configuration parameter.

```
# The class to use for running task instances in a subprocess
task_runner = StandardTaskRunner
```

When the `Task Runner` is retrieved, the parent class `__init__` method is executed. This `__init__` creates another command used to run the task. This command is made to be able to run in different environment. The common use case for this might be when we use distributed workers (?). <b>CMIIW</b>. Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/task/task_runner/base_task_runner.py#L91">code</a>.

The `StandardTaskRunner` inherits `BaseTaskRunner`.

The `Local Task Job` starts the `Task Runner`. Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/jobs/local_task_job.py#L91">code</a>.

<h2>(F) The Task Runner Executes the Operator</h2>

The `Task Runner` finally executes the command used to run the task (`full_cmd`). Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/task/task_runner/base_task_runner.py#L112">code</a>.

The `full_cmd` is used to execute the `run()` method in <b>TaskInstance.py</b>. Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/models/taskinstance.py#L975">code</a>.

The `run()` method checks the dependencies (<a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/models/taskinstance.py#L722">code</a>).

If the dependencies are met, then execute the `Operator` via its `execute()` method. Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/models/taskinstance.py#L914">code</a>.

<h2>(G) The Operator in Action</h2>

The `Operator` executes the task implementation. Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/operators/bash_operator.py#L116">code</a>.

After the `Operator` has finished (`Local Task Job` got the return code from the `Task Runner`), the `Task Runner` is then terminated.

<h2>(H) The Local Task Job Reports Back to the Executor</h2>

The `Executor` changes the state of the `Task Instance`. You can find it by inspecting the `Scheduler` log.

```
Changing state: key = (DAG_ID, Task_ID, execution_datetime, try_number)
```

Please refer to the <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/executors/sequential_executor.py#L46">code</a>.

---

<h1>The Conclusion</h1>

Whether the `Task Instance` was successful or not, if there's <b>no</b> `subprocess.CalledProcessError` exception, the STATE reported by the `Executor` to the `Scheduler` will be <b>SUCCESS</b>.

According to the source code (<a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/executors/sequential_executor.py#L49">here</a>), `CalledProcessError` exception means that the task was <b>failed to execute till finish</b>. This <b>does not relate to the return code</b> returned by the `Task Instance`.

The `Executor` does not consider the return code of the `Task Instance` because there was no process of evaluating the return code during the workflow (from <b>A</b> to <b>H</b>). Although the return code is retrieved as the sign to terminate the `Local Task Job` (<a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/jobs/local_task_job.py#L95">code</a>), there's no any further process for the return code, such as reporting it back to the `Executor`. Therefore, this should conclude that the `Executor` does not receive any information regarding the return code of the `Task Instance`.

In my opinion, since the termination of the `Local Task Job` denotes that the `Task Instance` <b>has finished</b> (got the return code), the `Executor` presumes that the `Task Instance` has completed already. It does not care about the completion status of the `Task Instance`. What it cares is that the `Task Instance` has been executed by all the layers below the `Executor`, such as the `Local Task Job`, `Task Runner`, and `Operator`. There's <b>no any exception returned by any layer</b> meaning that the task has been performed smoothly. That's why the status reported to the `Scheduler` is <b>success</b>.

Last but not least, an example of case when `CalledProcessError` exception happens can be found <a href="https://github.com/apache/airflow/blob/e82008d96c8aa78a08fd5f832615feed427d8ea6/airflow/jobs/local_task_job.py#L104">here</a>.

---

Thank you for reading.

I really appreciated any feedback.
