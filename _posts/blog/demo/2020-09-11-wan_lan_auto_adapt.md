---
layout: post
title: wan-lan自适应开发手册
categories: DEMO
description: 脚本开发，实现开机自动配置wan/lan
keywords: 自适应
mermaid: true
---

# wan-lan自适应开发手册

**目录**

* TOC
{:toc}

## 1 介绍

### 1.1 适用人员

适用于使用shell脚本进行openwrt功能开发的开发人员

### 1.2 开发环境

siflower SDK，siflower硬件平台

### 1.3 相关背景

VLAN（Virtual LAN），即“虚拟局域网”，可以使用户自由地根据实际需要分割广播域，在openwrt上可通过配置network文件实现vlan划分。但并不是每一个用户都了解vlan划分流程，因此路由器在面向普通
用户的时候，应当具备方便、简洁的特点：用户甚至不需要了解什么是vlan，怎么区分wan/lan口；随便插上网线就能实现上网功能。

### 1.4 功能概述

wan/lan 自适应开发脚本开机自启动，并在路由器的后台自动运行，自动根据当前的网线连接情况划分wan/lan口。如果不小心将外网网线插错到了lan口，脚本会自动将wan重新划分到该网口上，配置完成后，路由器依旧可以上网。

## 2 项目引用

- [config文件配置手册](/_posts/pw/configconfig文件配置手册.md)

