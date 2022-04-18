---
layout: post
title: HNAT介绍
categories: DEVELOP
description: HNAT介绍
keywords:  plan
mermaid: true
---

# HNAT介绍

**目录**

* TOC
{:toc}


## 1 适用人员

本项目适用于对网络以及Linux内核有基本了解的人员。

## 2 开发与测试环境

需要带hnat模块的芯片的开发板环境或者FPGA环境，镜像需要选中hnat模块驱动。

## 3 相关背景

- nat是网络地址转换的缩写，是IP数据包在通过路由器或防火墙时重写来源IP地址或目的IP地址的技术。这种技术主要用于多台主机只通过一个公有IP地址访问互联网的情况。nat又分为SNAT和DNAT两种，其中SNAT是源地址转换，DNAT是目的地址转换。SNAT是当内部主机要访问公网上的服务时，内部主机会主动发起连接，并由路由器或防火墙将内部地址（源地址）转换成公有ip。DNAT是路由器或防火墙的网关接收到公网返回的数据，将目的地址从路由器或防火墙的网关转换为内部主机的ip，下图是一个nat转换的例子。

![jekyll_exec](/assets/images/hnat_img/nat.png)

- hnat是硬件网络地址转换的缩写，顾名思义，就是使用硬件进行数据的nat转换。当一条tcp或udp连接建立之后，系统会通过网络驱动接口，将该连接所需的nat转换信息下发到hnat模块，从而由hnat模块完成后续数据包的nat转换，从而达到千兆级的转换速度。

## 4 功能概述

本文主要描述了HNAT硬件上的结构位置，实现的功能和性能，对外开放的接口，对接调试方法和测试补充等.

## 5 硬件结构

HNAT整个子系统位于GMAC模块中，在芯片中的位置如下:
![jekyll_exec](/assets/images/hnat_img/hnat_strcuture.png)


## 6 实现功能

HNAT子系统主要针对GMAC外挂千兆交换芯片的应用场景，实现LAN和WAN通信报文的NAT转换，并对转换后的报文做转发处理，发往HOST或者GE_Switch，注意本子系统只对符合ETH II，802.3，VLAN，PPPOE格式的，并且协议为UDP、TCP的IPV4报文做NAT处理，其它报文上送HOST处理；

支持的数据流向如下:
![jekyll_exec](/assets/images/hnat_img/hnat_flow.png)

- LAN（GE）<-->HOST ②<-->③<-->④ 
- LAN（GE）<-->WAN（GE） ②<-->③<-->①  需要做SNAT和DNAT
- LAN（GE）<-->WLAN ②<-->③<-->④<-->⑥
- WAN（GE）<-->HOST ①<-->③<-->④
- WAN（GE）<--> WLAN ①<-->③<-->④<-->⑥ 需要做SNAT和DNAT

由于NPU在最终的A28项目中删掉了，因为包含⑤的数据流将不在此列出．从上图的数据流向可以看出，HNAT完全支持以太网加速, WIFI加速场景．不仅如此其还支持众多Feature，如下：

* 支持以太网加速
  * hnat模块可以将转换后的数据包上送host或重新发出，通常情况下hnat在处理完来自switch的数据包之后（wan/lan），会转发回switch（lan/wan），这就是以太网部分的加速功能。
* 支持wifi加速功能
  * 根据配置，hnat模块同样可以对来自wifi侧的数据包进行snat加速，将wifi的数据包发送到wan。将wan->wifi的数据包dnat转换后从switch上送到host，再由host发送给wifi，这样可以hnat模块就可以实现对wifi的加速，提高wifi的性能。
* 支持不同HNAT模式；
  * hnat模块通过设置可以支持不同模式的nat，包括Basic NAT、Symmetric NAT、Restricted cone NAT、Port-Restricted cone NAT、Full cone NAT，根据不同的应用场景可以选择不同的nat模式。 
* 支持ip分片数据包；
  * hnat模块可以支持对ip分片的数据包进行nat处理，可以处理顺序/乱序情况下的多分片数据包（2分片及以上）。ip分片是为了处理ip数据包长度大于网卡mtu的情况，其中tcp协议具有分片功能，因此tcp数据包一般没有ip分片的情况，ip分片主要在udp数据包中出现。
