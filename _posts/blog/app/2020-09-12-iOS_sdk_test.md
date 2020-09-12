---
layout: post
title:  iOS接口测试手册
categories: APP
description: 介绍iOS接口如何测试
keywords: 开放平台
mermaid: true
---

# iOS SDK接口测试手册

**目录**

* TOC
{:toc}

# 目的

本文档介绍了如何测试iOS SDK中的接口

# 测试环境

运行环境：iOS 8及以上系统

测试工具： MAC操作系统、Xcode开发软件、苹果手机

# 硬件设备

支持的固件类型：路由器

# 相关文档

[iOS SDK集成指南](https://siflower.github.io/2020/07/29/iOS_sdk/#iOS-sdk%E9%9B%86%E6%88%90%E6%8C%87%E5%8D%97)

[Siwifi接口测试手册](https://siflower.github.io/2020/09/11/SiWiFi_interface_test/#42-app%E6%8E%A5%E5%8F%A3%E6%B5%8B%E8%AF%95)

## 1 接口调用条件

SDK所有API调用的前置条件是完成**用户登录**和**绑定路由器**

## 1.1 环境检测

确认手机可以连接internet，在手机浏览器中访问url https://cloud.siflower.cn/ 如能正常弹出页面，则表示和Siflower服务器连接正常。

![手机查看cloud](/assets/images/android_api_test/web_cloud.png)

## 1.2 用户登录

按以下示例调用接口完成登录，返回code为0表示登录成功

```
//参数
//loginKey：用户标识
//appKey： Siflower appid
[manager userLogin:@"43fbf9db189a937511658743a47ce2f7" andAppKey:@"c20ad4d76fe97759aa27a0c99bff6710" onresult:^(id ret, int code, NSString *msg){
        if(code == 0){
           //登录成功
        }else{
          //登录失败
        }
    }];

```

## 1.3 绑定路由器

| API     | 测试环境    |  调用条件  | 实现功能 |
| -------- | ------- | -------------------- | -------- |
| 路由器绑定 | 设备连接路由器**局域网**,路由器能正常联网  |  路由器管理密码验证通过  |   将路由器与用户绑定   |

## 1.3.1 环境检测

将手机连上路由器WiFi,在手机浏览器中输入路由器网关地址(如：192.168.4.1)。能正常弹出路由器管理页面，则环境准备就绪。

![手机查看4.1](/assets/images/android_api_test/web_41.png)

## 1.3.2 密码验证

绑定前需要验证路由器管理员密码，调用以下接口完成验证，返回code为0表示验证成功

```
[manager checkAdminPassword:alert.textFields.lastObject.text flag:false onresult:^(id ret, int code, NSString *msg) {
                if(code == 0){
                    //管理员密码验证成功
                } else {
                   //管理员密码验证失败
                }
            }];

```

## 1.3.3 路由器绑定

调用接口完成路由器绑定，返回code为0表示绑定成功

```
[manager bindSiRouter:^(id ret, int code, NSString *msg) {
        if(code == 0){
            //绑定路由成功
        }else{
            //绑定路由失败
        }
    }];

```

## 2 接口测试

完成[上述前置步骤](#1-接口调用条件)后，可进行API测试

以WiFi接口为例：

先进行wifi设置，将2.4G和5G param对象传入NSMutableArray中,array作为参数传入setRouterWifi函数

```language
    NSLog(@"setWifiInfo======");
    //5G
    SiWiFiSetParams *wifi5 = [[SiWiFiSetParams alloc]init];
    wifi5.password = self.pswText.text;
    wifi5.ssidold = @"siwifi-7fb8-2.4G";
    wifi5.ssidnew = @"Siflower123-2.4G";
    wifi5.encryption = @"psk2+ccmp";
    wifi5.enable = self.para5.enable;
    wifi5.channel = self.para5.channel;
    wifi5.signalmode = self.para5.signal;
    //2.4G
    SiWiFiSetParams *wifi24 = [[SiWiFiSetParams alloc]init];
    wifi5.password = self.pswText.text;
    wifi5.ssidold = @"siwifi-7fbc";;//
    wifi5.ssidnew = @"Siflower123";
    wifi5.encryption = @"psk2+ccmp";
    wifi5.enable = self.para5.enable;
    wifi5.channel = self.para5.channel;
    wifi5.signalmode = self.para5.signal;

    NSMutableArray *setArray = [NSMutableArray array];
    [setArray addObject:wifi5];
    [setArray addObject:wifi24];

    [manager setRouterWifi:setArray onresult:^(id ret, int code, NSString *msg) {
        NSLog(@"setRouterWifi ret == %@ ,code == %d,msg == %@",ret,code,msg);
        if(code == 0){
            NSLog(@"set success");
            [self labelAnimate:@"设置成功"];
        }else{
            [self labelAnimate:@"设置失败"];
        }
    }];

```

再调用get接口，查看获取的信息是否符合预期

```
[manager getRouterWifi:^(id ret, int code, NSString *msg) {
        if(code == 0){
            //获取wifi信息成功操作
        }else{
            NSLog(@"getRouterWifi error");
        }
    }];

```

最后可以到路由器web上查看设置是否正确

![web查看结果](/assets/images/android_api_test/view_web_result.png)

### 3 常见错误

下面总结了一些远程接口常见的错误码以及出现的原因

| code     | msg    |  出现原因  | 解决办法 |
| -------- | ------- | -------------------- | -------- |
| 2200 | internetapi  |  手机无互联网  |   使手机正常联网   |
| 2201 | null router array  |  无绑定路由器  |   需要先完成路由器绑定   |
| 2205   | user not logon  |    用户未登录     |   需要先进行用户登录  |
| 1011 | login fail  |  路由器管理员密码错误  |   传入正确的管理员密码   |

部分错误是由路由器端返回的，可以参考[路由器接口文档中的错误码](RM7136)
