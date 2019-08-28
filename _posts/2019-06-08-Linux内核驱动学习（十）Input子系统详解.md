---
layout: post
tags: [Linux驱动]
comments: true
---

## 前言
这次主要会学习Linux中对于输入设备统一封装的框架，在计算机组成原理中，我们可以知道计算机的组成主要分为五个部分：**控制器**，**运算器**，**存储器**，**输入**，**输出**。可见，输入作为其中的一个子系统，但是对于众多的设备来说，需要一套统一的规范。所以，在嵌入式系统中的外设，鼠标、键盘、按键、G-Sensor等等都可以注册为Input设备。Linux在用户层提供了相应的接口读取数据，这里我暂时只介绍在上一篇文章的基础上，如何编写一个`Input驱动`。

## 框架
首先还是先看一下`Input子系统`的整体框架，具体如下图所示；
![框架](https://img-blog.csdnimg.cn/20190514203212297.png)
## 如何实现`input device` 设备驱动？
如何编写`input`驱动的`sample`？在内核文档中已经有详细的介绍，虽然是英文的，但是相当规范，简单，详细，本文就是在此基础上展开，具体可以参考[内核input文档](https://www.kernel.org/doc/html/v4.12/input/input-programming.html)。

**在这里，我们需要在代码中做哪些工作呢？下面将会重点罗列成每一个步骤进行讲解。**

### 头文件
首先需要包含头文件`#include <linux/input.h>`，这个头文件包含了`input`设备的各种定义，结构体以及方法。

###	注册input_dev设备
首先定义一个用来注册`input device`的变量，这里会用到`input_dev`这个结构体。目前的情况是，我把需要的各种变量都封装到`gpio_demo_device `这个结构体中，但是，实际上这里暂时这关心`input_demo`就可以了。
```c
struct gpio_demo_device {

	struct platform_device *pdev;
	struct device *dev;
	struct gpio_platform_data 	*pdata;
	struct workqueue_struct		*gpio_monitor_wq;
	struct delayed_work gpio_delay_work ;
	struct input_dev    *input_demo;

};
```
在这里需要为我们创建的`gpio_demo_device`结构体定义一个变量`priv`，并且使用`devm_kzalloc`为其分配相应的内存。
```c
struct gpio_demo_device *priv;    
priv = devm_kzalloc(dev, sizeof(*priv) , GFP_KERNEL);
```
然后，我们再为`input_demo`分配内存；
```c
priv->input_demo = devm_input_allocate_device(priv->dev);
```
初始化`input_demo`；
```c
// kernel/include/uapi/linux/input-event-codes.h
priv->input_demo->name = "input-demo";
priv->input_demo->dev.parent = priv->dev;
priv->input_demo->evbit[0] = BIT_MASK(EV_KEY); //事件类型注册为EV_KEY
priv->input_demo->keybit[BIT_WORD(KEY_VOLUMEDOWN)] = BIT_MASK(KEY_VOLUMEDOWN); //按键值
```
`name`：当前初始化的`input device`的名字；
`evbit[0]`：配置为`BIT_MASK(EV_KEY)`，将事件类型注册为`EV_KEY`类型，具体有哪些类型如下所示；
所有定义都在`kernel/include/uapi/linux/input-event-codes.h`文件中。
除了`EV_KEY`之外，还有两种基本事件类型：`EV_REL`和`EV_ABS`。它们用于设备提供的相对值和绝对值。相对值可以是例如X轴上的鼠标移动。鼠标将其报告为与上一个位置的相对差异，因为它没有任何绝对坐标系可供使用。绝对事件即操纵杆和数字化仪 - 在绝对坐标系中工作的设备。
>/*
> Event types
 >*/
>#define EV_SYN			0x00
#define EV_KEY			0x01
#define EV_REL			0x02
#define EV_ABS			0x03
#define EV_MSC			0x04
#define EV_SW			0x05
#define EV_LED			0x11
#define EV_SND			0x12
#define EV_REP			0x14
#define EV_FF			0x15
#define EV_PWR			0x16
#define EV_FF_STATUS		0x17
#define EV_MAX			0x1f
#define EV_CNT			(EV_MAX+1)

`keybit[BIT_WORD(KEY_VOLUMEDOWN)]`：按键值设置为`BIT_MASK(KEY_VOLUMEDOWN)`，就是音量减的按键；
>BITS_TO_LONGS(), BIT_WORD(), BIT_MASK()
These three macros from bitops.h help some bitfield computations:
BITS_TO_LONGS(x) - returns the length of a bitfield array in longs for x bits
BIT_WORD(x)      - returns the index in the array in longs for bit x
BIT_MASK(x)      - returns the index in a long for bit x

完成初始化之后，然后注册`input_demo`；
```c
ret = input_register_device(priv->input_demo);
```

### 上报按键值
当用户发生了向系统输入一定信息的操作之后，`input device`需要将一些列信息上传，本文需要上传按键值。在这里要可以通过**中断触发**，或者**轮询**去检测用户的动作，前面已经有提及，这里不再赘述，
本文代码已经在附录中，已经涵盖这两种方式。
关于上报数据可以通过`input_report_key`和`input_sync`去完成。

### dev->open()和dev->close()
如果驱动程序必须重复轮询设备，因为它没有来自它的中断，并且轮询太昂贵而无法一直进行，或者如果设备中断资源比较稀缺，可以使用open和close回调来通知设备何时可以停止轮询或释放中断以及何时必须重新开始轮询或再次获取中断。为此，我们将以下代码添加到示例驱动程序：
```c
static int button_open(struct input_dev *dev)
{
        if (request_irq(BUTTON_IRQ, button_interrupt, 0, "button", NULL)) {
                printk(KERN_ERR "button.c: Can't allocate irq %d\n", button_irq);
                return -EBUSY;
        }
        return 0;
}

static void button_close(struct input_dev *dev)
{
        free_irq(IRQ_AMIGA_VERTB, button_interrupt);
}

static int gpio_demo_probe(struct platform_device *pdev)t(void)
{
        ...
        button_dev->open = button_open;
        button_dev->close = button_close;
        ...
}
```
>请注意，输入内核会跟踪设备的用户数，并确保仅在第一个用户连接到设备时调用`dev->open()`，并在最后一个用户断开连接时调用`dev-> close()` 。对两个回调的调用都是序列化的。
`open()`回调应该在成功的情况下返回0，或者在失败的情况下返回任何非零值。`close()`回调（无效）必须始终成功。

### 其他事件类型，处理输出事件
到目前为止的其他事件类型是：
`EV_LED` - 用于键盘LED。
`EV_SND` - 用于键盘蜂鸣声。
它们与按键事件非常相似，但它们朝另一个方向 - 从系统到输入设备驱动程序。如果您的输入设备驱动程序可以处理这些事件，则必须在`evbit`中设置相应的位， 以及回调例程：
```c
button_dev->event = button_event; 

int button_event(struct input_dev * dev，unsigned int type，
                 unsigned int code，int value){ 
        if(type == EV_SND && code == SND_BELL){ 
                outb（value，BUTTON_BELL）; 
                return 0; 
        } 
        return -1; 
}
```
这个回调例程可以从一个中断或一个BH调用（虽然这不是一个规则），因此不能休眠，并且不能花太长时间才能完成。

### 查看input device信息
系统启动之后，可以通过`cat /proc/bus/input/devices`来查看当前系统中已经注册的input device信息，这里可以看到已经注册成功的`input-demo`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190514203440485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)

## 附录
```c
#include <linux/module.h> 
#include <linux/init.h>

#include <linux/platform_device.h>
//API for libgpio
#include <linux/gpio.h>
//API for malloc
#include <linux/slab.h>
//API for device tree
#include <linux/of_platform.h>
#include <linux/of_gpio.h>
#include <linux/of_device.h>
//API for thread 
#include <linux/kthread.h>

#include <linux/delay.h>
#include <linux/mutex.h>
//API for delaywork
#include <linux/workqueue.h>

#include <linux/interrupt.h>
#include <linux/irq.h>

//API for input
#include <linux/input.h>

#define TIMER_MS_COUNTS		1000

// default value of dts 
#define DEFAULT_POLL_TIME	5
#define DEFAULT_MODE		1


struct gpio_platform_data {
	int mode;
	int count;
	int gpio_index; 
	struct mutex mtx;
	int poll_ms;
};


struct gpio_demo_device {

	struct platform_device *pdev;
	struct device *dev;
	struct gpio_platform_data 	*pdata;
	struct workqueue_struct		*gpio_monitor_wq;
	struct delayed_work gpio_delay_work ;
	struct input_dev    *input_demo;

    int gpio_irq;
};

static int gpio_parse_data(struct gpio_demo_device *di){

	int ret;
	struct gpio_platform_data *pdata;
	struct device *dev = di->dev;
	struct device_node *np = di->dev->of_node;

	pdata = devm_kzalloc(di->dev, sizeof(*pdata), GFP_KERNEL);
	if (!pdata) {
		return -ENOMEM;
	}
	di->pdata = pdata;
	// set default value for platform data
	pdata->mode = DEFAULT_MODE;
	pdata->poll_ms = DEFAULT_POLL_TIME * 1000;

	dev_info(dev,"parse platform data\n");

	ret = of_property_read_u32(np, "mode", &pdata->mode);
	if (ret < 0) {
		dev_err(dev, "can't get mode property\n");
	}
	ret = of_property_read_u32(np, "poll_time", &pdata->poll_ms);
	if (ret < 0) {
		dev_err(dev, "can't get poll_ms property\n");		
	}

	pdata->gpio_index = of_get_named_gpio(np,"input-gpio", 0);
	if (pdata->gpio_index < 0) {
		dev_err(dev, "can't get input gpio\n");
	}
	// debug parse device tree data
	dev_info(dev, "Success:mode is %d\n", pdata->mode);
	dev_info(dev, "Success:gpio index is %d\n", pdata->gpio_index);
	return 0;
}

static void gpio_demo_work(struct work_struct *work) {

	struct gpio_demo_device *di = container_of(work,
			     struct gpio_demo_device,
			     gpio_delay_work.work);

	struct gpio_platform_data *padta = di->pdata;
	int gpio_index,value;
	gpio_index = padta->gpio_index;
	if (!gpio_is_valid(gpio_index) ) {
		dev_err(di->dev, "gpio is not valid\n");
		goto end;
	}

	if ( (value = gpio_get_value(gpio_index) ) == 0) {
		dev_info(di->dev,"get value is %d\n",value);
	}else{
		dev_info(di->dev,"get value is %d\n",value);
	}
	end:
	queue_delayed_work(di->gpio_monitor_wq, &di->gpio_delay_work,
			   msecs_to_jiffies(di->pdata->poll_ms));
}

static int gpio_demo_init_poll(struct gpio_demo_device *di) {

	dev_info(di->dev,"%s\n", __func__);

	di->gpio_monitor_wq = alloc_ordered_workqueue("%s",
			WQ_MEM_RECLAIM | WQ_FREEZABLE, "gpio-demo-wq");


	INIT_DELAYED_WORK(&di->gpio_delay_work, gpio_demo_work);
	queue_delayed_work(di->gpio_monitor_wq, &di->gpio_delay_work,
			   msecs_to_jiffies(TIMER_MS_COUNTS * 5));


	return 0;
}

static irqreturn_t gpio_demo_isr(int irq, void *dev_id)
{
	struct gpio_demo_device *di = (struct gpio_demo_device *)dev_id;
	struct gpio_platform_data *pdata = di->pdata;

	BUG_ON(irq != gpio_to_irq(pdata->gpio_index));

    input_report_key(di->input_demo, KEY_VOLUMEDOWN, 1);
    input_sync(di->input_demo);
    input_report_key(di->input_demo, KEY_VOLUMEDOWN, 0);
    input_sync(di->input_demo);

	dev_info(di->dev, "%s\n", __func__);
	//printk("%s\n",__func__);
	return IRQ_HANDLED;
}

static int gpio_demo_init_interrupt(struct gpio_demo_device *di) {
	
	int irq, ret;
	int gpio_index = di->pdata->gpio_index;
	dev_info(di->dev,"%s\n", __func__);

	if (!gpio_is_valid(gpio_index)){
		return -1;
	}

	irq = gpio_to_irq(gpio_index);

	if (irq < 0) {
		dev_err(di->dev, "Unable to get irq number for GPIO %d, error %d\n",
				gpio_index, irq);
		gpio_free(gpio_index);
		return -1;
	}
	ret = devm_request_irq(di->dev, irq, gpio_demo_isr,
					IRQF_TRIGGER_FALLING,
					"gpio-demo-isr",
					di);
	if (ret) {
		dev_err(di->dev, "Unable to claim irq %d; error %d\n",
				irq, ret);
		gpio_free(gpio_index);
		return -1;
	}

	return 0;
}


static int gpio_demo_probe(struct platform_device *pdev){

	int ret;
	struct gpio_demo_device *priv;
	struct device *dev = &pdev->dev;
    
	priv = devm_kzalloc(dev, sizeof(*priv) , GFP_KERNEL);

	if (!priv) {
		return -ENOMEM;
	}
	priv->dev = dev; //important 

	ret = gpio_parse_data(priv);
	if (ret){
		dev_err(dev,"parse data failed\n");
	}
    
    priv->input_demo = devm_input_allocate_device(priv->dev);
    if(!priv->input_demo) {
        dev_err(dev,"Can't allocate input device\n");
        return -ENOMEM;
    }
    // kernel/include/uapi/linux/input-event-codes.h
    priv->input_demo->name = "input-demo";
    priv->input_demo->dev.parent = priv->dev;
    priv->input_demo->evbit[0] = BIT_MASK(EV_KEY);
    priv->input_demo->keybit[BIT_WORD(KEY_VOLUMEDOWN)] = BIT_MASK(KEY_VOLUMEDOWN);

    ret = input_register_device(priv->input_demo);
    if(ret) {
        dev_err(priv->dev, "register input device failed\n");
        input_free_device(priv->input_demo);
        return ret;
    }

    //input_set_capability(priv->dev, EV_KEY, KEY_VOLUMEDOWN);

	platform_set_drvdata(pdev,priv);

	if (priv->pdata->mode == 0){
		gpio_demo_init_poll(priv);
	} else {
		gpio_demo_init_interrupt(priv);
	}
	return 0;
}
#ifdef CONFIG_OF
static struct of_device_id gpio_demo_of_match[] = {
	{ .compatible = "gpio-demo"},
	{},
}

MODULE_DEVICE_TABLE(of,gpio_demo_of_match);
#else
static struct of_device_id gpio_demo_of_match[] = {
	{ },
}
#endif

static struct platform_driver gpio_demo_driver = {
	.probe = gpio_demo_probe,
	.driver = {
		.name = "gpio-demo-device",
		.owner = THIS_MODULE,
		.of_match_table = of_match_ptr(gpio_demo_of_match),
	}
};

static int __init gpio_demo_init(void){
	return  platform_driver_register(&gpio_demo_driver);
}

static void __exit gpio_demo_exit(void){
	platform_driver_unregister(&gpio_demo_driver);
}

late_initcall(gpio_demo_init);
module_exit(gpio_demo_exit);

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Gpio demo Driver");
MODULE_ALIAS("platform:gpio-demo");




```
