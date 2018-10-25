---
layout: post
title: Homework 1 - My Answers
---

1. What metric (and mean) should you use to compare the performance between different system configurations? Why?

    * In most cases, comparing IPC would be sufficient, as the program is the same. Its also fairly insightful, as it says something about the efficiency of the pipeline. However, when you change the compiler flags, the the instruction counts change, so its possible for the IPC to go up, but the overall performance goes down. (when changing frequency, IPC can go in the wrong direction too) Execution time is therefore the best bet across all configurations.

    * As for the mean, geometric mean is right for the unitless speedup quantity. If you said IPC in the previous answer, then harmonic mean would be the best choice.
 
2. For these three program properties – a) memory regularity, b) control regularity, and c) locality – name one statistic (or combination of statistics) from stats.txt that might help you categorize a workload as having that property. (eg. for control regularity, it is inversely proportional to the number of branch instructions … but you can think of a better one).

    a. memory regularity: This was the hardest, as its hard to see directly. The most direct way would be if you have a prefetcher, and you could see how many of the program's accesses were prefetched. More indirectly memory regularity can be correlated with a variety of things spatial or temporal locality (cache miss ratio) or prevalence of SIMD instructions.

    b. control regularity: number of branches, branch misprediction rate, BMPKI

    c. locality: basically any cache statistic (MPKI, cache miss rate, etc)
 

3. For each microbenchmark, how would you describe/characterize its a) memory regularity, b) control regularity, c) locality?

    * LFSR: low memory regularity (random access), high control regularity, low locality
    
    * Merge: High-ish memory regularity (streaming access), low control regularity (dependences), mostly high, but shifts from temporal to spatial
    
    * MM: high memory regularity, high control regularity, high locality
    
    * Sieve: High-ish memory regularity (some changing pattern at an outer loop, but not too complex), medium-high control (data-dependent branch, but it is predictable), medium-high locality (worse as the program runs due to higher strides)

 

4. For each microbenchmark, explain which microarchitecture parameter is it most sensitive to (in other words, what do you think the “bottleneck” is), and justify using reasoning or statistics.

 
    * (I wouldn't necessarily limit the "Correct" answers to these:)
     
    
    * LFSR: Memory latency/bandwidth (large cache miss rates)
    
    * Merge: Mispredictions (need refactoring of control dependence to data dependence), for a large enough array bottleneck will be memory bandwidth
    
    * Sieve: Execution bound (eg. issue width) at current data sizes, but for larger N I believe it would become memory bandwidth bound as the locality would get worse
    
    * MM: Execution bound for any size array (for single core)

 

5. Pick a microbenchmark; propose one application enhancement, ISA enhancement, and microarchitecture enhancement that you think would be very effective for that microbenchmark.

    * Example MM:

    * Application Enhancement: Tiling for each level of the cache hierarchy. Manual loop unrolling/software-pipelineing etc. Loop interchange for stride-1 access patterns. * *

    * ISA Enhancement: Wider vector instructions, or -cool Tensor Core-like Instruction (2d multiply reduce -- https://devblogs.nvidia.com/programming-tensor-cores-cuda-9/)

    * Microarchitecture: Increase issue width, more FUs or deeper pipelining, higher-bandwidth to L1 cache, more lanes in vector unit, vector bypassing in SIMD unit 6.

 
6. Which CPU model is more sensitive to changing the memory technology? Why?

    *  The inorder core is supposed to be more sensitive because its memory latency is hard to hide (but OOO core has dynamic reordering to hide memory latency). But the experiments + gem5 model of HBM were probably not sufficient to show that.

