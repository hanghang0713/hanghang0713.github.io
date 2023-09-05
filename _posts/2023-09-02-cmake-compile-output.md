---
layout: post
title:  "CMake 编译命令的输出"
tags: C++ CMake
category: Tools
---

# CMake 编译命令的输出信息 {ignore=true}

[toc]

## 语法
`set(CMAKE_EXPORT_COMPILE_COMMANDS ON)`

## 作用
主要用于生成 `compile_commands.json` 文件，

## 注意
主要用于 `Makefile Generators` 和 `ninja Generators`, 其他编译器会忽略该选项