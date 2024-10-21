---
layout: default
title: Multi-Threading And Fibers - Game Engine Study
description: Marcin Zalewski - November 1, 2024
---

...

## Table of contents
1. [Theory](#theory)
    1. [A brief background of multi-processing](#theory1)
    2. [Multi-threading game loops](#theory2)
    3. [Fibers... What are they?](#theory3)
2. [Implementation](#implementation)
    1. [Using Fibers](#implementation1)
3. [Conclusion](#conclusion)
    1. [Further reading](#conclusion1)
    2. [Sources](#conclusion2)

## Theory <a name="theory"></a>

Nowadays commercial game-engines heavily use multi-threading to support playable frame rates. There are many different approaches for integrating multi-threading into an application, and in this section we will explore the different approaches used over the years.

### A brief background of multi-processing <a name="theory1"></a>

The processor manufacturing industry in 2004 encountered a problem with heat dissipation, which prevented them from producing faster CPUs. In turn, the multi-processor manufacturers shifted their focus to producing multi-processor CPUs. This switch had effect on the Moore's Law, which predicts an approximate doubling in transistor counts every 18 to 24 months. And that statement still holds true today. But in 2004, its assumed correlation with with doubling processor speeds was shown to be no longer valid, as single threaded performance advancements started to dissipate.

<figure align="center" class="image">
<img src="assets/images/multi-threading-fibers/TransistorCountOverTheYears.png" alt="Transistor count over the years."/>
<figcaption> <a href="https://www.semianalysis.com/p/a-century-of-moores-law"> Transistor count over the years. Douglas Herz. A Century of Moore’s Law, Febrary 04, 2023 </a> </figcaption>
</figure>

As a result of this switch to multi-core systems, the game engines at the time turned to parallel processing techniques. This can be seen with systems such as the Xbox 360 and PlayStation 3, where the game engines no longer relied on a single main game loop to service their subsystems. Designing multi-threaded programs is much harder than single-threaded ones. Most game companies took a few years to switch their game engines to completely utilize multi-threading. They transformed the engines step by step, where they selected subsystems and parallelized them. By 2008, most commercial game engines had completed their turn to multi-processing, with different approaches and varying degrees of parallelism.

### Multi-threading game loops <a name="theory2"></a>

...

#### Fork And Join

One of utilizing the multi-core hardware is to use the so called divide-and-conquer algorithms, often called fork and join. 
The idea here is to divide the work into smaller chunks, distribute these onto the hardwares cores (fork), and merge the results once all of the smeller work chunks have been completed (join). In practice this looks similar to a single-threaded approach, but with some major parts being parallelized. Let's take a look at a visual example.

<figure align="center" class="image">
<img src="assets/images/multi-threading-fibers/JoinAndFork.jpg" alt="Join and Fork diagram."/>
<figcaption> Join and Fork architecture </figcaption>
</figure>

We can see that the master thread forks the task into 3 processes, 2 of them being other threads. And later on the processes join and the master thread again forks into 4 tasks until they are completed and joins again. In this case the master thread is doing a part of the work we want to do, but it is also possible to let the entirety of the work to be done by other threads and let the master thread do some other work in the mean time, for example preparing for or even doing the next fork. Also notice how I have named the thread that is dividing up the work "master thread", this is because the forking doesn't have to be done by the main thread of the program and multiple forks can be done one after the other to later be joined together.

#### One Thread Per Subsystem

Another famous approach to multitasking is to assign engine subsystems to their own cores. The main thread controls and synchronizes the operations of these major subsystems threads and also continues to handle a share of the engines high level-logic. On hardware with multiple physical CPUs or cores, this approach allows these subsystems to execute in parallel. This design is well suited for subsystems that do relatively isolated work, for example the renderer or the audio engine. We can depict such an architecture with the following diagram.

<figure align="center" class="image">
<img src="assets/images/multi-threading-fibers/OneThreadPerSubsystem.jpg" alt="One thread per subsystem diagram."/>
<figcaption> One thread per subsystem architecture </figcaption>
</figure>

One problem with this approach is that each thread represents their own course-grained chunk of work, for example all physics calculations. This puts a restriction on how the various cores can be utilized. If one of the subsystems has not completed its work, the progress of other threads may be blocked. Or if one subsystem has no work to do at the moment, and just sleeps, it doesn't utilize its thread to the fullest and could run other calculations in the background, such as the beginning of the Dynamics Thread seen in the diagram above.

#### Jobs

A different way to take advantage of multi-core hardware is to divide up work in small, relatively independent tasks (jobs). A job can be thought of as simply just a pairing of data and a function that operates on that data. When a job is created, it is placed in a queue, waiting until the next available thread can pick it up and run it.

<figure align="center" class="image">
<img src="assets/images/multi-threading-fibers/Jobs.jpg" alt="Jobs diagram."/>
<figcaption> Jobs architecture </figcaption>
</figure>

The fact that jobs are small and independent of one another helps to maximize processor utilization, while providing more flexibility. It relieves some restrictions that would come with the "one thread per subsystem" approach. For example, this design nicely scales up with the amount of cores present, where you can just offload work to more threads. In this blog post we well dive deeper into and implement this design.

### Fibers... What are they? <a name="theory3"></a>

Before we get to the actual implementation, we have to talk about fibers. You can think about fibers like a partial thread, which contains a user provided stack space, small context state of the fiber and saved registers. However, fibers are executed by a thread. This happens by taking a normal thread and swapping the registers and the stack pointer, also called "switching a fiber". We also have the difference between cooperative multi-threading, which leaves the control of scheduling to the developer, and preemptive multi-threading, which gives the control of scheduling to the OS. Fibers also have a nice benefit here, as fibers are cooperative multi-threading, that means that a fiber can never be preempted, because the user has control over switching fibers on a thread, by explicitly calling a function to switch fibers. Because switching fibers is just swapping out the registers and the stack pointer, it has minimal overhead, because there is no thread context switching between fibers. Only saving/restoring registers. To make it more clear, let's take a look at a diagram, which shows how fibers fit in a program.

<figure align="center" class="image">
<img src="assets/images/multi-threading-fibers/FiberInProgramUsage.jpg" alt="Fibers in a program diagram."/>
<figcaption> How fibers fit in a program </figcaption>
</figure>

As seen from the diagram, user's logic can be run either on a regular thread or a fiber, which itself runs on a thread. As explained before, the developer has control over scheduling fibers, the OS will not help with this. Also the user can create a lot more fibers than threads, as things like stack memory allocation are a lot more controllable by developers. Since we save the registers in a fiber, we can stop the execution of a function in the middle, to run a more important function, and resume the waiting fiber later on when needed. Though not shown in the diagram, a system that uses fibers commonly use thread affinity to lock threads to dedicated cores to avoid potential context switching and in turn, better performance. The number of background threads doesn't need to be high, commonly only a few low priority threads are needed for blocking operations like IO.

There are more things to explore, which we will take a look at with more detail while implementing our job system.

## Implementation <a name="implementation"></a>

...

### Using Fibers <a name="implementation1"></a>

...

## Conclusion <a name="conclusion"></a>

...

### Further reading <a name="conclusion1"></a>

...

### Sources <a name="conclusion2"></a>

- [1] [*Douglas Herz. A Century of Moore’s Law, Febrary 04, 2023*](https://www.semianalysis.com/p/a-century-of-moores-law)
- [2] [*Jason Gregory. Game Engine Archiechure 3th edition, August 17, 2018*](https://www.gameenginebook.com)
- [2] [*Christian Gyrling. Parallelizing the Naughty Dog Engine Using Fibers, March 2-6, 2015*](https://www.youtube.com/watch?v=HIVBhKj7gQU&t=1399s)

---
*If you see this, I would like to thank you for keeping with my blog post until the end. You can find more of my blog posts at the [main page](https://mmzala.github.io/blog/). Feel free to reach out through my e-mail (marcinzal24@gmail.com) with any questions or comments. You can also find me on [LinkedIn](https://www.linkedin.com/in/marcin-zalewski-6a17231a4/) if you would rather reach out to me that way.*