---
layout: post
tags: [Linux驱动]
comments: true
---

## 前言
前面是如何操作`GPIO`进行输出，这里我重新实现了一个`gpio`的驱动，可以获取外部信号的输入。[gpio-demo.c](https://github.com/hotsauce1861/linux-driver-learning/blob/master/gpio/gpio-demo.c)中已经包括检测一个`gpio`的信号，并且包含了中断和轮询两种方式，可以通过设备树里的`mode`属性进行选择。
## 设备树
本文检测的输入引脚是`GPIO3_D0`，具体的设备树如下所示；
```c
gpio-demo {
		compatible = "gpio-demo";
		input-gpio = <&gpio3 RK_PD0 GPIO_ACTIVE_LOW>;
		mode = <1>; // 0:poll 1:interrupt
		poll_time = <1000>; //ms		
};
```
- `compatible`：设备兼容属性为`gpio-demo`，与后面的驱动代码中的
`gpio_demo_of_match[] = { { .compatible = "gpio-demo"}, {}, }` 需要相同；
- `input-gpio`：这个属性值通过`of_get_named_gpio`来获取；
- `mode`：用于判断当前的工作模式是轮询还是中断；
- `poll_time`：轮询模式下的周期，间隔多少毫秒会读取一次`gpio`的状态；

对于设备树的解析，单独封装了一个接口；
```c
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
```
## 两个结构体
### gpio_platform_data
`gpio_platform_data`主要是对设备树中众多属性的封装；
```c
struct gpio_platform_data {
	int mode;
	int count;
	int gpio_index; 
	struct mutex mtx;
	int poll_ms;
};
```
### gpio_demo_device
`gpio_demo_device`是与设备驱动中相关资源的封装，包括工作队列等等；
```c
struct gpio_demo_device {
	struct platform_device *pdev;
	struct device *dev;
	struct gpio_platform_data 	*pdata;
	struct workqueue_struct		*gpio_monitor_wq;
	struct delayed_work gpio_delay_work ;
	int gpio_irq;
};
```
## 两种方式
在驱动的`probe`函数中，先通过`gpio_parse_data`解析设备树文件，从而获取`mode`属性的值：
- `0`：`gpio_demo_init_poll`初始化进入轮询工作模式；
- `1`：`gpio_demo_init_interrupt`初始化进入中断工作模式；
```c
static int gpio_demo_probe(struct platform_device *pdev){
	...
	ret = gpio_parse_data(priv);
	if (ret){
		dev_err(dev,"parse data failed\n");
	}
	...
	if (priv->pdata->mode == 0){
		gpio_demo_init_poll(priv); //轮询
	} else {
		gpio_demo_init_interrupt(priv);//中断
	}
}
```
### 轮询
在轮询工作模式下，已经通过`gpio_demo_init_poll`对工作队列进行初始化，之后，后启动运行`gpio_demo_work`任务，并在规定的调度时间内，重复检测运行这个任务。
通过`gpio_get_value(gpio_index)`读取`GPIO3_D0`上的电平状态，如果需要对边沿信号进行处理还需要做改动，本文只能对电平信号进行处理。
```c
static void gpio_demo_work(struct work_struct *work) {

	struct gpio_demo_device *di = container_of(work,
			     struct gpio_demo_device,
			     gpio_delay_work.work);

	struct gpio_platform_data *padta = di->pdata;
	int gpio_index,value;
	//获取gpio索引号
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
```
### 外部中断
中断的申请和初始化在`gpio_demo_init_interrupt`函数中已经实现，如下所示；
通过`gpio_to_irq`接口获取相应`GPIO`上的软件中断号，然后通过`devm_request_irq`申请中断；
```c
static int gpio_demo_init_interrupt(struct gpio_demo_device *di) {
	...
	// 获取gpio上的中断号
	irq = gpio_to_irq(gpio_index);
	...
	//申请中断
	ret = devm_request_irq(di->dev, irq, gpio_demo_isr, 
					IRQF_TRIGGER_FALLING, //下降沿
					"gpio-demo-isr", //中断名称
					di);
	...
}
```
其中，每次外部发送一个下降沿信号，就会触发中断并进入`gpio_demo_isr`这个中断服务程序；下面来看一下这个`gpio_demo_isr`，在这里可以做一些我们想做的事情；
```c
static irqreturn_t gpio_demo_isr(int irq, void *dev_id)
{
	struct gpio_demo_device *di = (struct gpio_demo_device *)dev_id;
	struct gpio_platform_data *pdata = di->pdata;

	BUG_ON(irq != gpio_to_irq(pdata->gpio_index));
	//TODO 
	dev_info(di->dev, "%s\n", __func__);
	return IRQ_HANDLED;
}
```
最终，我只在中断服务程序中打印了一下串口信息，方便验证。
## 总结
通过这次学习和总结，总体了解了以下几点；
- 通过`delayed_work`对`GPIO`进行轮询操作，后面会再深入学习一下；
- 学习了对于`GPIO`上的中断申请，目前对于中断还是刚好够用的阶段，中断的篇幅较长，可以对其原理做一下学习，还有内核中中断的机制；
- 学习了内核中读取设备树的几个接口；
- 学习了platform设备驱动模型的框架；
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
