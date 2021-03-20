---
title: 'Modifying the Code Profiler to Use Custom sort_stats Sorters'
date: 2019-08-02
permalink: /posts/2019/08/pyspark-profiling-custom-sorters/
tags:
  - spark
  - python
  - profiling
  - custom stats sorters
---

Code profiling is simply used to assess the code performance, including its functions and sub-functions within functions. One of its obvious usage is code optimisation where a developer wants to improve the code efficiency by searching for the bottlenecks in the code.

In PySpark, the profiler is built on top of Python’s cProfile. The full source code of the profiler module can be found <a href="https://github.com/apache/spark/blob/master/python/pyspark/profiler.py">here</a>.

To use the PySpark’s profiler, one can use the following ways:

```python
conf = SparkConf().set("spark.python.profile", "true”)
sc = SparkContext('local', 'spark_profiling', conf=conf)

spark = SparkSession(sc)
```

SparkContext actually has an argument used to specify the profiler class, that is <b>profiler_cls</b>. By default, PySpark uses <I>BasicProfiler</I> as the profiler class.

However, this <i>BasicProfiler</i> inherits the <I>Profiler</I> class and implement the <I>show()</I> method from <I>Profiler</I>. The <I>show</I> method uses a very specific sorter (“time” and “cumulative”) which acts as the “orderBy” parameters. Well, sometimes we might want to use different sorter. Fortunately, PySpark allows us to create custom profilers to fulfil our specific requirements. To create a custom profiler, one can implement the following code:

```python
from pyspark import BasicProfiler

class MyProfiler(BasicProfiler):
	def show(self, id):
		""" Print the profile stats to stdout, id is the RDD id """
		stats = self.stats()
		if stats:
		    print("=" * 60)
		    print("Profile of RDD<id=%d>" % id)
		    print("=" * 60)
		    
		    # use different sorters
		    stats.sort_stats(“ncalls”, “time”, “name”).print_stats()


conf = SparkConf().set("spark.python.profile", "true")

# pass the custom profiler as the profiler class
sc = SparkContext('local', 'spark_profiling', conf=conf, profiler_cls=MyProfiler)
spark = SparkSession(sc)
```

Problem solved.

The problem is, I was still curious about this custom sorter implementation. At first I thought that it was not pleasant for me to create a custom profiler only to replace the sorting parameters. Most of the time I only need to modify the sorters, by the way. The rest of the <I>show()</I> method is actually the same with the one in the <I>BasicProfiler</I>. Therefore, I decided to create a bit modification in the PySpark’s profiler module.

Basically, I added a <I>setter</I> and <I>getter</I> method to the <I>Profiler</I> class in python/pyspark/profiler.py. Here’s the full modified module.

```python
class Profiler(object):
	def __init__(self, ctx):
		… the code is the same as the original method

	def profile(self, fun):
		… the code is the same as the original method

	def stats(self):
		… the code is the same as the original method

	# add a setter
	def set_sort_stats_sorters(self, sorter):
		self._sort_stats_sorters = sorter

	# add a getter
	def get_sort_stats_sorters(self):
		return self._sort_stats_sorters

	def show(self, id):
		stats = self.stats()
		if stats:
			print(“=“ * 60)
			print(“Profile of RDD<id=%d>” % id)
			print(“=“ * 60)

			# get the custom sorters
			sorter = self.get_sort_stats_sorters()
			stats.sort_stats(*sorter).print_stats()

	def dump(self, id, path):
		… the code is the same as the original method
```

Seems good and simple. Now, how to use the <I>setter</I> method? Check out the following code snippet.

```python
conf = SparkConf().set("spark.python.profile", "true")

sc = SparkContext('local', 'spark_profiling', conf=conf)
sc.profiler_collector.profiler_cls.set_sort_stats_sorters((BasicProfiler, ['ncalls', ‘time’, 'name'])
spark = SparkSession(sc)
```

If you want to use the <I>setter</I> for a custom profiler, just modify the above code snippet a bit.

```python
class MyProfiler(BasicProfiler):
	def show(self, id):
		""" Print the profile stats to stdout, id is the RDD id """
		stats = self.stats()
		if stats:
		    print("=" * 60)
		    print("Profile of RDD<id=%d>" % id)
		    print("=" * 60)
		    
		    # use different sorters
		    sortby = self.get_sort_stats_sorters()
		    stats.sort_stats(*sortby).print_stats()


conf = SparkConf().set("spark.python.profile", "true")

sc = SparkContext('local', 'spark_profiling', conf=conf, profiler_cls=MyProfiler)
sc.profiler_collector.profiler_cls.set_sort_stats_sorters((MyProfiler, ['ncalls', ‘time’, 'name'])
spark = SparkSession(sc)
```

Thanks for reading!
