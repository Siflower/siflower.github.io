---
layout: post
title: 虚拟机安装及编译环境配置手册
categories: SYSTEM 
description: 介绍如何安装虚拟机、ubuntu以及配置开发环境
keywords: ubuntu 开发环境
mermaid: false
---

* TOC
{:toc}

# 虚拟机安装及编译环境配置手册

## 虚拟机软件下载安装

[虚拟机下载](https://my.vmware.com/cn/web/vmware/downloads/#products_atoz)  

## 虚拟机软件安装

### 下载

点击上述虚拟机下载链接，下载虚拟机安装包，这里推荐的是VMware虚拟机  

- 选择产品列表VMware Workstation Pro  
  
    ![vm_1](/assets/images/quick_image/vm_1.png)

- 选择产品后跳转点击GO to download  
  
    ![vm_2](/assets/images/quick_image/vm_2.png)

- 选择download now ，本文选择15.5.6版本
    这里需要登陆，请自行注册账号登陆进入下载页面  

    ![vm_3](/assets/images/quick_image/vm_3.png)

- 开始下载  
  
    ![vm_4](/assets/images/quick_image/vm_4.png)

### 安装

- 双击下载好的安装包，我这里以15.0版本为例   
  
    ![vm_5](/assets/images/quick_image/vm_5.png)  

- 选择安装位置，默认选择C盘，建议单独使用一个空闲较大的盘安装  
  
    ![vm_6](/assets/images/quick_image/vm_6.png)  

- 开始安装  
  
    ![vm_7](/assets/images/quick_image/vm_7.png)  

- 安装完成  
  
    ![vm_8](/assets/images/quick_image/vm_8.png)  

## 新建虚拟机及ubuntu安装


- 如果不想从头开始安装虚拟机及设置ubuntu

    这里提供一个已配置好的虚拟机和对应使用的ubuntu iso镜像，可以直接在VMware中打开使用，无需设置  
    [点击下载已配置好的虚拟机](https://pan.baidu.com/s/1oD9oUzOmidX7pLGZ76Vg9Q) 提取码：g4po    
    此虚拟机对应的ubuntu iso镜像  
    [点击下载搭配虚拟机的ubuntu版本](https://pan.baidu.com/s/1hW9ZSpNkHx8FD967P-ZnNA) 提取码：upym   

    **注：提供的默认虚拟机密码为1**

- 如果想要从头开始新建虚拟机和安装一遍Ubuntu，请参考下文步骤新建虚拟机，在虚拟机中安装Ubuntu

### 下载ubuntu版本

点击[ubuntu下载](https://ubuntu.com/download/desktop)，进入官网下载相应的ubuntu-desktop版本  
提前将ubuntu版本iso下载好，后续新建虚拟机时会用到

**为了更顺利的进行开发，我们推荐使用ubuntu作为默认的编译环境，ubuntu14.04、ubuntu16.04都是经过详细测试的操作系统版本，使用其它系统版本ubuntu可能会带来其它的编译问题**  

### VMware新建虚拟机

按照正常步骤按照安装好虚拟机。

### 虚拟机安装ubuntu

- 运行虚拟机，进入安装界面，选择简体中文，安装ubuntu
  
    ![vm_26](/assets/images/quick_image/vm_26.png)

- 选择安装ubuntu时下载更新

    ![vm_27](/assets/images/quick_image/vm_27.png)

- 选择清除整个磁盘并安装ubuntu
    
    ![vm_28](/assets/images/quick_image/vm_28_2.png)

- 选择继续
  
    ![vm_29](/assets/images/quick_image/vm_29_2.png)

- 选择上海

    ![vm_30](/assets/images/quick_image/vm_37.png)

- 键盘选择
  
    ![vm_31](/assets/images/quick_image/vm_38.png)

- 输入用户信息

    ![vm_32](/assets/images/quick_image/vm_39.png)

- 开始安装

    需要一点时间安装，请耐心等待  

    ![vm_33](/assets/images/quick_image/vm_40.png)

- 安装完成，会提示重启，点击重启

    ![vm_34](/assets/images/quick_image/vm_41.png)

- 等待重启完成后进入桌面
  
    第一次进入可能桌面会是这种较小的尺寸，打开终端  
    输入 xrandr -s 1920x1440可以暂时调节尺寸为全屏  

    ![qp_1](/assets/images/quick_image/qp_1.png)

    ![qp_2](/assets/images/quick_image/qp_2.png)

    注意：这个指令关机后在开机会失效，每次开机需要重新输入设置  
    如果想要每次进来默认全屏，需要安装vmware tool进行设置  
    请参考[vmware_ubuntunc全屏调节](https://jingyan.baidu.com/article/f7ff0bfc13d02f2e26bb13af.html)  

### ubuntu编译环境设置

- 更新软件列表
  
    ```
    sudo apt-get update
    ```


    ![env_1](/assets/images/quick_image/env_1.png)

    更新完成如下图  

    ![env_1_ok](/assets/images/quick_image/env_1_ok.png)

- 安装编译需要的组件

    ```
    sudo apt-get install git-core build-essential libssl-dev libncurses5-dev unzip gawk zlib1g-dev
    ```


    ![env_2](/assets/images/quick_image/env_2.png)

    安装完成如下图  

    ![env_2_ok](/assets/images/quick_image/env_2_ok.png)

    详细参考[openwrt官网编译推荐](https://oldwiki.archive.openwrt.org/doc/howto/buildroot.exigence)

- 安装svn和mercurial

    ```
    sudo apt-get install subversion mercurial
    ```
    ![env_2_2](/assets/images/quick_image/env_2_2.png)


- 安装git，设置git账户

    ![env_3](/assets/images/quick_image/env_3.png)

    ![env_4](/assets/images/quick_image/env_4.png)

- 下载源码
  
    参考[快速入门手册](https://siflower.github.io/2020/08/05/quick_start/)获取下载链接，使用git clone下载源码

    ![env_5](/assets/images/quick_image/env_5.png)

    由于github服务器的问题，下载时间可能较长，请耐心等待下载完成
    
    下图表示下载完成

    ![env_6](/assets/images/quick_image/env_6.png)

- 编译源码 
  
    进入openwrt源码目录，第一次编译必须使用脚本编译SDK已有的项目

    - 开始编译

     ![env_7](/assets/images/quick_image/env_7.png)

    - 编译成功如下图所示，并且会在同级目录生成一个bin文件

     ![env_8](/assets/images/quick_image/env_8.png)

- vim设置
  
  使用以下指令打开vimrc文件，参考[vim配置](http://www.ruanyifeng.com/blog/2018/09/vimrc.html)进行配置

  ```
    sudo vi /etc/vim/vimrc
  ```


至此虚拟机安装、ubuntu设置以及siflower编译环境搭建结束，后续可以在此环境进行siflower平台功能修改和开发


## FAQ

**Q：在安装过程中，遇到了如下的错误，如何解决?**  

![faq_1](/assets/images/quick_image/faq_1.png)

A：在创建虚拟机向导的时候我们如下页面选择了Ubuntu，而不是Ubuntu64，下载的镜像是64位的导致的这个错误
需要重新选择再完成设置  

![faq_2](/assets/images/quick_image/faq_2.png)

![faq_3](/assets/images/quick_image/faq_3.png)

**Q：打开虚拟机时，遇到了如下的错误，如何解决?**  

![faq_4](/assets/images/quick_image/faq_4.png)

A：这个是因为电脑设置没有支持虚拟化，需要进入bios设置  
根据不同电脑主板型号，开机时进入bios,一般在Advanced、Security、BIOS Features、Configuration下面，找到 Intel Virtualization Technology，选择 Enabled，然后保存, 退出即可  






