---
layout: post
title:  "Deepseek V3/R1 intra/inter node all-to-all communication"
date:   2025-02-26 08:00:00 -0000
categories: HPC, CUDA
---

Recently, DeepSeek V3 made headlines by being able to train 14.8 trilion tokens using only 2.788 million H800 GPU hours. This was estimated to be several times more efficient than approaches that did not incorporate DeepSeek's LLM and training infrastructure designs.

In the initial 2024-12-26 announcement for [DeepSeek-V3](https://arxiv.org/pdf/2412.19437), and subsequently the publishing of the [inference repo](https://github.com/deepseek-ai/DeepSeek-V3), one part that remained missing stood out to me; which is on how the cross GPU communication kernels are implemeneted.

> ### 3.2.2. Efficient Implementation of Cross-Node All-to-All Communication
>
> ...
> In detail, we employ the warp specialization technique (Bauer et al., 2014) and partition
> 20 SMs into 10 communication channels. During the dispatching process, (1) IB sending, (2)
> IB-to-NVLink forwarding, and (3) NVLink receiving are handled by respective warps. The
> number of warps allocated to each communication task is dynamically adjusted according to the
> actual workload across all SMs. Similarly, during the combining process, (1) NVLink sending,
> (2) NVLink-to-IB forwarding and accumulation, and (3) IB receiving and accumulation are also
> handled by dynamically adjusted warps. In addition, both dispatching and combining kernels
> overlap with the computation stream, so we also consider their impact on other SM computation
> kernels. Specifically, we employ customized PTX (Parallel Thread Execution) instructions and
> auto-tune the communication chunk size, which significantly reduces the use of the L2 cache
> and the interference to other SMs.

From the quote above in their paper, I'm also curious about what customized PTX instructions are used. Given that these are missing in their repo, I suspected it was a secret sauce that they didn't intend to share.

Fortunately, I was proven wrong as the DeepSeek team published the [DeepEP repo](https://github.com/deepseek-ai/DeepEP) on 2025-02-26.

Thus my post below will attempt to analyze the key designs in the communication kernel.

# Table of contents
1. [Introduction](#introduction)
2. [The Problem: Scalable MoE in LLMs](#problem)
3. [Key Design Choices for Efficient Communication](#key_design)
4. [Communication Kernel Implementation Details](#implementation)
5. [Appendix](#appendix)

# The Problem: Scalable MoE in LLMs <a name="problem"></a>

pass

# Key Design Choices for Efficient Communication <a name="key_design"></a>

pass

# Communication Kernel Implementation Details <a name="implementation"></a>

pass

# Appendix: Links to the paper / github repo <a name="appendix"></a>

pass