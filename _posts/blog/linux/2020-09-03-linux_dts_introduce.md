---
layout: post
title: Linux DTS说明文档
categories: LINUX
description: Linux DTS说明文档
keywords: 文档开发
mermaid: true
---
# Linux DTS说明文档   

**目录**
* TOC
{:toc}

# 1. 介绍
本文主要介绍Linux的Device Tree的用法

## 1.1. 适用人员
本文适用于需要了解DTS用法的开发人员

## 1.2. 开发环境
- 可以正常编译通过的Siflower SDK环境
  该环境的搭建请参考[快速入门](https://siflower.github.io/2020/08/05/quick_start)

# 2. 设备树
## 2.1. 设备树用法
本文介绍如何为新的机器或板卡编写设备树，它旨在概要性的介绍设备树概念，以及如何使用它们来描述机器或者板卡。
有关设备树数据格式的完整技术描述，请参阅devicetree-specification规范。如何获取规范文档，请参考后面章节[获取规范文档](#3-获取规范文档)

### 2.1.1. 基本数据格式
设备树（Device Tree）是一种包含节点和属性的简单树形结构。属性是键值对，节点则可能包含属性和子节点。 例如，下面是一个.dts格式的简单设备树:

```
/dts-v1/;

/ {
    node1 {
        a-string-property = "A string";
        a-string-list-property = "first string", "second string";
        // hex is implied in byte arrays. no '0x' prefix is required
        a-byte-data-property = [01 23 34 56];
        child-node1 {
            first-child-property;
            second-child-property = <1>;
            a-string-property = "Hello, world";
        };
        child-node2 {
        };
    };
    node2 {
        an-empty-property;
        a-cell-property = <1 2 3 4>; /* each number (cell) is a uint32 */
        child-node1 {
        };
    };
};
```
上面这个设备树，显然没有实际用处，因为它没有描述任何信息，但是它确实显示了节点的结构和属性：
- 一个简单的root节点：“/”
- 一组子节点：“node1”和“node2”
- 一组node1的子节点：“child-node1”和“child-node2”
- 一堆分散在设备树中的属性

属性是简单的键-值对，其中的值可以是空的，也可以包含任意的字节流。虽然数据类型没有编码到数据结构中，但是有一些基本的数据表示可以在设备树源文件中表示。
- 文本字符串（以NULL终止）以双引号表示： `string-property = "a string"`;
- “单元格”是由尖括号分隔的32位无符号整数： `cell-property = <0xbeef 123 0xabcd1234>`;
- 二进制数据用方括号定界： `binary-property = [0x01 0x23 0x45 0x67]`;
- 可以使用逗号将不同表示形式的数据连接在一起： `mixed-property = "a string", [0x01 0x23 0x45 0x67], <0x12345678>`;
- 逗号还用于创建字符串列表： `string-list = "red fish", "blue fish"`;

## 2.2. 基本概念
为了了解如何使用设备树，我们将从一台简单的机器开始，并构建一个设备树来逐步描述它。

### 2.2.1. 实例设备
有以下机器（基于MIPS InterAptiv的设备），由Siflower设计制造，名为“sf19a28-soc”：
- 1个32位MIPS CPU (型号为MIPS InterAptiv)
- 处理器本地总线连接到内存映射串口、spi总线控制器、i2c控制器、中断控制器和外部总线桥
- 基于0x0的256MB SDRAM
- 2个基于0x18300000和0x18301000的串行端口
- 基于0x19d00000的GPIO控制器
- 基于0x18200000的SPI控制器，带有以下器件 Nor-flash “w25p80”
- 以太网设备位于0x10000000
- 基于0x18100000的i2c控制器， 基于i2c控制器的电源管理单元(PMU). 对应的从设备的地址为110000 (0x30)

### 2.2.2. 初始结构
第一步是为设备铺设骨架结构。这是有效设备树所需的最低要求。在此阶段，您要唯一标识机器。

```
/dts-v1/;

/ {
    compatible = "siflower,sf19a28-soc";
};
```

“`compatible`”表明系统的名字。它包含一个以"`<manufacturer>,<model>`"(“制造商”，“品牌”)形式组成的字符串。准确的表明设备非常重要，而且需要包含制造商的名称以避免命名冲突。 由于操作系统将使用compatible值来决定如何在机器上运行，因此将正确的数据写入此属性中非常重要。理论上，一个操作系统唯一识别一台设备只要有“`compatible`”属性就够了。如果所有设备细节都是硬编码的，那么操作系统可以专门在Device Tree的最顶层的“`compatible`”属性中查找"`compatible`"即可

### 2.2.3. CPUs
下一步是为每个CPU进行描述。每个CPU都添加了一个名为“ cpus”的容器节点和一个子节点。在这种情况下，该系统是MIPS的四核interAptiv系统。
```

/dts-v1/;

/ {
compatible = "siflower,sf19a28-soc";

cpus {
	cpu@0 {
		compatible = "mips,interAptiv";
	};
	cpu@1 {
		compatible = "mips,interAptiv";
	};
	cpu@2 {
		compatible = "mips,interAptiv";
	};
	cpu@3 {
		compatible = "mips,interAptiv";
	};
};
};
```
每个cpu节点中的compatible属性是一个字符串，该字符串以表格的形式指定确切的cpu模型  `<manufacturer>,<model>`，就像顶级属性一样。

稍后将向cpu节点添加更多属性，但是我们首先需要讨论更多基本概念。

### 2.2.4. 节点名称
首先，我们需要了解命名的规则。每一个节点必须有一个名字，名字的形式必须是“`<name>[@<unit-address>`”, `<name>`是一个简单的ascii字符串，最大长度为31字节。通常，节点是根据它所代表的设备来命名。比如，一个3com公司的网络适配器的名字可能会是“ethernet”，而不是“3com509”。如果这个节点描述设备需要一个地址，则包含“`<unit-address>`”字段。通常，这个地址是访问该设备寄存器所需要的首地址。而且会列在node的“`reg`”属性里。关于“`reg`”属性将在本文后续的内容中介绍。兄弟节点的名字不能相同，但是通常都是`<name>`字段相同，而`<unit-address>`字段不同（ (比如, serial@18300000 & serial@18301000).）

### 2.2.5. 设备
系统中的每个设备都由一个设备树节点表示。下一步是为每个设备的节点填充树。现在，在我们可以讨论地址范围和irq的处理方式之前，新节点将保持空白。
```

/dts-v1/;

/ {
compatible = "siflower,sf19a28-soc";

cpus {
	cpu@0 {
		compatible = "mips,interAptiv";
	};
	cpu@1 {
		compatible = "mips,interAptiv";
	};
	cpu@2 {
		compatible = "mips,interAptiv";
	};
	cpu@3 {
		compatible = "mips,interAptiv";
	};
};

gic: interrupt-controller@1bdc0000 {
    compatible = "mti,gic";
};


ethernet@10000000 {
	compatible = "siflower,sfax8-eth";
};
i2c@18100000 {
	compatible = "siflower,sfax8-i2c";
	pmu@30 {
		compatible = "siflower, sfax8-pmu";
	};
};
spi@18200000 {
	compatible = "siflower,sfax8-spi";
	w25q128@0 {
			compatible = "w25q128", “m25p80”;
	};
};
serial@18300000 {
	compatible = "siflower,sfax8-uart";
};
serial@18301000 {
	compatible = "siflower,sfax8-uart";
};

pinctrl {
	compatible = "siflower,sfax8-pinctrl";
	gpio@19d00000 {
		compatible = "siflower,sfax8-gpio";
	};
};
};
```
在上面这个树中，已经为系统中的每个设备都添加了一个节点，层次结构反映了设备如何连接到系统。比如，外部总线上的设备是外部总线节点的子节点，i2c设备是i2c总线控制器节点的子节点。通常，层级结构表示的是从CPU的角度来看系统的视图。
这个设备树还不能使用，是因为它缺少设备之间的连接信息，接下来会添加进来。
需要注意的是：
- 每一个节点都有一个“`compatible`”属性
- flash节点的“`compatible`”属性包含了两个字符串，下一节将说明为什么
- 就像之前提到的，节点名字反映的是设备的种类，而不是代表具体的品牌型号。 请参阅规范文档，其中列出了已定义的通用节点名。应该尽可能使用这些节点名，而不要发明新的名字。

### 2.2.6. 理解“compatible”属性性
设备树中每一个表示设备的节点都要有“`compatible`”属性。 “`compatible`”属性是操作系统用来决定将哪个设备驱动程序绑定到这个设备的关键。

“`compatible`”是一个字符串列表，列表中的第一个字符串以“制造商”，“型号”的形式指定确切的设备。接下来的字符串表示该设备兼容的其他设备。例如，Freescale MPC8349片上系统（Soc）有一个串行设备，执行National Semiconductor的 ns16550 寄存器接口。那么Freescale MPC8349的串行设备的“`compatible`”属性就是`“fsl,mpc8349-uart”，“ns16550”`.在这个例子里，`“fsl,mpc8349-uart”`表示确切的设备，`“ns16550”`表示它在寄存器级别与National Semiconductor的ns16550串行设备兼容。这里的“ns16550”没有制造商的名字，是由于历史原因。所有新的兼容性设备名称，都需要加制造商的名字。这种做法允许将现有设备驱动程序绑定到新设备，同时仍然惟一地标识确切的硬件。

警告：不要使用通配符来实现兼容性，比如 `"fsl,mpc83xx-uart"`或者类似的值。但芯片供应商总是会修改命名规则，一旦等到其改变打破了你的通配符假设，再来修改就已经晚了。所以，请选择一个具体的芯片型号，然后再兼容性列表里面列出所兼容的芯片型号。
树中代表设备的每个节点都必须具有兼容属性。兼容是操作系统用来决定将哪个设备驱动程序绑定到设备的关键。

## 2.3. 如何寻址
可寻址的设备使用以下属性将地址信息编码到设备树中：
- `reg`
- `＃address-cells`
- `＃size-cells`

每一个可寻址设备都有一个叫“reg”的属性，reg由一系列元组构成，形式是`reg = <address1 length1 [address2 length2] [address3 length3] ... >`。也就是address、length交替出现。每一个元组代表一个设备使用的地址范围。每一个地址的值是一个或者多个32位int型数构成的列表，称为cells。长度的取值也是一个cells列表，或者为空。
由于address和length字段都是可变大小的变量，因此父节点中的`#address-cells`和`#size-cells`属性用于说明每个字段中有多少个单元格。换句话说，正确地解释reg属性需要父节点的`#address-cells`和`#size-cells`值。要了解这一切是如何工作的，让我们将寻址属性添加到示例设备树中，从cpu开始。`#address-cells`表示address的长度，`#size-cells`表示length的长度，如果为0，则表示没有。

### 2.3.1. CPU寻址
当谈论寻址时，CPU节点代表最简单的情况。每个CPU都分配有一个唯一的ID，并且没有与CPU ID相关的大小。
```
    cpus {
        #address-cells = <1>;
        #size-cells = <0>;
        cpu@0 {
            compatible = "mips,interAptiv";
            reg = <0>;
        };
        cpu@1 {
            compatible = "mips,interAptiv";
            reg = <1>;
        };
	     cpu@2 {
            compatible = "mips,interAptiv";
            reg = <2>;
        };
        cpu@3 {
            compatible = "mips,interAptiv";
            reg = <3>;
        };
    };
```
在cpu节点中，将`#address-cells`设置为1，将`#size-cells`设置为0。这意味着子reg值是一个uint32数，它是一个没有size字段的地址。在这种情况下，为这两个cpu分配了地址0和1。对于cpu节点，`#size-cells`为0，因为每个cpu只分配一个地址，没有其他的地址。
您还会注意到reg值与节点名中的@符号后面的值相同。按照习惯，如果节点具有reg属性，那么节点名必须包含单元地址，而且是reg属性中的第一个地址的值。。

### 2.3.2. 内存映射设备
与CPU节点只有一个地址值不同，内存映射设备会分配一个它需要响应的地址区间。`#size-cells`表示reg中length的位宽，如果是32位宽就是1,64位宽就是2.在下面的示例里，每个address值是1个单元格（32位），每个length也是一个单元格（32位），这在32位操作系统中是非常常见的。在64位系统可能将`#address-cells`、`#size-cells`都设置为2，从而完成64位地址空间的寻址。
```
/dts-v1/;

/ {
    #address-cells = <1>;
    #size-cells = <1>;

...
gic: interrupt-controller@1bdc0000 {
    compatible = "mti,gic";
    reg = <0x1bdc0000 0x20000>;
    interrupt-controller;
    #interrupt-cells = <3>;
    mti,reserved-ipi-vectors = <0 8>;
    timer {
        compatible = "mti,gic-timer";
        interrupts = <GIC_LOCAL 1 IRQ_TYPE_NONE>;
        clocks = <&cpupll 0>;
    };
};

ethernet@10000000 {
	compatible = "siflower,sfax8-eth";
	reg = <0x10000000 0x6CFFFF>;
};
i2c@18100000 {
	compatible = "siflower,sfax8-i2c";
	reg = <0x18100000 0x1000>;
	pmu@30 {
		compatible = "siflower, sfax8-pmu";
	};
};
spi@18200000 {
	compatible = "siflower,sfax8-spi";
	reg = <0x18200000 0x1000>;
	w25q128@0 {
		compatible = "w25q128", “m25p80”;
	};
};
serial@18300000 {
	compatible = "siflower,sfax8-uart";
	reg = <0x18300000 0x1000>;
};
serial@18301000 {
	compatible = "siflower,sfax8-uart";
	reg = <0x18301000 0x1000>;
};

pinctrl {
	compatible = "siflower,sfax8-pinctrl";
	#address-cells = <1>;
	#size-cells = <1>;
	gpio@19d00000 {
		compatible = "siflower,sfax8-gpio";
		reg = <0x19d00000 0x100000
		       0x19e30000 0x100000>;
	};
};
```
每个设备都分配有一个基地址，并分配了它的区域大小。在此示例中，为GPIO设备地址分配了两个地址范围；即：0x19d00000 ... 0x19dfffff和0x19e30000..0x19f2ffff。

有一些设备挂载的总线的寻址方式不同。比如，有些设备挂载的总线是通过不同的片选信号线来区分设备。 由于每个父节点都为其子节点定义了寻址域，因此可以选择一种最匹配的方式来描述其子节点的寻址方式。下面的代码显示了一种挂载到外部总线的设备的寻址方式，并将片选信号编码到地址域中。
```
    palmbus@10000000 {
        compatible = "palmbus";
        #address-cells = <1>; 
        #size-cells = <1>; 

        ethernet: ethernet@0000000 {
            compatible = "siflower,sfax8-eth";
        };
		uart0: serial@8300000 {
            compatible = "siflower,sfax8-uart";
        };

        uart1: serial@8301000 {
            compatible = "siflower,sfax8-uart";
        };


```
总线“palmbus”使用2个单元格来表示地址域，一个是片选号，一个是该片选的设备的基地址偏移。“length”字段仍然是一个单元格，因为只有地址的偏移需要一个范围。所以在这个例子里，每一个reg包含3个单元格：片选，地址偏移，偏移的范围。

由于地址域包含在节点及其子节点中， 所以父节点可以自由地定义任何对总线有意义的寻址方案。设备节点不需要考虑本节点之外的地址域的情况。地址映射必须按照地址域一个一个的进行。

### 2.3.3. 非内存映射设备
有一些设备不是地址映射设备。他们可以有地址范围，但是不能被CPU直接访问。相反，父设备的驱动将代替CPU间接的访问设备。以I2C设备为例，每一个设备分配一个地址，但是没有长度和地址范围。看起来就像为CPU分配地址一样。
```
    i2c@18100000 {
		compatible = "siflower,sfax8-i2c";
		reg = <0x18100000 0x1000>;
		#address-cells = <1>;
		#size-cells = <0>;
		pmu@30 {
			compatible = "siflower, sfax8-pmu";
			reg = <0x30>;
		};
    };
```
### 2.3.4. ranges（地址转换）
我们已经讨论了如何为设备分配地址，但目前这些地址只在设备节点地址域有意义。它还没有描述如何将这些地址映射到CPU可以使用的地址。 根节点总是以CPU的视角来描述地址空间。根节点的子节点已经使用了CPU的地址域，因此不需要任何显式映射。例如，serial@18300000设备被直接分配地址0x18300000。
一些非根节点的子节点的设备没有直接使用CPU地址域。 为了获得内存映射地址，设备树必须指定如何将地址从一个域转换到另一个域。ranges属性正是为这一目的而设计的。
下面就是一个简单的例子，展示了一个包含ranges属性的设备树。
```
/dts-v1/;
/ {
    compatible = "acme,coyotes-revenge";
    #address-cells = <1>;
    #size-cells = <1>;
    ...
    external-bus {
        #address-cells = <2>
        #size-cells = <1>;
        ranges = <0 0  0x10100000   0x10000     // Chipselect 1, Ethernet
                  1 0  0x10160000   0x10000     // Chipselect 2, i2c controller
                  2 0  0x30000000   0x1000000>; // Chipselect 3, NOR Flash
 
        ethernet@0,0 {
            compatible = "smc,smc91c111";
            reg = <0 0 0x1000>;
        };
 
        i2c@1,0 {
            compatible = "acme,a1234-i2c-bus";
            #address-cells = <1>;
            #size-cells = <0>;
            reg = <1 0 0x1000>;
            rtc@58 {
                compatible = "maxim,ds1338";
                reg = <58>;
            };
        };
 
        flash@2,0 {
            compatible = "samsung,k8f1315ebm", "cfi-flash";
            reg = <2 0 0x4000000>;
        };
    };
};
```
ranges是一个地址转换列表。ranges表的每一项都是一个组元，包含子地址、父地址和子地址空间区域的大小。每一个字段的大小由子节点的 #address-cells 的值，父节点的 #address-cells 的值，以及子节点的 #size-cells 值决定。 对于我们示例中的外部总线，子地址是2个单元格，父地址是1个单元格，子地址大小也是1个单元格。因此，三项ranges翻译如下：
```
  Offset 0 from chip select 0 is mapped to address range 0x10100000..0x1010ffff
  Offset 0 from chip select 1 is mapped to address range 0x10160000..0x1016ffff
  Offset 0 from chip select 2 is mapped to address range 0x30000000..0x30ffffff
```

或者，如果父地址空间和子地址空间相同，则节点可以添加一个空的“ranges”属性。空“ranges”属性的存在意味着子地址空间中的地址将1:1映射到父地址空间。
您可能会问，为什么要使用地址转换，而所有这些都可以用1:1映射来编写。 有些总线(如PCI)具有完全不同的地址空间，其详细信息需要暴露给操作系统。其他的DMA引擎需要知道总线上的真实地址。有时需要将设备分组，因为它们共享相同的软件可编程物理地址映射。是否应该使用1:1映射在很大程度上取决于操作系统和具体的硬件设计信息。
您还应该注意到，i2c@1,0节点中没有ranges属性。原因在于，与外部总线不同，i2c总线上的设备不是映射到CPU地址域中的内存。相反，CPU通过i2c@1,0设备间接访问rtc@58设备。缺少范围属性意味着设备不能被除其父设备之外的任何设备直接访问。。

## 2.4. 中断如何工作
与遵循树的自然结构的地址范围转换不同， 中断信号可以起源于或者终止于板卡上的任何设备。 与设备树中自然表示的设备寻址不同，中断信号的表示独立于设备树节点之间的连接。通常用下面的四个属性来描述一个中断连接：
- `interrupt-controller` - 一个空属性，声明一个接收中断信号的设备节点
- `#interrupt-cells` -  这是中断控制器节点的一个属性。它声明中断控制器的 interrupt specifier（中断描述符）占用多少单元格(类似于`#address-cells`和`#size-cells`)。
- `interrupt-parent` - 一种包含指向中断控制器句柄指针的属性；如果没有该属性，节点也可以从其父节点继承该属性
- `interrupts` - 包含一系列的interrupt specifier的属性，每一个interrupt specifier表示设备发出的一个中断信号

一个interrupt specifier包含1个或多个单元格的数据（具体多少个单元格由“`#interrupt-cells`”属性决定）， 它指定设备连接到哪个中断输入。大多数设备只有一个中断输出，如下面的例子所示，但是在一个设备上可以有多个中断输出。 interrupt specifier的含义完全取决于中断控制器设备的绑定。每个中断控制器可以决定需要多少单元格来唯一地定义一个中断输入。

以下代码将中断连接添加到我们的Siflower sf19a2890 dtsi：
```
/dts-v1/;

/ {
	compatible = "siflower,sf19a28-soc";
    #address-cells = <1>;
    #size-cells = <1>;

...
gic: interrupt-controller@1bdc0000 {
    compatible = "mti,gic";
    reg = <0x1bdc0000 0x20000>;
    interrupt-controller;
    #interrupt-cells = <3>;
    mti,reserved-ipi-vectors = <0 8>;
    timer {
        compatible = "mti,gic-timer";
        interrupts = <GIC_LOCAL 1 IRQ_TYPE_NONE>;
        clocks = <&cpupll 0>;
    };
};

ethernet@10000000 {
	compatible = "siflower,sfax8-eth";
	reg = <0x0000000 0x6CFFFF>;
	interrupts = <GIC_SHARED 16 IRQ_TYPE_NONE>;
};
i2c@18100000 {
	compatible = "siflower,sfax8-i2c";
	reg = <0x18100000 0x1000>;
	interrupts = <GIC_SHARED 217 IRQ_TYPE_NONE>;
	pmu@30 {
		compatible = "siflower, sfax8-pmu";
	};
};
spi@18200000 {
	compatible = "siflower,sfax8-spi";
	reg = <0x18200000 0x1000>;
	interrupts = <GIC_SHARED 223 IRQ_TYPE_NONE>;
	w25q128@0 {
    	compatible = "w25q128", “m25p80”;
	};
};
serial@18300000 {
	compatible = "siflower,sfax8-uart";
	reg = <0x18300000 0x1000>;
	interrupts = <GIC_SHARED 226 IRQ_TYPE_NONE>;
};
serial@18301000 {
	compatible = "siflower,sfax8-uart";
	reg = <0x8301000 0x1000>;
	interrupts = <GIC_SHARED 227 IRQ_TYPE_NONE>;
};

pinctrl {
	compatible = "siflower,sfax8-pinctrl";
	#address-cells = <1>;
	#size-cells = <1>;
	gpio@19d00000 {
		compatible = "siflower,sfax8-gpio";
		reg = <0x19d00000 0x100000
		       0x19e30000 0x100000>;
        interrupts = <GIC_SHARED 246 IRQ_TYPE_NONE>,
                <GIC_SHARED 247 IRQ_TYPE_NONE>,
                <GIC_SHARED 248 IRQ_TYPE_NONE>,
                <GIC_SHARED 249 IRQ_TYPE_NONE>;
	};
};
```
注意事项：

- 机器具有单个中断控制器`gic: interrupt-controller@1bdc0000`。
- 标签`'gic：'` 已添加到中断控制器节点，并且该标签用于将phandle分配给palmbus节点中的	`interrupt-parent`属性。此中断父级值成为系统的默认值，因为所有子节点都将继承该值，除非明确覆盖它。
- 每个设备使用中断属性来指定不同的中断输入线。
- `＃interrupt-cells`为3，因此每个中断说明符都有3个单元。此示例使用以下通用模式：使用第一个单元指定中断的类型，第二个单元编码中断行号，第三个单元编码标志，例如高电平有效对低电平有效，边沿敏感对电平敏感。对于任何给定的中断控制器，请参阅控制器的绑定文档以了解说明符的编码方式。

## 2.5. Device Specific Data
在通用的属性之外，还可以在节点里添加任意的属性和子节点。任何操作系统所需的数据都可以加进来，只要遵守以下的规则：
首先， 新的特定于设备的属性名应该有一个制造商名字作为前缀，这样它们就不会与现有的标准属性名冲突。
其次，必须在 binding 中记录属性和子节点的含义，以便设备驱动程序的作者知道如何解释数据。 binding 记录了特定兼容值的含义、它应该具有的属性、它可能具有的子节点以及它表示的设备。每个惟一的“ compatible”属性都应该有自己的 binding (或声明与另一个“ compatible”属性相兼容)。新设备的 binding 记录在此它的wiki中。有关文档格式和审查流程的描述，请参考[嵌入式Linux wiki](https://elinux.org/Main_Page)。
第三，在devicetree-discuss@lists.ozlabs.org邮件列表中发布新的binding以供审查。新的binding的代码审查能发现许多常见的错误，这些错误将来会导致问题。。

## 2.6. Special Nodes
### 2.6.1. 别名（aliases node）
完整路径通常引用一个特定节点，例如 `serial@18300000`，但是当用户真正想知道的是“哪个设备是uart0？”时，这变得很麻烦。别名节点可用于将短别名分配给完整的设备路径。例如：
```
aliases {
	spi0 = &spi0;
	uart0=&uart0;
	uart1=&uart1;
};
```
在为设备分配标识符时，操作系统支持使用别名。
可以发现，在这里使用了一个新的语法。`“属性= &label;`”这个语法将标签引用的完整节点路径指定为字符串属性。这与文章前面出现的，`phandle = < &label >`不同（前面是把一个phandle插入到cell中）;

### 2.6.2. chosen 节点
chosen节点不代表真实的设备，而是用作在固件和操作系统之间传递数据的地方，比如boot参数。所选节点中的数据不代表硬件。通常，所选节点在.dts源文件中为空，并在启动过程中时填充。
在我们的示例系统中，固件可能会将以下内容添加到chosen节点:

```
    chosen {
        bootargs = "console=ttyS0,115200n8 rootfstype=squashfs,jffs2 rdinit=/sbin/init";
    };
```

# 3. 获取规范文档
规范文档的获取方法有很多种，本章节只是介绍了一种通过github获取的方法
1. 浏览器直接输入https://github.com/devicetree-org/devicetree-specification， 或者点击后面链接接直达[devicetree-specification](https://github.com/devicetree-org/devicetree-specification)的github界面

2. 选择devicetree-specification的tags，从而获取自己想要的版本的规范文档, 网页的操作流程参考后面的图片, 注意红色方框和箭头指示的内容

![1](/assets/images/bsp/linux_dts1.png)
![2](/assets/images/bsp/linux_dts2.png)
![3](/assets/images/bsp/linux_dts3.png)

# 4. 项目引用
- [嵌入式linux之DTS的用法](https://elinux.org/Device_Tree_Usage)
- [github Device Tree项目](https://github.com/devicetree-org/devicetree-specification)
- [嵌入式Linux wiki](https://elinux.org/Main_Page)

