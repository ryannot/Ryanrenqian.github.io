---
title: 2020-04-11-DesigningNetworkDesignSpaces

tags: 
        - CVPR2020
        - NAS
        - Paper
---

## NAS：简介

neural architecture search (NAS)： 即一种神经网络搜索框架，旨在帮助设计更为合适的网络结构。 由于近年来，人工设计的网络如VGG，ResNet，GoogleNet，ResNetX等等取得了非常大成功，但同时也由于这些网络设计中存在太多人工的痕迹，比如GooleNet的网络设计。这些经典网络的结构也为神经网络设计提供了一些理论支持，但是在此基础上是否还能够产生新的更好的网络了？答案是肯定的，正如CNN替代了手动设计的滤波器一样，相信NAS也会取代手动设计的神经网络。

## 亮点

## 论文简介：

目标也是新颖点：

通常的NAS 在一系列特定的网络中寻找一个最好的模型，但是这篇文章主要探究通常的网络设计方法，怎么设计网络更好，更高效。

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

   原理说明：

   从一个具体的例子来看，Group conv本身就极大地减少了参数。比如当输入通道为256，输出通道也为256，kernel size为3×3，不做Group conv参数为256×3×3×256。实施分组卷积时，若group为8，每个group的input channel和output channel均为32，参数为8×32×3×3×32，是原来的八分之一。而Group conv最后每一组输出的feature maps应该是以concatenate的方式组合。 

   Alex认为group conv的方式能够增加 filter之间的对角相关性，而且能够减少训练参数，不容易过拟合，这类似于正则的效果。

3. Block，Stage之间的关系：![image-20200411104313381](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwgv0r8uuj30nc0bmmy3.jpg)

4.  错误经验分布函数 EDF：
   ![image-20200427192200983](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8jnuq7dxj30bi026q2w.jpg)

   给定一个model set，错误小于error的model所占据的比例。反映了该模型集的quality。

   ![image-20200427192236516](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8jof09z7j30bw08ujru.jpg)

   

## AnyNet space

结构由三部分组成，简单的stem，跟着是进行主体计算的body，然后是进行预测类别head。为了主要专注于决定模型准确性和计算的body部分的设计，作者使用尽量简单的head和stem。

![image-20200411104924744](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8l3d3sgqj30ls0ay3z7.jpg)



在实验中大部分都是用带有分组卷积（group convolution）的残差bottleneck结构-X block

![image-20200411104838608](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdwgv84hypj30pa0ictba.jpg)

The AnyNetX design space has 16 degrees of freedom as each network consists of 4 stages and each stage i has 4 parameters: the number of blocks $d_i$ , block width $w_i$ , bottleneck ratio $b_i$, and group width $g_i$. 

(16·128·3·6)4 ≈ 1018 possible model configu- rations in the AnyNetX design space.

目标：探索能够帮助我们理解和精炼网络的设计空间的设计原则，而不是搜索最佳的模型。 因此作者需要实现以下目标：

1. 简化设计空间的结构
2. 提高设计空间的可解释性
3. 提高或者维持设计空间的quality
4. 维持设计空间的模型的多样性

那他是怎么做的了：

1. 初始化的无限制的网络空间$AnyNetX_A$

2. $AnyNetX_B$: $AnyNetX_A$ 所有stage共享的bottleneck ratio bi=b的空间。 以相同的settings，采样并从$AnyNetX_B$训练500个模型。 这说明相对于$AnyNetX_A$，共享bottleneck ratio的$AnyNetX_B$并没有造成准确性的下降，因此可以将设计空间从$AnyNetX_A$缩小为$AnyNetX_B$.

   ![image-20200427202446480](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8lh34hxbj30aw08q0tc.jpg) 

3. 同样的，为了进一步减少设计空间，作者探究了$AnyNetX_B$在use a *shared* group width gi = g for all stages对于设计空间quality的影响，即为$AnyNetX_C$,同样的进行EDFs分析，结果如下：
   ![image-20200427203123193](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8lnyu8k0j309408074n.jpg)

   由此可见，设计空间还可以从$AnyNetX_B$缩小到$AnyNetX_C$。此时$AnyNetX_C$相对于$AnyNetX_A$减少了6个自由度，并减少四个数量级的设计空间大小。由于$AnyNetX_C$已经大量减少了设计空间，作者可以检验$AnyNetX_C$中的好的网络和坏的网络的特征。

   ![image-20200427203627141](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8lt9252cj31020i6tcp.jpg)

   如上图，可见好的模型的width随着深度的加深而增加$w_{i+1} ≥ w_i$（上）而差的模型则相反（下）

