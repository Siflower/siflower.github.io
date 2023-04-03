---
layout: post
title: HNAT介绍
categories: 32M_MEMORY
description: HNAT介绍
keywords: HNAT 硬加速
topmost: true
mermaid: true
---

* TOC
{:toc}


# 1 适用人员

本项目适用于对网络以及Linux内核有基本了解的人员。

# 2 开发与测试环境

需要带hnat模块的芯片的开发板环境或者FPGA环境，镜像需要选中hnat模块驱动。

# 3 相关背景

- nat是网络地址转换的缩写，是IP数据包在通过路由器或防火墙时重写来源IP地址或目的IP地址的技术。这种技术主要用于多台主机只通过一个公有IP地址访问互联网的情况。nat又分为SNAT和DNAT两种，其中SNAT是源地址转换，DNAT是目的地址转换。SNAT是当内部主机要访问公网上的服务时，内部主机会主动发起连接，并由路由器或防火墙将内部地址（源地址）转换成公有ip。DNAT是路由器或防火墙的网关接收到公网返回的数据，将目的地址从路由器或防火墙的网关转换为内部主机的ip，下图是一个nat转换的例子。

![nat.png](/assets/images/hnat_img/nat.png)

- hnat是硬件网络地址转换的缩写，顾名思义，就是使用硬件进行数据的nat转换。当一条tcp或udp连接建立之后，系统会通过网络驱动接口，将该连接所需的nat转换信息下发到hnat模块，从而由hnat模块完成后续数据包的nat转换，从而达到千兆级的转换速度。

# 4 功能概述

本文主要描述了HNAT硬件上的结构位置，实现的功能和性能，对外开放的接口，对接调试方法和测试补充等.

# 5 硬件结构

HNAT整个子系统位于GMAC模块中，在芯片中的位置如下:
![hnat_strcuture.png](/assets/images/hnat_img/hnat_strcuture.png)


# 6 实现功能

HNAT子系统主要针对GMAC外挂千兆交换芯片的应用场景，实现LAN和WAN通信报文的NAT转换，并对转换后的报文做转发处理，发往HOST或者GE_Switch，注意本子系统只对符合ETH II，802.3，VLAN，PPPOE格式的，并且协议为UDP、TCP的IPV4报文做NAT处理，其它报文上送HOST处理；

支持的数据流向如下:
![hnat_flow.png](/tfl/pictures/202112/tapd_49476398_1639648362_11.png)


- LAN（GE）<-->HOST ②<-->③<-->④
- LAN（GE）<-->WAN（GE） ②<-->③<-->①  需要做SNAT和DNAT
- LAN（GE）<-->WLAN ②<-->③<-->④<-->⑥
- WAN（GE）<-->HOST ①<-->③<-->④
- WAN（GE）<--> WLAN ①<-->③<-->④<-->⑥ 需要做SNAT和DNAT

由于NPU在最终的A28项目中删掉了，因为包含⑤的数据流将不在此列出．从上图的数据流向可以看出，HNAT完全支持以太网加速, WIFI加速场景．不仅如此其还支持众多Feature，如下：
- 支持以太网加速
	- hnat模块可以将转换后的数据包上送host或重新发出，通常情况下hnat在处理完来自switch的数据包之后（wan/lan），会转发回switch（lan/wan），这就是以太网部分的加速功能。
- 支持wifi加速功能
	- 根据配置，hnat模块同样可以对来自wifi侧的数据包进行snat加速，将wifi的数据包发送到wan。将wan->wifi的数据包dnat转换后从switch上送到host，再由host发送给wifi，这样可以hnat模块就可以实现对wifi的加速，提高wifi的性能。
