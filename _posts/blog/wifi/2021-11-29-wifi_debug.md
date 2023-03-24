---
layout: post
title: wifi调试
categories: WIFI
description: 介绍wifi调试相关命令
keywords: wifi
mermaid: true
---
# WiFi调试

**目录**
* TOC
{:toc}

## 1 介绍

### 1.1 适用人员

 - 熟悉wifi配置，见[WiFi架构和配置手册](https://siflower.github.io/2020/08/12/wifi_architecture_and_configuration_manual/)

### 1.2 开发环境

 - 正常的编译和运行环境，见[快速入门](https://siflower.github.io/2020/08/05/quick_start/)

### 1.3 功能概述

在无线路由器实际使用过程，有时难免会遇到设备掉线、设备无法上网、网络卡顿、设备无法连接等情况，本文主要针对与实际使用过程中遇到无线问题时，通过查看日志和调试信息，辅助定位问题。

## 2 调试环境

### 2.1 Uart串口调试

- 调试工具：主流调试软件有minicom、putty、smarTTY、sscom等；
- 接线：串口线接TX、RX和GND即可；
- 波特率：115200

### 2.2 SSH调试

- 调试工具：主流调试软件有ssh、putty、smarTTY等；
- 连接：电脑有线连接路由器lan口、或电脑无线连接路由器；
- 默认网关：192.168.4.1
- 默认账户：root
- 默认密码：admin

### 2.3 无线抓包

使用omnipeek抓包软件和抓包网卡，抓取空中包。

### 2.4 日志查看

- 查看系统日志指令
```
    logread
```
- 查看内核日志指令
```
    cat /proc/kmsg
```
## 3 WiFi调试

### 3.1 调试节点路径

- 2.4G调试节点
```
    /sys/kernel/debug/ieee80211/phy0/siwifi/
```
- 5G调试节点
```
    /sys/kernel/debug/ieee80211/phy1/siwifi/
```

![path.png](/assets/images/wifi_debug/path.png)

### 3.2 txq节点

通过txq节点可以看出实时的每个sta的tx队列负载情况：
- tid：表示队列优先级参数；
- status：表示当前队列状态；
- ready：队列缓存包个数；
- fastready：软加速队列缓存包个数；
- credit：表示队列剩余可链接给hwq的skb个数；
- ps_drop1：ps_on_drop
- ps_drop2：ps_active_drop
- ps_drop3：ps_off_drop
- total：总发包数
- success：总发包成功数
- amsdu：amsdu最大聚合大小
- max_nb：amsdu最大聚合个数

![txq.png](/assets/images/wifi_debug/txq.png)

### 3.3 stats节点

stats节点可以读取wifi驱动中tx和rx的ampdu和amsdu聚合度的统计数据。
- TXQs CFM balances：表示五个发送队列，0-4优先级依次降低；
- done：表示tx，已发送包的数量；
- received：表示rx，已接收包的数量；
- AMSDU：表示amsdu在0-3的各个聚合度下的收发包统计；
- AMPDU：表示mpdu在0-63的各个聚合度下的收发包统计；

![stats.png](/assets/images/wifi_debug/stats.png)

### 3.4 trx_stats节点

trx_stats节点可以读取wifi驱动中tx和rx的统计数据。
- rxdataind：表示驱动接收到rx包的数量；
- tx drop full：表示驱动tx队列已满而导致的丢包个数；

![trx.png](/assets/images/wifi_debug/trx.png)

### 3.5 lmactx节点

lmactx节点可以读取lamc中tx的统计数据。
- signal MPDU：表示对应队列发送的单包数量；
- total：表示对应mpdu长度的包发送的数量；
- successfull：表示对应队列（或者对应长度的mpdu）的发送成功的单包（或者聚合包）数量；
- rate（%）：表示对应队列（或者对应长度的mpdu）的发送成功率；

![lmactx.png](/assets/images/wifi_debug/lmactx.png)

### 3.6 lmacrx节点

lmacrx节点可以读取lamc中rx的统计数据。
- total：表示lmac收到rx包数量；
- uploads：表示lamc发送到umac的包数量；
- RX ctrl_frame：表示收到的控制帧的数量；
- RX mgmt frame：表示收到的管理帧的数量；
- RX data frame：表示收到的数据帧的数量；
- Key-invalid mgmt upload：表示解析失败的管理帧的数量；

![lmacrx.png](/assets/images/wifi_debug/lmacrx.png)

### 3.7 hwq节点

hwq节点可以读取五个队列中tx的统计数据。
- credits：表示剩余可用队列大小；
- size：表示hwq队列总大小；
- need_processing：表示hwq是否需要遍历，并发送；
- cfm_cnt：表示收到confirm的tx包数量；
- push_cnt：表示已push的tx包数量；

![hwq.png](/assets/images/wifi_debug/hwq.png)

### 3.8 mib节点

mib节点查看硬件收发包相关的统计信息。
- dot11_wep_excluded_count(0x0800) ：表示已丢弃的未加密帧数；
- dot11_fcs_error_count(0x0804)：表示接收FCS错误帧数；
- nx_rx_phy_error_count(0x0808)：表示RX时报告的PHY错误数；
- nx_rd_fifo_overflow_count(0x0808)：表示RX收到包，因为没有buffer而丢包的总数；

![mib.png](/assets/images/wifi_debug/mib.png)

### 3.9 rc节点

rc节点主要用于查看sta的tx和rx收发包速率情况，在使用该节点前需要先打开rc功能，打开方法：
```
  echo 1 > /sys/kernel/debug/ieee80211/phy0/siwifi/enable_rc
```
通过对于sta的mac地址下有stats和rx_stats节点，其中：
- stats节点可以查看当前sta tx方向的发包速率和成功率；
- rx_stats节点可以查看当前sta rx方向的发包速率和成功率，以及rssi信息；
- fixed_rate节点可以设置固定速率，设置的值为速率的index值；

![rc.png](/assets/images/wifi_debug/rc.png)

**T t p含义说明：**
- T表示拥有最高吞吐率(highest throughput)的速率；
- t表示拥有次高吞吐率(second highest throughput)的速率；
- P则表示拥有最高EWMA probability的速率。

### 3.10 组播转单播节点
multicast_group节点可以查看组播组信息

![group.png](/assets/images/wifi_debug/group.png)

group：表示组播组
member：表示组播组的成员

### 3.11 无线中继节点

![repeater.png](/assets/images/wifi_debug/repeater.png)

第一列包含 vif的ifname、vif的macaddr、当前表项数量、最大sta数量、最近表项更新的时间戳
MAC HASH TABLE：按mac地址的hash值排列表项，分别为表项的hash值、mac地址、ip地址、上一次使用时间，hash值从小到大排列
IP HASH TABLE：按ip地址的hash值排列表项，分别为表项的hash值、mac地址、ip地址、上一次使用时间，hash值从小到大排列

### 3.12 radar调试节点
- radar_debug_printk：该节点是一个可读可写。读写函数都是对siwifi_mod_params下的radar_debug_printk变量进行操作，默认为0。写入1时，开始打印上报的radar脉冲相关信息。
- radar_debugmode：该节点可读写。在处理的radar脉冲符合信道跳转条件时，即使监测到雷达需要通知上层改变信道，但若该变量非0值，则只是打印符合跳转要求但目前为debug模式，不切换频道。
- radar_freq_delta：该节点可读可写，extern一个siwifi_radar.c中定义的全局变量freq_delta并对其进行读写。在siwifi_radar.c的pri_detector_get()获取给定频率和类型的pri_detector时，用于判断pri_detector的freq是否满足传入频率±freq_delta的要求，若不满足则另行创建一个满足该要求的pri_detector。pri_detector初始化为20，表明允许频率误差在±20之间，允许通过向该节点写值改变该增量。
- pulses_prim：该节点可读，列出当前上报的各个类型的有效radar波的统计情况。
- radar_event节点：该节点可读，手动触发一次radar上报后的处理函数。

### 3.13 mpinfo调试节点

![mpinfo.png](/assets/images/wifi_debug/mpinfo.png)

- total:总的统计时间（周期性，周期到后数据清零后重新统计），单位us；
- cca:统计时间周期内，主信道cca busy时间，单位us；
- cca20:统计时间周期内，20MHz从信道的cca busy时间，单位us；
- cca40:统计时间周期内，40MHz从信道的cca busy时间，单位us；
- tx:统计时间周期内，当前band发包占用信道时间，单位us（暂未实现）；
- rx:统计时间周期内，当前band收到数据包占用信道时间，单位us；
- noise0:当前信道第一根天线（LB1或HB1）的噪声，该值为实时值，单位dBm；
- noise1:当前信道第二根天线（LB2或HB2）的噪声，该值为实时值，单位dBm；

noise0、noise1信道噪声亦可通过读寄存器实时获取：
`2.4G：devmem 0x1110b228`
`5G：devmem 0x1790b228`

寄存器bit位含义：
bit 0 - 7 为noise0的值
bit 8 - 15为noise1的值

## 4 系统状态查看

### 4.1 查看内存情况

```
    cat /proc/meminfo
```
- MemTotal：表示总内存大小；
- MemFree：表示空闲内存大小；
- MemAvailable：表示可用内存大小；
- Buffers：表示给文件的缓冲大小；
- Cached：表示高速缓冲存储器使用的大小；
- SwapCached：表示被高速缓冲存储用的交换空间大小；
- Active：表示活跃使用中的高速缓冲存储器页面文件大小；
- Inactive：表示不经常使用的高速缓冲存储器页面文件大小；
- Active(anon)： 表示活跃的匿名内存大小；
- Inactive(anon)：表示不活跃的匿名内存大小；
- Active(file)：表示活跃的file内存大小；
- Inactive(file)：表示不活跃的file内存大小；
- Unevictable：表示不能被释放的内存页大小；
- Mlocked：表示mlock系统调用锁定的内存大小；
- SwapTotal：表示交换空间总大小；
- SwapFree：表示空闲交换空间大小；
- Dirty：表示等待被写回到磁盘的大小；
- Writeback：表示正在被写回的大小；
- AnonPages：表示未映射的页的大小；
- Mapped：表示设备和文件映射的大小；
- Slab：表示内核数据结构缓存的大小；
- SReclaimable：表示可收回slab的大小；
- SUnreclaim：表示不可收回的slab的大小；
- KernelStack：表示内核占用的内存大小；
- PageTables：表示管理内存分页的索引表的大小；
- NFS_Unstable：表示不稳定页表的大小；
- Bounce：表示在低端内存中分配一个临时buffer作为跳转，把位于高端内存的缓存数据复制到此处消耗的内存；
- WritebackTmp：USE用于临时写回缓冲区的内存；
- CommitLimit：表示当前可供用户使用的内存大小；
- Committed_AS：表示当前分配给系统的内存大小；
- VmallocTotal：表示虚拟内存大小；
- VmallocUsed：表示已经被使用的虚拟内存大小；
- VmallocChunk：表示 malloc 可分配的最大的逻辑连续的内存大小；

![meminfo.png](/assets/images/wifi_debug/meminfo.png)

### 4.2 查看进程情况

```
    ps
```
- PID：表示进程的ID；
- USER：表示进程所有者；
- VSZ：表示该进程使用掉的虚拟内存量；
- STAT：表示该进程当前的状态；
  - R ：该程序目前正在运作，或者是可被运作；
  - S ：该程序目前正在睡眠当中 (可说是 idle 状态啦！)，但可被某些讯号(signal) 唤醒。
  - T ：该程序目前正在侦测或者是停止了；
  - Z ：该程序应该已经终止，但是其父程序却无法正常的终止他，造成 zombie (僵尸) 程序的状态
- COMMAND：表示该进程的运行指令；

![ps.png](/assets/images/wifi_debug/ps.png)

### 4.3 查看cpu使用情况

```
    top
```
- 第一行物理内存：
  - used：表示使用中内存大小；
  - free：表示空闲内存大小；
  - shrd：表示共享内存大小；
  - buff：表示缓存内存大小；
  - cached：表示缓冲内存总量大小；
- 第二行CPU状态：
  - usr：表示用户占用CPU百分比；
  - sys：表示系统占用CPU百分比；
  - nic：表示改变过优先级的进程占用CPU百分比；
  - idle：表示CPU空闲率；
  - io：表示io等待占用CPU百分比；
  - irq：表示硬中断占用CPU百分比；
  - sirq：表示软中断占用CPU百分比；
- 第三行进程状态：
  - PID：表示进程id；
  - PPID：表示进程的另一个id；
  - USER：表示进程所有者；
  - STAT：表示进程的状态；
  - VSZ：表示表示该进程使用掉的虚拟内存量；
  - %VSZ：表示表示该进程使用掉的虚拟内存量百分
  - %CPU：表示进程CPU占用百分比；
  - COMMAND：表示进程的实际调用指令；

![top.png](/assets/images/wifi_debug/top.png)

## 5 寄存器调试

### 5.1 寄存器使用方法

- 读：demmev  [寄存器物理地址]
  如：devmem  0x1110b35c

![read.png](/assets/images/wifi_debug/read.png)

- 读：demmev  [寄存器物理地址]  32  [要写入的值]
  如：devmem 0x1110b35c 32 0x00010001

![write.png](/assets/images/wifi_debug/write.png)

### 5.2 调整CCA门限

- 开关动态CCA方法：
```
    echo 9  > /sys/kernel/debug/ieee80211/phy0/siwifi/diags/force_trigger_type
    cat /sys/kernel/debug/ieee80211/phy0/siwifi/diags/mactrace
```

串口中看到ignore cca=1表示关闭，ignore cca=0表示打开；
- 调整CCA门限
  - 20P寄存器：
`2.4G 0x1110b3ac`
`5G 0x1790b3ac`
该寄存器中bit 0 - 7 为CCA20PRISETHRDBM（能量检测Energy Detection，拉高门限）、bit 12 - 19为CCA20PFALLTHRDBM（能量检测Energy Detection，拉低门限）、 bit 20 - 28为INBDCCA20PPOWMINDBM（载波侦听Carrier Sense），通过修改其值去调整门限，注意CCA20PRISETHRDBM要比CCA20PFALLTHRDBM大3个db。

  - 20S寄存器：
  `2.4G 0x1110b3d4`
  `5G 0x1790b3d4`
该寄存器中bit 0 - 7 为CCA20SRISETHRDBM（能量检测Energy Detection，拉高门限）、 bit 12 - 19为CCA20SFALLTHRDBM（能量检测Energy Detection，拉低门限）、 bit 20 - 28为INBDCCA20SPOWMINDBM（载波侦听Carrier Sense），通过修改其值去调整门限，注意CCA20SRISETHRDBM要比CCA20SFALLTHRDBM大3个db。

  - 40S寄存器：
  `2.4G 0x1110b3d8`
  `5G 0x1790b3d8`
该寄存器中bit 0 - 7 为CCA40SRISETHRDBM（能量检测Energy Detection，拉高门限）、 bit 12 - 19为CCA40SFALLTHRDBM（能量检测Energy Detection，拉低门限）、 bit 20 - 28为INBDCCA40SPOWMINDBM（载波侦听Carrier Sense），通过修改其值去调整门限，注意CCA40SRISETHRDBM要比CCA40SFALLTHRDBM大3个db。

- 注意
带宽为20M时调整20P相关能量检测阈值；
带宽为40M时调整20P和20S相关能量检测阈值；
带宽为80M时调整20P、20S和40S相关能量检测阈值；

将2.4G对应的值上调，示例
![CCA.png](/assets/images/wifi_debug/CCA.png)

- 关闭CCA检测：
```
/** 关闭2.4G CCA检测 */
devmem 0x1110b3b0 32 0x0
/** 关闭5G CCA检测 */
devmem 0x1790b3b0 32 0x0
```

### 5.3 动态降带宽

相关寄存器：
`2.4G 0x11080310`
`5G 0x17880310`
该寄存器中bit 3为dropToLowerBW，通过修改其值来打开和关闭降带宽功能，默认关闭，设为1为开启，设为0为关闭。
示例:

![bw.png](/assets/images/wifi_debug/bw.png)

### 5.4 调整TX传输

相关寄存器：
`2.4G 0x11100860`
`5G 0x17900860`
该寄存器中 bit 20 - 23为20BW的时域窗口、 bit 24 - 27为40BW的时域窗口、 bit28 - 31为80BW的时域窗口，针对当前ap的带宽调节相应的时域窗口（数值越大，吞吐应越高），查看对tx有无改善。
将2.4G 20BW时的时域窗口从0改为1，示例:

![tx.png](/assets/images/wifi_debug/tx.png)

### 5.5 调整TX EDCA参数

1，手动调整前需要关闭动态edca调整，以2.4G为例：
```
    echo 1  > /sys/kernel/debug/ieee80211/phy0/siwifi/disable_wmm_edca
```
- 调整BE队列的edca参数为0x211：
```
    devmem 0x11080204 32 0x211
```

- BK队列：
2.4G 寄存器地址：0x11080200
5G 寄存器地址：0x17880200
寄存器各个bit位值含义：

```c
/**
 * @name EDCA_AC_0 register definitions
 * <pre>
 *   Bits           Field Name   Reset Value
 *  -----   ------------------   -----------
 *  27:12           txOpLimit0   0x0
 *  11:08               cwMax0   0xA
 *  07:04               cwMin0   0x4
 *  03:00               aifsn0   0x7
 * </pre>
 *
 * @{
 */
 ```

- BE队列：
2.4G 寄存器地址：0x11080204
5G 寄存器地址：0x17880204
寄存器各个bit位值含义：

```c
/**
 * @name EDCA_AC_1 register definitions
 * <pre>
 *   Bits           Field Name   Reset Value
 *  -----   ------------------   -----------
 *  27:12           txOpLimit1   0x0
 *  11:08               cwMax1   0xA
 *  07:04               cwMin1   0x4
 *  03:00               aifsn1   0x3
 * </pre>
 *
 * @{
 */
 ```

- VI队列：
2.4G 寄存器地址：0x11080208
5G 寄存器地址：0x17880208
寄存器各个bit位值含义：

 ```c
 /**
 * @name EDCA_AC_2 register definitions
 * <pre>
 *   Bits           Field Name   Reset Value
 *  -----   ------------------   -----------
 *  27:12           txOpLimit2   0x5E
 *  11:08               cwMax2   0x4
 *  07:04               cwMin2   0x3
 *  03:00               aifsn2   0x2
 * </pre>
 *
 * @{
 */
```

- VO队列：
2.4G 寄存器地址：0x1108020c
5G 寄存器地址：0x1788020c
寄存器各个bit位值含义：

```c
/**
 * @name EDCA_AC_3 register definitions
 * <pre>
 *   Bits           Field Name   Reset Value
 *  -----   ------------------   -----------
 *  27:12           txOpLimit3   0x2F
 *  11:08               cwMax3   0x3
 *  07:04               cwMin3   0x2
 *  03:00               aifsn3   0x2
 * </pre>
 *
 * @{
 */

```


## 6 启动脚本

### 6.1 RTS、CTS选择
修改/lib/sfwifi.sh脚本中的rts_cts_change值（当前默认值为2）可以控制默认是否发送CTS和RTS

![rts-cts.png](/assets/images/wifi_debug/rts-cts.png)

- 0：表示不发送rts/cts
- 1：表示发送self_cts
- 2：表示发送rts/cts

### 6.2 TX AMPDU 最大聚合包个数
修改/lib/sfwifi.sh脚本中的ampdu_max_cnt值（当前默认值为32）可以控制AMPDU最大聚合包数目
注意：受芯片端空间限制，当前最大只能支持32个。

### 6.3 AMSDU 最大聚合包个数
TX方向最大支持数为4，RX方向最大支持数为6.
2.4G修改路径`/sys/module/sf16a18_lb_fmac/parameters/amsdu_maxnb`
5G修改路径`/sys/module/sf16a18_hb_fmac/parameters/amsdu_maxnb`

![amsdu-nb.png](/assets/images/wifi_debug/amsdu-nb.png)

## FAQ