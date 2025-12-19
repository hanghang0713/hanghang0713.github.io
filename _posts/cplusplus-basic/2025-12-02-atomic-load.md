---
layout: post
title: "std::atomic"
tags: C++ 
categories: [C++, Basics]
---

# std::atomic

```c++
template<class T>
struct atomic;
```

> Each instantiation and full specialization of the std::atomic template defines an atomic type. If one thread writes to an atomic object while another thread reads from it, the behavior is well-defined (see memory model for details on data races).
> 
> In addition, accesses to atomic objects may establish inter-thread synchronization and order non-atomic memory accesses as specified by std::memory_order.
>
> std::atomic is neither copyable nor movable.

`std::atomic` 是一个提供了原子操作的模板类，它对指定的类型提供原子操作。

此外，对原子类型的访问可能会建立线程间的同步，并根据 `std::memory_order` 对非原子内存访问进行排序。

**原子操作（atomic operation）**： 不可中断的一个或者一系列操作。如果该操作不能原子地执行，则要么执行完所有步骤，要么一步也不执行

**为什么需要原子操作**：

- 防止数据竞争：在多线程环境下，当多个线程同时访问同一块内存时，如果不使用原子操作，可能会出现数据不一致的问题。例如，一个简单的 i++ 操作（读取 i 的值，加一，然后写回）在两个线程同时执行时，最终结果可能只增加了1，而不是预期的2，因为它不是一个不可分割的整体。

- 保证操作的不可中断性：原子操作一旦开始，就会持续执行直到结束，中间不会被其他线程打断。这种不可分割的特性是实现线程安全的基础。

- 提供高效的并发控制：原子操作通常比使用锁（如互斥锁）更高效，因为它们通常是基于硬件指令实现的，避免了线程切换的开销。

- 实现更复杂的同步机制：原子操作是实现更高级的同步机制（如互斥锁）的基础。通过原子操作，可以构建出更复杂的同步原语，从而实现更精细的并发控制

简单来说，就是为了实现在多线程编程时，避免数据竞争，保证数据的一致性。

我们可能会想，使用锁也能实现上面的需求，但是原子操作锁的效率更高。


## Memory Model

> Defines the semantics of computer memory storage for the purpose of the C++ abstract machine.
>
> The memory available to a C++ program is one or more contiguous sequences of bytes. Each byte in memory has a unique address.

**Byte**: 字节是内存中最小的可寻址单元。它被定义为一个连续的比特序列，其大小足以容纳大量数据。

**Memory Location**: 被如下的 `object representation` 所占用的存储空间.
- `scalar type that is not a bit-field`
- `the largest contiguous sequence of bit-fields of non-zero length`
 
## std::atomic 基本用法

```c++
#include <atomic>
#include <iostream>
#include <thread>
#include <vector>

namespace {

std::atomic_int acnt;
int cnt;

void f() {
    for (auto n{10000}; n; --n) {
        ++acnt;
        ++cnt;
        // Note: for this example, relaxed memory order is sufficient,
        // e.g. acnt.fetch_add(1, std::memory_order_relaxed);
    }
}

}  // namespace

int main() {

    std::vector<std::thread> pool;
    for (int n = 0; n < 10; ++n) {
        pool.emplace_back(f);
    }

    for (auto &n : pool) {
        n.join();
    }

    std::cout << "The atomic counter is " << acnt << '/n'
              << "The non-atomic counter is " << cnt << '/n';
}
```
![atomic result](/assets/images/cplusplus-basic/atomic_result.png)

## std::memory_order 


### 什么是非原子访问内存

## std::atomic 与 锁的区别

## 参考
[What exactly is std::atomic?](https://stackoverflow.com/questions/31978324/what-exactly-is-stdatomic)