---
layout: post
title: 矽路由性能测试手册
categories: TEST
description: 介绍矽路由性能测试手册
keywords: Performance Testing
mermaid: true
---

# 矽路由性能测试手册

**目录**

* TOC
{:toc}

## 1 介绍

本文基于SF19A28产品，描述了Openwrt平台功能下矽路由系列产品的环境搭建，测试步骤以及测试结果。

## 2 测试环境

### 2.1 测试硬件

|序号|设备名称|型号|用途|数量|备注|
|--|--|--|--|--|--|
|1|待测产品|SF19A28_EVB|待测设备|1||
|2|测试电脑|任意|有线和无线两种方式连接路由器|3||
|3|测试网卡|任意|有线和无线两种方式连接路由器|3|千兆有线网卡及具备双频2T2R以上配置无线网卡|

### 2.2 测试软件

|序号|软件/操作系统|版本号|功能|备注|
|--|--|--|--|--|
|1|操作系统|Windows|PC||
|2|IE浏览器|IE11|访问路由器配置界面||
|3|Iperf|2.0.9|性能测试||
|4|串口工具|putty/SecureCRTPortable|抓取信息，运行命令||
|5|IxChariot|6.7|性能测试||

## 3 IxChariot软件

### 3.1 IxChariot简介

IxChariot是美国IXIA公司的推出的针对应用层性能测试的一款软测试工具。IxChariot在应用层性能测试领域已得到业界的广泛认可。IxChariot能够评估网络应用的性能和容量，对网络和设备进行压力测试，测试设备及网络在不同应用、不同参数下的吞吐量、时延、丢包、反应时间等性能参数。

Chariot 由两部分组成：控制端 Console 和远端 Endpoint，两者都可安装在普通 PC 或者 服务器上，控制端安装在 Windows 操作系统上，Endpoint 支持各种主流的操作系统。控制端为该产品的核心部分，控制界面（也可采用命令行方式）、测试设计界面、脚本选择及编制、结果显示、报告生成以及 API 接口等都由控制端提供。Endpoint可根据实际测 试的需要安装在单个或者多个终端处，负责从控制端接收指令、完成测试并将测试数据上报到控制端。

### 3.2 IxChariot软件下载

网盘下载地址：
链接：https://pan.baidu.com/s/1xMkNt4IWT3JjZdMj5SVuWA
密码：w9mx

### 3.3 IxChariot软件安装

IxChariot软件安装分为软件Console端安装及Endpoint端安装。

#### 3.3.1 IxChariot软件Console端安装

电脑初次安装 IxChariot大约需要 10 分钟左右，全部按照默认选择，直到安装完成，会出现下图界面，点击Finish 完成安装。

![install_finished](/assets/images/throughput/install_finished.png)

此时Ixchariot还是不能使用，需要破解方可。方法如下：  
以管理员权限运行IxChariot-6.70.44-Crack 文件（此运行前一定要将电脑端的杀毒软件全部关闭），如下图。

![Crack_IxChariot_1](/assets/images/throughput/Crack_IxChariot_1.png)

点击Patch,如下图所示：

![Crack_IxChariot_2](/assets/images/throughput/Crack_IxChariot_2.png)

此时找到程序安装目录，选中并打开AppsLM.dll 文件，这样就完成破解了。

![Crack_IxChariot_3](/assets/images/throughput/Crack_IxChariot_3.png)

然后再运行桌面上的IxChariot快捷图标，进入IxChariot Test界面，如下图：

![Open_IxChariot](/assets/images/throughput/Open_IxChariot.png)

#### 3.3.2 IxChariot软件Endpoint端安装

软件Endpoint端可以安装在路由器，也可安装在PC及手机端。

##### 3.3.2.1 路由器端安装Endpoint

打开路由器的串口工具，按照如图6步骤进行安装。（在此之前需要先安装winSCP，将Endpoint安装包上传至路由器的根目录下，具体操作步骤参考文末附录）

![endpoint_router](/assets/images/throughput/endpoint_router.png)

##### 3.3.2.2 PC端安装Endpoint

需要注意的是Win 7 和XP 安装用的是不同版本的endpoint 软件,需要根据电脑系统位数选择相应的工具版本进行安装，另外建议任意pc端都安装上Console和Endpoint。

![PC_Endpoint_Install](/assets/images/throughput/PC_Endpoint_Install.png)

##### 3.3.2.2 手机端安装Endpoint

手机上安装Endpoint软件如下图：

![Mobile_Endpoint_Install](/assets/images/throughput/Mobile_Endpoint_Install.png)