* 支持最大1024条NAPT表和512条DIP表以及128条MAC表；
  * HNAT子系统最大支持1024条NAPT表项，512个不同的destination IP以及128条不同的MAC表项，由于NAPT和DIP数据采用hash压缩算法存储，因此由于hash冲突，存在较小的概率情况下，实际能支持的条目数小于1024条NAPT表项或512条destination IP表项。
* 根据配置信息进行nat转换
  * hnat模块可以识别ETH II、VLAN、PPPOE、VLAN+PPPOE四种类型的数据包，从而可以支持不同场景下的应用。根据驱动配置的信息，hnat可以根据原数据包中的ip/port等信息判断是否需要进行SNAT或DNAT。当需要进行SNAT时，hnat模块会修改原数据包中sip、sport、dmac、smac信息，当需要进行DNAT时，hnat模块会修改原数据包中dip、dport、dmac、smac信息，从而与协议栈处理结果相同，可以做到正常通信与加速功能。
 
## 7 性能测试

HNAT性能测试包含日常Release版本的IxChariot/Iperf性能测试以及打流仪2544性能标定的测试，对应版本的测试报告见redmine#[9045](http://redmine.siflower.cn/redmine/issues/9045)，摘取部分测试结果如下：
|　测试项　| 未加速 | hnat加速 | 
| ------- | ------ | ----- | 
| lan-wan吞吐 | 275Mbps | 932Mbps | 
| wan-lan吞吐 | 203Mbps | 936Mbps | 


## 8 对接接口

### 8.1 Openwrt系统对接接口

### 8.1.1 使能/关闭HNAT

在Openwrt系统中可以通过修改firewall配置文件来使能/关闭HNAT，详细配置如下：
```
root@OpenWrt:/# cat /etc/config/firewall 

config defaults
        option syn_flood '1'
        option input 'ACCEPT'
        option output 'ACCEPT'
        option forward 'REJECT'
        option flow_offloading '1'
        option flow_offloading_hw '1'
        ...
```
其中flow_offloading/flow_offloading_hw对应hnat配置规则，将其置'0'表示关闭，默认为'1'表示开启．
flow_offloading是Linux内核netfilter模块的配置选项，Netfilter是Linux内核中的一个软件框架，不仅有nat的功能，也具备数据包内容修改、以及数据包过滤等防火墙功能，通过控制Linux内核netfilter的模块，来实现网络数据包的处理和转发．

### 8.1.2 LAN口IP发生变化时，HNAT配置自动更新

LAN口IP被用做hnat判断报文是否需要转发（内网到外网）的依据，也是在DNAT时用做替换报文的sip．因此在LAN口IP发生变化时，HNAT配置信息需要同步更新．这在Openwrt中通过hnat_update_interface.sh向驱动更新LAN口名称来实现．在网络启动或重启时，脚本会调用HNAT驱动提供的debugfs接口将LAN口名称更新到驱动中去，驱动会根据名称自动跟踪IP的变化。
详细对应的hnat_update_interface.sh脚本位置如下:
```
root@OpenWrt:/# ls /usr/bin/hnat_update_interface.sh
/usr/bin/hnat_update_interface.sh
```

### 8.1.3 WAN口IP发生变化时，HNAT配置自动更新

WAN口IP与NETMASK被用做hnat判断报文是否需要转发（内网到外网）的依据．因此WAN口IP发生变化时，HNAT配置信息也需要同步更新．这在Openwrt中通过hnat_update_interface.sh向驱动更新WAN口名称来实现．在网络启动或重启时，脚本会调用HNAT驱动提供的debugfs接口将WAN口名称更新到驱动中去，驱动会根据名称自动跟踪IP的变化。
详细对应的hnat_update_interface.sh脚本位置如下:
```
root@OpenWrt:/# ls /usr/bin/hnat_update_interface.sh
/usr/bin/hnat_update_interface.sh
```

### 8.2 HNAT驱动对接接口

