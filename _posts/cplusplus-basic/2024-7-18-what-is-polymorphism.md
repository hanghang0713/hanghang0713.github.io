---
layout: post
title:  "C++ 中的多态"
tags: C++ 
category: cplusplus basic 
---

# C++ 中的多态 {ignore=true}

多态，一个通用的定义是某事物以几种不同的形式出现。
在编程语言理论和类型论中，多态是使用单个符号来表示多个不同的类型。
C++ 中，多态可以分为编译时多态和运行时多态

## C++ 中的多态是什么

## 为什么需要多态，它解决了什么问题

## 多态是如何实现的
编译时多态和运行时多态

#### 编译时多态
通过函数和操作符重载,或者函数模板

**重载（overloading）**
> C++ lets you specify more than one function of the same name in the same scope. These functions are called overloaded functions, or overloads. Overloaded functions enable you to supply different semantics for a function, depending on the types and number of its arguments.
> [Function Overloading](https://learn.microsoft.com/en-us/cpp/cpp/function-overloading?view=msvc-170)

为了编译函数调用，编译器首先会执行 `名称查找 name lookup`，对于函数来说，可能涉及 参数相关的查找 `argument-dependent lookup`。对于函数模板，可能涉及模板参数推导 `template argument deduction`
因此当同一个名称，执行多个实体的时候，就称之为重载，编译器必须确定使用哪个重载，简单来说就是参数与实参最匹配的重载。
配置最合适的重载的过程就叫做重载决议 `overload resolution`
- 构建候选的函数集
- 修建函数集,得到可行的函数集
- 分析集合,确定单个最佳的可行函数

**函数模板**
函数模板定义了一系列的函数
``` C++
template < parameter-list > function-declaration
```

> 函数模板本身不是类型或函数。仅包含模板定义的源文件不会生成任何代码。为了出现任何代码，必须实例化模板：必须确定模板参数，以便编译器可以从类模板生成实际的函数

###### 显示实例化
显示实例化包括下面这些格式:
- ```template return-type name < argument-list > ( parameter-list );``` 
- ```template return-type name ( parameter-list ) ;```		
- ```extern template return-type name < argument-list > ( parameter-list);```	
- ```extern template return-type name ( parameter-list ) ;```

显式实例化定义强制实例化它们所引用的函数或成员函数。它可以出现在程序中模板定义之后的任何位置