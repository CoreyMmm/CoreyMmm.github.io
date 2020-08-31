---
layout: post
title: 'TEMPONet sEmg Recognization阅读整理（二）'
date: 2020-08-20
author: Tianmeng Wu
color: rgb(131,175,155)
cover: '../assets/TEMPONet/图片13.png'
tags: EMG GAP8 RISCV TCN


---

> Robust Real-Time Embedded EMG Recognition Framework Using Temporal Convolutional Networks on a Multicore IoT Processor
>
> author:Marcello Zanghieri, Simone Benatti , Alessio Burrello , Victor Kartsch , Francesco Conti , and Luca Benini

# EXPERIMENTAL RESULTS

### A. 模型性能试验

​      首先作者介绍了一种增量训练集并进行实验的方法，来验证经过不同数量数据集训练后的模型在inter-session数据集上的准确率表现，并与SVM做对比实验。

![]({{ "/assets/TEMPONet/图片5.png" | absolute_url }})

![]({{ "/assets/TEMPONet/图片6.png" | absolute_url }})

上图可以看出，随着训练量的增加，inter-session准确率下降的幅度越来越小，但本文提出的TCN效果仍然要优于SVM，具体量是4.8%。

![]({{ "/assets/TEMPONet/图片7.png" | absolute_url }})

但作者又提出，判断错误的样本都集中在瞬时动作上，而瞬时的动作可以通过隐马尔科夫模型从数据集上去除，因此又给出了模型在去除瞬时动作数据的数据集上的表现。

![]({{ "/assets/TEMPONet/图片8.png" | absolute_url }})

训练集为Nina DB6 1-5session,去除瞬时动作。

![]({{ "/assets/TEMPONet/图片9.png" | absolute_url }})

训练集为作者采集的20个数据集中的1-10session，去除瞬时动作。

此外，本文提出的模型更加地具有稳定性（smooth?).

![]({{ "/assets/TEMPONet/图片10.png" | absolute_url }})

​    总的来说，文章提出的TCN是比SVM好的,无论是在Nina DB6还是在作者提供的20个数据集上，都有更高的准确率和更稳定的输出。

   此外，模型在作者给出的数据集上训练可以取得更高的准确率，这是由于Nina DB6只有抓握的动作，不具有足够的多样性。





### B. 嵌入式平台部署性能试验

为了便于实验比较，作者也将SVM进行了量化，并与量化后的TCN进行比较，结果如下。

![]({{ "/assets/TEMPONet/图片11.png" | absolute_url }})

TCN在量化后准确率下降度低于SVM，特别是在20数据集上的inter-sessioin情况，并且需要的内存量更低。

  最后作者测试了整个系统的功耗、能量耗费等其他数据，均比其他的平台实现要好。

![]({{ "/assets/TEMPONet/图片12.png" | absolute_url }})



