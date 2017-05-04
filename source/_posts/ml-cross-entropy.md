---
title: Machine Learning - Cross-Entropy Loss for Softmax
categories: 机器学习
tags: [机器学习,数学]
date: 2017-05-04
mathjax: true
toc: true
---

When we use Cross-Entropy Loss as **Cost Function** for Softmax, we need to know its partitial derivatvies. Here I want to do some mathematics inference for Cross-Entropy Loss,

**May Fourth be with you.**
<!--more-->
## Defintions ##
### Hypothesis Function ###
$h(x^{(i)}) = \frac{1}{\sum\_{j=1}^Ke^{\theta\_jx^{(i)}}}\left[\begin{matrix}e^{\theta\_1x^{(i)}}\\\\ e^{\theta\_2x^{(i)}}\\\\\vdots\\\\e^{\theta\_kx^{(i)}}\end{matrix}\right]$

In vector form
$h(x^{(i)}) = \frac{1}{\sum\_{j=1}^Ke^{\theta\_jx^{(i)}}}e^{\Theta x^{(i)}}$

where $x^{(i)}$ is $N\times1$ vector for a single training data
$\Theta$ is $K\times N$ weight matrix, so $\theta\_i$ is $1\times N$ vector
### Cross-Entropy Loss Function ###
$E(\theta)=-\frac{1}{N}\sum_{i\in N}y^{(i)}\circ log(h(x^{(i)}))$

where $y^{(i)}$ is a $1\times K$ labeled data vector, which is in $[0, ...,0, 1, 0, ..., 0]$ form
and $log(h(x^{(i)}))$ is $1\times K$ vector
## Partial Derivatives ##
Let's define
$z^{(i)} = \Theta\cdot x^{(i)}$, where is $K \times 1$ vector
$o^{(i)} = h(x^{(i)})$

$\frac{\partial E}{\partial z^{(i)}} = \frac{\partial }{\partial z^{(i)}}[-\frac{1}{N}\sum\_{n\in N}(y^{(n)}\circ log(o^{(n)}))]$

Here we can remove $\sum_{n\in N}$ because $z^{(i)}$ only affects $y^{(i)}\circ log(o^{(i)}))$
$\frac{\partial E}{\partial z^{(i)}}=-\frac{1}{N} \frac{\partial }{\partial z^{(i)}}(y^{(i)}\circ log(o^{(i)}))$

Let's assume $y\_j^{(i)}=1$ when $j=k$ and $y\_j^{(i)}=0$ when $j \neq k$
$y^{(i)}\circ  log(o^{(i)})=\sum_{j=1}^Ky_j^{(i)}log(o\_j^{(i)})=log(o\_k^{(i)}),$ where $y\_k^{(i)}=1$

$\frac{\partial E}{\partial z^{(i)}}=-\frac{1}{N} \frac{\partial }{\partial z^{(i)}}log(o\_k^{(i)})\cdot I$, where $I$ is $K\times 1$ vector
$=-\frac{1}{N} \frac{\partial }{\partial z^{(i)}}log(\frac{e^{z\_k^{(i)}}}{\sum\_{j=1}^Ke^{z\_j^{(i)}}})\cdot I$
$=-\frac{1}{N} \frac{\partial }{\partial z^{(i)}}({z\_k^{(i)}}- log(\sum\_{j=1}^Ke^{z\_j^{(i)}}))\cdot I$
$=-\frac{1}{N} (I\_k-\frac{\partial }{\partial z^{(i)}}log(\sum_{j=1}^Ke^{z\_j^{(i)}})\cdot I),$   where $I\_k=y^{(i)}$ because $y\_k^{(i)}=1$
$=-\frac{1}{N} (y^{(i)}-\frac{1}{\sum\_{j=1}^Ke^{z\_j^{(i)}}}\frac{\partial }{\partial z^{(i)}}\sum\_{j=1}^Ke^{z\_j^{(i)}}\cdot I)$

Let's consider every element for the vector form: 
$\frac{\partial }{\partial z\_k^{(i)}}\sum\_{j=1}^Ke^{z\_j^{(i)}} = e^{z\_k^{(i)}}$
so
$\frac{1}{\sum\_{j=1}^Ke^{z\_j^{(i)}}}\frac{\partial }{\partial z^{(i)}}\sum\_{j=1}^Ke^{z\_j^{(i)}}\cdot I=\frac{1}{\sum\_{j=1}^Ke^{z\_j^{(i)}}}\left[\begin{matrix}e^{\theta\_1x^{(i)}}\\\\ e^{\theta\_2x^{(i)}}\\\\\vdots\\\\e^{\theta\_kx^{(i)}}\end{matrix}\right]=o^{(i)}$
So 
$\frac{\partial E}{\partial z^{(i)}}=-\frac{1}{N} (y^{(i)}-o^{(i)})$
In vector form
$\frac{\partial E}{\partial z}=-\frac{1}{N} (y-o)$
So for
$\frac{\partial E}{\partial \Theta}=\frac{\partial E}{\partial z}\frac{\partial  z}{\partial \theta}
=-\frac{1}{N} (y-o)\frac{\partial}{\partial \Theta}(\Theta\cdot x)=-\frac{1}{N} (y-o)\cdot x$

## Summary #
$\delta = \frac{\partial E}{\partial z}=-\frac{1}{N} (y-h(x))$

$\frac{\partial E}{\partial \Theta}=-\frac{1}{N} (y-h(x))\cdot x$
