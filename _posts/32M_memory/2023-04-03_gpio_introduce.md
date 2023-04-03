---
layout: post
title: GPIO使用手册
categories: 32M_MEMORY
description: GPIO使用
keywords: Linux GPIO
topmost: true
mermaid: true
---

* TOC
{:toc}


# 1. 介绍

本文档是基于Linux 2.6.39内核版本对GPIO驱动调试过程以及注意事项。

## 1.1 适用人员

平台开发人员。

# 2. 主要结构体

```
/*主要用于存放gpio_chip、irq_chip、基地址、时钟、中断号等信息。*/
struct sfax8_gpio_chip {
        void __iomem            *base;
        struct gpio_chip        chip;
        struct platform_device  *pdev;
        spinlock_t              lock;

        unsigned                irq_base;
        struct irq_chip         ic;
        int                     irq[SFAX8_NUM_BANKS];
        struct sfax8_gpio_irqdata irq_data[SFAX8_NUM_BANKS];
        unsigned int            irq_type[PAD_INDEX_MAX];
        struct clk              *clk;
};
```

```
/*主要用于存放request、free、direction_input、direction_output、get、set、ngpio等参数和接口。*/
struct gpio_chip {
        const char              *label;
        struct device           *dev;
        struct module           *owner;
        int                     (*request)(struct gpio_chip *chip,
                                                unsigned offset);
        void                    (*free)(struct gpio_chip *chip,
                                                unsigned offset);
        int                     (*direction_input)(struct gpio_chip *chip,
                                                unsigned offset);
        int                     (*get)(struct gpio_chip *chip,
                                                unsigned offset);
        int                     (*direction_output)(struct gpio_chip *chip,
                                                unsigned offset, int value);
        int                     (*set_debounce)(struct gpio_chip *chip,
                                                unsigned offset, unsigned debounce);
        void                    (*set)(struct gpio_chip *chip,
                                                unsigned offset, int value);
        int                     (*to_irq)(struct gpio_chip *chip,
                                                unsigned offset);
        void                    (*dbg_show)(struct seq_file *s,
                                                struct gpio_chip *chip);
        int                     base;
        u16                     ngpio;
        const char              *const *names;
        unsigned                can_sleep:1;
        unsigned                exported:1;
#if defined(CONFIG_OF_GPIO)
        /*
         * If CONFIG_OF is enabled, then all GPIO controllers described in the
         * device tree automatically may have an OF translation
         */
        struct device_node *of_node;
        int of_gpio_n_cells;
        int (*of_xlate)(struct gpio_chip *gc, struct device_node *np,
                        const void *gpio_spec, u32 *flags);
#endif
};
```

```
/*该结构体主要用来存放了中断的状态查询、中断屏蔽、中断解除屏蔽和中断触发类型设置。*/
struct irq_chip {
        const char      *name;
        unsigned int    (*irq_startup)(struct irq_data *data);
        void            (*irq_shutdown)(struct irq_data *data);
        void            (*irq_enable)(struct irq_data *data);
        void            (*irq_disable)(struct irq_data *data);
        void            (*irq_ack)(struct irq_data *data);
        void            (*irq_mask)(struct irq_data *data);
        void            (*irq_mask_ack)(struct irq_data *data);
        void            (*irq_unmask)(struct irq_data *data);
        void            (*irq_eoi)(struct irq_data *data);
        int             (*irq_set_affinity)(struct irq_data *data, const struct cpumask *dest, bool force);
        int             (*irq_retrigger)(struct irq_data *data);
        int             (*irq_set_type)(struct irq_data *data, unsigned int flow_type);
        int             (*irq_set_wake)(struct irq_data *data, unsigned int on);
        void            (*irq_bus_lock)(struct irq_data *data);
        void            (*irq_bus_sync_unlock)(struct irq_data *data);
        void            (*irq_cpu_online)(struct irq_data *data);
        void            (*irq_cpu_offline)(struct irq_data *data);
        void            (*irq_print_chip)(struct irq_data *data, struct seq_file *p);
        unsigned long   flags;
        /* Currently used only by UML, might disappear one day.*/
#ifdef CONFIG_IRQ_RELEASE_METHOD
        void            (*release)(unsigned int irq, void *dev_id);
#endif
};
```

