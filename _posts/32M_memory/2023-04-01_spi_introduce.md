---
layout: post
title: SPI驱动说明
categories: 32M_MEMORY
description: SPI驱动
keywords: Linux SPI
topmost: true
mermaid: true
---

* TOC
{:toc}


# 1. 介绍

本文档是基于Linux 2.6.39内核版本对SPI驱动调试过程以及注意事项。该版本同3.x版本以上的内核略有不同，本文进行说明。以4+32项目为例，驱动代码路径`/target/linux/siflower/Linux-2.6.39/drivers/spi/sfax8_spi.c`。

## 1.1 适用人员

平台开发人员。

## 1.2 相关背景知识

SPI相关知识。

# 2. 对接驱动

## 2.1 设备对接

linux 2.6.39版本内核没有Linux dts设备树，与h8898项目的spi设备注册不同。设备对接时，需要访问 `/target/linux/siflower/Linux-2.6.39/arch/mips/siflower/spi.c`文件进行配置。在4+32项目中，利用了平台设备驱动框架，配置展示如下：
```
    static struct sfax8_spi_info sfax8_spi_master_data;

    //spi设备资源
    static struct resource sfax8_spi_resources[] = {
            //内存资源
            {
                    .start  = SIFLOWER_SPI2_REGISTER_BASE,                              //起始地址
                    .end    = SIFLOWER_SPI2_REGISTER_BASE + SIFLOWER_SPI2_OFFSET - 1,   //结束地址
                    .flags  = IORESOURCE_MEM,
            },

            //中断资源
            {
                    .start  = SIFLOWER_SPI_IRQ,                                     //中断信息起始地址与结束地址一致，表示设备中断号
                    .end    = SIFLOWER_SPI_IRQ,
                    .flags  = IORESOURCE_IRQ,
            },
    };

    //平台设备设定为spi设备
    static struct platform_device sfax8_spi_device = {
            .name           = "sfax8-spi",
            .id             = 0,
            .dev            = {
                    .platform_data = &sfax8_spi_master_data,
            },
            .num_resources  = ARRAY_SIZE(sfax8_spi_resources),
            .resource       = sfax8_spi_resources,
    };

    //注册spi设备
    void __init sfax8_register_spi(struct sfax8_spi_info *info,
                                   struct spi_board_info *devices, int num)
    {
            sfax8_spi_master_data = *info;
            spi_register_board_info(devices, num);
            platform_device_register(&sfax8_spi_device);
    }
```

有了spi设备注册，m25p80的驱动将会拉高拉低gpio管脚8，去使用已经注册的spi设备进行mtd的分区操作。对应设置在`/target/linux/siflower/Linux-2.6.39/arch/mips/siflower/spi.c`文件

```
struct flash_platform_data sfax8_flash_data ={
        .type       =     "mx25l3205d",
        .parts      =     sfax8_spi_partitions,     //mtd分区配置
        .nr_parts   =     ARRAY_SIZE(sfax8_spi_partitions),
};

//m25p80设备板级信息
struct spi_board_info sfax8_spi_board_info[] __initdata = {
        {
                .modalias               = "m25p80",         //SPI NOR驱动设备
                .platform_data          = &sfax8_flash_data,//SPI NOR驱动模式选择
                .controller_data        = &sfax8_hw,        //gpio 8号管脚控制
                .max_speed_hz           = 20000000,         //最大频率
                .bus_num                = 0,                //总线号
                .chip_select            = 0,                //片选信号
                .mode                   = SPI_MODE_0,       //片选模式，与片选信号对应
        },
};

static struct sfax8_spi_info sfax8_spi_info __initdata = {
        .num_chipselect = ARRAY_SIZE(sfax8_spi_board_info),
};

//m25p80设备注册spi设备
void __init sfax8_init_spi(void)
{
        sfax8_register_spi(&sfax8_spi_info, sfax8_spi_board_info,
                           ARRAY_SIZE(sfax8_spi_board_info));
}

```

## 2.2 驱动描述

1.  由sfax8-spi驱动去申请一个spi\_master对象，初始化其必要的成员变量；

```

    master = spi_alloc_master(&pdev->dev, sizeof(*sfspi));
            if (!master) {
                    dev_err(&pdev->dev, "failed to allocate spi master\n");
                    return -ENOMEM;
            }
```
2.  spi\_master对象依次去实现spi设备创建，spi传输，以及spi设备释放等流程;

```

    master->setup = sfax8_spi_setup;
            master->transfer = sfax8_spi_transfer;
            master->cleanup = sfax8_spi_cleanup;
            master->bus_num = pdev->id;
            master->num_chipselect = info->num_chipselect;
            master->mode_bits = SPI_CPOL | SPI_CPHA | SPI_CS_HIGH;
```
3.  由sfspi-spi驱动执行获取中断、设备资源、寄存器、中断相应，执行spi协议传输

```

    sfspi->irq = platform_get_irq(pdev, 0);
            if (sfspi->irq < 0) {
                    error = -EBUSY;
                    dev_err(&pdev->dev, "failed to get irq resources\n");
                    goto fail_put_clock;
            }

            res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
            if (!res) {
                    dev_err(&pdev->dev, "unable to get iomem resource\n");
                    error = -ENODEV;
                    goto fail_put_clock;
            }

            res = request_mem_region(res->start, resource_size(res), pdev->name);
            if (!res) {
                    dev_err(&pdev->dev, "unable to request iomem resources\n");
                    error = -EBUSY;
                    goto fail_put_clock;
            }

            sfspi->regs_base = ioremap(res->start, resource_size(res));
            if (!sfspi->regs_base) {
                    dev_err(&pdev->dev, "failed to map resources\n");
                    error = -ENODEV;
                    goto fail_free_mem;
            }

            error = request_irq(sfspi->irq, sfax8_spi_interrupt, 0,
                                "sfax8-spi", sfspi);
            if (error) {
                    dev_err(&pdev->dev, "failed to request irq\n");
                    goto fail_unmap_regs;
            }

            sfspi->wq = create_singlethread_workqueue("sfax8_spid");
            if (!sfspi->wq) {
                    dev_err(&pdev->dev, "unable to create workqueue\n");
                    goto fail_free_irq;
            }
            INIT_WORK(&sfspi->msg_work, sfax8_spi_work);
            INIT_LIST_HEAD(&sfspi->msg_queue);
            sfspi->running = true;
```
4.  将sfspi-spi驱动注册加入内核

```

    static int __init sfax8_spi_init(void)
    {
            return platform_driver_probe(&sfax8_spi_driver, sfax8_spi_probe);
    }
    module_init(sfax8_spi_init);
```

新增驱动需要对应在`drivers/spi/Makefile`和`driver/spi/Kconfig`中增加内容。
```
    obj-$(CONFIG_SPI_SFAX8)			+= sfax8_spi.o
```

```
config SPI_SFAX8
	tristate "Siflower SFAX8 SPI controller"
	help
	  This enables using the Siflower SFAX8 SPI controller in master
	  mode.

	  To compile this driver as a module, choose M here. The module will be
	  called sfax8_spi.

```

# 3. 调试说明

*   m25p80驱动使用spi传输协议实现mtd分区操作，如果是spi设备配置，以及spi驱动配置有误。那么在启动内核阶段，则会停滞在内核的分区模块。
*   可以通过示波器的抓取tx、rx的波形。芯片作为master，flash作为slave，可以抓取flash管脚的电波信号。

