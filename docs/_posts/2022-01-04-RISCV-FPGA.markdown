---
layout: post
title:  "Building an FPGA softcore RISCV processor"
date:   2022-01-04 19:56:01 -0500
categories: FPGA, OS
---
As part of the series on tracing how computers print "Hello World", I started building an FPGA, to someday run KernOS. KernOS currently only supports 32bit x86 instructions so it's nowhere immiment. But I got bored with OS dev.

# Table of contents
1. [Demonstration](#demonstration)
2. [Previous attemps](#previous)
3. [An opening](#opening)
4. [Python emulation](#python)
5. [FPGA toolchain](#toolchain)
6. [Implementing RISCV in FPGA](#verilog)

# Demonstration <a name="demonstration"></a>
Our RISCV FPGA running main.c which sends "Hello World" to the PC through UART
![alt text][image1] <!-- show gif of hello world from uart -->

{% highlight bash %}
#! /bin/bash
riscv-none-embed-gcc -Os -march=rv32i -mabi=ilp32 -nostdlib main.c
{% endhighlight %}

[image1_UART]: ./images/uart.gif
