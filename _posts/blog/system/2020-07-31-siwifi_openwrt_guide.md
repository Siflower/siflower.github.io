---
layout: post
title: SiWiFi定制Openwrt用户手册
categories: system
description: 介绍SiWiFi定制Openwrt系统
keywords:  siwifi openwrt 
mermaid: true
---

* TOC
{:toc}

# SiWiFi定制Openwrt用户手册

## 1 介绍

### 1.1 适用人员

适用于想要了解siflower openwrt系统定制的人员

### 1.2 开发环境

siflower SDK环境

### 1.3 相关背景

本文档的目的是描述Siflower路由器软件的体系结构，以及编译和安装的准则，针对希望在Siflower 硬件平台上开发路由器功能的人员提供帮助

## 2 项目引用

### 2.1 参考文档

[opwewrt1806发行说明](https://openwrt.org/zh/releases/18.06/notes-18.06.0)

siflower软件SDK获取以及使用参考[快速入门手册](待添加)

## 3 开发详情

### 3.1 功能设计和流程
  
- Siflower路由器软件概述
  
  本节介绍Siflower路由器软件解决方案的主要组件,下图显示了路由器软件的最高层次

  ![img1](/assets/images/siwifi_image/img1.png) 

#### 3.1.1 openwrt架构

根据GNU通用公共许可证第2版的规定，OpenWrt项目是免费软件。它拥有一个完整的软件堆栈，可由OEM和其他设备实现者移植并在自己的硬件上运行。此地图显示openwrt项目提供的主要软件

  ![img2](/assets/images/siwifi_image/img2.png)

##### 3.1.1.1 源码结构

现在我们使用openwrt18.06作为我们的基本openwrt项目分支，它是2018年7月由openwrt.org发布的最新稳定版本  
下表显示了OpenWrt提供的主要源文件夹和包

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

更多的细节可以参考官方[openwrt guide](https://wiki.openwrt.org/doc/guide-developer)

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
我们保持所有其他文件夹是干净的

#### 3.1.2 内核结构

现在我们使用Linux内核版本4.14.90，它是适配于最新的openwrt18.06的内核版本。 
Openwrt项目在标准内核上有一系列补丁，可以在网络上进行优化或支持上层文件系统。

##### 3.1.2.1 源码结构

内核源代码作为一个包放在dir target / linux /中。 源代码组织：

  ```
  target / linux / generic / patches（openwrt官方给linux的基本补丁）
  target / linux / generic / config-4.14（基于openwrt官方对linux的基本配置）
  target / linux / siflower / sf19a28-fullmask / config-4.14（供应商配置到linux）
  ```

##### 3.1.2.2 修改内核

编译时内核源代码将被提取到build_dir文件夹中

- 你可以在build_dir/目录中对相应内核代码文件修改并重新编译固件，但是请注意，当你使用命令(make clean)清除时，这个目录将被清理干净
  
  ```
  build_dir/target-mipsel_mips-interAptiv_musl/linux-siflower_sf19a28-fullmask/linux-4.14.90/
  ```

- 直接在openwrt-18.06同级目录linux-4.14.90/下修改对应文件，然后回到openwrt-18.06目录编译

##### 3.1.2.3 将补丁添加到内核

Quilt是OpenWrt使用的默认修补工具，通常我们需要一系列步骤来创建一个内核如下的新修补程序

  ```
  make target/linux/{clean,prepare} V=s QUILT=1 // make sure the kernel source is clean
  cd to linux source dir like below:
  build_dir/target-mipsel_mips-interAptiv_musl/linux-siflower_sf19a28-fullmask/linux-4.14.90/
  quilt series // display current patches in kernel
  quilt new platform/001_test.patch // add a new patch which name should be in order
  quilt add drivers/mtd/mtdpart.c // make a association between source file and current patch
  do whatever modification you like
  quilt refresh // effect changes into patch
  cd - // return to top dir
  make target/linux/update // collect patches into vendor dir
  ls target/linux/siflower/patches/001_test.patch // now patch is available and you can upload it
  ```

  关于打patch的说明参考   

  官方说明[openwrt patches](https://wiki.openwrt.org/doc/devel/patches)  

  [openwrt 打patch方法](https://blog.csdn.net/shenwanjiang111/article/details/52252249)

##### 3.1.2.4 siflower package

siflower官方wifi驱动、以太网驱动等均不开放源码，以提供.ko的方式放置在package/kernel/目录

  ```
  package/kernel/sf_smac/  gmac驱动
  package/kernel/sf_smac/  wifi驱动
  package/kernel/sf_switch/ switch驱动
  package/kernel/sf_gswitch/ gswitch驱动
  package/kernel/sf_hnat/  hwNat驱动

  ```

**注：如果需要对wifi驱动，以太网驱动等做订制或者修改，请联系矽昌官方，对接相关流程进行开发**

#### 3.1.3 FLASH布局

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

参考[FLASH分区开发手册](待添加)  

### 3.2 openwrt编译流程

#### 3.2.1 代码下载

siflower SDK放置于GitHub仓库中

SDK下载地址获取及参考[快速入门](待添加)源码下载

#### 3.2.2 编译环境安装

完整编译openwrt固件需要在ubuntu下在siflower SDK目录进行编译

环境安装以及编译说明参考[快速入门](待添加)环境准备章节

#### 3.2.3 编译

- 第一次编译
  
  进入openwrt-18.06根目录，使用如下脚本进行编译

  ```
  ./make.sh ac28 fullmask //ac28代表硬件版型，可选ac28 ac22等，需要使用哪个硬件就编译哪一个
  ```


- 后续编译
  
  在不修改版型的情况下，调试过程进行二次编译或者自定义编译可以使用下列方式进行，这个主要是可以进行差异化编译，先配置编译器，进入文件系统目录，执行命令  

  ```
  make menuconfig
  ```


  进入menuconfig界面进行选择，保存后使用如下命令编译

  ```
  make -j1 V=s
  ```


有关SDK编译的具体说明参考[快速入门](待添加)源码编译章节


#### 3.2.4 目标镜像

##### 3.2.4.1 镜像说明

- 如果使用make.sh脚本进行编译，还会将目标镜像拷贝到openwrt-1806根目录下生成一个按照make.sh脚本中命名方式的镜像  
  
- 如果使用make 指令编译，会在bin/targets/siflower/下生成目标镜像  
  
**注：二者都是一样的镜像，只是命名方式不同，详情参考make.sh代码实现参考脚本代码**

##### 3.2.4.2 镜像烧录

镜像烧录参考[快速入门](待添加)更新镜像章节

### 3.3 调试

#### 3.3.1 串口

对于POSIX系统，推荐minicom作为默认终端软件  
对于windows系统请使用串口工具  
默认情况下，波特率设置为115200  
具体串口安装使用参考[快速入门](待添加)串口调试章节

#### 3.3.2 JTAG

我们在soc上使用MIPS interAptiv  
如果打算使用JTAG调试，则必须准备一个MIPS Debuger并在您的计算机上安装了codescap  
推荐使用 SysProbe SP55调试器

#### 3.3.3 GDB

参考[gdb使用官方说明](https://wiki.openwrt.org/doc/devel/gdb)

