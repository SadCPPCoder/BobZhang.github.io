---
title: Bitmap RGB24 4字节对齐
date: 2017-06-22 09:54:15
tags:
	- C++
	- Bitmap
categories:
	- 技术
	- 图像
	- C++
---
# Bitmap RGB24 4字节对齐

因为Bitmap支持最少2种颜色，并且可以支持RLC等压缩方式，所以Bitmap在嵌入式领域使用还是比较广泛的。
最近在写一个Bitmap相关的小工具时，发现自己处理出来的Bitmap，系统总是不能正常地识别，但是读出来的数据又都是正确的。而且同样的图片，通过系统工具生成的Bitmap，总会大一些，大小也超过图片`LengthxWidthXColotBit + Header`。

## 问题

需求：**将RGB32或RGB24的Bitmap转换成为RGB565的Bitmap，并且可以将RGB565转成RGB32和RGB24。**
实现算法：

``` cpp
pixel565[0] = color[0] >> 3;// R
pixel565[1] = color[1] >> 2;// G
pixel565[2] = color[2] >> 3;// B
```
代码在RGB32、RGB24转RGB565和RGB565转RGB32过程中没有问题，但是在RGB565转RGB24时，出现了问题。
使用高宽均为10px的图片，转出来的图片，系统不能正确识别显示。但是用Windows绘图程序绘制出来的同样大小的图片却能正确显示。通过比较文件大小发现：
<table style="border=1 cellspacing=0"><tr><th>自己工具生成图片大小</td><th>绘图生成图片大小</td></tr><tr><td>374</td><td>354</td></tr></table>
<!--more-->
## 查阅

通过查阅资料，了解到这其实时Bitmap的一个特性，<font color=#ff0000>Bitmap每一行像素数据都要4字节对齐</font>，如果不够则用0x00填充。
对于10x10的图片，每行有10个像素，每个像素是24bit，所以一行数据是`10x24b=30B`，但是要对齐4字节的话需要将每行数据对齐到`32Byte`，总共有10行，所以会多出20Bytes。

## 解决

所以修改处理算法code：
``` cpp
int count = 0;
int alignNum = 4 - (pixelMat.cols % 4);
int alignData = 0;
MatConstIterator_<Vec3b> pixel, colorend;
for(pixel = pixelMat.begin<Vec3b>(), colorend = pixelMat.end<Vec3b>();pixel != colorend;++pixel)
{
    outFile.write((char*) &(*pixel)[0], 1);
    outFile.write((char*) &(*pixel)[1], 1);
    outFile.write((char*) &(*pixel)[2], 1);
    if(++count % pixelMat.cols == 0)
    {
        // 4 bytes align
        outFile.write((char *) &alignData, alignNum);
    }
}
```
**由此联想，在读取或存储Bitmap图片时需要关注图片一行数据的像素值所占的空间是否进行4字节对齐！**


