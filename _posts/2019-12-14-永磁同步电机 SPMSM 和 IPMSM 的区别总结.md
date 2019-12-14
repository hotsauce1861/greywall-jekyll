---
layout: post
tags: [motor control]
comments: true
---

## 永磁同步电机的分类
永磁同步电机根据转子上永磁体的位置不同，可以分为；
- 表贴式永磁同步电机——`SPMSM`;
- 内置式永磁同步电机——`IPMSM`;

不过有时候又会看到凸极特性和隐极特性，这里主要是由于电机构造不同导致的Lq与Ld的差别，因此，表贴式和内置式两种电机在控制上又有所不同，下面来看一下；
| | | |
|---|---|---|
|电机|	特性|	电感分量|
|SPMSM|	隐极|	Lq = Ld|
|IPMSM|	凸极|	Lq > Ld|

## 表贴式电机dq轴电感发布
![](/img/20191214/SPMSM.png)

## 内置式电机dq轴电感发布
![](/img/20191214/IPMSM.png)

至于为何会造成Lq和Ld上的差别，这里我理解还是不太透彻，原因是内置式电机的转子将一部分铁芯的位置占据了，所以，导致d轴的电感分量相对q轴的电感分量要小，下面看在看一张图；

![](/img/20191214/Rotor.png)

**上面的解释不够透彻,在看下面一段解释；**

>Because the number of stator coil turns for theIPMSM (48 turns) was 25% less than that for theSPMSM (64 turns), the IPMSM must have alower inductance than the SPMSM proportionalto the square of the difference in the number ofturns if all other conditions are the same.However the size of the actual air gap issignificant because the PM of the designedSPMSM was thick. Also, the V-type placementof the PMs in the IPMSM increases the core areabetween the air gap and PM and increases the qaxis inductance. Hence, the inductance of theIPMSM, especially the q-axis inductance, wasmore significant than that of the SPMSM despitethe former having fewer coil turns than the latter.

[论文](/img/20191214/EVS28_IPMSM_VS_SPMSM.pdf)