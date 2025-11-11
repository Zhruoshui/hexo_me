---
title: qemu模拟开发板开发
abbrlink: 34218
date: 2025-02-06 19:34:54
tags:
  - 虚拟机
description:
categories:
  - 开发技能
cover: https://image.aruoshui.fun/i/2025/11/11/nyfpoz-0.webp
swiper_index:
---

# 参考文章

{% link 如何配置 QEMU 虚拟机网络, https://huaweicloud.csdn.net/6707aefbe2ce0119e0a1e3eb.html?dp_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6NjUyNDYwLCJleHAiOjE3NDA2MjcwNjIsImlhdCI6MTc0MDAyMjI2MiwidXNlcm5hbWUiOiJBX3J1b3NodWkifQ.UvK-zaDEx_il-jNwsV5R23KQ9JmvEZ0mjEbYDA-PcRU&spm=1001.2101.3001.6650.15&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Eactivity-15-129685202-blog-131290211.235%5Ev43%5Epc_blog_bottom_relevance_base4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Eactivity-15-129685202-blog-131290211.235%5Ev43%5Epc_blog_bottom_relevance_base4&utm_relevant_index=21, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp %} 

# 使用qemu模拟A9处理器并启动linux全过程记录

## qemu是什么
qemu是一个可以虚拟成硬件的软件，随心所欲地使用调试环境，不同架构的硬件平台并运行虚拟机

## 安装过程
1. 拉取官方源码分支8.2 ，只克隆最近的 5 次提交的历史记录
   `git clone https://gitlab.com/qemu-project/qemu.git .-branch stable-8.2 --depth 5` 
   - 克隆完会发现有很多嵌套的子模块需要克隆，比如`u-boot`，这里使用`git submodule`可以查询还有什么子模块。
   - 下载子模块使用`git submudule update --init --recursive --depth 5` 追踪并递归最新子模块

2. 源码编译、
   {% link Hosts/Linux, https://wiki.qemu.org/Hosts/Linux, https://image.aruoshui.fun/i/2025/02/21/uqa4z4-0.webp %} 
   - 安装工具
   `sudo apt-get install git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev ninja-build`
   ```powershell
    sudo apt-get install git-email
    sudo apt-get install libaio-dev libbluetooth-dev libcapstone-dev libbrlapi-dev libbz2-dev
    sudo apt-get install libcap-ng-dev libcurl4-gnutls-dev libgtk-3-dev
    sudo apt-get install libibverbs-dev libjpeg8-dev libncurses5-dev libnuma-dev
    sudo apt-get install librbd-dev librdmacm-dev
    sudo apt-get install libsasl2-dev libsdl2-dev libseccomp-dev libsnappy-dev libssh-dev
    sudo apt-get install libvde-dev libvdeplug-dev libvte-2.91-dev libxen-dev liblzo2-dev
    sudo apt-get install valgrind xfslibs-dev 
   ```
   - 编译
    ```powershell
    mkdir build
    cd build
    ../configure
    make
    ```

## 模拟vexpress-a9
#### 先下载一下Linux源码
```powershell
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.14.7.tar.xz
tar xvf linux-4.14.7.tar.xz
```
#### 编译内核
   下面命令把vexpress_defconfig作为配置文件保存为.config，并根据这个config中的配置进行编译。
   ```powershell
   make CROSS_COMPILE=arm-linux-gnueabi- ARCH=arm vexpress_defconfig
   make CROSS_COMPILE=arm-linux-gnueabi- ARCH=arm -j8
   ```
   编译得到内核文件arch/arm/boot/zImage，Qemu启动时需要指定使用这个映像文件。
#### 制作根文件系统
   - 文件系统是对存储设备上的数据进行组织的机制，用户与操作系统进行交互的主要工具就是通过**文件系统调用**
   - 根文件系统是Linux内核启动后的第一个挂载的文件系统，主要由最基本的shell命令、各种库、字符设备、配置脚本组成
```powershell
cd busybox-1.35.0/
make defconfig
make CROSS_COMPILE=arm-linux-gnueabi- -j8
make install CROSS_COMPILE=arm-linux-gnueabi- -j8
```
   切换到busybox的上级目录，并使用如下脚本制作镜像：
```powershell
#!/bin/bash

mkdir -p rootfs/{dev,etc/init.d,lib}
touch rootfs/etc/init.d/rcS
#这里用双引号可能会报错
echo -e '#!/bin/sh\n' > rootfs/etc/init.d/rcS
cp busybox-1.35.0/_install/* -r rootfs/
#sudo cp -P /usr/arm-linux-gnueabihf/lib/* rootfs/lib/
sudo cp /usr/arm-linux-gnueabihf/lib/sf/* rootfs/lib/

#ln -s bin/busybox rootfs/init
#ln -s -f bin/busybox rootfs/sbin/init

sudo mknod rootfs/dev/tty1 c 4 1
sudo mknod rootfs/dev/tty2 c 4 2
sudo mknod rootfs/dev/tty3 c 4 3
sudo mknod rootfs/dev/tty4 c 4 4

sudo chown root:root -R rootfs/*
sudo  chmod  777 rootfs/etc/init.d/rcS

qemu-img create -f raw disk.img 512M
mkfs -t ext4 ./disk.img
mkdir  -p   tmpfs
sudo mount -o loop ./disk.img tmpfs/
sudo cp -r rootfs/* tmpfs/
sudo umount tmpfs
file disk.img
```
### 启动~！
```shell
qemu-system-arm \
	-M vexpress-a9 \
	-m 512M \
	-kernel linux-4.14.7/arch/arm/boot/zImage \
	-dtb linux-4.14.7/arch/arm/boot/dts/vexpress-v2p-ca9.dtb \
	-nographic \
	-append "root=/dev/mmcblk0 rw console=ttyAMA0" \
	-sd disk.img

```
关于`qemu-system-arm`命令各个参数详细解释如下：

`-M vexpress-a9`：表示使用vexpress-a9开发板的配置；
`-m 512M`：表示这只内存为512M；
`-kernel xxx/arch/arm/boot/zImage`：表示使用哪个内核镜像；
`-dtb xxx/arch/arm/boot/dts/vexpress-v2p-ca9.dtb`：表示使用哪个dtb文件；
`-nographic`：表示不启动图形化界面；
`-append`：表示设置kernel的cmdline；
`-sd disk.img`：表示使用sd卡上某个文件作为根文件系统；
`qemu-system-arm -M help`：可以查看支持的板子情况。


## 嵌入式启动
### 嵌入式bootloader
- 类似于PC的BIOS、硬件自检是否正常
- 加载系统镜像到RAM
- 设置不同的启动方式

### 常见的启动方式
- NOR/NAND flash 启动
- SD卡启动
- Bootloader从网络加载Linux内核启动

### u-boot
前面我们已经可以正常启动linux了，这是qemu帮忙做好的工作，现在自己来配置启动程序
#### 下载
下载最新的稳定版就行：
https://ftp.denx.de/pub/u-boot/u-boot-2025.01.tar.bz2
#### 修改配置文件
- 进入解压好的目录，用编辑器打开makefile，修改以下配置：
`CROSS_COMPILE ?= arm-linux-gnueabi-`
- 再修改配置config.mk
`ARCH := arm`
#### 编译
`make vexpress_ca9x4_defconfig`
`make -j4`
#### qemu网络功能设置
配置QEMU与主机的网络连接 
• 采用桥接(bridge)的网络连接与Host通信 
• 需要主机内核tun/tap模块支持 
这部分参考：

{% link 如何配置 QEMU 虚拟机网络, https://www.zhaixue.cc/qemu/qemu-u-boot.html, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp %} 

#### NFS服务
{% link 挂载NFS根文件系统, https://www.zhaixue.cc/qemu/qemu-mount_nfs_rootfs.html, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp %} 

