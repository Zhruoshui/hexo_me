---
title: about
date: 2024-03-14 17:34:13
aside: false 
top_img: https://image.aruoshui.fun/i/2024/12/31/vkrcfa-0.webp

---

<table>
    <tr style="height: 50px;">
        <td style="font-size: 1.8em;"><strong>张恒👨‍💻</strong></td>
        <td style="font-size: 1.5em;"><strong>求职意向：嵌入式软件开发工程师</strong></td>
        <td rowspan="5">
            <img src="https://image.aruoshui.fun/i/2026/01/09/igh7j6-0.webp" height="200" alt="">
        </td>
    </tr>
    <tr>
        <td>政治面貌：中共党员</td>
        <td>电话：18183863998</td>
    </tr>
    <tr>
        <td>出生年月：2002年12月</td>
        <td>邮箱：Aruoshui_Zh@outlook.com</td>
    </tr>
    <tr>
        <td>现住址：北京市丰台区</td>
        <td>学历：统招一本</td>
    </tr>
    <tr>
        <td colspan="2">博客：https://blog.aruoshui.fun/</td>
    </tr>
</table>


<hr/>

### 🏆校园经历
<h4 style="display: flex;justify-content: space-between;">
<span>2021-09 ~ 2025-07</span><span>北京信息科技大学</span><span>计算机科学与技术</span>
</h4>

- 绩点**3.69/4.0**，年级前**15%**
- 担任人工智能社团副社长、AI_LAB无人机群体感知组成员
- 多次获得学校学习优秀奖学金和科技创新奖学金
- 十六届国际先进机器人及仿真技术大赛，人工智能物流挑战赛国赛一等奖[2023年]
- 一作身份发表一篇EI会议论文[2023年]


### 😎专业技能
熟悉 C/C++ 嵌入式开发，使用 Python 进行自动化脚本编写及算法验证
具备 MCU 和 SOC 开发基础，理解 Bootloader 的原理
了解 ARM 架构及 ARM 汇编，使用过 ARM 架构处理器 M、A系列
熟悉 FreeRTOS 和 μc/OS Ⅱ ，运用队列、信号量、互斥锁等开发实际任务
能够配置常见的通信协议，例如：UART，I2C，SPI。具备各类常见传感器及模块的开发能力
熟悉 Linux 开发，了解 Linux 驱动层开发，具有设备树移植及内核裁剪的经验
理解 Linux 启动过程，能够参与内核调试，能够使用QEMU进行片上测试及开发
具备深度学习理论及 Pytorch 深度学习模型的搭建的基础，具备嵌入式 AI 领域应用开发能力
掌握Git、GCC、CMake、Docker、Shell脚本等常见开发工具的使用
有一定硬件基础，能看懂基本原理图，简单的3d建模



<hr/>

### 💻项目经历
<h4 style="display: flex;justify-content: space-between;">
<span>2024-8 ~ 2024-12</span><span>基于OpenIPC的数字图传系统开发</span><span>延伸项目</span>
</h4>
适用于无人机FPV图传、工业巡检等需要轻量化低延迟视频传输的领域，通过"
Jetson+OpenIPC"组合，便于扩展AI检测功能，是对无人巡检摄像头采集的一次全新尝试

- 完成嵌入式Linux系统深度定制：基于Buildroot构建OpenIPC系统镜像，SSC338Q芯片搭载系统，系统启动Bootloader
- 双频无线传输：采用WFB-ng协议栈搭建5.8GHz/2.4GHz双频链路（RTL8812au网卡MIMO模式）
- 实现端到端低延迟流水线：集成GStreamer多媒体框架（硬件加速H.264编解码），优化视频采集→编码→传输→解码全链路时延，达成43ms超低延迟，支持1080P@60fps高清视频流


<h4 style="display: flex;justify-content: space-between;">
<span>2023-11 ~ 2024-12</span><span>变电站巡检机器人（实验室横向课题）</span><span>组长</span>
</h4>

