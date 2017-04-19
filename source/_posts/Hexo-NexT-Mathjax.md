---
title: Hexo & NexT & Mathjax
categories: 备忘 
tags: Hexo
toc: true
mathjax: true
date: 2017-03-06
---

为了做学习笔记和分享，开始折腾Hexo博客。遇到一些坑，备忘一下。
### Hexo 安装
一Google一大把，不单独介绍了
### Mathjax
Mathjax是一个基于Latex语法的JS库，对于写数学公式非常棒。Coursera的数学公式就是用Mathjax渲染的。
Google & 折腾了半天，本来想自己手动修改添加Mathjax，结果发现NexT theme本来就支持Mathjax，简直又惊又喜。
写Markdown的时候，需要用$$把公式括起来。
<!--more-->
### NexT Theme
NexT theme的安装配置很简单，直接按照[NexT官网][1]的步骤即可。
NexT官网上的介绍很详细，而且是中文的，初入Hexo可以仔细阅读。
#### NexT & Mathjax
Mathjax和Markdown一起工作有一个坑。
下划线"\_" 在Markdown里面是标记_斜体字_，而下划线"\_"在Latex语法中是表示下标，比如"\$x_i\$"表示 $x\_i$ 
Markdown优先于Mathjax，会把下划线渲染成斜体字，导致一些MathJax公式不能正常显示。
Google了一下，有两种方案
 - 对MathJax公式中的下划线转义。但是不能保证公式在各平台的一致性。
 
 - 修改渲染器
 有人做了一个新的渲染器，优先渲染对Latex语法进行渲染。
 安装方式如下
 ```bash
 npm uninstall hexo-renderer-marked --save
 npm install hexo-renderer-markdown-it --save
 ```
 重新hexo clean & generate一下即可
 但是新的hexo-renderer-markdown-it有不少问题，不支持NexT的<\!--more-->, 对TOC支持也有问题。我折腾了半天，最终还是回到了第一种方法。

### 写博客
推荐在线写Markdown的网站[Cmd Markdown编辑阅读器][2]
支持Mathjax公式，但是deploy到Hexo&NexT时有些小问题，需要最终手动调整。
比如Mathjax的下划线转义问题。
写好后，导出成Markdown，在放到hexo/source/_posts即可

### Nice to Have
NexT还有不少第三方的功能集成，比如评论，打赏等等，等有时间继续折腾一下。
但是Nice to Have一般就是Never to Have......

  [1]: http://theme-next.iissnan.com/
  [2]: https://www.zybuluo.com/mdeditor
