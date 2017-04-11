---
title: Ubuntu上同时玩转三种模式的Tensorflow with GPU
categories: 机器学习
tags: [机器学习,tensorflow,Docker]
date: 2017-04-11
toc: true
---
本人Linux\Python\Docker\Tensorflow都是小白……本文记录如何在Ubuntu上同时玩转native 'pip' & virtualenv & docker的tensorflow with GPU，并尝试运行mnist_deep示例来进行验证，从而搭建比较灵活的深度学习开发环境。
<!--more-->
前期的开发环境可以参考我之前的[博客文章][1]
Tensorflow的安装请参考[官方网站][2]
## cuda环境变量 ##
cuda安装完成后，需要编辑.bashrc，加入相关cuda环境变量，否则运行示例时会报错。
```shell
export CUDA_HOME=/usr/local/cuda-8.0
export CUDA_ROOT=/usr/local/cuda-8.0
export PATH=$PATH:$CUDA_ROOT/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CUDA_ROOT/lib:$CUDA_ROOT/lib64:$CUDA_ROOT/extras/CUPTI/lib64
```
## native 'pip' 安装 ##
native 'pip'的tensorflow with GPU的安装和官网上介绍的无异，就不单独介绍了。
不过安装好的tensorflow并没有mnist_deep.py的示例，只有从github上clone下来。
Clone the TensorFlow repository
```
git clone https://github.com/tensorflow/tensorflow
```
然后把在tensorflow/tensorflow/examples/拷贝到系统的tensorflow安装目录中。tensorflow安装目录可以用如下方式查看
```bash
python -c 'import os; import inspect; import tensorflow; print(os.path.dirname(inspect.getfile(tensorflow)))'
```
mnist_deep.py示例在tensorflow/tensorflow/examples/tutorials/mnist/中
运行
```zch
python mnist_deep.py 
```
第一次运行有初始化和下载数据，开始得比较慢。然后就开始运算迭代，风扇狂转。迭代20000次后,就会输出结果0.992.
```zch
I tensorflow/stream_executor/dso_loader.cc:135] successfully opened CUDA library libcublas.so.8.0 locally
I tensorflow/stream_executor/dso_loader.cc:135] successfully opened CUDA library libcudnn.so.5 locally
I tensorflow/stream_executor/dso_loader.cc:135] successfully opened CUDA library libcufft.so.8.0 locally
I tensorflow/stream_executor/dso_loader.cc:135] successfully opened CUDA library libcuda.so.1 locally
I tensorflow/stream_executor/dso_loader.cc:135] successfully opened CUDA library libcurand.so.8.0 locally
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting /tmp/tensorflow/mnist/input_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting /tmp/tensorflow/mnist/input_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting /tmp/tensorflow/mnist/input_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting /tmp/tensorflow/mnist/input_data/t10k-labels-idx1-ubyte.gz
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE3 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.1 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX2 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use FMA instructions, but these are available on your machine and could speed up CPU computations.
I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:910] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
I tensorflow/core/common_runtime/gpu/gpu_device.cc:885] Found device 0 with properties: 
name: GeForce GTX 1050
major: 6 minor: 1 memoryClockRate (GHz) 1.493
pciBusID 0000:01:00.0
Total memory: 1.95GiB
Free memory: 1.45GiB
I tensorflow/core/common_runtime/gpu/gpu_device.cc:906] DMA: 0 
I tensorflow/core/common_runtime/gpu/gpu_device.cc:916] 0:   Y 
I tensorflow/core/common_runtime/gpu/gpu_device.cc:975] Creating TensorFlow device (/gpu:0) -> (device: 0, name: GeForce GTX 1050, pci bus id: 0000:01:00.0)
step 0, training accuracy 0.14
step 100, training accuracy 0.82
step 200, training accuracy 0.96
step 300, training accuracy 0.9
step 400, training accuracy 0.96
step 500, training accuracy 0.9
step 600, training accuracy 1
...
step 19900, training accuracy 1
test accuracy 0.9919
```

我第一次运行出现了GPU OOM的如下错误，我的显卡是GTX 1050，内存2G。
```
W tensorflow/core/common_runtime/bfc_allocator.cc:274] **********************************************************************************xxxxxxxxxxxxxxxxxx
W tensorflow/core/common_runtime/bfc_allocator.cc:275] Ran out of memory trying to allocate 957.03MiB.  See logs for memory state.
W tensorflow/core/framework/op_kernel.cc:993] Resource exhausted: OOM when allocating tensor with shape[10000,32,28,28]
```
原因是示例代码最后对测试集(test)上做评估的时候，测试集数据太大。需要修改mnist_deep.py，来分批评估测试集。
原代码
```python
    print('test accuracy %g' % accuracy.eval(feed_dict={
        x: mnist.test.images, y_: mnist.test.labels, keep_prob: 1.0}))
```
修改后的代码
```python
    batch_size = 50
    batch_num = int(mnist.test.num_examples / batch_size)
    test_accuracy = 0
    for i in range(batch_num):
       batch_tx, batch_ty = mnist.test.next_batch(batch_size);
       test_accuracy += accuracy.eval(feed_dict={x: batch_tx, y_: batch_ty, keep_prob: 1.0})
    test_accuracy /= batch_num;
    print('test accuracy %g' %test_accuracy)
```
## virtualenv 安装 ##
virtualenv安装也与官方网站上介绍的无异，也不单独介绍了。
进入virtualenv环境
```
source ~/tensorflow/bin/activate 
```
virtualenv环境可以访问宿主资源，直接找到之前安装的示例文件
运行
```zch
python mnist_deep.py 
```
退出virtualenv
```
deactivate
```
## Docker 安装 ##
Docker的安装比较麻烦一点儿。
也是按照官网上的步骤一步一步的来，尤其是要安装nvidia-docker
安装好后，测试。测试通过会显示显卡驱动等信息。
```
# Test nvidia-smi
nvidia-docker run --rm nvidia/cuda nvidia-smi
```
选择Docker Container时，安装最新的GPU版本：latest-gpu
Docker是在独立环境中运行的，需要mount本地的examples目录，才能保证能够运行示例。
```
sudo nvidia-docker run -it -v /home/$USER/tensorflow/examples:/home/tensorflow/examples --name tensorflow gcr.io/tensorflow/tensorflow:latest-gpu

```
上面命令中的tensorflow是容器名字，-v后的参数就是mount的本地目录。
然后新开一个终端，用容器名字tensorflow进入容器
```
sudo nvidia-docker exec -it tensorflow bash
```
找到mount的examples目录，运行示例
```
python mnist_deep.py
```

至此，我们就可以只用一个本地代码库来在三种模式下运行调试tensorflow程序了。
  [1]: http://rickdyang.me/2017-03/install-tensorflow/
  [2]: https://www.tensorflow.org/install/install_linux