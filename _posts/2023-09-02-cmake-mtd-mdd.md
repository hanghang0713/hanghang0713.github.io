---
layout: post
title:  "MTD 与 MDD 的区别"
tags: C++ compiler-params
category: Tools 
---

# MTD 与 MDD 的区别 {ignore=true}

[toc]

## 背景：
> error LNK2038: 检测到“RuntimeLibrary”的不匹配项: 值“MTd_StaticDebug”不匹配值“MDd_DynamicDebug”

## MTd 

## MDd

## 如何选择

## cmake 中如何设置

`set(CMAKE_CXX_FLAGS_RELEASE "/MT")`
`set(CMAKE_CXX_FLAGS_DEBUG "/MTd")`


`set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")`

## TODO：
两种语法的区别