在网卡驱动接口函数中（net_device_ops）可以注册驱动的offload函数，这样当一条数据流建立起来之后，内核会调用驱动中对应的flowoffload函数，将数据流nat所需的信息下发到驱动中，从而完成hnat模块相关的配置。

以太网驱动中对应的flowoffload接口如下:

```
static const struct net_device_ops sgmac_netdev_ops = {
    .../*其他函数*/
    .ndo_flow_offload_check = sgmac_ndo_flow_offload_check,//flowoffload检查函数
    .ndo_flow_offload = sgmac_ndo_flow_offload,//flowoffload配置函数
};
```

如果要将信息配置到hnat模块中，则需要使用hnat提供的注册函数sf_hnat_ops_register获取hnat的接口，并调用hnat中的offload接口将nat信息传递给hnat模块。详细请参考redmine#[6775](http://redmine.siflower.cn/redmine/issues/6775)

以太网驱动中调用hnat offload接口实现如下:

```
int sgmac_ndo_flow_offload(enum flow_offload_type type,
				struct flow_offload *flow,
				struct flow_offload_hw_path *src,
				struct flow_offload_hw_path *dest) {
			struct net_device * pndev = src->dev;


			struct sgmac_priv *priv = netdev_priv(pndev);
			return priv->phnat_priv->ndo_flow_offload(priv->hnat_pdev, type, flow, src, dest);//将网卡驱动得到的nat信息传递给hnat模块
}
```

### 8.3 HNAT内核对接接口

在Linux内核4.14版本以后就支持硬件nat功能，其主要通过Netfilter框架实现．Netfilter框架中制定了五个数据包的钩子点，分别是PRE_ROUTING、INPUT、OUTPUT、FORWARD与POST_ROUTING。由于路由器主要功能是转发，因此路由器中数据包主要经过PRE_ROUTING、FORWARD与POST_ROUTING，输入与输出（INPUT与OUTPUT）较少使用。flowoffload规则配置在FORWARD中，当数据包经过FORWARD点时，符合规则的数据包相关的nat信息就会被配置到驱动中。

![jekyll_exec](/assets/images/hnat_img/netfilter.png)

对应实现的具体代码目录如下：
linux-4.14.90-dev/linux-4.14.90/net/netfilter/

以下为HNAT异步增删处理，简要介绍，详情请参考[HNAT异步增删流程]()  
两个内核 workqueue 用于异步处理表项:  
1、 flow_offload_hw_work 为内核添加和删除硬加速表项的 workqueue。  
2、nf_flow_offload_work_gc 为内核遍历整个加速表的 workqueue。  
如果某条软加速表项标记了 TEARDOWN ，内核会调用 flow_offload_del 删除。如果要删除的是硬加速表项，内核会先使用 nf_flow_offload_hw_del 删除硬件表项，然后在下一次 gc 中根据标志位决定是否删除软件表项（只有确定硬加速表项删除后才会删除软加速表项）  
异步处理后，驱动只需要标记该软加速表项为 HW_DEAD ，然后等待超时后由内核 gc workqueue 删除软加速表项。

## 9 HNAT对接调试方法

Siflower通过debugfs节点暴露出去了一套私有Debugfs接口，专门用于HNAT的Debug调试．详细使用方法见示例．

**示例：**

- 获取所有接口的帮助信息:  
  命令:  ```echo help > /sys/kernel/debug/hnat_debug```  
  结果展示如下:  

  ```
  echo help > sys/kernel/debug/hnat_debug
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
  echo log mode  >  /sys/kernel/debug/hnat_debug
  intro: mode:0 for disbable hnat log, mode:1 for enable hnat log
  echo addifname [is_wan] [index] [ifname] >  /sys/kernel/debug/hnat_debug
  intro: add interface to monitor ip change. is_wan 0 for lan 1 for wan
  demo: echo addifname 0 1 br-lan2 > /sys/kernel/debug/hnat_debug
  echo addlan [index] [addr] [prefix_len] <ifname> >  /sys/kernel/debug/hnat_debug
  echo addwan [index] [addr] [prefix_len] <ifname> >  /sys/kernel/debug/hnat_debug
  echo dellan [index] <if_remove_name> >  /sys/kernel/debug/hnat_debug
  echo delwan [index] <if_remove_name> >  /sys/kernel/debug/hnat_debug
  echo getlan > /sys/kernel/debug/hnat_debug
  echo getwan > /sys/kernel/debug/hnat_debug

  ```

