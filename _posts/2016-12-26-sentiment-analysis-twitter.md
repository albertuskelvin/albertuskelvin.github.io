---
title: 'Sentiment Analysis for Twitter using WEKA'
date: 2016-12-26
permalink: /posts/2016/12/sentiment-twitter-weka/
tags:
  - machine learning
  - natural language processing
  - sentiment analysis
  - twitter
  - weka
  - java
---

## Introduction

One of the most important things that can be a signal of a successful product is the users want to use it since it fulfills their needs. To achieve that point, the executive people from companies need to evaluate their products performance when officially released to public. One way to do that is by knowing the users reaction towards the product's quality. By knowing the users reaction, they can improve the quality of production for they can learn about users expectation and sure it may help them to pay more attention to user-driven development (UDD), a processes in which the needs of users of a product become the top priority in the product development processes. 

Talking about the way to evaluate a product's quality, the executive people can find the public opinion in social media, such as Twitter, Facebook, Instagram, etc. They can find their product and analyze the users opinion towards it. They can do it manually, yet off course it will take a long time to get the evaluation process done. So, they need something that can do this job automatically so that they can be more productive rather than only just determining the positive or negative opinion from every tweet.

One of the common application used to do this job is Sentiment Analyzer. This app can determine how positive or negative a statement text is. It implements Natural Language Processing as the core technology to analyze and understand the content of a sentence text. In this brief article, I will explain about this application used for classifying tweet's sentiment extracted from Twitter. So, let's start!
  
-----

## Programming Languages

Programming Languages:

<ul>
	<li>Java (JSP, Servlet)</li>
</ul>

Frameworks and Libraries:

<ul>
	<li>WEKA</li>
	<li>AngularJS</li>
	<li>Bootstrap</li>
	<li>jQuery</li>
	<li>twitter4j (Java library for the Twitter API)</li>
</ul>

-----

## Supporting Model

This application uses a classification method called <b>SlidingWindow</b> which determines the model to be used when the <b>"disagreement"</b> condition is met. The <b>"disagreement"</b> is a condition where the core classifier can not classify the tweet into the class of positive or negative.<br />

If we choose to use <b>SlidingWindow</b>, there are two possible conditions when we receive the result of classification, namely:<br />   

<ul>
	<li><b>If the core classifier returns the <i>positive</i> or <i>negative</i> as the predicted class,</b>
		<ul>
			<li>put the corresponding instance in the data train. We should also check whether the total number of instance is more than 1000. If so, remove the first instance</li>
		</ul>
	</li>
	<li><b>If the core classifier returns <i>nan</i> as the predicted class,</b>
		<ul>
			<li>put the corresponding instance in the data test and it will be classified in the end of the process</li>
			<li>For this condition, we should consider these possibilities too:
				<ul>
					<li><b>If the number of instances in the data train is more than 0,</b>
						<ul>
							<li>decides upon a "disagreed" document by applying the learned model based on the last 1000 "agreed" documents (stored in "train" instances)</li>
						</ul>
					</li>
					<li><b>Else,</b>
						<ul>
							<li>unknown class for the train data is empty and can not do the training process. In this condition, we can just return random class as the final result.</li>
						</ul>
					</li>
				</ul>
			</li>
		</ul>
	</li>
</ul>

If we choose <b>not</b> to use <b>SlidingWindow</b>, then there are also two possible conditions, namely:<br />

<ul>
	<li><b>If the core classifier returns the <i>positive</i> or <i>negative</i> as the predicted class,</b>
		<ul>
			<li>set the list of predicted class come from three possibilities</li>
		</ul>
	</li>
	<li><b>If the core classifier returns <i>nan</i> as the predicted class,</b>
		<ul>
			<li>decides upon a "disagreed" (out = nan) document by applying the learned model based on the previously built model</li>
		</ul>
	</li>
</ul>

-----

## General Concept

