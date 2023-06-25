---
layout: post
title:  "Matmul in CUDA"
date:   2023-05-09 00:00:01 -0500
categories: NN
---
How does a CUDA kernel work? How does a matmul CUDA kernel look like?

# Table of contents
1. [Demonstration](#demonstration)
2. [CUDA](#verilog)

# Demonstration <a name="demonstration"></a>

Here's the CUDA code for a matmul

{% highlight bash %}
#! /bin/bash
riscv-none-embed-gcc -Os -march=rv32i -mabi=ilp32 -nostdlib main.c
{% endhighlight %}

Calling it from python

# CUDA <a name="cuda"></a>

warp and stuff

programming model, of threads and blocks

what are blocks?


threadIdx is CUDA thread id executing the kernel
thread block is x,y,z dimension of thread
