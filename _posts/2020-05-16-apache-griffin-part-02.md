---
title: 'Apache Griffin for Data Validation: Yay & Nay'
date: 2020-05-16
permalink: /posts/2020/05/apache-griffin-pros-cons/
tags:
  - data quality
  - apache griffin
  - machine learning
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/05/apache-griffin-high-level-overview/">post</a>, I mentioned that there are several observed points regarding Griffin during my exploration.

Before we start, here are several notes.

<ul>
<li>Leveraged docker with image from apachegriffin/griffin_spark2</li>
<li>Only batch validation</li>
<li>Spark local mode</li>
<li>Built Griffin from master branch (0.6.0-SNAPSHOT) because the latest release (0.5.0) had a problem in loading the data sources</li>
</ul>

Continue.

## Yay Points

### Yay 1

As you can see in the <i>Data Quality Configuration</i>, Griffin enables the users to specify multiple data connectors. An example of this point is the following.

```
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
]
```

The above example illustrates that we specified two data sources labeled as <i>src</i> and <i>tgt</i>. Both sources are in the parquet format. Well, actually, we can also leverage other data source types with the corresponding connectors, such as hive, kafka, file based (parquet, avro, csv, tsv, text), jdbc based (mysql, postgresql), and custom (cassandra, elasticsearch).

For more information regarding the data connectors, please visit this <a href="https://github.com/apache/griffin/blob/master/griffin-doc/measure/measure-configuration-guide.md#data-connector">page</a>.

### Yay 2

Griffin supports storing records that don't meet the constraints (bad records). However, this bad records are only stored in HDFS currently.

To store the bad records, just add the following in the <i>Data Quality Configuration</i>.

```
"out": [
  {
    "type": "record",
    "name": "file_name"
  }
]
```

For the sake of clarity, here's the complete example.

```
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
            "type": "record",
            "name": "file_name"
          }
        ]
      }
    ]
  },
```

The bad records should be stored in `hdfs:///griffin/persist`. Run the following command to get the full path.

```
hdfs dfs -ls -R /griffin/persist
```

Run the following to show the file content.

```
hdfs dfs -cat <PATH_TO_BAD_RECORDS_FILE>
```

### Yay 3

The next one is that Griffin supports multiple sinks for metrics output. Several examples are console, HDFS, HTTP, MongoDB, and custom sink.

For more information on this sinks please visit this <a href="https://github.com/apache/griffin/blob/master/griffin-doc/measure/measure-configuration-guide.md#sinks">page</a>.

To specify the sinks, just add the following to the <i>Environment Configs</i>.

```
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
]
```

The above example shows that we provide two options for outputting the metrics, namely console and HDFS.

To select the relevant sinks from the specified options, add the following to the <i>Data Quality Configs</i>.

```
"sinks": [
    "CONSOLE",
    "HDFS"
]
```

### Yay 4

Griffin supports renaming for the metrics' name. Here's a simple example.

```
"details": {
          "source": "src",
          "target": "tgt",
          "miss": "miss_count",
          "total": "total_count",
          "matched": "matched_count"
}
```

From the above example, we rename <i>miss</i> metric to <i>miss_count</i>. The same also applies for <i>total</i> and <i>matched</i>.

Please visit the <a href="https://github.com/apache/griffin/blob/master/griffin-doc/measure/measure-configuration-guide.md#rule">rule</a> section for more details.

### Yay 5

Griffin supports API service.

Please visit this <a href="https://github.com/apache/griffin/blob/master/griffin-doc/docker/griffin-docker-guide.md">page</a> for more information.

---

## Nay Points

There are also several limitations for Griffin.

### Nay 1

I tried to specify several configs for <i>Uniqueness</i> constraint, such as the following.

