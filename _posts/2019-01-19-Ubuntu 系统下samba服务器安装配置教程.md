---
layout: post
tags: [Ubuntu]
comments: true
---

## 前言

本来想要按照原先的写作习惯，进行一些的铺垫，引证和概念介绍，但是这个场合感觉还是开门见山比较好，毕竟重点是一看就懂，快速设置，快速安装。

## 环境搭建

### 安装

```shell
sudo apt-get update
sudo apt-get install samba -y
```

### 配置

打开配置文件，一般默认的配置文件路径是`/etc/samba/smb.conf`

这个文件中并没有给出特别详细的例子，想要具体了解的话，可以查看manpage手册。

```shell
sudo man smb.conf
```

如果你对此不赶兴趣，只想快速配置，可以参考下面的Example，经过亲测有效。

## Examples

### 1 创建共享（任何人都可以访问）

将以下代码添加到`/etc/samba/smb.conf` 文件中。

```shell
[test]
   comment = share folder
   browseable = yes
   path = /yourpath
   create mask = 0755
   directory mask = 0755
   writeable = yes
   public = yes
   guest ok = yes
```

为需要共享的文件夹设置读写权限

```
sudo chmod 777 -R /yourpath
```

重启smbd

```shell
sudo /etc/init.d/smbd restart
```



### 2 单用户权限（需要密码访问）

#### 添加samba用户

这里插播一下，简单说明如何添加 samba user，同时设置密码，并记住这个密码，远程登陆的时候需要用到。
`yourusername`必须是和**系统中已经存在的系统用户相同**，例如安装Ubuntu系统时，会提示你设置一次**用户名**，或者时通过`useradd`添加的用户。
```shell
sudo smbpasswd -a yourusername
New SMB password:
Retype new SMB password:
Added user yourusername.
#Failed to find entry for yourusername.
```

如果出现以上执行结果，表示一切顺利，添加用户成功。

> 如果出现`Failed to find entry for yourusername.`，提示失败，则需要检查一下当前系统是否已经存在 yourusername 这个用户。yourusernamecat /etc/passwd | grep yourusername



#### 配置参数

```shell
[test]
   comment = share folder
   browseable = yes
   path = /yourpath
   create mask = 0755
   directory mask = 0755
   writeable = yes
   valid users = yourusername
   public = yes
   available = yes
   read only = no
```



### 3 支持游客访问（单用户拥有管理员权限）

简单说明一下这种情况，   系统用户`yourusername`对`/yourpath`拥有权限，但是其他用户通过游客方式进行访问，并且只有读权限。

```shell
[test]
   comment = share folder
   browseable = yes
   path = /yourpath
   create mask = 0755
   directory mask = 0755
   writeable = yes
   guest ok = yes
   public = yes
   available = yes
```

完成以上配置后需要更改`/yourpath`的用户组和用户。

```shell
sudo chgrp yourusername /yourpath
sudo chown yourusername /yourpath
```

>注意：由于系统在创建用户的时候会默认将用户添加到与用户名相同的一个群组中，修改用户组的时候具体要根据`/etc/passwd`文件里的用户组信息为准。



