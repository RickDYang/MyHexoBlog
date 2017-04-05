---
title: Mathematics Behind Backpropagation in Neural Network
categories: 深度学习
tags: [机器学习, 深度学习,数学,学习笔记]
date: 2017-03-31
toc: true
mathjax: true
---

[Ufldl][1] introduces a provement for backpropagation in neural network. Let's summarize here and include sparsity penalty.
<!--more-->
### What's $\delta_i^{(l)}$ ###
Definition
$\delta_i^{(l)} = \frac{\partial}{\partial z_i^{(l)}}J(W,b)$
where $l$ is $lth$ layer in neural network, and $i$ is $ith$ node in this $lth$ layer.
In vector form, we can define $\delta^{(l)} = \frac{\partial}{\partial z^{(l)}}J(W,b)$
So we calculate in vector form (ignore subscript $i$) to simplify.
Firstly we have the following equation by definition,
$z^{(l)} = a^{(l-1)}\cdot W^{(l-1)} + b^{(l-1)}$, where $l = 2, 3, ..., L$
$a^{(l)} = f(z^{(l)})$, where $f$ is signal function. 
Typically we choose sigmoid function to be $f$. 
$f(z)=\frac{1}{1+e^{-z}}$
Sigmoid function has this property.
$f'(z)=(1-f(z))f(z)$
### Backpropagation without sparsity ###
Cost function : $J(W,b) = \frac{1}{2}(y-h_{W,b}(x))^2$
Let's define $L$ is the output layer.
>$\delta^{(L)} = \frac{\partial}{\partial z^{(L)}}J(W,b) = 
\frac{\partial}{\partial z^{(L)}}\frac{1}{2}(y-h_{W,b}(x))^2$
$= \frac{\partial}{\partial z^{(L)}}\frac{1}{2}(y - a^{(L)})^2
= \frac{\partial}{\partial z^{(L)}}\frac{1}{2}(y - f(z^{(L)}))^2$
$= -(y - f(z^{(L)}))\cdot f'(z^{(L)})$

For $l = L-1, L-2, ..., 2,$ we calculates $\delta^{(l)} = ((W^{(l)})^T\delta^{(l+1)}) .* f'(z^{(l)})$
Assume it is true for $l$ layer, 
>For $l-1$, 
$\delta^{(l-1)} = \frac{\partial}{\partial z^{(l-1)}}J(W,b) =\frac{\partial}{\partial z^{(l)}}J(W,b)\cdot \frac{\partial}{\partial z^{(l-1)}} z^{(l)} $
$= \delta^{(l)}\cdot \frac{\partial}{\partial z^{(l-1)}} z^{(l)}
= \delta^{(l)}\cdot \frac{\partial}{\partial z^{(l-1)}}(a^{(l-1)}W^{(l-1)}+b^{(l-1)})$
$=\delta^{(l)}W^{(l-1)}\cdot \frac{\partial}{\partial z^{(l-1)}}a^{(l-1)}
= \delta^{(l)}W^{(l-1)}\cdot \frac{\partial}{\partial z^{(l-1)}}f(z^{(l-1)})$
$=\delta^{(l)}W^{(l-1)}\cdot f'(z^{(l-1)})$
By mathematics conduction, prove done.

So we have
$\delta^{(l)} = \frac{\partial}{\partial z^{(L)}}J(W,b)=((W^{(l)})^T\delta^{(l+1)}) .* f'(z^{(l)})$
Backpropagation uses a dynamic programming method to calculate $\delta^{(l)}$ to improve the performance.
For partitial derivative of our parameters.
$\frac{\partial}{\partial W^{(l)}}J(W,b) = \frac{\partial}{\partial z^{(l+1)}}J(W,b)\cdot \frac{\partial}{\partial W^{(l)}}z^{(l+1)}$
$=\delta^{(l+1)}\cdot \frac{\partial}{\partial W^{(l)}}z^{(l+1)}
=\delta^{(l+1)}\cdot \frac{\partial}{\partial W^{(l)}}(a^{(l)}W^{(l)}+b^{(l)})$
$=\delta^{(l+1)}a^{(l)}$

