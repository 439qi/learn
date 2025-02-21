---
title: 同步
created: 2025-02-17
tags:
  - sync
  - TBD
---
> [!cite]- References  
> [内核同步原语 – Mark 的滿紙方糖言](https://blog.mygraphql.com/zh/notes/low-tec/kernel/5-sync/synchronizeation-primitives/#%E4%BB%80%E4%B9%88%E6%98%AF%E5%90%8C%E6%AD%A5%E5%8E%9F%E8%AF%AD)  
## 同步原语  

在多线程/进程场景下，为了正确高效地解决同步问题，抽象出一系列同步技术，称为**同步原语(Synchronization Primitives)**  

| 技术                                                                      | 说明                         | 适用范围            |
| ----------------------------------------------------------------------- | -------------------------- | --------------- |
| **CPU 本地变量(Per-CPU variables)**                                         | 对于一个相同的数据类型，每个 CPU 拥有专用的实例 | 所有 CPU          |
| **原子操作(Atomic operation)**                                              | 原子化的 `读取-修改-写入` 一个计数器      | 所有 CPU          |
| [**内存屏障(barrier)**](Multi-thread/Synchronization_Primitives/Barrier.md) | 避免程序的指令被编译器或 CPU 自动重排顺序    | 本地 CPU 或 所有 CPU |
| **自旋锁(Spin lock)**                                                      | 让 CPU 不停忙着读内存标记位的忙等待锁      | 所有 CPU          |
| **信号量(Semaphore)**                                                      | 挂起等待                       | 所有 CPU          |
| **写优先锁(Seqlocks)**                                                      |                            | 所有 CPU          |
| **禁用本地中断**                                                              |                            | 本地 CPU          |
| **禁用本地软中断**                                                             |                            | 本地 CPU          |
| **读-复制-更新(Read-copy-update, RCU)**                                      | 无锁地并发读写基于指针的数据结构           | 所有 CPU          |