### 3.4 IxChariot软件使用

进入主程序界面后，选择Add Pair ，也就是增加一条通道，如下图所示：

![use_ixchariot_1](/assets/images/throughput/use_ixchariot_1.png)

然后就会弹出一个窗口要求你输入测试参数，如下图：

![use_ixchariot_2](/assets/images/throughput/use_ixchariot_2.png)

测试一般选择throughput 来测试路由器的吞吐量。

![ixchariot_script](/assets/images/throughput/ixchariot_script.png)

然后点击Run来开始测试。
点击RUN菜单下的 set run options可以设置测试的时间等其他参数。

![set_ixchariot_test_time](/assets/images/throughput/set_ixchariot_test_time.png)

点击throughput 可以查看具体数据，如下图：

![Test_Results](/assets/images/throughput/Test_Results.png)

补充两点：

- 1.如果点击run 后没有数据参数，请检查endpoint 服务是否开启，可手动重新启动服务。
  
- 2.无线信号干扰对无线吞吐量测试的影响非常大，避免在无线信号环境复杂的场合做无线测试。

### 3.5 测试实例

#### 3.5.1 实例1 以太网发射

测试拓扑图：

![Eth_Tx_Topology](/assets/images/throughput/Eth_Tx_Topology.png)

测试步骤：  
(1) 以太网连接路由与PC，路由连接串口工具。  
(2) 打开IxChariot软件，输入相应的IP地址。  
(3) 点击Run来进行测试，如下图所示，测试结果为94.623Mbps。  

![Eth_Tx_Result](/assets/images/throughput/Eth_Tx_Result.png)

#### 3.5.2 实例2 以太网接收

测试拓扑图：

![Eth_Rx_Topology](/assets/images/throughput/Eth_Rx_Topology.png)

测试步骤：  
(1) 以太网连接路由与PC，路由连接串口工具。  
(2) 打开IxChariot软件，输入相应的IP地址。  
(3) 点击Run来进行测试，如下图所示，测试结果为94.741Mbps。  

![Eth_Rx_Result](/assets/images/throughput/Eth_Rx_Result.png)

#### 3.5.3 实例3 以太网双向

测试拓扑图：

![Eth_Both_Direction](/assets/images/throughput/Eth_Both_Direction.png)

测试步骤：

(1) 路由与PC1和PC2均用以太网连接，路由连接串口工具。  
(2) 打开IxChariot软件，添加两个通道，分别输入相应的IP地址,如图。  

![Eth_Both_Direction_Test](/assets/images/throughput/Eth_Both_Direction_Test.png)

(3) 点击Run或 来进行测试，如图所示，测试结果：176.036Mbps。

![Eth_Both_Direction_Result](/assets/images/throughput/Eth_Both_Direction_Result.png)

#### 3.5.4 实例4 百兆以太网–>2.4G wifi

测试拓扑图：

![Eth_To_2.4G](/assets/images/throughput/Eth_To_2.4G.png)

测试步骤：  
(1)PC1以太网连接路由的百兆LAN口，PC2连接路由的2.4Gwifi，路由连接串口工具。  
(2)登录路由网页设置页面，将2.4Gwifi 频宽设置为40MHz。  
(3) 打开IxChariot软件，输入PC1和PC2的IP地址。  
(4) 点击Run进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：83.919Mbps  

![Eth_To_2.4G_Result](/assets/images/throughput/Eth_To_2.4G_Result.png)

#### 3.5.5 实例5 2.4G wifi–>百兆以太网

测试拓扑图：

![2.4G_To_Eth_Topology](/assets/images/throughput/2.4G_To_Eth_Topology.png)

测试步骤：
(1) PC1以太网连接路由的百兆LAN口，PC2连接2.4Gwifi，路由连接串口工具。  
(2)登录路由网页设置页面，将2.4Gwifi 频宽设置为40MHz。  
(3)打开IxChariot软件，输入PC2和PC1的IP地址。  
(4) 点击Run进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：89.153Mbps  

![2.4G_To_Eth_Result](/assets/images/throughput/2.4G_To_Eth_Result.png)

#### 3.5.6 实例6 2.4G wifi–>千兆以太网

测试拓扑图：

![2.4G_To_Eth_Topology](/assets/images/throughput/2.4G_To_Eth_Topology.png)

测试步骤：
(1) PC1以太网连接路由的千兆WAN口，PC2连接路由的2.4G wifi，路由连接串口工具。
(2)登录路由网页设置页面，将2.4Gwifi 频宽设置为40MHz。
(3)打开IxChariot软件，输入PC2和PC1的IP地址。
(4) 点击Run进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：100.577Mbps

