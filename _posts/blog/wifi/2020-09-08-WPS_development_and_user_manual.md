---
layout: post
title: WPS开发及使用手册
categories: WIFI
description: 介绍WPS开发及使用
keywords: wifi wps
mermaid: true
---
# WPS开发及使用手册


**目录**

* TOC
{:toc}

## 1 介绍

### 1.1 适用人员

- 掌握基本的wifi配置方法，见[WiFi架构和配置手册](https://siflower.github.io/2020/08/12/wifi_architecture_and_configuration_manual/)

### 1.2 开发环境

- 正常编译环境
- 能烧录SDK镜像(其中包含wifi模块)的硬件平台
- 可以访问路由系统，使用串口线或SSH

### 1.3 相关背景

WPS是由Wi-Fi联盟所推出的Wi-Fi安全防护设定(Wi-Fi Protected Setup)标准，该标准推出的主要原因是为了解决长久以来无线网络加密认证设定的步骤过于繁杂艰难之弊病，使用者往往会因为步骤太过麻烦，以致干脆不做任何加密安全设定，因而引发许多安全上的问题。WPS用于简化Wi-Fi无线的安全设置和网络管理。

### 1.4 功能概述

WPS主要包括两种快速接入的方法：PIN （personal identifiy number） and PBC （push button configuration）。

## 2 项目详情

### 2.1 WPS实现

#### 2.1.1 编译

make menuconfig需要选中：
```
Network--->hostapd-utils
```


#### 2.1.2 配置

配置文件/etc/config/wirless 添加 option wps_pushbutton '1'
如：

```
config wifi-iface
option device radio0
option ifname wlan0
option network lan
option mode ap
option ssid siflower
option encryption psk2+ccmp
option key 12345678
option wps_pushbutton '1'
```


#### 2.1.3 连接

首先由手机等设备发起WPS-PBC或WPS-PIN连接，然后路由器作出相应处理。

##### 2.1.3.1 pbc连接

调用命令：

```
hostapd_cli -i <ifname> wps_pbc
```
ifname为wireless中要使用WPS功能的wifi节点名称，如wlan0表示2.4g，wlan1表示5g，则```hostapd_cli -i wlan0 wps_pbc```会将正在搜寻WPS的设备连上2.4g网络。


##### 2.1.3.2 pin连接

调用命令
```
hostapd_cli -i <ifname> wps_pin any <pin_num>
```
如```hostapd_cli -i wlan1 wps_pin any 80919544```

### 2.2 网页设置

todo

## 3 FAQ