4. 基于在$AnyNetX_C$上的发现，设计了$w_{i+1} ≥ w_i$的$AnyNetX_D$空间。EDFs分析如下：![image-20200427203940836](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8lwlrhu9j30es0amjsl.jpg)

5. 在上述实验中，作者还发现了另一个有趣的趋势：stage $w_i$除了随着i增加，也会随着stage $d_i$增加，尽管在最后一个stage没有并没有变化，但是还是设计了$d_{i+1} ≥ d_i$的$AnyNetX_E$。结果发现相对于$AnyNetX_D$确实获得了提升。

   ![image-20200427204400150](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8m1sv81nj30ca0a0q3h.jpg)

   此时，加入了$w_i$和$d_i$的限制，使得设计空间降低为4！。而$AnyNetX_A$的复杂度为$O(10^7)$ .

##  RegNet Design Space

为了进一步观察模型结构，作者将$AnyNetX_E$中最好的20个模型可视化出来。如下图：

![image-20200427205318955](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8masjcrgj30bi08gt9b.jpg)

灰色的线表示每一个单独的模型，实黑色的线是$w_j =48·(j+1)for 0 ≤ j ≤ 20$为参考线（可以理解为拟合线）说明宽度的增长趋势。拟合的方式为：

被被称为linear parametrization

![image-20200427212310611](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8n5v8t0yj30vm0amjtg.jpg)

![image-20200427212337939](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8n6c9fzbj30vo0cugnx.jpg)

1. $u_j$ block的$w_j$，等式2表示随着block增加而宽度增加
2. 等式3，4表示每个Block宽度与Stage的关系

总之，宽度随着深度一起增加。

紧接着在$AnyNetX_E$中最好的模型中进行了拟合上的验证。如下图，说明该曲线符合规律分布

![image-20200427212601312](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8n8tf4mwj30su0ca40u.jpg)

然后比较$AnyNetX_E$与$AnyNetX_C$之间的fitting error $e_{fit}$($e_{fit}$表示参数与上述等式的吻合程度，或者说拟合方差) versus network error![image-20200427212856699](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8nbvpzeuj30z409oac6.jpg)

观测到两点：

1. 所有的设计空间中最好模型都符合好的线性拟合
2. $e_{fit}$ 的平均水平从$AnyNetX_C$到$AnyNetX_E$均得到提升，这说明线性参数化的有效性，即$w_i$和$d_i$的关系。

因此，为了验证该关系，作者根据这几个等式从$AnyNetX_E$得到的设计空间$RegNet$。特化6个参数d, $w_0$, $w_a$, $w_m$ (and also b, g)

![image-20200427213839314](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8nlyuex4j30ym09kacn.jpg)

1. 稍微的提升。$w_m ≥ 2$ 效果更好
2. $w_0=w_a$进一步提升，同时简化了$u_j =w_a·(j+1)$
3. 随机搜索效率进一步提升，大概32个左右的随机模型中就能产生好的模型

### Summary of Design Space Summary

![image-20200427214934498](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8nxbmjlrj30x008iac3.jpg)

### generalization比较

![image-20200427215128124](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8nzaw9zhj30zi0patg4.jpg)

通过上图可知，尽管RegNet的参数自由度更小，但是泛化性依然很强能够适应新的设置。

### Analyzing the **RegNetX** Design Space

设置：100 model, 25 epoch, LR = 0.1

#### **RegNet** trend![image-20200427220213650](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8oansl5zj30zo0fa428.jpg)

the *depth of best models is stable across regimes* (top-left), with an optimal depth of ∼20 blocks (60 layers). This is in contrast to the common practice of using deeper models for higher flop regimes.

1. **值得注意到**是这个现象违背经验，即使用更深的模型需要更高的FLOP
2. 最好的模型使用*bottleneck ratio* b *of 1.0* (top-middle)

#### Complexity analysis

由于没有一个固定分析网络的方法，发现activations(激活数)能够严重的影响运行时间（意思是activations反映了运行时间）。对于最好的模型群，activations随着FLOP的平方增加，参数线性增加，

![image-20200427221152375](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge8okj2p59j30z40gwtfj.jpg)



