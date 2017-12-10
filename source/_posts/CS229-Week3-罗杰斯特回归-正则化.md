---
title: CS229 Week3 罗杰斯特回归&正则化
date: 2017-03-26 18:40:21
categories: ML/CS229
mathjax: true
tags: [Machine Learning,CS229]
---
<!--more-->
# 第三周 
## 6 Logistic Regression 逻辑回归/两分类问题
### 6.1 分类问题Classification and Representation
分类问题：0或1
坏样本很大影响预测linear function
### 6.2 假说表示 Hypothesis Representation
$h_\theta(x)=g(\theta^Tx)$
逻辑函数、激励函数$g(z)=\frac{1}{1+e^{-z}}$
### 6.3 判定边界
* Decision Boundary：output0,1的input间的边界
	* linear decision boundary
	* non-linear decision boundary
	* eg.$h_\theta(x)=g(\theta^T(x))>=0.5$when$\theta^T(x)>=0$，$\theta^T(x)$就是边界

### 6.4 代价函数
### Logistic Regression Model
* cost function:convex
$Cost(h_\theta(x_i) ，y_i)$

### 6.5 简化的成本函数和梯度下降
* simplified cost function and gradient descent
$$Cost(h_\theta(x_i) ，y_i)=-ylog(h_\theta(x))-(1-y)log(1-h_\theta(x))$$
	* We can fully write out our entire cost function as follows:
>$$h=g(\theta^T X)$$
>$$J(\theta)=\frac{-1}{m}\sum_{i=1}^{m}[y_i log(h_\theta(x_i))+(1-y_i)log(1-h_\theta(x_i))]$$

	* Gradient Descent
	Remember that the general form of gradient descent is:$$\theta_j:=\theta_j-\alpha \frac{\sigma J(\theta)}{\sigma \theta_j}$$
	Repeat:$$\theta_j:=\theta_j-\frac{\alpha}{m}\sum_{i=1}^{m}(h_\theta(x_i)-y_i)x_{ij}$$
	* Partial derivative of J(θ)
	First calculate derivative of sigmoid function:$σ(x)′=(\frac{1}{1+e^{−x}})′=σ(x)(1−σ(x))$
	Now we are ready to find out resulting partial derivative:$$\frac{\sigma J(\theta)}{\sigma \theta_j}=\frac{1}{m}\sum_{i=1}^{m}(h_\theta(x_i)-y_i)x_{ij}$$

### 6.6 高级优化 advanced optimization
高级优化算法：
> * 共轭梯度法
* BFGS (变尺度法)
*  L-BFGS (限制变尺度法

这三种算法收敛得远远快于梯度下降。
同时这三种算法内部有线性搜索(line search)算法，来尝试不同的学习率$\alpha$。
### 6.7 多类别分类：一对多 Multiclass Classification
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-7c5ee37f07e1f299.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 7 正则化(Regularization)
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-e0af36f4f2b8fd69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>如上所示，x次数越高拟合越好，但是预测能力会减弱。
### 7.1 过拟合的问题
解决过拟合问题：
> * 丢弃一些不能帮助我们正确预测的特征。可以是手工选择保留哪些特征， 或者使用一些模型选择的算法来帮忙（例如 PCA）
* 正则化。 保留所有的特征，但是减少参数的大小（ magnitude）
    1. 正则化的目的：防止过拟合！
    2. 正则化的本质：约束（限制）要优化的参数。
    
### 7.2 代价函数
7.1中$$h_\theta(x)=θ_0+θ_1*x_1+θ_2*x_2^2...$$式子可以看出，是那些高次项导致了过拟合的产生，如果我们能让这些高次项的系数接近于 0 的话，我们就能很好的拟合了。
因此我们选择加大最小化代价函数过程中对高次项的惩罚。如下式对所有θ都增大了惩罚：
>$$J(θ)= \frac{1}{2m}[\sum_{1}^{m}(h(x_i) - y_i)^2+\lambda\sum_{j=1}^n\theta_i^2]$$
此处$$\lambda$$过大，则会把所有的参数都最小化了，导致模型变成 hθ(x)=θ0,
过小，起不到作用。

### 7.3 正则化的线性回归
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-cb1b4a39a7e42631.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 7.4 正则化的逻辑回归
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-f7fa0b7add30df0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