In this part, I will explain the core principal of this Sentiment Analysis and some code snippets so that you can understand the backend-logic better.

### Extracts Twitter Data

<ul>
	<li><b>Sets the query</b></li>
	<ul>
		<li>
			We begin by fetching user's query and assign the value to a global variable.
		</li>
	</ul>
	<li><b>Extracts tweet</b></li>
	<ul>
		<li>
			Afterwards, we fetch the corresponding tweets which contain the keyword inputted by the user. This step uses Twitter API so it does need some authentication tokens (you can get these tokens from your Twitter's developer account). 
		</li>
	</ul>
	<li><b>Sets the tweet text</b></li>
	<ul>
		<li>
			To achieve the optimum result of classification, we only need the text feature from the original tweet. It means that we do not need another supporting features, such as published time, username and name of person posting the tweet, location when he/she was posting the tweet, etc. So, we just store the tweet text into a list. Simple!		
		</li>
	</ul>
	<li><b>Sets the amount of tweet</b></li>
	<ul>
		<li>
			The last step for this stage is we just need to store the number of extracted tweets.
		</li>
	</ul>
</ul>

### Sentiment Analysis

<ul>
	<li><b>Initializes Sentiment Processor</b>
		<ul>
			<li><b>Sentiment Analyser</b>
				<ul>
					<li><b>Trainer</b>
						<ul>
							<li><b>Initializes BidiMap objects for text, feature, and complex representation</b>
								<ul>
									<li>
										We use <b>DualHashBidiMap</b> that stores the pair of <i>String</i> and <i>Integer</i>. The content of this BidiMap is all of the attributes from the dataset of every tweet's representation, namely <b>text</b>, <b>feature</b>, and <b>complex</b>. I will explain these three representations in the next sub-point.
									</li>
								</ul>
							</li>
							<li><b>Trains lexicon model</b>
								<ul>
									<li>It is a supervised learning using LibSVM classifier</li>
									<li>It trains the model for the lexicon-based representation</li>
									<li>It saves the model in order to use it on the provided test sets</li> 
									<li>The rest of the model representation forms will be created on-the-fly because of the minimum term frequency threshold that takes both train and test sets into consideration</li>
								</ul>
							</li>
							<li><b>Trains text model</b>
								<ul>
									<li>The concept is this model will determine the sentiment value based on the whole opinion or division of opinions</li>
									<li>It builds and saves the text-based model built on the training set</li>
									<li>The training set has already been preprocessed, where every special symbol is converted into a string representing the symbol (example: converts @user into 'usermentionsymbol', www.example.com into 'urllinksymbol', etc)</li>
									<li>The general steps are:
										<ul>
											<li>Retrieves a new instances with a filter inside</li>
											<li>Saves the instances to a file named according to the type of representation. In this case, we use text representation, so the name will be 'T.arff'</li>
											<li>Writes the attributes from the filtered instances (can be retreived from 'T.arff') to a file in 'attributes' folder (text.tsv). The attributes are the tokenized words and the representation is based on the used tokenizer</li>
											<li>Creates classifier (NaiveBayesMultinomial), build model, and save model</li>
										</ul>
									</li>
								</ul>
							</li>
							<li><b>Trains feature model</b>
								<ul>
									<li>The concept is this model will determine the sentiment value based on the Twitter features, such as links, mentions, and hash-tags</li>
									<li>It builds and saves the feature-based model built on the training set</li>
									<li>The training set has already been preprocessed, where every special symbol is converted into a string representing the symbol (example: converts @user into 'usermentionsymbol', www.example.com into 'urllinksymbol', etc)</li>
									<li>The general steps are:
										<ul>
											<li>Retrieves a new instances with a filter inside</li>
											<li>Saves the instances to a file named according to the type of representation. In this case, we use feature representation, so the name will be 'F.arff'</li>
											<li>Writes the attributes from the filtered instances (can be retreived from 'F.arff') to a file in 'attributes' folder (feature.tsv). The attributes are the tokenized words and the representation is based on the used tokenizer</li>
											<li>Creates classifier (NaiveBayesMultinomial), build model, and save model</li>
										</ul>
									</li>
								</ul>
							</li>
							<li><b>Trains complex model</b>
								<ul>
									<li>The concept is this model will determine the sentiment value based on the combination of text and POS (Part of Speech). Each word is assigned to the corresponding POS</li>
									<li>It builds and saves the complex-based model built on the training set</li>
									<li>The training set has already been preprocessed, where every special symbol is converted into a string representing the symbol (example: converts @user into 'usermentionsymbol', www.example.com into 'urllinksymbol', etc)</li>
									<li>The general steps are:
										<ul>
											<li>Retrieves a new instances with a filter inside</li>
											<li>Saves the instances to a file named according to the type of representation. In this case, we use complex representation, so the name will be 'C.arff'</li>
											<li>Writes the attributes from the filtered instances (can be retreived from 'C.arff') to a file in 'attributes' folder (complex.tsv). The attributes are the tokenized words and the representation is based on the used tokenizer</li>
											<li>Creates classifier (NaiveBayesMultinomial), build model, and save model</li>
										</ul>
									</li>
								</ul>
							</li>
						</ul>
						
This is an example of code for training the text-based representation:<br />
						
<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/sentanalysisweka/z_trainText.png?raw=true" alt="text-based representation training" />
<br />
					
</li>
<li><b>Polarity Classifier</b>
  <ul>
    <li><b>Initializes BidiMap objects for text, feature, and complex representation</b>
      <ul>
        <li>We create objects of <b>DualHashBidiMap</b> and initialize the value with the previous <b>DualHashBidiMap</b> explained in the <b>Trainer</b> sub-point</li>
      </ul>
    </li>
    <li><b>Initializes filter (StringToWordVector) and tokenizer (NGramTokenizer)</b>
      <ul>
        <li>We choose <b>StringToWordVector</b> as the filterer and <b>NGramTokenizer</b> as the tokenizer. The minimum and maximum size for the tokenizer is 2</li>
      </ul>	
    </li>
    <li><b>Initializes classifiers (MNB and LibSVM)</b>
      <ul>
        <li>Read the model which is built previously</li>
        <li>We use <b>NaiveBayesMultinomial</b> for the text, feature, and complex representation</li>
        <li>We use <b>LibSVM</b> for the lexicon representation</li>
        <li>We also build an instance for every representation. It is built from the data train (<b>T.arff</b>, <b>F.arff</b>, and <b>C.arff</b>)</li> 
      </ul>
    </li>
  </ul>
					
This is an example of code for initializing filterer and tokenizer:<br />
						
<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/sentanalysisweka/z_stwvngram.png?raw=true" alt="initializes filterer and tokenizer" />
<br />
						
This is an example of code for initializing classifiers:<br />
						
<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/sentanalysisweka/z_initClassifier.png?raw=true" alt="initializes classifiers" />
<br />
					
</li>
<li><b>Tweet Preprocessor</b>
<ul>
<li><b>Initializes preprocessor for lexicon, text, feature, and complex representation</b>
<ul>
  <li>We create the object of attributes that will be used to preprocess all the representations before doing the classification. By preprocessing the words, we normalize them to a new sentence without any ambiguities. Here are the preprocessor for every representation:
    <ul>
      <li><b>Lexicon Preprocessor</b>
        <ul>
          <li>Sets the general score for every lexicon (use SentiWordNet as the resource)</li>
          <li><b>Gets the abbreviations and store them in a hash-table</b>
            <ul>
              <li>Fetches the list of abbreviations and return its contents in a hash-table</li>
              <li>Here are the examples: <i>afk=away from keyboard</i>, <i>aka=also known as</i>, <i>asap=as soon as possible</i></li>
              <li>From the examples, the fetched results will be stored in a hash-table: <i><afk, away from keyboard></i>, <i><aka, also known as></i>, <i><asap, as soon as possible></i></li>
            </ul>
          </li>
          <li><b>Gets the happy and sad emotions and store them in a linked list</b>
            <ul>
              <li>Fetches the list of the happy emoticons from a dictionary and store them in a linked list</li>
              <li>Here are the examples: <i>:-)</i>, <i>:)</i>, <i>;)</i></li>
              <li>From the examples, the fetched results will be stored in a linked list: <i>[:-), :), ;)]</i></li>
            </ul>
          </li>
          <li><b>Gets the list of positive and negative words</b></li>
        </ul>
      </li>
      <li><b>Text Preprocessor</b>
        <ul>
          <li><b>Gets the abbreviations and store them in a hash-table</b>
            <ul>
              <li>Fetches the list of abbreviations and return its contents in a hash-table</li>
              <li>Here are the examples: <i>afk=away from keyboard</i>, <i>aka=also known as</i>, <i>asap=as soon as possible</i></li>
              <li>From the examples, the fetched results will be stored in a hash-table: <i><afk, away from keyboard></i>, <i><aka, also known as></i>, <i><asap, as soon as possible></i></li>
            </ul>
          </li>
          <li><b>Gets the happy and sad emotions and store them in a linked list</b>
            <ul>
              <li>Fetches the list of the happy emoticons from a dictionary and store them in a linked list</li>
              <li>Here are the examples: <i>:-)</i>, <i>:)</i>, <i>;)</i></li>
              <li>From the examples, the fetched results will be stored in a linked list: <i>[:-), :), ;)]</i></li>
            </ul>
          </li>
        </ul>
      </li>
      <li><b>Feature Preprocessor</b>
        <ul>
          <li>Gets the abbreviations and store them in a hash-table</li>
          <li>Gets the happy and sad emoticons then store them in a linked list</li>
          <li>Creates the combination of dot symbol and store them in a linked list</li>
          <li>Creates the combination of exclamation symbol and store them in a linked list</li>	
        </ul>
      </li>
      <li><b>Complex Preprocessor</b>
        <ul>
          <li>This preprocessor only returns the Part of Speech (POS) of tweets</li>
        </ul>
      </li>
    </ul>
  </li>
