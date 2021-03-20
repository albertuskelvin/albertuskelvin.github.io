---
title: 'RDD to DF Gave a StopIteration Exception'
date: 2019-03-06
permalink: /posts/2019/03/rdd-to-df-stopiteration/
tags:
  - spark
  - rdd
  - dataframe
  - stopiteration
---

I made a silly mistake a few days ago - well, yes.

I was working on a data sampling related experiment. The algorithm is pretty simple but requires me to convert the inputted dataframe to an RDD first. The sampling process will be applied on the RDD. Afterwards, as the final step, the sampled RDD is converted to its dataframe representation.

Now, here's the problem. When the sampled RDD was converted to its dataframe form, I got a **StopIteration** exception. Based on what I know, such an exception occurs when an iterator is done and it raises **StopIteration** exception when we call its **next** method. Well, however, it's not considered as an error.

I tried to search for some solution in the context of Spark. One of them was to use **createDataFrame()** instead of **toDF()** to convert an RDD to DF. According to an online reference, the latter method couldn't be applied on RDD rows. However, my previous experiences show that it was fine to use **toDF()** for RDD-to-DF conversion. But since I was curious, I decided to give it a try. The result? This annoying exception still occurred -_-.

Another solution search cycle. Found some references again and this time they stated that the primary cause of such an issue was the python version compatibility. Like, what?? FYI, I was using Python 3.7 at that time.

So, yeah, I quickly created a new environment and overrode the Python version - made it to 3.6. Executed the code again and voila!

Here are the references I used to solve the problem:

<ul>
  <li><a href="https://stackoverflow.com/questions/51700960/runtimeerror-generator-raised-stopiteration-every-time-i-try-to-run-app">"RuntimeError: generator raised StopIteration" every time I try to run app</a></li>
  <li><a href="https://github.com/PyCQA/astroid/issues/539">RuntimeError raised while running under Python 3.7</a></li>
</ul>

Hope it helps.
