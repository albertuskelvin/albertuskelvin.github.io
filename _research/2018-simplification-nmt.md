---
title: "Sentence Simplification for Indonesian Using Neural Machine Translation"
collection: research
type: "Research Project"
permalink: /research/2018-simplification-nmt
venue: 
date: 2018-06-08
location: 
---

## ABSTRACT

Sentence simplification makes complex sentence easier to understand. Poor-literacy readers have difficulties in understanding complex sentences. People with aphasia, dyslexia, autism, and hearing disorders also experience the difficulties. Research in sentence simplification has been conducted for many languages. However, very few researches have been conducted for Indonesian. The previous research used rule based approach to simplify complex sentences. In this research, I implemented Neural Machine Translation (NMT) approach for Indonesian sentence simplification. NMT needs a training data consisting of pairs of complex and simple sentence. The training data was created by translating Simple English Wikipedia (SEW) parallel corpus into Indonesian. Experiments were conducted by using Attention-based Encoder-Decoder LSTM. Hyperparameters tuning was done to find the best model. Experimental result showed that the best model had an accuracy of 82.04% and perplexity of 4.378 for validation. Whereas, it gave 0.570 as the average of BLEU-4 score. The experiment using SEW corpus shows that the NMT gave positive result for Indonesian sentence simplification. The accuracy could be improved by using better and not translated corpuses.

## INTRODUCTION

Text simplification plays a role in facilitating information access for readers having disabilities in processing text information. Many related researches have been conducted for some languages. However, very few researches conducted for Indonesian. Previous research used rule-based reasoning and surface expression rules [1]. That approach relied on the performance of Part-of-Speech Tagger. The simplification rules had to be created manually as well.

Statistical approaches for English were implemented by using Simple Wikipedia as the data source. The corpus was created by using certain sentence alignment algorithm like [2], [3]. The simplification was done by applying Statistical Machine Translation [4] dan Neural Machine Translation [5].

However, there is no statistical approach for simplifying sentences in Indonesian. The lack of corpus become the main cause. Therefore, this research implemented Neural Machine Translation (NMT) or deep learning based translation. The research processes started from corpus creation, model training, and model evaluation. The evaluation for each model was conducted based on the accuracy and perplexity on validation data.

## METHODOLOGY

There were 3 main parts in this research, namely Indonesian simplification corpus creation, model training, and model evaluation. The English corpus was used as there was no Indonesian simplification corpus. The parallel corpus was taken from [2] which consists of 492.993 pairs of complex and simple sentences. Corpus from [2] gives better simplification accuracy than other Simple Wikipedia based corpuses.

The parallel corpus was translated to Indonesian using Google Translate. The total number of translated samples was 12.281 which consisted of 11.000 training samples and 1.281 validation samples.

The corpus was normalized by applying several additional processes, such as separation of suffix “-nya” and removal of non-ASCII characters. Several parameters for converting samples into form that can be recognized and processed by machine, such as vocabulary size 5.002 (complex) and 5.004 (simple), maximum sentence length 500, and letter lowercasing.

Hyperparameters tuning was done during model training. The model was based on Encoder-Decoder LSTM with Attention mechanism. The total number of conducted experiments was 17. The optimized model hyperparameters were the size of word embedding, number of layers and hidden states in the encoder-decoder, attention type (dot and general), and the type of encoder layer. Dropout value of 0.3, epochs 40 and batch size 128 were used. The SGD or ADAM were used as the optimization method. The learning rate for SGD and ADAM was 1 and 0.001 respectively. The learning rate was reduced by 50% at every epoch after epoch 20.

The evaluation result is the best 3 models which were determined based on the accuracy and perplexity on validation samples. The testing was done by calculating the average score of BLEU-4 for the 3 models. The testing used beam size of 5. The size of testing samples was 30. The testing samples were taken from [3]. The calculation of BLEU-4 used smoothing function (method4).

## RESULTS AND DISCUSSION

<img src="https://github.com/albertuskelvin/albertuskelvin.github.io/blob/master/images/research/Summer2018_01/table_1_txt_simp_research.png?raw=true"/>

<b>Table 1. Hyperparameters of the best 3 models</b>

Table 1 shows the hyperparameters of the best 3 models. Experiments showed that the small number of layers and hidden states gave better result. General as the type of Attention gave better result than Dot. The use of bidirectional LSTM also improved the model accuracy on validation data.

<img src="https://github.com/albertuskelvin/albertuskelvin.github.io/blob/master/images/research/Summer2018_01/table_2_txt_simp_research.png?raw=true"/>

<b>Table 2. The result of evaluation and testing of the best 3 models</b>

Table 2 shows that the best model has an accuracy of 82.04% and perplexity of 4.378. The best 3 models used ADAM as the optimization method. However, the highest average score of BLEU-4 (0.577) was owned by the model with an accuracy of 81.81%.

Testing with different corpus caused there was no average score of BLEU-4 which was close enough to 1. This shows that the translation pattern of a corpus affects the model performance.

## CONCLUSIONS

<ol>
<li>The architecture of the best model has:
<p>
<ul>
    <li>Word embeddings: 256</li>
    <li>Number of layer: 1</li>
    <li>Hidden states: 256</li>
    <li>Attention type: General</li>
    <li>Encoder layer: BI-LSTM</li>
</ul>
</p>
</li>
<br/>
<li> The best model has an accuracy of <b>82.04%</b> and a perplexity of <b>4.378</b> on validation data<br/><br/></li>
<li> The experimental results show that the NMT gave positive results for Indonesian sentence simplification. The model performance could be improved by using corpuses with better translation pattern and not a result of translation</li>
</ol>

## REFERENCES

[1] R. Widyastuti, M. Fachrurrozi, and N. Yusliani, “Simplification complex sentence in indonesia language using rule-based reasoning,” in Proc. 1st Int. Conf. Computer Science and Engineering 2014, 2014, pp. 85-88. Accessed on: May., 14, 2018. [Online]. Available:https://www.neliti.com/publications/224332/simplification-complex-sentences-in-indonesia-language-using-rule-based-reasoning

[2] T. Kajiwara and M. Komachi, “Building a monolingual parallel corpus for text simplification using sentence similarity based on alignment between word embeddings,” in Proc. COLING 2016, 26th Int. Conf. Computational Linguistics: Tech. Paper, 2016, pp. 1147-1158. Accessed on: May., 15, 2018. [Online]. Available: http://aclweb.org/anthology/C16-1109

[3] W. Hwang, H. Hajishirzi, M. Ostendorf, and W. Wu. (2015). Aligning sentences from standard wikipedia to simple wikipedia. Presented at NAACL-HLT. [Online]. Available:http://ssli.ee.washington.edu/tial/projects/simplification

[4] W. Xu, C. Napoles, E. Pavlick, Q. Chen, and C. C. Burch, “Optimizing statistical machine translation for text simplification,” Transactions of the Association for Computational Linguistics, vol. 4, pp. 401-415, July 2016. [Online]. Available: http://aclweb.org/anthology/Q16-1029. [Accessed May. 15, 2018]

[5] S. Nisioi, S. Stajner, S. P. Ponzetto, and L. P. Dinu, “Exploring neural text simplification models,” in Proc. 55th Ann. Meeting Association for Computational Linguistics, 2017, pp. 85-91. Accessed on: May., 15, 2018. [Online]. Available: doi:10.18653/v1/P17-2014
