---
id: probability.conditional
title: 条件概率：改变你正在看的范围
slug: conditional-probability
domain: probability
level: bridge
status: published
summary: 先明确条件筛选后的样本范围，再理解条件概率与常见的方向混淆。
estimated_minutes: 12
prerequisites: []
related:
  - function.graph
  - probability.equally-likely
  - probability.statistics.variance
---

## 信息改变了分母

考虑一个简化的有限模型：100 个等可能结果中，事件 \(B\) 包含 20 个结果，其中 8 个同时属于 \(A\)。

得知 \(B\) 已发生后，我们只在这 20 个结果里讨论。于是

\[
P(A\mid B)=\frac{P(A\cap B)}{P(B)}=\frac{8}{20}=0.4.
\]

条件概率的分母是条件事件的概率，而且要求 \(P(B)>0\)。

## 为什么不能交换条件方向？

如果 \(A\) 一共有 10 个结果，那么

\[
P(B\mid A)=\frac{8}{10}=0.8.
\]

两次的交集相同，筛选后的总范围却不同，所以结果不相等。

## 独立是一种额外关系

若 \(A\) 与 \(B\) 独立，则 \(P(A\cap B)=P(A)P(B)\)。当 \(P(B)>0\) 时，这等价于 \(P(A\mid B)=P(A)\)：知道 \(B\) 并不改变对 \(A\) 的概率判断。

不能从“两个事件名称不一样”判断独立，也不能从“互斥”判断独立。两个非零概率的互斥事件，一旦知道一个发生，另一个就不可能发生。

## 检查理解

重新画出 100 个格子的示意图，圈出 \(A\)、\(B\) 与它们的交集。用两种颜色标出两次条件概率的分母。

不使用公式，你能解释为什么两个方向的条件概率可能不同吗？
