---
id: trigonometry.unit-circle
title: 单位圆：三角函数不只是直角三角形
slug: unit-circle
domain: geometry
level: bridge
status: published
summary: 从圆上一点的坐标理解正弦、余弦、符号与周期。
estimated_minutes: 12
prerequisites:
  - function.graph
related:
  - trigonometry.sine-wave
  - trigonometry.right-triangle.ratios
  - geometry.circle.inscribed-angle
---

## 把角度放到圆上

以原点为圆心，半径为 1。一个从正横轴逆时针转过角度 \(\theta\) 的点，坐标定义为

\[
P=(\cos\theta,\sin\theta).
\]

余弦是横坐标，正弦是纵坐标。这把直角三角形里的定义扩展到了任意角度。

:::demo
type: unit-circle
angle: 45
animate: false
:::

## 投影会告诉你符号

从第一象限转到第二象限，横坐标变成负数，纵坐标仍为正数。于是第二象限的余弦为负、正弦为正。

转一整圈后，点回到原处，因此正弦和余弦都以 \(2\pi\) 弧度为周期。

演示的角度显示使用度数，计算会先换成弧度：\(\theta_{rad}=\theta_{deg}\pi/180\)。

## 恒等式来自圆的方程

因为点始终在单位圆上，圆的方程给出

\[
\cos^2\theta+\sin^2\theta=1.
\]

这条式子直接来自几何，而不是需要孤立记忆的技巧。

## 试着解释

拖动到 90°、180°、270°。每次先预测两种投影的值，再检查演示。

如果半径变为 3，点的坐标会变成什么？为什么正弦、余弦的定义仍可从坐标除以半径得到？
