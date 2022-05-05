---
title: nn-Meter
date: 2021-10-10 22:17:52
tags: 论文阅读
---

nn-Meter: Towards Accurate Latency Prediction of Deep-Learning Model Inference on Diverse Edge Devices (Mobisys 2021 Best paper)

背景：理论，单设备，边缘+推理+推理时间预测。思路亮点：找融合算子，测融合算子运行时间，指出算子计算时延和算子配置存在非线性关系。

<!-- more -->

## 研究背景

边缘端实现高效**模型推理**是实现边缘智能的关键环节。推理延迟成为各类移动端和边缘设备上衡量DNN推理模型是否高效的关键指标，如何**准确预测推理延迟**是设计各类高效神经网络的关键问题。

## 研究问题

但是目前预测神经网络模型推理延迟存在以下几个问题：

- 对于不同的边缘设备（例如，移动CPU/GPU和各种AI加速器）和不同的推理框架（例如，TFLite和OpenVINO），**工程量**是巨大的。即使在单个设备上，测量NAS任务中的大量模型也可能非常耗时，在实际工作中基于测量的方法不可行。

- 基于FLOPs的预测方法被广泛应用，这是一种简单的方法，但并不是延迟的直接度量，因为底层硬件可能存在的各类并行优化使得此类方法**不准确**。

- 基于神经网络基本操作符的推理延迟预测不考虑计算图的运行时**操作符融合**优化，从而导致模型延迟差异。

- 基于图形卷积网络（GCN）预测各种设备上NASBench201数据集的延迟是最近出现的工作，但是这种基于模型图的方法很大程度上依赖于已测试的模型结构，可能不适用于许多看不见的模型结构。

总之，已有工作或者在实际中不可行，或者没有考虑底层各类运行时优化，或者对未知模型缺乏预测理论依据，因此需要建立一种适用于边缘设备底层**算子融合优化**，具有**理论依据**的推理延迟**准确预测**工具。

## 算法设计的思路和挑战

下图系统框架展示了实现DNN模型精确延迟预测的两个核心组件：**内核检测**和**自适应数据采样**。从概念上讲，前者自动将目标模型划分为一组内核，后者从大空间中采样最有利的配置，以构建准确的内核级延迟预测器。然后，对于每个内核，提取特征并预测延迟。

![](/img/nn-Meter/arch.png)

- 内核检测。

    它包括精心设计的测试用例来检测**两个算子之间的融合规则**（融合后即内核），以及一个搜索模型中所有核的算法。通过离线收集所有的融合规则，对于在线模型预测，核搜索算法递归地将这些规则应用于目标模型以查找所有核。

    寻找包含融合规则的核有两个难点：许多推理后端是**闭源代码**，无法从源代码中获得内核；另外CNN模型任意，为了支持模型推理时间预测，检测融合规则方法应该独立于特定模型图。

- 自适应数据采样。

    受限离线为目标设备上的所有内核构建基于机器学习的时延预测器。对于每个内核，它通过一个迭代的采样过程来采样最有益的配置(h,w,cin,cout,stride,k)。采样器从先验可能性分布中采样，该分布描述了模型CNN设计中考虑的内核配置。我们设计了一个测试集来评估采样数据的质量，对于**预测误差较大**的数据，在其周围执行**细粒度**通道数采样。

    造成误差较大的原因是部分操作符的推理时延和一些参数并不是简单的线性模式，而是**阶跃式**函数，这种非线性反映了硬件优化的复杂性。

## 融合规则检测判据和算子配置方法

由于融合通过将同一元素上的计算连接在一起，从而减少了延迟，因此使用连接和分离运算符的运行时间差作为判断是否发生融合的度量。也就是说，对于两个操作符op1和op2，如果操作符的时间遵循下式，则它们被视为被融合为Op1++Op2。

![](/img/nn-Meter/formula.png)

然后从计算图的根节点遍历整张图，依次根据此判据判定不同算符是否有融合规则。

![](/img/nn-Meter/1.png)

![](/img/nn-Meter/2.png)

如以上两张图所示，如果直接随机采样会忽略许多关键数据，拟合得到的预测结果不准确。由于算子配置在样本空间中呈现非均匀分布（一些极端情况的采样实际不会用到），首先采用**剪枝**将这类情况排除，然后运行一个**迭代过程**来围绕不准确的预测数据采样得到更多的数据，直到预测精度满足要求。对于非线性数据关系，采用**随机森林**为每个内核构建推理实验预测器。

最终所有内核的预测推理时延求和就是一个模型的推理时延。

## 复现和思考

没有复现，原因有二：首先对一个模型进行精准预测，和调度关系不是很大，可以作为调度的支撑，但不是主线；其次代码依赖较多，复现需要耗费时间较长。

另外这篇文章首先扩大问题规模说明推理延迟预测算法的必要性，然后在方法阶段提出用剪枝剪去现实中不存在的可能性，有些离谱。

不过收集数据集不容易，工作量还是有的。