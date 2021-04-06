---
layout: post
title: HNAT对接和使用手册
categories: DEVELOP
description: HNAT对接和使用手册
keywords:  plan
mermaid: true
---

# HNAT对接和使用手册

**目录**

* TOC
{:toc}

## 1 介绍

- 适用人员

本项目适用于对网络以及Linux内核有基本了解的人员。

- 开发与测试环境

需要带hnat模块的芯片或者FPGA环境，镜像需要选中hnat模块驱动。

- nat简介

nat是网络地址转换的缩写，是IP数据包在通过路由器或防火墙时重写来源IP地址或目的IP地址的技术。这种技术主要用于多台主机只通过一个公有IP地址访问互联网的情况。nat又分为SNAT和DNAT两种，其中SNAT是源地址转换，DNAT是目的地址转换。SNAT是当内部主机要访问公网上的服务时，内部主机会主动发起连接，并由路由器或防火墙将内部地址（源地址）转换成公有ip。DNAT是路由器或防火墙的网关接收到公网返回的数据，将目的地址从路由器或防火墙的网关转换为内部主机的ip，下图是一个nat转换的例子。

![jekyll_exec](/assets/images/hnat_img/nat.png)

- hnat功能概述

hnat是硬件网络地址转换的缩写，顾名思义，就是使用硬件进行数据的nat转换。当一条tcp或udp连接建立之后，系统会通过hnat驱动将该连接nat转换所需的信息下发到hnat模块，从而由hnat模块完成后续数据包的nat转换，从而达到千兆级的转换速度。

## 2 项目引用

- hnat驱动对接参考redmine#[6775](http://redmine.siflower.cn/redmine/issues/6775)

## 3 开发详情

### 3.1 hnat主要功能

1. 支持以太网加速

hnat模块可以将转换后的数据包上送host或重新发出，通常情况下hnat在处理完来自switch的数据包之后（wan/lan），会转发回switch（lan/wan），这就是以太网部分的加速功能。

2. 支持wifi加速功能

根据配置，hnat模块同样可以对来自wifi侧的数据包进行snat加速，将wifi的数据包发送到wan。将wan->wifi的数据包dnat转换后从switch上送到host，再由host发送给wifi，这样可以hnat模块就可以实现对wifi的加速，提高wifi的性能。

3. 根据配置信息进行nat转换

hnat模块可以识别ETH II、VLAN、PPPOE、VLAN+PPPOE四种类型的数据包，从而可以支持不同场景下的应用。根据驱动配置的信息，hnat可以根据原数据包中的ip/port等信息判断是否需要进行SNAT或DNAT。当需要进行SNAT时，hnat模块会修改原数据包中sip、sport、dmac、smac信息，当需要进行DNAT时，hnat模块会修改原数据包中dip、dport、dmac、smac信息，从而与协议栈处理结果相同，可以做到正常通信与加速功能。

4. 支持ip分片数据包

hnat模块可以支持对ip分片的数据包进行nat处理，可以处理顺序/乱序情况下的多分片数据包（2分片及以上）。ip分片是为了处理ip数据包长度大于网卡mtu的情况，其中tcp协议具有分片功能，因此tcp数据包一般没有ip分片的情况，ip分片主要在udp数据包中出现。

5. 支持不同模式

hnat模块通过设置可以支持不同模式的nat，包括Basic NAT、Symmetric NAT、Restricted cone NAT、Port-Restricted cone NAT、Full cone NAT，根据不同的应用场景可以选择不同的nat模式。

6. 支持最大1024条NAPT表和512条MAC表

hnat模块支持1024条NAPT表和512条MAC表，可以同时对1024条不同ip/port的数据流进行加速，最大可以支持512个不同的设备mac地址。

7. fpga上吞吐测试的性能

由于fpga的cpu频率限制，没有hnat加速时，nat转换速率在15Mbps左右，当使用hnat加速之后，速率有明显提升。（fpga上的网卡接口最大只支持100M模式，所以iperf测试数据最大为95Mbps）

