---
title: 几个关于C++语言特性的小问题
date: 2023-01-29 22:05:57
tags: Essay
---

宏，静态库，内存分配和Core Dump

<!--more-->

## 宏

除了作为变量替换之外，宏还可以作为字符串替换：有两种情况。

第一种，被替换对象是被标点或空格分割开的“独立字符串”，用#表示。
第二种，被替换对象被一个独立字符串真包含，用##表示，当被替换对象位于独立字符串开头，则在被替换对象的结尾加##表征；当被替换对象位于独立字符串结尾，则在被替换对象的头部加##表征；当被替换对象位于独立字符串中间，则在被替换对象的开头和结尾都加##表征。

例子：

```C++
#define REGISTER_LAYER_CREATOR(type, creator) \
  LayerRegisterer g_creator##type(#type, creator); \
```

## 静态库

静态库方便移植，速度快，但是体积大，编译慢，这个都知道。除此之外，静态库中的static变量不会被事先声明，如果在头文件定义static变量，可能有重复定义的风险。

怎么解决呢？制作一个只有可能被应用程序引用的头文件，在其中定义需要预先定义的static变量。

例子：不同layer初始化注册map表。

```C++
/* register_layer.h */

#ifndef REGISTER_LAYER_H
#define REGISTER_LAYER_H

#include "u1/layer_factory.h"
#include "u1/layer/full_layer.h"
// ...

REGISTER_LAYER_CLASS(Full);
// ...

#endif
```

## 内存分配

1. C++为了避免手动free或者delete的问题引入std::shared_ptr<>，即智能指针。智能指针调用对象方法时和一般指针一样，变成一般指针时用.get()方法.

2. 当具有独显时：

- C++仅有分配内存的能力，显存的分配用CUDA接口；

- gpu计算单元不能访问内存，cpu也不能访问显存，

- 显存和内存可以互通，但是显存分配不提供智能指针，仅提供原始类型的指针。

## core dump

一般逻辑问题查找日志基本可以解决。Core Dump没有日志报错信息，或者日志容易出问题。这时需要用调试工具。

使用gdb+cmake编译时，添加-g编译选项，如下段代码。用ulimit -c nolimited取消core dump文件的大小限制。更改/proc/sys/kernel/core_pattern的core dump文件输出位置和命名，然后用gdb <程序名> <修改后的coredump文件名> 进入gdb调试。在gdb中用bt查找调用栈，一般能定位内存出问题的位置。

```python
# CMakeLists.txt
ADD_DEFINITIONS(-g -D_REENTRANT -D_FILE_OFFSET_BITS=64 -DAC_HAS_INFO
        -DAC_HAS_WARNING -DAC_HAS_ERROR -DAC_HAS_CRITICAL -DTIXML_USE_STL
        -DAC_HAS_DEBUG -DLINUX_DAEMON)
```




