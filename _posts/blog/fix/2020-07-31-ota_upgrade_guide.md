---
layout: post
title: OTA系统升级开发手册
categories: CUSTOM
description: 介绍siflower OTA升级开发
keywords:  OTA 
mermaid: true
---


# OTA系统升级开发手册

* TOC
{:toc}

## 1. 介绍  

### 1.1 适用人员  

适用于使用siflower管理页面开发OTA功能的人员

### 1.2 开发环境

ubuntu系统环境
已完成编译的siflower SDK
已完成注册的siflower 开放平台
正常工作的siflower 硬件版型

### 1.3 相关背景

OTA系统升级主要是为使用siflower luci界面的客户，提供OTA升级管理功能，保证产品出厂后对软件进行统一的管理升级，保证软件实时更新  
OTA升级功能已经集成在代码中，使用siflower luci管理页面时可以直接使用  
目前介绍的OTA功能开发说明，是基于sflower管理页面  
**使用siflower管理页面OTA功能进行升级管理会增加服务器额外流量，请先联系矽昌官方，获取siflower管理页面，以及服务器的使用权限，即可按照本文说明进行操作**

### 1.4 功能概述

OTA升级在路由器需要的情况下，可以向服务器请求是否有新的可升级镜像，服务器检查路由器发送过来的信息，判断是否需要升级，可以升级的情况下，提供新镜像的信息和下载地址给路由器    
路由器收到服务给到的新镜像信息后，下载新的升级镜像，并且对路由器进行升级

## 2 项目引用

### 参考文档  

