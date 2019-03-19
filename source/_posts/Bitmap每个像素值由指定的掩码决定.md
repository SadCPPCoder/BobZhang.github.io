---
title: Bitmap每个像素值由指定的掩码决定
date: 2017-06-22 18:31:59
tags:
	- C++
	- Bitmap
categories:
	- 技术
	- 图像
	- C++
---
# Bitmap每个像素值由指定的掩码决定

前面写了一个小工具来转换Bitmap到不同的颜色深度，程序主要的逻辑是：
`Bitmap文件头 + Bitmap信息头 + Bitmap数据`。
在RGB32（RGB888）转RGB565的过程中也有遇到一个小问题：输出的颜色和原图的颜色显示不正确，但是像素值是正确的。
但是使用GIMP导出的图片又是正确的。

通过比较发现内容上有一定的差异，如下图，左边是不正确的图片，右边是正确的图片：
{% asset_img compare.png %}

<font size="2px" color=#ff0000>**注:**图中使用的是[MadEdit](https://sourceforge.net/projects/madedit/ "MadEdit")工具查看二进制数据。</font>

比较发现差异的地方在于：***biCompression = 3***

查阅资料发现：表示每个像素值由指定的掩码决定。

而多余的16个字节依次就是RGBA的颜色掩码，这个掩码来与颜色值进行与运算，得到颜色值。 