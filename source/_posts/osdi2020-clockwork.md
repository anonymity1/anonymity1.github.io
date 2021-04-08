---
title: clockwork
date: 2021-04-07 10:31:59
tags: osdi2020
---
Serving DNNs like Clockwork: Performance Predictability from the bottom up (OSDI' 2020) ，德国Max-Planck 软件系统研究所和美国Emory university合作的一篇文章。

由于卷积运算的复杂性，在应用层面执行DNN模型推理可能会存在一定的时延，当同时有多个DNN模型需要执行的时候，调度不同的DNN模型就会产生累计时延和尾延迟（tail latency），从而导致无法满足SLO（service-level objectives）。本文观察到DNN模型的执行时间是可以预测的，但是由于现有系统在多个层次上执行不同的调度策略，导致预估DNN模型执行时间非常困难，因此本文从底层构建了一个DNN推理模型调度系统，实现对DNN模型的执行时间预测，从而实现了更高质量的DNN模型调度。

<!-- more -->

## 动机和方法

预测DNN推理模型执行时间对于执行。为什么已有系统难以实现对DNN推理模型执行时间的预测？从底层的Hardware level，OS level到顶层的Application level，在硬件层面指令乱序执行，操作系统层面各个进程竞争执行，以及应用层面竞争需要的计算存储资源，多级的竞争和调度使得预测一个DNN推理模型的执行时间非常艰难，所以这篇文章提出打破各个层面的松耦合，构建一个紧耦合系统，实现在顶层的DNN调度，从而可以确保对推理模型执行时间的预测。

## 实验

宣称26 KLOC C++代码，实验环境私人小型集群。

12 servers：每个server有32 cores，768G RAM,  2 v100 GPUs

## 感受

各种奇葩单词看的我有点难受，阅读体验极差，没有找到开源实现，影响力有待验证。
