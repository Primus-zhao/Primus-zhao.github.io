---
title: "Post: Show attend and tell"
date: 2017-09-09
categories:
  - Post
last_modified_at: 2017-09-09T13:34:25-05:00
---

最近在学习show and tell模型，这个模型主要有两个版本，这两我们首先分析Bengio团队提出的有attention model的版本。这个版本是2015年的论文中提出，在分析之后我们也会对image caption当前比较流行的模型进行介绍和分析。

### why attention? 
image captioning 的任务就是通过看图生成对应的描述。从整体角度来看，就是在辨识图中物体的同时再将这些物体之间的关系表述出来。当前的比较流行的思路是通过CNN来获得图中物体的辨识（隐式而非显式），获得了多物体的抽象信息后使用RNN模型来学习这些抽象信息之间的关系。对于单个图片来讲，CNN是肯定有能力将其中的物体信息表示出来的。根据CNN的特性，这些信息会保留在不同的channel当中，所以如何从这些channel当中得到所需的feature就成了关键，而这也是attention model能够登场的最重要原因。

### what's attention?
attention model从粗浅的意义上来将实际上就是从输入信息到选择权值的一个映射，由于可以通过神经网络来进行建模，所以可以与其他深度学习模型同步训练从而得到一个很好的结果。值得注意的是尽管原理简单，但是在具体应用的时候得到的权重和原模型的输入有多种不同的结合方式，其中带来的结果差异实际上值得细致分析。再进一步对attention model 进行细分的话可以分为hard和soft两种model,前者得出的权重具有one-hot的性质，即只会从原模型输入中取单一的channel，而后者则是所有输入的一个权重混合。这篇文章同时采取了两种方案并进行了对比，对比结果我们会在后边具体解释。

### how to attend?
话不多说，直接上公式
对于每个CNN channel输出的 \\(\{a_1, a_2,...,a_L\}\\), 我们可以采用如下的公式来计算每个channel对应的attention值：

$$e_ti=f_{att}(a_i, h_{t-1})$$

然后采用softmax方法：

$$\alpha_{ti} = \frac{exp(e_{ti})}{\sum_{k=1}^L{exp(e_{tk})}}$$

这样我们就得到了一组attention值\\(\alpha_{ti}\\), 之后我们通过如下的公式来得到新的输入向量：

$$\hat{z_t}=\phi(\{a_i\}, \{\alpha_i\})$$

在通过计算attention value的过程中我们可以看到，attention值完全是由本channel输入和前一状态的hidden state \\(h_{t-1}\\)来决定的，即由当前输入和历史信息共同来求解。
当然，在进行第一轮计算时是没有隐藏层信息的，论文里给出的方法是采用两个单独的前馈网络来处理，具体公式如下：

$$c_0=f_{init, c}(\frac{1}{L}\sum_{i}^L{a_i})$$

$$h_0=f_{init, h}(\frac{1}{L}\sum_{i}^L{a_i})$$

这种解决方案实际上是等价于在初始状态时之通过输入值而不考虑隐藏层状态来进行计算，其实与将隐藏层直接设为0意思上差不多，但是由于因为了新的网络，所以拟合能力应该强些， 在初始状态训练的过程中不会影响后续状态训练出的LSTM权值。
当然，这还不算完， 为了计算输出概率（其实就是计算应该是哪个词），最后又引入了一组全连接和幂函数，形式如下：

$$p(y_t|a, y_1^{t-1})\propto exp(L_o(Ey_{t-1}+L_hh_t+L_z\hat{z_t})))$$

其中的\\(L_o, L_h, L_z\\)通过训练得到。
文章之后进行了Hard和Soft attention的分析，为了保证思路的连贯性，我们先略过它们，最后再统一进行分析。
为了完成训练，我们还必须确定loss function, 本论文中提出的形式是：

$$L_d=-log(P(y|x))+\lambda\sum_i^L(1-\sum_t^C\alpha_{ti})^2$$

