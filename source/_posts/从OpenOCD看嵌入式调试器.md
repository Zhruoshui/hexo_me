---
title: 从OpenOCD看嵌入式调试器
description: 嵌入式调试器硬件、软件架构一览，以OpenOCD出发
categories:
  - 嵌入式开发
abbrlink: 43170
date: 2025-10-22 10:40:57
tags: 
  - 嵌入式调试
  - openocd
  - probe-rs
cover: https://image.aruoshui.fun/i/2025/12/22/f93l60-0.webp
swiper_index:
---

# 参考文章

{% link blog, https://hackaday.com/2012/09/27/beginners-look-at-on-chip-debugging/,  https://image.aruoshui.fun/i/2025/12/22/f201jo-0.webp%} 

{% link OpenOCD User’s Guide, https://openocd.org/doc/html/index.html,  https://image.aruoshui.fun/i/2025/12/22/f201jo-0.webp%}

{% link OpenOCD User’s Guide, Telnet, https://image.aruoshui.fun/i/2025/12/22/hc73tt-0.webp%}

https://learn.sparkfun.com/tutorials/arm-programming/jtag-and-swd
https://embeddedinventor.com/swd-vs-jtag-differences-explained/
https://www.corelis.com/education/tutorials/jtag-tutorial/what-is-jtag/
https://www.xjtag.com/about-jtag/what-is-jtag/
https://research.kudelskisecurity.com/2019/05/16/swd-arms-alternative-to-jtag/
https://vlsitutorials.com/jtag-test-access-port-and-tap-controller/
https://www.allaboutcircuits.com/technical-articles/introduction-to-jtag-test-access-port-tap/


# 调试器(debug-probe)
在嵌入式系统工程中，调试器并非简单的外设工具，而是连接开发者逻辑思维与物理芯片底层运行状态的神经中枢。随着微处理器架构从早期的单核八位演进到复杂的异构多核架构，调试器的作用已从基础的程序下载扩展到了实时的系统洞察与性能剖析。

## OpenOCD（Open On-chip Debugger）
OpenOCD是一款开源的跨平台片上调试器，提供针对嵌入式设备的调试、系统编程和边界扫描功能。其工作方式就是代替了原有那些调试适配器提供的相关工具和驱动， 直接通过普通的 USB 驱动访问适配器，进而访问目标硬件。
**说白了就是：一个用于嵌入式开发的调试中间层工具，能让你通过 JTAG/SWD 接口 与芯片内部进行交互**
> OpenOCD 是由 Dominic Rath 作为他 2005 年在 Augsburg 应用科学大学毕业论文的一部分而创建,论文链接:http://openocd.org/files/thesis.pdf

## 纵向：打通“软件命令”到“物理电平”的链路
OpenOCD通过JTAG、SWD等硬件接口与目标设备连接，使开发者能够进行源代码级调试、内存和寄存器的查看和修改，以及运行时性能分析等操作。
![OpenOCD 调试链路的逻辑关系图](https://image.aruoshui.fun/i/2025/12/22/fp29yl-0.webp)
逻辑关系分为三个核心层次：**用户交互层**、**中转逻辑层**、以及**物理硬件层**。 

**这里有一个STLink更具体的实例：**
![OpenOCD调试器](https://image.aruoshui.fun/i/2025/12/22/gvgs1w-0.webp)

### **用户交互层：指令的起点**
在这一层，并不直接操作硬件，而是操作“调试客户端”。
   * **GDB (GNU Debugger):** 这是最常用的客户端。当入 `step`（单步执行）或 `break`（设置断点）时，GDB 会将这些指令打包成 **GDB Remote Serial Protocol (RSP)** 协议。具体的GDB调试命令请参考我的其他文章： [GDB调试工具](https://blog.aruoshui.fun/posts/14250.html)
     * **GDB 之所以能知道代码运行到了哪一行、某个变量的值是多少，全靠它读取了 .elf 文件**，它像一份“地图”，告诉 GDB 代码行号对应的内存地址、变量名在哪里。

        | 组成部分 | 作用 |
        | --- | --- |
        | **ELF Header** | 文件的“身份证”，规定了是 32 位还是 64 位、大端还是小端、目标 CPU 架构（如 ARM, RISC-V）等。 |
        | **Program Header Table** | 告诉操作系统（或烧录工具）哪些部分需要**加载到内存**里跑。 |
        | **Section Header Table** | 描述文件内容的逻辑分类（如代码段、数据段、调试段）。 |
        | **Data (Sections)** | 真正的内容：包括：1. `.text`: 编译后的机器指令。2. `.data`: 已初始化的全局变量。3. `.debug`: 调试专用的元数据。 |
   * **Telnet / TCL:** OpenOCD 自身也开了一个“后门”。可以通过 Telnet 连接到 OpenOCD，直接输入底层指令（如 `reset halt`）。更多的相关指令请参照：[15 General Commands](https://openocd.org/doc/html/General-Commands.html)
        ```shell
        Usage: telnet [OPTION...] [HOST [PORT]]
        Login to remote system HOST (optionally, on service port PORT)

        General options:

        -4, --ipv4                 use only IPv4
        -6, --ipv6                 use only IPv6
        -8, --binary               use an 8-bit data transmission
        -a, --login                attempt automatic login
        -b, --bind=ADDRESS         bind to specific local ADDRESS
        -c, --no-rc                do not read the user's .telnetrc file
        -d, --debug                turn on debugging
        -e, --escape=CHAR          use CHAR as an escape character
        -E, --no-escape            use no escape character
        -K, --no-login             do not automatically login to the remote system
        -l, --user=USER            attempt automatic login as USER
        -L, --binary-output        use an 8-bit data transmission for output only
        -n, --trace=FILE           record trace information into FILE
        -r, --rlogin               use a user-interface similar to rlogin

        -?, --help                 give this help list
            --usage                give a short usage message
        -V, --version              print program version

        Mandatory or optional arguments to long options are also mandatory or optional
        for any corresponding short options.
        Report bugs to <bug-inetutils@gnu.org>.
        ```
---

### **中转逻辑层：OpenOCD 的核心**
这是 OpenOCD 进程内部最复杂的环节
#### 1. Server 模块
OpenOCD 启动后会监听多个端口：
* **3333 端口：** 专门给 GDB 用的。
* **4444 端口：** 给 Telnet 命令行用的。

它负责把收到的网络封包拆解，交给内部逻辑处理。具体需要的配置参看[服务器配置](https://openocd.org/doc-release/html/Server-Configuration.html)

#### 2. 脚本引擎 (Tcl Interpreter)
OpenOCD 极其灵活，因为它内置了一个 Tcl 解释器。 `.cfg` 配置文件（比如 `target/stm32f1x.cfg`）本质上是一段代码，告诉 OpenOCD：
* 这个芯片的 IDCODE 是什么？
* 它的 Flash 起始地址在哪里？
* 它支持多少个硬件断点？

一个典型的配置文件由三部分组成：
1. interface(接口)——每个不同的调试器对应一个接口
2. board(板子)——每个不同的板子都对应着一个配置文件
3. target(目标芯片)——目标芯片是指那些集成了中央处理器（CPU）以及其他JTAG测试访问端口（JTAG TAPs）的芯片。

>看起来是不是挺麻烦的？

实际上几乎所有设置信息都来自其他人提供的配置文件，**也就是说你直接用别人写好的配置即可**。OpenOCD会提供一个名为`scripts`的目录（在Linux系统中通常位于`/usr/share/openocd/scripts`下），其中包含了各种可执行的脚本。主板和开发工具的厂商也可能提供此类配置文件，个人用户网站也同样可能提供。

> https://openocd.org/doc-release/html/OpenOCD-Project-Setup.html 中提到：
> **最佳情况是：只需包含两个文件，它们就能处理所有其他相关事务。** 第一个文件是接口配置文件，用于设置设备的接口参数。 第二个文件则是针对特定开发板的文件，它会配置 JTAG 接口（通过引用某个 `target.cfg` 文件来实现）、声明所有可用的 Flash 内存，并且之后你只需按照截止日期完成任务即可，无需再进行其他任何操作。

还有一些，JTAG/OpenOCD 调试更顺畅，开发者在软件代码和硬件设置上需要做的辅助性调整，同样详见[OpenOCD-Project-Setup](https://openocd.org/doc-release/html/OpenOCD-Project-Setup.html)。

##### 如何写好配置文件呢？这里不做介绍，日常中用别人写好的一般没问题，如果遇到确实需要自己编写测试文件的情况，请参照官方的[配置文件指南](https://openocd.org/doc-release/html/Config-File-Guidelines.html)，我认为用到再去查阅即可

#### 3. 传输层 (Transport)/通讯协议
OpenOCD 会根据配置选择是使用 **JTAG**（涉及状态机切换）还是 **SWD**（涉及双线串行协议）。将高级的“读内存”请求转换为一连串的“位操作（Bit-banging）”逻辑。

##### 1. JTAG
> JTAG的核心是一个扫描路径，它通过在芯片内部集成一系列的移位寄存器（称为边界扫描寄存器）来实现。这些寄存器连接在芯片的引脚和内部逻辑之间，形成一个链式结构。

最初的 JTAG 标准 IEEE 1149.1 定义了 5 个引脚，这 5 个引脚统称为 **Test Access Port（TAP）**：
![连接结构](https://image.aruoshui.fun/i/2025/12/22/10fuvcr-0.webp)
> IEEE 1149.7 定义了一个减少了引脚的 JTAG（只保留 TMSC（测试序列号数据）和 TCK（测试时钟）这两个引脚），被称为紧凑型 JTAG （Compact JTAG，cJTAG），它是为了满足不断增长的测试现代电子板和系统的需要而开发的。

- TDI（Test Data Input）：测试数据输入引脚，用于将测试数据串行输入到芯片内部的边界扫描寄存器。
- TDO（Test Data Output）：测试数据输出引脚，用于将芯片内部处理后的测试数据串行输出。
- TMS（Test Mode Select）：测试模式选择引脚，通过该引脚的不同电平组合可以选择不同的测试状态，如测试逻辑复位、数据移位、数据捕获等。
- TCK（Test Clock）：测试时钟引脚，为测试数据的传输和处理提供时钟信号，确保数据的同步操作。
- TRST（Test Reset，可选）：测试复位引脚，用于将JTAG测试逻辑复位到初始状态。
  
以上仅仅是信号线，除此之外还可能有一些其他引脚

- VTREF： 接口信号电平参考电压一般直接连接 Vsupply，通常也是必选的。这个可以用来确定 JTAG 接口使用的逻辑电平！我们常用的 J-Link、ULINK 等都可以由 5V 电压供电，然后其内部则可以转换输出 1.8V ~ 5V 从而直接给芯片供电。
- System Reset ( nSRST)： 可选项，与目标板上的系统复位信号相连，可以直接对目标系统复位。同时可以检测目标系统的复位情况，为了防止误触发应在目标端加上适当的上拉电阻。
- Return Test Clock ( RTCK)： 可选项，由目标端反馈给仿真器的时钟信号，用来同步 TCK 信号的产生，不使用时直接接地。
![JTAG](https://image.aruoshui.fun/i/2025/12/22/z5xb46-0.webp)

详细的调试协议（定义如何通过 JTAG 接口读取边界扫描寄存器）由架构厂商定义，例如 ARM 在 《ARM® Debug Interface v5》规范中给出了详细介绍 DP；RISC-V 则在 《RISC-V Debug Specification》中有详细的描述 DMI。
![调试协议](https://image.aruoshui.fun/i/2025/12/22/10hluhi-0.webp)


##### 2. SWD
>SWD（Serial Wire Debug）即串行调试接口，是ARM公司开发的一种用于调试ARM Cortex系列微控制器的串行通信协议，它为嵌入式系统开发人员提供了一种高效、便捷的方式来对芯片进行调试和编程。

与将 TAP 链接在一起的 JTAG 相反，SWD 使用称为 **DAP（Debug Access Port，调试访问端口）** 的总线。在这个 DAP 上，有一个主站（DP，Debug Port，调试端口）和一个或多个从站（AP，Access Port，接入端口），类似于 JTAG TAP。DP 使用包含 AP 地址的数据包与 AP 通信。
![体系结构](https://image.aruoshui.fun/i/2025/12/22/10lrq8j-0.webp)

SWD 主要通过两根信号线进行通信：
- SWCLK（Serial Wire Clock）：串行时钟线，为数据传输提供时钟信号，确保数据的同步传输。发送端和接收端依据这个时钟信号来协调数据的发送和接收，保证数据传输的准确性。
- SWDIO（Serial Wire Data Input/Output）：串行数据线，是一个双向信号引脚，用于在调试器和目标芯片之间传输数据和命令。在不同的时刻，它既可以作为数据输入线，接收来自调试器的数据和命令；也可以作为数据输出线，将目标芯片的状态信息和调试结果反馈给调试器。

---

### 物理硬件层：从比特到电平
这是链路的最后一公里，涉及两个关键实体：
1. Debug Adapter (适配器/调试器)，[OpenOCD官方罗列了一份调试适配器的说明和链接](https://openocd.org/doc-release/html/Debug-Adapter-Hardware.html)（如 ST-Link（**生态闭合**）, J-Link（缺点一个字：**贵**。但同样有国产的，我自己使用的就是淘的jlinkv9）, DAP-Link（**开源**、充斥在各大购物网站），以及[调试器配置](https://openocd.org/doc-release/html/Debug-Adapter-Configuration.html)
   * **作用：** OpenOCD 通过系统的 USB 驱动（如 Windows 的 WinUSB 或 Linux 的 libusb）控制它。
   * **转换：** 适配器接收 OpenOCD 发来的 USB 数据包，然后迅速地操作其引脚，产生符合 JTAG/SWD 时序的脉冲。
> **官方提醒：** 由于OpenOCD最初专注于JTAG，你可能会发现在某些地方它错误地假设JTAG是唯一的传输协议正在使用。请注意，OpenOCD的最新版本正在移除这一限制。JTAG仍然比大多数其他传输工具更为实用。其他传输不支持边界扫描作，或者可能特定于某一芯片厂商。有些传输可能仅适用于编程闪存，而非调试。

2. Target MCU (目标芯片)
这是链路的终点。
   * **芯片内部调试逻辑：** 芯片内部有一个小小的“调试代理人”（在 ARM 里叫 **DAP**, Debug Access Port）。
   * **执行指令：** 当物理引脚上的电平信号满足协议时，DAP 会强行停止 CPU 内核，读取寄存器值，或者把数据写入 Flash。





