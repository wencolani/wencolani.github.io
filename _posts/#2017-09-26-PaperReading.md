---
layout: post
category: Paper reading
title: Knowledge Transfer for Out-of-Knowledge-Base Entities, A Graph Neural Network Approach
tagline: by wencolani
tags: [paper, incremental, KG embedding]
---


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
* $ T(\textbf{v}) = \textbf{v} $ --- (identity)
* $ T(\textbf{v}) = tanh(\textbf{Av}) $ --- (single tanh layer)
* $ T(\textbf{v}) = ReLU(\textbf{Av}) $ --- (single ReLU layer)
这里的transition function 也可以换成其他的神经网络，比如batch-normalization、residual connection 和 LSTM等。

注意参数A的定义可以不是一个固定的全局的参数，而是与当前OOKB实体参与的三元组中的关系相关的 (注：这样肯定能提高效果的). 在single ReLU layer 中本文也添加了batch normalization.

#### pooling function

pooling function 主要目的是通过多个OOKB entity的表示结果生成一个向量表示，pooling的方法主要有以下三种：

* sum pooling: <script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"> \\( P(S) = \sum_{i=1}^N x_i \\)</script>
* acerage pooling: 
* max pooling

(文中也使用了stacking and unrolling，不过不太了解)

### output model
output model借用了TransE的方法，使用|h + r -t| 作为score function，并且采用margin-based objective function. 相对于之前TrasnE的pairwise-margin，本文提出了absolute-margin，这样的定义同样可以使得可以生成更多的负样本，可以提高最终embedding的效果。

## Experiment
本文也进行了常规的knowledge base embedding的三元组分类的实验，即没有OOKB实体存在，在WN11和FB13上的实验均比TransE要好一些。但是我更关注OOKB实体的问题，所以主要介绍关于OOKB实体的实验。
### 数据集的构造
本文实验数据集的构造过程非常值得借鉴。以WordNet11s数据集为例， 构造过程如下：

1. 选取OOKBentity：从WN11的test数据集中选取N=1000, 3000, 5000三元组。然后有三种策略选取OOKB entity：
	1. 将选取出的三元组中的所有head entity当作是OOKB实体
	1. 将选取出的三元组中的所有tail entity当作是OOKB实体
	1. 将选取出的三元组中的所有head entity 和 tail entity均当作OOKB实体
然后去掉候选OOKB实体中与任何非OOKB实体都没有链接的实体。

1. 将原来训练数据集中的三元组划分入两个类别：将不含有选出的OOKB实体的三元组留在训练数据集中，将含有选出的OOKB实体的三元组放入auxiliary set中。

由于OOKB entity是新提出的问题，没有baseline， 本文自己构造了baseline： 通过对TransE在训练数据集上的训练结果，直接选取和OOKB相连接的实体的向量表示并进行三种不同方式的pooling。

![](/img/2017-09-26-result.png)

# Paper information
* Knowledge Transfer for Out-of-Knowledge-Base Entities: A Graph Neural Network Approach
* Takuo Hamaguchi, Hidekazu Oiwa, Masashi Shimbo, and Yuji Matsumoto
* IJCAI 2017
* [article link](https://www.ijcai.org/proceedings/2017/0250.pdf)
* [code link](https://github.com/takuo-h/GNN-for-OOKB)
