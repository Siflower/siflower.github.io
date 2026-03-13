---
layout: post
title: SIFLOWER ATETOOL使用手册
categories: WiFi6
description: 介绍SIFLOWER方案ATE自动测试工具使用
keywords: ate_tool test
mermaid: true
---

# SIFLOWER ATETOOL使用手册

**目录**

* TOC
{:toc}


## 1 介绍

本文基于公司SIFLOWER芯片方案介绍Openwrt系统下使用SFWiFiCalibrationTool工具测试产品射频性能，功能涵盖晶振校准 测试，TX校准测试，TX校验测试，RX校验测试，TX/RX的单发校验测试几个方面。本文档只介绍工具使用方法，对于SIFLOWER方案的ATE Command不做过多说明。使用工具旨在简化使用人员操作复杂的命令组合，达到同样的测试效果。

## 2 环境准备

|序号|设备名称|型号|用途|数量|备注|
|--|--|--|--|--|--|
|1|待测产品|任意|待测设备|1|待测板子须具备ATE测试功能的Openwrt环境|
|2|测试电脑|任意|用于测试操作|1||
|3|综测仪|任意|用于完成TX/RX测试|1|具备11a/b/g/n/11ac/11ax协议要求的双频综测仪|
|4|RF线缆|任意|用于搭建测试环境|5|频率要求支持到6G|
|5|串口线|任意|用于发送command控制板子|1||

## 3 SIFLOWER ATETOOL工具获取

获取最新的ATE TOOL工具
内部链接：https://drive.weixin.qq.com/s?k=AJYAmAeaAAkHMq11Ic
客户使用请联系矽昌提供。

## 4 SIFLOWER ATETOOL工具使用

本工具支持基于SIFLOWER芯片方案的ATE测试，包括TX/RX两个部分：
工具界面如下：

![tool-ui.png](/assets/images/wifi6/ate_tool/tool-ui.png)


### 4.1 SIFLOWER ATETOOL工具配置文件说明

sf_config.txt文件配置说明：

|字符串|值|说明|
|--|--|--|
|wifi_evolution|6|配置为6，进行wifi6相关测试，配置为其他值进行非wifi6相关测试|
|Dut_ip|192.168.1.1|待测板的IP地址|
|Instrument_Module|1/2|配置为1表示使用iqXstream-5G模组A;配置为2表示使用iqXstream-5G模组B|
|Instrument_ip|192.168.100.254|测试仪表IP地址|
|xo_default_value|0x35|频偏校准初始值|
|Serial_port|15|串口继电器串口号|
|rfport|2/3/4/5|使用的iqXstream-5G RF端口设置，分别对应RF1A/B,RF2A/B,RF3A/B,RF4A/B|
|txpower_idx_2g|31|遍历测试时初始gain值|
|txpower_idx_5g|31|遍历测试时初始gain值|
|fullpacket|0/1|TX测试时是否开全包.0:半包，1：全包|
|IQComp|0/1|是否开启仪表IQComp配置，0：不开，1：开启|
|txdebug|0/1|TX测试时是否开启debug功能，开启后测试异常时会停止保留状态|
|txretry|0/1|TX测试时是否先尝试性控制板子TX，判断EVM正常后测试继续进行|
|frame_cnt|1000|RX测试时仪表的发包数量|
|Correction_Freq|0/1/2/3/4|仪表的Correction_Freq配置，分别对应AUTO，STF，LTF，DATA，SIG|
|testcount|1000|进行单项压力测试时设置的测试次数|
|Infinitetest|0/1|是否进行压力测试，0，不做压力测试，配置为1；进行testcount设置的测试次数进行测试
|checkevm|0/1|TX测试时是否判断EVM，配置为1且EVM大于-20时，测试停止|

### 4.2 SIFLOWER ATETOOL工具TX测试

参照SIFLOWER方案手动测试搭建测试环境，环境搭建好以后给板子上电，待板子启动完毕，在串口工具界面输入“ifconfig br-lan 192.168.1.1”，修改板子的ip为192.168.1.1，在原有测试拓扑上增加一根网联连接PC2与待测板子LAN口，并将PC2网卡的IP地址设成“192.168.1.xx”网段，通过PC端ping -t 192.168.1.1来查看PC2与板子连接情况，待连接正常后，在串口界面输入依次输入`sfwifi reset fmac`和`ifconfig wlan0 up`、`ifconfig wlan1 up`指令初始化测试环境，接着输入`ate_server`指令，使板子进入如下图的SFATETESTTOOL测试模式，此外在PC2上打开SIFLOWER ATE TOOL测试工具。

![ate_server.png](/assets/images/wifi6/ate_tool/ate_server.png)

板子ATE初始化完成后点击工具界面左上角Connect DUT，工具即与板子建立连接，在工具右下角的log区域会提示工具与板子连接情况：如下图：

![connect_ok.png](/assets/images/wifi6/ate_tool/connect_ok.png)

当连接异常时工具会弹框提示连接失败，如下图示：

![connect-fail.png](/assets/images/wifi6/ate_tool/connect-fail.png)

正常连接后，依次从左到右选择如下图所示的参数选项，单击工具TxStart按钮即开始控制板子进行TX测试。

![s_tx.png](/assets/images/wifi6/ate_tool/s_tx.png)

