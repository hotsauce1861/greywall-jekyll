---
layout: post
tags: [Linux驱动]
comments: true
---

<!-- TOC -->
- [前言](#前言)
- [功能](#功能)
- [如何使用](#如何使用)
- [设备树](#设备树)
- [API](#api)
- [总结](#总结)
<!-- /TOC -->

## 前言
`GPIO(General Purpose Input/Output)`通用输入/输出接口，是十分灵活软件可编程的接口，功能强大，十分常用，`SOC`也非常依赖`GPIO`，在实际应用中几乎都能看到它的影子，在Linux内核驱动的学习中，这部分相对来说也是比较基础的，但是涉及的东西其实相对来说也比较多，感觉还是很有必要学习和总结一下。

## 功能
正如之前所说，`GPIO`是通用输入输出接口，所以，相应的内核驱动中GPIO的基本功能总体可以总结为以下几点： 
- **输出设定电平**：可以根据用户的需要，向驱动写入相应的值（比如1或0）然后`GPIO`输出高低电平(高=1；低=0)；
- **读取输入电平**：可以读取`GPIO`上输入的高低电平；实际的应用比如按键或者其他一些传感器的信号；
- **触发外部中断**：输入信号可以作为中断信号，包括边沿触发，高电平触发，低电平触发等等；

## 如何使用
关于如何使用`GPIO`，从 `Linux 2.6.3x`以后就开始有`gpiolib`库了，大大简化了操作`GPIO`的流程，如何在内核中添加`gpiolib`的支持呢？可以参考下面的做法；
```shell
make menuconfig
```
```shell
Device Drivers  --->
    -*- GPIO Support  --->
        [*]   /sys/class/gpio/... (sysfs interface)
```
究其原理的话，追本溯源可能篇幅会很长，后面再现学现卖，对于单纯使用`GPIO`，感觉应该有以下几个步骤：
- 在你的设备节点里添加`gpios`属性；例如：
`gpios = <0 0 GPIO_ACTIVE_LOW>;`
- 在驱动中解析设备树中的节点`gpios`；
- 调用`gpiolib`的接口可以在驱动中对`gpio`进行操作；
- 设置具体的`gpio`的功能：
	- 设置为输出引脚；
	- 设置为输入引脚；
	- 设置为外部中断；		

## 设备树
相应的设备树可以写成
```c
gpio_keys {
 	compatible = “gpio-keys”;
 	...
	button@1 {
		wakeup-source;
		linux,code = <KEY_ESC>;
		label = “ESC”;
	 	gpios = <&gpio0 0 GPIO_ACTIVE_HIGH>;
 	};
};
```
## API
|函数| 功能 |
|--|--|
| `bool gpio_is_valid(int number)` | 	判断当前`gpio`是否有效 |
| `int gpio_request(unsigned gpio, const char *label)` |  申请`gpio`的资源|
| `void gpio_free(unsigned gpio)` | 释放已经申请的`gpio`资源 |
| `int gpio_direction_input(unsigned gpio)` | 设置为输入 |
| `int gpio_direction_output(unsigned gpio, int value)` |设置为输出  |
| `int gpio_get_value(unsigned gpio)`|  获取输入值|
| `void gpio_set_value(unsigned gpio, int value)`|  设置输出值|
| `int gpio_to_irq(unsigned gpio)`|  获取`gpio`上的中断号|
| `int irq_to_gpio(unsigned int irq)`| 获取中断号对应的`gpio` |
|`int devm_gpio_request_one(struct device *dev, unsigned gpio,unsigned long flags, const char *label)`| 为`gpio`分配外部中断资源|
|`void devm_gpio_free(struct device *dev, unsigned int gpio)`| 释放已经申请的中断资源|
这里先大致介绍一下一般会用到的接口，还有一些遗漏，以后会慢慢补充。

## 总结
这里我简单介绍了`gpio`，罗列了一下Linux操作`gpio`可能会涉及到的知识点，包括需要对在设备树里注册节点，一些`gpiolib`的常用接口，这里还没有给出详细的实现代码，后面会对每一个部分的使用单独进行介绍，包括输入输出，中断唤醒，Input设备等等，篇幅和能力有限，如有错误，不吝赐教。


