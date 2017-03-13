---
title: Study Notes II - Coursera Machine Learning
mathjax: true
categories: 机器学习
tags: [机器学习, Coursera, 数学, 学习笔记]
date: 2017-03-13
toc: true
---
Study notes part 1 for machine learning in Coursera. Will demonstrate some mathematics behinds lessons and summarize keys points.

### 1. Neural Network
**Cost Function**
$J(\Theta) = - \frac{1}{m} \sum\_{i=1}^m \sum\_{k=1}^K \left[y^{(i)}\_k \log ((h\_\Theta (x^{(i)}))\_k) + (1 - y^{(i)}\_k)\log (1 - (h\_\Theta(x^{(i)}))\_k)\right] + \frac{\lambda}{2m}\sum\_{l=1}^{L-1} \sum\_{i=1}^{s\_l} \sum\_{j=1}^{s\_{l+1}} ( \Theta\_{j,i}^{(l)})^2$
$K$ = number of output classes or classifications.
$L$ = total numbers of layers including input & ouput layers
$s_l$ = numbers of units (not including bias unit) in layer $l$
<!--more-->

- Hypothesis part
$-\frac{1}{m} \sum\_{i=1}^m \sum\_{k=1}^K \left[y^{(i)}\_k \log ((h_\Theta (x^{(i)}))\_k) + (1 - y^{(i)}\_k)\log (1 - (h\_\Theta(x^{(i)}))\_k)\right]$
where $y$ is $K$ length vector which ith elment is 1 only when $y$ is ith class while others are 0.
vectorization form:
$-\frac{1}{m} \sum\_{i=1}^m (y \log (h\_\theta (x)) + (1 - y)\log (1 - h\_\theta(x)))$
The above part is just sum of each data for cost of final hypothesis vs. real result.
Pseudo code in Octave
```Octave
for i = 1:m
    % calculate final hx
    hx = x;
    for j = 2:L
        hx = sigmoid([1 hx] * theta(j-1)); % 1 is to add bias unit
    end
    % end of calculate final hx
    J += - log(hx) * y - log(1 - hx) * (1 - y);
end
J = J / m;
```

- Regularizion part
$\frac{\lambda}{2m}\sum\_{l=1}^{L-1} \sum\_{i=1}^{s\_l} \sum\_{j=1}^{s\_{l+1}} ( \Theta\_{j,i}^l)^2$
$\Theta$ in each layer is matrix to map value from one layer to next layer. So $\Theta$ has three dimentions notation.
The above regulariztion part is just square sum of all $\theta$ except bias unit in entire $\Theta$ network.
Pseudo code in Octave:
```Octave
for i = 1:L - 1
    J_r += sum(sum(theta(i).^2));
end
J_r = J_r * lambda/(2 * m);
```
### 2. Backpropagation Algorithm
Backpropagation Algorithm steps:
>- For each trainng sample
Forward propagation to calculte all hypothesis values in each layer till to output layer, $a^l$ for $l=2, 3, ..., L$
- Compute $\delta^L=a^L - y$
- Compute $\delta^{L-1}, \delta^{L-2},\dots,\delta^2$, by $\delta^l = ((\Theta^l)^T \delta^{l+1}) \cdot (a^l(1 - a^l))$
where $g'(z^l) = a^l(1 - a^l)$, as mentioned in my previous note
- Sum $\Delta^l := \Delta^l + \delta^{l+1}(a^l)^T$
- $D^l\_{i,j} := \dfrac{1}{m}\left(\Delta^l\_{i,j} + \lambda\Theta^l\_{i,j}\right)$
#### 2.1. Backpropagation Implementation
Pseudo code in Octave:
```Octave
for i = 1:m
    % Forward propagation
    a(1) = x;
    for j = 2:L
        a(j) = sigmoid([1 a(j-1)] * theta(j-1)); % 1 is to add bias unit
    end
    % end Forward propagation
    
    delta(L) = y - a(L);
    % Backpropagation
    for j = L - 1 :2
        % theta_ is theta exclude bias unit
        delta(j) = (theta_(j) * delta(j+1)).*(a(j).*(1 - a(j)));
        theta_grad(j) += delta(j+1) * [1 a(j)];
    end
    % end Backpropagation
end
 % theta_ is theta exclude bias unit
theta_grad = theta_grad / m + theta_ * lambda / m;
```
#### 2.2. Mathematics behind backpropagation
Backpropagation algorithm is like that a pound of water (input) flow into a (neural) network. We get each water in each level(layer) until the final layer(output). Every $\theta$ is like a pipe between each node. we want to know speed of water in a pipe (partial derivative) , so we use backward calculation to deduce the derivatives.
Backpropagation algorithm is a kind of **DP** (dynamic programming) algorithm, which reduce duplicate calculation to save time.

