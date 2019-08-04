---
title: 图表的 Scale
date: 2019-08-04 13:07:18
tags: 技术
---

Scale 是图表的核心部分，它用于数据和显示的相互映射，数据点绘图位置需要用 Scale 确定，从画布的位置反过来获取对应位置的的数据点，也需要 Scale。Scale 通常是实现了如下的接口:

```
interface IScale {
  domain: [number, number]
  range: [number, number]
  toDomain(rangeValue: number): number
  toRange(domainValue: number): number
}
```

domain 是原始数据的范围，如同函数`f(x) = y`，domain 相当于 x 的取值范围，即定义域；range 相当于函数的输出范围，即值域，在显示时就是 canvas 画图的坐标范围。比如将一组`[1, 2, 3, 4, 5, 6]`的数据映射到画布的`[0, 50]`范围，我们可以说，domain 是`[1, 6]`, range 是`[0, 50]`。

将上面这组数据进行绘图时，要用到`toRange`方法，如果 Scale 是线性的，`f(domain)= range = A * domain + B` 中，A = 10, B = -10, 对于每一个 domain 的输入值，其输出是：

```
f(domain)= 10 * domain - 10 = range
f(1) = 0
f(2) = 10
f(3) = 20
f(4) = 50
f(5) = 40
f(6) = 50
```
这是`toRange`的计算。相应地，`toDomain`的计算是：

```
f(domain)= 10 * domain - 10 = range
g(range) = 0.1 * range + 1
g(0) = 1
g(10) = 2
g(20) = 3
g(30) = 4
g(40) = 5
g(50) = 6
```

这样就完成了线性映射。现在图表里只用到了线性映射，假如要做 Log Scale 功能，就要参考一下  `d3` 的实现。

## 平移
在平移时，range 的范围并没有发生变化，需要变化的 domain，假如画布需要向右移动 10px，原有的画布范围是 50px 宽，显示的是 domain = `[1, 6]`, `f(domain) = 10 * domain - 10`, 在平移后，`f(domain) = 10 * domain`。这时，如果要显示 range = 0 的点，需要 domain = 0；如果需要显示 range = 50 的点，需要domain = 5。所以新的domain = `[0, 5]`。

另一种计算方法是

```
f(domain) = 10 * domain = range
g(range) = 0.1 * range = domain
```
得到

```
g(0) = 0.1 * 0 = 0
g(50) = 0.1 * 5 = 5
```
和上面的分析方法相同。

## 缩放

在平移时，range 的范围并没有发生变化，需要变化的 domain。假如需要画图当大一倍，中心点是 25px，则：

```
对于原来的domian, 在放大后
f'(range) = (range - 25) * 0.5 + 25
f‘(0）= 12.5
f'(50) = 37.5
只显示原来在[12.5, 37.5]的range区间的domain值
因为f(domain) = 10 * domain - 10
即 g(range) = 0.1 * range + 1 = domain
g(12.5) = 2.25
g(37.5) = 4.75
```
所以在放大后，domain 变为`[2.25, 4.75]`，只需要 2.25 到 4.75 的值，就能填满画布。

放大一倍要乘以 0.5，可以理解为固定 domain ，将显示的窗口作倒数倍数变化；比如放大一倍，相当于显示原来的一半区域就满足要求。


## 改变画布大小
改变画布大小时，range 的范围作等量的变化。比如画布增加 50px ，那么新的 range 为`[0, 100]`。

对于 domain 有两种处理方式，一种是 domain 不变，必然地各个点之间的距离变大，出现拉伸效果。
另一种处理是，domain 也跟随变化，保持显示比例。

```
因为f(domain) = 10 * domain - 10
即 g(range) = 0.1 * range + 1 = domain
g(0) = 1
g(100) = 11
```
所以 domain 范围变为`[1, 11]`

上述计算结果中，domain 的取值可能为负数，这里的正负并没有太大意义。在实际应用中，比如`@antv/g2`，会将 domain 和 range 归一化处理。