---
layout: post
title: Siflower软件服务介绍
categories: SYSTEM
description: 介绍Siflower方案软件架构和功能
keywords:  siwifi openwrt 
mermaid: true
---

# Siflower软件服务介绍

* TOC
{:toc}

## 1 介绍

### 1.1 适用人员

适用于想要了解siflower openwrt系统的人员

### 1.2 开发环境

siflower SDK环境

### 1.3 相关背景

本文档的目的是描述Siflower路由器软件的体系结构，给Siflower 硬件平台上开发路由器功能的人员提供帮助

## 2 项目引用

### 2.1 参考文档

opwewrt1806[发行说明](https://openwrt.org/zh/releases/18.06/notes-18.06.0)

siflower软件SDK获取以及使用参考[快速入门手册](https://siflower.github.io/2020/08/05/quick_start/)

## 3 开发详情

### 3.1 功能设计和流程

- Siflower路由器软件概述

  本节介绍Siflower路由器软件解决方案的主要组件,下图显示了路由器软件的提供的所有功能

  ![img1](/assets/images/siwifi_image/img3.png) 

#### 3.1.1 openwrt架构

根据GNU通用公共许可证第2版的规定，OpenWrt项目是免费软件。它拥有一个完整的软件堆栈，可由OEM和其他设备实现者移植并在自己的硬件上运行。此地图显示openwrt项目提供的主要软件

  ![img2](/assets/images/siwifi_image/img2.png)

##### 3.1.1.1 源码结构

现在我们使用openwrt18.06作为我们的基本openwrt项目分支，它是2018年7月由openwrt.org发布的最新稳定版本,下表显示了OpenWrt提供的主要源文件夹和包

|文件夹|描述|
|---|---|
|tools	|获取代码和编译时使用的主机端工具|
|toolchain|包括内核头文件，C库，交叉编译器，调试器	|
|target	|定义供应商文件和图像工具|
|package	|包含OpenWrt提供的所有基本包|
|include	|包含主要的Makefiles和编译规则|
|scripts	| 包括配置脚本，补丁脚本，软件源脚本|
|dl	| 编译时包含所有下载包|
|build_dir|	编译时的临时文件以及提取的源代码|
|staging_dir	|编译环境包括常见的头文件和工具链|
|feeds	|所有可选软件包由openwrt或thirdparty提供|
|bin	|包含输出文件|

更多说明可以参考官方[openwrt guide](https://wiki.openwrt.org/doc/guide-developer)

##### 3.1.1.2 主要服务

下表是Openwrt启动时的主要服务

|服务 |描述	|
|---|---|
|dropbear	|为小型内存环境设计的小型SSH2服务器/客户端|
|dnsmasq	|它旨在为LAN提供耦合的DNS和DHCP服务|
|telnetd|telnet服务器的Telnet守护进程|
|uhttpd|小巧的单线程HTTP服务器|
|netifd	|网络接口管理器服务|
|odhcpd|用于ipv6的DHCP服务器|
|ubusd	|进程间通信服务|
|logd	|记录用户空间的服务|
|ntpd	|网络时间同步守护进程|
|hostapd|IEEE 802.1x / WPA / EAP / RADIUS认证器|

##### 3.1.1.3 Siflower移植代码

所有用于OpenWrt的siflower移植代码都放置在target / linux / siflower /中 
Linux的siflower移植代码都放在package / kernel /中，我们保持所有其他文件夹是干净的。

##### 3.1.1.4 Siflower Openwrt框架介绍

基于Openwrt源生框架，Siflower进行了自有服务开发，相应功能代码位于package/siflower下。

用户界面我们开发一套用于商用的网页界面，再Luci中作为一个独立包可以选择。
用户接口以http接口的方式对外提供了一套可以控制路由器功能的API接口。

对于底层和中间层我们也进行了大量的开发和优化，以linux 模块 api + 上层应用程序的方式提供一些系统应用。

APP应用方面，我们开发了IOS和Android应用，云端服务。可以支持远程路由器操作，OTA等功能。

##### 3.1.1.5 Siflower Openwrt服务介绍

基于Openwrt源生代码，Siflower开发了多种个性化服务，详细服务介绍如下：

* **WAN/LAN自适应服务介绍**

  > 针对用户不清楚wan口lan口，插错网线导致不能上网的情况，我们进行了wan/lan自适应的开发，其会自动根据当前的网线连接情况划分wan/lan口。如果不小心将外网网线插错到了lan口，路由器会自动将wan重新划分到该网口上，成功配置后路由器依旧可以上网。开发使用脚本语言，开机自启动。

  开发详情参考[wan-lan自适应开发手册](https://siflower.github.io/2020/09/11/wan_lan_auto_adapt/)  

* **Repeater自动弹窗服务介绍**

  > 针对中继器产品形态，提供独立的中继器页面，以及用户连接之后，手机/PC自动跳转浏览器显示配置页面，快速引导用户完成配置。

  开发详情参考[中继器配置相关说明](https://siflower.github.io/2020/09/08/repeaterConfigGuide/)

* **设备上下线服务介绍**

  > 设备上下线服务用于检测设备上下线状态，在网页和APP上正确显示。可以通过远程/本地方式，app显示通知。


* **设备管理**

  > 可以针对每台连接到路由器的设备进行管理，包括url限制，流量限制，时间限制，拉黑等操作。


* **WiFi双频合一**

  > 2.4G 5G 合并ssid+密码设置，同时根据客户设备的接入情况，选择性的引导客户接入5G网络。

  开发详情参考[wifi双频合一使用手册](https://siflower.github.io/2020/09/09/band_single_ssid/)

* **WiFi无感接入**

  > 通过siflower私有协议，设备和路由器之间可以跳过扫描+输入密码流程，直接连接路由器。

* **一键中继**

  > 一键中继功能，路由器虚拟出一个station设备，并通过WPS和上级路由连接，然后通过路由器本身的wifi网络，提供其他设备接入，扩展wifi覆盖。


* **AC/AP**

  > 路由器本身也支持作为AC，最多可以控制128个AP，进行AP配置管理，带宽管理。

* **租赁网络**

  > 在绑定路由器后，用户可以开启租赁网络功能。该功能为开放网络，但需要认证。
  认证方式包括免费模式、支付宝微信付费模式、短信验证模式。主要用于陌生用户有偿或无偿使用用户网络。
  现支持大部分手机在连接到网络时主动弹出认证页面，未主动弹出的可以在浏览器访问页面进行验证

* **管理网页/app**

  > siflower自行设计了适用于路由器、86v、AC、中继器等不同产品的管理页面，并且支持在管理页面快速客制化修改和二次开发。
    同时也开发了Android/IOS 矽路由app进行路由设备功能管理，我们也提供android/ios SDK供客户进行二次开发。

  网页开发参考[管理网页客制化手册](https://siflower.github.io/2020/07/31/manage_web_custom_guide/) [管理网页开发手册](https://siflower.github.io/2020/08/05/manage_web_develop_guide/)

  app开发参考[Android SDK集成指南](https://siflower.github.io/2020/07/29/android_sdk/) [iOS SDK集成指南](https://siflower.github.io/2020/08/05/iOS_SDK/)

##### 3.1.1.6 Siflower Linux模块介绍

为了保证对上层服务的应用支持，我们在底层开发了统一的操作接口，对接不同外部硬件。同时也开发一些基于我们硬件特有的底层功能。

* **ethtool_ops**

  > 对于使用外部switch芯片，我们开发了特别的ethtool_ops接口。  
  除了支持GPHY的速度双工获取/设置，也支持GSWITCH每个口的速度双工获取/设置。

  参考[有线网络和服务介绍](https://siflower.github.io/2020/09/08/ethernetGuide/)

* **WiFi动态功耗调整**

  >  根据设备信息动态调整wifi的发射功率，降低功耗。

* **hnat_ops**

  > hnat模块操作接口，向上层提供nat offload功能。

  参考[HNAT对接和使用手册](https://siflower.github.io/2020/09/11/hnat_use_guide/)

* **pkt send tool**

  > 底层开发工具，支持自定义任何报文字段和长度，发送时间，带宽等，方便进行性能优化。


* **支持的外围芯片列表**

  目前Siflower有线网络已支持如下外围芯片，如需增加新的外围设备可联系我们进行支持。

  | 设备类型 | 设备型号 |
  | -- | -- |
  | 千兆PHY | Realtek8211 |
  | 千兆SWITCH | Realtek8367C |
  | 千兆SWITCH | Intel7084 |
  | 千兆SWITCH | IP1707 |

  外围芯片对接参考[外网switch芯片对接和使用手册](https://siflower.github.io/2020/09/11/new_switch_import_guide/)

##### 3.1.1.7 Siflower 其它应用服务介绍

* **WiFi ATE TOOL**

  > 基于siflower芯片的ATE测试工具SFATETESTTOOL,可以用来测试WIFI TX、RX的性能。
  SFATETESTTOOL工具基于操作方式的不同分为两款，第一个是通过Windows系统的操作界面来控制的（SFATETESTTOOL UI）,
  另一个是基于串口输入命令行来控制的(SFATETESTTOOL Command)

  使用详情参考[ATE TOOL使用手册]()

* **siflower pcba test tool**

  > 使用PCBA测试工具，在产线上对产品进行测试，旨在自动化、批量化、快速地完成产品基本功能的测试。
  包含WiFi校准测试，以太网测试，WiFi耦合测试，按键LED测试,写入mac地址和产品信息等基本功能

  使用详情参考[PCBA TEST TOOL使用手册]()

#### 3.1.2 FLASH布局

Flash分区：

  <table class="inline">
    <tbody><tr class="row0">
      <th class="col0"> Layer0 </th><td class="col1 centeralign" colspan="8">  raw flash 16M </td>
    </tr>
    <tr class="row1">
      <th class="col0"> Layer1 </th>
          <td class="col1 centeralign" rowspan="1" colspan="3">  uboot  partitions  </td>
              <td class="col2 centeralign" rowspan="3" colspan="1"> <strong><code>mtd3</code></strong> factory <br />64K</td>
              <td class="col3 centeralign" colspan="3">  <strong><code>mtd4</code></strong><br /> firmware <br />15744K </td>
    </tr>
    <tr class="row2">
      <th class="col0"> Layer2 </th>
          <td class="col1 centeralign" rowspan="2" > <strong><code>mtd0</code></strong> <br /> spl_loader <br />128k  </td>
          <td class="col2 centeralign" rowspan="2">  <strong><code>mtd1</code></strong> <br /> uboot <br />384K    </td>
          <td class="col3 centeralign" rowspan="2">  <strong><code>mtd2</code></strong> <br /> uboot_env <br />64k   </td>
          <td class="col4 centeralign" rowspan="2">   <strong><code>mtd5</code></strong> <br /> kernel <br /> 1477K <br /> uImage_lzma</td>
          <td class="col5 centeralign" rowspan="1" colspan="2">  <strong><code>mtd6</code></strong>
          <br/>rootfs <br />14267K<br />mounted: "<code>/</code>" </td>
    </tr>
    <tr class="row3">
      <th class="col0"> Layer3 </th>
          <td class="col1 centeralign" colspan="1">                                                          <strong><code>/dev/root</code></strong> <br />
                  mounted: "<code>/rom</code>"<br />5371K<br />
                  root.squashfs (increase in 256K for mkfs with block size 256K)
              </td>
              <td class="col2 centeralign"  colspan="1">             
                  <strong><code>mtd7</code></strong> <br /> rootfs_data <br /> 8896K<br />
                  mounted: "<code>/overlay</code>" <br />
                  used:632K
              </td>
    </tr>
  </tbody></table>


每个分区的描述：

|分区|描述|
|---|---|
|Spl	| uboot的第一阶段，负责将Uboot加载到dram和init hw中|
|Uboot|	负责从spi中提取uImage.lzma到dram并跳转到内核|
|Uboot-env	|存储uboot使用的通用参数，例如波特率|
|Factory	| Store参数在重置或升级时不会被删除|
|Linux	|标准Linux内核与硬件交互|
|Rootfs|除内核以外的所有openwrt文件系统|
|Rootfs_data|rootfs_data Jffs2 rw文件系统|

详细介绍参考[FLASH分区开发手册](https://siflower.github.io/2020/09/08/flashPartitionGuide/)

#### 3.1.3 软件代码获取

siflower SDK代码使用及获取参考[快速入门手册](https://siflower.github.io/2020/08/05/quick_start/)

