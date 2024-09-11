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

- CLI Command Line Interface
- GUI Graphical User Interface
- API Application Programming Interface
- ABI 






## 引用
[what-is-an-application-binary-interface-abi](https://stackoverflow.com/questions/2171177/what-is-an-application-binary-interface-abi)