---
layout: post
title: What to Read
---

One the questions I often get from prospective students is: "as a
newcomer into computer architecture, what should I read to help
prepare me for research".  This post is my detailed answer to that question...

The truth is, textbooks don't really cut it; or at least they aren't enough.
While they may be able to give a
good, broad introduction to the field, they simultaneously tend not to go deep
enough to build a robust understanding, or broad enough to expose the reader to
the vast amount of current and prior work [^1].  Considering any fast moving 
field, they are quickly outdated, and the view of the world that they present
can feel uninspiring.

Probably the "right thing to do" is to read as many papers as possible from the
recent top architecture conferences, and this post is by no means recommending
against doing that.  However, understanding these at a deep level, as well as
their implications for the field, is really challenging for a novice (especially
if you don't have a really solid understanding from a great introductory course).
 Its too easy to walk away from any architecture paper with the content-free takeaway of  
"oh, this idea looks cool, it should work".  Also, and no offense to all authors of
architecture papers, but the way that authors position their work in relation to the field 
does not always exhibit enough intellectual maturity to point out the flaws
or negative tradeoffs that would help readers put the
work in an appropriate context.

So what to do?  In my opinion, a really good option is the short-form book (think thesis-length
written-lectures),
as exemplified by 
the [Morgan Claypool Synthesis Lectures](http://www.morganclaypool.com/toc/cac/1/1). Why do these tend 
to be better?  First, they are written by
people who are passionate about their sub-field, as there is practically no monetary or other
reward for doing it[^2].  Second, these books go in depth, but are byte-sized enough to not require a
huge time commitment to read in their entirety.  Third, they are short enough to be easily updatable --
in fact, many of the lectures in this series have 2nd editions because so much had changed since when
they were published even one decade earlier. 

There's a lot of lectures available now, and typically the way you'd want to read them is 
to pick up the one that sounds most like what you're currently interested in learning about.
That said, I strongly recommend the following books as general reading that any architect
should be aware of.

* On-Chip Networks -- Natalie Enright Jerger, Tushar Krishna, Li-Shiuan Peh
  * This is simply a gem.  The field of networks on chip is a seriously fascinating field, and I never would
    have appreciated it if it hadn't have been for this book (sorry Mark Hill).  This book presents both a
    ground-up view, explains the relationship to past and ongoing research, and has a large number of in-depth
    case studies that really build a solid foundation of knowledge.

* Customizable Computing -- Yu-Ting Chen, Jason Cong, Michael Gill, Glenn Reinman, Bingjun Xiao
  * Hardware specialization, by nature, is a really hard topic to get a handle on, because it opens up the
    design space of possible architecture techniques and involves multiple levels of the computing stack,
    from applications, operating     systems, hardware/software interfaces, and down to technology.  This 
    book covers a big portion of this space in a methodical way and understandable way.

* Multi-Core Cache Hierarchies -- Rajeev Balasubramonian, Norman P. Jouppi, Naveen Muralimanohar
  * This is actually a really nice companion to the on-chip networks book.  It does a great job in explaining
    basic principles, and does a particularly good job of explaining the state-of-the-art and directions in
    the field.  I hope the authors consider keeping this book up to date -- I found it extremely helpful
    and interesting.

* A Primer on Memory Consistency and Cache Coherence -- Daniel J. Sorin, Mark D. Hill, David A. Wood 
  * Memory consistency is an important and tricky topic for architecture newcomers and experienced folks
    alike.  I consider this book to be the only reasonable way to start learning about these topics,
    and I would strongly recommend this to any architect working on multicore systems.

* Processor Microarchitecture: An Implementation Perspective -- Antonio González, Fernando Latorre, Grigorios Magklis
  * This does the job of a more traditional textbook in giving an explanation of the microarchitecture
    principles behind general purpose cores (focusing less on research, and more on a historical perspective).  
    That said, it is a great reference, much easier to read than a 
    textbook (in my opinion), and it covers a set of topics that every architect should be aware of.

[^1]: I also find textbooks quite dry.  If you like, blame it on the limited attention span of today's youth.  
[^2]: And I should know, [because I wrote one](http://www.morganclaypool.com/doi/abs/10.2200/S00531ED1V01Y201308CAC026).
