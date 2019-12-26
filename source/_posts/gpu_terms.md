---
title: GPU的渲染架构、渲染管线和渲染模式分类
date: 2019-12-23
tags: [TBR, IMR, GPU, 统一渲染, 分离式渲染, 固定功能渲染管线, 可编程渲染管线]
---

通过阅读本文你将能够知道：
1. 统一渲染架构和分离式渲染架构是什么以及二者的区别
1. 固定功能渲染管线和可编程渲染管线是什么以及二者的区别
1. 立即渲染模式和基于TILE的渲染模式是什么以及二者的区别

# NVIDIA GPU历史简单介绍

NVIDIA在1999年推出GeForce 256时，提出了GPU这个概念。

随后NVIDIA陆续推出了GeForce 2系列、GeForce 3系列、GeForce 4系列、GeForce 5系列、GeForce 6系列、GeForce 7系列和GeForce 8系列的GPU。

在推出GeForce 3系列GPU时，NVIDIA采用了可编程的渲染管线。此前的GPU都是固定功能的渲染管线。

在推出GeForce 8系列GPU时，NVIDIA采用了统一渲染架构。此前的GPU都是分离式渲染架构。

也是在推出GeForce 8系列GPU时，NVIDIA开始给GPU的架构命名。架构名都是以在科学发展历史中做出过突出贡献的人的名字来命名。GeForce 8系列的架构名称为[Tesla](https://en.wikipedia.org/wiki/Tesla_(microarchitecture))，这是NVIDIA第一个采用统一渲染的架构。

随后NVIDIA又陆续推出了[Fermi](https://en.wikipedia.org/wiki/Fermi_(microarchitecture) "Fermi (microarchitecture)")、[Kepler](https://en.wikipedia.org/wiki/Kepler_(microarchitecture) "Kepler (microarchitecture)")、[Maxwell](https://en.wikipedia.org/wiki/Maxwell_(microarchitecture) "Maxwell (microarchitecture)")、[Pascal](https://en.wikipedia.org/wiki/Pascal_(microarchitecture) "Pascal (microarchitecture)")、[Volta](https://en.wikipedia.org/wiki/Volta_(microarchitecture) "Volta (microarchitecture)")、[Turing](https://en.wikipedia.org/wiki/Turing_(microarchitecture) "Turing (microarchitecture)")等架构。

Fermi架构是NVIDIA第一个支持通用计算的架构。

在上面的介绍中出现了一些概念：可编程渲染管线、固定功能渲染管线、统一渲染架构、分离式渲染架构，在下面的章节中将详细介绍这些概念。这里提出来是为了让大家看到这些概念出现的时间顺序：
1. 固定功能渲染管线
1. 可编程渲染管线
1. 分离式渲染架构
1. 统一渲染架构

# 固定功能渲染管线和可编程渲染管线

一开始，GPU设计者想要完成顶点渲染这个功能，那么他会选择一个顶点光照算法，如Phong，然后使用硬件实现该算法。这样做的好处很明显：速度快。但一旦你选择了Phong光照算法，那么这个GPU就只支持这个算法，因为GPU的硬件已经固定死了。即使出现了一个更好的光照算法，你也无法更新。这个时候的GPU几乎所有的算法都是硬件加速实现，更准确地说，是把硬件加速的图形算法单元整合在一起组成一个GPU。

然后，可编程渲染管线应运而生。顶点着色和片段着色是可编程的，着色程序都运行在一个微处理器上。通常，这个微处理器被称为Shader Core。用户可以通过编写不同的染色程序自定义图形渲染的效果，这是一件令人兴奋的事情！

# 分离式渲染架构和统一渲染架构

由于顶点着色程序对精度要求较高，而片段着色程序要求较低，并且一般情况下，少量的顶点会生成大量的片段，所以GPU设计者设计了两类Shader Core：一类专门处理顶点着色程序，称为顶点着色器；另一类专门处理片段着色程序，称为片段着色器。并且顶点着色器的数量少于片段着色器的数量。这便是分离式渲染：顶点着色和片段着色在各自专门的硬件单元中进行。

慢慢地会发现，处理小三角形比较多的场景时，顶点着色器利用率很高，而部分片段着色器空闲；处理大三角形比较多的场景或者渲染纹理较多的场景时，顶点着色器部分空闲，而片段着色器利用率很高。

那是不是可以设计一个Shader Core，它既可以做顶点着色，也可以做片段着色以提高硬件资源的利用率？答案是肯定的。

统一渲染由此诞生！

基于统一渲染架构，Shader Core被挖掘出了更多的使用方法，比如通用计算。如果GPU被用来做通用计算，那么GPU中的图形算法硬件加速器（比如光栅化）是不工作的。

# 立即渲染模式和基于TILE的渲染模式

芯片架构通常要考虑三个核心要素：功耗、性能和面积。我们可以简记为PPA（Power、Performance、Area）

一个好的GPU架构需要针对GPU产品的应用场景，在PPA组成的三角形中选择一个好的平衡点。比如移动端的GPU更加注重功耗和面积，而桌面端的GPU更加注重性能。

立即渲染模式的GPU侧重于性能，而基于TILE的渲染模式的GPU侧重于功耗。因此，前者常见于桌面级GPU，而后者常见于移动端GPU。

二者的详细介绍，请参看我的另一篇博客《基于TILE的渲染》。

# 总结

1. 如果顶点/片段着色算法是使用固定的硬件加速器实现，那么该GPU采用的就是固定功能的渲染管线；如果顶点/片段着色算法是可编程替换的，那么该GPU采用的就是可编程的渲染管线。
1. 如果GPU中存在两种类型的Shader Core，一种只能运行顶点着色程序，另一种只能运行片段着色程序，那么该GPU就是分离式渲染架构；如果GPU中的所有Shader Core既可以运行顶点着色程序，也可以运行片段着色程序，那么该GPU就是统一渲染架构。
1. 如果GPU渲染逻辑中，顶层遍历元素是图元，那么该GPU采用的是立即渲染模式；如果GPU渲染逻辑中，顶层遍历元素是TILE，那么该GPU采用的是基于TILE的渲染模式。

# 参考
- [Unified Shader Model](https://en.wikipedia.org/wiki/Unified_shader_model)
- [A look at the PowerVR graphics architecture: Tile-based rendering](https://www.imgtec.com/blog/a-look-at-the-powervr-graphics-architecture-tile-based-rendering/)
- [The Mali GPU: An Abstract Machine, Part 2 - Tile-based Rendering](https://community.arm.com/developer/tools-software/graphics/b/blog/posts/the-mali-gpu-an-abstract-machine-part-2---tile-based-rendering)

