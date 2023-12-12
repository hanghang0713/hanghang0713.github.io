---
layout: post
title:  "关于线程池，我们在谈论什么"
tags: C++ Concurrency
category: Concurrency 
---

# 关于线程池，我们在谈论什么 {ignore=true}

[toc]

## 背景
```C++
template <typename T, typename... Args>
auto EnQueue(T&& t, Args&&... args) -> std::future<typename std::invoke_result<T&&, Args&&...>::type> {
    using return_type = typename std::invoke_result<T, Args...>::type;

    auto task = std::make_shared< std::packaged_task<return_type()> >(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
 
    // ...
}
```
在实现一个 C++ 的线程池时，基本都会实现上面这个函数。它会将一个任务加入到一个任务队列中 `std::queue<std::function<void()>>`，并且返回一个 `std::future` 对象，任务完成时，可以通过 `std::future` 拿到最终的结果。

## std::result_of, std::invoke_result
在返回 `std::future` 对象时，我们不知道它的类型，但是我们又需要返回它的类型，因此需要通过某种方法来推测它的类型。


> Deduces the return type of an INVOKE expression at compile time.

通过 `std::result_of` 或者 `std::invoke_result` 就可以实现上面的功能。 

### 定义
```C++
template< class F, class... ArgTypes >
class result_of<F(ArgTypes...)>;

template< class F, class... ArgTypes >
class invoke_result;
```

- F cannot be a function type or an array type (but can be a reference to them);
- if any of the Args has type "array of T" or a function type T, it is automatically adjusted to T*;
- neither F nor any of Args... can be an abstract class type;
- if any of Args... has a top-level cv-qualifier, it is discarded;
- none of Args... may be of type void.

### 使用示例
```C++
struct S
{
    double operator()(char, int&);
    float operator()(int) { return 1.0; }
};

double Func(char, int&);

int main() {
    std::result_of<S(char, int&)>::type d = 3.14; 
    static_assert(std::is_same<decltype(d), double>::value, "");

    std::invoke_result<S,char,int&>::type b = 3.14;
    static_assert(std::is_same<decltype(b), double>::value, "");

    
    std::invoke_result<decltype(Func)(C, char, int&)>::type g = 3.14;
    static_assert(std::is_same<decltype(g), double>::value, "");
}
```

对于函数 `Func` 不能作为参数传递给 `std::invoke_result`,  第一个参数必须是一个类型，因此需要使用 `decltype`.


## decltype 
> Inspects the declared type of an entity or the type and value category of an expression.

`decltype` 用于检查表达式的类型，在标准的符号声明类型难以声明或者无法声明时，`decltype` 非常有用。

### 使用示例
```C++
struct A { double x; };
const A* a;
 
decltype(a->x) y;       // type of y is double (declared type)
decltype((a->x)) z = y; // type of z is const double& (lvalue expression)

const int& getRef(const int* p) { return *p; }
static_assert(std::is_same_v<decltype(getRef), const int&(const int*)>);

auto getRefFwdBad(const int* p) { return getRef(p); }
static_assert(std::is_same_v<decltype(getRefFwdBad), int(const int*)>,
    "Just returning auto isn't perfect forwarding.");

decltype(auto) getRefFwdGood(const int* p) { return getRef(p); }
static_assert(std::is_same_v<decltype(getRefFwdGood), const int&(const int*)>,
    "Returning decltype(auto) perfectly forwards the return type.");

auto getRefFwdGood1(const int* p) -> decltype(getRef(p)) { return getRef(p); }
static_assert(std::is_same_v<decltype(getRefFwdGood1), const int&(const int*)>,
    "Returning decltype(return expression) also perfectly forwards the return type.");

auto f = [i](int a, int b) -> int { return a * b + i; };
auto h = [i](int a, int b) -> int { return a * b + i; };
static_assert(!std::is_same_v<decltype(f), decltype(h)>,
    "The type of a lambda function is unique and unnamed");
```

## std::bind
```C++
template< class F, class... Args >
/* unspecified */ bind( F&& f, Args&&... args );
```

`std::bind` 是一个函数模版，会生成一个转发函数调用的包裹器。调用这个包裹器就相当于调用被包裹的函数，并将部分参数绑定到 `args`.
- 对于非静态的成员函数或者指向非静态数据成员的指针，第一个参数必须是指向其成员将被访问的对象的引用或指针。
- 绑定的参数会被拷贝或者移动，如果需要传递引用，需要使用 `std::ref` 或者 `std::cref`。
- 重复的占位符是允许的

### 使用举例
```C++
void f(int n1, int n2, int n3, const int& n4, int n5)
{
    std::cout << n1 << ' ' << n2 << ' ' << n3 << ' ' << n4 << ' ' << n5 << '\n';
}
 
int g(int n1)
{
    return n1;
}
 
struct Foo
{
    void print_sum(int n1, int n2)
    {
        std::cout << n1 + n2 << '\n';
    }
 
    int data = 10;
};

int main() {
    using namespace std::placeholders;

    auto f1 = std::bind(f, _2, 42, _1, std::cref(n), n);

    auto f2 = std::bind(f, _3, std::bind(g, _3), _3, 4, 5);

    Foo foo;
    auto f3 = std::bind(&Foo::print_sum, &foo, 95, _1);
    f3(5);

    auto f5 = std::bind(&Foo::data, _1);
    std::cout << f5(foo) << '\n';
    std::cout << f5(std::make_shared<Foo>(foo)); << '\n';
}
```

## std::forward
`std::forward` 利用模版参数推导和引用折叠的技巧，可以非常完美的保持参数的左值性或者右值性，从而实现完美转发。


### 使用举例
```C++
template<class T>
void wrapper(T&& arg)
{
    // arg is always lvalue
    foo(std::forward<T>(arg)); // Forward as lvalue or as rvalue, depending on T
}
```
使用 `&&` 作为参数类型时，在函数内部， `arg` 永远作为左值传递（对于一个函数调用,实参可以是左值或右值,但形参在函数内部总是一个左值），而 `T` 会被推断成不同的类型，`std::forward<T>` 会根据 `T` 的类型实现完美将 `arg` 转成对应的左值和右值，从而实现完美转发。

## std::packaged_task
`std::packaged_task` 一般与 `std::future`,`std::async` 一起使用，它会包装一个可调用的对象以便异步使用，它会将结果或者异常存储在一个共享状态中，通过 `std::future` 对象访问。

### 使用举例
```C++
int f(int x, int y) { return std::pow(x,y); }
 
void task_lambda()
{
    std::packaged_task<int(int,int)> task([](int a, int b) {
        return std::pow(a, b); 
    });
    std::future<int> result = task.get_future();
 
    task(2, 9);
 
    std::cout << "task_lambda:\t" << result.get() << '\n';
}
 
void task_bind()
{
    std::packaged_task<int()> task(std::bind(f, 2, 11));
    std::future<int> result = task.get_future();
 
    task();
 
    std::cout << "task_bind:\t" << result.get() << '\n';
}
 
void task_thread()
{
    std::packaged_task<int(int,int)> task(f);
    std::future<int> result = task.get_future();
 
    std::thread task_td(std::move(task), 2, 10);
    task_td.join();
 
    std::cout << "task_thread:\t" << result.get() << '\n';
}
 
int main()
{
    task_lambda();
    task_bind();
    task_thread();
}
```

`std::packaged_task` 只能移动，因此在最初的代码里面，使用了 `std::make_shared`, 而不能直接进行拷贝。