实现变电站场景下3D环境感知与路径规划，针对复杂电力设备环境优化BEVDet算法，检测精度提升15%（需补充具体指标），支持实时障碍物分类（含导线悬挂物、绝缘子破损等电力场景特有目标）
- 3D目标检测算法选取与ROS部署。
- 参与模型部署ROS节点的编写及测试，设计多摄像头数据同步机制（六路海康GSM相机RTSP流接入+图像畸变校正），实现巡检机器人环视检测。
- 构建ROS多节点数据管道（含图像预处理、BEV特征融合模块）

<h4 style="display: flex;justify-content: space-between;">
<span>2024-5 ~ 2024-8</span><span>基于GD32的无人机飞控</span><span>组长</span>
</h4>

针对市面无人机飞控系统封闭性强、二次开发困难的问题，社团实验室基于国产GD32F470微控制器设计开源飞控系统，实现飞行姿态控制、稳定悬停及集群通信扩展能力，为后续多机协同研究提供硬件与算法基础
- 任务调度：FreeRTOS多任务调度框架搭建，划分传感器采集（IMU/气压计/GPS）、PID控制、通信等任务优先级
- 传感器及数据： IMU 传感器、气压计、GPS 等传感器获取飞行数据，数据进行飞行控制。
- 数据传输任务：负责创建队列进行数据传输并进行覆盖，通过使用互斥锁防止冲突，并建立临界区确保数据放入时的一致性。
- 数据处理工作：负责处理陀螺仪和加速度计的数据。通过利用偏置值确定方差和重力加速度缩放因子进行校准，结合二阶低通滤波减少噪音，最终保证了输出的实时性，并提高了输出准确率10%。


<h4 style="display: flex;justify-content: space-between;">
<span>2023-03 ~ 2023-11</span><span>基于机器视觉的蘑菇分类识别小程序（国家级大创项目）</span><span>组长</span>
</h4>

针对设计识别野生蘑菇的轻量化视觉模型，实现蘑菇分类检测，部署并开发小程序实现上层应用，相关成果发表于IEEE GCRAIT会议（一作）。
- 基于MobileViTv3设计多模态特征融合架构，结合CNN的局部特征提取能力与Transformer的全局语义建模优势。
- 通过知识蒸馏技术（教师模型：VIT，学生模型：MobileViTv3）压缩模型规模，参数量由5.2M降至2.3M（压缩率55.8%），FLOPs优化至728M，推理速度提升2.1倍，Top-1准确率仅下降1.5%（91.2%→89.7%）。
- 参与部署于Jetson orin nano平台，通过TensorRT量化模型至FP16精度，实现实时推理（32 FPS），CPU/GPU利用率分别稳定在18%/76%，内存占用＜450MB，满足户外无服务器场景需求。


<h4 style="display: flex;justify-content: space-between;">
<span>2023-03 ~ 2023-11</span><span>基于STM32的智能物流机器人</span><span>组长</span>
</h4>
基于微处理器和传感器的小型机器人在模拟的工厂完成货物运送、环境避障、识别货物及机械臂抓取、跟踪指示物并放置货物，最终返回停车充电台的任务。项目获得了国赛一等奖的成绩

- 构建基于有限状态机的任务调度系统，协调路径规划、避障决策、视觉识别等核心功能模块
- 基于OpenMV嵌入式视觉模块，采用YOLOv3-Tiny模型实现货物识别（95%准确率）
- 设计全向麦轮底盘驱动方案，采用角度PID闭环控制实现±2°转向精度
- 通过Ymodem协议进行蓝牙OTA升级STM32固件



<hr/>



<style>
    #write {
        padding: 25px 25px 0px;
    }
    hr {
        margin: 6px;
    }
    li {
        margin: 4px;
    }
    p {
        margin: 4px 13px;
    }
    li p{
        margin: 5px 0;
    }
    h1 {
        margin: 8px 15px;
    }
    h3 {
        margin: 9px;
    }
    h4 {
        margin: 7px;
    }
    figure {
        margin: 7px 0px;
    }
    blockquote {
        padding-left: 16px;
    }
    /* 链接下划线 */
    a {
        text-decoration:underline;
    }
    /* 图片阴影效果 */
    img {
        box-shadow: 0px 0px 10px rgba(0,0,0,.5);
    }
    /* 表格样式，去除边框显示 */
    table, table td, table tr, table th, th {
        font-weight: normal;
        padding: 3px 13px;
        border: 0px;
        background-color: #ffffff;
    }
</style>