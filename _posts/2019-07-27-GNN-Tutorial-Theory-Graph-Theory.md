---
layout: article
title: GNN 教程：图神经网络不能用来学习什么?
key: GNN_tutorial_theory_cannot_learn
tags: GNN
category: blog
pageview: true
date: 2019-07-27 12:00:00 +08:00
---

## 1 引言

这篇论文讨论图神经网络能力的限制，论文中讨论的图神经网络模型是所有通过消息传递构建的图神经网络模型。首先，在当模型的深度(层数)，宽度(每层Embedding的维度)，节点的可区分度，层与层之间网络的复杂程度(表达能力)都达到一定条件的时候，GNN 是图灵完备的。另外，研究发现当GNN的深度和宽度受限的手气，其表达能力会被大幅削弱。这篇论文主要的贡献是讨论了两个方面：

- **图神经网络能够学到什么** 

  第三节会证明GNN是一个通用近似函数，能够近似任何可以被图灵机近似的函数。具体来说，GNN是一个通用近似函数如果一下四个强条件能够达到 1)层数足够多 2)每一层足够宽 3) 每个节点都是可区分的(与其他节点不同) 4) 每一层内的计算函数具有足够的表达能力

- **图神经网络不能学到什么**

  第四节讨论了在层数 $d$ 和深度 $w$ 不足的情况下，GNN的能力会被限制住。假设 $G$ 是神经网络的输入。第四节讨论了下述问题的下界：

  - 检测图中是否有特定长度的环
  - 验证给定 $G$ 的子图是哪一种图，比如连通图(connected graph)，包含环(circle)，生成树(spanning tree)，二部图(bipartite graph)，路径(path)
  - 两个节点之间近似的最短路径，最小割，最小生成树
  - 查找最大独立集(maximum independent set)，最小顶点覆盖(minimum vertex cover)，或者节点着色(chromatic coloring)
  - 计算 $G$ 的近似半径(diameter) ，近似周长(girth)



## 2 GNN结构



## 3  GNN 是通用近似函数






