</ul>
								
This is an example of code for retrieving features attributes, such as abbreviations, happy and sad emoticons, etc:<br />
								
<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/sentanalysisweka/z_ctortxtproc.png?raw=true" alt="retrieves feature attributes" />
<br />
								
</li>
<li><b>Initializes Part of Speech (POS) tagger</b>
  <ul>
    <li>We create an object that will be used to analyze the POS using this tagger: <b>wsj-0-18-left3words-distsim.tagger</b></li>
  </ul>
</li>
</ul>
</li>
<li><b>Initializes filterer (StringToWordVector) and tokenizer (NGramTokenizer)</b>
<ul>
<li>We use <b>StringToWordVector</b> as the filterer and <b>NGramTokenizer</b> as the tokenizer</li>
</ul>
</li>
<li><b>Initializes data train and data test in case of we need to use sliding window</b>
<ul>
<li>In the <b>Supporting Model</b> section, I've explained briefly about the classification method that will be used for this application</li>
<li>When we choose to use this method, we need to provide an alternative way in case we receive the <b>nan</b> class as the predicted class. To do that, we have to create a new data train which will be a place for storing the current instance and training the new model based on the "agreed" instances in that data train</li>
</ul>

This is an example of code for creating a new data train and data test for the case of using sliding window:<br />

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/sentanalysisweka/z_inituseSW.png?raw=true" alt="creates new data train and data test for sliding window" />
<br />
						