```
"evaluate.rule": {
    "rules": [
      {
        "dsl.type": "griffin-dsl",
        "dq.type": "UNIQUENESS",
        "out.dataframe.name": "unique",
        "rule": "first_name",
        "details": {
          "source": "src",
          "target": "tgt",
          "unique": "unique"
        },
        "out": [
          {
            "type": "metric",
            "name": "unique_first_name"
          },
          {
            "type": "record",
            "name": "invalid_unique_first_name"
          }
        ]
      },
      {
        "dsl.type": "griffin-dsl",
        "dq.type": "UNIQUENESS",
        "out.dataframe.name": "unique",
        "rule": "last_name",
        "details": {
          "source": "src",
          "target": "tgt",
          "unique": "unique"
        },
        "out": [
          {
            "type": "metric",
            "name": "unique_last_name"
          },
          {
            "type": "record",
            "name": "invalid_unique_last_name"
          }
        ]
      },
      {
        "dsl.type": "griffin-dsl",
        "dq.type": "UNIQUENESS",
        "out.dataframe.name": "unique",
        "rule": "first_name, last_name",
        "details": {
          "source": "src",
          "target": "tgt",
          "unique": "unique"
        },
        "out": [
          {
            "type": "metric",
            "name": "unique_first_last_name"
          },
          {
            "type": "record",
            "name": "invalid_unique_first_last_name"
          }
        ]
      }
    ]
  }
```

From the above example, there are three configs for <i>Uniqueness</i> constraint. The first config validates the constraint for column <i>first_name</i>. The second one validates column <i>last_name</i>. The last one validates multiple column of <i>first_name</i> and <i>last_name</i>.

The problem is that Griffin only returns the metrics of the <b>last</b> config, namely for <i>unique_first_last_name</i>.

I checked the metrics stored in HDFS and the result was still the same. However, the bad records stored in HDFS are completely fine for all the configs (<i>invalid_unique_first_name</i>, <i>invalid_unique_last_name</i>, and <i>invalid_unique_first_last_name</i>).

### Nay 2

Next, I observed that how Griffin computes the duplication metrics for the <i>Uniqueness</i> constraint.

If you see the <a href="https://github.com/apache/griffin/blob/master/griffin-doc/measure/measure-configuration-guide.md#rule">rule</a> for uniqueness, there are the following parameters.

```
dup: the duplicate count name in metric, optional.

num: the duplicate number name in metric, optional.

duplication.array: optional, if set as a non-empty string, 
the duplication metric will be computed, and the group metric name is this string.
```

From the above description, the duplication metrics are specified via the <i>dup</i> and <i>num</i>. This is how these metrics are computed.

```
1. dup -> count the number of occurrence for each category.
2. num -> count the number of occurrence for each "dup".
```

I'm pretty sure that the above doesn't make you understand. Let's take a look at an example.

```
df = {A: [X, Y, X, X, Y, Z, Z, P, P, P]}

dup -> df.groupby(A).count()

result => {X: 3, Y: 2, Z: 2, P: 3}

dup_result => result - 1 => {X: 2, Y: 1, Z: 1, P:2}

========

From "dup_result", we know that there are two categories with one duplicate count (Y & Z) and two categories with two duplicate counts (X & P).

The number of categories becomes the value for "num".

{"duplicate count": "number of categories"} -> {1: 2, 2: 2}
```

The duplication metrics are returned as the following.

```
Suppose that duplication.array = "dup_metric"

Then, 

"dup_metric": [
	{
		"dup": 1,
		"num": 2
	},
	{
		"dup": 2,
		"num": 2
	}
]
```

Well, what's the concern then?

If you observe the computation approach, it's obvious that <b>Griffin requires huge memory when the duplication metrics is activated</b>. Should keep this point in mind when choosing to use the duplication metrics.

### Nay 3

Bad records for <i>Completeness</i> constraint are not stored intuitively.

Let's take a look at an example.

Suppose we have the following dataset.

```
col_a    |    col_b
==========================
"apache" |    "griffin"
""       |    "spark"
"hello"  |    "world"
"batch"  |    ""
"batch"  |    "streaming"
""       |    ""
==========================
```

