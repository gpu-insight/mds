---
title: Running CUDA Samples Based on GPGPU-Sim
date: 2020-04-02 20:19:04
tags: GPGPU-Sim
---

# CUDA Samples
CUDA Samples是示范CUDA编程概念的软件包。自CUDA 4.1开始，CUDA Toolkit搭载了CUDA Samples安装包。CUDA Samples的目录结构如下：

- 0_Simple
- 1_Utilities
- 2_Graphics
- 3_Imaging
- 4_Finance
- 5_Simulations
- 6_Advanced
- 7_CUDALibraries
- EULA.txt
- Makefile
- bin
- common

# Incompatible GCC Version
如果你安装的是CUDA Toolkit 9.0, 它所兼容的GCC Version最高到 gcc-6. 如果你的环境是Ubuntu 19.10，那么在编译CUDA Samples之前，你需要降级GCC Version到6以下，方法如下：
```bash
sudo apt install gcc-4.8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 10
```
`update-alternatives` 更新`/etc/alternatives`目录下的`gcc`链接指向`/usr/bin/gcc-4.8`, 它的最后的参数`10`是指priority.

NOTE: 安装CUDA Toolkit时会进行GCC Compatibility检查，但是可以通过下面选项忽略这个检查：
```bash
sudo sh cuda_9.0.176_384.81_linux.run --override
```

# Build Samples
构建CUDA Samples可以在顶层目录构建所有Samples, 也可以到单个目录下构建一个Sample. 为了能够让可执行文件能够链接到GPGPU-Sim的`libcudart.so`,需要这样运行构建命令：
```bash
make NVCCFLAGS="--cudart shared"
```
编译好Sample后，将`GPGPU-Sim/configs/tested-cfgs`目录下的任意一个配置目录下所有文件拷贝到Sample所在目录，就可以直接基于GPGPU-Sim运行CUDA Sample.

References:
[0. GPGPU-Sim安装过程](/gpgpusim_install/)
