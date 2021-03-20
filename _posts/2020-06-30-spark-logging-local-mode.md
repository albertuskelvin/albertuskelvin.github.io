---
title: 'Running Local Mode Spark with Logging via spark-submit'
date: 2020-06-30
permalink: /posts/2020/06/spark-logging-local-mode/
tags:
  - spark
  - logging
  - local
---

Below is a script for running spark via `spark-submit` (local mode) that utilizes logging.

### File: run.sh

```
# === RUN WITH SPARK SUBMIT ===
# Please adjust the spark-submit arguments accordingly

SPARK_SUBMIT_PATH="<please fill>"
LOG4J_PROPERTIES_PATH="<please fill>"
EXTRA_JAVA_OPTIONS="-Dlog4j.configuration=file:${LOG4J_PROPERTIES_PATH}"

${SPARK_SUBMIT_PATH} \
--master local[*] \
--conf "spark.driver.extraJavaOptions=${EXTRA_JAVA_OPTIONS}" \
--class <main_class> \
<path_to_application_jar> \
[application_arguments]
```