</li>
</ul>
</li>
<li><b>Tweet initialization</b>
<ul>
<li><b>Initializes the list of tweet sentiment</b>
<ul>
<li>Empty the list which will store the final predicted classes</li>
</ul>
</li>
<li><b>Initializes the list of preprocessed tweet</b>
<ul>
<li>Empty the list which will store the preprocessed tweets</li>
</ul>
</li>
<li><b>Initializes the list of class distribution</b>
<ul>
<li>Empty the list which will store the class probability</li>
</ul>
</li>
</ul>
</li>
<li><b>Get Polarity</b>
<ul>
<li><b>Starts the whole process including preprocesses the given tweet, creates different representations of it (stored in "all[]" Instances) and tests it in the PolarityClassifier class</b>
<ul>
<li><b>Preprocesses the lexicon, text, feature, and complex</b>
<ul>
  <li>Generally, the steps for preprocessing are:
    <ul>
      <li>Replaces emoticons - current tweet is altered to "happy"/"sad"</li>
      <li>Replaces Twitter features, such as links, mentions, hash-tags</li>
      <li>Replaces Consecutive Letters - replaces more than 2 repetitive letters with 2</li>
      <li>Replaces Negation - if current is a negation word, then current = "not"</li>
      <li>Replaces Abbreviations - if current is an abbreviation, then replace it</li>
    </ul>
  </li>
