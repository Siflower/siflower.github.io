---
layout: post
title: SIFLOWER校准方案说明手册
categories: WiFi6
description: 介绍SIFLOWER校准方案
keywords: calibration
mermaid: true
---

# SIFLOWER校准方案说明手册

**目录**

* TOC
{:toc}


## 1 介绍

本文针对SIFLOWER芯片方案的WiFi校准相关指令进行说明，适用于生产校准开发相关人员。

为了便于其它厂商装备开发快速对接，实现基于 ate_cmd 的系统下手动校准方案；以及基于 ate_server + atetool 工具的自动校准方案。

校准频点
2g校准3个点(1 7 13 的 11ax 40M mcs11）
5g校准5个点(36 64 100 的11ax 160M mcs11 ，132 149的 11ax 80M mcs11)
目前方案暂时测试以上几个点，后续根据大批量射频测试数据进行调整修改。

## 2 校准步骤

### 启动wifi

同时起 2.4G 和 5G WiFi 指令如下：
```
sfwifi reset fmac
ifconfig wlan0 up
ifconfig wlan1 up
```
注：**wlan0** 代表 **5G WiFi**；**wlan1** 代表 **2.4G WiFi**。

### 2.1 XO校准

- 在系统下使用ate_cmd命令控制板子做2.4G 的TX常发;

- 通过仪器获取TX结果，并将获取到的ppm与设定范围（-2.5 +2.5）做对比，在门限范围则停止XO校准，将此XO值写入板子
如超出设定范围，则通过ate_cmd调整xo值，如此重复，直至ppm满足设定范围（-2.5 +2.5）。

具体操作，举例如下：
1. 在ate_init进入ate模式后，选择任意信道模式发送调制信号如：2412_11n_20M_mcs7
`ate_cmd wlan1 fastconfig -f 2412 -c 2412 -w 1 -m 2 -i 7 -B 1 -p 120 -y`

2. 使用综测仪测量频偏值

3. 输入下面的命令调整频偏，通过仪器可观察变化。
`ate_cmd wlan1 fastconfig -O value`
value 的取值范围是 0x00-0x7f(0~127)
**注意**:使用 -O 指令时 value 参数值，必须是用10进制，需要转换一下。其中 value 的值需要不断调整修改，不同的值会影响频偏的变化，然后观察频偏结果，当频偏满足 ±2.5ppm 后，进行步骤4。

4. 保存调整好的频偏值
输入下面的命令储存校准值,比如校准好频偏后value结果为31，则需要将31转化为0x1f写入
`ate_cmd save 0x1f1f`

5. 输入下面的命令停止发送，完成XO校准
`ate_cmd wlan1 fastconfig -q`

### 2.2 TX校准

1. 初始化power_save文件
`ate_cmd init_power_save`

2. 使用ate_cmd常发需要测试的各个频点；

3. 仪器读取测试结果与evm标准和目标功率门限做比较(首先判断EVM，在evm满足的前提下，调整gain值满足目标功率门限)，如满足设定则板端保存当前测试项目的gain值作为校准值；如不满足，则调整-p 参数后再次重复步骤2，直至获取的power和EVM的测试结果与设定相符，满足后保存此gain值；

4. ate_cmd会将最后一次满足目标功率和校准值写入power_save_ant1.txt 和power_save_ant2.txt对应的位置

所有校准点校准完如下：
一路

![ant1.png](/assets/images/wifi6/calibration/ant1.png)

二路

![ant2.png](/assets/images/wifi6/calibration/ant2.png)


### 2.3 校准值偏移

对于没有校准的点，完成一路二路所有校准点后，使用`ate_cmd tx_calibrate_over`开始在校准频点的基础上进行偏移，计算出完整的两个校准表。

**注：gain_offset.txt中的offset值需要按照客户板子大批量实际射频测试情况进行调整偏移量，一般来说由射频测试完整之后提供，然后按照txt格式生成一个客户自己的偏移值文档放入软件对应位置即可**

查看最终校准值保存文件：
一路2G和5G

![offset1.png](/assets/images/wifi6/calibration/offset1.png)

二路2G和5G

![offset2.png](/assets/images/wifi6/calibration/offset2.png)

### 2.4 校准表写入

当完成校准值偏移后，就可以将最终的power_save_ant1.txt 和power_save_ant2.txt的内容写入factory分区了
```
ate_cmd wlan0 fastconfig -p save_all
```
至此完成一个校准流程，校准值写入后，下次重启生效

### 2.5 校准值综测

在生产的时候，产线往往需要进行不下电综测，此时可以读取power_save_ant1.txt 和power_save_ant2.txt中的校准值出来发射功率验证，只需要将-p 后的值固定写203即可，常发时则会根据前面的信道/带宽/模式/速率去拿出对应txt的校准值出来发射对应功率

如读取5G ant2 powe_save_ant2.txt保存值常发校验
```ate_cmd wlan0 fastconfig -f 5180 -c 5210 -w 3 -m 5 -i 11 -B 2 -p 203 -y```

### 2.6 校准值验证

如果在校准完成之后，需要降factory分区保存的校准值出来发射功率验证，只需要将-p 后的值固定写200即可，常发时则会根据前面的信道/带宽/模式/速率去拿校准分区对应的校准值出来发射对应功率

如读取5G ant2 factory分区校准值校验
```ate_cmd wlan0 fastconfig -f 5180 -c 5210 -w 3 -m 5 -i 11 -B 2 -p 200 -y```

## 3 校准表说明

### 3.1 完整的校准表

![calibration1.png](/assets/images/wifi6/calibration/calibration1.png)

![calibration2.png](/assets/images/wifi6/calibration/calibration2.png)

### 3.2 校准表解析

校准表组成
| 说明 | size |
| :-: | :-: |
| 2.4G ant1 校准表大小 | 676 |
| 5G ant1 校准表大小 | 2525 |
| 2.4G ant2 校准表大小 | 676 |
| 5G ant2 校准表大小 | 2525 |

校准表内容解释：
校准表按照 2.4G/5G 按照信道模式速率依次连续写入，这里方便说明做了分行处理

  - 2.4G

    每行的52个数据分别代表一个2.4G信道，每个值按照 11b/11g/11n_20M/11n_40M/11ax_20M/11ax_40M 的速率从低到高依次排列。
    每一行代表2.4G各个信道，总共13个信道 。

	![24G.png](/assets/images/wifi6/calibration/24G.png)
  - 5G

    每行101个数据分别代表一个5G信道，每个值按照 11a/11n_20M/11n_40M/11ac_20M/11ac_40M/11ac_80M/11ax_20M/11ax_40M/11ax_80M/11ax_160M 的速率从低到高依次排列。
	每一行代表5G各个信道，总共25个信道。

	![5G.png](/assets/images/wifi6/calibration/5G.png)


## 附录：校准点指令

```
24G_1
ate_cmd wlan1 fastconfig -f 2412 -c 2422 -w 2 -m 5 -i 11 -B 1 -p 1 -y
ate_cmd wlan1 fastconfig -f 2442 -c 2442 -w 2 -m 5 -i 11 -B 1 -p 7 -y
ate_cmd wlan1 fastconfig -f 2472 -c 2462 -w 2 -m 5 -i 11 -B 1 -p 13 -y

5G_1
ate_cmd wlan0 fastconfig -f 5180 -c 5250 -w 4 -m 5 -i 11 -B 1 -p 1 -y
ate_cmd wlan0 fastconfig -f 5320 -c 5250 -w 4 -m 5 -i 11 -B 1 -p 3 -y
ate_cmd wlan0 fastconfig -f 5500 -c 5570 -w 4 -m 5 -i 11 -B 1 -p 7 -y
ate_cmd wlan0 fastconfig -f 5660 -c 5690 -w 3 -m 5 -i 11 -B 1 -p 11 -y
ate_cmd wlan0 fastconfig -f 5745 -c 5775 -w 3 -m 5 -i 11 -B 1 -p 15 -y

24G_2
ate_cmd wlan1 fastconfig -f 2412 -c 2422 -w 2 -m 5 -i 11 -B 2 -p 1 -y
ate_cmd wlan1 fastconfig -f 2442 -c 2442 -w 2 -m 5 -i 11 -B 2 -p 7 -y
ate_cmd wlan1 fastconfig -f 2472 -c 2462 -w 2 -m 5 -i 11 -B 2 -p 13 -y

5G_2
ate_cmd wlan0 fastconfig -f 5180 -c 5250 -w 4 -m 5 -i 11 -B 2 -p 1 -y
ate_cmd wlan0 fastconfig -f 5320 -c 5250 -w 4 -m 5 -i 11 -B 2 -p 3 -y
ate_cmd wlan0 fastconfig -f 5500 -c 5570 -w 4 -m 5 -i 11 -B 2 -p 7 -y
ate_cmd wlan0 fastconfig -f 5660 -c 5690 -w 3 -m 5 -i 11 -B 2 -p 11 -y
ate_cmd wlan0 fastconfig -f 5745 -c 5775 -w 3 -m 5 -i 11 -B 2 -p 15 -y
```
