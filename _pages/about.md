---
permalink: /
title: "自我介绍"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

我叫曹泽平，本科就读于四川大学环境工程专业；研究生就读于南开大学资源与环境专业。我勤于思考，喜欢探索数据的规律。硕士期间完成多项科研工作，包括：

## 1. 机动车地方测试工况构建

作为国际通行的车辆能耗与排放评估体系，统一汽车测试工况（如国际通用的 WLTC、中国采用的 CLTC 等）是规范化的标准测试程序，其主要功能是通过模拟不同驾驶场景下的车辆运行状态，对机动车的燃油/电能消耗率、尾气排放量等核心指标进行科学测定与认证。

在国家、国际层级上，当前的统一工况起着举足轻重的作用。然而，当研究视角切换到地区尺度时，统一工况便难以反映不同地区的差异。因此，需要构建足以反映某一地区车辆运动学特征的速度-时间曲线，即地方测试工况。

基于上述背景，完成了两种驾驶循环的构建方法：

### （1）马尔科夫-贪婪算法构建法

该方法通过构建马尔科夫链模拟车辆的速度转移特性，并结合贪婪算法在满足工况约束条件的前提下生成符合目标特征的速度-时间曲线。

![马尔科夫-贪婪算法构建示意图](https://github.com/user-attachments/assets/4b325272-8f0b-4b73-a226-3a4fb9cbc7de)

📎 GitHub源码仓库：[Starbutterflies/Generate-Markov-Driving-Cycles](https://github.com/Starbutterflies/Generate-Markov-Driving-Cycles)


### （2）离散化-随机马尔科夫构建法：
该方法假设机动车的下一运动状态可以完全由当前状态决定。因此，该方法使用真实驾驶数据的频率来估计任意状态下，进入其他状态的概率。最后使用约束条件，保存合格的地方工况。程序框图如下：  

![图片2](https://github.com/user-attachments/assets/1653035a-b130-4a34-89b6-25d630fd5d8f)

📎 GitHub源码仓库：[Starbutterflies/Discretization-Markov-Driving-Cycle](https://github.com/Starbutterflies/Discretization-Markov-Driving-Cycle/tree/master)

## 2.机动车能耗、排放机器学习建模  
机动车能耗、排放模型是评估和管理交通污染的核心工具。运用模型对机动车排放和能耗的定量表征不但具有科学意义，更具有决策意义。然而，现有模型的建模方法论主要成型于二十年前，主流模型的建模思路均是通过少量的“代用参数”（速度、比功率等）对机动车排放和能耗进行表征。随着机器学习、人工智能技术的发展，数据驱动的模型开始在排放建模中展现出强大的潜力。此类模型不再局限于传统物理意义明确的单一参数，而是能够从高维、多源的运行与环境变量中自动学习出复杂的非线性映射关系，实现对排放行为更精细的刻画。这种方法在提升模型拟合精度的同时，也增强了模型在不同使用场景下的泛化能力。在这样的背景下，完成了以下工作：
### （1）机动车能耗建模  
数据是实测数据：
![image](https://github.com/user-attachments/assets/cb4f2651-7ff2-49ec-81b6-2ae18db860b6)


输入特征：
参数包括：VSP, RPA, J, slope, avg.v,  var.v, avg.a, var.a, sum.h, Δs等。
部分参数的计算公式：  

$$
VSP = v \left[ 1.1a + 9.81 \left( a \tan(\sin \theta) \right) + 0.132 \right] + 0.000302v^3
$$  

$$
RPA = \frac{ \int_{T-5}^{T} (v_i \times a_i^+) \, dt }{x}
$$  

$$
J = \frac{da}{dt} = \frac{d^2v}{dt^2}
$$  

目标值的计算公式：  

$$E_{BEV} = C_f U_{bi} I_{bi} (t_{i+1} - t_i)^{\zeta_i}$$

$$E_{ICEV} = \frac{F_{Ci}}{F_E}$$

$$E_{PHEV} = \frac{F_{Ci}}{F_E} + C_f U_{bi} I_{bi} (t_{i+1} - t_i)^{\zeta_i}$$  

建模方法为GridsearchCV，即为网格搜索，使用XGBOOST完成建模。

### （2）重型车辆NOx排放建模  
数据是从OBM数据平台中下载的数据。  
输入特征和目标值：  
![image](https://github.com/user-attachments/assets/ca117639-ae0b-4164-835a-d049a8e4569f)

其中， $$v$$ 为速度， $$v_{t-1}$$ 为前一秒的车辆速度， $$v_{t+1}$$ 为后一秒的车辆速度（下同）。 $$T_{out}$$ 为发动机净输出扭矩， $$T_{f}$$ 为发动机摩擦扭矩，RPM为发动机转速， $$F_{fuel}$$ 为车辆燃油消耗率， MAF为进气量， $$T_{SCR-in}$$ 为SCR装置的入口温度， $$T_{SCR-out}$$ 为SCR装置的出口温度。

分为两种模型，第一种是没有经过SCR装置的NOx排放模型；第二种是经过SCR装置的NOx排放模型。
目标值的计算公式：

$$
NO_{\text{mass},x,\text{up},t} = \frac{0.001 \times 46}{3600 \times 22.4} \times \frac{1}{\rho_e} \times NO_{x,\text{up-stream},t} \times \left( F_{\text{fuel},t} \times \rho_{\text{fuel}} + \text{MAF}_t \right)
$$



$$
NO_{\text{mass},x,\text{down},t} = \frac{0.001 \times 46}{3600 \times 22.4} \times \frac{1}{\rho_e} \times NO_{x,\text{down-stream},t} \times \left( F_{\text{fuel},t} \times \rho_{\text{fuel}} + \text{MAF}_t \right)
$$




建模流程采用了贝叶斯优化，流程图如下：

![mermaid-diagram-2025-05-05-001757](https://github.com/user-attachments/assets/d598c604-8678-43fa-9d25-3c684affb2cb)

📎 GitHub源码仓库：[Starbutterflies/zeping_only](https://github.com/Starbutterflies/zeping_only/blob/master/bst_bayes.py)



## 3.重型货车OBD数据分析  
NOx对人体健康、大气环境、生态环境等都会造成相当的影响。机动车是NOx排放的主要贡献者。随着机动车电气化的推行和国家标准的提高，移动源排放的NOx已经有了极大的改善。然而，重型货车往往有较高的车身重量，燃烧柴油，并难以实现电动化，已经成为了移动源NOx排放的最主要来源。为控制重型货车的NOx排放，国家出台了国VI标准，规定所有新出厂的重型货车都必须安装OBD装置。在这样的背景下，完成了下列研究。
### （1）重型货车NOx排放分析框架  
**灵感来源**：创新点实际上来自于一些表格数据转化为图像的研究，就像下图那样。  
![image](https://github.com/user-attachments/assets/892f1594-6c46-435e-b187-14318db2b1b8)
将数据转化为图像有三个好处。其一，可以压缩数据量，只保留关键的排放规律。其二，可以更加方便的置入深度神经网络中，挖出更深层次的排放规律（原始的表格数据并不能反映整体的排放特性，只能反映局部的时序特征）。其三，可视化的图像比表格化的图像更加直观。

### （2）重型货车NOx数据填补
