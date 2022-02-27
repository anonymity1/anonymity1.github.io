---
title: pyplot
date: 2022-01-12 10:58:54
tags: 算法
---

数据不一定很好看，但是图一定要好看。

<!--more-->

## 子图绘制

```python
ax = plt.subplots(nrows:int, ncols:int) 
# ax[row][col]用来画子图，比较麻烦，一般不用。
```

## 基本信息

```python
# 横纵坐标：命名plt.xlabel，设置间隙和坐标名称plt.xticks，
# 图例和网格：图例plt.legend，网格plt.grid
# 图片展示和保存：展示plt.show，保存plt.savefig 
```

## 折线图

```python
# 折线：plt.plot(x, y)
# 标记设置：标记类型marker，标记大小markersize
# 线条设置：宽度linewidth，颜色color，类型linestyle，
```

颜色信息、线条类型设置和标记类型设置如下表

![](/img/pyplot/color.jpg)

![](/img/pyplot/linestyle.jpg)

![](/img/pyplot/marker.jpg)

## 柱状图和条形图

柱状图

```python
# 柱状：plt.bar(left, height, width=0.8, bottom=None)
# 基本：相比折线图，left和height对应x和y，width是柱体的宽度，bottom是柱体的底部
# 柱体设置：颜色facecolor（fc），填充hatch
# 描边设置：颜色edgecolor（ec），宽度linewidth，类型linestyle
```

条形图

```python
# 柱状的旋转变体条形：plt.barh(bottom, width, height=0.8, left=None)
```

常见柱体填充
```python
hatch = ['//','\\\\','xx','--','o','/','+','.','*','-','|']
```

## 散点图
```python
# 散点图：plt.scatter(x, y, s) s表示区域大小
# 点类型：样式marker，宽度linewidth
# 点颜色：颜色color，颜色可以是序列，alpha透明度，cmap='Blues'表示对应的色盘
```

用法举例
```python
plt.scatter(np.random.rand(100), np.random.rand(100), s=(1+np.random.rand(100))*100, c=np.random.rand(100), cmap='viridis', alpha=0.5)
plt.colorbar()
plt.show()
```
