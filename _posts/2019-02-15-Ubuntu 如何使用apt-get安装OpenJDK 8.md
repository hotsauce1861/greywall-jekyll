---
layout: post
tags: [Ubuntu]
comments: true
---

`Android 8.1` 系统编译的时候需要安装`OpenJDK 8`，这里如果可以自己下载源码编译安装，当然本想编译Android系统的我，不想兴师动众。可以通过`apt-get install openjdk-8-jdk`指令直接进行安装，但是需要在源里添加相应的package。

## 添加ppa仓库
这个是[OpenJDK 8 ppa仓库](https://launchpad.net/~openjdk-r/+archive/ubuntu/ppa)。
```
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update 
sudo apt-get install openjdk-8-jdk
```
## 设置openjdk版本
```
sudo update-alternatives --config java
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215212617626.png)
## 查看java 版本
```
java -version
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215212900802.png)
