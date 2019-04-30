---
layout: page
title: Mini-project 2
---

Due: May 14th before class

## Summary: 

Analytical models of performance are extremely useful to get quick upper/lower
bound estimates.  They are even sufficient for understanding the performance of
real systems when such systems are simple enough that their behavior can be captured
abstractly with a few equations. 

Fortunately, deep neural networks are quite amenable to modeling, especially when
using data representations that do not depend on the data (ie. dense
matrices are easy, but sparse matrices are a little harder). The goal of the assignment 
is to get familiar with analytical modeling techniques, by using models to predict
performance of accelerator hardware.

## Part 1: GPU Modeling

In mini-project 1, you wrote a CUDA kernel, and tried some optimizations to improve the
performance on the V100 architecture.  Now, you will try to build an analytical model to 
capture the performance tradeoffs of *your implementation* on the V100, 
at least given a certain range of kernels.  (ie. your model will be specific to this GPU and
your implementation of the kernels as written for CUDA)

Specifically, the inputs for your model should include problem-specific variables:
* The dimensions of the matrices
* The dimensions of any tiling factors

The model should also include architecture specific inputs, if you deem them to be important.  Eg.:
* Bandwidth to memory
* Bandwidth to l2 Cache (and l1 cache?)
* Number of processors (vector length?)
* Capacity of scratchpad?

The output of the model should be the execution time, which you should compare against the real performance data you have already gathered (or you may gather new data).  Your goal is to minimize the error (you may use mean-squared error or any other metric).  Note that I don't expect you to use machine learning methods (eg. regression) to derive the parameters of the model, though you could if you wanted to.

The form of this model is up to you.  However, one approach is to calculate the
compute bound time (execution time given a perfect memory system), the buffer
bandwidth bound time (execution time given perfect compute and memory), and the
memory bound time (execution time given perfect buffer and compute bandwidth).
Then simply take the maximum of the above execution times.  You may need to
reason about what data fits in the buffer, given the tiling that you performed
(either implicitly in cache, or explicitly in scratchpad).  This will help you calculate the
relative bandwidth difference between the cache and memory.

Required Experiments:
1. Change one matrix size dimension over some range: how does your model compare to the GPU's measured performance on your kernel?
2. Change one tiling parameter over some range: how does your model compare to the GPU's measured performance on your kernel?
3. Use your model to test sensitivity to two architecture parameters.
   Specifically, you might try varying the L2 cache size by 2x, or the compute
by 2x.  Does this confirm of change the previous conclusions you made about the
architecture bottlenecks?

Questions:
* What was your modeling approach?  Any challenges?  (show the model)
* What was your model's average error for these workloads?
* Were there any arbitrary constants in your model?  What do you think they represent?

## Part 2: Accelerator Modeling

What about accelerators?  Can we model them even though we cannot validate them against a simulator?  Sure!

Here, your goal is to model some accelerator hardware for DNNs.  You may choose
one of the accelerators we've discussed before (eg. DianNao), or invent a new
accelerator that you find easy to model. : )

Depending on the accelerator you choose, it might be possible to validate using the
data from the paper.  This may or may not be a good idea given that most of the data
in papers is made up using (sometimes wrong) analytical/simulator models anyways.  

In addition to showing the model, choose one of the following experiments:
* Compare your performance against the data in the associated paper (if possible).  Was it similar?  Why or why not?
* Compare your projected accelerator's performance against the GPU for a range of input dimensions (similar to required experiment 1 from Part 1).  Does the accelerator relieve any bottlenecks? Or is it slowed down by bottlenecks not on the GPU.
* Use your model to test sensitivity to architecture parameters -- what is the main bottleneck of the accelerator?

### Caveats
* Your models may be specific to one "dataflow" (loop schedule)
* Your models don't need to be valid over the entire range of input/architecture parameters. That would be really hard!

### What to turn in: 
Turn in the answers to the questions, and any associated graphs.

### How to turn in:
I will make an assignment on CCLE where you should turn your report. 

 