- [openwrt uci官方文档](https://oldwiki.archive.openwrt.org/zh-cn/doc/techref/uci)

## 3 开发详情

### 3.1 基础指令

#### 3.1.1 uci指令

以太网相关配置是用network文件存储的，路径为/etc/config/network。我们可以直接手动修改该文件，也可以通过在串口下直接键入[uci指令](https://oldwiki.archive.openwrt.org/zh-cn/doc/techref/uci)修改network配置，还可以在shell中编写程序执行uci指令实现一定的功能。uci常用指令有：

|指令|作用|
|---|---|
|uci get network.lan.ipaddr|获取lan节点下的ip选项的值|
|uci set network.test=interface|network里加一个interface类型的节点|
|uci set network.test.a="abc"|向test节点下的a选项赋值（不存在a则创建此选项）|
|uci set network.@switch_vlan[0].vlan=1|将第一个switch_vlan节点下的vlan值改为1|
|uci delete network.test.a|删除a选项|
|uci delete network.test|删除test节点|
|uci commit|保存修改|

更多地了解如何使用uci指令配置config文件，请参考[config文件配置手册](/_posts/pw/config文件配置手册.md)或[openwrt uci官方文档](https://oldwiki.archive.openwrt.org/zh-cn/doc/techref/uci)

#### 3.1.2 vlan划分

VLAN（Virtual LAN），即“虚拟局域网”，可以使用户自由地根据实际需要分割广播域。AC22镜像network配置中vlan相关部分如下所示：

```
config interface 'lan'
        option ifname 'eth0.1'
        option force_link '1'
        option macaddr '10:16:88:3a:8d:f4'
        option type 'bridge'
        option proto 'static'
        option ipaddr '192.168.4.1'
        option netmask '255.255.255.0'
        option ip6assign '60'
        option group '0'
        option rps_cpus '2'
        option xps_cpus '2'

config interface 'wan'
        option ifname 'eth0.2'
        option force_link '1'
        option macaddr '10:16:88:3a:8d:f5'
        option rps_cpus '1'
        option xps_cpus '0'
        option proto 'dhcp'
...

config switch
        option name 'switch0'
        option reset '1'    
        option enable_vlan '1'
                                
config switch_vlan
        option device 'switch0'
        option vlan '1'
        option ports '0 1 2 3 16t'
                             
config switch_vlan
        option device 'switch0'
        option vlan '2'      
        option ports '4 16t'    

```

lan所在的eth0.1被划到了vlan 1，对应port 0 1 2 3，也就是第1,2,3,4号口。wan所在的eth0.2被划到了vlan 2,对应第5号口。如果想要将5号口划为lan，3号口划为wan，则只需将wan对应的ports改为'2 16t'，lan对应的ports改为'0 1 3 4 16t'。uci对应的指令为：

```
uci set network.@switch_vlan[0].ports='0 1 3 4 16t'
uci set network.@switch_vlan[1].ports='2 16t'
uci commit
```

更多vlan划分的内容，请参考[以太网wan/lan划分手册](toadd)

#### 3.1.3 udhcpc

udhcpc指令可以判断当前口是否可以获取到ip，从而判断该网口是否应该配置为wan口，可直接在串口键入。指令为"udhcpc -n -q -t 1 -i {port}" 。{port}为需要检测的端口，如br-lan，eth0.2。

- 获取到ip时会返回：

```
udhcpc (v1.23.2) started
Sending discover...
Sending select for 192.168.14.100...
Lease of 192.168.14.100 obtained, lease time 43200
```

- 无法获取ip时会返回：

```
udhcpc (v1.23.2) started
Sending discover...
No lease, failing
```

利用以上指令，可以在shell脚本里通过以下方法判断某个端口是否能够获取到ip。若获取到ip,则变量rv被赋值为1。

```
rv=0
udhcpc -n -q -t 1 -i eth0.x>&- && rv=1
```

### 3.2 功能设计流程

#### 3.2.1 自启动实现

openwrt在开机启动时，会自动执行路径为/etc/rc.local的文件。因此在rc.local的末尾部分加上运行某脚本的指令，那么这个脚本就会实现开机自启动。例如：现有一个可执行脚本test.sh放在/bin目录下。如果在rc.local最后（exit 0之前）添加/bin/test.sh，这个脚本就会开机自启动。

#### 3.2.2 wan/lan自适应流程

- 1 开机时将network进行初次配置（新划分5个vlan均为dhcp）

- 2 将检测这5个vlan口是否拿到ip，若能则划分这个口为wan口，开机启动划分完成。

- 2 将lan口proto设置为dhcp，利用udhcpc指令循环检测br-lan是否获取到ip。若能获取到，说明使用过程中网线插错，则进行重新配置。

- 3 配置新的wan/lan并重启网络。待配置完成，继续循环检测直到下一次配置。

流程图如下所示：

```flow
st=>start: 路由器启动
config=>operation: 初次配置
wait=>operation: 循环等待
ifchange=>condition:  lan口是否拿到ip
change=>operation: 配置wan口
sub1=>subroutine: sleep等待配置完成
st->config->wait->ifchange
ifchange(yes)->change->wait
ifchange(no)->wait
```

### 3.3 代码实现（以AC22为例）

- 1 将示例自启动脚本test.sh放置在需要的位置，如/etc/config
  
```
#!/bin/sh

set_wan() {
    case $1 in
        1)
            ports_wan="0 16t"
            ports_lan="1 2 3 4 16t"
            ;;
        2)
            ports_wan="1 16t"
            ports_lan="0 2 3 4 16t"
            ;;
        3)
            ports_wan="2 16t"
            ports_lan="0 1 3 4 16t"
            ;;
        4)
            ports_wan="3 16t"
            ports_lan="0 1 2 4 16t"
            ;;
        *)
            ports_wan="4 16t"
            ports_lan="0 1 2 3 16t"
            ;;
    esac
    uci set network.@switch_vlan[0].ports="$ports_lan"
    uci set network.@switch_vlan[1].ports="$ports_wan"
    uci commit
}

checkport() {
    a=0
    rv=0
    for i in `seq 1 5`
    do
        {
            udhcpc -n -q -t 1 -i eth0."$i">&- && rv=$i
            [ $rv -ne 0 ] && echo $rv > /tmp/wanlan_set
        }&
    done
    wait
    if [ -f "/tmp/wanlan_set" ]; then
        a=`cat /tmp/wanlan_set`
        rm /tmp/wanlan_set
    fi
    echo $a
}

init() {
	uci set network.lan.proto='dhcp'
	uci set network.@switch_vlan[0].ports='0 16t'
	uci set network.@switch_vlan[1].ports='1 16t'
	for i in `seq 3 5`
	do
		uci set network.wan"$i"='interface'
		uci set network.wan"$i".ifname=eth0."$i"
		uci set network.wan"$i".proto=dhcp
	done
	for i in `seq 3 5`
	do
		a=`uci add network switch_vlan`          
		uci set network.$a.vlan="$i"             
		uci set network.$a.ports="`expr $i - 1` 16t"
	done
	uci commit
	/etc/init.d/network restart
}

dinit() {
	for i in `seq 3 5`
	do
		uci delete network.wan"$i"
		uci delete network.@switch_vlan[2]
	done
	uci set network.lan.proto='static'
	uci commit
}

start() {
    init
    port=0
    for i in `seq 1 5`
    do
        port=$(checkport)
        if [ $port -ne 0 ]; then
            set_wan $port
            break
        fi
        sleep 2
    done
    dinit
    /etc/init.d/network restart
}

waiting() {
    rv=0
    while true
    do
        udhcpc -n -q -t 1 -i br-lan>&- && rv=1
        if [ $rv -eq 1 ]; then
            start
            rv=0
        fi
        sleep 3
    done
}
start
#循环检测会耗费cpu资源，因此若不添加waiting函数则只在开机时配置一次。
waiting

```

- 2 修改/etc/rc.local，使其开机运行/etc/config/test.sh

```
# Put your custom commands here that should be executed once
# the system init finished. By default this file does nothing.

/etc/config/test.sh
exit 0
```

### 3.4 TODO 

以上代码实现的实例为仅适用于p10的初级版本，此功能需要根据不同版型做开发，同时可以考虑更换不同的逻辑和方法实现代码优化。目前版本的代码自适应需要花费15-20s左右的时间，时间略长。后续开发将围绕以下目标进行：

- 优化算法和流程，减少自适应的配置时间

- 开发一套模板，能试用于所有版型

## 4 测试用例

### 4.1 测试环境配置

一台待已放置自启动脚本的待测路由、串口、能获取到ip并上网的网线。

### 4.2 测试流程

- 1、将网线插入路由器任意网口，重启路由，同时在串口观察是否打印log。等待重启完成后，ping www.baidu.com检测是否能上网。

- 2、将网线拔出，插入另一个口，同时在串口观察是否打印log。然后等待约30s，再次检测是否能上网
  
### 4.3 测试结果

重启后或者插拔网线更换网口后，会有自适应脚本的相应log打印；等待一段时间后，路由器能正常上网，说明功能正常。

## 5 FAQ

- **Q: 脚本在开机后没有自启动，如何检查？**

  A：在/etc/rc.local中，运行脚本指令之前打log，确认自启动脚本是否被调用。若此log未出现，说明rc.local有问题而非自启动脚本本身的问题。之后在脚本第一行打log,并用ps指令查看脚本是否已在后台运行。若log已打但脚本未运行，则可判断是脚本逻辑问题导致运行结束。

- **Q:配置后无法上网，如何检查？**

  A：首先查看/etc/config/network文件是否按照规范配置，wan口有没有划分正确；/etc/init.d/network restart重启网络，差看是否是因为配置未生效；网线连电脑查看是否是外部网络的问题。
