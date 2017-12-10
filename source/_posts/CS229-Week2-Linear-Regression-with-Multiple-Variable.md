---
title: CS229 Week2 Linear Regression with Multiple Variable
date: 2017-03-26 18:31:01
categories: ML/CS229
mathjax: true
tags: [Machine Learning,CS229]
---
<!--more-->

# 第二周 
## 4 多变量线性回归(Linear Regression with Multiple Variables)
### 4.1 多维特征Multiple Features
例如房子的多维特征
$$x_{2}=
\begin{bmatrix}
1\\
3\\
\end{bmatrix}$$
支持多变量的假设$$h_\theta(x)=θ_0+θ_1*x_1+...θ_n*x_n$$

即：$h_\theta(x)=θ^{T}X$
### 4.2 多变量梯度下降
代价函数：
$$J(θ_0,θ_1..θ_n)= \frac{1}{2m}\sum_{1}^{m}(h(x_i) - y_i)^2$$
同样，根据批量梯度下降算法：
$$\theta_n=\theta_n-\alpha\frac{1}{m}\sum^{m}_{i=1}(h_\theta(x_i)-y_i)x_{n,i}$$
### 4.3 梯度下降法实践 1-特征缩放
使得所有特征的尺度都尽量缩放到-1到1之间
### 4.4 梯度下降法实践 2-学习率
学习率即步长：通常有
>α=0.01， 0.03， 0.1， 0.3， 1， 3， 10

### 4.5 特征和多项式回归
线性回归不一定符合数据本身。
因此有时需要曲线来适应数据。如：
$h_\theta(x)=θ_0+θ_1*x_1+θ_2*x_2^2$
*注：如果我们采用多项式回归模型，在运行梯度下降算法前， 特征缩放非常有必要*
### 4.6 正规方程
梯度下降算法来寻找cost的最小值很方便，但是对于某些线性回归问题，正规方程方法是更好的解决方案。 (**如抛物线的最小值**)
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-140309ba5f75e299.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 4.7 正规方程及不可逆性

## 5 安装matlab、Octave，Matlab入门

