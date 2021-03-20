---
title: 'Java Heap and Stack Memory (Behind the Scene)'
date: 2017-01-11
permalink: /posts/2017/01/java-heap-stack-memory/
tags:
  - java
  - heap memory
  - stack memory
---

### Introduction

When we want to create an object in Java, what actually happens inside the memory? How does memory allocation work when our Java program is executed? Why does Java have Garbage Collection (GC) and what are the benefits of this feature?

Briefly explain, Java Runtime Memory consists two sections, namely Heap Memory and Stack Memory. Both spaces reside on computer's RAM and now we'll talk about memory allocation in terms of Java application.

-----

### Java Heap Memory

Heap memory is used to store any created objects in Java which is reffered by the reference variables from Stack memory. It also becomes a space for String pool which is used to store the String values in Java (every String values has a unique address inside the pool).

Talking about the size of memory, Heap memory has bigger size than Stack memory and we can manage the size using **-Xms** and **-Xmx** JVM option in which it defines the startup and maximum size respectively. When the heap memory is full or the space needed exceeds the maximum size, Java runtime will throw **java.lang.OutOfMemoryError: Java Heap Space** as an error message.

Furthermore, heap memory has an ability to keep the efficiency of the memory usage as the Garbage Collection (GC) runs on this space. GC will deallocate any objects that does not have references from Stack memory.

-----

### Java Stack Memory

Stack memory is used to store short-lived methods containing local primitive variables and reference variables. The processes inside this memory is simple in which whenever a method is called, a new block will be allocated. The principle of this block's allocation is LIFO (Last-In-First-Out). In addition, the allocated block can contain any local primitive variables and references variables that refer an object in the heap memory. When the method execution is finished, the block for the corresponding method will be removed (deallocated) and the stack pointer will focus on the below block.

Talking about the size of memory, Stack memory size is very less than Heap memory and we also can manage the size using **-Xss** argument. When the stack size is full, an error message **java.lang.StackOverFlowError** will be thrown.

-----

### Sample Code

We'll use this code as the base for the tutorial.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/java/java0_6.png?raw=true" alt="Sample Code" />

-----

### Local Primitive and Reference Variables

In the main() method, we can see that there are several variable declarations, such as **val0_int** which has integer as the type of data, **obj** which is an object of class Object, and **x** as an object of class sample.

Based on those variable declarations, there are two types of variables, namely **local primitive variable** and **reference variable**. **Local primitive variables** are the basic types of data, such as byte, short, int, float, boolean, and char. It stores primitive values and it is allocated in Stack memory. So, the actual composition for this **local primitive variables** is the pair of the name of variable and the value (var_name, var_value).

On the other hand, **reference variables** are any class that is instantiable which also includes arrays. The examples are String, Random, int[], String[], and any custom classes. It stores addresses and it is allocated in Stack memory in which the allocation has two sections, namely the pair of the variable name and the memory address is stored in Stack memory, whereas the pair of the memory address and the actual object is stored in Heap memory. So, the composition in the Stack memory is (var_name, mem_addr) and in the Heap memory is (mem_addr, object).

Therefore, based on the above explanation, we can say that **val0_int** is a **local primitive variable** in which the primitive value is 1, whereas **obj** and **x** are **reference variables** in which when we use **new** operator, Java Runtime will create the corresponding object in the Heap memory and return the address.

Here is a simple illustration of local primitive and reference variable in the Java Runtime Memory.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/java/java0_5.png?raw=true" alt="Local Primitive and Reference Variables" />

-----

### Behind the Scene

This is how memory allocation and deallocation work in terms of Java application:

<ul>
	<li>
		When main() method is found, Java Runtime will create a block in the Stack memory. This block contains local primitive variables defined in main() method, namely <b>val0_int</b>. Because we create an object in line 21 and 23, a reference variable will be stored in the block whereas the object will be stored in the heap memory. The reference variable in the Stack refers to the object in the Heap which means the reference variables store the memory address of the object in the Heap.
		
		<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/java/java0_0.png?raw=true" alt="Memory allocation for main() method" />

	</li>
	<li>
		When we call <b>func1</b> in line 24, a new block for <b>func1</b> will be created above the main() method's block and Java Runtime will reserve a space for <b>val1_int</b> within the block.
		
		<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/java/java0_1.png?raw=true" alt="Memory allocation for func1() method" />

	</li>
	<li>
		Now we're inside <b>func1</b> and we're calling <b>func2</b> from this method (line 7). The same action will be executed, that is a new block for <b>func2</b> will be created above the func1's block. However, there's a little difference for the memory allocation. It is because we have a String in this method in which we can not only store the local variable in Stack memory, but we have to store the String value in the Heap, spesifically in the <b>String Pool</b>. This String value will get a unique address in the pool and the variable stored in Stack memory will refer to this address. So, we can say that we'll have an element containing the pair of local variable and the address of String value in the String pool.
		
		<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/java/java0_2.png?raw=true" alt="Memory allocation for func2() method and String pool" />

	</li>
	<li>
		After allocating a space for each of the local primitive variables from <b>func2</b>, we can see that the <b>func2</b> instructions have already completed. It means that Java Runtime can deallocate the block reserved for <b>func2</b> and now the stack pointer focus on the <b>func1</b>'s block and executes the next instructions in <b>func1</b>. However, <b>func1</b> does not have any instructions again after calling <b>func2</b>, so its block will also be deallocated and now we focus on main() method.
		
		<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/java/java0_3.png?raw=true" alt="Memory deallocation for func1() and func2() method" />

	</li>
	<li>
		The next instruction of the main method is calling <b>func3</b> with an object as the parameter. When we get to this point, Java Runtime will create a new block for <b>func3</b> above the main's block. As the method receives an argument which is a reference variable, Java Runtime allocates a space for it in the Stack in which it refers to the object (obj) in the Heap. Moreover, Java Runtime will also allocate a space for String variable in the block in which it refers to the value resided in the String pool.
		
		<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/java/java0_4.png?raw=true" alt="Memory allocation for func3() method" />

	</li>
	<li>
		Afterwards, the overall instructions are completed and the stack memory will be deallocated. When the stack is free, the objects and the String pool in the Heap will not have any references variables that refer to them, so Garbage Collection (GC) will deallocate these elements and free the Heap memory.
	</li>
</ul>

-----

### References

<ul>
	<li><a href="https://www.youtube.com/watch?v=UcPuWY0wn3w">https://www.youtube.com/watch?v=UcPuWY0wn3w</a></li>
	<li>Personal Documents</li>
</ul>
