<!-- TOC -->
- [准备工作](#准备工作)
- [什么是内核模块](#什么是内核模块)
- [编写 hello.c](#编写-helloc)
- [模块编译](#模块编译)
- [相关指令](#相关指令)
- [测试结果](#测试结果)
    - [模块加载](#模块加载)
    - [模块卸载](#模块卸载)
<!-- /TOC -->

## 准备工作

在进行以下操作前，首先我准备了一台电脑，并且安装了虚拟机，系统是Ubuntu16.04。如果是开发板，那么需要安装交叉编译器，但是目前我只在虚拟机的Ubuntu16.04系统里完成驱动模块的加载和卸载，实现了一个最最简单的内核模块，并且通过这个最简单的驱动，学习最基本的概念。

## 什么是内核模块

模块是可以根据实际需要可以动态加载和卸载到内核中的代码。它们扩展了内核的功能，而无需重启系统，就可以进行模块加载，并工作。例如，一种类型的模块是设备驱动程序，它允许内核访问连接到系统的硬件。没有模块，我们必须构建整个内核并将新功能直接添加到内核映像中。除了拥有更大的内核之外，这还有一个缺点，就是每次我们想要新功能时都需要我们重新编译内核并烧录到设备。

## 编写 hello.c

```c
#include <linux/init.h>		//所有模块都会需要这个头文件
#include <linux/module.h>	//下面的宏需要

static int __init hello_init(void){
        printk(KERN_INFO "module init success\n");
        return 0;
}

static void __exit hello_exit(void){
        printk(KERN_INFO "module exit success\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");	//开源协议
MODULE_AUTHOR("作者");
MODULE_DESCRIPTION("功能描述");
```

这是一个简单内核模块程序，可以动态加载和卸载。虽然没有实际的功能。
模块加载的时候系统会打印`module init success\n`
模块卸载的时候系统会打印`module exit success\n`

## 模块编译

```makefile
obj-m := hello.o
PWD := $(shell pwd)
KVER := $(shell uname -r)
KDIR :=/lib/modules/$(KVER)/build/

all:
        $(MAKE) -C $(KDIR) M=$(PWD)

clean:
        rm -rf *.o *.mod.c *.mod.o *.ko *.symvers *.order *.a
```

将`hello.c`和`Makefile`放在同一路径下进行编译，编译成功，会在当前路径下生成`hello.ko`，这就是我们将要加载到内核的模块。

```shell
make
```



## 相关指令

| name     | function                   |
| -------- | -------------------------- |
| lsmod    | 查看已经加载到内核中的模块 |
| insmod   | 加载模块到内核中           |
| rmmod    | 从内核卸载模块             |
| depmod   | 生成模块所需要的依赖       |
| modprobe | 很强大的指令(-h)           |

## 测试结果

### 模块加载

加载`hello.ko`模块到内核中

```shell
insmod hello.ko
```

如果模块加载成功的话，可以查看模块

 ```shell
lsmod | grep hello
 ```

成功加载会显示以下结果

```shell
hello                  16384  0 
```

并且可以查看内核打印的消息

```shell
dmesg | grep "init success"
```

```shell
[ 4160.003247] module init success
```

### 模块卸载

```
rmmod hello.ko
```

成功卸载`hello`模块后，可以查看内核是否正常打印出我们预设在程序的打印信息。

```shell
dmesg | grep "exit success"
```
可以看到终端上显示`module exit success`，说明通过`rmmod`成功卸载`hello.ko`
```shell
[ 7160.003247] module exit success
```
这时候，如果再通过`lsmod`去查看当前的内核模块，就会发现`hello.ko`已经消失不见了。
