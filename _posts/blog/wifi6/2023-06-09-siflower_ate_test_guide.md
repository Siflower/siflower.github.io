---
layout: post
title: SIFLOWER射频测试手册
categories: WiFi6
description: 介绍SIFLOWER方案手动射频测试
keywords: ate_cmd test
mermaid: true
---

# SIFLOWER射频测试手册

**目录**

* TOC
{:toc}


## 1 介绍

本文基于公司SIFLOWER芯片方案介绍Openwrt系统下手动及使用ATE工具测试产品射频性能，包括TX/RX两方面测试，帮助客户或使用者了解和评估产品射频性能。本文档只介绍command测试操作方法，至于对产品进行校准，请使用PCBA产测工具。

## 2 环境准备

|序号|设备名称|型号|用途|数量|备注|
|--|--|--|--|--|--|
|1|待测产品|任意|待测设备|1|待测板子须具备ATE测试功能的Openwrt环境|
|2|测试电脑|任意|用于测试操作|1||
|3|综测仪|任意|用于完成TX/RX测试|1|具备11a/b/g/n/11ac/11ax协议要求的双频综测仪|
|4|RF线缆|任意|用于搭建测试环境|5|频率要求支持到6G|
|5|串口线|任意|用于发送command控制板子|1||

## 3 SIFLOWER 协议命令参数说明

### 3.1 Band参数说明

|Band|wlan0|wlan1|
|--|--|--|
| 频段 | 5G | 2.4G |

### 3.2 Mode参数说明

|值|0|2|4|5|
|--|--|--|--|--|
|模式|11a/b/g|11n|11ac|11ax|

### 3.3 DataRate参数说明

- 当帧格式为11b时：

|值|0|1|2|3|
|--|--|--|--|--|
|DataRate（Mbps）|1|2|5.5|11|

- 当帧格式为11g时：

|值|4|5|6|7|8|9|10|11|
|--|--|--|--|--|--|--|--|--|
|DataRate（Mbps）|6|9|12|18|24|36|48|54|

- 当帧格式为11a时：

|值|0|1|2|3|4|5|6|7|
|--|--|--|--|--|--|--|--|--|
|DataRate（Mbps）|6|9|12|18|24|36|48|54|

- 当帧格式为11n时：

|值|0|1|2|3|4|5|6|7|
|--|--|--|--|--|--|--|--|--|
|DataRate（Mbps）|MCS0|MCS1|MCS2|MCS3|MCS4|MCS5|MCS6|MCS7|

- 当帧格式为11ac HT20MHz时：

|值|0|1|2|3|4|5|6|7|8|
|--|--|--|--|--|--|--|--|--|--|
|DataRate（Mbps）|MCS0|MCS1|MCS2|MCS3|MCS4|MCS5|MCS6|MCS7|MCS8|

- 当帧格式为11ac HT40/HT80 MHz时：

|值|0|1|2|3|4|5|6|7|8|9|
|--|--|--|--|--|--|--|--|--|--|--|
|DataRate（Mbps）|MCS0|MCS1|MCS2|MCS3|MCS4|MCS5|MCS6|MCS7|MCS8|MCS9|

- 当帧格式为11ax HT20/HT40/HT80/HT160 MHz时：

|值|0|1|2|3|4|5|6|7|8|9|10|11
|--|--|--|--|--|--|--|--|--|--|--|--|--|
|DataRate（Mbps）|MCS0|MCS1|MCS2|MCS3|MCS4|MCS5|MCS6|MCS7|MCS8|MCS9|MCS10|MCS11|

### 3.4 BandWidth参数说明

|值|0|1|2|3|4|
|--|--|--|--|--|--|
| BandWidth | 20MHz (11a/b/g) | 20MHz (11n/ac/ax) | 40MHz (11n/ac/ax) | 80MHz (11ac/ax) | 160MHz (11ax) |

## 4 SIFLOWER 协议命令组成及说明

本部分包括TX与RX测试两部分，命令为上文3小节所描述的参数组成。

### 4.1 TX 测试命令

| 命令头部 | Freq | Center-freq | BW | Mode | DataRate | Power_level | 命令尾部 |
|--|--|--|--|--|--|--|--|--|--|
| ate_cmd Band fastconfig | -f 频率 | -c 中心频率 | -w 带宽 | -m 模式 | -i 速率 | -p gain | -y |

如上TX命令结构:
- Band 代表 wlan0/wlan1。
- gain 代表TX功率发射 gain 等级，0~127 可选;

如想要控制板子做 11ac 40M ，5500信道，速率为MCS9，增益等级为15的短TX power，命令结构如下：
`ate_cmd wlan0 fastconfig -f 5500 -c 5510 -w 2 -m 4 -i 9 -p 15 -y`

如控制板子停止TX，使用如下指令：
`ate_cmd wlan0 fastconfig -q`

### 4.2 RX测试命令

| 命令头部 | Freq | Center-freq | BW | 命令尾部 |
|--|--|--|--|--|--|
| ate_cmd Band fastconfig | -f 频率 | -c 中心频率 | -w 带宽 | -r |

如上RX命令结构：
- Band 代表wlan0/wlan1。

如综测仪发射20MHz，中心频率为5180的信号，板端通过以下指令开始接收和查看RX接收情况
`ate_cmd wlan0 fastconfig -f 5180 -c 5180 -w 0 -r`

