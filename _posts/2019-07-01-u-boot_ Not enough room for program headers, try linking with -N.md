---
layout: post
tags: [uboot]
comments: true
---

在编译`u-boot`的时候出现了以下错误：
```shell
arm-linux-gnueabi-ld.bfd: u-boot: Not enough room for program headers, try linking with -N
arm-linux-gnueabi-ld.bfd: final link failed: Bad value
Makefile:1208: recipe for target 'u-boot' failed
```
解决方案可以参考这个[patch](https://www.mail-archive.com/u-boot@lists.denx.de/msg235861.html);
或者在`Makefile`中添加一条语句，修改链接参数
```shell
FLAGS_u-boot += $(call lmZ-option, --no-dynamic-linker)
```
具体如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190701182018792.png)
