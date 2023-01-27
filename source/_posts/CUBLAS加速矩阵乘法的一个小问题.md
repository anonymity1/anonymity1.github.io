---
title: CUBLAS加速矩阵乘法的一个小问题
date: 2023-01-27 14:14:49
tags: Essay
---

矩阵、张量在内存和显存中均以一维方式存储。

<!--more-->

在CUBLAS中，GPU的计算单元读取矩阵的方式是按列读取和放回显存。我们习惯内存、显存以及cpu以行优先的方式读取。CUBLAS提供的API就不会很习惯，因此做一个封装使得外界使用时仍然按照习惯写法，但是内部仍用CUBLAS加速。

采用的方式调用CUBLAS的API的时候将A和B顺序反过来。这是因为CUBLAS读取的是转置，若$C=AB$，则$C^T=B^T \times A^T$，而$C^T$在GPU计算单元写回的时候是列优先顺序，则按照行优先读取的时候就是$C$，所以在封装内部调用CUBLAS的API的时候将A和B顺序反过来即可。

代码如下

```C++
void acc_gemm(
  const cublasOperation_t cuTransA, const cublasOperation_t cuTransB, 
  const int M, const int N, const int K, 
  const float alpha, const float* A, const float* B, 
  const float beta, float* C
) {
  int ldb = (cuTransB == CUBLAS_OP_N) ? N : K;
  int lda = (cuTransA == CUBLAS_OP_N) ? K : M;
  cublasHandle_t cublas_handle_;
  CHECK_CUBLAS(cublasCreate(&cublas_handle_));
  CHECK_CUBLAS(cublasSgemm(cublas_handle_, cuTransB, cuTransA, N, M, K, &alpha, B, ldb, A, lda, &beta, C, N));
  CHECK_CUBLAS(cublasDestroy(cublas_handle_));
}
```