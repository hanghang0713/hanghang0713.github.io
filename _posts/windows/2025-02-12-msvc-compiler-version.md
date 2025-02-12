---
layout: post
title:  "windwos 中编译器版本号的问题"
tags: C++ windows msvc
category: msvc 
---

# windwos 中编译器版本号的问题

[toc]

## 背景
因为最近从 conan1 迁移到 conan2, 其中的编译器参数和编译器的版本发生了变化，发现对这些版本号和部分概念不是很清晰，因此在这里记录一下。

conan1 中对应的 compiler 参数为 `visual studio` 对应的 compiler.version 为 `17` 等 
conan2 中对应的 compiler 参数为 `msvc` 对应的 compiler.version 为 `194`

而对应的关系为
- 191 对应 Visual Studio 2017
- 192 对应 Visual Studio 2019
- 193 对应 Visual Studio 2022 (早期版本)
- 194 对应 Visual Studio 2022 (更新版本)

## 其他概念
vcvars_ver = 14.3


## 引用
https://viml.nchc.org.tw/lots-of-version-number-of-visual-studio/