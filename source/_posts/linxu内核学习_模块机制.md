---
title: linxu内核学习笔记
abbrlink: 6793
date: 2025-02-25 19:52:45
tags:
description:
categories:
cover:
swiper_index:
---

# 参考文章
{% link 知乎, https://zhuanlan.zhihu.com/p/666861211,  https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp%} 

{% link CSDN, https://zhuanlan.zhihu.com/p/666861211,  https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp%} 

{% link github, https://zhuanlan.zhihu.com/p/666861211,  https://image.aruoshui.fun/i/2025/01/13/p2416y-0.webp%} 

# 软件分层思想
1. BSP：嵌入式底层系统开发，调试程序，让特定的系统跑在特定的板子上，具体的工作包括调Bootloader程序，加载操作系统内核，文件系统的加载，外设驱动程序的开发。
2. 驱动开发：驱动与底层硬件直接打交道，充当了硬件与应用软件中间的桥梁。
3. 应用层：网络服务层、文件系统、虚拟化
4. 基础服务：进程、内存管理、VFS、中断、任务调度

# 模块化设计
1. 更细粒度的模块划分、模块依赖
2. 功能、模块、子系统、框架
3. Linux内核中的OOP思想
   1. 结构体
   2. 函数指针


# 宏内核与微内核
1. 宏内核：宏内核将操作系统的所有核心功能集中到一个大的内核空间中，所有服务（如设备驱动、文件系统、内存管理等）都运行在内核态，紧密集成在一起。这种设计简化了模块间的交互，提高了性能，但增加了复杂性和维护难度。‌
2. 微内核：微内核只保留最基本的系统功能，如进程管理、内存管理和消息传递等，其他高级功能如设备驱动、文件系统和网络协议等在用户空间以服务的形式运行。这种设计提高了系统的模块化和可扩展性，但可能会降低性能。‌

#### 内核的加载
{% link 向Linux内核添加新功能的静态加载与动态加载, https://blog.csdn.net/qq_45143522/article/details/138073674,  https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp%} 
1. 静态加载：将新功能的源码与内核其它代码一起编译进uImage内核镜像文件内。
2. 动态加载：新功能源码与内核其它源码不一起编译，而是独立编译成内核的插件(被称为内核模块）文件.ko

# 内核模块
