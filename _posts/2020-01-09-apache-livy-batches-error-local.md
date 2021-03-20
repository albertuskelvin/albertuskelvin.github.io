---
title: 'User Sessions Addition Error When Submitting Spark Job to Apache Livy via Local Mode'
date: 2020-01-09
permalink: /posts/2020/01/apache-livy-error-on-spark-job-file-submission/
tags:
  - livy
  - spark
  - batches
---

A few days back I tried to submit a Spark job to a Livy server deployed via local mode. The procedure was straightforward since the only thing to do was to specify the job file along with the configuration parameters (like what we do when using `spark-submit` directly).

However, the following error response was returned after the job submission.

```
{"msg":"requirement failed: Local path <path/to/job/file> cannot be added to user sessions."
```

Turns out that Livy actually provides a way to configure this problem. We might attack this issue by specifying the proper value for `livy.file.local-dir-whitelist` variable in `livy.conf`.

According to the documentation, this `livy.file.local-dir-whitelist` variable stores a list of local directories from where files are allowed to be added to user sessions. It's empty by default which means that users can only refer to remote URIs when submitting the Spark job.

So the solution should be straightforward. We could just set the `livy.file.local-dir-whitelist` to all the directories in which the Spark job files are stored.

In addition, I also tried to use an alternative workaround. Just add `local:/` before the job path worked for me.

```
curl -X POST -d '{"file": "local:/path_to_job_file"}' -H "Content-Type: application/json" <livy_server_address>:<livy_server_port>/batches
```

Hope it helps.
