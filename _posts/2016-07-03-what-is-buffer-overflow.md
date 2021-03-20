---
title: 'What is Buffer Overflow?'
date: 2016-07-03
permalink: /posts/2016/07/what-is-buffer-overflow/
tags:
  - computer organization and architecture
  - buffer overflow
  - assembly
---

Buffer Overflow is one of code's exploitation technique which uses buffer weakness. In addition, buffer is a block or space for saving datas.

The example of buffer is suppose we have an array of char that can contain 5 datas (can be written as char ArrChar[5]). This means we have a container named ArrChar that can contain 5 datas which is classified as character value. The container itself can be referred as buffer.

Then, why can this buffer overflow technique successfully run? Evidently, it happens because the programmer was less careful in choosing code for getting user input. It means that the code used for getting user input doesn't do checking of the length of that input. One example for that code is **gets()** which is used for receiving string input. Although the programmer had declared an array of char for saving that string input with certain length of character, the function **gets()** will not check whether the total number of character got input is more than 5 or not.

Then, what's the effect of that non-checking process? From the concept of **Stack Frame**, one of the stack's content is a value of **return address** from called function/ procedure. That **return address** is used as the starting point for executing the next code after calling previous function/ procedure. When the length of character exceeds the limit of buffer, then there is a possibility that the input character can be filling the value of **return address** (_overwrite_). As the result of that process, after calling the relevant function/ procedure, the program will not execute the actual next code, yet it'll execute the code that has the replaced **return address**.    

To understand further about this _overwrite_ process, I'll explain more about **Stack Frame** in the next section.
