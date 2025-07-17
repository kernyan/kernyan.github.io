---
layout: post
title:  "Conv2D GPU Baselines: cuDNN and CUTLASS Performance and Analysis (Part 2)"
date:   2025-07-17 08:00:00 -0000
categories:
- CUDA
- CPU
mathjax: true
---

In [Part 1: Tuning Conv2D from CPU to GPU from Scratch](/cuda/cpu/2025/07/12/tuning-convolution2d-from-CPU-to-maximum-GPU-perf-from-scratch.html), we optimized a convolutional kernel from a naive nested loop on CPU to a multi-threaded NHWC version with OpenMP, bringing runtime from 150 seconds to 3.4 seconds.

Now, we turn to the GPU and ask: how fast can industry-standard libraries like **cuDNN** and **CUTLASS** make this kernel? And how much of the GPU's peak compute can they actually use?

Below, we’ll benchmark both libraries, analyze their internals with Nsight Compute, and compare them against the theoretical peak we calculated in Part 1.


# Table of Contents
1. [Why Use cuDNN and CUTLASS](#why-use-cudnn-and-cutlass)
2. [Benchmark Setup](#benchmark-setup)
3. [Results: cuDNN vs CUTLASS](#results-cudnn-vs-cutlass)
4. [What’s Under the Hood?](#whats-under-the-hood)
5. [Performance Summary So Far](#performance-summary-so-far)
6. [What's Next?](#whats-next)
7. [Appendix](#appendix)

# Why Use cuDNN and CUTLASS

### cuDNN

cuDNN is NVIDIA's production-grade GPU library used by most deep learning frameworks (PyTorch, TensorFlow). It's hand-tuned for every GPU generation and uses advanced algorithms like:
- Winograd convolution
- Implicit GEMM
- Tensor Core acceleration
- Kernel auto-tuning for each shape

As cuDNN is closed source, we can only infer their operation via metrics that NSight profile reveals.

### CUTLASS

CUTLASS is an open-source CUDA C++ template library developed by NVIDIA. At its core is a collection of abstractions for implementing high-performance GEMM and convolution kernels on GPUs.

One of its core components is **CuTe** (CUDA Tensor), which provides tensor views with fine-grained control over layout, shape, and tiling. Instead of manually computing strides or worrying about memory bank conflicts, you can declaratively specify how tensor elements should map across threads, warps, and threadblocks in a memory-efficient way.

The general idea of CUTLASS in implementing a Convolution kernel is it provides high parameterized C++ CUDA templates. It decomposes the kernel into the following parts,

- **Input & weight iterators**: for coalesced shared memory loads  
- **Implicit GEMM mapping**: to transform NHWC × KRSC into GEMM tiles  
- **Device-specific MMA ops**: using CUDA or Tensor Cores  
- **Epilogue**: to store accumulator registers into global memory efficiently


We'll use CUTLASS and cuDNN for benchmarking against how well they perform on my platform. And CUTLASS as the framework we'll gradually deconstruct in future posts.


# Benchmark Setup

We used the same Conv2D configuration:


| Parameter | Value                 |
|----------:|-----------------------|
| Input     | NHWC `(10, 224, 224, 128)`
| Filter    | KRSC `(128, 3, 3, 128)`
| Output    | `(10, 224, 224, 128)`
| Padding   | 1                     |
| Stride    | 1                     |
| Dilation  | 1                     |
| Data type | `FP32`, `FP16`        |
| Device    | NVIDIA RTX 2070 Super |

We benchmarked:
- cuDNN in both **NCHW** and **NHWC** formats.
- CUTLASS with **SIMT** (FPU) and **Tensor Core** (FP16) kernels.


# Results: cuDNN vs CUTLASS

| Library         | Layout | Algo       | Data Type | Time (ms) | % of Theoretical Peak* |
|-----------------|--------|------------|-----------|-----------|------------------------|
| cuDNN           | NCHW   | Winograd   | FP32      | 15.94 ms  | 100.4 %                |
| cuDNN           | NHWC   | Winograd   | FP32      | 17.15 ms  | 93.3 %                 | 
| CUTLASS         | NHWC   | CUDA Core  | FP32      | 27.96 ms  | 43.4 %                 |
| CUTLASS         | NHWC   | Tensor Core| FP16      |  6.23 ms  | 32.1 %                 |

\* Theoretical peak: 16 ms (FP32 CUDA core), 2 ms (FP16 Tensor Core), from [Part 1](/cuda/cpu/2025/07/12/tuning-convolution2d-from-CPU-to-maximum-GPU-perf-from-scratch.html#roofline-cpu-vs-gpu)


It may seem surprising that cuDNN appears to exceed the theoretical peak of 16 ms. But this is due to the fact that cuDNN’s Winograd implementation does not compute a full 3×3 convolution directly. Which transforms the problem into a compressed domain where the number of multiplications is greatly reduced.

Let's take a digression to measure how many fp32 instruction cuDNN saved from using Winograd.

## Digression: Winograd vs Direct Convolution Flops count

We can do so by using NSight to get the number of FMA ops it performs. Recall that in Part 1, we calculated the number of FMA ops as `N*P*Q*K*C*R*S = 73.987 billion`. Let's verify that a naive convolution performs that many ops, using 

`ncu --metrics smsp__sass_thread_inst_executed_op_fp32_pred_on.sum`


```
  manual_conv2d_kernel(const float *, const float *, float *, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int) (3920, 1, 2)x(256, 1, 1), Context 1, Stream 7, Device 0, CC 7.5
    Section: Command line profiler metrics
    --------------------------------------------------- ----------- --------------
    Metric Name                                         Metric Unit   Metric Value
    --------------------------------------------------- ----------- --------------
    smsp__sass_thread_inst_executed_op_fp32_pred_on.sum        inst 74,836,500,480
    --------------------------------------------------- ----------- --------------
```

We see that the total FP32 op executed on a naive Conv2D kernel is very close to our calculated FMA op counts, at 74.0 and 74.8 billion ops respectively.

Next, we measure the number of FP32 ops performed by cuDNN,

| Kernel                              | Op Count (Billions) |
|-------------------------------------|---------------------|
| Winograd Forward Data 4x4           | 1.233              |
| Winograd Forward Filter 4x4         | 0.001              |
| Volta SGEMM 128x64 NN               | 29.830             |
| Winograd Forward Output 4x4         | 1.355              |
| Total cuDNN                         | 32.419             |

Based on this reduced operation count (~32.4 billion ops total after including transforms), cuDNN's effective compute workload is about **43%** of direct Conv2D. This means its actual peak runtime should be **6.91 ms** — not 16 ms — making its real utilization ~43.4%.

| Library         | Layout | Algo       | Data Type | Time (ms) | % of Theoretical Peak* |
|-----------------|--------|------------|-----------|-----------|------------------------|
| cuDNN           | NCHW   | Winograd   | FP32      | 15.94 ms  | 43.4 %                 |
| cuDNN           | NHWC   | Winograd   | FP32      | 17.15 ms  | 40.3 %                 | 
| CUTLASS         | NHWC   | CUDA Core  | FP32      | 27.96 ms  | 43.4 %                 |
| CUTLASS         | NHWC   | Tensor Core| FP16      |  6.23 ms  | 32.1 %                 |


# What’s Under the Hood?

| Library     | Algorithm   | Layout | Tensor Cores | Tiling |
|-------------|-------------|--------|---------------|--------|
| cuDNN       | Winograd    | NHWC   | Yes           | Dynamic (opaque) |
| CUTLASS     | Implicit GEMM | NHWC | Optional      | Explicit templates |

For example, in CUTLASS:
- Tensor Core kernels use fragments (`mma.sync`) over shared memory tiles.
- CUDA Core (SIMT) kernels use FMA loops over global/shared memory tiles.
- All kernels follow a `load → compute → epilogue` pipeline.

# Performance Summary So Far

| Version         | Time    | Speedup over CPU | % of Peak attainable |
|------------------|--------|------------------|----------------------|
| Naive CPU        | 150 s  | 1×               | 0.24 |
| CPU (NHWC + OMP) | 3.4 s  | 44×              | 10.6  |
| cuDNN NCHW       | 15.9 ms | 9,434x           |   43.4 |
| cuDNN NHWC       | 17.2 ms  | 8,721x          |   40.3 |
| CUTLASS SIMT     | 28 ms  | 5,357x            |   43.4 |
| CUTLASS Tensor   | 6.2 ms   | 24,194×        | 32.1  |


# What's Next?

cuDNN and CUTLASS are fast - but how do they actually work?

In the next posts, we'll build a Conv2D CUDA kernel *from scratch*, mimicking CUTLASS pipeline:
- shared memory staging
- threadblock tiling
- register-level accumulation
- split-k gemm tiling
- epilogue accumulation and write back

Our goal is to build a **bare-metal CUDA kernel** — without any external header — that matches or even exceeds the hardware efficiency of these production-grade libraries.

By writing our own `Conv2D` kernel from scratch, we’ll better understand:
- Where real GPU performance comes from,
- How tiling and memory staging impact latency,
- And what it really takes to hit 40%+ of peak performance.


# Appendix:

We include the grid and block size for the various kernels before, and briefly highlight their notable features,

### cuDNN NCHW (15.94 ms)

| Function Name                       | Grid Size       | Block Size     | Duration [msecond] |
|------------------------------------|-----------------|----------------|--------------------|
| vectorized_elementwise_kernel      | 62720, 1, 1     | 128, 1, 1      | 0.93               |
| vectorized_elementwise_kernel      | 144, 1, 1       | 128, 1, 1      | 0.01               |
| winogradForwardData4x4             | 196, 128, 1     | 256, 1, 1      | 3.94               |
| winogradForwardFilter4x4           | 4, 16, 1        | 32, 8, 1       | 0.01               |
| volta_sgemm_128x64_nn              | 392, 2, 36      | 128, 1, 1      | 8.19               |
| winogradForwardOutput4x4           | 196, 128, 1     | 256, 1, 1      | 2.86               |

cuDNN performs a Winograd transform on the input and filter, applies GEMM, and then transforms the output back from the Winograd domain.

### cuDNN NHWC (17.15 ms)

| Function Name                       | Grid Size       | Block Size     | Duration [msecond] |
|------------------------------------|-----------------|----------------|--------------------|
| vectorized_elementwise_kernel      | 62720, 1, 1     | 128, 1, 1      | 0.93               |
| vectorized_elementwise_kernel      | 144, 1, 1       | 128, 1, 1      | 0.01               |
| nhwcToNchwKernel                   | 1568, 4, 10     | 256, 1, 1      | 1.41               |
| nhwcToNchwKernel                   | 1, 4, 128       | 256, 1, 1      | 0.01               |
| generateWinogradTilesKernel        | 4, 32, 1        | 32, 4, 1       | 0.01               |
| _5x_cudnn_volta_scudnn_winograd_128x128_ldg1_ldg4_relu_tile148t_nt_v1 | 40, 28, 14 | 256, 1, 1 | 13.30              |
| nchwToNhwcKernel                   | 1568, 4, 10     | 256, 1, 1      | 1.48               |

Here, cuDNN is actually transforming NHWC into NCHW before performing winograd. 

This is somewhat surprising, as NHWC is typically considered more optimal for GPU memory access. However, it suggests that cuDNN’s Winograd kernels may be optimized specifically for NCHW, and the overhead of layout transformation is worth the tradeoff.

### CUTLASS CUDA Core (27.96 ms)

| Function Name                       | Grid Size       | Block Size     | Duration [msecond] |
|------------------------------------|-----------------|----------------|--------------------|
| vectorized_elementwise_kernel      | 62720, 1, 1     | 128, 1, 1      | 0.93               |
| vectorized_elementwise_kernel      | 144, 1, 1       | 128, 1, 1      | 0.00               |
| vectorized_elementwise_kernel      | 62720, 1, 1     | 128, 1, 1      | 0.63               |
| Kernel                             | 3920, 1, 1      | 256, 1, 1      | 26.40              |

CUTLASS provides a clean implicit GEMM kernel via `DefaultConv2dFprop`. With `float32` data and SIMT path, we reach **28 ms**. Although this is slower than cuDNN, accounting for the reduced operation count in cuDNN's Winograd implementation reveals that both achieve similar hardware utilization — around 43.4% of theoretical peak.

### CUTLASS Tensor Core (6.23 ms)

| Function Name                       | Grid Size       | Block Size     | Duration [msecond] |
|------------------------------------|-----------------|----------------|--------------------|
| vectorized_elementwise_kernel      | 62720, 1, 1     | 128, 1, 1      | 0.95               |
| vectorized_elementwise_kernel      | 144, 1, 1       | 128, 1, 1      | 0.00               |
| vectorized_elementwise_kernel      | 62720, 1, 1     | 128, 1, 1      | 0.32               |
| Kernel                             | 1960, 1, 1      | 256, 1, 1      | 4.96               |

By switching to `__half` types and enabling Tensor Core kernels, CUTLASS hits **6.23 ms** — the fastest kernel among all tested configurations. However, its hardware utilization is the lowest at 28.9%.
