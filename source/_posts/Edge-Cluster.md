---
title: heterogeneous IoT
date: 2021-10-12 16:03:09
tags:
---

几篇关于边缘集群调度的论文，一起总结一下。

DeepThings: Distributed Adaptive Deep Learning Inference on Resource-Constrained IoT Edge Clusters(TCADICS 2018)

Adaptive Parallel Execution of Deep Neural Networks on Heterogeneous Edge Devices(SEC 2019)

DeepSlicing: Collaborative and Adaptive CNN Inference With Low Latency(TPDS 2020)

Towards Efficient Inference: Adaptively Cooperate in Heterogeneous IoT Edge Cluster(ICDCS 2021)

背景：集群调度框架或算法，边缘+层级粒度+水平或纵向划分。

<!-- more -->

## 研究背景

边缘设备计算资源有限，无法支持来自多台终端的多个模型在时延限制内完成推理任务。在具有**并行能力的边缘集群**执行这些推理任务是一种流行的解决方案。如何设计一种有时延保障的**面向边缘集群的推理模型调度**方案是实现终端多模型高效协同推理的关键环节。

## 系统框架

DeepThings(TCAD 2018)

在具有**轻量级内存资源**的物联网边缘集群上分块执行各卷积层。针对早期卷积层的内存占用问题，将原始CNN层堆栈分割成**独立可分配**的执行单元，每个执行单元在每个物联网设备内具有**更小的内存占用**和最大的内存重用。同时根据集群内节点处理这些小分区的进度，动态平衡边缘集群节点之间的工作负载，实现在时变处理需求下边缘集群的分布式CNN推理。

![](/img/edge-cluster/DeepThings.png)

DeepSlicing(TPDS 2020)

延续了DeepThings的工作，在系统设计上支持具有DAG结构的CNN模型的定制化划分加速，同时为用户提供了一组API，用于获取**实时任务状态**和**数据位置**、**执行细粒度调度**和**及时回收内存**，从而为用户提供了定制自己的调度策略的能力。

## 调度算法

Adaptive Parallel Execution of DNNs(SEC 2019)

针对边缘集群的**通信开销**问题，提出一种自适应权衡**跨层融合**和**层内分区**的动态规划算法，根据异构边缘的计算能力和网络条件决定DNN模型的最佳分区和并行化方法。

![](/img/edge-cluster/Adaptive-Parallel.png)

Towards Efficient Inference(ICDCS 2021)

提出了一种用于在异构物联网边缘集群上进行CNN推理的**流水线**合作方案(PICO)，并设计一种基于动态规划的算法在最大时延约束和动态工作负载条件下，**最大化吞吐量**的最佳并行策略。

流水线模型在最大时延限制下，寻找并行方案优化最大吞吐量的建模方式

![](/img/edge-cluster/formula.png)
