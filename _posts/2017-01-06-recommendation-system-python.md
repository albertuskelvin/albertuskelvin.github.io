---
title: 'Build a Simple Recommendation System using Python'
date: 2017-01-06
permalink: /posts/2017/01/recommendation-system-python/
tags:
  - recommendation system
  - collaborative filtering
  - python
---

### Introduction

Have you ever visited sites providing services for movies, dating, food, music, books, shopping, or even jokes? Have you ever noticed that in certain condition you suddenly find out several options of product that attract your attention? Have you ever thought that when you choose to click those options, it is one of the company's strategies to make you buy their products with larger amount (they also hope that you will spend your time longer on their site and come back again for you think that the provided products are interesting)?

So, if you are interested in finding the answers for those enquiries, the primary question is how do the provider / company provide those options that matched your preferences? The answer is they use a feature called the **recommendation system**. This feature will try to find your preferences by analyzing your purchases history and by doing this, you can say that you are acquainted by the system. Since every person has different preferences, the system will try to build a user model that stores your evaluation for each type of product. Your evaluation can be represented as ratings from one to five, as yes/no voting, etc.

Therefore, one of solution to build the user model with high performance is to collect more information from the other users. The core principle is the system will search for a large group of users and find a smaller set which have similar tastes as yours. Afterwards, the system will fetch the products they buy or like and combine them as a list of pre-user preferences. Next, the list will be processed again by calculating the probability of preference in which these values will be utilized to build a new list based on ranking (highest probability will be stored in the first place and the lowest will in the last place). This new list will be returned to the current user as the recommended items. This kind of technique is called as **collaborative filtering**.

As I mention before that the system need to collect another information from a smaller set of users, so it needs a method to find the similarities among users. There are several methods that can be implemented and I'll explain them in the next sub-section of this article.

-----

### Building Preferences Dictionary

In this article, we will use movies as the type of product. The user can rate a movie from one to five where one is the lowest sentiment value and five is the highest sentiment value.

So, let's start by creating a small dictionary that stores the rating for the movies chosen by every user. The simplest way is to represent it in JSON (JavaScript Object Notation) format (it is recommended to store the dictionary in a database when you have a large dataset). Here is the code.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/recsys/recsys_dict0.png?raw=true" alt="Preferences Dictionary 0" />

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/recsys/recsys_dict1.png?raw=true" alt="Preferences Dictionary 1" />

-----

### Finding Similar Users

After creating the dictionary used for storing user's preferences, the next step is to find the users which have similar tastes. These users can be presumed as a small group in which every user's choices may also be accepted by the other user with high probability.

