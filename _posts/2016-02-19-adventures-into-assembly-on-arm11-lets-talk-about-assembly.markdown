---
layout: post
title: Adventures into Assembly on ARM11
date: 2016-02-19 20:16:11.000000000 +00:00
---
Let’s talk about Assembly.

Infact, let’s talk about everything you need to begin writing assembly, inspecting your binary files, and understanding what's going on at a syscall level.

Primarily, we’re going to be focusing on **assembly language**, **as**, **ld**, **objdump** and **strace**.

This post isn’t groundbreaking, but, if you’re a newbie to assembly or under the hood low level operations of a Unix-like system, this may be a good read for you.

## AS — The GNU Assembler 
AS is the GNU families assembler. It’s designed to take the output from GCC and build object files for linking with LD into executable binaries (which we will talk about later).

GCC is a cross language compiler from GNU, it can compile many languages, including C, and produce many outputs, including; executable binaries, object files and even assembly. This post is focused primarily on assembly that we will write from scratch, and as such we won’t be spending much time looking at GCC.

When we execute AS, it will produce an ‘object file’, this is the machine code produced by the assembler (AS). It’s platform specific and designed to be ran through a linker in order to generate an executable binary.

Typical usage of AS:

```shell
$ as -o errcode.o errcode.s
```

.s is the file extension of an assembly source file.

## LD — The GNU Linker 

Computer programs are generally composed of one or more modules/parts, it is the linkers job to turn these into one consolidated output. We will use LD for building us an executable file that we can run to see our application working.

Typical usage of LD:

```shell
$ ld -o errcode errcode.o
```

## Assembly 

At this stage we know the two foundations to eventually create an executable from some assembly code, but what about writing some actual assembly? This is the interesting part, as most of us know, assembly differs architecture to architecture, processor to processor and manufacturer to manufacturer. Whilst tools like AS try to make assembly consistent cross architectures, more low level details such as register locations are all independent.

In this post I’m focusing on ARM11, specifically the Raspberry Pi Zero. However, with enough Google-foo and manual reading, you should be able to port this over to any machine type.

All processors should have a freely available Technical Reference Manual, the Raspberry Pi Zero TRM is here: [http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.set.arm11/index.html](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.set.arm11/index.html) (Core ARM1176JZ).

For this post, we’re only really interested in the core register set and the ability to load a register with an address, I won’t be going into any indepth assembly language.

Let’s write some assembly that does nothing but simply returns a 204 Unix exit code.

If you’re not familiar with Unix, programs may close with a code which represents the status of that program, 0 is typically successful where anything else is an error, we will write a program than returns a 204 exit code.

To put this into context, continuous integration systems such as Jenkins may use this exit code to detect a successful or failed build.

```shell
$ vim errcode.s
```

Application code:

```shell
.text

.global _start
_start:
  mov r0, $204 /* Set exit code to 204 in register r0 */ 
  mov r7, $1 /* Load syscall number into register r7 (1 for exit) */
  swi $0 /* Invoke the system call */
```

Assembly programs can be split up into three sections; **.text** being the main application code, **.data** for declaring and initialising any constants and **.bss** to declare variables, for this example we simply have a .text section.

We must then define our entry point into our assembly code so AS knows where to look to begin compiling.

```shell
.text
.global _start
```

Next we must actually define the entry block to the application.

```shell
_start:
```

Now for the interesting part. Next we must begin our work with the CPUs registers. Simply put, a register is a holding place within the CPU for data and instructions. We move things into registers to perform calculations on them, execute calls to the Linux kernel and many other operations during a program's execution lifecycle.

In order to throw an exit code from our application, we must first understand that we need to interact with the Kernel, we do so through a software interrupt, through this interrupt we must ask the Kernel to exit our program and throw a particular status code.

