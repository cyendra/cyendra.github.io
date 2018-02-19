---
layout: post
title: Deep Learning Review 阅读笔记
categories: DeepLearning
description: 2015年，Yann LeCun，Yoshua Bengio 和 Geoffrey Hinton 深度学习领域的这三大神级人物，为了纪念人工智能60周年，合作在《自然》上发表深度学习的综述性文章，介绍了深度学习的基本原理和核心优势，详细介绍了CNN、分布式特征表示、RNN及其不同的应用，并对深度学习技术的未来发展进行展望。
keywords: DeepLearning
---

## 背景
2015年，Yann LeCun，Yoshua Bengio 和 Geoffrey Hinton 深度学习领域的这三大神级人物，为了纪念人工智能60周年，合作在《自然》上发表深度学习的综述性文章，介绍了深度学习的基本原理和核心优势，详细介绍了CNN、分布式特征表示、RNN及其不同的应用，并对深度学习技术的未来发展进行展望。

## 笔记

### 前言
首先介绍了机器学习在现代各个领域中的作用，之后引出深度学习对传统机器学习的优势：特征学习，把原始数据经过简单模型转换成更高层次的抽象表达，并通过图片识别举例说明。强调深度学习的核心：特征不是人工设计，而是从数据中学到的。最后介绍了深度学习的重大进展，并展望未来更大的成功。

### 监督学习
介绍机器学习的常见形式：监督学习。包含对象类别的数据集样本，衡量输出质量分数的目标函数，模型内部的可调参数权重，通过反梯度调整权重。随机梯度下降SGD算法在小sample集合上计算梯度，与其他算法相比有速度快的优势。训练结束后要通过测试集检验泛化能力。  
介绍传统的线性分类器，通过超平面把空间分割为两部分。接着提到线性分类器、其他浅层分类器的缺点，对复杂样本特征无力，需要工程技术和领域知识手工设计特征。而深度学习的优势是可以学习得到特征。深度学习的结构是多层简单模型组成，每层对输入进行变化，增加可选择性和不变性。

### 反向传播
随机梯度下降训练神经网络，反向传播求解目标函数关于多层神经网络权重梯度。利用链式求导法则，核心：目标函数对某层输入的导数通过下一层导数求得。从最后一层向前计算每层的梯度。  
又介绍了前馈神经网络的结构，每层加权求和，传给非线性激活函数ReLU等。ReLU比tanh、sigmoid学习速度更快。输入输出层以外的叫做隐含层，看作对输入的非线性变换，使最后一层线性可分。之后提到了神经网络和反向传播的发展历史，随着理论和实践学术界认识到，由于鞍点，局部最优解并无危害。  
无监督学习，通过预训练初始化网络权重，在语音识别等领域取得成果。对小数据集可以防止过拟合，提高少标签样本的泛化能力。提到一种叫做卷积神经网络CNN的深度前馈网络，更易训练并比全连接泛化性能更好。  

### 卷积神经网络
CNN处理多维数组数据，1D信号、序列、2D图像声音、3D视频等。四个关键：局部链接、权值共享、池化、多网络层。典型结构：最初卷积层感知上一层特征的局部链接、池化层把相似特征合并。之后接更多卷积核全连接层。同样用反向传播训练权值。    
卷积层结构：feature map组成，filter bank链接上一层feature map的局部加权加和非线性函数ReLU，同一个feature map共享filter bank，不同feature map使用不同filter bank。 
池化层计算局部块的最大值，减少表达维度和对数据有平移不变性。池化层使很多自然信号的高级特征抽象对前一层特征变化有健壮性。 之后介绍了卷积和池化层的灵感来源视觉神经细胞。又介绍了基于卷积网络的应用。  

### 卷积网络图像理解
cnn大量用于图像各个领域如人脸识别。由ImageNet竞赛开始重视，利用了GPU、ReLU、dropout、样本变换等技术，带来了计算机视觉革命。随着软硬件进步，训练时间也压缩到几小时。又介绍了各大公司对cnn性能和应用上的进展。 

### 词向量和语言处理
首先介绍了词向量的优点：泛化适应新特征值的组合、指数级优势。从one-hot形式的向量通过学习得到语义特征向量。词向量广泛用户自然语言处理中。理论依据：逻辑启发式符号代表事物，神经网络式利用活动载体、权值矩阵、标量非线性化，向量空间中语义相关的词靠近，实现语义表达功能。传统方法为n-grams方法，基于统计词出现频率。

### 递归神经网络
rnn对序列输入有更好效果，一次处理一个输入序列，网络维护过去时刻的状态向量。通过训练编码器网络，得到thought vector可以保存句子语义信息，通过喂给解码器可以进行机器翻译。与cnn结合可以将图片翻译为句子，也掀起了热潮。  
rnn展开后可以视为共享权重的深度前馈神经网络，难以学习长期信息。LSTM有特殊隐式单元记忆细胞，记录长期信息。在机器翻译等领域表现良好。  
近几年进展，问答系统、复杂推理。  

### 未来展望
1. 无监督学习长期越来越重要。
2. 人类视觉方面有更多进步。
3. RNN更高的理解自然语言。
4. 结合复杂推理学习系统。

## 重要知识
1. 深度学习对原始数据进行简单非线性变换，得到高层次更抽象的特征表达。从而有了复杂函数的学习能力。各层特征都不是人工设计，而是从数据中学到的。不需要人工参与特征工程也是深度学习的优势。
2. 监督学习基础：有标签的数据集样本、目标函数、模型参数权重。通过SGD等方法计算梯度，更新参数。
3. 反向传播：链式求导，目标函数对于某层输入的导数可以通过对该层输出的导数求得。问题：局部最优解、鞍点。
4. CNN结构：卷积层（局部连接、权值共享）、池化层、全连接层。处理图像等多位数组。
5. 词向量：向量空间中语义相关的词彼此靠近。
6. RNN：处理序列输入，保存历史信息。处理机器翻译领域。

## 感悟
这是一篇综述类型的文章，主要以行业进展和简单原理介绍为主，通过这篇文章，读者可以对深度学习领域的现状有一个大致的认知。如果对文章内提到的各种应用有兴趣，还需要去找对应的论文来看。理论方面，也只给出了关键词，对具体原理还要继续深入学习。