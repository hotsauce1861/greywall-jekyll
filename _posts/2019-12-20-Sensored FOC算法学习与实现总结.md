@[toc]
# 基于STM32的有感FOC算法学习与实现总结
## 1 前言
`Field Oriented Control` 磁场定向控制 (`FOC`)，`FOC`是有效换向的公认方法。`FOC`的核心是估计转子电场的方向。一旦估计了转子的电角度，就将电动机的三相换相，以使定子磁场垂直于转子磁场。本文参考了`TI`，`microchip`的相关文档，基于`STM32F103`系列单片机实现了带编码器的`FOC`算法，实现了对通用伺服电机（表贴式`PMSM`）的控制。

## 2 FOC算法架构
`FOC`算法的整体架构如下图所示，采用了双闭环的控制系统，包括速度环和电流环，也叫转矩环，而传统的伺服驱动器还需要位置环，图中并未给出，这个后面另外描述，反馈部分采用双电阻采样，和增量编码器。
![图一 FOC](https://img-blog.csdnimg.cn/20191222200615680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
所以，从上图可以了解到，实现`FOC`算法总共需要以下几个部分；
 - 坐标变换，由于`PMSM`是非线性的复杂系统，为了实现控制上的解耦，需要进行坐标变换；
 	-  `Clark`变换；
 	-  `Park`变换；
 - `SVPWM`模块；
 - 反馈量采集部分
    -	相电流采集
	-	编码器信号采集
 - 闭环控制部分可以分为三个环节；当然，根据需求，双闭环也比较常见；
	 - 位置环
	 - 速度环
	 - 电流环
	 
下面会对每个环节的关键部分做一下介绍，具体的实现与细节由于篇幅有限会另外开篇幅做介绍。
## 3 坐标变换
$OABC$三相坐标到静止坐标系$\alpha\beta$坐标系可以分为恒幅值变换和恒功率变换，两者的主要区别就是变换系数不同，下文统一使用恒幅值变换。
### 3.1 Clark变换
三相电流ABC分别为$i_{A}$，$i_{B}$，$i_{C}$，根据基尔霍夫电流定律满足以下公式：
$$i_{A}+i_{B}+i_{C} = 0$$
静止坐标系$\alpha\beta$，$\alpha$轴的电流分量为$i_{\alpha}$，$i_{\beta}$，则`Clark`变换满足以下公式：

$$i_{\alpha} = i_{A} \\
i_{\beta} = \cfrac{1}{\sqrt{3}}*i_{A}+\cfrac{2}{\sqrt{3}}*i_{B}$$
### 3.2 Park变换
`Park`变换的本质是静止坐标系$\alpha\beta$乘以一个旋转矩阵，从而得到$dq$坐标系，其中；
- $d$ 轴又叫直轴，方向与转子磁链方向重合；
- $q$ 轴又叫交轴，方向与转子磁链方向垂直；

所以，帕克变换又叫交直变换，由静止坐标系$\alpha\beta$上的交流量最终变换到$dq$坐标系上的直流量；
`Park`变换满足以下公式；
$$i_{d}=i_{\alpha}*cos\theta+i_{\beta}*cos\theta \\
i_{q}=-i_{\alpha}*cos\theta+i_{\beta}*cos\theta$$

### 3.3 Park反变换
`Park`又叫直交变换，满足以下公式：
$$i_{\alpha}=i_{d}*cos\theta-i_{q}*sin\theta \\
i_{\beta}=i_{d}*cos\theta+i_{q}*cos\theta$$


## 4 SVPWM
实际的马鞍波如下图所示；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191222203648694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
## 5 反馈部分
反馈部分需要采集相电流，电角度和速度，如下图所示；
**红**色曲线表示 $i_{A}$；
**黄**色曲线表示 $i_{B}$；
**蓝**色曲线表示电角度 $\theta_{e}$；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191222203747537.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
图中黄色箭头所指的点，可以看到满足以下条件：
$$\theta_{e} = 0 \\
i_{A} = 0$$
### 5.1 相电流
相电流采样通常有三种方案；
- 单电阻采样；
- 双电阻采样；
- 三电阻采样；
### 5.2 电角度和转速
电角度的测量需要通过对编码器进行正交解码，`STM32`的`TIM`定时器自带编码器接口，可以很轻松实现对正交编码器的正交编码；

## 6 闭环控制
### 6.1 电流环
最终给出电流闭环的结构，如下图所示；![在这里插入图片描述](https://img-blog.csdnimg.cn/20191222211221385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
> 红色曲线表示 $i_{\alpha}$ 
黄色曲线表示  $i_{\beta}$ 
粉色曲线表示  $i_{q}$ 
蓝色曲线表示  $i_{d}$ 

由于使用的表贴式`PMSM`，满足以下条件：
$$L_{d} = L_{q} = L_{s}$$
所以，$d$轴和$q$轴可以共用同一套`PI`参数，可以通过经验试凑法进行参数整定，或者可以通过测量电机参数，计算`PI`参数的大致范围，然后再进行细调。
### 6.2 速度环
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019122318425137.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
电流环调节稳定之后，速度环需要调整速度PI控制器，这里可以参阅如何调试`PI`参数。
### 6.3 位置环
红色曲线表示给定位置；
黄色曲线表示实际位置；
粉色曲线表示给定转速；
蓝色曲线表示实际转速；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191222211914790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
## 写在最后
经过一段时间的调试，终于完成了从零到一的`FOC`算法框架，由于能力有限，有的地方理解不到位，需要细加斟酌，如有错误的地方，希望斧正，另外由于`FOC`内容较多，篇幅较长，时间有限，后续会进一步进行补充，细节的部分会单独开篇进行讨论。