There is a Linux man page that documents this function and the arguments it accepts: [http://man7.org/linux/man-pages/man3/exit.3.html](http://man7.org/linux/man-pages/man3/exit.3.html).

You can see it accepts a single argument, the status, or in other words, exit code.

In a high level programming language like C, this could probably all be achieved in one single line of code, however, assembly is far more low level than that, so we must deal with placing things into registers and invoking a call to the Kernel manually.

```shell
mov r0, $204 /* Set exit code to 204 in register r0 */
mov r7, $1 /* Load syscall number into register r7 (1 for exit) */
swi $0 /* Invoke the system call */
```

Assemble and link it to see the results:

```shell
$ as -o errcode.o errcode.s
$ ln -o errcode errcode.o
$ ./errcode
$ echo $? => 204
```

`echo $?` simply returns the status of the last command executed on the command line, here we can see 204 returned, the number we expected.

Here is a quick guide to syscall numbers in Linux: [http://asm.sourceforge.net/syscall.html](http://asm.sourceforge.net/syscall.html).

Breakdown:

* .text — The main application code
* .global _start — The entry point to the * application for AS to assemble
* start: — Begin the entry point
* mov — Copy data from one location to another
* swi — Perform a software interrupt

## Inspection 

The most exciting thing about developing any program at such a low level is inspecting every part of the application. Here we’re going to disassemble our binary file and see what the assembler and linker made of our assembly code.

### objdump — The GNU binary inspector 

objdump let’s us disassemble most binary files. We can use it to inspect what our linkers transformed our object code into at assembly/memory level.

Let’s disassemble our errcode binary.

```shell
$ objdump -ds errcode
```

This output may differ slightly system to system, but this can be used as a reference:

```shell
errcode: file format elf32-littlearm

Contents of section .text:

  10054 cc00a0e3 0170a0e3 000000ef …..p……

Contents of section .ARM.attributes:

  0000 41130000 00616561 62690001 09000000 A….aeabi……

  0010 06010801 ….

Disassembly of section .text:

  00010054 <_start>:

    10054: e3a000cc mov r0, #204 ; 0xcc

    10058: e3a07001 mov r7, #1

    1005c: ef000000 svc 0x00000000
```

We can see in this dump, the file format of the binary if elf32-littlearm, indicating this binary will only run on littlearm processor types.

Further we can see memory addresses which have been allocated for use in the .text section of our application. You can see 10054 allocated in *‘Contents of section .text’* and then re-referenced in *‘Disassembly of section .text’*, indicating this is the memory location of the ‘start’ block of the program.

Whilst the information in this file is not useful right now, it can become useful later for debugging applications and figuring out what's going on under the hood of your application in memory or otherwise.

Why not write the equivalent program in C, compile it, objdump the executable GCC provides you with and compare? Compare file sizes and differences between the C and assembly versions, get a feel for what C does and how you’ve simplified it in assembly. Afterall, many C functions are just wrappers around assembly and system calls.

```shell
#include <stdlib.h>

int main(){
     exit(204);
     return 0;
}
```

GCC compilation of C:

```shell
$ gcc -o errcode.c errcode
$ objdump -ds errcode
```

### strace — Trace system calls and signals 

We know we’ve written an application that performs a syscall to the Kernel to request sys_exit ($1) as a software interrupt to end the application and return a custom exit code.

We also know there is a Linux man page that documents this function and the arguments it accepts.

From reading this man page we can see that the function conforms to the following signature:

```shell
void exit(int status);
```

Let’s debug that and actually verify our assembly setup performs the underlying system call we require.

As a recap, our assembly setup is as below (with comments omitted) :

```shell
mov r0, $204
mov r7, $1
swi $0
```

strace is the go to tool on many Linux systems to trace system calls (syscalls). It shows what low level interaction a program is having with the Kernel and how that program is going about it’s business.

```shell
strace ./errcode
```

The output of this program returns:

```shell
execve(“./errcode”, [“./errcode”], [/* 17 vars */]) = 0

exit(204) = ?

+++ exited with 204 +++
```

There you have it, clearly the assembly is calling the exit() function on the Linux kernel to request the program is terminated.

Take your analysis and learning further with the below example which takes advantage of constants and memory addresses, it’s a simple ‘Hello World’ program which writes to stdout:

```shell
.data
msg: 
     .ascii “Hello World\n” 

.text
.globl _start
_start: 
     mov r0, $1 /* Move 1 into r0 register for syscall (file descriptor stdout) */ 
     ldr r1, =msg /* Load the 32bit msg constant into the r1 register */ 
     mov r2, $12 /* Load size of 32bit constant into register r2 */     
     mov r7, $4 /* Load syscall number into register r7 (4 for write) */ 
     swi $0 /* Invoke the system call */ 

     mov r0, $0 /* Set exit code to 0 in register r0 */ 
     mov r7, $1 /* Load syscall number into register r7 (1 for exit) */
     swi $0 /* Invoke the system call */

Compare to the equivalent C program, examine binary sizes and the disassembled binary output.
```

Happy hacking.

Visit my [website](https://www.jacobclark.xyz), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
