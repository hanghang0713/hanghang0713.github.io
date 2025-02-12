---
layout: post
title:  "what is a deb package"
tags: linux deb
category: linux 
---

# what is a deb package {ignore=true}
deb 是一种文件格式，同时也是 Debian 发行版本以及它的衍生版本的软件包格式的文件名扩展。

Debian package 是标准的 `Unix ar archives`，包含两个 `tar archives`:
- 控制信息
- 可安装的数据

DPKG 提供了基础的安装和管理 Debian package 的功能。通常，最终用户不会直接使用 DPKG 管理软件，而是使用 APT 或其他的 APT 前端，比如 aptitude 和 Synaptic

Debian package 可以转换成其他格式的软件包，也可从其他格式的软件包转换回来， 可以使用 `alien` 工具完成这些操作。
也可以使用 `check install` 或者 Debian Package Maker 从源码创建一个 Debian package. 

# Debian control file
2

## 引用
- https://www.debian.org/doc/manuals/debian-faq/pkg-basics.en.html