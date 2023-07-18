+++
author = "psykomal"
title = "L 1.2..3...RAM!"
date = "2023-07-18"
description = ""
tags = [
    "blog", "12-days", "cache", "optimization", "low-level"
]
+++




This article will talk about Cache levels, their impact on performance, and touch on Data-Oriented Design. 


# The Facts

Here are some facts/truths/constraints which come with the current computer architecture:


{{< figure
		  src="cache_imgs/lscpu.png"
		  caption="$lscpu"
>}}

{{< figure
		  src="cache_imgs/caches.png"
		  caption="L1, L2, L3 heirarchy"
>}}



- These caches are designed to store frequently accessed data and instructions closer to the CPU, reducing the time taken to fetch data from the main memory.
- L1 and L2 are per CPU core, and L3 is shared in most general-purpose modern architectures


{{< figure
		  src="cache_imgs/part101_infographics_v08.png"
		  caption="http://ithare.com/infographics-operation-costs-in-cpu-clock-cycles/"
>}}

{{< figure
		  src="cache_imgs/data-locality-chart.png"
		  caption="From Game Programming Patterns"
>}}


Few Observations:
- As you can see there is an order of magnitude cost between L1, L2, L3, and RAM reads
- CPU operations at the front are way cheaper than memory ops
- Kernel call is costly (because usually they involve memory ops)
- A thread context switch is very costly


### A close enough mental model

{{< figure
		  src="cache_imgs/cache_line.png"
		  caption="Cache Line"
>}}


- A cache line is the smallest unit of data that can be loaded into a cache memory. Caches are organized into a hierarchy of levels, such as L1, L2, and L3 caches, and each level is divided into multiple cache lines
- When the CPU accesses data from the main memory, it fetches a whole cache line from the main memory into the cache, even if the CPU only needs a smaller portion of that cache line. This behavior is known as cache line granularity.
- Typical cache line sizes in modern processors are usually 64 bytes or 128 bytes


### Natural Alignment and Memory Layout

