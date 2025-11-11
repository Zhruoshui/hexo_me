---
title: CapsWriter-Offline
description: 一个 PC 端的语音输入、字幕转录工具

tags:
  - 语音识别
categories:
  - 工具
cover: 'https://qnwebstaticstorage.aoscdn.com/lightpdf/wp-content/uploads/2022/03/speech-to-text2-1.jpg.webp'
swiper_index: 1
abbrlink: 55158
date: 2024-03-13 22:39:32

---

{% note info flat %}有时候需要记录一些视频资料，光手打记录累且麻烦，国内有做语音转录很牛的（如：讯飞输入法），但是碍于到🪜后的访问速度。最近发现了一款离线语音输入软件，而且能方便的进行字幕转录，不管是视频学习还是给自己做的视频配音，都很不错{% endnote %}

# 来源
{% link 一个 PC 端的语音输入、字幕转录工具, https://github.com/HaujetZhao/CapsWriter-Offline, https://qnwebstaticstorage.aoscdn.com/lightpdf/wp-content/uploads/2022/03/speech-to-text2-1.jpg.webp %}

# 功能
- 1. 按下键盘上的`大写锁定键`，录音开始，当松开`大写锁定键`时，就会识别你的录音，并将识别结果立刻输出
- 2.  将音视频文件拖动到客户端打开，可以转录生成srt字幕文件

# 详细展示
<div align=center class="aspect-ratio">
    <iframe src="//player.bilibili.com/player.html?aid=623379019&bvid=BV1tt4y1d75s&cid=1399238093&p=1" 
    scrolling="no" 
    border="0" 
    frameborder="no" 
    framespacing="0" 
    high_quality=1
    danmaku=1 
    allowfullscreen="true"> 
    </iframe>
</div>

# 特性
- 完全离线、无限时长、低延迟、高准确率、中英混输、自动阿拉伯数字、自动调整中英间隔
- 热词功能：可以在 `hot-en.txt` `hot-zh.txt` `hot-rule.txt` 中添加三种热词，客户端动态载入
- 日记功能：默认每次录音识别后，识别结果记录在 `年份/月份/日期.md` ，录音文件保存在 `年份/月份/assets`
- 关键词日记：识别结果若以关键词开头，会被记录在 `年份/月份/关键词-日期.md`，关键词在 `keywords.txt` 中定义
- 转录功能：将音视频文件拖动到客户端打开，即可转录生成 `srt` 字幕
- 服务端、客户端分离，可以服务多台客户端
- 编辑 `config.py` ，可以配置服务端地址、快捷键、录音开关……

# 下载:
参照[官方开源库](https://github.com/HaujetZhao/CapsWriter-Offline)即可，也有其他大佬改的GUI：[增加图形界面包](https://github.com/HaujetZhao/CapsWriter-Offline/pull/53)，基于`pyside6`，并添加了语音翻译功能
![alt text](https://github.com/H1DDENADM1N/CapsWriter-Offline/blob/GUI-(PySide6)-and-Portable-(PyStand)/assets/start_server_or_client_in_tray.gif?raw=true)

# 其他
关于工具中的核心，`sherpa-onnx`部署的`Paraformer`模型，细节可以参照以下卡片：

{% link sherpa-onnx, https://k2-fsa.github.io/sherpa/onnx/index.html,https://s2.loli.net/2024/03/05/SyojJXzxWFNIYT8.jpg %}

{% link Paraformer, https://www.modelscope.cn/models/iic/speech_paraformer-large-vad-punc_asr_nat-zh-cn-16k-common-vocab8404-pytorch/summary,https://s2.loli.net/2024/03/05/SyojJXzxWFNIYT8.jpg %}

{% link sherpa-onnx, https://github.com/k2-fsa/sherpa-onnx, https://s2.loli.net/2024/03/05/SyojJXzxWFNIYT8.jpg %}