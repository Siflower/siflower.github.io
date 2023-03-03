---
layout: post
title: WiFi国家码设置与修改
categories: WIFI
description: 介绍WiFi国家码设置与修改
keywords: wifi country mode
mermaid: true
---
# WiFi国家码设置与修改

**目录**  
* TOC
{:toc}


## 1 介绍

### 1.1 适用人员  

 - 熟悉wifi配置，见[WiFi架构和配置手册](https://siflower.github.io/2020/08/12/wifi_architecture_and_configuration_manual/)

### 1.2 开发环境  

 - 正常的编译和运行环境，见[快速入门](https://siflower.github.io/2020/08/05/quick_start/)

### 1.3 功能概述  

本文介绍了研发人员如何查看、设置、修改wifi国家码相关知识。


## 2.国家码的设置    

### 2.1 串口直接设置  
`iw reg set <国家码alpha2>`   

例如美国的国家码是US，则设置美国国家码的命令是：`iw reg set US`  
修改后通过`iw reg get`可以看到修改后的国家信道，如下图所示:  

![get.png](/assets/images/wifi_country_mode/get.png)  

该图主要关注几个地方：  
(1) country US，表示当前遵循的是美国允许的信道列表;  
(2) DFS-FCC表示该国的信道规划遵循的是FCC标准；  
(3) **(5170 - 5250 @ 80)**表示其中允许的一段频带为5170M到5250M，对应的信道是36到46信道，因为36信道带宽20M，它的中心频率是5180M，占据的频带是5170M ~ 5190M；@80表示的是允许的最大带宽是80M，即VHT80；  	
(4) 30、23、40表示的是该频带所允许的最大发射功率，单位为dbm；  
(5) DFS表明该频带是该国划定的雷达信道； 
(6) (57240 - 63720)是802.11ad协议的内容，使用60GHz频段，目前不支持；  
(7) (2402 - 2482 @ 40)规定了2.4G频段的信道列表，相比5G频段，各国的2.4G相对一致。大部分国家都是2402M到2482M频带，也就是1 ~ 13信道，目前只有日本支持了14(2484M)信道;  

### 2.2 通过配置文件配置  
- 通过`/etc/config/wireless`文件配置，修改country后的国家字段，同样是使用alpha-2国家码，需要注意的是通过wireless文件配置，**必须同时修改wlan0和wlan1的country选项**，保存后通过wifi reload 命令生效。同样可以通过`iw reg get` 查看修改后的国家信道列表。  

- 各国允许的中心频率和带宽不尽相同，例如可能会存在161信道、80M带宽，修改前为CN，修改后为EG(埃及)的情况。这时候由于埃及所允许的5g信道列表只有36到64信道，带宽最大为20M，所以修改后会出现wlan1起不来的情况，此时（或者提前）需要将信道和带宽设为EG所允许的信道和带宽。  
	
- 值得注意的是修改wireless文件后iw reg get可以得到正确的修改后的国家码以及对应的信道列表，但是**通过iw reg set XX修改国家码，wireless文件对应的country国家码却不会对应的发生改变**，因此比较建议通过wireless文件修改。  


## 3.国家码的修改

### 3.1 使用openwrt系统  

通过wireless-regdb将db.txt转换为regulatory.db文件来查找国家码，如果有修改国家信道列表的需求，比如说某个国家的信道表不符合要求，则只需修改`build_dir/target-mipsel_mips-interAptiv_musl/wireless-regdb-2017-10-20-4343d359`目录下的`db.txt`即可。  

### 3.2 使用原生linux的cfg80211   

需要make menuconfig使能**CONFIG_CFG80211_INTERNAL_REGDB**宏，并修改`net/wireless/db.txt`里的国家信道表，重新编译，编译后会生成**regdb.c**文件。

### 3.3 使用如backports的linux系统  

在openwrt上根据需求做修改后，提供生成的对应的`build_dir/target-mipsel_mips-interAptiv_musl/wireless-regdb-2017-10-20-4343d359`目录下的`regulatory.db`文件   
db.txt中由类似下图中一个一个的国家及其信道表所组成，修改时首先找到对应的国家，具体的修改则根据需求具体实现  

![db.png](/assets/images/wifi_country_mode/db.png)  
	
## 4. 如何根据信道顺从表修改信道

![channel.png](/assets/images/wifi_country_mode/channel.png)
	
![channel2.png](/assets/images/wifi_country_mode/channel2.png)
	
以图上的埃及为例。首先需要找到埃及对应的alpha-2国家码，找到为EG，然后根据上面说的3种情况找到对应的db.txt中EG的信道列表，如果是使用openwrt系统（2.1）或linux系统（2.3）的情况，需要打patch，**wireless-regdb在本地的patch放在package/firmware/wireless-regdb/下**。  
另外要明白各个信道对应的中心频率，可以通过**iw phy0/phy1 info 获取驱动支持的所有信道及其中心频率**。  

- 如第二张图的红色方框里边显示，db.txt中**一般将5G信道划分为4个频带**，一一对应表格与db.txt中的划分范围是否一致；  
- 关注允许的最大带宽和最大发射功率，5G信道允许的带宽包括20M、40M、80M、160M，这张图中**80M和160M的信道只列出了第一个20M主信道，实际上每个子信道都是可以用的**，标N/A则表明该国家不支持这个带宽，最大发射功率如红色圆框所注； 
- 最后比对雷达信道DFS标识是否正确。
	