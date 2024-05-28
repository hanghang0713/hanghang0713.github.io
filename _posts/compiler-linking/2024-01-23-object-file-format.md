---
layout: post
title: 目标文件里面有什么
description: 程序员的自我修养-链接，装载与库
tags: C++ compiler windows
category: C++ 
---

## 目标文件的格式
编译器编译后产生的文件就叫做**目标文件**，目标文件从结构上将，就是已经编译后的可执行文件的格式。

### 可执行文件格式
现在流行的可执行文件格式包括 `PE-COFF(windows)` 与 `ELF(linux)`，它们都是 `COFF` 格式的变种. 目标文件就是编译后但是未链接的中间文件(windows 下的 .obj 与 Linux 下的 .o)。

不止是目标文件，动态链接库(DLL, dynamic Linking Library, windows 下的 dll 和 linux 下的 .so)与静态库(Static Linking Library, windows 下的 .lib 与 linux 下的 .a)都按照可执行文件的格式进行存储。

![object file format](/assets/images/compile-basic/object_file.png "目标文件格式")

### 目标文件结构
目标文件的内容至少包含有被编译后的机器指令代码，数据，符号表，调试信息，字符串等。一般情况下，目标文件会按照不同的属性，以 `Section` 的形式存储。
机器指令经常被放在 `代码段`，全局变量和局部静态变量通常放在 `数据段`。
总体来说，程序源代码被编译后主要分为两种段：程序指令和程序数据。代码段属于程序指令。数据段和 `.bss` 属于程序数据。

> 指令和数据分开存放的好处是什么
> 程序和指令被映射到两个不同的需存区域，可以分别设置权限，避免只读的指令被无意间修改
> CPU 缓存一般是指令缓存和数据缓存分开，分开存储可以提高缓存的命中率
> 当运行多个程序的副本时，只读资源可以共享。可以节省大量空间


### 目标文件的内容 
![object file format section](/assets/images/compile-basic/obj_file_section.png "一个简单的目标文件的段信息")

从上面的信息可以看到，常见的段信息有:
- .text 代码段
- .data 数据段
- .bss 存放未初始化的全局变量和局部静态变量，不占据实际的空间
- .rodata 只读数据段
- comment 注释信息段
- .note.gnu.property 堆栈提示段

![onject file structure](/assets/images/compile-basic/section-structure.png "段的结构示意图")

#### 代码段

> objdump 常用参数
> -s 将所有段的内容通过十六进制的方式打印出来
> -d 将包含指令的段进行反汇编

![text section](/assets/images/compile-basic/text-section.png "代码段")

可以看到 .text 段里面包含的正是 SimpleSection.c 里面两个函数的指令。

#### 数据段和只读数据段
![data section](/assets/images/compile-basic/data-section.png "数据段")

.data 段存放的是 **已经初始化的全局静态变量和局部静态变量**。
.data 段大小是 8，正好是源码中的 `global_init_var` `static_var` 中的两个变量的大小。每个变量4个字节。
.data 段中 0x54 0x00 0x00 0x00 对应的正是 `global_init_var` 的值 84
.data 段中 0x40 0x00 0x00 0x00 对应的正是 `static_var` 的值 64

.rodata 段中存放的是只读数据，一般是只读变量和字符串常量。

> 单独设立 .rodata 段有很多好处
> 从语义上支持 const 语法
> 操作系统可以将 .rodata 映射成只读, 保证程序的安全性
> 嵌入式系统，可以将 .rodata 放入 ROM 只读存储器中，保证访问储存器的正确性

.rodata 段的大小是 4， 存放的是源码中的 "%d\n" 的字符串常量
.rodata 中的 `25640a00` 对应的正是这个字符串的 ASCII 字节序，最后以 `\0` 结尾

#### BSS 段
![bss section](/assets/images/compile-basic/bss-section.png "BSS 段")

