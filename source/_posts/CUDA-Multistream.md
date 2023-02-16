---
title: CUDA Multistream
date: 2023-02-16 13:48:48
tags: Essay
---

CUDA利用多流提升并行效率，在执行基于深度神经网络的推理上，效果不是很明显。

<!--more-->

## 正文

MultiStream机制**理论上**可以使用在时空图上未被利用的资源。如下图所示

![](/img/CUDA-Multistream/时空图.jpg)

横轴代表运算器，纵轴表示不同的时间阶段。在一段时间内，GPU使用蓝色的计算资源执行推理任务，而红色的计算资源则是浪费掉的计算资源。

在串行结构下，该部分资源被浪费掉，运算器在该时间段内属于空转状态。

Multistream就是希望多个流并行，使得推理1的大核和推理2的小核可以同时执行。也就是红色区域尽可能少。

## 基于caffe的多流推理实现

caffe组织推理的过程分为四层结构：syncedmem -> blob -> layer -> net

syncedmem是最底层的线性数据结构，用来维护cpu和gpu的存储同步。blob是更高一层的抽象，维护张量的形状和数据，是所有张量操作的基本类型。layer是构成神经网络的基本单元，net就是最终的推理结构。

Caffe除了完成推理，还要执行训练，对异构设备的适应，以及对单独CPU执行的支持，因此Caffe对于内存进行syncedmem这一层的封装。我们只需要在前向推理时实现多流并发，syncedmem这一层封装可以不需要，可以在blob内部完成内存和显存的数据转化和分配。

除此之外，内存分配、释放、执行都需要指定流的标号，对应API也需要更改。

```C++
// Malloc: cudaMalloc()
cudaMallocAsync(&data_gpu_ptr_, sizeof(float) * count_, stream);
// Free: cudaFree()
cudaFreeAsync(&data_gpu_ptr_, stream);
// Memcpy: cudaMemcpy()
cudaMemcpyAsync(data_gpu_ptr, data_cpu_ptr, sizeof(float) * count_, cudaMemcpyHostToDevice, stream);
// Exec: 
/* curand; cublas; ...*/
im2col_kernel<<<CUDA_GET_BLOCKS(num_kernels), CUDA_NUM_THREADS, 0, stream>>>();
```










