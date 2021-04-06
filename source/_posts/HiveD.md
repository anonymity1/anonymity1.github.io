---
title: HiveD
date: 2021-04-06 10:48:03
tags: osdi2020
---
多租户在GPU集群上进行DL训练是一种较为常见的场景，然而目前GPU集群资源分配是基于quota的，即每个任务指定需要的GPU数目，这样的粗粒度资源调度方式容易造成更长的累积延迟，本文因此提出了一种更细粒度GPU集群资源调度方式。（研究机构：北大，Microsoft，港大）

<!-- more -->

## 动机

这是因为现有在公共GPU集群上为多租户分配训练资源的方式是基于quota的，即分配GPU的数目。公共GPU集群调度器为租户和私有集群里一样多的GPU数目，但是这样的GPU效率可能由于某些原因（单一node计算能力不强，没有足够GPU）造成当前租户任务完成时间增加，从而产生队列累积延迟。

公共GPU集群上可能会有多个node，每个node上有多个GPU。如果一个DL任务需要64个GPU，那么集群调度器产生一个8node，每个node分配8GPU的对应affinity。同时会有些很小的DL任务，这些任务可能只需要1-3个GPU，但是这样的DL任务也会占据一个node，由于很多的node被这样的小DL任务占据，集群调度器无法为需要64个GPU的DL任务的affinity找到对应的配置，所以这个大DL任务只能等待，由此产生了累积延迟。

这个问题似乎是可以被解决的，有些类似于内存管理中的memory fragmentation问题。

（这个动机似乎有点easy），不过底层资源没有池化，而是分层存在，确实是一个现实的问题。

## 解决方案

本文提出了HiveD系统，确保共享GPU执行时是safety的。HiveD将GPU资源分为Virtual Private Cluster（VC）和physical Cluster两层。HiveD将每个租户表示为一个VC，每个VC会预先安排一系列的cell，这里的cell表示需要多少资源。

cell里的资源是分层的，这篇文章作者将资源分为5层：GPU -> PCIe switch -> CPU socket -> node -> Rack，通过分层实现细粒度资源调度。（这个分层有些疑惑，同时每一层内各个cell之间的通信恐怕也需要考虑）

如果将资源认为是池化的，一些常规的调度器，比如k8s default schedular会根据任务需要资源的数量直接分配资源，但是因为GPU集群里资源没有池化，所以需要再封装一层，将node以及更细粒度的资源池化，这也是这篇文章的核心idea。

同时由于分层资源的塔式结构，底层资源与上层资源有隶属关系，需要根据这种隶属关系调整分配的细粒度资源。这篇文章称之为buddy cell，当所有的buddy cell都是空闲的时候，那么就可以合并成一个更粗粒度的资源，在分配细粒度的资源时，通过考虑这种隶属关系，尽可能保持粗粒度的资源没有被占用，从而保证大的DL任务可以被正常执行而不用延迟。

最后是关于低优先级VC，这类VC可以被随时抢占，所以分配cell时，优先分配到那些没有被大DL任务占据的buddy cell当中，这样可以保证大概率不被抢占，同样对于guaranteed VC，优先分配buddy cell里没有低优先级任务的VC，从而降低抢占发生的次数。

## 实验环境

2019年11月实际部署在[openPAI](https://github.com/microsoft/pai)项目中，集群当中有800多台GPU，Nvidia和AMD的，也包括200多台Azure GPU虚拟机。执行的任务包括NLP任务（BERT），AutoML实验，以及几百个单GPU任务。

项目构成：7700行golang代码，以及js，shell脚本和yaml配置文件。

工作流：和k8s的默认调度器配合工作，能够复用k8s调度器的基本逻辑和相关配置。

trace-based experiment: USENIX Annual Technical Conference (ATC' 19）

GPU cluster: 96-GPU cluster 部署在Azure上（24台NC2，NC2：4 个NVIDIA K80）

对比算法：没有用HiveD，而使用Quota的YARN-CS，Gandiva，Tiresias

## 启发

应用到NUMA架构或者非PS架构的GPU集群，云当中应用HiveD。
