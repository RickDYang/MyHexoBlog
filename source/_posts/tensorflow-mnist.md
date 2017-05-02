---
title: Learning Tensorflow from MNIST samples
categories: 机器学习
tags: [机器学习,tensorflow]
date: 2017-05-02
mathjax: true
toc: true
---

Tensorflow定义了一套机器学习的框架，其中内建了很多机器学习的基础方法，包括梯度下降方法和代价函数的自动求导，而这些恰恰是实现机器学习时最容易出错的方法。有了这些帮助，使用者可以更关注于模型，而省去很多细节的实现。我从Tensorflow自带的MNIST Samples代码入手说明其使用方法。
更多的细节可以参考[TensoFlow官网][1]。
<!--more-->
## Source Code ##
MNIST Samples在Tensorflow 1.0上的位置是：tensorflow/examples/tutorials/mnist/
一般安装不会下载MNIST Samples代码，只有从Github取源码才能拿到。MNIST Samples共有下面4种实现方法：
- mnist_softmax.py
- mnist_softmax_xla.py
- mnist_with_summaries.py
- mnist_deep.py
我会对每个做相关说明

## 数据准备 ##
mnist samples会从[MNIST][2]上下载数据，默认是下载到tmp目录。而Ubuntu每次重启就会清空temp目录，考虑到国内的网速和避免重复下载，可以自己下载放到固定的路径，并修改code的路径。
注意默认的文件名如下：
```python
  TRAIN_IMAGES = 'train-images-idx3-ubyte.gz'
  TRAIN_LABELS = 'train-labels-idx1-ubyte.gz'
  TEST_IMAGES = 't10k-images-idx3-ubyte.gz'
  TEST_LABELS = 't10k-labels-idx1-ubyte.gz'
```
修改默认下载路径，以mnist_softmax为例
修改'/tmp/tensorflow/mnist/input_data'，可用相对路径
```python
if __name__ == '__main__':
  parser = argparse.ArgumentParser()
  parser.add_argument('--data_dir', type=str, default='/tmp/tensorflow/mnist/input_data',
                      help='Directory for storing input data')
```
而load好的数据会放在
```python
mnist.train.images
mnist.train.labels
mnist.test.images
mnist.test.labels
```
## Softmax Regression ##
mnist_softmax.py实现了Softmax Regression。
### 构造模型 ###
TensorFlow首先是构造机器学习的模型
x为输出，即$784=28\times28$的向量，图形像素数据，定义为placeholder，需要在后面进行绑定
w为权重，即$784\times10$的矩阵，是参数，定义为Variable
b为bias，即$10$的向量
y的计算公式，即$y=x\cdot W +b$
这里y并没有赋任何具体值，而它其实是从x到y的“计算公式”
```python
# Create the model
  x = tf.placeholder(tf.float32, [None, 784])
  W = tf.Variable(tf.zeros([784, 10]))
  b = tf.Variable(tf.zeros([10]))
  y = tf.matmul(x, W) + b
```
### 定义代价函数和梯度求解算法 ###
y_即监督算法的label数据，和x一样需要在后面训练时进行“绑定”，所以定义成placeholder
tf.nn.softmax_cross\_entropy\_with\_logits即定义了代价函数Cross-entropy Loss， $J(\theta)=-\frac{1}{m}\sum\_{i=1}^my\_ilog(h\_i(\theta))$
0.5即Learning Rate
```python
# Define loss and optimizer
  y_ = tf.placeholder(tf.float32, [None, 10])
  
  cross_entropy = tf.reduce_mean(
      tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y))
  train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
```
### 训练 ###
Session是TensorFlow中的一个重要概念，一个机器学习的任务都是在Sesssion中完成的。
用Mini-bactch方式进行训练$100\times1000$
这时通过feed\_dict把x & y\_绑定成训练数据集
```python
sess = tf.InteractiveSession()
  tf.global_variables_initializer().run()
  # Train
  for _ in range(1000):
    batch_xs, batch_ys = mnist.train.next_batch(100)
    sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})
```
### 测试结果 ###
测试也是用Session的方式运行的，之前训练的结果就在Session中。
tf.argmax即最返回最大值的Index值，tf.argmax(y, 1)即预测中概率最大的Index值，y\_为$(0,0,..., 1, ...,0)$形式的向量，tf.argmax(y\_, 1)即返回1的Index值。
feed\_dict这里绑定成测试数据集。
```python
 # Test trained model
  correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
  accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
  print(sess.run(accuracy, feed_dict={x: mnist.test.images,
                                      y_: mnist.test.labels}))
```
至此，你会发现不用实现代价函数及其偏导的计算。这部分工作都由TensorFlow自动帮你完成了！而这部分工作是复杂而容易出错。
## Deep Neural Networks ##
mnist_deep.py实现了一个Deep Neural Networks的模型，用的是CNN(Convolutional Neural Networks)模型。
### 构造Deep Neural Networds Graphic##
依旧是先构造输入和期望输出的placeholder
```python
  # Create the model
  x = tf.placeholder(tf.float32, [None, 784])

  # Define loss and optimizer
  y_ = tf.placeholder(tf.float32, [None, 10])
 ```