我们可以看到，除了我们常见的负log概率值作为loss以外，还添加了另外的一个惩罚项，即\\(\lambda\sum_i^L(1-\sum_t^C\alpha_{ti})^2\\)，此项的作用有两点，第一点是尽量让所有时间序列的注意力权值加和为1, 即括号中间的\\((1-\sum_t^C\alpha_{ti})^2\\), 而括号外边的项作用则是计算所有channel的损失，并通过\\(\lambda\\)来对惩罚项进行平衡。
在这里我们针对这个loss function多说两句，事实上后面针对注意力函数的惩罚项是个经验性的公式，并未给出统计或者概率上的解释，论文中直接指出加入了该项以后，得到的BLEU值会得到提升。从逻辑角度来讲，我们希望注意力机制保证在LSTM前进的过程中均匀地关注各个部分，这也和这个想的引入不谋而合。此外还有一个小trick，事实上在计算z值的过程中，我们采用的计算公式为

$$\phi(\{a_i\}, \{\alpha_i\})=\beta\sum_t^L\alpha_ia_i$$

这个beta并不是一个超参，而是通过
$$\beta_t=\sigma(f_\beta(h_{t-1}))
通过这个\\(\beta\\),得到的模型将更多的关注图像中的物体。对这一个点如何来理解呢？对这一点的解释非常有趣，如果粗浅的理解的话，增加的模型的自由度保证了这个模型具有更强的学习能力，所以能够在生成过程中更好的进行tracking。如果针对公式细致的进行理解的话，我们可以看到为了迎合生物学的观点，也为了增加泛化能力，我们隐性的引入了一个假设，对于单次的注意力提取，我们要求所有权值的和为1（非直接要求，而是通过softmax函数来实现）。这在保证了泛化性能，同时与认知生理学统一的同时也带来了限制。
通过这个\\(\beta\\),得到的模型将更多的关注图像中的物体。对这一个点如何来理解呢？对这一点的解释非常有趣，如果粗浅的理解的话，增加的模型的自由度保证了这个模型具有更强的学习能力，所以能够在生成过程中更好的进行tracking。如果针对公式细致的进行理解的话，我们可以看到为了迎合生物学的观点，也为了增加泛化能力，我们隐性的引入了一个假设，对于单次的注意力提取，我们要求所有权值的和为1（非直接要求，而是通过softmax函数来实现）。这在保证了泛化性能，同时与认知生理学统一的同时也带来了限制。即虽然我们关注了object（假设各个channel提取出的是各个物体），但是我们在后续处理的时候给予这个物体的权值有限，但是我们在后续处理的时候给予这个物体的权值有限，没有办法通过增大权值的方式将他们从背景中更好的抽离出来，（注意力权值只实现了不同物体之间相对权值的调节，即物体的选择，而对将物体从背景中抽离无能为力)，而\\(\beta\\)参数的引入解决了这一问题。值得注意的一点是，这个\\(\beta\\)只由上一个LSTM的隐藏层的状态决定，而不受当前输入的影响，这主要也是为了保证不至于出现交叉影响的现象
## 注意力机制
这篇文章作为注意力机制的几篇奠基性文章之一(此外还有bengio大神的neural translation和Minh的RAM), 其对注意力机制的作用及描述还是分析的非常全面到位的。涵盖了Hard和soft两种不同的注意力机制。其中的Hard attention机制结合了随机特性。接下来我们按照顺序依次分析Hard和soft两种不同机制。
1 Stochastic Hard Attention
对于hard attention来说，其主要机制就是每次选取单个channel的图片，所以其产生的attention value具有one-hot的性质。具体的操作方式是让每个channel的值以\\(\{\alpha_i\}\\)的概率取1,这也就形成了一个multinoulli分布的模型。具体的数学表达的方式如下：

$$p(s_{t,i}=1|s_{j<t}, a)=\alpha_{t,i}$$ 
$$\hat{z_t}=\sum_is_{t,i}a_i$$
