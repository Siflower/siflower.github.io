---
layout: post
title: 以太网收发包接口说明手册
categories: DEVELOP
description: 以太网收发包
keywords:  eth trx
mermaid: true
---

# TS介绍

**目录**

* TOC
{:toc}


## 1 适用人员

本项目适用于基于siflower以太网源码，需要了解siflower 以太网方案收发包接口的人员。

## 2 开发与测试环境

siflower开发板及源码环境

## 3 功能概述

说明解释了网卡驱动如何接收数据包，以及如何在数据包从网络流向用户端程序时监视和调整网络堆栈的每个组件，如何调用具体的收发包函数。

## 4 开发详情

### 4.1 收包部分

#### 4.1.1 大致流程

1. 加载网卡驱动，做初始化（包括poll函数）
2. 网卡驱动需要在内存中申请一个缓冲区叫 sk_buffer
3. 当收到数据包，就把数据包的 sk_buffer分配好，描述符存进ring buffer
4. 网卡读取（通过DMA）ring buffer信息，获取sk_buffer地址和大小
5. 网卡（通过 DMA）将包 copy 到内核内存中的 ring buffer
6. 网卡会向CPU发起一个硬中断，告诉CPU有网络数据到达。
7. CPU调用网卡驱动注册的硬中断响应程序
8. 将网卡设备dev放入poll_list之中，然后调用软中断
9. 驱动调用napi，开始轮询（poll），从ring buffer收包
10. 这些数据包以skb的形式传入更上层的协议栈。
11. 协议栈处理包，将他们传入socket接收队列
    
![收包1.png](/assets/images/eth_trx_image/tx1.png)

![收包2.png](/assets/images/eth_trx_image/tx2.png)


#### 4.1.2 具体代码部分

以e1000网卡驱动为例子，使用napi机制。```/linux-4.14.90/drivers/net/ethernet/intel/e1000/e1000_main.c```

网卡驱动的初始化由e1000_probe开始。

硬中断中，e1000_probe→e1000_open，open函数对网络设备的数据结构做了初始化，特别是调用的setup_rx_resources为环形缓冲区做了初始化。

e1000_open→e1000_setup_rx_resources、e1000_request_irq。request_irq向系统申请中断号。

request_irq→e1000_intr。e1000_request_irq注册了硬中断处理函数为e1000_intr，其先添加网络设备dev到poll_list之后又调用了_napi_schedule。

来到```/linux-4.14.90-dev/linux-4.14.90/net/core/dev.c```

_napi_schedule代码中 __raise_softirq_irqoff(NET_RX_SOFTIRQ);触发了一个值为 NET_RX_SOFTIRQ 的软中断，在软中断向量表中找到对应的中断程序执行，
也就是网卡初始化函数net_dev_init（）函数中的代码：open_softirq(NET_RX_SOFTIRQ, net_rx_action);之中的net_rx_action（）

net_rx_action()中的napi_poll里面，又使用了网卡注册的poll方法来处理数据，这里的poll方法是e1000_clean，里面主要是缓冲队列中接收全部数据,将

sk_buffer中的网络数据包送到内核协议栈中注册的ip_rcv函数中

![收包3.png](/assets/images/eth_trx_image/tx3.png)

![收包4.png](/assets/images/eth_trx_image/tx4.png)

![收包5.png](/assets/images/eth_trx_image/tx5.png)


小结：

  ```
    __napi_schedule
         ->进入软中断
             ->net_rx_action
                ->napi_poll
                    ->驱动注册的poll
                        ->napi_gro_receive
                            -> napi_skb_finish
                                ->napi_receive_skb_internal
                                    ->napi_receive_skb_finish
                                        ->napi_receive_skb_core
                                            ->deliver_skb
```

最后到pt_prev->func这一句结束，后面就是协议栈的部分。

上述流程在/openwrt-18.06/package/kernel/sf_gmac/src/sf_gmac.c中原理也一样，使用platform总线的形式来体现。具体函数```sgmac_probe```:

```
        ->sgmac_netdev_ops 设备操作函数的初始化
        ->netif_napi_add将sgmac_poll设置为napi_poll函数
        ->sgmac_interrupt为中断处理函数
            ->napi_schedule
               ->net_rx_action
                 ->napi_poll
                    ->sgmac_rx
                        ->netif_receive_skb  or  napi_gro_receive
```

![收包6.png](/assets/images/eth_trx_image/tx6.png)

![收包7.png](/assets/images/eth_trx_image/tx7.png)

![收包8.png](/assets/images/eth_trx_image/tx8.png)

### 5.2 发包部分

#### 5.2.1 大致流程

1. 将用户要发的数据逐渐加上APP头部，TCP头部，IP头部，以太网头部，形成以太网帧。
2. 用户数据从应用层、网络层、到数据链路层，传入sk_buffer
3. 再如同rx步骤一般，将描述符传入一个tx ring buffer,又网卡读取，
4. DMA到网卡缓存区，发送
5. 发送完毕后给CPU发起硬中断
6. CPU再触发软中断进行skb的清理和tx ring buffer

![fa5.png](/assets/images/eth_trx_image/rx1.png)

#### 5.2.2 具体代码部分

dev_queue_xmit： netdevice子系统的入口函数，在该函数中，会先获取设备对应的qdisc，如果没有的话（如loopback或者IP tunnels），就直接调用dev_hard_start_xmit，否则数据包将经过Traffic Control模块进行处理，再进入dev_hard_start_xmit函数。


其中dev_queue_xmit已经完成了选择正确的发送队列，；保存当前的硬中断（IRQ）状态，并通过调用 local_irq_save 禁用 IRQ；获取当前 CPU 的 struct softnet_data 实例；将 qdisc 添加到 struct softnet_data 实例的 output 队列中；


>在/openwrt-18.06/package/kernel/sf_gmac/src/sf_gmac.c

可以得知设备操作函数结构体sgmac_netdev_ops的start_xmit为sgmac_xmit，它会启动数据包的发送。当系统调用驱动程序的xmit函数时，需要向其传入一个sk_buffer结构体指针，使得驱动程序能获取从上层传递下来的数据包。

sgmac_xmit（）:
```
    ->dev_kfree_skb_any（）       linux-4.14.90-dev/linux-4.14.90/net/core/dev.c
    函数 dev_kfree_skb_irq 可以将 skb 添加到队列中以便稍后释放。 设备驱动程序通常使用它来推迟释放消耗的 skb。
        ->__dev_kfree_skb_irq（）
            ->raise_softirq_irqoff（NET_TX_SOFTIRQ）调用了一个值为NET_TX_SOFTIRQ的一个中断函数
                ->net_tx_action
                    ->completion_queue  while 循环将遍历这个列表并__kfree_skb 释放每个 skb 占 用的内存
                    ->output queue 
                        ->tx_map
                            ->...netdev_tx_sent_queue  到此发送完成
```

## 6 FAQ

