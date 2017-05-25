---
title: 编译优化TensorFlow
mathjax: true
categories: 备忘, TensorFlow
tags: [备忘]
date: 2017-05-25
---
TensorFlow release版没有对特定硬件进行编译优化，如果要优化TensorFlow的运行效率，需要自己编译。
这里记录一下过程以备忘。
<!--more-->
## 前言 ##
如果你运行TensorFlow时，出现下面的Warning，那么表明TensorFlow没有针对你的CPU进行编译。恭喜你有机会可以尝试编译TensorFlow来优化。
```
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE3 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.1 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX2 instructions, but these are available on your machine and could speed up CPU computations.
W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use FMA instructions, but these are available on your machine and could speed up CPU computations.
```
## Installing TensorFlow from Sources ##
按照TensorFlow官网上的[Installing TensorFlow from Sources][1]一步一步来，当然请仔细阅读第一段了解其风险。
另外有几点细节需要特别说明。
如果你之前安装过TensorFlow with GPU，那么就不需要特别的准备了。
注意配置前，需要安装Bazel，按官网[Install Bazel on Ubuntu][2]来
运行./configure进行配置的时候，注意检查一下依赖的路径是否正确。
走到下面这步时，需要到[Nvidia官网][3]上检查一下你的显卡对应的compute capability，我的GTX 1050是6.1
```
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size.
[Default is: "3.5,5.2"]: 6.1
```
### 编译优化 ###
在bazel编译pip packages时，要用下面的命令来针对硬件进行优化
注意其优化项msse4.2 & mfma & mavx2
```
bazel build -c opt --copt=-mavx --copt=-mavx2 --copt=-mfma --copt=-mfpmath=both --copt=-msse4.2 --config=cuda -k //tensorflow/tools/pip_package:build_pip_package
```
漫长的编译完成后，安装编译好的pip package
注意最后的tensorflow-***.whl的名字，每个版本的编译结果会有不同。
```
bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
sudo pip install /tmp/tensorflow_pkg/tensorflow-1.1.0-py2-none-any.whl
```

  [1]: https://www.tensorflow.org/install/install_sources
  [2]: https://bazel.build/versions/master/docs/install-ubuntu.html
  [3]: https://developer.nvidia.com/cuda-gpus