---
title: CS224d Pset 1
date: 2017-04-04 20:02:21
mathjax: true
categories: NLP/CS224d
tags: [CS224d,NLP]
---
<!--more-->

![image.png](http://upload-images.jianshu.io/upload_images/2812342-d2e4d20879071fd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
根据题意，

> * $h=sigmoid(xw_1+b_1)$
> * $\widehat y=softmax(hw_2+b_2)$
> * m维度x输入以如下的矩阵形式，每行为一个样例
> 
$$\begin{bmatrix} 
x_{D_1}\ x_{D_2}\ ...\ x_{D_m}\\ 
x_{D_1}\ x_{D_2}\ ...\ x_{D_m}\\ 
x_{D_1}\ x_{D_2}\ ...\ x_{D_m}\\ 
.\\
.\\
x_{D_1}\ x_{D_2}\ ...\ x_{D_m}\\ 
\end{bmatrix}
$$
> * 所以根据上述网络图
> 加入有样例数N，隐藏层$D_h=h$，输出层y有$D_y=n$维
> 输入X矩阵 N*m
> W1矩阵m*h
> W2矩阵h*n
> 最终得到N*n的矩阵输出

## Softmax(10 points)
### (a)求证softmax输入常数c不影响结果。
$$(softmax ( \mathbf{x} + c  ))_i = \frac{\exp (\mathbf{x}_i + c ) }{\sum_{j=1}^{dim( \mathbf{x})}{\exp(\mathbf{x}_j+c )}} = \frac{\exp(c )\exp ( x_i)}{\exp ( c )\sum_{j=1}^{dim(x )}\exp( x_j)} = \frac{\exp ( x_i)}{\sum_{j=1}^{dim(x )}\exp( x_j)} = ( softmax ( \mathbf{x}) )_i$$
### (b)实现softmax。见Github

## Neural Network Basics (30 points)
### (a)推导sigmoid函数的导数。
$$σ′(x)=σ(x)(1−σ(x))$$
### (b)交叉熵的导数
首先，相比方差代价函数，交叉熵代价函数用于sigmoid会使得梯度更新更明显。
**softmax:**
$$softmax(x_i)=\frac{e^{x_i}}{\sum_k^m e^{x_k}}$$
**softmax求导:**
当$j = i$：
$$\frac{\partial S_j}{\partial x_i}
=\frac{\partial }{\partial x_i}(\frac{e^{x_j}}{\sum_k^m e^{x_k}})\\
=\frac{\partial }{\partial x_i}(\frac{e^{x_j}}{e^{x_1}+e^{x_2}...+e^{x_k}})\\
=\frac{e^{x_i}*({e^{x_1}+e^{x_2}...+e^{x_k}})-e^{x_i}*e^{x_i}}{(e^{x_1}+e^{x_2}...+e^{x_k})^2}\\
=S_i-S_i^2\\
=S_i(1-S_i)
$$
当$j \neq i$：
$$\frac{\partial S_j}{\partial x_i}=\frac{\partial }{\partial x_i}(\frac{e^{x_i}}{\sum_k^m e^{x_k}})\\
=\frac{\partial }{\partial x_i}(\frac{e^{x_j}}{e^{x_1}+e^{x_2}...+e^{x_k}})\\
=\frac{0*({e^{x_1}+e^{x_2}...+e^{x_k}})-e^{x_j}*e^{x_i}}{(e^{x_1}+e^{x_2}...+e^{x_k})^2}\\
=-S_jS_i\\
$$

所以
$$
\frac{\partial S_j}{\partial x_i} = 
\begin{cases} 
S_i(1 – S_i),&\quad i = j \\
-S_i S_j,&\quad i \neq j
\end{cases}$$

[**CE(Cross-entropy)**](https://en.wikipedia.org/wiki/Cross_entropy):
$$CE(y,\widehat y)=-\sum_k y_k log({\widehat y_k})$$
$$其中 \widehat y_k = sigmoid(x_k)$$
求导：
$$\begin{split} 
\frac{\partial CE(y,\widehat y)}{\partial x_i} 
&=-\sum_k y_k\frac{\partial \log \widehat  y_k}{\partial x_i} \\ 
&=-\sum_ky_k\frac{1}{\widehat y_k}\frac{\partial \widehat y_k}{\partial x_i} \\ 
\end{split} $$
$$其中
\sum_ky_k\frac{1}{\widehat y_k}\frac{\partial \widehat y_k}{\partial x_i} = 
\begin{cases} 
y_i(1-\widehat y_i),&\quad k = i \\
\sum_{k\neq i}y_k\frac{1}{\widehat y_k}({-\widehat y_k \widehat y_i}) ,&\quad k \neq i
\end{cases}$$


$$\begin{split} 
所以带入&=-y_i(1-\widehat y_i)+\sum_{k\neq i}y_k({\widehat y_i}) \\ 
&=-y_i+{y_i \widehat y_i+\sum_{k\neq i}y_k({\widehat y_i})} \\ &={\widehat y_i\left(\sum_ky_k\right)}-y_i \\ 
&=\widehat y_i-y_i
\end{split} $$
其中因为有onehot，$\sum_ky_k=1,y_i=1,y_{i \neq k}=0$

### (c)推导单隐层神经网络的梯度
![image.png](http://upload-images.jianshu.io/upload_images/2812342-d2e4d20879071fd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
根据题意，
$h=sigmoid(xw_1+b_1)$
$\widehat y=softmax(hw_2+b_2)$
令：
$z_1=xw_1+b_1$
$z_2=hw_2+b_2$

* 该网络cost function的导数
$$\begin{split} 
\frac{\partial CE}{\partial x} 
&= \frac{\partial CE}{\partial z_2}*\frac{\partial z_2}{\partial h}*\frac{\partial h}{\partial z_1}*\frac{\partial z_1}{\partial x}\\
&= (\widehat y-y)*(w_2^T)*(sigmoid^{'}(z_1))*(w_1^T)\\
&= (\widehat y-y)*(w_2^T)*(sigmoid(z_1)(1−sigmoid(z_1)))*(w_1^T)\\
&= (\widehat y-y)*(w_2^T)*(sigmoid(xw_1+b_1)(1−sigmoid(xw_1+b_1)))*(w_1^T)
\end{split} $$

### (d)神经网络的参数数量计算
输入$D_x=m$，输出$D_y=n$
再加上h个$b_1和D_y个b_2$
共有参数$$(D_x+1)*h+(h+1)*D_y$$

### (e)sigmoid的激活函数和梯度的代码
完成q2_sigmoid.py

### (f)梯度检查代码
完成q2_gradcheck.py

### (g)神经网络代码
完成q2 neural.py

## word2vec(40 points + 5 bonus)
### (a)
待
### (b)
待
### (c)
待
### (d)
待
### (e) 
q3_word2vec.py
### (f)
q3_sgd.py
### (g)测试运行
python q3 run.py
### (h)
在q3_word2vec.py中实现CBOW
## 情感分析
### (a) 
q4_softmaxreg.py
### (b)
解释当分类语料少于三句时为什么要引入正则化（实际上在大多数机器学习任务都这样）
### (c)
q4 sentiment.py
### (d)
绘图

## Appendix
[Pset1 tutorial](http://cs224d.stanford.edu/assignment1/index.html)
[Pset1 assignment1](http://cs224d.stanford.edu/assignment1/assignment1.pdf)
[Pset1 solutions Code](http://cs224d.stanford.edu/assignment1/assignment1_sol.zip)
[Pset1 solutions](http://cs224d.stanford.edu/assignment1/assignment1_soln)
[斯坦福cs224d 大作业测验1与解答 ](http://blog.csdn.net/han_xiaoyang/article/details/51760923)

```Python
import numpy as np
if __name__ == "__main__":
## 实验说明axis是tuple作用是对某维度的投影，max则是和投影平行最大的面
## 二维数组 0 对x的投影 最大行； 1对y投影最大列
## keepdims值为True则是保持维度
    y = np.array([[1,2],
                   [3,4]])
    ## print len(x.shape)
    print np.max(y,axis=0,keepdims=True)
    print np.max(y,axis=0,keepdims=False)
    print np.max(y,axis=1,keepdims=True)
    
    x = np.array([[[1,2],
                   [3,4]],
                  [[5,6],
                   [7,8]]])
    print np.max(x,axis=(0,1),keepdims=True)
```


