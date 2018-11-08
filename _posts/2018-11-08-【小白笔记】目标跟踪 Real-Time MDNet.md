---
layout:     post
title:      【小白笔记】目标跟踪 Real-Time MDNet
date:       2018-11-08
author:     tominute
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Tracking
    - MDNet
---
这是ECCV18的一篇文章。[论文地址](http://openaccess.thecvf.com/content_ECCV_2018/papers/Ilchae_Jung_Real-Time_MDNet_ECCV_2018_paper.pdf)。
   该论文主要提出了一种改进的MDNet，通过引入Mask-RCNN的ROIAlign缩减网络大小从而达到实时的目的，有不对的地方欢迎讨论~
# 1.主要贡献 
贡献一：利用ROIAlign的方法可以提取更加精确的位置的特征从而提高定位精度；
贡献二：损失项中加入多任务损失使得模型能更有效判别不同域的视频的目标。
# 2.知识准备
### 2.1 MDNet
MDNet是一个纯深度的目标跟踪方法，训练时首先在每一个视频中根据目标的位置用高斯分布，均匀分布和随机分布结合的方法采样取得ROI框，提取对应图像patch然后输入网络最后一层全连接层后softmax输出2个值分别为目标和背景的概率，然后根据预先计算的groundtruth计算loss反传，所谓MD即是训练时仅最后一层FC层根据不同类的视频(可以直接每一个视频是一类)二不同，即仅有前面的层共享参数，目的是学习到更鲁棒的参数，检测的时候去掉最后一层用新的FC层使用第一帧的信息finetune，此外使用了一些样本选择的策略提高性能。
### 2.2 ROIAlign
这是ROIpooling的改进版，在Mask-RCNN中提出，首先说ROIpooling是fast-RCNN中为了减少计算量使用的方法，根据提取多个ROI的特征然后pooling到固定的size，然后在输入到后续网络中，而ROIAlign是为了缓解ROIpooling的量化损失，通过双线性插值获得更加精确的位置的目标特征。
# 3.改进
### 3.1 网络结构
![图一](https://img-blog.csdnimg.cn/20181102153336293.JPG)
前面三层卷积层用于提取图像的特征，中间的ROIAlign作者进行了一些修改后面再说，用于提取各个ROI的特征，这个操作降低了特征维度同时保证了精度，后面又接了三个全连接层用于二分类。
### 3.2 自适应的ROIAlign
由于卷积中间的pooling层会带来分辨率的损失，从而影响定位精度所以作者去掉了第二层卷积层后的pooling，直接进行第三层的空洞卷积，从而可以获得两倍于原始网络的特征大小。
由于原来的ROIAlign仅仅使用了邻近的格点来计算双线性插值，这样对大目标会有部分信息损失，所以作者自适应的调整了格点的间隔，即双线性插值的带宽由ROI的size决定，和w/w'成比例，w为第三层卷积后ROI区域的宽度，w'为ROIAlign后的宽度。后面输出了7*7的特征图，再接一个pooling变成3*3的大小。根据作者的说法虽然这个改进很小但是实际提升明显，算是个trick了，作者的解释是跟踪不同于检测，小的误差会逐渐积累导致性能的明显下降。
![图二](https://img-blog.csdnimg.cn/20181102153347314.JPG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx,size_16,color_FFFFFF,t_70)
### 3.3 内嵌判别性实例的预训练
小标题说的挺拗口其实就是改了一下loss，引入了内嵌实例的loss，即使得不同域的目标在特征空间的距离相互更远，这样能学到更有判别力的特征。MDNet仅是在每一个域中区分目标和背景，而当目标们有相似的外观时就不能有效判别不同域中的目标，所以作者loss中嵌入了其他视频中的目标来使相互之间更有判别力。原始的最后FC层输出的特征2D维(前景加背景，D是域的个数)送给了softmax进行二分类，输出目标和背景的概率，此外。2D维特征还要送到另一个softmax来输出多域中的各个目标的概率，即
![图三](https://img-blog.csdnimg.cn/20181102153357381.JPG)
所以网络的目标loss即这个多任务损失，形式如下，前面一项是原来的loss，后面一个是新加的项，目的很显然，把当前目标和其他目标区分开来，加权进行平衡。
![图四](https://img-blog.csdnimg.cn/20181102153404548.JPG)
![图五](https://img-blog.csdnimg.cn/20181102153413451.JPG)
![图六](https://img-blog.csdnimg.cn/20181102153422484.JPG)
图形解释如下
![图七](https://img-blog.csdnimg.cn/20181102153430666.JPG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx,size_16,color_FFFFFF,t_70)
### 3.4 在线跟踪
和MDNet没有什么区别，预训练完去掉多域的D支最后的FC层，仅保留一支FC层用视频的第一帧finetune，3层FC层，卷积层不动，ROI从上一帧目标周围按高斯分布采集，输入网络，目标概率最大的即为跟踪目标，最后用边框回归提升精度方法和MDNet一样。模型更新也和MDNet一样就不说了。
# 4. 实验
貌似没有使用MDNet的预训练集，而是用imageNet-VID上的视频进行训练。速度可以达到50fps左右，但是性能稍微下降了，尤其是overlap大于80%的部分是不如其他对比算法的，作者猜测这是CNN方法和ROIpooling的限制，速度上去了导致定位精度还是不足。但是注意到原始MDNet加上新的损失项可以提高性能1%左右，即MDNet+IEL。OTB100的结果如下：
![图八](https://img-blog.csdnimg.cn/20181102153438710.JPG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx,size_16,color_FFFFFF,t_70)
