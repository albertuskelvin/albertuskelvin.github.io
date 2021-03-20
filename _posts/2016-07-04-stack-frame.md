---
title: 'Stack Frame'
date: 2016-07-04
permalink: /posts/2016/07/stack-frame/
tags:
  - computer organization and architecture
  - buffer overflow
  - stack frame
  - assembly
---

To discuss about this stack frame, we'll see from Assembly language point of view.

Basically, a program has functions which support the execution of that program. That functions run for being called by the previous function. This calling event which is usually called as **calling convention** has a concept related to stack. It means that when every function is called, then there'll be space formed which has these default elements, namely the function's **return address** and **argument values**. In addition, the space created can be also called as **stack frame**. This is an example of a **stack frame**.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/computerorg/stackframe00.png?raw=true" alt="Stack Frame" />

The picture above is a stack's illustration. The green part is a space created after calling function **Foo()**, where the elements are **return address** and **argument values** for function **Foo()**. 

FYI: Besides the general register like %eas, %ebx, and %edx, Assembly language also has pointer registers like %esp and %ebp. This %esp is also known as **stack pointer** that work especially for stack management.

Now pay attention, shortly after calling function **Foo()**, register %esp will refer to section contains **return address**.

Then, suppose there is an Assembly's instruction like this.

> pushl %ebp [1]<br />
> movl %esp, %ebp [2]

The first code means we enter the register %ebp into the stack. The concept of this entry is same as the push concept in the general stack, namely the newest element becomes the _Top_ of the stack. In this case, **ebp** becomes the _Top_ of the stack and it's placed after the block contains **return address**. Look also for this time, register %esp refers to the block containing **saved ebp**.

The second code means we move the address of register %ebp into the address of register %esp, where the current address of %esp is at the block contains **saved ebp**. The effect is now register %ebp refers to the block containing **saved ebp**.
 
Then, what can be concluded from that brief explanation? As we can see that the pointer %esp always refers to the block within the stack which has the lowest address (the block at the very bottom). So, every time we do **Push** action into the stack, automatically register %esp will move to that newly pushed block. The other conclusion is the second code which has function for moving register %ebp after the block containing **return address** is for becoming the **base pointer** or in other words as a _counter_ for doing the address calculation on a certain function's stack. This event is handled by register %ebp because register %esp is dynamic, which means it always be laying itself at the very bottom address, so that it's not effective enough if we want to make it as an address's marker on the stack. Because register %ebp is static, then we can define the address of register %ebp zero and the block above register %ebp worths multiple of 4, which means the block containing **return address** has higher address value than the address refered by register %ebp, namely **%ebp+4**. Also, block containing the first argument value worths **%ebp+8**, etc. For any blocks that located below register %ebp (which means has the lower address), then the block's address can be calculated with **%ebp-4** (one block below %ebp), **%ebp-8** (two blocks below %ebp), etc. In addition, the second code above will always be there when the concerned function is being called.

Then, what happen if we call another function from function **Foo()**? Here's the illustration.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/computerorg/stackframe01.jpg?raw=true" alt="Stack Frame" />

Suppose function **Foo()** calls function **Bang()**. So, for this case function **Foo()** can be refered as the **caller function** and function **Bang()** can be refered as **called function**. From the above illustration, we can make an analogy that function **Foo()** has a stack's space named _Caller Frame_ and function **Bang()** has a stack's space named _Current Frame_. What does it mean? It means that after function **Foo()** call function **Bang()**, then there'll be directly available a stack's space for function **Bang()** where the construction is same as function **Foo()**'s, where the default element is **return address** and **argument values** of function **Bang()**.

We can see that before function **Bang()** was called, register %esp is located at block containing **saved %ebp** on _Caller Frame_ after executing two assembly's code (see above explanation). After function **Foo()** calls function **Bang()**, there'll be a Calling process happens from function **Foo()** to function **Bang()**. Then the above two codes will be executed again where now register %esp is located at the block containg **saved %ebp** on function **Bang()**'s stack frame. Next, pointer %ebp has been moved to the same address with register %esp in purpose of making it as reference in block's address determination on stack frame.
 
So, it can be concluded that every time a Calling process happens, the action that will be conducted by Operating System is providing a stack frame for the called function. The location of that stack frame is after the Caller's stack frame. As we can see here that the newly created stack frame has lower address than Caller's stack frame.  


### Conclusion

The basic concept of Calling process is laid within stack frame, where a called function will have a frame containing important datas such as **argument values**, **variables**, and **return address**. A frame is controlled by two pointer registers, namely %ebp (frame base pointer) and %esp (stack pointer).

Register %ebp functions as **frame counter** that functions as address's marker of block on a frame. Every frame has a register %ebp. Here's the illustration (actually register %esp should be located at the very bottom. Please adjust by yourself).

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/computerorg/stackframe02.png?raw=true" alt="Stack Frame" />

For the next section, we'll see how buffer overflow technique works.