- Natural alignment, also known as data alignment, refers to the practice of arranging data in memory at addresses that are multiples of the data's size. Most modern computer architectures have memory alignment requirements, where data types should start at specific memory addresses for optimal performance and efficiency. When data is properly aligned, the CPU can fetch it more efficiently, reducing the number of memory access cycles and improving overall performance.
- From x86 processors [optimization manual](http://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-optimization-manual.pdf), 

> For best performance, align data as follows:
>
> - Align 8-bit data at any address.
> - Align 16-bit data to be contained within an aligned 4-byte word.
> - Align 32-bit data so that its base address is a multiple of four.
> - Align 64-bit data so that its base address is a multiple of eight.
> - Align 80-bit data so that its base address is a multiple of sixteen.
> - Align 128-bit data so that its base address is a multiple of sixteen.


{{< figure
		  src="cache_imgs/mem_layout.png"
		  caption="Memory Layout"
>}}


```C

// Natural Alignment: 8 bytes
// Size: 24 bytes (8 + 8 + 8)

struct {
	a: u32,
	b: u64,
	c: u32
}


// Natural Alignment: 8 bytes
// Size: 16 bytes (4 + 4 + 8)

struct {
	a: u32,
	b: u32,
	c: u64
}


// Natural Alignment: 8 bytes
// Size: 24 bytes (8 + 8 + 8) + 1 (1 is packed in the u32 empty 4/8)

struct {
	a: u32,
	b: u32,
	c: u64,
	d: bool
}

```



# Optimzation


## The OG Example

```C

// Version 1

#include <stdio.h>
#include <stdlib.h>

main () {
  int i,j;
  static int x[4000][4000];
  for (i = 0; i < 4000; i++) {
    for (j = 0; j < 4000; j++) {
      x[j][i] = i + j; }
  }
}

// ==========================================

// Verison 2

#include <stdio.h>
#include <stdlib.h>

main () {
  int i,j;
  static int x[4000][4000];
  for (j = 0; j < 4000; j++) {
     for (i = 0; i < 4000; i++) {
       x[j][i] = i + j; }
   }
}


```


- C uses row-major layout (Fortran uses column-major) which means row data is laid out closer in the memory than columnar. This means, one a single row element fetch, 64 bytes (cache lines) worth of neighbor data also gets picked and placed in the cache so as to avoid cache misses. For more, read - [Why does the order of the loops affect performance when iterating over a 2D array?](https://stackoverflow.com/questions/9936132/why-does-the-order-of-the-loops-affect-performance-when-iterating-over-a-2d-arra)
- In the above example, Version 2 will be faster than Version 1 since it is accessing elements row-wise which will lead to fewer cache misses and many hits. Whereas in Version 1, every memory access is a cache miss if the row and column size is sufficiently large


## Data-Oriented Design

- Data-Oriented Design (DOD) is a programming paradigm and architectural approach that focuses on organizing data and optimizing data access patterns for improved performance and cache efficiency. The primary goal of DOD is to design software systems and data structures in a way that maximizes data locality and minimizes memory access bottlenecks, leading to better CPU cache utilization and overall performance.
- DoD design takes cache architecture into consideration and its main goal is to squeeze maximal performance from the tools we have.
- Data-Oriented Design is especially relevant in performance-critical applications, such as video games, simulations, scientific computing, and real-time systems.


Key Principles of Data-Oriented Design:

1. Data Contiguity: DOD emphasizes keeping related data close together in memory, preferably in contiguous blocks, to promote spatial locality. This reduces cache misses and improves data access speed.
    
2. Data Layout Optimization: Data-Oriented Design focuses on structuring data in a way that minimizes padding and aligns data elements to match the CPU's natural alignment requirements. This optimization aims to make data structures compact and cache-friendly.
	
3. Cache Consciousness: DOD seeks to maximize the utilization of CPU caches by designing algorithms and data structures that work well with the cache hierarchy. This includes designing data access patterns that exploit temporal and spatial locality to minimize cache misses.
    
4. Avoiding Object Overhead: DOD aims to minimize the overhead introduced by object-oriented programming constructs, like vtables, virtual functions, and extra memory overhead for object metadata.
    
5. Array-of-Structs (AoS) to Struct-of-Arrays (SoA) Transformation: In some cases, DOD promotes transforming data from an AoS layout (where each object contains all its properties) to an SoA layout (where all properties are stored in separate arrays). This transformation can improve cache efficiency and data locality.


Some practical tips from the land of DOD (straight out of Andrew Kelley's Practical DOD):
- Use DOD when there are too many objects in the program like games, compilers, graphics
- Identify when there are many things in memory, and make each of them smaller
- Use indexes instead of pointers. 
	- Pointers are typically 8 bytes, which can be replaced with u32 int as manually maintained pointers. For putting this into practice - The Brain Dump, 'Handles are the better pointers': [floooh.github.io/2018/06/17/handles-vs-pointers.html](https://floooh.github.io/2018/06/17/handles-vs-pointers.html)
	- Caution: Watch out for type safety
- Store booleans out-of-band
	- Instead of storing booleans in a struct, we can create separate True and False lists. Will both help memory and read/write performance depending on use-cases
- Eliminate padding with Struct-of-Arrays
	- This can be thought of as Columnar indexes vs Row indexes. Based on use-cases like computing stats on one dimension, Struct-of-Arrays gives better performance since all similar data is closely laid out in the memory reducing the cache misses
- Store sparse data in hashmaps
	- Rather than sparse data taking unnecessary space in several struct-like objects, weed them out to hashmaps saving space and improving data locality
- Use "encodings" instead of polymorphism
	- Bring out common elements into one struct and encode. Very useful and done in games. Check out the Practical DOD video itself for more details.



## Closing Notes

- Modern Games use DOD extensively and there is a whole other paradigm called ECS (Entity Component Systems) for that itself. 
- The following awesome resources helped in building this article
	- [Andrew Kelley - Practical DOD](https://vimeo.com/649009599)
	- [CS Primer](https://csprimer.com)
	- [Data Locality - Game Programming Patterns](https://gameprogrammingpatterns.com/data-locality.html)
	- https://stackoverflow.com/questions/25612029/structure-padding-what-is-the-purpose-of-natural-alignment
	- GPT
- More references
	- [DOD book](https://www.dataorienteddesign.com/dodbook/node3.html)



