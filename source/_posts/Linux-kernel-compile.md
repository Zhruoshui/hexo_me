---
title: Linux_kernel
abbrlink: 59205
date: 2025-02-13 19:16:28
tags:
description:
categories: 嵌入式Linux
cover:
swiper_index:
---

# 参考文章
{% link 内核模块, https://blog.csdn.net/weixin_45093118/article/details/139397775, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp %} 


# Linux kernel开发模式
![开发模式](https://image.aruoshui.fun/i/2025/02/13/tu9wiw-0.webp)

# 如何构建kernel
1. 获取配套的交叉编译工具链
 - SOC原厂提供:NXPST Rockchip Amlogic Allwinnertech 等
 - 社区下载:Linrao Debian ARM Bootlin下载kernel源码
2. 获取Linux Kernel主线LTS源码
 - Git方式获取: git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
 - 压缩版下载: https://mirrors.edge.kernel.org/pub/linux/kernel/获取芯片原厂Kernel源码
3. Host下配置开发环境
 - 安装必要依赖包
 - 解压配置合适的工具链
4. 指定编译板子配置文件
 - make BOARDNAME defconfig
5. 编译
 - 编译内核镜像 make -jN
 - 编译设备树 make dtbs
 - 编译安装模块驱动 make modules

## kernel两种源码
1. 官方主线版本
   - 遵循内核的开发模式
   - 可能尚未包含特定领域的最新特性
2. 芯片厂商自己的内核源
   - 芯片厂商优先关注硬件的支持
   - 与Linux有非常重要的增量支持
   -  许多SOC供应商同时参与主线的开发
3. 许多内核社区维护自己的内核版本
   - 架构社区(ARM、MIPS、PowerPC等)设备驱动社区(12C、SPI、USB、PCI、网络等)他社区(如real-time等)
   - 无官方发布，仅用于分享工作和为主线版本做出贡献。

# kernel目录介绍

```powershell
linux/
├── arch/             # 架构特定的代码，包括不同硬件平台的特定代码。
├── block/            # 块设备层, 包括文件系统和磁盘驱动程序。
├── crypto/           # 加密算法
├── Documentation/    # 存放内核文档和说明。
├── drivers/          # 包含各种设备驱动程序，如网络、声卡、USB等。
├── fs/               # 提供文件系统的实现，包括虚拟文件系统（VFS）以及各种具体文件系统如ext4、FAT等。
├── include/          # 存放公共头文件，供内核和模块使用。
├── init/             # 包含启动和初始化代码。
├── ipc/              # 提供进程间通信（IPC）机制的实现，如管道、消息队列等。
├── kernel/           # 包含内核核心功能的实现，如调度器、定时器等。
├── lib/              # 提供通用库函数和工具函数。
├── mm/               # 管理内存分配和页面管理。
├── net/              # 处理网络协议栈和网络驱动程序相关代码。
├── samples/          # 示例代码，展示一些特定功能或API的使用方法。
├── scripts/          # 包含一些编译脚本和工具
├── security/         # 提供安全模块和安全相关的功能
├── sound/            # 各种音频设备的驱动程序、音频接口的实现以及其他与音频处理和控制相关的代码文件
├── tools/            # 辅助开发、调试和分析Linux内核和相关组件的工具和实用程序
├── usr/              # 存放用户级别软件和相关文件的位置。
└── virt/             # 用于提供有关虚拟化技术的信息

```

# 配置Kernel编译目标环境变量
主要是配置交叉编译，需要定义`ARCH`和`CROSS_COMPILE`的方式
- 在make 命令中传递两个参数`make ARCH = arm CROSS_COMPILE = arm-linux`
  但是这样配置在make的时候会容易忘记传，导致编译半天不可用
- 将两个变量设置为系统变量，使用`export`命令就行
  但只能工作在当前的终端，如果想要长久使用，可以将设置放入`~/.bashrc`文件中，之后刷新`source ~/.bashrc`就行


# 配置单板配置文件
这里对于嵌入式平台案例（ARM32以上）
- 默认配置文件可用，通常适用于每个CPU系列
- 存储在arch/<arch>/configs/中，并且只是最小的.config文件（仅与默认设置有一些不同）
- make help以查找是否可用于您的平台、
- 要加载默认的配置文件，只要运行`make <CPU>_defconfig`
- 这将覆盖现有的.config文件
- 进入配置菜单界面进行更改配置 （make menuconfig）

# 内核配置项
Linux内核配置项主要包括以下几个方面：
### 1. 处理器类型和特性
- 选择支持的处理器架构（如x86, ARM, MIPS等）。
- 配置处理器特性，如SMP（对称多处理）支持、CPU频率调整等。

### 2. 内存管理
- 配置内存管理选项，如支持大内存页、内存压缩等。
- 选择不同的内存分配器（如SLAB, SLOB, SLUB）。

### 3. 设备驱动
- 选择支持的硬件设备，如网络设备、存储设备、USB设备等。
- 配置特定设备的驱动程序，如NVIDIA GPU驱动、Intel网卡驱动等。

### 4. 文件系统
- 选择支持的文件系统（如ext4, btrfs, xfs等）。
- 配置文件系统的特性，如日志支持、压缩支持等。

### 5. 网络
- 配置网络协议支持，如IPv4, IPv6, TCP/IP等。
- 选择支持的网络功能，如防火墙、NAT、QoS等。

### 6. 安全
- 配置内核安全选项，如SELinux, AppArmor等。
- 选择支持的安全模块和功能。

### 7. 模块支持
- 配置内核模块支持，允许动态加载和卸载模块。
- 选择编译为模块的驱动程序和功能。

### 8. 调试和开发
- 配置内核调试选项，如内核调试信息、内核日志级别等。
- 选择支持的开发工具和功能，如KDB、KGDB等。

### 9. 电源管理
- 配置电源管理选项，如休眠、挂起等。
- 选择支持的电源管理功能，如CPU频率调整、动态电源管理等。

### 10. 其他
- 配置内核版本信息、编译选项等。
- 选择支持的其他功能，如虚拟化、实时内核支持等。

这些配置项可以通过`make menuconfig`、`make xconfig`等工具进行图形化配置，也可以通过直接编辑`.config`文件进行手动配置。

# 内核选项
执行`make menuconfig`等工具后，可以直接在图形界面中配置
比如：![config](https://image.aruoshui.fun/i/2025/02/14/sfr3ik-0.webp)

相关的配置选项有：
![选项](https://image.aruoshui.fun/i/2025/02/14/sgsc0l-0.webp)


# 编译内核
- 在内核源码中执行
- 可以最大化利用多个CPU和I/O资源，记住内核数目/线程，执行 make -j <ncpus * 2 或者 ncpus + 2>
- 可以使用ccache编译器缓存： `export CROSS_COMPILE = 'ccacheriscv64-linux'`

# 生成的镜像文件
![镜像文件](https://image.aruoshui.fun/i/2025/02/14/sl0ec4-0.webp)

# 安装
嵌入式中一般直接网络拷贝替换、tftp服务器替换重新启动就行

# 编译设备树
执行`make dtbs`

# 内核模块
由于Linux内核的整体架构庞大且组件多，若将所有需要的功能都打包到内核，会导致内核会很大且臃肿，为了解决这个问题，Linux提供了Module的机制。
Module机制就是在编译内核时本身并不需要包含所有功能，在需要某些功能时再将对应模块动态的加载到内核中，且模块一旦被加载就和内核中其他部分完全一样，每个内核模块在文件系统中存储为一个单独的文件，必须访问文件系统才能使用驱动模块

## 嵌入式主机模块驱动
需要指定 INSTALL_MOD_PATH 变量来生成模块相关文件并将模块安装在 <目标根文件系统> 而不是 <主机根文件系统>中。示例命令：
`make INSTALL_MOD_PATH=<dir>/ modules_install`

# 使用uboot引导嵌入式设备
## 引导加载程序：
嵌入式系统通常使用特定的引导加载程序，例如 U-Boot（Universal Boot Loader）或 Barebox。这些引导加载程序能够加载内核映像并配置启动参数。
## 配置引导加载程序（ENV）：
在引导加载程序中，需要配置引导命令，指定内核映像的位置、启动参数以及设备树（Device Tree）等信息。这通常在引导加载程序的环境变量中进行配置。
## 加载设备树（Device Tree）：
对于许多嵌入式系统，设备树是一个重要的概念。设备树描述了硬件的结构和配置信息，使得相同的内核映像可以在不同的硬件平台上运行。引导加载程序可能会加载设备树文件并传递给内核。
## 加载内核映像：
引导加载程序通过网络、存储设备或其他途径加载 Linux 内核映像到内存中。这通常是 zImage 或 Image 文件。
## 设置启动参数(bootargs)：
引导加载程序设置启动参数，例如 root 文件系统的位置、内核命令行参数等。这些参数可能包括根文件系统的位置、内核的命令行参数等 。
## 转交控制权给内核：
引导加载程序将控制权转交给内核，使得内核可以开始执行。内核初始化过程中会进行硬件初始化、加载驱动程序等操作。
## 用户空间初始化：
内核初始化完成后，启动用户空间进程。这可能涉及到使用 init 程序或其他初始化系统(busybox)。
用户空间操作系统启动：
一旦用户空间初始化完成，操作系统开始运行，用户可以开始使用嵌入式系统(APP)。 

因此，典型的启动过程是：
- 在内存中的地址 X 加载 zImage
- 在内存中的地址 Y 加载<board>.dtb
- 用bootz X – Y 启动内核（中间的 - 表示没有 initramfs）

# Linux内核启动命令行参数概述

Linux内核启动命令行参数用于在系统启动时传递各种配置选项给内核。这些参数可以通过GRUB或其他引导加载程序进行设置。以下是一些常见的内核启动命令行参数：

## 1. 基本参数
- **root=**：指定根文件系统的设备。例如：`root=/dev/sda1`。
- **init=**：指定启动时运行的初始进程。例如：`init=/bin/bash`。
- **ro**：以只读方式挂载根文件系统。
- **rw**：以读写方式挂载根文件系统。

## 2. 内存管理
- **mem=**：指定可用内存大小。例如：`mem=512M`。
- **quiet**：减少启动时的输出信息。
- **verbose**：增加启动时的输出信息。

## 3. 网络
- **ip=**：指定静态IP地址。例如：`ip=192.168.1.10::192.168.1.1:255.255.255.0::eth0:off`。
- **net.ifnames=0**：禁用新的网络接口命名规则，使用传统的命名方式（如eth0）。

## 4. 文件系统
- **fstab**：指定文件系统的挂载选项。例如：`fstab=LABEL=ROOT / ext4 defaults 1 1`。
- **rootflags=**：指定根文件系统的挂载选项。例如：`rootflags=data=writeback`。

## 5. 安全
- **selinux=0**：禁用SELinux。
- **apparmor=0**：禁用AppArmor。

## 6. 调试
- **debug**：启用内核调试信息。
- **earlyprintk**：在早期启动阶段输出调试信息。例如：`earlyprintk=vga,keep`。
- **initcall_debug**：调试内核初始化调用。

## 7. 电源管理
- **acpi=off**：禁用ACPI支持。
- **noapic**：禁用APIC支持。

## 8. 其他
- **console=**：指定控制台设备。例如：`console=ttyS0,115200`。
- **nmi_watchdog=0**：禁用NMI watchdog。
- **panic=**：指定内核panic时的等待时间（以秒为单位）。例如：`panic=10`。

这些参数可以通过在GRUB配置文件中设置或在引导加载程序的命令行中直接传递。例如，在GRUB配置文件中，可以在`linux`行添加这些参数：

```grub
menuentry 'Linux' {
    linux /boot/vmlinuz-5.10.0-21-amd64 root=/dev/sda1 ro quiet
    initrd /boot/initrd.img-5.10.0-21-amd64
}
```

在启动时，这些参数会被传递给内核，以控制内核的行为和配置。


# 升级Kernel子版本
其实就是打补丁
1. 完整的压缩包版本，下载之后解压
2. 版本之间的增量补丁按照正确的顺序应用了正确的补丁以升级到下一个版本。那么可以参考快速下载补丁并应用。

补丁是两个源代码版本之间的差异，使用 `diff` 工具或更复杂的版本控制系统生成


## patc命令使用
patch 命令：
- 在标准输入上获取补丁内容。
- 将补丁描述的 修改应用到当前目录。

Pacth命令使用示例：
```powershell
patch -p <n> < diff_file
cat diff_file | patch -p <n>
xzcat diff_file.xz | patch –p <n>
zcat diff_file.gz | patch –p <n>
```

注意事项：

n : 文件路径中要跳过的目录级别数。 (-p: prune)
您可以使用 -R 选项反向应用补丁。
您可以使用 --dry-run 选项测试补丁。

## 应用Linux Patch补丁​
两种类型的 Linux 补丁：

- 要么适用于之前的稳定版本（从 x.< y-1 > 至 x.y ）
- 或者对当前稳定版本进行修复（从 x.y 至 x.y.z）

可以下载 gzip 或 xz（小得多）压缩文件。始终为patch -p1 生成。

需要在顶层内核源码目录下运行patch命令。

![例子](https://image.aruoshui.fun/i/2025/02/14/sztbt0-0.webp)


# linux busybox
## 根文件系统
所谓制作根文件系统，就是创建各种目录，并且在目录里创建相应的文件。例如：在/bin目录下放置可执行程序，在/lib下放置各种库等等。通常配合chroot命令使用。

## busybox
Busybox将众多的UNIX命令集合进一个很小的可执行程序中，可以用来替代GNU fileutils、shellutils等工具集。Busybox中各种命令与相应的GNU工具相比，所能提供的选项比较少，但是也足够一般的应用了。Busybox主要用于嵌入式系统。

在创建根文件系统的时候，如果使用Busybox的话，只需要在/dev目录下创建必要的设备节点，在/etc目录下增加一些配置文件即可，当然，如果Busybox使用动态链接，那么还需要再/lib目录下包含库文件。

详细的配置使用可以看
{% link linux busybox详解, https://blog.csdn.net/m0_66596286/article/details/135559755, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp %} 


