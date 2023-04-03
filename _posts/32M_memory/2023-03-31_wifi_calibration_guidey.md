---
layout: post
title: 校准方案说明
categories: 32M_MEMORY
description: WiFi校准
keywords: calibration
topmost: true
mermaid: true
---

* TOC
{:toc}


# 1. 介绍

本文针对SIFLOWER芯片方案的WiFi校准相关指令进行说明，适用于生产校准开发相关人员。


# 2. 校准方案

系统下校准方案可通过ate_cmd校准指令手动校准，或者使用ate_server搭配ate_tool工具自动校准。
校准的最终目的都是将校准值写入factory分区对应位置，供WiFi驱动加载使用。

## 2.1 SIFLOWER 协议命令参数说明

### 2.1.1 Band参数说明

| Band  | wlan0  | wlan1 |
| :-- | :-- | :-- |
| 频段 | 2.4G | 5G |

### 2.1.2 Mode参数说明

| 值 | 0 | 2 | 4 |
| :-- | :-- | :-- | :-- |
| 模式 | 11a/b/g | 11n | 11ac |

### 2.1.3 DataRate参数说明

- 当帧格式为11b时：

| 值 | 0| 1 | 2 | 3 |
| :-- | :-- | :-- | :-- | :-- |
| DataRate（Mbps）| 1 | 2 | 5.5 | 11 |

- 当帧格式为11g时：

| 值 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| DataRate（Mbps） | 6 | 9 | 12 | 18 | 24 | 36 | 48 | 54 |

- 当帧格式为11a时：

|值 |0 |1 |2 |3 |4 |5 |6 |7 |
|:--|:--|:--|:--|:--|:--|:--|:--|:--|
|DataRate（Mbps） |6 |9 |12 |18 |24 |36 |48 |54 |

- 当帧格式为11n时：

|值 |0 |1 |2 |3 |4 |5 |6 |7 |
|:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |
|DataRate（Mbps） |MCS0 |MCS1 |MCS2 |MCS3 |MCS4 |MCS5 |MCS6 |MCS7 |

- 当帧格式为11ac HT20MHz时：

|值 |0 |1 |2 |3 |4 |5 |6 |7 |8 |
|:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |
|DataRate（Mbps） |MCS0 |MCS1 |MCS2 |MCS3 |MCS4 |MCS5 |MCS6 |MCS7 |MCS8 |

- 当帧格式为11ac HT40/HT80 MHz时：

|值 |0 |1 |2 |3 |4 |5 |6 |7 |8 |9 |
|:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |
|DataRate（Mbps） |MCS0 |MCS1 |MCS2 |MCS3 |MCS4 |MCS5 |MCS6 |MCS7 |MCS8 |MCS9 |

### 2.1.4 BandWidth参数说明

|值 |0 |1 |2 |3 |
|:-- |:-- |:-- |:-- |:-- |
|BandWidth |20MHz（11a/b/g） |20MHz(11n/ac) |40MHz(11n/ac) |80MHz(11ac) |

### 2.1.5 Frame BandWidth参数说明

|值 |0 |1 |2 |3 |
|:-- |:-- |:-- |:-- |:-- |
|BandWidth |20MHz（11a/b/g） |20MHz(11n/ac) |40MHz(11n/ac) |80MHz(11ac) |

注意：测试时BandWidth参数与Frame BandWidth参数保持一致。

### 2.1.6 SIFLOWER 协议命令组成及说明

本部分包括TX与RX测试两部分，命令为上文2.1小节所描述的参数组成。

- TX 测试命令

| 命令头部 |Tx_Frame_longth |Freq |Center-freq |BW |Frame-BW |Mode |DataRate |Guard Interval |Power_level |命令尾部 |
|:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |:-- |
|ate_cmd Band fastconfig |-l 发射帧长度 |-f 频率 |-c 中心频率 |-w 带宽 |-u 帧带宽 |-m 模式 |-i 速率 |-g 传输间隔 |-p gain |-y |

