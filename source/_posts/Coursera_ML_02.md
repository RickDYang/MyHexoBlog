---
title: Study Notes II - Coursera Machine Learning
mathjax: true
categories: 机器学习
tags: [机器学习, Coursera, 数学, 学习笔记]
date: 2017-03-13
toc: true
---
Study notes part 2 for machine learning in Coursera. Will demonstrate some mathematics behinds lessons and summarize keys points.

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
```matlab
for i = 1:m
    % calculate final hx
    hx = x(i);
    for j = 2:L
        hx = sigmoid([1 hx] * theta(j-1)); % 1 is to add bias unit
    end
    % end of calculate final hx
    J += - log(hx) * y - log(1 - hx) * (1 - y);
end
J = J / m;
```

- Regularization part
$\frac{\lambda}{2m}\sum\_{l=1}^{L-1} \sum\_{i=1}^{s\_l} \sum\_{j=1}^{s\_{l+1}} ( \Theta\_{j,i}^{(l)})^2$
$\Theta$ in each layer is matrix to map value from one layer to next layer. So $\Theta$ has three dimentions notation.
The above regulariztion part is just square sum of all $\theta$ except bias unit in entire $\Theta$ network.
Pseudo code in Octave:
```matlab
for i = 1:L - 1
    J_r += sum(sum(theta(i).^2));
end
J_r = J_r * lambda/(2 * m);
```
### 2. Backpropagation Algorithm
Backpropagation Algorithm steps:
- For each trainng sample
Forward propagation to calculte all hypothesis values in each layer till to output layer, $a^l$ for $l=2, 3, ..., L$
- Compute $\delta^{(L)}=a^{(L)} - y$
- Compute $\delta^{(L-1)}, \delta^{(L-2)},\dots,\delta^{(2)}$, by $\delta^{(l)} = ((\Theta^{(l)})^T \delta^{(l+1)}) \cdot (a^{(l)}(1 - a^{(l)}))$
where $g'(z^{(l)}) = a^{(l)}(1 - a^{(l)})$, as mentioned in my previous note
- Sum $\Delta^{(l)} := \Delta^{(l)} + \delta^{(l+1)}(a^{(l)})^T$
- $D^{(l)}\_{i,j} := \dfrac{1}{m}\left(\Delta^{(l)}\_{i,j} + \lambda\Theta^{(l)}\_{i,j}\right)$

#### 2.1. Backpropagation Implementation
Pseudo code in Octave:
```matlab
for i = 1:m
    % Forward propagation
    a(1) = x(i);
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

- Partial Derivative of Hypothesis part
$\frac\partial {\partial \Theta\_{ij}^{(l)}}[-\frac{1}{m} \sum\_{i=1}^m (y \log (h\_\theta (x)) + (1 - y)\log (1 - h\_\theta(x)))]$
for $\theta\_{ij}^{(l)}$ it means in layer $l$ map $x\_i^{(l)}$ to $x\_{j}^{(l+1)}$
**What is $\delta\_i^{(l)}$**
Definition
$\delta_i^{(l)} = \frac{\partial}{\partial z_i^{(l)}}J(\Theta)$
where $l$ is $lth$ layer in neural network, and $i$ is $ith$ node in this $lth$ layer.
In vector form, we can define $\delta^{(l)} = \frac{\partial}{\partial z^{(l)}}J(\Theta)$
So we calculate in vector form (ignore subscript $i$) to simplify.
Firstly we have the following equations by definition,
$z^{(l)} = a^{(l-1)}\cdot \theta^{(l-1)} $, where $l = 2, 3, ..., L$
$a^{(l)} = f(z^{(l)})$, where $f$ is sigmoid function. 
To simplify, we just use one sample $(x, y)$ and one dimention form for $x, y, \theta$ to conduct. The provement is same in vector form.
**Chain Rule Theorem:** if $h(x)=f(g(x))$, $\frac{\partial}{\partial x}h(x)
=\frac{\partial}{\partial g(x)} f(g(x))\cdot\frac{\partial}{\partial x}g(x)$
Let's prove:
>**i. For output layer $L$** 
$\delta^{(L)} = \frac{\partial}{\partial z^{(L)}}J(\Theta) 
= \frac{\partial}{\partial a^{(L)}}J(\Theta)\cdot  \frac{\partial}{\partial z^{(L)}}a^{(L)}$
$= \frac{\partial}{\partial a^{(L)}}(-ylog(a^{(L)})-(1-y)log(1-a^{(L)}))\cdot  \frac{\partial}{\partial z^{(L)}}a^{(L)}$
$= (-\frac{y}{a^{(L)}} + \frac{1-y}{1-a^{(L)}})\cdot  \frac{\partial}{\partial z^{(L)}}a^{(L)}
= (-\frac{y}{a^{(L)}} + \frac{1-y}{1-a^{(L)}})\cdot f'(z^{(L)})$
$=(-\frac{y}{a^{(L)}} + \frac{1-y}{1-a^{(L)}})\cdot a^{(L)}(1-a^{(L)})$
$=a^{(L)}-y$
**ii.For $l = L-1, L-2, ..., 2,$**
$\delta^{(l)} = \frac{\partial}{\partial z^{(l)}}J(\Theta) =\frac{\partial}{\partial z^{(l+1)}}J(\Theta)\cdot \frac{\partial}{\partial z^{(l)}} z^{(l+1)} $
$= \delta^{(l+1)}\cdot \frac{\partial}{\partial z^{(l)}} z^{(l+1)}
= \delta^{(l+1)}\cdot \frac{\partial}{\partial z^{(l)}}(a^{(l)}\theta^{(l)})$
$=\delta^{(l+1)}\theta^{(l)}\cdot \frac{\partial}{\partial z^{(l)}}a^{(l)}
= \delta^{(l+1)}\theta^{(l)}\cdot \frac{\partial}{\partial z^{(l)}}f(z^{(l)})$
$=\delta^{(l+1)}\theta^{(l)}\cdot f'(z^{(l)})$
**iii. So for $\Delta^{(l)}$** 
$\Delta^{(l)}=\frac{\partial}{\partial{\theta^{(l)}}}J(\Theta)=\frac{\partial}{\partial{z^{(l+1)}}}J(\Theta)\cdot \frac{\partial}{\partial{\theta^{(l)}}}z^{(l+1)}$
$= \delta^{(l+1)} \cdot \frac{\partial}{\partial{\theta^{(l)}}}z^{(l+1)}
= \delta^{(l+1)} \cdot \frac{\partial}{\partial{\theta^{(l)}}}(a^{(l)}\theta^{(l)})$
$=\delta^{(l+1)}a^{(l)}$
- Partial Derivatives of Regularization part.
Conduction is trivial
$\frac\partial {\partial \Theta\_{ij}^l}(\frac{\lambda}{2m}\sum\_{l=1}^{L-1} \sum\_{i=1}^{s\_l} \sum\_{j=1}^{s\_{l+1}} ( \Theta\_{j,i}^l)^2)
= \frac{\lambda}{m}\Theta\_{ij}^l$


