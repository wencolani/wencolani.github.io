---
layout: post
title: (Paper Reading) Knowledge Transfer for Out-of-Knowledge-Base Entities: A Graph Neural Network Approach
date: 2017-09-26 15:32:24.000000000 +09:00
---

* Knowledge Transfer for Out-of-Knowledge-Base Entities: A Graph Neural Network Approach
* Takuo Hamaguchi, Hidekazu Oiwa, Masashi Shimbo, and Yuji Matsumoto
* IJCAI 2017

## Main Idea

这篇文章提出了一个新的问题：关于out-of-knowledge-base entity的预测。以往的knowledge base embedding方法做链接预（link prediction）测和三元组分类(triple classification)时所用的测试数据集里都假设所有的实体的关系都已经在训练数据集里出现过，即每个实体和关系都有相关的向量表示。而out-of-knowlege-base entity 的定义如下：

> out-of-knowledge-base entity problem arises when new entity(OOKB entities) occur in the relation triplets that are given to the system after training.