展示几个最主要的debugsf接口使用方法如下:

- 开启hnat debug log
  hnat debug log主要打印hnat entry添加/删除时候的信息以及辅助判断的诊断信息.  
  开启命令:  ```echo log mode(mode == 1/2/3) > /sys/kernel/debug/hnat_debug```  
  关闭命令:  ```echo log 0 > /sys/kernel/debug/hnat_debug```   
  结果展示如下:  

  ```
    root@OpenWrt:/# echo log 1 > /sys/kernel/debug/hnat_debug
	  root@OpenWrt:/# [ 7870.644711] [hnat info] is_inat 1 add tbl 1 index 362 hash ptr 0
	  [ 7870.650846] [hnat info] is_inat 0 add tbl 1 index 100 hash ptr 0
	  [ 7870.656902] add napt 3 valid 1
  	root@OpenWrt:/# echo log 0 > /sys/kernel/debug/hnat_debug

	  root@OpenWrt:/# echo log 2 > /sys/kernel/debug/hnat_debug
	  root@OpenWrt:/# [ 7910.646568] hnat offload ADD type=0 
	  [ 7910.650178] !!!!!!!!!!!!!!!!!!!!!flow_offload flags=0x49 	timeout=0xba8a0 l4 proto= 0x6 l3proto=0x 
	  [ 7910.659542] ORIGINAL: src:[192.168.6.100:44480] -> dest:[23.29.105.232:80]
	  [ 7910.666540] REPLY: src:[23.29.105.232:80] -> dest:[192.168.5.203:44480]
	  [ 7910.673156] src=========
	  [ 7910.675723] ==================hw path flags=0x3 vlan_proto=0x81 vlan_id=0x1 pppoe_sid=0x0 devname0
  	[ 7910.685061]  src_mac=00:9d:7d:86:20:08 dst_mac=b0:83:fe:9a:2e:f4
	  [ 7910.691384] dest=========
	  [ 7910.694151] ==================hw path flags=0x3 vlan_proto=0x81 vlan_id=0x2 pppoe_sid=0x0 devname0
	  [ 7910.703711]  src_mac=00:9d:7d:86:20:09 dst_mac=a8:5a:f3:00:3a:98 
	  [ 7910.710237] [hnat notice]sf_hnat_search_vlan search success index 0 is_add 1 vlan_id 1
	  [ 7910.718230] [hnat notice]sf_hnat_search_vlan search success index 1 is_add 1 vlan_id 2
	  [ 7910.726292] [hnat notice]sf_hnat_search_rt_pub_net search success index 0
	  [ 7910.733148] add napt 3 valid 1
	  [ 7910.736287] [hnat notice]sf_hnat_search_dip search success index 0, ref_count 14
	  [ 7910.743707] [hnat notice]sf_hnat_search_dip search success index 1, ref_count 14
	  [ 7920.565252] [hnat log]  del napt index  3
	  [ 7920.569316] del napt  3 valid 0
	  root@OpenWrt:/# echo log 0 > /sys/kernel/debug/hnat_debug

	  root@OpenWrt:/# echo log 8 > /sys/kernel/debug/hnat_debug
	  root@OpenWrt:/# [ 8049.053989] [hnat log] get lansubnet 0 prefix 24 ip 6406a8c0
	  [ 8049.059797] [hnat log] get wansubnet 0 prefix 24 ipcb05a8c0
	  root@OpenWrt:/# echo log 0 > /sys/kernel/debug/hnat_debug
  ```