| | 未加速 | hnat加速 | 
| ------- | ------ | ----- | 
| lan-wan吞吐 | 15Mbps | 95Mbps | 
| wan-lan吞吐 | 12Mbps | 95Mbps | 
| lan-wan吞吐(pppoe) | 10Mbps | 95Mbps | 
| wan-lan吞吐(pppoe) | 8Mbps | 95Mbps | 
| lan-wan吞吐(分片) | 10Mbps | 95Mbps | 
| lan-wan吞吐(10条流) | 10Mbps | 95Mbps | 

详细测试数据可以参考redmine#[6767](http://redmine.siflower.cn/redmine/issues/6767)

### 3.2 驱动对接iptable规则

1. 配置iptable flowoffload规则

Netfilter是Linux内核中的一个软件框架，不仅有nat的功能，也具备数据包内容修改、以及数据包过滤等防火墙功能。iptables是位于用户空间的应用软件，通过控制Linux内核netfilter的模块，来实现网络数据包的处理和转发。通过命令配置flowoffload规则

```
iptables -I FORWARD 1 -m conntrack --ctstate NEW,RELATED,ESTABLISHED -j FLOWOFFLOAD --hw
```

在网卡驱动接口函数中（net_device_ops）可以注册驱动的offload函数，这样当一条数据流建立起来之后，内核会调用驱动中对应的flowoffload函数，将数据流nat所需的信息下发到驱动中，从而完成hnat模块相关的配置。

```
static const struct net_device_ops sgmac_netdev_ops = {
    .../*其他函数*/
    .ndo_flow_offload_check = sgmac_ndo_flow_offload_check,//flowoffload检查函数
    .ndo_flow_offload = sgmac_ndo_flow_offload,//flowoffload配置函数
};
```

如果要将信息配置到hnat模块中，则需要使用hnat提供的注册函数sf_hnat_ops_register获取hnat的接口，并调用hnat中的offload接口将nat信息传递给hnat模块。详细请参考redmine#[6775](http://redmine.siflower.cn/redmine/issues/6775)

```
int sgmac_ndo_flow_offload(enum flow_offload_type type,
        struct flow_offload *flow,
        struct flow_offload_hw_path *src,
        struct flow_offload_hw_path *dest)
{
#ifdef CONFIG_SFAX8_HNAT_ENABLE
	g_priv->hnat_out_ops->ndo_flow_offload(type, flow, src, dest);//将网卡驱动得到的nat信息传递给hnat模块
#endif
    return 0;
}
```

2. 钩子函数

Netfilter框架中制定了五个数据包的钩子点，分别是PRE_ROUTING、INPUT、OUTPUT、FORWARD与POST_ROUTING。由于路由器主要功能是转发，因此路由器中数据包主要经过PRE_ROUTING、FORWARD与POST_ROUTING，输入与输出（INPUT与OUTPUT）较少使用。flowoffload规则配置在FORWARD中，当数据包经过FORWARD点时，符合规则的数据包相关的nat信息就会被配置到驱动中。

![jekyll_exec](/assets/images/hnat_img/netfilter.png)

3. udp测试

详情参考redmine#[6897](http://redmine.siflower.cn/redmine/issues/6897)

### 3.3 debug节点

#### 3.3.1 使用方法
在sf_hnat_debug.c中，sf_hnat_debug_write函数实现了具备debug等功能的节点。在串口执行以下指令：

```
echo help > /sys/kernel/debug/hnat_debug
```

即可获取到当前debug节点具备的功能及使用说明：

```
echo natmode <mode> >  /sys/kernel/debug/gmac_debug
intro: 0-BASIC_MODE, 1-SYMMETRIC_MODE, 2-FULL_CONE_MODE, 3-RESTRICT_CONE_MODE, 4-PORT_RESTRICT_CONE_MODE
echo rd_speed >  /sys/kernel/debug/hnat_debug
echo readl [addr] [number] >  /sys/kernel/debug/hnat_debug
echo writel [addr] [data] >  /sys/kernel/debug/hnat_debug
echo tabread [tab_no] [depth] >  /sys/kernel/debug/hnat_debug
echo tabwrite [tab_no] [depth] [data(5)] >  /sys/kernel/debug/hnat_debug
intro: dump all entry of table and crc table, 1 for napt, 0 for arp
echo dump <napt/arp> >  /sys/kernel/debug/hnat_debug
echo stat  >  /sys/kernel/debug/hnat_debug
intro: add/del lan ip and netmask to register incording to register_num
echo addlan [register_num] [addr] [netmask] >  /sys/kernel/debug/hnat_debug
demo: echo addlan 2 0xc0a80400 0xffffff00 > /sys/kernel/debug/hnat_debug
echo dellan [register_num] >  /sys/kernel/debug/hnat_debug
demo: echo dellan 2 > /sys/kernel/debug/hnat_debug
intro: show lan ip and netmask in all 8 registers
echo getlan >  /sys/kernel/debug/hnat_debug
intro: update wan masklen to hnat and get wanmasklen
echo updatewanmask [wan_masklen] > /sys/kernel/debug/hnat_debug
demo: echo updatewanmask 24 > /sys/kernel/debug/hnat_debug
echo getwanmask > /sys/kernel/debug/hnat_debug
```

