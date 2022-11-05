---
layout: post
title: SF19A2890 CPU时钟设置介绍
categories: LINUX
description: 介绍如何设置CPU时钟
keywords: 文档开发
mermaid: true
---

# SF19A2890 CPU时钟设置介绍

**目录**

* TOC
{:toc}


## 适用人员

本项目适用于对网络以及Linux内核有基本了解的人员。

## 开发与测试环境

Siflower开发板/产品板测试镜像。

## 相关背景

CPU时钟是计算机中最小的时间单位，更小的时钟周期意味着更高的工作频率。
晶振产生正弦信号->PLL将其变成方波信号并输出给CPU->CPU的multiplier将信号倍频后输出给CPU的核心->CPU核心每收到一个信号就执行一步操作。

## 功能概述

本文主要介绍CPU时钟的设置，代码原理和硬件原理等．

## 实现原理  
### 配置PLL  
cpu clk = CPU_PLL / 分频比  
频分比默认为二分频  
计算公式如下图：  
![计算公式](/assets/images/uboot_development_manual/CPUformula.png)  

> Fref ：参考时钟，一般为外部晶振频率，一般是40MHZ  
> Refdiv：参考时钟，分频参数。  
> Fbdiv：升频参数，实现整数部分。  
> Frac：升频参数，实现小数部分。（小数部分暂不支持，使用的时候忽略）  
> Postdiv1：升频后，再做分频参数1。  
> Postdiv2：升频后，再做分频参数2。  

在函数中，各个参数所对应的比特位。++并且要注意，分子部分要<=3200Mhz++

| Refdiv | Postdiv2 | Postdiv1 | Frac | Fbdiv |
|----|----|----|----|----|
| [47:42] | [41:39] | [38:36] | [35:12] | [11:0] |

### 具体代码  
![文件代码](/assets/images/uboot_development_manual/uboot_code.png)  

代码位置为**uboot/bare_spl/common/clk.c**   
调用set_pll_ratio_with_params函数来设置pll。  
函数原型：set_pll_ratio_with_params(int pll_type, unsigned long long pll_para);  
其中，第一个参数表示不同pll的不同地址，0代表cpu_pll  
|地址| 类型|对应数值|
|----|----|----|
|0x19E0_1000|cpu_pll|0|
|0x19E0_1040|ddr_pll|1|
|0x19E0_1080|cmn_pll|2|


第二个参数，例如0x49000000028，共44位,“49”在36-43位，可以设置Refdiv,Postdiv2,Postdiv1三个参数  

![对应数值](/assets/images/uboot_development_manual/calculate.png)  

得出Refdiv=1,Postdiv2=1,Postdiv1=1  
以此类推，“28”在0-8位，可以设置Fbdiv，此处Fbdiv为40  
Fref在例子中定义为40Mhz  
最终得出cpu_pll:  
![最终得数](/assets/images/uboot_development_manual/example_result.png)


### 寄存器地址  
cpu_clk的地址如下
> 0x19E0_1500	cpu_clk    


若要配置cpu_clk分频，要在如下地址
> clk_ratio[7:0]	CM_CFG_BASEADDR+0x04	clock ratio for div_clk from mux_clk_s2	  

在cpu_clk地址偏移0x04，即0x1504中修改。对应头文件的CPU_CLK_DIV  
注意该寄存器配置的分频比应该为寄存器里的值加1  
0-> 1分频  
1-> 2分频  
2-> 3分频  

parmmeter寄存器参数如下：CM_PLL_BASEADDR=0x19E01000  
| bit | 偏移地址 | 说明 |
|----|----|----|
| int_pll_para[7:0] | CM_PLL_BASEADDR+0x04|parameter signals for PLL|
| int_pll_para[15:8] | CM_PLL_BASEADDR+0x04|parameter signals for PLL|
| int_pll_para[15:8] | CM_PLL_BASEADDR+0x08|parameter signals for PLL|
| int_pll_para[23:16] | CM_PLL_BASEADDR+0x0C|parameter signals for PLL|
| int_pll_para[31:24] | CM_PLL_BASEADDR+0x010|parameter signals for PLL|
| int_pll_para[39:32] | CM_PLL_BASEADDR+0x014|parameter signals for PLL|
| int_pll_para[47:40] | CM_PLL_BASEADDR+0x018|parameter signals for PLL|  

## FAQ
