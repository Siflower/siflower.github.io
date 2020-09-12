---
layout: post
title: 管理网页客制化手册
categories: CUSTOM
description: 介绍如何客制化修改管理网页
keywords: 网页 客制化
mermaid: false
---


# 管理网页客制化手册

* TOC
{:toc}

## 1 介绍

### 1.1 适用人员  

适用于利用siflower SDK针对管理网页做客制化修改的人员  

### 1.2 开发环境

ubuntu系统
已编译通过的siflower SDK环境  
正常运行的siflower 硬件平台  
PC网页浏览器

### 1.3 相关背景

管理网页客制化为用户提供可以自定义修改各类siflower设备管理页面风格的功能，并且提供管理网客制化demo指导客户在siflower管理网页面基础上做修改    

### 1.4 功能概述

此文档描述了修改siflower管理网页内部风格的功能，比如背景颜色、字体颜色和所有用到的图标的方法，便于用户量身打造一款属于自己的管理网页 
并且提供相关网页开发demo示例  
如果想要对网页功能做开发，参考[管理网页开发手册](待添加)  

## 2 项目引用

### 2.1 内部相关   

参考openwrt1806 luci界面移植[redmine #6263](http://redmine.siflower.cn/redmine/issues/6263) 
参考siflower网页客制化代码流程[redmine #7194](http://redmine.siflower.cn/redmine/issues/7194)  

## 开发详情  

### 3.1 功能设计

管理网页客制化修改主要是页面使用图标修改，以及整体或者部分背景颜色修改字体格式修改，  
为了便于用户修改，图片目前统一存储在一个文件夹，风格颜色字体等由一个配置文件来进行修改，具体说明如下  

#### 3.1.1 图片修改  

目前支持客制化的图标包含页面刷新加载图标（coverLoading.gif）、保存加载图标（saveLoading.gif）、企业logo图标（logo.png）以及其他页面中用到的小图标(sibasic.png)  
管理页面使用的所有的图片文件都存放在image文件夹中，路径如下  

```
feeds/luci/themes/luci-theme-material-siflower/htdocs/luci-static/material/image/
```


- **页面刷新加载图标**
    图片格式是gif的小动画，分辨率是80*80，位深度是8。默认样式如下图所示：  
    ![coverLoading.gif](/assets/images/web_custom_image/coverLoading.png)  
    该图标的使用位置，可以登录管理页面查看。如果想对该图片做更新，要求新设计的图片大小、分辨率、位深度必须和该图片保持完全一致，重新设计的图片命名也必须和该图片完全一致

- **保存加载图标**
    图片格式是gif 的小动画，分辨率是16*16，位深度是8。默认样式如下图所示：  
    ![saveLoading.gif](/assets/images/web_custom_image/saveLoading.png)  
    该图标的使用位置，可以登录管理页面查看。如果想对该图片做更新，要求新设计的图片大小、分辨率、位深度必须和该图片保持完全一致，重新设计的图片命名也必须和该图片完全一致

- **企业logo图标**
    如果想在管理页面替换为用户的logo，替换此图片即可    
    图片格式是png ，分辨率是124*15，位深度是32。 默认样式如下图所示：  
    ![logo.png](/assets/images/web_custom_image/logo.png)  
    **注：默认是底色是透明色，字样为白色，由于在页面中看不清，把底色涂成黑色了**

- **页面中其它小图标**
    为了节省存储空间，放到一个png 格式的图片文件中，图片格式是png， 分辨率是740*368，位深度是32。默认种类、样式和排布如下图所示：  
    ![sibasic.png](/assets/images/web_custom_image/sibasic.png)  
    关于这些图标的使用位置，可以打开管理网页进行查看。如果想要更新其中部分或全部的图标，需要在上面图片的基础上修改，新替换的图标在上述图片中的大小和位置，不然会显示异常

#### 3.1.2 字体背景颜色修改

- 目前客制化页面管理支持字体和背景颜色的修改，字体和背景颜色的修改在配置文件style中修改
  
  在源码中文件路径

    ```
    target/linux/siflower/base-files/etc/config/style
    ```


    > 所有的字体和背景颜色的修改，都是通过修改这个配置文件style 来进行的。该配置文件通过编译工具打开，里面包含两部分:  
    > 一部分是config setting font，修改字体颜色  
    > 一部分是config setting background，修改背景颜色的

- style配置文件说明
  
    **字体颜色修改说明config setting font**  

    可以设置所有字体的颜色，内有注释，‘#’开头的均为注释。可以使用#ffffff代表白色，也可以使用#fff表示，或者用white、rgb（255,255,255）、rgba（255,255,255,1）等，具体规则与CSS中相同    
    默认配置如下：  

    ```
    * text color
        config setting 'font' 

    * all text base // base为最基础的颜色，它几乎对所有字体生效，但会被覆盖，优先级最低 
        option base 'black'

    * top nav text //  nav代表导航栏，有顶部导航栏和左侧导航栏两个，header是顶部导航栏
        option header 'white'

    * bottom device info and technical support // foot是底部的软件版本和技术支持热线的位置
        option foot '#999'

    * title // title是高级设置中正文部分的标题
        option title '#df0007'

    * all button  // button表示所有按钮，disabled表示无法使用的按钮
        option button 'white'
        option input_disabled 'gray'

    * help title  // help表示页面中的帮助窗口，有标题和正文
        option help_and_wds 'white'

    * help text  
        option help_text 'black'

    * table top toolbar  // toolbar表示在表格上的工具栏，一般有删除、添加等
        option ToolBar '#8c000a'

    * note tip notice  //  notice表示一些需要注意的文字
        option note_tip '#fb6e52

    * common setting // 表示常用设置和设置向导页面中，红色背景的文字，比如表头，错误提示
    * for red background , use white text
        option com_base 'white'

    * for dark background , use white text //com_ left 表示常用设置中 左侧暗色背景中的文字
        option com_left 'white'
    ```


    **背景颜色修改说明config setting background**

    可以设置所有的背景色，并指出渐进色，内有注释，可以使用各种方法表示颜色，同font  
    默认配置文件内容如下：

    ```
    * background color 
        config setting 'background'

    * bottom background // html_and_body 设置页面最基本的背景色，仅对高级设置起作用
        option html_and_body 'white'

    * for all select and input // select_input 设置所有选择框和输入框的背景色
        option select_input '#efefef'

    * for all disabled input // input_disable 设置无法使用的输入框的背景色
        option input_disabled '#efefef'

    * for all button // button表示所有按钮，disabled表示无法使用的按钮
        option button '#df0007'
        option button_disabled 'darkgray'

    * top nav background // header是顶部导航栏颜色，此处代表使用渐变色
        option header 'linear-gradient(red,#9b1b1b)'

    * at bottom background, technical support // 是底部的软件版本和技术支持热线的位置
        option foot '#dfdfdf'

    * left nav background // nav设置高级设置中左侧导航栏的背景色
        option left_nav '#353535'

    * help and wds div top background // help表示页面中的帮助窗口，有标题和正文
        option help_and_wds_Top '#df0007'

    * help inner background
        option Help 'white'

    * common setting // bigBg 是常用设置和设置向导的背景色
  
    * biggest background
        option bigBg '#353535'

    * right background at common setting // rightBg 是常用设置中右边主体的背景色
        option rightBg 'white'

    * for red background , use white text // com_base 表示常用设置和设置向导页面中，红色背景的文字，比如表头，错误提示
        option com_base '#df0007'

    * login table background // login_table 表示登录页面中的白色背景
        option loginTable 'white'
    ```


    **其它页面信息**  
 
    ```
    * website and technical support set
        config setting 'text'

    * website  // 页面底部公司官网信息
        option website 'www.siflower.com.cn'
        option website_address 'http://www.siflower.com.cn'

    * technical support translate  // 页面底部电话
        option zh_technical_support '技术支持热线 +021 5131-7015'
        option en_technical_support 'Technical support hotline +021 5131-7015'
    ```


- 示例一：把顶部导航栏字体改成黑色
  
  ```
    * top nav text 
    option header 'black'
  ```


  效果如下  

  ![change_1.png](/assets/images/web_custom_image/change_1.png) 


- 示例二：修改顶部导航栏背景色为蓝色渐变
  
  ```
    * top nav background 
    option header 'linear-gradient(#00aeff,#017ac0)'
  ```


  效果如下   

  ![change_2.png](/assets/images/web_custom_image/change_2.png)  


### 3.2 关键点

- style配置文件位于target/linux/siflower/base-files/etc/config/style，此处是公共版型使用的style

- 可以单独放到特定版型的base-files-AF19A28-版型/etc/config中，这样修改style后编译，效果就只对特定版型生效

- 图片大小和类型一定要与文章图片修改说明一致，否则替换会有显示问题


## 4 测试demo

修改界面风格为蓝色，更换管理页面所有图标，更换log  
按照如下方式进行修改

- 修改chaos_calmer_15_05_1/target/linux/siflower/base-files/etc/config/style如下
    
    ```
    * text color
    config setting 'font'

    * all text base
        option base 'black'

    * top nav text
        option header 'white'

    * bottom device info and technical support
        option foot '#999'

    * title
        option title '#00aeff'

    * all button
        option button 'white'
        option input_disabled 'gray'

    * help title
        option help_and_wds 'white'
    * help text
        option help_text 'black'

    * table top toolbar
        option ToolBar '#00aeff'

    * note tip notice
        option note_tip '#00aeff'

    * common setting
    * for red background , use white text
        option com_base 'white'

    * for dark background , use white text
        option com_left 'white'

    * background color
    config setting 'background'

    * bottom background
        option html_and_body 'white'

    * for all select and input
        option select_input '#efefef'

    * for all disabled input
        option input_disabled '#efefef'

    * for all button
        option button '#00aeff'
        option button_disabled 'darkgray'

    * top nav background
        option header 'linear-gradient(#00aeff,#017ac0)'

    * at bottom background, technical support
        option foot '#dfdfdf'

    * left nav background
        option left_nav '#353535'

    * help and wds div top background
        option help_and_wds_Top '00aeff'

    * help inner background
        option Help 'white'

    * common setting
    * biggest background
        option bigBg '#353535'

    * right background at common setting
        option rightBg 'white'

    * for red background , use white text
        option com_base '#00aeff'

    * login table background
        option loginTable 'white'

    * website and technical support set
        config setting 'text'

    * website
        option website 'www.amediatech.cn'
        option website_address 'http://www.amediatech.cn'

    * technical support translate
        option zh_technical_support '技术支持热线 +0755 28106665'
        option en_technical_support 'Technical support hotline +0755 28106665'
     ```


- 替换图片  
    1)替换sibasic.png  
    ![sibasic_test](/assets/images/web_custom_image/sibasic_test.png)  

    2)替换logo.png  
     选择你自己的logo图标进行替换

    3)替换coverLoading.gif  

    ![coverLoading_test](/assets/images/web_custom_image/coverLoading_test.png)  

