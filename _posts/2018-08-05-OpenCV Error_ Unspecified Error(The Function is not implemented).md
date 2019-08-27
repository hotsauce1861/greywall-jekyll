---
layout: post
tags: [OpenCV]
comments: true
---

Ubuntu 或者 Debian 系统显示窗口的时候遇到了这个问题

error: (-2:Unspecified error) The function is not implemented. Rebuild the library with Windows, GTK+ 2.x or Carbon support. If you are on Ubuntu or Debian, install libgtk2.0-dev and pkg-config, then re-run cmake or configure script in function 'cvNamedWindow'

##解决办法
```shell
apt-get install install libgtk2.0-dev pkg-config
```
安装成功之后，重新运行cmake对源码进行编译，安装。
