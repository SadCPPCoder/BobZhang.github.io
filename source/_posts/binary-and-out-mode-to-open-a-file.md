---
title: binary and out mode to open a file
date: 2017-12-21 10:17:34
tags:
	- C++
	- Input/Output
categories:
	- 技术
	- C++
---
# binary and out mode to open a file
When I use binary and out mode to open a exist file, and to modify the 4th and 8th byte data to 0x78, I found that the whole file will be rewrite, like below:
``` cpp
#include <fstream>
#include <iostream>
using namespace std;
 
int main(int argc, char **argv)
{
    fstream out;
    out.open("./test.txt", ios_base::binary | ios_base::out);
 
    uint8_t data = 0x78;
 
    out.seekp(4, ios_base::beg);
    out.write((char *)&data, 1);
 
    out.seekp(8, ios_base::beg);
    out.write((char *)&data, 1);
 
    out.close();
    return 0;
}
```
<!--more-->
And the file original data is :

```
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
```

But the file modified is:

```
00 00 00 00 78 00 00 00 78
```

I have tried ios::app and ios::ate, the ios::app will out put my data at the end of the file and the skeep do not work.

Finally, I thought out that the file is just a out file, so the api will not see the original data of the file, so I should add the ios::in.

``` cpp
#include <fstream>
#include <iostream>
using namespace std;
 
int main(int argc, char **argv)
{
    fstream out;
    out.open("./test.txt", ios_base::binary | ios_base::out | ios_base::in);
 
    uint8_t data = 0x78;
 
    out.seekp(4, ios_base::beg);
    out.write((char *)&data, 1);
 
    out.seekp(8, ios_base::beg);
    out.write((char *)&data, 1);
 
    out.close();
    return 0;
}
```

And finally, I got the right data.

```
FF FF FF FF 78 FF FF FF 78 FF FF FF FF FF FF FF
```