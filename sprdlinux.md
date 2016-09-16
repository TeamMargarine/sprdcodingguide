 Sprdlinux Programming Guideline
=====

目录  (Content)
-----
[TOC]

**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [ Sprdlinux Programming Guideline](#)
	- [目录  (Content)](#)
	- [ 前言](#)
	- [ 第一章: 命名 (Naming)](#)
		- [公司名 (Manufacturer Name)](#)
		- [平台名 (Family Name)](#)
		- [芯片名 (Chip Name)](#)
		- [板名 (Board Name)](#)
		- [目录名 (Directory Name)](#)
		- [文件名 (File Name)](#)
			- [.dts/.dtsi文件](#)
			- [.c/.h/.S文件](#)
		- [函数名 (Function Name)](#)
		- [变量名 (Variable Name)](#)
		- [宏名 (Macro Name)](#)
		- [作者名 (Author Name)](#)
	- [第二章：文件布局 ( Layout )](#)
		- [硬件描述](#)
		- [平台初始化](#)
		- [驱动模块](#)
		- [头文件](#)
		- [固件](#)
	- [第三章：DT描述 ( Device Tree ) ](#)
		- [总线拓扑结构](#)
		- [命名](#)
		- [文件分层](#)
		- [属性binding](#)
		- [aliases的妙用](#)
		- [虚拟设备](#)
		- [Interrupt controller](#)
		- [clock](#)
		- [power management](#)
	- [第四章：子系统 ( Sub-system & Module )](#)
		- [基本输出 ( stdout )](#)
		- [pinctrl](#)
		- [时钟](#)
		- [中断](#)
		- [内存控制](#)
		- [DMA](#)
		- [电源管理](#)
			- [PSCI](#)
		- [Framebuffer](#)
		- [按键和LED](#)
		- [多媒体](#)
		- [全局寄存器问题](#)
		- [sipc](#)
	- [第五章：ARM64兼容](#)
		- [初始化](#)
		- [DT](#)
		- [ABI](#)
	- [第六章：工具 ( Tools )](#)
	- [第七章：社区 ( Community )](#)

 前言
-----
编程是一门语言艺术——不开玩笑。
程序强调理性逻辑，语言讲究感性表达。
逻辑不正确，程序无法运行；表达不优雅，文章难成佳作。

计算机编程语言诞生于上世纪60年代，历史不足百年，但数百万程序员笔耕不缀，汗牛充栋。Linux内核代码历时20多年，已达千万行之巨；sprd kernel 近2年的提交，累计也已有100多万行。
我经历过一个公司，早期只追求程序运行结果，而忽视代码表达，还说什么“多快好省，坏了再整”——可后来负责“坏了再整”的那些人，没有一个不边看代码边骂娘的。

编程有如行文，又胜过行文，不仅要结构清晰，逻辑正确，而且要表达优雅，简洁易懂。
外在和内在巧妙融合，理性和感性完美统一，当事物达到这种境界，人们就会称之为艺术。

 第一章: 命名 (Naming)
-----
古语云：名不正，则言不顺；言不顺，则事不成。
虽然是讲治世之道，但细细想来，似乎和编程规律也隐约相通。
Linux kernel集万众之力，成万千代码。目录名、文件名、函数名、变量名、宏名，乃至作者名，如果各自为政，恐怕早就一团乱麻了。
综合各方分析，采众家之长，Sprdlinux社区版约定如下：

### 公司名 (Manufacturer Name)

>- 全称 **"Spreadtrum"**
>- 缩写 **"sprd"**

全称仅在注释、描述、介绍中提及公司时使用。
文件、变量、函数中只使用缩写，宏名按惯例使用大写 **“SPRD”**

> 在现有代码中存在sc的使用，如mach-sc。
> sc是spreadtrum芯片前缀，如sc7731。
> 作为后来者，sc极易和upstream中的既有名称混淆重复。

### 平台名 (Family Name)

我司芯片除ARM核部分，SoC架构基本相似，均源于shark.
名称纷繁的原因多源于市场策略和通信制式的不同。而按SoC惯例，是以cpu为视角来划分。
因此缩减平台名称如下:
>- **tiger**: Cortex-A5MP2 32bit架构
>- **shark**: cortex-A7MP4 32bit架构(scx30)，含原shark, tshark,  pike 
>- **sharkl**: cortex-A7MP4 32bit 或 cortex-A53 64bit架构(scx35)，含原sharkl, tsharkl(sharkl64).
>- **whale**:  cortex-A53MP8 Big-LITTLE架构

-  _争议_：upstream曾存在一个平台也叫做shark, 目前已被移除。
但多数情况下，不要安排SHARK单独使用，而是和SPRD组合，如： **ARCH_SPRD_SHARK**
也不要设计这样的目录：mach_shark，因为shark,sharkl都包含在mach-sprd里。
可以避免重名的问题。

- _关于scx_: 现有平台命名有使用“scx+数字”的方式。
	- 这种命名无任何官方依据
	- 和芯片型号或者手机型号组合后看上去很罗嗦： sprd_scx35l_sc9630.dts

### 芯片名 (Chip Name)

>- **主力**芯片型号 **sc####**

因我司芯片型号较为灵活，因此**引脚兼容**、**配置相同**的芯片统一用最初命名的型号代表，称之为该型号兼容。也就是说，没有重新流片的芯片，无论remark成什么型号，都使用最初的型号代表。
比如： sc1234和sc1155仅仅因为销售策略而型号不同，因为sc1234最先使用，sc1155称之为sc1234 compatible，不再另行命名。

### 板名 (Board Name)

>- 手机型号**sp####xxx[-....]**

如 sp7730gga；如果同一个型号又有区分，则在后部添加描述，如sp7731gga-openphone, sp7731gga-customer等。

### 目录名 (Directory Name)

以下相对于kernel源代码目录树根目录:

- 包含公司名、平台名、芯片名、板名的目录，不应位于第一、二级目录（firmware目录除外）
-  公司名(**sprd**)应用作独立的目录名用以和其他厂商区分，不应和功能连体，尤其是一大堆并列，如当前tsharl_re分支中的drivers/media/下：

**Looks BAD:**
>   ./drivers/media/**sprd_gsp**
>   ./drivers/media/**sprd_dma_copy**
>    ./drivers/media/**sprd_rotation**
>    ./drivers/media/**sprd_isp**
>    ./drivers/media/**sprd_dcam**
>    ./drivers/media/**sprd_sensor**
>    ./drivers/media/**sprd_scale**
>    ./drivers/misc/**sprd_vsp**
>    ./drivers/misc/**sprd_cproc**
>    ./drivers/misc/**sprd_srt**
>    ./drivers/misc/**sprd_2351**
>    ./drivers/misc/**sprd_jpg**
>    ./drivers/misc/**sprd_otp**

**Feel better?**
>    ./drivers/media/**sprd**/dma_copy
>    ./drivers/media/**sprd**/rotation
>    ./drivers/media/**sprd**/isp
>    ./drivers/media/**sprd**/dcam
>    ./drivers/media/**sprd**/sensor
>    ./drivers/media/**sprd**/scale
>    ./drivers/misc/**sprd**/vsp
>    ./drivers/misc/**sprd**/cproc
>    ./drivers/misc/**sprd**/srt
>    ./drivers/misc/**sprd**/2351
>    ./drivers/misc/**sprd**/jpg
>    ./drivers/misc/**sprd**/otp

- ARM32体系架构目录：arch/arm/**mach-sprd/**
- 驱动目录： drivers/[ sub-systems ]/**sprd/**

### 文件名 (File Name)

#### **.dts/.dtsi文件**
基本格式:  公司名 + [ 平台名 | 芯片名 ] + [ 板名 ].[ dtsi | dts ]
 
> 例如：
> ![dt-files- relationship](https://raw.githubusercontent.com/sprdlinux/guideline/master/images/dt-relations.png)

**sprd-shark.dtsi**：描述shark平台所共有的硬件资源，如cpu, interrupt-controller, uarts, etc.
**sprd-sc7730.dtsi** ：首先包含sprd-shark.dtsi中的内容，描述(override) sc7730这款芯片的差异，比如cp的配置资源等。
**sprd-sp7730gga-openphone.dts**：代表基于上述芯片的最终手机成品，在包含上述2个文件的基础上，补充描述如内存配置，pinctrl-mux，端口使能，以及在板外设等。 

#### .c/.h/.S文件

基本格式:  [ 公司名 | 平台名 ] + 功能 + [ 芯片名 | 板名 ].[ c | h | S ]

-  功能与体系架构无关，或存在于所有平台，且又为我司独有，使用 **公司名+功能**
> 例:  sprd-debug.c

- 功能仅存在于某一平台中，使用 **平台名+功能**
> 例:  sound/soc/sprd/shark-codec.c

- 功能仅存在于某一芯片，使用 **功能+芯片名**
> 例：drivers/pinctrl/pinctrl-sc7730.c

- 片外外设或片内外设IP非sprd独有，应按其芯片型号或厂商名称独立命名；如kernel upstream中已有该厂商文件，应遵从其模式，不应在sprd目录下另行划分。

- 特别的，如某子系统有其特有文件命名惯例，则遵循其惯例。

- 原则上，当很多名字接近文件在同一目录时，重复的字段应优先用作前缀。
> 例： 
>  good              |  bad
>  ------------------|  --------------
>  pinctrl-sc7730.c   | sc7730-pinctrl.c
>  pinctrl-sc7731.c   | sc7713-pinctrl.c

### 函数名 (Function Name)

- 函数名均由小写字母、下划线构成，禁止花体、驼峰体、大小写混排、以及匈牙利命名法。

- 函数名应望文知意，简洁易懂，不宜使用过长或过于生癖的单词。适当使用缩写，但缩写形式如有惯例应遵循，不宜自创。

- 模块内函数如无需其他模块引用，应定义为static类型；导出函数(EXPORT)应加**sprd_**前缀，以防污染命名空间。

- 以下划线'__'开头的函数名仅在确有必要时使用。常见情况是用来实现对同一函数多种调用方式，比如实现缺省参数；或是表示和体系架构相关的底层实现，比如__raw_readl()。

### 变量名 (Variable Name)

- 变量名规则与函数命名规则相同，但变量名应普遍短于函数名。

- 局部变量尽量使用一个单词达意。但同时尊重命名习惯，比如常用i,j命名for循环变量。

### 宏名 (Macro Name)

- 通常使用全部大写字母和下划线构成，对函数的宏重命名使用小写。

- 名称选取和函数一致，简洁易懂为宜。

- Kconfig宏应联用大写的 **公司名 + [ 平台名 | 芯片名 | 板名 ]** 做区分。

> 例1：公司所有平台共有 **ARCH_SPRD**，**SPRD_MEM_POOL**
> 例2：某一平台特有 **ARCH_SPRD_SHARK**，**SPRD_SHARK_DMC_FREQ**
> 例3：某一芯片特有 **ARCH_SPRD_SC7730**，**SOUND_SPRD_SC7730**
> 例4：某型号手机或某类型板卡特有： **MACH_SPRD_SP7730GGA**，**MACH_SPRD_OPENPHONE** 

- 寄存器相关的宏定义应使用与芯片spec描述一致的名称，便于检索。

- 模块内部使用的宏，无需使用SPRD前缀，避免重复罗嗦。如串口驱动模块中对寄存器偏移量的宏定义。

### 作者名 (Author Name)

- 名在前，姓在后，首字母大写，中间空格分开。比如Orson Zhai，而非orson.zhai，OrsonZhai或zhai orson。

- 原则上使用护照上姓名的汉语拼音，应避免使用中文，因为可能会在某些终端下显示为乱码。

第二章：文件布局 ( Layout )
-----

兵法讲谋定而后动，编程亦如是。尤其是大型项目，文件布局有如派兵布阵，是项目架构设计的直接体现。文件目录层次安排不是简单将文件归类，而是与整体架构模型关系密切。
Linux kernel的目录层次总体上简洁而清晰，但arch/arm/下的布局曾一度混乱不堪。幸运的是，被Linux掌门人Linus Tovalds毒舌批评之后，ARM kernel社区开始了基于dt(device tree)的大幅度清理，效果立竿见影。
因此本文不再探讨非dt情况——他们都该被扫入历史垃圾堆。

### 硬件描述

    {kernel_path}/arch/arm/boot/dts/
    {kernel_path}/arch/arm64/boot/dts/
    
所有ARM架构的SoC芯片, 板级相关的硬件描述文件都放于此处，包括dts和dtsi。
dts为板级描述，dtsi为SoC或family描述。

    {kernel_path}/arch/arm/boot/dts/include/dt-bindings
    -> {kernel_path}/include/dt-bindings/
  
 和硬件描述相关的头文件，与dtsi不同之处在于其内容多为对dt binding值的宏定义。
 
    {kernel_path}/Documentation/devicetree/bindings/
    
 描述和介绍binding用法用例的文档，必须和dt文件一同提交。
 
### 平台初始化

    {kernel_path}/arch/arm/mach-sprd/

放置32位ARM初始化文件。
ARM64已经取消了这个目录，可以想见arm32之所以保留也是照顾历史兼容性而已。
除了DT_MACHINE_START宏和极少数初始化代码，大部分初始化工作应被分解到drivers和bootloader中。

### 驱动模块

    {kernel_path}/drivers/
    ├── base
    │   ├── power
    │   └── regmap
    ├── block
    │   └── zram
    ├── bluetooth
    ├── bus
    ├── clk
    ├── clocksource
    ├── cpufreq
    ├── cpuidle
    │   └── governors
    ├── devfreq
    ├── dma
    ├── dma-buf
    ├── gpio
    ├── gpu
    │   ├── drm
    ├── hid
    │   ├── i2c-hid
    │   └── usbhid
    ├── hwmon
    │   └── pmbus
    ├── hwspinlock
    ├── i2c
    │   ├── busses
    │   └── muxes
    ├── idle
    ├── iio
    │   ├── adc
    │   ├── common
    │   │   ├── hid-sensors
    │   ├── dac
    │   ├── gyro
    │   ├── light
    │   ├── magnetometer
    │   ├── orientation
    │   ├── temperature
    │   └── trigger
    ├── input
    │   ├── keyboard
    │   ├── misc
    │   ├── mouse
    │   ├── tablet
    │   └── touchscreen
    ├── iommu
    ├── irqchip
    ├── leds
    │   └── trigger
    ├── mailbox
    ├── media
    │   ├── common
    │   ├── i2c
    │   │   └── soc_camera
    │   ├── platform
    │   │   ├── soc_camera
    │   ├── radio
    │   ├── tuners
    │   ├── usb
    │   └── v4l2-core
    ├── memory
    ├── mfd
    ├── misc
    ├── mmc
    │   ├── card
    │   ├── core
    │   └── host
    ├── mtd
    │   ├── chips
    │   ├── devices
    │   ├── lpddr
    │   ├── maps
    │   ├── nand
    │   ├── spi-nor
    │   └── ubi
    ├── net
    │   ├── wireless
    ├── nfc
    ├── oprofile
    ├── phy
    ├── pinctrl
    ├── platform
    ├── power
    │   ├── avs
    │   └── reset
    ├── powercap
    ├── pwm
    ├── rapidio
    │   ├── devices
    │   └── switches
    ├── regulator
    ├── reset
    ├── rtc
    ├── soc
    ├── spi
    ├── thermal
    ├── tty
    │   ├── serial
    ├── usb
    ├── vhost
    ├── video
    │   ├── backlight
    │   ├── console
    │   ├── fbdev
    │   │   ├── core
    │   └── logo
    ├── virt
    ├── virtio
    ├── watchdog


- 对应与dt硬件描述中的节点，每个驱动文件各归其位。
- 对于soc特有一些驱动，可考虑新近增加的 drivers/soc目录。
- 实在找不到合适目录放置的驱动，比如ion，可参考drivers/staging/目录，自行划分一个android目录存放。一旦社区决定最终的位置，应按upstream修改。

### 头文件

    {kernel_path}/include/

此目录用于存放需要跨模块引用的函数声明、宏定义等头文件。
换言之，如果不是暴露给整个内核调用的公共函数，不要在这里放置。
模块私有的头文件，应该和各模块c文件放在一起。

### 固件

    {kernel_path}/firmware/
    
有些固件没有开放代码，无法通过编译得到。
这个目录并不是正式存放固件的位置（不符合GPL约定）。
但没有这些固件，某些功能将无法使用。
作为权宜之计，我们可以把固件二进制文件放于此处。

第三章：DT描述 ( Device Tree ) 
-----

嵌入式系统 (Embedded System)在刚出世的时候，没人能预见到他们会象现在这么庞大和复杂。当时的系统功能简单到可以由一名普通工程师在1天内完全用汇编语言实现。
而现在一片普通的ARM9 SoC就可能包含数十个外设IP。 因为是片内外设，且没有类似PnP的机制来自动侦测，所有的外设资源信息都需要在软件中描述。
在DT机制出现之前，上述工作可以形容为一场灾难——数以千计的长宏和层层嵌套的头文件足以令人抓狂。
DT则改观了这一切，他用类似Jason语言的形式，将SoC和板级外设表示为节点拓扑。易读，结构清晰、灵活。

- 关于DT规则的详述，请参考官方文档：Power.org Standard for Embedded Power Architecture Platform Requirements (ePAPR)

- 内部文档请参考：_Spreadtrum Device Tree_ <i class="icon-file"></i>[sprd_device_tree_updated_0924.doc] , written by Allen & Mark

- Binding文档请参考：<i class="icon-folder"> '''{kernel_path}/Documentation/devicetree/bindings/'''

一般性内容，本章不再赘述，仅针对一些具体现象和问题进行探讨。

### 总线拓扑结构

dts是对硬件的描述，由设备节点构成。其脉络一般按片内/片外总线拓扑展开。
当前写法dev node都归在root下，如果芯片简单也不无不可，但我司芯片寄存器规划比较独特，同族芯片的继承关系各具风格，从维护角度应遵循惯例为宜。

    /include/ "skeleton.dtsi"
    
    / {
    	#address-cells = <1>;
    	#size-cells = <1>;
    
      	soc {    /* 片内外设的父节点 */
    		compatible = "simple-bus";  /* 声明simple-bus，公共代码会遍历节点并注册设备 */
    		reg = <0x0 0x80000000>;  /* 声明地址范围，与手册memory map呼应，读者了然于心 */
    		ranges;
    
    		ap_ahb: ahb@20000000 {   /* ahb较多，通过地址区分；利用label释义 */
    			compatible = "simple-bus";
    			reg = <0x20000000 0x1900000>;
    			ranges;
    
    		}
    		pub: memory-controller@30000000 {   /* memory-controller是保留名称，此节点为ddr控制器 */
    			compatible = "sprd, shark-dmc";
    			reg = <0x30000000 0xe0000>;  
    		}
    
    		aon_apb: apb@40000000 {  /* apb有多个，通过地址分别，label释义 */
    			compatible = "simple-bus";
    			reg = <0x40000000 0x420000>;
    			ranges;
    
			    /* 以下为aon_apb下的设备节点 */
    
    			syscnt: timer@40230000 {
    				compatible  = "sprd,shark-syscnt";
    				reg = <0x40230000 0x1000>;
    				interrupts = <0 118 0x0>,
    			};
    
    			aon_timer: timer@40050000 {
    				compatible = "sprd,shark-timer";
    				reg = <0x40050000 0x1000>;
    				interrupts = <0 28 0x0>;
    			};
    
    			timer0: timer@40220000 {
    				compatible = "sprd,shark-timer";
    				reg = <0x40220000 0x1000>;
    				interrupts = <0 29 0x0>;
    			};
    
    			timer1: timer@40320000 {
    				compatible = "sprd,shark-timer";
    				reg = <0x40320000 0x1000>;
    				interrupts = <0 119 0x0>;
    			};
    
    			timer2: timer@40330000 {
    				compatible = "sprd,shark-timer";
    				reg = <0x40330000 0x1000>;
    				interrupts = <0 121 0x0>;
    			};
    		}
    		
    		gpu: apb@60000000 {     /* 注意寄存器地址按从小到大顺序排列，按芯片arch分层列出 */
    			compatible = "simple-bus";
    			reg = <0x60000000 0x200000>;
    			ranges;
    
    		}
    
    		mm: ahb@60800000 {
    			compatible = "simple-bus";
    			reg = <0x60800000 0x700000>;
    			ranges;
    			
    			csi: csi-controller {
    				compatible = "sprd,shark-csi";
    				reg = <0x60c00000 0x100000>;
    			}
    
    		}
    
    		ap_apb: apb@70000000 {
    			compatible = "simple-bus";
    			reg = <0x70000000 0x10000000>;
    			ranges;
    
			    /* 以下为0x70000000地址段设备 */
			    
    			uart0: serial@70000000 {
    				compatible  = "sprd,shark-serial";
    				interrupts = <0 2 0x0>;
    				reg = <0x70000000 0x1000>;
    				clocks = <&clock 60>;
    			};
    
    			uart1: serial@70100000 {
    				compatible  = "sprd,shark-serial";
    				interrupts = <0 3 0x0>;
    				reg = <0x70100000 0x1000>;
    				clocks = <&clock 61>;
    			};
    
    			uart2: serial@70200000 {
    				compatible  = "sprd,shark-serial";
    				interrupts = <0 4 0x0>;
    				reg = <0x70200000 0x1000>;
    				clocks = <&clock 62>;
    			};
    
    			uart3: serial@70300000 {
    				compatible  = "sprd,shark-serial";
    				interrupts = <0 5 0x0>;
    				reg = <0x70300000 0x1000>;
    				clocks = <&clock 63>;
    			};
    
    			i2c0: i2c@70500000 {
    				compatible = "sprd,shark-i2c";
    				interrupts = <0 11 0x0>;
    				reg = <0x70500000 0x1000>;
    				#address-cells = <1>;
    				#size-cells = <0>;
    			};
    
    			i2c1: i2c@70600000 {
    				compatible = "sprd,shark-i2c";
    				interrupts = <0 12 0x0>;
    				reg = <0x70600000 0x1000>;
    				#address-cells = <1>;
    				#size-cells = <0>;
    			};
    
    			i2c2: i2c@70700000 {
    				compatible = "sprd,shark-i2c";
    				interrupts = <0 13 0x0>;
    				reg = <0x70700000 0x1000>;
    				#address-cells = <1>;
    				#size-cells = <0>;
    			};
    
    			i2c3: i2c@70800000 {
    				compatible  = "sprd,shark-i2c";
    				interrupts = <0 14 0x0>;
    				reg = <0x70800000 0x1000>;
    				#address-cells = <1>;
    				#size-cells = <0>;
    			};
    
    			spi0: spi@70a00000 {
    				compatible  = "sprd,shark-spi";
    				interrupts = <0 7 0x0>;
    				reg = <0x70a00000 0x1000>;
    				clock-names = "clk_spi0";
    			};
    			spi1: spi@70b00000 {
    				compatible  = "sprd,shark-spi";
    				interrupts = <0 8 0x0>;
    				reg = <0x70b00000 0x1000>;
    				clock-names = "clk_spi1";
    			};
    			spi2: spi@70c00000 {
    				compatible  = "sprd,shark-spi";
    				interrupts = <0 9 0x0>;
    				reg = <0x70c00000 0x1000>;
    				clock-names = "clk_spi2";
    			};
    		}
    
    	}
    };

以上写法是笔者读完sc9630 Arch文档后，据其整理而成。可以看出，ic架构图中的模块布局，在dt中有直接对应，日后增删节点、芯片继承改写，都方便查找对比。

### 命名

- 节点命名
1. 节点名用来表示此节点的设备类型，而非其他。用词应靠近kernel device class name。
    > - Pro:  **memory-controller** { ... }
    > - Cons:  
    > -- **public** { ... }      设备功能描述不清，区域模块，可用做别名。
    > -- **shark-dmc** { ... }   太过具体，更适合用在compatible string中。
    
      
2.  ePAPR 2.2.2章节中，有列出常用设备节点名称，如果属于此列表中名称，应优先选用
    > - Pro:  **serial@70000000** { ... }  serial为2.2.2节保留字
    > - Con:  **uart@70000000** { ... }  uart没什么不好，但应尊重惯例

3. @后缀地址应有意义，惯例和属性reg中的第一个地址相同
    > - Pro:  **i2c@80** { reg = <**0x80**, 0x10>; ... }
    > - Cons:  
    > -- **i2c@1** { reg = <**0x80**, 0x10>; ... }
    > -- **i2c_1** { ... }  可用于label，在alias中引用
    
4. 在Linux kernel中，如设备为厂商特有，应加厂商前缀，用逗号隔开，并撰写对应binding-Doc.
    > - Pro:  **sprd,adi** { ... }
    > - Cons:
    > -- **adi** { ... }
    > -- **sprd_adi** { ... }  

- compatible string命名

 1. 多个compatible strings，从左至右，从最精确匹配到最宽松。
    > 如:  compatile = "sprd,**sc7731**", "sprd,**shark**", "arm,cortex-a7";
         
2.  建议以芯片型号区分不同的ip。
    > 如:  
    > **compatile = "sprd,sc7731-uart";**
    > **compatile = "sprd,sc9630-uart";**

3. 尽量采用公共string，利用公共代码处理。
    > - Pro:  compatible = "**fixed-clock**";
    > - Con:  compatible = "**sprd,fixed-clock**";    
        
### 文件分层

- dt文件可以层层包含，尽可能复用描述
> 如:  
>  - shark.dtsi —— shark族片内基本节点
>  - sc2723.dtsi —— A-die ip节点描述
>  - sc7731.dtsi —— 芯片arm核描述，特有ip节点，和override属性
>  - sp7731gga.dts —— 板级描述，包括片外节点，和override属性

- 同一个节点路径，后面可override前面的属性。
    
        /* shark.dtsi 芯片文件 */
	    / { 
    	 cpus {
    		uart0: serial@70000000 {
    			clock-frequency = <40000000>;
    		 };
    	 };
    	};
    
    -------	
	     /* sp7731gea.dts** 板级文件 */
	     
	     /dts-v1/; 
	     /include/ "shark.dtsi"
	     &uart0 {
	         clock-frequency = <26000000>;
	     };

最终编译的dtb中, clock-frequency是26000000。

### 属性binding

- 私有属性增加要谨慎，绝非必要，不应随意添加。属性应该是描述ip外围的资源配置情况，内部的资源应该是在驱动内部绑定的。

        scproc: scproc@0x50800000 {
            compatible = "sprd,scproc";
            sprd,name = "cppmic";   /* 这是纯软件的概念，和硬件没有什么关系吧？ */
            sprd,ctrl-reg = <0x114 0xff 0xb0 0xff>; /* 这个用宏定义在驱动程序中可以吗？ */
            sprd,ctrl-mask = <0x01 0xf0000 0x100 0xf0000>; /* 用宏定义在驱动程序中？ */
            sprd,iram-data = <0xe59f0000 0xe12fff10 0x0>;  /* 用 memory-region属性 */
            reg = <0x50800000 0x8000>,
            interrupts = <0 0x7c 0x0>; 
        };

如此精简后，其实没有一个真正需要定制的私有属性。

- 每增加一个属性，都要在配套binding文档中说明，并与代码同步提交。文档中不仅要讲述binding含义，还要写出用例。
> 文档目录位置：{Kernel_src}/Documentation/devicetree/bindings/

- 替代方案

1. **编译宏**：比如软件策略、字符串等。比如上例中的模块名字静态配置应该用编译宏，可以在menuconfig中填写。
2. **动态配置**：可以用module/kernel parameter，通过bootloader传过来。
dt有一个特定节点bootargs { ... }，可用于存放上述参数

>   特别的，如果配置数据(configuration data)确实需要，也应加sprd前缀，如 sprd, config = < .... >;

### aliases的妙用

利用of_alias_get_id()函数获取设备id号。
**Pro**

    alias {
		    sdhci1 = &sdhci1;
		    ....
	}

 **Con**
 
         sdhci1: sdhci@f5118000{
                 reg = <0xf5118000 0x1000>;                
                 id = <1>;


### 虚拟设备

- 纯软件的虚拟设备不应写在dts中

> 如: debug_node { .... }

- 组合设备(composite device) 社区可能会接受。

> 如: battery_charger { ... }
> 可能会涉及比如 adc, gpio, timer多个节点。

battery-charger {
    .....
              timers {
                    polling = < &ap_t0_timer0 >;
              }
              adc {
	                channel = < &adc2, 3 >;
              }
              gpio {
                    led = < &gpio6 >
    .....
}

### Interrupt controller
### clock
Cons:

	 clk_mcu: clk_mcu {
             compatible = "sprd,composite-dev-clock";
             #clock-cells = <0>;
             reg = <0x20e00054 0x7 0x20e00038 0x70>; 
             /* select reg and divider reg */

### power management

第四章：子系统 ( Sub-system & Module )
-----

### 基本输出 ( stdout )
### pinctrl
### 时钟
### 中断
### 内存控制
### DMA
### 电源管理
#### PSCI
### Framebuffer
### 按键和LED
### 多媒体
### 全局寄存器问题
### sipc

第五章：ARM64兼容
-----

### 初始化
### DT
### ABI

第六章：工具 ( Tools )
-----

第七章：社区 ( Community )
-----





--
> Created by Orson Zhai (orson.zhai@gmail.com)
> Written with [StackEdit](https://stackedit.io/).