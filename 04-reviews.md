---
layout: page
title: Reviews
---

Students are expected to have completed the assigned readings before class and
actively participate in discussions.
 
A review may be up to 600 words in length, and must contain the following sections:
* a short paragraph summarizing the problem and goal/contributions of paper
* a short paragraph summarizing the paper's methods and results
* a short paragraph giving your opinion of what is good and bad about the paper.

Grades will be be on the scale of Excellent(5), Satisfactory(4) and Unsatisfactory(2).  We may sample review grading to manage TA load (all students will be graded for the same paper reviews, but not all reviews will necessarily be graded).

Discussion before class with other students or in groups is encouraged, but
sharing reviews before completing and submitting is prohibited.

### Advice on reading papers
* [Efficient reading of papers in science and technology](http://john.regehr.org/reading_list/efficientReading.pdf)
* [How to read and evaluate technical papers](http://www.cs.kent.edu/~jmaletic/howtoread.html)
* [Task of the referee](http://jmlr.csail.mit.edu/reviewing-papers/smith-advice.pdf)

## Example Reviews

The below are some example acceptable reviews written by students in a similar class.  

### Example Review 1  ([Latency Lags Bandwidth](https://dl.acm.org/citation.cfm?id=1022596))
The paper focuses on the consistent trend of imbalance in the improvement of latency to the improvement of bandwidth(b/w), across different technology areas. The author explained some plausible reasons behind the imbalance and described a few commonly used techniques used to cope with this. 

Across technologies such as processor, memory, network or disk, it is observed that b/w has improved much faster than latency. More specifically, across these technologies; in the time taken by b/w to double, latency has only improved by around 30%. This trend holds true across technological milestones of varying time intervals, in all these areas. We know b/w can be expressed as product between frequency(inverse of latency) and amount of bits/data accessed/transferred in parallel. Now, with Moore's law holding good, transistors are becoming smaller and faster. Faster transistor helps improving both latency and b/w. Chips are having more of transistors as well, which helps b/w, as parallelism can be leveraged from larger number of transistors, but it hurts latency as it requires coordinating among large number of transistors. Moreover as wire delay does not scale well, larger chips make it longer to travel across chip, thus hurting latency. From the definition of b/w its clear that improvement(decrease) of latency directly helps improvement(increase) of b/w, but not vice versa. Improvement in parallelism in accessing data helps b/w, but not latency. Distance sets a lower limit on latency as we can travel no faster than light, whereas there is no such bound on the amount of parallelism that can be exploited to help b/w. Another reason may be the commercial motivation to improve b/w more, as probably its easier to convince customer with higher b/w than with lower latency. Sometimes improvement in b/w comes at the cost of latency as techniques helping b/w,e.g buffering or adding more modules to exploit parallelism, may hurt latency. S/w overhead may hurt latency more as its effect on b/w may be more easily amortized over larger messages. Author notices that there are techniques which try to cope with this. Among them caching is a widely used; where locality of reference is exploited to service many requests with smaller latency. Duplicating data also helps hiding some latency. Prediction is another way to overlap or hide latency. But author thought that this disparity may turn more ugly, as all of these techniques have already reached its' pinnacle. Author suggested that while designing a system, it will be wiser to pick a design that better leverages improvement in b/w than one which relies more on latency. 

The paper makes a nice observation about the disparity between b/w and latency and tries explain few reasons behind it. The author gives some valuable suggestions on coping with this in designing a new system. But the question that remained unanswered was why should we worry about this disparity in b/w and latency? It was not clear, why the author thought that finding a new trick to hide latency, will be tougher now. The author opined that b/w is redundant; but with advent of multicores chips, does it hold true in regard to memory b/w requirement?

### Example Review 2 ([Scaling to the End of Silicon with EDGE Architectures](http://www.cs.utexas.edu/users/cart/trips/publications/computer04.pdf))
This article, written by a research team at the University of Texas, is the description of an ISA called TRIPS. The goal of this new ISA is to improve on energy efficiency and performance. The group explains how their architecture works, as well as the advantages and drawbacks of their approach.

The TRIPS ISA is an instantiation of a class of instruction sets called Explicit Data Graph Execution(EDGE). The most significant characteristic of these instruction sets is that instructions communicate directly rather than through registers and cache. The producer of a piece of data will forward the data onto the instruction(s) that needs it, rather than just putting that data to a register. One of the major benefits, the group suggests, is that the processor doesn't have to create the dependency graph at run-time, because the compiler encodes that information into the instructions. The compiler is now doing work which better suits it, and the processor isn't wasting power with extra complex logic. The TRIPS group had planned to build a prototype chip of their proposed micro-architecture by 2005. This prototype includes two interconnected trips cores running at 500 mhz. Each core contains the components for 4 execution threads, each having 4 of it's own ALU's, and interleaved data and instruction caches. To prove that compilers for TRIPS are a reasonable goal, the group has modified the SCALE research compiler for their ISA. The compilation works in the usual fashion for the parsing and basic optimizations. The group added predication and hyperblock formation, two new performance enhancing features that work particularly well with their ISA. They also added a new back-end which will schedule the code to the physical ALU's, attempting to maximize concurrency and minimize routing distances for results. The group claims that the benefits of TRIPS, including fine grain parallelism, flexibility, and low power consumption, make it a worth ISA, even in the face of dramatic changes for industry.

I think this paper was really fascinating and eye opening. I had never considered such a different approach for a computer architecture. Yet it manages to leverage upon many designs which came before it. Their case is convincing, but somehow I doubt that their ISA, or any EDGE based ISA, will be able to replace x86 in the near future. Without a radical performance or power improvement, I think we'll be seeing the status quo for quite some time. 



