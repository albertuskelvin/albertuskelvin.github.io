---
title: 'Python-like to Algorithm Specification'
date: 2017-12-07
permalink: /posts/2017/12/python-code-to-algorithm/
tags:
  - python programming language
  - algorithm
  - natural language
---

An interesting paper: http://www.phontron.com/paper/oda15ase.pdf

_Title: Learning to Generate Pseudo-code from Source Code using Statistical Machine Translation_

The primary goal of the research was to develop a system that translates python programming language into pseudocode (algorithm specification). The extensibility states that the system should be able to translate any programming languages into the relevant specification.

On the research implementation, they used two approaches to handle the task. The first one is using phrase-based translation. It simply translates each word in the source code into the corresponding natural language representation. A drawback emerges when we use this approach. The result of translation might not be in the right sequence. To overcome this issue, a procedure for re-ordering the word sequence must be applied.

On the other hand, the second approach used tree diagram to represent the source code's internal structures. The difficulty of implementing this technique appears when the sub-task called Derivation Selection is executed. In this step we need to cluster the source code's entities that have been parsed in the previous step. This clustering step is needed so that the system knows which primary instruction is being executed.

In this article, I will show the process of developing such system. The developed system has the same goal with the one developed by NAIST's researchers.

**-- Improving the flexibility --**

Source code is written by using pre-defined keywords and syntax. This means that the programmers have already had the guidelines to write the source code by using the selected programming language. However, there is no guarantee that the level of tidiness of writing is good enough. For instance, the number of spacing between identifiers might not be always one.

Therefore, a preprocessing step was needed to handle this issue. The preprocessing included two filtering task. The first one was removing the whitespaces between a mathematical operator (+, -, *, /, etc) and expression (ex. a+b*c), while the second one was adding a single whitespace between an identifier (if, for, while, etc) and conditional operator (>=, <, ==, <=, >, etc).

This figure illustrates the algorithm for the first preprocessing task.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/revised_opmaths_1.png?raw=true" alt="Algorithm for the first preprocessing task" />

**Fig. 1 The code for the 1st preprocessing task**

While this figure illustrates the algorithm for the second preprocessing task.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/revised_operators_1.png?raw=true" alt="Algorithm for the second preprocessing task" />

**Fig. 2 The code for the 2nd preprocessing task**

As we can see from the above illustration, there are two revision steps for appending a single whitespace between an identifier and mathematical operator. This is caused by the fact that there are two similar mathematical operators for the case of assignment. Those two operators are = and ==. If we only used the first revision step, the case where we need to do conditional instruction will not be executed properly. Simply put, a conditional instruction if a==b will be altered into if a = = b.

For the sake of clarity, suppose our input code is shown in the figure 3 below.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/input_code_2.png?raw=true" alt="Input code" />

**Fig. 3 Input source code in Python-like language**

And here is the result of both of preprocessing tasks.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/revise_op_id_2.png?raw=true" alt="Result of both of preprocessing tasks" />

**Fig. 4 The result of the 1st and 2nd preprocessing task**

**-- Intermediate representation --**

Before a line of code is transformed into their corresponding algorithm specification (in natural language), an intermediate representation for every keyword is needed. This representation makes the mapping process becomes more efficient.

This figure illustrates the creation of intermediate representation.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/inter_rep_1.png?raw=true" alt="The creation of intermediate representation" />

**Fig. 5 The code for creating the intermediate representation**

There are two search processes in creating the representing list. The first process tries to search for the source code's identifiers. The identified word will then be extracted in order to fetch its expression in the term of pseudocode. In addition, the list of identifiers and conditional operators contain several instances which are represented in this form => if:jika, while:selama, in:pada, for:untuk setiap and so on. For the sake of clarity, here is the illustration of the list of identifiers.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/identifiers_1.png?raw=true" alt="The identifiers"/>

**Fig. 6 The list of identifiers in Python programming language**

And this figure below is the illustration of the list of conditional operators.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/operators_1.png?raw=true" alt="the list of conditional operators"/>

**Fig. 7 The list of operators in Python programming language**

In short, here is the illustration of how the intermediate representation looks like when our input code is as shown in the figure below.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/input_code_1.png?raw=true" alt="the input code"/>

**Fig. 8 The input source code (in a text file)**

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/inter_rep_2.png?raw=true" alt="the intermediate representation"/>

**Fig. 9 The intermediate representation of the input code**

From fig. 9 we can see the rule behind the creation of representation. For instance, the identifier will be converted into a single word in a list, while the conditional operators have more than one elements in their list. Let's take an example. A list containing 'kurang dari', 'a', '2' as its elements means that a conditional operator '<' has two arguments, namely a and 2. The actual form of this interlingua is **a < 2**.

In addition, there is a special token called END which indicates the end of a line of source code. This token will be used for adding the newline when the system generates the algorithm specification.

**-- Mapping into the algorithm specification --**

This becomes the last step in which the system generates the relevant algorithm specification from the intermediate representation. This figure 10 exhibits the source code of mapping process.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/mapping_1.png?raw=true" alt="translation to pseudocode"/>

**Fig. 10 The code for mapping the interlingua into specification**

The mapping process is simple since we already had the logical form of the input source code. However, we need to handle several specific case, such as the natural language generation for the assignment operator (=) and mathematical operator (%). In order to yield the specification statement with high degree of naturalness, we might not want x sama dengan 1 as the pseudocode of x = 1. On the contrary, the option of assign nilai x dengan 1 is preferred over the previous one.

Furthermore, the system also considers the method description. This ability only applies when there is only one method in a line of code. More explanation on this are provided in the next section.

**--- Method description ---**

Method description is a task of extracting method's elements, such as the name and arguments. This task was accomplished by making use of regular expression.

As stated in the previous section (last paragraph), the description task will only be implemented when the corresponding line of code only has a single method. For improving the system's capability in considering the multiple methods in a line of code, we need to adjust the regular expression.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/method_desc_1.png?raw=true" alt="revised method structure"/>

**Fig. 11 Check whether we need to describe the method**

From the fig. 11, we can see that the used regex pattern was (\S+)\s*\(\s*\S+\s*(?:,\s*\S+\s*)*\). It simply searched for the occurrence of method's name and arguments. The pattern for the method's name was (\S+)\s*, while the pattern for the method's arguments was \(\s*\S+\s*(?:,\s*\S+\s*)*\). The figure below shows how the system extracts the name and arguments.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/method_desc_2.png?raw=true" alt="how the system extracts the name and arguments"/>

**Fig. 12 The extraction of method's name and arguments**

The system exploited two methods in Python regular expression, videlicet the start() and end() method. The former retrieve the first index of character in the string that matches the pattern. Whereas, the latter retrieve the last index of character that matches the pattern. In addition, this last index is added by one according to the official method's definition.

After the name and arguments have been extracted, the old method's declaration was replaced by a string stating the method's description. In this case, when the old method's declaration is func(a, b), then the replacement would be method_bernama_func_yang_memiliki_argumen_a,b.

**-- Final show --**

Using the same input (fig. 8), here is the generated algorithm specification.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/python-algo-to-code/final_spec_1.png?raw=true" alt="the generated algorithm specification"/>

**Fig. 13 The generated algorithm specification**

