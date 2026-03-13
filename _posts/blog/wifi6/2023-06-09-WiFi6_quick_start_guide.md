---
layout: post
title: WiFi6快速入门手册
categories: WIFI6
description: Siflower WiFi6芯片系统平台快速入门指引
keywords: wifi6
mermaid: true
---

# WiFi6快速入门手册

**目录**

* TOC
{:toc}


## 1 介绍

本文档主要介绍Siflower WiFi6 系统架构，包括编译、新版型引入及私有驱动介绍。

### 1.1 适用人员

使用Siflower硬件进行Openwrt系统开发的技术人员，要求具备基础脚本和Makefile编写能力。

### 1.2 开发环境

Openwrt系统编译环境

Siflower代码环境（请联系Siflower申请代码仓库权限）

开发板或产品板调试环境

## 2 编译说明

### 2.1 代码目录

![directory.png](/assets/images/wifi6/quick_start/directory.png)

- linux-5.10
内核目录，涉及到内核修改，dts修改，驱动修改，在此目录下进行

- Openwrt-master
Openwrt目录，代码修改，系统镜像编译，在此目录下进行

### 2.2 编译说明

- 编译指令
在Openwrt-master目录下执行`./make.sh mpw0_evb_v3`，即可在该目录下生成带板型信息的镜像文件，首次编译失败，可采用`make -j1 V=s`继续编译
当前只支持了mpw0_evb_v3产品板板型，后续会继续新增板型，按新增板型名称编译即可
- 镜像位置
  - 使用make.sh脚本编译会在目录下生成镜像
  - 使用make指令编译生成镜像目录为：`bin/targets/siflower/`

### 2.3 烧录说明
与WiFi5系统一致，具体操作可参考 [镜像更新](https://siflower.github.io/2020/08/05/quick_start/#216-%E6%9B%B4%E6%96%B0%E9%95%9C%E5%83%8F)


## 3 新版型引入

新板型引入包含uboot引入，Openwrt引入两方面，下面将介绍如何在Openwrt添加新板型。

Openwrt引入新版型包含系统和内核两部分。

### 3.1 Openwrt关于不同版型的配置介绍

**所有关于版型的配置内容统一归类在target目录下。**

siflower芯片相关配置在`target/linux/siflower`目录下。

![target.png](/assets/images/wifi6/quick_start/target.png)

其中 sf21h8898-mpw0 目录代表了当前mpw0芯片的配置目录

![mpw0.png](/assets/images/wifi6/quick_start/mpw0.png)

**config-5.10-evb-v3**
表示了不同版型对应kernel config，此文件由make kernel_menuconfig 生成

**base-files**
目录用于存放用户希望预先存放到rootfs下指定路径的文件
>该目录下文件路径和rootfs下一致，在相应路径下防止文件，编译完成后，体现到rootfs中，该目录下文件对于所有siflower 版型通用。

![base-files.png](/assets/images/wifi6/quick_start/base-files.png)

**sf21h8898_mpw0_evb_v3_def.config**
`target/linux/siflower `目录下的`sf21h8898_mpw0_evb_v3_def.config`为config文件，为对应版型的Openwrt文件配置。

>config 文件为在编译根目录下使用make menuconfig 选择不同Openwrt软件模块后生成
后文实例中会具体介绍。

### 3.2 kernel关于不同版型的配置

kernel中有关版型的配置为dts配置部分，kernel本身config，在Openwrt中已有描述。

`linux-5.10/arch/arm64/boot/dts/siflower/`为siflower dts所在路径位置。

>sf21h8898.dtsi 为所有siflower 芯片通用dts配置项
sf21h8898-mpw0-evb-v3.dts为evb_v3版型配置项。

### 3.3 Openwrt新版型引入示例

下文为增加一个8898-mpw0芯片型号，板型名为evb_test的新版型的示例。

#### 3.3.1 Openwrt 增加配置

##### 通用配置引入

- 在`target/linux/siflower/image/sf21h8898-mpw0.mk`文件中新增板型，并指定其对应config与dts

![mk.png](/assets/images/wifi6/quick_start/mk.png)

- 在`target/linux/siflower/sf21h8898-mpw0/`文件夹中新增板型对应内核config文件`config-5.10-evb-test`，可以先拷贝一份，之后更新

- 在`target/linux/siflower/`文件夹中新增openwrt对应config文件`sf21h8898_mpw0_evb_test_def.config`，拷贝为.config，并make menuconfig选上对应板型，若没有对应板型，删除.config、tmp/、build_dir/文件夹后重新拷贝

![copy.png](/assets/images/wifi6/quick_start/copy.png)

![make_menuconfig.png](/assets/images/wifi6/quick_start/make_menuconfig.png)

- 将修改后的.config覆盖回去

![cp2.png](/assets/images/wifi6/quick_start/cp2.png)

- 在`make.sh`中新增板型对应指令

![make.png](/assets/images/wifi6/quick_start/make.png)

#### 3.3.2 linux增加配置

- 在内核`arch/arm64/boot/dts/siflower/Makefile`文件中新增板型对应

![makefile.png](/assets/images/wifi6/quick_start/makefile.png)

- 在`arch/arm64/boot/dts/siflower/`文件夹中新增对应dts

![dts.png](/assets/images/wifi6/quick_start/dts.png)

- 回到openwrt根目录执行`make kernel_menuconfig`，保存即可

- 板型建立完成，可以用`./make.sh mpw0_evb_test `编译，生成带有板型、commit和tag的镜像位于根目录

## 4 siflower私有驱动说明

当前只支持evb_v3产品板板型，目录为`package/kernel/siflower`

![siflower.png](/assets/images/wifi6/quick_start/siflower.png)

>**wifi**
sf_wif目录下的sf21x2880_fmac.ko
**gmac**
kmod_sf_gmac目录下的sf_gmac.ko、sfxgmac_dma.ko、sf_xgmac.ko、sfxgmac_mdio.ko
**dpns**
kmod_sf_dpns目录下的dpnsCommon.ko、sf_netlink.ko、sf_tmu.ko
**factory**
sfax8-factory-read目录下的sfax8_factory_read.ko
**genl_user**
sf_genl_user/files/目录下的sf_genl_user，上层与dpns驱动通信服务

## FAQ
