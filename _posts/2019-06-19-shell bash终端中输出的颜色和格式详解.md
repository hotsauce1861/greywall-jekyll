---
layout: post
tags: [shell]
comments: true
---

ANSI/VT100终端和终端仿真器不只是能够显示黑色和白色文本; 由于转义序列，它们可以显示颜色和格式化文本。这些序列由Escape字符（通常用“`^[`” 或"`<Esc>`"表示）组成，后跟一些其他字符："`<Esc>[`*FormatCode*`m`"。
在Bash中，<Esc>可以使用以下语法获取字符：
`\e`
`\033`
`\x1B`
例子：

| 代码（Bash） | Preview |
|--|--|
| echo  -e  "\ e [31mHello World \ e [0m" |  ![hello world](https://img-blog.csdnimg.cn/20190619111931351.png)|
| echo  -e  "\ 033 [31mHello \ e [0m World" |![hello world](https://img-blog.csdnimg.cn/2019061911202157.png) |
>注意：
>1. 该命令的-e选项echo启用转义序列的解析。
>2.  "\e[0m"序列删除所有属性（格式和颜色）。在每个彩色文本的末尾添加它是个好主意。;）
> 3. 本页中的示例使用Bash，但`ANSI/VT100`转义序列可用于各种编程语言。

## 1） 格式
### 1.1 Set
|code  | Description | Example | Preview |
|--|--|--|--|
| 1 | 粗体/高亮 | `echo  -e  “Normal \ e [1mBold” `|  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619134438396.png)|
| 2 | 变暗 |`echo  -e  “Normal \ e [2mDim”` |![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619134533628.png) |
| 4 | 下划线 | `echo  -e  “Normal \ e [4mUnderlined”`| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619134709590.png)|
| 5 | 闪烁 |`echo  -e  “Normal \ e [5mBlink”` |![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619134657313.gif) |
| 7 | 反转（反转前景色和背景色）|`echo  -e  “Normal \ e [7minverted”` | ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619134719632.png)|
| 8 | 隐藏（对密码有用）| `echo  -e  “Normal \ e [8mHidden”`|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619134739654.png) |
### 1.2 Reset
|Code	|Description	|Example	|Preview|
|--|--|--|--|
|0	|重置所有属性	| echo -e "\e[0mNormal Text"| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619135720596.png) |
|21|	重置粗体/高亮	|echo -e "Normal \e[1mBold \e[21mNormal"|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619135736113.png)|
|22|	重置变暗	|echo -e "Normal \e[2mDim \e[22mNormal"|![在这里插入图片描述](https://img-blog.csdnimg.cn/2019061913581248.png)|
|24|	重置下划线	|echo -e "Normal \e[4mUnderlined \e[24mNormal"|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619135828236.png)|
|25|	重置闪烁|echo -e "Normal \e[5mBlink \e[25mNormal"|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619135843795.gif)|
|27|	重置反显	|echo -e "Normal \e[7minverted \e[27mNormal"|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619135900464.png)|
|28|	重置隐藏	|echo -e "Normal \e[8mHidden \e[28mNormal"|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619135931669.png)|
## 2）8/16 Colors
### 2.1 前景（文字）
以下颜色适用于大多数终端和终端仿真器2）， 请参阅兼容性列表以获取更多信息。
> 颜色可能因终端配置而异。

