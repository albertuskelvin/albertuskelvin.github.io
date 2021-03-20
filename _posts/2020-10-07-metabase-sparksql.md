---
title: 'Using SparkSQL in Metabase'
date: 2020-10-07
permalink: /posts/2020/10/metabase-sparksql/
tags:
  - spark sql
  - metabase
  - hive
  - hadoop
  - mysql
  - sqoop
---

Basically, Metabase’s SparkSQL only allows users to access data in the Hive warehouse. In other words, the data must be in Hive table format to be able to be loaded.

So, there would be two possible cases:
- The needed data already resides within the Hive warehouse
- There’s a need for some additional data that resides outside the Hive warehouse e.g. in MySQL and PostgreSQL

Before we look at how to use SparkSQL for each case, let’s do a simple setup for our Hive environment.

# Setting Up Hive Tables

For simplicity, we’re going to use docker for this task.

<b>Step 1.</b> Clone this <a href="https://github.com/big-data-europe/docker-hive">repo</a> and within the cloned folder start the containers.

`> docker-compose up -d`

<b>Step 2.</b> Go inside the `docker-hive-master_hive-server_1` container.

`> docker exec -it docker-hive-master_hive-server_1 bash`

<b>Step 3.</b> Connect to the Hive server via beeline client.

`> /opt/hive/bin/beeline`

`beeline> !connect jdbc:hive2://localhost:10000`

<b>Step 4.</b> By default, there are no username & password. Simply press enter and let’s create a sample database called `metabase_db`.

`CREATE DATABASE metabase_db;`

<b>Step 5.</b> As an example, we’re going to use a sample data from titanic.csv. This CSV data will then be loaded into a Hive table. Please download titanic.csv if you don’t have it already.

<b>Step 6.</b> Basically, we may load CSV data that resides within the local file of the container or HDFS. For simplicity, this example will load CSV data located in the local file of the container.

<b>Step 7.</b> Since our Hadoop environment is set up via docker, we need to transfer the titanic.csv data into the `docker-hive-master_hive-server_1` container.

`> docker cp <path_to_csv_in_host> <container_id>:<path_to_csv_in_container>`

<b>Step 8.</b> Create a sample Hive table called `titanic_hive`.

```
CREATE TABLE IF NOT EXISTS titanic_hive (
Survived int, Pclass int, Name string, Sex string, Age int, SiblingsSpousesAboard int, ParentsChildrenAboard int, Fare double) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ‘,’ 
LINES TERMINATED BY ‘\n’ 
STORED AS TEXTFILE 
tblproperties("skip.header.line.count”=“1”);
```

<b>Step 9.</b> Load the CSV data to the created Hive table.

`LOAD DATA LOCAL INPATH <path_to_csv_in_container> OVERWRITE INTO TABLE titanic_hive;`

# Case 1. The data is already within the Hive warehouse

<b>Step 1.</b> Access the Hive tables via Metabase’s SparkSQL. Go to the Admin page and select `Add Database`. Please make sure that the Hive server is started.

<b>Database:</b> metabase_db<br/>
<b>Host:</b> localhost<br/>
<b>IP:</b> 10000<br/>
<b>Username:</b> please fill in with free text since our server doesn’t require username & pass<br/>
<b>Pass:</b> please fill in with free text since our server doesn’t require username & pass<br/>

<b>Step 2.</b> The `metabase_db` should have been loaded. Click it to view all the stored tables including our `titanic_hive table`.
	
# Case 2. There’s a need for some additional data that resides outside the Hive warehouse

Suppose that we’d like to visualize data from outside the Hive warehouse e.g. the data resides within MySQL RDBMS.

To do so, we’d need to transfer the MySQL tables to the Hive warehouse. We could just perform the following steps:
- Export all the MySQL tables
- Manually create the Hive table for each MySQL table
- Manually load the exported MySQL tables into the created Hive tables

Since there are manual works for the above approach, it seems to work just fine when the number of tables and the number of columns are relatively small. However, there might be a problem of productivity when the number is relatively large.

One of the alternative ways is by utilizing sqoop. It’s basically a tool for transferring data from RDBMS to HDFS and from HDFS to RDBMS. With sqoop, we don’t need to perform manual tasks of creating Hive tables and loading the exported MySQL tables into the Hive tables.

In this example, we’re going to use MySQL as the RDBMS where our additional data resides.

<b>Step 1.</b> Download sqoop.

<b>Step 2.</b> Download the MySQL connector and place it within `$SQOOP_HOME/lib`.

<b>Step 3.</b> Move sqoop to our container.

`> docker cp <path_to_sqoop_home_in_host> <container_id>:<path_to_sqoop_home_in_container>`

<b>Step 4.</b> Suppose that the additional data resides in a MySQL database called `additional_db` and in a specific MySQL table called `titanic_additional`.

<b>Step 5.</b> Import the additional data and create a Hive table called `titanic_hive_additional` for it.

```
./$SQOOP_HOME/bin/sqoop import 
--connect jdbc:mysql://host.docker.internal:3306/additional_db?serverTimezone=UTC 
--username <username_for_mysql_server> 
--password <password_for_mysql_server> 
--table titanic_additional 
--hive-import 
--hive-database metabase_db
--create-hive-table 
--hive-table titanic_hive_additional 
-m 1
```

NB: Please see `./$SQOOP_HOME/bin/sqoop import --help` for more information on `import` arguments

<b>Step 6.</b> In case you found an error message related to `HIVE_CONF_DIR` or similar to `"HiveConf class is not found"`, just download `hive-common-3.1.2.jar` and place it to `$SQOOP_HOME/lib` within the container.

<b>Step 7.</b> Just FYI, the imported MySQL table will be stored temporarily in HDFS e.g. `/user/root/` before being loaded into the Hive table.

<b>Step 7.</b> Access the Hive tables via Metabase’s SparkSQL. Go to the Admin page and select `Add Database`. Please make sure that the Hive server is started.

<b>Database:</b> metabase_db<br/>
<b>Host:</b> localhost<br/>
<b>IP:</b> 10000<br/>
<b>Username:</b> please fill in with free text since our server doesn’t require username & pass<br/>
<b>Pass:</b> please fill in with free text since our server doesn’t require username & pass<br/>

<b>Step 8.</b> The `metabase_db` should have been loaded. Click it to view all the stored tables including our `titanic_hive` and `titanic_hive_additional` tables.
