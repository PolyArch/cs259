---
layout: page
title: Syllabus
---

### Learning Machines
* Instructor: Tony Nowatzki
* Term: Spring 2021
* Textbook: None! 
* [Zoom Join URL for Class](https://ucla.zoom.us/j/94930257393?pwd=SU56NjFEVVhZMmx2NFk4VTJ3ZWJnUT09)
  - This will be public for now -- if we need to use a private one I will make an update later.

### Course Objectives

The advancements and overwhelming success of Machine Learning has profoundly
affected the future of computer architecture. Not only is performing learning
on big-data the leading application driver for future architectures, but also
machine learning techniques can be used to improve hardware efficiency for a
wide variety of application domains.

This course will explore, from a computer architecture perspective, the
principles of hardware/software codesign for machine learning. One thrust of
the course will delve into accelerator, CPU, and GPU enhancements for ML
algorithms, including parallelization techniques. The other thrust of the
course will focus on how machine learning can be used to optimize conventional
architectures by dynamically learning and adapting to program behavior.

Several important specific goals are:

* Develop skills in domain-specialization (reason about how application/domain properties can be exploited with hardware mechanisms) 
* Gain understanding of the current state of the art within acceleration for machine learning, both in academia and in industry.

Also, there are some general goals which hold for any architecture/hardware course:

* Gain intuition and reasoning skills regarding fundamental architecture tradeoffs of hardware design choices (performance/area/power/complexity/generality).
* Understand microarchitecture techniques for extracting parallelism and exploiting locality.
* Learn about evaluation methods, including simulation, analytical modeling, and mechanistic models.  

### Course Components:

Logistically, this course has 4 components.  
* *Participation: Online forum discussion:* During this course we will read a number of research papers from
  literature.  We will discuss these on the CCLE online forum (if this doesn't work well, we will
  return to piazza). See the [discussion page]({{site.baseurl}}/04-discussion/) for more instructions.

* *Participation: In-class discussion:*  We will also discuss papers in class, and one way you can earn participation
points is to ask questions or provide your thoughts either audibly, or in the live chat during class.
 Remember, this class will be fun and interesting if you make it so!

* Mini-Project (15%) There will be two mini projects which can also be performed in groups:
  - Parallelizing a machine learning kernel using CUDA on our V100 GPU. 
  - Build a ML-accelerator simulator, which is correct and produces accurate performance estimates.

* Leading Class Discussion (15%) Each student (or group of 2) will lead 1 lecture/discussion

* *Project:*  Project (40%) Group based research/implementation project with 1-4 students. Please see the project handout, and feel free to use Piazza to help form groups. You will need to propose a project by the beginning of the 5th week of class, so please start thinking early. See [project]({{site.baseurl}}/08-project/) page for more details.

## Grade Breakdown
* 30% In-class or online discussion
    - Points are calculated as min(forum + participation,100%).  
    - That means you can earn full points by:
      * Participating in every class with at least one thoughtful comment, or
      * Writing an insighful review for every in-class discussion
      * A combination of the above.
* 15% Mini-projects
    - 7.5% Each
* 15% Leading Class Discussion
* 40% Project

