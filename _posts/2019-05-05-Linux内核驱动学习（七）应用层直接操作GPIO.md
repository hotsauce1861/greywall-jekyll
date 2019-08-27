@[toc]

## 简介

前面通过`libgpio`的方式介绍了内核空间对`GPIO`进行操作的接口，其做了较好的封装，同时`Linux`系统的`sysfs`机制已经在系统路径下`/sys/class/gpio`注册了相应的节点，通过读写该节点下的文件就能轻松的完成`GPIO`输入输出配置以及引脚状态的获取。

## 原理图
我使用的`Rockchip`的`px30`，引脚是`GPIO3_D0`，具体硬件肯定会不同，注意参考`soc`的`datasheet`和硬件原理图，先定位正确需要操作的`GPIO`，千里之行始于足下。
![Datasheet](https://img-blog.csdnimg.cn/20190504161951205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
## 节点
在`/sys/class/gpio`路径下有`export`和`unexport`这两个文件；`GPIO3_D0`在这里是`120`，具体硬件和数字的对应关系后面会继续讲到；
```shell
echo 120 > /sys/class/gpio/export
```
执行以上这条指令后，会在`/sys/class/gpio/`下生成`gpio120`节点，简单看一下该路径下都有哪些文件；

```shell
$ /sys/class/gpio/gpio120
$ active_low device direction edge power subsystem uevent value
```
### 设置为输出
```shell
$ cd /sys/class/gpio/gpio120
$ echo 0 > active_low
$ echo out > direction
$ echo 1 > value	#输出高
$ echo 0 > value	#输出低
```
另一种情况，设置`active_low`为`1`，就会出现另一种情况；
```shell
$ cd /sys/class/gpio/gpio120
$ echo 1 > active_low
$ echo out > direction
$ echo 1 > value	#输出低
$ echo 0 > value	#输出高
```
由此看出，`active_low`的作用已经很明显了，后面没有特别指出的情况下，`active_low`的值默认为`0`；一表胜过千言万语，简单整理一个表格，如下所示；
|active_low  | value | 实际GPIO输出  |
|--|--|--|
| 0 | 1 | high|
| 0 | 0 | low|
| 1 | 1 | low|
| 1 | 0 | high|

### 设置为输入
```shell
$ cd /sys/class/gpio/gpio120
$ echo int > direction
$ cat value 	#读取GPIO的电平状态
```
## 映射关系
`Rockchip px30`平台的`GPIO`总共分为`GPIO0~GPIO3`四组，每一组最多有32个`GPIO`，依次分为`A`，`B`，`C`，`D`四个小组，每组最多8个，对于硬件上实际没有达到`8`个的情况下，计算偏移的时候，也按照`8`来计算。RK平台可以参考[dt-bindings/pinctrl/rockchip.h](https://github.com/rockchip-linux/kernel/blob/develop-4.4/include/dt-bindings/pinctrl/rockchip.h)。其他平台的话，如果有源代码可以参考以下厂商给出的具体定义，并结合`SOC`的原理图和硬件原理图，来计算。
具体计算如下表所示；依次类推；
|引脚| 计算 |
|--|--|
|GPIO3_D0  | `3*32 + 3*8 + 0 = 120` |
|GPIO3_D1 |`3*32 + 3*8  + 1 = 121` |
|GPIO2_A1 |`2*32 + 0*8  + 1 = 65` |
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190504164651732.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
## debugfs
debugfs 是 Linux系统下为了方便驱动开发人员对驱动调试的文件系统。
```shell
$ cat /sys/kernel/debug/gpio
```
可以通过`debugfs`查看`gpio-120`硬件上的实际输出和软件上是否相符合；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190504165210723.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
## pwm demo 
强迫症的我简单写了一个模拟`pwm`的`shell`，虽然比较鸡肋，因为是占空比，频率都是不可调的，单纯是为了看一下控制的效果，前提是已经执行`echo 120 > export`这条指令并且成功生成相应的节点，代码简单如下；
```shell
 #!/bin/bash
 GPIO=120
 i=0
 value=0
 while [ 1 -eq 1 ]
 do
     i=$(($i + 1))
     if [ $(( $i % 2 )) -eq 0 ]
     then
         value=0
     else
         value=1
     fi
     echo "current i is $i"
     echo "current value is $value"
     echo $value > /sys/class/gpio/gpio$GPIO/value
     usleep 1000
 done
```
用示波器测量`GPIO3_D0`引脚的波形，和预期的一样；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190504170152293.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
