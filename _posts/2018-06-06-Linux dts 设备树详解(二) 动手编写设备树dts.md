---
layout: post
tags: [Device tree]
comments: true
---

[Linux dts 设备树详解(一) 基础知识](https://blog.csdn.net/u010632165/article/details/89847843)
[Linux dts 设备树详解(二) 动手编写设备树dts](https://blog.csdn.net/u010632165/article/details/91488811)

---
@[toc]

## 前言
在简单了解概念之后，我们可以开始尝试写一个简单的设备树，从而加深对设备树整体架构以及部分语法的理解，因为整体知识面比价庞杂，无法面面俱到，本文旨在笔者学习之初对于设备树常用部分的总结与归纳。因为会涉及到很多硬件信息的绑定，详细的可以查阅Linux内核源码下的文档`Documentation/devicetree/bindings`。具体如下图所示；
![设备树文档](https://img-blog.csdnimg.cn/20190612082729898.png)
## 硬件结构
1个双核`ARM Cortex-A9`32位处理器； 
ARM本地总线上的内存映射区域分布有两个串口（分别位于`0x101F1000`和`0x101F2000`）
`GPIO`控制器（位于`0x101F3000`） 
`SPI`控制器（位于`0x10170000`） 
中断控制器（位于`0x10140000`）
外部总线桥上连接的设备如下：
SMC `SMC91111`以太网（位于`0x10100000`）
`I2C`控制器（位于`0x10160000`） 
64MB NOR Flash（位于`0x30000000`） 
外部总线桥上连接的I2C控制器所对应的I2C总线上又连接了`Maxim DS1338`实时钟（I2C地址为`0x58`） 
具体如下图所示；
![硬件结构图](https://img-blog.csdnimg.cn/2019061209102279.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
## 设备树dts文件
那么，如何将上面的硬件结构，通过设备树语言描述成`dts`文件呢？具体我实现在下图，并且做出了详细的解释。其中需要注意的有以下几个属性：
- `compatitable`：兼容性属性；
- `#address-cells`,`#size-cells`：地址编码所采用的格式；
- `节点名称@节点地址`：例如`gpio@101f3000`，这里名称和地址要和实际的对应起来；
- `标签`：例如`interrupt-parent = <&intc>;`，这这里的`intc`就是一个标签(label)，通过`&`可以获取它的值，这里可以简单的理解成一个变量，然后在下面需要对这个标签进行另外的解析，例如`intc:interrupt-controller@10140000`；所以，这两个地方的`intc`都是对应起来的。
最后，具体的实现可以参考下图；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190612083206967.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
