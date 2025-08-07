---
title: 无线链路Wifibroadcast
abbrlink: 22013
date: 2025-01-14 17:29:35
tags:
description:
categories:
cover: https://image.aruoshui.fun/i/2024/12/31/vqssft-0.webp
swiper_index:
---


# 参考文章
{% link 参考文章, https://zhuanlan.zhihu.com/p/666861211, https://img95.699pic.com/xsj/17/7k/tc.jpg%21/fh/300 %} 




# 远程无线编写

通过无线网卡接收带有 radiotap 头的 802.11 数据包，并进行解析、过滤、FEC（前向纠错）、加密解密等处理。我们可以将其划分为几个模块来分析其功能和设计思路。

---

## 🧩 模块结构概览

| 模块 | 类名 | 主要职责 |
|------|------|----------|
| 接收器 | `Receiver` | 负责打开网卡设备、设置混杂模式、过滤特定数据包、循环读取原始数据包 |
| 聚合器 | `Aggregator` | 负责 FEC 解码、数据包去重、会话密钥管理、丢包恢复等 |
| 数据流 | - | 实际上聚合器还负责将数据流输出到指定的目的地 |

---

## 🔍 Receiver 类详解

### 构造函数：初始化网卡

```cpp
Receiver::Receiver(const char *wlan, int wlan_idx, uint32_t channel_id, BaseAggregator *agg, int rcv_buf_size)
```

- 使用 `pcap_create()` 创建一个 pcap 设备。
- 设置混杂模式（promiscuous mode）、超时时间、非阻塞模式等。
- 检查链路层封装类型是否为 `DLT_IEEE802_11_RADIO`（即包含 radiotap header）。
- 编译 BPF 过滤器，只捕获特定以太网头部格式的数据包：
  
  ```cpp
  ether[0x0a:2]==0x5742 && ether[0x0c:4] == 0x%08x
  ```
  
  这表示只接收 EtherType 为 `0x5742` 并且特定字段匹配 `channel_id` 的数据包。

- 将 pcap 设置为非阻塞模式，并获取可轮询的文件描述符用于后续事件监听。

### 析构函数：释放资源

```cpp
Receiver::~Receiver()
{
    close(fd);
    pcap_close(ppcap);
}
```

### 核心方法：接收并处理数据包

```cpp
void Receiver::loop_iter(void)
```

该函数每次调用都会尝试从 pcap 中读取一个数据包，然后：

1. 使用 `ieee80211_radiotap_iterator` 解析 radiotap 头部，提取以下信息：
   - 天线编号（antenna）
   - 频率（freq）
   - RSSI（信号强度）
   - 噪声（noise）
   - MCS 索引（调制编码策略）
   - 带宽（bandwidth）
   - 是否自注入包（self_injected）

2. 对 FCS 和错误帧进行检查与处理。

3. 移除 radiotap 头部后，将数据体传给 Aggregator 的 `process_packet()` 方法。

---

## 🧠 Aggregator 类详解

### 构造函数：初始化 FEC、会话密钥等

```cpp
Aggregator::Aggregator(const string &keypair, uint64_t epoch, uint32_t channel_id)
```

- 加载公私钥对（用于解密接收到的数据包）。
- 初始化 FEC 参数（k/n），以及 FEC 缓冲区环（ring buffer）。
- 初始化统计计数器（如丢包、纠错成功等）。

### FEC 相关方法

#### 初始化 FEC

```cpp
void Aggregator::init_fec(int k, int n)
```

- 分配内存空间，初始化 FEC 编解码器（使用外部库，如 `libfec`）。
- 清空 FEC 缓冲区环。

#### 释放 FEC 资源

```cpp
void Aggregator::deinit_fec()
```

- 释放所有 FEC 缓存空间，关闭 FEC 引擎。

### 核心逻辑：数据包处理

```cpp
void Aggregator::process_packet(...)
```

虽然你没有提供这部分完整代码，但根据上下文可以推断它完成了以下任务：

1. **解密**：使用 libsodium 库对接收到的数据包进行解密。
2. **FEC 解码**：
   - 将数据包加入 FEC 缓冲环中。
   - 当收集到足够数量的 fragment 后尝试恢复丢失的 packet。
3. **数据重组**：
   - 如果 FEC 成功恢复数据，则重新组装成完整数据块。
   - 若失败则标记为丢包或错误包。
4. **输出数据**：
   - 将恢复后的数据发送到下一个阶段（例如视频流播放、写入文件、转发等）。

---

## 🛡️ 安全与加密机制

- 使用了 **NaCl/libsodium** 提供的安全通信接口：
  - `crypto_box_SECRETKEYBYTES`
  - `crypto_box_PUBLICKEYBYTES`
  - 用于对传输数据进行加密和解密，确保数据在空中传输时的安全性。

---

## ⚙️ FEC 前向纠错机制

- 使用了 Reed-Solomon 或类似 FEC 编码方式（由 `libfec` 提供）。
- 允许接收端在部分数据包丢失的情况下仍能恢复原始数据。
- 在无人机图传、远程控制等场景下非常重要，可以容忍一定程度的网络丢包。

---

## 📊 统计与调试信息

- 包含大量统计数据：
  - `count_p_all`: 总共收到的包数
  - `count_p_dec_err`: 解密错误的包数
  - `count_p_lost`: 丢失的包数
  - `count_p_fec_recovered`: FEC 恢复成功的包数
- 可用于性能监控、丢包率评估、QoS 控制等。

---

## 📌 总结：整个系统的工作流程

1. **启动 Receiver**：
   - 打开指定网卡设备，设置混杂模式、BPF 过滤器。
2. **持续接收数据包**：
   - 每次调用 `loop_iter()` 获取一批数据包。
   - 解析 radiotap header，提取元数据（天线、频率、RSSI、MCS 等）。
3. **交给 Aggregator 处理**：
   - 解密 → FEC 解码 → 数据重组 → 输出到目的地。
4. **统计与反馈**：
   - 记录各种状态，可用于前端显示或自动调整 FEC 参数等。

---

## ✅ 可优化点 / 注意事项

- **性能瓶颈**：频繁的内存分配/释放（尤其在 FEC 缓冲区中），建议使用对象池或预分配机制。
- **多线程支持**：当前代码是单线程处理，若需高性能可考虑引入多线程模型。
- **FEC 动态调整**：可根据实时信道质量动态调整 FEC 的 k/n 比例，提升带宽利用率。
- **错误处理增强**：目前很多异常直接抛出 runtime_error，建议更详细的日志记录或回调机制。

---

如果你还有其他部分的代码需要解析（比如 `process_packet` 函数、FEC 实现细节、或者发射端 wfb-tx 的代码），欢迎继续提供！