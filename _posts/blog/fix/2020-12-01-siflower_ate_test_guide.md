---
layout: post
title: SIFLOWER射频测试手册
categories: PRODUCE
description: 介绍SIFLOWER方案手动及ATE工具射频测试
keywords: ate_cmd test
mermaid: true
---

# SIFLOWER射频测试手册

**目录**

* TOC
{:toc}

## 1 介绍

本文基于公司SIFLOWER芯片方案介绍Openwrt系统下手动及使用ATE工具测试产品射频性能，包括TX/RX两方面测试，帮助客户或使用者了解和评估产品射频性能。本文档只介绍command测试操作方法，至于对产品进行校准，请使用PCBA产测工具

## 2 环境准备

|序号|设备名称|型号|用途|数量|备注|
|--|--|--|--|--|--|
|1|待测产品|任意|待测设备|1|待测板子须具备ATE测试功能的Openwrt环境|
|2|测试电脑|任意|用于测试操作|1||
|3|综测仪|任意|用于完成TX/RX测试|1|具备11a/b/g/n/11ac协议要求的双频综测仪|
|4|RF线缆|任意|用于搭建测试环境|5|频率要求支持到6G|
|5|串口线|任意|用于发送command控制板子|1||


## 3 SIFLOWER 协议命令参数说明

### 3.1 Band参数说明

|Band|wlan0|wlan1|
|--|--|--|
|频段|2.4G|5G|

### 3.2 Mode参数说明

|值|0|2|4|
|--|--|--|--|
|模式|11a/b/g|11n|11ac|

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

### 3.4 BandWidth参数说明

|值|0|1|2|3|
|--|--|--|--|--|
|BandWidth|20MHz（11a/b/g）|20MHz(11n/ac)|40MHz(11n/ac)|80MHz(11ac)|

### 3.5 Frame BandWidth参数说明

|值|0|1|2|3|
|--|--|--|--|--|
|BandWidth|20MHz（11a/b/g）|20MHz(11n/ac)|40MHz(11n/ac)|80MHz(11ac)|

注意：测试时BandWidth参数与Frame BandWidth参数保持一致。

## 4 SIFLOWER 协议命令组成及说明
本部分包括TX与RX测试两部分，命令为上文3小节所描述的参数组成。

### 4.1 TX 测试命令

|命令头部|Tx_Frame_longth|Freq|Center-freq|BW|Frame-BW|Mode|DataRate|Guard Interval|Power_level|命令尾部|
|--|--|--|--|--|--|--|--|--|--|--|
|ate_cmd Band fastconfig| -l 发射帧长度| -f 频率| -c 中心频率| -w 带宽| -u 帧带宽| -m 模式| -i 速率| -g 传输间隔| -p gain| -y|

如上TX命令结构:
- Band代表wlan0/wlan1;
- 传输间隔代表Short GI或者Long GI,分别用0和1表示;
- gain代表TX功率发射等级，0~31可选;

如想要控制板子做11ac 40M ，5500信道，速率为MCS9，功率等级为30的短距发射，命令结构如下：

```
ate_cmd wlan1 fastconfig  -l 1024 -f 5500 -c 5510 -w 2 -u 2 -m 4 -i 9 -g 0 -p 30  -y
```

如控制板子停止TX，使用如下指令：

```
ate_cmd wlan1 fastconfig -q
```

**注意：停止2.4G与5G 时wlan参数不同。**

### 4.2 RX测试命令

|命令头部|Freq|Center-freq|BW|Frame-BW|命令尾部|
|--|--|--|--|--|--|
|ate_cmd Band fastconfig|-f 频率| -c 中心频率| -w 带宽| -u 帧带宽|-r|

如上RX命令结构：
- Band代表wlan0/wlan1;
- 带宽与帧带宽相一致。

如综测仪发射20MHz，中心频率为5180的信号，板端通过以下指令开始接收和查看RX接收情况

```
ate_cmd wlan1 fastconfig -f 5180 -c 5180 -w 0 -u 0 -r
```

查看结果

```
ate_cmd wlan1 fastconfig -R 
```

控制板子停止RX，使用如下指令：

```
ate_cmd wlan1 fastconfig -k
```

注意停止2.4G与5G 时wlan参数不同。

## 5 SIFLOWER 方案手动测试

测试拓扑如下：

![Test_topolgy](/assets/images/ate_test/Test_topolgy.png)

如图：
- PC1通过网线连接综测仪；
- 综测仪与板子天线之间用RF线缆连接；
- PC2通过串口线连接板子，用于控制板子。
  
### 5.1 XO校准（频偏校准）

