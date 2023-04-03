---
layout: post
title: 4+32项目整体方案介绍
categories: 32M_MEMORY
description: 整体方案介绍
keywords: 整体方案介绍
topmost: true
mermaid: true
---

* TOC
{:toc}


# 1.介绍

## 1.1 适用人员

适用于想要了解siflower openwrt系统的人员

## 1.2 开发环境

siflower SDK环境

## 1.3 相关背景

本文档的目的是描述Siflower路由器软件的体系结构，给Siflower 硬件平台上开发路由器功能的人员提供帮助。

本项目基于WiFi5芯片SF19A2890开发，具备千兆有线网络，2X2双频段WiFi
- 内核版本：Linux-2.6.39
- openwrt版本：OpenWrt-aa
- flash大小：4M
- DDR大小：32M
- WiFi最大用户数：20
- WiFi各频段最大VAP：4

# 2. 开发详情
## 2.1 主要服务

| 服务 | 描述 |
| :--- | :---: |
| dnsmasq | 提供LAN侧DHCP与DNS服务 |
| uhttpd | HTTP服务器 |
| netifd | 网络接口管理服务 |
| syslogd | 用户空间记录服务 |
| klogd | 内核空间记录服务 |
| ubusd | 进程间通信服务 |
| ntpd | 网络时间同步守护进程 |
| hostapd | IEEE 802.1x / WPA / EAP / RADIUS认证器 |
| udhcpc | 提供WAN侧DHCP与DNS服务 |

## 2.2 目录架构

| 文件夹 | 描述 |
| :--- | :---: |
|tools | 获取代码和编译时使用的主机端工具 |
| toolchain | 包括内核头文件，C库，交叉编译器，调试器 |
| target | 内核及板型、镜像等配置，主要关注/target/linux/siflower/文件夹，在此文件夹中我们进行对板型的配置定义，/target/linux/siflower/Linux-2.6.39/为内核代码 |
| package | 包含OpenWrt提供的所有基本包及siflower自定义的软件包，/package/kernel/siflower/为siflower自定义内核驱动所在位置 |
| include | 包含主要的Makefiles和编译规则 |
| scripts | 包括配置脚本，补丁脚本，软件源脚本 |
| dl | 编译时包含所有下载包 |
| staging_dir | 编译环境包括常见的头文件和工具链 |
| bin | 包含输出文件 |

##2.3 Siflower服务介绍

- 管理页面：基于uhttpd+html+cgi架构完成，支持简单网络、WiFi设置
- eswitch_procfs：eswitch模块控制接口，控制节点位于/proc/esw_procfs，目前支持的外网芯片为千兆SWITCH Intel7084，使用方法与18.06基本相同，参考[有线网络和服务介绍 ](https://siflower.github.io/2020/09/08/ethernetGuide/#%E7%A7%81%E6%9C%89%E6%8E%A5%E5%8F%A3)
- gmac_procfs：gmac模块控制接口，控制节点位于/proc/gmac_procfs
- hnat_procfs：hnat模块控制接口，控制节点位于/proc/hnat_procfs，具体参考[HNAT介绍](https://siflower.github.io/2020/09/11/hnat_use_guide/)
- wifi_procfs：wifi模块控制/调试接口，控制节点位于：/proc/lb/siwifi/（2.4G），/proc/hb/siwifi/（5G），具体使用方法参考[wifi调试](https://siflower.github.io/2021/11/29/wifi_debug/)
- WiFi ATE TOOL：基于siflower芯片的ATE测试工具，与板端server服务配合，进行WiFi TX、RX性能测试及校准
- 流量统计
- WPS
- WDS（三地址/四地址）
