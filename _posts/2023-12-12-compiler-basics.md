---
layout: post
title: 编译的过程
description: 程序员的自我修养-链接，装载与库
tags: C++ compiler windows
category: C++ 
---

# 编译和链接
一个 Hello World 程序编译运行一般需要进行四个步骤:预处理，编译，汇编，链接
![compilation process](/assets/images/compile-basic/compilation_process.png "编译过程分解")


## 预处理
预处理过程主要处理源码文件中以 "#" 开始的预编译指令，例如 `#include`, `#define`

预编译后文件名后族是 `.i`。windows 下直接执行下面的预编译指令可以得到一个预编译后的文件，例如 `hello.i`. 
```powershell
cl /P /C .\hello.cpp 
```

### 预处理后发生了什么
![compile file size](/assets/images/compile-basic/compile_file_size.png "预编译文件大小对比")

首先看一下预处理后的文件对比，源码文件只有 1KB 不到，但是预编译后的产物竟然有 1658KB。

![source file](/assets/images/compile-basic/source_file.png "源码文件")
![process file 1](/assets/images/compile-basic/process_file_1.png "预编译文件内容头部")
![process file 2](/assets/images/compile-basic/process_file_2.png "预编译文件内容尾部")

对比 `hello.cpp` 与 `hello.i` 文件，可以发现: 
- 我们在源码文件中使用 `#include <iostream>` 包含的头文件被直接拷贝到了预处理后的文件中。
- 注释（通过不同的选项可以选择保留或去掉）被删掉了
- `#define` 定义的宏被展开在了所有使用的地方，删除了原来的 `#define`
- 添加了 `#line` 说明行号和文件名
- 处理了 `#ifdef` 等预编译指令，可以看到预编译后的文件中，实际并没有不满足条件的代码
- 保留了 `#pragma` 编译器指令

总结上面文件对比得出的结论，可以看到实际效果与书中（《程序员的自我修养》）的一致：
- 将所有的 `#define` 删除，展开宏定义
- 处理条件编译预编译指令 `#if`, `#ifdef`
- 处理 `#include` 预编译指令，将包含的文件插入到指令的位置
- 删除所有的注释
- 添加行号和文件名标识
- 保留所有的 #pragma 编译器指令，留给编译器使用 

## 编译
编译就是将预编译后的文件进行一系列的词法分析，语法分析，语义分析以及优化，并生成对应的汇编代码文件。
```powershell
cl /FA /EHsc /c .\hello.cpp
```
通过上面的命令可以生成汇编文件 `hello.asm`(/EHsc 指定异常处理模型。 /c 指定在不链接的情况下进行编译)。
![asm file size](/assets/images/compile-basic/asm_file_size.png "汇编文件大小对比")

对比上面的预编译文件大小，可以发现汇编后的文件大小缩小了很多。
（TODO: 了解中间做了哪些动作导致文件的变化）

## 汇编
汇编器是将汇编代码转变为机器可以执行的指令。经过预编译，编译，和汇编之后会输出目标文件

## 链接
汇编过程为什么不能直接输出可执行文件而是目标文件？
链接过程到底包含哪些内容？
为什么需要链接？





## 相关
- c1060 错误是什么
- cl.exe 为什么会消耗掉这么多内存
- 编译器的内存限制是多少
