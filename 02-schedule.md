---
layout: page
title: Schedule
---

This schedule is subject to change.  The arrow indicate the speculative head of the schedule reorder buffer (please do not complete reviews on/after the head).


| Date                 | Topic                                        | Readings                                                                                                                                                                        | Review                                                                                                                            |
|----------------------|----------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| Sept 27th            | Intro [slides][lec1]                                   | [Cramming More Components Onto Integrated Circuits][moore65]                                                                                                                    |                                                                                                                                   |
| Oct 2nd              | ISA Basics                                   | [Instructions Sets and Beyond: Computers, Complexity, and Concurrency. IEEE Computer, September 1985.][isas85], [Compilers and Computer Architecture(RISC/CISC Debate)][wulf81] |                                                                                                                                   |
| Oct 4th $\leftarrow$ | Pipelining                                   | [Synth Lec: Processor Microarchitecture][synth-proc-micro],[Alternative Implementations of Two-Level Adaptive Branch Prediction][two-level-bp]                                  | [Power Struggles: Revisiting the RISC vs. CISC Debate on Contemporary ARM and x86 Architectures][power-struggles]                 |
| Oct 9th              | Multi-Issue                                  |                                                                                                                                                                                 | [Design Tradeoffs for the Alpha EV8 Conditional Branch Predictor][seznec_alphaev8]                                                |
| Oct 11th             | OOO Basics                                   |                                                                                                                                                                                 |                                                                                                                                   |
| Oct 16th             | Modern OOO                                   | [Implementing Precise Interrupts in Pipelined Processors][smith-precise]                                                                                                        | [Instruction Issue Logic for High-Performance, Interruptible, Multiple Functional Unit, Pipelined Computers][sohi-issue]          |
| Oct 18th             | Instruction Flow Optimizations               | [Understanding Scheduling Replay Schemes][spec-sched-replay]                                                                                                                    | [Continual Flow Pipelines][cfp]                                                                                                   |
| Oct 23rd             | Memory Dataflow                              |                                                                                                                                                                                 | [Fire-and-Forget: Load/Store Scheduling with No Store Queue at All][fnf]                                                          |
| Oct 25th             | Exam 1 (due 26th)                            |                                                                                                                                                                                 |                                                                                                                                   |
| Oct 30th             | Cache Basics                                 | [Multi-Core Cache Hierarchies][synth-cache]                                                                                                                                     | [Improving Direct-Mapped Cache Performance by the Addition of a Small Fully-Associative Cache and Prefetch Buffers][victim-cache] |
| Nov 1st              | Cache Optimizations                          | [Synthesis Lecture: A Primer on Prefetching][synth-prefetch]                                                                                                                    | [Criticality Aware Tiered Cache Hierarchy: A Fundamental Relook at Multi-Level Cache Hierarchies(ISCA 2018)][crit-aware-cache]    |
| Nov 6th              | Virtual Memory                               | [Virtual memory in contemporary microprocessors][vmem]                                                                                                                          |                                                                                                                                   |
| Nov 8th              | DRAM                                         | [The Memory System: You Can't Avoid It, You Can't Ignore It, You Can't Fake It][synth-dram] (1.2-1.4)                                                                           | [Unison Cache: A Scalable and Effective Die-Stacked DRAM Cache][dram-cache]                                                       |
| Nov 13th             | Virtualization/Advanced Memory               | [Accelerating two-dimensional page walks for virtualized systems][2d-virt-page]                                                                                                 |                                                                                                                                   |
| Nov 15th             | Multithreading                               | [Multithreading Architecture][synth-multi]                                                                                                                                      | [Niagara: A 32-way multithreaded sparc processor][niagara]                                                                        |
| Nov 20th             | Exam 2 (due 21st)                            |                                                                                                                                                                                 |                                                                                                                                   |
| Nov 27th             | Multiprocessing + Speculative Multithreading | [Speculative Multithreaded Processors][spec-multi]                                                                                                                              |                                                                                                                                   |
| Nov 29th             | Security (Spectre/Meltdown)                  | [Meltdown][meltdown]                                                                                                                                                            | [Spectre Attacks: Exploiting Speculative Execution][spectre]                                                                      |
| Dec 4th              | SIMD + GPUs                                  | [NVIDIA Tesla: A Unified Graphics and Computing Architecture][tesla]                                                                                                            | [The Cray-1 Computer System, Communications of the ACM, January 1978][cray1]                                                      |
| Dec 6th              | Challenges Ahead & Specialization            | [Understanding sources of inefficiency in general-purpose chips][gen-purp-innef]                                                                                                |                                                                                                                                   |
[lec1]: {{ site.baseurl }}/assets/ucla/01-intro.pdf

[sohi-issue]: https://dl.acm.org/citation.cfm?id=78592
[fnf]: https://dl.acm.org/citation.cfm?id=1194844
[perceptron-bp]: https://www.cs.utexas.edu/~lin/papers/hpca01.pdf
[mipsr10k]: http://ieeexplore.ieee.org/document/491460/
[wavescalar]: http://wavescalar.cs.washington.edu/wavescalar.pdf
[two-level-bp]:https://dl.acm.org/citation.cfm?id=139709
[elm]: https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=4563875
[prefetch-tax]: http://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=1261824
[wulf81]: http://www.eecg.toronto.edu/~moshovos/ACA06/readings/wulf-compilers-and-architecture.pdf
[risc-throwdown]: http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.111.1776&rep=rep1&type=pdf
[predication]: http://web.eecs.umich.edu/~mahlke/papers/1995/mahlke_isca95.pdf
[cfp]: https://dl.acm.org/citation.cfm?id=1024407
[mudge-power]: https://ieeexplore.ieee.org/document/917539/
[selective-cache]: https://dl.acm.org/citation.cfm?id=320119
[victim-rep]: https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=1431568
[seznec_alphaev8]: https://dl.acm.org/citation.cfm?id=545249
[ia64]:https://ieeexplore.ieee.org/document/877947
[smith-precise]: https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=4607
[spec-sched-replay]: http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.65.8819
[victim-cache]: https://dl.acm.org/citation.cfm?id=325162
[moore65]: https://www.cs.utexas.edu/~fussell/courses/cs352h/papers/moore.pdf
[isas85]: http://ieeexplore.ieee.org/document/1663000/?reload=true&arnumber=1663000
[power-struggles]: https://research.cs.wisc.edu/vertical/papers/2013/hpca13-isa-power-struggles.pdf
[vmem]: https://ieeexplore.ieee.org/document/710872?arnumber=710872
[synth-proc-micro]: https://www.morganclaypool.com/doi/pdf/10.2200/S00309ED1V01Y201011CAC012
[synth-prefetch]: https://www.morganclaypool.com/doi/pdf/10.2200/S00581ED1V01Y201405CAC028
[synth-dram]: https://www.morganclaypool.com/doi/abs/10.2200/S00201ED1V01Y200907CAC007
[synth-multi]: https://www.morganclaypool.com/doi/abs/10.2200/S00458ED1V01Y201212CAC021
[synth-cache]: https://www.morganclaypool.com/doi/abs/10.2200/S00365ED1V01Y201105CAC017
[niagara]: http://www-hydra.stanford.edu/publications/ieeemicro_niagara.pdf
[crit-aware-cache]: https://ieeexplore.ieee.org/document/8416821
[spectre]: https://spectreattack.com/spectre.pdf
[meltdown]: https://meltdownattack.com/meltdown.pdf
[tesla]: https://ieeexplore.ieee.org/abstract/document/4523358
[cray1]: http://portal.acm.org/citation.cfm?doid=359327.359336
[gen-purp-innef]: https://dl.acm.org/citation.cfm?id=1815968
[2d-virt-page]: https://dl.acm.org/citation.cfm?id=1346286
[spec-multi]:https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=917542
[dram-cache]: https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7011375
