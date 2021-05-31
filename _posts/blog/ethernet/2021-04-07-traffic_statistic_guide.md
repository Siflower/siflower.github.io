---
layout: post
title: 流量统计介绍
categories: DEVELOP
description: 流量统计介绍
keywords:  plan
mermaid: true
---

# TS介绍

**目录**

* TOC
{:toc}


## 1 适用人员

本项目适用于对网络以及Linux内核有基本了解的人员。

## 2 开发与测试环境

Siflower开发板/产品板测试环境，镜像需要选中sf_ts模块。

## 3 相关背景

TS是Traffic Statistic的缩写，即流量统计模块，是Siflower自己开发的一个集流量统计，拉黑，限速为一体的功能模块。

## 4 功能概述

本文主要介绍TS模块的功能，实现原理，对外接口和调试方法等．

## 5 开发详情

### 5.1 支持功能

* 流量统计功能
  * 针对设备进行流量统计，计算设备每秒的上传／下载速度并在网页显示；

* 拉黑功能
  * 针对设备进行拉黑设置；

* 限速功能
  * 针对设备进行上传/下载速度带宽限制；

### 5.2 实现原理

#### 5.2.1 流量统计功能

流量统计功能实现主要通过Netfilter框架在不同Hook点实现对报文的信息统计即可，但是Siflower TS流量统计唯一区别是需要考虑硬件nat进行报文转换的情况，即流量统计需要同时统计走软件的报文和走硬件转发的报文．

- 走软件报文统计
Netfilter框架主要包含5个数据包的钩子点，分别是PRE_ROUTING、INPUT、OUTPUT、FORWARD与POST_ROUTING，针对设备的流量统计search依据是mac地址，因此统计上传速度／流量就需要找出上行流量中报文源mac地址是统计设备的点，统计下载速度／流量就需要找出下行流量中报文目的mac地址是统计设备的点．早期我们主要通过NF_INET_FORWARD HOOK点进行上行报文统计，NF_BR_LOCAL_OUT HOOK点进行下行报文统计，由于担心Bridge中的hook点可能影响lan-wifi的吞吐，因此最终将下行报文统计HOOK点移到了NF_INET_POST_ROUTING，而在NF_INET_POST_ROUTING HOOK点报文mac地址还没有进行NAT转换，因此到达此处的下行流量的mac地址还是外网到路由器wan的mac, 要想依据mac地址进行下行流量统计, 需要进行一步mac地址转换操作．
上传／下载流量统计HOOK详细介绍:
NF_INET_FORWARD：上传流量统计点．此点是转发之前的点, 转发的流量是lan(br-lan)和wan(eth0.2)之间的流量； 即到达此处报文的mac地址是lan pc到bridge(br-lan)或者外网到路由器wan的；是 MAC地址转换之前的点, 所以可以用来统计上行流量, 但是无法统计下行流量；　　
NF_INET_POST_ROUTING：下载流量统计点．NFPROTO_IPV4协议的最后一个点, 此处可以统计到下行流量, 但是此处有一个关键点是mac地址转换(NAT)是在POSTROUTING挂载点之后才被执行, 所以到达此处的下行流量的mac地址还是外网到路由器wan的mac, 要想依据mac地址进行下行流量统计, 需要进行一步mac地址转换操作, 具体实现见redmine#[7429](http://redmine.siflower.cn/redmine/issues/7429)；

- 走硬件的报文统计
走硬件转发的报文需要hnat模块报文计数功能支持，hnat模块详细介绍见:[HNAT介绍文档](https://siflower.github.io/2020/09/11/hnat_use_guide/)。
目前hnat仅支持DNAT时依据dmac查询报文数量，因此走硬件的报文统计仅支持下载流量统计，且由于仅支持查询报文数量，无法查询报文长度，因此走硬件的下行流量默认使用的mtu 1500进行的报文统计，和实际下载流量略有差异．

#### 5.2.2 设备拉黑功能

拉黑功能主要靠ipset命令实现，通过ipset可以创建一个基于mac地址的list表，然后通过iptable命令将src是这个表里面的成员全部drop掉，网页实际通过调用脚本命令实现此功能．
使用Siflower芯片实现此功能时需要额外注意一点，在进行设备拉黑时需要先将设备从hnat硬件转发表中踢除，避免设备流量走硬件偷跑了，这部分逻辑在Netfilter框架中通过维护一份"黑名单"，在flowoffload entry添加时，检测设备是否拉黑来决定hw flowoffload entry的添加，以及将已存在的设备hw flowofflad entry回收来实现，从源头锁死设备拉黑情况．

#### 5.2.3 设备限速功能

限速功能核心是tc命令，通过tc命令配置qdisc/calss/filter等参数实现数据流的限速功能，网页对接功过调用脚本实现此功能.
注意此功能同拉黑一样需要先将设备从hnat硬件转发表中踢除，避免设备流量走硬件偷跑，因此此功能使能后被限速设备上网只能走软件．

## 6 对接接口

### 6.1 Openwrt系统对接接口

流量统计功能需要支持在网页实时显示设备上传/下载速度情况，因此TS模块通过文件系统节点对外暴露当前连接设备的流量统计情况．
获取当前连接设备流量统计的命令和结果展示如下：(单位Byte)
```
root@OpenWrt:/# cat proc/ts
mac                upload               download             upspeed    downspeed  maxups     maxdowns
11:22:33:44:55:11  1331299              919097               0          115        9576       17241040  
11:22:33:44:55:12  303                  112                  52         4552       251        21060  
```
网页通过字符串截取结合上述命令完成对设备流量统计的展示.

### 6.2 Linux内核对接接口

限速/拉黑功能在进行设备踢除时需要在内核Netfilter框架中维护一份黑名单，用于在设备限速/拉黑时禁止添加hw flowofflad规则，详细支持代码位于net/netfilter/xt_FLOWOFFLOAD.c中，目前已开源给所有客户．

## 7 调试方法

目前可以通过如下命令开启TS Debug日志，设备拉黑／限速时会有设备信息打印出来．
日志开启/关闭命令：
```
echo 6 11:22:33:44:55:66 > proc/ts
```
命令说明：echo 6 任意mac地址 > proc/ts，使用一次是开启日志，再次使用即关闭日志．

## 8 已知问题

HNAT仅支持DNAT时依据dmac查询报文数量，因此走硬件的流量仅支持下载流量统计，网页上上传速度是被MASK掉了，而且由于仅HNAT目前针对设备仅支持查询报文数量，无法查询报文长度，因此走硬件的下行流量默认使用的mtu 1500进行的报文统计，和实际下载流量略有差异．

## 9 项目引用

- hnat驱动对接参考redmine#[7429](http://redmine.siflower.cn/redmine/issues/7429)

## 10 FAQ

