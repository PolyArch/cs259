---
layout: page
title: Homework 0
---

Many architecture simulators exist (simplescalar, gems, Simics,
Multi-2sim, Sniper, PTLsim, ZSim, SIMFLEX, MARSS), but Gem5 may
be the only one that is in active development with a large
community, can run many ISAs and microarchitectures, has
full-system support, and has a "textbook" to help newcomers -- so, lets use it!

This "assignment" is to go through the introduction and part 1 of the 
[online book](http://learning.gem5.org/book/index.html).

The most important parts of these chapters is:
* downloading and building gem5
* understanding the configuration script, (basic use and modification)
* learning how to run gem5 
* figuring out the statistics

However, its good to go through the rest of Part 1 (and perhaps other
chapters eventually) as it may provide a broader understanding will be
useful in later homeworks, and could be useful for the project.
The author of the tutorial also has a [YouTube video](https://www.youtube.com/watch?v=5UT41VsGTsg) of the tutorial that may be useful.

# Advice 
One important piece of advice as we begin to use
simulators -- its absolutely critical to [not treat them as a
black box](http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7155440).  Simulators make many *many* assumptions that are
either not true, do not make any sense,  or are specific to a particular implementation.  If you don't make yourself aware of these assumptions, then its impossible to draw meaningful conclusions.  This will be difficult when using a new (and awfully complex) simulator like gem5, but is part of the learning process.


Is gem5 not useful because it contains [modeling, specification
and abstraction
errors](http://www.eecs.umich.edu/eecs/about/articles/2014/ispass_2014-1.pdf)
-- no of course not!  An important mantra for the jaded architect:

> All models are wrong, and some are useful

