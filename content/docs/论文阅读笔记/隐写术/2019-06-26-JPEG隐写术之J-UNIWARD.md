+++
title = "阅读笔记 - Universal distortion function for steganography in an arbitrary domain"
summary = "本文对JPEG经典隐写术J-UNIWARD进行介绍，首先介绍方向滤波器组，然后解释失真函数的设计思想，最后介绍UNIWARD的加性近似过程。"
authors = ["duyang"]
tags = ["科研","JPEG"]
toc = true
categories = ["科研"]
series = ["JPEG隐写术"]
date = "2019-06-26"
lastmod = "2019-06-26"
draft = false
+++


## 论文信息

| Title                                                        | Authors                                             |
| ------------------------------------------------------------ | --------------------------------------------------- |
| Universal distortion function for steganography in an arbitrary domain | Vojtˇech Holub, Jessica Fridrich and Tomáš Denemark |
| Digital Image Steganography Using Universal Distortion       | Vojtˇech Holub and Jessica Fridrich                 |

## 摘要

近几年来，最成功的隐写术是在嵌入秘密数据的同时最小化一个适合定义的失真函数。而由于一些高效实用的编码存在（能够达到逼近率失真边界的嵌入效果），对于从事隐写术的科研工作者来说，本质上唯一剩下的任务就是设计失真函数。

这篇paper中提出了一种通用失真设计方案，UNIWARD（UNIversal WAvelet Relative Distortion，通用小波相对失真）。UNIWARD可以适用任何域，如空域（S-UNIWARD），JPEG域（J-UNIWARD）还有基于边信息的JPEG域（SI-UNIWARD）。

最近的隐写分析技术，如RM（Rich Model，富模型）可以通过使用局部多项式模型来很好地拟合clean edges上的改变，因而可以有效检测出clean edges上的改变。因此嵌入算法需要能够将数据嵌入在纹理或噪声区域，这些区域从任何方向上都不容易被建模，因此难以被检测。

UNIWARD的嵌入失真是**载体图像的方向滤波器组分解中的系数的相对变化总和**。这种方向性使得嵌入变化区域多集中在那些难以建模在多个方向的区域，如纹理和噪声区域，而避免了光滑区域或者clean edges被修改。

## 符号解释

| Notation        | Expalanation                               |
| --------------- | ------------------------------------------ |
| $$\pmb{X}$$     | cover的量化后DCT系数矩阵                   |
| **$$\pmb{Y}$$** | stego的量化后DCT系数矩阵                   |
| $$X_{ij}$$      | cover中第$i$行第$j$列的量化后DCT系数值 |
| $$Y_{ij}$$      | stego中第$i$行第$j$列的量化后DCT系数值 |
| $$n_1$$         | 行数                                       |
| $$n_2$$         | 列数                                       |


## 方向滤波器组（Directional filter bank）

方向滤波器组包括三个线性平移不变滤波器，他们的核用$\pmb{B}=\{\pmb{K}^{(1)},\pmb{K}^{(2)},\pmb{K}^{(3)}\}$表示。

这三个滤波器通过计算方向残差$\pmb{W}^{(k)}=\pmb{K}^{(k)}\star \pmb{X}$来分别从水平、垂直和对角线方向来评估给定图像$\pmb{X}$的光滑程度，其中$'\star'$表示镜像填充（mirror-padded）卷积操作使得方向残差$\pmb{W}^{(k)}$的尺寸与cover的量化后DCT系数矩阵尺寸一致，都是$n_1\times n_2$。**镜像填充的作用是防止在边界处引入embedding artifacts。**

滤波器组的选择是任意的，而在这里作者根据比对各种滤波器组之后，最终选择了从一维低通（和高通）小波分解滤波器来构建滤波器组的核：

$$\pmb{K}^{1}=\pmb{h}\cdot\pmb{g}^T,\pmb{K}^{2}=\pmb{g}\cdot\pmb{h}^T,\pmb{K}^{3}=\pmb{g}\cdot\pmb{g}^T.$$

