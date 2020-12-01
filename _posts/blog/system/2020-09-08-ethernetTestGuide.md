---
layout: post
title: 以太网测试介绍
categories: SYSTEM
description: 介绍如何进行以太网测试
keywords: 以太网 测试
mermaid: true
---

# 以太网测试介绍


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

在进行有线网络测试时，可以分为基础功能测试，性能测试和压力测试等3方面进行，详细可参考内部redmine：[以太网测试](http://redmine.siflower.cn/redmine/issues/7259)。

### 基础功能测试

基础功能测试包含Link测试，数据通路测试，LED测试，长Ping测试等，用于检测有线网络基本功能是否正常，注意下面提到的各种基础功能测试需要测试不同双工模式下的基础功能。

#### Link测试

将PC网线在所有网口上进行插拔测试，检测所有端口的Link状态是否正常；

#### 数据通路测试

LAN口接PC，检测LAN口PC是否能获取到IP，是否能Ping通；WAN口接外网，检测网口是否能获取到IP，是否能Ping通外网；

#### 百米网线测试

PC通过100米网线连接路由器，检测PC是否能正常link，数据收发是否正常。

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

以百兆口Iperf host tx tcp测试为例，提供截图如下：
![iperf_tcp](/assets/images/ethernet_test/host_tx_tcp_iperf.png)


#### IxChariot性能测试

Windows系统下可以通过IxChariot进行以太网性能测试，IxChariot为控制台，客户端程序为endpoint，其中endpoint支持windows、linux、Openwrt、Android等多种设备；详细使用方法可以通过其官网查看：[IxChariot官网](https://www.ixiacom.com/zh/products/ixchariot)

以千兆口IxChariot host tx tcp测试为例，提供截图如下：
![ixChariot_tcp](/assets/images/ethernet_test/host_tx_tcp_IxChariot.png)

#### 思博伦综测仪性能测试
思博伦综测仪测试需要使用到思博伦综测仪，目前只能去东方有线实验室测试或者借一台综测仪设备在公司测试，使用方法可以询问东方有线测试人员，可以参考网上提供的使用方法：[思博伦综测仪使用方法-百度](https://wenku.baidu.com/view/efeb861ffad6195f312ba624.html)

以百兆思博伦1518报文lan-->wan测试为例，提供截图如下：
![sbl_test](/assets/images/ethernet_test/1518-up-SBL.png)

### 压力测试

以太网压力测试可以很好的检验有线网络的可靠性，目前提供了以下2种压力测试方法。

#### 断连Iperf压力测试

在进行Iperf压力测试时，通过切换连接PC的速度双工模式，可以模拟以太网在高吞吐的情况下插拔网线是否会出现问题，用于测试以太网在极端情况下的稳定性。测试通过测试脚本实现，测试脚本见内部redmine：[以太网测试](http://redmine.siflower.cn/redmine/issues/7259)。

#### insmod/rmmod驱动压力测试

通过编写驱动的insmod/rmmod压力测试脚本，进行驱动ko的rmmod/insmod循环压力测试，用于测试驱动的稳定性，检测是否会出现内存泄漏/panic等，可以有效的检测驱动稳定性，测试脚本见内部redmine：[以太网测试](http://redmine.siflower.cn/redmine/issues/7259)。

### 测试环境配置

Linux环境下通过命令安装iperf/ethtool等工具进行测试，Windows环境通过安装IxChariot软件进行性能测试，路由器端通过拷贝/内置测试脚本进行测试。

### 测试流程和测试结果

基础功能测试功能正常视为正常，性能测试达到审核标准视为正常，压力测试系统不挂，数据能通视为正常。

### 测试报告

Siflower提供完整详细的测试报告，如需要请联系Siflower管理人员进行获取。

## 项目引用

### 参考文档

[快速入门](https://siflower.github.io/2020/08/05/quick_start/)

## FAQ