</ul>
								
This is an example of code for preprocessing tweets:<br />
								
<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/sentanalysisweka/z_getprocessed.png?raw=true" alt="preprocesses tweets" />
<br />
								
  </li>
  <li><b>Initializes the instances (data test) of lexicon, text, feature, and complex</b>
    <ul>
      <li>Fetches all instances created before (instances that contains data test)</li>
    </ul>
  </li>
</ul>
</li>
</ul>
</li>
<li><b>Execution of Algorithm</b>
<ul>
<li><b>Gets the instances of every data test representation and applies a filter to it</b>
<ul>
  <li>Returns the instances of text-based representations and apply filter to it in order the format of data test is same with the format of data train</li>
</ul>
</li>
<li><b>Reformates the instances of lexicon, text, feature, and complex</b>
<ul>
  <li>Removes the attributes from the data test that are not used in the data train</li>
  <li>Moreover, it also alters the order of every representation's attributes according to the train files</li>
</ul>
</li>
<li><b>Apply the classifier</b>
<ul>
  <li>This is the main method that sets up all the processes of the Ensemble classifier. It returns the decision made by the two classifiers, namely:
    <ul>
      <li><b>Classifier for text, feature, and complex representation (HC)</b>
        <ul>
          <li>Gets the probability for each class (positive/negative)</li>
          <li>Uses double[] preds = mnb_classifiers[i].distributionForInstance(test.get(0))</li>
          <li>Text-based representation
            <ul>
              <li>Distribution for positive (hc[0]): <b>preds[0]*31.07</b></li>
              <li>Distribution for negative (hc[1]): <b>preds[1]*31.07</b></li>
            </ul>
          </li>
          <li>Feature-based representation
            <ul>
              <li>Distribution for positive (hc[2]): <b>preds[0]*11.95</b></li>
              <li>Distribution for negative (hc[3]): <b>preds[1]*11.95</b></li>
            </ul>
          </li>
          <li>Complex-based representation
            <ul>
              <li>Distribution for positive (hc[4]): <b>preds[0]*30.95</b></li>
              <li>Distribution for negative (hc[5]): <b>preds[1]*30.95</b></li>
            </ul>
          </li>
        </ul>
      </li>
    </ul>
  </li>
  <li><b>Classifier for lexicon representation (LC)</b>
    <ul>
      <li>Gets the probability for each class (positive/negative)</li>
      <li>Presumes that the value is stored in a variable called lc_value</li>
    </ul>
  </li>
  <li><b>Counts the probabilities</b>
    <ul>
      <li>HC classifier 
        <ul>
          <li>Positive score (ps): <b>(hc[0] + hc[2] + hc[4]) / 73.97</b></li>
          <li>Negative score (ns): <b>(hc[1] + hc[3] + hc[5]) / 73.97</b></li>
          <li>Final score (hc_value): <b>(1 + ps - ns) / 2</b></li>
        </ul>
      </li>
      <li>Comparison between HC and LC 
        <ul>
          <li>If hc_value < 0.5 AND lc_value > 0.5: output is negative</li>
          <li>Else if hc_value > 0.5 AND lc_value < 0.5: output is positive</li>
          <li>Else: output is nan</li>
        </ul>
      </li>
    </ul>
  </li>