We then execute the <i>Completeness</i> constraint to check whether our data has missing values.

To keep it simple, let's just use a short notation here.

```
Completeness(col_a) = 4/6
Completeness(col_b) = 4/6
Completeness(col_a, col_b) = 3/6
```

How about the stored bad records?

```
Completeness(col_a)

{}
{}

Completeness(col_b)

{}
{}

Completeness(col_a, col_b)

{"col_b": "spark"}
{"col_a": "batch"}
{}
```

The problem is, how to interpret the above bad records? This is worse for <i>Completeness(col_a)</i> and <i>Completeness(col_b)</i> only return empty dictionaries.

This is how we interpret the bad records for <i>Completeness(col_a, col_b)</i>.

```
`{"col_b": "spark"}` means that a certain row with "col_b" equals to "spark" has missing value for "col_a".
However, we don't know which row it is.

The same rule applies to `{"col_a": "batch"}`.
```

### Nay 4

The point is that we need another alternative approach for other validations, such as value ranges (min, max). It seems that we can't directly leverage the available constraints for this validation.

Looking at the available constraints, it's obvious we can't use <i>Uniqueness</i>, <i>Completeness</i>, or <i>Timeliness</i>. However, we can use <i>Accuracy</i> constraint with a bit engineering.

If you read the <a href="https://github.com/apache/griffin/blob/master/griffin-doc/measure/dsl-guide.md#accuracy">documentation</a>, it's obvious that the <i>Accuracy</i> constraint leverages "LEFT JOIN" as the core computation such as the following (simplified).

```
We have a source data S and a target data T.
We would like to compare how many records in T matches records in S.

We can use "Accuracy" for this purpose.

1. Get the miss records of T that don't match with S.

joined_table = S LEFT JOIN T ON <ALL_COLUMNS>

Select all records where T.* are NULL (missed records).

2. Count the number of missed records.
```

Now, let's get back to our original problem of validating value ranges.

Since "LEFT JOIN" is the core of the <i>Accuracy</i> computation, we need another table to join. For the case of value ranges, we could create a dummy table with two columns (min & max) and a single row like the following.

```
DUMMY TABLE

MIN	|	MAX
===================
10	|	50
```

Suppose below is our source table.

```
VALUE
=====
5
10
20
50
90
100
```

Now, here's the part of config for <i>Accuracy</i> validation.

```
"data.sources": [
    {
      "name": "src",
      "connector": {
        "type": "file",
        "config": {
          "format": "parquet",
          "paths": [path/to/source/table]
        }
      }
    },
    {
      "name": "dummy",
      "connector": {
        "type": "file",
        "config": {
          "format": "parquet",
          "paths": [path/to/dummy/table]
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
        "rule": "src.VALUE >= dummy.MIN AND src.VALUE <= dummy.MAX",
        "details": {
          "source": "src",
          "target": "dummy"
        },
        "out": [
          {
            "type": "metric",
            "name": "accu"
          },
          {
            "type": "record",
            "name": "invalid_records"
          }
        ]
      }
    ]
}
```

Let's take a look at how Griffin retrieves the bad records for the above config and examples.

```
Get the missed records

SELECT src.* FROM src LEFT JOIN dummy 
ON src.VALUE >= dummy.MIN AND src.VALUE <= dummy.MAX 
WHERE (NOT (src.VALUE IS NULL)) AND (dummy.MIN IS NULL AND dummy.MAX IS NULL)

Left Join Result

VALUE   |   MIN   |   MAX
==========================
5       |   NULL  |   NULL
10      |   10    |   50
20      |   10    |   50
50      |   10    |   50
90      |   NULL  |   NULL
100     |   NULL  |   NULL
==========================

Select Result (missed records) where src.VALUE is NOT NULL and dummy.MIN & dummy.MAX is NULL

VALUE
=====
5
90
100
=====
```

End.
