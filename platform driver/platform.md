### platform driver modle

Linux在2.6版本引入了设备驱动模型，设备驱动模型负责统一实现和维护一些特性，诸如：热插拔、对象生命周期、用户空间和驱动空间的交互等基础设施。

设备驱动模型主要包含：类（class）、总线（bus）、设备（device）、驱动（driver），它们的本质都是内核中的几种数据结构的“实例”:驱动的本质是device_driver结构体类型，各种不同的驱动其实就是device_driver的各种实例。

总线、设备、驱动，这三者有着密切的联系。在内核中，设备和驱动是分开注册的。而总线就是连接设备和驱动之间的纽带。这么做是为了

* 让算法和数据分离，驱动源码中不携带数据，只负责算法（对硬件的操作方法）；而设备则负责携带硬件信息。
* 最大程度保持驱动的独立性和适应性，并且可以实现一个驱动对应多个设备。

### platform总线

设备驱动模型下，总线是连接设备与驱动的纽带。对于依附于USB\I2C\SPI总线的设备来说不是问题（有爹有妈的孩子不愁没对象）。但是某些设备没有依附bus,无法基于设备驱动模型（享受不了福利）。内核发明了虚拟的平台总线platform（相当于珍爱网），挂接在platform上的设备和驱动。称为platform device（会员） 和 platform driver（老司机）。

流程：
* 提供并注册platform device/设备节点
* 提供并注册platform driver
* platform bus内的match函数会不停的匹配driver和device(根据driver中的of_match_table中的compatible)
* 一旦匹配成功，调用driver的probe函数正式执行drive中的步骤

### platform总线驱动的独立性

一个platform总线驱动程序可以对应多个设备，设备的变化不会影响驱动。

实现机制：

* 设备将底层信息（设备名称、使用的中断号、寄存器信息）传递给驱动，驱动代码保持不变，只需要根据参数操作底层，就可以适应设备的变化。
* 现代驱动设计理念是算法和数据分离，驱动源码不携带数据，只负责算法（对硬件操作），以此保持驱动的独立性和适应性。
* 老内核的platform包含一个device的结构体，内部有一个void *platform_data，保存设备底层信息。当driver中probe()函数执行时，platform_device会作为参数传递进去，驱动间接操作void *platform_data，从而操作硬件。新内核直接在设备节点属性中存放数据，驱动通过API读取节点的数据。

首先应该进入mach-xxx.c完成platform设备的注册。先来看看mach-xxx.c中的情况，如果我们要写新的platform_device，要注意mach-xxx.c内有没有重复功能的。在该mach-xxx.c搜寻“platform”，寻找专门存放platform_device的数组，发现里面并没有led，看来我们要自己从头开始写了。

* 第一步：创建适用于我们设备的platform_data类型
* 第二步：为一个具体设备实例化一个platform_data，用来存放该类设备的底层信息
* 创建一个具体platform设备（实例化一个platform_device），并把各种信息和platform_data填充入该设备
* 把platform设备丢到专门存放platform_device的数组中，开机时系统会注册数组中所有设备



```ruby
/*sjh_add*/

/*第一步：创建一个适用于我们设备的platform_data类型*/
struct s5pv210_led_platdata {
    unsigned int         gpio;
    unsigned int         flags;
    char            *name;
    char            *def_trigger;
};

/*第二步：为一个具体设备实例化一个platform_data*/
static struct s5pv210_led_platdata x210_led1_pdata = {
    .gpio       = S5PV210_GPJ0(3),
    .flags      = NULL,
    .name       = "led1",
    .def_trigger    = NULL,
};

/*第三步：实例化一个platform_device，正式创建设备*/
static struct platform_device x210_led1 = {
    .name       = "s5pv210_led",//要和platform驱动中的名字对应
    .id     = 1,
    .dev        = {
    .platform_data  = &x210_led1_pdata,//底层信息
    },
};

/* 第四步：把我们的platform_device添加进数组，开机时系统会注册数组中所有设备*/
static struct platform_device *smdkc110_devices[] __initdata = {
/*sjh_add*/
    &x210_led1,
#ifdef CONFIG_FIQ_DEBUGGER
    &s5pv210_device_fiqdbg_uart2,
#endif

...
}
```
如果需要添加多个led设备，重复实例化platform_data和platform_device。由于驱动和设备之间通过name匹配，所以不同的led设备name保持一致，但id不同。

驱动部分代码：
```ruby
extern struct s5pv210_led_platdata {
    unsigned int         gpio;
    unsigned int         flags;
    char            *name;
    char            *def_trigger;
};

static int s5pv210_led_probe(struct platform_device *pdev)
{
        /*导入自留地格式s5pv210_led_platdata，这样我们才能解析参数*/
        struct s5pv210_led_platdata *pdata = pdev->dev.platform_data;
        int ret = -1;

        /*这是一个例子，我们如何通过传入的参数获得led的gpio编号信息*/
        gpio_num = pdata->gpio;

        /*各种注册、初始化操作*/
        ...

        return 0;
}

/*卸载模块将触发remove函数*/
static int s5pv210_led_remove(struct platform_device *pdev)
{
        struct s5pv210_led_platdata *pdata = pdev->dev.platform_data;

        /*各种注销操作*/
    return 0;
}

/*定义我们的platform_driver。注意name要和platform_device中相同*/
static struct platform_driver s5pv210_led_driver = {
    .probe      = s5pv210_led_probe,
    .remove     = s5pv210_led_remove,
    .driver     = {
        .name       = "s5pv210_led",
        .owner      = THIS_MODULE,
    },
};

/*模块与卸载加载函数，在里面分别添加platform驱动的注册和卸载函数*/
static int __init s5pv210_led_init(void)
{
        return platform_driver_register(&s5pv210_led_driver);
}

static void __exit s5pv210_led_exit(void)
{
        platform_driver_unregister(&s5pv210_led_driver);
}

module_init(s5pv210_led_init);
module_exit(s5pv210_led_exit);

//模块描述信息
MODULE_LICENSE("GPL");
MODULE_AUTHOR("taurenking");
MODULE_DESCRIPTION("S5PV210 LED driver");
MODULE_ALIAS("S5PV210 LED driver");         
```

### 新内核下platform总线驱动的编写方法

对于platform bus驱动而言，新内核的改变在于：

platform device不再需要在mach-xxx中注册，而是直接以节点形式定义在设备树dts中。比如imx6dl-hummingboard.dts内的ir-receiver设备

```ruby
/ {
    model = "SolidRun HummingBoard DL/Solo";
    compatible = "solidrun,hummingboard", "fsl,imx6dl";

    ir_recv: ir-receiver {
        compatible = "gpio-ir-receiver";
        gpios = <&gpio1 2 1>;
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_hummingboard_gpio1_2>;
    };
```

驱动程序将直接和设备树的节点进行配对，具体方法是在platform driver中定义一个of_match_table，只要里面的compatible与设备节点的compatible相同，那么就触发probe函数。

有关设备的私有数据，新内核不再使用platform_data，而是直接在节点中定义各种属性，在驱动中调用特定API获取。
参考：
* [设备驱动模型与sysfs](http://blog.csdn.net/qq_28992301/article/details/52381868)
* [基于platform总线的驱动分析](http://blog.csdn.net/qq_28992301/article/details/52385518)