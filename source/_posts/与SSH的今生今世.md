---
title: "与SSH的今生今世😅"
description: "就是因为他我服务器被黑😅"
tags:
  - 端口
  - 网络传输协议
  - 加密
  - 网络安全
categories:
  - 开发技能
cover: 'https://image.aruoshui.fun/i/2024/12/31/vsj31n-0.webp'
abbrlink: 12699
date: 2024-03-14 20:43:03
---

{% timeline 跟踪日志,blue %}

<!-- timeline 2024/3/14 -->
学习SSH原理
<!-- endtimeline -->

<!-- timeline 2024/3/15 -->
SSH操作及SSH的免密登录
<!-- endtimeline -->


<!-- timeline 2024/3/20 -->
TCP_wapper的原理使用
<!-- endtimeline -->

{% endtimeline %}

# 参考文章
{% link 一口气把 SSH 原理搞懂了, https://zhuanlan.zhihu.com/p/666861211, https://img95.699pic.com/xsj/17/7k/tc.jpg%21/fh/300 %}
{% link SSH协议握手核心过程, https://www.bilibili.com/video/BV13P4y1o76u/?spm_id_from=333.337.search-card.all.click&vd_source=6718fb46dbdd603565f483b04b4cdb50, https://s2.loli.net/2024/03/18/L4b2k8WlnsZzF3O.jpg %}
{% link 万字详解SSH, https://blog.csdn.net/weixin_53946852/article/details/125754556, https://s2.loli.net/2024/03/18/HQAVcrZC1NKT3pO.png %}


<div align=center class="aspect-ratio">
    <iframe src="//player.bilibili.com/player.html?aid=900637560&bvid=BV13P4y1o76u&cid=835703824&p=1" 
    scrolling="no" 
    border="0" 
    frameborder="no" 
    framespacing="0" 
    high_quality=1
    danmaku=1 
    allowfullscreen="true"> 
    </iframe>
</div>

# 我与SSH的今生今世

{% timeline ,blue %}

<!-- timeline 第一次用 -->
第一次用是学习Linux的时候，VMTools有时候老挂，几行的命令又懒得手敲，于是用到了xshell来远程连接Linux机，当时仅仅以为是一个连接虚拟机的工具，没有仔细研究。
<!-- endtimeline -->

<!-- timeline 方便的工具 -->
在大创的时候，嵌入式部署模型，买了一块英伟达的Jetson Orin nano嵌入式开发板，当时使用远程连接终端MobaXterm来连接开发板（虽然有图形界面，但是还是更喜欢Windows敲代码），软件里内置了很多远程连接工具（SSH、X11、RDP、VNC、FTP、MOSH 等），也学习了很多SSH的命令。 
<!-- endtimeline -->

<!-- timeline 留下隐患 -->
- 到后边用Linux系统越来越多，每次我都会配置好SSH，用工具连接好，但是由于每次为了连接方便，好记住，忽略了一个重大问题，就是配置的密码很简单(主要是为了好记住)，用了这么久也没出过什么问题，所以也就不以为然了。
- 直到我自己租了一台服务器，用来部署网站和搭服务，一贯操作培好了SSH，密码依旧是那简单的123456😅，依旧是用着看似没问题，但是这个安全隐患已经深深埋入了。
<!-- endtimeline -->

<!-- timeline 爆出问题 -->
就在2月27日，突然腾讯云发出告警，我寻思我一个破服务器还能被入侵？经过排查，查到了系统的登录日志，毫不意外，除了我的IP，还藏着几个漂亮国IP，看了各种博客，终于找到了被入侵的原因，就是SSH，破译这样的脑瘫密码根本不用费功夫，分分钟就攻破了，跟离谱的就是我刚配置完SSH，过了三天他们就已经登上了我的服务器。
<!-- endtimeline -->

{% endtimeline %}
我与SSH就是这么个故事，希望写下这篇博客也能让自己记住这个遭遇。总的来说，还是安全意识不够，理论永远代替不了实际，尽管常常被灌输安全知识，甚至有点不耐烦，但是说到底，可乘之机还是自己创造的。



# SSH原理
## 什么是SSH
SSH是一种网络协议，基于非对称加密，用于计算机之间的加密登录。最早的时候，互联网通信都是明文通信，一旦被截获，内容就暴露无疑。1995年，芬兰学者Tatu Ylonen设计了SSH协议，将登录信息全部加密，成为互联网安全的一个基本解决方案，迅速在全世界获得推广，目前已经成为Linux系统的标准配置。
## 加密技术

### 对称加密算法（DES）
采用{% emp 单钥密码系统 %}的加密方法，同一个密钥可以同时用作信息的加密和解密，需要对加密和解密使用相同密钥的加密算法，其实整个过程跟房东租房一样，只有两把钥匙开门，租客一把，房东一把，交换信息都需要这个密钥。
![对称加密的过程](https://s2.loli.net/2024/03/18/ODbykA4d6ZSIBms.png)
### 非对称加密（RSA）
- 非对称密钥不是一个加密密钥，而是由两个元素（即私钥和公钥）组成，这两个元素组成了密钥对。 
- 公钥，顾名思义，可以与任何人共享，因此个人和组织无需担心其安全分发问题。私钥必须妥善保管。 它仅由生成密钥对的人管理，不与任何人共享。 {% emp 需要加密消息的用户将使用公钥，但只有持有私钥的人才能对其进行解密 %}。
![非对称加密](https://learn.microsoft.com/zh-cn/training/wwl-sci/describe-concepts-of-cryptography/media/key-pair-generation.png)
当 Quincy 想要向 Monica 发送安全消息时，他使用她的公钥来加密纯文本并创建已加密文本。 然后，Quincy 用他喜欢的任意方式将已加密文本发送给 Monica。 当 Monica 收到已加密文本时，她使用自己的私钥对该文本进行解密，从而将其恢复为纯文本。
![加密及解密过程](https://learn.microsoft.com/zh-cn/training/wwl-sci/describe-concepts-of-cryptography/media/asymmetric-encryption-process.png)
### 对称加密与非对称加密区别
为什么非对称加密更加安全，但是还是需要对称加密呢？接下来看看两者的区别：
{% checkbox plus green checked, 对称加密，成本是比较低(机器资源消耗少)，速度也是很快的 %}
{% checkbox minus yellow checked, 非对称加密，成本比对称加密高很多(机器资源消耗的多)，速度也慢 %}
{% checkbox plus blue checked, 对称加密使用同一个密钥进行加密和解密，密钥本身也在网络上文明传输，也容易被黑客获取 %}
{% checkbox minus red checked, 非对称加密，加密使用公钥，解密使用私钥。更加安全 %}
### 信息安全性的安全措施
但是对称加密和非对称加密都有一个同样的问题，怎么安全的将密钥发给对方，又不会被中间人知道具体密钥？这时候就需要`Diffie Hellman`密钥交换

#### DH算法用于交换密钥
交换密钥的目的是生成仅双方共享的密钥(共享的秘密)
#### 交换密钥的基本过程
- 双方确定公开的内容
- 用各自的私钥分别对公共内容加密（加密本质就是数学运算）并发送给对方
- 这时双方使用自己的密钥对收到的内容加密（要设计运算保证最后结果相同，也就是两步运算的顺序是可以调换的），双方就得到了共同的结果（作为公共密钥）
- 这样就实现了安全的将密钥传递给对方的目的
- 由于私钥没有被传递所以监听者无法得到最终的公共密钥
![过程](https://s2.loli.net/2024/03/18/gkCxK19MdoXs6eJ.jpg)
过程中的参数`P`,`G`,以及公式都是公开的，两边运算之后就可以用这个公共秘钥进行对自己的秘钥的加密。 
其实在过程中，可以理解到，黑客由于不知道双方的随机数（各自保留的），所以面临的是这样的问题
这里为什么黑客无法破解这个公共秘钥呢？
![](https://s2.loli.net/2024/03/18/E3OhcJGlN6aoKzT.jpg)
核心其实在于：过程是一个离散对数问题 {% emp 正向运算简单、逆向困难 %}
黑客除了双方的随机数不知道，其他都可以获取到，其实需要破解的是`?`的值是多少
小一些的数破解起来容易，但是数一旦大了，逆向破解起来就很困难。
#### 中间人篡改问题

如果黑客劫持了数据，发现自己解不开，又不想放过，就加上自己的秘钥分别发送给双方，成为隐形的中间人，还是会互相干扰。
{% note info flat %}参考：[什么是哈希（Hsah）算法，哈希算法的作用以及Java中常见的哈希算法的使用案例。](https://blog.csdn.net/2301_77852117/article/details/131643540){% endnote %}
为了解决这个问题，需要使用哈希算法比对哈希值可以确认信息是否篡改，但哈希值也可被篡改，这里就涉及到了SSH精髓的地方

## SSH协议握手过程
为了完全理解这个不分部分，还需要进行实验，使用抓包工具`wireshark`来进行测试。
### TCP和版本信息握手
![](https://s2.loli.net/2024/03/18/Mq2TFsyjLvYEeck.jpg)
因为SSH1和SSH2两个协议互不兼容，加密方式也不相同，所以要对协议版本进行握手认证。

### 密钥交换初始化 KEXINIT
- 临时秘钥是用来后续生成共享秘钥使用的
- 服务端生成安全秘钥，只要客户端也有这个安全秘钥，加密后信息就不容易被破解了，但是为了保证客户端也能有一模一样的安全秘钥，且服务端不能把自己的安全秘钥发送过去，这个时候就要使用到了前面提到的DH算法，SSH这里使用的是加强版的DH算法
- 服务端把自己的临时公钥发送给客户端，就能生成相同的公共秘钥了
![密钥交换初始化过程](https://s2.loli.net/2024/03/18/Fmbp3MUTk2ZAGva.jpg)

### 防止中间人
避免中间人篡改，得到与中间人一样的共享安全密钥，就要使用到哈希来证明信息没有篡改

### ECDH秘钥交换初始化和ECDH秘钥交换回复

![](https://s2.loli.net/2024/03/18/2wAfBva8GTdVOHI.jpg)
服务端用自己的Host私钥加密了交换哈希值，中间人不知道服务端私钥，如果用其他私钥进行加密，所得到的哈希值就不同

### 可恶的中间人
SSH最危险的就是首次连接，如果服务端密钥指纹没有经过确认就信任，就有可能出现中间人攻击，反过来，如果首次连接即确认了服务端身份，那么后续只要没有警告，那么与服务端的链接就是安全的。 

其实就跟你首次建立SSH连接，会提示你如下：
![](https://s2.loli.net/2024/03/18/SYO3cTUpbXisRde.png)
如果之后你输入登录密码，如果这个时候被监听了，那么结果还是寄了。 

{% checkbox green checked, 关于这个情况，你登录的时间是随机的，一般中间人不会一直蹲守你，所以相比较还是很安全的，除非你一直连续登录，中间人找到了你的登录习惯😂 %}

# SSH的优点
1. 安全性： 数据传输是加密的，可以防止信息泄漏。
2. 身份验证：防止未经授权的用户访问远程系统。
3. 远程管理：可通过SSH协议登录远程服务器并执行命令，无需直接物理访问设备。
4. 端口转发：SSH支持 端口转发功能，可以安全地传输其他协议和应用程序。
5. 传输速度: 数据传输是压缩的，可以提高传输速度。


# SSH基本用法
{% tabs test1 %}
<!-- tab 方式一-->
```bash
ssh -p 22 user@host
```
参数：
-p：指定端口号。
user：登录的用户名。
host：登录的主机。

由于默认端口是22，用这个默认端口号的时候，可以省略，直接用以下形式：
```bash
ssh user@host
```

此外，如果本地正在使用的用户名与远程登录的用户名一致，登录用户名也是可以省略的，即如下：
```bash
ssh host
```
<!-- endtab -->

<!-- tab 方式二：跳板连接 -->
- 跳板连接用于在{% emp 不直接暴露目标主机 %}的情况下进行安全访问。
- 通过跳板连接，用户可以首先连接到中间设备，然后再通过中间设备连接到目标主机。
- 中间设备不一定是跳板机，只需要安装ssh服务就可以。
![](https://s2.loli.net/2024/03/20/Y15IDPaF3N7GZmJ.jpg)

```bash
ssh -t IP1  ssh -t IP2.... ssh -t 目标IP
# IP1和IP2为跳板机的IP地址，先跳转到IP1，再跳转到IP2
#两次跳转成功后，才能远程连接到目标设备

#举例#
ssh -t 192.168.2.102 ssh -t 192.168.2.103 ssh -t 192.168.2.74 
#跳转两次，从当前设备远程连接IP地址为192.168.2.74的主机或者服务器
```

<!-- endtab -->


<!-- tab 方式三：远程控制-->
```bash
ssh 目标设备的IP地址 命令
#远程控制目标主机使用命令 并将命令执行结果返回本机

#主机B的IP地址为xx 
[root@A ~] ssh xx ls #查看主机B家目录下有哪些目录或文件 
```

<!-- endtab -->
{% endtabs %}


# SSH免密登录

## 原理
在SSH免密登录过程中，客户端和服务器之间通过密钥对进行身份验证，而不是使用传统的密码验证方式。SSH会自动使用密钥对进行验证，而无需输入密码。

## 步骤
下面在两个ubuntu系统上有演示：
客户端：192.168.131.138
服务端：192.168.131.139
### 在客户端生成密钥文件
```bash
ssh-keygen   #生成密钥文件 

-t #指定加密方式 
#不加此选项，默认使用rsa方式

Enter file in which to save the key(/root/.ssh/id_rsa): 直接回车
#选择密钥文件存放的位置                 （默认路径）

Enter passphrase (empty for no passphrase): 
#对密钥文件进行加密,设置密码后，访问文件需要输入密码

#一般不输入密码 直接回车
Enter same passphrase again: 

```


最后会生成两个文件 
```bash
cd /root/.ssh
ls 
```

.pub为密钥文件

## 将公钥复制到SSH服务器上
```bash
ssh-copy-id -i 公钥文件  [用户名]@IP地址 
#将密钥文件传过去
#下次就可以免密登录
#以root用户登录时 可省略
```

## 通过SSH连接尝试登录到服务器
成功免密登录服务器192.168.131.139
![](https://s2.loli.net/2024/03/21/1RYNrC2lGnJSUqF.png)



# TCP_wapper的原理和运用
## TCP_wapper的原理

TCP_Wrappers是一个工作在第四层(传输层)的的安全工具，对有状态连接 (TCP)的特定服务进行安全检测并实现访问控制，界定方式是凡是调用libwrap.so库文件的的程序就可以受TCP Wrappers的安全控制。它的主要功能就是控制谁可以访问，常见的程序有rpcbind、vsftpd、sshd，telnet。

TCP_Wrappers有一个TCP的守护进程叫作tcpd。 
以ssh为例，每当有ssh的连接请求时，tcpd即会截获请求，先读取系统管理员所设置的访问控制文件，符合要求，则会把这次连接原封不动的转给真正的ssh进程，由ssh完成后续工作；如果这次连接发起的ip不符合访问控制文件中的设置，则会中断连接请求，拒绝提供ssh服务。

![图示](https://s2.loli.net/2024/03/21/i7FdP1rcKf8NWwu.png)

## 设置黑白名单

![](https://s2.loli.net/2024/03/21/QVElYwjmWHFnOi5.png)

/etc/hosts.allow 设置允许访问 tcp 服务程序的策略（白名单）
/etc/hosts.deny 设置禁止访问 tcp 服务程序的策略 （黑名单）

拒绝单个 IP 使用 ssh 远程连接:
配置文件:
```bash
hosts.allow:空着
hosts.deny: sshd:192.168.88.20
```

拒绝某一网段使用 ssh 远程连接:
```bash
hosts.allow:空着
hosts.deny: sshd:192.168.88

```

仅允许某一IP 使用 ssh 远程连接
```bash
hosts.allow: sshd:192.168.88.20
hosts deny: sshd:ALL
```
