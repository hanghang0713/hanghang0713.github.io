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