.bss 段存放的是 **未初始化的全局变量和局部静态变量** 
更准确的说法是 .bss  段为他们预留了空间
有些编译器会将未初始化的全局变量放在 .bss 段，有些则只是预留空间
编译单元内部可见的静态变量一定会放在 .bss 段

```C++
static int x1 = 0
static int x2 = 1
```

上面的代码中，x1 会被放在 .bss 段，x2 会被放在 .data 段。因为 x1 = 0, 可以认为是未初始化的，会被优化掉以节约空间。因为 .bss 不占磁盘空间 (因为 .bss 段的大小在段表中记载了。不像其他段，还有 Contents)

#### 其他段
除了 .text .data .bss 三个最常用的段以外，ELF 文件也有可能包含其他的段。

![other section](/assets/images/compile-basic/other-section.png "其他 段")

以 `.` 作为前缀的，表示这些表的名字是系统保留的。应用程序可以用一些非系统保留的名字作为段名。
比如我们可以在 ELF 文件中插入一条 “music” 的段，脸面存放一首 MP3 音乐

``` shell
# 可以使用 objcopy 工具将一个二进制文件作为目标文件中的一个段

objcopy -I binary -O elf32-i386 -B i386 image.jpg image.o
```

![customize section](/assets/images/compile-basic/customize-section.png "自定义段")

有些时候，我们可能需要将某些变量或者代码放到指定的段中，以实现某些特殊的功能。

```C
__attribute__((section("FOO"))) int global = 42
__attribute__((section("BAR"))) void fun() {}
```

### ELF 文件结构描述

#### ELF 文件头
ELF 目标文件格式的最前部是 ELF 文件头，它包含了描述整个文件的基本属性：ELF 文件版本，目标机器型号，程序入口地址等。

> `readelf` 命令可以详细查看 ELF 文件 

![elf header](/assets/images/compile-basic/elf-header.png "ELF 文件头")
从上面的图中可以看到，ELF 文件头中定义了：
- ELF 魔数
- 文件机器字节长度
- 数据存储方式
- 版本
- 运行平台
- ABI 版本
- ELF 重定位类型
- 硬件平台
- 硬件平台版本
- 入口地址
- 程序头入口和长度
- 段表的位置和长度、数量

ELF 文件头结构及相关常数被定义在 `/usr/include/elf.h` 里面。ELF 文件有 32 位 和 64 位两个版本，对应的 ELF 文件头结构也有两个版本，分别是 `ELF32_Ehdr` 和 `ELF64_Ehdr`。
![elf header define](/assets/images/compile-basic/elf-header-define.png "ELF 文件头定义")

除了 `e_ident` 中定义了 `Class` `Data` `Version` `OS/ABI`  `ABI Version` 这5个参数外，其他的参数和 `readelf` 指令输出的信息都是一一对应的

### 段表
段表中保存了段的基本属性，描述了 ELF 的各个段的信息，编译器，链接器和装载器都是依靠段表来定位和访问各个段的属性的。

> 相较于 `objdump -h`, 使用 `readelf -S` 可以查看更加详细的段表结构

![detailed section header table](/assets/images/compile-basic/detailed-section-header-table.png "段表的详细信息")

段表的结构比较简单，它是一个以 `Elf_Shdr` 结构体为元素的一个数组，数组元素的个数等于段的个数。`Elf_Shdr` 又被称为段描述符。ELF 段表的第一个元素是无效的段描述符，用 `NULL` 表示

> 数组的存放方式
ELF 文件里面很多地方都使用了与段表类似的数组，一般定义一个固定的长度，然后依次存放，就可以使用下标来索引。

![elf section define](/assets/images/compile-basic/elf-section-def-header.png "ELF 段描述符")

**段的类型** 段的名字只在编译和链接的时候有意义。主要决定段的属性的是段的类型 `sh_type`. 和段的标志位 `sh_flags`
![elf section type](/assets/images/compile-basic/section-type.png "ELF 段的类型定义")

