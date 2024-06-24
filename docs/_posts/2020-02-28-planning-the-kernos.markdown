---
author: kernyan9
comments: true
date: 2020-02-28 03:53:19+00:00
layout: post
link: http://kernyan.com/2020/02/28/planning-the-kernos/
slug: planning-the-kernos
title: Planning the KernOS
wordpress_id: 1919
tags:
- KernOS
---




As initial experimentation, I got a working x86 kernel binary. The key things it accomplishes are that it can be identified by a boot-loader, and subsequently transfers control to print characters to a VGA device [see [https://github.com/kernyan/KernOS](https://github.com/kernyan/KernOS)].







Next, to get better clarity on the design direction, let's lay out some requirements and constrains,







Properties I want,







  1. Running on a modern processor (i.e. clock cycles in the order of GHz)
  2. Modern memory capacity (i.e. at least 4GB of memory)
  3. OS to be written in modern C++, but not wasteful of memory. Also assumes no library support
  4. Portable to multiple architecture (i.e. all assembly codes have to be neatly placed in their own architecture specific directories)
  5. Initial implementation targets x86 






As a starting point, I want reliable and convenient debugging support as I expect this to be a huge time saver especially in debugging assembly code. Thus, I need







  1. Utility function to dump memory content in specified range
  2. Unit tests on modular functions to prevent accidental behavior change
  3. Implication of 2) is that I want to move away from the common technique of using globals for OS data structures to the extend possible, otherwise those globals has to be easily mocked for testing
  4. Pretty printing of data structure contents
  5. Gdb hooks, maybe?






With those constrains and requirements specified, what part of the OS component listed in [[https://kernyan.com/2020/01/15/building-an-operating-system-kernos/](https://kernyan.com/2020/01/15/building-an-operating-system-kernos/)] do we write first?







Let's scan the immediate components and see what interfaces/data structures that are repeatedly needed,







  1. Process creation/switching
    * data structure to represent process, and its properties
    * assembly to save stack/register, and jump
    * rescheduling - tracking ready processes, ordered by priority
  2. Dynamic memory allocation
    * Partitioning heap memory into chunks, 
    * Finding suitably sized unused chunks, marking as used
    * Returning and coalescing memory,
    * Memory pool for each process (to avoid one process hogging all free memory), maybe?
  3. Coordination of concurrent processes
    * Semaphores that track waiting processes






Of the above, 1) tracking ready processes, 2) tracking memory chunks, and 3) tracking waiting processes all require the similar feature of being able to link together similar data structures. Since we don't have a heap allocator yet, we need a data structure that describes a collection of homogeneous but arbitrary data types, that doesn't require dynamic memory.







One solution is to have our data structure to have contiguous memory layout. One can imagine the data structure to have pointers that each point to the next item. However we cannot directly point to the item we are describing because we also need to know how to get from that item to the next one. Instead we introduce an "entry" data type that our data structure contains. Each "entry" would contain the information of the actual memory location (or indices into a C array if we assume the items are contiguous in memory as well) of the item, AND the next "entry". Thus when following sequences of items, we remain entirely within our own data structure. 







Essentially, I am describing a linked-list with the requirement that every list node are contiguous.







Things to note of the linked-list that we need,







  1. We want the following atomic functions
    * remove item from list (and joining remaining parts of the list)
    * add item to list
  2. Ideally the list also accommodates ordering by priority which is ready for process rescheduling






Let's build that next.



