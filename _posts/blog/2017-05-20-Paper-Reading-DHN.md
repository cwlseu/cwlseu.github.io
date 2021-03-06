---
layout: post
title: "论文笔记：DHN, 基于神经网络的hash算法"
categories: [blog]
tags: [深度学习]
description: 基于神经网络的hash算法Deep hashing network for Efficient similarity Retrival
---
* content
{:toc}                                            

## 引言

进行图像搜索引擎加速，不得不提到hash算法。之前的有很多基于监督的hash算法是基于在现有的特征的基础上进行hash函数的学习。现在图像引入神经网络，这个两个过程就被结合到一起了。CNNH可以说是首先将hash函数学习融入到神经网路中的先驱。这不后来就出现了Deep hashing Network

## 文章来源

[deep-hashing-network-aaai16]<http://ise.thss.tsinghua.edu.cn/~mlong/doc/deep-hashing-network-aaai16.pdf>

> 本课题的主要目标
学习一个非线线性的hash函数$f:x |-> h \in \{-1, 1\}^K$，使得可以使用k-bit的hash 码表示x, x是R^n空间的一点。同时应该满足保持点对之间的相似性。

![@贝叶斯目标函数](https://cwlseu.github.io/images/deephash/DHN-1.jpg)

## 主要贡献

1. a fully-connected hashing layer to generate compact binary hash
codes;
2. a pairwise crossentropy loss layer for similarity-preserving learning
3. a pairwise quantization loss for controlling hashing quality

## 架构图

![@贝叶斯目标函数](https://cwlseu.github.io/images/deephash/DHN.jpg)


## 目标函数计算

![@条件概率计算](https://cwlseu.github.io/images/deephash/DHN-2.jpg)
我们可以看出，较小的Hamming距离对应较大的内积<hi, hj>，那么p(1|hi,hj) 的值将比较大，也就
是说明hi 和hj是相似的；反之，p(0|hi,hj)的值将比较大，hi 和hj是不相似的.
![@条件概率计算](https://cwlseu.github.io/images/deephash/DHN-3.jpg)
通过上面两个式子，我们可以将贝叶斯目标函数转化为如下目标函数
![@目标函数转换](https://cwlseu.github.io/images/deephash/DHN-4.jpg)

但是又有一个问题:Q函数是离散的，不好求导，不容易进行BP学习。那么我们就采用
![@目标函数中Q的转换](https://cwlseu.github.io/images/deephash/DHN-7.jpg)进行代替。

最后，总结一下现在的目标函数：
![@文章中采用的模板函数](https://cwlseu.github.io/images/deephash/DHN-8.jpg)

文章中还详细说明了如何推导梯度函数，如果有兴趣不妨拿文章来钻研一下。


## DataSet

**NUS-WIDE1** is a public web image dataset. We follow the settings in (Liu et al. 2011; Lai et al. 2015) and use the subset of 195,834 images that are associated with the 21
most frequent concepts, where each concept consists of at least 5,000 images.
**CIFAR-10** is a dataset containing 60,000 color images in 10 classes, and each class has 6,000 images in size 32×32.
**Flickr3** consists of 25,000 images collected from Flickr, where each image is labeled with one of the 38 semantic concepts.

## 实验参数

We implement the DHN model based on the open-source
Caffe framework (Jia et al. 2014). We employ the AlexNet
architecture, finetune convolutional layers conv1–conv5 and fully-connected layers fc6–fc7 that were copied from the pre-trained model, and train hashing layer fch, all via back-propagation. As the
fch layer is trained from scratch, we set its learning rate to
be 10 times that of the lower layers. We use the mini-batch
stochastic gradient descent (SGD) with 0.9 momentum and
the learning rate annealing strategy implemented in Caffe,
and cross-validate the learning rate from $10^{-5}$ to $10^{-2}$ with
a multiplicative step-size 10. We choose the quantization
penalty parameter λ by cross-validation from $10^{−5}$ to 100
with a multiplicative step-size 10. We fix the mini-batch size
of images as 64 and the weight decay parameter as 0.0005.

## 结果

### 与其他Hash方法比较

![@实验结果](https://cwlseu.github.io/images/deephash/DHN-Table1.jpg)
其中LSH，SH， ITQ是非监督方法，其他其中属于监督方法。使用deeplearning 的方法时端到端的方式。其他的方法采用SIFT或者GIST特征值。实验结果表明，DHN的效果比当前的所有的Hash方法都是好的，以KSH作为baseline, DHN分别在NUS-WIDE,
CIFAR-10, and Flickr实现了MAP增加16.3%, 25.8%和12.7%.

### 不同bit数目的实验结果

![@NUS-WIDE dataset实验结果](https://cwlseu.github.io/images/deephash/DHN-Fig3.jpg)
![@CIFAR-10实验结果](https://cwlseu.github.io/images/deephash/DHN-fig4.jpg)
从两幅曲线中来看，从c结果中看出，DHN的top-n结果是最好的。从a图中可以看出，在bits比较少的时候，DHN是比较好的，而且比当前最好的结果DNNH的效果都好，但是当bit数比较多的时候，我们的结果反而比较差。我们比较可以看出，24bit的时候比48bit的时候DHN结果比48bit的时候都好，从而可以得出DHN的压缩效果比较好。


### 损失函数的有效性

![@目标函数的不同组合的结果实验结果](https://cwlseu.github.io/images/deephash/DHN-Table2.jpg)
We investigate several variants of DHN: DHN-B is the DHN
variant without binarization (h ← sgn(zl) not performed), which may serve as an upper bound of performance. DHNQ is the DHN variant without the quantization loss (λ = 0). DHN-E is the DHN variant using widely-adopted pairwise quared loss![@之前的损失函数](https://cwlseu.github.io/images/deephash/DHN-12.jpg) instead of the pairwise cross-entropy loss.

## 我的想法

实验结果中可以看出，
1. 损失函数中加入**量化损失**函数的使用是有效的，
2. 量化损失函数的问题转化中可以学到一个绝对值的近似函数logcosh(x)函数。

## 后续论文

[1]. [Liong_Deep_Hashing_for_2015_CVPR_paper]<http://www.cv-foundation.org/openaccess/content_cvpr_2015/papers/Liong_Deep_Hashing_for_2015_CVPR_paper.pdf>

[2]. [Simultaneous Feature Learning and Hash Coding with Deep Neural Networks]<https://arxiv.org/pdf/1504.03410.pdf>