![2.4G_To_Gmac_Eth_Result](/assets/images/throughput/2.4G_To_Gmac_Eth_Result.png)

#### 3.5.7 实例7 千兆以太网–>2.4G wifi

测试拓扑图：

![GEth_TO_2.4G_pology](/assets/images/throughput/GEth_TO_2.4G_pology.png)

测试步骤：
(1) PC1以太网连接路由的千兆WAN口，PC2连接路由的2.4G wifi，路由连接串口工具。
(2)登录路由网页设置页面，将2.4Gwifi 频宽设置为40MHz。
(3)打开IxChariot软件，输入PC1和PC2的IP地址。
(4) 点击Run进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：102.697Mbps

![GEth_TO_2.4G_pology](/assets/images/throughput/GEth_TO_2.4G_pology.png)

#### 3.5.8 实例8 百兆以太网–>5G wifi

测试拓扑图：

![Eth_To_5G](/assets/images/throughput/Eth_To_5G.png)

测试步骤：
(1) PC1以太网连接路由，PC2连接5Gwifi，路由连接串口工具。
(2)打开IxChariot软件，输入PC1和PC2的IP地址。
(3) 点击Run进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：94.059Mbps

![Eth_To_5G_Result](/assets/images/throughput/Eth_To_5G_Result.png)

#### 3.5.9 实例9 5G wifi–>百兆以太网

测试拓扑图：

![5G_To_Eth_pology](/assets/images/throughput/5G_To_Eth_pology.png)

测试步骤：
(1) PC1以太网连接路由，PC2连接5Gwifi，路由连接串口工具。
(2)打开IxChariot软件，输入PC2和PC1的IP地址。
(3) 点击Run进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：92.729Mbps

![5G_To_Eth_Result](/assets/images/throughput/5G_To_Eth_Result.png)

#### 3.5.10 实例10 5G wifi–>千兆以太网

测试拓扑图：

![5G_To_Eth_pology](/assets/images/throughput/5G_To_Eth_pology.png)

测试步骤：
(1) PC1以太网连接路由的千兆WAN口，PC2连接路由的5G wifi，路由连接串口工具。
(2)打开IxChariot软件，输入PC2和PC1的IP地址。
(3) 点击Run进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：177.335Mbps

![5G_To_Gmac_Eth_Result](/assets/images/throughput/5G_To_Gmac_Eth_Result.png)

#### 3.5.11 实例12 千兆以太网–>5G wifi

测试拓扑图：

![Gmac_Eth_to_5G_pology](/assets/images/throughput/Gmac_Eth_to_5G_pology.png)

测试步骤：
(1) PC1以太网连接路由的千兆WAN口，PC2连接路由的5G wifi，路由连接串口工具。
(2)打开IxChariot软件，输入PC1和PC2的IP地址。
(3) 点击Run进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：207.377Mbps

![Gmac_Eth_to_5G_Result](/assets/images/throughput/Gmac_Eth_to_5G_Result.png)

#### 3.5.12 实例12 2.4G wifi发射

测试拓扑图：

![2.4G_Tx_pology](/assets/images/throughput/2.4G_Tx_pology.png)

测试步骤：
(1) PC连接2.4Gwifi，路由连接串口工具。
(2)登录路由网页设置页面，将2.4Gwifi 频宽设置为40MHz。
(3)打开IxChariot软件，输入路由和PC的IP地址。
(4)点击Run来进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：79.485Mbps

![2.4G_Tx_Result](/assets/images/throughput/2.4G_Tx_Result.png)

#### 3.5.13 实例13 2.4G wifi接收

![2.4G_Rx_pology](/assets/images/throughput/2.4G_Rx_pology.png)

测试步骤：
(1) PC连接2.4Gwifi，路由连接串口工具。
(2)登录路由网页设置页面，将2.4Gwifi 频宽设置为40MHz。
(3)打开IxChariot软件，输入PC和路由的IP地址。
(4) 点击Run来进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：86.058Mbps

![2.4G_Rx_Result](/assets/images/throughput/2.4G_Rx_Result.png)

#### 3.5.14 实例14 5G wifi发射

![5G_Tx_pology](/assets/images/throughput/5G_Tx_pology.png)

测试步骤：
(1) PC连接5Gwifi，路由连接串口工具。
(2)打开IxChariot软件，输入路由和PC的IP地址。
(3)点击Run来进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：81.045Mbps

