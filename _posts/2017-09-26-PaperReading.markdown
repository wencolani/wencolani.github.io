---
layout: post
title: (Paper Reading) Knowledge Transfer for Out-of-Knowledge-Base Entities, A Graph Neural Network Approach
date: 2017-09-26 15:32:24.000000000 +09:00
---

* Knowledge Transfer for Out-of-Knowledge-Base Entities: A Graph Neural Network Approach
* Takuo Hamaguchi, Hidekazu Oiwa, Masashi Shimbo, and Yuji Matsumoto
* IJCAI 2017

## Main Idea

这篇文章提出了一个新的问题：关于out-of-knowledge-base(OOKB) entity的预测。以往的knowledge base embedding方法做链接预（link prediction）测和三元组分类(triple classification)时所用的测试数据集里都假设所有的实体的关系都已经在训练数据集里出现过，即每个实体和关系都有相关的向量表示，如果出现新的实体，那么模型需要进行重新训练。

out-of-knowlege-base entity 的问题如下：

> out-of-knowledge-base entity problem arises when new entity(OOKB entities) occur in the relation triplets that are given to the system after training.

解决OOKB的问题主要在于如何从已有的实体和关系的向量表示里获得OOKB实体的向量表示，本文主要利用了OOKB实体和知识库中已有的实体的链接关系得到其向量表示。

## Proposed Model

本文借鉴了[Graph-NN]的方法, 将生成OOKB entity的向量表示的过程分为propagation model 和 output model.

### Propagation model

Propagation model 分为两个部分： trandition function 和 pooling function.

#### transition function

transition function 的主要目标是通过每个与OOKB实体具有链接关系的三元组中已有的实体和关系推测出当前OOKB实体的向量表示。具体定义如下：
* <img src="http://www.forkosh.com/mathtex.cgi? T(\textbf{v}) = \textbf{v}"> --- (identity)
* <img src="http://www.forkosh.com/mathtex.cgi? T(\textbf{v}) = tanh(\textbf{Av})"> --- (single tanh layer)
* <img src='http://www.forkosh.com.mathtex.cgi? T(\textbf{v}) = ReLU(\textbf{Av})'> --- (single ReLU layer)
这里的transition function 也可以换成其他的神经网络，比如batch-normalization、residual connection 和 LSTM等。

注意参数A的定义可以不是一个固定的全局的参数，而是与当前OOKB实体参与的三元组中的关系相关的 (注：这样肯定能提高效果的). 在single ReLU layer 中本文也添加了batch normalization.

#### pooling function

pooling function 主要目的是通过多个OOKB entity的表示结果生成一个向量表示，pooling的方法主要有以下三种：

* sum pooling: <img src='http://www.forkosh.com/mathex.cgi? P(s) = \sum_{i=1}^N x_i '>
* acerage pooling: 
* max pooling

(文中也使用了stacking and unrolling，不过不太了解)

### output model
output model借用了TransE的方法，使用|h + r -t| 作为score function，并且采用margin-based objective function. 相对于之前TrasnE的pairwise-margin，本文提出了absolute-margin，这样的定义同样可以使得可以生成更多的负样本，可以提高最终embedding的效果。
