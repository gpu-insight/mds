---
title: Life of A Triangle - Nvidia's Logical pipeline
date: 2019-12-14
tags: GPU
---
# 前言
自从具有突破性的Fermi架构发布已经过去5年了，是时候更新它低层图形体系结构的原理了。Fermi是第一个实现完全可扩展图形引擎的Nvidia GPU，并且它的核心架构可以在Kapler和Maxwell中找到。文章后面特别是下面图片“压缩管线知识”必须作为各种公共资料的基础，例如关于GPU架构的白皮书或者是GTC教程（GPU Technology Conference）。本文关注图形视角下的GPU如何工作，尽管Shader程序如何执行的一些原理与计算相同。
- [Fermi Whitepaper](http://www.hardwarebg.com/b4k/files/nvidia_gf100_whitepaper.pdf)
- [Kepler Whitepaper](http://www.nvidia.com/content/PDF/kepler/NVIDIA-Kepler-GK110-Architecture-Whitepaper.pdf)
- [Maxwell Whitepaper](http://international.download.nvidia.com/geforce-com/international/pdfs/GeForce_GTX_980_Whitepaper_FINAL.PDF)
- [Fast Tessellated Rendering on Fermi GF100](http://international.download.nvidia.com/geforce-com/international/pdfs/GeForce_GTX_980_Whitepaper_FINAL.PDF)
- [Programming Guidelines and GPU Architecture Reasons Behind Them](http://international.download.nvidia.com/geforce-com/international/pdfs/GeForce_GTX_980_Whitepaper_FINAL.PDF)

# 管线架构图
{% asset_img "fermipipeline.png" "Fermi结构" %}

# GPU 是超级并行工作站
为什么这么复杂呢？因为在图形阶段我们必须处理由于创建大量变量导致的数据扩充。每一次的drawcall可能产生不同数量的三角形。在裁剪之后的三角形的数量与原始创建的三角形数量不同。经过背面剔除和深度剔除之后，并不是所有的三角形都需要显示在屏幕上。三角形的屏幕尺寸就意味着它需要数百万的像素或者不需要任何像素。

因此，现代GPU使图元（点、线、三角形）遵循逻辑管线，而非物理管线。在早期G80统一渲染架构之前的硬件中（Dx9硬件，ps3，xbox360），渲染管线在芯片中以不同的阶段表示，任务会按照顺序从一个阶段传到下一个阶段，G80本质上根据负载来重用顶点着色和片段着色组件的一些单元，但实际上它仍然有一个串行的图元/光栅化过程。在Fermi架构中，管线变成完全并行，这意味着芯片通过重复使用用芯片上的多个引擎来实现逻辑流水线。

假设有两个三角形A和B，它们的部分工作可能处于不同的逻辑管线阶段。A已经经过转换并且需要进行光栅化，它的某些像素可能已经在运行像素着色器指令，而其他的一些像素可能被深度缓存(Z-cull)拒绝，其他的一些像素可能已经被写入帧缓冲区，有一些可能还在等待。接下来，我们可以获取三角形B的顶点。因此，尽管每个三角形都必须经过逻辑步骤，但是很多三角形可以在其生命周期的不同步骤中被处理。渲染三角形到屏幕上的任务可以分成许多小的任务甚至是可以并行运行的小的任务。将每个任务调度到可用资源，而资源不限于某种类型的任务（像素着色与顶点着色并行运行）。

想想一下散开的河流。并行管线流之间是相互独立的，每一条都有自己的时间线，有一些可能会比其他的有更多分支。如果我们将GPU进行处理所基于的三角形部分的单元或者drawcall当前正在处理的单元进行代码着色的话，看起来会像是七彩的闪烁灯。

# GPU体系结构

{% asset_img "fermipipeline_maxwell_gpu.png" "Fermi结构" %}

由于NVIDIA具有类似的原理架构，有一个Giga Therad Engine可以管理所有正在进行的工作。GPU划分为多个GPC（Graphics Processing Cluster），每一个GPC有多个SM(Streaming Multiprocessor)和一个光栅引擎（Raster Engine）。处理器中有很多的互联网络，最值得一提的是Crossbar，它允许跨GPC或其他功能单元（如ROP子系统（Render Output Unit））进行工作迁移。

程序员想要执行的任务(着色程序的执行)实在SM中进行的，它包含许多核，这些核执行线程的数学运算。一个线程可以是顶点或线程着色器的调。这些核和其他单元由Warp调度器驱动，它将一组32个线程作为Warp管理，并将要执行的指令移交给调度单元。代码逻辑由调度程序处理，而不是在内核内部进行处理，调度器只会看到类似"将寄存器4234与寄存器4235相加并存储在4230中"，与非常智能的CPU核相比，GPU核显得相当弱。GPU将智能性提升到更高的层次，它进行的整个集合的工作。

GPU中实际存在多少个这样的单元（每个GPC中有多少个SM，GPU中有多少个GPC）取决于芯片的配置。上图中的GM204有4个GPC，每个GPC中有4个SM。但Tegra X1具有1个GPC和2个SM。两者均采用Maxwell设计，SM设计本身（核的数量，指令集，调度器）也随着时间的推移一代又一代的发生着变化（图1），并且帮助提高芯片的效率，使其可以从高端台式机扩展到笔记本电脑再到移动设备。

# 逻辑管线
为了简单起见，省略了一些细节，我们假设drawcall引用一些索引缓存或者顶点缓存已经填充了数据并且存放在GPU的DRAM中，并且仅使用顶点或者片段着色器。

{% asset_img "fermipipeline_begin.png" "Fermi结构" %}

1. 程序通过图形API（DX/GL）调用drawcall。Drawcall在某个时刻到达驱动中，驱动程序会进行一些检查以验证是否合法。并且将命令以GPU可读的编码形式放进命令缓冲区(pushbuffer)。GPU端可能会出现很多瓶颈，这就是为什么程序员必须很好地使用api和利用当前GPU的功能的技术很重要的原因。
2. 
---
参考资料：
https://developer.nvidia.com/content/life-triangle-nvidias-logical-pipeline