- 读取hnat转换计数
  
  读取hnat SNAT和DNAT报文计数以及dump当前hnat部分寄存器值．  
  命令: ```echo stat > /sys/kernel/debug/hnat_debug```  
  结果展示如下:

  ```
    root@OpenWrt:/# echo stat > sys/kernel/debug/hnat_debug 
    [99013.647334] napt full count 0 udp aging count 0 napt hash full 0 dip hash full 0
    [99013.654895] CSR REGISTERS START:
    [99013.658154] addr 0x4000:data 0x000d5550  addr 0x4004:data 0x0240004f  addr 0x4008:data 0x00000011  addr 0x400c:data 0x00400000
    [99013.658168] addr 0x4010:data 0x00000064  addr 0x4014:data 0x00000100  addr 0x4018:data 0x00000100  addr 0x401c:data 0x00000000
    [99013.669753] addr 0x4020:data 0x00000080  addr 0x4024:data 0x00000080  addr 0x4028:data 0x00440040  addr 0x402c:data 0x00000020
    [99013.681410] addr 0x4030:data 0x00000000  addr 0x4034:data 0x012c067e  addr 0x4038:data 0x00000000  addr 0x403c:data 0x00000000
    [99013.692982] addr 0x4040:data 0xc0a80400  addr 0x4044:data 0x00000000  addr 0x4048:data 0x00000000  addr 0x404c:data 0x00000000
    [99013.704546] CSR REGISTERS END:
    [99013.719199] COUNTER REGISTERS START:
    [99013.722986] addr 0x4100:data 0x00000c78  addr 0x4104:data 0x00000000  addr 0x4108:data 0x00000c78  addr 0x410c:data 0x00000000
    [99013.723009] addr 0x4110:data 0x00000000  addr 0x4114:data 0x00000c78  addr 0x4118:data 0x00000000  addr 0x411c:data 0x00000c78
    [99013.734596] addr 0x4120:data 0x00000000  addr 0x4124:data 0x00000000  addr 0x4128:data 0x00000000  addr 0x412c:data 0x00000149
    [99013.746213] addr 0x4130:data 0x00000149  addr 0x4134:data 0x00000149  addr 0x4138:data 0x00000000  addr 0x413c:data 0x00000149
    [99013.757768] addr 0x4140:data 0x00000000  addr 0x4144:data 0x0000000e  addr 0x4148:data 0x00000000  addr 0x414c:data 0x00000149
    [99013.769313] addr 0x4150:data 0x00000000  addr 0x4154:data 0x00000149  addr 0x4158:data 0x00000000  addr 0x415c:data 0x00000149
    [99013.780906] addr 0x4160:data 0x00000149  addr 0x4164:data 0x00000000  addr 0x4168:data 0x00000000  addr 0x416c:data 0x0000001e
    [99013.792542] addr 0x4170:data 0x00000000  addr 0x4174:data 0x00000374  addr 0x4178:data 0x00000000  addr 0x417c:data 0x00000000
    [99013.804146] addr 0x4180:data 0x00000000  addr 0x4184:data 0x00000000  addr 0x4188:data 0x00000000  addr 0x418c:data 0x00000000
    [99013.815745] addr 0x4190:data 0x00000000  addr 0x4194:data 0x00000099  addr 0x4198:data 0x00000000  addr 0x419c:data 0x00000000
    [99013.827324] COUNTER REGISTERS END:
    [99013.842488]  GMAC reciv all 3192 pkts 
    [99013.846274]  rx 0 pkts need to snat
    [99013.849772]  rx 884 pkts need to dnat
    [99013.853610]  tx 30 pkts need to snat
    [99013.857239]  0 pkts are hit to GMAC
    [99013.860737]  host send 329 pkts to hnat
    [99013.864723] ===============RX=============
    [99013.868866] rx_enter_sof_cnt        3192                , rx_enter_eof_cnt         3192                
    [99013.878412] rx_2host_frame_sof_cnt  3192                , rx_2host_frame_eof_cnt   3192                
    [99013.888005] rx_enter_drop_cnt       0                   , rx2tx_data_frame_cnt     0                   
    [99013.897560] rx_enat_cnt             0                   , rx_inat_cnt              884                 
    [99013.907173] rx2tx2host_cnt          0                   , rx2tx_drop_cnt           153                 
    [99013.916745] ===============TX=============
    [99013.920892] tx_sof_frame_cnt        329                 , tx_eof_frame_cnt         329                 
    [99013.930508] tx_exit_sof_cnt         329                 , tx_exit_eof_cnt          329                 
    [99013.940132] tx_receive_rx_frame_cnt 0                   , tx_nohits_frame_cnt      14                  
    [99013.949713] Hnat_rcv_status_cnt     329                 , Hnat_tx_status_cnt       0                   
    [99013.959303] Hnat_rcv_txack_cnt      329                 , Hnat_gen_rxack_cnt       0                   
    [99013.968899] Hnat2mtl_ack_cnt        329                 , Mtl_2hnat_rdyn_cnt       329                 
    [99013.978454] Timeout_eof_cnt         0                   , Tx_enat_cnt              30                  
    [99013.988015] =============ERROR============
    [99013.992324] rx2tx_errpkt_cnt        0                   , rx_total_err_cnt         0                   
    [99014.001881] rx_gmii_err_cnt         0                   , rx_crc_err_cnt           0                   
    [99014.011501] rx_length_err_cnt       0                   , rx_iphdr_err_cnt         0                   
    [99014.020956] rx_payload_err_cnt      0
  ```

