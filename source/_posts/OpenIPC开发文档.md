---
title: OpenIPC开发文档
abbrlink: 9117
date: 2025-01-09 15:13:14
tags:
description:
categories:
cover:
swiper_index:
---

{% timeline 跟踪日志,blue %}

<!-- timeline 2024/1/9 -->
通过 TFTP 和 UART 逐步安装 OpenIPC 固件。
<!-- endtimeline -->

<!-- timeline 2024/**/** -->

<!-- endtimeline -->


<!-- timeline 2024/**/** -->

<!-- endtimeline -->

{% endtimeline %}


# 参考文章
{% link 参考文章, https://github.com/OpenIPC/wiki/blob/master/en/installation.md, https://image.aruoshui.fun/i/2025/01/13/p2416y-0.webp %} 
# 说明
核心就是利用网络摄像头，而且网络监控的市场很大，网络摄像头可选择的范围很大，可以根据主控芯片、图像传感器的型号，以及板载的串口（刷鞋固件、通信），USB(连接WIFI网卡)的情况自由选择
我这里选择的是 **SigmaStar SsC338Q  分辨率4K@20FPS  内置2Gb DDR3**

# 架构：
![架构](https://image.aruoshui.fun/i/2025/01/02/sf653e-0.webp)
# 我的工作任务：
- 硬件环境的搭建
  - openipc固件的烧写
  - 天空端电路搭建
  - 地面端环境的烧写
- 固件的编译和编译脚本
- 裁剪的openipc linux内核中执行启动脚本的过程
- rtl8812网卡天空端驱动（内核配置的）、rtl8812au地面端驱动（开源项目）
- 摄像头驱动（开源的），视频编码器（闭源方案）
- wfb-ng数据链路
- 地面端解码方案（安卓、linux、嵌入式开发板）

# 固件安装
这里是利用的TFTP和UART来安装固件
### SoC识别
SoC（系统级芯片）包括相机的 CPU 内核以及所有必要的外围设备，例如相机和网络接口。**这部分由摄像头厂商确定**，需要看IC标记丝印来查看，或者使用ipctool软件识别SoC型号

### 闪存芯片大小
通常是摄像头8引脚的一个芯片，可以通过U-boot启动过程确认，也可以查看丝印（丝印包含64，即是8M，包含128，即16M）

### TFTP服务器
TFTP（Trivial File Transfer Protocol）服务器是一种简单的文件传输协议服务器，用于在网络设备之间传输文件。与FTP（File Transfer Protocol）相比，TFTP更轻量级，通常用于在局域网（LAN）中传输小文件，如配置文件、固件更新等。 

说白了就是用它来传固件的

### 连接相机的UART端口
最好是看摄像头手册 好用的就是一个usb转ttl的串口适配器
![](https://image.aruoshui.fun/i/2025/01/13/xl7rg7-0.webp)

将适配器上的引脚连接到 UART 端口的可能触点。使用标准电源适配器为相机供电。如果幸运，就可以在终端窗口中看到 Booting log。在某些情况下，如果在屏幕上看到乱码文本而不是引导内核，需要将连接速度更改为 57600 bps，然后重试。RX、TX


### 访问bootloader
通过按计算机键盘上的组合键来引导加载程序控制台， 在 bootloader 启动和 Linux 内核启动之前。 在上电的时候疯狂enter

进入u-boot的命令行
```powershell
setenv ipaddr 10.81.1.230;setenv serverip 10.81.1.102
mw.b x21000000 x x1000000
tftpboot 0x21000000 openipc-ssc338q-fpv-16mb.bin
tftp 0x21000000 openipc-ssc338q-fpv-16mb.binsf probe 0;sf lock 0;
sf erase 0x0 0x1000000;sf write 0x21000000 0x0 0x1000000
reset
```

 U-Boot 命令系列显示了一个典型的嵌入式系统启动流程，尤其是在网络启动（TFTP）和闪存操作方面的详细步骤。

1. **`setenv ipaddr 10.81.1.230` 和 `setenv serverip 10.81.1.102`**
这两个命令设置了 U-Boot 环境变量，用于指定网络设置,用于通过 TFTP 下载内核或文件系统镜像：
   - **`ipaddr`**：这设置了 U-Boot 启动时使用的本机 IP 地址，`10.81.1.230` 是设备将会使用的 IP 地址。
   - **`serverip`**：指定了 TFTP 服务器的 IP 地址，`10.81.1.102` 是存储内核镜像的服务器。
1. **`mw.b x21000000 x x1000000`**
    清空或初始化内存区域，确保内存中没有残留数据
   - **`mw.b`** 是 U-Boot 中的 "Memory Write Byte" 命令，它用于在内存中写入数据。
   - **`x21000000`** 是目标内存地址，这里是指将数据写入设备内存的地址 `0x21000000`（通常是设备的 RAM）。
   - **`x`** 是要写入的值，代表数据的内容。
   - **`x1000000`** 是写入的字节数，表示要写入的字节数为 `0x1000000`（即 16MB）。



2. **`tftpboot 0x21000000 openipc-ssc338q-fpv-16mb.bin`**
    通过网络从 `10.81.1.102` 服务器上获取 `openipc-ssc338q-fpv-16mb.bin` 文件，并将其存储到设备内存中，准备进行闪存写入。
   - **`tftpboot`** 命令从 TFTP 服务器下载文件，并将文件加载到指定的内存地址。
   - **`0x21000000`** 是目标内存地址，这里表示将下载的文件存储在内存地址 `0x21000000` 开始的位置。
   - **`openipc-ssc338q-fpv-16mb.bin`** 是要下载的文件名，通常是嵌入式设备的固件或内核镜像文件。



4. **`sf probe 0; sf lock 0;`**
    在闪存上执行写入操作时，相关区域不会被意外覆盖。
   - **`sf probe 0`**：初始化并识别闪存设备，`0` 是闪存设备的编号。在 U-Boot 中，`sf` 是指闪存（SPI Flash），这个命令确保设备能够识别并与闪存进行通信。
   - **`sf lock 0`**：锁定闪存的第一个区域，通常用于防止闪存区域被意外擦除或写入。
  
5. **`sf erase 0x0 0x1000000`**
    这个命令用于擦除闪存的特定区域，擦除操作确保闪存上的旧数据被清除，为写入新的镜像做准备。：
   - **`sf erase 0x0 0x1000000`** 表示擦除从地址 `0x0` 开始，大小为 `0x1000000`（即 16MB）的闪存区域。
  
6. **`sf write 0x21000000 0x0 0x1000000`**
   将内核镜像（或者其他固件文件）写入到设备的闪存中，为下次启动准备好内核。
   - **`sf write`** 用于将数据从内存写入闪存。
   - **`0x21000000`** 是内存中的数据起始地址，前面的 TFTP 操作已经将内核镜像存放在该地址。
   - **`0x0`** 是闪存的起始地址，表示将数据写入闪存的第一个位置。
   - **`0x1000000`** 是写入的字节数，表示将 16MB 的数据从内存写入闪存。



7. **`reset`**
   - **`reset`** 命令重启设备，使设备重新启动并从闪存或其他启动介质加载操作系统。



U-Boot 命令，设备完成了以下操作：

1. 配置了网络设置（`ipaddr` 和 `serverip`）。
2. 清空并初始化了内存区域。
3. 从 TFTP 服务器下载了一个固件镜像到内存。
4. 对闪存进行了操作，包括擦除和写入内核镜像。
5. 最终触发系统重启，准备从新写入的镜像启动。


### 保存原始固件
### 固件烧写
### 串口登录设置
### 网络配置及远程登录
### 连接wifi网卡
### 检查WiFi模块的识别情况
### 生成和安装 WFB-NG 的密钥配对
### 编辑 wfb.conf 以设置正确的 wifi 频道
### 在相机上配置 majestic.yaml 文件

# 视频延时组成

视频链路：摄像头==》编码==》传输==》解码==》显示
整个图传系统的延时主要是：编码延时、传输延时、解码延时：
1. 摄像头数据采集延时(camera)
2. 编码器编码延时(H264 codec)
3. 无线网络延时(wfb_ng)： ~ 5ms
4. 解码器解码延时(H264 decoder)
5. 显示器刷新延时(monitor refresh rate)

## 解码延时
解码延时中包含
- Wfb 解包时间
- 内核队列延迟 
- 硬件解码时间

### gstreamer
Jetson 的 NVIDIA V4L2 解码器（如 nvv4l2decoder）通过专用硬件模块（如 NVDEC）实现 H.264/H.265 视频流的硬解码，显著降低 CPU 负载并减少处理延迟

**V4L2 驱动：**
V4L2 是 Linux 内核中用于视频设备的标准接口。
- 用户空间程序（如 GStreamer）访问摄像头数据。
- Jetson 平台上的 V4L2 驱动经过优化，能够直接将数据传递到硬件加速器（如 NVDEC 和 NVENC），从而减少 CPU 的负载。

当摄像头驱动捕获到一帧数据后，会将其放入内核中的缓冲区队列中。GStreamer 应用程序通过 V4L2 接口从内核队列中读取数据。

![v4l2](https://image.aruoshui.fun/i/2025/04/21/xce35n-0.webp)

**GStreamer 缓冲区与网络弹性**

动态缓冲管理：
- 流媒体场景中，通过监听 GST_MESSAGE_BUFFERING 消息动态调整缓冲区大小。例如，当缓冲级别低于 100% 时暂停流水线，待缓冲恢复后继续播放，避免因网络波动导致的数据饥饿。

时钟同步机制：
- GStreamer 全局时钟（如 GST_CLOCK_TYPE_REALTIME）的丢失会触发重新同步。处理 GST_MESSAGE_CLOCK_LOST 消息时，需暂停并重启流水线以重建时钟基准。

使用 queue 插件限制缓冲区大小
`udpsrc port=5600 ! queue max-size-buffers=2 ! rtph265depay ! ...`

基础的解码脚本：
```powershell
#!/bin/bash
current_date=$(date +'%Y%d%m_%H%M%S')
cd ~/Videos

if [[ $1 == "save" ]]
then
	gst-launch-1.0 -e udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H265' ! rtph265depay ! h265parse ! tee name=t ! queue ! mppvideodec ! xvimagesink sync=false t. ! queue ! matroskamux ! filesink location=record_${current_date}.mkv
else
	gst-launch-1.0 udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H265' ! rtph265depay ! h265parse ! mppvideodec ! xvimagesink sync=false
fi

```

在 jetson 上查看是否调用硬件解码的方法是使用 jtop 工具，具体方法参考[《 NVIDIA查看CPU、内存、GPU使用情况 》](https://blog.csdn.net/zong596568821xp/article/details/80268034)

[jetson上的硬件解码器](https://docs.nvidia.com/jetson/archives/r35.6.1/DeveloperGuide/SD/Multimedia/AcceleratedGstreamer.html)



# 地面站
## 接收端网卡
目前开发的地面站支持的大多是rtl8812au这一款网卡（主要是网卡驱动问题）
并支持monitor 模式

## monitor模式
### 什么是monitor模式
WiFi Monitor模式需要WiFi芯片本身支持，并且驱动要支持相应的接口。

在非Monitor模式 （平时正常使用的状态）下，内核会将802.11帧封装成普通网络帧传递给上层； 而在Monitor模式 下，内核则会直接将802.11帧传给上层，不会进行封装，用户层就通过接口拿到RAW包，可以按802.11帧格式进行包解析处理。

在Linux内核中，hostap_80211_rx 函数是IEEE 802.11接收无线skb的tasklet函数，其作用是处理802.11网卡传递过来的数据包。倘若网卡被设置成monitor模式，该函数中会调用如下分支：
```c
 if (local->iw_mode == IW_MODE_MONITOR) {

    
     monitor_rx(dev, skb, rx_stats);
    
     return;
    
 }
```
在monitor_rx 函数中，主要是prism2_rx_80211 函数，将带有802.11头的skb直接发送给netif。netif为linux内核网络数据包的标准框架。在prism2_rx_80211中，在skb里补充了一个抓包的头，给用户提供更多的包信息。这个头对应的数据结构为linux_wlan_ng_cap_hdr ，具体声明如下：
```c
 struct linux_wlan_ng_cap_hdr {

    
     __be32 version;
    
     __be32 length;
    
     __be64 mactime;
    
     __be64 hosttime;
    
     __be32 phytype;
    
     __be32 channel;
    
     __be32 datarate;
    
     __be32 antenna;
    
     __be32 priority;
    
     __be32 ssi_type;
    
     __be32 ssi_signal;
    
     __be32 ssi_noise;
    
     __be32 preamble;
    
     __be32 encoding;
    
 } __packed;
```
### 如何开启
使用如下命令可以实现：
`iwconfig wlan0 mode Monitor`
其调用了如下ioctl来配置：
`ret = ioctl(skfd, SIOCSIWMODE, &wrq);`
对应的配置模式，通过wrq参数来定义。而上面的skfd则由下面操作获取：
`skfd = socket(AF_INET, SOCK_DGRAM, 0);`


## 网卡选择
网卡官方支持：
RTL8812AU、ar9271、rtl8812eu

https://forums.developer.nvidia.com/t/rtl8822ce-access-point-mode/288083
板载算力板上的网卡型号为rtl8812CE，不支持monitor mode


## WIfibroadcast的原理分析：
### 远距离wifi技术：
{% link Wi-Fi极限谈1：最大传输距离的“标准”答案, https://zhuanlan.zhihu.com/p/121872101,  https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp%} 

2.4GHz 频率
RSSI（Received Signal Strength Indicator）值代表设备接收到的信号强度，通常用负值表示。RSSI 值范围一般如下：

接近 0（如 -30）: 信号非常强，设备与信号源的距离较近。
中等值（如 -50 至 -70）: 信号质量良好，适合正常通信。
较弱值（如 -80 至 -90）: 信号较弱，可能会影响通信质量。
非常弱（小于 -90）: 信号几乎不可用，设备可能会断开连接。
解读你的 RSSI 值
你当前的 RSSI 值是 -41：

这是一个非常强的信号，表示设备离信号源很近，通信质量应该非常好。

pkt/s代表每秒传输的数据包数量（packets per second）。这是一种衡量网络性能的指标，用于表示网络接口在一秒钟内可以发送或接收的数据包的数量。这个数值可以帮助评估网络的负载情况和传输效率。较高的pkt/s值意味着网络接口能够处理更多的数据包

8 Mbps (MCS #1 调制) 涉及到的是无线网络通信中的两个概念：数据传输速率和调制编码方案（MCS，Modulation and Coding Scheme）。

1. **8 Mbps**：指的是数据传输速率，即每秒可以传输的数据量为8兆比特。这是一个衡量网络速度的指标，表示理论上一秒钟内可以传送8兆比特（Mb）的数据。

2. **MCS #1**：MCS代表调制编码方案，它用于指定在无线通信中使用的调制方式和编码率。不同的MCS索引号对应着不同的调制方式和编码率组合，从而影响数据传输速率和可靠性。MCS #1通常指使用相对较低复杂度的调制和编码策略，以确保更稳定的传输质量，特别是在信号条件不是最优的情况下。

对于802.11n标准（Wi-Fi 4），MCS #1一般对应于使用BPSK（二进制相移键控）调制和1/2编码率。这意味着每个符号携带1个比特的信息，并且有一半的数据位被用于前向纠错编码，以增强数据传输的可靠性。在单空间流（Single Spatial Stream）和20 MHz带宽的条件下，这种配置可以达到大约7.2 Mbps到8 Mbps的数据传输速率。

因此，“8 Mbps (MCS #1调制)”意味着在网络使用特定的调制和编码设置（在此例中为MCS #1，涉及BPSK调制和1/2编码率）时，能够实现的最大理论数据传输速率为8 Mbps。需要注意的是，实际传输速率可能会受到环境因素、设备性能等多种因素的影响。


#### 目标板配置
`devices/ssc338q_fpv_openipc-urllc-aio/br-ext-chip-sigmastar/configs/ssc338q_fpv_openipc-urllc-aio_defconfig#L103`

`BR2_PACKAGE_WIFIBROADCAST=y`配置好选项

#### 软件版配置
`general/package/wifibroadcast/wifibroadcast.mk#L7`

`WIFIBROADCAST_VERSION = 24.08`

#### 视频数据发送 & 接收

`general/package/wifibroadcast/files/wifibroadcast#L109-L117`

运行：
```powershell
start_drone_wfb() {
	wfb_tx -p "$stream" -u "$udp_port" -R "$rcv_buf" -K "$keydir/$unit.key" -B "$bandwidth" \
		-M "$mcs_index" -S "$stbc" -L "$ldpc" -G "$guard_interval" -k "$fec_k" -n "$fec_n" \
		-T "$pool_timeout" -i "$link_id" -f "$frame_type" -C 8000 "$wlan" > /dev/null &
}

start_gs_wfb() {
	wfb_rx -c "$udp_addr" -u "$udp_port" -p "$stream" -K "$keydir/$unit.key" -i "$link_id" "$wlan" > /dev/null &
}
```


### datalink
`devices/ssc338q_fpv_openipc-urllc-aio/br-ext-chip-sigmastar/configs/ssc338q_fpv_openipc-urllc-aio_defconfig#L102`

BR2_PACKAGE_DATALINK=y

#### 软件版配置
`general/package/datalink/files/telemetry#L13-L16`

```powershell
if [ ! -f /usr/bin/telemetry_rx ] && [ ! -f /usr/bin/telemetry_tx ]; then
	ln -s /usr/bin/wfb_rx /usr/bin/telemetry_rx
	ln -s /usr/bin/wfb_tx /usr/bin/telemetry_tx
fi
```

`general/package/datalink/files/telemetry_drone.conf#L15-L20`

```powershell
stream_rx=144
stream_tx=16
link_id=7669206
frame_type=data
port_rx=14551
port_tx=14550
```


`general/package/datalink/files/telemetry_gs.conf#L15-L20`
```powershell
stream_rx=16
stream_tx=144
link_id=7669206
frame_type=data
port_rx=14651
port_tx=14650
```

#### 数据发送 & 接收
```powershell
start_drone_telemetry() {
	if [ "$one_way" = "false" ]; then
		telemetry_rx -p "$stream_rx" -u "$port_rx" -K "$keydir/$unit.key" -i "$link_id" "$wlan" > /dev/null &
	fi
	telemetry_tx -p "$stream_tx" -u "$port_tx" -K "$keydir/$unit.key" -B "$bandwidth" \
		-M "$mcs_index" -S "$stbc" -L "$ldpc" -G "$guard_interval" -k "$fec_k" -n "$fec_n" \
		-T "$pool_timeout" -i "$link_id" -f "$frame_type" "$wlan" > /dev/null &
}

start_gs_telemetry() {
	if [ "$one_way" = "false" ]; then
		telemetry_tx -p "$stream_tx" -u "$port_tx" -K "$keydir/$unit.key" -B "$bandwidth" \
			-M "$mcs_index" -S "$stbc" -L "$ldpc" -G "$guard_interval" -k "$fec_k" -n "$fec_n" \
			-T "$pool_timeout" -i "$link_id" -f "$frame_type" "$wlan" > /dev/null &
	fi
	telemetry_rx -p "$stream_rx" -u "$port_rx" -K "$keydir/$unit.key" -i "$link_id" "$wlan" > /dev/null &
}
```

## 🧱 **WiFiBroadcast (WFB-TX) 架构分析图**

```
main(int argc, char* const *argv)
│
├── 命令行参数解析（getopt）
│    ├── 设置运行模式：LOCAL / DISTRIBUTOR / INJECTOR
│    ├── FEC 参数：k, n, fec_delay, fec_timeout
│    ├── 网络配置：UDP 端口 / Unix socket / 接收方地址列表
│    ├── 加密参数：keypair 文件路径
│    ├── 日志设置：log_interval
│    ├── 控制接口：control_port
│    └── 物理层参数：bandwidth, MCS index, VHT mode, STBC/LDPC 等
│
├── 系统熵检查（确保随机数安全）
│    └── 检查 /dev/random 的熵池是否足够用于加密 key 生成
│
├── libsodium 初始化
│    └── sodium_init()：初始化加密库，用于 FEC session key 和数据加密
│
├── radiotap header 初始化
│    └── 根据物理层参数构造 IEEE80211_RADIOTAP_HDR，用于注入原始帧
│
├── 根据 tx_mode 启动不同主循环
│
│    ┌────────────────────────────────────────────────────────────┐
│    │ case LOCAL: 本地发送模式（发射端）                        │
│    │   ├─ 使用 local_loop_udp 或 local_loop_unix                │
│    │   ├─ 打开 raw socket 向 wlanX 注入原始数据包               │
│    │   └─ 创建 LocalTransmitter 实例负责 FEC 编码和加密         │
│    │                                                            │
│    │    ┌──────────────────────────────────────────────────┐    │
│    │    │ LocalTransmitter                                  │    │
│    │    │   ├── FEC 编码 (Reed-Solomon k/n)                 │    │
│    │    │   ├── 数据分块与 FEC 分组                           │    │
│    │    │   ├── 加密（使用 libsodium）                       │    │
│    │    │   ├── 时间戳与 epoch 管理                          │    │
│    │    │   └── tags 管理（用于 FEC block 标识）             │    │
│    │    └──────────────────────────────────────────────────┘    │
│    │                                                            │
│    │    ┌──────────────────────────────────────────────────┐    │
│    │    │ data_source_local(): 主事件循环                   │    │
│    │    │   ├── 监听输入源（如 stdin 或 video pipe）        │    │
│    │    │   ├── 触发 FEC 编码                               │    │
│    │    │   ├── 发送 FEC block 到所有目标设备               │    │
│    │    └──────────────────────────────────────────────────┘    │
│    │                                                            │
│    └────────────────────────────────────────────────────────────┘
│
│    ┌────────────────────────────────────────────────────────────┐
│    │ case DISTRIBUTOR: 接收端（分布式接收器）                  │
│    │   ├─ 使用 distributor_loop_udp 或 distributor_loop_unix    │
│    │   ├─ 多路复用多个 UDP/Unix socket                         │
│    │   └─ 创建 RemoteTransmitter 实例处理 FEC 解码              │
│    │                                                            │
│    │    ┌──────────────────────────────────────────────────┐    │
│    │    │ RemoteTransmitter                              │    │
│    │    │   ├── FEC 解码                                 │    │
│    │    │   ├── 包重组                                   │    │
│    │    │   ├── 支持 FEC 超时机制                        │    │
│    │    │   └── 数据恢复并注入到 wlanX 设备              │    │
│    │    └────────────────────────────────────────────────┘    │
│    │                                                            │
│    │    ┌──────────────────────────────────────────────────┐    │
│    │    │ data_source(): 主事件循环                       │    │
│    │    │   ├── 接收远程 FEC block                        │    │
│    │    │   ├── FEC 解码并恢复丢失的数据包                │    │
│    │    │   ├── 控制指令处理（修改 FEC 参数等）           │    │
│    │    └────────────────────────────────────────────────┘    │
│    │                                                            │
│    └────────────────────────────────────────────────────────────┘
│
│    ┌────────────────────────────────────────────────────────────┐
│    │ case INJECTOR: 测试注入器                                │
│    │   └─ injector_loop()：模拟 FEC block 注入测试流量          │
│    └────────────────────────────────────────────────────────────┘
│
├── 异常捕获（try-catch）
│    └── 捕获 runtime_error 并输出错误信息后退出
│
└── 正常退出
```

---

## 📊 总结说明

| 模块 | 功能 |
|------|------|
| main() | 程序入口，负责参数解析、初始化、启动主循环 |
| LocalTransmitter | 负责 FEC 编码、加密、时间戳管理、标签分配 |
| RemoteTransmitter | 负责 FEC 解码、丢包恢复、数据注入到 wlanX |
| data_source / data_source_local | 主事件循环，处理数据流和控制信号 |
| FEC 编解码 | 基于 Reed-Solomon(k,n)，支持前向纠错 |
| 加密机制 | 使用 libsodium 库进行 session key 管理和数据加密 |
| Radiotap Header | 封装物理层参数，兼容多种无线网卡 |
| 控制接口 | 支持通过 control_port 动态调整 FEC 参数 |
| 日志系统 | 输出运行状态、丢包率、延迟等关键指标 |

---

## 📎 示例命令对照


| 模式 | 命令示例 | 对应函数 |
|------|----------|----------|
| 发射端 | `wfb_tx -u 5600 -k 8 -n 12 -f data wlan0` | `local_loop_udp()` |
| 接收端 | `wfb_rx -d -u 5600 host:5600` | `distributor_loop()` |
| 本地测试 | `wfb_rx -U /tmp/wfb -d host:port` | `distributor_loop_unix()` |
| 注入测试 | `wfb_rx -I 10000 wlan0` | `injector_loop()` |

## Linux 内核启动过程
一旦 U-Boot 把内核加载到内存并将控制权交给内核，Linux 内核就开始启动。内核启动的过程可以分为几个重要的阶段：


## 内核说明
### 内核初始化：

解压内核：如果内核是压缩格式（uImage），它会首先被解压。
设置系统环境：内核初始化过程中，会初始化硬件环境，包括内存、CPU、时钟、I/O 控制器等。
启动内核主线程：内核开始执行第一个用户空间进程，通常是 init 进程。
设备驱动加载：内核会根据设备树（Device Tree，DT）或者硬编码的设备信息来加载相关硬件设备的驱动，确保硬件能够被正确识别并与操作系统交互。

内核调度和管理资源：内核的调度器开始工作，管理系统资源，如 CPU、内存、I/O 等。

### init 进程和用户空间启动
在 Linux 内核初始化完毕后，控制权会传递给第一个用户空间进程 init，它通常会启动系统中所有的守护进程和服务。

- 挂载 /proc 文件系统。
- 检查并挂载支持的根文件系统。
- 通过 overlay 或 overlayfs 文件系统进行联合挂载，允许在嵌入式系统中以最小化的方式实现根文件系统的可写层。
- 使用 pivot_root 更改根文件系统。
- 挂载其他系统关键文件系统（如 /proc、/dev 和 /overlay）。

#### RootFS 的内容
   
在 Buildroot 中，根文件系统（RootFS）是由多个目录和文件构成的，这些目录和文件用于支持系统运行。常见的文件夹有：

/bin/：基本的可执行文件，如 sh、ls 等。
/sbin/：系统管理程序，如 init、ifconfig 等。
/etc/：系统配置文件，如网络配置、服务启动配置等。
/lib/：共享库。
/tmp/：临时文件。
/proc/：虚拟文件系统，提供系统信息。
/sys/：虚拟文件系统，提供内核信息。
/dev/：设备文件。

```powershell
├── bin
├── dev
├── etc
├── init
├── lib
├── lib32 -> lib
├── linuxrc -> bin/busybox
├── media
├── mnt
├── opt
├── overlay
├── proc
├── rom
├── root
├── run
├── sbin
├── sys
├── tmp
├── usr
├── utils
└── var
```


在这些目录中，init 文件是根文件系统中用于启动系统的第一个进程。

#### init 与文件系统的关系
在Linux嵌入式系统的开发和维护过程中，了解系统启动和服务管理机制是至关重要的。init 文件本身就是嵌入在构建的文件系统中的。


其中，/etc/init.d/目录扮演了关键角色，它包含了用于初始化、启动、重启或停止各种服务的脚本。
init.d是指包含一系列 Shell脚本 的目录，这些脚本用于控制服务（也称为守护进程）的生命周期。当系统启动时，init进程会根据预定义的规则执行这些脚本，以确保必要的服务能够正确启动。

```powershell
rcK         S01syslogd  S30customizer  S40network   S60crond     S98datalink
rcS         S02klogd    S35modules     S49ntpd      S70vendor    S98vtun
S01seedrng  S02sysctl   S38mdev        S50dropbear  S95majestic  S99rc.local
```




#### 系统脚本运行情况
{% folding rcS%}

先看一下rcS文件，它是本目录在开机时最先启动的文件。也是系统进入多用户模式之前的初始化脚本。它会根据配置启动各种系统服务，通常会依赖其他脚本来执行特定的任务。

rcS通常用于：
- 初始化基本系统服务
- 挂载文件系统
- 启动 init.d 目录下的所有 S 开头的脚本
- init.d 目录结构遵循了 SysV init 启动方式，rcS 主要是用来遍历 S 开头的脚本，并依次执行它们。
- 在使用 **Buildroot** 进行内核裁剪之后，init进程的启动工作流会根据 `/etc/init.d` 中的启动脚本顺序来执行。

```powershell
#!/bin/sh
export SENSOR=$(fw_printenv -n sensor)
export UPGRADE=$(fw_printenv -n upgrade)
export TZ=$(cat /etc/TZ)

for i in /etc/init.d/S??*; do
	[ ! -f "$i" ] && continue
	case "$i" in
		*)
			$i start
			;;
	esac
done
```
- 循环遍历 `/etc/init.d/` 目录下所有以 `S` 开头且后面跟有两位数字（`S??*`）的文件。
- `[ ! -f "$i" ] && continue` 检查每个文件是否是普通文件（不是目录等其他类型），如果不是普通文件，则跳过该文件。
- `case "$i" in *) $i start ;; esac` 对每个文件（即每个服务脚本）执行 `start` 参数。触发服务脚本的启动。服务脚本内会定义如何启动服务（如启动系统守护进程、初始化硬件设备、配置网络等）。

{% endfolding %}


{% folding rcK%}

与rcS相对
这个脚本通常在系统关闭或者进入单用户模式时运行。它主要用来停止一些服务，清理系统资源。
```powershell
#!/bin/sh

for i in $(ls -r /etc/init.d/S??*); do
	[ ! -f "$i" ] && continue
	case "$i" in
		*)
			$i stop
			;;
	esac
done
```

{% endfolding %}


{% folding S01seedrng%}

作用是启动一个随机数生成服务，确保系统在启动时能够使用持久化的种子数据来生成加密级别的随机数。其主要作用是增强系统的随机性，确保用于加密和安全任务时生成的随机数质量高，防止系统依赖不安全或预测性的随机数。脚本还支持动态配置种子存储位置和一些特定的安全选项。

{% endfolding %}

{% folding S01syslogd%}
`S01syslogd` 脚本的作用是启动和管理 `syslogd` 服务，`syslogd` 是一个日志守护进程，用于收集、存储和转发系统日志。这个脚本确保 `syslogd` 正常运行，并在系统启动时自动启动它。

```bash
DAEMON="syslogd"
PIDFILE="/var/run/$DAEMON.pid"
DAEMON_ARGS="-n -C64 -t"
```
- `DAEMON="syslogd"`：定义了守护进程的名称，即 `syslogd`，它是负责处理系统日志的服务。
- `PIDFILE="/var/run/$DAEMON.pid"`：定义了守护进程的 PID 文件位置，`/var/run/syslogd.pid` 用于存储 `syslogd` 的进程 ID。
- `DAEMON_ARGS="-n -C64 -t"`：定义了 `syslogd` 启动时的参数：
  - `-n`：告诉 `syslogd` 以非守护进程模式启动，即不将其转为后台进程。
  - `-C64`：设置日志缓冲区大小为 64 KB。
  - `-t`：标记每条日志信息，通常用于调试日志输出。

### `start` 函数
```bash
start() {
	echo -n "Starting $DAEMON: "
	start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "$DAEMON" -- $DAEMON_ARGS
	if [ $? -eq 0 ]; then
		echo "OK"
	else
		echo "FAIL"
	fi
}
```
- `start` 函数用于启动 `syslogd`：
  - `start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "$DAEMON" -- $DAEMON_ARGS`：该命令通过 `start-stop-daemon` 启动守护进程 `syslogd`，并将其参数传递给进程。选项解析：
    - `-b`：使进程在后台运行。
    - `-m`：以守护进程的方式启动。
    - `-S`：启动进程时保持锁定。
    - `-q`：安静模式，不显示多余信息。
    - `-p "$PIDFILE"`：指定存储 PID 文件的位置。
    - `-x "$DAEMON"`：指定要启动的守护进程程序，即 `syslogd`。
    - `-- $DAEMON_ARGS`：传递参数给 `syslogd`，如日志缓冲区大小等。

- 启动成功后输出 `OK`，失败则输出 `FAIL`。

### `stop` 函数
```bash
stop() {
	echo -n "Stopping $DAEMON: "
	start-stop-daemon -K -q -p "$PIDFILE"
	if [ $? -eq 0 ]; then
		rm -f "$PIDFILE"
		echo "OK"
	else
		echo "FAIL"
	fi
}
```
- `stop` 函数用于停止 `syslogd`：
  - `start-stop-daemon -K -q -p "$PIDFILE"`：使用 `start-stop-daemon` 停止守护进程，`-K` 表示终止进程，`-q` 表示安静模式，`-p "$PIDFILE"` 用于指定 PID 文件。
  - 停止成功后，删除 PID 文件并输出 `OK`，否则输出 `FAIL`。

### 处理脚本参数
```bash
case "$1" in
	start|stop)
		$1
		;;

	restart|reload)
		stop
		sleep 1
		start
		;;

	*)
		echo "Usage: $0 {start|stop|restart|reload}"
		exit 1
		;;
esac
```
- 脚本通过 `case` 语句根据传入的参数来决定执行哪个操作：
  - `start`：调用 `start` 函数，启动 `syslogd`。
  - `stop`：调用 `stop` 函数，停止 `syslogd`。
  - `restart` 或 `reload`：先停止进程，等待 1 秒后重新启动，确保服务能平滑重启。
  - 如果参数无效，输出使用提示并退出脚本。
{% endfolding %}


{% folding S02klogd%}
**klogd** 会收集内核级别的日志信息，通常与 **syslogd** 配合使用，将内核日志发送到合适的位置。
这个 `S02klogd` 脚本是一个 **SysV-style** 启动脚本，专门用于管理 `klogd` 进程（Kernel Logging Daemon）。它遵循传统的 init 脚本格式，使用 `start-stop-daemon` 命令来启动和停止 `klogd`，并且能够创建 `PIDFILE` 以便于进程管理。

---

## **详细解析**
这个脚本的主要作用是控制 `klogd`（Kernel Log Daemon）的启动、停止和重启，并且遵循 SysV init 脚本的标准格式。它的结构清晰，主要由以下几个部分组成：

### **1. 变量定义**
```sh
DAEMON="klogd"
PIDFILE="/var/run/$DAEMON.pid"
KLOGD_ARGS=""
```
- `DAEMON="klogd"`：定义守护进程的名称。
- `PIDFILE="/var/run/$DAEMON.pid"`：定义进程的 PID 文件路径。
- `KLOGD_ARGS=""`：定义 `klogd` 运行时的附加参数，默认是空的。

### **2. 读取默认配置**
```sh
[ -r "/etc/default/$DAEMON" ] && . "/etc/default/$DAEMON"
```
- 这里会检查 `/etc/default/klogd` 文件是否可读，如果存在，则加载它的内容。
- 这个设计使得 `klogd` 的启动参数可以在 `/etc/default/klogd` 里定义，而不是硬编码在脚本中，增强了灵活性。

### **3. `start()` 函数**
```sh
start() {
	printf 'Starting %s: ' "$DAEMON"
	start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "/sbin/$DAEMON" \
		-- -n $KLOGD_ARGS
	status=$?
	if [ "$status" -eq 0 ]; then
		echo "OK"
	else
		echo "FAIL"
	fi
	return "$status"
}
```
- 先打印 `"Starting klogd: "` 提示信息。
- 使用 `start-stop-daemon` 启动 `klogd`：
  - `-b`：后台运行（daemonize）。
  - `-m`：创建 `PIDFILE`。
  - `-S`：启动服务（start）。
  - `-q`：安静模式（quiet），不打印额外信息。
  - `-p "$PIDFILE"`：指定 `PIDFILE`。
  - `-x "/sbin/$DAEMON"`：执行 `/sbin/klogd`。
  - `-- -n $KLOGD_ARGS`：传递 `-n` 选项给 `klogd`，表示不创建 `PIDFILE`，因为 `BusyBox` 版 `klogd` 不会自己管理 `PIDFILE`，所以这里用 `start-stop-daemon` 处理。
- `status=$?` 获取 `start-stop-daemon` 的退出状态码：
  - `0` 代表成功，打印 `"OK"`。
  - 非 `0` 代表失败，打印 `"FAIL"`。
- `return "$status"` 返回状态码，以便于外部脚本检查。

### **4. `stop()` 函数**
```sh
stop() {
	printf 'Stopping %s: ' "$DAEMON"
	start-stop-daemon -K -q -p "$PIDFILE"
	status=$?
	if [ "$status" -eq 0 ]; then
		rm -f "$PIDFILE"
		echo "OK"
	else
		echo "FAIL"
	fi
	return "$status"
}
```
- 先打印 `"Stopping klogd: "` 提示信息。
- `start-stop-daemon -K -q -p "$PIDFILE"`：
  - `-K` 选项用于停止（kill）进程。
  - `-q` 选项是安静模式（quiet）。
  - `-p "$PIDFILE"` 选项指定 `PIDFILE`，以便找到并杀死 `klogd` 进程。
- 如果进程成功终止：
  - 删除 `PIDFILE`。
  - 打印 `"OK"`。
- 如果失败，打印 `"FAIL"` 并返回错误码。

### **5. `restart()` 函数**
```sh
restart() {
	stop
	sleep 1
	start
}
```
- 先调用 `stop()` 终止进程。
- `sleep 1` 等待 1 秒，确保进程完全退出。
- 重新调用 `start()` 启动 `klogd`。

### **6. 命令行参数解析**
```sh
case "$1" in
	start|stop|restart)
		"$1";;
	reload)
		restart;;
	*)
		echo "Usage: $0 {start|stop|restart|reload}"
		exit 1
esac
```
- 允许 `start|stop|restart` 作为参数，直接调用相应的函数。
- `reload` 其实是 `restart`，因为 `klogd` 本身没有 `reload` 机制，所以用 `restart` 代替。
- 其他情况打印用法说明，并返回 `exit 1` 表示参数错误。

---

{% endfolding %}


{% folding S02sysctl %}

**sysctl** 用于配置和调整内核参数。这个脚本会设置一些内核的运行时参数，比如文件系统、内存、网络等方面的配置。


---
### **脚本概述**
这个 `S02sysctl` 脚本用于管理 `sysctl` 配置文件的加载。它读取多个配置文件，并将其应用到内核参数中。它还根据是否存在 `logger` 命令来决定是将输出发送到系统日志中，还是直接输出到标准输出。

### **详细结构**

1. **变量定义**
    ```sh
    PROGRAM="sysctl"
    SYSCTL_ARGS=""
    ```
    - `PROGRAM`：定义要运行的程序，这里是 `sysctl`。
    - `SYSCTL_ARGS`：用于传递给 `sysctl` 的附加参数，默认为空。

2. **加载配置**
    ```sh
    [ -r "/etc/default/$PROGRAM" ] && . "/etc/default/$PROGRAM"
    ```
    - 通过检查 `/etc/default/sysctl` 文件是否可读，来加载该文件中的配置。

3. **配置文件源定义**
    ```sh
    SYSCTL_SOURCES="/etc/sysctl.d/ /usr/local/lib/sysctl.d/ /usr/lib/sysctl.d/ /lib/sysctl.d/ /etc/sysctl.conf"
    ```
    - 这是一个包含多个路径的列表，`sysctl` 配置文件会在这些路径中查找，并按顺序加载。

4. **日志功能：`run_logger()` 和 `run_std()`**
    - `run_logger()`：如果 `logger` 可用，将 `sysctl` 的输出发送到系统日志。
    - `run_std()`：如果 `logger` 不可用，将输出直接发送到标准输出或错误输出。

5. **启动函数 `start()`**
    ```sh
    start() {
        printf '%s %s: ' "$1" "$PROGRAM"
        status=$("$run_program" 4>&1)
        echo "$status"
        if [ "$status" = "OK" ]; then
            return 0
        fi
        return 1
    }
    ```
    - 根据配置执行 `sysctl`，并根据 `status` 输出结果。

6. **命令行参数处理**
    ```sh
    case "$1" in
        start)
            start "Running";;
        restart|reload)
            start "Rerunning";;
        stop)
            :;;
        *)
            echo "Usage: $0 {start|stop|restart|reload}"
            exit 1
    esac
    ```
    - 提供了 `start|stop|restart|reload` 的参数，执行相应的操作。对于 `stop`，没有实际操作，`:` 是一个空操作。




{% endfolding %}

{% folding S30customizer %}
---

### 功能概述
1. **脚本入口**
   - 使用 `case "$1"` 判断传入参数（如 `start` 或 `stop`）。
   - 通常由 `/etc/init.d/` 框架调用，传入参数为 `start`。

2. **主要功能**
   - 设置系统时间。
   - 执行自定义化脚本（`customizer.sh`）。
   - 配置无线网络（`wireless.sh`）。
   - 配置多路复用器和 GPIO（`muxes.sh`）。
   - 检查 MAC 地址。

{% endfolding %}


{% folding S35modules %}
负责加载内核模块，这里没有用到。
{% endfolding %}

{% folding S38mdev %}
**mdev** 是一个轻量级的设备管理工具，类似于 **udev**，它会管理和创建设备节点，确保设备被正确识别和配置，由于系统没有使用设备接口，没有使用。
{% endfolding %}

{% folding S40network %} 

管理网络接口的启动和停止，包括有线网络和无线网络。通过读取 U-Boot 环境变量来动态配置网络设备，并根据设备类型调用不同的初始化逻辑。
---

### **脚本内容**

#### **1. 读取 U-Boot 环境变量**
```bash
dev=$(fw_printenv -n wlandev)
mac=$(fw_printenv -n wlanmac)
net=$(fw_printenv -n netaddr_fallback)
```
- **`fw_printenv`**：
  - 这是一个工具，用于读取 U-Boot 的环境变量。
  - `-n` 参数表示只输出变量值，而不包含变量名。
- **变量含义**：
  - **`dev` (`wlandev`)**：
    - 表示无线网络设备的类型或标识符（如 USB、SDIO 或 Modem）。
  - **`mac` (`wlanmac`)**：
    - 表示无线网卡的 MAC 地址。
  - **`net` (`netaddr_fallback`)**：
    - 表示默认的网络地址（如 IP 地址），当没有其他配置时使用。
    - 默认值为 `192.168.2.10`。

---

#### **2. 配置无线网络接口**
```bash
set_wireless() {
	path=/etc/wireless
	if $path/usb "$dev" || $path/sdio "$dev"; then
		[ -n "$mac" ] && ip link set dev wlan0 address "$mac"
		ifup wlan0
	elif $path/modem "$dev"; then
		ifup usb0
		ifup eth1
	fi
	[ -e /sys/class/net/eth0 ] && ifconfig eth0 "${net:-192.168.2.10}"
}
```
- **功能**：
  - 根据 `dev` 的值判断无线设备的类型，并执行相应的初始化逻辑。
- **逻辑分支**：
  1. **USB 或 SDIO 设备**：
     ```bash
     if $path/usb "$dev" || $path/sdio "$dev"; then
         [ -n "$mac" ] && ip link set dev wlan0 address "$mac"
         ifup wlan0
     ```
     - 检查 `/etc/wireless/usb` 或 `/etc/wireless/sdio` 脚本是否支持当前设备。
     - 如果支持：
       - 设置无线网卡 `wlan0` 的 MAC 地址（如果有 `wlanmac`）。
       - 启动无线接口 `wlan0`。
  2. **Modem 设备**：
     ```bash
     elif $path/modem "$dev"; then
         ifup usb0
         ifup eth1
     ```
     - 如果设备是 Modem 类型，则启动 `usb0` 和 `eth1` 接口。
  3. **回退配置**：
     ```bash
     [ -e /sys/class/net/eth0 ] && ifconfig eth0 "${net:-192.168.2.10}"
     ```
     - 如果存在 `eth0` 接口，则为其分配一个默认 IP 地址（`netaddr_fallback` 或 `192.168.2.10`）。

---

#### **3. 启动网络服务**
```bash
start() {
	echo "Starting network..."
	ifup lo
	if [ -n "$dev" ]; then
		set_wireless
	else
		ifup eth0
	fi
}
```
- **功能**：
  - 启动网络服务，按以下顺序：
    1. 启动本地回环接口 `lo`。
    2. 如果存在无线设备（`dev` 不为空），调用 `set_wireless` 函数配置无线网络。
    3. 如果没有无线设备，直接启动有线接口 `eth0`。

---

#### **4. 停止网络服务**
```bash
stop() {
	echo "Stopping network..."
	ifdown lo
	ifdown -f wlan0
	ifdown -f usb0
	ifdown -f eth1
	ifdown -f eth0
}
```
- **功能**：
  - 停止所有网络接口，包括：
    - 本地回环接口 `lo`。
    - 无线接口 `wlan0`。
    - USB 网络接口 `usb0`。
    - 以太网接口 `eth1` 和 `eth0`。
  - `-f` 参数强制关闭接口，即使接口不存在也不会报错。

---

#### **5. 脚本入口**
```bash
case "$1" in
	start|stop)
		$1
		;;

	restart|reload)
		stop
		start
		;;

	*)
		echo "Usage: $0 {start|stop|restart|reload}"
		exit 1
		;;
esac
```
- **功能**：
  - 根据传入参数执行相应的操作：
    - `start`：启动网络服务。
    - `stop`：停止网络服务。
    - `restart` 或 `reload`：先停止再启动网络服务。
    - 默认：打印用法提示并退出。

---

### **总结**

#### **核心功能**
1. **动态配置网络接口**：
   - 根据 U-Boot 环境变量（`wlandev`、`wlanmac`、`netaddr_fallback`）动态配置网络设备。
   - 支持多种无线设备类型（USB、SDIO、Modem）。
2. **启动和停止网络服务**：
   - 启动时按需配置无线或有线网络接口。
   - 停止时关闭所有网络接口。

#### **运行流程**
1. **启动流程**：
   - 启动本地回环接口 `lo`。
   - 如果存在无线设备，调用 `set_wireless` 配置无线网络。
   - 如果没有无线设备，直接启动有线接口 `eth0`。
2. **停止流程**：
   - 关闭所有网络接口。

{% endfolding %}

{% folding S49ntpd %}
网络时间协议守护进程，同步系统时间，校准系统的时钟。
###  `S49ntpd` 脚本

这个脚本的主要功能是管理 NTP 守护进程（`ntpd`）的启动、停止和重启。它通过 `start-stop-daemon` 工具来控制守护进程的生命周期，并使用 PID 文件来跟踪进程状态。

---

### **脚本内容**

#### **1. 变量定义**
```bash
DAEMON="ntpd"
PIDFILE="/var/run/$DAEMON.pid"
DAEMON_ARGS="-n"
```
- **`DAEMON`**：
  - 表示要管理的守护进程名称，这里是 `ntpd`（网络时间协议守护进程）。
- **`PIDFILE`**：
  - 存储 `ntpd` 进程的 PID 文件路径，用于跟踪进程状态。
- **`DAEMON_ARGS`**：
  - 传递给 `ntpd` 的启动参数：
    - `-n`：表示以非后台模式运行（但实际会通过 `start-stop-daemon` 后台化）。

---

#### **2. 启动函数**
```bash
start() {
	echo -n "Starting $DAEMON: "
	start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "$DAEMON" -- $DAEMON_ARGS
	if [ $? -eq 0 ]; then
		echo "OK"
	else
		echo "FAIL"
	fi
}
```
- **功能**：
  - 启动 `ntpd` 守护进程。
- **关键命令**：
  ```bash
  start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "$DAEMON" -- $DAEMON_ARGS
  ```
  - `-b`：以后台模式运行进程。
  - `-m`：创建 PID 文件。
  - `-S`：启动进程。
  - `-q`：静默模式，不输出额外信息。
  - `-p "$PIDFILE"`：指定 PID 文件路径。
  - `-x "$DAEMON"`：指定要启动的可执行文件。
  - `-- $DAEMON_ARGS`：传递给守护进程的参数。
- **错误处理**：
  - 检查 `start-stop-daemon` 的返回值：
    - 如果成功（返回值为 0），打印 `OK`。
    - 如果失败，打印 `FAIL`。

---

#### **3. 停止函数**
```bash
stop() {
	echo -n "Stopping $DAEMON: "
	start-stop-daemon -K -q -p "$PIDFILE"
	if [ $? -eq 0 ]; then
		rm -f "$PIDFILE"
		echo "OK"
	else
		echo "FAIL"
	fi
}
```
- **功能**：
  - 停止 `ntpd` 守护进程。
- **关键命令**：
  ```bash
  start-stop-daemon -K -q -p "$PIDFILE"
  ```
  - `-K`：发送信号终止进程。
  - `-q`：静默模式。
  - `-p "$PIDFILE"`：根据 PID 文件找到目标进程并终止。
- **清理工作**：
  - 如果成功停止进程，则删除 PID 文件。
- **错误处理**：
  - 检查 `start-stop-daemon` 的返回值：
    - 如果成功，打印 `OK`。
    - 如果失败，打印 `FAIL`。

---

#### **4. 脚本入口**
```bash
case "$1" in
	start|stop)
		$1
		;;

	restart|reload)
		stop
		sleep 1
		start
		;;

	*)
		echo "Usage: $0 {start|stop|restart|reload}"
		exit 1
		;;
esac
```
- **功能**：
  - 根据传入参数执行相应的操作：
    - `start`：调用 `start` 函数启动 `ntpd`。
    - `stop`：调用 `stop` 函数停止 `ntpd`。
    - `restart` 或 `reload`：先停止再启动 `ntpd`。
    - 默认：打印用法提示并退出。
- **注意**：
  - 在 `restart` 和 `reload` 中，停止后等待 1 秒再启动，避免资源冲突。

---

### **总结**

#### **核心功能**
1. **启动 NTP 守护进程**：
   - 使用 `start-stop-daemon` 后台运行 `ntpd`，并生成 PID 文件。
2. **停止 NTP 守护进程**：
   - 根据 PID 文件终止 `ntpd` 进程，并清理 PID 文件。
3. **支持重启和重载**：
   - 提供 `restart` 和 `reload` 操作，方便重新配置或恢复服务。

#### **运行流程**
1. **启动流程**：
   - 检查是否可以启动 `ntpd`。
   - 使用 `start-stop-daemon` 启动守护进程，并记录 PID。
2. **停止流程**：
   - 根据 PID 文件终止进程，并删除 PID 文件。
3. **重启流程**：
   - 先停止，再启动，确保服务重新加载。
{% endfolding %}

{% folding S50dropbear %}    
**dropbear**，这是一种轻量级的SSH服务器，允许远程访问系统。这个脚本会初始化SSH服务，允许通过SSH连接。
{% endfolding %}

{% folding S60crond %} 
启动 **crond**，即定时任务守护进程。它负责执行预定的定时任务，比如周期性地运行某些脚本或程序。
{% endfolding %}

{% folding S70vendor %} 
这个脚本通常用于执行供应商特定的初始化任务，为了加载厂商的特定驱动、配置或者服务。

###  `S70vendor` 脚本

这个脚本的主要功能是在系统启动时加载特定厂商的模块（可能包括驱动程序或其他硬件相关的初始化逻辑）。它通过调用 `ipcinfo` 工具获取设备的厂商信息，并动态加载与该厂商相关的模块。

---

### **脚本内容**

#### **1. 脚本入口**
```bash
case "$1" in
	start)
		echo "Loading vendor modules..."
		vendor=$(ipcinfo -v)
		load_"$vendor" -i
		;;

	stop)
		;;

	*)
		echo "Usage: $0 {start}"
		exit 1
		;;
esac
```
- **功能**：
  - 根据传入参数执行相应的操作：
    - `start`：加载厂商模块。
    - `stop`：当前为空，表示不支持停止操作。
    - 默认：打印用法提示并退出。

---

#### **2. 加载厂商模块**
```bash
echo "Loading vendor modules..."
vendor=$(ipcinfo -v)
load_"$vendor" -i
```
- **功能**：
  - 使用 `ipcinfo -v` 获取设备的厂商信息。
  - 动态调用与厂商相关的加载函数（如 `load_<vendor>`）。
- **关键命令**：
  1. **`ipcinfo -v`**：
     - 这是一个工具，用于查询设备的硬件或固件信息。
     - `-v` 参数返回设备的厂商名称（如 `sony`、`samsung` 等）。
  2. **`load_"$vendor"`**：
     - 动态构造函数名，例如：
       - 如果 `vendor="sony"`，则调用 `load_sony`。
       - 如果 `vendor="samsung"`，则调用 `load_samsung`。
  3. **`-i` 参数**：
     - 传递给加载函数的参数，可能是初始化选项。

---

#### **3. 停止分支**
```bash
stop)
	;;
```
- **功能**：
  - 当前为空，表示该脚本不支持停止操作。
- **可能原因**：
  - 厂商模块通常是内核模块或硬件驱动，加载后无需显式卸载。
  - 或者，卸载逻辑由其他脚本（如 `rcK` 或其他 `Kxx` 脚本）处理。

---

### **总结**

#### **核心功能**
1. **动态加载厂商模块**：
   - 使用 `ipcinfo` 工具获取设备的厂商信息。
   - 根据厂商信息调用对应的加载函数（如 `load_sony` 或 `load_samsung`）。
2. **支持启动操作**：
   - 脚本仅支持 `start` 操作，用于加载厂商模块。
3. **不支持停止操作**：
   - 当前未实现停止逻辑，可能由其他机制处理。

#### **运行流程**
1. **启动流程**：
   - 打印提示信息：“Loading vendor modules...”。
   - 调用 `ipcinfo -v` 获取厂商名称。
   - 动态调用对应的加载函数（如 `load_<vendor>`），并传递 `-i` 参数。
2. **停止流程**：
   - 当前未实现停止逻辑。

#### **适用场景**
- 该脚本适用于嵌入式设备（如 IP 摄像头）中加载厂商特定的硬件模块或驱动程序。
- 它通过动态调用的方式支持多种厂商，灵活性较高。
  
---

{% endfolding %}

{% folding S95majestic %}
###  `S95majestic` 脚本

这个脚本的主要功能是管理 `majestic` 守护进程的启动、停止、重启和重载。它通过 `start-stop-daemon` 工具来控制守护进程的生命周期，并使用 PID 文件来跟踪进程状态。相比之前的 `ntpd` 脚本，该脚本增加了对 `reload` 操作的支持。

---

### **脚本内容**

#### **1. 变量定义**
```bash
DAEMON="majestic"
PIDFILE="/var/run/$DAEMON.pid"
DAEMON_ARGS="-s"
```
- **`DAEMON`**：
  - 表示要管理的守护进程名称，这里是 `majestic`。
  - `majestic` 是 OpenIPC 项目中的一个核心组件，通常用于处理视频流（如 RTSP、HTTP 等）。
- **`PIDFILE`**：
  - 存储 `majestic` 进程的 PID 文件路径，用于跟踪进程状态。
- **`DAEMON_ARGS`**：
  - 传递给 `majestic` 的启动参数：
    - `-s`：可能是以静默模式或后台模式运行的选项。

---

#### **2. 启动函数**
```bash
start() {
	echo -n "Starting $DAEMON: "
	start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "$DAEMON" -- $DAEMON_ARGS
	if [ $? -eq 0 ]; then
		echo "OK"
	else
		echo "FAIL"
	fi
}
```
- **功能**：
  - 启动 `majestic` 守护进程。
- **关键命令**：
  ```bash
  start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "$DAEMON" -- $DAEMON_ARGS
  ```
  - `-b`：以后台模式运行进程。
  - `-m`：创建 PID 文件。
  - `-S`：启动进程。
  - `-q`：静默模式，不输出额外信息。
  - `-p "$PIDFILE"`：指定 PID 文件路径。
  - `-x "$DAEMON"`：指定要启动的可执行文件。
  - `-- $DAEMON_ARGS`：传递给守护进程的参数。
- **错误处理**：
  - 检查 `start-stop-daemon` 的返回值：
    - 如果成功（返回值为 0），打印 `OK`。
    - 如果失败，打印 `FAIL`。

---

#### **3. 停止函数**
```bash
stop() {
	echo -n "Stopping $DAEMON: "
	start-stop-daemon -K -q -p "$PIDFILE"
	if [ $? -eq 0 ]; then
		rm -f "$PIDFILE"
		echo "OK"
	else
		echo "FAIL"
	fi
}
```
- **功能**：
  - 停止 `majestic` 守护进程。
- **关键命令**：
  ```bash
  start-stop-daemon -K -q -p "$PIDFILE"
  ```
  - `-K`：发送信号终止进程。
  - `-q`：静默模式。
  - `-p "$PIDFILE"`：根据 PID 文件找到目标进程并终止。
- **清理工作**：
  - 如果成功停止进程，则删除 PID 文件。
- **错误处理**：
  - 检查 `start-stop-daemon` 的返回值：
    - 如果成功，打印 `OK`。
    - 如果失败，打印 `FAIL`。

---

#### **4. 重启函数**
```bash
restart)
	stop
	sleep 3
	start
	;;
```
- **功能**：
  - 先调用 `stop` 函数停止 `majestic`。
  - 等待 3 秒（避免资源冲突）。
  - 再调用 `start` 函数重新启动 `majestic`。
- **注意**：
  - 等待时间（`sleep 3`）可以防止频繁操作导致的问题。

---

#### **5. 重载函数**
```bash
reload)
	killall -1 "$DAEMON"
	;;
```
- **功能**：
  - 向 `majestic` 发送 `SIGHUP` 信号（信号编号为 1），触发其重新加载配置。
- **关键命令**：
  ```bash
  killall -1 "$DAEMON"
  ```
  - `-1`：发送 `SIGHUP` 信号。
  - `$DAEMON`：目标进程名称。
- **用途**：
  - `SIGHUP` 通常用于通知守护进程重新读取配置文件，而无需完全重启。

---

#### **6. 脚本入口**
```bash
case "$1" in
	start|stop)
		$1
		;;

	restart)
		stop
		sleep 3
		start
		;;

	reload)
		killall -1 "$DAEMON"
		;;

	*)
		echo "Usage: $0 {start|stop|restart|reload}"
		exit 1
		;;
esac
```
- **功能**：
  - 根据传入参数执行相应的操作：
    - `start`：启动 `majestic`。
    - `stop`：停止 `majestic`。
    - `restart`：先停止再启动 `majestic`。
    - `reload`：向 `majestic` 发送 `SIGHUP` 信号以重载配置。
    - 默认：打印用法提示并退出。

---

### **总结**

#### **核心功能**
1. **启动 `majestic` 守护进程**：
   - 使用 `start-stop-daemon` 后台运行 `majestic`，并生成 PID 文件。
2. **停止 `majestic` 守护进程**：
   - 根据 PID 文件终止进程，并删除 PID 文件。
3. **支持重启和重载**：
   - 提供 `restart` 和 `reload` 操作，方便重新加载配置或恢复服务。
4. **动态配置管理**：
   - `reload` 操作允许在不停止服务的情况下重新加载配置。

#### **运行流程**
1. **启动流程**：
   - 检查是否可以启动 `majestic`。
   - 使用 `start-stop-daemon` 启动守护进程，并记录 PID。
2. **停止流程**：
   - 根据 PID 文件终止进程，并删除 PID 文件。
3. **重启流程**：
   - 先停止，再启动，确保服务重新加载。
4. **重载流程**：
   - 向 `majestic` 发送 `SIGHUP` 信号，触发配置重载。

#### **适用场景**
- 该脚本适用于嵌入式设备（如 IP 摄像头）中管理视频流服务。
- 它通过标准化的方式管理 `majestic` 守护进程，适合资源受限的环境。

---

{% endfolding %}


{% folding S98datalink %}
###  `S98datalink` 脚本

这个脚本的主要功能是管理数据链路服务（如 LTE 模块、ZeroTier 网络和 Wi-Fi 广播）。它根据设备的硬件信息（通过 `ipcinfo` 和 U-Boot 环境变量）以及配置文件 `/etc/datalink.conf` 的内容，动态启动或停止相关服务。

---

### **脚本内容**

#### **1. 变量定义**
```bash
chip=$(ipcinfo -c)
fw=$(grep "BUILD_OPTION" "/etc/os-release" | cut -d= -f2)
```
- **`chip`**：
  - 使用 `ipcinfo -c` 获取设备的芯片型号。
  - 示例是 `ssc338q` 或其他 SoC 名称。
- **`fw`**：
  - 从 `/etc/os-release` 文件中提取 `BUILD_OPTION` 的值。
  - 示例输出可能是 `lte` 或其他构建选项。

---

#### **2. 加载配置文件**
```bash
if [ -e /etc/datalink.conf ]; then
	. /etc/datalink.conf
fi
```
- **功能**：
  - 如果存在 `/etc/datalink.conf` 文件，则加载其内容。
  - `. /etc/datalink.conf` 表示将该文件的内容作为当前脚本的一部分执行。
- **用途**：
  - 配置文件可能包含以下变量：
    - `usb_modem`：是否启用 USB LTE 模块。
    - `use_zt`：是否启用 ZeroTier 网络。
    - `zt_netid`：ZeroTier 网络 ID。
    - `telemetry`：是否启用遥测功能。

---

#### **3. 启动 LTE 数据链路**
```bash
start_lte() {
	echo "Starting fpv datalink..."
	if [ "$usb_modem" = "true" ]; then
		echo "Starting lte modem configuration..."
	fi

	if [ "$use_zt" = "true" ]; then
		echo "Starting ZeroTier-One daemon..."
		zerotier-one -d &
		if [ ! -f "/var/lib/zerotier-one/networks.d/$zt_netid.conf" ]; then
			sleep 8
			zerotier-cli join "$zt_netid" > /dev/null
			echo "Don't forget to authorize my.zerotier.com!"
		fi
	fi

	if [ "$telemetry" = "true" ]; then
		telemetry start
	fi

	exit 0
}
```
- **功能**：
  - 启动 LTE 数据链路相关的服务。
- **逻辑分支**：
  1. **USB LTE 模块**：
     ```bash
     if [ "$usb_modem" = "true" ]; then
         echo "Starting lte modem configuration..."
     fi
     ```
     - 如果 `usb_modem="true"`，表示启用了 USB LTE 模块，并打印提示信息。
  2. **ZeroTier 网络**：
     ```bash
     if [ "$use_zt" = "true" ]; then
         echo "Starting ZeroTier-One daemon..."
         zerotier-one -d &
         if [ ! -f "/var/lib/zerotier-one/networks.d/$zt_netid.conf" ]; then
             sleep 8
             zerotier-cli join "$zt_netid" > /dev/null
             echo "Don't forget to authorize my.zerotier.com!"
         fi
     fi
     ```
     - 如果 `use_zt="true"`，启动 ZeroTier 守护进程（`zerotier-one`）。
     - 如果尚未加入指定的网络（`$zt_netid`），则等待 8 秒后尝试加入，并提醒用户在 ZeroTier 控制台授权设备。
  3. **遥测功能**：
     ```bash
     if [ "$telemetry" = "true" ]; then
         telemetry start
     fi
     ```
     - 如果 `telemetry="true"`，启动遥测服务。

---

#### **4. 启动分支**
```bash
case "$1" in
	start)
		if [ -n "$(fw_printenv -n wlandev)" ]; then
			exit 0
		fi

		if [ ! -f /etc/system.ok ]; then
			tweaksys "$chip"
		fi

		if [ "$fw" = "lte" ]; then
			start_lte
		fi
	
		echo "Starting wifibroadcast service..."
		wifibroadcast start
		;;
```
- **功能**：
  - 根据传入参数执行相应的操作。
- **逻辑分支**：
  1. **检查无线设备**：
     ```bash
     if [ -n "$(fw_printenv -n wlandev)" ]; then
         exit 0
     fi
     ```
     - 如果 U-Boot 环境变量中存在 `wlandev`，直接退出脚本。
     - 这可能是因为无线设备已由其他脚本（如 `S40network`）处理。
  2. **系统初始化**：
     ```bash
     if [ ! -f /etc/system.ok ]; then
         tweaksys "$chip"
     fi
     ```
     - 如果 `/etc/system.ok` 文件不存在，调用 `tweaksys` 函数对系统进行初始化。
     - `tweaksys` 可能是一个自定义函数，用于调整系统配置以适配特定芯片。
  3. **启动 LTE 数据链路**：
     ```bash
     if [ "$fw" = "lte" ]; then
         start_lte
     fi
     ```
     - 如果 `BUILD_OPTION=lte`，调用 `start_lte` 函数启动 LTE 相关服务。
  4. **启动 Wi-Fi 广播**：
     ```bash
     echo "Starting wifibroadcast service..."
     wifibroadcast start
     ```
     - 启动 Wi-Fi 广播服务（`wifibroadcast`）。

---

#### **5. 停止分支**
```bash
stop)
	echo "Stopping wifibroadcast service..."
	wifibroadcast stop
	;;
```
- **功能**：
  - 停止 Wi-Fi 广播服务。

---

#### **6. 默认分支**
```bash
*)
	echo "Usage: $0 {start|stop}"
	exit 1
	;;
```
- **功能**：
  - 如果传入参数无效，打印用法提示并退出。

---

### **总结**

#### **核心功能**
1. **动态加载配置**：
   - 根据 `/etc/datalink.conf` 文件的内容，决定启用哪些服务（如 LTE 模块、ZeroTier 网络、遥测功能）。
2. **启动数据链路服务**：
   - 包括 LTE 模块、ZeroTier 网络、Wi-Fi 广播等。
3. **系统初始化**：
   - 根据芯片型号调用 `tweaksys` 函数进行系统调整。
4. **支持启动和停止操作**：
   - 提供 `start` 和 `stop` 操作，分别用于启动和停止服务。

#### **运行流程**
1. **启动流程**：
   - 检查无线设备是否存在。如果存在，直接退出。
   - 如果系统未初始化，调用 `tweaksys` 函数。
   - 根据 `BUILD_OPTION` 决定是否启动 LTE 数据链路。
   - 启动 Wi-Fi 广播服务。
2. **停止流程**：
   - 停止 Wi-Fi 广播服务。

#### **适用场景**
- 该脚本适用于嵌入式设备（如 IP 摄像头或无人机）中管理数据链路服务。
- 它通过动态加载配置和硬件信息，灵活地支持多种网络和服务。

---

{% endfolding %}

{% folding S98vtun%}
启动 **vtun**，这是一个虚拟隧道工具，通常用于建立加密隧道，这里没有用到
{% endfolding %}

{% folding S99rc.local %}
**rc.local** 是启动过程的最后一个步骤，通常用于执行最后的初始化任务或者自定义命令。系统初始化完成后，这里可以添加需要的启动命令，或者启动一些不属于其他服务的应用。
{% endfolding %}
 
##### 工作流：
1. **系统初始化阶段**：`rcS` 脚本会启动，并执行一些基础的系统配置，包括随机数生成、日志守护进程、内核参数配置等。
2. **服务启动**：之后，系统会依次启动一些基本的服务，如内核模块加载、设备管理、网络配置、时间同步等。
3. **特定应用和服务**：进入更具体的应用服务启动，如SSH服务、定时任务、供应商服务、特定功能应用等。
4. **最后的清理和自定义配置**：`rc.local` 负责执行最后的清理、日志保存、或者启动一些额外的定制化服务。

##### 天空端启动总结
```powershell
/etc/init.d/rcS
 ├──> S30customizer
 │   ├──> /usr/share/openipc/customizer.sh
 │   └──> sh /usr/share/openipc/wireless.sh
 ├──> S98datalink
 │   ├──> tweaksys ssc33x  // configure majestic, h265, 1080p, udp://127.0.0.1:5600
 │   └──> wifibroadcast start
 │       ├──> [video]  // udp_port == 5600
 │       │   └──> wfb_tx -p "$stream" -u "$udp_port" -R "$rcv_buf" -K "$keydir/$unit.key" -B "$bandwidth" \
 │       │           -M "$mcs_index" -S "$stbc" -L "$ldpc" -G "$guard_interval" -k "$fec_k" -n "$fec_n" \
 │       │           -T "$pool_timeout" -i "$link_id" -f "$frame_type" -C 8000 "$wlan" > /dev/null &
 │       └──> telemetry start  // telemetry_rx == wfb_rx, telemetry_tx == wfb_tx, port_rx == 14551, port_tx == 14550
 │           ├──> mavfwd --channels "$channels" --master "$serial" --baudrate "$baud" -p 100 -t -a "$aggregate" \
 │           │           --out 127.0.0.1:$port_tx --in 127.0.0.1:$port_rx > /dev/null &
 │           ├──> telemetry_rx -p "$stream_rx" -u "$port_rx" -K "$keydir/$unit.key" -i "$link_id" "$wlan" > /dev/null &
 │           └──> telemetry_tx -p "$stream_tx" -u "$port_tx" -K "$keydir/$unit.key" -B "$bandwidth" \
 │                             -M "$mcs_index" -S "$stbc" -L "$ldpc" -G "$guard_interval" -k "$fec_k" -n "$fec_n" \
 │                             -T "$pool_timeout" -i "$link_id" -f "$frame_type" "$wlan" > /dev/null &
 └──> S95majestic
```

.bin文件角色：作为固件被内核加载，初始化摄像头硬件。

数据流路径：摄像头 → 内核驱动（通过固件） → /dev/video0节点 → 用户空间应用（通过V4L2接口）。

关键检查点：设备节点存在性、内核日志中的固件加载记录、V4L2工具（如v4l2-ctl）测试。



## 地面站选择
### 虚拟机Ubuntu
### 泰山派Android系统
### 泰山派Ubuntu
### 算力版jetson
### ROS端部署：


```cpp
#include <iostream>
#include <thread>
#include <opencv2/opencv.hpp>
 
 
void startCamera() {
	cv::VideoCapture cap;
	cap.open("clip.mp4");
	while (true) {
		cv::Mat frame;
		//方法一：>>析取器
		cap >> frame;  //每个循环从cap中解析一帧，赋给frame, 
		if (frame.empty()) {
			break;
		}
		//cv::imshow("frame", frame);
		//cv::waitKey(1);
		std::cout<<"frame :"<<frame.cols<<" "<<frame.rows<<std::endl;
	}
	cap.release();
}
 
void startGStream(std::string gst_src) {
	cv::VideoCapture cap;
	// "rtspsrc location=rtsp://stream.strba.sk:1935/strba/VYHLAD_JAZERO.stream latency=4000 ! rtph264depay ! h264parse ! omxh264dec ! nvvidconv !  video/x-raw, width=1280, height=720, format=BGRx ! videoconvert ! appsink"
	// "filesrc location=clip.mp4 ! qtdemux ! h264parse ! omxh264dec ! nvvidconv ! appsink"
	// "v4l2src device=/dev/video0 ! video/x-raw, width=1280, height=720 ! videoconvert ! appsink"
	cap.open(gst_src, cv::CAP_GSTREAMER);
	while (true) {
		cv::Mat frame;
		//方法一：>>析取器
		cap >> frame;  //每个循环从cap中解析一帧，赋给frame, 
		if (frame.empty()) {
			break;
		}
		//cv::imshow("frame", frame);
		//cv::waitKey(1);
		std::cout<<"frame :"<<frame.cols<<" "<<frame.rows<<std::endl;
	}
	cap.release();
}
 
 
// "rtspsrc location=rtsp://stream.strba.sk:1935/strba/VYHLAD_JAZERO.stream latency=4000 ! rtph264depay ! h264parse ! omxh264dec ! nvvidconv !  video/x-raw, width=1280, height=720, format=BGRx ! videoconvert ! appsink"
// "filesrc location=clip.mp4 ! qtdemux ! h264parse ! omxh264dec ! nvvidconv ! appsink"
// "v4l2src device=/dev/video0 ! video/x-raw, width=1280, height=720 ! videoconvert ! appsink"
std::string get_rtsp_h264_gst(std::string rtsp_uri, int width, int height, int latency)
{
	std::string gst_str = "rtspsrc location=" + rtsp_uri+ " latency="+ std::to_string(latency)+ " ! rtph264depay ! h264parse ! omxh264dec ! nvvidconv !  video/x-raw, width="+std::to_string(width)+", height="+std::to_string(height)+", format=BGRx ! videoconvert ! appsink";
	std::cout<<"gst:"<<gst_str<<":"<<std::endl;
	return gst_str;
}
std::string get_rtsp_h265_gst(std::string rtsp_uri, int width, int height, int latency)
{
	std::string gst_str = "rtspsrc location=" + rtsp_uri+ " latency="+ std::to_string(latency)+ " ! rtph265depay ! h265parse ! omxh265dec ! nvvidconv !  video/x-raw, width="+std::to_string(width)+", height="+std::to_string(height)+", format=BGRx ! videoconvert ! appsink";
	std::cout<<"gst:"<<gst_str<<":"<<std::endl;
	return gst_str;
}
std::string get_mp4_h264_gst(std::string file_name, int width, int height)
{
	std::string gst_str = "filesrc location=" + file_name+ " ! qtdemux ! h264parse ! omxh264dec ! nvvidconv !  video/x-raw, width="+std::to_string(width)+", height="+std::to_string(height)+", format=BGRx ! videoconvert ! appsink";
	std::cout<<"gst:"<<gst_str<<":"<<std::endl;
	return gst_str;
}
std::string get_v4l2_gst(std::string device_id, int width, int height)
{
	std::string gst_str = "v4l2src device=" + device_id+ " !  video/x-raw, width="+std::to_string(width)+", height="+std::to_string(height)+", format=BGRx ! videoconvert ! appsink";
	std::cout<<"gst:"<<gst_str<<":"<<std::endl;
	return gst_str;
}
 
 
void startGStream(std::string rtsp_uri, int width, int height, int latency) {
	std::string gst_str = get_rtsp_h264_gst(rtsp_uri, width, height, latency);
start:
	cv::VideoCapture capture;
	capture.open(gst_str, cv::CAP_GSTREAMER);
	while (true) {
		cv::Mat frame;
		//方法一：>>析取器
		capture >> frame;  //每个循环从cap中解析一帧，赋给frame, 
		if (frame.empty()) {
			break;
		}
		//cv::imshow("frame", frame);
		//cv::waitKey(1);
		std::cout<<"frame :"<<frame.cols<<" "<<frame.rows<<std::endl;
	}
 
	capture.release();
	std::cout<<" ...................................................release "<<std::endl;
	goto start;
}
 
 
int main(int argc, char** argv){
 
	//std::string gst_src = "rtspsrc location=rtsp://stream.strba.sk:1935/strba/VYHLAD_JAZERO.stream latency=4000 ! rtph264depay ! h264parse ! omxh264dec ! nvvidconv !  video/x-raw, width=1280, height=720, format=BGRx ! videoconvert ! appsink";
	// if (argc > 1){
	// 	gst_src = argv[1];
	// }
	// startGStream(gst_src);
 
	std::string file_src = "rtsp://stream.strba.sk:1935/strba/VYHLAD_JAZERO.stream";
	int width = 1280;
	int height = 720;
	int latency = 5000;
	if (argc > 4){
		file_src = argv[1];
		width = atoi(argv[2]);
		height = atoi(argv[3]);
		latency = atoi(argv[4]);
	}
	startGStream(file_src, width, height, latency);
	std::cout<<"finished."<<std::endl;
	return 0;
}

```

配合ROS实现

```cpp
#include <ros/ros.h>
#include <sensor_msgs/Image.h>
#include <cv_bridge/cv_bridge.h>
#include <opencv2/opencv.hpp>
#include <image_transport/image_transport.h>
#include <thread>
#include <atomic>

class OpenCVGStreamerNode {
public:
    OpenCVGStreamerNode(ros::NodeHandle& nh) : it(nh), running(true) {
        // 从参数服务器获取 GStreamer 管道配置
        nh.param<std::string>("gstreamer_pipeline", gstreamer_pipeline,
                               "udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H265' ! rtph265depay ! h265parse ! mppvideodec ! videoconvert ! appsink");
        nh.param<std::string>("output_topic", output_topic, "/openipc_camera/image");

        // 发布图像话题
        image_pub = it.advertise(output_topic, 1);

        // 启动图像捕获线程
        capture_thread = std::thread(&OpenCVGStreamerNode::captureImages, this);
    }

    ~OpenCVGStreamerNode() {
        running = false;
        if (capture_thread.joinable()) {
            capture_thread.join();
        }
    }

private:
    void captureImages() {
        // 使用 GStreamer 管道配置初始化 VideoCapture
        cv::VideoCapture cap(gstreamer_pipeline, cv::CAP_GSTREAMER);
        if (!cap.isOpened()) {
            ROS_ERROR("Failed to open video stream using GStreamer pipeline: %s", gstreamer_pipeline.c_str());
            return;
        }

        cv::Mat frame;
        while (running) {
            cap >> frame;  // 从视频流捕获一帧
            if (frame.empty()) {
                ROS_WARN("Received empty frame from video stream");
                continue;
            }

            // 将 OpenCV 图像转换为 ROS 图像消息
            sensor_msgs::ImagePtr msg = cv_bridge::CvImage(std_msgs::Header(), "bgr8", frame).toImageMsg();
            image_pub.publish(msg);
        }
    }

    image_transport::ImageTransport it_;
    image_transport::Publisher image_pub_;
    std::string gstreamer_pipeline;
    std::string output_topic;
    std::thread capture_thread;
    std::atomic<bool> running;
};

int main(int argc, char** argv) {
    ros::init(argc, argv, "opencv_gstreamer_node");
    ros::NodeHandle nh;

    // 创建 OpenCV GStreamer 节点
    OpenCVGStreamerNode opencv_gstreamer_node(nh);
    ros::spin();
    return 0;
}

```

```cpp


#include <ros/ros.h>
#include <sensor_msgs/Image.h>
#include <cv_bridge/cv_bridge.h>
#include <opencv2/opencv.hpp>
#include <image_transport/image_transport.h>
#include <thread>
#include <atomic>

class OpenCVGStreamerNode {
public:
    OpenCVGStreamerNode(ros::NodeHandle& nh) : it(nh), running(true) {
        // 从参数服务器获取 GStreamer 管道配置
        nh.param<std::string>("gstreamer_pipeline", gstreamer_pipeline,
                               "udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H265' ! rtph265depay ! h265parse ! nvv4l2decoder ! nv3dsink -e");
        nh.param<std::string>("output_topic", output_topic, "/openipc_camera/image");

        // 发布图像话题
        image_pub_ = it_.advertise(output_topic, 1);

        // 启动图像捕获线程
        capture_thread = std::thread(&OpenCVGStreamerNode::captureImages, this);
    }

    ~OpenCVGStreamerNode() {
        running = false;
        if (capture_thread.joinable()) {
            capture_thread.join();
        }
    }

private:
    void captureImages() {
        // 使用 GStreamer 管道配置初始化 VideoCapture
        cv::VideoCapture cap(gstreamer_pipeline, cv::CAP_GSTREAMER);
        if (!cap.isOpened()) {
            ROS_ERROR("Failed to open video stream using GStreamer pipeline: %s", gstreamer_pipeline.c_str());
            return;
        }

        cv::Mat frame;
        while (running) {
            cap >> frame;  // 从视频流捕获一帧
            if (frame.empty()) {
                ROS_WARN("Received empty frame from video stream");
                continue;
            }

            // 将 OpenCV 图像转换为 ROS 图像消息
            sensor_msgs::ImagePtr msg = cv_bridge::CvImage(std_msgs::Header(), "bgr8", frame).toImageMsg();
            image_pub_.publish(msg);
        }
    }

    image_transport::ImageTransport it_;
    image_transport::Publisher image_pub_;
    std::string gstreamer_pipeline;
    std::string output_topic;
    std::thread capture_thread;
    std::atomic<bool> running;
};

int main(int argc, char** argv) {
    ros::init(argc, argv, "opencv_gstreamer_node");
    ros::NodeHandle nh;

    // 创建 OpenCV GStreamer 节点
    OpenCVGStreamerNode opencv_gstreamer_node(nh);
    ros::spin();
    return 0;
}
```