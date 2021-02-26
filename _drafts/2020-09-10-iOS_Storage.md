---
layout: post
title: iOS 存储功能使用手册
categories: APP
description: 介绍如何使用iOS SDK提供的存储功能
keywords: iOS SDK开发
mermaid: true
---

# iOS 存储功能使用手册


**版权所有©上海矽昌微电子有限公司2019。保留一切权利。**
非经本公司许可，任何单位和个人不得擅自摘抄、复制本文档内容的部分或全部，并不得以任何形式传播。

**商标申明**
SiFlower、矽昌和矽昌其它商标均为上海矽昌微电子有限公司的商标，本文档提及的其它所有商标或注册商标，由各自的所有人拥有。

**注意**
您购买的产品、服务或特性应受矽昌公司商业合同和条款的约束，本文档所描述的全部或部分产品、服务或特性可能不在您的购买和使用范围内。除合同另有约定，矽昌公司对文档的内容不做任何明示或暗示的声明和保证。

**上海矽昌微电子有限公司**
地址：上海市浦东新区祖冲之路887弄84号楼408室
网址：http://www.siflower.com/
客户服务电话：021-51317015
客户服务传真：
客户服务邮箱：


**目录**

* TOC
{:toc}

# 简介

## 使用背景

本文档为接入Siflower iOS SDK的开发者提供在App中如何访问USB存储设备内容提供指导。

## 适用人员

本文适用于在Siflower路由器硬件产品上，基于iOS SDK上进行实现访问USB存储设备内容的iOS APP研发人员。

## 前提条件

