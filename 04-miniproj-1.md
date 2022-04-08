---
layout: page
title: Mini-project 1
---

### Summary: 
The goal of the assignment is to get some experience working with data-parallel hardware and programming models, as well as the basic parallelization/locality aspects of deep neural network computations.   

Due: April 22rd, 11:59pm

I recommend doing this assignment in groups of 2 (pair programming can work well), but 3 or 1 is okay as well.  

You'll use Canvas's groups feature to turn in your assignment into Canvas.  

### Getting Started:

You should have gotten an email for setting up an account
on tetracosa.cs.ucla.edu. Please don't use more than 50GB, or we will run out of space! 

After setting up, you may want to look at some samples from NVIDIA; these can be setup by running:

```
bash /usr/local/cuda-10.1/bin/cuda-install-samples-10.1.sh .
cd NVIDIA_CUDA-10.1_Samples
make -j24
```

For a simple starting code, you can look at:

```
vim NVIDIA_CUDA-10.1_Samples/0_Simple/vectorAdd
```

You can run “deviceQuery” in the cuda samples under 1_Utilities to figure out various GPU parameters, which could help in terms of optimization or analysis of performance.

```
NVIDIA_CUDA-10.1_Samples/1_Utilities/deviceQuery/deviceQuery
```

In general, I recommend using the installed 10.1 version of cuda.  You may want to put "/usr/local/cuda-10.1/bin" in your PATH.

### Tutorials:

This is a [good practice tutorial to start
with](https://developer.nvidia.com/blog/even-easier-introduction-cuda/), and it
also has links to a number of other useful tutorials.

The official [programming guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html) 
is useful for in-depth explanations of all the features.

### Task:  
Implement and evaluate a CUDA version of a fully connected layer (ie. matrix-vector multiply) and 3D convolution kernel, with parameters from VGG:

* Conv1:  Nx=224 Ny=224 Kx=3  Ky=3  Ni=64   Nn=64 
* Conv2:  Nx=14  Ny=14  Kx=3  Ky=3  Ni=512 Nn=512
* Class1: Ni=25088 Nn=4096
* Class2: Ni=4096 Nn=1024

Please try for two different batch sizes; batch size = 1, and batch size = 16.  (Batch size 16 can be trivially implemented by executing batch size 1 several times, but you may not get the best performance doing that.)

A possible starting point are the kernels the [fp-diannao repo](https://github.com/PolyArch/fp-diannao), which are simple CPU implementations based on the DianNao data-layouts (but with fp datatypes).

You may do the following:
* Change the data layout
* Use any feature of the GPU you like (shared memory, tensor cores, inter-warp reductions)
* Parallelize using any strategy  (across batches or across/within feature maps)
* If you would prefer to use fp16/int16, that's fine too...

### What to turn in:

1. What was your basic parallelization strategy?  Are there limitations in scaling this strategy, or consequences for throughput or latency?

2. What is the execution time of each kernel? (and throughput if you use batching) 
 
3. What do you suspect is the limiting factor for performance of each kernel (compute,dram,scratchpad)?  Where does the implementation fall on the roofline model for your particular GPU?

    For this, you can/should measure different bandwidths using nvprof.  Eg:
  
    ```
    /usr/local/cuda-10.1/bin/nvprof -m dram_read_throughput ./vectorAdd
    ```
  
    A good description of other b/w measurements is here:
    https://stackoverflow.com/questions/37732735/nvprof-option-for-bandwidth

4. How does the implementation compare with CUDNN?  (use same batch size in CUDNN)
  
    Probably the easiest way to do this without understanding CUDNN is to use
    [DeepBench from Baidu](https://github.com/baidu-research/DeepBench)
    
    You should be able to modify the “code/kernels/conv_problems.h” or
    “gemm_problems.h” for whatever parameters you are using for batch size.  Then
    compile and run the tests in the  “code/nvidia” folder.  (all_reduce won’t
    compile, but this shouldn’t be a problem)
    
    I've gathered [some results](https://docs.google.com/spreadsheets/d/1LRDl_3oUGBdZlpaJv6JSguBBw9Mj6ut5QuQojfrapbs/edit#gid=0) on the V100 machine for interesting kernel sizes, and
    the results are here.  You may use these if you like.
    
5. What optimizations did you find most useful/least useful?

6. Turn in the cuda kernel as an attachment in the submission.  Don't zip everything together though, make sure source and PDF are turned in separately.


