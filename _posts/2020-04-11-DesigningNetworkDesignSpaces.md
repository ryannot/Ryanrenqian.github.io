---
title: 2020-04-11-DesigningNetworkDesignSpaces

tags: 
        - CVPR2020
        - NAS
        - Paper
---

## NAS：简介

neural architecture search (NAS)： 即一种神经网络搜索框架，旨在帮助设计更为合适的网络结构。 由于近年来，人工设计的网络如VGG，ResNet，GoogleNet，ResNetX等等取得了非常大成功，但同时也由于这些网络设计中存在太多人工的痕迹，比如GooleNet的网络设计。这些经典网络的结构也为神经网络设计提供了一些理论支持，但是在此基础上是否还能够产生新的更好的网络了？答案是肯定的，正如CNN替代了手动设计的滤波器一样，相信NAS也会取代手动设计的神经网络。

## 论文简介：

该论文是He Kaiming新发表在CVPR2020上的文章， 分量就不用多说了，让我们一起来看看这篇文章吧。

在NAS领域中，将设计网络的设计空间作为模型结构的参数集，也就是说不同的模型参数代表着不同的模型结构。寻找合适模型参数才是优化的方向。举一个栗子，初始化的参数空间为A，通过NAS优化，不断寻找其中更好的空间区域如从A->B->C。分布错误（右图）因此获得了提升, 模型因此能够获得更强劲的性能（robust and generalize）。

![image-20200411092822243](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwgu76h6bj30iu07kdh0.jpg)

通常NAS都专注于设计一个新的独立的网络，通过给定一系列可能的网络结构搜索空进中进行搜索，NAS能够在搜索空间中找到一个合适的模型。但是这样做是由局限性的，他的结果是将一个简单的网络调整到一个特别的setting，但不能加深我们对网络设计原则的理解，特别是设计一些容易理解的，泛化模型。

作者在这里结合手动设计（设计原则）的模型和NAS（优化结构）的长处，构建了一个能够参数化一系列网络的设计空间。

## 知识点复习

1. bottleneck 结构
   ![image-20200411103716240](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwguaf35tj30is0ge75k.jpg)

   ![image-20200411103746812](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwguqhjwbj30p60d6gn6.jpg)

2. group conv：普通的卷积过程如下

   ![image-20200411103928833](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwgut5eouj30z80g0jwd.jpg)
   下图中将输入数据分成了2组（组数为g），需要注意的是，这种分组只是在深度上进行划分，即某几个通道编为一组，这个具体的数量由（C1/g）决定。因为输出数据的改变，相应的，卷积核也需要做出同样的改变。即每组中卷积核的深度也就变成了（C1/g），而卷积核的大小是不需要改变的，此时每组的卷积核的个数就变成了（C2/g）个，而不是原来的C2了。然后用每组的卷积核同它们对应组内的输入数据卷积，得到了输出数据以后，再用concatenate的方式组合起来，最终的输出数据的通道仍旧是C2。也就是说，分组数g决定以后，那么我们将并行的运算g个相同的卷积过程，每个过程里（每组），输入数据为H1×W1×C1/g，卷积核大小为h1×w1×C1/g，一共有C2/g个，输出数据为H2×W2×C2/g。![image-20200411104140070](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwguwyn17j30zw0dowj7.jpg)

3. Block，Stage之间的关系：![image-20200411104313381](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwgv0r8uuj30nc0bmmy3.jpg)

## AnyNet space

结构由三部分组成，stem，body，head。

The AnyNetX design space has 16 degrees of freedom as each network consists of 4 stages and each stage i has 4 parameters: the number of blocks di , block width wi , bottleneck ratio bi, and group width gi. 

![image-20200411104924744](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwgv3wzf8j30ls0ayjsa.jpg)

![image-20200411104838608](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwgv84hypj30pa0ictba.jpg)

