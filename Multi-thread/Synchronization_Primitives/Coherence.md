---
title: Coherence
aliases:
  - 缓存一致性
created: 2025-02-19
tags:
  - TBD
---
> [!cite]- References  
> [coherence, consistency and memory model - 知乎](https://zhuanlan.zhihu.com/p/58589781)  
> [GitHub - kaitoukito/A-Primer-on-Memory-Consistency-and-Cache-Coherence: A Primer on Memory Consistency and Cache Coherence (Second Edition) 翻译计划](https://github.com/kaitoukito/A-Primer-on-Memory-Consistency-and-Cache-Coherence?tab=readme-ov-file)  

## Incoherence 问题  

Incoherence 指的是当多个参与者访问同一数据的多个副本（如在缓存中），且至少一个访问是写入操作时，各个副本的内容不一致的问题  

Coherence 协议定义了一组规则，以防止 Incoherence 问题  
Coherence 协议存在多个变种，但本质上都是通过将写入操作传播到所有缓存中，使得该操作对其余参与者**可见(visible)**  
