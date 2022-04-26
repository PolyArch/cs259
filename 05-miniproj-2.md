---
layout: page
title: Mini-project 2
---

### Notes

Due: May 12th, 11:59pm

Teams: 2 is recommended, 1 or 3 is okay.

### Goal 

Performance models are extremely useful to understand and optimize real
systems, as well as to explore novel designs.  Sometimes, performance modeling
is complex ...  but the computational kernels within DNN Layers are simple and
homogeneous enough to enable simple modeling.  This is especially true when
using data representations that do not depend on the data (i.e. dense matrices
are easy, but sparse matrices are a little harder).  Even simple analytical
models, composed of a few equations, can be effective.

The goal of the assignment is to get familiar with modeling techniques, by using models 
to predict performance of accelerator hardware.  

### Requirements

The only requirements for this assignment are to 
  1. Provide a model of some accelerator (GPU or other hardware), 
  2. Provide some validation of that model (either measurement 
     or a real system, simulation, or by analyzing published data).
  3. Your should try for a model that is better than the
      roofline model (bandwidth bound/compute bound).  
  4.  Your report should explain any insights learned from the modeling experience.  

The remainder of this document provides a suggested approach.

### GPU Modeling for DNN Kernels

To get experience with modeling, we need to choose some interesting-enough
accelerator, and we need to make sure we can validate the model (at least to
some degree).  The default project is to model your own matrix multiply or
convolution kernel implementations (you only need to do one kernel, but for 
different kernel dimensions), running on tetracosa on the TitanV GPU.  

The interestingness of this project is somewhat dependent on the optimization-level
of your code -- if your code is super optimized (unlikely :p) then a roofline model
should be sufficient.  If your code is extremely poor (e.g. only runs on one SM), then
many of the interesting microarchitecture interactions are not there.  A medium-performing
kernel is actually the most work to get right and the most interesting!  You should also
feel free to tweak the kernel as you go along.

If your performance is indeed too poor to be fun to model, then you could consider one
of the following two alternatives.  The first is to use the matrix multiplication routine
within the sample code.  This is a blocked implementation and uses shared memory -- not too
bad, not the best (perfect for analyzing!).

The final alternative is more out-there -- basically
the idea is to model the CUDNN implementation of matrix multiplication (e.g. using deep bench to run experiments).  
This is a bit more challenging, as it is basically blind modeling -- you do not know the implementation.
You'll have to guess how it's implemented (to predict how well the kernels can reuse memory etc.), then
check how well your model predicts performance, and try again.  The good thing is that it's highly-highly optimized.
Of course, you are also allowed to model other kernels on the GPU, or accelerators from the literature.


### Model Type

Models come in many forms, from mechanistic/analytical models to regression based models.
By default, I strongly suggest building mechanistic model.  A mechanistic model uses 
parameters and internal variables that have meaning corresponding to hardware behavior.
The advantage for us is that a mechanistic model can tell you something about the underlying
system, which is the purpose of this assignment.  
For example, a model designed by hand may have the memory bandwidth of the
device as a parameter.  By tuning this parameter, you may actually observe whether and to what
degree the "peak" memory bandwidth can be obtained.  

### Model inputs and outputs

