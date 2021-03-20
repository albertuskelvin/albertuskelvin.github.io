---
title: 'Accessing Resources with Extra Classpath as spark-submit Config'
date: 2020-07-17
permalink: /posts/2020/07/spark-submit-extra-classpath/
tags:
  - spark
  - extra classpath
  - resources
  - scala
---

A few days ago I came across a case where a module needs access to the `resources` directory.

For the sake of clarity, here’s the basic project structure.

```
src/
	main/
		resources/
		scala/
	test/
		resources/
		scala/
```

Here’s an example of the module that accesses a file in `resources` directory in `main`.

```scala
import java.nio.charset.StandardCharsets

import com.google.common.io.Resources

object X {
	private def readResource(name: String): String = 
	        Resources.toString(Resources.getResource(name), StandardCharsets.UTF_8)
}
```

Here’s an example of the spark-submit script to run the application.

```
# file: run.sh

SPARK_SUBMIT_PATH="<please fill>"

${SPARK_SUBMIT_PATH} \
--master local[*] \
--class <main_class> \
[path_to_application_jar] \
[application_arguments]
```

Running the above script would result in the an exception telling that the resource file couldn’t be found.

The solution is simple. We only need to pass another spark-submit config called `extraClassPath` such as the following.

```
# file: run.sh

SPARK_SUBMIT_PATH="<please fill>"
RESOURCES_DIRECTORY_PATH="<please fill>"

${SPARK_SUBMIT_PATH} \
--master local[*] \
--conf "spark.driver.extraClassPath=${RESOURCES_DIRECTORY_PATH}" \
--class <main_class> \
[path_to_application_jar] \
[application_arguments]
```

From the above `run.sh` file, `RESOURCES_DIRECTORY_PATH` refers to the directory containing all the resource files.
