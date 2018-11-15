---
layout:     post
title:      【技术向】Quadprog++安装及使用示例（C++版quadprog函数）
date:       2018-09-07
author:     tominute
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Quadprog
    - C++
---
# 1.简介
quadprog++是Luca Di Gaspero写的C++库，实现了matlab版的quadprog函数大部分功能，quadprog是二次规划的求解函数，功能强大，不了解的可以自行百度，这个C++版本的速度经过实测比matlab快100倍以上，Luca Di Gaspero版权所有（Copyright (C) 2007-2016 Luca Di Gaspero, MIT License）
  
# 2.下载及安装  
### 2.1安装（Linux部分）
作者github上提供了👉[下载地址](https://github.com/liuq/QuadProgpp)，由于提供的是linux安装方法所以我们先在linux下安装，想要在window下使用的同学直接跳到第2.2节。
在linux下依次输入以下命令可完成安装，这里假设没有root权限，cmake时指定了安装位置。

    $ git clone https://github.com/liuq/QuadProgpp.git
	$ cd QuadProgpp
    $ cmake -D CMAKE_INSTALL_PREFIX=/home/yourname/local/quad .
	$ make
    $ make install
    
然后在你安装的位置就可以找到头文件和编译好的文件了，然后就可以在linux下使用了，作者提供了示例文件main.cc，如果想要在windows下使用请看下面。

### 2.2安装（windows部分）
还是先下载好他的原文件然后打开visual studio，我这里是vs15，新建工程然后建立一个测试文件quad.cpp,然后添加源文件Array.cc,QuadProg++.cc和头文件Array.hh,QuadProg++.hh。
![1](/img/20180907/1.jpg)

# 3. 使用示例
可以直接参考作者给的文件，我这里写了一个更简单的文件仅供参考

```
#include "Array.hh"
#include "QuadProg++.hh"
#include <stdio.h>
using namespace quadprogpp;
using namespace std;

int main()
{
	Matrix<double>G(2,2);
	G[0][0] = 0.988;
	G[1][1] = 1.8739;
	G[0][1] = 0;
	G[1][0] = 0;
	Vector<double>g(2);
	g[0] = -0.58;
	g[1] = -0.87;
	Matrix<double>CE(2, 1);
	CE[0][0] = 1;
	CE[1][0] = 1;
	Vector<double>ce(1);
	ce[0] = -1;
	Matrix<double>CI(2, 2);
	CI[0][0] = 1;
	CI[1][1] = 1;
	CI[0][1] = 0;
	CI[1][0] = 0;
	Vector<double>ci(2);
	ci[0] = 0;
	ci[1] = 0;
	Vector<double>x(2);
	clock_t startTime, endTime;
	startTime = clock();
	for (int i = 0; i < 1000; i++)
	{	
		solve_quadprog(G, g, CE, ce, CI, ci, x);
		//cout << x[0] << " " << x[1] << endl;
	}
	endTime = clock();
	double total_time = (double)(endTime - startTime);
	total_time = total_time *1000.0/ CLOCKS_PER_SEC;
	cout << total_time << endl;
	return 0;
}
```

作者提供了两个类似matlab的数据类型，Matrix和Vector用于保存solve_quadprog函数的输入数据，从opencv的mat类型转换到Matrix也很方便，函数形式如下：\\
min 0.5 * x G x + g0 x\\
s.t.\\
    CE^T x + ce0 = 0\\
    CI^T x + ci0 >= 0\\

需要注意矩阵维度需要满足如下条件:\\
G: n * n；\\
g0: n；\\
CE: n * p；\\
ce0: p；\\
CI: n * m；\\
ci0: m；\\
x: n\\
调用的函数为:\\
double solve_quadprog(Matrix<double>& G, Vector<double>& g0, \\
                      const Matrix<double>& CE, const Vector<double>& ce0, \\
                      const Matrix<double>& CI, const Vector<double>& ci0, \\
                      Vector<double>& x);\\
输出即保存在x中。
欢迎讨论~~

