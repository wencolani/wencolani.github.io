---
layout: post
category: Paper reading
title: Distributed Representations of Sentences and Documents
tagline: by wencolani
tags: [paper, paragraph embedding, word2vec, NLP]
---

本文提出了一个非监督学习方法Paragraph Vectore，从文本中学习句子，段落，和文本的向量表示。

# Motivation:
很多机器学习方法要求输入是固定长度的特征向量，词袋(bag-of-word)是最常用的方法，但词袋有两个最重要的缺点：

* 失去了词序信息。虽然bag-of-n-gram考虑了词序的信息，但是会面临新的问题：data-sparsity和 high-dimensionality。
* 忽略了词的语义信息(semantics of words)。比如“苹果”和“梨”。

# Adavantages of Paragraph Vectore
* Pagraph vector 是一个非监督的模型。可以应用于标注不足的数据。

* Paragraph Vectore 是固定长度的向量，它表示的文本长度是可变的，可以从sentence到paragraph到text。

* Paragraph vector acts a memory that remembers that is missing from the current context -- or the topic of the paragraph. 我的理解是在预测文本中的下一个词时，通常会取这个词前后的固定长度的词(context)，虽然多数情况下, context是非常有效的信息，但有时候这些词对于预测所需要的信息的覆盖是补全的，这就是paragraph vector存在的意义，它可以捕捉并记住文本的主题信息，并辅助预测。


# Method:

##  Paragraph Vector: a distributed memory model(PV-DM)

![](/img/2017-11-02-PargraphVector1.png)

* 任务目标：给定一个段落中固定长度的文本内容context，结合当前所处段落一个预测下一个词是什么。
* 方法： 和word2vec相似，每个词用一个向量表示，每个段落也用一个向量表示。连接(concatinate)paragraph vector和所有context的word vector作为输入，预测下一个词。

## Paragraph Vector without word ordering: Distributed bag of words(PV-DBOW)
![](/img/2017-11-02-PargraphVector2.png)

* 任务目标：用paragraph vector预测从当前文本中随机生成的词。
* 方法：每次随机梯度下降的迭代中，选取当前段落中一个固定窗口的文本，从这个窗口的文本中随机选取一个词，then form a classifi-
cation task given the Paragraph Vector.

## final paragraph vector
combiantion of PV-DM and PV-DBOW.
# Experiments

## Sentiment Analysis 
* 数据集： Standford sentiment treebank dataset with 8544 training sentences, 2210 testsentence and 1101 calidation sentences. 每个句子标有0.0到1.0的极性(由人生成的)。

* Evaluation: 
	* 5-way fine-grained classification: {Very Negative, Negative, Neutral, Positive,
Very Positive}
	* 2-way coarse-grained classification: {Negative, Positive}


* 方法：
	* training: 先学习训练样本中的sentence和subphrases的向量表示，然后用这些学习得的向量表示学习逻辑回归分类器。
	* testing: 固定所有的词向量，对每个test的句子学习词向量，然后讲句子向量表示输入之前学习得的分类器，对句子做情绪分类。
	* notice: 选词窗口大小为8，向量维度为400。

* 结果：
![](/img/2017-11-02-result2.png)

在多句子数据集IMDB dataset上情绪分析的结果如下：
![](/img/2017-11-02-result2.png)

## Information Retrieval
* 数据构造：首先对于每个查询，记录下搜索引擎返回的top10的结果，然后在这些结果数据集上构造数据集如下：对一个查询构造结果三元组，其中前两个是这个查询的答案，另一个不是。 构造的查询结果三元组80% for training, 10% for validation, 10% for testing.
* 测试方法：计算结果三元组中三个paragraph vector之间的距离。测试error是指，在测试三元组中，前两个来自相同查询的结果之间的距离比来第一和第三个来自不同查询结果之间的距离大。
* 结果如下：

![img-w1](/img/2017-11-02-result3.png)

# Thinking
paragraph vector是很直观的想法。
本文的实验设计很值得借鉴，对于非监督方法而言实验设计尤其中重要。

* Distributed Representations of Sentences and Documents
* Quoc Le, Tomas Mikolov 
* Proceedings of the 31st International Conference on Machine Learning, PMLR 32(2):1188-1196, 2014.
* [article link](https://arxiv.org/pdf/1405.4053.pdf)
* [code link]()
