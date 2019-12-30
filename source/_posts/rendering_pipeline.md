---
title: GPU渲染管线
date: 2019-12-30
tags: [GPU, 渲染管线]
---

通过阅读本文，你将能够知道：
1. 可编程渲染管线和固定功能渲染管线分别是什么以及二者的区别
1. OpenGL渲染管线和GPU渲染管线的区别

# 渲染管线分类
渲染管线可分为可编程渲染管线和固定功能渲染管线。详细介绍请参看[《GPU的渲染架构、渲染管线和渲染模式分类》]([https://www.jianshu.com/p/08fab2c70d0a](https://www.jianshu.com/p/08fab2c70d0a)
)

# OpenGL渲染管线

我们可以在各种地方看到`OpenGL渲染管线`这个关键词：计算机图形学、OpenGL官方网站、各大GPU厂商的官网等等。你会发现他们说的都不太一样。

## 计算机图形学视角
OpenGL最初的内部流程是一组有序的处理步骤，这些步骤被组织为一个双通道的渲染流水线。流水线阶段当然是固定的---也就是说，它们无论接受到什么样的输入数据都会执行特定的操作---因此，这样的工作方式成为固定功能的OpenGL流水线。

![计算机图形学视角下的OpenGL固定功能渲染管线](https://upload-images.jianshu.io/upload_images/1293315-d1a726141c4cf19a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中，黑色方框部分一般称为**几何流水线**，Geometry Pipeline；橘色方框部分一般称为**像素流水线**，Pixel Pipeline。

![计算机图形学视角下的OpenGL可编程渲染管线](https://upload-images.jianshu.io/upload_images/1293315-dc6b1881319df7d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中，绿色方框部分是可编程的。应用程序通过使用Shader来控制这些阶段内所进行的操作。Shader指的是一些比较短的程序段，它们被加载到OpenGL程序中，并最终加入到OpenGL流水线的适当的处理单元中，替换掉流水线中原来的固定功能。

## OpenGL官方视角

## 各大GPU厂商视角

# GPU渲染管线

# 总结

# 参考
- 《计算机图形学》第四版
