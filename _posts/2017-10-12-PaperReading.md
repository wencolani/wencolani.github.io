---
layout: post
category: Paper reading
title: Hybrid computing using a neural network with dynamic external memory 
tagline: by wencolani
tags: [paper, nature, memory, nature]
---

* Hybrid computing using a neural network with dynamic external memory 
* Google Deepmind
* Nature October 2016
* [article link](http://www.nature.com/nature/journal/v538/n7626/abs/nature20101.html?foxtrotcallback=true)
* [code link](https://github.com/deepmind/dnc)

# Overview 
这篇文章提出了一个神经网络框架DNC(Differentiable neural computer)搭载了一个可读可写的外部memory模块。是NTM(Neural Turing Machine)的升级版。

下面这张论文中的图很好地说明了整个DNC的框架：

![](/img/2017-10-12-structure.png)

整个框架流程可以分为四个部分：

##  Controller:
 controller是一个recurrent的结构，输入输出均为向量，并会产生一系列interface parameters输入到读写部分。整个模型的输入是由controllor接受的。
 整个controllor采用的是deep LSTM结构，含有input gate, forget gate, output gate. controllor输出的output vector和interface vector都由hidden vector生成。

##  Read and write head:
_read head_ 会接受由controllor产生的interface parameters包括：
* read key：和memory的每一行的content计算相似度然用于读操作的寻址，用于计算read weight。
* read strengths: 每一个read head都会分配一个read strength，可看作是当前时刻t每个read head的读操作权重。
* read mode: 整个模型定义了三种读的方式,['C']: content lookup using a read key；['F']:reading out location forward in the order they were writteh; ['B']: reading out location backword in the order they were written.

每一个read head都会根据controllor的输入和当前的memory生成一个read vector，并反馈给controllor。

_write head_ 会接受由controllor产生的interface papameters包括：
* write key: 和read key 类似。
* write strength: 和read strength相似，但通常write head只有一个，而read head可能有多个。
* erase vector: 用于控制前一时刻t-1的memory的忘记程度。
* write vector：和write weight一起控制当前的写入内容。
经过write head 操作后，会更新t-1时刻的momoey并生成时刻t修改后的memory。

## Momory:
Memory 表示成一个N\*W的矩阵M。

## Memory usage and temproal links:
* _usage vector_(各元素取值为0到1之间)用来记录和衡量整个memory各个位置(通常以矩阵的行 为单位)的使用状况，如果当前行的memory usage为1，则不能对此位置进行任何寻址并进行读写操作，必须等待free gate将此位置重新释放。
* _temporal link matrix_ 记录的memory中每个位置的写的顺序(图中用黑色的有向箭头表示)。维护这个temporal link matrix的原因是在一些场景下比如序列化的指令(sequence of instruction),比如bAbI数据集上对于对于逻辑推理能力的测试，输入数据的顺序是很重要的。

# Experiment
论文主要包含了三个实验。
* Synthetic question answering
* Graph experiment
*  Block puzzle experiment

这里就不详细介绍了

# Thinking
个人认为这篇文章最精彩的地方在于整体框架的设计，尤其是对memory的利用，比如如何进行读写操作，读写操作各自都有怎样不同的策略，整个memory的使用有什么上层的策略比如usage等。仔细看论文后面Method部分会体会更深刻。




