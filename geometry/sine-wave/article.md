---
id: trigonometry.sine-wave
title: 正弦波：圆周运动如何变成波
slug: sine-wave
domain: geometry
level: bridge
status: published
summary: 连接单位圆与周期变化，理解振幅、频率、相位分别控制什么。
estimated_minutes: 12
prerequisites:
  - trigonometry.unit-circle
related:
  - calculus.derivative.local-linearization
---

## 一个方向上的投影

单位圆上一点匀速转动。如果只记录它的纵坐标，就得到随角度变化的正弦函数。把时间放到横轴上，正弦波就出现了。

## 三个参数，各自控制一件事

\[
y=A\sin(\omega x+\varphi).
\]

- 振幅 \(A\) 决定上下变化的最大幅度，这里演示约定 \(A\ge0\)。
- 角频率 \(\omega\) 决定输入变化时，角度变化多快。周期为 \(T=2\pi/|\omega|\)（\(\omega\ne0\)）。
- 相位 \(\varphi\) 决定起始位置。\(\omega>0\) 时，增大相位让图像向左平移。

:::demo
type: sine-wave
amplitude: 1
frequency: 1
phase: 0
animate: false
:::

## 先预测，再调节

1. 振幅从 1 变成 2，周期会变化吗？
2. 频率从 1 变成 2，同一段横轴里能看到多少个周期？
3. 相位增加 \(\pi/2\)，输入为 0 时的输出是多少？

每一次都先预测，再用滑块检验。只观察动画而不提出问题，往往会产生“看懂了”的错觉。
