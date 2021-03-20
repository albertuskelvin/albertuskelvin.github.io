---
title: 'Permanent and Temporary External Table in BigQuery'
date: 2020-11-17
permalink: /posts/2020/11/create-bigquery-external-tables/
tags:
  - gcp
  - bigquery
  - external table
---

In BigQuery, an external data source is a data source that we can query directly although the data is not stored in BigQuery's storage. We can query the data source just by creating an external table that refers to the data source instead of loading it to BigQuery.

The external table is categorized into permanent and temporary table.

According to the GCP docs, here is the definition for each type of external table.

```
A permanent table is a table that is created in a dataset and is linked to your external data source. 
Because the table is permanent, you can use access controls to share the table with others 
who also have access to the underlying external data source, 
and you can query the table at any time.

When you query an external data source using a temporary table, you submit a command that includes a query and 
creates a non-permanent table linked to the external data source. 
When you use a temporary table, you do not create a table in one of your BigQuery datasets. 
Because the table is not permanently stored in a dataset, it cannot be shared with others. 
Querying an external data source using a temporary table is useful for one-time, 
ad-hoc queries over external data, or for extract, transform, and load (ETL) processes.
```

In this post, specifically, we're going to look at how to create permanent and temporary table to query an external data source located in Cloud Storage.

Note that we'll use bq command-line tool to create the external table.

---

# Permanent external table

We specify the external table's schema using the followings:
- A table definition file (stored on the local machine)
- An inline schema definition
- A JSON schema file (stored on the local machine)

To create a permanent external table by using a <b>table definition file</b>, execute the following command.

```
bq mk \
--external_table_definition=definition_file \
dataset.table
```

- `definition_file`: the path to the table definition file stored on local machine
- `dataset`: the name of the dataset
- `table`: the name of the external table

<b>Example</b>: Let's create an external table called `mytable` within a dataset `mydataset`. We use a table definition located in `/tmp/mytable_def`.

`bq mk --external_table_definition=/tmp/mytable_def mydataset.mytable`

To create a permanent external table by using an <b>inline schema definition</b>, execute the following command.

```
bq mk \
--external_table_definition=schema@source_format=Cloud Storage URI \
dataset.table
```

- `schema`: the schema definition in the format `field:data_type,field:data_type`
- `source_format`: the format of the source data, such as CSV, NEWLINE_DELIMITED_JSON, AVRO, or DATASTORE_BACKUP (DATASTORE_BACKUP is also used for Filestore)
- `Cloud Storage URI`: the location of the source data in Cloud Storage
- `dataset`: the name of the dataset
- `table`: the name of the external table

<b>Example</b>: Let's create an external table called `sales` within a dataset `mydataset`. The table's schema is `Region:STRING,Quarter:STRING,Total_sales:INTEGER`. The source format is CSV.

```
bq mk \
--external_table_definition=Region:STRING,Quarter:STRING,Total_sales:INTEGER@CSV=gs://mybucket/sales.csv \
mydataset.sales
```

To create a permanent external table by using a <b>JSON schema file</b>, execute the following command.

```
bq mk \
--external_table_definition=schema@source_format=Cloud Storage URI \
dataset.table
```

- `schema`: the path to the JSON schema file on local machine.
- `source_format`: the format of the source data, such as CSV, NEWLINE_DELIMITED_JSON, AVRO, or DATASTORE_BACKUP (DATASTORE_BACKUP is also used for Firestore)
- `Cloud Storage URI`: the location of the source data in Cloud Storage
- `dataset`: the name of the dataset
- `table`: the name of the external table

<b>Example</b>: Let's create an external table called `sales` within a dataset `mydataset`. We use a table's JSON schema definition located in `/tmp/sales_schema.json`.

```
bq mk \
--external_table_definition=/tmp/sales_schema.json@CSV=gs://mybucket/sales.csv \
mydataset.sales
```

After the permanent table is created, we can run a query against the table as if it were a native BigQuery table.

---

# Temporary external table

We specify the external table's schema using the followings:
- A table definition file (stored on the local machine)
- An inline schema definition
- A JSON schema file (stored on the local machine)

To query a temporary external table by using a <b>table definition file</b>, execute the following command.

```
bq --location=location query \
--external_table_definition=table::definition_file \
'query'
```

- `location`: the name of your location (optional). For example, if you are using BigQuery in the Tokyo region, you can set the flag's value to `asia-northeast1`
- `table`: the name of the temporary external table
- `definition_file`: the path to the table definition file on local machine
- `query`: the query executed againts the temporary external table

<b>Example</b>: Create a temporary external table called `sales` with a table definition file located on `sales_def` on local machine. In addition, we execute a query `SELECT Region, Total_sales FROM sales` againts the external table.

```
bq query \
--external_table_definition=sales::sales_def \
'SELECT
  Region,
  Total_sales
FROM
  sales'
```

To query a temporary external table by using an <b>inline schema definition</b>, execute the following command.

```
bq --location=location query \
--external_table_definition=table::schema@source_format=Cloud Storage URI \
'query'
```

- `location`: the name of your location (optional). For example, if you are using BigQuery in the Tokyo region, you can set the flag's value to `asia-northeast1`
- `table`: the name of the temporary external table
- `schema`: the schema definition in the format `field:data_type,field:data_type`
- `source_format`: the format of source data, such as CSV, NEWLINE_DELIMITED_JSON, AVRO, or DATASTORE_BACKUP (DATASTORE_BACKUP is also used for Firestore)
- `Cloud Storage URI`: the location of the source data in Cloud Storage
- `query`: the query executed againts the temporary external table

<b>Example</b>: Create a temporary external table called `sales` with schema `Region:STRING,Quarter:STRING,Total_sales:INTEGER`. The source data format is CSV located in `gs://mybucket/sales.csv`. In addition, we execute a query `SELECT Region, Total_sales FROM sales` againts the external table.

```
bq query \
--external_table_definition=sales::Region:STRING,Quarter:STRING,Total_sales:INTEGER@CSV=gs://mybucket/sales.csv \
'SELECT
  Region,
  Total_sales
FROM
  sales'
```

To query a temporary external table by using a <b>JSON schema file</b>, execute the following command.

```
bq --location=location query \
--external_table_definition=schema_file@source_format=Cloud Storage URI \
'query'
```

- `location`: the name of your location (optional). For example, if you are using BigQuery in the Tokyo region, you can set the flag's value to `asia-northeast1`
- `schema_file`: the path to the JSON schema file on local machine
- `source_format`: the format of source data, such as CSV, NEWLINE_DELIMITED_JSON, AVRO, or DATASTORE_BACKUP (DATASTORE_BACKUP is also used for Firestore)
- `Cloud Storage URI`: the location of the source data in Cloud Storage
- `query`: the query executed againts the temporary external table

<b>Example</b>: Create a temporary external table called `sales` with schema specified in a JSON file (`/tmp/sales_schema.json`). The source's format is CSV and is located in `gs://mybucket/sales.csv`. In addition, we execute a query `SELECT Region, Total_sales FROM sales` againts the external table.

```
bq query
--external_table_definition=sales::/tmp/sales_schema.json@CSV=gs://mybucket/sales.csv
'SELECT Region, Total_sales FROM sales'
```