![5G_Tx_Result](/assets/images/throughput/5G_Tx_Result.png)

### 3.5.15 实例15 5G wifi接收

![5G_Rx_pology](/assets/images/throughput/5G_Rx_pology.png)

测试步骤：
(1) PC连接5Gwifi，路由连接串口工具。
(2)打开IxChariot软件，输入PC和路由的IP地址。
(3)点击Run进行测试，路由串口界面输入top命令获取CPU占比。如下图所示，测试结果：108.161Mbps

![5G_Rx_Result](/assets/images/throughput/5G_Rx_Result.png)

## 4 Iperf软件

### 4.1 Iperf简介
Iperf是由NLANRDAST开发作为一个现代的替代测量最大TCP和UDP带宽性能。iperf可以调整各种参数和UDP特性。iperf报告带宽，时延抖动，数据包丢失。是一个TCP/IP和UDP/IP的性能测量工具，能够提供网络吞吐率信息，以及震动、丢包率、最大段和最大传输单元大小等统计信息；从而能够帮助我们测试网络性能，定位网络问题。

### 4.2 Iperf软件下载
iperf 软件下载地址
链接：https://pan.baidu.com/s/14JIFue9ct6NVa6Hd1lu21g
提取码：3snr

### 4.3 Iperf软件常用参数说明
-s 以server模式启动，如：iperf -s
-c host以client模式启动，host是server端地址，如：iperf -c 222.35.11.23

通用参数
-i 以秒为单位显示报告间隔，如：iperf -c 192.168.5.1 -i 2
-l 缓冲区大小，默认是8KB, 如：iperf -c 192.168.5.1 -l 16
-p 指定服务器端使用的端口或客户端所连接的端口，如：iperf -s -p 9999; iperf -c 192.168.5.1 -p 9999
-u 使用udp协议
-w 指定TCP窗口大小，默认是8KB

Client端专用参数
-d 同时进行双向传输测试
-t 测试时间，默认10秒, 如：iperf -c 192.168.5.1 -t 5

### 4.4 Iperf软件使用

#### 4.4.1 实例1 以太网发射

测试拓扑图：

![Eth_Tx](/assets/images/throughput/Eth_Tx.png)

测试步骤：

(1)PC以太网连接路由，PC装有IPerf，路由连接串口工具。
(2)PC作为Server端，打开终端进到iperf软件所在目录下。路由作为Client端。
(3)参数说明如下：
- TCP测试参数说明：
- Server端参数是“iperf –s –i 1 -w 2M”；
- Client端参数是“iperf –c Server端IP -i 1 –t 120 -w 2M ”
- (窗口大小设为2M，双向加-d)
- UDP测试参数说明：
- Server端参数是“iperf -s –u –i 1 ”；
- Client端参数是“iperf –c –u Server端IP -i 1 –t 120 –b 200M ”
- (带宽大小设为200M，双向加-d)

(4)测试结果：95.7Mbps

#### 4.4.2 实例2 以太网接收

测试拓扑图：

![Eth_Rx](/assets/images/throughput/Eth_Rx.png)

测试步骤：

(1)PC以太网连接路由，PC装有IPerf，路由连接串口工具。
(2)PC作为Client端，打开终端进到iperf软件所在目录下，路由作为Server端
(3)参数说明如下：
- TCP测试参数说明：
- Server端参数是“iperf –s –i 1 -w 2M”；
- Client端参数是“iperf –c Server端IP -i 1 –t 120 -w 2M ”
- (窗口大小设为2M)
- UDP测试参数说明：
- Server端参数是“iperf -s –u –i 1 ”；
- Client端参数是“iperf –c –u Server端IP -i 1 –t 120 –b 200M ”
- (带宽大小设为200M)

(4)测试结果：95.7Mbps。

#### 4.4.3 实例3 以太网双向

测试拓扑图：

![Eth_TRx](/assets/images/throughput/Eth_TRx.png)

测试步骤：

(1)PC以太网连接路由，PC装有IPerf，路由连接串口工具。
(2)PC端打开终端进到iperf软件所在目录下。
(3)参数说明如下：
- TCP测试参数说明：
- PC端参数是“iperf –s –i 1 -w 2M”；
- 路由器参数是“iperf –c PC的IP -i 1 –t 120 -w 2M -d”
- (窗口大小设为2M, 双向加-d)
- UDP测试参数说明：
- PC端参数是“iperf -s –u –i 1 ”；
- 路由器参数是“iperf –c –u PC的IP -i 1 –t 120 –b 200M -d”
- (带宽大小设为200M, 双向加-d)

