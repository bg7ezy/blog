---
title: HackRF 扫描LTE基站 支持中国的TDD-LTE
date: 2017-10-19 16:07:06
tags:
categories:
zhuanzai: http://www.hackrf.net/2014/04/hackrf-lte-scan/
---

LTE-Cell-Scanner的TDD功能支持作者[jxj同学](http://sdr-x.github.io/)目前完成了该软件对于HackRF的支持。

据作者说HackRF的效果明显比rtlsdr要好不少，噪声系数彽了很多。

技术细节见[为LTE小区搜索程序加入HackRF支持（使用纯C/C++语言操作HackRF）](http://www.hackrf.net/2014/04/lte-hackrf/)

 

以下是jxj同学发布的新闻:

LTE小区搜索软件加入对HACKRF的支持，有望将来解调LTE SIB信息。

感谢 [http://hackrf.net/](http://hackrf.net/) 借给我HackRF板供调试！

OpenCL加速的 TDD/FDD LTE小区搜索与跟踪源代码: [https://github.com/JiaoXianjun/LTE-Cell-Scanner](https://github.com/JiaoXianjun/LTE-Cell-Scanner)

因为HackRF带宽可达20MHz，远高于rtl-sdr电视棒，因此有了HackRF，将来就可能加入解码LTE SIB信息的功能（程序原来主要针对rtl-sdr电视棒设计，受限于rtl-sdr电视棒带宽，只能解码LTE MIB）

视频演示
（国内） [http://v.youku.com/v_show/id_XNjk3Mjc1MTUy.html](http://v.youku.com/v_show/id_XNjk3Mjc1MTUy.html)
（国外） [http://www.youtube.com/watch?v=3hnlrYtjI-4](http://www.youtube.com/watch?v=3hnlrYtjI-4)


<iframe height=498 width=510 src="http://player.youku.com/embed/XMTY1MTI3NjMyNA==" frameborder=0 allowfullscreen></iframe>
