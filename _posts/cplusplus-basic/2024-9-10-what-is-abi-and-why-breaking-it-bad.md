---
layout: post
title: "What is ABI and why breaking it is bad"
tags: C++ 
categories: [C++, Basics]
---

# 什么是 ABI 

ABI 是 `Application Binary Interface` 的缩写。

在了解 ABI 之前，我们先看一下接口的定义：

> In computing, an interface is a shared boundary across which two or more separate components of a computer system exchange information. 
The exchange can be between software, computer hardware, peripheral devices, humans, and combinations of these.
Some computer hardware devices, such as a touchscreen, can both send and receive data through the interface, while others such as a mouse or microphone may only provide an interface to send data to a given system

在计算中，接口是计算机系统的两个或多个独立组件交换信息的共享边界。

>In some object-oriented languages, especially those without full multiple inheritance, the term interface is used to define an abstract type that acts as an abstraction of a class. 
It contains no data, but defines behaviours as method signatures. A class having code and data for all the methods corresponding to that interface and declaring so is said to implement that interface.
Furthermore, even in single-inheritance-languages, one can implement multiple interfaces, and hence can be of different types at the same time 

在面向对象的语言中，接口用来定义抽象类型，它不包含任何数据，但将行为定义为方法签名。

因此，接口是一种类型定义；在任何可以交换对象的地方（例如，在函数或方法调用中），可以根据其实现的接口或基类之一来定义要交换的对象的类型，而不是指定特定的类。这种方法意味着可以使用任何实现该接口的类。

**常见的接口类型**

> Interface: It is an "existing entity" layer between the functionality and consumer of that functionality.
 An interface by itself doesn't do anything. It just invokes the functionality lying behind.

更通俗的讲，我们在理解接口时，可以认为接口是在功能性和消费者之间的一个中间层，根据这个定义，我们可以分析一下常见的一些接口类型.

- **CLI Command Line Interface**
    - **functionality**: 用于解决某些目的的软件功能
    - **existing entities**: commands
    - **consumer**: user

- **GUI Graphical User Interface**
    - **functionality**: 用于解决某些目的的软件功能
    - **existing entities**: window, buttons etc
    - **consumer**: user

- **API Application Programming Interface**
    - **functionality**: 用于解决某些目的的软件功能
    - **existing entities**: functions, Interfaces
    - **consumer**: another program/application.

对于 ABI, 是否可以按照上面的格式来进行分析?

## 描述

在计算机软件中，应用程序二进制接口（ ABI ）是两个二进制程序模块之间的接口。通常，这些模块之一是库或操作系统设施，另一个是由用户运行的程序。

### ABI 里面包含什么
- 处理器的指令集，寄存器文件结构，堆栈组织，内存访问类型
- 数据类型，布局，对齐方式
- 调用约定，它控制了函数参数如何传递，以及如何检索返回值
- 系统调用号，应用程序应该如何对操作系统进行系统调用
- 目标文件，程序库的二进制格式

### 谁定义了 ABI 
对于 API 来说，一般是接口的提供者定义了 API 相关的信息。
对于 ABI 来说，一般是由操作系统和编译器定义了 ABI 相关的信息：
- 操作系统：操作系统定义了系统调用的 ABI，包括系统调用的编号、参数如何传递等。例如，Linux、Windows 等操作系统都有自己的系统调用 ABI。
- 编译器：编译器定义了函数调用的 ABI，包括参数如何传递、函数如何返回等。例如，GCC、Clang、MSVC 等编译器都有自己的函数调用 ABI。
- 运行时库：运行时库定义了数据类型、异常处理等的 ABI。例如，C++ 的运行时库定义了类的布局、虚函数表的组织、异常的抛出和捕获等 ABI

### 谁需要使用 ABI 
ABI 是两个二进制程序模块之间的接口，因此将这些模块进行打包的时候，需要用到 ABI 信息，这样才能正确的将多个对象文件链接在一起，所以一般是链接器需要使用到 ABI。

## 为什么会产生 ABI 不兼容的问题
对于 API 来说，当 API 发生变化的时候，可能就会产生不兼容的问题，比如函数签名的改变。
对于 ABI 来说，当 ABI 发生变化的时候，可能就会产生不兼容的问题。




## 引用
[what-is-an-application-binary-interface-abi](https://stackoverflow.com/questions/2171177/what-is-an-application-binary-interface-abi)
[Application binary interface](https://en.wikipedia.org/wiki/Application_binary_interface) 
