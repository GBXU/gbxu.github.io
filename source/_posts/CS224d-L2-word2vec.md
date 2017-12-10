---
title: CS224d L2 word2vec
date: 2017-03-26 19:18:29
categories: NLP/CS224d
mathjax: true
tags: [CS224d,NLP]
---
<!--more-->

## 1 WordNet
* 缺少细微差别（同义词间）
* 缺少新词
* 主观性大
* 人力成本
* 难离散化

## 2 “one-hot” representation
$$x_{2}=
\begin{bmatrix}
0\\
0\\
0\\
1\\
0\\
\end{bmatrix}$$

存在问题：

* 词汇增加，维度增大
* 缺少词汇间的Similarity特征

## 3 distributional similarity based representation
### 3.1 full documents vs window size of（5~10）

* word-documents cooccurrence matrix =》 被称为LSA(latent semantic analysis 浅层语义分析 OR 隐形语义分析)
> inverted fille index 倒排文件索引【魏骁勇教授[网络数据挖掘](http://#)】

* window => syntactic(POS) & semantic
![image.png](http://upload-images.jianshu.io/upload_images/2812342-7cf8e74211fffe84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.2 问题
* 文档增大，规模增大
* 维度很大(1 在很多文档出现 2 每个文档都很多词)
* 分类模型稀疏
* 不够健壮??

### 3.3 解决方法：降维
* SVD（奇异值分解）
* [Learning representations by back-propagating errors](http://www.iro.umontreal.ca/~vincentp/ift3395/lectures/backprop_old.pdf)(Rumelhart et al.,1986)
* [A Neural Probabilistic Language Model](http://jmlr.csail.mit.edu/papers/volume3/bengio03a/bengio03a.pdf)(Bengio et al., 2003)
* [Natural Language Processing (almost) from Scratch](http://www.jmlr.org/papers/volume12/collobert11a/collobert11a.pdf)(Collobert & Weston,2008)
* [word2vec](https://code.google.com/p/word2vec/)(Mikolov et al. 2013)

```
# SVD的实现
import matplotlib.pyplot as plt
import numpy as np
la = np.linalg
words = ["I","like","enjoy",
         "deep","learning","NLP","flying","."]
# I like deep learning.
# I like NLP.
# I enjoy flying.
# windows size is 1
#             I like enjoy deep learning NLP flying
X = np.array([[0,2,1,0,0,0,0,0], # I
             [2,0,0,1,0,1,0,0], # like
             [1,0,0,0,0,0,1,0], # enjoy
             [0,1,0,0,1,0,0,0], # deep
             [0,0,0,1,0,0,0,1], # learning
             [0,1,0,0,0,0,0,1], # NLP
             [0,0,1,0,0,0,0,1], # flying
             [0,0,0,0,1,1,1,0]]) # .
U,s,Vh = la.svd(X,full_matrices=False)
print(U)
plt.xlim(-0.8*1.1,0.2*1.1)
plt.ylim(-0.8*1.1,0.8*1.1)
for i in xrange(len(words)):
    # print(i)
    plt.text(U[i,0],U[i,1],words[i])
plt.show()
```
>![output](http://upload-images.jianshu.io/upload_images/2812342-46935d7f76d0c592.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.4 the he has 等词的过滤
* frequency
    * min(t,100)：频数t与某个临界值的比较
    * 直接ignore：频数t过大直接忽略
* 扩大window size：扩大size增加对临近词的计数
* 用[皮尔逊相关系数](https://segmentfault.com/q/1010000000094674)代替计数，并置负数为0：-1<相关系数r<1

*注：一些有趣的语义Pattern* 
*来源：*[An improved model of semantic similarity based on lexical co-occurence](http://tedlab.mit.edu/~dr/Papers/RohdeGonnermanPlaut-COALS.pdf)

## 3.5 word2vec
因为SVD存在的问题：
* 矩阵运算量
* 新词的更新
* 不同于其他DL模型的学习框架

提出利用word2vec(Mikolov et al.2013)：
目标函数
>$$J(\theta)=\frac{1}{T}\sum_{t=1}^{T}\sum_{-c<=j<=c,j\neq0}logP(w_{t+j}|w_t)$$
$$P(w_0|w_t)=\frac{e^{v_{w_0}^{'T}v_{w_i}}}{\sum_{w=1}^{W}e^{v_{w}^{'T}v_{w_i}}}$$

求导
>![image.png](http://upload-images.jianshu.io/upload_images/2812342-bdd45a629b1dccaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 3.6 word2vec的线性关系
apple - apples ≈ car - cars

## 3.7 GloVe
* word2vec 训练慢，统计不高效使用，但是能度量不止相似性，对其他任务也有好的表现
* Based count 训练快，高效使用统计，但是主要用于相似性度量，小规模不合适

GloVe
* 训练更快
* 大规模语料算法的扩展性也很好
* 小语料表现得也很好

**Word embedding matrix（词嵌入矩阵）,提前训练好的词嵌入矩阵**

## Appendix
* For more details, please click the note [第二讲：词向量](http://www.52nlp.cn/%e6%96%af%e5%9d%a6%e7%a6%8f%e5%a4%a7%e5%ad%a6%e6%b7%b1%e5%ba%a6%e5%ad%a6%e4%b9%a0%e4%b8%8e%e8%87%aa%e7%84%b6%e8%af%ad%e8%a8%80%e5%a4%84%e7%90%86%e7%ac%ac%e4%ba%8c%e8%ae%b2%e8%af%8d%e5%90%91%e9%87%8f)
* [Softmax回归：多类别的Logistic回归](http://ufldl.stanford.edu/wiki/index.php/Softmax%E5%9B%9E%E5%BD%92)
* [TREC](http://trec.nist.gov/)(short for "Text REtrieval Conference")
* [深度学习与自然语言处理(1)_斯坦福cs224d Lecture 1 笔记](http://blog.csdn.net/han_xiaoyang/article/details/51567822)
