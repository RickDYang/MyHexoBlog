---
title:  Stanford UFLDL - Self-Taught Learning
categories: 机器学习
tags: [机器学习]
date: 2017-04-12
mathjax: true
toc: true
---

尝试从[Stanford UFLDL][1]的课后练习入手，从代码角度说明Self-Taught Learning算法流程。

自我学习Self-Taught即从大量**无标注**的数据中学习特征（features），然后在用**标注**数据进行训练。而从互联网我们可以抓取大量的无标注数据来帮助我们更好地改进我们的机器学习算法，尤其是我们无法直观或者简单地找到数据特征时。
课后练习从MNIST入手，讲述了整个自我学习的流程
<!--more-->
## 流程 ##
- 从大量无标注数据学习特征
- 将标注数据的输入数据转化相应特征
- 标注数据的特征作为输入进行机器学习训练。

## 构造数据 ##
数据源是MNIST——手写数字库。
练习将MNIST的训练数据划分从2组，0-4是**标注**组，而1-5是**非标注**组。
而**标注**组分成了训练集和测试集。
```Octave
% 0-4 are labeled while 5-9 is unlabeled
labeledSet   = find(mnistLabels >= 0 & mnistLabels <= 4);
unlabeledSet = find(mnistLabels >= 5);

% divide labeled set to train & test set
numTrain = round(numel(labeledSet)/2);
trainSet = labeledSet(1:numTrain);
testSet  = labeledSet(numTrain+1:end);

trainData   = mnistData(:, trainSet);
testData   = mnistData(:, testSet);

% unlabeled set
unlabeledData = mnistData(:, unlabeledSet);
```
## 学习特征 ##
用之前实现的sparse autoencoder算法来学习特征（从之前练习中把相应依赖文件拷贝过来即可）。
传入的数据即是unlabeledData，最后得到训练好的opttheta，而W1 & b1则是我们期望得到的特征。
```Octave
% sparse autoencoder train
options = optimset('MaxIter', maxIter);
[opttheta, cost] = fmincg(@(p) sparseAutoencoderCost(p, ...
                                   inputSize, hiddenSize, ...
                                   lambda, sparsityParam, ...
                                   beta, unlabeledData), ...
								theta, options);
								
W1 = reshape(opttheta(1:hiddenSize * inputSize), hiddenSize, inputSize);
```
我们把W1视图化后，你会发现其表现了数字的某种笔画特征
![此处输入图片的描述][2]
## 特征转换 ##
用我们上面计算出来W1对训练和测试数据进行转换，得到的结果是sparse autoencoder算法的隐藏层中的值。
```Octave
trainFeatures = feedForwardAutoencoder(opttheta, hiddenSize, inputSize, ...
                                       trainData);

testFeatures = feedForwardAutoencoder(opttheta, hiddenSize, inputSize, ...
                                       testData);
```
即从opttheta得到W1 & b1，然后计算$a_1 = f(x\cdot W_1 + b_1)$
feedForwardAutoencoder实现
```Octave
function [activation] = feedForwardAutoencoder(theta, hiddenSize, visibleSize, data)

% We first convert theta to the (W1, W2, b1, b2) matrix/vector format, so that this 
% follows the notation convention of the lecture notes. 

W1 = reshape(theta(1:hiddenSize*visibleSize), hiddenSize, visibleSize);
b1 = theta(2*hiddenSize*visibleSize+1:2*hiddenSize*visibleSize+hiddenSize);

%  Instructions: Compute the activation of the hidden layer for the Sparse Autoencoder.

activation = sigmoid(bsxfun(@plus, W1 * data, b1));

end
```
## 算法训练 ##
用之前实现的Softmax算法来学习特征（从之前练习中把相应依赖文件拷贝过来即可）。
传入的数据是trainFeatures & trainLabels
```Octave
lambda = 1e-4;  
options.maxIter = maxIter;
softmaxModel = softmaxTrain(hiddenSize, numLabels, lambda, ...
                            trainFeatures, trainLabels, options);
```
注意softmaxTrain的input size是hiddenSize，因为Softmax的输入已经变成了trainFeatures。
### 检验测试 ###
传入testFeatures & testLabels
```Octave
pred = softmaxPredict(softmaxModel, testFeatures);

% Classification Score
fprintf('Test Accuracy: %f%%\n', 100*mean(pred(:) == testLabels(:)));
```
至此我们实现了整个自我学习的流程。

  [1]: http://ufldl.stanford.edu/wiki/index.php/UFLDL_Tutorial
  [2]: http://ufldl.stanford.edu/wiki/images/8/84/SelfTaughtFeatures.png