查看结果
`ate_cmd wlan0 fastconfig -R `

结果读取成功后会收到如下log：

```
receive 20M OK = %d, receive 40M OK = %d, receive 80M OK = %d, receive 160M OK = %d
```

- 其中，**receive 20M OK = %d** 中 %d 的值表示带宽为 20MHz 时收到的包；
**receive 40M OK = %d** 中 %d 的值表示带宽为 40MHz 时收到的包；
**receive 80M OK = %d** 中 %d 的值表示带宽为 80MHz 时收到的包；
**receive 160M OK = %d** 中 %d 的值表示带宽为 160MHz 时收到的包；

控制板子停止RX，使用如下指令：
`ate_cmd wlan0 fastconfig -k`

### 4.3 初始化天线配置命令

| 命令头部 | Antenna |
| :-: | :-: |
| ate_cmd Band fastconfig | -a 天线 |

**Band** 代表 **wlan0/wlan1**。
**天线**可选取值为 **0/1/2/3**。
其中，**wlan0** 代表控制 **5G** 的两路天线；**wlan1** 代表控制 **2.4G** 的两路天线。
**-a 0** 代表两路天线都关闭；
**-a 1** 代表初始化一路天线，即打开一路关闭二路；
**-a 2** 代表初始化二路天线，即打开二路关闭一路；
**-a 3** 代表两路天线都打开。

目前板子 WiFi 起来之后，默认四路天线都处于打开状态。可根据实际需求使用上述指令初始化单双路天线。如：

```
// 初始化 5G 1路
ate_cmd wlan0 fastconfig -a 1
// 初始化 2.4G 2路
ate_cmd wlan1 fastconfig -a 2
```

## 5 SIFLOWER 方案手动测试

测试拓扑如下：

![Test_topolgy.png](/assets/images/wifi6/ate_test/Test_topolgy.png)

如图：
- PC1通过网线连接综测仪；
- 综测仪与板子天线之间用RF线缆连接；
- PC2通过串口线连接板子，用于控制板子。

### 5.1 XO校准（频偏校准）

完成上面的准备工作后，测试开始。然后观察串口打印信息，等待板子启动完毕后在串口工具界面分别输入下列指令启动wifi：

```
sfwifi reset fmac
ifconfig wlan0 up
ifconfig wlan1 up
```
TODO ：初始化指令、切天线指令

1. 输入下面的命令发送调制信号5180.
`ate_cmd wlan0 fastconfig -f 5180 -c 5180 -w 1 -m 2 -i 7 -p 12 -y`

2. 使用仪器测量频偏.

3. 输入下面的命令调整频偏，通过仪器可观察变化。其中value的值需要不断调整修改，不同的值会影响频偏的变化，然后观察频偏结果，当频偏小于5ppm后，进行步骤4
>调整策略:
首先设置一个初始值0x00  则从0x00开始，然后每次在之前的基础上增加0x01. 比如第一次设置0x02,第二次0x03,第三次0x04(最大到0x7f)....直至频偏到正常范围
也可以从如0x40开始，往上下调整(范围0x0~0x7f)....直至频偏到正常范围

`ate_cmd wlan0 fastconfig -O value`
 **使用指令时value参数值，必须是用10进制，需要转换一下,0x14 则value写20**
在配置XO值的同时，会将 value 值以16进制的形式存放到 factory 分区，同时也会存放在 /lib/firmware/siwifi_settings.ini 文件内。重启板子，XO值生效。

4. 输入下面的命令停止发送，完成XO校准。
`ate_cmd wlan0 fastconfig -q`

5. 2G 和5G需要单独校准XO，即按照上述步骤重复操作2.4G，再储存另外一个XO值。

### 5.2 SIFLOWER 方案手动TX测试

**测试流程**
如上测试拓扑搭建完成后，给板子上电，然后观察串口打印信息，等待板子启动完毕后在串口工具界面分别输入下列指令启动wifi：

```
sfwifi reset fmac
ifconfig wlan0 up
ifconfig wlan1 up
```

待上述指令执行完毕后，在串口工具界面输入如4.1小节的TX Start测试命令，板子即进入TX工作模式

借助综测仪自带工具，如极致汇仪的WLAN Meter查看TX发射情况，正常发送如下图：

![TX_Result.png](/assets/images/wifi6/ate_test/TX_Result.png)

注意：当要控制板子由一个TX状态切换到另一个TX状态，如查看另一个信道或频率的TX情况时，请先在原TX状态下停止板子发射，更改TX参数后再在控制板子发射。

### 5.3 SIFLOWER 方案手动RX测试

- 板子启动后，在串口工具界面分别输入下列指令启动wifi：

```
sfwifi reset fmac
ifconfig wlan0 up
ifconfig wlan1 up
```

- 待上述指令执行完毕后，使用如下命令让板子开始进入接收状态
 `ate_cmd wlan0 fastconfig -f 5180 -c 5180 -w 0 -r`
- 此时使用综测仪发送要测试的wifi信号
- 使用如下指令查看接收结果
 `ate_cmd wlan0 fastconfig -R`

注意：当要控制板子由一个RX状态切换到另一个RX状态，如查看另一个信道或频率的RX情况时，请先在原RX状态下停止板子接收，更改RX参数后再在控制板子进入接收状态。

- 接收停止指令
`ate_cmd wlan0 fastconfig -k`