开放平台使用参考[矽昌开放平台用户手册](https://siflower.github.io/2020/09/12/open_platform_user_guide/)

factory分区值使用参考[FLASH分区开发手册](https://siflower.github.io/2020/09/08/flashPartitionGuide/)

### 内部相关   

参考[redmine6302](http://redmine.siflower.cn/redmine/issues/6302) 

参考[redmine65](http://redmine.siflower.cn/redmine/issues/65) 

参考[redmine99](http://redmine.siflower.cn/redmine/issues/99) 

## 3 开发详情  

### 3.1 功能设计和流程 

#### 3.1.1 功能设计

- 流程图
  
  ```mermaid
  graph LR
      SiflowerOpen服务器-->|发送下载信息|SiflowerDevice;  
      SiflowerDevice-->|发送更新请求|SiflowerOpen服务器;
  ```

- 服务器提供的接口
  
  目前服务器提供有两个OTA升级镜像查询接口 

  >lookImgVersion  
  上传单个ROM 信息，服务器返回单个ROM更新信息  
  lookImgVersions  
  上传多个ROM 信息，服务器返回多个ROM更新信息  

- 路由器上传的json信息
  
  ```
  {
      "imagetype":0,
      "romtype":0,
      "version":"openwrt_release_p10_fullmask_rel_4.1.3_77434d7",
      "env":0,
      "chip":"fullmask",
      "sn":"ffffffffffffffffff"
  }
  ```

  **主要参数：**  

  romtype：表示对应的版型，主要通过矽昌开放平台注册项目，生成硬件产品获取  
  version：本地当前软件版本号，软件版本号是编译镜像时生成的，不需要用客手动修改  
  env：0表示稳定版本，1表示开发版本，可根据客户需求选择  
  chip：芯片类型，该参数默认为fullmask，不需要客户手动修改  
  sn：路由器sn号,sn号是路由器启动后与服务器交互，有服务器自动生成，不需要客户手动修改  
  imagetype：该参数默认为0，不需要客户手动修改  

- 服务器返回的信息
  
  ```
  {
  "code": 0,
  "msg": "ok",
  "data": {
  "currentVersion": {
  "objectId": "1039767039989776384",
  "createAt": "2018-09-12 14:45:56",
  "updateAt": "2018-09-12 14:45:56",
  "checksum": "",
  "path": "https://cloud.siflower.cn/v1/file/download?id=5b8ce5b83d222b11b54f6135",
  "version": "openwrt_master_p10_fullmask_3.1.0_0ad1212",
  "size": 8912900,
  "releaseNote": "eeeeeeeeee",
  "imageType": 0,
  "romtype": 0,
  "env": 0,
  "chip": "fullmask",
  "major": 3,
  "minor": 1,
  "build": 0,
  "revision": 0,
  "key": null,
  "force": false,
  "stop": false
  },
  "updateToVersion": {
  "objectId": "894887131028127745",
  "createAt": "2017-08-08 19:44:52",
  "updateAt": "2018-09-12 13:56:30",
  "checksum": "611b1a424f124828ee8b6fc78d6ecdf2",
  "path": "https://cloud.siflower.cn/v1/file/download?id=5b8ce5b83d222b11b54f6135",
  "version": "1.3.0",
  "size": 7077892,
  "releaseNote": null,
  "imageType": 0,
  "romtype": 0,
  "env": 0,
  "chip": "mpw0",
  "major": 1,
  "minor": 3,
  "build": 0,
  "revision": 0,
  "key": null,
  "force": true,
  "stop": false
  }
  },
  "other": null
  ```


  **主要返回值**  

  updateToVersion: 表示将要升级的新版本的具体信息  
  该json块内  
  force: 表示该版本是否需要强制升级，如果是强制升级的版本，我们需要在后台检查到的时候，进行静默升级  
  checksum: 用来校验下载镜像的完整性  
  version: 用来和本地版本号进行对比  
  path: 代表了需要升级镜像的URL

  其他情况下，服务器会返回不需要升级及升级错误原因

#### 3.1.1 OTA流程

目前使用siflowerOTA升级功能，需要使用的开放平台作为服务器，可以对项目信息以及软件版本做统一管理。整个使用流程如下

  ```mermaid  
      graph LR;
          开放平台注册-->申请产品-->获取OTA参数-->手动写入分区-->上传镜像到开放平台-->测试OTA功能-->通过生产写入-->通过开放平台项目进行管控软件升级;
  ``` 

- 注册账号，生成项目 
  
  参考[矽昌开放平台用户手册](https://siflower.github.io/2020/09/12/open_platform_user_guide/)  
  
  ![open_page](/assets/images/ota_image/open_page.png) 

- 申请硬件产品 
  
  - 生成产品信息，获取产品对应的romtype  
  
    生成的romtype用于升级时判断，productkey用于app绑定  
    ![product_1](/assets/images/ota_image/product_1.png)  

  - 产品镜像列表
  
    ![product-2](/assets/images/ota_image/product_2.png)

    产品固件为后续维护镜像的上传列表。可以选择正式版本和开发版本  
  
      >若上传的版本名称带master，则将镜像传到开发版本列表  
      若上传的版本名称不带master，则将镜像传到正式版本列表  

- 设置OTA参数  
  
  romtype作为OTA升级的时候最重要的参数，需要预先写入板子中保存，目前有两种方式保存romtype  

  1)在产品出厂的时候写入factory分区中的romtype，这个是在产品出厂的时候写入的，可以通过串口或者ssh进入系统后台用以下指令读取  

    ```
    cat sys/devices/platform/factory-read/rom_type  
    cat sys/devices/platform/factory-read/rom_type_flag
    ```  

    其中rom_type_flag必须为"rt"时romtype才有效  
    手动修改指令如下，比如修改romtype为0x31，可以通过串口或者ssh进入系统后台用以下指令进行修改

    ```
    printf "\x31\x00\x00\x00"| dd of=/dev/mtdblock3 bs=1 count=4 seek=161   
    printf "rt"| dd of=/dev/mtdblock3 bs=1 count=2 seek=159  
    ```


  2)通过修改配置文件实现修改OTA参数  

    **当系统没有从factory分区中获得OTA参数**,系统就会从配置文件/etc/config/basic_setting中获取OTA参数，如下图配置文件中的‘ota’就是OTA的参数  
    ![ota_arg](/assets/images/ota_image/ota_arg.png)

- 上传两个镜像到开放平台固件列表
  
  - 上传镜像
  
    ![ota_up_bin](/assets/images/ota_image/ota_up_bin.png)  

  - 镜像列表
  
    ![ota_bin_list](/assets/images/ota_image/ota_bin_list.png)

- 测试OTA升级
  
  当OTA参数正确，并且开放平台镜像上传正确后，可以进入管理网页->高级设置->设备管理—>软件升级->ota在线升级  

   ![ota_page](/assets/images/ota_image/ota_page.png)

- 生产写入OTA信息
  
  测试OTA升级没有问题后，后续正式生产可以将OTA信息通过pcba测试工具写入factory分区，用于出厂后软件版本管理  

  参考[生产工具说明](https://developers.siflower.cn/#/documentDisplay?docId=75)关于board.ini配置文件的说明

- 产品软件管理
  
  当设备写入相关信息后，用户可以通过开放平台的项目控制台，已经项目镜像版本管理，对设备注册情况，软件更新升级做统一管控

### 3.2 代码架构和实现 

#### 3.2.1 软件流程

目前siflower的luci管理页面已默认支持OTA功能，获取到romtype后，按照上述流程即可直接使用OTA功能

- OTA升级流程图
  
  ```mermaid
  graph TB;
      开始(开始)-->获取OTA信息(获取OTA信息)-->判断1{根据服务器返回的信息判断是否有版本更新}--是-->下载镜像(下载镜像);
      判断1{根据服务器返回的信息判断是否有版本更新}--否-->结束(结束);
      下载镜像(下载镜像)-->判断2{服务器返回的checksum与本地文件的checksum是否相同}--是-->升级(升级);
      判断2{服务器返回的checksum与本地文件的checksum是否相同}--否-->结束(结束)
  ```

#### 3.2.2 关键接口

- 网页升级

  在feeds/luci/modules/luci-mod-admin-full-siflower/luasrc/siwifi/networkImpl.lua中  
  ota_upgrade函数，完成了本地版本信息整理，调用相关服务器接口，解析服务器返回信息  

  在需要升级的情况下，将升级所需要的下载地址等信息存入“/tmp/ota_info” 文件中  
  然后fork进程  
  ota upgrade进程会负责读取 /tmp/ota_info 中的信息，下载完整镜像，并且进行升级   

- app升级
  
  如果是通过app升级，升级过程中会通过sysutil.sendCommandToLocalSever(cmd,cmd_ret),将下载进度通过服务器，远程通知给app  
  升级过程中会不断更新/tmp/upgrade_status 文件，来表示升级进度，便于页面显示进度

  在feeds/luci/modules/luci-mod-admin-full-siflower/luasrc/controller/api/sfsystem.lua中  
  提供了ota_check、ota_check2、ota_upgrade等接口给app调用  
  ota_check2 返回 升级镜像的相关信息，用于显示  
  ota_upgrade 启动ota升级流程，升级过程中，路由器会通过服务器向app传递升级镜像下载进度


- 后台OTA升级实现

  - 后台升级是指通过预设的定时启动程序进行定时的OTA升级请求  
  
    主要是通过启动package/siflower/bin/utils/files/base-files/sbin/auto_ota 程序进行自动升级，该程序也实现了类似networkImpl中的ota_upgrade接口功能  
    向服务器请求升级镜像信息，在判断force返回值为1的情况下，会调用otaupgrade进行强制升级  

  - 用户操作
  
    用户可通过siwifi.cn->高级设置->设备管理->软件升级中的是否自动升级开关选择是否进行后台OTA升级，自动升级开关默认为打开状态  

  - auto_ota实现
  
    自动OTA的时间设置  
    可以根据需求在编译镜像前通过修改对应板型basic_setting配置文件中时间节点的值来修改自动OTA的时间  
    ![ota_page](/assets/images/ota_image/ota_auto_time.png)



### 3.3 OTA使用关键点

- 开放平台注册，获取romtype
  
- OTA请求时上传的参数一定要正确
  
  可以在系统平台通过curl进行测试或者利用第三方postman等工具测试参数是否正确

  如果OTA请求失败请仔细检查post参数

- 开放平台镜像上传正确，且路由器自身软件必须在镜像列表

## 4 测试demo

- 在已有的SDK开发环境通过脚本编译两版版本号不同的镜像
  
  因为脚本编译会调用当前tag来生成镜像名字，我们可以利用这个方法来编译两版不同的镜像

  编译第一个镜像之前打tag 1.0.6，然后进行编译

    ```
    git tag -a 1.0.6 -m “test 1.0.6”
    ```


  编译第二次再打一次tag 1.0.7，然后再编译

    ```
    git tag -a 1.0.7 -m “test 1.0.7”
    ```


  测试完后删除tag

    ```
    git tag -d 1.0.6
    git tag -d 1.0.7
    ```


- 获取准备测试的两版镜像的校验值  

  ![ota_test1](/assets/images/ota_image/ota_test1.png)  

- 上传至开放平台产品固件列表
  
  先上传1.0.6再上传1.0.7

  ![ota_test2](/assets/images/ota_image/ota_test2.png)  

- 修改本地romtype参数与开放平台一致
  
  下图为开放平台rom_type信息  

    ![ota_test3](/assets/images/ota_image/ota_test3.png)  

  按照下图进行修改  
 
    ![ota_test4](/assets/images/ota_image/ota_test4.png)  

  **注意：修改后，重启板子这两个值才会在factory分区生效**  

- 将先上传的镜像版本烧录到本地
  
  **OTA升级时需要保证本地烧录的镜像在镜像列表，且镜像版本不是最新的版本**

- 登陆网页开始OTA测试
  
## 5 测试用例  

### 5.1 测试环境

- 测试环境
  
  windows PC  
  串口调试工具  
  正常编译的siflower SDK  
  正常工作的siflower硬件平台  
  siflower开放平台产品控制台  

### 5.2 测试方法

- 测试目标
  
  编译两版镜像，上传开放平台，通过管理网页测试OTA升级功成功从低版本升级至高版本

- 按照测试demo进行修改和准备
  
- 进入网页升级
  
  - 将设备通过wan口接通外网  

  - 打开管理网页点击OTA升级  

    ![ota_test5](/assets/images/ota_image/ota_test5.png)  

  - 同时接上串口打看logread -f查看升级日志，便于出错调试  

    ![ota_test6](/assets/images/ota_image/ota_test6.png)  

- 升级完成后重启查看镜像版本  
  
  ![ota_test7](/assets/images/ota_image/ota_test7.png)  

### 5.3 测试结果

OTA升级成功，查看镜像版本已更新  

## FAQ

**Q：OTA升级提示失败？**    
 A：1) 通过串口logred -f指令查看请求的json参数是否正确  
 2) 确认本地烧录版本在固件列表  
 3) 检查校验值是否上传正确  
 4) 检查网络环境  
