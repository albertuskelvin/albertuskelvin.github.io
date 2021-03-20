---
title: 'Statistical Machine Translation - Language Models'
date: 2017-01-11
permalink: /posts/2017/01/smt-language-models/
tags:
  - statistical machine translation
  - language models
  - natural language processing
---

### Introduction

In the first article of the **Natural Language Processing and Machine Translation** series, we talk about an overview of machine translation, especially how this kind of system works. It is also explained that the primary purpose of Machine Translation (MT) system is to translate the source language into the target language by considering the meaning of text.

Generally, MT system is divided into two categories, namely **Rule-based MT** and **Statistical MT**. Yet in this article I will only talk about Statistical MT.

So, what is Statistical Machine Translation (SMT)? We can see that it is formed by three words, namely _statistical_, _machine_, and _translation_. Now let's take a look at each of the word's meaning. _Machine_ means that the translation process is done by software rather than human. _Translation_ means that the system takes source text in a language and convert it to the target text in a different language with the similar semantic. _Statistical_ means that we don't use human who is expert in the certain language to give an explanation about the language, including the structure of language, grammatical rules, lexicon, etc, yet we use statistic calculation on the text. When we talk about statistic calculation, we do need a large enough corpus that stores many text in a certain language. After we got this corpus, we'll use it as the reference for the calculation. The calculation is mainly focused on probability of a word, phrase, or sentence to be composed in a certain language.

### What makes a translation good?

A MT system is considered to have a good quality of translation if it fulfills all these requirements, namely:

<ul>
	<li>
		The target text has the similar meaning as the source text. The focus here is on the semantic aspect in which it is considered to be good when the target text can deliver the similar enough objective as the original one.
	</li>
	<li>
		The target is grammatically correct. The focus here is on the syntactic aspect in which the target text should have the right structure (ex. the sequence of words composing the whole sentence is correct).
	</li>
</ul>  

Based on the requirements, SMT has its own feature to handle every aspects of a good translation. To handle the first requirement, SMT uses **translation model** which ensures the correctness of semantic, whereas to handle the latter one, SMT uses **language model** which ensures the correctness of structure. In this tutorial, we will only talk about **language model**.

### Language Model

Language model is used to determine the appropriateness of a word, phrase, or sentence in a certain language. Let's say we use sentence for the sample, then the language model will give a score stating the probability of composing the sentence in a certain language (ex. English). The score ranges from 0 to 1, where a high probability means that the sentence is appropriate to be composed in a certain language.

Language model uses a large enough corpus as its dictionary. It calculates the probability by searching for the sentence in the corpus. Therefore, we can find the probability of a certain word to occur in a certain language just by searching the number of the word in the corpus and divide the value by the total number of words in the corpus. For example, we want to find the probability of word _car_ in an English corpus. We find that the number of the word _car_ is 5000 and the total words is 100000000, so the probability is 5/100000.

However, we can also find the probability of a sentence to occur in a certain language. Because a sentence is just a sequence of words, we can use the concept of conditional probability to solve this problem. If **A0,A1,A2,...,An** denotes a sentence where **Ai** denotes a word in the sentence, then the probability of a sentence to occur in a certain language is denoted by P(A0,A1,A2,...,An) which means we have to calculate the probability of a word A0 AND a word A1 AND a word A2 AND so on till An. To achieve the result, we use the chain rule: 

<table>
	<tr>
		<td>P(A0, A1, A2, ..., An) = P(A0) P(A1 | A0) ... P(An | A0, A1, ..., An-1)</td>
	</tr>
</table>

From the formula, we can see that it computes the conditional probability. When we have <b>P(An | A0, A1, ..., An-1)</b>, it means that we need to compute the probability of a word **An** to be the next word of the sentence **A0A1...An-1**. Let's say we want to compute the probability of sentence _I am sleeping_, so we use the formula and we get <b>P(I am sleeping) = P(I) P(am | I) P(sleeping | I am)</b>.

When we want to compute the probability of a word given the sequence of words beforehand (probability of _sleeping_ after the sentence _I am_), we can estimate the value by calculating the number of _I am sleeping_ in the corpus and divide the value by the number of _I am *_ in the corpus, where * can be any words as long as it appears in the corpus.

### N-Gram Language Model

We have known that to compute the probability of a word **W** to appear as the next element in a sequence of **XYZ**, we do need to compute the number of sentences consisting of **XYZ** and **W** which means we need to find the sentences (**XYZW**) in the corpus. However, there's still a possibility that we can not find the sentences as the corpus does not contain any sentence which is created from all the possibility of the word's permutation. In case of this, we'll get 0 as the probability whilst the sentence might be a part of the language.

So, how to handle this problem? Let's get acquainted with **N-Gram Language Model**. It is actually the result of _Markov assumption_ in which when we compute the conditional probability, we presume that every word only depends on the specified amount of the preceding words (we can denote the amount as **K**). Let's take an example with a sentence which says _I am reading a book_ and we use K = 2. With these data, we can apply the formula, namely 

<table><tr><td>P(I am reading a book) = P(I) P(am | I) P(reading | am) P(a | reading) P(book | a)</td></tr></table>

in which we can see that every word depends on exactly one preceding word. This type of N-Gram is called by **bigram**. We can also use K=1 (**unigram**), K=3 (**trigram**), and so forth, yet the smaller K more likely give better result as the probability to find sentence in the corpus is higher.

### Smoothing

So, until now we've solved the dependency problem which means we are not depended on the corpus too much. Next, how about any words that is rare, such as _absorbefacient_ or _blue wine_? Of course the probability that we don't find these words in the corpus is high, nevertheless we can still resolve this problem with a method called **Smoothing**.

What is **Smoothing**? It is an assumption where we add one or more virtual words (denoted by **VW**) to the corpus so that the probability of any rare words is not zero (we will get a small probability but not zero). For example, assume that the word _absorbefacient_ is not found in the corpus so we add an imaginary word _absorbefacient_ (may be more than one) and a custom dictionary containing some chosen words in the corresponding language. The purpose of a custom dictionary is to store all the rare words which the probability will be computed.

Let's take an example with a word _sleep_ where the initial number of word sleep in the corpus is 100, the value of VW is 1, the size of our custom dictionary is 5, and the size of corpus is 100000000.

> P(sleep) = (100 + 1) / (100000000 + 5) = 101/100000005

### References

<ul>
	<li><a href="http://veredshwartz.blogspot.co.id/2015/09/language-models.html">http://veredshwartz.blogspot.co.id/2015/09/language-models.html</a></li>
	<li><a href="http://www.statmt.org/">http://www.statmt.org/</a></li>
</ul> 
