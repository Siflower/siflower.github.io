---
layout: post
title: DTS替代注册说明
categories: 32M_MEMORY
description: DTS替代注册说明
keywords: Linux DTS
topmost: true
mermaid: true
---

* TOC
{:toc}


# 1. 适用人员

使用siflower 4+32项目SDK，对linux内核有一定了解的人员。

# 2. 使用说明

- 目前4+32项目没有添加设备树文件，需要我们自己想总线注册platform设备，设备里主要包含寄存器地址信息等资源

## 以wifi设备驱动注册为例
- 参照代码:package/kernel/siflower/sf_smac/src/bb_src/umac/fullmac/siwifi_main.c


1.由于没有设备树 我们需要定义platform_device结构体 向总线注册设备
```
static struct platform_device wifi_lb_fmac_device = {
    .name       = SF_WIFI_MODENAME,
    .id = 0,
    .resource   = sf_wifi_lb_fmac_resources,
    .num_resources  = ARRAY_SIZE(sf_wifi_lb_fmac_resources),
    .dev    = {
        .platform_data = &lb_mod_param
    },
};
```

  - name   为设备名，需要和 platform_driver里的name保持一致
  - id  设备号，由于wifi是同一个驱动下有两个设备  所以这里引用id 做区分，0为2.4g 1为5g
  - resource  为硬件资源 ,结构体数组中存放多个寄存器开始、结束地址，以及寄存器类型

```
static struct resource sf_wifi_lb_fmac_resources[] = {

    {
        .start  = SF16A18_WIFI_LB_BASEADDR,
        .end    = SF16A18_WIFI_LB_ENDADDR,
        .flags  = IORESOURCE_MEM,
    },
    {
        .start  = 0x1c000000,
        .end    = 0x1c007fff,
        .flags  = IORESOURCE_MEM,
    },
    {
        .start  = SF16A18_WIFI_LB_SBD_IRQ,
        .end    = SF16A18_WIFI_LB_SBD_IRQ,
        .flags  = IORESOURCE_IRQ,
    },
};
```
- num_resources 资源数量  就是sf_wifi_lb_fmac_resources数组的成员数量
 - dev 携带设备私有数据

2. platform_device_register(&wifi_lb_fmac_device) 注册设备
3. siwifi_platform_register_drv 注册驱动  在有设备树的情况下 是直接进行这一步的
4. 定义platform_driver
```
static struct platform_driver siwifi_hp_drv = {
    .probe = siwifi_hp_probe,
    .remove = siwifi_hp_remove,
    .driver = {
        .owner = THIS_MODULE,
        .name = SF_WIFI_MODENAME,
    }
};
```
   - probe 驱动执行函数
   - remove 驱动注销函数
   - drver.name 驱动名 __这里要和device名一致__

   __这里如果有设备树 是有一个of_match_table匹配表  用来去设备树中匹配设备__


5. 除了要手动注册设备外 模块退出 也要 手动注销设备
`platform_device_unregister(&wifi_hb_fmac_device);`


# 3. 总结

使用设备树的方式需要在设备树中定义相应的 device 节点，并在驱动程序中解析设备树节点信息来注册 device 设备。这种方式的好处在于，设备树可以描述系统硬件的层次结构和属性信息，使得驱动程序更加灵活、可移植。同时，设备树还可以通过 overlay 的方式动态修改系统硬件配置，而不需要重新编译内核。

而不使用设备树的方式则需要在驱动程序中直接编写 device 相关的代码，包括寄存器操作和中断处理等。这种方式的好处在于，可以直接访问硬件寄存器，更加高效，同时也可以减少代码量和复杂度。

总的来说，使用设备树的方式更加灵活、可移植，但是需要解析设备树节点信息来注册设备，相对复杂一些；而不使用设备树的方式则更加直接、高效，但是需要直接编写硬件相关代码，相对麻烦一些。