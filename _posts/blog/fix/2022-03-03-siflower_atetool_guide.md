---
layout: post
title: SIFLOWER ATETOOL使用手册
categories: PRODUCE
description: 介绍SIFLOWER方案ATE自动测试工具使用
keywords: ate_cmd test
mermaid: true
---

# SIFLOWER ATETOOL使用手册

**目录**

* TOC
{:toc}

## 1 介绍

本文基于公司SIFLOWER芯片方案介绍Openwrt系统下使用SFWiFiCalibrationTool工具测试产品射频性能，功能涵盖晶振校准 测试，TX校准测试，TX校验测试，RX校验测试，TX/RX的单发校验测试几个方面，并适配KeySight，LitePoint以及Itest三个厂 家的仪器，本文档只介绍工具使用方法，对于SIFLOWER方案的ATE Command不做过多说明

## 2 环境准备

|序号|设备名称|型号|用途|数量|备注|
|--|--|--|--|--|--|
|1|待测产品|任意|待测设备|1|待测板子须具备ATE测试功能的Openwrt环境|
|2|测试电脑|任意|用于测试操作|1||
|3|综测仪|任意|用于完成TX/RX测试|1|具备11a/b/g/n/11ac协议要求的双频综测仪|
|4|RF线缆|任意|用于搭建测试环境|5|频率要求支持到6G|
|5|串口线|任意|用于发送command控制板子|1||
|6|功分器|任意|用于搭建测试环境|1|要求频率支持2G-6G|

## 3 测试拓扑搭建

测试环境搭建如下图：

![atetool_1](/assets/images/atetool_image/atetool_1.png)

## 4 工具下载

由于工具比较大，请前往百度网盘下载：（链接：https://pan.baidu.com/s/14LajR5d10XVy52ZDWXfsiQ 提取码：SiFi)


## 5 工具介绍

在提供的工具目录中有SFWiFiCalibrationTool.exe文件，双击打开即可。后续更新也是直接替换此exe
干净未使用过的电脑会提示缺少相应的dll文件，此时 将help文件夹下的dll文件拷贝至C:\Windows\SysWOW64目录下（64位电脑系统），此外因工具适配NI仪器，需要安装help目 录下的visa540_runtime.exe，安装完成即可正常打开测试工具。

## 5.1 工具界面

工具界面如下图：

![atetool_2](/assets/images/atetool_image/atetool_2.png)

工具界面主要包括三个方面的功能：
1)晶振校准测试。
2)MacByPass方式的TX/RX测试。
3)非MacByPass方式的TX/RX测试。

MacByPass 方式的TX/RX测试包括macbypasstxandrx，SetPower，MacByPassManualTest，以及MacBypassTXCalibration四个page，每个page完成相应功能测试。

### 5.1.1 频偏校准测试

![atetool_3](/assets/images/atetool_image/atetool_3.png)

图中编辑框0x1010表示设置的频偏校准初始值，可以更改；
SET_XO_Value 按钮表示单击后按照编辑框中频偏校准初始值作为XO值；
Save_XO_Value按钮表示单击后将编辑框中的频偏校值写入板子；
Start_XO按钮表示以配置文件中的设定值为初始值开始进行XO校准，校准值PPM在±2.5范围后，将最后的XO值写入板子；
Stop_XO表示单击后停止XO校准

注意因频偏校准时采用5G信号作为调制信号进行频偏调整，故实际测试时需要先初始化5G天线，进入ate模式并开启板子ate_server。
在串口界面键入ate_init hb1为初始化5G天线1，在串口界面键入ate_init hb2为初始化5G天线2，二者任意初始化一根即可。
初始化完成在串口界面键入ate_server，使板子进入监听状态，此时工具就可以通过网线连接并控制板子。

**示例：**
- 1,按照第三节所示搭建好测试环境，板子上电后在串口界面键入ate_init hb1,待初始化完成，如下图所示：

![atetool_4](/assets/images/atetool_image/atetool_4.png)

注意：需要测试那一路天线就初始化对应的天线

```
ate_init lb1 初始化2.4G 天线1路
ate_init lb2 初始化2.4G 天线2路
ate_init hb1 初始化5.8G 天线1路
ate_init hb2 初始化5.8G 天线2路
```
- 2,待hb1初始化完成，在串口界面键入ate_server,使板子进入监听状态，如下图所示:

![atetool_5](/assets/images/atetool_image/atetool_5.png)