在开发前，您需要:

  * 熟悉相关硬件产品的功能和使用
  
  * 硬件产品支持USB存储功能

  * 使用SiWiFi Openwrt SDK完成产品固件的调试  [快速入门](https://siflower.github.io/2020/08/05/quick_start/)
  
  * 完成 [iOS SDK集成](https://siflower.github.io/2020/08/05/iOS_SDK/)

  * 了解iOS App开发、上线、samba协议、流媒体文件播放等
 
## SDK 存储功能提供内容

通过SDK提供的存储功能API实现对矽昌路由器产品在局域网状态下读写USB存储设备中的内容，如：

  * 文件操作：文件查看、文件删除、增加文件、修改文件、文件上传、文件下载；

  * 流媒体文件：提供流媒体文件http数据流；


## 开发环境要求

开发环境：MAC操作系统+Xcode开发软件

运行环境：iOS 8及以上系统

# SDK集成

## 资源包导入

在集成存储功能SDK时需要将SiRouterApi.framework导入到项目中。使用过程中可直接将SiRouterApi.framework文件直接拖入项目中即可，同时，注意在项目的Build Phases->Embed Frameworks中添加framwork，如下图所示：
   
![set_framework](/assets/images/iOS-SDK-Storage/ios-sdk-usb-framework.png)

## SDK依赖库

SDK导入完成后需要在项目中添加相关依赖系统库:

![capabilities](/assets/images/iOS-SDK-Storage/ios-sdk-usb-lib.png)

更多关于SDK集成和使用，请参考[iOS SDK集成指南](https://siflower.github.io/2020/08/05/iOS_SDK/)

# 存储功能使用

## 存储功能接口

接口见SFSMBProvider.h中，接口包含了文件操作的增、删、查、改接口。
使用SFSMBProvider类需要引用<SiRouterApi/SFSMBProvider.h>头文件即可。

## 获取存储samba路径

在局域网状态下，samba路径为路由的网关地址，例如网关地址为192.168.4.1，那么samba路径为：smb://192.168.4.1/usb

## 初始化

初始化接口代码：

```
SFSMBProvider *provider = [SFSMBProvider sharedSmbProvider];

```

## 实现Samba认证代理方法

继承SFSMBPrividerDelegate类并重写smbRequestAuthServer方法。
这里的认证方案需要遵循路由器端设置的Samba认证方式。一般默设置为允许所有人访问，则可直接跳过认证。

接口示例：
```

- (SFSMBAuth *) smbRequestAuthServer:(NSString *)server
                               share:(NSString *)share
                           workgroup:(NSString *)workgroup
                            username:(NSString *)username
{
    //当路由器端设置了samba认证为允许访客访问时，这里直接跳过认证。
    if ([share isEqualToString:@"IPC$"] ||
        [share hasSuffix:@"$"] || [share isEqualToString:@"usb"])
    {
        return nil;
    }
    
    SFSMBAuth *auth = _cachedAuths[server.uppercaseString];
    if (auth) {
        
        NSLog(@"cached auth for %@ -> %@ (%@) %@:%@", server, share, auth.workgroup, auth.username, auth.password);
        return auth;
    }
    
    NSLog(@"ask auth for %@/%@ (%@)", server, share, workgroup);
    
    dispatch_async(dispatch_get_main_queue(), ^{
        //跳转至认证界面，并输入认证信息
        [self presentSmbAuthViewControllerForServer:server
                                              share:share
                                          workgroup:workgroup
                                           username:username];
    });
    
    return nil;
}

```

# 文件操作

在进行存储设备文件操作前，必须知道存储文件的samba路径。

## 获取存储文件列表

基于samba路径，获取该路径下的所有内容。

接口示例：

```
    [provider fetchAtPath:path
                     auth:_defaultAuth
                    block:^(id result)
     {
         if ([result isKindOfClass:[NSError class]]) {
		//error
         } else {
             if ([result isKindOfClass:[NSArray class]]) {
                 //文件列表
                 self->_items = [result copy];
             } else if ([result isKindOfClass:[SFSMBItem class]]) {
                 //单个文件
                 self->_items = @[result];
             }
            }
         }
    }];

```

## 文件夹删除

基于文件夹的samba路径，删除该路径下的所有内容。

接口示例：

```
[[SFSMBProvider sharedSmbProvider] removeFolderAtPath:item.path auth:_defaultAuth block:^(id res){
                    NSLog(@"Remove folders:%@", res);
}];
```

## 文件删除

基于文件的samba路径，删除该路径文件。

接口示例：

```
[[SFSMBProvider sharedSmbProvider] removeAtPath:item.path auth:_defaultAuth block:^(id result) {
                    
    NSLog(@"Remove completed:%@", result);
    if (result == nil) {
        NSLog(@"remove error");
    }else{
        if (![result isKindOfClass:[NSError class]]) {
            NSLog(@"remove OK");
        }else{
            NSError *error = (NSError *)result;
            NSLog(@"error is %ld",(long)error.code);
            if (error.code == 11) {
                [[SFSMBProvider sharedSmbProvider] removeFolderAtPath:item.path auth:_defaultAuth block:^(id res){
                    NSLog(@"Remove folders:%@", res);
                }
            }
        }
    }
}];
```

## 重命名

基于文件的samba路径，对该路径文件重新命名。

接口示例：

```
[[SFSMBProvider sharedSmbProvider] renameAtPath:path newPath:newPath block:^(id result){
            NSLog(@"Rename completed:%@", result);
            if (![result isKindOfClass:[NSError class]]) {
                [self reloadPath];
            }else{
                UIAlertView *alert = [[UIAlertView alloc]initWithTitle:@"Prompt" message:@"Rename_fail_prompt" delegate:self cancelButtonTitle:@"Confirm" otherButtonTitles: nil];
                [alert show];
            }
            [self.selectedArray removeAllObjects];
 }];

```

## 创建文件

基于需要创建文件的samba路径，创建该路径下的文件。

接口示例：

```
NSString *path = [_path stringByAppendingSMBPathComponent:fileName];
    SFSMBProvider *provider = [SFSMBProvider sharedSmbProvider];
    [provider createFileAtPath:path overwrite:YES auth:_defaultAuth block:^(id result){
        if ([result isKindOfClass:[SFSMBItemFile class]]) {
            [self reloadPath];
        } else {
            NSLog(@"%@", result);
        }
    }];

```

## 创建文件夹

基于需要创建文件夹的samba路径，创建该路径下的文件夹。

接口示例：

```
NSString *path = [_path stringByAppendingSMBPathComponent:folderName];
    SFSMBProvider *provider = [SFSMBProvider sharedSmbProvider];
    id result = [provider createFolderAtPath:path auth:_defaultAuth];
    if ([result isKindOfClass:[SFSMBItemTree class]]) {
        
        NSMutableArray *ma = [_items mutableCopy];
        [ma addObject:result];
        _items = [ma copy];
        
    } else {
        NSLog(@"%@", result);
    }
}];

```

## 文件复制

基于文件的samba路径，复制该路径下的文件到新的路径下。

接口示例：

```
[provider copyFromPath:item.path toPath:tmp_path overwrite:YES auth:_defaultAuth progress:nil block:^(id result){
                        if ([result isKindOfClass:[SFSMBItemFile class]]) {
                            NSLog(@"file : %@", result);
                            
                        } else {
                            NSLog(@"%@", result);
                        }
                        
 }];

```

## 文件夹复制

基于文件夹的samba路径，复制该路径下的所有文件到新的文件夹路径下。

接口示例：

```
[provider copyFolderFromPath:item.path toPath:tmp_path overwrite:YES auth:_defaultAuth progress:nil block:^(id result){
              NSLog(@"%@", result);
}];

```

## 文件上传

将本地文件的路径，复制该路径下文件到samba路径下。

接口示例：

```
[[SFSMBProvider sharedSmbProvider] copyLocalPath:path smbPath:smbPath overwrite:YES auth:_defaultAuth block:^(id result){
            NSLog(@" upload : %@",result);
            if (![result isKindOfClass:[NSError class]]) {
                [self reloadPath];
            }
}];

```

## 文件下载

将本地文件的路径，复制该路径下文件到samba路径下。

接口示例：

```
- (void) download
{
    __weak __typeof(self) weakSelf = self;
    [SFSMBItemFile readDataOfLength:1024*1024
                         block:^(id result)
     {
         FileViewController *p = weakSelf;
         if (p) {
             //下载进度
             [p updateDownloadStatus:result];
         }
     }];
}

```

更多文件操作接口，请查阅SFSMBProvider类中提供的API。

# 多媒体文件在线播放

对于USB存储的文件中类似文本、图片等文件可以使用下载完成后进行再打开，但是对于多媒体文件，例如视频、音频的文件需要打开时，SDK提供了实时播放功能，该功能设计原理是SDK提供搭建HTTP Server，将文件流通过http方式实时传输。
具体使用方法如下：

## 启动HTTP Server

接口示例：

```
-(void)startHttpServer{
    if (_httpServerStarted) {
        return;
    }
    _httpserver = [[HTTPServer alloc] init];
    [_httpserver setPort:16918];
    [_httpserver setType:@"_http._tcp."];
    //default webPath is smb://192.168.4.1/usb
    NSLog(@"Setting document root: %@", smbWebPath);
    if (smbWebPath) {
        [_httpserver setDocumentRoot:smbWebPath];
        NSLog(@"Setting document root: %@", smbWebPath);
    }else{
        NSString *webPath = @"smb://192.168.4.1/usb";//NSTemporaryDirectory();
        NSLog(@"Setting document root: %@", webPath);
        [_httpserver setDocumentRoot:webPath];
    }
    [_httpserver setConnectionClass:[HTTPConnection class]];
    [self startServer];
    
}

- (void)startServer
{
    // Start the server (and check for problems)
    NSError *error;
    if([_httpserver start:&error])
    {
        NSLog(@"Started HTTP Server on port %hu", [_httpserver listeningPort]);
        self.httpServerStarted = YES;
    }
    else
    {
        NSLog(@"Error starting HTTP Server: %@", error);
    }
}

```

## 设置HTTP Server根路径

这里的HTTP Server根路径指的是需要在哪个路径下搭建HTTP Server。假设需要在USB存储的根目录下搭建HTTP Server，那么HTTP Server的根路径就是samba根路径如（smb://192.168.4.1/usb）。

接口示例：

```
[_httpserver setDocumentRoot:webPath];

```

## 多媒体文件播放

在上述HTTP Server启动完成之后，根据可实现对samba文件转成流媒体http数据流，然后在的相关文件播放器进行实时播放。

距离说明文件路径转换：
假设需访问HTTP Server根路径下的某个文件，该文件的samba路径是smb://192.168.4.1/usb/test.txt,那么HTTP的路径就是http://192.168.4.1:16918/test.txt。

接口示例：

```
- (void)initVideoView {
    NSArray *arrayPath = [_smbFile.path pathComponents];
    NSMutableArray *components = [NSMutableArray array];
    if (arrayPath.count > 3) {
        for (int i=0; i<arrayPath.count; i++) {
            if (i>2) {
                [components addObject:arrayPath[i]];
            }
        }
    }
    NSLog(@"_smbFile.path : %@",arrayPath[1]);
    NSString *abpath = [NSString pathWithComponents:components];
    
    NSString *string = [NSString stringWithFormat:@"http://%@:16918/%@",[self getIPAddress],abpath];
    NSLog(@"URL %@",string);
    
    NSString *urlStr = [string stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]];
    urlStr = [urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    
    [self presentViewController:[[IJKVideoViewController alloc] initWithURL:[NSURL URLWithString:urlStr]] animated:YES completion:^{
        [_downloadButton setEnabled:YES];
    }];

}

```

# 测试

## (1) 路由器USB存储设备测试

将USB存储设备插入路由器之后，可在串口工具命令行下输入：

```
ls /mnt/sda1/
```

如果能正常显示USB存储设备中的内容则表示USB存储设备与路由器连接正常。
如果不显示则需要确认：
* 产品硬件是否有问题；
* 更换USB存储设备尝试;

## (2) 路由器USB存储设备Samba服务测试

可在局域网环境下，使用一台Windows系统的PC，查看能否通过samba正常访问存储设备内容。
假设路由器的网关为192.168.4.1，那么在资源管理器地址栏中输入：
```
\\192.168.4.1\usb 
```

![wintest](/assets/images/iOS-SDK-Storage/ios-sdk-usb-win-test.png)

如果正常显示USB存储设备内容，则表示samba正常。
如果不显示，或出错，需确认：
* 检查路由器USB存储设备测试结果是否正常
* 查看路由器端samba服务进程是否启动：可在串口工具命令行下输入 ps 查看是否有/usr/sbin/smbd进程
* 尝试路由器端重启samba服务：可在串口工具命令行下输入：./etc/init.d/samba restart

## (3) 手机与局域网路由器连接测试

* 在手机wifi界面中查看网关地址，如：192.168.4.1
* 手机浏览器中访问网关地址(http://192.168.4.1)，如能正常弹出页面，则表示连接正常。如连接失败，则需要确认手机是否正确连接到矽昌路由器的WiFi。
* 如无法访问，请确认手机连接的WiFi是否为访客网络（是否开启了允许访问内网资源）或租赁网络，建议切换连接到主人网络。

# FAQ

## 1 路由器USB存储设备异常处理

* 检查路由器USB存储设备测试结果
* 检查硬件USB模块是否正常工作
* 检查路由器软件[usb驱动配置选项](https://siflower.github.io/2020/09/03/usb_driver/#4-usb%E9%A9%B1%E5%8A%A8%E9%85%8D%E7%BD%AE%E9%80%89%E9%A1%B9)


## 2 局域网下路由器USB存储设备Samba访问失败

* 检查路由器USB存储设备Samba服务测试结果是否正常

