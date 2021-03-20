---
title: 'Implementing Balanced Random Forest via imblearn'
date: 2019-04-12
permalink: /posts/2019/04/brf-imblearn/
tags:
  - balanced random forest
  - imblearn
  - machine learning
---

Have you ever heard of <a href="https://imbalanced-learn.readthedocs.io/en/stable/api.html">imblearn</a> package? Based on its name, I think people who are familiar with machine learning are going to presume that it's a package specifically created for tackling the problem of imbalanced data. If you do a deeper search, you're gonna find its GitHub repository <a href="https://github.com/scikit-learn-contrib/imbalanced-learn">here</a>. And yes, once again, it's a Python package for playing with imbalanced data.

Since it was built for imbalanced learning, the core feature should be **sampling** APIs. There are some sampling algorithms presented on the site starting from undersampling, oversampling, and hybrid-approach (combination of under- and over-sampling). I was working on a project which required me to apply balanced random forest (BRF) algorithm. For those who want to know more about BRF, I've actually mentioned about it on my previous <a href="https://albertuskelvin.github.io/posts/2019/03/balanced-random-forest/">post</a>. However, I didn't know if **imblearn** had an API for BRF algorithm. So I decided to create my own BRF's implementation (code from scratch). I still remember that there were several bottlenecks that I think would reduce the overall performance. One of the bottlenecks was explained briefly in my previous <a href="https://albertuskelvin.github.io/posts/2019/03/balanced-random-forest/">post</a>.

Another bottleneck occurred as I was using Spark's built-in random forest classifier. As you might have already known, Spark's RF classifier has its own model saving method. If we save a list containing Spark's RF classifiers, I think it might not work as expected as based on my previous observation, Spark created few directories storing information related to the classifier. Yet, I haven't tried to use scikit-learn's random forest classifier yet. Random forest in scikit-learn can be stored using common method, e.g. using dill. Have you tried it? How was the result?

And last but not least, the last bottleneck was related to model's predicting performance. By performance, I mean how long a BRF predicts a data test. Since I was using Spark's RF classifier at its core, the prediction process done by a single RF classifier took a pretty long time (I should investigate more about this issue). Therefore, I think it wasn't a good idea to implement this BRF algorithm manually and from scratch.

Based on those bottlenecks, I finally managed to find that **imblearn** had an API for BRF. I checked the reference and the title and the author were the same as the paper I read before -> <i>Chen, Chao, Andy Liaw, and Leo Breiman. "Using random forest to learn imbalanced data." University of California, Berkeley 110 (2004): 1-12.</i>

The implementation using **imblearn** is greatly straightforward. Here a simple code taken from the official site.

<pre>
from imblearn.ensemble import BalancedRandomForestClassifier
from sklearn.datasets import make_classification

X, y = make_classification(n_samples=1000, n_classes=3, n_informative=4, 
                           weights=[0.2, 0.3, 0.5], random_state=0)

clf = BalancedRandomForestClassifier(max_depth=2, random_state=0)
clf.fit(X, y)  # doctest: +ELLIPSIS

print(clf.feature_importances_)

print(clf.predict([[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]]))
</pre>

You can check the site to try out different model's attributes.

To save and load the model, just use the common pickle operation (assuming that you're using Python). Here's an example.

<pre>
import dill

# save model
with open(model_path, 'wb') as f:
      dill.dump(clf, f)
      
# load model
with open(model_path, 'rb') as f:
      brf_model = dill.load(f)
</pre>

Greatly neat, right?
