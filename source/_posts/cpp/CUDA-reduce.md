---
title: 理解CUDA并行机制--以Reduce为例
date: 2023-5-23 23:06:08
tags:
- [CS]
- [CUDA]
- [C/C++]
categories: 
- [CS]
- [CUDA]
- [C/C++]
mathjax: true
description: CUDA Sample给出优化Reduce的七步方法，其中通过循环展开理解Warp与SIMD概念。
top_img: 
cover: /image/CUDA/reduce_cover.png
---

# CUDA Reduce Optimization

Start from Kernal 3: Sequential Addressing

```c++
__global__ void reduce(int* g_idata, int* g_odata) {
    extern __shared__ int sdata[];
    
    unsigned int tid = threadIdx.x;
    unsigned int i = blockIdx.x * blockDim.x + threadIdx.x;
    sdata[tid] = g_idata[i];
    __syncthreads();
    
    for (unsigned int s = blockDim.x / 2; s > 0; s >> 1) {
        if (tid < s) {
            sdata[tid] += sdata[tid + s];
        }
        __syncthreads();
    }
}
```

![reduce](/image/CUDA/reduce.png)

Half of threads is idle during reduce, so we want to halve the number of blocks and replace single load like this(kernal 4):

![reduce](/image/CUDA/first-add.png)

```c++
    unsigned int tid = threadIdx.x;
    unsigned int i = blockIdx.x * blockDim.x * 2 + threadIdx.x;
    sdata[tid] = g_idata[i] + g_idata[i + blockDim.x];
    __syncthreads();
```

Instructions are SIMD synchronous within a warp, so when s <= 32, we can unroll the last warp(kernal 5):

```c++
__device__ void warpReduce(volatile int* sdata, int tid) {
    sdata[tid] += sdata[tid + 32]; 
    sdata[tid] += sdata[tid + 16]; 
    sdata[tid] += sdata[tid + 8]; 
    sdata[tid] += sdata[tid + 4]; 
    sdata[tid] += sdata[tid + 2]; 
    sdata[tid] += sdata[tid + 1]; 
}
```

```c++
for (unsigned int s=blockDim.x/2; s>32; s>>=1) {
    if (tid < s)
    	sdata[tid] += sdata[tid + s];
    __syncthreads();
}
if (tid < 32) warpReduce(sdata, tid);
```

it uses the mechanism of warp, we will talk about it in the next chapter.

When we consider the blockSize, we know that the max number of threads is 1024. So we can completely unroll the loop when compiling. (Using template is a trick to ensure the num of threads.)(kernel 6)

```c++
template <unsigned int blockSize>
__global__ void reduce(int* g_idata, int* g_odata) {
    extern __shared__ int sdata[];
    
    unsigned int tid = threadIdx.x;
    unsigned int i = blockIdx.x * blockDim.x * 2 + threadIdx.x;
    sdata[tid] = g_idata[i] + g_idata[i + blockDim.x];
    __syncthreads();
    
    if (blockSize >= 1024)  {
        if (tid < 512) {
            sdata[tid] += sdata[tid + 512];
        }
        __syncthreads();
    }
    if (blockSize >= 512)  {
        if (tid < 256) {
            sdata[tid] += sdata[tid + 256];
        }
        __syncthreads();
    }
    if (blockSize >= 256)  {
        if (tid < 128) {
            sdata[tid] += sdata[tid + 128];
        }
        __syncthreads();
    }
    if (blockSize >= 128)  {
        if (tid < 64) {
            sdata[tid] += sdata[tid + 64];
        }
        __syncthreads();
    }
    // if blockSize == 64 or less, we can use warp to reduce computation.
    if (tid < 32) {
        warpRuduce<blockSize>(sdata, tid);
    }
    
    if (tid == 0) g_odata[blockIdx.x] = sdata[0];
}
```

# Grid, block, thread and warp in CUDA

From a hardware perspective: 

1. SP(streaming processor): CUDA core. A thread executes on a SP.
2. SM(streaming multiprocessor): One SM has multiple SPs and has SFU, shared memory, register file and **warp scheduler**.

Form a software perspective:

1. thread: a CUDA program has been executed with many threads.

2. block: many threads are grouped into a block. Threads in the same block can call __syncthreads() to synchronize, also, they can use shared\_memory to communicate.

3. grid: many blocks are grouped into a grid.

   ![example](/image/CUDA/1.png)

## Warp:

SPs in SM are divided into many groups(warps), every warp has 32 threads(SP), and SPs in the same warp work together, execute the same instuctions.

Every thread has its own register and local memory.

# Reference

[CUDA编程：深入理解GPU中的并行机制（八） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/455866677)

[理解CUDA中的thread,block,grid和warp - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/123170285)

[CUDA 学习（一）Reduction - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/365581043)

[GPU概念：Thread, Block, Grid, Warp, SP, SM_gpu wrap_Kevyn7的博客-CSDN博客](https://blog.csdn.net/Kevyn7/article/details/115899559)
