---
layout: post
title: Parallesim 初学笔记
tags: [Parallelism, CS61C]
---

CS61C 初学笔记

## 引入Parallelism的原因&简介

由于能耗, 散热等原因, 硬件的性能提升无法一直维持指数增长, 陷入了瓶颈,  为了应对这种情况提出了并行的概念来提高程序的执行速度. 并行的实现一方面需要多个CPU(多核), CPU支持多线程等硬件基础, 另一方面也需要也需并发程序的编写.

其实之前学习的流水线也可看作一种广义的并行, 在流水线执行过程中, CPU不同部分同时处理不同指令的不同部分, 但这篇笔记内的并行特指数据并行以及线程并行, 并且没有涉及Parallel Requests的内容

## Parallelism的局限性

根据Amdahl’s Law, 只有当程序存在可并行部分, 且并行部分所占的比重较大时引入并行才能较为显著地提高程序的执行效率. 而当程序并行部分不多时进行并行改善的只是可并行部分的执行速度, 对于程序整体的提升并不显著.

## Parallel Instruction

### 设计方法

SIMD(Single Instruction Multiple Data) 对多个数据同时执行同一个数据操作, 其实现为:

1. 在硬件上设置了多word的更长的寄存器和对应的数据操作, 将多个word读取到寄存器中, 此时对一整个寄存器进行同一个数据操作就等于同时对于多个数据执行同一个操作.
2. 设计了Intrinsics函数, 通过调用Intrinsics函数显式地指明要进行SIMD操作

### 优化程序

由于存在SIMD这种并行方式, 所以在

1. 编写for循环的情况下可以增加不长, 一次性读取多个相邻的word存入寄存器并进行数据操作.
2. 考虑到程序的每个循环都会在执行到末尾的情况下进行判断和跳转指令, 所以可以引入Loop Unrolled, 在修改后的一个循环中中执行原先多个循环中的操作, 最后才进行跳转(但这种情况增加了代码量和存储指令的内存大小, 也有其缺陷)

## Parallel Thread

### 定义

就我的理解来看

MIMO (Multiple Instruction Multiple Data) 有两种情况:

1. Program: Compiled, assembled and linked code that’s ready to be loaded and run.

   Multi Processing 对应了用多个处理器并行地运行多个任务

   Chrome & Firefox

2. Threads: Unit of execution within a process.

   Multi Thread 在一个核上同时运行多个任务, 可以看作是一个程序拆分成多个可并行的部分并执行

   Tabs of Chrome

所以同一个Process的不同Threads共享一部分内存(Code, Static), 节约了内存, 也方便线程间进行交互.

### 设计方法

每个不同的Thread都有自己的PC, Reg, Stack

当一个Thread要进行IO等消耗较长时间的操作时, CPU可以切换到另一个可以直接执行的Thread, 通过这种方式充分利用CPU(当然要在Thread的消耗比等待一个Thread要小的情况下)

### 文件锁 & 原子操作

由于不同线程可能会对同一片内存进行读写, 所以会出现Data Races的情况, 在这种情形下就要进行同步

单纯地划出一个内存锁来作为中介不能解决这个问题(因为内存锁依然能被共同访问,这个问题依然是递归地被伸展了)所以可以:

1. Atomic swap of register ↔ memory−Pair of instructions for “linked” read and write▪write fails if memory location has been “tampered” with after linked read (没有涉及)
2. 引入原子操作的概念, 对锁的读写操作只需一条指令, 这一条指令在运行的时候绝对不会被其他线程的其他指令干扰(因为同时只涉及一个核)

ps : OpenMP库留待之后做lab的时候熟悉, Thread对于Cache的影响等
