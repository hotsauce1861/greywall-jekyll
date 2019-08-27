# Linux platform设备驱动模型

@[toc]
## 前言

为什么要往平台设备驱动迁移？这里需要引入设备，总线，驱动这三个概念。上一篇字符型设备驱动的实现实际将设备和驱动集成到同一个文件中实现，如果这里有**硬件A的驱动**，**硬件B的驱动**，**硬件C的驱动**，然后有三类用户**接口E**，**接口F**和**接口G**，这里用户接口是提供给用户层调用的接口，每一种接口又必须兼容这三种硬件，按照原来的实现方式，为了适配所有的使用需求，理论上会出现**A+E**，**A+F**，**A+G**，**B+E**，**B+F**，**B+G**，**C+E**，**C+F**，**C+G**，这几种实现方式，而表现在代码中的则是

```c
#if A
#elif B
#elif C
#endif
```

当然，目前接口数量和硬件数量不是很庞大的时候，维护上暂时不会造成太大的问题，所以，这里引入了设备/总线/驱动的机制，实现了驱动和设备之间的解耦，这里我的理解是和设计模式中的中介者模式比较相似。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215210856227.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)


## 框架

大致地整理了一下**platform**设备驱动模型的整体框架。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215210840297.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)



## 设备与驱动的分离

### 设备（device）

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/cdev.h>
#include <linux/fs.h>
#include <linux/slab.h>
#include <linux/platform_device.h>

static struct platform_device *character_dev;

static int __init cnc_platform_character_init(void){

	int ret = 0;	
	character_dev = platform_device_alloc("cnc_platform_character", -1);	
	if (!character_dev)		
		return -ENOMEM;	
	ret = platform_device_add(character_dev);	
	if (ret) {		
		platform_device_put(character_dev);	
		printk("\n\n\n\n\n Success platform_device_put(character_dev)\n\n\n\n\n");
		return ret;	
	}
	printk("\n\n\n\n\n Failed platform_device_put(character_dev)\n\n\n\n\n");
	return 0;
}
module_init(cnc_platform_character_init);

static void __exit cnc_platform_character_exit(void){
	printk("%s call\n",__func__);
	platform_device_unregister(character_dev);

}
module_exit(cnc_platform_character_exit);

MODULE_VERSION("1.0");
MODULE_LICENSE("GPL");
```



### 驱动（driver）

```c
#include <linux/init.h>
#include <linux/types.h>
#include <linux/module.h>
#include <linux/cdev.h>
#include <linux/fs.h>
#include <linux/slab.h>
#include <linux/platform_device.h>
#include <linux/miscdevice.h>

#include <linux/of_device.h>

#define DRIVER_DATA_SIZE 	4096

struct cnc_character_st{
	struct cdev device;
	u8	data[DRIVER_DATA_SIZE];
	struct miscdevice miscdev;
};

//TODO
static ssize_t cnc_character_read (struct file * fd, char __user * data, size_t len, loff_t * offset){
	ssize_t ret = 0;
	return ret;
}

//TODO
static ssize_t cnc_character_write (struct file * fd, const char __user * data, size_t len, loff_t * offset){
	ssize_t ret = 0;
	return ret;
}

//TODO
static long cnc_character_unlocked_ioctl (struct file * fd, unsigned int data, unsigned long cmd){
	long ret = 0;
	return ret;
}

//TODO
static int cnc_character_open (struct inode * node, struct file * fd){
	int ret = 0;
	return ret;
}
//TODO
static int cnc_character_release (struct inode * node, struct file * fd){
	int ret = 0;
	return ret;
}


static const struct file_operations cnc_character_ops = {
	.owner = THIS_MODULE,
	.read = cnc_character_read,
	.write = cnc_character_write,
	.open = cnc_character_open,
	.unlocked_ioctl = cnc_character_unlocked_ioctl,
	.release = cnc_character_release,
};


static int cnc_character_probe(struct platform_device *pdev){

	int ret = 0;
	struct cnc_character_st *character_dev;

	character_dev = devm_kzalloc(&pdev->dev, sizeof(*character_dev),GFP_KERNEL);

	character_dev->miscdev.minor = MISC_DYNAMIC_MINOR;
	character_dev->miscdev.name = "cnc_platform_character";
	character_dev->miscdev.fops = &cnc_character_ops;
	//ret = misc_register(&character_dev->miscdev);
	platform_set_drvdata(pdev, character_dev);
	ret = misc_register(&character_dev->miscdev);

	if(ret < 0){
		return ret;
	}
	return 0;
	
}

static int cnc_character_remove(struct platform_device *pdev){
	
	struct cnc_character_st *gl = platform_get_drvdata(pdev);
	printk("%s call\n",__func__);
	misc_deregister(&gl->miscdev);
	return 0;
}

static struct platform_driver cnc_character_driver = {
	.driver = {
		.name = "cnc_platform_character",
		.owner = THIS_MODULE,
	},
	.probe = cnc_character_probe,
	.remove = cnc_character_remove, 
};

module_platform_driver(cnc_character_driver);

MODULE_VERSION("1.0");
MODULE_LICENSE("GPL");
```

### 匹配（match）
函数`static int platform_match(struct device *dev, struct device_driver *drv)`在内核`drivers/base/platform.c`中，其源代码如下：
```c
static int platform_match(struct device *dev, struct device_driver *drv)
{
	struct platform_device *pdev = to_platform_device(dev);
	struct platform_driver *pdrv = to_platform_driver(drv);

	/* When driver_override is set, only bind to the matching driver */
	if (pdev->driver_override)
		return !strcmp(pdev->driver_override, drv->name);

	/* Attempt an OF style match first */
	if (of_driver_match_device(dev, drv))
		return 1;

	/* Then try ACPI style match */
	if (acpi_driver_match_device(dev, drv))
		return 1;

	/* Then try to match against the id table */
	if (pdrv->id_table)
		return platform_match_id(pdrv->id_table, pdev) != NULL;

	/* fall-back to driver name match */
	return (strcmp(pdev->name, drv->name) == 0);

}
```
从代码中可以得知，`platform_match`主要根据四种情况对设备和驱动进行匹配。
根据注释可以知道，首先判断是否已经设置`driver_override`，后面只绑定到匹配的驱动程序。
 - 根据设备树风格的匹配；
 - 根据ACPI风格的匹配；
 - 匹配ID表（即platform_device设备名是否出现在platform_driver的ID表内）
 - 匹配platform_device设备名和驱动的name成员


## 参考

https://blog.csdn.net/clam_zxf/article/details/80675395

https://www.cnblogs.com/chenfulin5/p/5690661.html

http://blog.chinaunix.net/uid-25622207-id-2778126.html