测试结果保存到程序exe路径HB_loop_send_check_macbypass.txt文件中，如下图所示：

![hb-tx-log.png](/assets/images/wifi6/ate_tool/hb-tx-log.png)

测试2.4g时类似如上操作

此外，工具还提供全遍历TX测试，配合配置文件使用，在sf_config.txt文件中配置测试要测试的信道，在sf_traverse.txt文件中配置要测试的速率
如下图是测试信道，可增减：

![test_channel.png](/assets/images/wifi6/ate_tool/test_channel.png)

如下图是测试速率，可增减：

![test_rate.png](/assets/images/wifi6/ate_tool/test_rate.png)

以上信道和速率配置为0表示不测试，配置为1表示测试。

配置文件配置好后，打开工具，连接dut，选择仪表后，单击ALL_TX按键即可开始测试，如下图所示：

![all-tx.png](/assets/images/wifi6/ate_tool/all-tx.png)

测试结果保存到HB_loop_tx.txt文件中，如测试2.4G,测试结果将在LB_loop_tx.txt文件中保存。

此外工具结合串口继电器使用，可对单独或者多个测试项进行压测，当使用串口继电器时，需要配置串口号，右键“我的电脑”，“设备管理器”查看串口号，如下图：

![com.png](/assets/images/wifi6/ate_tool/com.png)

然后打开工具sf_config.txt配置文件,配置串口号，如下图：

![config.png](/assets/images/wifi6/ate_tool/config.png)

并将Test_Set节点下的automatic_test配置为1，如下图：

![automatic_test](/assets/images/wifi6/ate_tool/automatic_test.png)

配置完成后打开工具，不必勾选Connect_Dut,选择需要测试的BAND,2.4G或5G，然后依次选择要测试的信道，带宽，模式等，如下图所示：

![set.png](/assets/images/wifi6/ate_tool/set.png)

注意当要测试160M带宽时，channel选择5180和5500。
此后选择使用的仪表类型，设置好测试的txpower index初始值和测试次数，即可TxStart按钮开始测试，如下图所示：

![tx_start.png](/assets/images/wifi6/ate_tool/tx_start.png)

工具界面TX_Index配置txpower index初始值，TestTimes配置测试次数，测试完成后日志保存在HB_loop_send_check_macbypass.txt文件中。

### 4.3 SIFLOWER ATETOOL工具RX测试

保持如6.2.1所搭建的测试环境，待板子上电启动完成，依照6.2.1同样方法使板子进入SF ATE TEST TOOL测试模式，并打开工具，点击工具界面左上角Connect DUT，使工具即与板子建立连接，正常连接后，依次从左到右选择如下图所示的参数选项，单击工具RxStart按钮即开始控制板子进行RX测试。

如下图所示：

![s-rx.png](/assets/images/wifi6/ate_tool/s-rx.png)
测试结果保存到程序exe路径HB_loop_rx_min_sens_macbypass.txt文件中，如下图所示：

![hb-rx-log.png](/assets/images/wifi6/ate_tool/hb-rx-log.png)

此外，工具还提供全遍历RX测试，配合配置文件使用，在sf_config.txt文件中配置测试要测试的信道，在sf_traverse.txt文件中配置要测试的速率，配置方法如6.2.1TX测试截图，要遍历的接收灵敏度范围在配置文件rx_spec.txt文件中进行配置。
rx_spec.txt文件配置说明如下图：

![rx_spec.png](/assets/images/wifi6/ate_tool/rx_spec.png)

如图中两个红色框框住部分，表示将在-30到-31这个区间遍历11ax 160M MCS11的接收灵敏度，这个区间可根据测试需求进行配置，其余rx测试项同样的配置方法。

完成sf_config.txt，rx_spec.txt，sf_traverse.txt相关配置后，打开工具，连接dut，选择使用仪表，单击ALL_RX按键即可开启全遍历RX测试，如下图所示：

![all-rx.png](/assets/images/wifi6/ate_tool/all-rx.png)

测试完成后测试结果保存到程序exe路径HB_loop_rx_min_sens.txt中，如下图所示：

![all_rx_log.png](/assets/images/wifi6/ate_tool/all_rx_log.png)

### 4.4 退出测试模式

当要退出测试模式时，在串口工具界面输入```sfwifi reset fmac```命令，即可退出测试模式。

## 5 异常处理

1.当遇到任何warning，error等问题，首先通过```sfwifi reset fmac```或者重启板子尝试解决。
2.如果串口卡死或者板子死机，请重启板子。
3.测试时如果发现功率为负，或者抓不到功率信号
 硬件方向:
  1) 检查板端馈线是否焊接正确，或者有无扣紧
  2) 检查仪器RF口线材是否松动
  3) 线材是否有问题，需要更换
 软件方向
  1) 指令参数是否输入正确/工具是否选择正确
  2) 仪器端设置是否与板端发送信号设置相对应

   如果以上措施都检查没问题还是不正常，请检查板子RF电路是否存在问题。

4.测试时发现功率偏低，或者EVM偏低
  1）是否未调节合适的gain值
  2）EVM偏低，检查测试时仪器是否打开全包
  2）功率偏低，检查是否未补偿线损到仪器
  3）evm偏低，检查线是否焊接好，或者线是否有问题
  4）板子RF电路是否存在问题