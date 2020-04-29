---
layout: page
title: Mini-project 2
---

## Notes

Due: May 13th, 11:55pm
Teams: Yes, preferrably pairs

## Goal 

Analytical models of performance are extremely useful to get quick upper/lower
bound estimates.  They are even sufficient for understanding the performance of
real systems when such systems are simple enough that their behavior can be captured
abstractly with a few equations. 

Fortunately, deep neural networks are quite amenable to modeling, especially when
using data representations that do not depend on the data (ie. dense
matrices are easy, but sparse matrices are a little harder). The goal of the assignment 
is to get familiar with analytical modeling techniques, by using models to predict
performance of accelerator hardware.

## GPU Modeling for DNN Kernels

To get experience with modeling, we need to choose some interesting-enough
accelerator, and we need to make sure we can validate the model (at least to
some degree).  The default project is to model CUDNN on the V100 GPU, for 
some set of convolution and matrix-multiply kernels.  Alternative projects could
be to model other kernels on the GPU, or accelerators from the literature.  Alternatives
will be discussed later.

Models come in many forms, from mechanistic/analytical models to regression based models.
By default, I suggest building mechanistic model.  The advantages are 1. no prior statistics
background is required, and 2. the model can actually tell you something about the underlying
system.  For example, a model designed by hand may have the memory bandwidth of the
device as a paramter.  By tuning this paramter, you may actually observe whether and to what
degree the "peak" memory bandwidth can be obtained.  

## Model inputs and outputs

For hardware, I suggest at least the following as possible inputs:

The model should also include architecture specific inputs, if you deem them to be important.  Eg.:
* Bandwidth to memory
* Bandwidth to l2 Cache (and l1 cache?)
* Number of processors or floating point units (vector length?)
* Capacity of scratchpad?

While the model you create will inevitably be somewhat specific to the V100 architecture, keeping
the paramters abstract will enable you to adapt the model to accelerator architectures as well.

Similarly, the model should be robust to the problem size.  Ie. for matrix multiply it should
accept an [M,N,K].

The output of model should capture the performance, and the easiest way to do that is to
compute the execution time. This way, you will be able to compare
against the real performance data that you could gather from say CUDNN.  
Your goal is to minimize the error (you may use mean-squared error
or any other metric).  

## Model Form

The form of this model is up to you.  However, one approach is to calculate the
compute bound time (execution time given a perfect memory system), the buffer
bandwidth bound time (execution time given perfect compute and memory), and the
memory bound time (execution time given perfect buffer and compute bandwidth).
Then simply take the maximum of the above execution times.  You may need to
reason about what data fits in the buffer, given the tiling that you performed
(either implicitly in cache, or explicitly in scratchpad).  This will help you calculate the
relative bandwidth difference between the cache and memory.

## What to turn in?

For the report, please include the following, along with any supporting graphs or figures:

* Explanation:  Explain the chosen model, including rationale and any required tuning.  Explain
any challenges faced, and how you addressed them (or would address them).  If your model
contains arbitrary constants, please explain why they exist or what they might mean.

* Validation:  For validation, explain the set of kernel paramters you use to validate your model.
Then present an analysis that shows the error across some range of kernels.  Explain what parameter
settings the model performs well or poorly on. 

* Architecture Insight: Present an analysis of one or two paramters of your model that shed light
on some aspect of the hardware itself.  For example, does the peak computation throughput/memory
bandwidth suggested by your model align with the datasheet.  You might try varying the L2 cache 
size by 2x, or the compute throughput by 2x within the model to check the sensitivity.
Which hardware paramters should future GPUs change if they
want to perform better on convolution or matrix multiplication? 

In a separate attachment on CCLE, please also include the model code in whatever language you choose.

## Frequently Asked Questions

* *How well do we need to do?*

  A naive model would DRAM-bound time (assuming perfect reuse) 
and compute-bound time (assuming pefect computation), and take the minimum.  
Your approach should do better than that.

* *Do we need to think about loop scheduling and tiling?*

  My guess is that you will.  My suggested solution is to perform analysis similar to the DianNao
lecture -- you can code up this analysis pretty easily.  Then you could search for good
settings of tiling factors.  Is this the only way to do it .. no, but I don't know if higher-abstraction
techniques will work as well, maybe?

* *Can I use regression / ML techniques?*

  There's something poetic about using ML (especially DNNs) to predict DNN performance.  One thing I can
say for sure is that the performance is not linear with respect to inputs, and the only way to do well
with such a model is overfitting.  

  So ... you may try to use some ML model if you like, but 1. you have to use a non-linear model, and 2. you
should use proper statistical techniques to avoid overfitting (sufficient data, cross-validation, etc ... you
probably know more than I do).  As for the insight question in the report, 
you must still do your best to provide architecture
insights, or explain why that's not possible with your model. : )

* *Do I need to gather more data?*

  If you are using a mechanistic model (not regression), you might be able to use the data we already gathered?
But feel free to gather more from CUDNN etc.  If you are using regression/ML, you will almost certainly need
more data.

* *Is it okay if I use existing modeling tools?*

  If you can figure out how to use eg TimpeLoop for this project, that would be fascinating.  In general its okay.

* *Does the model need to be valid for all parameters*
  No, that would be really hard!  Do your best to be general, but where you can't be, figure out why.

* *Is it okay if the model is bad?*
  Sure, its just a mini-project after all.

## Alternative Models 

Lets say you are bored of GPUs, or you really want to model some particular accelerator for this project --
this can be an alternative.  The project above can more-or-less work in the same way for an accelerator
from the literature of your choice -- ie. the report structure and experiments will mostly be similar.

Keep in mind you need some way to validate the model.  This likely means you'll have to somehow have to
reproduce some results from, lets say, the original paper which proposes an accelerator.  You may choose
to validate final performance, or other secondary measures (memory bandwidth) if they are reported in the
paper.

Finally, note that you can't choose DianNao to model, since we more or less validated much of their data in
class.

 
