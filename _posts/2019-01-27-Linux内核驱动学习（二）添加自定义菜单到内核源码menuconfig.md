@[toc]
## 目标

Kernel：Linux 4.4

我编写一个简单的`hello world`Linux 内核模块后，已经可以通过`insmod`动态加载到系统内核中，并通过`rmmod`卸载模块。但是出于学习的目的，我想把这个内核添加到**Linux源码**中，并且可以通过`Kconfig`进行配置，在`make menuconfig`的指令下，可以生成相应的菜单可以进行配置。

## drivers/Kconfig

在Linux内核源码路径下，可以找到`drivers`文件夹路径，这里保存的是各种驱动程序。在终端输入`make menuconfig`可以在终端上显示一个用户界面能对内核进行相应的配置。

`Device Drivers` 就是驱动的配置选型的菜单。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190127171959280.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)

将光标移动到`exit`退出当前界面，返回到最初的终端。

```shell
sudo cd drivers
sudo mkdir demo
vi Kconfig # 或者使用 gedit Kconfig
```

在`drivers`路径下创建一个文件夹`demo`，打开`Kconfig`，把**drivers/demo/Kconfig**的文件路径添加到`drivers/Kconfig`中

```shell
menu "Device Drivers"
source drivers/demo/Kconfig //只需要添加这一行
endmenu
```



## demo下的Kconfig 和 Makefile

在`demo`下添加`Kconfig`和`Makefile`，当前`demo`下的文件列表

```shell
├── demo_gpio.c
├── Kconfig
├── Makefile
```

### Kconfig

```shell
menuconfig DEMO_DRIVERS #DEMO_DRIVERS 菜单
    tristate "demo drivers" 
    help
        demo

if DEMO_DRIVERS 
config DEMO_PLATFORM_GPIO # 菜单子项
    tristate "the most simplest driver"	#子项显示内容
    help
        Driver learning # help显示内容        
endif

```



### Makefile

```shell
obj-$(CONFIG_DEMO_PLATFORM_GPIO) +=demo_gpio.o
```



### demo_gpio.c

```c
#include <linux/init.h>
#include <linux/module.h>

static int __init demo_gpio_init(void){
        printk(KERN_INFO "demo gpio module init\n");
        return 0;
}

module_init(demo_gpio_init);

static void  __exit demo_gpio_exit(void){
        printk(KERN_INFO "demo gpio module exit\n");
}

module_exit(demo_gpio_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("gw");

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190127172041190.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190127172100421.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
最后，可以看到，`demo_gpio`已经成功添加到menuconfig的里了。


