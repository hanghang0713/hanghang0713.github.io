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