To find the similarity score, we can use several techniques, such as Euclidean Distance, Pearson Correlation Score, Manhattan Distance, Jaccard Coefficient, etc. Yet in this article, we will only cover two techniques, namely Euclidean Distance and Pearson Correlation Score. You can find more information about other techniques for comparing items at [http://en.wikipedia.org/wiki/Metric_%28mathematics%29#Examples](http://en.wikipedia.org/wiki/Metric_%28mathematics%29#Examples).

### _Euclidean Distance_

This technique is very simple for it implements the characteristic of pythagoras theorem, which means you can just calculate the square root of the sum of all the squares of the difference scores between two points.

So, how do we implement this technique exactly? The simplest way is by creating a two (or more) dimensional chart where the movies are used as the axes and the users as the items on the chart. This chart can be called as the **preference space** and the idea is the closer two users are in the preference space, the more similar their preferences are.

After finding the idea behind this technique, we can simply use the pythagoras theorem to calculate the distance between two users. The equation is **distance = sqrt(pow(x1-x2, 2) + pow(y1-y2, 2))** where x1 and x2 are the positions of every user based on the horizontal line of the space, whereas y1 and y2 are the positions based on the vertical line of the space. Obviously, this equation can be used for the space whose dimensional value is more than two.

By using that equation, we'll get the distance between two users and we can see that two users are presumed to have similar tastes when the distance score approaches zero. Yet, we have to change this characteristic in which the high return value should indicate that two users are similar. You can achieve this by simply state that the equation will return 1 for the highest similarity prediction and 0 for the lowest similarity prediction. To implement this approach, we can modify our equation for calculating the distance by adding one to the final square root value and inverting it. In other words, the equation will become **distance = 1 / (1 + sqrt(pow(x1-x2, 2) + pow(y1-y2, 2)))**. This equation will return value from 0 to 1. We add 1 in the denominator so that the we do not do zero division.

This is the implementation code for this technique:

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/recsys/recsys_euclidean.png?raw=true" alt="Euclidean Distance" />

<br />

### _Pearson Correlation Score_

If you want to use the other technique which is more complicated yet more powerful then you should try Pearson Correlation Score. The correlation score is used to measure the fitness score of two sets of data on a straight line. The data used for this technique is different from the previous technique in which we use movies as the axes and the users as the items on the chart. However, in this case we'll do vice versa where the users will be used as the axes and the movies as the items on the chart.

Now, what is the basic idea behind this technique? The traditional way is it draws a straight line that comes as close to all the items on the chart as possible. The perfect correlation score (one) is when the straight line touches every item on the chart and this case happens when both users give the same score for every movie rated by them. 

Talking about correlation score, it has an interesting aspect, namely it enables this technique to resolve a problem when both users have relatively similar preferences. If one user tends to give higher scores than the other, this technique will still say that both users have similar preferences. But, this approach works if the difference between the rating given by every user is consistent. It can be understood from the straight line which still touches most of the items on the chart.

So, how about the implementation code for this technique? 

First step, we do need to find the movies rated by both users and if there are no ratings in common, we can not calculate the similarity score as it is clear that both users do not have similar preferences at all. The last thing for this step is we store those movies in a list.

For the next three steps, we still access the list in which we add up the ratings score, sum up the squares of the ratings, and add up the product of the ratings. These values will be used to measure the correlation score.

This is the implementation code for this technique:

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/recsys/recsys_pearson.png?raw=true" alt="Pearson Correlation Score" />

<br />

-----

### Ranking the Similar Preferences

In this step we have had methods for comparing two users and finding their similarity score. We do this by selecting one user and compare his/her preferences with the other users providing rating for the movies rated by both users. 

Afterwards, we will rank them so that we know whose recommendation I should take when deciding on a movie. We can do this by simply sorting the similarity scores stored in the list where the highest score appears at the top. This is the implementation code for this step:

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/recsys/recsys_ranking.png?raw=true" alt="Ranking the Similar Preferences" />

-----

### Creating Recommendations

Till now we have built a system that can provide several users whose advice should be considered when a user wants to choose a movie. In other words, when a user wants to read reviews from the other users, he/she may select a user whose similarity score is very close to his/her score (this is the purpose of ranking process).

However, the primary goal of this system is to provide movies recommendation (movies that have not been chosen by a user yet). Therefore, the similar approach to this problem is when we give a user's name as an input, the system will provide a list of recommended movies rather than a list of the other users whose advice might good enough to consider.

A traditional way to find recommended movies for me is by looking at the person who has tastes similar to mine and get the movies they like that I have not watched yet. But, this approach could give me information that has high inaccuracy. The simplest example of this problem is there is a possibility that the similar user I selected has not watched another movies that I might like. So, in this case the options are limited and I just wait for that similar user to expand his/her movies collection. The actual action is the system should be able to predict any movies I might like not only come from the similar user but also by considering ratings provided by the other users. The system could also find the suitable movies based on my preferences towards the item characteristic. In this case, the system will analyze the content of the items and predict whether the characteristic is match with my preferences.

To solve this issue, we need to give a weighted score that ranks the users. Take the rating of all the other users and multiply how similar they are to me by the rating they gave for each movie. This table will give you a better understanding.

<table>
	<tr>
		<td><b>Users</b></td>
		<td><b>Similarity</b></td>
		<td><b>Lucy</b></td>
		<td><b>S x Lucy</b></td>
		<td><b>Lights Out</b></td>
		<td><b>S x Lights Out</b></td>
		<td><b>Finding Nemo</b></td>
		<td><b>S x Finding Nemo</b></td>
	</tr>
	<tr>
		<td>user00</td>
		<td>0.99</td>
		<td>3.0</td>
		<td>2.97</td>
		<td>2.5</td>
		<td>2.48</td>
		<td>3.0</td>
		<td>2.97</td>
	</tr>
	<tr>
		<td>user01</td>
		<td>0.38</td>
		<td>3.0</td>
		<td>1.14</td>
		<td>3.0</td>
		<td>1.14</td>
		<td>1.5</td>
		<td>0.57</td>
	</tr>
	<tr>
		<td>user03</td>
		<td>0.89</td>
		<td>4.5</td>
		<td>4.02</td>
		<td>-</td>
		<td>-</td>
		<td>3.0</td>
		<td>2.68</td>
	</tr>
	<tr>
		<td>user04</td>
		<td>0.92</td>
		<td>3.0</td>
		<td>2.77</td>
		<td>3.0</td>
		<td>2.77</td>
		<td>2.0</td>
		<td>1.85</td>
	</tr>
	<tr>
		<td>user05</td>
		<td>0.66</td>
		<td>3.0</td>
		<td>1.99</td>
		<td>3.0</td>
		<td>1.99</td>
		<td>-</td>
		<td>-</td>
	</tr>
	<tr>
		<td>Total</td>
		<td>-</td>
		<td>-</td>
		<td>12.89</td>
		<td>-</td>
		<td>8.38</td>
		<td>-</td>
		<td>8.07</td>
	</tr>
	<tr>
		<td>Sim. Sum</td>
		<td>-</td>
		<td>-</td>
		<td>3.84</td>
		<td>-</td>
		<td>2.95</td>
		<td>-</td>
		<td>3.18</td>
	</tr>
	<tr>
		<td>Total/Sim. Sum</td>
		<td>-</td>
		<td>-</td>
		<td>3.35</td>
		<td>-</td>
		<td>2.83</td>
		<td>-</td>
		<td>2.53</td>
	</tr>
</table>

The above table shows the users having similar tastes with **user06** and three movies that **user06** has not rated yet. Also, the column with **S x (movie's name)** gives the similarity score multiplied by the rating, so a user who is similar to **user06** will contribute more to the overall score than a person who is different from **user06**.

Moreover, we can just use the **Total** values as the base of movies ranking. This **Total** values tell us that the system considers all reviews provided by every user that has similar tastes with the _current user_. This diversity makes the recommending process become balance.

However, if we analyze further, we can see that if we use the **Total** value for ranking the recommended movies then the movies rated by more users would have a big advantage, yet the movie's characteristic might not be suitable with the _current user_'s preference. To handle this problem, we can utilize the similarity scores in which their addition will divide the **Total** values. The summation of the similarity scores depends on whether the corresponding user has rated that movie. For example, _Lucy_ is rated by all users so we sum up all the similarity scores. However, _Lights Out_ is not rated by _user03_ so we skip the similarity score given by _user03_. This approach can reduce the tendency of **Total** values to dominate the recommending process.

So, after normalizing the **Total** values, we can just use the result represented by this formula: **Total / Sim. Sum**. We use this value to rank the movies so that the user know the best recommended movie.

In addition, we can see that we would get different recommendation scores when we implement different similarity metric (ex. Euclidean Distance, Pearson Correlation Score, etc). This is the primary element that affects the recommendation scores. It should be considered according to your application's type.

This the implementation code for this step:

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/recsys/recsys_getrecom.png?raw=true" alt="Creating Recommendations" />

<br />

You've now built a complete recommendation system!

-----

### References

The primary resource is <a href="http://it-ebooks.directory/book-0596529325.html">Programming Collective Intelligence</a>. It's an interesting book and I recommend you to learn from there.
 
Here are some great materials to support your learning process:

<ul>
	<li><a href="https://www.toptal.com/algorithms/predicting-likes-inside-a-simple-recommendation-engine">Predicting Likes Inside a Simple Recommendation Engine</a></li>
	<li><a href="http://www.cs.carleton.edu/cs_comps/0607/recommend/recommender/itembased.html">Item-based Collaborative Filtering</a></li>
	<li><a href="http://siplab.tudelft.nl/sites/default/files/sigir06_similarityfusion.pdf">Unifying User-based and Item-based Collaborative Filtering Approaches by Similarity Fusion</a></li>
	<li><a href="https://cseweb.ucsd.edu/~jmcauley/cse255/reports/wi15/Guanwen%20Yao_Lifeng_Cai.pdf">User-Based and Item-Based Collaborative Filtering Recommendation Algorithms Design</a></li>
</ul>
