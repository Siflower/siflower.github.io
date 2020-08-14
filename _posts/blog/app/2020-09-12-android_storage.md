---
layout: post
title: Android存储SDK集成指南
categories: APP
description: 介绍存储SDK使用
keywords: 开放平台
mermaid: true
---

# Android存储SDK集成指南

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

    本文档为接入Siflower Android存储SDK的开发者提供开发存储功能的指导

# 前提条件

在App开发前，您需要:

* 熟悉相关硬件产品的功能和使用

* 使用SiWiFi Openwrt SDK完成产品固件的调试，且固件中包含USB模块，具体可参考 [USB驱动开发手册中的自动挂载配置](https://siflower.github.io/2020/09/03/usb_driver/#42-usb-%E8%87%AA%E5%8A%A8%E6%8C%82%E8%BD%BD%E9%85%8D%E7%BD%AE)

* 了解Android开发、上线等一系列流程

# 开发环境

JAVA版本： JDK1.8及以上

Android SDK版本： API 15及以上

Android 开发工具： Android Studio

# 硬件设备

支持的固件类型：支持USB的路由器版型

# 功能概述

Android处于 **路由器局域网网络** 中时，通过samba协议，访问路由器存储设备上的数据

# Demo程序

[Android SDK demo 程序](https://github.com/siflower-company/SiRouterSDK-Demo-Android)

## demo 架构介绍
  
demo主函数中，StorageManager的使用示例包含了存储功能的各种操作

[MainActivity](https://github.com/siflower-company/SiRouterSDK-Demo-Android/blob/master/app/src/main/java/sirouter/sdk/siflower/com/sirouterapi/MainActivity.java)

* localGetFile()、localRenameFile()、localPasteFile()等方法包含了路由器的文件查看、创建、删除、复制粘贴等操作
* downloadSMBFile()、uploadSMBFile()等包含设备与路由器之间的文件上传下载等操作

## 1 SDK下载

[点击下载Android SDK](https://cloud.siflower.com.cn)

包含一个.aar文件：locallibrary-release.aar

## 2 Android Studio引入SDK

将.aar文件放入app的libs目录下

![figure1](/assets/images/android_storage_sdk/storage_lib.png)

在app的build.gradle中添加

```language
    implementation(name:'storagelibrary-release', ext:'aar')
```

其他引用支持

```language
    implementation 'commons-io:commons-io:2.5'
    implementation "io.reactivex.rxjava2:rxjava:2.2.4"
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
```

## 3 AndroidManifest.xml 设置

在项目的AndroidManifest中添加权限支持

```language
 <uses-permission android:name="android.permission.INTERNET" />
```

## 4 SDK 初始化

存储功能的主要类为StorageManager，调用接口前需要创建该类的实例

传入的参数为路由器的ip网关地址

```language
String routerIp = "192.168.4.1";
storageManager = new StorageManager(routerIp);
```

### 4.1 检测连接状态

存储功能需要在局域网中使用，sdk提供了检测连接状态的接口,连接正常则在onConnected中返回

```language
storageManager.check_local_network(new FileStateListener() {
        @Override
        public void onConnected() {
            Log.e(TAG,"connect success");
        }

        @Override
        public void onError(SFStorageException e) {
            Log.e(TAG,"connect error "+e.getMessage);
        }
    });
```

## 5 存储操作

存储的操作是通过文件目录的路径进行的，根目录为"/USB/",所有文件都在该目录下，可以通过StorageManager.baseuri获得根目录对象

### 5.1 获取文件列表

获取文件列表

参数| 说明| 示例
:-|:-|:-
uri| 目标文件夹路径(注意要以"/"结尾) | "/USB/my_picture/"
FileListListener|回调函数| 成功则获取到文件夹下的文件列表

示例代码

```language
   storageManager.getSMBFileList(StorageManager.baseUri, new FileListListener<SmbFile>() {
               @Override
               public void onGetFile(List<SmbFile> sfSmbFile) {
                   Log.e(TAG," sfs file size "+sfSmbFile.size());
               }

               @Override
               public void onError(SFStorageException e) {
                   Log.e(TAG," sfs file size error "+e.getMessage());
               }
   });
```

### 5.2 获取单个文件

获取单个文件

参数| 说明| 示例
:-|:-|:-
uri| 目标文件的完整路径 | "/USB/test1.txt"
SingleObserver|回调函数| 返回文件对象

示例代码

```language
   storageManager.getSMBFileDetail(StorageManager.baseUri + "test1.txt", new SingleObserver<SmbFile>() {
        @Override
        public void onSubscribe(Disposable d) {

        }

        @Override
        public void onSuccess(SmbFile smbFile) {
            Log.e(TAG,"on success "+new Gson().toJson(smbFile));
        }

        @Override
        public void onError(Throwable e) {

        }
    });
```

### 5.3 修改文件名称

修改文件名称

参数| 说明| 示例
:-|:-|:-
uri| 目标文件夹路径(注意要以"/"结尾) | "/USB/my_file/"
oldName| uri下要修改的文件名 | "weeklyreport.xls"
oldName| 修改后的名称 | "weeklyreport111.xls"
SingleObserver| 回调函数 | 成功则在success中返回修改后的文件名

```language
   storageManager.renameSmbFile(StorageManager.baseUri, "weeklyreport.xls", "weeklyreport111.xls", new SingleObserver<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onSuccess(String s) {

            }

            @Override
            public void onError(Throwable e) {

            }
        });
```

### 5.4 文件复制粘贴

文件复制粘贴，支持同一目录下的批量操作

参数| 说明| 示例
:-|:-|:-
fileNames| 复制的文件名列表 | \[ "test1.txt","test2.txt" \]
current| 文件目录 | "/USB/my_test1/"
dest| 目标文件目录 | "/USB/my_test2/"
FileListListener|回调函数| 成功返回粘贴的文件列表

```language
   List<String> names = new ArrayList<>();
   names.add("test1.txt");
   names.add("test2.txt");
   storageManager.copySmbFile(names, StorageManager.baseUri+"my_test1/", StorageManager.baseUri + "my_test2/", new FileListListener<SmbFile>() {
              @Override
              public void onGetFile(List<SmbFile> sfSmbFile) {
                  Log.e(TAG,"success ");
              }

              @Override
              public void onError(SFStorageException e) {
                   Log.e(TAG,"copy error "+e.getMessage());
              }
       });
```

### 5.5 文件删除

文件删除，支持同一目录下的批量操作

参数| 说明| 示例
:-|:-|:-
current| 文件目录 | "/USB/my_test1/"
fileNames| 删除的文件名列表 | \[ "test1","test2" \]
FileListListener|回调函数| 成功返回删除的文件列表

```language
   List<String> names = new ArrayList<>();
   names.add("weeklyreport-copy.xls");
   storageManager.deleteSmbFile(StorageManager.baseUri, names, new FileListListener<String>() {
        @Override
        public void onGetFile(List<String> sfSmbFile) {
            Log.e(TAG,"delete success "+sfSmbFile.size());
        }

        @Override
        public void onError(SFStorageException e) {
            Log.e(TAG,"delete error "+e.getMessage());
        }
    });
```

## 6 文件上传下载

### 6.1 权限申请

在 AndroidManifest.xml文件中添加

```language
 <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
 <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

Android 6.0以上的系统需要在调用函数的Activity中动态申请权限，参考代码如下

```language
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            List<String> permissions = new ArrayList<>();
            if (ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) == PackageManager.PERMISSION_DENIED) {
                permissions.add(Manifest.permission.READ_EXTERNAL_STORAGE);
            }
            if (ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) == PackageManager.PERMISSION_DENIED) {
                permissions.add(Manifest.permission.WRITE_EXTERNAL_STORAGE);
            }

            if (!permissions.isEmpty()){
                ActivityCompat.requestPermissions(this,permissions.toArray(new String[permissions.size()]),1);
            }
    }
```

### 6.2 上传

将手机中的文件上传至路由器。 调用函数: storageManager.uploadSmbFile()

参数| 说明| 示例
:-|:-|:-
file| 手机本地的文件对象 | "/storage/emulated/0/siwifi/download/weekly1234.xls"
uri| 目标路由器目录 | "/USB/my_files/"]
FileListListener|回调函数| 成功返回删除的文件列表

```language
    File root = Environment.getExternalStorageDirectory();
    File directory = new File(root.getAbsolutePath() + "/siwifi/download");
    storageManager.uploadSmbFile(new File(directory, "weekly1234.xls"), StorageManager.baseUri+"my_files/, new FileTransListener() {
          @Override
          public void onProgress(Integer integer) {

          }

          @Override
          public void onError(SFStorageException e) {

          }
    });
