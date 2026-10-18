---
id: limit.intuition
title: 极限：关心靠近时发生什么
slug: limit-intuition
domain: calculus
level: bridge
status: published
summary: 把“函数在一点的值”与“靠近这一点的趋势”分开，理解极限的真正问题。
estimated_minutes: 12
prerequisites:
  - function.graph
related:
  - calculus.derivative.local-linearization
---

## 一个看起来无法代入的式子

考虑

\[
f(x)=\frac{x^2-1}{x-1},\qquad x\ne1.
\]

直接代入 \(x=1\) 会出现分母为零。但我们可以问另一个问题：**当输入接近 1 时，输出接近哪里？**

当 \(x\ne1\) 时，因式分解得到 \(f(x)=x+1\)。因此输入靠近 1 时，输出靠近 2。

## 趋势与点上的值分开

我们写

\[
\lim_{x\to1} f(x)=2.
\]

这句话不要求原函数在 1 处有定义。就算人为定义 \(f(1)=100\)，靠近 1 时的趋势仍然是 2。

“可以靠近”也不等于“永远碰不到”。极限研究的是邻近输入的行为；函数是否在该点取值，是另一个问题。

## “足够接近”应该有多准确？

给定你要求的输出误差 \(\varepsilon>0\)，我们需要能找到一个输入范围 \(\delta>0\)，使得所有满足 \(0<|x-1|<\delta\) 的输入都有 \(|f(x)-2|<\varepsilon\)。

在本例中 \(|f(x)-2|=|x-1|\)，所以选择 \(\delta=\varepsilon\) 就能做到。这不是有限次试算，而是对整个邻域的保证。

## 检查理解

如果定义 \(g(x)=1\)（\(x<0\)）和 \(g(x)=2\)（\(x\ge0\)），那么在 0 两侧看到的趋势不同，所以双侧极限不存在。

用自己的话解释：为什么改变 \(g(0)\) 一个点的值，无法修补两侧趋势的冲突？
