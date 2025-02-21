---
title: Consistency
aliases:
  - 内存一致性
  - 内存连贯性
created: 2025-02-19
tags:
---
> [!cite]- References  
> [coherence, consistency and memory model - 知乎](https://zhuanlan.zhihu.com/p/58589781)  
> [GitHub - kaitoukito/A-Primer-on-Memory-Consistency-and-Cache-Coherence: A Primer on Memory Consistency and Cache Coherence (Second Edition) 翻译计划](https://github.com/kaitoukito/A-Primer-on-Memory-Consistency-and-Cache-Coherence?tab=readme-ov-file)  
> [理解 c++ 内存一致性模型 | Weakyon Blog](https://weakyon.com/2023/07/23/understanding-of-the-cpp-memory-order.html)  

Consistency/Memory Consistency/Memory Consistency Model/Memory Model 是关于在 **共享内存(shared memory)** 的环境下，loads 和 stores 的规则，以确保共享内存的**正确性(Correctness)**  
> 本文仅限于共享内存多核的 Consistency，不包括分布式系统中的 consistency 模型等  

为了支持 Consistency，硬件需要提供 [Cache Coherence](Multi-thread/Synchronization_Primitives/Coherence.md)，但 Consistency 本身并不涉及缓存和 Coherence    