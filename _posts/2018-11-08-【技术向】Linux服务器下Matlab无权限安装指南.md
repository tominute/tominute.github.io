---
layout:     post
title:      【技术向】Linux服务器下Matlab无权限安装指南
date:       2018-11-08
author:     tominute
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Linux
    - matlab
---

我在网络上看了一些博客，觉得有的介绍有点多余且版本过老不适用，现根据我的经验分享一下安装流程。
# 1.安装前的准备
理论上Linux64位系统各发布版都可以适用，引用块中为linux参考命令可以根据实际情况修改，需要安装JAVA是因为matlab的图形安装指导界面是java写的。
### 1.1原料下载：
MATLAB R2016b Linux64安装包
Matlab 2016b Linux64 Crack破解包
JAVA Linux安装包(当前最新的java8 191版)
所有原料可以在这边下载：
R2016b_glnxa64_dvd1.iso
R2016b_glnxa64_dvd2.iso
Matlab 2016b Linux64 Crack.zip
jre-8u191-linux-x64.tar
[百度云下载地址](https://pan.baidu.com/s/1Z120_zLJzwMMRhzCZF_Vpw)
提取码：o4lz
### 1.2解压
由于iso文件Linux无权限无法解压，需要直接在windows下解压好传到服务器端。
破解包和java安装包直接传到服务器上就好，java安装十分简单，只需解压就可以了，所以解压到你的安装位置。

> unzip Matlab 2016b Linux64 Crack.zip
> tar -zxvf jre-8u191-linux-x64.tar

### 1.3配置Java

> vim ~/.bashrc

然后在其中最后加入如下语句

> export JAVA_HOME=/home/your/path/jre1.8.0_191
> export JAVA_BIN=\$JAVA_HOME/bin
> export JAVA_LIB=\$JAVA_HOME/lib
> export CLASSPATH=.:\$JAVA_LIB  
> export PATH=\$JAVA_BIN:\$PATH 

然后保存退出刷新一下

> source ~/.bashrc


然后看看是否安装成功

> java -version

成功则会出现

> java version "1.8.0_191"
> Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
> Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)

# 2.安装

进入MatlabR2016b文件夹，先给一些文件授予权限

> chmod 777 install
> cd /your/path/Matlab_R2016b/sys/java/jre/glnxa64/jre/bin
> chmod +x java
> cd /your/path/Matlab_R2016b/bin/glnxa64/
> chmod 777 install_unix

然后回到MatlabR2016b目录，主要不要在tmux或者screen中执行，然后直接输入

> ./install

然后会看到弹出的图形界面，如果没有直接finish了那说明前面的有错，在install_unix文件中将最后第966行的eval "\$java_cmd 2> /dev/null"改为eval "\$java_cmd"，然后重新执行install程序可以看到输出的错误。
进入图形界面后，选择使用密钥安装
![图一](https://img-blog.csdn.net/20181019180151503?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![图二](https://img-blog.csdn.net/20181019180200353?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
然后在破解包Matlab 2016b Linux64 Crack文件夹中找到readme文件，复制其中第一个长的密钥，输入
![图三](https://img-blog.csdn.net/20181019180211142?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
然后选择安装位置


![图四](https://img-blog.csdn.net/2018101918022468?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![图五](https://img-blog.csdn.net/20181019180406114?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
最后安装完成
![图六](https://img-blog.csdn.net/20181019181007243?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
然后将crack文件夹中的/R2016b/bin/glnxa64/文件夹下的文件全部拷贝到安装目录对应的地方，如果没有写入权限需要先给权限。

> cd /home/your/install/path/R2016b/bin/
> chmod -R 777 glnxa64/
> cd /home/your/path/MatlabCrack/R2016b/bin/glnxa64/
>  mv * /home/your/install/path/R2016b/bin/glnxa64/

然后设置一下环境变量

> vim ~/.bashrc

最后加入

> export PATH=/home/your/install/path/R2016b/bin:$PATH

退出后刷新一下

> source ~/.bashrc

还剩最后一步了
在任意目录下输入matlab等待图形界面的弹出

> matlab

![图八](https://img-blog.csdn.net/20181019182913927?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
现在需要用破解包中的证书激活
![图八](https://img-blog.csdn.net/20181019183258143?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
然后就完成了~
![图九](https://img-blog.csdn.net/20181019183308268?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
最后检验一下成果，在你的任意目录输入matlab即可，也可以输入无图形界面模式命令如下

> matlab -nodesktop -nosplash

可以看到命令窗口如下显示
![图十](https://img-blog.csdn.net/20181019183844210?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI3MzE4ODgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
证明安装成功了
喜欢就点个赞吧~




