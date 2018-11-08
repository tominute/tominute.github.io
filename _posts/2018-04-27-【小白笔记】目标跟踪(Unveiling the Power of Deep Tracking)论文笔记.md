---
layout:     post
title:      【小白笔记】目标跟踪(Unveiling the Power of Deep Tracking)论文笔记
date:       2018-04-27
author:     tominute
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Tracking
---

# 1.主要贡献
这篇文章18年四月份挂在Arxiv上，现在中了ECCV18，是Martin作为3作的一篇文章，性能比ECO提升了一大截。下面就来说一下这篇文章吧，有不对的地方欢迎一起讨论~ 


----------


  
<br />**贡献1**：该论文探究了深度特征和手工特征分别对目标跟踪的影响，主要分析了不同样本扩增方法和精度/鲁棒性平衡两方面的影响，得出两类特征应该分别处理的结论，深度特征更应该关注于鲁棒性，手工特征更关注精度，使用了样本扩展和调节精度/鲁棒性平衡参数的方法可以显著提高深度特征下的跟踪性能；  
<br />**贡献2**：提出了一种新的跟踪测试结果质量测量方法，结合这种方法计算融合两种特征下响应结果的加权系数，得到最终的响应map。
# 2.主要思路


----------
题目叫揭示深度跟踪器的强大力量，别被题目唬住，其实思想很简单。我们知道深度特征对跟踪效果有很大提升但是限于分辨率低，因此鲁棒性有余而精度不足，反观手工特征，分辨率高，但是特征表达信息不足不能涵盖高层语义信息，所以当目标外观变化较大时经常会跟踪失败，就是鲁棒性不足。那么我们如果能有效的融合这两种特征，在鲁棒性和精度之间取得一个平衡那么性能会得到一定提高。带来两个问题，第一怎么发挥两类特征的最大潜能，第二怎么融合，这就是这篇文章的工作了。

----------

### 2.1怎么发挥深度特征的潜能
----------
对于手工特征的潜能，这些年的工作以及做的差不多了，很难继续挖掘，而对深度特征的潜能，从样本的角度出发的方法并不多，我们知道对于深度学习，大数据很重要，而跟踪任务所给的数据是很少的，只有第一帧的标注信息和每一帧图像，所以样本很少，那么我们可以进行样本扩充，这是第一种方法，第二，判别性跟踪器总有一个高斯形的期望输出map，高斯宽度由参数$\sigma$决定，越宽代表训练时取的正样本越多，这个参数作者称为精度/鲁棒性平衡参数，调节这个参数这也是一种方法。

**样本扩增**：
样本扩增可以提升鲁棒性但是会降低精度这是显然的，所以这个方法到底有没有用作者在深度特征和手工特征下分别做了实验，作者实验了五种扩增方法：翻转，旋转，平移，模糊，dropout。解释下最后一个就是按一定概率随机设置某些通道值为0，为了保证总能量不变，因此剩下的通道值要乘一个扩大的比例系数。
结果如图a，还是和预计一致，该方法会提高深度特征的性能但是会降低手工特征的性能。
![这里写图片描述](//img-blog.csdn.net/20180427194923026?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**精度/鲁棒性平衡参数**：
精度是表示跟踪定位的准确度，鲁棒性是说明跟踪的成功率。而这个高斯宽度参数恰恰是这两者之间平衡的一个参数，这个参数调小说明我们认为的正样本都是很接近目标的，其他都是负样本，强调了精度；这个参数调大说明我们扩大了正样本的区域让模型学习到目标的更多信息，强调了鲁棒性。这个参数对跟踪的影响作者也做了实验，结果如上图b。
作者实验使用的框架是ECO，用的深度特征是resnet50的第四个卷积块的激活层，手工特征是hog和cn。
这两个方法都用上可以使深度特征的性能提升5.8个点。

### 2.2怎么融合
----------
**检测响应图的精度/鲁棒性量化方法**
融合方法很多论文都用过，就是加权求和，不同在于这里的系数是计算出来的。那么怎么计算这里作者提出了一个新的衡量响应值质量(即精度和鲁棒性)的方法。
首先这个方法必须能够同时对精度和鲁棒性进行量化，从响应map看，精度意味着该点的尖锐程度，越尖说明这个点的响应值的精度越高；而鲁棒性意味着这个尖峰处的响应值和离它最近的尖峰响应值之间的差，作者叫margin，margin越小，说明了另一个尖峰的响应值和它越接近，说明鲁棒性越差。
下面这个式子正好可以衡量每一处响应值的质量
![这里写图片描述](//img-blog.csdn.net/20180427194939759?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
其中![这里写图片描述](//img-blog.csdn.net/20180427194950953?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这里t代表了响应map中的某些点坐标，t的取值是有限的，比如取出map中的局部峰值点。分母的取值范围是0~1。
要验证这个质量表达式是否能衡量精度和鲁棒性需要从两个极限方面考虑，第一是t*和t相差很大时，这时表达式可化为
![这里写图片描述](//img-blog.csdn.net/20180427195004922?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
第二是t*和t很接近时，经过推导表达式化为(这里要求map连续且二阶可导)
![这里写图片描述](//img-blog.csdn.net/20180427195017761?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
分子代表了该点的最小曲率，因此可以量化该点的尖锐程度，这两种情况对照精度和鲁棒性在map上的表示含义都可以说明这个表达式是可以同时量化精度和鲁棒性的。
**目标预测**
就像之前说的我们需要根据上一节响应的质量衡量方法来计算加权系数。
![这里写图片描述](//img-blog.csdn.net/20180427195031135?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
就是在一堆可能的t值中找到那个质量最大的t*，建模如下：
![这里写图片描述](//img-blog.csdn.net/20180427195043925?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这里引入一个松弛变量，将模型等价转换为一个标准的二次优化问题然后扔给QP工具包解决就好了。
![这里写图片描述](//img-blog.csdn.net/20180427195056757?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
最后给出论文地址：[论文地址](https://arxiv.org/abs/1804.06833)
