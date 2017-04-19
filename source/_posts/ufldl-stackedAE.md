---
title:  Stanford UFLDL - Deep Networks
categories: 机器学习
tags: [机器学习,深度学习]
date: 2017-04-17
mathjax: true
toc: true
---
这次依旧从从Stanford UFLDL的课后练习入手，从代码角度说明Deep Neural Networks的算法流程。
Deep Neural Networks是隐藏层层数大于1的神经网络。课后练习也是从MNIST入手，以一个有2层隐藏层的为例子，构造一个典型的多分类问题的深度网络的算法流程。
<!--more-->
## 流程 ##
- 模型定义
首先定义神经网络的隐藏层，包括层数，每层的节点数
- 分层训练
利用稀疏自编码算法对每个隐藏层分别进行训练，最后的输出层用Softmax方法进行训练。这和之前自我学习（Self-Taught）的流程非常类似。
- 参数微调
得到每层的参数后，用这些参数作为初值，再在整个神经网络中进行训练，从而得到最终的最优参数。
- 验证
最后在测试集对结果进行验证。

## 准备 ##
需要之前实现的sparseAutoencoderCost & feedForwardAutoencoder & softmaxTraining等。
另外调试中可以暂时把maxIter和训练数据集调小，增加开发效率，否则程序运行会非常慢。等一切测试通过后，再调到需要的值。
```matlab
maxIter = 20; % 200

trainData = trainData(:, 1:100);
trainLabels = trainLabels(1:100,:);
```
## 分层训练 ##
用稀疏自编码训练第一层， sae1Theta是随机初始化的，训练的结果在sae1OptTheta
```matlab
%  Randomly initialize the parameters
sae1Theta = initializeParameters(hiddenSizeL1, inputSize);

options = optimset('MaxIter', maxIter);
[sae1OptTheta, cost] = fmincg(@(p) sparseAutoencoderCost(p, ...
                                   inputSize, hiddenSizeL1, ...
                                   lambda, sparsityParam, ...
                                   beta, trainData), ...
								sae1Theta, options);
```
转化为第一层的输出
```matlab
[sae1Features] = feedForwardAutoencoder(sae1OptTheta, hiddenSizeL1, ...
                                        inputSize, trainData);
```
用第一层的输出继续用稀疏自编码训练第二层
```matlab
%  Randomly initialize the parameters
sae2Theta = initializeParameters(hiddenSizeL2, hiddenSizeL1);

[sae2OptTheta, cost] = fmincg(@(p) sparseAutoencoderCost(p, ...
                                   hiddenSizeL1, hiddenSizeL2, ...
                                   lambda, sparsityParam, ...
                                   beta, sae1Features), ...
								sae2Theta, options);
```
最后用第二层的输出用Softmax训练最后的输出层
```matlab
[sae2Features] = feedForwardAutoencoder(sae2OptTheta, hiddenSizeL2, ...
                                        hiddenSizeL1, sae1Features);

%  Randomly initialize the parameters
saeSoftmaxTheta = 0.005 * randn(hiddenSizeL2 * numClasses, 1);

lambda = 1e-4;  
options.maxIter = maxIter;
softmaxModel = softmaxTrain(hiddenSizeL2, numClasses, lambda, ...
                            sae2Features, trainLabels, options);

saeSoftmaxOptTheta = softmaxModel.optTheta(:);
```

