---
title: Machine Learning is Fun
date: 2016-09-10 15:39:49
categories: ML
mathjax: false
tags: [Machine Learning,CS229]
---
<!--more-->


## Machine Learning is Fun

正值课程需要，想起起初有试着阅读Martin T.Hagan等人编写的《神经网络设计》和周志华的《机器学习》。但是现在阅读了一篇Adam Geitgey教授的[《Machine Learning is Fun!》](https://medium.com/@ageitgey/machine-learning-is-fun-80ea3ec3c471#.c3znoc9lr)，“举个例子”让我颇为兴趣。感谢搬运工的[中文翻译](http://blog.jobbole.com/67616/)。

![代价函数](http://upload-images.jianshu.io/upload_images/2812342-2c43a99cfcfc2cae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这篇文章用了个估算房价的例子，引出多元线性回归的算法，同时提到了以下几个知识点：

**非线性数据**
>     很多其他类型的机器学习算法可以处理（如神经网络或有核向量机）

**过拟合**
>     碰上这样一组权重值，它们对于你原始数据集中的房价都能完美预测，
>     但对于原始数据集之外的任何新房屋都预测不准。
>     这种情况的解决之道也有不少（如正则化以及使用交叉验证数据集）

**同时**，`你只能对实际存在的关系建模`并且可以使用`机器学习算法库来解决实际问题`
