---
title: linxu-kernel-interrupt
abbrlink: 18712
date: 2025-04-15 19:07:59
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

# 中断

{% note info modern %}
CPU在正常运行期间，由外部或者内部引起的事件，让CPU停下当前正在运行的程序，转而去执行触发他的中断所对应的程序，处理完中断对应的程序以后在回来继续执行。这个就是中断。举例:同学A现在正在厨房做饭，突然电话响了，然后A关火去接电话。接完电话在回去开火继续做饭。这个过程就是一个中断的一个过程。
{% endnote %}

## 中断类型
- 同步中断由CPU本身产生，又称为内部中断。这里同步是指中断请求信号与代码指令之间的同步执行，在一条指令执行完毕后，CPU才能进行中断，不能在执行期间。所以也称为异常（exception）。

- 异步中断是由外部硬件设备产生，又称为外部中断，与同步中断相反，异步中断可在任何时间产生，包括指令执行期间，所以也被称为中断（interrupt）。

- 异常又可分为可屏蔽中断（Maskable interrupt）和非屏蔽中断（Nomaskable interrupt）。而中断可分为故障（fault）、陷阱（trap）、终止（abort）三类。
 
# 中断子系统框架
- CPU
- 中断控制器
- 外设
- 中断向量表
- 中断号
- Linux内核中断子系统
- 中断编程接口
- 具体的外设驱动

## 没那么简单
学操作系统或者单片机时候都知道中断是重点，而且也能自己分析和配置中断程序并执行，但是在Linux内核中中断的实现复杂的多
中断大致可以分为以下几个步骤：

- 当前正在执行的程序
- 保存被打断的上下文
- 中断向量表
- 找到发生中断的设备
- 中断服务程序
- 退出中断,调度程序运行
    - 恢复被打断的上下文
    - 回到被打断的程序,继续执行...
    - 恢复高优先级的进程的上下文
    - 切换到高优先级的程序执行...

在C语言的函数调用中，也常常需要保存上下文信息到栈帧当中，可能是某些寄存器或者状态寄存器的信息，函数调用完成之后需要返回

![函数调用](https://image.aruoshui.fun/i/2025/04/15/w9sief-0.webp)
但在中断中，每次中断位置不固定，编译过程中不会像函数调用一样保存上下文，中断过程中需要自己存

# 中断控制器
- 负责处理各种中断
- 优先级、屏蔽、使能

## • SGI：16 Software Generated Interrupts
1. 中断号ID0~ID15，用于多核之间通讯
• PPI：16 external Private Peripheral Interrupts
2. 每个core私有的中断，如本地时钟，ID16~ID31
• SPI：Shared Peripheral Interrupt
3. 所有core共享的中断，可以在多个core上运行
– 支持范围可配置：32~1019，步进32，从ID32开始

![中断](https://image.aruoshui.fun/i/2025/07/09/yt9q00-0.webp)

# 中断号
1. HW interrupt ID
2. IRQ number
3. IRQ_domain

HW Interrupt ID是硬件层面的中断标识符，IRQ Number是操作系统用来管理和响应中断的内部编号，而IRQ Domain则是一种机制，用于处理不同来源的中断如何被映射到IRQ编号的问题。这三者共同作用，确保操作系统能够有效地管理和响应来自各种硬件设备的中断请求。

# GIC 处理中断流程

![中断流程](https://image.aruoshui.fun/i/2025/07/10/xrfb9j-0.webp)

1. GIC检测到使能的中断发生，将中断状态设为pending
2. GIC的仲裁器将最高优先级的pending中断发送到指
定的CPU interface
3. CPU interface根据配置，将中断信号发送到CPU
4. CPU应答该中断，读取寄存器获取interrupt ID，GIC
更新中断状态为active
»Pending --> active
»Pending --> active and pending 中断重新产生
»Active  active and pending 若中断状态为active
5. CPU处理完中断后，发送EOI（End Of Interrupt）信号给GIC，通知中断控制器该中断已处理完成，GIC将中断状态从active清除，允许后续中断继续响应。

**仲裁器的作用**
当有多个设备或进程同时请求同一个资源（比如总线、内存、I/O端口）时，仲裁器决定谁可以优先使用该资源。仲裁器可以实现公平访问（如轮询法），也可以实现优先级访问（如高优先级设备先获得资源）。
