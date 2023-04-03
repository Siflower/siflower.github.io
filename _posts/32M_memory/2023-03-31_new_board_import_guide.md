---
layout: post
title: 新的板型引入指南
categories: 32M_MEMORY
description: 如何建立新板型
keywords: 新板型
topmost: true
mermaid: true
---

* TOC
{:toc}


# 1. 适用人员

使用siflower 4+32项目SDK，有一定代码基础的技术人员。

# 2. 使用环境

- 硬件基础：需要有SDK支持的硬件平台
- 编译环境：Ubuntu环境
- 代码仓库：`/external/custom_release_code-AA`
需要申请仓库权限请联系Siflower

# 3. 新建版型

## 3.1 建立编译版型

修改make.sh脚本，新增编译版型

  ![make_sh.png](/assets/images/4+32/make_sh.png)


## 3.2 新建版型config文件

版型配置文件在```target/linux/siflower/```目录
- 新建版型配置```openwrt_config_a28_ac28```
- 新建版型patch（非必要）```patches-2.6.39_ac28```
- 新建内核config`target/linux/siflower/config-2.6.39_ac28`

## 3.3 建立版型目录ac28

![ac28.png](/assets/images/4+32/ac28.png)

- 修改target.mk

![target_mk.png](/assets/images/4+32/target_mk.png)

- 进入profiles目录，新增版型mk，**注意板型命名不能用-分割，可以使用_下划线**

![board_mk.png](/assets/images/4+32/board_mk.png)

## 3.4 在`target/linux/siflower/image/Makefile`中增加对应板型编译信息

![image_makefile.png](/assets/images/4+32/image_makefile.png)

若分区与默认ac28板型不同，还需在`/target/linux/siflower/Linux-2.6.39/arch/mips/siflower/spi.c`文件中增加或修改分区信息，类似于4.14.90板型dts的spi分区信息

![spi.png](/assets/images/4+32/spi.png)

分区固定在`target/linux/siflower/ac28/config-default`文件中
![mtd.png](/assets/images/4+32/mtd.png)

## 3.5 回到openwrt根目录make menuconfig配置

```cp target/linux/siflower/openwrt_config_a28_ac28 .config```
```make menuconfig```
注意：**如果make menuconfig选择时没有新版型选项，删除tmp/ 和 .config.old，再执行**
新建版型后如下

![menuconfig.png](/assets/images/4+32/menuconfig.png)

保存退出
将生成的config文件覆盖回target目录下的文件，使openwrt配置永久保存
`cp .config target/linux/siflower/openwrt_config_a28_ac28`
`make kernel_menuconfig`保存内核配置


## 3.6 开始编译
```./make.sh a28_ac28```
编译成功后会在根目录生成一个带版型名称版本号的镜像

![bin1.png](/assets/images/4+32/bin1.png)

如果不使用脚本编译，直接make，生成的镜像位于`/bin/siflower`目录下

![bin2.png](/assets/images/4+32/bin2.png)


# FAQ

- 编译问题：

![faq1.png](/assets/images/4+32/faq1.png)
```make tools/mklibs/clean```
然后重新编，可以使用make -j1 V=s

- 板型命名问题
在挂载squashfs文件系统时，使用原本命名的SF19A28-AC28板型编译，无法生成jffs2及squashfs类型bin文件，在`target/linux/siflower/image/Makefile`中增加debug日志，发现当板型中含有“-”时，识别到的板型为空，无法正确编译生成bin文件，更换为‘_'后可以识别，编译逻辑与18.06有很大区别

![faq2.png](/assets/images/4+32/faq2.png)
