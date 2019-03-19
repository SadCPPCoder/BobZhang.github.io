---
title: 读取文件时，使用file.eof()判断结尾注意事项
date: 2017-06-21 09:24:50
tags:
	- C++
	- IO
categories:
	- 技术
	- C++
---
# 读取文件时，使用file.eof()判断结尾注意事项

今天写一个小功能需要读取文件，在判断文件结尾时使用了以下语句：
``` cpp
while(infile.eof() && infile.good())
{
    infile.read((char*)&data, 1);
    encodeData = data ^ CODE;
    outfile.write((char*)&encodeData, 1);
}
```
**结果最后一个字符会读取两次，后来换了种方式，事先读取文件长度，再循环，就能争取读取文件**：
``` cpp
for(int i=3; i<file_len; ++i)
{
    infile.read((char*)&data, 1);
    encodeData = data ^ CODE;
    outfile.write((char*)&encodeData, 1);
}
```
<font color=#ff0000>最后了解到，是因为read函数需要读取失败时才将eofbit置位，所以置位时，循环还会执行一次。修改代码就没有问题了：</font>
``` cpp
while(infile.read((char*)&data, 1) && !infile.eof() && infile.good())
{
    encodeData = data ^ CODE;
    outfile.write((char*)&encodeData, 1);
}
```