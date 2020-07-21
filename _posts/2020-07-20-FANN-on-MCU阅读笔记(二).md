---
layout: post
title: 'FANN-on-MCU阅读整理（二）'
date: 2020-07-21
author: Tianmeng Wu
color: rgb(131,175,155)
cover: '../assets/FANN/infiniwolf.png'
tags: ARM RISCV MLP

---

> FANN-on-MCU : An Open-Source Toolkit for Energy-Efficient Neural Network Inference at the Edge of the Internet of Things  
>
> author:Xiaying Wang, *Student Member, IEEE,* Michele Magno, *Senior Member, IEEE,*
>
> Lukas Cavigelli, *Member, IEEE,* Luca Benini, *Fellow, IEEE*

# 四、框架介绍与实验分析

### 1.FANN-on-MCU使用介绍

​        通常，开发一个在板上运行的智能设备通常需要有以下步骤：

 ![]({{ "/assets/FANN/step.png" | absolute_url }})

但FANN-on-MCU框架可以允许开发者通过一行指令直接生成完整的、可以在PULP上执行的、且经过性能优化充分发挥指定硬件处理器优势的、可以直接编译运行的C代码。

这个框架包含一个可以将训练后的网络直接转化成C函数库的脚本，以及对应的性能评估函数和参数列表文件。使用该框架后，开发一个此种类型需要做的工作如下：

1. 使用FANN训练你的模型并保存；

2. 将训练好的模型转换成定点运算（可选，使用FANN_SAVE_TO_FIX）

3. 在模型和数据集上运行FANN-on-MCU框架脚本生成目标平台下的代码

4. 建立你的工程项目并进行性能评估

 同时，当程序在不同的处理器上运行时，数据存储于不同的存储器会对性能有较大影响，文章也提出了对应的存储器预估方法，从而生成最适合于应用的存储器选择方案，其公式如下：

![]({{ "/assets/FANN/formula.png" | absolute_url }})



### 2.实验分析

  作者做了详细的实验对三种不同平台运行MLP的运算性能进行了详细评估。

##### 实验1：

（1）将FANNCoretex生成的未优化代码和FANN-on-MCU生成的代码在相同的ARM Cortex-M4处理器上运行进行对比实验；（2）将FANN-on-MCU生成的代码分别在RISCV单核和多核处理器上进行对比实验。且以上实验均分别在定点运算和浮点运算两个版本间进行对比。其结果如下：

![]({{ "/assets/FANN/e1.png" | absolute_url }})

 可以得出： **（1）在MLP运行过程之中，激活函数的计算过程约占总体运算量的12%，进而可以知道，MLP算法的计算瓶颈在于权重矩阵的相乘部分。**

​                     **（2）定点运算比浮点运算快约15%。**

​                     **（3）无论定点还是浮点运算，8核并行版本可以提供6倍以上的加速。**

这些结果，也与通过最内层循环的汇编指令量预估的运行时间比一致。

![]({{ "/assets/FANN/e11.png" | absolute_url }})



##### 实验2：

​    这部分详细比较了，算法的权重矩阵运算部分，在不同的处理器上计算速度随输入输出量变化的情况。以下a和b分别代表了单层网络在ARM cortexM4 和 pulp IBEX单核（仅使用基本指令）上，随着输出输出量的增大，一些数据不得不放到次一级的存储上，进而出现运算时间的增大，当模型大到一定程度时，模型将因没有足够大的空间存储参数而无法运行。比如对于一个ARM Cortex处理器，如果模型参数总量大于SRAM总量，则超出部分的数据将不得不存放于FLASH Memory中，进而影响I/O的效率。PULP平台上的Mr.Wolf与此类似。

![]({{ "/assets/FANN/e21.png" | absolute_url }})

​       接着作者对比了在PULP  RI5CY 单核（使用PULP拓展指令，主要是硬件循环指令和地址自增的访存指令）相对IBEX 和RI5CY多核相对单核情况下的算法加速比，如下图a和b：

![]({{ "/assets/FANN/e22.png" | absolute_url }})

​       然后比较了PULP  RI5CY 单核相对ARM cortexM4 和RI5CY多核相对ARM cortexM4单核情况下的算法加速比，如下图：

![]({{ "/assets/FANN/e23.png" | absolute_url }})

可以的出如下结论：

​         **（1）单核PULP RI5CY 相对IBEX处理器可以提供2.2倍加速。**

​         **（2）单核PULP RI5CY相对ARM Cortex可以提供2倍加速，多核PULP RI5CY相对ARM Cortex可以提供13.5倍加速。**



##### 实验3：

​      在固定输入输出的情况下，通过变化隐藏层的数量以及隐藏层神经元数量，来评估整个网络的计算性能。通过从极小的数量不断增加到较大的数量，实验结果如下：

![]({{ "/assets/FANN/e31.png" | absolute_url }})

结论：

​       **（1）IBEX核心 和CortexM4的计算速度是接近的。**

​       **（2）由于拓展的指令集，使得一个单核RI5CY比CortexM4快两倍。**

​       **（3）随着隐藏层和隐藏单元数量的增加，多核RI5CY几乎持续地维持在一个加速比增加的状态，但在参数数量超过L1缓存容量时，也能看到一个明显的下降；但总的来说，同样在超过一级缓存容量时，在CortexM4访问Flash Memory 比在RI5CY中访问L2缓存 耗费的时间更多。**







###### 总结：文章提出的FANN-on-MCU为部署在ARM CortexM系列处理器和RISCV PULP处理器在人工智能应用提供性能优秀的自动代码生成框架，且同时支持定点和浮点运算；同时也在两类处理器之间进行了详细的计算性能比较。实验结果显示，多核RISCV 实现版本相对单核有7.1倍的加速比，相对ARM cortex 有13倍的加速比。这篇文章也给出了一些适合于此框架的应用场景，同时给出了其对电量的消耗情况。

 ![]({{ "/assets/FANN/final.png" | absolute_url }})



