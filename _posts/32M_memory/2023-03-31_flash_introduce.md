---
layout: post
title: flash分区说明
categories: 32M_MEMORY
description: 分区介绍
keywords: flash
topmost: true
mermaid: true
---

* TOC
{:toc}


# 1. 适用人员

本文适用于使用FLASH进行软件开发的技术人员

# 2. 功能概述

本文主要介绍了系统各个分区的功能以及如何自定义修改flash分区。

# 3. factory分区

目前flash大小共4M，系统分区划分如下：

| 分区名称 | 分区大小 |
| :-: | :-: |
| spl-loader | 32K |
| u-boot | 96K |
| firmware | 3960K |
| factory | 8K |

本文主要关注factory分区。

## 3.1 factory分区详细划分

| index | counts(Bytes) | content | P10h example |Usage |
| :-: | :-: | :-: | :-: | :-: |
| 0-5 | 6 | mac起始地址 | 0xa8,0x5a,0xf3,0,0,0 | 必须有一个唯一mac地址 |
| 6-21 | 16 | sn号 |  全0xff  |app使用（由云服务器将为每台设备分配一个sn号）|
| 22 | 1 | sn号设置确认码 |0xff |app使用（由云服务器配置） |
| 23-26 | 4 | 区分pcba和uboot |  0   |必须为0，在spl中进行判断，跳转uboot还是pcba代码 |
|27-28 | 2| 硬件版本号确认码（'h''v'表示下面的硬件版本号是有效的） |'hv' |硬件版本号有效时才填写|
|29-60| 32| 硬件版本号 | 'A18_XC-LY801-C4_V3_20190412  ' | app使用，仅在网页上显示，可以不配置 |
|61-62| 2| 国家码 | 'CN' |影响wifi信道选择，如果为0xff，则为CN |
|63-64 | 2| 产品型号确认码（'m''v'表示下面的产品型号是有效的） | 'mv' | 产品型号有效时才填写|
|65-96 | 32| 产品型号 | 'XC1200'  |  app使用，仅在网页上显示，可以不配置 |
|97-100 | 4| 硬件特性（默认为0xffffffff，每一位表示一个特性） |0xfffffffe  |必须确认硬件规格后填写！|
|101-102 | 2| 公司名称确认码（'v''d'表示下面的公司名称是有效的） |'vd' |公司名称有效时才填写|
|103-118 | 16| 公司名称 |   'siflower'    | app使用，可以不配置 |
|119-120 | 2| 产品秘钥确认码（'p''k'表示下面的产品秘钥是有效的） |  'pk'  | 产品秘钥有效时才填写|
|121-152| 32| 产品秘钥  （改成32个字节） |'c51ce410c124a10e0db5e4b97fc2af39'  |app使用，可以不配置（使用时秘钥需要根据不同设备向PM申请）|
| 153-154 | 2 | 登录信息确认码（'l''i'表示下面的登录信息是有效的）| 'li' | 登录信息有效时才填写 |
| 155-158 | 4 | 登录信息 | 0xffffffff | 控制telnet server，ssh server，uart等（目前仅实现telnet）|
| 174-177 | 4 | 基准温补温度 | 0x28 | 开启温补功能时作为基准温补温度 |
| 178-181 | 4 | gmac delay值 | 0x30 0x33 | 前两个字节是tx delay，后两个字节是rx delay |

从2048开始，后面4KB存放WiFi的校准信息。其中最前面两个字节存放**WiFi version**信息；而后两个字节存放**XO value**频偏校准值；
接着便是校准表信息，依次是**2.4G一路、5G一路、2.4G二路、5G二路**的校准表，目前默认使用双路校准表，gain值范围是0~15。

## 3.2 增加factory分区内容

若后续需要在factory分区中增加存放新的内容，可在factory_read节点中添加新的属性。
factory_read代码：`OpenWrt-aa/package/kernel/siflower/sfax8-factory-read/src/sf_factory_read_entry.c`
参照代码，在 **sfax8_factory_read_nodes** 结构体数组中添加新的成员，然后在 **sf_fatory_read_entry.c** 中的 **save_value_from_factory_to_host** 函数内参照其他成员的实现完成值的读取功能。
此外，为了使其它程序也能获取到新增属性的值，还需要在 **sf_fatory_read_entry.c** 中的 **sf_get_value_from_factory** 函数内参照其他成员的代码实现读取功能，同时需在 **sfax8_factory_read.h** 中的枚举 **sfax8_factory_read_action** 内把新增成员加上。


