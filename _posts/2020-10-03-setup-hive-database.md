---
title: 'Setting Up Database in Hive Environment'
date: 2020-10-03
permalink: /posts/2020/10/setup-hive-database/
tags:
  - hive
  - database
  - hadoop
---

In this post, we're going to look at how to set up a database along with the tables in Hive.

I'll use docker to set up the Hive environment.

<b>Step 1.</b> Go to the following <a href="https://github.com/big-data-europe/docker-hive">repo</a> and clone the repo.

<b>Step 2.</b> Run `docker-compose up -d` in the same directory as the location of `docker-compose.yml`.

<b>Step 3.</b> Create a bash session in the `hive-server` container with `docker exec -it hive-server bash`.

<b>Step 4.</b> Connect to the Hive server with `/opt/hive/bin/beeline -u jdbc:hive2://localhost:10000`.

<b>Step 5.</b> By default, Hive server won't ask for the username & password.

<b>Step 6.</b> Let's create a database called `example_db`. To do so, use `CREATE DATABASE example_db`.

<b>Step 7.</b> Use the newly created database with `USE example_db`.

<b>Step 8.</b> Now let's create a table called `example_table`. To do so, use the following command.

```
CREATE TABLE IF NOT EXISTS example_table (ColA int, ColB int, ColC double) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n' 
STORED AS TEXTFILE 
tblproperties("skip.header.line.count”=“1”);
```

<b>Step 9.</b> Load your data into the `example_table`. To do so, ensure that your data has been moved to the docker container.

```
# copy the data from host to docker container (execute the command outside the container)
docker cp <file_path_in_host> container_id:<file_path_in_container>

# load the data to the hive table
LOAD DATA LOCAL INPATH '<path_to_your_data_in_container>' OVERWRITE INTO TABLE example_table;
```

<b>Step 10.</b> The data has been loaded to the hive table.
