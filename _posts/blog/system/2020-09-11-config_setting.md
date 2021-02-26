---
layout: post
title: config文件配置手册
categories: SYSTEM
description: 介绍config文件及配置过程
keywords: 配置文件
mermaid: true
---
# config文件配置手册

**目录**

* TOC
{:toc}

## 1 介绍

### 1.1 适用人员

适用于需要使用config配置文件的开发人员，具备串口访问路由器的基础操作即可。

### 1.2 开发环境

已烧录正常镜像的siflower硬件平台、可访问路由的串口工具或ssh

### 1.3 功能概述

config文件为openwrt用于存储配置的文件，方便用户进行配置。配置完成后会有相应的程序进行解析实现其功能。本文档详细介绍了config文件的路径及结构，以及如何配置config文件，并对当前siflower镜像中已有的config文件实例进行了说明。

## 2 项目引用

- [openwrt uci官方文档](https://oldwiki.archive.openwrt.org/zh-cn/doc/techref/uci)

- [openwrt 网络设置官方文档](https://oldwiki.archive.openwrt.org/zh-cn/doc/uci/network)

- [wifi架构和配置手册](https://siflower.github.io/2020/08/12/wifi_architecture_and_configuration_manual/)

## 3 config配置文件详情

### 3.1 文件位置

config为openwrt的配置文件，可以提前写好直接编译进镜像（ddns、sicloud等），也可以通过脚本在开机启动时自动生成（network、wireless）。配置文件存在于代码库的如下四个位置：

- 1 feeds/luci/modules/luci-base/root/etc/config

- 2 package/base-files/files/etc/config

- 3 target/linux/siflower/base-files/etc/config

- 4 target/linux/siflower/sf19a28-fullmask(mpw1)/base-files-SF19Ax8-xxxx/etc/config

其中１，２，3为公共部分，4为私有部分。4中除了fullmask还可以选择mpw1，xxxx为对应的版型（AC22，AC28，EVB）。编译镜像时，会将公共部分和版型对应的私有部分的所有config文件编译进镜像中，烧录后可在/etc/config/路径下查看。当１，2，3，4存在同名文件时，优先顺序由高到低依次为：1 > 4 > 3 > 2。编译镜像时，优先级高的文件会将优先级低的覆盖

### 3.2 文件结构

config文件大多由具名节点和匿名节点两种节点组成，节点下有两种元素：option选项和list列表，结构如下所示：
- 具名节点：

```
config <section-type> '<section-name>'
        option <name1> '<value1>'
        list <name2> '<value2>'
        list <name2> '<value3>'
...
```

- 匿名节点：

```
config <settion-type>
        option <name1> '<value1>'
        option <name2> '<value2>'
...

config <settion-type>
        option <name1> '<value1>'
        option <name2> '<value2>'
...
```

注意项：

- 1、不存在同名的节点（例如config interface 'lan'），但可以存在多个同类型的匿名节点（例如config switch_vlan）。匿名节点并非没有名字，而是“隐藏”起来，可以将@<settion-type>[n]作为它的“名字”来调用，n表示第n+1个。例如@switch_vlan[1]就是第2个switch_vlan的“名字”。所有节点下选项的值均可以直接更改，或者通过uci指令进行配置。

- 2、UCI 允许只有节点类型的匿名节点存在，节点类型和名字建议使用单引号包含以免引起歧义。

- 3、节点中可以包含多个option选项或list列表，但需要避免相同的选项名存在于同一个节点,否则只有一个生效

- 4、列表的名字如果相同,则相同名字的值将会被当作数组传递给相应程序。

### 3.3 uci配置指令

UCI是集中式配置信息管理接口(Unified Configuration Interface)的缩写，他是OpenWrt引进的一套配置参数管理系统。UCI管理了OpenWrt下最主要的系统配置参数并且提供了简单、容易、标准化的人机交互接口。UCI中已经包含了网络配置、无线配置、系统信息配置等作为基本路由器所需的主要配置参数。同时UCI也可以帮助开发人员快速的建立一套基于OpenWrt的智能路由产品控制界面。更多信息可参考[openwrt uci官方文档](https://oldwiki.archive.openwrt.org/zh-cn/doc/techref/uci)

- uci包含如下指令：

```
export     [<config>]
import     [<config>]
changes    [<config>]
commit     [<config>]
add        <config> <section-type>
add_list   <config>.<section>.<option>=<string>
show       [<config>[.<section>[.<option>]]]
get        <config>.<section>[.<option>]
set        <config>.<section>[.<option>]=<value>
delete     <config>[.<section[.<option>]]
rename     <config>.<section>[.<option>]=<name>
revert     <config>[.<section>[.<option>]]
```

- 可使用的参数有：

```
-c <path>  set the search path for config files (default: /etc/config)
-d <str>   set the delimiter for list values in uci show
-f <file>  use <file> as input instead of stdin
-m         when importing, merge data into an existing package
-n         name unnamed sections on export (default)
-N         don't name unnamed sections
-p <path>  add a search path for config change files
-P <path>  add a search path for config change files and use as default
-q         quiet mode (don't print error messages)
-s         force strict mode (stop on parser errors, default)
-S         disable strict mode
-X         do not use extended syntax on 'show'
```

- 常用指令说明：

|指令|说明|
|---|---|
|uci get \<config>.\<section>|取得节点类型|
|uci get \<config>.\<section>.\<option>|取得一个值|
|uci show|显示全部 UCI 配置|
|uci show \<config>|显示指定文件配置|
|uci show \<config>.\<section>|显示指定节点名字配置|
|uci show \<config>.\<section>.\<option>|显示指定选项配置|
|uci changes \<config>|显示尚未生效的修改记录|
|uci show -X \<config>.\<section>.\<option>|匿名节点显示(-X 参数可以显示出匿名节点的 ID)|
|uci add \<config> \<section-type>|增加一个匿名节点到文件|
|uci set \<config>.\<section>=\<section-type>|增加一个节点到文件中|
|uci set \<config>.\<section>.\<option>=\<value>|增加一个选项和值到节点中|
|uci add_list \<config>.\<section>.\<option>=\<value>|增加一个值到列表中|
|uci set \<config>.\<section>=\<section-type>|修改一个节点的类型|
|uci set \<config>.\<section>.\<option>=\<value>|修改一个选项的值|
|uci delete \<config>.\<section>|删除指定名字的节点|
|uci delete \<config>.\<section>.\<option>|删除指定选项|
|uci delete \<config>.\<section>.\<list>|删除列表|
|uci del_list \<config>.\<section>.\<option>=\<string>|删除列表中一个值|
|uci commit \<config>|生效修改|

### 3.4 uci指令实例

以下以AC28的network配置文件为例，列出了常用uci指令的具体使用方法。

- AC28 network：

```
config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config interface 'lan'
        option type 'bridge'
        option ifname 'eth0.1'
        option proto 'static'
        option ipaddr '192.168.4.1'
        option netmask '255.255.255.0'
        option ip6assign '60'

config device 'lan_dev'
        option name 'eth0.1'
        option macaddr '10:16:88:c6:44:40'

config interface 'wan'
        option ifname 'eth0.2'
        option proto 'dhcp'

config interface 'wan6'
        option ifname 'eth0.2'
        option proto 'dhcpv6'

config device 'wan_dev'
        option name 'eth0.2'
        option macaddr '10:16:88:c6:44:41'
        
config switch
        option name 'switch0'
        option reset '1'
        option enable_vlan '1'

config switch_vlan
        option device 'switch0'
        option vlan '1'
        option ports '1 2 0 5t'

config switch_vlan
        option device 'switch0'
        option vlan '2'
        option ports '3 5t'
```

|指令|作用|
|---|---|
|uci show network.wan|列出network中wan节点所有选项的值|
|uci show network.@switch_vlan[0]|列出类型为switch_vlan的第一个节点的所有选项值|
|uci get network.lan.proto|获取lan节点下的proto选项的值，此处返回"static"|
|uci set network.wan.proto="pppoe"|将wan节点下的proto选项的值设置为“pppoe”|
|uci set network.wan2=interface|network里添加一个interface类型的节点，名为wan2|
|uci add network switch_vlan|network里添加一个switch_vlan类型的匿名节点|
|uci set network.@switch_vlan[0].vlan=1|将第一个switch_vlan节点下的vlan值改为1|
|uci delete network.lan.ipaddr|删除lan节点下的ipaddr选项|
|uci delete network.@switch_vlan[2]|删除第三个switch_vlan节点|
|uci commit network|保存对network文件的修改（可省略network，保存所有修改）|

## 4 测试用例

### 4.1 测试环境配置

具有任意config文件的路由(例如network)、能访问路由的串口工具或ssh

### 4.2 测试流程

在串口依次键入以下指令：

```
uci set network.new_section="interface"
uci set network.new_section.new_option="option_value"
uci add network another_section
uci set network.@another_section[0].new_option2="option_value2"
uci commit
uci show network
```

### 4.3 测试结果

若配置成功，则在串口执行uci show指令后可见如下输出：

```
...
network.new_setcion=interface
network.new_setcion.new_option='option_value'
network.@another_section[0]=another_section
network.@another_section[0].new_option2='option_value2'
```

查看/etc/config/network文件，末尾添加了新的配置：

```
config interface 'new_setcion'
        option new_option 'option_value'
                            
config another_section
        option new_option2 'option_value2'
```

## 5 config文件实例

详细介绍了下列config文件中每个配置选项的功能。

### 5.1 /etc/config/wireless

1、wifi-device配置选项：

  | 选项 |值类型|默认值|描述 | 
  | :---: |:---:|:---:| :---: |
  |wifi-device|string|radio0|驱动设备名称|
  |type|string|mac80211|驱动类型，目前固定为"mac80211"。|
  |country|string|CN|国家码，2个大写字母，默认为CN，表示中国（China），国家码会影响信道和发射功率。|
  |channel|string/int|-|信道，默认2.4G为auto（自动选取最优信道），可以按需修改为固定信道，不同国家信道限制不同，如中国地区2.4G信道为1~13 ，5G信道为36~64 、149~165。|
  |txpower_lvl|int|2|发射功率，可设值为0、1、2。该值越大，表示功率越大。|
  |max_all_num_sta|int|40|驱动所能连接设备个数的最大值|
  |netisolate|boolean|0|设备隔离，如果设置为1，则从该device下的设备无法访问同一网桥（bridge）中其它bssid的设备。|
  |noscan|boolean|0|值为1时，表示不扫描周围信道。|
  |path|string|-|对应驱动在/sys/devices/下的节点，一般不作修改。默认2.4G为"platform/11000000.wifi-lb"，5G为"platform/17800000.wifi-hb"。|
  |htmode|string|20MHz|带宽模式，2.4G支持20MHz/40MHz，5G支持20MHz/40MHz/80MHz。|
  |hwmode|string|-|wifi工作模式，2.4G支持11b/11g/11n，5G支持11n/11a/11ac，最终的模式是由htmode和hwmode共同决定的|
  |disabled|boolean|0|0表示启用该驱动设备，1表示关闭该驱动设备。|
  |ht_coex|boolean|/|值为1时，表示带宽20MHZ/40MHZ共存，与htmode有一定联系。|

2、wifi-iface配置选项

  | 选项 |值类型|默认值|描述 |
  | :---: |:---:|:---:| :---: |
  |wifi-iface|string|default_radio0|wifi-iface节点名称|
  |device|string|-|对应wifi-device驱动名称，默认2.4G为radio0，5G为radio1。|
  ifname|string|wlan0|网卡(iface)的名称，使用ifconfig时会显示对应名称。|
  |network|string|lan|对应的网桥（bridge）名称，如果需要把wifi加入到lan口则配置该值为lan。|
  |mode|string|ap(sta、minotor)|ap对应热点，sta对应站点（station），monitor对应监听模式。默认为ap模式。|
  |ssid|string|SiWIFi-****|wifi的名称，最大不超过32位。支持中文，但在串口会显示为"..."。默认名称中的数字来源于mac地址。|
  |encryption|string|none|加密方式，"none"表示不加密，如果想加密，建议改成"psk2+ccmp"。|
  |key|string|12345678|wifi密码，psk2需设置8位以上。当加密方式为不加密（none）时此选项不生效，而其他加密方式必须配置密码。|
  |hidden|boolean|0|是否隐藏热点，1表示隐藏，0表示不隐藏。隐藏后设备只能通过手动添加SSID才能连接wifi。|
  |wpa_group_rekey|int|3600|刷新GTK（广播/多播加密密钥）的时间间隔（以秒为单位）。若不设置此项，则使用CCMP / GCMP作为组密码时默认为86400秒（每天一次），使用TKIP作为组密码时默认为600秒（每10分钟一次）。|
  |isolate|boolean|0|连接此wifi的各设备之间是否隔离，1表示隔离，0表示不隔离。|
  |group|int|-|bridge中的分组，各个不同的group之间在bridge中是不能互相访问的。默认2.4G为0，5G为1。|
  |netisolate|boolean|0|如果配置为1，则从该bssid下的设备无法访问同一bridge中其它bssid的设备。|
  
  更多详细信息可参考[wifi架构和配置手册](https://siflower.github.io/2020/08/12/wifi_architecture_and_configuration_manual/)。

### 5.2 /etc/config/network

一、Interfaces配置选项，如wan/lan等。interface类型的节声明了逻辑网络接口，可以为这些接口指定IP地址、别名、物理网络接口名称、路由规则及防火墙规则。

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|ifname|string|eth0.1|相关联的物理网络接口名称(用ifconfig可看到)|
|proto|string|dhcp|指定接口协议,可选dhcp,static,pppoe|
|type|string|-|如果设置为"bridge"，将建立一个包含ifname所述接口的网桥|
|macaddr|mac address|-|指定接口MAC地址|

针对指定协议的类型，还可能存在额外的选项：

1、static协议

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|ipaddr|ip address|-|IP地址|
|netmask|netmask|-|子网掩码|
|gateway|ip address|-|默认路由|
|bcast|ip address|-|广播地址(不设置会自动生成)|
|ip6addr|ipv6 address|-|为接口指派给定的IPv6 地址 (CIDR notation)|
|ip6gw|ipv6 address|-|为接口指派给定的IPv6默认网关|
|dns|list of ip addresses|-|DNS服务器（1个或多个）|

2、pppoe协议

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|username|string|-|宽带账号|
|password|string|-|宽带密码|
|pppd_options|ip address|-|运营商指定的IP地址|
|dns|list of ip addresses||手动设置DNS服务器|
|mtu|number|1480|数据包MTU字节|
|peerdns|number|-|运营商指定的IP地址|
|connectmode|number|1|连接模式，可选0按需连接，1自动连接，2手动连接|
|demand|number|-|自动断线等待时间|

3、dhcp协议

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|gateway|string|-|如果设置为0.0.0.0，将设置DHCP协议获取的默认网关|
|dns|list of ip addresses|-|指定DNS服务器（1个或多个）|

二、switch配置选项。switch节点负责交换芯片VLAN的划分。在OpenWrt系统内部，每个VLAN都会有一个独立的interface与它对应，即便它们实际上属于同一个硬件。

1、switch配置选项

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|switch|string|switch0||
|reset|boolean|1||
|enable_vlan|boolean|1|启用vlan|

2、switch_vlan配置选项

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|device|string|switch0|指定device属性|
|vlan|munber|-|vlan id|
|ports|string|-|指定对应的port口|

### 5.3 /etc/config/basic_setting

|节点|选项|值类型|默认值|描述 |
|---| :---: |:---:|:---:| :---: |
|onlinewarn|enable|boolean|0|是否启用wifi设备响铃功能|
|guide|enable|boolean|0|暂无接口使用此参数|
|lease_wifi|speed|number|0|租赁网络限速|
|country_code|enable|boolean|0|网页“主人网络”中是否开启“国家及地区”设置|
|mode|mode_code|number|0|syslog模式，可选0，1，2，对应syslog越详细|
|speed|enable|boolean|0|暂无接口使用此参数|
|kerHealth|enable|boolean|0|暂无接口使用此参数|
|auto_ota|enable|boolean|1|启用ota自动升级|
|ota|romtype|number|-|romtype值|
|ota|chip|string|-|芯片类型，fullmask等|
|ota|imagetype|number|-|镜像类型|
|dev_mode|mode|string|ap|暂无接口使用此参数|
|updateKeyMode|enable|boolean|0|暂无接口使用此参数|

### 5.4 /etc/config/ddns

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|service_name|string|oray.com|服务提供者|
|domain|string|-|域名信息|
|lookup_host|string|ddns.oray.com||
|username|string|-|用户名|
|password|string|-|密码|
|interface|string|wan||
|ip_source|string|network||
|ip_network|string|wan||
|dns_server|list of ip addresses|114.114.114.114|dns服务器|

### 5.5 /etc/config/dhcp

1、dnsmasq

| 选项 |值类型|默认值|描述 |
|:---:|:---:|:---:|:---:|
|domainneeded|boolean|1|告诉dnsmasq禁止将"不带dots或domain parts的名称的查询"转发到上游名称服务器|
|boguspriv|boolean|1|拒绝对/ etc / hosts中不存在相应条目的私有IP范围的反向查找|
|filterwin2k|boolean|0|不转发公共名称服务器无法回答的请求|
|localise_queries|boolean|1|如果在/etc/hosts中为主机名分配了多个地址，请选择IP地址以匹配传入接口|
|rebind_protection|boolean|1|通过丢弃上游RFC1918响应来启用DNS重新绑定攻击保护|
|rebind_localhost|boolean|1|允许上游127.0.0.0/8响应|
|local|string|/lan/|从/etc/hosts查找该域的DNS条目|
|domain|domain name|lan|DNS域分发给DHCP客户端|
|expandhosts|boolean|1|若在/etc/hosts中找得到本地域，则将本地域部分添加到其中|
|nonegcache|boolean|0|若缓存响应为否定的“没有这样的域”，则将其禁用|
|authoritative|boolean|1|强制dnsmasq进入权威模式|
|readethers|boolean|1|从/etc/ethers读取静态租约条目（在SIGHUP上重新读取）|
|leasefile|string|/tmp/dhcp.leases|将DHCP租约存储在此文件中|
|resolvfile|string|/tmp/resolv.conf.auto|指定备用解析文件|
|nonwildcard|boolean|1|仅绑定配置的接口地址，而不绑定通配符地址|
|localservice|boolean|1|仅从地址位于本地子网（即服务器上存在接口的子网）的主机上接受DNS查询|
|dhcpscript|string|/lib/netifd/dhcplease|dhcp脚本路径|

2、dhcp

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|interface|string|-|指定与此DHCP地址池关联的接口；必须是/etc/config/network中定义的接口之一|
|start|number|100|指定与基础接口的网络地址的偏移量，以计算可以租借给客户端的最小地址。跨子网可能大于255|
|limit|number|150|指定地址池的大小（例如，start=100，limit=150，最大地址为249）|
|leasetime|string|12h|指定分发给客户端的地址的租用时间|
|dhcpv6|string|-|指定应启用（服务器），中继（中继）还是禁用（禁用）DHCPv6服务器|
|ra|string|-|指定应启用（服务器），中继（中继）还是禁用（禁用）路由器广告|
|ra_default|boolean|-|ipv6相关，暂无接口使用此参数|
|ignore|boolean|-|dnsmasq是否应忽略此池,若指定为1则忽略|
|maindhcp|boolean|-|暂无接口使用此参数|
|leasefile|string|-|leasefile是否为/var/dhcp.lease|
|leasetrigger|string|-|暂无接口使用此参数|
|loglevel|number|-|暂无接口使用此参数|

### 5.6 /etc/config/sicloud

1、addr

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|ip|string|cloud.siflower.cn|服务器IP地址|
|port|number|443|端口号|
|version|string|v4|服务器版本|
|cloudtype|number|0||

2、leaseserver

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|ip|ip address|139.196.176.186|服务器地址|
|port|number|8051|端口号|
|httpsport|number|8052|http端口号|

### 5.7 /etc/config/siwifi

1、hardware

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|sn|string|-|设备sn号|
|romtime|string|-|设备romtype值|

### 5.8 /etc/config/system

1、system

| 选项 |值类型|默认值|描述 | 
| :---: |:---:|:---:| :---: |
|zonename|string|Asia/Shanghai|地区|
|timezone|string|CST-8|时区|
|hostname|string|SiWiFi8df5|主机名|
|hostnameset|boolean|1||

2、ntp

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|server|list|-|ntp服务器地址|
|enabled|boolean|1|是否使用ntp服务器|
|enable_server|boolean|0|是否启用ntp服务器|

### 5.9 /etc/config/firewall

1、defaults。defaults节定义了不依赖于特定区域的防火墙全局设置

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|syn_flood|boolean|1|允许SYN flood保护|
|drop_invalid|boolean|-|丢弃任何没有匹配到已有连接的包|
|disable_ipv6|boolean|-|禁用IPv6防火墙设置 1 (Firewall v2 and later)|
|input|string|ACCETP|INPUT链缺省策略(ACCEPT, REJECT, DROP)|
|forward|string|ACCEPT|FORWARD链缺省策略(ACCEPT, REJECT, DROP)|
|output|string|REJECT|OUTPUT缺省策略(ACCEPT, REJECT, DROP)|

2、zones

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|name|string|-|唯一的zone名称|
|network|list|-|与此zone绑定的interface|
|masq|boolean||-指定是否伪装从该zone传出的流量|
|mtu_fix|boolean|-|为传出区域流量启用MSS限制|
|input|string|-|传入区域流量的默认策略（接受，拒绝，拒绝）|
|forward|string|-|转发区域流量的默认策略|
|output|string|-|传出区域流量的默认策略|

3、redirect

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|src|zone name|-|指定流量源区域，必须引用已定义的区域名称之一|
|src_ip|ip address|-|匹配来自指定源IP地址的传入流量|
|src_dip|ip address|-|对于DNAT，匹配定向到给定目标IP地址的传入流量。对于SNAT，将源地址重写为给定地址|
|src_mac|mac address|-|匹配来自指定mac地址的传入流量|
|src_port|port or range|-|匹配来自客户端主机上给定源端口或端口范围的传入流量|
|src_dport|port or range|-|对于DNAT，匹配定向到该主机上给定目标端口或端口范围的传入流量。对于SNAT，将源端口重写为给定值|
|proto|protocol name or number|tcpudp|使用给定的协议匹配传入流量|
|dest|zone name|-|指定流量目标区域，必须引用定义的区域名称之一|
|dest_ip|ip address|-|对于DNAT，将匹配的传入流量重定向到指定的内部主机。对于SNAT，匹配定向到给定地址的流量|
|dest_port|port or range|-|对于DNAT，将匹配的传入流量重定向到内部主机上的给定端口。对于SNAT，匹配定向到给定端口的流量|
|target|string|DNAT|生成规则时要使用的NAT目标（DNAT或SNAT）|
|family|string|any|为其生成iptables规则的协议族（ipv4，ipv6或任何协议）|
|reflection|boolean|1|如果设置为0，则禁用此重定向的NAT反射(适用于DNAT目标)|

4、forwrading

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|src|zone name|-|指定流量源区域，必须引用已定义的区域名称之一|
|dest|zone name|-|指定流量目标区域，必须引用已定义的区域名称之一|
|family|string|-|为其生成iptables规则的协议族（ipv4，ipv6或任何协议）|

5、rule

| 选项 |值类型|默认值|描述 |
| :---: |:---:|:---:| :---: |
|src|zone name|-|指定流量源区域，必须引用定义的区域名称之一|
|src_ip|ip address|-|匹配来自指定源IP地址的传入流量|
|src_mac|mac address|-|匹配来自指定mac地址的传入流量|
|src_port|port or range|-|匹配来自客户端主机上给定源端口或端口范围的传入流量|
|proto|protocol name or number|-|使用给定的协议匹配传入流量|
|dest|zone name|-|指定流量目标区域，必须引用定义的区域名称之一|
|dest_ip|ip address|-|匹配定向到指定目标IP地址的传入流量|
|dest_port|port or range|-|匹配定向到此主机上给定目标端口或端口范围的传入流量|
|target|string|-|针对匹配流量的防火墙操作（接受，拒绝，删除）|
|family|string|-|为其生成iptables规则的协议族（ipv4，ipv6或任何协议）|

## 6 FAQ

- **Q：已更改配置并使用uci commit，为什么配置未生效？**
  
  A：uci commit只是保存config文件配置。要使配置生效，还需要将相关进程重启一遍。例如：修改network配置后需要执行/etc/init.d/network restart；修改wireless后需要执行wifi reload
