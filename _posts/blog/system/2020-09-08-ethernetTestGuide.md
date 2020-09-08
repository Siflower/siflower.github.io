---
layout: post
title: 以太网测试介绍
categories: system
description: 介绍如何进行以太网测试
keywords: 以太网 测试
mermaid: true
---

# 以太网测试介绍

**版权所有©上海矽昌微电子有限公司2019。保留一切权利。**

非经本公司许可，任何单位和个人不得擅自摘抄、复制本文档内容的部分或全部，并不得以任何形式传播。

**商标申明**

SiFlower、矽昌和矽昌其它商标均为上海矽昌微电子有限公司的商标，本文档提及的其它所有商标或注册商标，由各自的所有人拥有。

**注意**

您购买的产品、服务或特性应受矽昌公司商业合同和条款的约束，本文档所描述的全部或部分产品、服务或特性可能不在您的购买和使用范围内。除合同另有约定，矽昌公司对文档的内容不做任何明示或暗示的声明和保证。

**上海矽昌微电子有限公司**

- 地址：上海市浦东新区祖冲之路887弄84号楼408室
- 网址：http://www.siflower.com/
- 客户服务电话：021-51317015
- 客户服务传真：
- 客户服务邮箱：


**目录**

* TOC
{:toc}


## 适用人员

本文适用于想要进行Siflower硬件以太网测试的人员。

## 测试环境

Linux/Windows PC，开发板测试环境。

## 相关背景

随着产品形态的完善，对有线网络功能/性能的测试也越来越严格，本文档主要针对此情况汇集了所有以太网测试相关案例进行汇总阐述。

## 以太网测试

在进行有线网络测试时，可以分为基础功能测试，性能测试和压力测试等3方面进行，详细可参考：[以太网测试](http://redmine.siflower.cn/redmine/issues/7259)。

### 基础功能测试

基础功能测试包含Link测试，数据通路测试，LED测试，长Ping测试等，用于检测有线网络基本功能是否正常，注意下面提到的各种基础功能测试需要测试不同双工模式下的基础功能。

#### Link测试

将PC网线在所有网口上进行插拔测试，检测所有端口的Link状态是否正常；

#### 数据通路测试

LAN口接PC，检测LAN口PC是否能获取到IP，是否能Ping通；WAN口接外网，检测网口是否能获取到IP，是否能Ping通外网；

#### LED测试

检测网线Link/数据收发时，LED状态是否正常；

#### 长Ping测试

检测长时间Ping时，查看是否丢包，检测网络稳定性；

### 性能测试

目前常见的性能测试方法有：Iperf测试，IxChariot测试和思博轮综测仪测试。

#### 性能测试测试项

有线网络性能测试常见测试项如下：  
* 单口Host TX TCP/UDP；
* 单口Host RX TCP/UDP；
* 单口Host 双向 TCP/UDP；
* LAN-LAN SWITCH 单向TCP/UDP；
* LAN-LAN SWITCH 双向TCP/UDP；
* LAN-WAN NAT 上行TCP/UDP；
* LAN-WAN NAT 下行TCP/UDP；
* LAN-WAN NAT 双向TCP/UDP；
* ALL PORT上行；
* ALL PORT下行；

#### Iperf性能测试

使用iperf可以快捷简便的进行以太网性能测试，Linux系统下通过```man iperf```或者```iperf -h```可以查看iperf使用方法。  
不过使用iperf时需要注意以下几点：
- 测试1000M网口的时候，iperf单条流使用单核无法达到最大性能，需要加上-P 4 使用多线程进行发送；
- Iperf测试路由端和PC端iperf版本最好保持一致，如都使用2.0.12；
- 如果发现双向测试-d参数无法使用时，可以起2个server和client，手动跑2条流；
- 跑多条流时可以加上-p参数指定端口号；

#### IxChariot性能测试

Windows系统下可以通过IxChariot进行以太网性能测试，IxChariot为控制台，客户端程序为endpoint，其中endpoint支持windows、linux、Openwrt、Android等多种设备；详细使用方法可以通过其官网查看：[IxChariot官网](https://www.ixiacom.com/zh/products/ixchariot)

#### 思博伦综测仪性能测试
思博伦综测仪测试需要使用到思博伦综测仪，目前只能去东方有线实验室测试或者借一台综测仪设备在公司测试，使用方法可以询问东方有线测试人员，可以参考网上提供的使用方法：[思博伦综测仪使用方法-百度](https://wenku.baidu.com/view/efeb861ffad6195f312ba624.html)

### 压力测试

以太网压力测试可以很好的检验有线网络的可靠性，目前提供了以下2种压力测试方法。

#### 断连Iperf压力测试

在进行Iperf压力测试时，通过切换连接PC的速度双工模式，可以模拟以太网在高吞吐的情况下插拔网线是否会出现问题，用于测试以太网在极端情况下的稳定性。测试通过测试脚本实现，测试脚本见[以太网测试](http://redmine.siflower.cn/redmine/issues/7259)。

#### insmod/rmmod驱动压力测试

通过编写驱动的insmod/rmmod压力测试脚本，进行驱动ko的rmmod/insmod循环压力测试，用于测试驱动的稳定性，检测是否会出现内存泄漏/panic等，可以有效的检测驱动稳定性，测试脚本见[以太网测试](http://redmine.siflower.cn/redmine/issues/7259)。

### 测试环境配置

Linux环境下通过命令安装iperf/ethtool等工具进行测试，Windows环境通过安装IxChariot软件进行性能测试，路由器端通过拷贝/内置测试脚本进行测试。

### 测试流程和测试结果

基础功能测试功能正常视为正常，性能测试达到审核标准视为正常，压力测试系统不挂，数据能通视为正常。

### 测试报告

以repeater性能测试报告为例，提供详细性能测试报告如下：[repeater性能测试报告](待添加)

## 项目引用

### 参考文档

[快速入门](/_posts/blog/system/2020-08-05-quick_start.md)

## FAQ
