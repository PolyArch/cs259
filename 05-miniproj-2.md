---
layout: page
title: Mini-project 2
---

### Notes

* Due: May 17th, 11:59pm

* Teams up to 4 allowed.  (you can switch teams if needed)
 
### Goal 

Performance models are extremely useful to understand and optimize real
systems, as well as to explore novel designs.  
Relatively simple performance modeling modeling is possible for the
dense versions of DNN Layers layers you studied. There's no complex data structures,
and it's easy to reason about parallelism and reuse by breaking a large problem into
smaller versions of that problem. 

The goal of the assignment is to get familiar with modeling techniques, by using models 
to predict performance of accelerator hardware.  

### Overview of project

This is what you'll do in this project at a high level:

  0. Write a GPU kernel (e.g. convolution -- hopefully already done!)
  1. Develop a model of GPU performance for your kernels, parameterized by problem size and tile sizes.
  2. Validate that model by comparing to experimental runs.
  3. Repeat steps 1-2 (maybe step 0) to make your model more accurate, capturing more hardware behavior
  4. Hopefully learn something about parallelization and reuse modeling

In the next sections, I'll describe what I consider to be a default project, and a later section will
describe alternatives or tweaks that are acceptable.

### Default Project: Performance Model of Convolution on GPU

To get experience with modeling, we need to choose some interesting-enough
accelerator, and we need to make sure we can validate the model (at least to
some degree).  The default project is to model your own convolution kernel 
implementation, running on tetracosa on the TitanV GPU.  

The interestingness of this project is somewhat dependent on the optimization-level
of your code ... i.e. if your code is super slow, you might be able to create an accurate
model without much work :p.  
I think this is not actually a real problem -- my firm belief is that
by modeling the performance of your implementation, you'll see *why* your code is slow,
and you'll want to go and change it -- optimizing code is fun!  Feel free to tweak and
codesign your code with your model.

### Model Type

Models come in many forms, from mechanistic/analytical models to regression based models.
By default, I strongly suggest building mechanistic model.  A mechanistic model uses 
parameters and internal variables that have meaning corresponding to software and hardware behavior.
For example, you might have a parameter that is the L2 cache bandwidth, and this is used in
an equation to compute the operational-intensity on the L2 cache.

The advantage for us is that a mechanistic model can tell you something about the underlying
system, which is the purpose of this assignment.  
For example, by tuning the memory bandwidth parameter, you may observe whether and to what
degree the "peak" memory bandwidth can be obtained on a particular workload.

### Model inputs and outputs

**Inputs**: The input interface of the model should be quite simple.  At a minimum, your model should take as inputs:
* Problem Sizes (# input channels, filter sizes, etc.)
* Tile Parameters

To be clear, what I mean by tile parameters is the parameters that determine 
1. how the work is distributed across cores (GPU SMs), and 
2. how sub-volumes of tensors are stored in local memories.  

For example, you might have tile parameters that determine how to split the height/width/input-channels/output-channels to GPU cores.  Those same parameters might be used to determine what data is stored in scratchpad, etc.

**Outputs:** As for the output, please compute:
* Tera Operations per second (TOps)
* Achieved Operation Intensity -- i.e. ops / byte -- at each level of the memory hierarchy that you get reuse in.
* Is tile size valid? (e.g. false if tile exceeds scratchpad, if you are using scratchpad)
 
**Parameters:**I also urge you to use parameters for your model that have meaning:
The model should take as input some hardware specific inputs, including whatever you deem to be important.  E.g.:
* Bandwidth from memory, l2, l1 cache/scratchpad
* L1/L2/SPAD Capacities 
* Number of processors or floating point units (or tensor-core b/w if you were so brave)

Here's a [good resource](https://arxiv.org/pdf/1804.06826.pdf) for finding out about our GPU.  (we have a TitanV,
but the V100 is quite similar, besides the DRAM bandwidth)

Note that because convolution is so complex, I'm only requiring your model to be *valid* for the two
problem dimensions (conv1,conv2) from mini-proj1. I.e. for other problem sizes, your model is allowed
to produce not accurate results.  You may even have 2 somewhat different models for each problem size,
as the parallelization strategy and data-structure for each might be different.

### Model Form

