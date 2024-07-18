---
layout: post
title:  "智能指针"
tags: C++ smart pointer 
category: cplusplus basic 
---

# C++ 的智能指针 {ignore=true}
> Smart pointers enable automatic, exception-safe, object lifetime management.
[cppreference](https://en.cppreference.com/w/cpp/memory)

cppreference 中说明了智能指针的作用，支持 自动、异常安全的对象生命周期管理。



## 智能指针是什么

## 智能指针主要是为了解决什么问题

智能指针主要是为了解决 C++ 内存管理困难，内存泄漏，期望能够实现自动管理内存的目的。
该指针主要用于确保程序不存在内存和资源泄漏且是异常安全的。

> 其他语言是如何管理内存的?
[Rust 是如何管理内存的](https://learn.microsoft.com/zh-cn/training/modules/rust-memory-management/?source=recommendations) 


## 智能指针是如何解决该问题的，它的实现原理是什么


#### unique_ptr 的实现
unique_ptr 通过保存对象的所有权并独占，不支持拷贝。
![unique_ptr](/assets/images/cplusplus-basic/unique_ptr.png "智能指针")


#### shared_ptr 的实现
shared_ptr 通过指针保存对象的共享所有权，多个 shared_ptr 可能指向同一个对象。
释放的场景
引用计数的实现，通过使用一个指针指向同一块内存并且使用原子量进行操作。

![shared_ptr](/assets/images/cplusplus-basic/shared_ptr.png "智能指针的实现")


#### weak_ptr 的实现
weak_ptr 持有一种指向由 shared_ptr 管理的对象的弱引用。
需要访问对象时，必须先转化成 shared_ptr.
weak_ptr 一般有两种作用
- 当一个对象只在存在时才需要访问时，并且随时可能被人删除时。weak_ptr 可以跟踪该对象并获取临时所有权
- 当 shared_ptr 管理的对象陷入循环引用时，将一个对象设置为 weak_ptr


## 智能指针是否引入了新的问题，即它存在的缺陷是什么




## enable_shared_from_this
