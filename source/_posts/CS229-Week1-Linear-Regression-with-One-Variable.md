---
title: CS229 Week1 Linear Regression with-One Variable
date: 2017-03-26 16:34:36
categories: ML/CS229
mathjax: true
tags: [Machine Learning,CS229]
---
<!--more-->

# 吴恩达的Machine Learning
[Machine Learning](https://www.coursera.org/learn/machine-learning/)是Coursera上的一个课程。很受入门者的推崇。我在学习了BP算法(Back Propagation反向传播)之后开始观看。
# 第一周 
## 1 Introduction
### 1.1 welcome 
机器学习很有用也很有市场。
### 1.2 what is  ML
* 学习算法主要是监督学习和无监督学习
* 如何选择学习算法

### 1.3 监督学习Supervised Learning
两个例子：
* Regression回归问题（房价）：连续值
* Classification分类问题（癌症）：离散值

### 1.4 无监督学习Unsupervised Learning 聚类算法
作用：不知道数据集的信息，机器来找到不同的聚类和结构关系
例子：
* google　news(相关新闻会聚集在一起)
* 很多人的基因放在一起，来自动的辨认个体类型
* 应用于机器的协同工作
* 应用于社交网络分析
* 应用于公司客户的细分市场，针对性营销
* 应用于天文科学
* 应用于嘈杂声音中的区分（鸡尾酒会算法）

## 2 单变量线性回归(Linear Regression with One Variable)
### 2.1 - Model Representation 模型表示
我们的第一个学习算法是线性回归算法。
例子：房价预测
通过单变量size of house和price的关系，求拟合函数，用于预测价格。
>    				training set
    			learning algorithm
    size of house ->hypothesis-> estimated price
$h(x)=θ_0+θ_1*x$

### 2.2 代价函数
在建模的过程中，引入的parameters $ \theta_0$和$\theta_1$的初始值会导致建模误差(modeling error)，为了预测房价的准确，我们应该使得误差尽量小，于是引入cost function来衡量误差。
包括回归问题在内的大多数问题，cost function用误差平方代价函数都是很好用的：
>cost function：
$$(θ_0,θ_1)= \frac{1}{2m} \sum_{i=1}^{m} (h(x_i)-y_i)^2$$

### 2.3 代价函数的直观理解
> * hypothesis：$h(x)=θ_0+θ_1*x$
> * Parameters:$\theta_0$和$\theta_1$
> * Cost function:$$J(θ_0,θ_1)= \frac{1}{2m}\sum_{1}^{m}(h(x_i) - y_i)^2$$
> * Goal:$minJ(\theta_0,\theta_1)$
> ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-bf30661c645bc7d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 简化假设：$θ_0=0$即$h(x)=θ_1*x$，如此$J(θ_1)$变成一个二次函数，有最低点

### 2.4 代价函数的直观理解II
> ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-b56c51d68b5ad580.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 等线图（contour plot）：若不做简化，则价值函数就从平面二次函数变为三维中的曲面（MATLAB实现），就可以在自变量平面上画等值线

### 2.5 梯度下降 Gradient Descent
有了直观感受，我们的目标降低$J(\theta)$就是找到最低点。
*注：正规方程(normal equations)也能求J函数的最小值，但是计算量大，因此用梯度下降法*

>梯度下降：设置$(\theta_0,\theta_1,...\theta_n)$的初始值，按照梯度下降(下降最多的方向)设置新的$\theta$

批量梯度下降法的公式：
> ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-707e199e40dc0d38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 批量梯度下降法Batch gradient descent:每一步都用到了所有数据
> $\alpha$：learning rate
> $\theta$：同步更新

### 2.6 梯度下降的直观理解
梯度下降中，计算的$\frac{\sigma J(\theta)}{\sigma \theta_j}$可以说是斜率，当该值很小的时候，可能就是到达最小值了，但也可能是局部最小值。
>收敛到一个局部最小解上。或者凸函数（convex function）是全局最小值

### 2.7 梯度下降的线性回归 Gradient Descent For Linear Regression
回到之前的房价问题：
> ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-e60f608c00cc0e93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们将梯度下降法运用到线性回归模型上，得到梯度下降的公式：
> ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-09d84795241f9b91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 此处为”批量梯度下降”，在梯度下降每一步中，在计算微分求导项时，我们需要进行求和运算


## 3 Linear Algebra Review 线性代数
略略略

