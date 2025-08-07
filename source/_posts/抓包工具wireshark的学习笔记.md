---
title: 抓包工具wireshark的学习笔记
description: "网络包分析工具"
abbrlink: 26256
date: 2024-03-21 13:41:03
cover: https://image.aruoshui.fun/i/2024/12/31/vsjb7w-0.webp
tags:
 - 抓包
categories:
  - 必学开发技能 
---

{% timeline 跟踪日志,blue %}

<!-- timeline 2024/3/17 -->
wireshark的安装和基础使用
<!-- endtimeline -->

<!-- timeline 2024/3/27 -->
常见协议的抓包分析
<!-- endtimeline -->


{% endtimeline %}

# 参考文章
{% link 网络安全工具——Wireshark抓包工具, https://blog.csdn.net/p36273/article/details/130800459, https://blog.csdn.net/weixin_43603658/article/details/130236856 %}



<div align=center class="aspect-ratio">
    <iframe src="//player.bilibili.com/player.html?aid=524106622&bvid=BV1YM411Y7oU&cid=1000587951&p=1" 
    scrolling="no" 
    border="0" 
    frameborder="no" 
    framespacing="0" 
    high_quality=1
    danmaku=1 
    allowfullscreen="true"> 
    </iframe>
</div>



# Wireshark抓包入门操作
## 常见协议包
[ARP协议](https://blog.csdn.net/weixin_58783105/article/details/134986414?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-134986414-blog-131233662.235%5Ev43%5Epc_blog_bottom_relevance_base7&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-134986414-blog-131233662.235%5Ev43%5Epc_blog_bottom_relevance_base7&utm_relevant_index=2)
[ICMP协议](https://blog.csdn.net/zy_dreamer/article/details/132509931)
[TCP协议](https://zhuanlan.zhihu.com/p/603382668?utm_id=0)
[UDP协议](https://zhuanlan.zhihu.com/p/357080855)
[DNS协议](https://www.zhihu.com/question/23099454/answer/3197458866)
[HTTP协议](https://blog.csdn.net/lfm1010123/article/details/126293176)

## 查看本机要抓包的网络
cmd输入指令ipconfig找到对应的网络

## wireshark的混杂模式
混杂模式概述:混杂模式就是接收所有经过网卡的数据包，包括不是发给本机的包，即不验证MAC地址。普通模式下网卡只接收发给本机的包（包括广播包）传递给上层程序，其它的包一律丢弃。
![](https://s2.loli.net/2024/03/21/nhABoVcsGxSL5If.jpg)


## Wireshark过滤器使用

### wireshark常用过滤条件
1. 常用条件
```bash
‘eq’和’==’ 等同
and 并且
or 或者
‘!’ 和’not’ 取反
```

2. 针对IP地址过滤
```bash
1.源地址：ip.src == 192.168…
2.目的地址：ip.dst == 192.168.xx
3.不看源或目的地址：ip.addr == 192.168.xx
```
3. 针对协议过滤
```bash
1.某种协议的数据包 直接输入协议名字
如：http
2.排除某种协议
not tcp 或者 ！tcp
```
4. 针对端口过滤
```bash
1.捕获某端口的数据包
tcp.port == 80
tcp.srcport == 80
tcp.dstport == 80
2.捕获多端口
udp.port >=2048
```

5. 针对长度和内容过滤
```bash
1.长度过滤
data.len > 0
udp.lenth < 30
http.content_lenth <= 20
2.数据包内容过滤
http.request.uri matches “vipscu”(匹配http请求中含有vipscu字段的请求信息)
```

### 开启混淆模式，抓取接口上使用混杂模式直接进行抓包
![](https://s2.loli.net/2024/03/26/DP2nJ5WewyYztEM.png)

## arp协议分析
ARP（Address Resolution Protocol）协议工作在网络层和数据链路层之间，通常被认为是一个跨两层的协议。简单来说，它就像是一本“翻译词典”，帮助你的电脑将IP地址“翻译”成MAC地址，这样你的电脑才能在网上和其他设备进行通信。

![](https://s2.loli.net/2024/03/26/15GyJuRzl6SXY47.png)

### 抓包分析

#### 数据包格式

![ARP数据包的格式](https://s2.loli.net/2024/03/26/bFTZIovByQpcj18.png)

说明：
{% note info modern %}
1. Hardware Type：表示硬件地址类型，一般为MAC地址。它的值为1表示以太网地址。
2. Protocol Type：表示三层协议地址类型，一般为IP。它的值为0x0800即表示IP地址。它的值与包含IP数据报的以太网数据帧中的类型字段的值相同。
3. Hardware Length和Protocol Length：表示MAC地址和IP地址的长度，单位是字节。值分别是6和4。(6 * 8bit=48bit，4 * 8bit = 32bit)
4. Operation Code：指定了ARP报文的类型，包括ARP Request和ARP Reply。（1为ARP请求，2为ARP应答）
5. Source Hardware Address：指的是发送ARP报文的设备MAC地址（源MAC地址）
6. Source Protocol Address：指的是发送ARP报文的设备IP地址（源IP地址）。
7. Destination Hardware Address：指的是接收者MAC地址，在ARP Request报文中，该字段值为0（目的MAC地址）。
8. Destination Protocol Address：指的是接收者的IP地址（目的MAC地址）。
{% endnote %}

##### ARP Reply 响应包
向上边的图片就是一个ARP Reply 响应包

##### ARP Request 请求包
![ARP Request 请求包](https://s2.loli.net/2024/03/26/KSibOdGtvWDY19N.png)

##### ARP 缓存
当然如果每次都需要这样一个映射显然效率低下，于是便有了ARP缓存。{% emp 来存放IP地址和MAC地址关联信息 %}
发送信息前，查找ARP缓存表，存在对方MAC地址，直接封装成帧。
- 如果不存在，通过发送 ARP Request报文获取对方MAC地址。
- 如果目标在其他网络，源设备会先查找网关MAC地址，将数据发给网关，再转发。
- IP和MAC关系映射关系会放入ARP缓存表一段时间，有效期内都可查到，过了这个有效期会自动删除。

##### ARP的请求、响应、代理、IP冲突
这些点请详见：
{% note info flat %}参考：[ARP的请求、响应、代理、IP冲突](https://blog.csdn.net/weixin_58783105/article/details/134986414?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-134986414-blog-131233662.235%5Ev43%5Epc_blog_bottom_relevance_base7&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-134986414-blog-131233662.235%5Ev43%5Epc_blog_bottom_relevance_base7&utm_relevant_index=2){% endnote %}

这个博主的例子很详细清楚，我也不抄一遍了。


#### ARP存在一个ARP欺骗的问题
简单理解就是使得目标主机接收错误的IP和MAC绑定关系
攻击者可以发送虚假的ARP请求或应答报文，使得目标主机接收错误的IP和MAC绑定关系。那么发送给目标的数据就不再走网关了，而是到攻击者那里。如果攻击者拦截数据包不进行转发的话，本机就会断网。

{% note info flat %}参考：[更多详细信息请见](https://blog.csdn.net/weixin_58783105/article/details/134986414?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-134986414-blog-131233662.235%5Ev43%5Epc_blog_bottom_relevance_base7&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-134986414-blog-131233662.235%5Ev43%5Epc_blog_bottom_relevance_base7&utm_relevant_index=2){% endnote %}



## ICMP协议分析
### ICMP是什么
ICMP 的全称是 Internet Control Message Protocol(互联网控制协议)，它是一种互联网套件，它用于IP 协议中中传递控制信息和错误消息。

ICMP协议的主要功能如下：
1. 发现网络错误：当一个数据包在传输过程中出现错误时，ICMP协议通过向发送方发送错误通知来发现网络错误。
2. 检查网络是否可达：通过发送ICMP ECHO请求并接收ICMP ECHO回复消息，可以确定目标主机是否可达。
3. 发现主机错误：当一个主机无法正常工作时，ICMP协议通过向发送方发送错误通知来发现主机错误。
4. 发送路由信息：ICMP协议可以向其他主机发送路由信息，以帮助它们在网络中找到合适的路由。

#### 过程
![通知示意图](https://s2.loli.net/2024/03/26/lEe51a8wGVSQOHn.jpg)
在这个图中：
{% note info modern %}
主机 A 想要给主机 B 发送一个 IP 数据包，主机 A 发送的数据包经过路由器 1 到达了路由器 2 ，由于路由器 2 不知道主机 B 的 MAC 地址，所以路由器 2 发送了一个 ARP 请求，没有回应，再经过重试时间后再次发送，还没有回应。。。。。。经过多次 ARP 请求后没有得到回应后，路由器 2 就会给主机 A 发送一个 ICMP 消息，告诉其发送的 IP 数据包没有到达主机 B 。
{% endnote %}

#### 抓包分析
经常使用 ICMP 数据包的两个终端程序是 ping 和 traceroute。这里我ping了一下我的网关地址

##### 请求
![](https://s2.loli.net/2024/03/26/qB7OYsbeAnLDKhU.jpg)

##### 响应
![](https://s2.loli.net/2024/03/26/cO5n18fTqbzSELe.jpg)

其中注意type类型：
![通知类型](https://s2.loli.net/2024/03/26/JLHOoxMANk51vgK.jpg)
我上边的两个就属于回送请求和回送应答(类型 8 和 类型 0 )

##### 其他ICMP消息和IPv6的ICMPv6
其他消息和ICMPv6我这不方便测试，想了解的可以查看下边的链接：
{% note info flat %}参考：[更多的信息请你查看](https://blog.csdn.net/zy_dreamer/article/details/132509931){% endnote %}


## TCP
相信学过计网都很清楚整个过程，这里就不详细赘述了，如果想要了解所有的细节，请你移步：
{% note info flat %}参考：[TCP详解](https://zhuanlan.zhihu.com/p/603382668?utm_id=0){% endnote %}

TCP有如下的协议格式需要清楚，在抓包时可以针对分析：
{% note info modern %}
- 16位源端口号： 数据发送方的端口号，表示数据从哪里来
- 16位目标端口号：数据接收方的端口号，表示数据要到哪里去
- 32位序号：每一次通信TCP报文的编号
- 32确认序号：用于对发送方发送的报文的确认，为接收到的报文序号+1
- 4位首部长度：表示TCP报文头部有多少个4字节，因为4位最大表示15，所以TCP报文头最大长度为15*4=60
- 6位标志位：
URG: 紧急指针是否有效
ACK: 确认应答
PSH: 提示接收端应用程序立刻从TCP缓冲区把数据读走
RST: 表示要求对方重新建立连接
SYN: 请求建立连接，我们把携带SYN标识的称为同步报文段
FIN: 通知对方本端要关闭了，我们把携带FIN标识的为结束报文段
- 16位窗口大小：TCP流量控制的一个手段，这里指接收窗口，用于告知发送端本端的TCP缓冲区还能容纳多少字节的数据，这样发送方就可以控制发送的数据量
- 16位校验和：由发送方填充，接收端对TCP报文段执行CRC校验以检查数据是否损坏，这个校验不仅包含TCP头部，也包含数据部分
- 16位紧急指针：一个正的偏移量，它与序号字段相加表示最后一个紧急数据的下一个序号，所以这个字段是紧急指针相对当前序号的偏移，用于发送方向接收方发送紧急数据
- 0-40字节选项数据：存储一些可能需要的额外信息
{% endnote %}

### TCP三次握手过程
![](https://s2.loli.net/2024/03/26/PoaYF6ZewhHqtCT.jpg)
#### 抓包分析
{% note warning flat %}用到TCP的地方太多了，很不容易专门找一个等待建立连接的点，这个时候想到了之前写的SSH的一篇文章：[与SSH的今生今世😅](https://www.aruoshui.fun/posts/12699.html) 于是考虑在连接服务器或者虚拟机的时候进行抓包，果然成功了。{% endnote %}

以下是对ip：192.168.131.138的一台虚拟机连接的抓包 

![](https://s2.loli.net/2024/03/27/di9PtJzYKoaVhC1.png)
其中可以非常明显的看到TCP连接过程中如`SYN`，`ACK`等，其实变相帮助理解TCP协议了，比之前上实验课那个过程好太多了。

#### 连接请求
![连接请求](https://s2.loli.net/2024/03/27/IkuzirSglhtOyA8.png)
可以看到`Src Port`是本机用来监听的端口（随机），而虚拟机中SSH连接开放的端口 `Dst Port` 是`22` 

#### 连接请求确认
![连接请求确认](https://s2.loli.net/2024/03/27/2TDqpjCF8Vsu1UL.png) 

#### 确认
![确认](https://s2.loli.net/2024/03/27/Ro5ByXTtDb31kaC.png)

#### 利用wireshark自带的流量图工具获得TCP连接过程
在统计任务栏中选择流量图，点击之后流类型选择为TCP流，就可以更加清晰的了解整个过程了
![流量图](https://s2.loli.net/2024/03/27/qQ7gTbxuRGLN3fI.png)



### TCP连接断开的四次挥手
同样的断开连接我这里还是利用虚拟机
![](https://s2.loli.net/2024/03/27/SdOQf4YEXqsP8FA.png)
细节就不分析了：
![](https://s2.loli.net/2024/03/27/7T9eDZPdvb4LUIN.png)
![](https://s2.loli.net/2024/03/27/Dn5vGcP28sulzNo.png)

## HTTP协议
### 什么是HTTP协议
HTTP 协议简单来说就是客户端与服务器之间一发一收的模型
![HTTP协议的核心过程](https://s2.loli.net/2024/03/27/POgYUjucWlDAmx1.jpg)
### HTTP协议的抓包分析
这里利用`curl -I baidu.com`访问百度网站，触发HTTP协议
![](https://s2.loli.net/2024/03/27/hoEgIiAmXPdD5x6.png)
#### 建立TCP连接
这里就不过多赘述
#### 请求报文和响应报文
在请求和响应部分：
第一个包是，`我`向`百度`发送了一个「HTTP请求」，请求类型是HEAD
第三个包是，`百度`向`我`发送了一个「HTTP响应」，响应状态码是 200 OK
#### 释放连接
这里也不过多赘述

#### 分析HTTP协议报文
请求报文分为三个部分：请求行、请求头、请求体
响应报文分为四个部分：状态行、响应头、响应空行、响应体

##### 请求报文
![](https://s2.loli.net/2024/03/27/KZon164DcXCtbsN.png)
请求行：包含请求方法、请求URL、HTTP版本
请求头：包含请求的客户端的信息，一行一个请求头
请求体：POST等类型的请求才有请求体，这里没有

点开请求行，看里面的三个字段：
1. Request Method：请求方法，这里的请求方法是HEAD，用来获取报文首部
2. Request URI：请求的URL，因为我们没指定，所以默认是/
3. Request Version：请求的版本，因为用的是HTTP协议，所以这里显示HTTP协议的版本
  
请求头部分：
Host：目标主机
User-Agent：代理，也就是浏览器的类型。我们用的不是浏览器，所以这里显示的是命令curl
Accept：浏览器可接受的MIME类型

##### 响应报文
![](https://s2.loli.net/2024/03/27/4zVGmdrfe7oDPjE.png)
状态行：包含版本和响应状态码、状态信息
响应头：包含响应的服务器的资源信息，一行一个响应头 

点开状态行，可以看到里面有三个字段：
![](https://s2.loli.net/2024/03/27/bpDYtwJyr2Uu3ij.png)
1. Response Version：响应版本，因为使用的是HTTP协议，所以这里显示了HTTP的版本
2. Status Code：响应状态码，这里的 200 表示请求成功。
3. Response Phrase：响应状态码的提示信息

另外响应头：
{% note info modern %}
Date：服务端发送响应报文的时间
Server：服务器和相对应的版本
Last-Modified：请求的对象创建或者最后修改的时间
ETag：对象的标志值，如果对象修改了，这个值也会变，用来判断对象是否改变
Accept-Ranges：支持的范围单位
Content-Length：内容长度
Cache-Control：缓存控制
Expires：这个时间前，可以直接访问缓存副本
Connection：连接类型，Keep-Alive表示这是一个长链接，可以继续用这个连接通信
Content-Type：资源文件类型
{% endnote %}

##### HTTP追踪流
选中HTTP协议的数据包 - 右键 - 【追踪流】-【HTTP追踪流】可以看到请求跟响应的报文
![](https://s2.loli.net/2024/03/27/IxFOwLWZTRcuNUC.png)


## 黑客利用wireshark来获取用户账号密码
详细测试过程请自行搜索，我也在学习过程中
这是我自己找的一个教程：

{% link Wireshark抓取网站用户名密码, https://www.cnblogs.com/thespace/p/12731638.html %}

最后想说的就是要注意互联网安全！