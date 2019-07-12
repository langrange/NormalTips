Reference from [https://www.cnblogs.com/yangg518/p/5842651.html](https://www.cnblogs.com/yangg518/p/5842651.html)

## base points
库是一种可执行代码的二进制形式，可以被操作系统载入内存执行；
编译器 汇编器 链接器

Linux有两种库：静态库和共享库（动态库）
二者的区别在于**被载入的时刻不同**
静态库的代码**在编译过程中已经被载入可执行程序**，因此体积较大。
共享库的代码是在可执行程序运行时才载入内存的，**在编译过程中仅简单的引用**，因此代码体积较小。

## 库的产生
.a是静态库在Linux的后缀，它的产生分为两步：
+ Step 1. 源文件编译生存一堆.o
+ Step 2. ar命令将很多-o转换成.a

动态库的后缀是.so，它由gcc加特定参数编译产生

## 库文件的命名
在linux下，库文件一般放在/usr/lib和/lib下，
静态库的名字一般为libxxxx.a，其中xxxx是该lib的名称
动态库的名字一般为libxxxx.so.major.minor，xxxx是该lib的名称，major是主版本号， minor是副版本号

## 查看可执行文件依赖的库
ldd命令可以查看一个可执行程序依赖的共享库，
例如# `ldd /bin/lnlibc.so.6`
=> /lib/libc.so.6 (0×40021000)/lib/ld-linux.so.2
=> /lib/ld- linux.so.2 (0×40000000)
可以看到ln命令依赖于libc库和ld-linux库

## 如何定位共享库文件
当系统加载可执行代码时候，能够知道其所依赖的库的名字，但是还需要知道绝对路径。
此时就需要系统动态载入器(dynamic linker/loader)
对于elf格式的可执行程序，是由`ld-linux.so`来完成的，它先后搜索elf文件的 DT_RPATH段—环境变量LD_LIBRARY_PATH—/etc/ld.so.cache文件列表—/lib/,/usr/lib目录找到库文件后将其载入内存
如：export LD_LIBRARY_PATH=’pwd’
将当前文件目录添加为共享目录

## 如何让系统找到自己写的库
如果安装在/lib或者/usr/lib下，那么ld默认能够找到，无需其他操作。
如果安装在其他目录，需要将其添加到/etc/ld.so.cache文件中，步骤如下
+ Step 1. 编辑/etc/ld.so.conf文件，加入库文件所在目录的路径
+ Step 2. 运行ldconfig，该命令会重建/etc/ld.so.cache文件

# 用gcc生成静态和动态链接库的示例
我们通常把一些公用函数制作成函数库，供其它程序使用。
函数库分为静态库和动态库两种

## 示例：
hello.h 为函数库的头文件  
hello.c 为函数库的源程序
main.c 为测试库文件的主程序

程序1: hello.h
```
#ifndef HELLO_H 
#define HELLO_H 
  
void hello(const char *name); 
  
#endif
```
程序2：hello.c
```
#include <stdio.h> 
void hello(const char *name) { 
        printf("Hello %s!\n", name); 
}
```
程序3：main.c
```
#include "hello.h" 
 int main() 
 { 
     hello("everyone"); 
     return 0; 
 }
```
**思路分析：**
+ 通过编译多个源文件，直接将目标代码合成一个.o文件。
+ 通过创建静态链接库libmyhello.a，使得main函数调用hello函数时可调用静态链接库。
+ 通过创建动态链接库libmyhello.so，使得main函数调用hello函数时可调用静态链接库。

## 思路一：编译多个源文件  
`gcc -c hello.c`  
`gcc -c main.c`  
`gcc -o a hello.o main.o`  

## 思路二：静态库链接
`ar rcs libmyhello.a hello.o`  
`gcc main.c -static -L. -lmyhello`

未完待续...
