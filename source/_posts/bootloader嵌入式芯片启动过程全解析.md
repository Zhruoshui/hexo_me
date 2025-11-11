---
title: bootloader嵌入式芯片启动过程全解析
abbrlink: 50266
date: 2024-12-25 20:46:37
tags:
 - Bootloader
description:
categories: 
  - 底层开发
cover: https://image.aruoshui.fun/i/2025/11/11/nlr0ue-0.webp
swiper_index:
---

# 参考文章



{% link bootloader的全面解析, https://blog.csdn.net/qq_51004011/article/details/138376644, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp %} 
{% link GD32F450数据手册 %}
{% link STM32 开发必备-内存地址（*****）, https://zhuanlan.zhihu.com/p/648904738, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp %} 
{% link STM32、GD32固件升级IAP, https://blog.csdn.net/RMDYBW/article/details/140552321, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp %} 



# 什么是Bootloader
在嵌入式操作系统中，BootLoader是在操作系统内核运行之前运行。可以初始化硬件设备、建立内存空间映射图，从而将系统的软硬件环境带到一个合适状态，以便为最终调用操作系统内核准备好正确的环境。 
在嵌入式系统中，通常并没有像BIOS那样的固件程序（注，有的嵌入式CPU也会内嵌一段短小的启动程序），因此整个系统的加载启动任务就完全由BootLoader来完成。
**BootLoader 起到了桥梁的作用，连接了硬件启动与高级软件运行之间的环节，确保系统能够从一个初始、裸机的状态过渡到一个完整的、可操作的运行环境。**  


## 两种Bootloader
**MCU的Bootloader和嵌入式linux或pc有所不同，与不同芯片才用的存储架构有关**
1. MCU
MCU一般为单核或多核同构，主频小于1GHZ，也没有MMU内存管理单元，最多只能运行像FreeRTOS等的实时操作系统
MCU下程序运行的主要介质是NOR_FLASH(传统存储器单元，支持随机访问)，与RAM一样有分离的地址线和数据线，以字节长度精确寻址。

2. linux SOC
嵌入式linux的SOC一般将他的操作系统、文件系统和应用程序存放在nand flash（NAND Flash的内部结构更适合大容量、顺序读写的应用场景。它采用页和块的结构，通常需要使用控制器来管理读写操作。）  
在处理器运行代码时，先从nand中到sram内存中，比MCU的多了一步。 

本文以GD32单片机进行BootLoader的配置，后边会逐渐补上 

# Bootloader的作用
1. MCU中
   1. 关闭看门狗，初始化中断和trap向量表，进行时钟和外设初始化，让芯片正常运行起来。
   2. 提供CAN、UART、ETH等用于通讯功能的驱动，能够接收外部数据传输请求
   3. 提供FLASH的读写与擦除驱动，设计服务来对通讯端口接收到的更新代码进行校验、存储，以及跳转操作系统或后续应用程序代码。
   4. 如有必要，还会开发一些基础诊断服务，串口交互程序等等。
2. SoC
   1. 硬件初始化
   2. 内存管理
   3. 引导加载操作系统
      1. 从非易失性存储器（如Flash、EEPROM、NAND/NOR Flash等）中读取并验证操作系统的内核映像。
      2. 将内核映像加载到RAM中指定的位置，并按照内核所需的特定格式设置启动参数和环境变量。
   4. 固件升级
   5. 系统诊断与恢复
   6. 多重引导支持
      在某些系统中，BootLoader 可能支持选择加载不同的操作系统版本或应用程序，提供多启动选项，增强系统的灵活性和可定制性。

# 单片机的程序（指令）生成
交叉编译会将我们的c代码编译成二进制指令，然后编译器会将这些二进制指令烧录到单片机的程序存储区，cpu会去一条条的读取指令执行。

# 指令存放的位置
下面就是我在梁山派gd32F470用户手册里找的一张图片，因为我们将程序（指令）写到单片机中，必须是烧录到固定的位置里面。这样的话单片机才能去找到这些指令并执行以完成我们需要的功能。
![gd32的闪存基地址和构成](https://image.aruoshui.fun/i/2025/01/13/zdibfu-0.webp)

{% note info flat %}烧录进单片机的程序是有固定位置的，并且程序存储区是有固定的大小，所以当生成的程序合适时，程序存储区就可以同时存放几段程序代码。

在理论上来说，只需要确定好boot程序和app程序的大小，给他们在ROM内分配合适的区块。在需要进行代码跳转的时候，做好中断向量表重定向。就可以在一个单片机中存放多块不同的程序段。

{% endnote %}

# GD32的启动模式 
**启动模式只决定程序烧录的位置**，加载完程序之后会有一个重映射(映射到0x00000000地址位置)；真正产生复位信号的时候，CPU还是从开始位置执行。 

{% note success flat %}
1. 初始化堆栈指针 SP=_initial_sp，初始化 PC 指针=Reset_Handler
2. 初始化中断向量表
3. 配置系统时钟
4. 调用 C 库函数_main 初始化用户堆栈，然后进入 main 函数。
{% endnote %}  

1. GD32的三种启动方式  


1)主闪存存储器(Main Flash)启动：从GD32内置的Flash启动(0x0800 0000-0x0807 FFFF)，**一般我们使用JTAG或者SWD模式下载程序时，就是下载到这个里面，重启后也直接从这启动程序。**以0x08000000 对应的内存为例，则该块内存既可以通过0x00000000 操作也可以通过0x08000000 操作，且都是操作的同一块内存。

2)系统存储器(System Memory)启动：从系统存储器启动(0x1FFFF000 - 0x1FFF F7FF)，这种模式启动的程序功能是由厂家设置的。一般来说，我们选用这种启动模式时，是为了从串口下载程序，因为在厂家提供的ISP程序中，提供了串口下载程序的固件，可以通过这个ISP程序将用户程序下载到系统的Flash中。以0x1FFFFFF0对应的内存为例，则该块内存既可以通过0x00000000 操作也可以通过0x1FFFFFF0操作，且都是操作的同一块内存。

3)片上SRAM启动：从内置SRAM启动(0x2000 0000-0x3FFFFFFF)，既然是SRAM，自然也就没有程序存储的能力了，这个模式一般用于程序调试。SRAM 只能通过0x20000000进行操作，与上述两者不同。从SRAM 启动时，需要在应用程序初始化代码中重新设置向量表的位置。


用户可以通过设置BOOT0和BOOT1的引脚电平状态，来选择复位后的启动模式。如下图所示：
![启动模式](https://image.aruoshui.fun/i/2025/01/13/12mvsjd-0.webp)

GD32上电复位以后，代码区都是从0x00000000开始的，三种启动模式只是将各自存储空间的地址映射到0x00000000中。

![总结](https://image.aruoshui.fun/i/2025/01/14/bm83e-0.webp)

## 启动文件分析
### 栈定义 
栈的作用是用于局部变量，函数调用，函数形参等的开销，栈的大小不能超过内部SRAM 的大小。当程序较大时，需要修改栈的大小，不然可能会出现的HardFault的错误。
```armasm
Stack_Size      EQU     0x00000400    
;开辟的栈大小1kb，equ为伪指令
                AREA    STACK, NOINIT, READWRITE, ALIGN=3 
;开辟一段可读可写数据空间，ARER 伪指令表示下面将开始定义一个代码段或者数据段。此处是定义数据段。 
;ARER 后面的关键字表示这个段的属性。段名为STACK，可以任意命名；
;NOINIT 表示不初始化；READWRITE 表示可读可写，ALIGN=3，表示按照 8 字节对齐。
Stack_Mem       SPACE   Stack_Size
;SPACE 用于分配大小等于 Stack_Size连续内存空间，单位为字节。
__initial_sp
;__initial_sp表示栈顶地址。栈是由高向低生长的。
```
### 堆的定义
堆主要用来动态内存的分配，像 malloc()函数申请的内存就在堆中。
```armasm
Heap_Size       EQU     0x00000400
;开辟堆的大小为 0X00000200（512 字节）
                AREA    HEAP, NOINIT, READWRITE, ALIGN=3
;堆的名字为 HEAP，NOINIT 即不初始化，可读可写，8字节对齐。
__heap_base
;__heap_base 表示对的起始地址
Heap_Mem        SPACE   Heap_Size
__heap_limit
;__heap_limit 表示堆的结束地址。
```

### 向量表
向量表是一个WORD（ 32 位整数）数组，每个下标对应一种异常，该下标元素的值则是该 ESR 的入口地址。向量表在地址空间中的位置是可以设置的，通过 NVIC 中的一个重定位寄存器来指出向量表的地址。在复位后，该寄存器的值为 0。因此，在地址 0 （即 FLASH 地址 0）处必须包含一张向量表，用于初始时的异常分配。
```armasm
                AREA    RESET, DATA, READONLY
;定义一块代码段，段名字是RESET，READONLY 表示只读。
                EXPORT  __Vectors
                EXPORT  __Vectors_End
                EXPORT  __Vectors_Size
;使用EXPORT将3个标识符申明为可被外部引用，声明 __Vectors、__Vectors_End 和__Vectors_Size 具有全局属性。

__Vectors       DCD     __initial_sp                      ; Top of Stack
;__Vectors 表示向量表起始地址，DCD 表示分配 1 个 4 字节的空间。
;每行 DCD 都会生成一个 4 字节的二进制代码，中断向量表 存放的实际上是中断服务程序的入口地址。
;当异常（也即是中断事件）发生时，CPU 的中断系统会将相应的入口地址赋值给 PC 程序计数器，之后就开始执行中断服务程序。
                DCD     Reset_Handler                     ; Reset Handler
                DCD     NMI_Handler                       ; NMI Handler
                DCD     HardFault_Handler                 ; Hard Fault Handler
                DCD     MemManage_Handler                 ; MPU Fault Handler
                DCD     BusFault_Handler                  ; Bus Fault Handler
                DCD     UsageFault_Handler                ; Usage Fault Handler
                ·
                ·
                ·
                ·
__Vectors_End
;__Vectors_End 为向量表结束地址。

__Vectors_Size  EQU     __Vectors_End - __Vectors

                AREA    |.text|, CODE, READONLY
; __Vectors_Size则是向量表的大小，向量表的大小是通过__Vectors 和__Vectors_End 相减得到的。
```

### 复位程序
复位程序是系统上电后执行的第一个程序，复位程序也是中断程序 
```armasm
Reset_Handler   PROC
;定义了一个服务程序，PROC表示程序的开始。
                EXPORT  Reset_Handler                     [WEAK]
;使用EXPORT将Reset_Handler申明为可被外部引用，后面WEAK表示弱定义，如果外部文件定义了该标号则首先引用该标号，如果外部文件没有声明也不会出错。这里表示复位程序可以由用户在其他文件重新实现。
                IMPORT  SystemInit
                IMPORT  __main
;表示该标号来自外部文件，SystemInit()是一个库函数，在system_gd32f10x.c中定义的
;__main 是一个标准的 C 库函数，主要作用是初始化用户堆栈，这个是由编译器完成的
;该函数最终会调用我们自己写的main函数，从而进入C世界中。
                LDR     R0, =SystemInit
;从存储器中加载SystemInit到一个寄存器R0的地址中。
;R0~R3 寄存器通常用于函数入参出参或子程序调用。
                BLX     R0
;跳转到寄存器R0的地址，并根据寄存器的 LSE 确定处理器的状态，还要把跳转前的下条指令地址保存到 LR。
                LDR     R0, =__main
                BX      R0
;跳转到至指定寄存器的地址后，不会返回
                ENDP
;和PROC是对应的，表示程序的结束。
```
这里的__main和C语言中的main()不是一样东西，__main是C lib中的函数，也就是在Keil中自带的；而main()函数是C的入口，main()会被__main调用。


### 中断服务程序
我们平时要使用哪个中断，就需要编写相应的中断服务程序，只是启动文件把这些函数留出来了，但是内容都是空的，真正的中断复服务程序需要我们在外部的 C 文件里面重新实现，这里只是提前占了一个位置罢了。
```armasm
NMI_Handler     PROC
                EXPORT  NMI_Handler                       [WEAK]
                B       .
;B表示跳转，这里跳转到一个‘.’，即表示无线循环。
                ENDP
HardFault_Handler\
                PROC
                EXPORT  HardFault_Handler                 [WEAK]
                B       .
                ENDP
MemManage_Handler\
                PROC
                EXPORT  MemManage_Handler                 [WEAK]
                B       .
                ENDP
```

## GD32的启动流程分析
{% note success flat %}
与前面分析的一致
1. 初始化SP、PC、向量表
2. 设置系统时钟，接下来就会进入SystemInit函数中。
3. 初始化堆栈并进入main 

{% endnote %} 

MCU上电后从0x0800 0000处读取栈顶地址并保存，然后从0x0800 0004读取中断向量表的起始地址，这就是复位程序的入口地址，接着跳转到复位程序入口处，初始向量表，然后设置时钟，设置堆栈，最后跳转到C空间的main函数，即进入用户程序。
![总结](https://image.aruoshui.fun/i/2025/01/14/ndr0q-0.webp)


# BootLoader和APP的关系 

上面讲了MCU的整个启动的流程，无论是BootLoader还是APP都必须要按照上面的流程进行启动，只是APP的运行需要在BootLoader中进行跳转（**其实这种跳转就是走楼梯一级一级上**），即在BootLoader对MSP和PC进行重新赋值成APP.bin文件中的参数。
![关系](https://image.aruoshui.fun/i/2025/01/14/r1k86-0.webp)
1. BootLoader：验证下载新固件完整性，从固件备份区拷贝新固件数据到APP区，跳转到APP中；
2. APP：业务应用程序设计，下载新固件到备份区（APP_back）并复位。 

![flash分区](https://image.aruoshui.fun/i/2025/01/14/rzmpd-0.webp)
程序的启动，在bootloader引导程序中，我们会启用标志位，每个app对应一个标志位，如果判断成功就跳转到相应的地址去启动相应的程序。