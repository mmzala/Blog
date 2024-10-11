---
layout: default
title: Multi-Threading Using Fibers - Game Engine Study
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

Nowdays commertial game-engines use multi-threading heavily to support playable frame rates. There are many different approaches for integrating multi-threading into an application, and in this section we will explore the different approaches used over the years.

### A brief background of multi-processing <a name="theory1"></a>

The multi-processor industry in 2004 encountered a problem with heat dissipation, which prevented them from producing faster CPUs. In turn, the multi-processor manufacturers shifted their focus to producing multi-processor CPUs. This switch had effect on the Moore's Law, which predicts an approximate doubling in transistor counts every 18 to 24 months. And that statement still holds true today. But in 2004 its assumed correlation with with doubling processor speeds was shown to be no longer valid, as single threaded performance advencements started to dissipate.

<p align="center">
<img src="assets/images/multi-threading-fibers/TransistorCountOverTheYears.png" alt="Transistor count over the years."/>
</p>

As a result of this switch to multi-core systems, the game engines at the time turned to parallel processing techniques. This can be seen with systems such as the Xbox 360 and PlayStation 3, where the game engines no longer relied on a single main game loop to service their subsystems. Designing multi-threaded programs is much harder than single-threaded ones. Most game companies took a few years to switch their game engines to completely utilize multi-threading. They transformed the engines step by step, where they selected subsystems and parallelized them. By 2008, most commertial game engines had completed their turn to multi-processing, with different appraoches and varying degrees of parallelism.

### Multi-threading game loops <a name="theory2"></a>

...

### Fibers... What are they? <a name="theory3"></a>

...

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

---
*If you see this, I would like to thank you for keeping with my blog post until the end. You can find more of my blog posts at the [main page](https://mmzala.github.io/blog/). Feel free to reach out through my e-mail (marcinzal24@gmail.com) with any questions or comments. You can also find me on [LinkedIn](https://www.linkedin.com/in/marcin-zalewski-6a17231a4/) if you would rather reach out to me that way.*