# 2. GPIO管脚初始化

pinmux代码：`arch/mips/siflower/pinmux.c`
在该代码中对所有pin脚进行初始化操作：

```
static int sfax8_set_func( u32 pg, pad_func func)
{
    int tmp;
    if(pg > SF_MAX_PINGROUP)
                return -EINVAL;
    switch(func)
    {
        case FUNC0:
        case FUNC1:
        case FUNC2:
        case FUNC3:
            tmp = pg_readl(PAD_INDEX_REG1(pg));
            tmp &= ~(0x7 << 0);
            tmp |= (func << 0) | (0x1 << FUNC_SW_SEL_REG);
            pg_writel( PAD_INDEX_REG1(pg), tmp);

            tmp = pg_readl(PAD_INDEX_REG0(pg));
            tmp &= ~(0x1 << SW_OEN_REG);
            pg_writel(PAD_INDEX_REG0(pg), tmp);

            tmp = pg_readl(PAD_INDEX_REG0(pg));
            tmp |=(0x1 << SW_IE_REG);
            pg_writel(PAD_INDEX_REG0(pg), tmp);
            break;

        case GPIO_INPUT:
            tmp = pg_readl(PAD_INDEX_REG0(pg));
            tmp |= (0x1 << SW_OEN_REG);
            pg_writel(PAD_INDEX_REG0(pg), tmp);

            tmp = pg_readl(PAD_INDEX_REG0(pg));
            tmp |= (0x1 << SW_IE_REG);
            pg_writel(PAD_INDEX_REG0(pg), tmp);

            tmp = pg_readl(PAD_INDEX_REG1(pg));
            tmp |= (0x1 << FMUX_SEL_REG);
            pg_writel(PAD_INDEX_REG1(pg), tmp);

            sfax8_gpio_wr(GPIO_DIR(pg), 1);
            break;

        case GPIO_OUTPUT:
            tmp = pg_readl(PAD_INDEX_REG0(pg));
            tmp &= ~(0x1 << SW_OEN_REG);
            pg_writel(PAD_INDEX_REG0(pg), tmp);

            tmp = pg_readl(PAD_INDEX_REG0(pg));
            tmp &= ~(0x1 << SW_IE_REG);
            pg_writel(PAD_INDEX_REG0(pg), tmp);

            tmp = pg_readl(PAD_INDEX_REG1(pg));
            tmp |= (0x1 << FMUX_SEL_REG);
            pg_writel(PAD_INDEX_REG1(pg), tmp);

            sfax8_gpio_wr(GPIO_DIR(pg), 0);
            break;

        default:
            printk("sf_pad_set_func error! pin:%d func:%d \n", pg, func);
            return -EINVAL;
            break;
    }
    return 0;
}
```

可以直接通过调用`sfax8_set_func(pin, func)`函数来修改管脚功能

```
static int __init sf_pinmux_init(void)
{
    int i, err;
    int pin = 1;

    for(i = 0; i < SF_MAX_PINGROUP; i++){
        enum siflower_pingroup pin = i;
        err = sfax8_set_func(pin, 0);
    }
  .....
    return 0;
}
```

# 3. GPIO使用介绍

## GPIO接口调用

申请该gpio对应的资源，如果被占用了，返回错误，请求失败。

```
int gpio_request(unsigned gpio, const char *label)
```

配置gpio方向，输入/输出

```
int gpiod_direction_output(struct gpio_desc *desc, int value)
int gpiod_direction_input(struct gpio_desc *desc)
```

配置/获取该gpio当前电平

```
int gpiod_get_value(struct gpio_desc *desc)
void gpiod_set_value(struct gpio_desc *desc, int value)
```

配置/获取该gpio当前电平。

```
void gpio_free(unsigned gpio)
```