这种情况下的滤波器分别对应于二维的LH，HL和HH小波方向高通滤波器，并且残差与$\pmb{X}$的第一级未抽取小波LH，HL和HH方向分解一致。**滤波器只限于小波滤波器组，是因为小波表示是已知的能够为自然场景提供良好的去相关性（decorrelation）和能量紧凑化（energy compactification）**。


## 失真函数（Distortion function）

对于JPEG图像来说，计算cover和stego之间的失真，首先要将JPEG文件解压到空域，即此时的$\pmb{X}$和$\pmb{Y}$是对应的空域像素矩阵，而非DCT系数矩阵，失真可表示如下：
$$\mathrm{D}(\pmb{X,Y})=\mathrm{D}(J^{-1}(\pmb{X}),J^{-1}(\pmb{Y}))$$

$$\mathrm{D}(\pmb{X,Y})=\sum_{k=1}^3 \sum_{u,v} \frac{|W_{uv}^{k}(X)-W_{uv}^{k}(Y)|}{\epsilon+|W_{uv}^{k}(X)|}$$

其中$\epsilon$是一个正数标量，用于避免分母为0。从上式看出，分母越大，最终失真也就越小。也就是在cover矩阵中，更大的小波系数发生修改造成更小的失真，而这一般是发生在纹理/噪声区域以及图像边缘部分。而另一方面，如果至少一个小的小波系数发生了较大改变，都会造成很大的失真。因此，在至少一个方向上，上式的设计不鼓励嵌入修改发生在内容相对平滑（平滑也更容易被建模）的区域。


## UNIWARD的加性近似

### 为什么要使用加性近似？

在JPEG域中，当修改一个JPEG系数时，会同时影响一整个$8\times8$的像素块，因此有$23\times 23$大小的小波系数也会受到影响。因此当修改多个邻近的像素或者DCT系数时，造成的嵌入修改发生重叠，相互作用。因此失真$\mathrm{D}$是**非加性**的。

虽然存在一些使用非加性失真函数的嵌入方法，如Gibbs构造，但使用加性失真更容易进行嵌入。并且在UNIWARD中，相邻的嵌入修改造成的影响过于明显使得Gibbs构造的嵌入效果差强人意。UNIWARD不采用Gibbs构造的进一步解释可参考论文，这里不作过多赘述。

使用加性近似的显著优点是在全局设计上的便利性。因为嵌入过程可以直接通过使用隐写术中的一个标准工具 - STC码来实现。

### 如何定义加性近似？

任何失真函数$\mathrm{D}(\pmb{X,Y})$都可以用来嵌入到加性近似中，通过使用D来计算修改每个像素值或DCT系数$X_{i,j}$的成本$\rho_{ij}$。

将$X_{i,j}$修改成$Y_{i,j}$并保持其他系数值不变的代价定义如下：

$$\rho(\pmb{X},Y_{ij})=D(\pmb{X},\pmb{X}_{\sim ij}Y_{ij})$$

其中$\pmb{X}_{\sim ij}Y_{ij}$是只有第$ij$个元素发生改变的$\pmb{X}$。当$\pmb{X}=\pmb{Y}$时，$\rho=0$。

用$D_A(\pmb{X,Y})$来表示加性近似：

$$D_A(\pmb{X,Y} )=\sum_{i=1}^{n_1} \sum_{j=1}^{n_2} \rho_{ij} (\pmb{X}, Y_{ij} ) [X_{ij}\neq Y_{ij}]$$

其中$[S]$用来表示艾弗森括号，当括号中的状态S为真时，等于1，否则为0。

由于失真函数$\mathrm{D}(\pmb{X,Y})$中的绝对值存在，表明

$$\rho_{ij}(\pmb{X},X_{ij}-1)=\rho_{ij}(\pmb{X},X_{ij}+1)$$

这就使得UNIWARD可以使用三元嵌入操作，进一步提高嵌入效率。具体的嵌入算法构造可以使用STC码的三元多层版本。