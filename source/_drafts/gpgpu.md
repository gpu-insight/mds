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

