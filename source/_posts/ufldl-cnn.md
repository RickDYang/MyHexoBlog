---
title:  Stanford UFLDL - Convolution and Pooling
categories: 机器学习
tags: [机器学习,深度学习]
date: 2017-04-18
mathjax: true
toc: true
---
卷积和池化是对大图像数据集进行处理，减少机器学习的计算量，从而有效地进行学习训练。卷积神经网络（Convolution Neural Networks）简称CNN。
这次我们还是从[Stanford UFLDL][1]的课后练习入手，从代码角度说明卷积和池化算法的用法，至于背后的原理等以后搞懂了再分解。
<!--more-->
## 准备 ##
本练习的数据是[STL-10 dataset][2]，其中包含了大量彩色图片的训练和测试数据，大约有100M。
另外需要上次[Linear Decoder][3]练习中训练得到的参数 STL10Features.mat作为输入。
代码中load即可
```matlab
% load parameter from STL10Features.mat into optTheta, ZCAWhite, meanPatch
load('STL10Features.mat', 'optTheta', 'ZCAWhite', 'meanPatch');
```
此外本练习还需要之前实现的softmax regression方法，来做最终的训练。
## 卷积 Convolution ##
练习中实现有卷积算法的测试，当实现完成之后，运行练习，如果出现下列信息，就表示其实现正确。
```
Congratulations! Your convolution code passed the test.
```
**cnnConvolve的实现**
- 首先计算Whiten后的权重和偏移
卷积计算公式为
$f(W(T\cdot(x-\overline{x})) + b)=f(WT\cdot x - WT\cdot\overline{x} + b)$， 其中$T$为Whiten参数，而b为均值mean。
所以我们可以预先计算$W=WT$ & $b= b  - WT\cdot\overline{x}$以加快运算效率
```matlab
WT = W * ZCAWhite;
b = b - WT * meanPatch;
```
- 提取特征值feature
我们这里是用一个$8\times8$的3 channels的patches做卷积，而之前练习得到的W的输入即为一个大小为$8\times8\times3$的patch，所以WT的每一行即表示一个3 channels的patches，我们分别取出每个channel作为卷积的features。
```matlab
offset = patchDim * patchDim * (channel - 1 );
feature = reshape(WT(featureNum, offset + 1 : offset + patchDim * patchDim), patchDim, patchDim);
```
    由于调用conv2时，conv2会将输入的feature翻转，为了保持其正确性，我们需要对其先做反向翻转。
```matlab
feature = flipud(fliplr(squeeze(feature)));
```
- 计算卷积
完整算法如下
```matlab
for imageNum = 1:numImages
  for featureNum = 1:numFeatures

    % convolution of image with feature matrix for each channel
    convolvedImage = zeros(imageDim - patchDim + 1, imageDim - patchDim + 1);
    for channel = 1:3

      % Obtain the feature (patchDim x patchDim) needed during the convolution
	  offset = patchDim * patchDim * (channel - 1 );
      feature = reshape(WT(featureNum, offset + 1 : offset + patchDim * patchDim), patchDim, patchDim); % You should replace this

      % Flip the feature matrix because of the definition of convolution, as explained later
      feature = flipud(fliplr(squeeze(feature)));
      
      % Obtain the image
      im = squeeze(images(:, :, channel, imageNum));

      % Convolve "feature" with "im", adding the result to convolvedImage
      % be sure to do a 'valid' convolution
	  convolvedImage = convolvedImage + conv2(im, feature, 'valid');
      % ------------------------
    end
    
    % Subtract the bias unit (correcting for the mean subtraction as well)
    % Then, apply the sigmoid function to get the hidden activation
    convolvedImage = sigmoid(convolvedImage + b(featureNum)); 
    % ------------------------
    % The convolved feature is the sum of the convolved values for all channels
    convolvedFeatures(featureNum, imageNum, :, :) = convolvedImage;
  end
end
```
    conv2即对image做卷积，注意调用需要加valid参数
```
conv2(im, feature, 'valid');
```
    这里得到convolvedFeatures是一个（numFeatures, numImages, imageDim - patchDim + 1, imageDim - patchDim + 1）大小的数据，即为（4000，m，57， 57）大小的数据集合，最后2维数据即为patch feature卷积后的数据。

## 池化 Pooling ##
对得到的convolvedFeatures做平均池化，以减少数据量。
**cnnpool实现**
```matlab
numPools = floor(convolvedDim / poolDim);

for featureNum = 1:numFeatures
	for imageNum = 1:numImages
		pooledFeature = zeros(floor(convolvedDim / poolDim), floor(convolvedDim / poolDim));
		x = 1;
		for i=1:numPools
			y = 1;
			for j=1:numPools
				pooledFeature(i, j) = mean(mean(convolvedFeatures(featureNum, imageNum, x:x + poolDim - 1, y:y + poolDim - 1)));
				y = y + poolDim;
			end
			x = x + poolDim;
		end
		pooledFeatures(featureNum, imageNum, :, :) = pooledFeature;
	end
end
```
这里得到pooledFeatures是一个（numFeatures, numImages, numPools, numPools）大小的数据，这里为（4000，m，3， 3）大小的数据集合。
pooling实现后会进行简单测试，测试通过后，会输出
```
Congratulations! Your pooling code passed the test.
```
## 串起整个流程 ##
- 卷积和池化处理数据
由于数据较大，为避免内存不足，所以对features进行分批处理。为了简化说明，我去掉了部分log代码和test数据处理（test数据处理和training数据处理流程相同）。处理完成的数据储存在pooledFeaturesTrain中。
```matlab
for convPart = 1:(hiddenSize / stepSize)
    
    featureStart = (convPart - 1) * stepSize + 1;
    featureEnd = convPart * stepSize;
    
    Wt = W(featureStart:featureEnd, :);
    bt = b(featureStart:featureEnd);    
    
    % Convolving and pooling train images
    convolvedFeaturesThis = cnnConvolve(patchDim, stepSize, ...
        trainImages, Wt, bt, ZCAWhite, meanPatch);
    pooledFeaturesThis = cnnPool(poolDim, convolvedFeaturesThis);
    pooledFeaturesTrain(featureStart:featureEnd, :, :, :) = pooledFeaturesThis;   
end
```
- Softmax Regression
用pooledFeaturesTrain和trainLabels数据进行Training
```matlab
softmaxX = permute(pooledFeaturesTrain, [1 3 4 2]);
softmaxX = reshape(softmaxX, numel(pooledFeaturesTrain) / numTrainImages,...
    numTrainImages);
softmaxY = trainLabels;

softmaxModel = softmaxTrain(numel(pooledFeaturesTrain) / numTrainImages,...
    numClasses, softmaxLambda, softmaxX, softmaxY, options);
```
    用pooledFeaturesTest和testLabels数据进行Testing
```matlab
softmaxX = permute(pooledFeaturesTest, [1 3 4 2]);
softmaxX = reshape(softmaxX, numel(pooledFeaturesTest) / numTestImages, numTestImages);
softmaxY = testLabels;

[pred] = softmaxPredict(softmaxModel, softmaxX);
```
- 最后结果
整个完整训练和测试集跑完，总共花了将近140分钟。得到Pooled Features数据存在cnnPooledFeatures.mat中，将近400M。
最后得到准确率为
```
Train Done
Accuracy: 79.969%
```
  [1]: http://ufldl.stanford.edu/wiki/index.php/UFLDL_Tutorial
  [2]: http://ufldl.stanford.edu/wiki/resources/stlSubset.zip
  [3]: http://ufldl.stanford.edu/wiki/index.php/Exercise:Learning_color_features_with_Sparse_Autoencoders