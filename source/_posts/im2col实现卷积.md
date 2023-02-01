---
title: im2col实现卷积
date: 2023-02-01 13:04:05
tags: Essay
---

im2col实现繁琐，逻辑复杂，有必要记下推导过程。

<!--more-->

## 正文

二维卷积的朴素实现包括7个循环：batch_size，输出通道数量、输入通道数量、卷积核高、卷积核宽、沿宽方向滑动距离、延高方向滑动距离。以此类推，三维卷积9个循环。

这么多循环放在CUDA中，不利于优化，且缓存不命中，频繁访存执行效率低。考虑到**卷积核和输入张量对应点的相乘，与向量点乘的运算过程是一样的**，而矩阵乘法是向量点乘的堆叠，因此可以设计一种先将输入转化成矩阵，然后执行矩阵的相乘加速卷积速度的方法。

## 推导过程

方法改变，结果不改变，应保证转化+矩阵相乘与朴素实现的输出结果一样。

朴素卷积输出张量$O$的维度是

[batch_size, conv_out_channels, spatial_output_shape_h, spatial_output_shape_w]，

卷积层保存的权重$W$维度是

[conv_out_channels, conv_in_channels, kernel_h, kernel_w]。

卷积核的维度就是

[conv_in_channels, kernel_h, kernel_w]

设$O = W \* D$，$D$是输入张量转化而来形成的矩阵。考虑到张量在内存和显存中均是一维存储，可以将$W$视作行数为conv_out_channels，列数为conv_in_channels \* kernel_h \* kernel_w的矩阵，$O$视作行数为batch_size \* conv_out_channels，列数为spatial_h \* spatial_w的矩阵。

转化得来的矩阵$D$的行数需要是$W$的列数，列数是$O$的列数，才能满足$O = W \* D$的表达式。同时左矩阵的行向量与右矩阵的列向量点乘需要等价于卷积核和张量对应区域的相乘。

考虑batch_size中的输入张量独立，因此可将$O$划分成batch_size个张量，$D$的行数为conv_in_channels \* kernel_h \* kernel_w，列数为spatial_h \* spatial_w，从而首先确定维度。其次，根据$O$中列号对应的输出张量空间位置，计算在**原始输入张量的空间位置中**的哪些元素与卷积核相乘，在输入张量中按行展开这些元素，按卷积核通道序号次序放入列号对应的列向量。

计算方式如下代码所示。

```C++
/* compute the output_spatial_shape */
int height_col = (height + 2 * pad_h - (dilation_h * (kernel_h - 1) + 1)) / stride_h + 1;
int width_col = (width + 2 * pad_w - (dilation_w * (kernel_w - 1) + 1)) / stride_w + 1;

CUDA_KERNEL_LOOP(index, conv_in_channels * height_col * width_col) {
  /* compute the first position in input spatial tensor */
  const int c_im = h_index / height_col / width_col; // the number of conv_in_channels
  const int h_offset = h_col * stride_h - pad_h;
  const int w_offset = w_col * stride_w - pad_w;
  const float* data_im_ptr = data_im;
  
  data_im_ptr += (c_im * height + h_offset) * width + w_offset; // the first position
  /* the corresponding position */
  for (int i = 0; i < kernel_h; i++) {
    for (int j = 0; j < kernel_w; j++) {
      int h_im = h_offset + i * dilation_h;
      int w_im = w_offset + j * dilation_w;
      *data_col_ptr = (h_im >= 0 && w_im >= 0 && h_im < height && w_im < width) ?
        data_im_ptr[i * dilation_h * width + j * dilation_w] : 0;
      data_col_ptr += height_col * width_col; // next row
    }
  }
}
```

从输出张量反推输入张量位置是计算的关键步骤，对应即是h_offset、w_offset以及data_im_ptr的计算过程。卷积的整体前向传播过程即分为两步。

```C++
void ConvolutionLayer::Forward(const std::vector<Blob*>& bottom, const std::vector<Blob*>& top, cudaStream_t stream) {
  const float* weight = this->blobs_[0]->gpu_data();
  for (int i = 0; i < bottom.size(); i++) {
    const float* bottom_data = bottom[i]->gpu_data();
    float* top_data = top[i]->mutable_gpu_data();
    for (int n = 0; n < num_; n++) {
      // bottom_data -> col_buffer_
      // weight * col_buffer_ = top_data 
      im2col(bottom_data + n * bottom[0]->count(channels_), channels_, 
        bottom[0]->shape(channel_axis_ + 1), bottom[0]->shape(channel_axis_ + 2),
        kernel_h_, kernel_w_, pad_h_, pad_w_, stride_h_, stride_w_, dilation_h_, dilation_w_,
        col_buffer_->mutable_gpu_data(), stream
      );
      acc_gemm(CUBLAS_OP_N, CUBLAS_OP_N, 
        conv_out_channels_, conv_out_spatial_count_, kernel_count_, 
        1.0, weight, col_buffer_->mutable_gpu_data(), 0.0, top_data, stream
      );
      if (bias_term_) {
        acc_gemm(CUBLAS_OP_N, CUBLAS_OP_N, 
          conv_out_channels_, conv_out_spatial_count_, 1,
          1.0, this->blobs_[1]->gpu_data(), bias_multiplier_->gpu_data(), 
          1.0, top_data + n * conv_out_channels_ * kernel_count_, stream  
        );
      }
    }
  }
}
```