|Code|	Color|	Example|	Preview|
|--|--|--|--|
|39	|默认前景色	|echo  -e  “Default \ e [39mDefault”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141052508.png) |
|30|	黑色	|echo  -e  “Default \ e [30mBlack”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141101784.png)  |
|31|	红色	|echo  -e  “Default \ e [31mRed”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141135101.png) | 
|32|	绿色	|echo  -e  “Default \ e [32mGreen”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141152927.png) |
|33|	黄色	|echo  -e  “Default \ e [33mYellow”|![在这里插入图片描述](https://img-blog.csdnimg.cn/2019061914121546.png) |
|34|	蓝色	|echo  -e  “Default \ e [34mBlue”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141224273.png) |
|35|	品红	|echo  -e  “Default \ e [35mMagenta”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141237138.png)|
|36|	青色	|echo  -e  “Default \ e [36mCyan”|![在这里插入图片描述](https://img-blog.csdnimg.cn/2019061914124782.png)|
|37|	浅灰	|echo  -e  “Default \ e [37mLight grey”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141257476.png)|
|90|	深灰色	|echo  -e  “Default \ e [90mDark grey”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141310443.png)|
|91|	红灯	|echo  -e  “默认\ e [91mLight red”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141319563.png)|
|92|	浅绿色	|echo  -e  “Default \ e [92mLight green”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141329662.png)|
|93|	淡黄色	|echo  -e  “Default \ e [93mLight yellow”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141342103.png)|
|94|	浅蓝	|echo  -e  “Default \ e [94mLight blue”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141353126.png)|
|95|	浅洋红色	|echo  -e  “Default \ e [95mLight magenta”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141404538.png)|
|96|	浅青色	|echo  -e  “Default \ e [96mLight cyan”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141416743.png)|
|97|	白色	|echo  -e  “Default \ e [97mWhite”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141425171.png)|

### 2.2 背景
| Code	| Color	| Example| 	Preview| 
| --| --| --| --| 
| 49	| 默认背景颜色	| echo  -e  “Default \ e [49mDefault”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142042919.png)| 
| 40	| 黑色	| echo  -e  “Default \ e [40mBlack”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141729838.png) | 
| 41	| 红色	|echo  -e  “Default \ e [41mRed”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141715955.png)| 
| 42	| 绿色	| echo  -e  “Default \ e [42mGreen”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141813132.png) | 
| 43	| 黄色	| echo  -e  “Default \ e [43mYellow”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141818655.png) | 
| 44	| 蓝色	| echo  -e  “Default \ e [44mBlue”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141826855.png) | 
| 45	| 品红	| echo  -e  “Default \ e [45mMagenta”|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141843323.png) | 
| 46	| 青色	| echo  -e  “Default \ e [46mCyan”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141851119.png)| 
| 47	| 浅灰	| echo  -e  “Default \ e [47mLight grey”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141859165.png)| 
| 100| 	深灰色	| echo  -e  “Default \ e [100mDark grey”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141906860.png)| 
| 101| 	红灯	| echo  -e  “默认\ e [101mLight red”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141940807.png) | 
| 102| 	浅绿色	| echo  -e  “默认\ e [102mLight green”|  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619141948164.png)| 
| 103| 	淡黄色	| echo  -e  “默认\ e [103mLight yellow”|![在这里插入图片描述](https://img-blog.csdnimg.cn/201906191419546.png) | 
| 104| 浅蓝	| echo  -e  “Default \ e [104mLight blue”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142001372.png) | 
| 105	| 浅洋红色	| echo  -e  “Default \ e [105mLight magenta”| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142009959.png) | 
| 106	| 浅青色	| echo  -e  “Default \ e [106mLight cyan”|  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142015180.png) | 
| 107	| 白色	| echo  -e  “Default \ e [107mWhite”|  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142021305.png)| 
## 3）88/256颜色
某些终端（参见兼容性列表）可以支持88或256种颜色。以下是允许您使用它们的控制序列。

注意：颜色编号256仅由vte支持（GNOME终端，XFCE4终端，Nautilus终端，终结者......）。

注意2：88色终端（如rxvt）与256色终端的颜色图不同。要显示88色终端颜色映射，请在88色终端中运行“ 256-colors.sh ”脚本。
### 3.1 前景（文字）
要使用前景中的256种颜色之一（文本颜色），控制序列为“ `<Esc>[38;5;`ColorNumber`m` ”，其中ColorNumber是以下颜色之一：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142347178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
**Example**:
```shell
echo -e "\e[38;5;82mHello \e[38;5;198mWorld"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142534957.png)
```shell
for i in {16..21} {21..16} ; do echo -en "\e[38;5;${i}m#\e[0m" ; done ; echo
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142540367.png)
### 3.2 背景色
要在背景上使用256种颜色中的一种，控制序列为“ `<Esc>[48;5;`ColorNumber`m` ”，其中ColorNumber是以下颜色之一：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142629366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
**Example:**
```shell
echo -e "\e[40;38;5;82m Hello \e[30;48;5;82m World \e[0m"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142741691.png)
```shell
for i in {16..21} {21..16} ; do echo -en "\e[48;5;${i}m \e[0m" ; done ; echo
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142746703.png)
## 4）组合属性
终端允许属性组合。属性必须用分号（“ ;”）分隔。
|Description	|Code (Bash)	|Preview|
|--|--|--|
|Bold + Underlined	|echo -e "\e[1;4mBold and Underlined"|![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619142956913.png)|
|Bold + Red forground + Green background	|echo -e "\e[1;31;42m Yes it is awful \e[0m"| ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619143007175.png)|
## 5）终端兼容性
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019061914310225.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
>表中使用的符号：
“ ok”：终端支持。
“ ~”：终端以特殊方式支持。
“ -”：终端根本不支持。

## 6）示例程序
### 6.1 Colors and formatting (16 colors)
以下shell脚本显示了许多可能的属性组合（但不是全部，因为它一次只使用一个格式属性）。
```shell
#!/bin/bash
 
# This program is free software. It comes without any warranty, to
# the extent permitted by applicable law. You can redistribute it
# and/or modify it under the terms of the Do What The Fuck You Want
# To Public License, Version 2, as published by Sam Hocevar. See
# http://sam.zoy.org/wtfpl/COPYING for more details.
 
#Background
for clbg in {40..47} {100..107} 49 ; do
	#Foreground
	for clfg in {30..37} {90..97} 39 ; do
		#Formatting
		for attr in 0 1 2 4 5 7 ; do
			#Print the result
			echo -en "\e[${attr};${clbg};${clfg}m ^[${attr};${clbg};${clfg}m \e[0m"
		done
		echo #Newline
	done
done 
exit 0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619143725445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)
### 6.2 256 colors
以下脚本显示某些终端和终端仿真器（如XTerm和GNOME Terminal）上可用的256种颜色。
```shell
#!/bin/bash
 
# This program is free software. It comes without any warranty, to
# the extent permitted by applicable law. You can redistribute it
# and/or modify it under the terms of the Do What The Fuck You Want
# To Public License, Version 2, as published by Sam Hocevar. See
# http://sam.zoy.org/wtfpl/COPYING for more details.
 
for fgbg in 38 48 ; do # Foreground / Background
    for color in {0..255} ; do # Colors
        # Display the color
        printf "\e[${fgbg};5;%sm  %3s  \e[0m" $color $color
        # Display 6 colors per lines
        if [ $((($color + 1) % 6)) == 4 ] ; then
            echo # New line
        fi
    done
    echo # New line
done
 
exit 0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019061914383268.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2MzIxNjU=,size_16,color_FFFFFF,t_70)

## 参考
[Linux console codes manual (''man console_codes'')](http://linux.die.net/man/4/console_codes)
[XTerm Control Sequences](http://invisible-island.net/xterm/ctlseqs/ctlseqs.html)

