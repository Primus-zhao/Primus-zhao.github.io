---
title: "Post: Show attend and tell"
date: 2017-09-09
categories:
  - Post
last_modified_at: 2017-09-09T13:34:25-05:00
---

最近在学习show and tell模型，这个模型主要有两个版本，这两我们首先分析Bengio团队提出的有attention model的版本。这个版本是2015年的论文中提出，在分析之后我们也会对image caption当前比较流行的模型进行介绍和分析。
## 主体思路
image captioning 的任务就是通过看图生成对应的描述。从整体角度来看，就是在辨识图中物体的同时再将这些物体之间的关系表述出来。当前的比较流行的思路是通过CNN来获得图中物体的辨识（隐式而非显式），获得了多物体的抽象信息后使用RNN模型来学习这些抽象信息之间的关系。对于单个图片来讲，CNN是肯定有能力将其中的物体信息表示出来的。根据CNN的特性，这些信息会保留在不同的channel当中，所以如何从这些channel当中得到所需的feature就成了关键，而这也是attention model能够登场的最重要原因。
## attention model
attention model从粗浅的意义上来将实际上就是从输入信息到选择权值的一个映射，由于可以通过神经网络来进行建模，所以可以与其他深度学习模型同步训练从而得到一个很好的结果。值得注意的是尽管原理简单，但是在具体应用的时候得到的权重和原模型的输入有多种不同的结合方式，其中带来的结果差异实际上值得细致分析。再进一步对attention model 进行细分的话可以分为hard和soft两种model,前者得出的权重具有one-hot的性质，即只会从原模型输入中取单一的channel，而后者则是所有输入的一个权重混合。这篇文章同时采取了两种方案并进行了对比，对比结果我们会在后边具体解释。
## 具体分析
话不多说，直接上公式
对于每个CNN channel输出的
<a href="https://www.codecogs.com/eqnedit.php?latex=ax^{2}&space;&plus;&space;by^{2}&space;&plus;&space;c&space;=&space;0" target="_blank"><img src="https://latex.codecogs.com/gif.latex?ax^{2}&space;&plus;&space;by^{2}&space;&plus;&space;c&space;=&space;0" title="ax^{2} + by^{2} + c = 0" /></a>