完成上面的准备工作后，测试开始。然后观察串口打印信息，等待板子启动完毕后在串口工具界面输入“ate_init”  

1. 输入下面的命令发送调制信号2412.  

```
ate_cmd wlan0 fastconfig -l 1024 -f 2412 -c 2412 -w 1 -u 1 -m 2 -i 7 -g 0 -p 31 -y
```

2. 使用仪器测量频偏.  

3. 输入下面的命令调整频偏，通过仪器可观察变化。其中value的值需要不断调整修改，不同的值会影响频偏的变化，然后观察频偏结果，当频偏小于5ppm后，进行步骤4  
   调整策略:  
   首先设置一个初始值0x00  
   如果频偏**大于**0，则从0x00开始，然后每次在之前的基础上增加0x01. 比如第一次设置0x02,第二次0x03,第三次0x04(最大到0x3f)....直至频偏到正常范围  
   如果频偏**小于**0，则从0xc0开始，然后每次在之前的基础上增加0x01. 比如第一次设置0xc2,第二次0xc3,第三次0xc4(最大到0xff)....直至频偏到正常范围
                           
```
ate_cmd wlan1 fastconfig -O value
```

   **使用指令时value参数值，必须是用10进制，需要转换一下,0x14 则value写20**

4. 输入下面的命令储存校准值,比如校准好频偏后value结果为31，则需要将31转化为16进制0x1f进行保存

```
ate_cmd save 0x1f1f                                                
```

5. 输入下面的命令停止发送，完成XO校准。

```
ate_cmd fastconfig -q
```

### 5.2 SIFLOWER 方案手动TX测试

**天线控制指令**    
在使用ate_cmd指令之前，需要先使用ate_init指令进行初始化，可以在初始化时增加参数对天线进行控制

```
  ate_init //默认打开四路天线
  ate_init lb1  //打开lb1
  ate_init lb2  //打开lb2
  ate_init hb1  //打开hb1
  ate_init hb2  //打开hb2
```

如上测试拓扑搭建完成后，给板子上电，然后观察串口打印信息，等待板子启动完毕后在串口工具界面输入“ate_init”,如下图：

![ate_init](/assets/images/ate_test/ate_init.png)

等待大约10秒，板子ate_init成功，如下图：

![ate_init_ok](/assets/images/ate_test/ate_init_success.png)

在串口工具界面输入如4.1小节的TX Start测试命令，板子即进入TX工作模式，

![ate_cmd_tx](/assets/images/ate_test/ate_cmd_tx.png)

借助综测仪自带工具，如极致汇仪的WLAN Meter查看TX发射情况，正常发送如下图：

![TX_Result](/assets/images/ate_test/TX_Result.png)

注意：当要控制板子由一个TX状态切换到另一个TX状态，如查看另一个信道或频率的TX情况时，请先在原TX状态下停止板子发射，更改TX参数后再在控制板子发射。


- 单载波指令
 
如果需要发送单载波
1. 比如5G天线1路5500信道的80M带宽的单载波，使用如下命令

```
ate_cmd wlan1 fastconfig -f 5500 -c 5530 -w 3 –u 3 –p 31 -V 65 –o
```
**注意：**  
**-V 65 代表 ant0**

**-V 66 代表 ant1**

**采样的时候需要和中心频率一致**

停止发送单载波

```
ate_cmd wlan1 fastconfig -x
```

2. 比如2.4G天线2路的2437信道40M带宽单载波，使用如下命令

```
ate_cmd wlan0 fastconfig -f 2437 -c 2437 -w 2 –u 2 –p 31 -V 66 –o
```

停止发送单载波

```
ate_cmd wlan0 fastconfig -x
```

### 5.3 SIFLOWER 方案手动RX测试

- 板子启动后参照5.1先行让板子进入ATE测试模式
- 使用如下命令让板子开始进入接收状态
  
 ```
   ate_cmd wlan1 fastconfig -f 5180 -c 5180 -w 0 -u 0 -r
 ```
- 此时使用综测仪发送要测试的wifi信号
- 使用如下指令查看接收结果
  
 ```
   ate_cmd wlan1 fastconfig -R
 ```
测试结果如下图：

![RX_Result](/assets/images/ate_test/RX_Result.png)

注意：当要控制板子由一个RX状态切换到另一个RX状态，如查看另一个信道或频率的RX情况时，请先在原RX状态下停止板子接收，更改RX参数后再在控制板子进入接收状态。

- 接收停止指令
  
```
ate_cmd wlan1 fastconfig -k
```

## 6 SIFLOWER 方案ATE工具测试

