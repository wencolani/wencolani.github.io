---
layout: post
category: Paper reading
title: Global RDF Vector Space Embeddings
tagline: by wencolani
tags: [paper, kg embedding, rdf2vec, glove]
---

这篇文章提出了一个结合GloVe想法的RDF2Vec新方法。

# Motivation
RDF2Vec 方法只用了local pattern。

新提出的方法可以有效利用global patterns。

# Method

## building a co-occurrence matrix for graph data

在GloVe首先构造了word co-occurrence matrix, 本文的新方法首先对RDF数据构造了co-occurrence matrix。

最直观的构造方法：从任意节点出发，进行固定步数的深度搜索，将所有限定步数以内可达的节点均当作出发节点的context，并且对越远的节点赋予越低的权重。但是这样的方法主要有以下问题：

* 同一个节点可能通过不同长度的路径被达到
* 路径中可能存在环状结构
* 各个不同长度的可达节点数分布不均

所以本文采取了Personalized PageRank 来衡量其他节点作为某一节点的context的重要性。



# Paper information
* Global RDF Vector Space Embeddings
* Michael Cochez, Petar Ristoski, Simone Paolo Ponzetto, Heiko Paulheim2
* ISWC 2017
* [article link](https://iswc2017.semanticweb.org/wp-content/uploads/papers/MainProceedings/164.pdf)
* [code link](https://github.com/miselico/globalRDFEmbeddingsISWC)
