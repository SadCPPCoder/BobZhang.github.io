---
title: Hide Data into bitmap with ARGB8888 format
date: 2017-12-21 15:38:25
tags:
	- C++
categories:
	- 技术
	- C++
---
# Hide Data into bitmap with ARGB8888 format

最近工作上接触Bitmap比较多，再加上之前就看到说如何利用图片的透明标志位来存放私密信息。
所以今天就动手写了这么个小工具，可以将一定长度的文本文件的内容隐藏到图片中保存起来，需要使用时再从图片中解码提取出来。

bitmap.h: 声明Bitmap的文件头结构，Bitmap类和数据隐藏类。
``` cpp
#ifndef BMP_H
#define BMP_H
 
#include <fstream>
#include <string>
#include <iostream>
 
#define DEBUG 0
 
#define LOG std::cout
#define HEX std::hex
#define DEC std::dec
 
#define FLAG_0 (0xFF)
#define FLAG_1 (0xEE)
#define FLAG_2 (0xDD)
#define FLAG_3 (0xCC)
 
#define DATA_CODE (0xCC)
 
#define BYTE_PER_PIXEL (4)
 
#pragma pack(push, 1)
typedef struct
{
    uint16_t bfType; // Type of bitmap, in windows, always 'BM'
    uint32_t bfSize; // Size of the bitmap file
    uint16_t bfReserved1; // reserved feild
    uint16_t bfReserved2; // reserved feild
    uint32_t bfOffBits; // the start address of the bitmap data
}BMP_FILE_HEADER;
 
typedef struct
{
    uint32_t biSize; // size of this struct
    uint32_t biWidth; // image width, unit is pixel
    uint32_t biHeight; // image height, unit is pixel
    uint16_t biPlanes; // alsways 1
    uint16_t biBitCount; // color depth of the image, 1, 4, 8, 16, 24 and 32
    uint32_t biCompression; // compression type, 0=no compress, 1=RLE8 compress
                            // 2=RLE4 compress, 3=pixel color mask
    uint32_t biSizeImage; // pixel data size of the image
    uint32_t biXPelsPerMeter; // H resolution, pixel per meter
    uint32_t biYPelsPerMeter; // V resolution, pixel per meter
    uint32_t biClrUsed; // color number that used, 0=all color
    uint32_t biClrImportant; // important color amount, 0=all color important
}BMP_INFO_HEADER;
#pragma pacn(pop)
 
class Bitmap
{
private:
    BMP_FILE_HEADER m_fileHeader;
    BMP_INFO_HEADER m_infoHeader;
 
    std::string m_bmpPath;
    bool m_parseFlag;
public:
    Bitmap(std::string path);
    ~Bitmap();
    bool bmpParse();
    uint32_t getBmpSize();
    uint32_t getBmpDataAddr();
    std::string getBmpPath();
    uint32_t getBmpPixelNum();
};
 
class DataHider
{
private:
    uint8_t m_flags[4];
    uint8_t m_length[4];
    Bitmap m_bitmapHeader;
    std::string m_dataFilePath;
 
    bool isAHider();
    uint32_t getFileSize(std::ifstream &in);
    bool readTransByte(std::fstream &file, uint8_t &val, int pos);
    bool writeTransByte(std::fstream &file, uint8_t val, int pos);
    void length2Byte(uint32_t length);
    uint32_t byte2Length();
public:
    DataHider(std::string dataPath, std::string bmpPath);
    ~DataHider();
    bool doEncode();
    bool doDecode();
};
 
#endif // BMP_H
```
<!--more-->
bitmap.cpp: 实现上述声明。
``` cpp
#include "bitmap.h"
 
Bitmap::Bitmap(std::string path)
: m_fileHeader(), m_infoHeader(), m_bmpPath(path), m_parseFlag(false)
{
}
 
Bitmap::~Bitmap()
{
}
 
bool Bitmap::bmpParse()
{
    std::ifstream in;
    in.open(m_bmpPath, std::ios::binary);
 
    in.read((char *)&m_fileHeader, sizeof(m_fileHeader));
    /*char * p = (char *)&m_fileHeader;
    for(uint32_t i=0; i<sizeof(m_fileHeader); ++i)
    {
        in.read(p, 1);
        LOG << HEX << (int)*p << " ";
        ++p;
    }*/
    if( !in.good() )
    {
        LOG << "read bitmap file head error!\n";
        in.close();
        return false;
    }
 
    in.read((char *)&m_infoHeader, sizeof(m_infoHeader));
    /*    LOG << "read bitmap information header!\n";
    p = (char *)&m_infoHeader;
    for(uint32_t i=0; i<sizeof(m_infoHeader); ++i)
    {
        in.read(p, 1);
        LOG << HEX << (int)*p << " ";
        ++p;
    }*/
    if( !in.good() )
    {
        LOG << "read bitmap information header error!\n";
        in.close();
        return false;
    }
 
    in.close();
    m_parseFlag = true;
 
    #if 1 == DEBUG
        LOG << "Bitmap file header:"
            << "\nbfType: " << HEX << (int)m_fileHeader.bfType
            << "\nbfSize: " << (int)m_fileHeader.bfSize
            << "\nbfReserved1: " << (int)m_fileHeader.bfReserved1
            << "\nbfReserved2: " << (int)m_fileHeader.bfReserved2
            << "\nbfOffBits: " << (int)m_fileHeader.bfOffBits
            << "\nBitmap information header:"
            << "\nbiSize: " << (int)m_infoHeader.biSize << DEC << "\n";
    #endif
 
    return true;
}
 
uint32_t Bitmap::getBmpSize()
{
    if(!m_parseFlag)
    {
        LOG << "please do bitmap parse first!\n";
    }
    return m_fileHeader.bfSize;
}
 
std::string Bitmap::getBmpPath()
{
    return m_bmpPath;
}
 
uint32_t Bitmap::getBmpDataAddr()
{
    if(!m_parseFlag)
    {
        LOG << "please do bitmap parse first!\n";
    }
    return m_fileHeader.bfOffBits;
}
 
uint32_t Bitmap::getBmpPixelNum()
{
    return m_infoHeader.biWidth * m_infoHeader.biHeight;
}
 
DataHider::DataHider(std::string dataPath, std::string bmpPath)
: m_bitmapHeader(bmpPath), m_dataFilePath(dataPath)
{
}
 
DataHider::~DataHider()
{
}
 
bool DataHider::isAHider()
{
    return (m_flags[0] == FLAG_0) &
            (m_flags[1] == FLAG_1) &
            (m_flags[2] == FLAG_2) &
            (m_flags[3] == FLAG_3);
}
 
uint32_t DataHider::getFileSize(std::ifstream &in)
{
    if(!in.good())
    {
        LOG << "the data file has some error.\n";
        return false;
    }
 
    uint32_t pos =  in.tellg();
    in.seekg(0, std::ios_base::end);
    uint32_t size = in.tellg();
    in.seekg(pos, std::ios_base::beg);
 
    return size;
}
 
bool DataHider::readTransByte(std::fstream &file, uint8_t &val, int pos)
{
    file.seekg(m_bitmapHeader.getBmpDataAddr()+BYTE_PER_PIXEL*pos, std::ios_base::beg);
    #if DEBUG == 1
        LOG << "pos " << HEX << m_bitmapHeader.getBmpDataAddr()+BYTE_PER_PIXEL*pos << "\n";
    #endif
    file.read((char *)&val, 1);
    return file.good();
}
 
bool DataHider::writeTransByte(std::fstream &file, uint8_t val, int pos)
{
    file.seekp(m_bitmapHeader.getBmpDataAddr()+BYTE_PER_PIXEL*pos, std::ios_base::beg);
    file.write((char *)&val, 1);
    return file.good();   
}
 
void DataHider::length2Byte(uint32_t length)
{
    for(uint32_t i=0; i<4; ++i)
        m_length[i] = (uint8_t)(length >> (i*8)) & 0xFF;
    #if DEBUG == 1
        LOG << "length: " << length << "\n"
            << (int)m_length[0] << " "
            << (int)m_length[1] << " "
            << (int)m_length[2] << " "
            << (int)m_length[3] << "\n";
    #endif
}
 
uint32_t DataHider::byte2Length()
{
    uint32_t length = 0;
    uint32_t temp;
    for(uint32_t i=0; i<4; ++i)
    {
        temp = m_length[i];
    #if DEBUG == 1
        LOG << "temp: " << temp << "\n";
    #endif
        temp <<= (i*8);
    #if DEBUG == 1
        LOG << "temp: " << temp << "\n";
    #endif
        length += temp;
    }
 
    #if DEBUG == 1
        LOG << "length: " << length << "\n";
    #endif
 
    return length;
}
 
bool DataHider::doEncode()
{
    m_bitmapHeader.bmpParse();
     
    std::ifstream dataFile;
    dataFile.open(m_dataFilePath, std::ios::binary);
    if(!dataFile.good())
    {
        LOG << "open " << m_dataFilePath << " failed!\n";
        return false;
    }
    uint32_t dataSize = getFileSize(dataFile);
    if(dataSize > m_bitmapHeader.getBmpPixelNum() + 8) // flag size add length size
    {
        LOG << "data is too big to save into the bitmap!\n";
        dataFile.close();
        return false;
    }
 
    std::fstream bmpFile;
    bmpFile.open(m_bitmapHeader.getBmpPath(), std::ios_base::binary|
                    std::ios_base::out|std::ios_base::in);
                    // |std::ios_base::ate|std::ios_base::app
    if(!bmpFile.good())
    {
        LOG << "open " << m_bitmapHeader.getBmpPath() << " failed!\n";
        dataFile.close();
        return false;
    }
    dataFile.seekg(0, std::ios_base::beg);
 
    // flags
    m_flags[0] = FLAG_0;
    m_flags[1] = FLAG_1;
    m_flags[2] = FLAG_2;
    m_flags[3] = FLAG_3;
 
    for(uint32_t i=0; i<4; ++i)
        writeTransByte(bmpFile, m_flags[i], i);
 
    // length
    length2Byte(dataSize);
 
    for(uint32_t i=0; i<4; ++i)
        writeTransByte(bmpFile, m_length[i], i+4);
 
    // pos start at the 8th pixel
    int pos = 0;
    uint8_t data;
    while(pos < dataSize)
    {
        dataFile.read((char *)&data, 1);
        #if DEBUG == 1
            LOG << HEX << (int)data << " ";
        #endif
        data ^= DATA_CODE;
        writeTransByte(bmpFile, data, pos+8);
        ++pos;
    }
 
    bmpFile.close();
    dataFile.close();
    return true;
}
 
bool DataHider::doDecode()
{
    m_bitmapHeader.bmpParse();
     
    std::fstream bmpFile;
    bmpFile.open(m_bitmapHeader.getBmpPath(), std::ios_base::binary|std::ios_base::in);
    if(!bmpFile.good())
    {
        LOG << "open " << m_bitmapHeader.getBmpPath() << " failed!\n";
        return false;
    }
    bmpFile.seekg(0, std::ios_base::beg);
 
 
    // flags
    for(uint32_t i=0; i<4; ++i)
    {
        readTransByte(bmpFile, m_flags[i], i);
        #if DEBUG
            LOG << HEX << (int)m_flags[i] << " ";
        #endif
    }
 
    if( !isAHider() )
    {
        LOG << "this bitmap is not a hider file!\n";
        bmpFile.close();
        return false;
    }
 
    // length
    for(uint32_t i=0; i<4; ++i)
        readTransByte(bmpFile, m_length[i], i+4);
 
    #if DEBUG == 1
        LOG << (int)m_length[0] << " "
            << (int)m_length[1] << " "
            << (int)m_length[2] << " "
            << (int)m_length[3] << "\n";
    #endif
 
    uint32_t dataSize = byte2Length();
 
    std::ofstream dataFile;
    dataFile.open(m_dataFilePath, std::ios::binary);
    if(!dataFile.good())
    {
        LOG << "open " << m_dataFilePath << " failed!\n";
        bmpFile.close();
        return false;
    }
    // pos start at the 8th pixel
    int pos = 0;
    uint8_t data;
    while(pos < dataSize)
    {
        readTransByte(bmpFile, data, pos+8);
        data ^= DATA_CODE;
        dataFile.write((char *)&data, 1);
        ++pos;
         
    }
 
    bmpFile.close();
    dataFile.close();
    return true;
}
```

main.cpp: 主函数
``` cpp
#include <iostream>
#include "bitmap.h"
using namespace std;
 
int main(int argc, char **argv)
{
    cout << "Hello world!" << endl;
 
    if(argc < 4)
    {
        cout << "please type the right command!\n"
             << "DataHider encode ./data.txt ./ARGB8888.bmp\n"
             << "or DataHider decode ./data_out.txt ./ARGB8888.bmp\n";
        return -1;
    }
 
    string mode =  argv[1];
    string dataFile = argv[2];
    string bmpFile = argv[3];
 
    if(mode == "encode")
    {
        cout << "encoder\n";
        DataHider encoder(dataFile, bmpFile);
        encoder.doEncode();
    }
    else if(mode == "decode")
    {
        cout << "decoder\n";
        DataHider decoder(dataFile, bmpFile);
        decoder.doDecode();
    }
 
    return 0;
}
```