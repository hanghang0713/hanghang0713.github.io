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

## 为什么需要多态，它解决了什么问题
代码重复编写,提高了代码的通用性和可复用性

## C++ 中的多态是什么
编译时多态和运行时多态

### 编译时多态
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

###### 隐式实例化
当一段代码需要在定义了函数的上下文中引用这个函数时,并且函数还没有被显示实例化,就会发生隐式实例化.

```C++
#include <iostream>
 
template<typename T>
void f(T s)
{
    std::cout << s << '\n';
}
 
int main()
{
    f<double>(1); // instantiates and calls f<double>(double)
    f<>('a');     // instantiates and calls f<char>(char)
    f(7);         // instantiates and calls f<int>(int)
    void (*pf)(std::string) = f; // instantiates f<string>(string)
    pf("∇");                     // calls f<string>(string)
}
```

### 运行时多态
运行时多态是在运行时解析对象.通过虚函数和函数重写 (override) 来实现.

> [!note]  
> 如果需要实现运行时解析对象,那么在编译期就应该先用一个占位符类似的东西,比如指针.然后在实际运行时找到实际的对应的需要使用的对象

当派生类具有基类的成员函数之一的定义时，就会发生 `override`


#### 成员数据无法实现运行时多态
无法通过成员数据实现运行时多态，基类的引用永远执行基类的数据成员
```C+++
class Animal {
public:
    std::string color = "Black";
};

class Dog : public Animal {
public:
    std::string color = "Grey";
};

```
![data polymorphism](/assets/images/cplusplus-basic/data_polymorphism.png "成员数据无法实现多态性")

#### 基类和派生类中的同名函数
基类和派生类中的普通的同名成员函数也无法实现多态。因为静态链接。对该函数的调用仅由编译器设置一次，那就是位于基类中的函数。

> 利用该特性，有相关的技术 NVI

```C+++
class Animal {
public:
    void PrintClassName() {
        std::cout << "Animal" <<std::endl;
    }
};

class Dog : public Animal {
public:
    void PrintClassName() {
        std::cout << "Dog" <<std::endl;
    }
};

```
![non virtual function](/assets/images/cplusplus-basic/non-virtual-function.png "non virtual function")

#### 虚函数
虚函数是通过 `virtual` 关键字在基类中声明，并在派生类中重新定义(`overrride`) 的函数。
它告诉编译器执行后期绑定，并在运行时执行。

> 构造函数中不能使用虚拟调用的机制，因为派生类的重写还没有发生。

虚函数允许我们创建基类指针列表，并调用任何类型的派生类的方法，甚至不用关心具体的类型。

##### 那么编译器是如何进行运行时的决议的？
- vtable 存放函数指针的表，每个类维护一份
- vptr 指向 `vtable` 的指针，每个实例实例对象维护一份

![vtable and vptr](/assets/images/cplusplus-basic/vptr_vtable.png "虚函数表和虚函数表指针")

 编译器会在两个额外的地方添加代码来使用和管理虚函数表指针 `vptr`
 - 在每一个构造函数中，当一个对象正在被创建时，编译器会设置这个对象的 `vptr`，并将这个指针指到这个对象所属的类的 `vtable` 上
 - 在多态函数被调用的时候，编译器会首先使用基类的指针或者引用查找 `vptr`, 当找到 `vptr` 就可以访问它所指向的 `vtable`， 从而找到虚函数所在的地址，并调用该函数。

 > C++ 并没有严格规定运行时多态应该如何实现，只是编译器通常都是在同一基本模型上进行微小调整

 