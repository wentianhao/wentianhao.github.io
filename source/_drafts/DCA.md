---
title: DCA
date: 2021-05-31 09:28:39
tags:
categories:
---


## 实验

Datasets: AIDA CoNLL-YAGO (AIDA-train：946个文档，AIDA-A：216，AIDA-B：231)  

五个流行的公共数据集评估：AQUAINT,MSNBC,ACE2004,CWEB,WIKI

| Dataset | #mention | #doc | Mentions per doc | Gold recall
| :---: | :---: | :---: | :---: | :---: |
AIDA-train | 18448 | 946 | 19.5 | -
AIDA-A | 4791 | 216 | 22.1 | 97.3
AIDA-B | 4485 | 231 | 19.4 | 98.3
MSNBC | 656 | 20 | 32.8 | 98.5
AQUAINT | 727 | 50 | 14.5 | 94.2
ACE2004 | 257 | 36 | 7.1 | 90.6
CWEB | 11154 | 320 | 34.8 | 91.1
WIKI | 6821 | 320 | 21.3 | 92.4

对比方法：

- AIDA-light
- WNED *
- Global-RNN
- MulFocal-Att:多焦点注意力机制模型
- Deep-ED *
- Ment-Norm  *
- RLEL: 基于深度强化学习的LSTM模型

超参设置：

word、entity embedding : 300(维度)

K=7,I=5,H1=100，dropout=0.2 (最佳)


-------
local_ent_scores : 上下文局部得分 [7,8]

ent_coherence ： 积累实体和候选实体一致性得分 [1,8]

kng_coherence: 对角矩阵为知识库实体矩阵  [1,8]

p_e_m

tm sum(mt_emb,et_emb)

inputs cat([local_ent_scores,ent_coherence,kng_coherence,p_e_m,tm])  [8,5]

score 




