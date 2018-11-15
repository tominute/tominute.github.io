---
layout:     post
title:      【小白笔记】EAST_Learning Policies for Adaptive Tracking with Deep Feature Cascades
date:       2018-09-16
author:     tominute
header-img: img/post-bg-coffee.jpg
catalog: true
tags:
    - Tracking
---



   这是ICCV17的一篇spotlight文章。[论文地址](http://openaccess.thecvf.com/content_ICCV_2017/papers/Huang_Learning_Policies_for_ICCV_2017_paper.pdf)
   该论文主要提出了一种基于CNN特征跟踪的加速方法，用强化学习的思想选择可以判断跟踪结果的特征层，避免了继续的前向操作从而大大节省了时间。有不对的地方欢迎讨论~


  
#1.主要贡献  
EAST的一个主要motivation是在跟踪场景中大部分的跟踪都是简单的跟踪，较难的部分在整个跟踪视频中仅占一小部分，所以作者提出了一个自适应的深度特征流的跟踪方法，baseline是SiamFC，作者使用一个可学习的基于Q学习的agent模块来选择特征流在何层停止，简单的帧可以在较前的层判断完毕不会继续进行前向操作，所以平均的速度会更快，文中报告cpu速度达到了23.2fps，gpu速度达到了158.9fps。
![图一](https://img-blog.csdn.net/20180916182826763?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#2. 基本名词
###2.1特征流
特征流是文章的一个主要名词，很简单，CNN结构就是一个自然的流结果模型，文章使用CNN流的前几层进行一个决策，并且把特征流和强化学习结合起来。
###2.2强化学习
强化学习可以用来进行很好的决策，它主要依据历史判别轨迹和错误进行行动的选择，在检测领域已经有了成熟的应用。文章中使用了Q学习的方法
#3. 基本方法
由于深度特征包含丰富的信息，用于跟踪可以有很好的结果，所以最近很多跟踪算法都使用了深度特征，有点是选择了最后一层全连接层或者最后一层卷积层，有的是把一些卷积层特征组合起来或者分别选择其中表现最好的，但是这些方法速度不够快，因为在简单帧跟踪时根本不需要这么复杂的信息，浅层的特征就足够应付了。所以文章的目的就是找到这个能够很好的跟踪的最浅的层，一个很简答的方法是设置一个阈值，当得到的响应图的最大值小于阈值说明需要继续前向操作，该层的结果还不够有判别力，大于阈值的时候说明可以了，停止前向操作。但这个方法在相应图复杂的时候就不太好了，而且太简单也不好发文章，所以作者使用强化学习思想来学习这个决策的agent。
###3.1SiamFC
SiamFC是EAST算法的baseline，提出127*127的目标模板的z的特征，再提出255*255的搜索域的x的特征，然后进行卷积得到响应图，整个网络可以进行端到端的训练，根据响应图可以得到目标的相对位移。EAST的方法就是在每一层都计算出卷积的响应图，根据响应图得到一个决策，是否停止前向和位置的确定。每一层的响应图都会resize到17*17大小，尺度方面仅使用原始尺度，而不是像SiamFC的搞三个尺度。而EAST的尺度跟踪使用了基于响应图推理的方法，具体就是使用Q学习决策出一个尺度的action。
###3.2基于强化学习的决策模型
如图这个决策agent就是q-net，每一层后都会接一个q-net。这个的训练和设计方法是文章的重点。
![图二](https://img-blog.csdn.net/2018091618284122?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
强化学习的基本模型包含行动A(action)，回报R(reward)和状态S(state),这里的action不仅包含停止，还包含目标框的变形操作，即图中的几种scale操作。action的目标是减少定位的不确定性，R有正负，在agent训练的时候，根据当前决策的目标框和ground truth的交并比得到一个R值，越大R越大，通过最大化R，agent能学习到最好的决策来采取行动，在精度和效率上取得平衡。
action上作者选择了7个针对box的尺度变换和一个终止action，相对位置是根据响应图直接得到的。
state则是一个每层的响应图F和历史行动向量h的二元组，这里的F使用的是当前层和之前层响应图的平均，这是经验上的技巧；这里的h是个8维的向量，因为有8个action，是个one-hot结构，方法中采用了过去的4个action，变成单向量就是32维。
reward根据不同的A来计算如下
![图三](https://img-blog.csdn.net/20180916182849724?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
可以看到这样设计其实是对更深层的搜索进行了惩罚，一旦IOU小了就惩罚。
深度Q学习：
目标是选择一个当前层能得到最高奖励的动作，学习过程通过下式迭代
![图四](https://img-blog.csdn.net/20180916182857883?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
这个Q方程是通过上面那个网络学习的，输入是状态，拉成289维的响应图和32维的历史行动向量，Q网络只有两层128维的全连接层，输出一个8维的action向量，每层都是随机初始化，接了ReLu和dropout。
训练的时候Q网络和前面提特征的网络一起训练的；测试的时候就不接受奖励更新Q方程了。整个网络是在ImageNetVideo上训练的。Q学习部分使用了$\varepsilon$-贪婪的优化方法，即以一定的概率随机采取不同的action。
作者在网络之前使用像素层和多通道的hog层，这两层计算更快，加在特征流里可以使速度更快。
#4.实验
作者进行了全面的消融实验和比较实验，详情这里就不说了，最好的EAST算法在OTB2013上达到了0.638的AUC，OTB100上为0.629，GPU速度为160fps，CPU速度为23fps。

