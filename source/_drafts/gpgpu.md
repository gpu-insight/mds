# Microarchitecture Timing Model

## Objectives

- Explain wraps and branch divergence(coherence)
- Summarize the microarchitecture modeled by GPGPU-Sim
- Group the microarchitecture components into the different clock domains
    * which microarchitecture components: warp scheduler, scoreboard, operand collector, SIMD functional units
    * which clock domains: ???
- Explain why a DRAM timing model is needed
    * What is a DRAM timing model?

## Thread Hierarchy

1. Kernel = grid of blocks of warps of threads.

block: indexed by (x, y)
thread: indexed by (x, y, z)

2. Each block ==> a SIMT core as a unit of work

3. Warp = Scalar threads grouped to execute in lockstep

What is lockstep?

fault-torerant computer systems that run the same set of operations at the same time parallel

4. SIMD vs. SIMT

dont care INSTRUCTION WIDTH

eg.

add.16 arr1 arr2

arr1[i] + arr2[i]

5. Inside a SIMT Core (3.0)

5.1 Fetch and Decode

Each warp corresponds to a PC, ARB arbitrate from which PC to fetch. Based on the valid PC, fetch
instruction from the memory into I-Cache

Fetched instruction is decoded and stored in I-Buffer, I-Buffer is separated warp by warp, one warp's
I-Buffer may contain multiple entries. On instruction issue, only one instruction in a warp is issued.

6. ALU pipelines (Algrithm Logic Unit) 它是一条指令的执行单元(EX)

所以这里的Pipeline应该指的是指令流水线发射的


- ALU有两个功能单元: SP(除了超越函数外的所有指令)， SFU(只执行超越函数)
- SP每个时钟周期执行一条指令
- SFU多个时钟周期执行一条指令
- A SIMT Core = One SP + One SFU
- 两个单元的Input是分开的，Output是共享的

内存指令(Load, Store, Barrier)被发射到memory pipeline, 其他指令被发射到SP pipe

7. Memory Pipeline (LDST unit) 

- each SIMT core有4种不同的L1 memory: shared memory, data cache, constant cache, and texture cache.

- Shared Memory

in NVIDIA GPUs有16/32 banks, GPGPU-Sim可以配置

每个时钟周期一个Bank只提供一个地址

在一个时钟周期内对一个Bank的多个访问，会产生Bank Conflict

- Global Memory

片外DRAM, 最大且最慢的存储

8. 合并(Coalescing)

一个Warp里的线程对内存的访问可以合并，例如对连续的4-byte sized的区域访问，可以合并成128-byte的一个请求

- Write-back vs. Write through

Write-back 数据只是被写入到cache block ，有一个dirty bit，只有当dirty bit 被设置时，当write miss 时才会把数据写回主存，所以最新的数据有可能在处理器Cache，也有可能在主存里。当一个read发生时，如果
最新的数据在处理器Cache ，处理器就必须阻止主存回应read请求

Write-through 有存总有一份最新数据的copy

9 MSHR(Miss Status Holding Registers)


