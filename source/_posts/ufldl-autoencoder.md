---
title: Stanford UFLDL - Sparse Autoencoder 
categories: 深度学习
tags: [机器学习, 深度学习]
date: 2017-03-30
toc: true
mathjax: true
---
开始学习[Stanford UFLDL][1](Unsupervised Feature Learning and Deep Learning)，这门课有中文版，非常不错。
这是第一单元Sparse Autoencoder（自编码算法）的笔记总结。Sparse Autoencoder是用深度神经网络来学习恒等公式，即，$h(x)=x$
由于神经网络的隐藏层的节点数比输入层的节点少，所以不会出现平凡解（全是1）。自编码算法是一种非监督算法，也是一种压缩性算法。
<!--more-->

## 表述差异 ##
UFLDL上介绍的神经网络算法和Coursera Machine Learning上的表述方法有差异。UFLDL上没有给每一层加上一个等于1的bias unit，而是用参数b来表示bias。另外UFLDL用W来表示每个节点的权重。
计算函数表述为$h(W, b) = f(W\cdot x+b)$
这种差异会带来一些实现上的变化。
## 例题实现（Octave） ##
直接用例题来讲会比较清晰。
此课程的例题是实现一种图形压缩算法，输入是一个8x8的灰度图，神经网络只有一个隐藏层,节点有25个。所以整个神经网络构成一个64->25->64的一个映射。
和Coursera Machine Learning的编程题类似，代码下载解压后，主文件(train.m)有一堆m文件。大多数读写显示数据都提前实现好了，学习者只需要实现一些核心算法。
但例题中的代码视乎是针对Matlab，在Octave下有些问题。而我并不精通Octave，踩了一些坑。
### 支持方法 ###
例题需要实现sampleIMAGES.m来随机抽取图像样本，和computeNumericalGradient.m来验证sparseAutoencoderCost算法的实现正确性。
sampleIMAGES & 和computeNumericalGradient实现较简单，相关说明略。
#### 一些坑 ####
由于时间有限，我用简单方法跳过了这些坑。如果小伙伴们有更好的办法，请留言告诉我。
display_network.m里imagesc调用会出错，需要改成
```matlab
if opt_graycolor
    %h=imagesc(array,'EraseMode','none',[-1 1]);
    h=imagesc(array,'EraseMode','none');
else
    %h=imagesc(array,'EraseMode','none',[-1 1]);
    h=imagesc(array,'EraseMode','none');
end
```
train.m的梯度求解算法minFunc会出一个“lbfgsC”错误，目测可能lbfgsC是C文件而Octave对C支持有问题，没有深究原因。解决方法是把梯度算法替换成Machine Learning作业中用到的fmincg算法即可。
```matlab
options = optimset('MaxIter', 400);
[opttheta, cost] = fmincg(@(p) sparseAutoencoderCost(p, ...
                                   visibleSize, hiddenSize, ...
                                   lambda, sparsityParam, ...
                                   beta, patches), ...
								theta, options);
```
### 核心算法 ###
sparseAutoencoderCost.m是核心算法，需要计算Cost值和参数偏导值。
注：下面的算法使用vector形式实现的，因为用loop m实现运算速度会非常慢。但在实际的工作中，遇到的训练数据量会非常大，所以还是需要通过loop m来实现。
sparseAutoencoderCost最好按例题建议的方式，分步实现。
#### 调试准备 ####
如果一开始的采样数据很大（10000）时，运算速度会非常慢，影响调试效率，所以最好把采样数据调小，50即可。
```matlab
% sampleIMAGES.m
numpatches = 50;
```
当然sampleIMAGES不要基于hardcoded 10000来实现。
Visualization部分也要改一下。
```matlab
% train.m
display_network(patches(:,randi(size(patches,2),36,1)),8);
```
#### 无参数实现 ####
代价函数Cost Function：
$J(W,b;x, y)=\frac{1}{m}\sum\_{i=1}^m(\frac{1}{2}(h\_{W,b}(x)-y)$
对于Autoencoder来说，$y = x$
求$J(W,b;x, y)$：Forwardpropagation
```matlab
% data = x
% forward
a2 = sigmoid(bsxfun(@plus, W1 * data, b1));
a3 = sigmoid(bsxfun(@plus, W2 * a2, b2));
delta3 = a3 - data;
% end forward
J = sum(sum(delta3.^2))/ ( 2 * m);
```
求参数偏导值：Backwardpropagation
对首项 $L$
$\delta^{(L)} = (a^{(L)} - y)\cdot f'(z^{(L)})=(a^{(L)} - x)\cdot((1-a^{(L)})\cdot a^{(L)})$
注：这里$f(z)$是sigmoid函数，所以$f'(z^{(L)})=(1-a^{(L)})\cdot a^{(L)}$
对 $l=L-1, L-2, ..., 2,$
$\delta^{(l)}=(W^{(l)}\delta^{(l+1)})\cdot ((1-a^{(l)})\cdot a^{(l)})$
```matlab
% backward
delta3 = delta3 .* (a3.*(1 - a3));
delta2 = (W2' * delta3).* (a2.*(1 - a2));
% end backward

W1grad = delta2 * data' / m;
W2grad = delta3* a2' / m;
b1grad = sum(delta2, 2) / m;
b2grad = sum(delta3, 2) / m;

cost = J;
```
注意与Logistic Regression的delta3计算有不同的是，delta3需要考虑$f'(z^{(3)})$
实现完成之后可以用computeNumericalGradient来检查计算结果
#### 引入Regularization ####
$J\_w(W,b;x, y)=\frac{\lambda}{2}\sum\_{l=1}^{n\_l-1}\sum\_{i=1}^{s\_l}\sum\_{j=1}^{s\_l+1}
(W\_{ji}^{(l)})^2$
```matlab
J_w = (sum(sum(W1.^2)) + sum(sum(W2.^2))) * lambda / 2;
cost = J + J_w;
```
求偏导
$\frac{\partial}{\partial W\_{ij}^{(l)}}J\_w(W,b;x, y)=\lambda W\_{ij}^{(l)}$
```matlab
W1grad = delta2 * data' / m  + W1 * lambda;
W2grad = delta3* a2' / m  + W2 * lambda;
b1grad = sum(delta2, 2) / m;
b2grad = sum(delta3, 2) / m;
```
#### 引入稀疏性参数 ####
$\rho\_j=\frac{1}{m}\sum\_{i=1}^m[a\_j^{(2)}(x^{(i)})]$，即对$a^{(2)}$做平均
$J\_{sparse}(W,b)=\beta\sum\_{j=1}^{s\_2}(\rho log \frac{\rho}{\rho\_j}-(1-\rho) log \frac{1-\rho}{1-\rho\_j})$
```matlab
rho = sum(a2,2) / m;
rho1 = sparsityParam ./ rho;
rho2 = (1 - sparsityParam) ./ (1 - rho);

KL = sparsityParam * log(rho1) + (1 - sparsityParam) * log(rho2);
J_KL = beta * sum(KL);

cost = J + J_w + J_KL;
```
稀疏性参数对偏导的影响
$\delta^{(l)}=(W^{(l)}\delta^{(l+1)} + \beta\cdot(\frac{1-\rho}{1-\rho_i}-\frac{\rho}{\rho_i}))\cdot ((1-a^{(l)})\cdot a^{(l)})$
```matlab
delta2 = bsxfun(@plus, W2' * delta3, beta * (rho2 - rho1)).* (a2.*(1 - a2));
```
### 最后结果 ###
一切检验完毕，最后train.m会调用梯度求解算法计算出结果。
这里例题的计算速度主要受computeNumericalGradient的影响，如果确定computeNumericalGradient准确无误后，可以跳过这步，直接用梯度求解即可。

  [1]: http://ufldl.stanford.edu/wiki/index.php/UFLDL_Tutorial
