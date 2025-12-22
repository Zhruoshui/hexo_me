---
title: 从OpenOCD看嵌入式调试器
description: 嵌入式调试器硬件、软件架构一览，以OpenOCD出发
categories:
  - 嵌入式开发
abbrlink: 43169
date: 2025-10-22 10:40:57
tags: 
  - 嵌入式调试
  - openocd
  - probe-rs
cover: https://image.aruoshui.fun/i/2025/12/22/f93l60-0.webp
swiper_index:
---

# 参考文章
{% link 知乎, https://zhuanlan.zhihu.com/p/666861211,  https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp%} 

{% link CSDN, https://hackaday.com/2012/09/27/beginners-look-at-on-chip-debugging/,  https://image.aruoshui.fun/i/2025/12/22/f201jo-0.webp%} 

{% link github, https://zhuanlan.zhihu.com/p/666861211,  https://image.aruoshui.fun/i/2025/01/13/p2416y-0.webp%} 

{% link 稀土掘金, https://zhuanlan.zhihu.com/p/666861211,  https://image.aruoshui.fun/i/2025/09/22/hijfn6-0.webp%}

{% link arXiv, https://zhuanlan.zhihu.com/p/666861211, https://image.aruoshui.fun/i/2025/10/18/x4rcnb-0.webp %}


# 调试器(debug-probe)
在嵌入式系统工程中，调试器并非简单的外设工具，而是连接开发者逻辑思维与物理芯片底层运行状态的神经中枢。随着微处理器架构从早期的单核八位演进到复杂的异构多核架构，调试器的作用已从基础的程序下载扩展到了实时的系统洞察与性能剖析。

## OpenOCD（Open On-chip Debugger）
OpenOCD是一款开源的跨平台片上调试器，提供针对嵌入式设备的调试、系统编程和边界扫描功能。其工作方式就是代替了原有那些调试适配器提供的相关工具和驱动， 直接通过普通的 USB 驱动访问适配器，进而访问目标硬件。
> OpenOCD 是由 Dominic Rath 作为他 2005 年在 Augsburg 应用科学大学毕业论文的一部分而创建,论文链接:http://openocd.org/files/thesis.pdf

## 纵向：打通“软件命令”到“物理电平”的链路

### **实时系统洞察与执行控制**

调试器的核心价值在于其提供的“停止并凝视”（Stop and Stare）能力 1。在传统的代码插桩调试（如使用 printf 语句）中，开发者通过串口输出观察程序进度，但这种方法具有显著的局限性。printf 函数的执行不仅消耗闪存空间，还会引入不可预测的时间延迟，这种非确定性（Non-deterministic）往往会掩盖竞态条件或导致实时任务错过截止时间 1。

相比之下，基于硬件支持的调试器通过处理器的调试访问端口（DAP）或测试访问端口（TAP）直接控制内核指令流 3。这种机制允许开发者在不改变系统时序的前提下，精确地暂停执行、检查 CPU 寄存器状态以及访问整个内存映射空间 3。

| 功能维度 | 软件插桩 (printf) | 硬件调试器 (OCD) |
| :---- | :---- | :---- |
| **执行干扰** | 显著（改变运行时间） | 极小或无（硬件断点） |
| **内存访问** | 受限于软件定义的输出 | 完全访问（包括外设寄存器） |
| **执行控制** | 无法实现单步执行 | 支持单步、跳出、运行到指定位置 |
| **异常捕获** | 难以捕获硬错误（Hard Fault） | 可捕获实时异常并定位堆栈 |

### **硬件资源的深度访问与状态分析**

现代调试器能够与芯片内部的调试架构（如 ARM 的 CoreSight）紧密集成，从而实现对系统总线的透明访问 3。通过调试器，工程师可以实时观察 DMA 控制器的数据传输进度、ADC 的转换结果寄存器，甚至是缓存（Cache）的命中情况。这种深度访问能力对于解决复杂的底层硬件驱动问题和外设配置错误至关重要。

此外，调试器还承担了生产环境与开发环境之间的桥梁作用。它通过专用的协议（如 JTAG 或 SWD）向闪存（Flash）烧录固件，并验证数据的完整性 3。在多核系统中，高级调试器甚至能够同步控制多个内核的启动与停止，这对于解决核间通信（IPC）中的同步问题具有不可替代的作用 3。

# 主流开发板及其适配调试器

嵌入式市场呈现出明显的芯片原厂生态驱动特征。主要的半导体供应商通常会为其处理器系列开发专属的调试工具链，以确保对特定内核特性的最大化支持。

### **意法半导体（ST）生态：ST-Link 系列**

作为全球应用最广泛的微控制器系列，STM32 拥有最为成熟的调试工具链。ST-Link 是 ST 官方推出的调试器，其发展经历了从基础转换器到多功能接口的演进 8。

| 调试器型号 | 支持接口 | 核心特性 | 适配开发板系列 |
| :---- | :---- | :---- | :---- |
| **ST-LINK/V2** | SWIM, JTAG, SWD | 标准调试下载，支持 STM8 与 STM32 8 | STM32 Discovery, BluePill 10 |
| **ST-LINK/V2-ISOL** | JTAG, SWD | 具备 $1000 \\text{ Vrms}$ 数字隔离能力 8 | 高压、工业控制应用板卡 |
| **ST-LINK/V3SET** | SWD, JTAG, Bridge | 高速接口，支持 SPI/I2C/CAN 桥接 11 | STM32 高性能评估板 (EVAL) |
| **ST-LINK-V3MINI** | SWD | 体积小巧，集成虚拟串口 (VCOM) | STM32 Nucleo-64/144 9 |

ST-Link 的成功在于其与集成开发环境（IDE）如 STM32CubeIDE 的深度整合。板载的 ST-Link 能够通过 USB 同时枚举为三个接口：CDC（虚拟串口）、MSC（大容量存储，用于拖拽式烧录）和 DFU（用于自身固件升级） 9。

### **恩智浦（NXP）生态：MCU-Link 架构**

NXP 引入了统一的 MCU-Link 架构，旨在简化其 ARM Cortex-M 微控制器的开发流程。该架构基于高性能的 LPC55S69 双核微控制器，能够提供极高的通信吞吐量 12。

| 特性分类 | MCU-Link (标准/OB) | MCU-Link Pro |
| :---- | :---- | :---- |
| **主控芯片** | LPC55S69 (150 MHz) | LPC55S69 (150 MHz) 12 |
| **电压支持** | $1.2 \\text{ V} \\sim 5 \\text{ V}$ | $1.2 \\text{ V} \\sim 5 \\text{ V}$ 12 |
| **调试协议** | CMSIS-DAP, J-Link (可选) | CMSIS-DAP, J-Link (可选) 12 |
| **高级功能** | VCOM, SWO | 电流测量 ($\< 200 \\text{ nA}$), 目标供电 12 |

MCU-Link Pro 版本通过集成特殊的电流感测电路，允许开发者在调试代码的同时，实时监测目标系统的功耗曲线，这对于开发依赖电池供电的低功耗物联网设备极具价值 12。

### **德州仪器（TI）生态：XDS 系列**

TI 的 XDS 系列调试器以其对 DSP（数字信号处理）核心和高性能微处理器的卓越支持而著称。其产品线覆盖了从低成本入门级到高性能系统追踪的完整频谱 13。

* **XDS110**: 定位于平衡成本与功能的通用探针。它通过 30 针扩展接口支持 EnergyTrace 技术，能够为 MSP432 等超低功耗微控制器提供精细的能量消耗分析 15。  
* **XDS200**: 中端平衡方案，支持 IEEE 1149.1 (JTAG) 和 IEEE 1149.7 (cJTAG) 协议。相比 XDS110，它在数据传输带宽上具有明显优势，适合处理较大的固件下载 13。  
* **XDS560v2**: 专业级高性能调试器，支持系统追踪（System Trace）。其 Pro Trace 接收器版本能够捕获通过专用引脚输出的高速指令流，是分析复杂异构系统实时行为的关键工具 13。

### **乐鑫（Espressif）与沁恒（WCH）生态**

在物联网和新兴微控制器领域，乐鑫与沁恒分别建立了自己的调试规范。

* **ESP-Prog**: 乐鑫官方推出的调试板，核心为 FT2232HL 芯片。它不仅支持 ESP32 的 JTAG 调试，还集成了自动下载电路，能够通过控制 IO0 和 EN 引脚状态实现固件的自动烧录 18。  
* **内置 USB-JTAG**: 在 ESP32-S3 和 C3 等较新型号中，乐鑫将 USB 控制器集成在硅片内部。这使得开发者无需外部调试器，仅需一根 USB 线即可实现 JTAG 访问 20。  
* **WCH-LinkE**: 沁恒为 RISC-V 架构（如 CH32V 系列）设计的调试器。它支持 RISC-V 的两线调试接口，并可通过按键切换为传统的 ARM-SWD 模式，具有极高的灵活性 21。

### **第三方通用调试器：SEGGER J-Link**

在所有第三方调试器中，J-Link 被公认为工业标准。其广泛的架构支持（从 8051 到 Cortex-A 系列）和卓越的驱动稳定性使其成为专业开发者的首选 23。

| 模型对比 | RAM 下载速度 | 最大采样频率 | 核心技术 |
| :---- | :---- | :---- | :---- |
| **J-Link Base** | $1.0 \\text{ MB/s}$ | $15 \\text{ MHz}$ | 无限闪存断点 23 |
| **J-Link Ultra+** | $4.0 \\text{ MB/s}$ | $50 \\text{ MHz}$ | 高速 FPGA 逻辑加速 23 |
| **J-Link Pro** | $4.0 \\text{ MB/s}$ | $50 \\text{ MHz}$ | 以太网接口，支持 TCP/IP 远程调试 23 |
| **J-Link WiFi** | $1.0 \\text{ MB/s}$ | $15 \\text{ MHz}$ | 无线调试，消除物理线缆干扰 23 |

J-Link 的 RTT（Real-Time Transfer）技术是其核心竞争力之一。通过在内存中建立环形缓冲区，RTT 实现了比传统串口高出几个数量级的数据传输速度，且完全不占用 CPU 资源 23。

## **底层通信协议：JTAG, SWD 与 CMSIS-DAP 的深度演进**

调试器与目标芯片之间的对话建立在严谨的物理层与链路层协议之上。

### **JTAG (IEEE 1149.1) 与 cJTAG (IEEE 1149.7)**

JTAG 是最早期的片上调试标准，最初用于电路板级边界扫描测试。它使用五根核心信号线：TCK（时钟）、TMS（模式选择）、TDI（数据输入）、TDO（数据输出）和可选的 TRST（复位） 3。JTAG 的优势在于支持菊花链（Daisy-chain）连接，可以在一条总线上挂载多个芯片 3。

为了适应移动设备对引脚数量的严苛要求，cJTAG 将信号线压缩为两根，同时保留了 JTAG 的所有逻辑功能。这种改进在 TI 的无线连接芯片（如 CC26xx 系列）中得到了广泛应用 14。

### **SWD (Serial Wire Debug) 协议**

SWD 是 ARM 为 Cortex 内核开发的专用调试协议。它仅使用两根线：SWDIO（双向数据）和 SWCLK（时钟） 3。相比 JTAG，SWD 在引脚占用上减少了 $60\\% \\sim 75\\%$，且在高速传输时具有更好的信号完整性 3。SWD 还支持多跌落（Multi-drop）技术，允许在两根线上寻址多个调试端点 26。

### **CMSIS-DAP 标准的崛起**

为了打破各硬件供应商之间的专有协议壁垒，ARM 提出了 CMSIS-DAP 规范。这是一种基于 USB 的标准化通信接口，直接定义了主机如何通过 USB 控制调试接口的寄存器 4。

* **CMSIS-DAP v1**: 基于 USB HID 类。虽然不需要安装驱动程序，但受限于 HID 的轮询机制，数据传输速率通常限制在 $64 \\text{ KB/s}$ 左右 4。  
* **CMSIS-DAP v2**: 基于 USB Bulk 传输模式。它利用了 WinUSB 驱动的优势，显著提升了数据吞吐量，并能够更好地处理高速 SWO 追踪数据流 6。

## **当前主流调试软件生态：技术架构与应用分析**

软件工具链是调试器的灵魂。从开源的 OpenOCD 到现代化的 Probe-rs，软件层决定了开发者的交互体验。

### **OpenOCD：开源界的基石**

OpenOCD (Open On-Chip Debugger) 是目前嵌入式领域最强大的开源调试服务器软件。它采用了高度模块化的分层设计 27。

1. **接口层 (Interface)**: 负责管理各种硬件探针（如 ST-Link, J-Link, FT2232）。  
2. **目标层 (Target)**: 定义了不同内核（如 Cortex-M3, RISC-V, ESP32）的调试寄存器访问方法。  
3. **闪存层 (Flash)**: 包含数百种芯片的算法脚本，用于固件烧录。

OpenOCD 通过 Tcl 脚本进行配置，这赋予了其极强的定制能力 10。例如，在调试 STM32 时，可以通过 \-f interface/stlink.cfg \-f target/stm32f1x.cfg 指令快速启动服务器 10。在 $0.12.0$ 版本中，OpenOCD 显著增强了对 RISC-V 架构的支持，并改进了对 AArch64 追踪功能的支持 26。

### **PyOCD：Python 驱动的自动化调试**

PyOCD 是一个由 Python 编写的纯软件工具包，主要针对 ARM Cortex-M 设备。其最大的特色在于其“Python 原生”的 API 接口 29。

使用 PyOCD，开发者可以通过简单的 Python 脚本实现复杂的生产线测试自动化：

Python

from pyocd.core.helpers import ConnectHelper  
with ConnectHelper.session\_with\_chosen\_probe() as session:  
    target \= session.board.target  
    target.reset\_and\_halt()  
    target.write\_memory(0x20000000, 0xDEADBEEF) \# 内存写入 \[30\]  
    session.board.target.dp.write\_reg(0x8, 0x1) \# 直接寄存器操作 \[30\]

PyOCD 集成了完善的测试框架，通过其 automated\_test.py 脚本，开发者可以在 CI 流水线中自动验证固件在不同硬件探针上的烧录成功率和执行正确性 29。

### **Probe-rs：Rust 时代的高性能工具**

Probe-rs 是近年来崛起的新一代调试工具，完全由 Rust 语言编写。它旨在解决 OpenOCD 配置过于复杂且运行效率较低的痛点 5。

* **高性能**: 凭借 Rust 的内存安全和零成本抽象，Probe-rs 在 Flash 下载速度上常常能超越官方工具。  
* **易用性**: 它提供了 cargo-flash 和 cargo-embed 等集成工具，使得 Rust 开发者可以像运行本地应用一样运行嵌入式程序 32。  
* **现代交互**: Probe-rs 推出的 VS Code 扩展（probe-rs-debugger）通过微软的 DAP（Debug Adapter Protocol）协议，提供了比传统 GDB 更加流畅的图形化调试体验 32。

## **驱动与后端：USB 栈管理与系统兼容性研究**

在 Windows 环境下，调试器驱动的安装往往是初学者的“噩梦”。这涉及到了 USB 通信类别的深层差异。

### **WinUSB 与 HID 的博弈**

调试器主要通过两种方式与操作系统交互。HID（人机接口设备）类具有免驱优势，但由于其设计初衷是处理鼠标键盘等低频信号，其带宽受限且延迟较高 4。WinUSB 是微软推出的通用驱动框架，允许用户态应用程序直接进行批量传输（Bulk Transfer），这正是高速调试器所需要的 36。

当 OpenOCD 或 Probe-rs 需要访问一个传统的 ST-Link 或 ESP-Prog 时，通常需要使用 Zadig 工具手动将设备驱动替换为 WinUSB。然而，这种替换具有排他性。例如在 ESP32C3 这种集成了 CDC 和 JTAG 的复合设备上，如果将父设备替换为 WinUSB，可能会导致串口丢失 38。现代调试探针通过实现 CMSIS-DAP v2 解决了这一问题，它们在 USB 描述符中显式指定调试接口为 WinUSB，从而保持了功能的并存 11。

## **调试自动化的未来：CI/CD 与 AI 辅助**

嵌入式开发正逐步向 DevOps 模型转型。调试器在这一过程中扮演了硬件节点代理的角色。

### **硬件在线测试集群**

在成熟的 CI 流程中，实验室中维护着挂载了数十个不同调试器的测试机架。每当开发者提交代码，CI 服务器会自动调用 PyOCD 或 OpenOCD，将编译出的二进制文件分发到各个硬件节点上进行并行回归测试 40。

$$\\text{效率提升} \\approx \\frac{\\text{手动调试时间} \- \\text{自动化脚本编写时间}}{\\text{测试执行次数}}$$

### **AI 辅助调试的兴起**

随着大语言模型（LLM）的发展，Probe-rs 等工具正尝试将调试信息反馈给 AI 代理。通过 MCP（Model Context Protocol）协议，AI 可以直接读取处理器的寄存器快照和内存堆栈，从而根据反汇编结果自动分析系统崩溃（如 HardFault）的根本原因 43。这种“AI-Native”的硬件交互模式将极大地降低嵌入式开发的入门门槛。

## **总结与建议**

在选择嵌入式调试器时，开发者应当基于项目生命周期的不同阶段进行权衡：

1. **原型开发阶段**: 优先选择芯片原厂提供的集成工具（如 ST-Link 或板载调试器），以获得最完整的库支持和示例代码 8。  
2. **性能优化阶段**: 推荐使用 SEGGER J-Link 及其 RTT 技术，通过高速数据监测定位系统的性能瓶颈 23。  
3. **自动化生产与 CI 阶段**: PyOCD 与 Probe-rs 是更好的选择，它们的脚本化能力能够将调试过程完美融入现代软件开发流水线 31。

从 1986 年第一款真正意义上的嵌入式调试器 XRAY 发布至今，这一领域已经走过了近四十年的历程 1。从简单的电平翻转到如今的 AI 辅助分析，调试器的演进不仅是工具的迭代，更是嵌入式系统工程方法论的进步。掌握这些主流调试器及其背后的技术细节，是每一位专业嵌入式工程师通往高级架构师之路的必经门槛。

#### **引用的著作**

1. Debugging with printf() or not ... \- Embedded Software, 访问时间为 十二月 21, 2025， [https://blogs.sw.siemens.com/embedded-software/2013/04/08/debugging-with-printf-or-not/](https://blogs.sw.siemens.com/embedded-software/2013/04/08/debugging-with-printf-or-not/)  
2. For me, the main advantage of printf debugging is that it gives a lot of output, 访问时间为 十二月 21, 2025， [https://news.ycombinator.com/item?id=12746492](https://news.ycombinator.com/item?id=12746492)  
3. Debugging an Embedded System \- Day 1 \- Introducing different Tools, 访问时间为 十二月 21, 2025， [https://pyjamabrah.com/posts/debugging-arm64-day1-different-tools/](https://pyjamabrah.com/posts/debugging-arm64-day1-different-tools/)  
4. CMSIS DAP \- Handbook \- Mbed, 访问时间为 十二月 21, 2025， [https://os.mbed.com/handbook/CMSIS-DAP](https://os.mbed.com/handbook/CMSIS-DAP)  
5. Tooling \- The Embedded Rust Book \- Rust Documentation, 访问时间为 十二月 21, 2025， [https://doc.rust-lang.org/beta/embedded-book/intro/tooling.html](https://doc.rust-lang.org/beta/embedded-book/intro/tooling.html)  
6. Firmware for CoreSight Debug Access Port \- GitHub Pages, 访问时间为 十二月 21, 2025， [https://arm-software.github.io/CMSIS\_5/DAP/html/index.html](https://arm-software.github.io/CMSIS_5/DAP/html/index.html)  
7. 5.1. Introduction to OpenOCD — Nuclei Development Tool Guide 2025.10 documentation, 访问时间为 十二月 21, 2025， [https://doc.nucleisys.com/nuclei\_tools/openocd/intro.html](https://doc.nucleisys.com/nuclei_tools/openocd/intro.html)  
8. ST-LINK/V2 | Tool \- STMicroelectronics, 访问时间为 十二月 21, 2025， [https://www.st.com/en/development-tools/st-link-v2.html](https://www.st.com/en/development-tools/st-link-v2.html)  
9. Tutorial 9: ST Link Debugger in STM32 \- YouTube, 访问时间为 十二月 21, 2025， [https://www.youtube.com/watch?v=tx-bz9z6v2c](https://www.youtube.com/watch?v=tx-bz9z6v2c)  
10. Debugging STM32 and ESP32 targets in an IDE \- FAQ \- PlatformIO Community, 访问时间为 十二月 21, 2025， [https://community.platformio.org/t/debugging-stm32-and-esp32-targets-in-an-ide/4048](https://community.platformio.org/t/debugging-stm32-and-esp32-targets-in-an-ide/4048)  
11. Debug probes \- pyOCD, 访问时间为 十二月 21, 2025， [https://pyocd.io/docs/debug\_probes.html](https://pyocd.io/docs/debug_probes.html)  
12. MCU-Link Debug Probe Architecture | NXP Semiconductors, 访问时间为 十二月 21, 2025， [https://www.nxp.com/design/design-center/software/development-software/mcu-link-debug-probe-architecture:MCU-LINK-ARCHITECTURE](https://www.nxp.com/design/design-center/software/development-software/mcu-link-debug-probe-architecture:MCU-LINK-ARCHITECTURE)  
13. TMDSEMU200-U \- DigiKey, 访问时间为 十二月 21, 2025， [https://mm.digikey.com/Volume0/opasdata/d220001/medias/docus/694/TMDSEMU200-U\_Web.pdf](https://mm.digikey.com/Volume0/opasdata/d220001/medias/docus/694/TMDSEMU200-U_Web.pdf)  
14. XDS200 Debug Probe \- Texas Instruments, 访问时间为 十二月 21, 2025， [https://software-dl.ti.com/ccs/esd/documents/xdsdebugprobes/emu\_xds200.html](https://software-dl.ti.com/ccs/esd/documents/xdsdebugprobes/emu_xds200.html)  
15. TMDSEMU110-U \- XDS110 JTAG Debug Probe \- Evelta, 访问时间为 十二月 21, 2025， [https://evelta.com/tmdsemu110-u-xds110-jtag-debug-probe/](https://evelta.com/tmdsemu110-u-xds110-jtag-debug-probe/)  
16. TI XDS110 Debug Probe | Embedded Development Tool | TMDSEMU110-U, 访问时间为 十二月 21, 2025， [https://www.techonicsltd.com/ti-xds110-debug-probe-embedded-development-tool-tmdsemu110-u/](https://www.techonicsltd.com/ti-xds110-debug-probe-embedded-development-tool-tmdsemu110-u/)  
17. XDS560 High Performance Debug Probe, 访问时间为 十二月 21, 2025， [https://static6.arrow.com/aropdfconversion/6a53e24f63016f90e6a6d141ce6204e10167f5d7/pgurl\_xds560.pdf](https://static6.arrow.com/aropdfconversion/6a53e24f63016f90e6a6d141ce6204e10167f5d7/pgurl_xds560.pdf)  
18. Debug your embedded software with ESP32, ESP-PROG, and JTAG. \- GitHub, 访问时间为 十二月 21, 2025， [https://github.com/PBearson/ESP32-With-ESP-PROG-Demo](https://github.com/PBearson/ESP32-With-ESP-PROG-Demo)  
19. Introduction to the ESP-Prog Board \- Sparkfun, 访问时间为 十二月 21, 2025， [https://cdn.sparkfun.com/assets/b/1/3/7/b/DS-19099-ESP-PROG.pdf](https://cdn.sparkfun.com/assets/b/1/3/7/b/DS-19099-ESP-PROG.pdf)  
20. What is an easy way to get into debugging on ESP32s? \- Reddit, 访问时间为 十二月 21, 2025， [https://www.reddit.com/r/esp32/comments/1aoojgr/what\_is\_an\_easy\_way\_to\_get\_into\_debugging\_on/](https://www.reddit.com/r/esp32/comments/1aoojgr/what_is_an_easy_way_to_get_into_debugging_on/)  
21. WCH-LinkUserManual, 访问时间为 十二月 21, 2025， [https://akizukidenshi.com/goodsaffix/WCH-LinkUserManual.pdf](https://akizukidenshi.com/goodsaffix/WCH-LinkUserManual.pdf)  
22. WCH LinkE Online Download Debugger CH32V003 from Maker go on Tindie, 访问时间为 十二月 21, 2025， [https://www.tindie.com/products/adz1122/wch-linke-online-download-debugger-ch32v003/](https://www.tindie.com/products/adz1122/wch-linke-online-download-debugger-ch32v003/)  
23. SEGGER J-Link debug probes, 访问时间为 十二月 21, 2025， [https://www.segger.com/products/debug-probes/j-link/](https://www.segger.com/products/debug-probes/j-link/)  
24. J-Link Now Supports CMSIS-DAP \- SEGGER, 访问时间为 十二月 21, 2025， [https://www.segger.com/news/segger-j-link-now-supports-cmsis-dap/](https://www.segger.com/news/segger-j-link-now-supports-cmsis-dap/)  
25. JTAG Debugging \- ESP32 \- — ESP-IDF Programming Guide v5.5.1 documentation, 访问时间为 十二月 21, 2025， [https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/jtag-debugging/index.html](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/jtag-debugging/index.html)  
26. Releases · openocd-org/openocd \- GitHub, 访问时间为 十二月 21, 2025， [https://github.com/openocd-org/openocd/releases](https://github.com/openocd-org/openocd/releases)  
27. riscv-collab/riscv-openocd: Fork of OpenOCD that has RISC-V support \- GitHub, 访问时间为 十二月 21, 2025， [https://github.com/riscv-collab/riscv-openocd](https://github.com/riscv-collab/riscv-openocd)  
28. openocd 0.12.0-1 (amd64 binary) in ubuntu noble \- Launchpad, 访问时间为 十二月 21, 2025， [https://launchpad.net/ubuntu/noble/amd64/openocd/0.12.0-1](https://launchpad.net/ubuntu/noble/amd64/openocd/0.12.0-1)  
29. pyocd/pyOCD: Open source Python library for programming and debugging Arm Cortex-M microcontrollers \- GitHub, 访问时间为 十二月 21, 2025， [https://github.com/pyocd/pyOCD](https://github.com/pyocd/pyOCD)  
30. Automated tests \- pyOCD, 访问时间为 十二月 21, 2025， [https://pyocd.io/docs/automated\_tests.html](https://pyocd.io/docs/automated_tests.html)  
31. probe-rs/probe-rs: A debugging toolset and library for debugging embedded ARM and RISC-V targets on a separate host \- GitHub, 访问时间为 十二月 21, 2025， [https://github.com/probe-rs/probe-rs](https://github.com/probe-rs/probe-rs)  
32. probe-rs and cargo-embed \- Comprehensive Rust \- Google, 访问时间为 十二月 21, 2025， [https://google.github.io/comprehensive-rust/bare-metal/microcontrollers/probe-rs.html](https://google.github.io/comprehensive-rust/bare-metal/microcontrollers/probe-rs.html)  
33. Debugger for probe-rs \- Visual Studio Marketplace, 访问时间为 十二月 21, 2025， [https://marketplace.visualstudio.com/items?itemName=probe-rs.probe-rs-debugger](https://marketplace.visualstudio.com/items?itemName=probe-rs.probe-rs-debugger)  
34. libusb not work with HID device \- NTDEV \- OSR Developer Community, 访问时间为 十二月 21, 2025， [https://community.osr.com/t/libusb-not-work-with-hid-device/48981](https://community.osr.com/t/libusb-not-work-with-hid-device/48981)  
35. Thread: \[OpenOCD-devel\] OpenOCD on Windows and USB issues due to misunderstanding | OpenOCD \- Open On-Chip Debugger \- SourceForge, 访问时间为 十二月 21, 2025， [https://sourceforge.net/p/openocd/mailman/openocd-devel/thread/51CF7C64.1070007@akeo.ie/](https://sourceforge.net/p/openocd/mailman/openocd-devel/thread/51CF7C64.1070007@akeo.ie/)  
36. CMSIS-DAPv2 support · Issue \#585 · ARMmbed/DAPLink \- GitHub, 访问时间为 十二月 21, 2025， [https://github.com/ARMmbed/DAPLink/issues/585](https://github.com/ARMmbed/DAPLink/issues/585)  
37. ESP32-C3 LIBUSB error (openocd, probe.rs) (IDFGH-7149) · Issue \#133 · espressif/idf-installer \- GitHub, 访问时间为 十二月 21, 2025， [https://github.com/espressif/idf-installer/issues/133](https://github.com/espressif/idf-installer/issues/133)  
38. Parallel programing using pyocd \#1125 \- GitHub, 访问时间为 十二月 21, 2025， [https://github.com/pyocd/pyOCD/discussions/1125](https://github.com/pyocd/pyOCD/discussions/1125)  
39. Embedded CI/CD \- IAR, 访问时间为 十二月 21, 2025， [https://www.iar.com/embedded-development-tools/embedded-ci-cd](https://www.iar.com/embedded-development-tools/embedded-ci-cd)  
40. Streamline Your Development: Embedded CI/CD, its Value and Essential Tools, 访问时间为 十二月 21, 2025， [https://www.beningo.com/streamline-your-development-embedded-ci-cd-its-value-and-essential-tools/](https://www.beningo.com/streamline-your-development-embedded-ci-cd-its-value-and-essential-tools/)  
41. How to define your ideal embedded CI/CD pipeline, 访问时间为 十二月 21, 2025， [https://www.embedded.com/how-to-define-your-ideal-embedded-ci-cd-pipeline/](https://www.embedded.com/how-to-define-your-ideal-embedded-ci-cd-pipeline/)  
42. The Ultimate Guide to the Embedded Debugger (probe-rs) MCP Server \- Skywork.ai, 访问时间为 十二月 21, 2025， [https://skywork.ai/skypage/en/ultimate-guide-embedded-debugger/1981209361482960896](https://skywork.ai/skypage/en/ultimate-guide-embedded-debugger/1981209361482960896)