(4)测试结果：Tx 95.1Mbps Rx 65.5Mbps。

#### 4.4.4 实例4 wifi发射

测试拓扑图：

![wifi_Tx](/assets/images/throughput/wifi_Tx.png)

测试步骤：

(1)PC连接路由WIFI，PC上装有Iperf，路由连接串口工具。
(2)PC端打开终端进到iperf软件所在目录下。
(3)参数说明如下
- TCP测试参数说明：
- Server端参数是“iperf -s –l 32k”；
- Client端参数是“iperf –c Server端IP -i 1 –t 120 –l 32k –P 2 ” ；
- (多线程加-P xxx，双向加-d) (-l xxx代表数据长度)
- UDP测试参数说明：
- Server端参数是“iperf -s –u –l 32k”；
- Client端参数是“iperf –c –u Server端IP -i 1 –t 120 –b 300M –l 32k –P 2” ；
- (多线程加-P xxx，带宽大小设为300M，双向加-d) (-l xxx代表数据长度)

(4)测试结果：2.4G 84.6Mbps ， 5G 150 Mbps。

#### 4.4.5 实例5 wifi接收

测试拓扑图：

![wifi_Rx](/assets/images/throughput/wifi_Rx.png)

测试步骤：

(1)PC连接路由WIFI，PC上装有Iperf，路由连接串口工具。
(2)PC端打开终端进到iperf软件所在目录下。
(3)参数说明如下
- TCP测试参数说明：
- Server端参数是“iperf -s –l 32k”；
- Client端参数是“iperf –c Server端IP -i 1 –t 120 –l 32k –P 2 ” ；
- (多线程加-P xxx，双向加-d) (-l xxx代表数据长度)
- UDP测试参数说明：
- 服务器参数是“iperf -s –u –l 32k”；
- 客户端参数是“iperf –c –u Server端IP -i 1 –t 120 –b 300M –l 32k –P 2” ；
- (多线程加-P xxx，带宽大小设为300M，双向加-d) (-l xxx代表数据长度)

(4)测试结果：2.4G 89.7 Mbps ， 5G 140Mbps。

#### 4.4.6 实例6 WIFI双向

测试拓扑图：

![wifi_TRX](/assets/images/throughput/wifi_TRX.png)

(1)PC连接路由wifi 2.4G或5G，PC上装有Iperf，路由连接串口工具。
(2)PC端打开终端进到iperf软件所在目录下。
(3)参数说明如下
- TCP测试参数说明：
- PC端参数是“iperf -s –l 32k”；
- 路由端参数是“iperf –c PC端IP -i 1 –t 120 –l 32k –d ” ；
- (多线程加-P xxx，双向加-d) (-l xxx代表数据长度)
- UDP测试参数说明：
- PC端参数是“iperf -s –u –l 32k”；
- 路由端参数是“iperf –u –c PC端IP -i 1 –t 120 –b 300M –l 32k –d” ；
- (多线程加-P xxx，带宽大小设为300M，双向加-d) (-l xxx代表数据长度)

(4)测试结果：2.4G Tx 83Mbps, Rx 85.8Mbps;5G Tx 73Mbps, Rx 122Mbps。

## 5 附注

### 5.1 WinSCP安装
WinSCP 是一个 Windows 环境下使用的 SSH 的开源图形化 SFTP 客户端。同时支持 SCP 协议。它的主要功能是在本地与远程计算机间安全地复制文件，并且可以直接编辑文件。
可在官网上进行下载，官网地址：https://winscp.net/eng/docs/lang:chs。安装过程中一路点击“下一步”即可。
打开桌面winSCP快捷图标进行设置，将文件协议修改为SCP，输入路由的地址( 默认是192.168.4.1 )及用户名和密码（ 用户名和密码均为admin ）。如图所示。

![winSCP](/assets/images/throughput/winSCP.png)

设置后点击保存，再点击登录，弹出如图33对话框，查看即将连接的站点名称是否正确的，点击确认。再输入密码（ admin ）,点击“确认”，如下图。

![winSCP_login](/assets/images/throughput/winSCP_login.png)

登录完成后，将 pelinux_mipsle_720 （选中文件右击点击上传即可）上传至路由的路径下。上传路径最好选择根目录下,便于后续操作。

![Ple_winSCP](/assets/images/throughput/Ple_winSCP.png)
![Ple_winSCP_2](/assets/images/throughput/Ple_winSCP_2.png)