The model should take as input some hardware specific inputs, including whatever you deem to be important.  E.g.:
* Bandwidth from memory, l2, l1 cache/scratchpad
* Capacity memory, l2, l1 cache/scratchpad
* Number of processors or floating point units (or tensor-core b/w if it's being used)

While the model you create will inevitably be somewhat specific to the 
architecture and kernel implementation;
keeping the parameters abstract will enable you to adapt the
model to accelerator architectures or other kernels.

Similarly, the model should be (at least somewhat) robust to the problem size. 
I.e. for matrix multiply it should accept an [M,N,K].

The output of model should capture the performance, and the easiest way to do that is to
compute the execution time. This way, you will be able to compare
against the real performance data that you could gather from say CUDNN. (with e.g. deepbench)
Your goal is to minimize the error (you may use mean-squared error or any other metric).  

### Model Form

The form of this model is up to you.  However, one approach is to perform a bottleneck
analysis: i.e. calculate the
compute bound time (execution time given a perfect memory system), the buffer
bandwidth bound time (execution time given perfect compute and memory), and the
memory bound time (execution time given perfect buffer and compute bandwidth).
Then simply take the maximum of the above execution times.  You may need to
reason about what data fits in the buffer, given the tiling that you performed
(either implicitly in cache, or explicitly in scratchpad).  This will help you calculate the
relative bandwidth difference between the cache and memory. Equivalently, you can
reason about how much reuse you can get, and use that to figure out the overall
bandwidths.

My suggestion for implementing this project is to first implement an optimistic roofline
model, then compare it to your measured results across a range of kernel input 
parameters (i.e. [M,N,K] for mat-mul).  Compute the hardware utilization (Gflops / max Gflops),
and observe where you have errors in the model, especially performance
cliffs that indicate something fundamentally different is happening in hardware.   Then
incrementally increase the complexity of the model by capturing different aspects of
microarchitecture.

Don't worry about having a complex model, just try to beat the simple roofline model by
a significant fraction.

### References

For reference, I am providing the source code 
for [yet another loop schedule analyzer (yalsa)](https://github.com/PolyArch/yalsa).
It is a program which performs the loop analysis that we showed in class.  You can play
with loop scheduling order etc. It is basically a proof of concept for reuse analysis.
You can use the same approach, or use a much more focused model based on your own implementation.

Another option is to use existing tools like TimeLoop and MAERI.  I'm not sure
how well they will work for GPUs, but feel free to experiment!

### What to turn in?

For the report, please include the following, and make sure to use easy-to-read graphs in your analysis:

* Explanation:  Explain the chosen model, including rationale and any required tuning.  Explain
any challenges faced, and how you addressed them (or would address them).  If your model
contains arbitrary constants, please explain why they exist or what they might mean.

* Validation:  For validation, explain the set of kernel parameters you use to validate your model.
Then present an analysis that shows the error across some range of kernels.  Explain what parameter
settings the model performs well or poorly on. 

* Architecture Insight: Present an analysis of one or two parameters of your model that shed light
on some aspect of the hardware itself.  For example, does the peak computation throughput/memory
bandwidth suggested by your model align with the datasheet.  You might try varying the L2 cache 
size by 2x, or the compute throughput by 2x within the model to check the sensitivity.
Which hardware parameters should future GPUs change if they
want to perform better on convolution or matrix multiplication? 

In a separate attachment on canvas, please also include the model code in whatever language you choose.
Please do not zip the PDF and source together, as this makes it hard to read on Canvas.

### Frequently Asked Questions

* *How well do we need to do?*

A naive roofline-like model would compute DRAM-bound time (assuming perfect reuse) 
and compute-bound time (assuming perfect computation), and take the minimum.  
Your approach should do better than that.

* *Do we need to think about loop scheduling and tiling?*

I hope so.  My suggested solution is to perform analysis similar to the DianNao
lecture -- you can code up this analysis pretty easily (this is what YALSA does in a kindof
general way).  Or you could use techniques inspired by MAERI or Timeloop.  Feel free to
do something adhoc.

One challenge is that if you are modeling CUDNN, you won't know the schedule
or the settings of loop tiling factors.  It may be necessary to search over some
possible factors to find good settings.

* *I'm using Deepbench for CUDNN, and sometimes it uses weird algorithms like 
Winograd, do we have to predict that?*

No, you don't have to.  Either just focus on matrix multiply, or force
 the algorithm which is used in CUDNN to be GEMM.

* *Can I use regression / ML techniques?*

Even though this has been shown to work extremely well, I frankly don't think
you will learn much this way.  I only suggest this kind of approach if you can
train a model such that the parameters have a hardware meaning.

* *Do I need to gather more data?*

Yes, please run your one kernel with lots of different input dimensions.

* *Does the model need to be valid for all parameters*
The broader an input space you can cover, the more impressive the results will be.
Tiny matrices will be dominated by kernel-launch overheads, so don't forget to
model that.  Do your best to be general, but where you can't be, figure out why.

* *Can this project have overlap with the final project?*
Sure if you like!

* *Can I model DianNao for this project?*
No, since we more or less did this in class, and YALSA already provides some sort of model.
 