模型构建主函数
```python
def deepnn(x):
  """deepnn builds the graph for a deep net for classifying digits.
```
- 构造Convolutional Layers 
参数x_image是输入的placeholder x
Sample模型构造了2层Convolutional Layers
weight_variable是调用tf.truncated_normal来对Weight进行初始化的
bias_variable初始化为固定值0.1
conv2d即卷积算法
$\sigma$函数用的是relu即ReLU (Rectified Linear Unit)
Pooling用的是Max Pooling， h_pool1为下一层的输入
```python
 # First convolutional layer - maps one grayscale image to 32 feature maps.
  W_conv1 = weight_variable([5, 5, 1, 32])
  b_conv1 = bias_variable([32])
  h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)

  # Pooling layer - downsamples by 2X.
  h_pool1 = max_pool_2x2(h_conv1)
  
  # Second convolutional layer -- maps 32 feature maps to 64.
  ...  
```
- 定义输出层
第二CNN层的输出h_pool2作为输入, 而y_conv为网络输出的最终结果
Dropout是优化深度神经网络的一种方法，防止DNN overfitting，具体参考[分析 Dropout][3]。而keep_prob是表示训练中保留神经元的概率。
训练时keep_prob作为参数传入并绑定，所以这里定义成placeholder
```python
  # Fully connected layer 1 -- after 2 round of downsampling, our 28x28 image
  # is down to 7x7x64 feature maps -- maps this to 1024 features.
  W_fc1 = weight_variable([7 * 7 * 64, 1024])
  b_fc1 = bias_variable([1024])

  h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
  h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)

  # Dropout - controls the complexity of the model, prevents co-adaptation of
  # features.
  keep_prob = tf.placeholder(tf.float32)
  h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)

  # Map the 1024 features to 10 classes, one for each digit
  W_fc2 = weight_variable([1024, 10])
  b_fc2 = bias_variable([10])

  y_conv = tf.matmul(h_fc1_drop, W_fc2) + b_fc2
  return y_conv, keep_prob
```
### 定义代价函数和梯度求解算法 ###
1e-4即Learning Rate
```python
cross_entropy = tf.reduce_mean(
      tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y_conv))
  train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
  correct_prediction = tf.equal(tf.argmax(y_conv, 1), tf.argmax(y_, 1))
  accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
```
### 训练 ###
也是用Mini-bactch的方式训练，每次用50个数据做20000次迭代，并输出准确率
这里可以看到Dropout的用法，在训练时是keep_prob: 0.5；而在检验时是keep_prob: 1.0，即不使用Dropout。
使用Dropout可以使测试正确率提高0.2%左右
```python
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    for i in range(20000):
      batch = mnist.train.next_batch(50)
      if i % 100 == 0:
        train_accuracy = accuracy.eval(feed_dict={
            x: batch[0], y_: batch[1], keep_prob: 1.0})
        print('step %d, training accuracy %g' % (i, train_accuracy))
      train_step.run(feed_dict={x: batch[0], y_: batch[1], keep_prob: 0.5})
```
## 其他 ##
### Trace ##
mnist\_softmax\_xla.py中增加了输出trace的功能，输出文件为timeline.ctf.json
。目前还不清楚输出数据的意义。
### Tensor Board ###
mnist\_with\_summaries.py中增加输出Summaries的功能
默认是输出到/tmp/tensorflow/mnist/logs/mnist\_with\_summaries
也可以对其进行修改，支持相对路径
```python
parser.add_argument('--log_dir', type=str, default='/tmp/tensorflow/mnist/logs/mnist_with_summaries',
                      help='Summaries log directory')
```
运行完mnist\_with\_summaries.py后，运行命令行
```bash
tensorboard --logdir=logs/mnist_with_summaries
```
注：logs/mnist_with_summaries是输出目录的绝对或者相对路径
之后用浏览器访问http://127.0.1.1:6006 即可以参看各种图表，帮助分析模型的各种问题。
![Tensor Board][4]
之后有空再具体研究其详细使用方法。

## 总结 ##
TensorFlow很强大，内置了很多强大的库方法，让开发者专注于学习模型的设计。但是作为机器学习的初学者最好还是从基础性的算法学起才能更好的掌握其背后的原理，进而迸发出更多新的想法。


  [1]: https://www.tensorflow.org/get_started/
  [2]: http://yann.lecun.com/exdb/mnist/
  [3]: http://www.jianshu.com/p/ba9ca3b07922
  [4]: /images/tensor-board.png
