---
layout: post
tags: [Ubuntu]
comments: true
---

`Linux内核`或者`u-boot`进行`make menuconfig`的时候，如果系统上没有安装ncurses，就会出现以下报错
```shell
 *** Unable to find the ncurses libraries or the
 *** required header files.
 *** 'make menuconfig' requires the ncurses libraries.
 *** 
 *** Install ncurses (ncurses-devel or libncurses-dev 
 *** depending on your distribution) and try again.
 *** 
scripts/kconfig/Makefile:229: recipe for target 'scripts/kconfig/dochecklxdialog' failed
make[1]: *** [scripts/kconfig/dochecklxdialog] Error 1
Makefile:503: recipe for target 'menuconfig' failed
make: *** [menuconfig] Error 2
```
**解决方案：**
```shell
sudo apt-get install libncurses5-dev
```
**关于ncurses**
libncurses*这些库是基于系统用来在显示器上显示文本. 一个例子就是,ncurses用在内核的"make menuconfig"进程中。
libform* 在ncurses中使用表格；
libmenu* 在ncurses中使用菜单；
libpanel*在ncurses中使用面板。
Ncurses 依赖于: Bash, Binutils, Coreutils, Diffutils, Gawk, GCC, Glibc, Grep, Make, Sed.
