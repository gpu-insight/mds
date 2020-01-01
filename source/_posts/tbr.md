---
title: 基于TILE的渲染
date: 2019-12-22
tags: [TBR, GPU]
---

# 缩写

| 缩写 | 说明 |
| --- | --- |
| GPU | Graphics Processing Unit，图形处理单元。 |
| SC | Shader Core，渲染核。GPU中的微处理器，负责执行顶点着色程序、片段着色程序和通用计算程序。 |
| TBR | Tile-based Rendering，基于TILE的渲染。 |
| IMR | Immediate Mode Rendering，立即渲染模式。 |

# TILE

TILE的必应词典翻译是：地砖、瓦片。在这里，TILE是指将整个屏幕分成若干个小格。

TBR是将渲染一屏图像变换为渲染屏幕中的所有TILE。GPU中的一个SC渲染一个TILE，所以GPU中的SC越多，并行渲染TILE的能力就越强。

我们知道，屏幕显示的内容是帧缓冲区中的所有像素颜色，实际上，除了颜色之外，还有像素的深度和模板也会存储在显存中。

将整个屏幕划分成若干个TILE意味着，将颜色、深度、模板缓冲区划分成若干个小的存储。

## TILE的大小

一般来说，TILE的大小为32x32或者16x16。有的情况下，TILE还可以是矩形。

总之，TILE的大小一般都不大。

TILE足够小则意味着这个TILE对应的颜色、深度、模板缓冲区的存储在片上也可以放一份。这样做有什么好处呢？

## TILE的好处

假设GPU当前渲染的帧中包含100个三角形，且开启了深度模板测试和混合，那么，传统的渲染模式（立即渲染模式，IMR）每渲染一个三角形就需要从显存中读取这个三角形所覆盖的所有像素的颜色、深度和模板，渲染之后还需要立马写回显存。

耗时耗力！

{% asset_img "imr_data_flow.png" "立即渲染数据流" %}

而TBR会逐个遍历TILE，首先看都有哪些三角形覆盖了当前遍历到的TILE，最坏的情况下，某个TILE被100个三角形所覆盖，那么遍历这100个三角形，每渲染一个三角形(TILE覆盖部分而非三角形全部)则将结果放到这个TILE对应的片上高速缓存中，直到遍历完所有的三角形，最后将结果写回显存中的帧缓冲区。

可以看到，TBR将读写显存的频率大大降低。这首先带来的好处是低功耗，其次是高效率。

{% asset_img "tbr_data_flow.png" "分块渲染数据流" %}

# TBR

TBR的处理逻辑是这样的：

```c++
for (auto tile : tiles) {
    auto primitives = tile.getPrimitiveList();

    for (auto primitive : primitives) {
        auto fragments = primitive.getFragmentsInTile(tile);

        for (auto fragment : fragments) {
            fragment.render();
        }
    }
}
```

而IMR的处理逻辑是这样的：

```c++
for (auto drawcall : drawcalls) {
    auto primitives = drawcall.getPrimitives();

    for (auto primitive : primitives) {
        auto fragments = primitive.getFragments();

        for (auto fragment : fragments) {
            fragment.render();
        }
    }
}
```

可以看到TBR处理逻辑的第二行有一个操作是：`tile.getPrimitiveList`，这个操作是为了获取当前帧中覆盖了这个TILE的所有图元。

假设当前帧中有100个三角形，当第一个三角形所覆盖的TILE全部统计好了，也不能`立即渲染`，而需要等这100个三角形的覆盖情况都统计完成才能开始遍历TILE。这是因为如果直接开始遍历就和IMR的逻辑是一样的了，失去了TBR的优势。遍历TILE的前提便是每个TILE要能够完整表示屏幕中的一小部分，如果当前TILE中的内容还没准备好就直接开始渲染，其渲染结果虽然也可以放在片上，但是你不知道何时将片上存储的内容写回显存。

使用TBR渲染模式的GPU，通常会将染色好的顶点写回显存，然后再读出来将其构建成图元，根据图元的覆盖情况构建每个TILE的图元列表，图元列表也会被写回显存。一帧的所有TILE的图元列表构建完成后，再让SC读取出为其分配的TILE的图元列表。而采用IMR渲染模式的GPU是没有读写图元列表操作的。

虽然TBR多了读写图元列表的操作，但是相比于节省下来的读写缓冲区的操作，还是很划算的。

# 参考

- [A look at the PowerVR graphics architecture: Tile-based rendering](https://www.imgtec.com/blog/a-look-at-the-powervr-graphics-architecture-tile-based-rendering/)
- [The Mali GPU: An Abstract Machine, Part 2 - Tile-based Rendering](https://community.arm.com/developer/tools-software/graphics/b/blog/posts/the-mali-gpu-an-abstract-machine-part-2---tile-based-rendering)