</ul>

This is an example of code for applying the core classifier:<br />
						
<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/sentanalysisweka/z_apply.png?raw=true" alt="applies core classifier" />
<br />
						
</li>
<li><b>Check the Sliding Window</b>
  <ul>
    <li><b>Case 0: uses sliding window</b>
      <ul>
        <li>If HC and LC agree (positive/negative), then put this document in the data train</li>		
        <li>Else, add the document into the data test. It will be classified in the end of the process
          <ul>
            <li>If the number of data train's instances is more than 0 
              <ul>
                <li>Calls clarifyOnSlidingWindow
                  <ul>
                    <li>Adds an instance into the data train at the end of file</li>
                    <li>Sets a filter for the data train and store the filtered data train in a new instances</li>
                    <li>Prepares the data train and the data test</li>
                    <li>Builds classifier from the data train (1000 training data)</li>
                    <li>Predicts the class based on the model created before</li>
                  </ul>
                </li>
              </ul>
            </li>
            <li>Else, the final class is unknown for the data train is empty and can not do the training process</li>
          </ul>
        </li>
      </ul>
								
This is an example of code when we implement the sliding window:<br />
								
<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/sentanalysisweka/z_useSWtrue.png?raw=true" alt="implements sliding window" />
<br />
								
</li>
<li><b>Case 1: does not use sliding window</b>
  <ul>
    <li>If HC and LC agree (positive/negative), then sets the list of predicted class come from three possibilities</li>
    <li>Else,
      <ul>
        <li>Calls clarifyOnModel which decides upon a "disagreed" (output class = nan) document by applying the learned model based on the previously built model
          <ul>
            <li>Gets the text-based representation of the document</li>
            <li>Re-orders attributes so that they are compatible with the data train</li>
            <li>Finds the polarity of the document based on the previously built model, namely Liebherr, goethe, or Cisco</li>
          </ul>
        </li>
      </ul>
    </li>
									
This is an example of code when we do not implement the sliding window:<br />
									
<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/sentanalysisweka/z_useSWfalse.png?raw=true" alt="does not implement sliding window" />
<br />
									
</ul>
</li>
</ul>
</li>
<li><b>Done</b></li>
</ul>
</li>
</ul>
</li>
</ul>

-----

## References

Thanks to Petter Törnberg (Copyright 2013) for the demo code used to analyze the sentiment value of a word. I implemented the code on SWN3.java with several modifications.

You can read the theories behind the production of this application from these resources:

<ul>
    <li><a href="https://www.researchgate.net/publication/269093520_Feature_based_sentiment_analysis">https://www.researchgate.net/publication/269093520_Feature_based_sentiment_analysis</a></li>
    <li><a href="https://cs.nyu.edu/grishman/jet/guide/PennPOS.html">https://cs.nyu.edu/grishman/jet/guide/PennPOS.html</a></li>
    <li><a href="http://partofspeech.org/">http://partofspeech.org/</a></li>
    <li><a href="http://www.slideshare.net/Cataldo/aiia-14dart">http://www.slideshare.net/Cataldo/aiia-14dart</a></li>
</ul>

-----

## Project Display and Code

You can find the entire source code on my [Github](https://github.com/albertusk95/sentiment-analysis-twitter)