- Partial of Hypothesis part
$\frac\partial {\partial \Theta\_{ij}^l}[-\frac{1}{m} \sum\_{i=1}^m (y \log (h\_\theta (x)) + (1 - y)\log (1 - h\_\theta(x)))]$
for $\theta\_{ij}^l$ it means in layer $l$ map $x\_i^l$ to $x\_{j}^{l+1}$
To simplify, we just use one sample $(x, y)$ and one dimention form for $x, y, \theta$ to conduct. The provement is same in vector form.
Chain Rule Theorem: 
if $h(x)=f(g(x))$, $\frac{\partial}{\partial x}h(x)
=\frac{\partial}{\partial g(x)} f(g(x))\cdot\frac{\partial}{\partial x}g(x)$
Let's prove backpropagation formula: $\Delta^l=\delta^{l+1}a^l$
>**i.**
For simplification, let's define cost function $J(\theta,x)=-y log(h\_\theta(x))  -(1-y)log(1-h\_\theta(x))$
Recall derivatives formula in logistic regression
$ \frac{\partial}{\partial\theta}J(\theta,x) = (h\_\theta(x)-y) x,$
$h\_\theta(x)=g(\theta x),$ symmetrically we can get the following formula
$ \frac{\partial}{\partial x}J(\theta,x)= (h\_\theta(x)-y) \theta,$
**ii.**
For output layer $L$
$\Delta^{L-1}=\frac{\partial}{\partial{\theta^{L-1}}}J(\theta,x)=\frac{\partial}{\partial{\theta^{L-1}}}J(\theta^{L-1},a^{L-1})$
$= (h\_{\theta^{L-1}}(a^{L-1})-y)a^{L-1}$
$=(a^L-y)a^{L-1} = \delta^La^{L-1}$
So backpropagation algorithm is correct for output $L$ layer,
Also symmetrically $\frac{\partial}{\partial{a^{L-1}}}J(\theta,x)=\delta^L\theta^{L-1}$
**iii.**
Let's firstly prove: $\frac{\partial}{\partial a^l}J(\theta,x)=\delta^{l+1}\theta^l$
It is true for layer $L$. 
Assume it is true for $l$, for layer $l-1$
$\frac{\partial}{\partial{a^{l-1}}}J(\theta,x)=\frac{\partial}{\partial{a^l}}J(\theta,x)\cdot \frac{\partial}{\partial{a^{l-1}}}a^l=\delta^{l+1}\theta^l\cdot \frac{\partial}{\partial{a^{l-1}}}a^l$
$=\delta^{l+1}\theta^l\cdot\frac{\partial}{\partial{a^{l-1}}}h\_{\theta^{l-1}}(a^{l-1})= \delta^{l+1}\theta^l\cdot h\_{\theta^{l-1}}(a^{l-1})(1-h\_{\theta^{l-1}}(a^{l-1}))\cdot\theta^{l-1}$
$=\delta^{l+1}\theta^l\cdot a^l(1-a^l)\theta^{l-1}=[\delta^{l+1}\theta^l a^l(1-a^l)]\cdot\theta^{l-1}$
$=\delta^l\theta^{l-1}$  by backpropagation definition $\delta^1=\theta^l\delta^{l+1}a^l(1-a^l)$
So it is true for $l-1$.
By mathematical induction $\frac{\partial}{\partial a^l}J(\theta,x)=\delta^{l+1}\theta^l,$ for $l=L-1, ..., 1$
**iv.**
$\Delta^{l-1}=\frac{\partial}{\partial{\theta^{l-1}}}J=\frac{\partial}{\partial{a^l}}J\cdot \frac{\partial}{\partial{\theta^{l-1}}}a^l$
$=\delta^{l+1}\theta^l\cdot \frac{\partial}{\partial{\theta^{l-1}}}a^l=\delta^{l+1}\theta^l\cdot \frac{\partial}{\partial{\theta^{l-1}}}h\_{\theta^{l-1}}(a^{l-1})$
$=\delta^{l+1}\theta^l\cdot a^l(1-a^l)a^{l-1}=[\delta^{l+1}\theta^l\cdot a^l(1-a^l)]a^{l-1}$
$=\delta^la^{l-1}$
Done
- Partial Derivatives of Regularizion part.
Conduction is trivial
$\frac\partial {\partial \Theta\_{ij}^l}(\frac{\lambda}{2m}\sum\_{l=1}^{L-1} \sum\_{i=1}^{s\_l} \sum\_{j=1}^{s\_{l+1}} ( \Theta\_{j,i}^l)^2)
= \frac{\lambda}{m}\Theta\_{ij}^l$


