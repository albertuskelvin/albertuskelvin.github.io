---
title: 'Buffer Lab'
date: 2016-07-06
permalink: /posts/2016/07/buffer-lab/
tags:
  - computer organization and architecture
  - buffer lab
---

### Purpose:

Designed to help us in understanding stack frame, code injection, and usage of GDB (GNU Debugger).

### General explanation:

Buffer Lab has 4 levels starting from level 0 till level 3. Each level has different difficulty level and test one application of buffer overflow technique, where such applications including changing the return address of a functio/ procedure (redirecting), changing the variables value, changing the return value of a function/ procedure, or configuring the stack frame.
 
### Technical

> Buffer Lab will be given in a ZIP format containing 3 files, namely **bufbomb**, **makecookie**, and **hex2raw**.<br />

> File **bufbomb** is an executable file where your main activity will be done on this file. You are allowed to use GDB for doing any exploration to this file.<br />

> File **makecookie** is a file for generating cookie (a unique number represented by hexadecimal) regarding your username.<br />

> File **hex2raw** is used to convert string or char into hexadecimal representation so that it can be used for configuring stack frame.<br />

> To see all the variables within file **bufbomb**, use this command: **info var**<br />

> To see all the functions/ procedures within file **bufbomb**, use this command:<br />
> **objdump -d bufbomb | less**.<br />
> Press button **F** to go forward to the next function/ procedure and press button **B** to go backward to the previous function/ procedure.<br />


### Initial Configuration

Let we initiate the first configuration to record any important information we'll use later, such as the value of called function's address and certain variable's address.
 
FYI, my file **bufbomb** contains important functions such as **ah_shoo()**, **ah_choo()**, and few of procedures like **good_night()**, **shooting_star()**, and **annyeong()**.
 
FYI, I use Ubuntu 64 bit and Little Endian machine.

So, let's go!

First, compress **buflab-handout.tar** file which has 3 files like I've explained before. Save it on your own working directory.

Open your terminal and go to the directory of **bufbomb** file. Then, type this command:

> **Type this on your terminal:**<br />
> objdump -d bufbomb | less

That command will show all address from each functions/ procedures within **bufbomb** file. Try to find the address for each functions and procedures like I've said before.

08048b50 < good_night >: address **good_night()**

Address | Hexa code | Operation | Elements in operation
--- | --- | --- | ---
8048b50: | 83 ec 1c | sub | $0x1c, %esp
8048b53: | c7 04 24 64 a0 04 08 | movl | $0x804a064, (%esp)
8048b5a: | e8 b1 fc ff ff | call | 8048810 <puts@plt>
8048b5f: | c7 04 24 00 00 00 00 | movl | $0x0, (%esp)
8048b66: | e8 06 06 00 00 | call | 8049171 <validate>
8048b6b: | c7 04 24 00 00 00 00 | movl | $0x0, (%esp)
8048b72: | e8 d9 fc ff ff | call | 8048850 <exit@plt>

08048b77 < shooting_star >: address **shooting_star()**

Address | Hexa code | Operation | Elements in operation
--- | --- | --- | ---
8048b77:   |   83 ec 1c             |  sub  | $0x1c, %esp
8048b7a:   |   8b 54 24 20          |  mov  | 0x20(%esp), %edx
8048b7e:   |   8b 44 24 28          |  mov  | 0x28(%esp), %eax
8048b82:   |   3b 05 0c d1 04 08    |  cmp  | 0x804d10c, %eax
8048b88:   |   75 45                |  jne  | 8048bcf <shooting_star+0x58>
8048b8a:   |   80 fa 59             |  cmp  | $0x59, %dl
8048b8d:   |   75 26                |  jne  | 8048bb5 <shooting_star+0x3e>
8048b8f:   |   89 44 24 08          |  mov  | %eax, 0x8(%esp)
8048b93:   |   c7 44 24 04 8c a0 04 |  movl | $0x804a08c, 0x4(%esp)

08048bf3 < annyeong >: address **annyeong()**

Address | Hexa code | Operation | Elements in operation
--- | --- | --- | ---
8048bf3:   |   83 ec 1c             |  sub  | $0x1c, %esp
8048bf6:   |   a1 04 d1 04 08       |  mov  | 0x804d104, %eax
8048bfb:   |   3b 05 0c d1 04 08    |  cmp  | 0x804d10c, %eax
8048c01:   |   75 2f                |  jne  | 8048c32 <annyeong+0x3f>
8048c03:   |   83 3d 00 d1 04 08 ff |  cmpl | $0xffffffff, 0x804d100
8048c0a:   |   75 26                |  jne  | 8048c32 <annyeong+0x3f>
8048c0c:   |   89 44 24 08          |  mov  | %eax, 0x8(%esp)
8048c10:   |   c7 44 24 04 37 a2 04 |  movl | $0x804a237, 0x4(%esp)
8048c17:   |   08					|		|
8048c18:   |   c7 04 24 01 00 00 00 |  movl | $0x1, (%esp)

08048cec < ah_choo >: address **ah_choo()**

Address | Hexa code | Operation | Elements in operation
--- | --- | --- | ---
8048cec:   |   83 ec 4c             |  sub  | $0x4c, %esp
8048cef:   |   8d 44 24 18          |  lea  | 0x18(%esp), %eax
8048cf3:   |   89 04 24             |  mov  | %eax, (%esp)
8048cf6:   |   e8 5b ff ff ff       |  call | 8048c56 <Gets>
8048cfb:   |   b8 01 00 00 00       |  mov  | $0x1, %eax
8048d00:   |   83 c4 4c             |  add  | $0x4c, %esp
8048d03:   |   c3                   |  ret  |

08048d04 < ah_shoo >: address **ah_shoo()**

Address | Hexa code | Operation | Elements in operation
--- | --- | --- | ---
8048d04:   |   83 ec 0c             |  sub  | $0xc, %esp
8048d07:   |   e8 e0 ff ff ff       |  call | 8048cec <ah_choo>
8048d0c:   |   83 c4 0c             |  add  | $0xc, %esp
8048d0f:   |   c3                   |  ret  |

After finding the address for each function and procedure, try to look at this procedure **test()** in C language which will be our starting point of Buffer Overflow attack. 

> void test() {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int val;<br />

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/* Put canary on stack to detect possible corruption */<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;volatile int local = uniqueval();<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;val = ah_choo();<br />

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/* Check for corrupted stack */<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if (local != uniqueval()) {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("NO: Stack terkorupsi\n");<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} else if (val == cookie) {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("OK: ah_choo 0x%x\n", val);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;validate(3);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} else {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("NO: ah_choo 0x%x\n", val);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br />
> }<br />

And this is function **ah_choo()**.

> int ah_choo() {<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;char buf[32];<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gets(buf);<br />
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return 1;<br />
> }<br />

Next stop: Level 0!
