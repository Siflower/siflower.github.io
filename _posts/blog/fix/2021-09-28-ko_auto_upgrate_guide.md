---
layout: post
title: ko自动更新流程介绍
categories: CUSTOM
description: 介绍Siflower Pubic/UI-source-code ko自动更新流程
keywords: openwrt ko更新 
mermaid: false
---

# ko自动更新流程介绍
* TOC
{:toc}

## 1 介绍
### 1.1 适用人员
使用Siflower ko文件进行Openwrt系统开发的技术人员，要求具备基础脚本和Makefile编写能力。

### 1.2 开发环境
openwrt系统编译环境，环境搭建请参考[快速入门手册](https://siflower.github.io/2020/08/05/quick_start/)

### 1.3 相关背景
随着客制化需求的增加，人工手动编译更新ko文件的速率较低，有时难以满足客户及时获取符合自己需求的ko文件的需求，因此引入了系统自动编译更新ko，以提高工作效率。

## 2 功能概述
### 2.1 代码下载
- 账号注册
    - 代码下载及分支建立需要向Siflower申请开放权限，同意开放后需要提供相关邮箱进行账号注册，注册通过后Siflower会提供对应的账号以及密码，获取账号后使用账号密码登录[gerrit网站](10.0.18.30:8080)。
- 申请权限
    - 使用账号密码登录[矽昌通信](http://10.0.4.94:9001/)，在项目列表中选择external/public/UI-source-code，权限选择read，时长选择永久，申请权限：

    ![apply.png](/assets/images/ko_update/apply.png)
- 代码下载
    - 登录[gerrit网站](10.0.18.30:8080)，在BROUSE-Repositories中选择public/UI-source-code，复制链接下载代码到本地仓库：

    ![download.png](/assets/images/ko_update/download.png)
- 切换分支
 命令为：
```
  git checkout 对应分支
```

### 2.2 引入一个新的板型或使用公用板型
板型引入请参考[新的版型引入指南](https://siflower.github.io/2020/09/08/newBoardImportGuide/) ，以下以a28_ac28s板型为例。

### 2.3 修改相关文件
- 分别修改以下文件夹当前目录下的Makefile：
    - openwrt/package/kernel/sf_smac/

    ![smac.png](/assets/images/ko_update/smac.png)
    - openwrt/package/kernel/sf_gmac/ 

    ![gmac.png](/assets/images/ko_update/gmac.png)

    - openwrt/package/kernel/sf_ewitch/ 

    ![ewitch.png](/assets/images/ko_update/ewitch.png)
    - openwrt/package/kernel/sf_hnat/ 

    ![hnat.png](/assets/images/ko_update/hnat.png)
    - openwrt/package/kernel/sf_netlink/

    ![netlink.png](/assets/images/ko_update/netlink.png)
    - openwrt/package/kernel/sfax8-factory-read/

    ![factory.png](/assets/images/ko_update/factory.png)
- 修改openwrt根目录下的ko_make_auto.cfg文件，将NEED_MAKE参数改为1（1表示需要更新ko，0表示不需要），按照格式增加ko替代位置：

![cfg.png](/assets/images/ko_update/cfg.png)

### 2.4 本地编译镜像验证
若脚本编译（./make.sh 板型参数)失败，但指令编译（make V=s)成功，请检查生成镜像是否为bin文件，若不是，修改openwrt根目录下的make.sh文件逻辑，直至脚本编译成功：

![bin.png](/assets/images/ko_update/bin.png)

### 2.5 上传改动文件
- 将改动后的所有文件提交到外网gerrit对应仓库及分支，命令为：

```
  git commit -m "RM#XXXX message"  RM#XXXX为相应redmine号
  git push origin HEAD:refs/for/对应分支
```
- 提交完成后会收到邮件abandoned通知，代表提交已从外网转入Siflower内网，大约5分钟后内网会将提交合入相应分支，并在内网触发代码自动编译，大约30分钟可完成编译与ko更新，git log可查看到相应提交，如下图所示，git pull便可以获得更新的ko：

![gitlog.png](/assets/images/ko_update/gitlog.png)
- 若更新文件中只有ko_make_auto.cfg文件没有相应ko文件或没有更新文件，表示ko编译失败，请联系Siflower对提交代码进行本地验证。

## 3 项目引用
### 3.1 参考文档
- [快速入门手册](https://siflower.github.io/2020/08/05/quick_start/)
- [新的版型引入指南](https://siflower.github.io/2020/09/08/newBoardImportGuide/)

## 4 测试用例
更新ko后编译烧录镜像，正常运行且基础功能正常即更新成功。
