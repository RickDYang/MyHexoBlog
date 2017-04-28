---
title: What's RNN
categories: 机器学习
tags: [机器学习,深度学习, RNN]
date: 2017-04-27
mathjax: true
toc: true
---
RNNs（RECURRENT NEURAL NETWORKS）是目前我学到的最复杂的模型，其中的LSTM（Long Short Term Memery）更复杂。而Coursera上的[Neural Networks for Machine Learning][1]的相关章节也讲得我云里雾里，等我学完这课再来吐槽。
于是只得在网上找资料进一步学习，其中[RECURRENT NEURAL NETWORKS TUTORIAL][2]讲CNN讲得非常好，其中引用的相关文章也非常值得一读。而且其有[代码实现][3]，对程序员来讲，阅读代码也能从实现角度能更好地理解CNN。
这里也不再重复相关内容，只是简单记录一下要点。
<!--more-->
## 相关阅读材料 ##
- [RECURRENT NEURAL NETWORKS TUTORIAL][4]
RNN详细介绍，非常值得一读
- [Understanding LSTM Networks][5]
图文并茂，对理解LSTM有非常大的帮助

## 总结 ##
- Input is sequence
- Interate in sequence steps
- Intract with previous status
- $U, V, W$ also interpreted as $W\_{xh}, W\_{hh}, W\_{hy}$ are shared accross every iteration. And these parameters need to be **trained**.

### CNN公式 ###
![A recurrent neural network and the unfolding in time of the computation involved in its forward computation. Source: Nature][6]
From [RECURRENT NEURAL NETWORKS TUTORIAL][7]
**计算公式**
$s\_t=\sigma(U\cdot x\_t + W\cdot s\_{t-1})$
$o\_t = softmax(V\cdot s\_t)$ 
其中$\sigma$可以是sigmoid，tanh或者ReLU函数。
**Cost Function**
Cross-entropy Loss
$L(y,o)=-\frac{1}{N}\sum\_{n\in N}y\_nlog(o\_n)$
### LSTM公式 ###
![The repeating module in an LSTM contains four interacting layers.][8]
From [Understanding LSTM Networks][9]
$i=\sigma(x\_t\cdot U^i + s\_{t-1}\cdot W^i),$ —— Input Gate
$f=\sigma(x\_t\cdot U^f + s\_{t-1}\cdot W^f),$ —— Forget Gate
$o=\sigma(x\_t\cdot U^o + s\_{t-1}\cdot W^o),$ —— Ouput Gate
$g=tanh(x\_t\cdot U^g + s\_{t-1}\cdot W^g)$
$c\_t=c_{t-1}\circ f + g \circ i$
$s\_t = tanh(c\_t)\circ o$
其中$U^i, U^f, U^o, U^g$是维度相同的不同参数，都需要**训练**。$W^i,W^f,W^o,W^g$同理。
_梯度怎么求？_
[RECURRENT NEURAL NETWORKS TUTORIAL][10]没有给出明确的计算公式或者算法，而是用Theano的_自动求导_算法。这是下一步需要搞懂的事情。
### GRU公式 ###
$z=\sigma(x\_t\cdot U^z + s\_{t-1}\cdot W^z),$ —— Update Gate
$r=\sigma(x\_t\cdot U^r + s\_{t-1}\cdot W^r),$ —— Reset Gate
$h=tanh(x\_t\cdot U^h + (s\_{t-1}\circ r) W^h)$
$s\_t = (1-z)\circ h + z \circ s\_{t-1}$
## CNN思维导图 ##
![CNN思维导图][11]
## What's Next ##
- CNNs为什么能工作？
对我来说还是magical or mythical……
- Backpropagation Through Time
相关计算公式和算法细节
- LSTM/GRU的梯度
计算公式或者实现算法
- TensorFlow实现
准备用TensorFlow来行实现LSTM/GRU。

To be continued...


  [1]: https://www.coursera.org/learn/neural-networks/home/welcome
  [2]: http://www.wildml.com/2015/09/recurrent-neural-networks-tutorial-part-1-introduction-to-rnns/
  [3]: https://github.com/dennybritz/rnn-tutorial-rnnlm/
  [4]: http://www.wildml.com/2015/09/recurrent-neural-networks-tutorial-part-1-introduction-to-rnns/
  [5]: http://colah.github.io/posts/2015-08-Understanding-LSTMs/
  [6]: http://d3kbpzbmcynnmx.cloudfront.net/wp-content/uploads/2015/09/rnn.jpg
  [7]: http://www.wildml.com/2015/09/recurrent-neural-networks-tutorial-part-1-introduction-to-rnns/
  [8]: http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-chain.png
  [9]: http://colah.github.io/posts/2015-08-Understanding-LSTMs/
  [10]: http://www.wildml.com/2015/09/recurrent-neural-networks-tutorial-part-1-introduction-to-rnns/
  [11]: /images/cnn-mm.PNG
