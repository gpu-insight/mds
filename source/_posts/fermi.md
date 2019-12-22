---
title: Fermi架构介绍
date: 2019-11-11
tags: [Fermi, GPU]
---

# 介绍
Fermi是NVIDIA公司的GPU架构的名称。取这个名字是为了纪念美籍著名意大利物理学家Erico Fermi。

Erico Fermi造出了人类第一台可控核反应堆---芝加哥一号堆，被称为“原子能之父”，在1938年获得了诺贝尔物理学奖。

NVIDIA声称Fermi架构是世界上第一个完整地支持通用计算的GPU架构。

# 架构模型
{% asset_img "fermi-0.png" "Fermi结构" %}
{% asset_img "fermi-1.png" "SM结构" %}
{% asset_img "fermi-2.png" "SM执行Warp指令" %}

# 编程模型
## Application
Application可以包含一个或多个[Kernel](#Kernel)，通常用来做顶点/像素染色或者通用计算。与常见的CPU程序相比，Application的分支跳转语句相对较少；程序长度相对较短；算数运算类语句相对较多。

Application支持OpenGL、CUDA、OpenCL和Direct Compute API。

## Kernel
一个[Grid](#Grid)执行一个Kernel。这是CUDA和OpenCL中的概念，意指计算算法单元。需要注意的是它和操作系统中的内核不是同一个概念。

Kernel在C语言的基础上扩展了并行运算的语法，以取代串行的循环等语句。除此之外，它还支持C++、Fortran、Java、MATLAB和Python。

## Grid
Grid可以包含一个或多个[ThreadBlock](#ThreadBlock)。与Kernel是一一对应的关系。

## ThreadBlock
ThreadBlock可以包含最多48个[Warp](#Warp)，也就是1536个[线程](线程)。一个ThreadBlock里面的所有线程都运行在同一个SM上，他们之间可以协作和共享存储。

## Warp
一个Warp包含32个[Thread](#Thread)，不同ThreadBlock中的Warp可以并行执行。Warp是SM中的基本调度单元。

## Thread
多个Thread可以并行执行，Fermi采用SIMT（Single Instruction Multiple Threads）架构。Warp可以快速切换，这是因为每个线程都有自己独立的寄存器和私有存储，Warp切换时不需要数据搬运。
