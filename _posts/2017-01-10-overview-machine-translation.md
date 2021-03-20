---
title: 'Get Acquainted with Rule-based Machine Translation'
date: 2017-01-10
permalink: /posts/2017/01/machine-translation/
tags:
  - machine translation
  - natural language processing
  - rule-based nlp
---

### Introduction

Have you ever used Google Translate, Bing Translator, or BabelFish? Why do you choose to use those kind of apps? You simply try to find the meaning of the foreign language in more efficient way, don't you? Thus, the motivation of these apps is simple. They ease your needs when you are in a foreign place in which almost nobody speaks in your language. You just type the foreign language and it gives you the translated language. You even don't need to open your conventional dictionary, find the suitable index, and browse line by line till find the word you are looking for. Still, the interesting question is how does this machine translation software work?

-----

### General Operations

_Translation_ means taking a sentence in one language (source language) and yielding a new sentence in other language (target language) which has same meaning. _Machine_ means the translation process is done by software rather than humans. Generally, any machine translation (MT) software implements this workflow:

<ul>
	<li>Input Phase
		<ul>
			<li>Source Text</li>
			<li>Deformating</li>
			<li>Pre-editing</li>
		</ul>
	</li>
	<li>Analysis Phase
		<ul>
			<li>Morphological Analysis</li>
			<li>Syntax Analysis</li>
			<li>Semantic Analysis</li>
		</ul>
	</li>
	<li>Representation Phase
		<ul>
			<li>Internal Representation of Source Language</li>
			<li>Transfer to Internal Representation of Target Language</li>
		</ul>
	</li>
	<li>Generation Phase
		<ul>
			<li>Syntax Generation</li>
			<li>Semantic Generation</li>
		</ul>
	</li>
	<li>Output Phase
		<ul>
			<li>Reformating</li>
			<li>Post-editing</li>
			<li>Target Text</li>
		</ul>
	</li>
</ul>

Let's take a look at each of the phase.

### _Input Phase_

This is the step where the MT system receives the source language containing two portions, namely non-translation and translation materials. Non-translation materials are charts, figures, and any materials that do not need translation. Whilst the latter portion is the natural text.

Afterwards, the MT system will **deformat** the source text, where it removes any portions from the source text that do not need translation (non-translation materials). This sub-step returns a source text that contains only translation materials (natural text).

However, the current state of the source text may not be forming the effective sentence, which means that the source text is still too long, there are words that are repeated, the sentence is rambling, etc. In this case, we can handle this problem by segmenting the source text into a shorter text while still considering the semantic. This is done in the **pre-editing** sub-step and it returns the pre-processed source text to be sent to the text analyzer.

### _Analysis Phase_

We've got the preprocessed source text and we're ready to analyze the structure. When we'd like to analyze the text, we consider the analysis process towards several aspects, such as morphology, syntax, and semantic.

Let's first take a look at the definition of morphology according to <a href="https://en.wikipedia.org/wiki/Morphology_(linguistics)">Wikipedia</a>:

> Morphology is the study of words, how they are formed, and their relationship to other words in the same language. It analyzes the structure of words and parts of words, such as stems, root words, prefixes, and suffixes.

As we can see, **morphological analysis** simply determines the words attributes (elements that form the words, such as stems, root words, etc) and structures (sequence of POS, prefix, suffix, etc). We observe these things as we have to take them into our consideration when we want to convert the source language to the target language which may have different morphology. Furthermore, we can use morphological analysis result to build a new sentence in the target language in which the grammar's aspect is correct.

Next, we proceed to the **syntax analysis**. Generally, it determines the structural rules governing the composition of clauses, phrases, and words in the given text. An example of the sentence composition is the subject, predicate, and object. Syntax Analysis will try to find out the type of composition based on the part of speech that has been already determined by the morphological analysis. It also uses parsing technique to get the sentence's composition (syntactic tree). Syntax Analysis can be used to detect the language in which as an example it will detect a text is in English when the composition is based primarily on subject and predicate.

For the last analyzer, the MT system will check the **semantic** aspect of the source text. In this case, semantic talks about the structure and meaning of text. The system will try to understand the objective of the sentence and make a proper interpretation for the objective's model in which it utilizes structural information from the syntax analysis. The objective's model is a semantic network in which when we combine it with the syntax tree we'll get the internal (core) structure of a sentence.

### _Representation Phase_

This step has two portions, namely Internal Representation of Source Language and Transfer to Internal Representation of Target Language. Like I've explained before that we'll get a core structure of a sentence (which contains its syntax and semantic) when we merge syntax tree and semantic network. This result tells us that every type of language (source and target language) has their own internal structure that becomes the basic principle in building a sentence in the corresponding language. So, this step will fetch the internal structure of the source language from the analysis phase and build a new internal structure for the target language which may have a different representation. It then returns the result to the next phase where it will be used as the basic form to build the target language.

### _Generation Phase_

When we do the analysis process, the primary activities are we check the morphology to determine the attributes of words, the syntax to find out the composition of words, and the semantic to understand the objective of words. Now, we've got the basic form for the target language. We've known the way to build the composition and check the whole meaning (to ensure that the grammar is correct) simultaneously. The last thing we need to do is using the model to generate the whole target language. We can see that the language has two portions, namely syntax and semantic which means that we need to generate these aspects to get the whole target language. We can use syntax tree and semantic network to build the composition and the objective respectively.

### _Output Phase_

This is the last step in the process of machine translation. We've got the target language in the form of natural text yet we do need to remember that our initial input may contain another parts that are not included in the translation processes (non-translation materials). So, we need to reformat our final text so that it contains the excluded materials.

Afterwards, we do need to make sure that the quality of the text has good status. The text has good status when the quality of the translation is upto mark. This sub-step is executed after reformatting the text as the MT system needs to balance the syntax and semantic of the translated text by considering all types of translation elements (translation and non-translation).

Finally, we've got the target text as the result of the translation.

### References

<ul>
	<li><a href="http://language.worldofcomputing.net/category/machine-translation">Machine Translation Process</a></li>
	<li><a href="http://veredshwartz.blogspot.co.id/2015/08/machine-translation-1-overview.html">Machine Translation Overview</a></li>
</ul>

Here are some advanced materials to support your learning process:

<ul>
	<li><a href="https://devblogs.nvidia.com/parallelforall/introduction-neural-machine-translation-with-gpus/">Introduction to Neural Machine Translation with GPUs</a></li>
	<li><a href="http://neural-monkey.readthedocs.io/en/latest/machine_translation.html">Machine Translation using Neural Monkey</a></li>
	<li><a href="http://www.statmt.org/">Statistical Machine Translation</a></li>
</ul>
