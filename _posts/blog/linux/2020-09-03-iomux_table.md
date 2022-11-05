---
layout: post
title: Siflower IOMUX Table
categories: LINUX
description: Siflower IOMUX Table
keywords: 文档开发
mermaid: true
---
# Siflower IOMUX Table

**目录**

* TOC
{:toc}

# 1. 介绍

Siflower的芯片提供一些PAD脚，这些PAD脚可以配置为不同的功能,本文提供了Siflower sf19a2890 fullmaks芯片的IOMUX Table

## 1.1. 适用人员

- 有查看Siflower IOMUX Table的需求的人员

## 1.2. 开发环境
- 可以正常编译通过的Siflower SDK环境
  该环境的搭建请参考[快速入门](https://siflower.github.io/2020/08/05/quick_start)

# 2. SF19A2890 fullmask IOMUX Table

下面的表格就是Siflower的SF19A2890 fullmask的IOMUX表格, 在该表格中会说明Siflowr的每一个可以配置的PAD的支持配置成那些模式. 

| Index | Pin Name        | Default Mode | GPIO\_MODE | FUNC\_MODE0                       | FUNC\_MODE1 | FUNC\_MODE2 | FUNC\_MODE3        |
|-------|-----------------|--------------|------------|-----------------------------------|-------------|-------------|--------------------|
| 0     | JTAG\_TDO       | FUNC\_MODE0  | GPIO0      | JTAG\_TDO                         | UART1\_TXD  | ETH\_LED0   | CATIP\_SCLK        |
| 1     | JTAG\_TDI       | FUNC\_MODE0  | GPIO1      | JTAG\_TDI                         | UART1\_RXD  | ETH\_LED1   | CATIP\_SS          |
| 2     | JTAG\_TMS       | FUNC\_MODE0  | GPIO2      | JTAG\_TMS                         | UART1\_CTS  | ETH\_LED2   | CATIP\_MOSI        |
| 3     | JTAG\_TCK       | FUNC\_MODE0  | GPIO3      | JTAG\_TCK                         | UART1\_RTS  | ETH\_LED3   | PWM0               |
| 4     | JTAG\_RST       | FUNC\_MODE0  | GPIO4      | JTAG\_RST                         | NC          | PWM5        | CATIP\_MISO        |
| 5     | SPI\_TXD        | FUNC\_MODE0  | GPIO5      | SPI2\_TXD                         | NC          | NC          | NC                 |
| 6     | SPI\_RXD        | FUNC\_MODE0  | GPIO6      | SPI2\_RXD                         | NC          | NC          | NC                 |
| 7     | SPI\_CLK        | FUNC\_MODE0  | GPIO7      | SPI2\_CLK                         | NC          | NC          | NC                 |
| 8     | SPI\_CSN        | FUNC\_MODE0  | GPIO8      | SPI2\_CSN                         | NC          | NC          | NC                 |
| 9     | UART\_TX        | FUNC\_MODE0  | GPIO9      | UART0\_TXD                        | NC          | NC          | NC                 |
| 10    | UART\_RX        | FUNC\_MODE0  | GPIO10     | UART0\_RXD                        | NC          | NC          | NC                 |
| 11    | I2C\_DAT        | FUNC\_MODE0  | GPIO11     | UART0\_CTS                        | UART2\_RXD  | I2C0\_DAT   | NC                 |
| 12    | I2C\_CLK        | FUNC\_MODE0  | GPIO12     | UART0\_RTS                        | UART2\_TXD  | I2C0\_CLK   | NC                 |
| 13    | RGMII\_GTX\_CLK | GPIO\_MODE   | GPIO13     | RGMII0\_GTX\_CLK / RMII\_REF\_CLK | NC          | NC          |                    |
| 14    | RGMII\_TXCLK    | GPIO\_MODE   | GPIO14     | RGMII0\_TXCLK                     | NC          | NC          | NC                 |
| 15    | RGMII\_TXD0     | GPIO\_MODE   | GPIO15     | RGMII0\_TXD0 / RMII\_TXD0         | NC          | NC          | NC                 |
| 16    | RGMII\_TXD1     | GPIO\_MODE   | GPIO16     | RGMII0\_TXD1 / RMII\_TXD1         | NC          | NC          | NC                 |
| 17    | RGMII\_TXD2     | GPIO\_MODE   | GPIO17     | RGMII0\_TXD2                      | NC          | NC          | NC                 |
| 18    | RGMII\_TXD3     | GPIO\_MODE   | GPIO18     | RGMII0\_TXD3                      | NC          | NC          | NC                 |
| 19    | RGMII\_TXCTL    | GPIO\_MODE   | GPIO19     | RGMII0\_TXCTL / RMII\_TX\_EN      | NC          | NC          | NC                 |
| 20    | RGMII\_RXCLK    | GPIO\_MODE   | GPIO20     | RGMII0\_RXCLK                     | NC          | NC          | NC                 |
| 21    | RGMII\_RXD0     | GPIO\_MODE   | GPIO21     | RGMII0\_RXD0 / RMII\_RXD0         | NC          | NC          | NC                 |
| 22    | RGMII\_RXD1     | GPIO\_MODE   | GPIO22     | RGMII0\_RXD1 / RMII\_RXD1         | NC          | NC          | NC                 |
| 23    | RGMII\_RXD2     | GPIO\_MODE   | GPIO23     | RGMII0\_RXD2                      | NC          | NC          | NC                 |
| 24    | RGMII\_RXD3     | GPIO\_MODE   | GPIO24     | RGMII0\_RXD3                      | NC          | NC          | NC                 |
| 25    | RGMII\_RXCTL    | GPIO\_MODE   | GPIO25     | RGMII0\_RXCTL / RMII\_CRS\_DV     | NC          | NC          | NC                 |
| 26    | RGMII\_COL      | GPIO\_MODE   | GPIO26     | RGMII0\_COL                       | NC          | NC          | NC                 |
| 27    | RGMII\_CRS      | GPIO\_MODE   | GPIO27     | RGMII0\_CRS                       | NC          | NC          | NC                 |
| 28    | RGMII\_MDC      | GPIO\_MODE   | GPIO28     | RGMII0\_MDC / RMII\_MDC           | NC          | NC          | NC                 |
| 29    | RGMII\_MDIO     | GPIO\_MODE   | GPIO29     | RGMII0\_MDIO / RMII\_MDIO         | NC          | NC          | NC                 |
| 30    | HB0\_PA\_EN     | GPIO\_MODE   | GPIO30     | HB0\_PA\_EN                       | NC          | NC          | CATIP\_DEBUG\_IO0  |
| 31    | HB0\_LNA\_EN    | GPIO\_MODE   | GPIO31     | HB0\_LNA\_EN                      | NC          | NC          | CATIP\_DEBUG\_IO1  |
| 32    | HB0\_SW\_CTRL0  | GPIO\_MODE   | GPIO32     | HB0\_SW\_CTRL0                    | NC          | NC          | CATIP\_DEBUG\_IO2  |
| 33    | HB0\_SW\_CTRL1  | GPIO\_MODE   | GPIO33     | HB0\_SW\_CTRL1                    | NC          | NC          | CATIP\_DEBUG\_IO3  |
| 34    | HB1\_PA\_EN     | GPIO\_MODE   | GPIO34     | HB1\_PA\_EN                       | NC          | NC          | CATIP\_DEBUG\_IO4  |
| 35    | HB1\_LNA\_EN    | GPIO\_MODE   | GPIO35     | HB1\_LNA\_EN                      | NC          | NC          | CATIP\_DEBUG\_IO5  |
| 36    | HB1\_SW\_CTRL0  | GPIO\_MODE   | GPIO36     | HB1\_SW\_CTRL0                    | NC          | NC          | CATIP\_DEBUG\_IO6  |
| 37    | HB1\_SW\_CTRL1  | GPIO\_MODE   | GPIO37     | HB1\_SW\_CTRL1                    | NC          | NC          | CATIP\_DEBUG\_IO7  |
| 38    | LB0\_PA\_EN     | GPIO\_MODE   | GPIO38     | LB0\_PA\_EN                       | NC          | NC          | CATIP\_DEBUG\_IO8  |
| 39    | LB0\_LNA\_EN    | GPIO\_MODE   | GPIO39     | LB0\_LNA\_EN                      | NC          | NC          | CATIP\_DEBUG\_IO9  |
| 40    | LB0\_SW\_CTRL0  | GPIO\_MODE   | GPIO40     | LB0\_SW\_CTRL0                    | NC          | NC          | CATIP\_DEBUG\_IO10 |
| 41    | LB0\_SW\_CTRL1  | GPIO\_MODE   | GPIO41     | LB0\_SW\_CTRL1                    | NC          | NC          | CATIP\_DEBUG\_IO11 |
| 42    | LB1\_PA\_EN     | GPIO\_MODE   | GPIO42     | LB1\_PA\_EN                       | NC          | NC          | CATIP\_DEBUG\_IO12 |
| 43    | LB1\_LNA\_EN    | GPIO\_MODE   | GPIO43     | LB1\_LNA\_EN                      | NC          | NC          | CATIP\_DEBUG\_IO13 |
| 44    | LB1\_SW\_CTRL0  | GPIO\_MODE   | GPIO44     | LB1\_SW\_CTRL0                    | NC          | NC          | CATIP\_DEBUG\_IO14 |
| 45    | LB1\_SW\_CTRL1  | GPIO\_MODE   | GPIO45     | LB1\_SW\_CTRL1                    | NC          | NC          | CATIP\_DEBUG\_IO15 |
| 46    | CLK\_OUT        | GPIO\_MODE   | GPIO46     | CLK\_OUT                          | NC          | NC          | NC                 |
| 47    | EXT\_CLK\_IN    | FUNC\_MODE0  | GPIO47     | EXT\_CLK\_IN                      | NC          | NC          | NC                 |
| 48    | DRVVBUS0        | GPIO\_MODE   | GPIO48     | DRVVBUS0                          | NC          | NC          | NC                 |


# 3. 如何在shell命令行获取PAD脚的状态
## 3.1. kernel选择DEBUG_FS配置

可以通过make kernel_menuconfig选择, 然后编译镜像. 如何编译烧录镜像参考[快速入门](https://siflower.github.io/2020/08/05/quick_start)

DEBUG_FS的位置如下:
```
Kernel hacking  --->
    Compile-time checks and compiler options  ---> 
        [*]Debug Filesystem 
```
也可以通过直接修改修改target/linux/siflower/sf19a28-fullmask/config-3.18_$(board)文件,添加如下内容
`CONFIG_DEBUG_FS=y`

## 3.2. 命令

烧录选择了CONFIG_DEBUG_FS的镜像,在运行的时候就可以通过命令`cat /sys/kernel/debug/pinctrl/pinctrl/pins`,在结合IOMUX Table即可知道每一个PAD所处的模式, 该命令的显示结果如下所示:

```
root@OpenWrt:/# cat /sys/kernel/debug/pinctrl/pinctrl/pins   
registered pins: 51   
pin 0 (gpio-0) function func0 in lo; irq 31 (none)  
pin 1 (gpio-1) function func0 in lo; irq 32 (none)  
pin 2 (gpio-2) function func0 in lo; irq 33 (none)  
pin 3 (gpio-3) function func0 in lo; irq 34 (none)  
pin 4 (gpio-4) function func0 in lo; irq 35 (none)  
pin 5 (gpio-5) function func0 in lo; irq 36 (none)  
pin 6 (gpio-6) function func0 in lo; irq 37 (none)  
pin 7 (gpio-7) function func0 in lo; irq 38 (none)  
pin 8 (gpio-8) function gpio_in in hi; irq 39 (none)  
pin 9 (gpio-9) function func0 in lo; irq 40 (none)  
pin 10 (gpio-10) function func0 in lo; irq 41 (none)  
pin 11 (gpio-11) function gpio_out in hi; irq 42 (edge-both)  
pin 12 (gpio-12) function func0 in lo; irq 43 (none)  
pin 13 (gpio-13) function func0 in lo; irq 44 (none)  
pin 14 (gpio-14) function func0 in lo; irq 45 (none)  
pin 15 (gpio-15) function func0 in lo; irq 46 (none)  
pin 16 (gpio-16) function func0 in lo; irq 47 (none)  
pin 17 (gpio-17) function func0 in lo; irq 48 (none)  
pin 18 (gpio-18) function func0 in lo; irq 49 (none)  
pin 19 (gpio-19) function func0 in lo; irq 50 (none)  
pin 20 (gpio-20) function func0 in lo; irq 51 (none)  
pin 21 (gpio-21) function func0 in lo; irq 52 (none)  
pin 22 (gpio-22) function func0 in lo; irq 53 (none)  
pin 23 (gpio-23) function func0 in lo; irq 54 (none)  
pin 24 (gpio-24) function func0 in lo; irq 55 (none)  
pin 25 (gpio-25) function func0 in lo; irq 56 (none)  
pin 26 (gpio-26) function func0 in lo; irq 57 (none)  
pin 27 (gpio-27) function func0 in lo; irq 58 (none)  
pin 28 (gpio-28) function func0 in lo; irq 59 (none)  
pin 29 (gpio-29) function func0 in lo; irq 60 (none)  
pin 30 (gpio-30) function func0 in lo; irq 61 (none)  
pin 31 (gpio-31) function func0 in lo; irq 62 (none)  
pin 32 (gpio-32) function func0 in lo; irq 63 (none)  
pin 33 (gpio-33) function func0 in lo; irq 64 (none)  
pin 34 (gpio-34) function func0 in lo; irq 65 (none)  
pin 35 (gpio-35) function func0 in lo; irq 66 (none)  
pin 36 (gpio-36) function func0 in lo; irq 67 (none)  
pin 37 (gpio-37) function func0 in lo; irq 68 (none)  
pin 38 (gpio-38) function func0 in lo; irq 69 (none)  
pin 39 (gpio-39) function func0 in lo; irq 70 (none)  
pin 40 (gpio-40) function func0 in lo; irq 71 (none)  
pin 41 (gpio-41) function func0 in lo; irq 72 (none)  
pin 42 (gpio-42) function func0 in lo; irq 73 (none)  
pin 43 (gpio-43) function func0 in lo; irq 74 (none)  
pin 44 (gpio-44) function func0 in lo; irq 75 (none)  
pin 45 (gpio-45) function func0 in lo; irq 76 (none)  
pin 46 (gpio-46) function func0 in lo; irq 77 (none)  
pin 47 (gpio-47) function func0 in lo; irq 78 (none)  
pin 48 (gpio-48) function gpio_out in lo; irq 79 (none)  
pin 49 (gpio-49) function func0 in lo; irq 80 (none)  
pin 50 (gpio-50) function func0 in lo; irq 81 (none)  
root@OpenWrt:/#   
```

其中`pin 0 (gpio-0)`代表的就是index为0, 而且是gpio-0; `function func0` 代表的就是function的模式为func0; `in lo`代表的就是该pin的状态(是高电平还是低电平);`irq 42 (edge-both)`代表的是该pin注册的中断号是42, 且为edge-both触发方式(边沿触发).

中断的触发方式有:

| Interrupt trigger mode | Description |
|------------------------|-------------|
| none                   | 触发方式未知  |
| edge\-rising           | 上升沿触发    |
| edge\-falling          | 下降压触发    |
| edge\-both             | 边沿触发      |
| level\-high            | 高电平触发    |
| level\-low             | 低电平触发    |

Function有如下

| Function  | Description     |
|-----------|-----------------|
| func0     | 对应IOMUX Tabel查看 |
| func1     | 对应IOMUX Tabel查看 |
| func2     | 对应IOMUX Tabel查看 |
| func3     | 对应IOMUX Tabel查看 |
| gpio\_in  | GPIO的Input模式    |
| gpio\_out | GPIO的Output模式   |

in Pin脚的电平状态有如下

| status | Description |
|--------|-------------|
| lo     | 低电平       |
| hi     | 高电平       |

# FAQ
本文主要是提供Siflower的IOMUX Table, 关于IOMUX的具体应用请参考[Pinctrl 和 GPIO 使用手册](https://siflower.github.io/2020/07/30/pinctrl_gpio)