- 支持不同HNAT模式；
	- hnat模块通过全局设置可以支持不同模式的nat，包括Basic NAT、Symmetric NAT、Restricted cone NAT、Port-Restricted cone NAT、Full cone NAT，根据不同的应用场景可以选择不同的nat模式，当前默认为Symmetric NAT。
		- Full Cone NAT（完全锥型NAT）
			所有从同一个私网IP地址和端口（IP1:Port1）发送过来的请求都会被映射成同一个公网IP地址和端口（IP:Port）。并且，任何外部主机通过向映射的公网IP地址和端口发送报文，都可以实现和内部主机进行通信。
			这是一种比较宽松的策略，只要建立了私网IP地址和端口与公网IP地址和端口的映射关系，所有的Internet上的主机都可以访问该NAT之后的主机。
	- Restricted Cone NAT（限制锥型NAT）
		所有从同一个私网IP地址和端口（IP1:Port1）发送过来的请求都会被映射成同一个公网IP和端口号（IP:Port）。与完全锥型NAT不同的是，当且仅当内部主机之前已经向公网主机发送过报文，此时公网主机才能向私网主机发送报文。
	- Port Restricted Cone NAT（端口限制锥型NAT）
		与限制锥型NAT很相似，只不过它包括端口号。也就是说，一台公网主机（IP2:Port2）想给私网主机发送报文，必须是这台私网主机先前已经给这个IP地址和端口发送过报文。
	- Symmetric NAT（对称NAT）
		所有从同一个私网IP地址和端口发送到一个特定的目的IP地址和端口的请求，都会被映射到同一个IP地址和端口。如果同一台主机使用相同的源地址和端口号发送报文，但是发往不同的目的地，NAT将会使用不同的映射。此外，只有收到数据的公网主机才可以反过来向私网主机发送报文。
		这和端口限制锥型NAT不同，端口限制锥型NAT是所有请求映射到相同的公网IP地址和端口，而对称NAT是不同的请求有不同的映射。
	- Basic NAT
		在技术上实现比较简单，只支持地址转换，不支持端口转换。
- 支持ip分片数据包；
	- hnat模块可以支持对ip分片的数据包进行nat处理，可以处理顺序/乱序情况下的多分片数据包（2分片及以上）。ip分片是为了处理ip数据包长度大于网卡mtu的情况，其中tcp协议具有分片功能，因此tcp数据包一般没有ip分片的情况，ip分片主要在udp数据包中出现。
- 支持最大1024条NAPT表和512条DIP表以及128条MAC表；
	- HNAT子系统最大支持1024条NAPT表项，512个不同的destination IP以及128条不同的MAC表项，由于NAPT和DIP数据采用hash压缩算法存储，因此由于hash冲突，存在较小的概率情况下，实际能支持的条目数小于1024条NAPT表项或512条destination IP表项。
- 根据配置信息进行nat转换
	- hnat模块可以识别ETH II、VLAN、PPPOE、VLAN+PPPOE四种类型的数据包，从而可以支持不同场景下的应用。根据驱动配置的信息，hnat可以根据原数据包中的ip/port等信息判断是否需要进行SNAT或DNAT。当需要进行SNAT时，hnat模块会修改原数据包中sip、sport、dmac、smac信息，当需要进行DNAT时，hnat模块会修改原数据包中dip、dport、dmac、smac信息，从而与协议栈处理结果相同，可以做到正常通信与加速功能。

# 7 性能测试

HNAT性能测试包含日常Release版本的IxChariot/Iperf性能测试以及打流仪2544性能标定的测试，摘取部分测试结果如下：

|测试项 |未加速 |hnat加速 |
|:-- |:-- |:-- |
|lan-wan吞吐 |275Mbps |932Mbps |
|wan-lan吞吐 |203Mbps |936Mbps |

# 8 对接接口

## 8.1 Openwrt系统对接接口

## 8.1.1 使能/关闭HNAT

在Openwrt系统中，TCP流以及UDP双向跑流HNAT硬件加速模式是默认打开的，可以通过修改proc配置文件来使能/关闭HNAT，其中enable置'0'表示关闭，默认为'1'表示开启。详细配置如下：

```
root@OpenWrt:/# echo enable 0 > /proc/hnat_procfs
[  119.430000] disable hnat
root@OpenWrt:/# echo enable 1 > /proc/hnat_procfs
[  125.500000] enable hnat
root@OpenWrt:/#
```

UDP单向跑流，默认走HNAT硬件加速使能，其中udp_check置'0'表示udp单向跑流经过硬件加速，默认为'1'表示udp单向跑流走软件加速．配置如下：

