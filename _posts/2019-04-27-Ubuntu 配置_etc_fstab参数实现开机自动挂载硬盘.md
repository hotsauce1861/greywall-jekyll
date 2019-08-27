<!-- TOC -->
- [前言](#前言)
- [fstab](#fstab)
    - [参数含义](#参数含义)
- [实现步骤](#实现步骤)
    - [1  查看硬盘信息，并找到需要进行挂载的硬盘](#1--查看硬盘信息并找到需要进行挂载的硬盘)
    - [2 sudo mkfs.ext4 /dev/sdc](#2-sudo-mkfsext4-devsdc)
    - [3 sudo mkdir /home/diska](#3-sudo-mkdir-homediska)
    - [4 查看UUID](#4-查看uuid)
    - [5 配置/etc/fstab](#5-配置etcfstab)
<!-- /TOC -->

## 前言
不同于热插拔的设备，对于硬盘可能需要长期挂载在系统下，所以如果每次开机都去手动`mount`是非常痛苦的，当然`Ubuntu`系统的`GNOME`桌面自带的`gvfsd`也会帮你自动挂载，但是指向的路径却是按照`uuid`命名的，对于有强迫症的我而言，这是极其痛苦的，所以希望开机就可以自动挂载硬盘到指定路径。只关注具体**如何实现**可以直接跳过我的这些“废话”，直接移步到**实现步骤**。
## fstab
系统开机的时候会读取`/etc/fstab`这个文件中的内容，根据文件配置情况去挂载磁盘。`vi /etc/fstab`，打开`fstab`文件，具体如下图所示；
![fstab](https://img-blog.csdnimg.cn/20190427091034913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
### 参数含义
这里需要配置6个参数，`<file system>`，`<mount point>`，`<type>`，`<options>`，`<dump>`，`<pass>`；简单解释一下每个参数的含义，不能只见树木不见森林。
- **file system**
	文件系统，参考默认的`fstab`来看，这里只需要把硬盘的`UUID`正确配置即可；可以通过指令`blkid`，查看硬盘的`UUID`；
- **mount point**
	挂载路径，最终硬盘会被挂载到配置的这个路径下，但是这个路径必须先存在，提前创建好这个路径即可；
- **type**
	硬盘的文件系统类型，相应的有`ntfs`，`ext4`，`fat`，`vfat`等等，这里要根据实际情况设置，同样的也可以通过指令`blkid`，查看硬盘的`TYPE`；
- **options**
	   
| option |       description |
|--|--|
|defaults |       use default options: rw, suid, dev, exec, auto, nouser, and async. |
|  noauto | do not mount when "mount -a" is given (e.g., at boot time) |
|           user   |allow a user to mount|
|              owner|  allow device owner to mount|
|           comment or x-<name> |   for use by fstab-maintaining programs|
| nofail |  do not report errors for this device if it does not exist.|
- **dump**
	这个参数用来检查文件系统以多快频率进行备份，系统将认为其值为0，则不需要进行备份；设置成1暂时也没有实践过；
- **pass**
	这个参数用来决定在启动时需要被`fsck`扫描的文件系统的顺序，根文件系统"/"对应该字段的值应该为1，其他的应该逐渐递增，如果设置为0则表示不扫描。
## 实现步骤
### 1  查看硬盘信息，并找到需要进行挂载的硬盘
```shell
sudo fdisk -l
```
这里我需要对`/dev/sdc`进行挂载；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190427092606327.png)
### 2 sudo mkfs.ext4 /dev/sdc
该指令会格式化硬盘，所以请先备份数据，如果硬盘的`TYPE`就是`ext4`则无需进行这个步骤的操作
### 3 sudo mkdir /home/diska
创建硬盘需要挂载的路径，这个路径可以根据自己的需要随意命名；
### 4 查看UUID
```shell
$ blkid /dev/sdc 
$ /dev/sdc: UUID="b72a8f66-73d9-42d0-92cc-ae24bee6a309" TYPE="ext4"
```
### 5 配置/etc/fstab
打开`/etc/fstab`，根据对应的格式如下把`UUID`（步骤4中获取），挂载路径（步骤4中创建），配置到文件中；
```shell
# /home/diska was my persional disk
UUID=b72a8f66-73d9-42d0-92cc-ae24bee6a309 /home/diska   ext4 errors=remount-ro 0       0
```
配置完之后如下图所，记得保存；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190427093734851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
最后，重启系统，看一下硬盘是不是已经挂载上去了；
```shell
$ cat /proc/mounts | grep sdc
$ /dev/sdc /home/diska ext4 rw,relatime,errors=remount-ro,data=ordered 0 
```
OK，最终`sdc`成功地挂载到`/home/diska`路径下了。

