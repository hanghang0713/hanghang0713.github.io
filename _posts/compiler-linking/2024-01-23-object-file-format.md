---
layout: post
title: 目标文件里面有什么
description: 程序员的自我修养-链接，装载与库
tags: C++ compiler windows
category: C++ 
---

## 目标文件的格式
编译器编译后产生的文件就叫做**目标文件**，目标文件从结构上将，就是已经编译后的可执行文件的格式。

### 可执行文件格式
现在流行的可执行文件格式包括 `PE-COFF(windows)` 与 `ELF(linux)`，它们都是 `COFF` 格式的变种. 目标文件就是编译后但是未链接的中间文件(windows 下的 .obj 与 Linux 下的 .o)。

不止是目标文件，动态链接库(DLL, dynamic Linking Library, windows 下的 dll 和 linux 下的 .so)与静态库(Static Linking Library, windows 下的 .lib 与 linux 下的 .a)都按照可执行文件的格式进行存储。

![object file format](/assets/images/compile-basic/object_file.png "目标文件格式")

### 目标文件结构
目标文件的内容至少包含有被编译后的机器指令代码，数据，符号表，调试信息，字符串等。一般情况下，目标文件会按照不同的属性，以 `Section` 的形式存储。
机器指令经常被放在 `代码段`，全局变量和局部静态变量通常放在 `数据段`。
总体来说，程序源代码被编译后主要分为两种段：程序指令和程序数据。代码段属于程序指令。数据段和 `.bss` 属于程序数据。

> 指令和数据分开存放的好处是什么
> 程序和指令被映射到两个不同的需存区域，可以分别设置权限，避免只读的指令被无意间修改
> CPU 缓存一般是指令缓存和数据缓存分开，分开存储可以提高缓存的命中率
> 当运行多个程序的副本时，只读资源可以共享。可以节省大量空间


### 目标文件的内容 
![object file format section](/assets/images/compile-basic/obj_file_section.png "一个简单的目标文件的段信息")

从上面的信息可以看到，常见的段信息有:
- .text 代码段
- .data 数据段
- .bss 存放未初始化的全局变量和局部静态变量，不占据实际的空间
- .rodata 只读数据段
- comment 注释信息段
- .note.gnu.property 堆栈提示段

![onject file structure](/assets/images/compile-basic/section-structure.png "段的结构示意图")

#### 代码段

> objdump 常用参数
> -s 将所有段的内容通过十六进制的方式打印出来
> -d 将包含指令的段进行反汇编

![text section](/assets/images/compile-basic/text-section.png "代码段")

可以看到 .text 段里面包含的正是 SimpleSection.c 里面两个函数的指令。

#### 数据段和只读数据段
![data section](/assets/images/compile-basic/data-section.png "数据段")

.data 段存放的是 **已经初始化的全局静态变量和局部静态变量**。
.data 段大小是 8，正好是源码中的 `global_init_var` `static_var` 中的两个变量的大小。每个变量4个字节。
.data 段中 0x54 0x00 0x00 0x00 对应的正是 `global_init_var` 的值 84
.data 段中 0x40 0x00 0x00 0x00 对应的正是 `static_var` 的值 64

.rodata 段中存放的是只读数据，一般是只读变量和字符串常量。

> 单独设立 .rodata 段有很多好处
> 从语义上支持 const 语法
> 操作系统可以将 .rodata 映射成只读, 保证程序的安全性
> 嵌入式系统，可以将 .rodata 放入 ROM 只读存储器中，保证访问储存器的正确性

.rodata 段的大小是 4， 存放的是源码中的 "%d\n" 的字符串常量
.rodata 中的 `25640a00` 对应的正是这个字符串的 ASCII 字节序，最后以 `\0` 结尾