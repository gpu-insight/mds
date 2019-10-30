
---
title: GPGPU-Sim 安装过程
date: 2019-10-23 14:17:50
tags: GPGPU-Sim
---

# 1. 简介
## 1.1 GPU
GPU英文名称为Graphic Processing Unit，中文名称为图形处理器，主要用于计算机系统中的显示及图形处理，又称为Video Card(显卡)。

GPU具有一下特性：
- 针对高度并行的工作负载进行优化
- 高度可编程性
- 桌面级的超级计算机

## 1.2 GPGPU
异构计算（Heterogeneous Computing）是指在异构计算系统上进行的并行计算。  
GPGPU（General-purpose computing on graphics processing units）是一种利用处理图形任务的图形处理器来计算原本由中央处理器处理的通用计算任务。

{% asset_img "pic01_gpgpu.png" "gpgpu" %}

## 1.3 GPGPU-Sim
GPGPU-Sim 是一个时钟级别的GPU仿真模型，可以运行使用cuda或者OpenCL编写的GPU计算程序。GPGPU的github如下：
https://github.com/gpgpu-sim/gpgpu-sim_distribution

# 2. 安装 GPGPU Sim
本文介绍在虚拟机Centos7上配置安装GPGPU sim环境。  
需要准备环境：
- Linux 环境：Centos 7
- Cuda环境：Cuda Toolkit 7.5 https://developer.nvidia.com/cuda-downloads
- GPGPU sim：https://github.com/gpgpu-sim/gpgpu-sim_distribution （branch: dev）

## 2.1 安装Cuda
GPGPU-Sim支持的Cuda版本有：4.2, 5.0, 5.5, 6.0, 7.5, 8.0, 9.0, 9.1。
``` bash
yum localinstall cuda-repo-rhel7-7.5-18.x86_64.rpm
yum install cuda-tooklit-7-5
```
## 2.2 编译GPGPU-Sim
#### 2.2.1 安装依赖库
安装依赖:
```
# GPGPU-Sim dependencies:
yum install gcc
yum install gcc-c++
yum install make
yum install makedepend
yum install xorg-x11-utils
yum install bison
yum install flex
yum install zlib

# GPGPU-Sim documentation dependencies:
yum install doxygen
yum install graphviz

# AerialVision dependencies:
yum install python-pmw
yum install python-ply
yum install python2-numpy
yum install libpng12-devel
yum install python-matplotlib
```
#### 2.2.2 编译
编译前需要设置环境变量
```
export CUDA_INSTALL_PATH=/usr/local/cuda
export PATH=${CUDA_INSTALL_PATH}/bin:${PATH}
source setup_environment
```
之后可以直接使用make进行编译：
```
make -j8
```
## 2.3 运行demo
cuda的helloworld程序如下
```
/* file: hello.cu */
#include <stdio.h>
#include <cuda_runtime.h>

__global__ void add(int a, int b, int *c)
{
    *c = a + b;
}

int main()
{
    int c;
    int *dev_c;
    cudaMalloc((void **)&dev_c, sizeof(int));
    add<<<1, 1>>>(2, 7, dev_c);
    cudaMemcpy(&c, &dev_c, sizeof(int), cudaMemcpyDeviceToHost);
    printf("2 + 7 = %d\n", c);

    return 0;
}
```
编译程序时指定`--cudart shared`确保可执行程序动态链接到CUDA runtime库。
```
nvcc --cudart shared -o hello hello.cu
```
上文2.2.2中通过`source setup_environment`设置可执行程序连接到GPGPU-Sim编译的libcudart.so中。
运行ldd确保链接正确的libcudart.so:
```
ldd hello
```
运行程序之前需要拷贝GPGPU-Sim路径下的`configs/tested-cfgs/SM2_GTX480/`的配置文件到当前运行demo的路径下。
最后运行编译的可执行程序即可正常仿真:
```
./hello
```
