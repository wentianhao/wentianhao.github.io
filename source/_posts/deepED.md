---
title: Deep Joint Entity Disambiguation with local Neural Attention
katex: true
tags:
  - NLP
  - 实体链接
categories: 论文笔记
abbrlink: 8814
date: 2020-12-09 10:52:56
---

提出一种新的深度学习文档级实体消歧模型

## Key components
- Entity Embeddings：上下文单词和候选实体嵌入到同一公共向量空间
- 局部上下文的注意力机制
- 联合概率推理模型
<!-- more -->

## Contributions
- 实体嵌入
- 上下文注意力机制用于局部消歧
- 提出深度学习框架进行全局消歧

## Local Model

![Local Model](https://whh.plus/images/LocalModel.png)

mention-entity prior $\hat{p}(e|m)$ ：统计mention和entity hyperlink计数统计数据求平均概率(Wikipedia and YAGO dictionary)

soft Attention: $u(w)=max_{e∈\Gamma(m)}x_e^TAx_w$

hard Attention: $\overline{c}={w∈c|u(w)∈topR(u)},R\leq K$

weighted sum: $\beta(w)=\left\{\begin{matrix}\frac{exp[u(w)]}{\sum_{v∈\overline{c}}exp[u(v)]}. \text{ }  if w∈\overline{c}\\0   \text{ }     otherwise.\end{matrix}\right.$

$\beta$-weighted context-based entity-mention score ： $\Psi(e,c)=\sum_{w∈\overline{c}}\beta(w)x_e^TBx_w$

## Local Score Combination

$$\Psi(e,m,c)=f(\Psi(e,c),log\hat{p}(e|m))$$

## Document-Level Deep Model

### CRF Model
![CRF Model](https://whh.plus/images/crf-lbp.png)
$g(e,m,c)=\sum_{i=1}^n\Psi_i(e_i)+\sum_{i<j}\Phi(e_i,e_j)$
$$\Phi(e,e')=\frac{2}{n-1}x_e^TCx_{e'}$$

### Differentiable Inference

$m_{i->j}^{t+1}(e)=\underset{e'∈\Gamma(m_i)}{max}\{\Psi_i(e')+\Phi(e,e')+\underset{k\ne{j}}{\sum}\overline{m}_{k->i}^t(e')\}$

$\overline{m}_{i->j}^t(e)=log[\delta*softmax(m_{i->j}^t(e))+(1-\delta)*exp(\overline{m}_{i->j}^{t-1}(e))]$