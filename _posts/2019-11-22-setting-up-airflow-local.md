---
title: 'Setting Up & Debugging Airflow On Local Machine'
date: 2019-11-22
permalink: /posts/2019/11/setting-up-airflow-local-mode/
tags:
  - airflow
  - workflow management
  - python
---

Airflow is basically a workflow management system. When we’re talking about "workflow", we're referring to a sequence of tasks that needs to be performed to accomplish a certain goal. A simple example would be related to an ordinary ETL job, such as fetching data from data sources, transforming the data into certain formats which in accordance with the requirements, and then storing the transformed data to a data warehouse.

This article provides an explanation on how to set up Airflow in your local machine. You might want to check out the more thorough explanation on the official documentation. The approach used in this article is set in accordance with my own experience.

Since Airflow uses Python as its code base, I think the simplest way to install it is via `pip`.

```
pip install apache-airflow
```

Afterwards, we need to set a work directory for Airflow. This directory simply stores all the DAG definition files, airflow configuration, database for storing all information regarding the workflows, and so forth.

I set the work directory in the `bash_profile`.

```
nano ~/.bash_profile
```

Let’s say the Airflow’s work directory name is <b>airflow_home</b>. Add the following line to the `bash_profile`.

```
export AIRFLOW_HOME=path/to/the/airflow_home
```

Exit the `bash_profile` and reload the profile using `source ~/.bash_profile`.

The <b>airflow_home</b> directory should be empty at first. To initialise the work sub-directories and files within the <b>airflow_home</b> directory, simply just check the installed version of Airflow.

```
airflow version
```

The <b>airflow_home</b> should now contains all the needed work files.

As the initial action, we need to initialise the database used by Airflow to store all information related to workflows management (list all DAGs, DAG Runs, Task Instances, state of the Task Instances, etc.).

```
airflow initdb
```

For the first experiment, let's create a simple DAG. For more information regarding the jargons used in Airflow, I recommend you to visit the official documentation page.

To create a DAG's definition file, create a directory called `dags` in a location specified by `dags_folder` in <b>airflow.cfg</b> configuration file (in <b>airflow_home</b>).

Add a new Python file to the `dags` directory. Let's call it `bash_operator_dag.py`. Here's the content of the Python file.

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta

dag = DAG('bash_op', start_date=datetime(2019, 03, 30), schedule_interval=timedelta(days=1))

t1 = BashOperator(task_id='print_date', bash_command='date', dag=dag)

t2 = BashOperator(task_id='sleep', bash_command='sleep 5', retries=3, dag=dag)

t2.set_upstream(t1)
```  

The above DAG basically does the followings:

<ul>
<li>creates a workflow's DAG named <b>bash_op</b>. The DAG will be executed on the date specified by <b>start_date</b>. The DAG will be re-executed once a day (specified by <b>scheduled_interval</b>)</li>
<li>creates the first operator (<b>t1</b>) that will execute the task of printing current date to console</li>
<li>creates the second operator (<b>t2</b>) that will execute the task of delaying process for 5 secs</li>
<li>creates a task dependency in which <b>t2</b> will be executed upon the successful operation of <b>t1</b></li>
</ul>

Let's schedule the above DAG to be operated.

```
On Terminal A

# start the webserver
airflow webserver

On Terminal B

# start the scheduler
airflow scheduler
```

The first terminal's command is used to start the Airflow's local web server. This web server basically provides a dashboard that lets us know about the existing DAGs, DAG Runs, Task Instances, and so forth.

Meanwhile, the second terminal's command is used to start the scheduler. This scheduler manages when a certain Task should be executed. It also triggers the Operators to execute the Task it implements.

To check the dashboard, just visit `localhost:8080` on your browser.

To start the previous `bash_op` DAG, change the `OFF` option on the second column to `ON`, click the `Trigger Dag` and `Graph View` buttons on the last column.

After the above steps, you should be redirected to a new dashboard's page showing the DAG in graph mode. Just refresh the DAG till all the operators indicate that the tasks have been performed successfully.

In my case, I encountered an odd condition where the tasks were failed. If you see the previous DAG's definition file, both tasks are simple tasks that don't have complicated logics behind. It should definitely be performed successfully.

However, when I checked the operators' log file, nothing helped. Precisely, the log files themselves couldn't be accessed. The same thing also happened when I checked the Task Instance's details. For the sake of clarity, the error message was related closely to `ValueError: unknown locale: UTF-8`.

After a quick browsing, turned out that this problem typically occurs when running Python scripts on MacOS X.

Just add the following lines to the `~/.bash_profile`.

```
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
```

Don't forget to reload the profile with `source ~/.bash_profile`.

Re-open the Terminal used for the Airflow's web server and scheduler and re-start the web server and scheduler afterwards.

Back to the dashboard. Re-trigger the DAG and you should see that both Operators have successfully performed their Tasks. To confirm, just check the log files of both Operators.