根据说明执行echo指令，就可以调用debug节点中实现的函数及功能。以下三节详细介绍了利用debug节点更新wan/lan信息的功能。

#### 3.3.2 lan网段ip更新

在hnat.c中，用HWNAT_REG16_CSR到HWNAT_REG23_CSR共8组寄存器存放最8组lan ip的值，HWNAT_REG30_CSR和HWNAT_REG31_CSR共2组寄存器存放对应ip的masklen。debug节点提供了删除和添加lan ip+masklan，及查看所有lan ip和mask的指令。

- 在第i个寄存器添加lan信息(i表示寄存器编号，lanhex maskhax为 IP 和mask的十六进制数，echo时会添加0x头)。例如将ip为192.168.4.110，mask为255.255.255.0的lan信息更新到第二个寄存器：

```
echo addlan $i 0x$lanhex 0x$maskhax > /sys/kernel/debug/hnat_debug
e.g.
echo addlan 2 0xc0a8046e 0xffffff00 > /sys/kernel/debug/hnat_debug
```

- 删除第i个寄存器存放的lan信息，例如删除第二个寄存器的lan信息：

```
echo dellan $i > /sys/kernel/debug/hnat_debug 删除第i个寄存器存放的lan信息
e.g.
echo dellan 2 > /sys/kernel/debug/hnat_debug
```

- 查看所有lan ip和mask信息：

```
echo getlan > /sys/kernel/debug/hnat_debug
```

#### 3.3.3 wanmasklen更新

hnat中还需要知道当前wan口的masklen，并用phnat_priv结构体的wan_masklen元素储存。

- 查看当前wanmasklen

```
echo getwanmask > /sys/kernel/debug/hnat_debug
```

- 更新wan口的masklen

```
echo updatewanmask $wan_masklen > /sys/kernel/debug/hnat_debug
e.g.
echo updatewanmask 24 > /sys/kernel/debug/hnat_debug
```

#### 3.3.4 脚本实例

- **05-updatehnatlan**

在/etc/hotplug.d/iface/05-updatehnatlan脚本中，每当network启动或关闭时，所有interface up和down的信息都会传入/etc/hotplug.d/iface路径下的脚本中，以参数INTERFACE（包括lan、wan等）和ACTION（包括up和down）表示。通过这些信息筛选出lan和guset，并且在up时读取它们的ip和mask信息传入debug节点，down时通过debug节点删除信息，即可实现lan ip实时更新。05-updatehnatlan共支持7个lan和1个guest lan的更新。

- **80-updatewanmask**

与05-updatehnatlan同理，当INTERFACE为wan或wwan时，ACTION为up时，用ubus指令获取到当前wan口masklen，如果非0则进行更新，如果为masklen=32则减一，用masklen=31进行更新。


## 4 测试用例

- hnat测试环境与用例（目前为FPGA的测试环境与方法）

hnat的测试主要有两种方法，一种使用iperf进行速率测试，一种使用gmac发包工具进行测试。使用iperf进行测试时，环境为：fpga的gmac接口外接交换机，lan/wan分别接pc，wan pc做为iperf server，lan pc做为iperf client，进行iperf测试。gmac发包工具测试时，环境为：fpga的gmac接口对接发包工具，工具发送不同类型的数据包，检查转换结果和速率。具体测试用例和结果可以参考redmine#[6767](http://redmine.siflower.cn/redmine/issues/6767)

- TODO

a28芯片的情况下的测试环境和用例

## 5 FAQ
