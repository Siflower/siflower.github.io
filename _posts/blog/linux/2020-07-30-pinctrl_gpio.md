---
layout: post
title: Pinctrl 和 GPIO 使用手册
categories: LINUX
description: Pinctrl 和 GPIO 使用手册
keywords: 文档开发
mermaid: true
---
# Pinctrl 和 GPIO 使用手册

**目录**

* TOC
{:toc}

# 1. 介绍

Siflower的芯片提供一些PAD脚，这些PAD脚可以配置为不同的功能,本文就此做详细的介绍

## 1.1. 适用人员
开发人员应该具备以下技能：
- 熟悉C语言
- 熟悉linux Device Tree的使用
- 熟悉Linux Device Driver开发

## 1.2. 开发环境
- 可以正常编译通过的Siflower SDK环境
  该环境的搭建请参考[快速入门](https://siflower.github.io/2020/08/05/quick_start)

# 2. Pinctrl和GPIO配置简介
Siflower的芯片提供一些PAD脚, 在PAD模式下, 这些PAD脚处于默认状态, 用户必须要配置才能正常使用. 这些PAD脚可以配置成不同的模式,从而实现不同的功能. 以下表中的两个PAD脚为例

| Pin Name        | Default Mode  | GPIO\_MODE | FUNC\_MODE0                       | FUNC\_MODE1 | FUNC\_MODE2 | FUNC\_MODE3        |
|-----------------|---------------|------------|-----------------------------------|-------------|-------------|--------------------|
| I2C\_DAT        | FUNC\_MODE0   | GPIO11     | UART0\_CTS                        | UART2\_RXD  | I2C0\_DAT   | NC                 |
| I2C\_CLK        | FUNC\_MODE0   | GPIO12     | UART0\_RTS                        | UART2\_TXD  | I2C0\_CLK   | NC                 |

*note: 表中的内容节选自Siflower的IOMUX表格, 查看完整表格可以查看[Siflower IOMUX Table](https://siflower.github.io/2020/09/03/iomux_table)*

从上表可以看出, I2C_DAT(GPIO11)和I2C_CLK(GPIO12), 在配置成不同的模式的时候会有不同的功能,总共有GPIO_MODE, FUNC_MODE0,FUNC_MODE1,FUNC_MODE2,FUNC_MODE3五种模式可选,上面的两个PAD脚配置为不同的模式的时候,各自的功能见下表

| 配置的模式| 功能
| :-|:-|
| GPIO MODE| GPIO功能, 可以配置为INPUT和OUTPUT |
| FUNC_MODE0|配置为UART0的CTS和RTS管脚使用 |
| FUNC_MODE1|配置为UART2的RXD和TXD管脚使用 |
| FUNC_MODE2|配置为I2C的DAT和CLK管脚使用  |
| FUNC_MODE3|NC (Don't Care)  |

Siflower提供的PAD脚不仅有着普通的GPIO的功能,还可以由Siflower的SOC内部提供的Pinctrl(Pin脚控制器)配置成不同的模块的引脚,,例如spi,i2c,uart等   

Siflower提供Pinctrl的Linux Driver, 该Driver符合Linux标准的Pinctrl子系统的要求. 

**Note:**   
在linux提供的标准Pinctrl子系统中, Pinctrl子系统实际上也把GPIO一起管理起来，所有的GPIO操作也需要透过pinctrl子系统来完成，如果一个Pin已经被申请为GPIO，再通过Pinctrl子系统申请为其某个Function时就会返回错误
# 3. Pinctrl和GPIO使用说明

大部分时候, 我们只需要会配置linux的dts中与Pinctrl和GPIO相关的内容即可满足需求, 如果本章的内容没有解决问题, 可以查阅后续章节, 通过理解linux的驱动的实现细节, 从而解决问题. 下面就Pinctrl和GPIO的dts相关配置做详细描述.
## 3.1. DTS中配置Pinctrl
### 3.1.1. Pinctrl的配置　
Pinctrl驱动的主要工作就是配置GPIO的模式,从而实现不同的功能, 以下内容摘取自SF19A2890 FULLMASK芯片的DTS, 仅用作演示.
```
代码参考:linux-4.14.90-dev/linux-4.14.90/arch/mips/boot/dts/siflower/sf19a28_fullmask.dtsi

pinctrl: pinctrl {
    compatible = "siflower,sfax8-pinctrl";
    #address-cells = <1>;
    #size-cells = <1>;
    interrupt-parent = <&gic>;
    ranges;
    pad-base = <&grfgpio>;

    uart0 {
        uart0_txd: uart0-txd {
            sfax8,pins = <0  9 0 &pcfg_pull_pin_default>;
        };
        uart0_rxd: uart0-rxd {
            sfax8,pins = <0 10 0 &pcfg_pull_pin_default>;
        };
        uart0_cts: uart0-cts {
            sfax8,pins = <0 11 0 &pcfg_pull_pin_default>;
        };
        uart0_rts: uart0-rts {
            sfax8,pins = <0 12 0 &pcfg_pull_pin_default>;
        };
    };
    uart2 {
        uart2_rxd: uart2-rxd {
            sfax8,pins = <0 11 1 &pcfg_pull_pin_default>;
        };
        uart2_txd: uart2-txd {
            sfax8,pins = <0 12 1 &pcfg_pull_pin_default>;
        };
    };
    i2c0 {
        i2c0_dat: i2c0-dat {
            sfax8,pins = <0 11 2 &pcfg_pull_pin_default>;
        };
        i2c0_clk: i2c0-clk {
            sfax8,pins = <0 12 2 &pcfg_pull_pin_default>;
        };
    };
};
```
其中对PAD的配置体现如下形式:

```
sfax8,pins = <0  9 0 &pcfg_pull_pin_default>
```

sfax8, pins候的四个参数从左到右分别代表的意义如下:

|   0  | 9    | 0    | &pcfg_pull_pin_default|
| :--: | :--: | :--: | :--: | 
| group(固定为0)  | pin脚号（即GPIO号） | func（function参数）| 上下拉参数 |
  
**function参数**

| 参数 | 模式 |
| -- | -- |
| 0 | FUNC_MODE0 | 
| 1 | FUNC_MODE1 |
| 2 | FUNC_MODE2 |
| 3 | FUNC_MODE3 |
| 4 | GPIO INPUT |
| 5 | GPIO OUTPUT |

**上下拉参数**

| 参数 | 意义 |
| -- |-- |
| pcfg_pull_up | bias-pull-up |
| pcfg_pull_down | bias-pull-down |
| pcfg_pull_none | bias-disable |
| pcfg_pull_pin_default | bias-pull-pin-default |

我们使用pcfg_pull_pin_default即可, 也就是说上下拉参数使用系统默认的(上拉)

### 3.1.2. Pinctrl的使用
pinctrl在配置好了之后,还需要在对应的模块中声明,才会生效. 以下内容摘取自SF19A2890 FULLMASK芯片的DTS, 仅用作演示.
```
代码参考:linux-4.14.90-dev/linux-4.14.90/arch/mips/boot/dts/siflower/sf19a28_fullmask.dtsi

    i2c0: i2c@18100000 {
        compatible = "siflower,sfax8-i2c";
        reg = <0x18100000 0x1000>;
        clocks = <&pbusclk 0>;
        clock-frequency = <400000>;
        interrupts = <GIC_SHARED 217 IRQ_TYPE_LEVEL_HIGH>;
        #address-cells = <1>;
        #size-cells = <0>;
        pinctrl-names="default";
        pinctrl-0 = <&i2c0_clk &i2c0_dat>;
        status = "disabled";
    }; 

        uart2: serial@18302000 {
        compatible = "siflower,sfax8-uart";
        reg = <0x18302000 0x1000>;
        interrupts = <GIC_SHARED 228 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&uartclk 0>;
        dmas = <&gdma 14
        &gdma 15>;
        dma-names = "tx", "rx";
        pinctrl-names="default";
        pinctrl-0 = <&uart2_rxd &uart2_txd>;
        status = "disabled";
    };
```
如上内容所示, 在不同的模块内,我们通过pinctrl-names和pinctrl-0的引入,让pinctrl的配置生效   
> pinctrl-names="default"    

pinctrl-names建议就使用默认值"defalut"

> pinctrl-0 = <&i2c0_clk &i2c0_dat>   
> pinctrl-0 = <&uart2_rxd &uart2_txd>

pinctrl-0后面的内容就是pinctl配置处,各个Pin脚所使用的名字

### 3.1.3. Pinctrl的menuconfig
应该在Linux的kernel中配置pinctrl驱动,有以下两种配置方法
1. 首先保证需要使用的板型已经编译过一次(保证配置文件都已经生效, 参考[快速入门](https://siflower.github.io/2020/08/05/quick_start), 然后在openwrt-18.06目录下执行`make kernel_menuconfig`; 然后搜索`PINCTRL_SFAX8`;然后选中`PINCTRL_SFAX8`, 然后`make -j V=s`编译即可
2. 修改`openwrt-18.06/target/linux/siflower/芯片型号/config-4.14_板型`文件
e.g. openwrt-18.06/target/linux/siflower/sf19a28-fullmask/config-4.14_ac28
在上面的文件中寻找以下内容,如果没有则添加
```
CONFIG_PINCTRL=y
CONFIG_PINCTRL_SFAX8=y
```

## 3.2. DTS中配置GPIO
### 3.2.1. GPIO的配置
Siflower的GPIO的驱动是在Pinctrl驱动中统一管理的,在DTS中GPIO的描述也是在Pinctrl的范围内. 以下内容摘取自SF19A2890 FULLMASK芯片的DTS, 仅用作演示.
```
代码参考:linux-4.14.90-dev/linux-4.14.90/arch/mips/boot/dts/siflower/sf19a28_fullmask.dtsi

        gpio: gpio@19d00000 {                                                                                                                                                                 
            compatible = "siflower,sfax8-gpio";
            reg=<0x19d00000 0x100000>;
            interrupts = <GIC_SHARED 246 IRQ_TYPE_NONE>,
                     <GIC_SHARED 247 IRQ_TYPE_NONE>,
                     <GIC_SHARED 248 IRQ_TYPE_NONE>,
                     <GIC_SHARED 249 IRQ_TYPE_NONE>;
            clocks = <&pbusclk 0>;

            gpio-controller;
            #gpio-cells = <2>;
        
            interrupt-controller;
            #interrupt-cells = <2>;
        };
```
上面的内容是dts中对于Siflower的GPIO的描述,用户不用关注以上的内容,也不应该也不能修改相关的内容
### 3.2.2. GPIO的使用
**Note: 再次提醒**   
在linux提供的标准Pinctrl子系统中, Pinctrl子系统实际上也把GPIO一起管理起来，所有的GPIO操作也需要透过pinctrl子系统来完成，如果一个Pin已经被申请为GPIO，再通过Pinctrl子系统申请为某个Function时就会返回错误   

Siflower提供的GPIO在DTS中的使用场景如下:   
GPIO的用途之一就是作为按键或灯, Siflower的GPIO在用作按键和灯的时候, 使用的是linux标准的led子系统和Input子系统. 如下内容所示:
```
代码参考:linux-4.14.90-dev/linux-4.14.90/arch/mips/boot/dts/siflower/sf19a28_fullmask.dtsi

leds: gpio-leds {
    compatible = "gpio-leds";
    status = "disabled";
};

gpio_keys: gpio-keys {
    compatible = "gpio-keys";               
    #address-cells = <1>; 
    #size-cells = <0>; 
    poll-interval = <500>;
    status = "disabled";
};
```
使用的形式如下
> gpios = <&gpio 60 0>   
> gpio-leds = <&gpio 36 0>   
> cs-gpios = <&gpio 5 0>, <&gpio 6 0>  

其中gpios, gpio-leds, cs-gpios是不同的模块的驱动中从dts中获取gpio相关信息所使用的字符串,这个是根据实际的情况自定义的. 
用户主要使用的是第一种形式

> gpios = <&gpio 60 0> 

gpios | &gpio | 60 | 0
-|-|-|-
固定,驱动中用来获取参数的字符串 | 固定, Siflower GPIO的描述| GPIO号 | 可以使用0和1,代表active high和active low,主要用于GPIO按键,其他时候默认为0即可

使用的例子如下:
```
代码参考:linux-4.14.90-dev/linux-4.14.90/arch/mips/boot/dts/siflower/sf19a28_fullmask_ac28.dts

&gpio_keys {
    status = "okay";
    reset-btn@60 {
        label = "reset-btn";
        linux,code = <0x198>;
        gpios = <&gpio 60 1>;
        poll-interval = <10>;
        debounce-interval = <20>;
    };  
}

&wifi_lb {
    status = "okay";

    #address-cells = <1>;
    #size-cells = <0>;
    gpio-leds = <&gpio 36 0>;
    smp-affinity = <2>;
};

spi0: spi@18202000 {
    compatible = "siflower,sfax8-spi";
    reg = <0x18202000 0x1000>;
    num-cs = <1>;
    cs-gpios = <&gpio 8 0>;
    ...
```

# 4. Linux Kernel中GPIO和Pinctrl的使用
## 4.1. Pinctrl驱动
### 4.1.1. 使用方式
```
参考代码路径:linux-4.14.90-dev/linux-4.14.90/drivers/pinctrl/core.c

static int pinctrl_claim_hogs(struct pinctrl_dev *pctldev)
{
    pctldev->p = create_pinctrl(pctldev->dev, pctldev);
    if (PTR_ERR(pctldev->p) == -ENODEV) {
        dev_dbg(pctldev->dev, "no hogs found\n");

        return 0;
    }

    if (IS_ERR(pctldev->p)) {
        dev_err(pctldev->dev, "error claiming hogs: %li\n",
            PTR_ERR(pctldev->p));

        return PTR_ERR(pctldev->p);
    }

    kref_get(&pctldev->p->users);
    pctldev->hog_default =
        pinctrl_lookup_state(pctldev->p, PINCTRL_STATE_DEFAULT);         
    if (IS_ERR(pctldev->hog_default)) {
        dev_dbg(pctldev->dev,
            "failed to lookup the default state\n");
    } else {
        if (pinctrl_select_state(pctldev->p,
                     pctldev->hog_default))
            dev_err(pctldev->dev,
                "failed to select default state\n");
    }    

    pctldev->hog_sleep =
        pinctrl_lookup_state(pctldev->p,
                     PINCTRL_STATE_SLEEP);
    if (IS_ERR(pctldev->hog_sleep))
        dev_dbg(pctldev->dev,
            "failed to lookup the sleep state\n");

    return 0;
}
```
其中，pinctrl_lookup_state接口用于检索对应driver的dts相关信息中是否存在default state。如果检测到default状态后，会将对应的pin脚在pinctrl driver中的相关管脚复用、上下拉等配置配置好。

### 4.1.2. 配置顺序

管脚复用功能在具体各模块驱动probe加载之前将对应模块的管脚复用等配置检索并配置完成。具体的配置顺序见下：（下面顺序仅为部分模块，仅供参考）
```
 	pinctrl-sfax8  
	reg-dummy  
	sf16a18serial    (*4)  
	i2c_siflower      (*3)  
	alarmtimer  
	dma-pl330  
	sf16ax8-pmu  
	sf16ax8-regulator  
	sf16ax8-rtc  
	dwmmc_sfax8      (*2)  
	snd-soc-dummy  
	es8388s-codec  
	spdif-dit  
	sf16ax8-i2s  
	sf16ax8-pcm     (*2)  
	sfax8-spdif  
	soc-audio  
	mmcblk  
	siflower-audio  
```
**注：其中，由于pinctrl驱动需要的优先级较高，将其优先级调至2。**

**上述其余driver对应的管脚功能复用只需要在dts中将管脚复用配置参照EMMC设置好即可，具体的复用配置会在pinctrl.c中进行配置。（不需要额外独自配置）并且，暂时不支持两个有管脚复用的模块一起做上述配置。**



## 4.2. GPIO驱动
### 4.2.1. GPIO pin脚使用
```
参考代码路径:linux-4.14.90-dev/linux-4.14.90/drivers/leds/leds-gpio.c

static int create_gpio_led(const struct gpio_led *template,
    struct gpio_led_data *led_dat, struct device *parent,
    struct device_node *np, gpio_blink_set_t blink_set)
{
    int ret, state;

    led_dat->gpiod = template->gpiod;
    if (!led_dat->gpiod) {
        /*
         * This is the legacy code path for platform code that
         * still uses GPIO numbers. Ultimately we would like to get
         * rid of this block completely.
         */
        unsigned long flags = GPIOF_OUT_INIT_LOW;

        /* skip leds that aren't available */
        if (!gpio_is_valid(template->gpio)) {
            dev_info(parent, "Skipping unavailable LED gpio %d (%s)\n",
                    template->gpio, template->name);
            return 0;
        }

        if (template->active_low)
            flags |= GPIOF_ACTIVE_LOW;

        ret = devm_gpio_request_one(parent, template->gpio, flags,
                        template->name);
        if (ret < 0)
            return ret;

        led_dat->gpiod = gpio_to_desc(template->gpio);
        if (!led_dat->gpiod)
            return -EINVAL;
    }

    led_dat->cdev.name = template->name;
    led_dat->cdev.default_trigger = template->default_trigger;
    led_dat->can_sleep = gpiod_cansleep(led_dat->gpiod);
    if (!led_dat->can_sleep)
        led_dat->cdev.brightness_set = gpio_led_set;
    else
        led_dat->cdev.brightness_set_blocking = gpio_led_set_blocking;
    led_dat->blinking = 0;
    if (blink_set) {
        led_dat->platform_gpio_blink_set = blink_set;
        led_dat->cdev.blink_set = gpio_blink_set;
    }
    if (template->default_state == LEDS_GPIO_DEFSTATE_KEEP) {
        state = gpiod_get_value_cansleep(led_dat->gpiod);
        if (state < 0)
            return state;
    } else {
        state = (template->default_state == LEDS_GPIO_DEFSTATE_ON);
    }
    led_dat->cdev.brightness = state ? LED_FULL : LED_OFF;
    if (!template->retain_state_suspended)
        led_dat->cdev.flags |= LED_CORE_SUSPENDRESUME;
    if (template->panic_indicator)
        led_dat->cdev.flags |= LED_PANIC_INDICATOR;
    if (template->retain_state_shutdown)
        led_dat->cdev.flags |= LED_RETAIN_AT_SHUTDOWN;

    ret = gpiod_direction_output(led_dat->gpiod, state);
    if (ret < 0)
        return ret;

    return devm_of_led_classdev_register(parent, np, &led_dat->cdev);
}
  

```
以上内容为标准的led子系统里面调用GPIO接口的方法,在dts中wifi模块注册了一个灯用来显示wifi的状态. 
```
参考代码路径: linux-4.14.90-dev/linux-4.14.90/arch/mips/boot/dts/siflower/sf19a28_fullmask_ac28.dts

&wifi_hb {
    status = "okay";

    #address-cells = <1>;
    #size-cells = <0>;
    gpio-leds = <&gpio 12 0>;                                                                                                                                                                 
    smp-affinity = <3>;
};
```

## 4.3. GPIO具体接口简介

以下接口,均是linux的标准GPIO驱动的接口

### 4.3.1. 获取GPIO管脚号
```
	gpio1 = of_get_named_gpio(dev->of_node, "test-gpios", 0);
```
获取需要使用的GPIO管脚号。&quot;test-gpios&quot; 为dts中对应的名字，可以随意；0为使用GPIO管脚对应的位置，若写为1参照dts则为gpio31。

### 4.3.2. 申请GPIO
```
 	ret = devm_gpio_request(dev, gpio1, NULL); 
```
申请GPIO，gpio1为申请gpio管脚号，NULL为别名，可随意。

### 4.3.3. 配置为GPIO输出
```
 	gpio_direction_output(gpio_reset, 0);
```
将gpio_reset管脚设置为OUTPUT，0代表的是电平，即设置为低电平。（1为高电平）

### 4.3.4. 配置为GPIO输入
```
 	gpiod_direction_input(gpio_to_desc(gpio_reset));  
```
将gpio_reset 管脚设置为INPUT功能。

### 4.3.5. 配置电平
```
 	gpiod_set_value(gpio_reset,1);
```
设置电平，gpio_reset 为gpio管脚号，1为高电平（0为低电平）。

### 4.3.6. 获取电平
```
 	ret = gpiod_get_value(gpio_to_desc(gpio1));
```
获取gpio1 的电平。  



## 4.4. GPIO相关头文件

注：使用GPIO相关功能需要以下头文件：
```
 	#include <linux/of_gpio.h>
 	#include <linux/gpio.h>
```
## 4.5. GPIO在命令行中的使用
以下步骤均是通过串口和相应的硬件进行交互,关于串口的使用请参考[快速入门](https://siflower.github.io/2020/08/05/quick_start), 以下的操作是Linux标准驱动所支持的.

首先，gpio在linux系统起来之后的操作路径为：/sys/class/gpio  
若发现没有该路径的话，可以在linux的kernel menuconfig中选中Device Drivers > GPIOSupport > /sys/class/gpio/... (sysfs interface)  
```
	admin@SiWiFi:/sys/class/gpio# ls
	export     gpiochip0  unexport
```
进入/sys/class/gpio目录之后，我们可以看到上述的内容，其中，  
1. export文件用于通知系统需要导出控制的GPIO引脚编号;  
2. unexport 用于通知系统取消导出;  
3. gpiochipX目录保存系统中GPIO寄存器的信息，包括每个寄存器控制引脚的起始编号base，寄存器名称，引脚总数导出一个引脚的操作步骤;  

具体操作：  
1） 先导出一个需要操作的引脚，eg，操作GPIO11，操作见下：  
```
	admin@SiWiFi:/sys/class/gpio# echo 11 > export
	admin@SiWiFi:/sys/class/gpio# ls
	export     gpio11     gpiochip0  unexport
	admin@SiWiFi:/sys/class/gpio# cd gpio11/
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# ls
	active_low  direction   power       uevent
	device      edge        subsystem   value  
```
2） 这时，我们可以在gpio目录下看到我们到处的引脚GPIO11，进入该目录，可以看到相关的可配置参数direction、value等。  
3） 其中，direction用于配置输入输出方向，可接受参数包含了in、out、high、low，high/low同时设置方向为输出，并将value设置为相应的1/0。  
4） value用于配置GPIO的高低电平，为1或0。 eg， echo 1 > value。  
5） 我们可以在/sys/kernel/debug/gpio中获取当前配置为GPIO功能的管脚的具体状态。  
6） 使用unexport来取消该GPIO管脚的导出。eg,echo 11 > unexport  
以下是具体的一次GPIO使用操作，按照上述说明进行具体操作：  
```
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# cat value 
	0
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# echo out > direction 
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# ls
	active_low  direction   power       uevent
	device      edge        subsystem   value
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# cat direction 
	out
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# cat value 
	0
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# echo 1 > value 
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# cat value 
	1
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# echo low > direction 
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# cat value 
	0
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# 
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# cat /sys/kernel/debug/gpio 
	GPIOs 0-61, platform/pinctrl, gpio:
	 gpio-5   (?                   ) out hi    
	 gpio-6   (?                   ) out hi    
	 gpio-11  (sysfs               ) out lo    
	 gpio-57  (reset led gpio      ) in  lo    
	 gpio-60  (reset button gpio   ) in  hi    
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# 
	admin@SiWiFi:/sys/devices/pinctrl/gpio/gpio11# cd ../
	admin@SiWiFi:/sys/class/gpio# echo 11 > unexport 
	admin@SiWiFi:/sys/class/gpio# ls
	export     gpiochip0  unexport
	admin@SiWiFi:/sys/class/gpio# 
```

# 5. 项目引用

此项目参考文档
- [GPIO在用户层的操控](http://blog.csdn.net/aaronychen/article/details/50476019)

# 6. FAQ

1. 在模块驱动中调用pinctrl相关接口进行管脚复用配置时，如果两个模块配置了同一个GPIO管脚，则会在第二次配置时报错。也就是说，不允许同一个管脚在不释放的情况下重复进行管脚复用配置。  
2. GPIO上下拉的生效时间。  
经过查看仿真波形，当设置管脚配置为上/下拉状态时，需要等待11uS后，管脚的上/下拉状态才能真正稳定状态，在11uS器件管脚的电平在逐步变高/低。  
3. 查看模块内部寄存器，发现GPIO模拟内部本身就有触发类型设置，具体含义？？GPIO模块出去到CPU的中断类型是电平还是边沿？？  
分析：针对该问题，软件出了相关的仿真用例，查看仿真波形，可以确认以下信息：  
GPIO模块内部可配置的触发类型，是针对外部设备到GPIO模块的中断类型，每个PIN脚都可独立控制；  
而GPIO模块到CPU的中断类型统一为高电平。  
**注：A18中的所有到GIC的中断源，只有WIFI有一个中断是低电平触发，其他全部都是高电平触发（IC的RTL代码中统一将中断做了翻转，改为高电平触发）。**
4. A18各个管脚是否可以灵活配置下驱动电流？  
根据spec说明，SW_DS0_REG等就是配置管脚驱动电流的，每个管脚不论在任何功能配置下，都可以做驱动电流配置。  
分别取三个DS0 DS1 DS2三个同地址的，同bit位，然后三个bit组合的值为驱动电路配置。具体配置见下表：  

| DS2 | DS1 | DS0 | Low Level Output Current (mA) | High Level Output Current (mA) |
| --- | --- | --- | --- | --- |
| 0 | 0 | 0 | 6.3 | 9.3 |
| 0 | 0 | 1 | 9.4 | 14.0 |
| 0 | 1 | 0 | 12.6 | 18.6 |
| 0 | 1 | 1 | 15.6 | 23.3 |
| 1 | 0 | 0 | 18.7 | 27.9 |
| 1 | 0 | 1 | 21.8 | 32.5 |
| 1 | 1 | 0 | 24.7 | 37.2 |
| 1 | 1 | 1 | 27.7 | 41.8 |
