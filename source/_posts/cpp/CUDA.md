---
title: CUDA快速入门
date: 2023-5-23 22:06:06
tags:
- [CS]
- [CUDA]
- [C/C++]
categories: 
- [CS]
- [CUDA]
- [C/C++]
mathjax: true
description: 通过"CUDA BY EXAMPLE"快速入门CUDA
top_img: 
cover: /image/CUDA/cover.jpg
---

# Example about vector multiplication and reduce

<img src="/image/CUDA/1.png" alt="image-20230523221635232" style="zoom:50%;" />

```c++
// main.cu
#include "kernel.cuh"
#include <stdio.h>
#include <cmath>
#include <iostream>

cudaError_t addWithCuda(int *c, const int *a, const int *b, unsigned int size);

__global__ void addKernel(int *c, const int *a, const int *b)
{
    int i = threadIdx.x;
    c[i] = a[i] + b[i];
}


const int blocksPerGrid = imin(32, (N + threadsPerBlock - 1) / threadsPerBlock);
bool float_equal(float a, float b) {
    if (std::abs(a - b) < 1e1) return true;
    return false;
}

int main()
{
    float* a, * b, c = 0, * partial_c;
    float* dev_a, * dev_b, * dev_partial_c;

    a = new float[N];
    b = new float[N];
    partial_c = new float[blocksPerGrid];

    cudaMalloc((void**)&dev_a, N * sizeof(float));
    cudaMalloc((void**)&dev_b, N * sizeof(float));
    cudaMalloc((void**)&dev_partial_c, blocksPerGrid * sizeof(float));

    for (int i = 0; i < N; i++) {
        a[i] = i;
        b[i] = i * 2;
    }

    cudaMemcpy(dev_a, a, N * sizeof(float), cudaMemcpyHostToDevice);
    cudaMemcpy(dev_b, b, N * sizeof(float), cudaMemcpyHostToDevice);
    
    vector_add << <blocksPerGrid, threadsPerBlock >> > (dev_a, dev_b, dev_partial_c);

    cudaMemcpy(partial_c, dev_partial_c, blocksPerGrid * sizeof(float), cudaMemcpyDeviceToHost);

    for (int i = 0; i < blocksPerGrid; i++) {
        c += partial_c[i];
    }

    // CPU serial
    double result = 0;
    float* res_p = new float[blocksPerGrid];
    for (int i = 0; i < N; i++) {
        if (i == 368) {
            printf("");
        }
        float temp = a[i] * b[i];
        result = result + temp;
    }
    for (int i = 0; i < blocksPerGrid; i++) {
        res_p[i] = 0;
        for (int j = 0; j < threadsPerBlock; j++) {
            res_p[i] += a[i * threadsPerBlock + j] * b[i * threadsPerBlock + j];
        }
    }

    if (float_equal(result, c)) {
        std::cout << "Test pass." << std::endl;
    }
    else {
        std::cout << "Test error. CPU: " << result << ", GPU: " << c << std::endl;
    }
    cudaFree(dev_a);
    cudaFree(dev_b);
    cudaFree(dev_partial_c);

    free(a);
    free(b);
    free(partial_c);
    return 0;
}

// kernel.cuh
#ifndef _KERNEL_CUH_
#define _KERNEL_CUH_

#include "cuda_runtime.h"
#include "device_launch_parameters.h"

#define imin(a,b) (a<b?a:b)
const int threadsPerBlock = 256;
//const int N = 1 * 1024;
const int N = 33 * 1024;

__global__ void vector_add(float* a, float* b, float* c);

#endif // !_KERNEL_CUH_


// vector.cu
#include <cstdio>
#include "kernel.cuh"

extern void __syncthreads();

__global__ void vector_add(float* a, float* b, float* c) {
	__shared__ float res[threadsPerBlock];

	int tid = threadIdx.x + blockIdx.x * blockDim.x;
	int res_idx = threadIdx.x;

	float temp = 0;

	while (tid < N) {
		temp += a[tid] * b[tid];
		tid += blockDim.x * gridDim.x;
	}

	res[res_idx] = temp;
	__syncthreads();

	// reduce
	int i = blockDim.x / 2;
	while (i != 0) {
		if (res_idx < i) {
			res[res_idx] += res[res_idx + i];
		}
		__syncthreads();
		i = i / 2;
	}
	if (res_idx == 0) {
		c[blockIdx.x] = res[0];
	}
}
```

