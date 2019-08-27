---
layout: post
tags: [Openwrt]
comments: true
---

在编译mr3420的固件时，添加了luci、jamvm，但是最终编译的固件“**openwrt-ar71xx-generic-tl-mr3420-v1-squashfs-factory.bin**”的大小仅仅只有3.1MB，为何会如此小巧，心生疑惑下把该固件烧录到路由中，发现luci和java虚拟机都没有添加上去，然后才发现是固件生成失败了。提示如下：

> /bin/ar71xx/openwrt-ar71xx-generic-tl-mr3420-v1-squashfs-factory.bin
> [mktplinkfw] kernel length aligned to 1164800 [mktplinkfw] error:
> images are too big

根据提示可知是生成的固件大于flash的容量，所以要修改flash的大小。
找到/target/linux/ar71xx/image/Makefile ，进行修改。

*这里ar71xx具体应该视当前的硬件平台而定，例如MT7688则是ramips*

打开Makefile，搜索找到mr3420内容相关的位置，可以看到这里把RAM设置为64M，串口波特率设置为115200，猜测了一下最后的4M可能会有问题，按照实际的路由的flash大小改为16M。

    $(eval $(call SingleProfile,TPLINK,64kraw,RNXN360RT,rnx-n360rt,TL-WR941ND,ttyS0,115200,0x09410002,0x00420001,4M))
    $(eval $(call SingleProfile,TPLINK,64kraw,TLMR3220V1,tl-mr3220-v1,TL-MR3220,ttyS0,115200,0x32200001,1,4M))
    #$(eval $(call SingleProfile,TPLINK,64kraw,TLMR3420V1,tl-mr3420-v1,TL-MR3420,ttyS0,115200,0x34200001,1,4M))
    $(eval $(call SingleProfile,TPLINK,64kraw,TLMR3420V1,tl-mr3420-v1,TL-MR3420,ttyS0,115200,0x34200001,1,16M))
    $(eval $(call SingleProfile,TPLINK,64kraw,TLWA701NV1,tl-wa701n-v1,TL-WA901ND,ttyS0,115200,0x07010001,1,4M))
    $(eval $(call SingleProfile,TPLINK,64kraw,TLWA730REV1,tl-wa730rev1,TL-WA901ND,ttyS0,115200,0x07300001,1,4M))
    $(eval $(call SingleProfile,TPLINK,64kraw,TLWA7510NV1,tl-wa7510n,TL-WA7510N,ttyS0,115200,0x75100001,1,4M))
    $(eval $(call SingleProfile,TPLINK,64kraw,TLWA801NV1,tl-wa801nd-v1,TL-WA901ND,ttyS0,115200,0x08010001,1,4M))
    $(eval $(call SingleProfile,TPLINK,64kraw,TLWA830RV1,tl-wa830re-v1,TL-WA901ND,ttyS0,115200,0x08300010,1,4M))
    $(eval $(call SingleProfile,TPLINK,64kraw,TLWA901NV1,tl-wa901nd-v1,TL-WA901ND,ttyS0,115200,0x09010001,1,4M))
    $(eval $(call SingleProfile,TPLINK,64kraw,TLWA901NV2,tl-wa901nd-v2,TL-WA901ND-v2,ttyS0,115200,0x09010002,1,4M))

完成以上修改之后重新编译openwrt，得到的固件不再是3.1M，已经将近13M，烧录到路由器，发现Luci和java虚拟机都正常添加。

    -rw-r--r-- 1 root root      872  6月 29 09:04 md5sums
    -rw-r--r-- 1 root root 11927552  6月 29 09:04 openwrt-ar71xx-generic-root.squashfs
    -rw-r--r-- 1 root root 11796484  6月 29 09:04 openwrt-ar71xx-generic-root.squashfs-64k
    -rw-r--r-- 1 root root 16252928  6月 28 17:20 openwrt-ar71xx-generic-tl-mr3420-v1-squashfs-factory.bin
    -rw-r--r-- 1 root root 12976132  6月 28 17:20 openwrt-ar71xx-generic-tl-mr3420-v1-squashfs-sysupgrade.bin
    -rw-r--r-- 1 root root  1601316  6月 29 09:04 openwrt-ar71xx-generic-uImage-gzip.bin
    -rw-r--r-- 1 root root  1156912  6月 29 09:04 openwrt-ar71xx-generic-uImage-lzma.bin
    -rwxr-xr-x 1 root root  3509468  6月 29 09:04 openwrt-ar71xx-generic-vmlinux.bin
    -rwxr-xr-x 1 root root  3514532  6月 29 09:04 openwrt-ar71xx-generic-vmlinux.elf
    -rw-r--r-- 1 root root  1638400  6月 29 09:04 openwrt-ar71xx-generic-vmlinux.gz
    -rw-r--r-- 1 root root  1179648  6月 29 09:04 openwrt-ar71xx-generic-vmlinux.lzma
    -rwxr-xr-x 1 root root  1228877  6月 29 09:04 openwrt-ar71xx-generic-vmlinux-lzma.elf
    drwxr-xr-x 8 root root     4096  6月 28 14:14 packages
    -rw-r--r-- 1 root root     1352  6月 29 09:04 sha256sums


