---
layout: post
title:  undefined reference to test_main(int, char**)
description: 程序员的自我修养-链接，装载与库
tags: C++ compiler symbol
category: C++ 
---

#  undefined reference to test_main(int, char**)

在编译一个项目的单元测试的过程冲碰到了如下的链接错误：

```sh
/usr/bin/ld: /home/usr/.conan2/p/b/boostf81c235181458/p/lib/libboost_test_exec_monitor.a(test_main.o): in function `boost::detail::function::void_function_obj_invoker0<test_main_caller, void>::invoke(boost::detail::function::function_buffer&)':
test_main.cpp:(.text._ZN5boost6detail8function26void_function_obj_invoker0I16test_main_callervE6invokeERNS1_15function_bufferE[_ZN5boost6detail8function26void_function_obj_invoker0I16test_main_callervE6invokeERNS1_15function_bufferE]+0x6e): undefined reference to `test_main(int, char**)'
collect2: error: ld returned 1 exit status
```

## 分析过程

单元测试项目的 `CMakeLists.txt` 内容为：
```C++
find_package(Catch2 REQUIRED)
find_package(nlohmann_json REQUIRED)
include_directories(${CMAKE_SOURCE_DIR}/src)

function(test name)
    set(target_name "test_${name}")
    add_executable(${target_name} "${target_name}.cpp")
    target_link_libraries(${target_name} PRIVATE 
                            xxxsdk 
                            nlohmann_json::nlohmann_json
                            Catch2::Catch2WithMain
                        )
    add_test(NAME ${target_name} COMMAND ${target_name}) 
endfunction()


set(tests
    event
    task
)

foreach(test_name IN LISTS tests)
   test(${test_name}) 
endforeach()
```

可以看到单元测试的项目依赖了我们自己的核心库 `xxxsdk` 以及 `nlohmann_json` 还有 `Catch2` 这三个库。

### 问题现象

从编译信息的输出中可以看到，报错的是 `libboost_test_exec_monitor.a` 这个静态库中的 `test_main.o` 这个文件中的
`test_main` 函数未定义。

通过 `nm -C` 查看 test_main.o 文件中的符号情况：

![undefined test main](/assets/images/compiler-linking/undefined_test_main.png)

其中 `U` 表示 `Undefined`（未定义，需要外部提供）。可以清晰的看到 `test_main` 函数确实是未定义的， 这里通过输出我们很容易猜到是因为 `main` 函数中调用了 `test_main` 这个函数导致的。

通过上面的输出，也可以看到 `test_main` 这个符号在 `test_main.cpp` 文件中。通过查看源码我们可以发现确实只声明了但是没有定义这个符号。

![test main](/assets/images/compiler-linking/test_main.png)


因此问题就是我们为什么会使用到 `test_main` 这个符号。


```C++
    target_link_libraries(${target_name} PRIVATE 
                            xxxsdk 
                            nlohmann_json::nlohmann_json
                            Catch2::Catch2WithMain
                        )
```


重新查看 `CMakeLists.txt`，可以看到我们依赖了 `xxxsdk`, `xxxsdk` 中依赖了 `boost`, 从而引入了 `libboost_test_exec_monitor.a` 静态库导致问题的出现。

### 解决方法

- 方案一：调整链接顺序

```C++
    target_link_libraries(${target_name} PRIVATE 
                            Catch2::Catch2WithMain
                            xxxsdk 
                            nlohmann_json::nlohmann_json
                        )
```

- 方案二：修改 `xxxsdk`, 不引入 `boost` 相关的测试库。

### 原因分析

到这里其实还有一个问题没有解释清楚，就是我们期望使用 `Catch2` 这个测试框架来完成单元测试，也定义了相关的 `main()` 函数，为什么链接时却找到了 `boost` 测试库中相关的 `main()` 函数，导致了这个链接错误。

项目最开始的写法，会先链接 `xxxsdk` ，进而链接到 `boost`, 而 `boost` 里面导出了 `libboost_test_exec_monitor.a` 中的 `main` 这个符号，而即使后面我们链接 `Catch2` 时，即使 `Catch2` 中也定义了 `main` 函数，但是因为前面已经扫描到 `main` 这个符号了，所以 `Catch2` 中的 `main` 会被忽略掉。


> 参考阅读： 强符号与弱符号 --《程序员的自我修养——链接、装载和库》
