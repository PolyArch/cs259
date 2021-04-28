---
layout: page
title: Mini-project 2
---

### Notes

Due: May 14th, 11:59pm

Teams: 2 is recommended, 1 or 3 is okay.

You'll use CCLE groups feature to turn in your assignment into CCLE.


### Goal 

Performance models are extremely useful to understand and optimize real systems, 
as well as to explore novel designs.  Sometimes, performance modeling is complex ...
but fortunately, the layers of deep neural networks are quite amenable to modeling. 
This is especially true when
using data representations that do not depend on the data (ie. dense
matrices are easy, but sparse matrices are a little harder). 
Even simple analytical models, composed of a few equations, can be effective.

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
some degree).  The default project is to model CUDNN on the TitanV GPU, for 
some set of convolution or matrix-multiply kernels.  Alternative projects could
be to model other kernels on the GPU, or accelerators from the literature.  
You may also model *your own* CUDA implementations, if they are fast enough to
be interesting. 
 
Models come in many forms, from mechanistic/analytical models to regression based models.
By default, I suggest building mechanistic model.  The advantages are 1. no prior statistics
background is required, and 2. the model can tell you something about the underlying
system.  For example, a model designed by hand may have the memory bandwidth of the
device as a parameter.  By tuning this parameter, you may actually observe whether and to what
degree the "peak" memory bandwidth can be obtained.  

### Model inputs and outputs

For hardware, I suggest at least the following as possible inputs:

The model should also include architecture specific inputs, if you deem them to be important.  Eg.:
* Bandwidth from memory, l2, l1 cache/scratchpad
* Capacity memory, l2, l1 cache/scratchpad
* Number of processors or floating point units (or tensor-core b/w if it's being used)

While the model you create will inevitably be somewhat specific to the 
architecture, keeping the parameters abstract will enable you to adapt the
model to accelerator architectures as well.

Similarly, the model should be (at least somewhat) robust to the problem size. 
Ie. for matrix multiply it should accept an [M,N,K].

The output of model should capture the performance, and the easiest way to do that is to
compute the execution time. This way, you will be able to compare
against the real performance data that you could gather from say CUDNN. (with eg. deepbench)
Your goal is to minimize the error (you may use mean-squared error or any other metric).  

### Model Form

The form of this model is up to you.  However, one approach is to calculate the
compute bound time (execution time given a perfect memory system), the buffer
bandwidth bound time (execution time given perfect compute and memory), and the
memory bound time (execution time given perfect buffer and compute bandwidth).
Then simply take the maximum of the above execution times.  You may need to
reason about what data fits in the buffer, given the tiling that you performed
(either implicitly in cache, or explicitly in scratchpad).  This will help you calculate the
relative bandwidth difference between the cache and memory.


For reference, I am providing a the source code 
for [yet another loop schedule analyzer (yalsa)](https://github.com/PolyArch/yalsa).
It is a program which performs the loop analysis that we showed in class.  You can play
with loop scheduling order etc. It is basically a
 proof of concept for the type of analysis that might be necessary to get
good accuracy.  

Another option is to use existing tools like TimeLoop and MAERI.  I'm not sure
how well they will work for GPUs, but feel free to experiment!

### What to turn in?

For the report, please include the following, along with any supporting graphs or figures:

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

In a separate attachment on CCLE, please also include the model code in whatever language you choose.

### Frequently Asked Questions

* *How well do we need to do?*

  A naive model would DRAM-bound time (assuming perfect reuse) 
and compute-bound time (assuming perfect computation), and take the minimum.  
Your approach should do better than that.

* *Can I model my own GPU kernel implementations instead of CUDNN?*
Yes of course!  And you may even learn quite a bit this way, because the model
should explain why your performance is not reaching the peak.

HOWEVER: I would caution that you should only do this if your kernel
implementations are pretty good -- perhaps within 10x of min(CUDNN,roofline).
If your kernel performance is very low, there's a danger 
that either 1. you are hitting some difficult-to-model bottleneck
(eg. your data-layout is not great, causing many outstanding memory requests), 
or 2. it will be too easy because the performance is low
and doesn't scale in an interesting way.  Feel free to use your discretion.

* *Do we need to think about loop scheduling and tiling?*

I hope so.  My suggested solution is to perform analysis similar to the DianNao
lecture -- you can code up this analysis pretty easily (this is what YALSA does). Or you could use techniques inspired by MAERI or Timeloop.

One challenge is that if you are modeling CUDNN, you won't know the schedule
or the settings of loop tiling factors.  It may be necessary to search over some
possible factors to find good settings.

* *I'm using Deepbench for CUDNN, and sometimes it uses weird algorithms like 
Winograd, do we have to predict that?*

  No, you don't have to.  I suggest forcing the algorithm which is used in CUDNN to be GEMM.

* *Can I use regression / ML techniques?*

There's something poetic about using ML techniques to predict DNN performance.
So you may try to use some ML model if you like, but 1. I would avoid using
linear models, and they will likely be insufficient.
2. you should use proper statistical techniques to avoid overfitting
   (sufficient data, cross-validation, etc ...).  As for the insight question
in the report, you could do your best to provide some architecture insight, or
explain why that's not possible with your model. : )

* *Do I need to gather more data?*

It's very likely -- I suggest using deepbench for running CUDNN (or just running
with more parameters if you are using your own code).  
If you are using regression/ML, you will almost certainly need more data.

* *Does the model need to be valid for all parameters*
  No, that would be really hard!  Do your best to be general, 
but where you can't be, figure out why.

* *Can this project have overlap with the final project?*
Sure, that's not a bad idea!

* *Can I model DianNao for this project?*
No, since we more or less did this in class, and YALSA already provides some sort of model.
 

