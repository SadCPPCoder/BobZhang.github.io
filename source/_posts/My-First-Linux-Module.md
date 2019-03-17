---
title: My First Linux Module
date: 2019-03-17 17:48:29
tags:
	- Linux
	- Kernel Module
categories:
	- 技术
	- Linux
---
# My First Linux Module
Today, I successfully build my first linux hello module.
So, I write this blog to record this process.

1. First of all add a directory named hello in the kernel/driver, and add a file hello.c, write codes like bellow:

<!--more-->
```c
#include <linux/init.h>
#include <linux/module.h>

static int __init hello_init(void)
{
    printk(KERN_ERR " Hello, world!\n");
    return 0;
}
static void __exit hello_exit(void)
{
    printk(KERN_ERR " Goodbye, world!\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_AUTHOR("Bob, Zhang");
MODULE_LICENSE("Dual BSD/GPL");

MODULE_DESCRIPTION("A simple hello world demo");
MODULE_ALIAS("A simple module");
```

2. Then create a Kconfig file:
```
config HELLO
    tristate "HELLO WORLD Driver!"
    default m
    help
        HELLO WORLD
```
3. And create a Makefile file:
```
obj-m += hello.o
```
4. Next Add the Kconfig and Makefile into the Kconfig file and Makefile file in parent directory.

5. Finally run the commands bellow:
```
make ARCH=arm CROSS_COMPILE=$tool_prefix my_kernel_defconfig
make ARCH=arm CROSS_COMPILE=$tool_prefix modules
mkdir ./moduls_temp
make ARCH=arm CROSS_COMPILE=$tool_prefix modules_install INSTALL_MOD_PATH=./modules_temp
```

At last, the demo run like this:
{% asset_img example.png %}