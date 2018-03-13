---
layout: post
category: Paper reading
title: Towards Holistic Concept Representations, Embedding Relational Knowledge, Visual Attributes, and Distributional Word Semantics
tagline: by wencolani
tags: [paper, kg embedding, word2vec,image embedding,embedding,image,text,NLP]
---

这篇文章提出了一个融合kg embedding，image embedding 和 word embedding的方法。能够融合更多的信息。

# Motivation
知识库，图像和文字这三种方式分别记录了事物不同方面的信息。
其中知识库主要包含了这个世界上跨领域的、通用的知识，侧重于记录实体之间的关系信息。
图片记录了视觉信息，例如颜色和形状等。
而文字记录了一些知识库中不包含的信息，知识库中记录的都是相对稳定的关系信息。

这篇文章的目的是提出一种tri-modal的embedding学习方法，能够有效融合和利用三种不同的信息。

# Method
方法的整体思路如下：
![](/img/2017-11-03-method.png)

## Word Embedding:
* method: word2vec.


## KG Embedding：
* data: DBpedia
* method: TransE

## Visual Object：
* data: ImageNet 1k (1000 categories, each with > 1300 images), 这些图片是和WordNet中的synset对应的。
* method：Inception-V3

## Shared concept space(fusion):
每一个concept都有来自KG， 文本，图片的向量表示。融合三个表示的方法主要有以下几种：
* AVG：在计算相似度的场景下，分别计算和三个不同向量表示的相似度然后平均值。
* CONC：先concatinate三个向量表示，然后在计算相似度。
* SVD
* PCA
* AUTO：autoencoder

# Experiments

## word siilarity
![](/img/2017-11-03-result1.png)

## entity segmentation
![](/img/2017-11-03-result2.png)

## entity-type prediction

![](/img/2017-11-03-result3.png)

# Thinking
1. 多源融合的embedding是值得研究的方向。
1. 如何将多源的信息在同一个向量空间中表示也是可以研究的点。


# Paper information
* Towards Holistic Concept Representations: Embedding Relational Knowledge, Visual Attributes, and Distributional Word Semantics
* Steffen Thoma, Achim Rettinger, and Fabian Both
* ISWC 2017
* [article link](https://iswc2017.semanticweb.org/wp-content/uploads/papers/MainProceedings/260.pdf)
* [code link]()
