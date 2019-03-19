---
title: '巧用std::shared_ptr全局对象释放单例内存'
date: 2018-08-27 18:01:24
tags:
	- C
	- C++
categories:
	- 技术
	- C++
---
## 单例内存释放
在程序中如果存在者单例，同时它们还需要执行delete等操作的话，我们通常在main的末尾去掉用单例的析构函数，或着利用编译器的属性，来设置main函数后执行的函数```__attribute((destructor)) ```。
但是如果单例并不是在main函数所在的文件中定义的，那么就还需要想办法在main函数中访问它们，显得相对麻烦。

## 全局对象析构
在系统中全局对象都是由系统负责析构的，所以全局变量的析构函数，会在程序停止运行后，由系统调用执行。
``` cpp
#include <iostream>

class Test
{
public:
    Test(int a)
    {
        std::cout << "Test Input: " << a << "." << std::endl;
    }

    ~Test()
    {
        std::cout << "Test, destructor!" << std::endl;
    }
};

Test test(88);

int main( void )
{
    std::cout << "main...." << std::endl;
}
```
<!--more-->
上面代码执行结果如下：
```
zhangbob@ubuntu:~/code/test$ ./main
Test Input: 88.
main....
Test, destructor!
```
所以我们可以利用全局变量的这个特性，来析构单例对象。

## 巧用std::shared_ptr
但是如果每一次都需要写一个全局变量来负责析构单例对象，那么还是比较麻烦，因此便有了利用全局变量由系统负载析构的特点，定义一个全局str::shared_ptr对象，对象指定对单例进行析构。
因此不需要我们关心何时析构单例，也不需要添加很多代码，堪称方便。
* SingleObject.hpp
``` cpp
#ifndef SINGLEOBJECT_H
#define SINGLEOBJECT_H
 
class SingleObject
{
public:
    static SingleObject& GetInstance();
    static SingleObject* GetInstancePtr();
    static void DoSomeThing();
    static void DelObj();
private:
    static void NewObj();
    SingleObject();
    ~SingleObject();
     
    static SingleObject* msObj;
};
 
#endif // SINGLEOBJECT_H
```
* SingleObject.cpp
``` cpp
#include "SingleObject.hpp"
#include <iostream>
#include <memory>
 
std::shared_ptr<SingleObject> SingleObjectDeletor( SingleObject::GetInstancePtr(), [](SingleObject *){ SingleObject::GetInstance().DelObj(); });
 
SingleObject* SingleObject::msObj = NULL;
 
SingleObject::SingleObject()
{
    std::cout << "This is the constructor of SignleObject class!\n";
}
 
SingleObject::~SingleObject()
{
    std::cout << "This is the destructor of SingleObject class!\n";
}
 
void SingleObject::NewObj()
{
    if( NULL == msObj )
    {
        // lock
        msObj = new SingleObject();
         
        // unlock
    }
     
    std::cout << "New an object, address: " << +msObj << std::endl;
}
 
void SingleObject::DelObj()
{
    if( NULL != msObj )
    {
        delete msObj;
        std::cout << "Delete an object!\n";
    }
}
 
SingleObject* SingleObject::GetInstancePtr()
{
    if( NULL == msObj )
    {
        NewObj();
    }
     
    return msObj;
}
 
SingleObject& SingleObject::GetInstance()
{
    if( NULL == msObj )
    {
        NewObj();
    }
     
    return *msObj;
}
 
void SingleObject::DoSomeThing()
{
    std::cout << "SingleObject do some thing!\n";
}
```
main.cpp
``` cpp
#include <iostream>
#include "SingleObject.hpp"
 
int main(int argc, char **argv)
{
    SingleObject::GetInstance().DoSomeThing();
     
    return 0;
}
```
最后的输出结果参见下图：
{% asset_img result.png %}