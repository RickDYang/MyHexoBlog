---
title: Study Notes I - Coursera Machine Learning
mathjax: true
categories: 机器学习
tags: [机器学习, Coursera, 数学, 学习笔记]
date: 2017-03-05
toc: true
---
Study notes part 1 for machine learning in Coursera. Will demonstrate some mathematics behinds lessons and summarize keys points.


### 1. Linear Regression ###

#### 1.1 Cost Function ####

$J(\theta) = \frac{1}{2m}\sum\_{i=1}^{m}( h\_{\theta}(x^i) - y^i)^2 $

**Vectorized form:**

$J(\theta) = \frac{1}{2m}(h_{\theta}(x) - y)^2 
= \frac{1}{2m}(x\cdot\theta-y)^T\cdot(x\cdot\theta-y)$

<!--more-->

#### 1.2 Partial Derivative of Cost function ####
$\frac{\partial}{\partial\theta\_j}J(\theta) = \frac{1}{m}\sum\_{i=1}^{m} ( h\_{\theta}(x^i) - y^i) \cdot x^j=(h\_\theta(x)-y)\cdot x^j$

**Prove:**

$\frac{\partial}{\partial\theta\_j}J(\theta) =
 \frac{\partial}{\partial\theta\_j}\frac{1}{2}\left(h\_{\theta}\left( x\right) - y \right)^2 $
 
$= 2 \cdot \frac{1}{2}(h\_\theta(x)-y)\cdot\frac{\partial}{\partial\theta\_j}(h\_\theta(x)-y)$

$ = (h_\theta(x)-y) \cdot \frac{\partial}{\partial\theta\_j}(x\cdot \theta - y)$

$= (h_\theta(x)-y) \cdot x^j$

Derivative Chain Rule:

$(f \circ g)' = f'(g(x)) \cdot g'(x)$

**Vectorized form:**

$\frac{\partial}{\partial\theta}J(\theta) = x^T\cdot(x\cdot\theta - y)$

### 2. Logistic Regression ###

#### 2.1 Sigmoid Function ####
$g(z)=\frac{1}{1+e^{-z}}$

$g'(z)$ has this property: $g'(z) = g(z)(1-g(z))$

**Prove:**

$g'(z)=(\frac{1}{1+e^{-z}})'$

$= \frac{e^x}{(1+e^x)^2} = \frac{1+e^x - 1}{(1+e^x)^2}$

$= \frac{1}{1+e^{-z}} - (\frac{1}{1+e^{-z}})^2$

$=g(z)-g^2(z) $

$=g(z)(1-g(z))$

**Hyphotheses Function:**

$h_\theta(x) = g(\theta^Tx)$


#### 2.2 Cost Function ####
 $J(\theta)=\frac{1}{m}\sum\_{i=1}^{m}Cost(h\_\theta(x^i), y^i)$
 
 $Cost(h\_\theta(x), y)= -log(h\_\theta(x)) $ ,    if $y=1$
 
 $Cost(h\_\theta(x), y)= -log(1-h\_\theta(x)) $ ,  if $y=0$
 
 Why choose this kind of Cost function
 
 if $y=1$, 
 
  $Cost(h\_\theta(x), y) = 0$, when $h\_\theta(x) == 1$
  
  $Cost(h\_\theta(x), y) \to\infty$, when $h\_\theta(x) == 0$
  
 if $y=0$, 
 
  $Cost(h\_\theta(x), y) = 0$, when $h\_\theta(x) == 0$
  
  $Cost(h\_\theta(x), y) \to\infty$, when $h\_\theta(x) == 1$
  
 So this cost function reflects the relationship between our hyphotheses and real values.
 
 **One case presentation**

$Cost(h\_\theta(x), y)= -y\cdot log(h\_\theta(x))  -(1-y)\cdot log(1-h\_\theta(x))$
 
 **vectorized form**
 
 $h=g(X\theta)=\frac{1}{1+e^{-X\theta}}$
 
 $J(\theta)=\frac{1}{m}\cdot (-y^Tlog(h)-(1-y)^Tlog(1-h))$
 
 
#### 2.3 Partial Derivative of Cost function ####

$\frac{\partial}{\partial\theta\_j}J(\theta) = \frac{1}{m}\sum\_{i=1}^{m} ( h\_{\theta}(x^i) - y^i) \cdot x^j$

**Prove:**

$\frac{\partial}{\partial\theta\_j}J(\theta)
= \frac{\partial}{\partial\theta\_j}(-y\cdot log(h\_\theta(x))  -(1-y)\cdot log(1-h\_\theta(x)))$

$= \frac{\partial}{\partial h\_\theta(x)}(-y\cdot log(h\_\theta(x))  -(1-y)\cdot log(1-h\_\theta(x)))\cdot \frac{\partial}{\partial\theta\_j}h\_\theta(x)$

$ = (-y\cdot\frac{1}{h\_\theta(x)} - (1-y)\cdot\frac{1}{h\_\theta(x)-1})\cdot\frac{\partial}{\partial\theta\_j}h\_\theta(x)$

$ = (\frac{-y}{h\_\theta(x)} + \frac{1-y}{1-h\_\theta(x)})\cdot h\_\theta(x)(1-h\_\theta(x))\cdot x^j$

$ = (-y\cdot (1-h\_\theta(x)) + (1-y)\cdot h\_\theta(x)) \cdot x^j$

$ = (h\_\theta(x)-y) \cdot x^j$

**where,** $log'(x)=\frac{1}{x}, log'(1-x)=\frac{1}{x-1},$

$\frac{\partial}{\partial\theta\_j}h\_\theta(x)=\frac{\partial}{\partial\theta\_j}g(\theta \cdot x) = g(\theta x)(1 - g(\theta x))\cdot \frac{\partial}{\partial \theta\_j}(\theta\cdot x)
=h\_\theta(x)(1-h\_\theta(x))\cdot x^j$

 **vectorized form**
 
 $\frac{\partial}{\partial\theta}J(\theta)=X^T(g(X\theta)-y)$
