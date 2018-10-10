---
layout: post
title: Homework 1
---

## Overview

[preliminary -- TA is testing]

Due at TBD

Prerequisites: Please do HW0 first.

The goal of this homeworks are:
1. Get familiar with the basics of gem5 usage
2. Practice thinking across stack (application/ISA/microarchitecture)

## Step 1: Download/Compile the microbenchmarks

This homeworks uses a few simple microbenchmarks, which you can download below.

git clone https://github.com/PolyArch/cs251a-microbench.git

Just follow the steps to compile within the repo.

## Step 2: Experiments 

In this step, you will change various aspects of the the CPU model or program,
and observe gem5's performance and microarchitectural behavior 
on the four given microbenchmarks (mm,lfsr,merge,seive). 

### Setup
* You may either use your own gem5 configuration script created during the tutorial
or you can just use se.py (as long as you know basically what se.py is doing).  If you use your own configuration
script, you will probably want to add parameters to the python file so that its easy to change
CPU model, frequency, etc.  

* To make life easier, you will probably want to write a Makefile and script to run
the experiments and save data.

* Each run of the simulator
will produce a statistics file as an output -- save the statistics files
generated from each run. Warning: by default, gem5 will write the output file
to the same folder (m5out) every time. Make sure to move your output file
before each subsequent run. 

* There is one small issue with the compilation command in the Learning
gem5 book: it will not compile the MinorCPU model by default.  Edit the
build_opts file for x86 (```gem5/build_opts/X86```) to add the "MinorCPU" to it.

* Just so that you don't have to do a cross product of all combinations of simulator/compiler
parameters, assume the default configuration is:
    *  CPU: DerivO3CPU (the OOO core)
    *  compiler: gcc -O3 
    *  Frequency: 1Ghz
    *  Memory: DDR3_1600_8x8
    *  2-level cache hierarchy (parameters from from tutorial)

* Also feel free to do other experiments if it helps you answer the questions.

### Experiment Definition
1. Vary the CPU model. 

   Change the CPU model between DerivO3CPU and MinorCPU (OOO and inorder cores).

2. Vary the Compiler aggressiveness

   Change the compiler between -O1 and -O3.

   (for fun, try ```mm``` with ```-O2 -ffast-math -ftree-vectorize``` -- what happens??)

3. Vary the Clock frequency
   Vary the CPU clock from 1 GHz to 4 GHz. 

   (note that changing the frequency doesn't affect gem5's pipeline depth, or any other microarchitecture aspect)

4. Vary the Main Memory  (Do this for both CPU models.)

   Vary the memory between: DDR3_1600_8x8 (the default), which models DDR3 memory and 
   HBM_1000_4H_1x64, which models High Bandwidth Memory, a high performance memory often used in GPUs and network devices.

5. Vary the cache
   Try turning off the l2 cache.

### Note: 
You should end up with 4 * 7 = 28 gem5 runs. Each run takes approximately 1 minute on the datasets given, so this should be about 1/2 hour of runtime if fully scripted.

## Step 3: Analysis
After collecting all of the data from the previous step, analyze the statistics, and answer the following questions.

1. What metric (and mean) should you use to compare the performance between
   different system configurations? Why?

2. For these three program properties: a) memory regularity, b) control regularity, and 
   c) locality, name one statistic in gem5 that might help you categorize a workload.
      (eg. for control regularity, it is inversely proportional to the 
       number of branch instructions ... but you can think of a better one).

3. For each microbenchmark, how would you describe/characterize its a) memory regularity, b)
   control regularity, c) locality?  

4. For each microbenchmark, which microarchitecture parameter is it most sensitive to?  (in other words, 
   what do you think the "bottleneck" is).

5. Pick a microbenchmark; propose one application enhancement, ISA enhancement, and microarchitecture
   enhancement that you think would be very effective for that microbenchmark.

6. Which CPU model is more sensitive to changing the memory technology? Why?

### What to Hand In
Please turn in to CCLE a zip file containing the config.ini and stats.txt from your runs in gem5.

Additionally, separate from the above archive, create a file named hw1.pdf
that contains the data and answers to analysis questions.

### How we will grade this:
40 points for completing the experiments, and 10 points question.