注意：如果需要切换另外一根天线

```
ctrl + c 退出ate_server
然后重新执行ate_init 天线
```

- 3,打开工具

  按照Connect DUT->仪器选择->Start XO的步骤进行频偏校准即可

  ![atetool_6](/assets/images/atetool_image/atetool_6.png)


### 5.1.2  MacByPass方式的TX/RX测试

此模块如下图四个部分：

- macbypasstxandrx界面：

  ![atetool_7](/assets/images/atetool_image/atetool_7.png)

- setpower界面：

  ![atetool_8](/assets/images/atetool_image/atetool_8.png)

- MacByPassManualTest界面：

  ![atetool_9](/assets/images/atetool_image/atetool_9.png)

- MacBypassTXCalibration界面

  ![atetool_10](/assets/images/atetool_image/atetool_10.png)

此模块包含以上macbypasstxandrx，power，MacByPassManualTest，以及MacBypassTXCalibration四个page，每个page完成相应功能测试，下面对每个页面操作及功能进行分别介绍

#### 5.1.2.1  macbypasstxandrx功能页

界面的一大优势是操作方便，可在工具界面根据任选的channel，mode，rate等信息搭配所采用仪器快速完成一个或多个测试项测试，方便快速测试验证板子RF性能。
另外本页面增加了测试环境的线损自动测试功能，测试完成保存至工具目录下 sf_setup文件夹的wifi_atten_dut_1.txt文件中，还提供工具页面手填线损的功能。

- TxStart

  任选功能channel，mode，rate等参数操作，如下图：

  ![atetool_11](/assets/images/atetool_image/atetool_11.png)

  - 板子上电以后在串口界面键入ate_init lb1,天线初始化完成在串口接面键入ate_server使板子进入监听状态
  - 单击Connect DUT按钮，连接板子，正常连接后，工具log区域会打印connect ok
  - 单击2.4G单选按钮，测试5.8G时勾选5.8G单选按钮
  - 单击channel24g后按住键盘Ctrl键，就可以在右侧channel栏进行多个信道选择，并点击下方choose按钮进行确认，bw与 rate选项操作一致
  - channel，bw，mode，rate选项完成后在InstrumentOptions部分选择所使用的仪器
  - 如使用工具界面手填线损，则需要在2G_Cableloss或者5G_Cableloss编辑框填入相应的线损值；不使用工具界面手填线损，则自动调用wifi_atten_dut_1.txt文件里相应的信道线损值进行补偿

完整的操作步骤及界面如下图所示，其余功能测试操作步骤与此一致

  ![atetool_12](/assets/images/atetool_image/atetool_12.png)

最终测试结果会保存到测试工具目录下，如果是测试2.4G，则保存在LB_loop_send_check_macbypass.txt中，如果是测试5.8G，则会保存在HB_loop_send_check_macbypass.txt中，内容如下图所示：

  ![atetool_13](/assets/images/atetool_image/atetool_13.png)

- RxStart

  - 按照第三节所示搭建好测试环境，板子上电开机完成，在串口界面键入ate_init lb1,天线初始化完成在串口接面键入 ate_server使板子进入监听状态
  - 单击Connect DUT按钮，连接板子，正常连接后，工具log区域会打印connect ok！
  - 单击2.4G单选按钮，如测试5.8G时则勾选5.8G单选按钮
  - 单击channel24g后按住键盘Ctrl键，就可以在右侧channel栏进行多个信道选择，并点击下方choose按钮进行确认，bw与 rate选项操作一致
  - channel，bw，mode，rate选项完成后在InstrumentOptions部分选择所使用的仪器
  - 如使用仪器量测的线损作为补偿，不需要在2G_Cableloss或者5G_Cableloss编辑框填入相应的线损值，工具从线损配置文件件中读取。

完整的操作步骤及界面如下图所示，其余功能测试操作步骤与此一致：

![atetool_14](/assets/images/atetool_image/atetool_14.png)

最终测试结果会保存到测试工具目录下，如果是测试2.4G，则保存在LB_loop_rx_macbypass.txt中，如果是测试5.8G，则会保存在HB_loop_rx_macbypass.txt中，内容如下图所示：

![atetool_15](/assets/images/atetool_image/atetool_15.png)

- 线损测试

打开工具后将测试RF线回环接上仪器RF1和RF2,如下图所示：

![atetool_16](/assets/images/atetool_image/atetool_16.png)

