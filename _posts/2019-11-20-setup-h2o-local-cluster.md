---
title: 'Making H2O Cluster Information Shows Plausible Results for Total & Allowed Cores'
date: 2019-11-20
permalink: /posts/2019/11/setup-h2o-cluster-local-mode/
tags:
  - h2o
  - machine learning
  - big data
---

H2O provides a platform for building machine learning models in a scalable way. By focusing on scalability, it leverages the concept of cluster computing and therefore enables engineers to perform big data analytics.

Setting up a H2O cluster in local mode is pretty straightforward. If you use Python, H2O can be installed via `pip`. In addition, installing H2O needs few dependencies, such as the followings.

```
pip install requests
pip install tabulate
pip install "colorama>=0.3.8"
pip install future
```

Then, we can now install H2O.

```
pip install h2o
```

After the installation completed, we can now apply the functionalities. For the sake of simplicity, let's open up a Terminal and type `python`.

```
h2o.init()
```

The above instruction tells H2O to set up a cluster. It simply searches for any existing H2O instances in your local machine. If it didn't find any, it'll start a local H2O server in your machine.

In this post, we'll use local mode for setting up a H2O server meaning that we only have one available node (your own machine) in total.

One thing to note here is that when the cluster has been set up, you might find that the parameter of `H2O cluster total cores` and `H2O cluster allowed cores` are assigned to zero. This shouldn't happen as the total cores in the local machine should be more than zero.

The solution is pretty simple yet tricky. Several references on the internet say that we can just shut down the cluster and then start it up again afterwards. So it should be like `h2o.cluster().shutdown()` and then `h2o.init()`.

However, the above approach didn't work for me. After finding out that the number of total and allowed cores were zero, I just re-executed the `h2o.init()` command. It worked! Those two parameters finally showed plausible results.