## 5 测试用例 

### 5.1 测试环境配置

#### 5.1.1 测试方法

- 在正式编译镜像之前，可以在板子系统上做临时修改进行测试    
- Image 和style 内容修改完成后，可以先通过SSH 登录设备后台，替换管理页面风格，查看是否是预期想要的效果  
- 在打开管理页面后，通过SSH 登录，然后可以使用SCP工具将修改后的image 文件夹和style 文件传到设备的相应位置，以替换原文件，然后ctrl+F5刷新网页来查看效果。
- 文件位置如下：


> 图片路径：/www/luci-static/material/images/  
> 配置文件路径：/etc/config/style  


- 替换方法参考[快速入门]--winSCP工具使用
  
- 查看效果
  电脑切换到之前登录的路由器管理页面，点击Ctrl+F5，查看修改后的路由器管理页面修改后的效果

### 5.2 测试流程和测试结果  

- 进入siflower SDK
  
- 按照测试demo进行修改
  
- 保存编译，烧录镜像到板子，登陆管理网页  
  
- 查看结果  
  
    ![ret_1](/assets/images/web_custom_image/ret_1.png) 

    ![ret_2](/assets/images/web_custom_image/ret_2.png)    


## FAQ

**Q：路由器网页登陆界面的二维码如何替换或者取消？**  
 A：1) 如果客户自己开发了对应的app需要替换二维码，可以将二维码图片放在feeds/luci/theme/luci-theme-material-siflower/htdocs/luci-static/material/images/路径下，名称默认保存为QRcode.png  
    2) 如果需要取消此二维码显示，修改feeds/luci/modules/luci-mod-admin-full-siflower/luasrc/view/sysauth.htm 