```

### 6.3 下载

将路由器中的文件下载至手机。调用函数：storageManager.downloadSmbFile()

参数| 说明| 示例
:-|:-|:-
file| 手机本地的文件对象 | "/storage/emulated/0/siwifi/download/weekly1234.xls"
uri| 目标路由器目录 | "/USB/my_files/"]
FileListListener|回调函数| 成功返回删除的文件列表

```language
    File root = Environment.getExternalStorageDirectory();
    File directory = new File(root.getAbsolutePath() + "/siwifi/download");
    storageManager.downloadSmbFile(StorageManager.baseUri + "weeklyreport.xls", new File(directory, "weekly55464"), new FileTransListener() {
        @Override
        public void onProgress(Integer integer) {

        }

        @Override
        public void onError(SFStorageException e) {

        }
    });
```

## 7 测试

### 7.1 路由器环境检验

将存储设备接入路由器的USB口，使电脑连接到路由器局域网，在windows资源管理器中输入 "\\\\192.168.4.1\USB\" 能正常显示存储设备中的文件，则说明存储环境正常。

如果无法正常显示，请联系openwrt开发人员解决

![windows查看](/assets/images/android_storage_sdk/windows_test_smb_usb.png)

### 7.2 接口测试

以复制粘贴接口为例:

调用复制粘贴接口

```language
   List<String> names = new ArrayList<>();
   names.add("test.txt");
   storageManager.copySmbFile(names, StorageManager.baseUri+"test1/", StorageManager.baseUri + "test2/", new FileListListener<SmbFile>() {
              @Override
              public void onGetFile(List<SmbFile> sfSmbFile) {
                  Log.e(TAG,"success ");
              }

              @Override
              public void onError(SFStorageException e) {
                   Log.e(TAG,"copy error "+e.getMessage());
              }
```

查看文件是否粘贴到目标地址

![windows查看粘贴](/assets/images/android_storage_sdk/storage_copy_test2.png)

### 8 常见错误

| code     | msg    |  出现原因  | 解决办法 |
| -------- | ------- | -------------------- | -------- |
| 101 | fail to resolve   |  连接的WiFi不是矽昌路由器或ip地址传入错误  |   连接到正确的WiFi并重新获取ip地址  |
| 1011   | login fail  |    路由器管理密码设置错误      |   传入正确的密码  |