```
root@OpenWrt:/# echo udp_check 0 > /proc/hnat_procfs
[  194.890000] disable hnat udp bidirectional check
root@OpenWrt:/# echo udp_check 1 > /proc/hnat_procfs
[  198.430000] enable hnat udp bidirectional check
root@OpenWrt:/#
```

## 8.2 HNAT驱动对接接口

在sf_fast_path.c驱动接口函数中（sf_fast_path_init）初始化一个硬件加速队列（hnat_hw_work），这样当一条数据流建立起来之后，当数据包经过nf_hook_ops时，符合规则的数据包相关的nat信息就会被配置到驱动中。通过sf_flowoffload_hookfn钩子函数将数据流所需的信息下发到驱动的工作队列函数（sf_hnat_hw_work）中，从而完成hnat模块相关的配置。
以太网驱动中对应的接口如下:

```

int sf_fast_path_init(void)
{
	...
 	atomic_set(&curr_conn_cnt, 0);
	INIT_WORK(&hnat_hw_work, sf_hnat_hw_work);//初始化HNAT工作队列
    ...
	ret = nf_register_hooks(sf_nf_hook_ops, ARRAY_SIZE(sf_nf_hook_ops));//初始化钩子点
	if (ret < 0) {
		printk("err register nf hooks, ret:%d\n", ret);
		goto err_nf_hooks;
	}
	ret = nf_conntrack_register_notifier(&sf_conntrack_notifier);
	if (ret < 0) {
		printk("err register conntrack event notifier, ret:%d\n", ret);
		err_conn_ntf;
	}
	...
}
```

如果要将信息配置到hnat模块中，并调用hnat中的add接口将nat信息传递给hnat模块。基本原则为：添加时先加软件表项后加硬件表项，删除时先删硬件表现后删软件表项。
以太网驱动中调用工作队列接口实现如下:

```
static void sf_hnat_hw_work(struct work_struct *work)
{
	...
	if (entry->is_add) {
		ret = sf_hnat_entry_add(entry->ct, &entry->key);//添加nat硬件信息到HNAT驱动的entry数组中
		if (ret < 0)
			sync_sf_conn_list(entry->ct, ret);
		else
			sync_sf_conn_list(entry->ct, entry->key.napt_index);//添加软件信息
	}else {
		sf_hnat_entry_del(entry->key.napt_index);//删除硬件信息
		sync_sf_conn_list(entry->ct, -1);//删除软件信息
		}
	list_del(&entry->list);//删除nat链表信息
	kfree(entry);
 	}
}
```

## 8.3 加速表项下发/老化

nat硬件表项下发，通过sf_hnat_hw_work函数，当有nat信息下发下来，就将entry信息传入到队列中进行增删操作，该操作为异步操作，核心接口为：sf_hnat_add_entry（）

```
char sf_hnat_add_entry(struct sf_hnat_priv* phnat_priv, struct nf_conn *ct,
        struct sf_hashkey *sf_key, unsigned int* napt_index)
{
  	...
    spin_lock_bh(&hnat_napt_lock);
    ret = sf_hnat_search_napt(sf_key);//查找下发的nat信息是否已经存在数组当中
    if(ret >= 0){
        //if same napt with 2 ct ptr  set one free here
        if(sf_hnat_napt_key[ret].ct != sf_key->ct ){
            sf_hnat_napt_key[ret].ct = sf_key->ct;
            update_flow_count++;
        }
        *napt_index = ret;//信息已存在，更新信息
        spin_unlock_bh(&hnat_napt_lock);
        return 0;
    }

    ret = sf_hnat_add_napt(phnat_priv,  sf_key);//将nat信息sf_key写入数组中
    if( ret < 0){
        // printk("[hnat error] error add ptr add napt fail\n");
        goto err_add_napt_fail;
    }
    *napt_index = ret;
	...
    src_dip_index =  sf_hnat_add_dip(phnat_priv, &sf_hnat_napt_key[*napt_index] , 1);//将sip信息存到数组中
    dest_dip_index = sf_hnat_add_dip(phnat_priv, &sf_hnat_napt_key[*napt_index], 0);//将dip信息存到数组中
    ...

    phnat_priv->curr_napt_num++;

    if (phnat_priv->curr_napt_num >= FLOWOFFLOAD_HW_THRES) {
        if ((jiffies - phnat_priv->last_age_time) > (60*HZ)) {
            sf_hnat_random_aging();//表项的超时时间（60s）刷新硬加速表项，当连接数达到阈值800时，开始随机老化
            phnat_priv->last_age_time = jiffies;
        }
    }

```

