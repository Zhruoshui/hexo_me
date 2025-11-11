---
title: AutoCube
description: 闲逛github看到的一个有趣的项目
tags:
  - OpenCV
categories:
  - 好玩的项目
cover: 'https://image.aruoshui.fun/i/2024/12/31/vsjrzz-0.webp'
swiper_index: 2
abbrlink: 37842
date: 2024-03-05 19:32:05
---

{% timeline 跟踪日志,blue %}

<!-- timeline 2024/3/4 -->
按照文档配置完成，后续逐步掌握代码
<!-- endtimeline -->

{% endtimeline %}


# 项目来源
{% link 魔方自动还原, https://github.com/RexWzh/rubik_cube.py?tab=readme-ov-file, http://image.aruoshui.fun/i/2024/12/31/qrh9hd-0.webp %}

# 项目概述
利用opencv进行模版匹配识别，识别结果结合三阶魔方[科先巴二阶段算法](https://github.com/muodov/kociemba)，计算还原步数（根据算法，任何打乱的三阶魔方都可以在21步以内还原完成【不包括扭角捣乱的】，得到还原步数利用Python的[pyautogui](https://github.com/asweigart/pyautogui)库进行自动化执行）