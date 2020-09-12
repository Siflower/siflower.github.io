---
layout: post
title: Android SDK接口测试手册
categories: APP
description: 介绍SDK接口如何测试
keywords: 开放平台
mermaid: true
---

# Android SDK接口测试手册

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

# 目的

本文档介绍了如何测试Android SDK中的接口

# 测试环境

软件运行环境： JDK1.8及以上、Android SDK API 15及以上、完成[Siflower Android SDK的集成](https://siflower.github.io/2020/07/29/android_sdk/)

测试工具： Android Studio开发工具、android设备(手机、平板、电视等)

# 硬件设备

支持的固件类型：路由器

# 相关文档

[Android SDK集成指南](https://siflower.github.io/2020/07/29/android_sdk/#android-sdk%E9%9B%86%E6%88%90%E6%8C%87%E5%8D%97)

[Siwifi接口测试手册](https://siflower.github.io/2020/09/11/SiWiFi_interface_test/#42-app%E6%8E%A5%E5%8F%A3%E6%B5%8B%E8%AF%95)

## 1 接口类型

SDK的接口按调用条件分为 **局域网接口** 和 **远程接口** 两种类型，需要的测试环境也有所区别

| 接口类型     | 测试环境    |  调用条件  | 实现功能 |
| -------- | ------- | -------------------- | -------- |
| 局域网接口 | 设备在路由器局域网  |  路由器管理密码验证通过  |   对路由器设置   |
| 远程接口   | 设备连接上internet  |    完成用户登录，路由器完成绑定      |   修改用户信息，对路由器设置  |

## 2 局域网接口测试

### 2.1 环境准备

将手机连上路由器WiFi,在手机浏览器中输入路由器网关地址(如：192.168.4.1)。能正常弹出路由器管理页面，则测试环境准备就绪。

![手机查看4.1](/assets/images/android_api_test/web_41.png)

### 2.2 接口初始化

局域网接口通过LocalApi类实现接口调用。

首先需要通过代码确认环境，LocalApi提供了api进行检测

```language
    //检测WiFi是否连接上路由器
    // @param
    // wifi_mac:  当前手机连接wifi的mac地址
    // router_mac:  路由器产品mac地址的前6位
    Boolean wifiConnect = LocalApi.isSiwifi(wifi_mac,router_mac)
```

```language
    //获取网关地址
    // @param
    // context： Android的Context对象
    String ip = LocalApi.getGatewayIp(context)
```

确认wifi连接正确，获取到ip地址后，创建LocalApi对象,并设置管理员密码

```languague
    //获取网关地址
    // @param
    // apiVersion: api版本号，使用default即可
    LocalApi mLocalApi = new LocalApi(LocalApi.DEFAULT_APP_API_VERSION);

    mLocalApi.setmLocalIp(ip);
    mLocalApi.setAdminPassword("admin");
```

### 2.3 接口调用

调用以下方法：

```language
    //调用局域网接口的方法
    //@paramc
    //param:接口参数
    //Ret.class 接口返回值的类型，与param成对出现
    //SingleObserver 回调函数

    mLocalApi.executeApiWithSingleResponse(param,Ret.class).subscribe(new SingleObserver<Ret>)

```

以WiFi接口为例：

先进行wifi设置，传入参数对象setWiFiDetailParam和SetWiFiDetailRet.class，在onSuccess()和onError()中分别打印log，查看接口是否成功。

```language
    LocalApi localApi = new LocalApi(LocalApi.DEFAULT_APP_API_VERSION);
        localApi.setAdminPassword("admin");
        SetWiFiDetailParam setWiFiDetailParam = new SetWiFiDetailParam(LocalApi.DEFAULT_APP_API_VERSION);
        List<WifiParam> list = new ArrayList<WifiParam>();
        //2.4G
        WifiParam wifiParam24 = new WifiParam();
        wifiParam24.enable = 1;
        wifiParam24.encryption = "psk2+ccmp";
        wifiParam24.signalmode = 0;
        wifiParam24.channel = 6;
        wifiParam24.password = "12345678";
        wifiParam24.oldssid = "siwifi-7fb8-2.4G";
        wifiParam24.newssid = "Siflower123-2.4G";
        // 5G
        WifiParam wifiParam5 = new WifiParam();
        wifiParam5.enable = 1;
        wifiParam5.encryption = "psk2+ccmp";
        wifiParam5.signalmode = 0;
        wifiParam5.channel = 161;
        wifiParam5.password = "12345678";
        wifiParam5.oldssid = "siwifi-7fbc";
        wifiParam5.newssid = "Siflower123";

        list.add(wifiParam24);
        list.add(wifiParam5);
        setWiFiDetailParam.setSetting(new Gson().toJson(list));
        localApi.executeApiWithSingleResponse(setWiFiDetailParam,SetWiFiDetailRet.class).subscribe(new SingleObserver<SetWiFiDetailRet>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onSuccess(SetWiFiDetailRet ret) {
                Log.e(TAG,"success "+new Gson().toJson(ret));
            }

            @Override
            public void onError(Throwable e) {
                Log.e(TAG,"error "+e.getMessage());
            }
        });
```

成功则在logcat中可以查看到log

![设置成功](/assets/images/android_api_test/set_wifi_success.png)

由于修改了wifi名称，**手机需要重新连接到名为Siflower-123的WiFI**

再调用获取WiFi信息接口查看与设置的参数是否符合,可将获取到的结果转成json字串打印对比。

```language
    Single<GetWiFiDetailRet> getwifi = localApi.executeApiWithSingleResponse(getWiFiDetailParam, GetWiFiDetailRet.class);
    getwifi.subscribe(new SingleObserver<GetWiFiDetailRet>() {
        @Override
        public void onSubscribe(Disposable d) {

        }

        @Override
        public void onSuccess(GetWiFiDetailRet ret) {
            Log.e(TAG,"success "+new Gson().toJson(ret));
        }

        @Override
        public void onError(Throwable e) {

        }
    });
```

logcat中查看获取到的信息

![获取成功](/assets/images/android_api_test/get_wifi_success.png)

最后可以到路由器web页面查看是否符合预期

![web查看结果](/assets/images/android_api_test/view_web_result.png)

### 2.4 绑定接口

绑定接口将用户和路由器在后端服务器绑定，使用户可以通过远程接口设置路由器。绑定接口是一个特殊的局域网接口，需要传入用户对象作为参数

```language
    BindParam param = new BindParam(LocalApi.DEFAULT_APP_API_VERSION);
    param.setUserobjectid(sfuer.getObjectId());
    mLocalApi.executeApiWithSingleResponse(param,BindRet.class).subscribe(new SingleObserver<BindRet>() {
        @Override
        public void onSubscribe(Disposable d) {

        }

        @Override
        public void onSuccess(BindRet bindRet) {

        }

        @Override
        public void onError(Throwable e) {

        }
    });

```

绑定成功，在[用户登录](#321-用户登录)后就能在用户对象中读取绑定的路由器

### 2.5 常见错误

下面总结了一些局域网接口常见的错误码以及出现的原因

| code     | msg    |  出现原因  | 解决办法 |
| -------- | ------- | -------------------- | -------- |
| 2001 | null return  |  连接的WiFi不是矽昌路由器或ip地址传入错误  |   连接到正确的WiFi并重新获取ip地址  |
| 1011   | login fail  |    路由器管理密码设置错误      |   传入正确的密码  |

部分错误是由路由器端返回的，可以参考[路由器接口文档中的错误码](RM7136)

## 3 远程接口测试

### 3.1 环境准备

确认手机可以连接internet，在手机浏览器中访问url https://cloud.siflower.cn/ 如能正常弹出页面，则表示和Siflower服务器连接正常。

![手机查看cloud](/assets/images/android_api_test/web_cloud.png)

### 3.2 接口初始化

参照SDK集成文档中的[SDK初始化](https://siflower.github.io/2020/07/29/android_sdk/#4sdk-%E5%88%9D%E5%A7%8B%E5%8C%96)部分初始化

初始化后，还需要三个前置条件：用户登录、获取绑定的路由器、建立websocket连接:

#### 3.2.1 用户登录

```language
    SFUser.loginByExtra(mainActivity, "66666", new SFObjectResponseListener<SFUser>() {
        @Override
        public void onSuccess(SFUser sfUser) {
            Log.e(TAG, "login success" + new Gson().toJson(sfUser));
            Toast.makeText(MainActivity.this,"登陆成功",Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onError(SFException ex) {
            Log.e(TAG, "login fail code "+ ex.getCode()+" msg "+ex.getMessage());
        }
    });
```

登录成功，在logcat打印的json数据应看到如下格式的sfuser对象

#### 3.2.2 获取已绑定的路由器列表

```language
    List<Routers> routers = sfUser.getBinder();
```

如果获取到的列表为空，则表示未绑定路由器，需要先进行[路由器绑定](#24-绑定接口)。

#### 3.2.3 建立websocket连接

```language
//建立websocket连接
//@param
//sfUser: user对象
SiWiFiManager.getInstance().createRemoteConnection(sfUser, new RemoteConnectionListener() {
               @Override
               public void onConnectSuccess() {
                   Log.i(TAG, "on connection success");
               }

               @Override
               public void onConnectionClose(int code, String reason) {
                   Log.i(TAG, "on connection close");
               }

               @Override
               public void onFailure(Exception ex) {
                   Log.i(TAG, "on Failure");
               }
           });
```

完成以上三步，即完成初始化

### 3.3 接口调用

以 WiFi接口为例：
    先进行wifi设置，传入参数对象router和user,wifiparam和callback，在onSuccess()和onError()中分别打印log，查看接口是否成功。

```language
    List<WifiParam> list = new ArrayList<WifiParam>();
    WifiParam wifiParam24 = new WifiParam();
    wifiParam24.enable = 1;
    wifiParam24.encryption = "psk2+ccmp";
    wifiParam24.signalmode = 0;
    wifiParam24.channel = 1;
    wifiParam24.password = "12345678";
    wifiParam24.oldssid = "siwifi-7e80-2.4G";
    wifiParam24.newssid = "Siflower123-2.4G";
    WifiParam wifiParam5 = new WifiParam();
    wifiParam24.enable = 1;
    wifiParam24.encryption = "psk2+ccmp";
    wifiParam24.signalmode = 0;
    wifiParam24.channel = 161;
    wifiParam24.password = "12345678";
    wifiParam24.oldssid = "siwifi-7fbc";
    wifiParam24.newssid = "Siflower123";
    list.add(wifiParam5);
    list.add(wifiParam24);
    SiWiFiManager.getInstance().setWiFi(routers, mUser, list, new SingleObserver<SetWiFiDetailRet>() {
        @Override
        public void onSubscribe(Disposable d) {

        }

        @Override
        public void onSuccess(SetWiFiDetailRet setWiFiDetailRet) {
            Log.e(TAG,"set wifi success"+new Gson().toJson(setWiFiDetailRet));
        }

        @Override
        public void onError(Throwable e) {
            Log.e(TAG,"set wifi fail "+e.getMessage());
        }
    });
```

成功时会在logcat中看到成功日志

![获取成功](/assets/images/android_api_test/get_wifi_success.png)

远程接口不需要局域网环境，可以直接调用获取WiFi信息接口查看与设置的参数是否符合。

```language
SiWiFiManager.getInstance().getWifiObserve(routers, sfUser, new SingleObserver<List<WiFiInfo>>() {
        @Override
        public void onSubscribe(Disposable d) {

        }

        @Override
        public void onSuccess(List<WiFiInfo> wiFiInfos) {
            Log.e(TAG,"get wifi success "+new Gson().toJson(wiFiInfos));
        }

        @Override
        public void onError(Throwable e) {
            Log.e(TAG,"get wifi error "+e.getMessage());
        }
    });
```

![获取成功](/assets/images/android_api_test/get_wifi_success.png)


最后可以到路由器web上查看设置是否符合

![web查看结果](/assets/images/android_api_test/view_web_result.png)

### 3.4 常见错误

下面总结了一些远程接口常见的错误码以及出现的原因

| code     | msg    |  出现原因  | 解决办法 |
| -------- | ------- | -------------------- | -------- |
| 19 | cant find product  |  SDK appkey、appSecret传入错误  |   传入正确的appKey、appSecret   |
| 2002 | need login manual  |  未进行用户登录直接调接口  |   需要先进行用户登录   |
| 2006   | Unable to resolve host "cloud.siflower.cn": No address associated with hostname  |    手机网络异常     |   使手机正常联网，再重新调用  |
| 4 | cant find router  |  传入的Router对象错误  |   重新登录，获取最新的Router传入   |
| 9 | cant find user  |  传入的SFUser对象错误  |   重新登录，将得到的SFUser传入   |
| 2004 | time out  |  websocket连接断开，接口超时  |   重新建立websocket连接   |

部分错误是由路由器端返回的，可以参考[路由器接口文档中的错误码](#RM7136)