- dump实际设置到寄存器表项中的内容
  dump flowoffload entry实际设置到hant table寄存器中的值，主要用于检测debug entry设置是否正确．
  命令：```echo dump 1 > /sys/kernel/debug/hnat_debug```  
  结果展示如下:

  ```
    root@OpenWrt:/# echo dump 1 > /sys/kernel/debug/hnat_debug
	  [  925.355501] #current napt num 13 tcp 9 udp 4
	  [  925.359917] # hash full  0 dip hash full   0 add fail 0 update_flow 0  crc_clean 0
	  [  925.367512] nf dump total 13 tcp 9 udp 4 
	  [  925.371569] udp ageing  0  full ageing 0
	  [  925.375511] ####napt key, napt_index 1
	  [  925.379379] src:[192.168.6.100:55396] -> dest:[113.96.232.230:443]
	  [  925.385584] smac b0:83:fe:9a:2e:f4 vlanid 1 
	  [  925.385584]  router src  mac 00:9d:7d:86:20:08
	  [  925.394391] dmac a8:5a:f3:00:3a:98  vlanid 2 
	  [  925.394391]  router dest mac 00:9d:7d:86:20:09
	  [  925.403284] router:[192.168.5.203:55396]
	  [  925.407228] pppoe sid  0x0  proto 0 cur_ppoe_en 0
	  [  925.412026] valid = 1 src_vlan_index 0 dest_vlan_index	 1
	  [  925.417371] rt_pub_net_index = 0 src_dip_index 0 dest_dip_index 1
	  [  925.423570] ppphd_index = 0 dnat_to_host 0 is_dip_rt_ip_same_subnet 0
	  [  925.430137] lan_index = 0 wan_index 0
	  [  925.433832] sf_hnat_dump_napt_key===========end
	  [  925.433844] ####napt table
	  [  925.441189] >>>>>>>>read table no = 6:
	  [  925.444970] data0 = 0xe6 e8 60 71
	  [  925.448292] data1 = 0xbb 01 64 06
	  [  925.451705] data2 = 0xa8 c0 64 d8
	  [  925.455052] data3 = 0x40 86 0d 00
	  [  925.458375] data4 = 0x00 00 00 00
	  [  925.461788] data5 = 0x00 00 00 00
	  [  925.465127] data6 = 0x00 00 00 00
	  [  925.468440] >>>>>>>>read table no = 6 end
	  [  925.472495] napt crc table
	  [  925.475210] >>>>>>>>read table no = 0:
	  [  925.478967] data0 = 0x01 00 10 00
	  [  925.482381] data1 = 0x00 00 00 00
	  [  925.485712] data2 = 0x00 00 00 00
	  [  925.489026] data3 = 0x00 00 00 00
	  [  925.492443] data4 = 0x00 00 00 00
	  [  925.495777] data5 = 0x00 00 00 00
	  [  925.499093] data6 = 0x00 00 00 00
	  [  925.502525] >>>>>>>>read table no = 0 end
    ...
  ```

