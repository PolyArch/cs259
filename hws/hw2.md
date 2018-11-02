---
layout: post
title: Homework 2
---

## Overview

Due: end of day Friday November 9th

The goals of this homework are:
1. Get experience creating a new SimObject in gem5
2. Practice considering tradeoffs in cache design

## Step 1: Implement NMRU replacement policy

You can follow the tutorial here: [nmru tutorial]({{site.baseurl}}/hws/nmru-tut). This will walk you through how to create the NMRU policy.


## Step 2: Implement PLRU replacement policy

Follow similar steps as you did to implement NRU, but implement pseudo-LRU instead. Psuedo-LRU uses a binary tree to encode which blocks are less recently used than other blocks in the set. These [slides](https://ece752.ece.wisc.edu/lect11-cache-replacement.pdf) from Mikko Lipasti do a good job explaining the PLRU algorithm.

## Step 3: Architectural exploration

This time, the Entil CEO has tasked you with designing the L1 data cache of their new processor based on the out-of-order O3CPU. Due to it's memory intensity, Entil believe a better cache design could make their processor outperform the competition (AMM, Advanced Micro Machines if you're keeping track).

You can use the same benchmark suite as HW-1 from here: ```git clone https://github.com/PolyArch/cs251a-microbench.git```

You can choose from three replacement policies for the L1D cache: Random, NMRU, PLRU. As the associativity increases, the costs for NMRU and PLRU rises, whereas the cost for Random stays the same. Therefore, Random can be used with higher associativities than the other replacement policies. Additionally, because NMRU and PLRU must update the recently used bits in the tag they access, these policies limit the clock rate of the CPU. Note, the max clock of the O3 CPU is 2.3 GHz in this generation.

The constraints for these policies are summarized below.

|           |Random| NMRU	| PLRU |
|-----------|:----:|:----:|:-----:|
|Max assoc. |16    |8	    |8     |
|Lookup time|100 ps|500 ps|666 ps|

Clearly describe in a one page memo to the CEO of Entil, all of the configurations you simulated, the results of your simulations, and your overall conclusion of how to architect the L1 data cache. Additionally, answer the following specific questions for each workload:

Q1. Why does the 16-way set-associative cache perform better/worse/similar to the 8-way set-associative cache?

Q2. Why does Random/NMRU/PLRU/None perform better than the other replacement policies?

Q3. Is the cache replacement/associativity important for this workload, or are you only getting benefits from clock cycle? Explain why the cache architecture is important/unimportant.

### What to Hand In
Please turn in to CCLE a zip file containing the config.ini
and stats.txt for all the simulations in gem5.

Additionally, separate from the above archive, create a file named hw2.pdf
that contains the data and answers to analysis questions.

### How we will grade this:
40 points for completing the experiments, and 20 points question.
