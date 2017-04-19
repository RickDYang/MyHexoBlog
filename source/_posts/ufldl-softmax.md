---
title:  Stanford UFLDL - Softmax Regression
categories: 机器学习
tags: [机器学习]
date: 2017-04-12
mathjax: true
toc: true
---

尝试从[Stanford UFLDL][1]的课后练习入手，从代码角度说明Softmax Regression算法流程。

Softmax Regression解决的是多分类问题，如果是二分类问题，Softmax Regression即退化为Logistic Regression。
<!--more-->
## 假设函数 ##
$h(x) = \frac{1}{\sum\_{j=1}^ke^{\theta\_jx}}\left[\begin{matrix}e^{\theta\_1x}\\\\ e^{\theta\_2x}\\\\\vdots\\\\e^{\theta\_kx}\end{matrix}\right]$，其中 $k$ 为分类个数
$h(x)$就是 $k$ 维向量，表示每个分类的概率。
$\theta$为$k\times (n+1)$参数矩阵，而x为$(n+1)\times1$数据向量
**代码**
```matlab
M = theta * data;
M = bsxfun(@minus, M, max(M,[],1));

M = exp(M);
M = bsxfun(@rdivide, M, sum(M));
```
theta 是 $k \times (n+1)$ 矩阵而 data 是 $(n+1) \times m$矩阵，这里即计算$\Theta\cdot X$
而$e^{\theta\_jx}$可能会很大，加总后可能会越界。所以这里对数据进行一下处理，减去其中的最大值，使最终结果减小。而减去这个固定值，并不会影响最终$h(x)$的计算结果。
```matlab
M = theta * data;
M = bsxfun(@minus, M, max(M,[],1));
```
这段代码即计算最终$h(X)$的结果，计算$e^{\Theta \cdot X}$，并对结果进行归一化。
```matlab
M = exp(M);
M = bsxfun(@rdivide, M, sum(M));
```
## Cost Function ##
$J(\theta)=-\frac{1}{m}\left[\sum\_{i=1}^m\sum\_{j=1}^k1\\{y^{(i)}=j\\}log\frac{e^{\theta\_jx^{(i)}}}{\sum\_{l=1}^{l}e^{\theta\_lx^{(i)}}}\right]+ \frac{\lambda}{2}\sum\_{i=1}^k\sum\_{j=0}^n\theta\_{ij}$
其中$1\\{y^{(i)}=j\\} = 1,$ only if $y^{(i)}=j$, othewise $1\\{y^{(i)}=j\\} = 0$
偏导
$\frac{\partial}{\partial\theta\_j}J(\theta)=-\frac{1}{m}\left[\sum\_{i=1}^mx^{(i)}(1\\{y^{(i)}=j\\}-p(y^{(i)}=j\mid x^{(i)};\theta))\right]+\lambda\theta\_j$
$p(y^{(i)}=j\mid x^{(i)};\theta)=\frac{e^{\theta\_jx^{(i)}}}{\sum\_{l=1}^ke^{\theta\_lx}},$即$h(x^{(i)})\_j$
**代码**
生成$1\\{y^{(i)}=j\\}$矩阵，groundTruth为$k\times m$矩阵，groundTruth(i,j) == 1 only lables(j) == i，即第j数据的期望结果为i
```matlab
groundTruth = full(sparse(labels, 1:numCases, 1));
```
计算Cost Function
M即为上面计算的$h(X)$
```matlab
cost = - sum(sum(log(M).*groundTruth)) / numCases + lambda * sum(sum(theta.^2))/2;
```
计算偏导
```matlab
thetagrad =  - (groundTruth - M) * data' / numCases + lambda * theta;
```


  [1]: http://ufldl.stanford.edu/wiki/index.php/UFLDL_Tutorial