当数据包经过nf_hook_ops点时，且表项超时时间300s，表项达到最大值1024，也会进行随机老化操作。

```

static unsigned int sf_flowoffload_hookfn(unsigned int hook,
        struct sk_buff *skb, const struct net_device *in,
        const struct net_device *out,
        int (*okfn)(struct sk_buff *))
{
	...

    if (atomic_read(&curr_conn_cnt) >= FLOWOFFLOAD_HW_MAX) {
        if ((jiffies - g_last_age_time) > (300*HZ)) {
            sf_hnat_random_aging();
            g_last_age_time = jiffies;
        }
        return NF_ACCEPT;
     }
     ...


    conn->ct = ct;
    conn->napt_index = -1;
    spin_lock_bh(&sf_conn_lock);
    hash_add(sf_conn_hlist, &conn->hnode, key);
    atomic_inc(&curr_conn_cnt);
    spin_unlock_bh(&sf_conn_lock);
    set_bit(IPS_HNAT_HOLD_BIT, &ct->status);//打上hold标记，表示该表项存在于hnat表里
    nf_conntrack_get(&ct->ct_general);

    entry->ct = ct;
    entry->is_add = 1;
    memcpy(&entry->key, &sf_key, sizeof(sf_key));
    spin_lock_irqsave(&hnat_hw_lock, flags);
    list_add_tail(&entry->list, &hnat_hw_list);
    spin_unlock_irqrestore(&hnat_hw_lock, flags);

    schedule_work(&hnat_hw_work);

    }

```

随机老化操作如下：

```
void sf_hnat_random_aging(void)
{
	...
    hash_for_each_safe(sf_conn_hlist, bkt, node, tmp, obj, hnode) {
        bool random_aging = false;//默认随即老化不使能
        tuple = &obj->ct->tuplehash[IP_CT_DIR_ORIGINAL].tuple;
        if (tuple->dst.protonum == IPPROTO_TCP) {
            tcp_index++;
            if (!(tcp_index % 3))
                random_aging = true;
        }else {
            udp_index++;
            if (udp_index % 2)
                random_aging = true;
        }

        if(obj->pending_flags)//当工作队列已经存在，则不进行老化操作
            continue;

        if ((obj->napt_index >= 0) && random_aging) {
            obj->pending_flags = true;//一个新的老化任务，会打上penging的flags
            entry = kzalloc(sizeof(*entry), GFP_ATOMIC);
            if (entry) {
                entry->ct = obj->ct;
                entry->key.napt_index = obj->napt_index;
                spin_lock_irqsave(&hnat_hw_lock, flags);
                list_add_tail(&entry->list, &hnat_hw_list);
                spin_unlock_irqrestore(&hnat_hw_lock, flags);
            }
        }
    }
    spin_unlock_bh(&sf_conn_lock);
    schedule_work(&hnat_hw_work);//将老化任务加入到工作对列中
}

```

## 8.4 HNAT删除机制

HNAT删除操作采用了消息通知机制来进行HNAT驱动与协议栈之间的信息互通，当收到event事件的时候，从内核下发下来的事件，做相应的操作。

- 注册通知链

```
int sf_fast_path_init(void)
{
	...
    ret = nf_register_hook(&sf_nf_hook_ops);//注册hook钩子点，用来完成hnat模块相关的配置。

    ret = nf_conntrack_register_notifier(&sf_conntrack_notifier);//注册链接跟踪事件通知链
	...
}

```

- 当event发生时，就使用notifier_call_chain向sf_conntrack_notifier通知链表发送消息。

