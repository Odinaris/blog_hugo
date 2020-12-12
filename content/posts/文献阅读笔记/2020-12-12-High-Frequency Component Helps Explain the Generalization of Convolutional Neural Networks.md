+++
title = "阅读笔记 - High-Frequency Component Helps Explain the Generalization of Convolutional Neural Networks"
summary = "本文探究了图像数据频谱和CNN泛化行为之间的关系。本文首次注意到CNN能够捕获人类几乎无法察觉的图像高频分量。由这个观察产生了多种有关CNN泛化行为的猜想，包括对对抗样本的潜在解释，对CNN鲁棒性和准确性之间的讨论和一些理解训练启发的证据。"
authors = ["duyang"]
tags = ["科研","深度学习"]
toc = true
categories = ["科研"]
series = ["深度学习"]
date = "2020-12-12"
lastmod = "2020-12-12"
draft = false

+++


## 论文信息

| 标题                                                         | 作者                                            | 发表于    | 单位                       |
| ------------------------------------------------------------ | ----------------------------------------------- | --------- | -------------------------- |
| High-Frequency Component Helps Explain the Generalization of Convolutional Neural Networks | Haohan Wang, Xindi Wu, Zeyi Huang, Eric P. Xing | CVPR 2020 | Carnegie Mellon University |

## 摘要

本文探究了图像数据频谱和CNN泛化行为之间的关系。本文首次注意到CNN能够捕获人类几乎无法察觉的图像高频分量。由这个观察产生了多种有关CNN泛化行为的猜想，包括对对抗样本的潜在解释，对CNN鲁棒性和准确性之间的讨论和一些理解训练启发的证据。

## 贡献

- 解释了CNN准确率和鲁棒性之间存在的权衡，通过提供CNN如何利用图像高频分量的例子。
- 使用图像频谱作为工具，提供猜想解释CNN的几种泛化行为，尤其是记忆标注混乱的数据的能力。
- 提出了防御方法以帮助提升CNN的对抗鲁棒性以抵御简单的攻击，该方法不需要训练或者微调模型。

## 引言

深度学习的非直觉泛化行为让人感到震惊，例如记忆标记混淆的数据和对于对抗样本的脆弱性。受之前一些论文的启发，本文认为CNN的这种非直觉的泛化行为是人与模型之间的感知差异造成的，即CNN可以以比人可以看到的更高的粒度来查看数据。可以用下图表示这种区别：

<img src="https://i.loli.net/2020/12/12/WknwpqmzGZcXhvY.png" alt="image-20201212152259847" style="zoom:30%;" />

与论文“Adversarial examples are not bugs, they are features”不同的是，本文提出了一个对模型感知的高粒度的解释：CNN可以利用人类无法感知到的高频图像分量信息。下图展示了8张来自于CIFAR10数据集的测试样本，以及基于高频分量和低频分量部分对应的预测结果。在这些例子中，预测输出几乎完全由人眼不可见的图像高频分量决定。另一方面，人类看上去似乎与原始图像一致的低频分量被模型预测成不同的分类。

![image-20201212201651360](https://i.loli.net/2020/12/12/TbEVQDO7uKv4pdj.png)

基于以上经验观察，本文进一步探究了CNN的泛化行为并试图通过对输入的图像频谱作出不同响应来解释这些行为。

## 符号说明

|                Notation                | Expalanation             |
| :------------------------------------: | :----------------------- |
| $\langle\mathbf{x}, \mathbf{y}\rangle$ | 数据样本，图像及对应标签 |
|          $f(\cdot ; \theta)$           | 参数用$\theta$表示的CNN  |
|        $f(\cdot ; \mathcal{H})$        | 人类进行数据分类的结果   |
|            $l(\cdot,\cdot)$            | 损失函数                 |
|         $\alpha(\cdot,\cdot)$          | 预测准确率评价函数       |
|            $d(\cdot,\cdot)$            | 矢量距离评价函数         |
|         $\mathcal{F}(\cdot) $          | 傅里叶变换               |
|       $\mathcal{F}^{-1}(\cdot)$        | 逆傅里叶变换             |

## CNN对高频分量的利用

首先将原始数据分解成低频和高频两部分，用$\mathbf{x}=\{\mathbf{x}_l,\mathbf{x}_h\}$表示，其中$\mathbf{x}_l$表示低频分量（后续用LFC表示），$\mathbf{x}_h$表示高频分量（后续用HFC表示）。然后，得到下面四个公式：

$$\mathbf{z} =\mathcal{F}(\mathbf{x})$$

$$\mathbf{z}_{l},\mathbf{z}_{h}=t(\mathbf{z};r)$$

$$\mathbf{x}_{l}= \mathcal{F}^{-1}\left(\mathbf{z}_{l}\right)$$

$$\mathbf{x}_{h}=\mathcal{F}^{-1}\left(\mathbf{z}_{h}\right)z$$

其中$t(\cdot; r)$定义了一个阈值函数，通过使用一个超参数半径$r$来从$\mathbf{z}$中将低频和高频分量分离开来。

为了正式定义该阈值函数，首先考虑尺寸为$n\times n$，具有$\mathcal{N}$个可能的像素的灰度图像（单通道），即$\mathbf{x}\in \mathcal{N}^{n\times n}$。那么$\mathbf{z}\in \mathcal{C}^{n\times n}$，其中$\mathcal{C}$表示复数。$c_i,c_j$表示重心。然后，阈值函数可以正式定义如下：

$$\begin{array}{l}
\mathbf{z}_{l}(i, j)=\left\{\begin{array}{ll}
\mathbf{z}(i, j), & \text { if } d\left((i, j),\left(c_{i}, c_{j}\right)\right) \leq r \\
0, & \text { otherwise }
\end{array}\right. \\
\mathbf{z}_{h}(i, j)=\left\{\begin{array}{ll}
0, & \text { if } d\left((i, j),\left(c_{i}, c_{j}\right)\right) \leq r \\
\mathbf{z}(i, j), & \text { otherwise }
\end{array}\right.
\end{array}$$

其中距离评价函数采用欧几里得距离。

本文假设人类只能察觉到低频分量，而CNN可以察觉到低频和高频分量两者。即：

$$\mathbf{y}:=f(\mathbf{x} ; \mathcal{H})=f\left(\mathbf{x}_{l} ; \mathcal{H}\right)$$

因为CNN的训练可用下式表示：

$$\underset{\theta}{\arg \min } l(f(\mathbf{x} ; \theta), \mathbf{y})$$

所以等价于：

$$\underset{\theta}{\arg \min } l\left(f\left(\left\{\mathbf{x}_{l}, \mathbf{x}_{h}\right\} ; \theta\right), \mathbf{y}\right)$$

CNN可以学习到利用高频分量来最小化损失。因此，CNN的泛化行为对人类来说可能看起来是非直觉的。

## 总结

写这篇笔记的时候发现目前已有几篇相关阅读笔记，总结得比我深入清晰，与其我继续花时间在这“拾人牙慧”，不如直接贴上这些前辈的阅读笔记好了：

[CNN：我不是你想的那样](https://zhuanlan.zhihu.com/p/315601295)

[CMU团队解析CNN泛化能力：一切秘密都在数据中](https://zhuanlan.zhihu.com/p/248068207)

[（论文解读）High-frequency Component Helps Explain the Generalization of Convolutional Neural Networks](https://blog.csdn.net/qq_43414059/article/details/107423915)