在工具界面选择一起后单击CablelossTest按钮开始线损测试，如下图

![atetool_17](/assets/images/atetool_image/atetool_17.png)

测试完成后，线损值会自动保存到sf_setup文件夹的wifi_atten_dut_1.txt文件中


#### 5.1.2.2  SetPower功能页

本界面依据界面设定直接往factory分区写入gain值，完成单个点gain值写入，板子上电启动完成后在串口中键入ate_server使板子进入监听状态，打开测试工具，操作步骤如下图所示：

![atetool_18](/assets/images/atetool_image/atetool_18.png)

#### 5.1.2.3  MacByPassManualTest功能页

本页依据界面设定测试板子单点的TX/RX性能，需要手动设置并结合测试仪器使用。

- 手动TX测试

  TX测试部分操作步骤如下：板子上电启动完成，在串口界面键入ate_init lb1待天线初始化完成后在串口界面键入ate_server使板子进入监听状态，打开测试工具操作如下图：

  ![atetool_19](/assets/images/atetool_image/atetool_19.png)

  此时进入仪器界面即可查看到板子的发送功率，EVM，ppm等信息。

- 手动RX测试

  RX测试部分操作步骤如下：板子上电启动完成，在串口界面键入ate_init lb1待天线初始化完成后在串口界面键入ate_server使板子进入监听状态，打开测试工具操作如下图操作，待开启RX测试后，设置仪器端VSG的对应波形文件和发包数量，以及发包能量：

  ![atetool_20](/assets/images/atetool_image/atetool_20.png)

  **注意：**
  **1，RX测试需使测试板子置于屏蔽环境，排除干扰，（排除干扰方法开启RX测试时，在仪器没有发包之前，工具界面打印的receive data要求为0或者个位数增长，如有大量数据增长则表示环境有干扰），工具界面开启RX测试在前，仪器端发包在后，二者顺序不可颠倒。**
  **2，工具界面PacketNO表示发包数量，其数字要求与仪器发包数量相等。**
  **3，RX判断标准：要求receive data接近或等于发包数量，fcs_ok占发包数量的92%或以上（11b模式），其余模式要求fcs_ok占发 包数量的90%或以上,**

#### 5.1.2.3  MacBypassTXCalibration功能页

本界面完成TX校准测试，TX校验测试，RX校验测试，以及测试环境线损测试四个功能。
搭配极致汇仪与莱特波特两种仪器，配合测试工具同级目录中sf_setup文件下的三个配置文件完成各自功能项目的测试，其中wifi_atten_dut_1.txt文件保存线损值；
test_flow.txt文件保存测试信道及目标功率设定;
test_txpower_index.txt保存测试项的初始gain值以及各速率的power偏移值;

- TX校准测试

本部分完成对待测板进行校准，并将校准pass成功后的校准值写入factory分区。参照第三节所示测试拓扑搭建测试环境，板子上电开机完成在串口界面键入ate_server使板子进入监听状态，打开测试工具后参照下图步骤所示对板子进行校准：

![atetool_21](/assets/images/atetool_image/atetool_21.png)

校准完成后测试log会保存在测试工具同级目录下的TX_Calibration_Test.txt文件中，并将校准表的值打印到log中，如下图所示：

![atetool_22](/assets/images/atetool_image/atetool_22.png)

- TX校验测试

本部分依据配置文件配置的测试项完成对校准后的待测板进行校验，主要验证校准写入校准值的准确性，参照第三节所示测试拓扑搭建测试环境，板子上电开机完成在串口界面键入ate_server使板子进入监听状态，打开测试工具后参照下图步骤所示对板子进行校验：

![atetool_23](/assets/images/atetool_image/atetool_23.png)

校验完成后测试log会保存在测试工具同级目录下的TX_Verify_Test.txt文件中，如下图所示：

![atetool_24](/assets/images/atetool_image/atetool_24.png)

- RX校验测试

本部分依据配置文件配置的测试项对待测板进行RX遍历校验，参照第三节所示测试拓扑搭建测试环境，板子上电开机完成在串口界面键入ate_server使板子进入监听状态，打开测试工具后参照下图步骤所示对板子进行校验：

![atetool_25](/assets/images/atetool_image/atetool_25.png)

校验完成后测试log会保存在测试工具同级目录下的RX_Verify_Test.txt文件中，如下图所示：

![atetool_26](/assets/images/atetool_image/atetool_26.png)