```
struct nf_ct_event {
    struct nf_conn *ct;
    u32 pid;
    int report;
};

struct nf_ct_event_notifier {
    int (*fcn)(unsigned int events, struct nf_ct_event *item);
};
//接到通知后的回调处理函数，hnat ct状态不为hold且没有被破坏，正常进行增删操作，此操作为先添加软件ct表项，在添加nat硬件表项，先删除nat硬件表项，在删除nat软件表项。
static int sf_conntrack_event(unsigned int events, struct nf_ct_event *item)
{
	...

    if (unlikely(nf_ct_is_untracked(ct)))
        return NOTIFY_DONE;

    // only handle destory event
    if (unlikely(!(events & (1 << IPCT_DESTROY))))//IPCT_DESTROY事件，会在内核删除ct链表
        return NOTIFY_DONE;

    // only handle hnat hold ct
    if (!test_bit(IPS_HNAT_HOLD_BIT, &ct->status))//IPS_HNAT_HOLD_BIT事件，表明ct链表保持当前链接状态
        return NOTIFY_DONE;

    spin_lock_bh(&sf_conn_lock);
    conn = sf_conn_hash_lookup(ct, &key);
    spin_unlock_bh(&sf_conn_lock);
    if (!conn)
        return NOTIFY_DONE;

    // means a connection destoryed before hnat add, this will be delete in aging
    if (conn->napt_index < 0)
        return NOTIFY_DONE;

    entry = kzalloc(sizeof(*entry), GFP_ATOMIC);
    if (entry) {
        entry->ct = conn->ct;

       	...

    }
    schedule_work(&hnat_hw_work);//将entry添加到work队列中，进行增删操作
    return NOTIFY_DONE;
}
//注册回调函数
static struct nf_ct_event_notifier sf_conntrack_notifier = {
    .fcn  = sf_conntrack_event,
};
```

- hnat ct状态为hold或者被破坏，该流程在协议栈，可参考net/netfilter/nf_conntrack_core.c文件

```
第一种方式：
//正常情况打上IPS_DYING_BIT标记，走正常超时老化。event faile异常情况打上IPCT_DESTROY标记，另作判断。
static int kill_report(struct nf_conn *i, void *data)
{
    ...

    /* If we fail to deliver the event, death_by_timeout() will retry */
    if (nf_conntrack_event_report(IPCT_DESTROY, i,
                ¦   ¦ fr->pid, fr->report) < 0)
        return 1;

    /* Avoid the delivery of the destroy event in death_by_timeout(). */
    set_bit(IPS_DYING_BIT, &i->status);
    return 1;
}
void nf_conntrack_flush_report(struct net *net, u32 pid, int report)
{
	...
	//调用cleanup进行删除操作
    nf_ct_iterate_cleanup(net, kill_report, &fr);
}

第二种方式：
//直接删除所有的ct链表。
static int kill_all(struct nf_conn *i, void *data)
{
    return 1;
}
static void nf_conntrack_cleanup_net(struct net *net)
{
 i_see_dead_people:
    nf_ct_iterate_cleanup(net, kill_all, NULL);
    nf_ct_release_dying_list(net);
	...
}


void nf_ct_iterate_cleanup(struct net *net,
            ¦  int (*iter)(struct nf_conn *i, void *data),
            ¦  void *data)
{
    struct nf_conn *ct;
    unsigned int bucket = 0;

    while ((ct = get_next_corpse(net, iter, data, &bucket)) != NULL) {
        /* Time to push up daises... */
        if (del_timer(&ct->timeout)) {
            if (test_bit(IPS_HNAT_HOLD_BIT, &ct->status))
                ct->timeout.expires = jiffies + 60*HZ;
            death_by_timeout((unsigned long)ct);
        }
        /* ... else the timer will get him soon. */

        nf_ct_put(ct);
    }
}

static void death_by_timeout(unsigned long ul_conntrack)
{
	...
    //当hnat引用计数存在，并且没有超时的情况不允许协议栈老化删除，在hnat驱动删除
    if (test_bit(IPS_HNAT_HOLD_BIT, &ct->status)
            && (tuple->dst.protonum == IPPROTO_UDP)
            && (jiffies >= ct->timeout.expires)) {
        ct->timeout.expires = jiffies + 60*HZ;
        add_timer(&ct->timeout);
        return;
    }

	...
	//当IPCT_DESTROY标记存在时，内核删除ct list
    if (!test_bit(IPS_DYING_BIT, &ct->status) &&
    ¦   unlikely(nf_conntrack_event(IPCT_DESTROY, ct) < 0)) {
        /* destroy event was not delivered */
        nf_ct_delete_from_lists(ct);
        nf_ct_insert_dying_list(ct);
        return;
    }
    //正常流程超时老化
    set_bit(IPS_DYING_BIT, &ct->status);
    nf_ct_delete_from_lists(ct);
    nf_ct_put(ct);
}

```

