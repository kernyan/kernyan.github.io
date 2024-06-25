---
author: kernyan9
comments: true
date: 2020-01-15 01:05:35+00:00
layout: post
link: http://kernyan.com/2020/01/15/building-an-operating-system-kernos/
slug: building-an-operating-system-kernos
title: Building an operating system - KernOS
wordpress_id: 1896
categories:
- Operating System
tags:
- KernOS
---




I am starting a project of building an operating system, whose goal is motivated purely by education rather than practical application. Yet it should be sufficiently capable to make the learning meaningful. For instance, it should have a filesystem, scheduler, GUI, and build tools. 







There are many good operating system resources / guides (osdev, small OS book, SerenityOS, Xinu, etc.) out there. But I haven't come across one that fulfills the following,







  1. Fully reproducible (with both qemu and physical hardware),
  2. Incrementally building the operating system. With explanation of the theory without assuming concepts that haven't been elaborated earlier. And most of all, working code to go along.
  3. Documentation of the exploration, and thought process between going from one step to the next. For example, what should be the next step once the processor reads the boot stub and jumps to the initialization function? Should we set up a stack and free store? Or should we set up interrupt vector table? Why are we choosing one against the other? 
  4. Minimal feature so that the actual intent of learning an OS is not obscured. In this sense, studying linux code base is not what I was looking for.
  5. C++/C implementation because of my relative familiarity with those two languages compared to others
  6. Good practice, consistent coding-style, modular components, and unit tested






The OS should have the following components,







  1. boot loader
  2. kernel
    * processor, stack, and free store initialization; kernel data structures; system calls
    * scheduler
    * process creation / fork
  3. peripheral
    1. keyboard interrupt
    2. serial communication
  4. filesystem
  5. virtual memory map
  6. GUI
  7. user application
  8. networking
  9. build tools






Aside from the high level outline, I haven't much intuition as to which order to proceed, well, that's part of the learning and exploration process. This is going to take a while.



