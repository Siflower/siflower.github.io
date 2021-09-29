---
layout: post
title: SPI Nand Flash物料调试指南
categories: LINUX
description: 介绍如何调试spi nand flash
keywords: 文档开发
mermaid: true
---

# SPI Nand Flash物料调试指南

**目录**

* TOC
{:toc}

## 1 介绍
### 1.1 适用人员

熟悉驱动相关知识

### 1.2 开发环境

ubuntu系统

### 1.3 功能概述

通过在uboot、openwrt、linux下修改配置，添加新的nand flash物料并进行调试。

### 1.4 支持列表

Siflower目前支持的SPI Nand Flash型号如下表：

|       型号       |   厂商   | 大小  |
| :--------------: | :------: | :---: |
|  W25N01GVZEIG  | Winbond  | 1Gbit |
|W25N01GV2E2G5PG | Winbond  | 1Gbit |
|W25N01GVxxxG/T/R| Winbond  | 1Gbit |
|  MX35LF1GE4AB  | Macronix | 1Gbit |

## 2 新物料的调试
### 2.1 uboot修改

该步骤用于在uboot下添加新的nand flash物料。

* 首先查看 drivers/mtd/spi-nand/Kconfig 文件，确定是否有对应厂商的支持，有则忽略此步。本例对应厂商支持在文件中已有，若无则仿照如下形式添加。 

  ![uboot](/assets/images/bsp/Kconfig.png)

* 接着查看 drivers/mtd/spi-nand/spi-nand-base.c 中 spi_nand_table数组，确定要添加的产品型号是否已有支持，如果有则忽略此步，无则添加。如本例按下图形式添加。
  
  ![uboot](/assets/images/bsp/nand_table.png)
  
  其中SPI_NAND_INFO的定义如下，其意义如字面所述，此处不做过多说明。

  ![uboot](/assets/images/bsp/spi_nand_info.png)

* 该步为新的⼚商驱动代码添加，只是新添已⽀持⼚商的新型号物料应该忽略此步，如果添加新的⼚商则应进⾏此步。该步骤建⽴在修改者已经完全理解 uboot 代码中 spi nand flash 驱动的所有代码，且熟悉 spi nand flash 的 spec ⽂档各种相关概念的前提条件下。

  修改 drivers/mtd/spi-nand/spi-nand-base.c，添加新的厂商的配置表，下图中的addr_bytes, addr_bits, dummy_bytes, data_nbits 详⻅需添加的 spi nand flash 的 spec⽂档描述。

  ![uboot](/assets/images/bsp/macronix-table.png)

   并在spi_nand_bin_cfg_table中加入该配置表。

  ![uboot](/assets/images/bsp/spi_nand_bind_cfg_table.png)

### 2.2 openwrt修改

siflower 的 openwrt 的 linux4.14.90 已经⽀持如下⼏个⼚家的 spi nand flash ，分别是 micron, macronix, winbond, icmax, esmt, toshiba , gigadevice。

如果要添加新的⼚商，请参照 linux-4.14.90-dev/linux-4.14.90/drivers/mtd/nand/spi/ ⽬录下的已⽀持⼚商的代码并结合 https://www.kernel.org/doc/html/latest/driver-api/mtdnand.html 编写相关驱动，这部分是 kernel 驱动开发标准流程，不做详细介绍。

本⽂档不说明如何去写驱动，故后⾯只会就如何添加已⽀持⼚商的新的型号的flash 展开说明。

* 首先添加已支持厂商的新的spi nand flash，确定厂商的型号，然后添加对应的flash型号，本例添加的是MACRONIX厂商的spi nand flash，所以编辑代码的路径为linux-4.14.90-dev/linux-4.14.90/drivers/mtd/nand/spi/macronix.c。
  
  查看是否有对应型号的spi nand flash 的支持，如果有则忽略，没有则添加，添加的形式如下：
  ![openwrt](/assets/images/bsp/macronix_spinand.png)

  新增型号需要在代码中新增厂商：

  1. linux-4.14.90-dev/linux-4.14.90/include/linux/mtd/spinand.h
   
  ![openwrt](/assets/images/bsp/spi-nand-manufacturers.png)

  2. linux-4.14.90-dev/linux-4.14.90/drivers/mtd/nand/spi/spi_nand_core.c

  ![openwrt](/assets/images/bsp/spi-nand-core.png)

## 3 配置环境

假设板型为 sfa28_evb，原本使⽤的是 Nor Flash ，改为使⽤ MACRONIX 的MX35LF1G24AD 的 SPI Nand Flash ，以下的相关说明在此前提下展开。

### 3.1 uboot环境配置

* 修改sf_make.sh脚本，在对应的版型后面加上"nand=1"，如下图
  
  ![uboot](/assets/images/bsp/sf_make.png)

  执行 ./make.sh sfa28_evb fullmask ，应用一个已知版型的config配置。

* 执行 make menuconfig进行配置，修改config配置文件， 主要修改点如下：
  
  删除配置CONFIG_CMD_SF

  选择配置CONFIG_SPI_NAND

  选择配置CONFIG_CMD_SPI_NAND

  CONFIG_DEFAULT_DEVICE_TREE 修改为 "sfa28_fullmask_nand"

  CONFIG_SYS_EXTRA_OPTIONS 修改为 "SPI_NAND_BOOT"

  CONFIG_DM_SPI相关选项配置为下图

  ![uboot](/assets/images/bsp/dm_spi.png)

  CONFIG_SPI_FLASH相关选项配置为下图

  ![uboot](/assets/images/bsp/spi_flash.png)

  CONFIG_SPI_NAND 相关配置为下图， 注意 support 的 SPI NAND flash 只能选择⼀个，如果更改型号，请照此⽂档修改config 配置，或者直接修改对应config 配置⽂件。

  ![uboot](/assets/images/bsp/config_spi_nand.png)

* 保存并退出，然后执行 make savedefconfig ，在uboot项目根目录下会得到一个defconfig文件，用该文件覆盖对应版型的config配置文件即可，本例覆盖的是 configs/sfa28_fullmask_p20b_defconfig 文件。

* 再次执行 ./make.sh sfa28_evb fullmask ，即可得到最终镜像。
 
### 3.2 openwrt环境配置

* 先执行 ./make.sh a28_evb ，从而应用一个已有版型的config配置。

* 执行 make kernel_menuconfig ，修改config配置如下
  
  删除配置CONFIG_MTD_SPI_NOR

  选择配置 CONFIG_MTD_NAND

  选择配置 CONFIG_MTD_NAND_SPI

  选择配置 CONFIG_JFFS2_FS_WRITEBUFFER，并在此基础上选中"Remove cleanmarker when spi nand flash was used"这个选项。

  保存退出

* 修改dts，本例修改的路径为linux代码下的 linux-4.14.90/arch/mips/boot/dts/siflower/sf19a28_fullmask_evb.dts ，修改点如下图
  
  ![openwrt](/assets/images/bsp/dts.png)

   修改后如下图

  ![openwrt](/assets/images/bsp/dts-new.png)