如上TX命令结构:
	- Band代表wlan0/wlan1;
	- 传输间隔代表Short GI或者Long GI,分别用1和0表示;
	short gi:1
	long gi:0
	- gain代表TX功率发射gain等级，0~15可选;

如想要控制板子做11ac 40M ，5500信道，速率为MCS9，增益等级为15的短距发射，命令结构如下：
`ate_cmd wlan1 fastconfig -l 1024 -f 5500 -c 5510 -w 2 -u 2 -m 4 -i 9 -g 0 -p 15  -y`
如控制板子停止TX，使用如下指令：
`ate_cmd wlan1 fastconfig -q`
**注意：停止2.4G与5G 时wlan参数不同。**

- RX测试命令
-
|命令头部 |Freq |Center-freq |BW |Frame-BW |命令尾部 |
|:-- |:-- |:-- |:-- |:-- |:-- |
|ate_cmd Band fastconfig |-f 频率 |-c 中心频率 |-w 带宽 |-u 帧带宽 |-r |

如上RX命令结构：
	- Band代表wlan0/wlan1;
	- 带宽与帧带宽相一致。

如综测仪发射20MHz，中心频率为5180的信号，板端通过以下指令开始接收和查看RX接收情况
`ate_cmd wlan1 fastconfig -f 5180 -c 5180 -w 0 -u 0 -r`
查看结果
`ate_cmd wlan1 fastconfig -R `
控制板子停止RX，使用如下指令：
`ate_cmd wlan1 fastconfig -k`
**注意：停止2.4G与5G 时wlan参数不同。**

## 2.2 ate_cmd手动校准

ate_cmd作为siflower方案射频测试的控制命令，可以基于此命令对射频性能进行测试，也可以基于此命令的强大功能开发校准工具。

- 进入ate模式
在使用ate_cmd指令之前，需要先使用ate_init指令进行初始化，让板子进入ate模式，也可以在初始化时增加参数对天线进行控制。
```
ate_init //默认打开四路天线
ate_init lb1 //打开lb1
ate_init lb2 //打开lb2
ate_init hb1 //打开hb1
ate_init hb2 //打开hb2
```

- 退出ate模式
当各项测试完成后，可以使用 `sfwifi reset fmac` 退出，恢复正常模式。

### 2.2.1 频偏校准

1，在系统下使用ate_cmd命令控制板子做2.4G 或者5G 的TX常发;
2，通过仪器获取TX结果，并将获取到的ppm与设定范围（-2.5 +2.5）做对比，在门限范围则停止XO校准，将此XO值写入板子
如超出设定范围，则通过ate_cmd调整xo值，如此重复，直至ppm满足设定范围（-2.5 +2.5）。
具体操作，举例如下：
1，在ate_init进入ate模式后，选择任意信道模式发送调制信号如：2412_11n_20M_mcs7
`ate_cmd wlan0 fastconfig -l 1024 -f 2412 -c 2412 -w 1 -u 1 -m 2 -i 7 -g 0 -p 15 -y`
2，使用综测仪测量频偏值
3，输入下面的命令调整频偏，通过仪器可观察变化。
`ate_cmd wlan0 fastconfig -O value  //value的范围是0x00-0xff(0~255)`
注意：使用 -O 指令时value参数值，必须是用10进制，需要转换一下。其中value的值需要不断调整修改，不同的值会影响频偏的变化，然后观察频偏结果，当频偏满足±2.5ppm后，进行步骤4。
4，保存调整好的频偏值
输入下面的命令储存校准值,比如校准好频偏后value结果为31，则需要将31转化为0x1f写入
`ate_cmd save 0x1f1f`
5，输入下面的命令停止发送，完成XO校准
 `ate_cmd wlan0 fastconfig -q`

### 2.2.2 手动TX校准

