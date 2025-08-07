---
layout: 嵌入式linux
title: Buildroot
date: 2025-02-06 19:31:46
tags:
categories: 嵌入式Linux
---

# 参考文章
{% link Linux发行版列表, https://zh.wikipedia.org/wiki/Linux%E5%8F%91%E8%A1%8C%E7%89%88%E5%88%97%E8%A1%A8, https://image.aruoshui.fun/i/2025/02/12/s7cjv0-0.webp %} 



# Linux的启动过程
![过程](https://image.aruoshui.fun/i/2025/02/12/rb5kqt-0.webp)

# 发行版Linux操作系统
Linux发行版(也叫做GNUlinux发行版)，为一般用户预先集成好的Linux操作系统及各种应用软件。一般用户不需要重新编泽，在直接安装之后，只需要小幅度更改设置就可以使用，通常以软件包管理系统来进行应用软件的营理，Linux发行板通常包含了包括桌面环境办公包、媒体播放器、数据车等应用软件。这些保作系统通总由Linux内核、以及来自GNU计划的大量的函数库，和基于x Window或者Wayland的图形界面。

## Debian系列
Debian及其派生发行版使用deb软件包格式，并使用dpkg及其前端作为软件包管理器。
旗下最著名的就是Ubuntux系列和国内优麒麟和Deepin

## Red Hat系
Red Hat Linux和SUSE Linux是最早使用RPM格式软件包的发行版，如今RPM格式已广泛运用于众多的发行版。
旗下最著名的是CentOS

## GNU和Linux
GNU与Linux是密不可分的
1. GNU构建系统 - 包含autoconf和automake
2. GNU make   - GNU make 程序
3. GNU编译器套装(GNU Compiler Collection)
4. GNU Debugger-高级调试器(gdb)
5. GNUC函数库(glibc)-符合POSIX的C语言库
6. GNU pth-POSIX兼容操作系统的软件线程。
7. GNU m4-宏处理器
8. GTK+-GIMP工具包，包含GTK、+GDK和一套GLib库(由GIMP和GNOME使用)
9. GNOME-GNU网络对象模型环境(GNUNetwork Object Model Environment)GNU的官方桌面。
10. Bash-GNU的UNIX兼容shell
11. Coreutils- 基本命令、

# 编译构建系统
## 交叉编译
之所以需要交叉编译，就是由于目标平台ARM板资源受限
![](https://image.aruoshui.fun/i/2025/02/12/z106ea-0.webp)

## 交叉编译工具链
![工具链](https://image.aruoshui.fun/i/2025/02/13/ny3psg-0.webp)
- Build(构建机器)，使用GCC的源码，制作交叉编译工具链。 
- Host(主机)，使用交叉编译工具链，编译出程序。 
- Target(目标机器)，程序执行的地方

## 工具链元祖
autoconf 定义了system definitions 的概念，表示为tuples(元组)
系统定义描述了一个系统:CPU架构、操作系统、芯片厂商、ABI、C库
定义方式:
- `<arch>-<vendor>-<os>-<libc/abi>(完整名称)`
- `<arch>-<os>-<libc/abi>`

![工具链组成](https://image.aruoshui.fun/i/2025/02/13/o12v2i-0.webp)

![裸机和Linux工具链](https://image.aruoshui.fun/i/2025/02/13/p3zc07-0.webp)

## ABI
调用约定和反汇编就是ABI

详细文章可以看
{% link 什么是应用程序二进制接口ABI, https://zhuanlan.zhihu.com/p/386106883, https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp %} 

## 工具链和SDK的区别
![区别](https://image.aruoshui.fun/i/2025/02/13/p8085j-0.webp)

## 嵌入式Linux系统的编译构建
![过程](https://image.aruoshui.fun/i/2025/02/12/z10alx-0.webp)

## 交叉编译工具链
![说明](https://image.aruoshui.fun/i/2025/02/13/pan0pk-0.webp)


## 自动结构构建
前面展示了编译过程，在之前都是一部分手工编译，现在有了很多自动化编译工具，就比如Buildroot、OpenWrt、Yocto

![alt text](https://image.aruoshui.fun/i/2025/02/12/z10ie9-0.webp)


# Buildroot
buildroot是Linux平台上一个构建嵌入式Linux系统的框架。整个Buildroot是由Makefile脚本和Kconfig配置文件构成的。你可以和编译Linux内核一样，通过buildroot配置，menuconfig修改，编译出一个完整的可以直接烧写到机器上运行的Linux系统软件(包含boot、kernel、rootfs以及rootfs中的各种库和应用程序)。
**项目主要特点**
1. 制作启动映像
2. 不支持rpm deb等风格的包管理
3. 系统固件生成器从源代码构建所有组件
4. 注重简单
   
**支持**
1. 根文件系统映像
2. 内核、引导加载程序、工具链

## buildroot的使用
直接看buildroot的官方文档，介绍的相当详细，参考以下文章即可
{% link buildroot使用介绍, https://blog.csdn.net/maizaozao/article/details/139241440, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp %} 

## 具体项目实践