## 9 测试用例

HNAT测试用例包含多种测试方法，从前期芯片仿真验证, 到中期FPGA功能验证, 到后期系统级方案验证，以及常见吞吐性能测试；

### 9.1 仿真环境验证

仿真验证测试主要是初期针对hnat各模块基础功能进行功能验证，目前包含52个常规Case和18个Debug Case构成,每次芯片验证52个常规Case均PASS视为正常，详细Case列表见redmine#[9045](http://redmine.siflower.cn/redmine/issues/9045)．

### 9.2 FPGA的测试环境与方法

FPGA hnat的测试除了常规Case测试,还包含性能测试．其中性能测试主要有两种方法，一种使用iperf进行速率测试，一种使用gmac发包工具进行测试。使用iperf进行测试时，环境为：fpga的gmac接口外接交换机，lan/wan分别接pc，wan pc做为iperf server，lan pc做为iperf client，进行iperf测试。gmac发包工具测试时，环境为：fpga的gmac接口对接发包工具，工具发送不同类型的数据包，检查转换结果和速率。具体测试用例和结果可以参考redmine#[6767](http://redmine.siflower.cn/redmine/issues/6767)

### 9.3 HNAT系统级方案测试

HNAT系统级方案测试主要针对带有HNAT模块芯片的开发板上进行系统级别测试，包含多场景多功能测试，大体分类如下:
- SWITCH HNAT转换/吞吐；
- HOST HNAT转换/吞吐；
- GMAC-WIFI HNAT转换/吞吐；
- 多表项 + 多条流 + lan-wan双向HNAT；
- 分片报文HNAT转换；
- 性能验证完成单/多entry 10M/100M模式吞吐；
- FULL TABLE转换测试；
- 错误报文传输；
- Burst测试快速建链, 收发时增删表项；
- HNAT Jumbo报文；
- NAT模式验证；
- 性能测试中的PPS测试;
- HOST侧报文需要转换但是转换失败；
- router ip查表；
- TTL验证；
- HOST侧增/删 默认VLAN；
- LAN侧增/删 默认VLAN；
- 多条流测试；
- Burst测试中比例带宽测试；
- 性能测试中延时/抖动测试；  

详细测试结果见redmine#[9045](http://redmine.siflower.cn/redmine/issues/9045).

### 9.4 日常版本测试

日常发布软件版本时均会进行以太网性能测试，其中上行／下行吞吐测试均为HNAT硬件加速．详细测试结果见redmine#[9045](http://redmine.siflower.cn/redmine/issues/9045).

### 9.5 打流仪测试

使用打流测试仪进行常规2544测试,包含时延/抖动/吞吐/丢包/Burst等测试项，以及不同包长的测试，测试均PASS．详细测试结果见redmine#[9045](http://redmine.siflower.cn/redmine/issues/9045)．


## 10 HNAT已知问题

**问题一： Windows PC UDP跑流时hnat转换失败；**

**现象：** 
Linux PC进行UDP打流能达到1000M速率，Windows PC UDP打流只有200Mbps

**根因：**
Windows PC发送UDP报文时，其IPv4头flags中没有Don't Fragment标志位，导致hnat认为此报文为分片报文，从而在未开启分片功能情况下认为不需要进行hnat转换，而Linux PC有，所以fpga测试抓取报文时没有发现该问题；

**解决：**
1. hnat默认开启分片，并修改分片缓存老化定时寄存器0x10806064和0x1080606c，由0x4d00调整为0x2000，让非分片报文在分片功能开启情况下能快速发出；
2. tcp报文不受此问题影响；


**问题二： hant lan2wan打流压力测试偶现gmac dma卡住；**

**现象：**
lan2wan千兆打流压力测试偶现gmac dma卡住，系统无法继续工作；