## 测试 ##
分层训练完之后，其实就可以跳过第5步的微调开始进行结果测试了。这里我们就能看到未微调前的结果。
```matlab
[pred] = stackedAEPredict(stackedAETheta, inputSize, hiddenSizeL2, ...
                          numClasses, netconfig, testData);

acc = mean(testLabels(:) == pred(:));
fprintf('Before Finetuning Test Accuracy: %0.3f%%\n', acc * 100);
```
stackedAEPredict实现：
stack即为表示隐藏层参数的数据结构。
```matlab
function [pred] = stackedAEPredict(theta, inputSize, hiddenSize, numClasses, netconfig, data)

%% Unroll theta parameter
% We first extract the part which compute the softmax gradient
softmaxTheta = reshape(theta(1:hiddenSize*numClasses), numClasses, hiddenSize);

stack = params2stack(theta(hiddenSize*numClasses+1:end), netconfig);

y = data;
for d = 1:numel(stack)
	y= sigmoid(bsxfun(@plus, stack{d}.w *y, stack{d}.b));
end

y = softmaxTheta * y;

[M, pred] = max(y);
% -----------------------------------------------------------
end
```
## 参数微调 ##
### 微调代价函数 ###
微调代价函数的实现比较复杂而且容易出错，所以最好在程序一开始调用
```matlab
checkStackedAECost();
```
以验证实现的正确性。
**stackedAECost实现**
- Forwardpropagation
计算隐藏层的各层输出结果，并储存到layers{*}.a中
```matlab
% forwardpropagation
layers = cell(numel(stack) + 1, 1);
layers{1}.a =  data;
for i = 1:numel(stack)
	layers{i+1}.a = sigmoid(bsxfun(@plus, stack{i}.w *layers{i}.a, stack{i}.b));
end
% end forwardprogation
```
- Softmax计算
计算结果
```matlab
% softmax
i = numel(stack) + 1;

fx = softmaxTheta * layers{i}.a;
fx = bsxfun(@minus,fx , max(fx ,[],1));

fx = exp(fx);
fx = bsxfun(@rdivide, fx, sum(fx));

softmaxThetaGrad =  - (groundTruth - fx) * layers{i}.a' / M + lambda * softmaxTheta;
```
- backpropagation
由于是多层隐藏层，所以没有将稀疏性参数加入到Cost function里的计算中。这也印证了我之前所说的稀疏性参数公式只适用于单隐藏层的想法。
$\delta^{(l)}$的结果放在layers{i}.d中。而输出层用Softmax进行计算，即
$\delta^{(L)}=-\theta^T(I-P)\cdot f'(z^{(L)})=-\theta^T(I-P)\cdot (a^{(L)}(1-a^{(L)}))$
```matlab
% backpropagation
layers{i}.d =  - softmaxTheta' * (groundTruth - fx).* (layers{i}.a.*(1 - layers{i}.a));

for i = numel(stack):-1:2
	layers{i}.d = (stack{i}.w' * layers{i+1}.d).* (layers{i}.a.*(1 - layers{i}.a));
end

for i = numel(stack):-1:1
	stackgrad{i}.w =  layers{i+1}.d *  layers{i}.a' / M  + stack{i}.w * lambda;
	stackgrad{i}.b = sum(layers{i+1}.d, 2) / M;
end
```
- Cost Function
Softmax's cost function加上各层参数的Regularization
```matlab
% cost
cost = - sum(sum(log(fx).*groundTruth)) / M + lambda * sum(sum(softmaxTheta .^2)) / 2;

for i = 1:numel(stack)
	cost = cost + lambda * sum(sum(stack{i}.w.^2)) / 2;
end
```
实现完成后一定调用checkStackedAECost()来验证实现的正确性。
### 微调训练 ###
注意：初始的stackedAETheta是之前的分层训练结果，而不是随机初始化的值。
```matlab
options.maxIter = maxIter;
[stackedAEOptTheta, cost] = fmincg(@(p) stackedAECost(p, ...
                                    inputSize, hiddenSizeL1, ...
                                   numClasses, netconfig, lambda, ...
                                   trainData, trainLabels), ...
								stackedAETheta , options);
```
### 验证 ###
一切基本测试过后，就可以在整个训练数据集上进行测试了。
我的机器上一共跑了81分钟，才跑完整个训练过程，最后得到的结果：
```matlab
Before Finetuning Test Accuracy: 90.680%
After Finetuning Test Accuracy: 98.260%
```
比起在TensorFlow上几分钟跑完20000次迭代，准确率达到99.2%，确实还有很多改进的空间。