The form of this model is up to you.  However, one approach is to perform a bottleneck
analysis: i.e. calculate the
compute bound time (execution time given a perfect memory system), and the
memory bound time (execution time given perfect buffer and compute bandwidth).
Then simply take the maximum of the above execution times. 
Note that you'll need to do this hierarchically for every level of the memory system: L1/SPAD, L2, DRAM,
thinking about the tile sizes that are mapped to each level.
To do this, you'll need to 
reason about what data fits in the buffer, given the tiling that you performed
(either implicitly in cache, or explicitly in scratchpad).  

One starting point is a ["roofline" model](https://en.wikipedia.org/wiki/Roofline_model), 
which is a highly optimistic model that you
essentially already created in mini-proj 1.  A "roofline model" just means: assume the
best possible performance given the ideal-operational-intensity.
Another way to say that is: compute DRAM-bound time (assuming perfect reuse) 
and compute-bound time (assuming perfect computation), and take the minimum.  

From the basic roofline model, I suggest adding models for other levels of memory hierarchy.
After that, consider capturing other sorts of microarchitectural behavior, possibly but not definitely 
 including, and definitely not limited to:

* kernel launch time
* data layout
* cache-line alignment in tile-size computations
* control instruction overhead
* scratchpad load time
* effect of memory/l2 latency (e.g. if you don't have many threads on an SM)

All of that said, don't worry about having a complex model.  Your goal is to create a model with
just enough complexity to have good accuracy.

### Validation

You will validate your model against the execution times from real hardware.  For the 
two problem sizes, you need to validate the performance as different tiling factors change.
Your report should demonstrate robustness of your model for at least a few different tiling
dimensions -- thus, try different combinations for tile height, width, input channels, output channels.

I'm not particular on the metric for validation, one option is mean absolute percentage error.  But use
the metric you like.

### Goal

Your baseline for this work is the very simple "roofline" model discussed above.  Your goal
is to get a better performance prediction than that -- hopefully much better!

### What to turn in?

For the report, please include the following, and make sure to use easy-to-read graphs in your analysis:

* Explanation:  Explain the model you created, including rationale and any required tuning.  Explain
any challenges faced, and how you addressed them (or would address them).  If your model
contains arbitrary constants, please explain why they exist or what they might mean.

* Validation:  For validation, explain the set of kernel parameters you used to validate your model.
Then present an analysis that shows the error across some range of kernels.  Explain what parameter
settings the model performs well or poorly on. 

* Architecture Insight: Based on your model, which hardware parameters should future GPUs change if they
want to perform better on your implementation of convolution? To answer this question,
perform a sensitivity analysis on some parameters.

In a separate attachment on canvas, please also include the model code in whatever language you choose.
Please do not zip the PDF and source together, as this makes it hard to read on Canvas.

### Alternatives

If you *really* insist on modeling matrix multiplication instead of convolution, I will allow it.  However, matrix multiply is much much simpler, so you need to validate your model for a WIDE range of matrix multiplication sizes, not just the two from mini-proj1. Follow-up with me if that doesn't make sense.

If you really want to model an accelerator *other* than the GPU, you can also do that.  But you need to have some way to validate it.  Please see me if you want to go this route.

### Frequently Asked Questions

* *Do we need to think about loop scheduling and tiling?*

I hope so.  I suggest performing analysis similar to the DianNao lecture.

* *Can I use regression / ML techniques?*

Even though this has been shown to work extremely well, I frankly don't think
you will learn much this way.  I only suggest this kind of approach if you can
train a model such that the parameters have a hardware meaning.

* *Does the model need to be valid for all parameters*

In general no. If you are doing convolution, it only has to be correct for the two input sizes
from hw1.  However, the broader an input space you can cover, the more fun you will have.



<!--
### References

For reference, I am providing the source code 
for [yet another loop schedule analyzer (yalsa)](https://github.com/PolyArch/yalsa).
It is a program which performs the loop analysis that we showed in class.  You can play
with loop scheduling order etc. It is basically a proof of concept for reuse analysis.
You can use the same approach, or use a much more focused model based on your own implementation.

Another option is to use existing tools like TimeLoop and MAERI.  I'm not sure
how well they will work for GPUs, but feel free to experiment!
-->
 
