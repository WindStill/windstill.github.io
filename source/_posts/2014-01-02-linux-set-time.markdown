---
layout: post
title: "Linux设置系统时间和硬件时间"
date: 2014-01-02 11:46:08 +0800
comments: true
categories: Linux
---

今天发现我们测试环境的服务器时间不对，居然超前了。然后谷歌了下linux系统相的关时间查看与设置。Linux时钟分为 `系统时钟`（System Clock）和 `硬件时钟`（Real Time Clock，简称RTC）。`系统时钟是`指当前Linux Kernel中的时钟，而`硬件时钟`则是主板上由电池供电的时钟，这个硬件时钟可以在BIOS中进行设置。当Linux启动时，硬件时钟会去读取系统时钟的设置，然后系统时钟就会独立于硬件运作。

<!-- more -->

Linux中的所有命令都是采用的系统时钟设置。在Linux中，用于时钟查看和设置的命令主要有`date`、`hwclock`和`clock`。其中，`clock`和`hwclock`用法相近，只用一个就行，只不过clock命令除了支持x86硬件体系外，还支持Alpha硬件体系。看了下我们的测试环境止只有hwclock可用，clock没有安装。

###1. date 
* 查看系统时间

```
# date
```

* 设置系统时间

```
# date --set "01/02/14 10:19:20" (月/日/年 时:分:秒)
```
###2. hwclock/clock
* 查看硬件时间

```
# hwclock --show
或者
# clock --show
```

* 设置硬件时间

```
# hwclock --set --date="01/02/14 10:19:20"  (月/日/年 时:分:秒)
或者
# clock --set --date="01/02/14 10:19:20"  (月/日/年 时:分:秒)
```

###3. 硬件时间和系统时间的同步

硬件时钟同步到系统时钟：

```
# hwclock --hctosys（hc代表硬件时间，sys代表系统时间）
或者
# clock --hctosys
```

系统时钟同步到硬件时钟：

```
# hwclock --systohc
或者
# clock --systohc
```

###4. 时区的设置

```
root@testserver-01:~# tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent or ocean.
 1) Africa
 2) Americas
 3) Antarctica
 4) Arctic Ocean
 5) Asia
 6) Atlantic Ocean
 7) Australia
 8) Europe
 9) Indian Ocean
10) Pacific Ocean
11) none - I want to specify the time zone using the Posix TZ format.
#? 5 (输入5，亚洲)

Please select a country.
 1) Afghanistan		  18) Israel		    35) Palestine
 2) Armenia		  19) Japan		    36) Philippines
 3) Azerbaijan		  20) Jordan		    37) Qatar
 4) Bahrain		  21) Kazakhstan	    38) Russia
 5) Bangladesh		  22) Korea (North)	    39) Saudi Arabia
 6) Bhutan		  23) Korea (South)	    40) Singapore
 7) Brunei		  24) Kuwait		    41) Sri Lanka
 8) Cambodia		  25) Kyrgyzstan	    42) Syria
 9) China		  26) Laos		    43) Taiwan
10) Cyprus		  27) Lebanon		    44) Tajikistan
11) East Timor		  28) Macau		    45) Thailand
12) Georgia		  29) Malaysia		    46) Turkmenistan
13) Hong Kong		  30) Mongolia		    47) United Arab Emirates
14) India		  31) Myanmar (Burma)	    48) Uzbekistan
15) Indonesia		  32) Nepal		    49) Vietnam
16) Iran		  33) Oman		    50) Yemen
17) Iraq		  34) Pakistan
#? 9 (输入9，中国)

Please select one of the following time zone regions.
1) east China - Beijing, Guangdong, Shanghai, etc.
2) Heilongjiang (except Mohe), Jilin
3) central China - Sichuan, Yunnan, Guangxi, Shaanxi, Guizhou, etc.
4) most of Tibet & Xinjiang
5) west Tibet & Xinjiang
#? 1 (输入1，北京时间)

The following information has been given:

	China
	east China - Beijing, Guangdong, Shanghai, etc.

Therefore TZ='Asia/Shanghai' will be used.
Local time is now:	Thu Jan  2 11:55:36 CST 2014.
Universal Time is now:	Thu Jan  2 03:55:36 UTC 2014.
Is the above information OK?
1) Yes
2) No
#? Yes
Please enter 1 for Yes, or 2 for No.
#? 1 (输入1，确认)
```

###参考文章
[Linux操作系统下的时间设置方法介绍](http://os.51cto.com/art/200703/43943.htm)