# 9 HNAT对接调试方法

Siflower通过procfs节点暴露出去了一套私有Procfs接口，专门用于HNAT的Debug调试．详细使用方法见示例．
**示例：**
- 获取所有接口的帮助信息:
	命令:   `echo help >  /proc/hnat_procfs`
	结果展示如下:
	```
	echo help >  /proc/hnat_procfs
	root@OpenWrt:/# echo help > /proc/hnat_procfs
	[ 6452.580000]  echo natmode <mode> >  /proc/gmac_procfs
	[ 6452.590000]  intro: 0-BASIC_MODE, 1-SYMMETRIC_MODE, 2-FULL_CONE_MODE, 3-RESTRICT_CONE_MODE, 4-PORT_RESTRICT_CONE_MODE
	[ 6452.600000]  echo rd_speed >  /proc/hnat_procfs
	[ 6452.600000]  echo readl [addr] [number] >  /proc/hnat_procfs
	[ 6452.610000]  echo writel [addr] [data] >  /proc/hnat_procfs
	[ 6452.610000]  echo tabread [tab_no] [depth] >  /proc/hnat_procfs
	[ 6452.620000]  echo tabwrite [tab_no] [depth] [data(5)] >  /proc/hnat_procfs
	[ 6452.630000]  intro: dump all entry of table and crc table, 1 for napt, 0 for arp
	[ 6452.640000]  echo dump <napt/arp> >  /proc/hnat_procfs
	[ 6452.640000]  echo stat  >  /proc/hnat_procfs
	[ 6452.650000]  echo log mode  >  /proc/hnat_procfs
	[ 6452.650000]  intro: mode:0 for disbable hnat log, mode:1 for enable hnat log
	[ 6452.660000]  echo addifname [is_wan] [index] [ifname] >  /proc/hnat_procfs
	[ 6452.670000]  intro: add interface to monitor ip change. is_wan 0 for lan 1 for wan
	[ 6452.680000]  demo: echo addifname 0 1 br-lan2 > /proc/hnat_procfs
	[ 6452.680000]  echo addlan [index] [addr] [prefix_len] <ifname> >  /sys/kernel/debug/hnat_debug
	[ 6452.690000]  echo addwan [index] [addr] [prefix_len] <ifname> >  /sys/kernel/debug/hnat_debug
	[ 6452.700000]  echo dellan [index] <if_remove_name> >  /proc/hnat_procfs
	[ 6452.710000]  echo delwan [index] <if_remove_name> >  /proc/hnat_procfs
	[ 6452.720000]  echo getlan > /proc/hnat_procfs
	[ 6452.720000]  echo getwan > /proc/hnat_procfs
	[ 6452.730000]  echo getL2FastCount > /proc/hnat_procfs
	[ 6452.730000]  echo clearL2FastCount > /proc/hnat_procfs
	[ 6452.740000]  echo l2Fast 0/1 > /proc/hnat_procfs

	```

展示几个最主要的debugsf接口使用方法如下:
- 清空hnat表项
	 `echo vldclean > /proc/hnat_procfs`
