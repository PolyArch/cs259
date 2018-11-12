---
layout: post
title: Homework 3
---

## Overview

Due: Before Thanksgiving (midnight Nov. 21st)

Actually Due: Monday Nov. 26th (but do you really want to do work over Thanksgiving??)

Expected Duration: 1-3 Hours

Purpose The purpose of this assignment is to help you become familiar with
gem5's language for describing instruction sets. Perhaps some of your projects may need to
involve adding or modifying the implementation of an instruction, so this could be useful.

Here you will go through the ISA
files in src/arch/x86/isa and understand how instructions are decoded and
broken down into micro-ops which are ultimately executed. To get a better
understanding, you will implement a missing x87 instruction (FSUBR -- "reverse subtract"). 

Note that x87 is a subset of the x86 ISA. This subset was originally added to
provide the floating point support, but is not used much now. To test your
implementation of the instruction, you will write a small program that will use
this particular instruction through inline assembly feature of GCC. The program
then would be simulated using gem5.

This is based on [gem5 101 homework 2](http://www.gem5.org/Gem5_101).

## Background on Decoding in Gem5

Remember in X86, instructions are decoded into micro-ops.  A regular instruction 
is typically referred to as a macro-op, while the smaller
parts are referred to as micro-ops. To implement an instruction in gem5, we
first provide the ISA decoder with the information on the macro-op, then we
provide an implementation of the macro-op in terms of micro-ops. Finally, we
implement the micro-ops that are not already implemented. We will carry out
these steps for the FSUBR instruction. Our implementation of FSUBR will mirror
that of FSUB, whose implementation is already available in gem5.  The fastest and
most appropriate way to do this assignment will
involve basically pattern matching and copy and paste without needing to fully understand
completely how each piece works, since gem5's decoder is somewhat
annoyingly complex. : )