本节介绍一个SIFLOWER 方案ATE测试工具，测试结果与上文提到的手动命令测试方式一致，使用工具旨在简化使用人员操作复杂的命令组合，达到同样的测试效果。

### 6.1 SIFLOWER ATE TOOL工具获取

获取最新的ATE TOOL工具
链接：[百度网盘](https://pan.baidu.com/s/1K1HJ6kTpb_Hpsm92ozvhow) 
提取码：SiFi


### 6.2 SIFLOWER ATE TOOL工具使用

本工具支持基于SIFLOWER 芯片方案的ATE测试，包括TX/RX两个部分：

工具界面如下：

![ATE_TOOL](/assets/images/ate_test/ATE_TOOL.png)

#### 6.2.1 SIFLOWER ATE TOOL工具TX测试

参照5 SIFLOWER 方案手动测试搭建测试环境，环境打建好以后给板子上电，待板子启动完毕，在串口工具界面输入“ifconfig br-lan 192.168.4.1”，修改板子的ip为192.168.4.1，在原有测试拓扑上增加一根网联连接PC2与待测板子LAN口，并将PC2网卡的IP地址设成“192.168.4.xx”网段，通过PC端ping -t 192.168.4.1来查看PC2与板子连接情况，待连接正常后，在串口界面输入依次输入```ate_int```初始化测试环境```ate_server```，使板子进入如下图的SFATETESTTOOL测试模式，此外在PC2上打开SIFLOWER ATE TOOL测试工具。

输入```ate_init```大概5秒后，当见到下图中的log后，板子测试环境初始化完成。

![ATE_TOOL_init](/assets/images/ate_test/ate_init.png)

再输入```ate_server```,进入测试模式等待指令

![init_ok](/assets/images/ate_test/ate_server.png)

板子ATE初始化完成后点击工具界面左上角Connect DUT，工具即与板子建立连接，在工具右下角的log区域会提示工具与板子连接情况：如下图：

![connect_ok](/assets/images/ate_test/connect_ok.png)

当连接异常时工具会弹框提示连接失败，如下图示：

![connect_fail](/assets/images/ate_test/connect_fail.png)

正常连接后，依次从左到右选择如下图所示的参数选项，单击工具Start Tx按钮即开始控制板子进行TX测试。

![Start_Tx](/assets/images/ate_test/Start_Tx.png)

仪器界面可查看测试结果，如使用极致汇仪综测仪时，测试结果如下图：

![TX_Result](/assets/images/ate_test/TX_Result.png)

注意：当要控制板子由一个TX状态切换到另一个TX状态，如查看另一个信道或频率的TX情况时，请先在原TX状态下单击Stop TX按键，更改TX参数后再在控制板子发射。

#### 6.2.2 SIFLOWER ATE TOOL工具RX测试

保持如6.2.1所搭建的测试环境，待板子上电启动完成，依照6.2.1同样方法使板子进入SF ATE TEST TOOL测试模式，并打开工具，点击工具界面左上角Connect DUT，使工具即与板子建立连接，此时使用综测仪自带工具的VSG功能发送信号，在工具端设置好RX参数后即可在工具log区域查看板子rx情况。

工具RX参数设置需与综测仪发送参数设置相一致，如下图所示：

![RX](/assets/images/ate_test/RX.png)

设置完成点击Start RX即可在工具log区域查看板子接收情况。

注意：当要控制板子由一个RX状态切换到另一个RX状态，如查看另一个信道或频率的RX情况时，请先在原RX状态下单击Stop RX按键，更改RX参数后再控制板子发射。

### 6.3 退出测试模式

当要退出测试模式时，在串口工具界面输入“sfwifi reset fmac”命令，即可退出ate测试模式。

## 7 异常处理

1，当遇到任何warning，error等问题，首先通过```ate_init```尝试解决。
2，如果串口卡死或者板子死机，请重启板子。
3，测试时如果发现功率为负，或者抓不到功率信号
 硬件方向，
  1) 检查板端馈线是否焊接正确，或者有无扣紧
  2) 检查仪器RF口线材是否松动
  3) 线材是否有问题，需要更换
 软件方向
  1) 指令参数是否输入正确/工具是否选择正确
  2) 仪器端设置是否与板端发送信号设置相对应
   
   如果以上措施都检查没问题还是不正常，请检查板子RF电路是否存在问题。

4，测试时发现功率偏低，或者EVM偏低
  1）是否未调节合适的gain值
  2）EVM偏低，检查测试时仪器是否打开全包
  2）功率偏低，检查是否未补偿线损到仪器
  3）evm偏低，检查线是否焊接好，或者线是否有问题
  4）板子RF电路是否存在问题


