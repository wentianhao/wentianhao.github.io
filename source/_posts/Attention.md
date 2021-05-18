---
title: Attention Is All You Need
katex: true
date: 2020-10-10 09:59:03
updated: 2020-10-10 09:59:03
tags:    
    - NLP
categories: 论文笔记
---

![attention](https://whh.plus/images/attention.png)

## Introduction

传统的基于RNN的Seq2Seq模型难以处理长序列的句子，无法实现并行，还面临对齐的问题，本文提出Attention机制，加入Attention的Seq2Seq模型在各个任务上都有显著提升，创新点在于抛弃传统的Encoder-Decoder模型必须结合CNN或RNN的固有模式，只用Attention机制。
<!-- more -->

## Background

传统的编码器解码器一般使用RNN，这也是在机器翻译中最经典的模型，但RNN本身也存在局限，这类模型的发展大多从三个方面入手：

- input的方向性：单向或双向
- 深度： 单层或多层
- 类型：RNN，LSTM或GRU

由于无论输入如何变化，encoder给出的都是一个固定维数的向量，存在信息损失；在生成文本时，生成每个词所用到的语义向量都是一样的，这显然过于简单。

为解决上述问题，引入Transformer的概念，整个网络结构完全是由Attention机制组成，Attention机制是将单个序列的不同位置联系起来计算序列表示的一种注意机制。

![Attention](https://whh.plus/images/attention.jpg)

## Transformer
> Transformer is the first transduction model relying entirely on self-attention to compute representations of its input and output without using sequence aligned RNNs or convolution。

### 整体结构

先看看总体的结构(以论文中的机器翻译为例)

![Transformer](https://whh.plus/images/theTransformer.png)

Transformer本质是一个Encoder-Decoder的结构

![Transformer1](https://whh.plus/images/Thetransformer1.png)

编码器和解码器都是由六个部分组成的，编码器的输出作为解码器的输入

![Transformer2](https://whh.plus/images/Thetransformer2.png)

每个encoder中有self-attention和feed forward neural network，数据先经过self-attention模块，得到一个加权之后的特征向量$Z$，即$Attention(Q,K,V)$

$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}})V$

然后送入前馈神经网络，前馈神经网络有两层，第一层的激活函数是Relu，第二层的激活函数为线性激活函数，表示为：
$$FFN(Z)=max(0,ZW_1+b_1)W_2+b_2$$

Encoder结构如下：
![encoder](https://whh.plus/images/encoder.png)

Decoder的结构，相较于Encoder，多了个Encoder-Decoder Attention，两个Attention分别用于计算输入和输出的权值。

- Self-Attention : 当前翻译和已经翻译的前文之间的关系
- Encoder-Decoder Attention : 当前翻译和编码的特征向量之间的关系

Decoder结构如下：
![decoder](https://whh.plus/images/decoder.png)

### 输入编码

上面说明的是主要的网络框架，现在介绍数据输入。一般使用嵌入算法把每个输入字转化成向量
![embedding](https://whh.plus/images/embedding.png)

词嵌入的维度$d_{model}=512$，嵌入单词到输入序列，每个单词都会输入到每个编码器的两层
![encoder](https://whh.plus/images/encoder1.png)
Transform的关键特性，每个位置的词仅流过自己的编码器路径。self-attention中这些路径两两之间相互依赖，但前馈层没有这些依赖，因此各种路径在流过前馈层时并行执行。

#### 编码过程

一个向量序列作为Encoder的输入，将向量传入self-attention处理，进入FFNN，然后再将输出向上传到下一个Encoder
![encoder](https://whh.plus/images/encoder2.png)

### self-attention

self-attention是Transform最核心内容。Attention的核心是**输入向量的每一个单词学习一个权重**

例如：
>The animal didn't cross the street because it was too tired

加权后得到类似如下加权情况：

![attention](https://whh.plus/images/attention1.png)

当模型处理单词“it”时，self-attention允许将“it”和“animal”联系起来。当模型处理每个位置的词时，self-attention允许模型将句子的其他位置信息作为辅助线索来更好的编码当前词。

当编码“it”时(编码器的最后层输出)，部分attention集中于“the animal”，并将其表示合并进入到“it”的编码中

[Tensor2Tensor notebook](https://colab.research.google.com/github/tensorflow/tensor2tensor/blob/master/tensor2tensor/notebooks/hello_t2t.ipynb)

#### self-attention细节

根据编码器的输入向量。生成三个向量，query-vec,key-vec，value-vec，生成方法为分别乘以三个矩阵，这些矩阵在训练过程中需要学习。【不是每个词向量独享3个矩阵，而是所有输入共享3个转换矩阵；权重矩阵是基于输入位置的转换矩阵；】新向量的维度比输入词向量的维度要小(512->64)，并不是必须要小，是为了让多头attention的计算更稳定
![QKV](https://whh.plus/images/QKV.png)

$$Query_{vec}=W_Q*q_1$$

$$Key_{vec}=W_K*k_1$$

$$Values_{vec}=W_V*v_1$$

![QKV](https://whh.plus/images/QKV1.png)

Attention计算过程

- 将单词转化为嵌入向量(Embedding)
- 根据嵌入向量得到三个输入向量$q,k,v$
- 为每个向量计算一个score，$Q$与所有词的$K$以此点积得到
- 为了梯度的稳定，Transformer使用score归一化，即$/\sqrt{d_k}$
- 对score进行softmax激活函数
- softmax点乘value值$v$，得到加权的每个输入向量评分$v$(scaled Dot-Product Attention)
- 相加之后得到最终输入结果

![Self-Attention计算示例图](https://whh.plus/images/computeAttention.png)
$$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}})V $$

![Self-Attention矩阵表示](https://whh.plus/images/softmax.png)

#### 残差

在self-attention需要强调的最后一点是采用了残差网络中的short-cut结构，目的是为了解决深度学习中的退化问题。

![Self-Attention中的short-cut连接](https://whh.plus/images/short-cut.png)

向量和self-attention的相关图层可视化如下：

![Self-Attention中的short-cut连接](https://whh.plus/images/short-cut1.png)

Transformer可视化如下图：

![Transformer](https://whh.plus/images/transformer1.png)

## Multi-Head Attention

Multi-Head Attention相当于8个不同的self-attention的集成

- 将数据X分别输入到8个self-attention中，得到8个加权后的特征矩阵
![第一步](https://whh.plus/images/multi-head1.png)
- 将8个矩阵按列拼成一个大的特征矩阵
![第二步](https://whh.plus/images/multi-head2.png)
- 特征矩阵经过一层全连接后得到输出$Z$

整个过程：

![Multi-Head Attention](https://whh.plus/images/multiAttention.png)

同Self-Attention一样，Mukti-Head Attention也加入了short-cut机制

现在加入Attention heads后，重新看下当编码“it”时，哪些attention head会被集中
![it](https://whh.plus/images/multi-head3.png)

编码“it”时，一个attention head集中于“the animal”，另一个head集中于“tired”

## Positional Encoding

Transformer模型没有捕捉顺序序列的能力，无论句子怎么打乱，得到的结果都是类似的。为了解决这个问题，论文提出位置编码概念。

位置编码常见模式有两种：A.根据数据学习 B.自己设计编码规则。论文作者采用第二种方式，通常位置编码的长度为$d_{model}$的特征向量，便于和词向量进行单位相加
![位置编码](https://whh.plus/images/position.png)

$$PE(pos,2i)=sin\frac{pos}{10000^{\frac{2i}{d_{model}}}}$$

$$PE(pos,2i+1)=cos\frac{pos}{10000^{\frac{2i}{d_{model}}}}$$

$pos$ : 单词的位置

$i$ : 单词的维度




![Transformer](https://whh.plus/images/transformer.png)_模型架构图_

**参考文献**
1.[The Illustrated Transformer](https://zhuanlan.zhihu.com/p/75591049?from_voters_page=true)
2.[Attention is all you need-详解Transformer](https://www.cnblogs.com/zhanghaiyan/p/11079504.html)