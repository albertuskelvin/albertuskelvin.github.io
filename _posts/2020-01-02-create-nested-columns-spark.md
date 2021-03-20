---
title: 'Creating Nested Columns in PySpark Dataframe'
date: 2020-01-02
permalink: /posts/2020/01/create-nested-columns-spark/
tags:
  - spark
  - nested column
  - dataframe
---

A nested column is basically just a column with one or more sub-columns. Take a look at the following example.

```
=================
| nested_col	|
=================
| col_a	| col_b	|
=================
| dummy	| dummy	|
| dummy	| dummy	|
=================
```

The above dataframe shows that it has one nested column which consists of two sub-columns, namely <I>col_a</I> and <I>col_b</I>.

To make it brief, let's take a look at how we can create a nested column in PySpark's dataframe.

```python
from pyspark.sql.types import StringType, StructField, StructType

schema_p = StructType(
	[
		StructField('x', StringType(), True),
		StructField('y', StringType(), True)
	]
)

schema_df = StructType(
	[
		StructField('dummy_col', StringType(), True),
		StructField('p', schema_p, True)
	]
)
 
df = spark.createDataFrame([
			('dummy_a', ['x_a', 'y_a']), 
			('dummy_b', ['x_b', 'y_b']),
			('dummy_c', ['x_c', 'y_c'])
], schema_df)
```

The above code simply does the following ways:
<ul>
<li>Create the inner schema (<I>schema_p</I>) for column <b>p</b>. This inner schema consists of two columns, namely <b>x</b> and <b>y</b></li>
<li>Create the schema for the whole dataframe (<I>schema_df</I>). As you can see, we specify the type of column <b>p</b> with <I>schema_p</I></li>
<li>Create the dataframe rows based on <I>schema_df</I></li>
</ul>

The above code will result in the following dataframe and schema.

```
root
 |-- dummy_col: string (nullable = true)
 |-- p: struct (nullable = true)
 |    	|-- x: string (nullable = true)
 |    	|-- y: string (nullable = true)


==========================
| dummy_col |     p      |
==========================
| dummy_a   | [x_a, y_a] |
| dummy_b   | [x_b, y_b] |
| dummy_c   | [x_c, y_c] |
==========================
```

To access the sub-columns, we can use `dot (.)` character. Here are some of operations we can apply to the above dataframe.

```python
# select the second elements of column 'p'
df.select('p.y')

# filter out all the rows whose the first element is 'x_a'
df.filter(F.col('p.x') == 'x_a')

# group based on the first element of column 'p'
df.groupby('p.x').count()
```

However, this `dot (.)` mechanism can’t be applied on all the dataframe operations. One of the examples is `drop` operation.

```
df.drop('p.x').printSchema()

Resulting schema:
root
 |-- dummy_col: string (nullable = true)
 |-- p: struct (nullable = true)
 |    	|-- x: string (nullable = true)
 |    	|-- y: string (nullable = true)
```

Or even when we'd like to replace the sub-column name with a new one.

```
df.withColumnRenamed('p.x', 'new_px').printSchema()

Resulting schema:
root
 |-- dummy_col: string (nullable = true)
 |-- p: struct (nullable = true)
 |    	|-- x: string (nullable = true)
 |    	|-- y: string (nullable = true)
```

Hope it helps. Thank you for reading.
