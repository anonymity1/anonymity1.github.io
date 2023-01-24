---
title: caffe中blob的初始化方式
date: 2023-01-24 15:53:49
tags: Essay
---

caffe，又称blobflow。

<!--more-->


## 正文

caffe是用来实现各种各样神经网络前向推理，和反向传播的框架。神经网络由各种layer组合构成，caffe自身提供了卷积、全连接等层的实现。

由于layer的是存储密集、计算密集型的任务，且适用并行加速，因此涉及到cpu、内存、显存和gpu之间的切换。另外layer的输入、输出，自身权重都是高维张量构成，而数据在内存和显存的存储为一维结构，因此维度、数据的长度、显存和内存的同步这三个因素需要被考虑。

caffe用blob表征权重、输入和输出等张量。blob逻辑上描述了张量维度、展平后的数据长度。也描述了blob数据对应的导数数据，如果只做前向传播的话这部分数据是用不到的。blob的默认初始化不分配内存和显存，张量维度和展平后数据长度默认为0。blob的显示初始化对应分别对应权重张量和输入输出张量的存储分配。

权重的张量维度、展平后的数据长度在模型文件中被定义，因此caffe定义第一种初始化和保存方式：

```C++
void FromProto(const BlobProto& proto, bool reshape = true);
void ToProto(BlobProto* proto, bool write_diff = false) const;
```

针对前向推理以及反向传播的中间计算结果、即输入张量和输出张量，caffe定义了第二种初始化方式：这种初始化方式调用了Reshape函数，Reshape当中会为数据分配内存和显存。

```C++
explicit Blob(const vector<int>& shape);

template <typename Dtype>
Blob<Dtype>::Blob(const int num, const int channels, const int height,
    const int width)
  // capacity_ must be initialized before calling Reshape
  : capacity_(0) {
  Reshape(num, channels, height, width);
}
```

和第一种方式的区别是这种方式只定义张量维度，不对数据的值做要求，采用这种接口的原因是中间张量的数据并不是我们需要保存的权重，只是需要在显存中开辟张量存储的空间。

由于前向推理时大部分Layer的设计中，（1）输出张量的维度依赖输入张量的维度，（2）并且上一层的输出张量一般是下一层的输出张量，因此Layer在初始化时，一般要求输入张量bottom的维度确定并完成内存显存分配，在调用SetUp函数时再确定输出张量top的维度并完成存储空间分配。多个Layer构成的推理流在初始化时，也要遵循这一设定。

```C++
void SetUp(const std::vector<Blob*>& bottom, const std::vector<Blob*>& top) {
  CheckBlobCounts(bottom, top);
// bottom blobs need to be allocated shape and data memory before, where top blobs don't.
  LayerSetUp(bottom, top); 
  Reshape(bottom, top);
}
```
