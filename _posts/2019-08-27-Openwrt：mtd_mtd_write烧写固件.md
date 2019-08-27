---
layout: post
title: Openwrt：mtd_mtd_write烧写固件
---
@[toc]
## 1 查看当前系统分区信息
`cat /proc/mtd`
显示：
```shell
dev:    size   erasesize  name
mtd0: 00050000 00010000 "u-boot"
mtd1: 00020000 00010000 "u-boot-env"
mtd2: 00f80000 00010000 "firmware"
mtd3: 00107440 00010000 "kernel"
mtd4: 00e78bc0 00010000 "rootfs"
mtd5: 00810000 00010000 "rootfs_data"
mtd6: 00010000 00010000 "art"
```
其中，mtd2就是固件分区（firmware）

## 2 备份固件firmware
```shell
dd if=/dev/mtd2 of=/tmp/firmware_backup.bin
```
然后用winscp登陆路由器tmp目录，将固件文件`firmware_backup.bin`保存到电脑中。

## 3 恢复固件firmware
先用**winscp**将固件文件`firmware_backup.bin`传至路由器`tmp`目录,然后：
```shell
mtd -r write /tmp/firmware_backup.bin firmware
```
即可恢复，恢复完成路由器会自行重启

## 4 备份恢复Openwrt路由器配置
**备份自定义路由器信息，包括新安装软件:**
```shell
dd if=/dev/mtd5 of=/tmp/overlay.bin
```
**恢复备份设置并重启:**
先用winscp将备份文件overlay.bin传至路由器tmp目录,然后：
```shell 
mtd -r write /tmp/overlay.bin rootfs_data
```
**仅备份路由器配置:**
```shell
sysupgrade -b /tmp/back.tar.gz
```
**恢复路由器配置:**
```shell
sysupgrade -f /tmp/back.tar.gz
```
## 5 恢复Openwrt路由器默认设置
**删除/overlay分区所有文件，重启即恢复默认设置:**
```shell
rm -rvf /overlay/* && reboot
```
**使用mtd清除/overlay分区信息后重启即恢复默认设置:**
```shell
mtd -r erase rootfs_data
```

## 6 刷新路由器固件
**使用mtd刷新:**
先用winscp将固件文件`xxx.bin`传至路由器`tmp`目录,然后：
```shell
mtd -r write /tmp/xxx.bin firmware
```
刷新完成后路由器会自动重启。

**使用sysupgrade更新，推荐这种方式:**
`sysupgrade`相比`mtd`更加安全，变砖的可能性比较少；
先用winscp将固件文件`xxx.bin`传至`tmp`目录，然后：
```shell
sysupgrade /tmp/xxx.bin
```
