---
layout: page
title: Course Project
---

The final component of this course is a project; there is no final exam.  The
topic must be selected before exam 2, but I expect the majority of the work to happen afterwards.  This makes for about a 3-week project.

### Quick Facts

* Due Date: December 14th, 12pm (Noon)
* Subject: Anything you want, some suggestions are provided
* Work in teams: Yes, 2-3 Students
* Deliverables: Project Report + Source Code tarball

### Project Content

The goal of the project is to give you an opportunity to get hands-on
experience with architecture experimentation and maybe even research.  There
are two main options:

1. Hacksim:  Hack an architecture simulator to implement a new architecture or
   microarchitecture idea, and evaluate it on some meaningful workloads. Given
the limited time for this class, it is completely okay to re-implement an
existing technique.  The homeworks have prepared you for using
gem5<sup>1</sup>, so the natural tool to use will be gem5.  

2. Open-ended:  Propose a research idea and evaluate it using any means you
   like.  You are free to combine with ongoing research from your own studies,
or with another course, provided the scope of the project implementation 
submitted for this course is sufficient.  Please see me if this is unclear.

My guess is that most will opt for hacksim.

### Hacksim Ideas:

The following is a short list 

# Microarchitecture Ideas:
1. *Hack Branch Predictor:*  Branch prediction has been crucial to out-of-order
   processors success, and was an area of significant improvement for several
decades.  In this project, you could implement a new branch predictor (eg.
perceptron) within gem5, and compare to the state-of-the-art TAGE predictor.
🌶️🌶️

2. *gem5 Considered Harmful:*  Configure gem5 to be as similar as possible to a
   CPU and memory system that you have access to.  Write or gather some
microbenchmarks and figure out in what ways gem5 screws things up.  Bonus
points if you submit a patch to gem5 upstream.  🌶️🌶️

# Architecture Ideas:

1. *Tensor-Cores for CPUs:*  Propose a few extensions to x86 (or some
   easier-to-modify ISA like RISCV) for implementing faster
dense-linear-algebra.  Evaluate on matrix convolutions/multiplications from
Alexnet/VGG/Resnet.  Figure out whether the instruction extensions were worth
it, or whether existing SIMD is simply enough. 🌶️🌶️🌶️


# Report guidelines:




<sup>1</sup> I am kidding of course; nothing can actually prepare you or anyone
for modifying gem5.  



