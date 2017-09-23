---
title: "Infomation BottleNeck: 连Hinton都喜欢的文章"
date: 2017-09-16
categories:
  - Post
last_modified_at: 2017-09-16T22:00:25-05:00
---

深度学习模型的黑箱特性是非常令人头疼的问题。目前的各种解释都是基于函数拟合或者统计模型，虽然看起来像是那么回事，但是距离一个清楚明了又通透的解释还有距离。最近希伯来大学的Tishby教授关于深度学习解释的一篇文章引起了大家的广泛兴趣，因为其所论证的Information BottleNeck模型提出了一种基于Markov Chain和信息论模型的解释，连Hinton都忍不住为该文章打起了广告。下面我们来对文章进行一下分析：

## 信息论基础

为了理解Information BottleNeck，一些基本的信息论知识必不可少。下面我们来对信息论中的各种不同的熵的定义进行一个简要的介绍。
首先，什么是熵？一言不合就得上公式，对于某个随机变量x，其熵的计算方法为：

$$H(x) = -\sigma_{x\inX}p(x)logp(x)$$