- 开启hnat debug log
	hnat debug log主要打印hnat entry添加/删除时候的信息以及辅助判断的诊断信息.
	开启命令:   `echo log mode(mode == 1/2/3) > /proc/hnat_procfs`
	关闭命令:   `echo log 0 > /proc/hnat_procfs`
	结果展示如下:
	```
	root@OpenWrt:/# echo log 1 > /proc/hnat_procfs
	root@OpenWrt:/# [ 7870.644711] [hnat info] is_inat 1 add tbl 1 index 362 hash ptr 0
	[ 7870.650846] [hnat info] is_inat 0 add tbl 1 index 100 hash ptr 0
	[ 7870.656902] add napt 3 valid 1
	root@OpenWrt:/# echo log 0 > /proc/hnat_procfs

	root@OpenWrt:/# echo log 2 > /proc/hnat_procfs
	root@OpenWrt:/# [ 7910.646568] hnat offload ADD type=0
	[ 7910.650178] !!!!!!!!!!!!!!!!!!!!!flow_offload flags=0x49     timeout=0xba8a0 l4 proto= 0x6 l3proto=0x
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
	root@OpenWrt:/# echo log 0 > /proc/hnat_procfs

	root@OpenWrt:/# echo log 8 > /proc/hnat_procfs
	root@OpenWrt:/# [ 8049.053989] [hnat log] get lansubnet 0 prefix 24 ip 6406a8c0
	[ 8049.059797] [hnat log] get wansubnet 0 prefix 24 ipcb05a8c0
	root@OpenWrt:/# echo log 0 > /proc/hnat_procfs
	```

- 读取hnat转换计数
	读取hnat SNAT和DNAT报文计数以及dump当前hnat部分寄存器值．
	命令:   `echo stat > /proc/hnat_procfs`
	结果展示如下:

    ```
	  root@OpenWrt:/# echo stat > /proc/hnat_procfs
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
	  [99013.868866] rx_enter_sof_cnt        0                   , rx_enter_eof_cnt         0
	  [99013.878412] rx_2host_frame_sof_cnt  0                   , rx_2host_frame_eof_cnt   0
	  [99013.888005] rx_enter_drop_cnt       0                   , rx2tx_data_frame_cnt     0
	  [99013.897560] rx_enat_cnt             0                   , rx_inat_cnt              0
	  [99013.907173] rx2tx2host_cnt          0                   , rx2tx_drop_cnt           0
	  [99013.916745] ===============TX=============
	  [99013.920892] tx_sof_frame_cnt        0                   , tx_eof_frame_cnt         0
	  [99013.930508] tx_exit_sof_cnt         0                   , tx_exit_eof_cnt          0
	  [99013.940132] tx_receive_rx_frame_cnt 0                   , tx_nohits_frame_cnt      0
	  [99013.949713] Hnat_rcv_status_cnt     0                   , Hnat_tx_status_cnt       0
	  [99013.959303] Hnat_rcv_txack_cnt      0                   , Hnat_gen_rxack_cnt       0
	  [99013.968899] Hnat2mtl_ack_cnt        0                   , Mtl_2hnat_rdyn_cnt       0
	  [99013.978454] Timeout_eof_cnt         0                   , Tx_enat_cnt              0
	  [99013.988015] =============ERROR============
	  [99013.992324] rx2tx_errpkt_cnt        0                   , rx_total_err_cnt         0
	  [99014.001881] rx_gmii_err_cnt         0                   , rx_crc_err_cnt           0
	  [99014.011501] rx_length_err_cnt       0                   , rx_iphdr_err_cnt         0
	  [99014.020956] rx_payload_err_cnt		0
    ```

- dump实际设置到寄存器表项中的内容
  dump flowoffload entry实际设置到hant table寄存器中的值，主要用于检测debug entry设置是否正确．
  命令：`echo dump 1 > /proc/hnat_procfs`

  结果展示如下:

```
root@OpenWrt:/# echo dump 1 > /proc/hnat_procfs
[  925.355501] #current napt num 13 tcp 9 udp 4
[  925.359917] # hash full  0 dip hash full &nbsp; 0 add fail 0 update_flow 0  crc_clean 0
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
[  925.412026] valid = 1 src_vlan_index 0 dest_vlan_index    1
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
```