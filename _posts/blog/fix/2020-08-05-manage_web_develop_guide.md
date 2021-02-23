---
layout: post
title: 管理网页开发手册
categories: CUSTOM
description: 介绍如对siflower管理网页进行开发
keywords: 网页 开发
mermaid: false
---


# 管理网页客开发手册


* TOC
{:toc}


## 1 介绍  

### 1.1 适用人员  

熟悉网页前端开发  
熟悉lua脚本语言  
熟悉luci界面框架  
基于siflower管理网页做新功能开发的人员  

### 1.2 开发环境

ubuntu系统  
已编译完成的siflower SDK  
正常工作的siflower硬件版型
PC端浏览器

### 1.3 相关背景

管理网页开发手册为用户提供siflower设备管理页面新功能开发的指导  
并且提供基于管理网客开发demo说明让客户清楚如何在siflower管理网页进行新的功能开发  

**如果需要使用siflower管理页面，请先联系矽昌官方，获取siflower管理页面使用权限，即可按照本文说明进行开发**  

如果仅需要要对管理网页风格样式进行修改，参考[管理网页客制化手册](https://siflower.github.io/2020/07/31/manage_web_custom_guide/)  

## 2 项目引用

luci是openwrt上的web管理界面，luci采用了MVC三层架构，使用Lua脚本开发  
开发luci的配置界面不需要编辑任何的Html代码，只需要修改Model层就可以  
开发创建新的网页界面，则需要手动编写网页html代码，加入到luci代码中  

- openwrt luci中，web界面在系统/www下

    其他的界面由CGI程序生成,/www下放有index.html，仅仅用来直接跳转至/cgi-bin/luci  


- MVC模式：M(model)，V(view)，C(controller)  
  
    MVC读取配置文件的信息，然后输出到页面上，也就是MVC里面的读“数据库”，写“数据库”  
    只不在openwrt里面的“数据库”对应的是配置文件，读写方法也不太一样  

    Model（配置文件）: /etc/config/luci  
    View（页面文件）： /usr/lib/lua/luci/view/  
    Controller（控制器）：/usr/lib/lua/luci/controller/  

### 2.1 参考文档  

[openwrt的web界面开发](https://www.cnblogs.com/yizhizaiYI/articles/5194503.html)

[openwrt页面开发与luci介绍](https://www.jianshu.com/p/bfb93c4e8dc9)


## 3 开发详情  

### 3.1 功能设计

siflowe sdk网页功能开发与openwrt网页开发一致，使用标准的luci页面框架  
siflower在luci下建立了单独的文件夹，便于管理所有siflower web的开发，基于现有的管理页面做开发在此文件下进行即可

### 3.2 软件流程

本文将以在siflower管理页面-->高级设置-->设备管理  
增加一个把当前连接设备限速限时的功能为示例，对管理页面开发流程做说明  
需添加的期望功能页面如下  

![page_develop](/assets/images/web_custom_image/page_develop.png)

- controller/目录下的对应控制模块
  
  设备管理对应systemnew.lua模块，添加新的entry，并完成call调用函数实现功能   
  
    ```
    feeds/luci/moduldes/luci-mod-admin-full-siflower/luasrc/controller/admin/systemnew.lua
    ```
  
    - 添加代码

        ```
        ..
        ...
        entry({"admin", "systemnew", "syslog"}, template("new_siwifi/device_manager/device_limit") , _("device_limit"), 9);
        ...
        entry({"admin", "systemnew", "set_limit_mode_enable"}, call("set_limit_mode_enable")).leaf = true;
        entry({"admin", "systemnew", "set_limit_mode_config"}, call("set_limit_mode_config")).leaf = true;

        ..
        ...
        function get_green_mode_enable()
            local enable = deviceImpl.get_green_mode_enable()
            local result = {
                code = 0,
                msg = "OK",
                enable = enable,
            }
            luci.http.prepare_content("application/json")
            luci.http.write_json(result)
        end
    
        function set_green_mode_enable()
            local arg_list, data_len = luci.http.content()
            local arg_list_table = json.decode(arg_list)
            local enable = 0
            if arg_list_table["enable"] ~= 0 then
                enable = 1
            end
            local code = deviceImpl.set_green_mode_enable(enable)
            local result = {
                code = code,
                msg = "OK",
            }
            luci.http.prepare_content("application/json")
            luci.http.write_json(result)
        end
        ```


    controller目录代码以systemnew.lua举例说明，该目录下其他lua代码结构相同

    - module()
    
        开头调用module定义了模块入口，即会在板子系统/usr/lib/lua/luci/controller/admin/下建立一个systemnew.lua

        ```
        module("luci.controller.admin.systemnew", package.seeall)
        ```
    

    - function index()函数  
  
        定义了整个systemnew下的所有选项，如果有新增选项可以直接在此函数中按照规则添加处添加

    - entry函数  
 
        表示添加一个新的模块入口，函数原型如下  

        ```
        entry(path, target, title=nil, order=nil)
        ```


      > 参数path 是访问的路径  
        路径是按字符串数组给定的，比如“{"admin", "systemnew", "smart_audio"}”  
        那么就可以在浏览器里访问“http://192.168.4.1/cgi-bin/luci/admin/systemnew/smart_audio”来访问这个脚本   
        “admin”表示为管理员添加脚本  
        “systemnew”即为一级菜单名，即设备管理  
        “control”为菜单项名，即智能语音  
        添加好后，系统会自动在对应的菜单中生成菜单项，如增加功能页面示例图示  


      > 参数target 是调用目标  
        调用目标分为三种，分别是执行指定方法(Action)、访问指定页面(Views)以及调用CBI Module  
        第一种可以直接调用指定的函数，比如点击后恢复出厂设置，写为“call("reset")”，然后在该lua文件下编写名为reset的函数就可以调用  
        第二种可以访问指定的页面，比如写为“template(new_siwifi/device_manager/samrt_audio")”就可以调用/usr/lib/lua/luci/view/new_siwifi/device_manager/smart_audio.htm文件  
        第三种主要应用在配置界面，比如写为“cbi("firewall.lua")”就可以调用/usr/lib/lua/luci/model/cbi/firewall.lua文件


      > 参数title  
        这个参数是菜单的文本，_("smart_audio")，国际化


      > 参数order，菜单项顺序  
        这个参数是是同级菜单下，此菜单项的位置，从大到小

    - call调用的函数实现
  
        点击选项后功能实现函数，通过lua脚本语言完成实现，以set_password，设置管理员密码接口为例

        ```
        function set_password()
        --通过 luci.http.content() 获取json字符串参数
        local passwd_json = luci.http.content()
        local passwd_table = json.decode(passwd_json)
        local code = 0
        code = deviceImpl.setpasswd(passwd_table)
        sysutil.set_easy_return(code, nil)
        end
        ```

        在这个函数中又会调用feeds/luci/moduldes/luci-mod-admin-full-siflower/luasrc/siwifi/deviceImpl.lua中  
        siwifi网页接口setpasswd()来实现设置  

        ```
        function setpasswd(arg_list_table)
	    local username = "admin"
	    local oldpasswd = arg_list_table["oldpwd"]
	    local pwd = arg_list_table["newpwd"]
	    local code = 0
	    local authen_vaild = luci.sys.user.checkpasswd(username,oldpasswd)

	    if authen_vaild == false then
		    code = sferr.ERROR_NO_OLDPASSWORD_INCORRECT
	    else
		    code = luci.sys.user.setpasswd("admin", pwd)
	    end
	    sysutil.sflog("INFO","Password changed!")
	    return code
        end
        ``` 

- controller调用本地接口
  
    controller调用的siflower本地接口说明使用手册，请联系FAE获取

- 页面代码调用controller接口
  
    - 调用已有页面可以在entry函数中通过template调用view目录下已有界面

        /new_siwifi/commo_settings/   常用设置界面页面代码  
        /new_siwifi/device_manager/   设备管理界面页面代码  
        /new_siwifi/network_params/   网络参数界面页面代码  
        /new_siwifi/senior_user/      高级用户界面页面代码  
        /new_siwifi/wireless_setting/ 无线设置界面页面代码  

  
    - 新增页面需要通过编写html代码实现，需要开发人员会使用html语言开发前端界面，并将设计好的代码放置如下路径
  
        ```
        feeds/luci/moduldes/luci-mod-admin-full-siflower/luasrc/view/new_siwifi/
        ```

  
    - 网页代码中调用lua接口
  
        在html代码中通过script插入函数，实现对控制器中接口的调用，获取本地信息显示，将页面的值传入本地接口等功能   
        controller中的接口分为带参数请求和不带参数请求的接口

      - 不需要参数的接口
  
        例如在getLimitDevice()函数中可以直接通过调用XHR.get来请求对应路径的接口获取返回值  
        只需要接口路径正确，返回的结果将保存到result，返回值result.enable=1时，会在界面勾选框打钩   
        类似测试接口时  
        " get http://192.168.4.1/cgi-bin/luci/admin/systemnew/get_limit_mode_enable "  
    
        ```
        function getLimitDevice() {
            XHR.get('<%=luci.dispatcher.build_url("admin", "systemnew","get_limit_mode_enablee")%>', null,
                    function(x, result) {
                        ...
                    }
    
        ```


      - 需要传递参数调用的接口
  
        例如在setLimit()函数中，首先从页面获取将'开始时间'，'结束时间'，'限制速度'参数值，然后幅值赋值给一个变量param  
        通过 XHR.post调用接口将参数变量传到对应的lua接口  
        将限制开始时间，结束时间，限制速度的值，通过json格式传递给systemnew.lua中set_limit_mode_config接口  
        返回结果保存到result  
        类似接口测试时  
        " post {'start_time':6:00,'end_time':12:00, 'limit':50} http://192.168.4.1/cgi-bin/luci/admin/systemnew/set_limit_mode_config "   

        ```
        params = {'start_time':startTime,'end_time':endTime, 'limit':limit};
        console.log(params);
        XHR.post('<%=luci.dispatcher.build_url("admin", "systemnew","set_limit_mode_config")%>', params,
                function(x, result){
                        ...
                }

        ```

- 完整的设备限速功能demo代码路径如下，使用siflower界面的时可以查看

    页面代码

    ```
    feeds/luci/moduldes/luci-mod-admin-full-siflower/luasrc/view/new_siwifi/device_limit.htm
    ```


    Controller代码添加

    ```
    feeds/luci/moduldes/luci-mod-admin-full-siflower/luasrc/controller/admin/systemnew.lua

    function get_limit_mode_enable()
    function set_limit_mode_enable()
    function get_limit_mode_config()
    function set_limit_mode_config()
    function get_limit_mode_device()
    function remove_limit_mode_device()

    ```


    本地接口代码添加

    ```
    feeds/luci/moduldes/luci-mod-admin-full-siflower/luasrc/siwifi/deviceImpl.lua

    function get_limit_mode_enable()
    function set_limit_mode_enable()
    function get_limit_mode_config()
    function set_limit_mode_config()
    function get_limit_mode_device()
    function remove_limit_mode_device()

    ```


    **note1：为了demo的独立性，在controller中做了区别，当取消limit_device的注释后，编译测试页面才会生效**

    ![demo_flag](/assets/images/web_custom_image/demo_flag.png)  

    **note2：代码默认的是使用英文字符，界面显示中文字符需要去以下文件添加翻译，这样在网页切换为中文时会自动将对应的英文字符串进行转换**    

    ```
    feeds/luci/moduldes/luci-mod-admin-full-siflower/po/zh-cn/full.po
    ```
 
    ![page_develop_po](/assets/images/web_custom_image/page_develop_po.png)  

    

### 3.3 调试关键点

 管理页面开发新功能调试时，参考[SiWiFi接口测试手册](https://siflower.github.io/2020/09/11/SiWiFi_interface_test/)和[SiWiFi接口开发手册](https://siflower.github.io/2020/09/11/SiWiFi_interface_develop/)说明进行debug
  
## 4 测试用例

### 4.1 测试demo

 联系矽昌获取siflower页面使用权限，获取上文测试demo

### 4.2 测试流程

- siflower sdk环境 
  
- 获取使用siflower管理页面的权限
  
- 打开systemnew.lua取消"设备限速"选项entry注释
  
- 编译代码，生成镜像烧录到siflower硬件平台  
  
- 进入管理网页192.168.4.1进行新的页面功能测试  

## FAQ

**Q：如果不想用siflower管理页面，比如原生网页或者自行设计，可以开发吗？**  
 A：siflower网页接口支持openwrt原生网页，只要按照标准的luci架构开发，调用siflower提供的网页接口实现功能  

**Q：网页接口功能调用中遇到问题如何debug**  
 A： 在lua代码中应该通过多调用sysutil.sflog()通过打印日志来查看代码错误  
 执行网页功能的时候通过串口工具调用logread查看系统日志  
 相关接口调用时应该使用接口请求工具来检查接口参数是否正确，是否正确返回值  
 页面代码调试可以通过F12进入debug模式查看console返回值