There are many ways in which instructions are encoded in the x86 ISA. We will
focus on the x87 subset. [You can read more about instruction encoding in a
manual provided by
AMD](http://amd-dev.wpengine.netdna-cdn.com/wordpress/media/2008/10/24594_APM_v3.pdf).
Let's go through the file one_byte_opcodes.isa
(within ./gem5/src/arch/x86/isa/decoder) to understand how gem5 decodes instructions
from the x86 ISA. The file is written in a language designed specifically to
express instruction sets. The contents of the file are ultimately converted to
a C++ switch case. We first decode the top 5 bits of the opcode byte. There are
32 possible ways in which we can construct binary numbers using 5 bits. The
switch case lists all the possible cases.

All x87 instructions begin with an opcode byte in the range 0xD8 to 0xDF.
Therefore the topmost 5 bits always are 0x1B. For this case, we include the
file x87.isa. Let's jump to that file. In this file, we start with decoding the
bottom 3 bits. You can take a look at Table A-15 (page 443) in the [manual](http://amd-dev.wpengine.netdna-cdn.com/wordpress/media/2008/10/24594_APM_v3.pdf).
mentioned above for the instructions represented by different cases for the
bottom three bits. For example, FSUB and FSUBR are represented by opcodes 0xD8
and 0xDC, ie. the cases 0x0 and 0x4. To distinguish between the functionality
provided by these different opcodes for the same instruction, you will have to
understand the meaning of the ModRM field of the instruction. Read about it in
the manual linked to. 

## Hacking the Decoder for FSUBR

In the file x87.isa, you can check that we have FSUB
appearing the cases statements for 0x0 and 0x4. You can also observe that
FSUBR's implementation is missing (what we mean here, is that there is a function call
to ```fsubr()```, which is not implemented.  We'll need to replace that...).

1. As a first step, understand the difference between the two implementations required 
for the FSUBR instruction: one with opcode byte 0xD8 and the one with opcode byte 0xDC.
For this, you should read the description of FSUBR provided in the manual on
x87 instructions.  Note that there are equivalently two versions of FSUB: 
FSUB1 and FSUB2. Remember that x87 has a stack of operands st(0) is the head of the
stack...
*To be honest, even if you don't figure out the difference, you can still do the rest
of the assignment through pattern matching/trial and error.  *

2. Now, notice the three places in which the placeholder ```fsubr()``` appears in the file x87.isa.
   Replace the currently appearing statements with ones similar to those
specified for FSUB in the same file. By mentioning something like Inst::FSUBR1(...) or Inst::FSUBR2(...),
you are asking for that instruction to be used, instead of the placeholder,
which simply prints a warning that the instruction is not implemented.

3. Now, we need to provide an implementation of the FSUBR1 and FSUBR2 macro-op in terms of
   some micro-ops. Again we will mirror the implementation of the FSUB1 and FSUB2
instructions. Go to the directory src/arch/x86/isa/insts/x87/arithmetic/. This
directory holds the definition of different x87 arithmetic instructions in
terms of micro-ops. Take a look at how the FSUB instruction has been
implemented using micro-ops. FSUB1 and FSUB2 correspond to the two different
opcodes that we mentioned before. For each type, we have to provide three
different implementations: one that only uses registers (_R), one that reads one of
the operands from the memory (_M) using the address provided in the instruction and
the last one uses the address of the instruction pointer to read the operand (_P).
The micro-ops used for the three implementations should be straight forward to
understand (haha just kidding, at least the parts you need to modify are fairly straightforward).
  *Note that the way gem5's instruction parser works requires us to define all the three
  implentations for the FSUBR instruction. In all you should have six separate
  code blocks for FSUBR, like those specified for FSUB.*


4. Lastly, we need to provide an implementation of the micro-op subfp. You can
   check that the implementation is already available in the file:
src/arch/x86/isa/micro-ops/fpop.isa. So, you would not need to do anything for
this step.

5. Compile gem5 for x86 ISA to test that you did not make any mistakes in the
   implementation.
*There are many aspects of gem5's ISA language that we have not discussed at
all. Most of these aspects are not documented at all and one needs to figure
them out by going over the code in relevant files. (SAD!)*

6. Now, we will test the implementation of the FSUBR instruction. For this
   purpose, we will first write a short C program that reads a file with two
floating point numbers, subtracts them and prints the output. To make sure that
FSUBR is used for subtraction, we will explicitly use it using the inline
assembly feature of GCC.

Assembly instructions are written inline with the rest of the code using the
'asm' code block. This code block contains two portions: the instruction
portion, and the constraint portion. In instruction portion is a string
containing the assembly instructions. The GNU C compiler does not check this
string for correctness, so anything is allowed. The constraint portion
specifies what GCC can or cannot do with the input and output operands, what
registers or memory are affected by the instruction portion. There is
documentation available from GCC and other sources. Some recommend reading:

http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html

http://locklessinc.com/articles/gcc_asm/

Since it is sort of hard to understand how to correctly specify all the constraints related to assembly instructions, we are providing an implementation with some explanation.

```
  float subtract(float in1, float in2)
  {
    float ret = 0.0;
    asm ("fsubr %2, %0" : "=&t" (ret) : "%0" (in1), "u" (in2));
    return ret;
  }
```

The aim of the function is to subtract the two floating point input values and
return the result. To do so, we use FSUBR in the 'asm' code block. In our case
the instruction string is ```fsubr %2, %0```. After that we specify constraints on
the output operand, which we ask to be the variable ret. We then specify the
two input operands: in1 and in2. The letters ```t``` and ```u``` specify the top and
the second to top register of the x87 stack.

Ok, getting back to our main purpose. Use the function provided above in a C
program to test it out. 
Compile the program statically to generate a binary. You may also look at the
assembly code generated by the compiler using the -S flag.

Now simulate this program using gem5 (use any core model and parameter settings you like) -- if does a reverse subtract, you are done!


## What to Hand In to CCLE

Include a text file or PDF with three things:

1. Source code of your test program, and output from gem5 from running it.

2. A short paragraph on what your experiences or what  like/don't like about gem5 instruction decoding.

3. A patch file containing the changes made to
   src/arch/x86/isa/insts/x87/arithmetic/ and src/arch/x86/isa/decoder/x87.isa.
You can generate the patch using the command 'git diff src/arch/x86/isa >
/tmp/changes.patch'.

## How we will grade this:
80 points for completing the assignment, and 20 points for the question 2.