**根因：**
通过读寄存器发现hnat tx_fifo中有数据，tx_info_fifo无数据，gmac tx fifo无数据，tx_info_fifo和tx_fifo的反压阈值设置的不匹配导致tx info fifo出现了丢报文情况，从而导致发送卡死反压dma无法继续发送；

**解决：**
将tx_fifo将满阈值由200改为100，这样可以保证在最小包的情况下不会出现tx_info_fifo已满依然有数据进来的情况，修改后拷机ok；

**问题三：WIFI和以太网同时进行hnat转换时gmac dma卡住**

**现象：**
gmac压力拷机测试，wlan看腾讯短视频，lan wan进行iperf 100M tcp打流测试，100s内，系统日志可以看到 tx transmit timeout日志，此时tx dma已经不再搬运数据。

**根因：**
1. 通过读寄存器发现查表模块的tx_out_buf出现overflow情况，这样查表结果丢失导致tx_fifo、tx_info_fifo和查表中tx_out_buf信息对应不起来；
2. 当tx_info_fifo有需要serch的信息传递给modify时，modify会取读取查表tx_out_buf信息，由于结果丢失，导致modify获取不到查询结果信息，从而使整个发送通路卡住。

**解决：**
1. tx_out_buf深度16，每3个地址存放一个报文的查询结果，这样最大可以缓存5条报文的查表结果；测试结果见redmine#[9045](http://redmine.siflower.cn/redmine/issues/9045)
2. 查表模块中的路径延时为32拍，最多可以缓存4条64B报文；
3. 通过分析调整tx_out_buf发压阈值，由6改为2，这样当tx_out_buf起反压时，在立刻反压入口buf的同时可以吸收查表路径上的4条报文，这样就不会出现丢查询结果的情况；
4. 修改后拷机测试通过。

**问题四：测试仪打流小包在70%负载以上时出现丢包**

**现象：**
测试仪压力测试，64B和88B小包在70%以上负载的情况下出现丢包的问题。

**根因：**
1. 该问题和问题一类似；
2. 测试仪发出的UDP非分片报文ip头中的flag为0，hnat把该报文当做了分片报文进入了分片报文处理流程，对于128B以下的小包，目前分片缓存老化定时器设置的0x2000偏大不合适；
解决：将分片缓存老化定时寄存器0x10806064和0x1080606c的值调整到0x1500，即43us后，不再出现丢包现象。


**问题五：硬件在进行NAT转化时，对UDP没有做特殊处理**

**现象：**  
用户在使用淘宝消消乐、淘金币等游戏时，出现了页面加载不出来，关闭硬加速后恢复，通过控制加速表项学习确认UDP表项导致。抓包分析游戏过程交互为QUIC协议报文，在UDP硬加速学习流程控制端口为443的流不进硬加速后问题不在出现。当只有udp数据包且port = 443 端口的时候流入硬加速的时候，通过打印log发现udp流在进入硬加速的时候显示checksum=0，禁用checksum = 0的流进入硬加速，发现游戏可以正常运行没有卡顿，由此得知在硬加速处理的过程中，当checksum=0的时候做了某些处理。

**根因：**  
当udp checksum=0的数据进入时是不需要校验会被协议栈bypass，然而硬件做的是增量计算，如果只改动port，会在原有正确的checksum进行差分值计算并且添加上去，而我们硬件没有对udp checksum=0的数据做特殊处理，导致了硬件没有考虑到udp checksum =0 的数据而进行了增量计算，使得checksum=0变成了checksum!=0，所以接收不到来自server发送过来的报文。

**解决：**  
通过软件进行过滤过掉 udp checksum=0的状态，只有udp checksum != 0的状态的时候，当收到了来自client和server发送的报文，才会进入硬件加速，如果只接受到了一端的报文则不进入硬加速，预留了一个可以进行UDP checksum==0进入的5001端口，用于进行iperf跑流。具体问题分析请参考：淘宝消消网络延迟问题

## 11 项目引用

- hnat驱动对接参考redmine#[6775](http://redmine.siflower.cn/redmine/issues/6775)
- hnat测试用例结果见redmine#[9045](http://redmine.siflower.cn/redmine/issues/9045)


## 12 FAQ