第90行，有个img标签和p标签，去掉包含这俩的tr标签就行，或者加个display:none

    ```
    <tr style="display:none"><td colspan="3">
	    <div style="margin:auto;">
		    <img src="<%=media%>/images/QRcode.png" style="width:18em;margin:auto;margin-top:8em;"></img>
		    <p class="loginTip" style="margin:auto;margin-top:0px;"><%:scan me to download the app%><br><%:experience more features%></p>
	    </div>
    </td></tr
    ```


**Q：已经对页面进行修改，重新烧录镜像后再进入管理网页，网页无变化？**  
 A：1)首先确认修改有没有保存，且新编译的镜像烧录镜像是正确  
    2)如果上述确认无误，管理页面无变化，那应该是浏览器没有清除缓存，导致还是现实原有的页面，请刷新或者清除浏览器缓存再登陆路由器管理页面  

**Q：想要对页面做更多的修改，比如导航栏大小尺寸，菜单栏显示尺寸，该如何修改？**  
A：本文提供的修改方法仅限于不太熟悉前端开发的人员，可以通过以上方法做简单修改  
如果想要对页面做更多修改，需要熟悉前端开发的人员来对如routerStyle.css等htdocs目录下的样式控制文件进行修改  
代码路径feeds/luci/theme/luci-theme-material-siflower/htdocs/luci-static/material/