$\frac{\partial}{\partial b^{(l)}}J(W,b) = \frac{\partial}{\partial z^{(l+1)}}J(W,b)\cdot \frac{\partial}{\partial b^{(l)}}z^{(l+1)}$
$=\delta^{(l+1)}\cdot \frac{\partial}{\partial b^{(l)}}z^{(l+1)}
=\delta^{(l+1)}\cdot \frac{\partial}{\partial b^{(l)}}(a^{(l)}W^{(l)}+b^{(l)})$
$=\delta^{(l+1)}$
### Backpropagation with sparsity ###
The following conduction only works on the neural network with one single hidden layer. I do not know how it works in other scenarios. Leave it to be an open issue.
$\rho\_j^{(2)}=\frac{1}{m}\sum\_{i=1}^m(a\_j^{(2)})$ for all data
$S\_l$ is number of node in layer $l$.
**Sparsity pernality**
$KL\_j^{(2)} = \rho log\frac{\rho}{\rho\_j^{(2)}}+(1-\rho) log\frac{1-\rho}{1-\rho\_j^{(2)}}$
$k'(x)=(alog\frac{a}{x}+(1-a)log\frac{1-a}{1-x})'=-\frac{a}{x}+\frac{1-a}{1-x}$
For overal cost function
$J\_{sparse}(W,b) = J(W,b) + \beta\sum\_{j=1}^{S\_2}KL\_j^{(2)}$
Let's calculate
$\frac{\partial}{\partial w\_{i,k}^{(1)}}\beta\sum\_{j=1}^{\S\_2}KL\_j^{(2)} $
$=\frac{\partial}{\partial \rho\_{k}^{(1)}}\beta\sum\_{j=1}^{S\_2}KL\_j^{(2)}\cdot \frac{\partial}{\partial w\_{i,k}^{(1)}}\rho\_{k}^{(2)}$
$=\beta\sum\_{j=1}^{S\_2}\frac{\partial}{\partial \rho\_{k}^{(2)}}(\rho log\frac{\rho}{\rho\_j^{(2)}}+(1-\rho) log\frac{1-\rho}{1-\rho\_j^{(2)}})\cdot \frac{\partial}{\partial w\_{i,k}^{(1)}}\rho\_{k}^{(2)}$
$=\beta(-\frac{\rho}{\rho\_k^{(2)}}+\frac{1-\rho}{1-\rho\_k^{(2)}})\cdot\frac{\partial}{\partial w\_{i,k}^{(1)}}\rho\_{k}^{(2)}$
$=\beta(-\frac{\rho}{\rho\_k^{(2)}}+\frac{1-\rho}{1-\rho\_k^{(2)}})\cdot\frac{\partial}{\partial w\_{i,k}^{(1)}}\frac{1}{m}\sum\_{i=1}^m(a\_k^{(2)})$
$=\frac{\beta}{m}(-\frac{\rho}{\rho\_k^{(2)}}+\frac{1-\rho}{1-\rho\_k^{(2)}})\cdot\frac{\partial}{\partial z\_{k}^{(1)}}\sum\_{j=1}^m(a\_k^{(2)})\cdot \frac{\partial}{\partial w\_{i,k}^{(1)}}z\_k^{(1)}$
$=\frac{\beta}{m}(-\frac{\rho}{\rho\_k^{(2)}}+\frac{1-\rho}{1-\rho\_k^{(2)}})\cdot\frac{\partial}{\partial z\_{k}^{(1)}}\sum\_{j=1}^m(a\_k^{(2)})\cdot \frac{\partial}{\partial w\_{i,k}^{(1)}}z\_k^{(1)}$
$=\frac{\beta}{m}(-\frac{\rho}{\rho\_k^{(2)}}+\frac{1-\rho}{1-\rho\_k^{(2)}})\cdot \sum\_{j=1}^mf'(z\_k^{(1)})\cdot \frac{\partial}{\partial w\_{i,k}^{(1)}}z\_k^{(1)}$
$=\frac{\beta}{m}(-\frac{\rho}{\rho\_k^{(2)}}+\frac{1-\rho}{1-\rho\_k^{(2)}})\cdot \sum\_{j=1}^mf'(z\_k^{(1)})\cdot a\_i^{(1)}$
$=\frac{\beta}{m}\sum\_{j=1}^m(-\frac{\rho}{\rho\_k^{(2)}}+\frac{1-\rho}{1-\rho\_k^{(2)}})\cdot f'(z\_k^{(1)})\cdot a\_i^{(1)}$
Similarily,
$\frac{\partial}{\partial b\_{k}^{(1)}}\beta\sum\_{j=1}^{S\_2}KL\_j^{(2)} = =\frac{\beta}{m}\sum\_{j=1}^m(-\frac{\rho}{\rho\_k^{(2)}}+\frac{1-\rho}{1-\rho\_k^{(2)}})\cdot f'(z\_k^{(1)})$
So by backpropagation conduction, we can calculate
$\delta\_k^{(2)}+=\beta(-\frac{\rho}{\rho\_k^{(2)}}+\frac{1-\rho}{1-\rho\_k^{(2)}})\cdot f'(z\_k^{(1)})$
But $\delta\_k^{(2)} \not= \frac{\partial}{\partial z\_k^{(2)}}J\_{sparse}(W,b)$
### Open Questions for Parsity Penalty###
- Does Parsity Penalty works for neural network with one single hidden layer?
- If not, how does Parsity Penalty work for neural network with more than one hidden layers? How to implement the backpropagation algorithm ?
  [1]: http://ufldl.stanford.edu/wiki/index.php/Backpropagation_Algorithm
