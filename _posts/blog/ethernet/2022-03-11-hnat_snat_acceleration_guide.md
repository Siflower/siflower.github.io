---
layout: post
title: 以太网到WiFi加速流程介绍
categories: DEVELOP
description: 加速流程
keywords:  acceleration
mermaid: true
---

# 以太网到WiFi加速机制介绍

**目录**

* TOC
{:toc}


## 1 适用人员

本项目适用于对网络以及Linux内核有基本认识，需要了解siflower方案以太网到WiFi加速流程的人员。

## 2 开发与测试环境

Siflower开发环境、siflower芯片开发板或者产品板。

## 3 概述

本文主要从代码角度介绍siflower方案以太网到WiFi加速流程。

## 4 详情

### 4.1 硬件结构框图

芯片内部大致结构框图如下。

![accel1](/assets/images/acceleration/accel_1.png)

**模块介绍：**

1、2 是基于外围switch的物理Port划分出来的虚拟Interface，分别表示WAN和LAN，由sf_eswitch驱动负责管理控制；  
3 是矽昌3层硬件转发模块hw nat，简称hnat，由sf_hnat驱动负责管理控制；  
4 是矽昌千兆MAC，由sf_gmac以太网驱动负责管理控制；  
5 是mips CPU；  
6 是wifi模块，由sf_smac驱动负责管理控制；  

**数据流向：**

LAN -> WIFI数据流向如下：2 -> 4 -> 5 -> 6 （虽然物理上经过模块3，但实际模块3不进行任何数据处理，相当于bypass）  
WAN -> WIFI数据流向如下：1 -> 3 -> 4 -> 5 -> 6  
WIFI -> LAN数据流向如下：6->5->4->2  
WIFI -> WAN数据流向如下：6->5->4->3->1  

### 4.2 数据转发介绍

#### 4.2.1 LAN -> WIFI数据转发实现

LAN到WIFI数据转发为2层数据转发，由CPU负责进行转发，SF19A2890在整体转发链路中增加了自己的软加速实现，在协议栈报文处理流程中如果检测到lan-wifi的2层转发报文会提前进行转发。整体转发流程实现如下：

![accel2](/assets/images/acceleration/accel_2.png)

**流程关键点代码介绍：**

sgmac_rx: sf_gmac驱动代码的sf_gmac.c中，为以太网驱动中断收报处理函数；  
hook_dev_xmit_path： 内核协议栈net/core/dev.c中，为内核rx报文处理cpu分配之后的早期点；  

#### 4.2.2 WAN -> WIFI数据转发实现

WAN到WIFI为3层NAT数据转发，此过程会经过以太网硬加速nat转换，整体链路也涉及到以太网驱动和WIFI驱动处理。整体转发流程实现如下：  

![accel3](/assets/images/acceleration/accel_3.png)


**流程关键点代码介绍：**

硬件打上发送到wifi的特殊vlan tag： 软件自定义vlan tag，目前保留了4000~4015的vlan tag用于标记给wifi的报文；对应代码定义为phnat_priv->wifi_base，保留长度由最大wifi虚拟interface决定，涵盖pppoe情况下最大需要预留vlan tag为16；  
sgmac_rx: sf_gmac驱动代码的sf_gmac.c中，为以太网驱动中断收报处理函数；  
wifi_xmit_prepare： 真正实现位于sf_hnat驱动sf_hnat.c中，为以太网给wifi的报文预处理函数；  
协议栈rps流程： 内核协议栈net/core/dev.c中，为内核rx报文处理cpu分配之后的早期点；  

#### 4.2.3 WIFI -> LAN数据转发实现

WIFI到LAN数据转发同样为2层数据转发，由CPU负责进行转发，SF19A2890在整体转发链路中增加了自己的软加速实现，在协议栈报文处理流程中如果检测到wifi-lan的2层转发报文会提前进行转发。整体转发流程实现如下:

![accel4](/assets/images/acceleration/accel_4.png)

**流程关键点代码介绍：**

大致流程如下：

```
siwifi_rx_thread_process_skbs(): sf_smac驱动代码的siwifi_rx.c中，该接口为了将复制的ip端口信息在frag ip pkt上通过hnat搜索(为skb打上特殊的flags)，然后发送到上层协议栈中。
sf_hnat_search():sf_hnat驱动代码的sf_hnat.c中。
sgmac_xmit()：sf_gmac驱动代码的sf_gmac.c中。
hook_dev_xmit_path：内核协议栈net/core/dev.c中，为内核rx报文处理cpu分配后的早期点； 主要由backlog_skb_handler_register()和backlog_skb_handler_register()函数来对g_wlan_hook_fn进行做处理。
siwifi_rx_thread_process_skbs()//复制IP端口信息
	->netif_receive_skb()
		->netif_receive_skb_internal()
			->process_backlog()
				->hook_dev_xmit_path()//判断g_wlan_hook_fn是否为NULL(是则进行软加速，否则通过协议栈发出）
					->_netif_receive_skb()
```

#### 4.2.3 WIFI -> WAN数据转发实现

WIFI到WAN同样为3层数据转发，此过程会经过以太网硬加速nat转换，整体转发流程实现如下：

![accel5](/assets/images/acceleration/accel_5.png)

**流程关键点代码介绍：**

大致流程如下：

```
siwifi_rx_thread_process_skbs()//复制IP端口信息
	->sf_hnat_search()//打上特殊标记
		->netif_receive_skb()//进入协议栈
			->netif_receive_skb_internal()
				->process_backlog()//协议栈rps处理流程（在禁用本地irq的情况下调用，但在启用本地irq的情况下退出），然后循环出队，处理nat标记flags的skb
					->skb->dev->netdev_ops->ndo_start_xmit//硬加速
						->sgmac_xmit()//驱动程序的TX入口点
```

## 5 FAQ

