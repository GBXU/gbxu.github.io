---
title: CS229 Week4 Neural Networks
date: 2017-03-26 18:41:19
categories: ML/CS229
mathjax: false
tags: [Machine Learning,CS229]
---
<!--more-->

# 第四周 
## 8 Neural Networks: Representation
### 8.1 非线性假设
假使我们采用的都是 50x50 像素的小图片，并且我们将所有的像素视为特征，则会有2500 个特征，如果我们要进一步将两两特征组合构成一个多项式模型，则会有约 25002/2 个（接近 3 百万个）特征。普通的逻辑回归模型，不能有效地处理这么多的特征，这时候我们需要神经网络。
### 8.2 神经元和大脑
略
### 8.3 模型表示 1
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-6335a05ce38ef6bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 8.4 模型表示 2
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-3bfdec79df00166b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 8.5 特征和直观理解 1
在普通的逻辑回归中，我们被限制为使用数据中的原始特征x1,x2,...,xn，我们虽然可以使用一些二项式项来组合这些特征，但是我们仍然受到这些原始特征的限制。在神经网络中，原始特征只是输入层，在我们上面三层的神经网络例子中，第三层也就是输出层做出的预测利用的是第二层的特征，而非输入层中的原始特征，我们可以认为第二层中的特征是神经网络通过学习后自己得出的一系列用于预测输出变量的新特征。
### 8.6 样本和直观理解 II
略
### 8.7 多类分类
多类分类只需要输出层的label向量维度更大即可。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-8c3f55b3c3fc6421.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

