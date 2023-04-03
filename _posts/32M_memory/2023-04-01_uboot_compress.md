---
layout: post
title: uboot压缩介绍
categories: 32M_MEMORY
description: uboot压缩方案说明
keywords: uboot
topmost: true
mermaid: true
---

* TOC
{:toc}


# 1. uboot镜像压缩

## 1.1 uboot镜像压缩方式

1）在sf_make.sh编译脚本中，通过判断compress参数是否置为true，决定是否采用压缩算法。当该参数设置为true时，将uboot镜像文件进行压缩操作，利用lzma压缩算法进行压缩。

2）当uboot镜像文件被压缩后，需要在spl中读取到uboot镜像文件后，进行解压缩操作：

- 判断镜像文件header中的ih_comp判断是否进行压缩。该参数为IH_COMP_NONE （0）时，表示无压缩；该参数为 IH_COMP_LZMA（3）时，表示使用lzma压缩算法进行了压缩。

- 未采用压缩时，直接将镜像文件读取到启动地址，进行uboot启动；采用压缩时，将镜像文件读取到DDR之后，进行解压缩操作，镜像解压到uboot启动位置，并进行uboot启动。

# 2. uboot分区划分

## 2.1 原先分区划分


| 分区 | 开始地址 | 大小 |
| ------ | ------ | ------ |
| SPL | 0 | 0x20000 |
| uboot | 0x20000 | 0x60000 |
| uboot_env | 0x80000 | 0x10000 |
| factory | 0x90000 | 0x10000 |
| firmware | 0xa0000 | 0xf60000 |

## 2.2 压缩后分区划分


| 分区 | 开始地址 | 大小 |
| ------ | ------ | ------ |
| SPL | 0 | 0x8000 |
| uboot | 0x8000 | 0x18000 |
| firmware | 0x20000 | 0x3de000 |
| factory | 0x3fe000 | 0x2000 |

## 2.3 uboot CMD删减
	在uboot命令行中支持很多的CMD，为了减少uboot镜像文件大小，删减了部分不必要的命令，包含以下这些：


| 序号 | 命令 |
| ------ | ------ |
| 1 | bdinfo |
| 2 | coninfo |
| 3 | bootd |
| 4 | bootelf/boottvx |
| 5 | go |
| 6 | run |
| 7 | iminfo |
| 8 | imxtract |
| 9 | env export |
| 10 | env import |
| 11 | editenv |
| 12 | saveenv |
| 13 | env exists |
| 14 | md/mm/nm/mw/cp/cmp/base/loop |
| 15 | crc32 |
| 16 | dm |
| 17 | echo |
| 18 | spl update |
| 19 | itest |
| 20 | source |

