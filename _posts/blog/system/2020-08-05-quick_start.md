---
layout: post
title: 快速入门手册
categories: SYSTEM 
description: 介绍siflower平台快速入门的方法
keywords:  快速入门
mermaid: true
topmost: true
---

* TOC
{:toc}

# 快速入门手册

## 1 介绍

本文档主要介绍如何从头开始安装开发环境，将通过一个简单的例子来说明如何在电脑上搭建siflower方案的开发环境  
包括SDK代码下载、安装编译环境、编译和烧录

### 1.1 适用人员

主要适用于需要对siflower系统的开发人员  

### 1.2 开发环境

进行siflower方案开发需要准备：  
电脑：安装ubuntu14.04或ubuntu16.04版本操作系统  
硬件平台、电源线、串口线、网线、  

Siwifi的系统方案源于openwrt开源系统，必须基于GNU/Linux, BSD or MacOSX进行编译  
为了更顺利的进行开发，我们推荐使用ubuntu作为默认的编译环境，**ubuntu14.04、ubuntu16.04**都是经过详细测试的操作系统版本，其他系统版本可能会存在编译问题
windows下安装，参考[虚拟机安装及编译环境配置手册](https://siflower.github.io/2020/08/05/ubuntu_install_complile_env_config_guide/)这里有详细的关于虚拟机安装，ubuntu编译环境配置等步骤

## 2 快速入门详情

### 2.1 环境安装和开发流程

**快速入门流程图**

```mermaid
graph TB;
    环境准备(开发环境准备)-->源码下载(源码下载)-->源码编译(源码编译)-->烧录镜像(烧录镜像);
```

#### 2.1.1 环境准备

- 参考[虚拟机安装及编译环境配置手册](https://siflower.github.io/2020/08/05/ubuntu_install_complile_env_config_guide/)搭建开发环境

- 需要有SDK支持的硬件平台
  >AC28 路由板型(intel switch芯片)  
   EVB 开发板(realtek switch芯片)

  上述对应的硬件板型可以联系矽昌官方获取  
  如果是客户新做的硬件，需要参考[新的版型引入指南](https://siflower.github.io/2020/09/08/newBoardImportGuide/)根据硬件配置建立对应的板型配置后，再进行编译

#### 2.1.2 源码下载

目前siflower的开源代码放置在github服务器上，完成环境配置后，客户可以进行下载

一种是直接下载，第二种是将代码库fork到自己账户再下载

**注：推荐先fork代码库到自己的github账户，然后在ubuntu环境下，以git clone的方式下载自己fork的代码仓库，方便代码更新和管理**

- 直接下载：进入GitHub[官方网页](https://github.com)  
  
  - 在左上角的搜索框，输入要下载的代码，如Siflower/1806_SDK，点击enter按键即可进行搜索  
  
  ![quick_1](/assets/images/quick_image/quick_1.png)

  - 点击code按键选择下载方式

    为了方便代码更新和管理，推荐使用git clone方式下载  
  
  ![quick_2](/assets/images/quick_image/quick_2.png)

- fork后下载：将源码库fork到自己的github账户再进行下载，点击页面右上角的fork按键即可  
  
  ![quick_3](/assets/images/quick_image/quick_3.png)  

  进入自己账户主页 即可看到有相关项目

  ![quick_4](/assets/images/quick_image/quick_4.png)

  选择对应项目，可以看到右上角在自己项目信息下有fork的信息，然后再选择git clone方式下载

  ![quick_4_1](/assets/images/quick_image/quick_41.png)


- 下载完成如图
  
  ![quick_5](/assets/images/quick_image/quick_5.png)

  由于服务器的问题，github下载可能会出现由于下载速率慢造成的下载失败，解决方法参考FAQ  
  如果多次尝试仍然下载不了，请联系邮箱(irving.luo@siflower.com.cn)提供离线版本  
  **注意：**  
  **1，使用离线版本会存在代码更新不方便的问题，后续仍然需要通过git clone方式下载SDK，方便通过git 指令实时更新代码**
  **2，注意慎用zip压缩包下载，可能会导致缺少文件，导致编译失败**

#### 2.1.3 源码上传

- siflower SDK主分支

  - 代码更新
   siflower github源码我们保持它是干净的，统一针对所有开发人员使用的，客户原则上不允许向sdk master分支提交任何改动  
   有代码更新会由siflower内部进行发布更新到SDK，客户进行同步即可  

  - 如果开发人员发现重大公共性的bug，需要提交代码  
   1)可以在SDK issue进行发布，由矽昌来解决并更新代码    
   2)可以通过[pull requests](https://siflower.github.io/wiki/github_pull_reqest_submit/)的方法提交代码到master分支，由矽昌进行审核之后再合入代码  

- fork 分支
  
  下载源码时，选择fork代码库然后再下载  
  如果需要更新或者上传代码可以参考[pull requests](https://siflower.github.io/wiki/github_pull_reqest_submit/)的方法提交更新

#### 2.1.4 源码目录

![quick_5_1](/assets/images/quick_image/quick_5_1.png)

> openwrt-18.06/  
  这个是openwrt目录，代码修改，系统镜像编译，在此目录下进行  

>linux-4.14.90-dev/  
  这个是linux内核目录，涉及到内核修改，dts修改，驱动修改，在此目录下进行  

#### 2.1.5 源码编译

##### 2.1.5.1 openwrt编译

- 进入openwrt-18.06/目录

- 第一次编译请使用 make.sh 脚本进行编译
  
  ```
  ./make.sh a28_evb  
  ```

  首次编译需要通过脚本选择硬件平台进行编译，a28_evb为矽昌SF19A2890开发板，更换板型只需要更换此参数 

  |版型|编译命令|
  |--|--|
  |开发板|./make.sh a28_evb|
  |产品板|./make.sh a28_ac28| 

  如果是新建的版型，使用新建的版型名称编译即可。

  **如果编译出错请参考FAQ**

  首次编译成功后，会在编译根目录生成一个如openwrt1806_SDK_a28_evb_sf19a28_fullmask_rel_.bin的镜像  
  **如果用脚本编译出错后，使用FAQ提供的方法继续编译成功，此时不会生成镜像在根目录，再次使用脚本编译一次则正常生成**

- 后续编译
  第一次脚本编译通过后，在不更换板型的情况下，会默认使用根目录.config配置
  即使用make menuconfig修改之后，可以用以下指令直接编译

  ```
  make V=s
  make -j1 V=s
  ```

- 编译指令说明
   
  >make指令编译  
  会默认使用根目录下的.config文件选中相应的package进行编译  
  -j1 参数代表使用多少个线程同时编译，可以提升编译速度  
  V=s 参数表示显示详细的编译过程  


  >make.sh脚本编译  
  例如./make.sh a28_evb 脚本编译  
  会使用SDK中版型默认的配置文件  
  即target/linux/siflower/sf19a28_evb_fullmask_def.config来覆盖当前的.config文件(首次编译则直接选其取作为config)，选中config里相应的配置进行编译



  >使用脚本编译和使用指令的区别在于会使用target/linux/siflower/下默认板型配置文件，并且在根目录下会生成一个带版型名称和版本号的镜像  
  这个镜像与bin/target/siflower/目录下的一样，只是通过make.sh脚本对其重新命名，详情可以参考脚本中的代码实现


  调试板型固定时，修改代码后，使用make指令编译生成镜像测试功能即可  
  如果需要将.config的配置永久保存到默认版型配置，可以使用如下指令拷贝替换对应版型的默认配置，防止在清理编译环境时清除.config
  如果需要生成带板型名称以及版本号的镜像，请使用脚本编译  

  ```
  cp .config target/linux/siflower/sf19a28_evb_fullmask.config
  ```


  注意：更换版型或者对target/linux/sifower/下版型默认config文件做了修改，编译必须使用脚本，这样才会去调用对应的配置文件或者修改进行编译  
  如果只是通过make menuconfig修改那只是修改了根目录的.config

- 镜像路径
    
  不管使用脚本编译还是指令编译，编译成功后镜像都会生成在如下目录  
   
  ```
  /bin/target/siflower/openwrt-siflower-sf19a28-fullmask-squashfs-sysupgrade.bin
  ```

  使用脚本编译会额外在根目录生成一个名称带有分支名称板型名称版本号的镜像

- 单独编译
  
  - 单独编译内核包

  ```
  make target/linux/{clean,compile} V=s
  ```

  - 单独编译软件包
  
  ```
  make package/libs/mbedtls/{clean,compile} V=s
  ```


- 清理编译环境

  编译环境需要定期清理，这样会减小环境对编译的影响，减少不必要的错误

  ```
  1)make clean
  删除目录/bin和/build_dir的内容。 使清洁不会删除工具链，它也可以避免清除不同于您在.config中选择的体系结构/目标

  2)make dirclean
  删除目录/bin和/build_dir以及/staging_dir和/toolchain（=交叉编译工具）和/logs的内容。“Dirclean”是您的基本“全面清理”操作。

  3)make distclean
  将编译或配置的所有内容都删除，并删除所有已下载的提要内容和程序包源。**此指令谨慎使用**

  4）make target/linux/clean
  清理linux对象。

  5)make package/luci/clean
  清理luci包对象。
  ```

#### 2.1.6 更新镜像

##### 镜像获取 

- uboot镜像
  
  uboot镜像一般烧录好后不会轻易更改，除非硬件配置有改动(ddr/flash/switch等),请联系XC获取

- openwrt镜像 
  
  使用SDK编译的镜像即为对应的openwrt镜像，可以使用进行烧录更新

- 完整的FLASH镜像
  
  参考[PCBA工具使用手册](https://siflower.github.io/2020/09/10/pcba_tool_interface_guide/)FAQ章节PCBA镜像说明


##### 2.1.6.1 网页更新

如果路由器本身的系统是可以正常启动的，可以登录路由器的网页端进行镜像更新  

**原生界面**

- 打开浏览器，访问192.168.4.1，进入路由器登陆页面，登录密码admin
  ![quick_6_1](/assets/images/quick_image/quick_6_1.png) 

- 进入系统-->备份/升级-->刷写新的固件
  ![quick_7_1](/assets/images/quick_image/quick_7_1.png)

- 选择刷写固件，上传本地镜像
  ![quick_8_1](/assets/images/quick_image/quick_8_1.png)

- 上传成功之后，选择继续
  ![quick_8_2](/assets/images/quick_image/quick_8_2.png)

- 等待更新进程完成升级
  ![quick_8_3](/assets/images/quick_image/quick_8_3.png)

  更新镜像大约需要1-2分钟的时间，更新完毕系统会自动重启

**siflower界面**

- 打开浏览器，访问192.168.4.1，进入路由器登陆页面，登录密码admin  
  ![quick_6](/assets/images/quick_image/quick_6.png)  

- 进入高级设置-->设备管理-->软件升级
  在软件升级，点击浏览选择要升级的本地镜像  
  ![quick_7](/assets/images/quick_image/quick_7.png)  

- 点击软件升级  
  ![quick_8](/assets/images/quick_image/quick_8.png)  
  升级完成后，系统会自动重启，重启完成后，网页会自动跳转到登录页面

##### 2.1.6.2 串口更新

###### openwrt镜像更新

如果开发板内部已经有烧录好的uboot，但系统无法正常启动，那么我们只能从uboot阶段进行镜像更新，使用以太网口和串口配合来更新镜像  

- 在PC端安装一个串口应用，如“SmarTTY”  

- 用USB串口线连接PC和硬件板子串口，将波特率设置为115200；  

- 网线连接到板子上任意LAN口  
  ![quick_9](/assets/images/quick_image/quick_9.png)  

- 1.设置PC端的IP为静态IP 

  如192.168.4.100，默认网关必须和IP地址是同一网段的，且默认网关的最后一位必须为1，所以默认网关此时应设置为192.168.4.1，具体如下图所示：  

  ![quick_10](/assets/images/quick_image/quick_10.png)  

- 2.给路由器上电重启，同时敲回车键，进入Command模式  

  正常的uboot会在串口显示如下图的log信息  

  ![quick_11](/assets/images/quick_image/quick_11.png)  

- 3.在串口处输入 httpd 192.168.4.5
  
  此处的IP的最后一位不能和网关一样，即不能是1，可以用5或其他，按回车键显示如下log信息：  

  ![quick_12](/assets/images/quick_image/quick_12.png)

- 4.PC端使用浏览器访问192.168.4.5
  
  此处的IP与串口输入的IP必须一致，如果一切成功会显示如下图更新页面  

  ![quick_13](/assets/images/quick_image/quick_13.png)

- 5.选择要升级的镜像文件，点击Update firmware，开始更新  
  
  ![quick_14](/assets/images/quick_image/quick_14.png)

- 6.更新镜像大约需要1-2分钟的时间，更新完毕系统会自动重启  
  
  串口出现如上信息表示镜像更新完毕，启动完成  

  ![quick_15](/assets/images/quick_image/quick_15.png)

###### uboot更新

- 通过串口更新uboot,1-3步与更新openwrt镜像一致
  
- 4.PC端使用浏览器访问192.168.4.5/uboot.html
  
  此处的IP与串口输入的IP必须一致，如果一切成功会显示如下图更新页面

  ![update_uboot](/assets/images/quick_image/update_uboot.png)

- 5.选择要升级的uboot文件，点击Update firmware，开始更新  
  
  ![quick_14](/assets/images/quick_image/quick_14.png)

- 6.查看串口，开始更新，更新log如下，更新完成后会自动重新启动
  
  ![update_uboot_log](/assets/images/quick_image/update_uboot_log.png)

##### 2.1.6.3 IROM download更新

在板子硬件配置上**有usb otg接口**时，如果出现以下情况  
1，uboot镜像损坏，导致uboot无法正常启动  
2，flash为空，里面没有任何内容，需要烧录镜像  
可以通过Siflower的irom usb download下载工具将文件下载到FLASH中

此方法需要专业人员进行操作，获取irom下载工具以及具体操作方法请联系邮箱(irving.luo@siflower.com.cn) 

##### 2.1.6.4 烧录器更新

在板子硬件配置上**无usb otg接口**时，如果出现以下情况  
1，uboot镜像损坏，导致uboot无法正常启动  
2，flash为空，未烧录过任何镜像  
需要根据flash型号大小以及ddr型号，制作完整的flash镜像或者uboot镜像，通过FLASH烧录器对flash芯片进行烧录  


#### 2.1.7 串口调试

- 串口工具配置

- windows下串口工具  
  - 安装串口工具，点击[串口工具下载](http://sysprogs.com/SmarTTY/)  
  - usb转串口线接到电脑，确认端口号  
   ![tty_1](/assets/images/quick_image/tty_1.png)  
  - 打开串口工具，选择对应端口，波特率设置为1152000  
   ![tty_2](/assets/images/quick_image/tty_2.png)  
  - 串口另一端接到板子，上电，成功启动后，通过敲击键盘任意按键进入系统后台  
    串口连接板子方式如下:  
    绿线：连接RX口  
    白线：连接TX口  
    黑线：接地(GND)  
   ![tty_3](/assets/images/quick_image/tty_3.png)  
- ubuntu下串口工具
  - 默认情况下ubuntu已经安装了USB转串口驱动(pl2303)
    使用以下指令查看  
    ![minicom_1](/assets/images/quick_image/minicom_1.png)
  - minicom设置
    - 首先使用
       ```
      sudo minicom -s
      ``` 
    - 进入设置  
    ![minicom_s](/assets/images/quick_image/minicom_s.png)  

    - 选择Serial port setup，按照下图进行设置  
    ![minicom_set](/assets/images/quick_image/minicom_set.png)  
    此时所示光标在"Change which setting"上，键入"A"，此时光标移到第A项对应处:输入/dev/ttyUSB0  
    然后对波特率，数据位和停止位进行配置，键入"E"，波特率选为115200 8N1（数据位8，奇偶校验无，停止位1)，硬/软件流控制分别键入"F"和"G"，并且都选No  
    在确认配置正确之后，可键入回车返回上级配置界面  

    - 将其保存为默认配置（即save setup as dfl）  
    ![minicom_save](/assets/images/quick_image/minicom_save.png)  

    - 最后选择"Exit from Minicom"命令退出  
    ![minicom_exit](/assets/images/quick_image/minicom_exit.png)  

  - 然后输入命令打开串口，可以在最下方看到串口的配置信息  
    
    ```
    sudo minicom
    ```

    ![minicom_2](/assets/images/quick_image/minicom_2.png)  

  - 串口另一端接到板子，上电，成功启动后，通过敲击键盘任意按键进入系统后台   
    ![minicom_3](/assets/images/quick_image/minicom_3.png)  

- 进入系统后，即可通过linux命令进行操作调试


#### 2.1.8 putty/scp工具使用

##### 2.1.8.1 putty登陆板子系统

[ssh登陆工具下载](https://www.putty.org/)  

- 路由器通过网线或者无线与PC相连，打开带SSH的工具如putty，通过SSH登陆路由器
  
- 打开软件host name输入路由器IP 192.168.4.1  

  ![putty_1](/assets/images/quick_image/putty_1.png)

- 输入登陆账号及密码，默认都为admin  

  ![putty_2](/assets/images/quick_image/putty_2.png)

- 进入路由器后台系统，并进入根目录  
  
  ![putty_3](/assets/images/quick_image/putty_3.png)

- 查看相应目录和文件  
  
  ![putty_4](/assets/images/quick_image/putty_4.png)

##### 2.1.8.2 winSCP使用 

以替换网页图片和config文件为例介绍winSCP使用  

[winscp工具下载](https://winscp.net/eng/docs/lang:chs)  

- pc通过有线网络或者无线网络与板子相连
  
- 打开winSCP 工具，点击新建站点，弹出会话框，文件协议选择SCP， 主机名为192.168.4.1，端口号22，用户名admin， 密码admin  
  
  ![scp_1](/assets/images/quick_image/scp_1.png)  

- 点击登录，进入SCP模式，进入本地修改好的image文件夹，同时在路由器后台目录上找到image目录，将整个文件夹或者单个图片拖到路由器的目录  
  
  ![scp_2](/assets/images/quick_image/scp_2.png)  

- 点击是按钮，进行替换  
  
  ![scp_3](/assets/images/quick_image/scp_2.png)

- 然后进入本地修改好的style文件所在的文件夹，同时在路由器后台目录找到style文件所在的目录，将style文件拖到路由器的/etc/config/目录  


## FAQ

**Q：编译出现图示问题，怎么解决？**    

![faq2](/assets/images/quick_image/faq2.png)   

A：这个是由于编译指令漏了项目名或者项目名错误。具体支持的版型可以查看make.sh脚本代码中的参数选项。
例如编译ac28版型：./make.sh a28_ac28

**Q：第一次使用脚本编译出现图示问题，怎么解决？**  
![faq3](/assets/images/quick_image/faq3.png)   

![faq4](/assets/images/quick_image/faq4.png)   

A：出现图示问题是由于第一次脚本编译，有一些package的依赖没有选上导致编译不通过    
可以尝试在这个状态下继续使用  
```make -j1 V=s```继续编译  
或者```make -j4 V=s```、 ```make -j8 V=s```，使用多个线程，提高编译速度继续编译    

**如果是使用的虚拟机，请将虚拟机的内存改大，4G及以上**

**Q：脚本编译的项目如何添加？**  

A：因为硬件的不同，编译时需要对版型进行区分选择相应的配置文件   

参考[新的版型引入指南](https://siflower.github.io/2020/09/08/newBoardImportGuide/)  

**Q：如何判断编译成功？**   
A：出现图示log即为编译成功。编译成功会在/bin/target/siflower/下生成对应的镜像文件  

![faq5](/assets/images/quick_image/faq5.png)   

**Q：编译uboot出现下图错误如何解决？**    

![faq6](/assets/images/quick_image/faq6.png)     

A：这个是由于系统的dtc版本太低导致的，使用sudo apt-get install device-tree-compiler安装更新即可成功编译  

**Q：板子上电反复出现下图log，是什么原因？**  

![faq7](/assets/images/quick_image/faq7.png)  

A：这是由于FLASH为空，为烧录任何程序打印的芯片内部的log  

需要通过flash烧录器烧录镜像，或者使用siflower irom下载工具进行烧录

**Q：git代码下载不成功出现图示问题，怎么解决？**   

![faq1](/assets/images/quick_image/faq1.png)   

 A：(1)由于系统环境自带的git缓存buffer较小导致,可以参考[buffer修改](https://blog.csdn.net/zcmain/article/details/76855595) 行修改本地git缓存  

(2)网络状况不好导致，如果下载时下载速度一直在90kb/s以下，会导致下载失败，建议待网络环境良好或者更换下载时间段进行下载，可能仍然需要花长时间下载   

(3)如果办公环境下载速度一直在几十kb/s左右，可以尝试使用电脑无线网卡连接手机热点进行下载，速度会有很大提升，但是会有一点浪费流量，请谨慎选择此方法。

(4)使用gitee码云下载

- 注册一个[gitee账户](https://gitee.com/)

- 获取已有的github代码库链接，siflower即为https://github.com/Siflower/1806_SDK.git

- 进入gitee工作台，新建仓库
  
  ![faq1_1](/assets/images/quick_image/faq1_1.png)

- 开始导入github代码仓库到码云
  
  ![faq1_2](/assets/images/quick_image/faq1_2.png)

- 将需要导入的github链接填入

  ![faq1_3](/assets/images/quick_image/faq1_3.png)

- 点击创建，等待仓库导入
  
  ![faq1_4](/assets/images/quick_image/faq1_4.png)

 - 导入成功后点击下载，获取下载链接

  ![faq1_5](/assets/images/quick_image/faq1_5.png)

- 回到ubuntu环境，使用gitee下载链接下载代码

  ![faq1_6](/assets/images/quick_image/faq1_6.png)

  此时可以看到，下载速度可以到达5M/s，这是因为gitee服务器设置在国内，通过gitee将代码仓库导入后，会显著的提升下载速度  
  这样下载的区别是 clone 链接换成了目标项目在gitee中的链接

- 如何保持与github主仓库代码一致
  
  点击gitee代码仓库此按钮，即可强制同步github主分支的更新

  ![faq1_7](/assets/images/quick_image/faq1_7.png)





