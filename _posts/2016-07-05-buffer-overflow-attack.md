---
title: 'Examples of Buffer Overflow Attack'
date: 2016-07-05
permalink: /posts/2016/07/buffer-overflow-attacks/
tags:
  - computer organization and architecture
  - buffer overflow
---

In the earlier section we have learnt a bit about buffer overflow technique. The primary concept is flooding the stack frame with input exceeding the buffer limit so that we can manipulate any datas saved on the stack frame. Some things that can be done using this technique are change the return address so that the attackers can call any functions they want, change the content of variables so that the function executes corresponding code, or change the return value of a function.

Now, here is one illustration. 

Suppose we have a procedure **GetBuff** where inside that procedure there's an array of char named **buff** that can save 4 characters ending with '\0'. Then, we take input from a user with command **gets()** and show the input to the screen with command **puts()**. So, the corresponding code in C language is:

> void GetBuff() {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;char buff[4];<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gets();<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;puts();<br />
> }<br />

Procedure **GetBuff** is called by **main()** where its initial structure of stack frame can be illustrated by this image.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/computerorg/howboworks00.png?raw=true" alt="How BO works" />

From the above initial structure, we can see that there is pointer %ebp that is moved from **main()** so that its value is 0 seen from procedure **GetBuff** point of view. Pointer %esp is also located at address %ebp-4 where there is an allocation space for array of char buff in that block.

Now, let's we assume the important values for further usage:

> return address GetBuff() : 0x08048427<br />
> address of register %ebp on frame **GetBuff()** when being called: 0xbfea2fb8<br />

Now, we will do I/O process and see what happens with procedur **GetBuff**'s stack frame.

> input : AAA<br />
> output: AAA<br />

Character 'A' is represented as 41 in hexadecimal.

And next what happens with the procedure **GetBuff**'s stack frame.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/computerorg/howboworks01.png?raw=true" alt="How BO works" />

We can see that the allocated space for buff[4] has been filled with 3 characters 'A' and 1 character _null-terminated string_. If we look at the Assembly code, the register %esp located at address %ebp-4 (0xbfea2fb4) will have an element with value 0x00414141 (if the machine uses **Little Endian**) or 0x41414100 (if the machine uses **Big Endian**).

For the case of 3 characters input value, register %ebp still holds the initial value, likewise with the block containing return address at address %ebp+4. Therefore, when procedure **GetBuff** finishes the execution, the procedure's stack frame will be removed (_Pop_) and pointer %ebp and %esp will be going back to the address holding the value of those pointer registers.

Then, what about this case?

> input : AAAAA<br />
> output : Segmentation Fault<br />

Why can be a Segmentation Fault? Let's look at the stack frame's condition.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/computerorg/howboworks02.png?raw=true" alt="How BO works" />

The character entry process into the allocated space for buff[4] is same as previous case, the only difference is because **gets()** doesn't check whether the length of inputted string exceeds the buffer's limit or not, so the excess input of character 'A' will be stored at the above block (in this case %ebp). We can see that block %ebp is corrupted by input character 'A' and '\0'. This thing surely makes pointer %ebp doesn't have the actual value and makes pointer %ebp can't be returned into **main()**'s stack frame to be base pointer. Indeed it's true that after procedure **GetBuff** finished, program will be going back to **main()** at the return address 0x08048427, yet the address's calculation on **main()** will be broken as the value of %ebp is not the old value anymore. For preventing the further negative effects, program suspends the execution and shows up Segmentation Fault message which means program can't execute normally.

Then, how about the case when we want to change the return address so that we can manipulate the program's flow?

> input : AAAAAAAA<br />
> output : Segmentation Fault<br />

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/computerorg/howboworks03.png?raw=true" alt="How BO works" />

From the condition of above stack frame, we can see that block %ebp is fully exchanged into 0x41414141 and block containing return address is exchanged also into 0x08048400 which means it's not the valid return address. After procedure **GetBuff** executing **gets()**, program won't be going back to **main()** for executing the next code, yet it goes to function/ procedure which is located at 0x08048400. The point is there's a probability that the function at that address contains any mallicious code.

Next, for the case of 12 input characters will be fully changing return address. 

Now we can see that return address after calling procedure **GetBuff** is 0x41414141 which is not a valid address.

For the next section our discussion will be focused on how to do this technique through a challenge called **Buffer Lab**.