- 2.4G TX校准指令
指令通过ate_cmd加信道/模式/带宽/速率/等参数组合而成，可以根据需要测试的信道模式来进行发送，然后通过综测仪解析结果。
如发送ANT1_2412信道11n,40M带宽，mcs7信号的指令为
`ate_cmd wlan0 fastconfig -l 1024 -f 2412 -c 2412 -w 2 -u 2 -m 2 -i 7 -g 0 -p 15 -y`
这个指令可以控制板子2.4G常发，如果仪器测量的结果不满足需求，则调整-p后面的gain值参数来调整发射功率

- 2.4G 停止发送指令
`ate_cmd wlan0 fastconfig -q`
- 5G TX校准指令
指令通过ate_cmd加信道/模式/带宽/速率/等参数组合而成，可以根据需要测试的信道模式来进行发送，然后通过综测仪解析结果。
如发送ANT1_5180信道11ac,80M带宽，MSC9信号的指令为
`ate_cmd wlan1 fastconfig -l 1024 -f 5180 -c 5210 -w 3 -u 3 -m 4 -i 9 -g 0 -p 5 -y`
这个指令可以控制板子5G常发，如果仪器测量的结果不满足需求，则调整-p后面的power_idx参数来调整发射功率
- 5G停止发送指令
 `ate_cmd wlan1 fastconfig -q`

### 2.2.3 校准步骤

- 1，板子上电，系统启动完成后使用ate_init初始化（单路初始化，单路依次校准)；完成晶振校准写入XO值，并且写入WiFi version（写**V4**，使用双路天线校准表）。
- 2，板子通过ate_cmd指令进行TX发送；
- 3，仪器读取测试结果与evm标准和目标功率门限做比较(首先判断EVM，在evm满足的前提下，调整gain值满足目标功率门限)，如满足设定则板端保存当前测试项目的gain值作为校准值；如不满足，则调整-p 参数后再次重复步骤2，直至获取的power和EVM的测试结果与设定相符，满足后保存此gain值；(具体判断由PC端工具控制仪器实现，目标功率以及目标频偏也是工具这边设置）
- 4，将测试pass的gain值进行不同的offset补偿写入未校准的其它速率；
- 5，copy已经偏移完成的信道校准值到同band未测试的信道；
- 6，完成所有信道校准值，通过ate_cmd指令写入FLASH对应分区。

## 2.3 atetool工具校准

上述2.2部分是手动输入ate_cmd指令进行操作，目前提供的PC端atetool工具，便是基于此命令做成了自动化工具。

### 2.3.1 环境搭建

|序号 |设备名称 |型号 |用途 |数量 |备注 |
|:-- |:-- |:-- |:-- |:-- |:-- |
|1 |待测产品 |任意 |待测设备 |1 |待测板子须具备ATE测试功能的Openwrt环境 |
|2 |测试电脑 |任意 |用于测试操作 |1 | |
|3 |综测仪 |任意 |用于完成TX/RX测试 |1 |具备11a/b/g/n/11ac协议要求的双频综测仪 |
|4 |RF线缆 |任意 |用于搭建测试环境 |5 |频率要求支持到6G |
|5 |串口线 |任意 |用于发送command控制板子 |1 | |

测试电脑通过网线分别与待测板子及综测仪连接（注意对应网卡的网段设置）。
综测仪通过RF线缆与待测板子天线一一连接。
通过串口线将待测板子与测试电脑连接，用于控制板子。

### 2.3.2 校准步骤

环境准备完毕，给板子上电，待板子完全启动后，先输入`ate_init`进入ate模式，而后输入`ate_server`进入校准模式。
PC端打开atetool工具，点击`Connect DUT`，待工具界面显示“receive DUT: connect ok!”后，
点击`MacBypassTXCalibration`，进入自动化校准界面
选择对应的仪器（`LitePointDevice`或`ItestDevice`)
而后点击`TxCalibrationTest`，即可开始自动化校准
静待校准结束，校准完成后板端串口会打印出校准表
若要中断校准，点击`STOP_TEST`；若要断开与板端的连接，点击`DisConnectDUT`。

![calibration.png](/assets/images/4+32/calibration.png)