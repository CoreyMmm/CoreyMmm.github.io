---
layout: post
title: 'FANN-on-MCU阅读整理（一）'
date: 2020-07-20
author: Tianmeng Wu
color: rgb(131,175,155)
cover: '../assets/FANN/infiniwolf.png'
tags: ARM RISCV MLP

---

> FANN-on-MCU : An Open-Source Toolkit for Energy-Efficient Neural Network Inference at the Edge of the Internet of Things  
>
> author:Xiaying Wang, *Student Member, IEEE,* Michele Magno, *Senior Member, IEEE,*Lukas Cavigelli, *Member, IEEE,* Luca Benini, *Fellow, IEEE*
>

# 一、摘要和研究背景

​       随着边缘计算概念的兴起，越来越多的低功耗智能设备进入了研究人员的视线，也促使着许多机器学习算法转向了在资源有限的边缘设备上运行。这篇文章研发了一个名为FANN-ON-MCU的开源框架。这个框架主要面向轻量级、低耗能人工智能算法，专为基于RISCV芯片的PULP平台和ARM架构CortexM系列芯片而设计。目前，该框架可以自动训练多层感知机，并自动生成基于RISCV芯片或CortexM系列微处理器的高性能代码。此外，这篇文章详细分析了网络的大小和计算核心对能量效率的影响，也分析了主存访问效率和算法的并行粒度对算法加速的影响。在最后的实验结果中，基于8核心RISCV芯片的算法实现版本比单核的CortexM4实现,取得了22倍的加速比和69%的能耗降低比。

​       在目前的工业界中，基于ARM架构的CortexM系列芯片（比如Cortex-M7）占据了主要的单片机市场，而RISCV则代表着新一代的开源多核微处理器。RISCV多核芯片（例如文中使用的Mr.Wolf和GAP8）对精简指令集的ISA做了许多指令执行上的优化，有着低功耗和并行的特点。而该文章提出的框架，可以自动生成和部署针对两种芯片的MLP高性能代码，同时支持浮点运算和定点运算。

​       总的来说，这篇文章的contribution如下：

1. 发布了FANN-ON-MCU的开源框架。

2. 在内存大小的限制下，对该框架生成代码性能在RISCV芯片和ARM Cortex系列芯片条件下做了详细的对比。

3. 在嵌入式系统的多级存储结构下，展示了模型大小对处理器计算性能的影响。

4. 其他性能（加速和功耗方面）和代码实现上的贡献。

# 二、相关工作

###   	1.MLP和FANN

​        该文章使用的实验算法是多层感知机算法（MLP）。MLP是一个相对不复杂，又较为有效的算法，只包括输入层，输出层和若干隐藏层，参数也只有每个神经元的权重、偏差以及隐藏层的数量，而这些参数在真实应用中，往往通过端到端的训练取得，在计算时加载到内存中加入计算。从真实应用的角度来看，MLP相对于卷积神经网络等其他算法，对内存和算力的要求低很多，因此在医学数据处理或在一些对算法可解释性要求较高并经常人工提取特征的应用场景中有突出的效果。

![]({{ "/assets/FANN/mlp.png" | absolute_url }})

​      而FANN是一个简单易用，拥有图形化界面，自动训练并同时支持自动调整超参数的开源库，但不支持GPU训练，因为对于部署在嵌入式芯片上的应用来说，不可能拥有着海量的超参数（比如bert），但FANN的CPU实现也都是经过针对cache等的优化，能够快速运行。

### 	2.FANN-on-MCU

​       不同于我们耳熟能详的tensorflow或者caffee2这些面向云尺度部署应用的框架，该文章提出的框架是一种针对部署在单片机（MCU）应用的人工智能框架，在FANN提供的自动化模型训练的基础上，提供一种自动生成可直接编译运行于ARM Cortex系列芯片和基于RISCV的PULP平台（Paralle Ultra-Low-Power）的代码的解决方案。

​       在并行方面，ARM已经开发出了CMSIS-NN来提供并行优化加速。CMSIS全称为Cortex Microcontroller Software Interface Standard ，中文Cortex微处理器软件接口标准，而CMSIS-NN是一种为处理器降低神经网络计算压力的标准，优化神经网络计算速度，比如将某些浮点运算转化为定点运算，通过查表而不是直接计算来获取激活函数的输出结果。而本文框架针对ARM 芯片的工作正是基于CMSIS-NN而做的。而在基于RISCV处理器的PULP平台上，也有专用的拓展指令集，充分发挥其硬件优势。



# 三、低功耗处理器介绍

​       作者接下来介绍了后续实验中使用的三种平台和芯片——市场占有率最大的ARM Cortex M系列、新一代开源RISCV由GreenWave生产的Mr.Wolf、以及文中实现的用于测试的InfiniWolf。

### 	1.ARM Cortex M Series

  在ARM Cortex处理器部署应用有以下需要注意的硬件特性：

​    1.浮点运算。只有CortexM4F和M7支持浮点运算。

​    2.存储问题。其存储器由256～512KB左右的SRAM和1～2MB的非易失性闪存组成。其中的闪存一般用于预加载数据，而SRAM直接用于运行时的数据存储，因此模型大小、参数量是在此类处理器上运行模型必须考虑的问题。

   3.CMSIS。使用ARM公司推出的这个软件接口，可以加快模型的运行速度。这个接口提供了对某些指令的加速操作，或涉及DSP的特殊指令。

### 	2.基于RISCV的PULP平台（Mr.Wolf处理器）

![]({{ "/assets/FANN/wolf.png" | absolute_url }})

​       如图，Mr.wolf主要包括一个主处理器和一个8核协处理器，主处理器用于常规指令和运算，并可以将计算密度大的运算转交给协处理器并行处理。此外，包括两级由SRAM构成的缓存。L2分成shared memory和private memory，类似于一些intel 处理器中将cache分成数据cache和指令cache，以减少指令周期中的访存冲突；L1 主要用于暂存并行计算过程中的中间结果。两级缓存通过DMA交换数据。

 ![]({{ "/assets/FANN/instruction.png" | absolute_url }})

​      协处理器可以利用pulp extension 指令集中的拓展指令大幅度提升程序的运行速度。一个for循环指令的例子如上图：在采用精简指令集架构的处理器中，每一条指令都在一个时钟周期内完成，因此在最左边的代码中，循环执行一遍需要11个时钟周期，而使用扩展指令集中的带地址自增的访存指令，可以减少原指令中循环变量自增的运算过程，可以使循环时间缩短为8个时钟周期；在此基础上，使用硬件循环指令，省去变量运算和比较过程，进一步缩短到5个时钟周期；前面的优化都是使用lb指令一次加载一个字节的数据，若使用lw一次加载一个字，同时利用并行运算指令，则最终可以达到平均1.25个时钟周期输出一个结果。

### 	3.Infiniwolf

![]({{ "/assets/FANN/infiniwolf.png" | absolute_url }})

​         infiniwolf为文章用作实际测试的实验平台，主要包括RISCV的Mr.Wolf处理器和一个IBEX处理器以及传感器和相应的电源管理芯片。IBEX实现了基本的RV32IMC 精简指令集；而Mr.Wolf中包含有8个RI5CY核心，这些核心实现了拓展指令集，包括硬件循环、地址自增的访存指令、并行计算指令以及调用DSP进行计算的指令。