**段的标志位** 表示该段在进程虚拟地址空间中的属性。
![elf section flag](/assets/images/compile-basic/section-flag.png "ELF 段的标志位")

**段的链接信息** 如果段的类型是与链接相关的，比如重定位表，符号表等，那么 `sh_link` 和 `sh_info` 这两个成员包含的意义如下：
![elf section link info](/assets/images/compile-basic/section-link-info.png "ELF 段的链接信息")

### 重定位表
链接器在处理目标文件时，需要对目标文件中的某些部位进行重定位，这些重定位的信息都记录在 `ELF` 文件的重定位表里面，对于每个需要重定位的代码段或者数据段，都会有一个对应的重定位表，比如 `.text` 段的重定位表就是 `.rel.text`.

### 字符串表
将字符串集中起来存放到一个表中，然后使用字符串在表中的偏移来引用字符串。一般字符串表在 `ELF` 文件中也以段的形式保存。
- 字符串表 `.strtab` 保存普通的字符串
- 段表字符串表 `.shstrtab` 保存段表中用到的字符串， 比如段名

`e_shstrndx` 表示段表字符串表在段表中的下标，因此只要分析 `ELF` 文件头，就可以得到段表和段表字符串表的位置，从而解析整个 `ELF` 文件。

## 链接的接口——符号
链接过程的本质就是将不同的目标文件拼接在一起，可以形成一个整体。为了能够顺利拼接，因此需要有固定的规则才可以。
我们将函数和变量统称为符号，函数名或变量名称为符号名。

链接过程很关键的一点是对符号的管理，每一个文件都有一个相应的符号表 `Symbol Table` 每个定义的符号有一个值，叫做符号值。

- 定义在本目标文件中的全局符号， 可以被其他目标文件引用。
- 在本目标文件中引用的符号，定义在其他目标文件。一般叫做 **外部符号**
- 段名，由编译器产生的符号
- 局部符号，编译单元内部可见。对于链接过程没有作用
- 行号信息

### ELF 符号表结构
ELF 文件中的符号表往往是一个段 `.symtab`，
![elf symbol table](/assets/images/compile-basic/elf-symbol-struct.png "ELF 符号表结构")


**符号类型和绑定信息**
![elf type and bind](/assets/images/compile-basic/symbol-type-and-bind.png "ELF 符号类型和绑定信息")

**符号所在段**
如果符号定义在本目标文件中，那么这个成员表示段在段表中的下标.
如果不是定义在本目标文件中.或者对于某些特殊符号.定义如下:
![symbol special index](/assets/images/compile-basic/symbol-index-special.png "特殊的段下标")

**符号值**
如果一个符号是一个函数或者变量的定义,则这个符号的值 `st_value` 就是这个函数和变量的地址.
但是下面几种情况,需要特殊对待:
- 目标文件中,是符号的定义,但是下标不为 `SHN_COMMON` 类型的. 那么符号值表示符号在段中的偏移
- `SHN_COMMON` 类型, 符号值表示对齐属性.
- 可执行文件中, 符号值表示符号的虚拟地址,动态链接器会用到这个地址


![symbol summarize](/assets/images/compile-basic/symbol-summarize.png "符号表")

- fun1 和 main 函数都定义在一个 .c 文件中,所在位置都为代码段, 因此 NDX 为 1. 它们是函数,所以类型和绑定属性分别为 STT_FUNC 和 STB_GLOBAL
- printf 在 SimpleSection.c 文件中被引用,但是没有定义, NDX 是 SHN_UNDEF
- global_init_var 已初始化的全局变量,定义在 .bss 段中
- global_uninit_var 未初始化的全局变量, 它是一个 SHN_COMMON 类型.
- static_var.x 静态变量,编译单元内部可见.
- STT_SECTION 类型的符号,表示下标为 Ndx 所在段的段名, 比如 2 号符号表示 .text


