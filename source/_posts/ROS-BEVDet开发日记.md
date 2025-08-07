---
title: ROS_BEVDet开发日记
tags:
  - BEV
  - 目标检测
categories:
  - AI
description: 本文用于记录BEVDet在ROS上部署的开发日志
cover: 'https://image.aruoshui.fun/i/2024/12/31/vsl54c-0.webp'
password: bevdet
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
theme: xray
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
swiper_index: 5
abbrlink: 14994
date: 2024-05-27 15:06:24
---

{% timeline 开发跟踪日志,blue %}

<!-- timeline 2024/5/18 -->
于jetson orin nano板子上部署好ROS_BVEDet环境，TensorRT部署
<!-- endtimeline -->

<!-- timeline 2024/5/19 -->
模型Onnx转换，跑通ROS结点，完成推理
 
 ![结果](/img/1.gif)
<!-- endtimeline -->

<!-- timeline 2024/5/24 -->
考试周项目延期
<!-- endtimeline -->

<!-- timeline 2024/5/27 -->
BEVDet源码关于图像接口已经阅读完毕，正在进行摄像头的接入工作（修改之前写好的结点），由于项目的依赖关系，需要另外阅读学习，对其中数据包进行深入学习，其中有：
{% note info flat %}
参考：[BEVDet by TensorRT、C++](https://github.com/LCH1238/bevdet-tensorrt-cpp)
项目特色：
- 结合了调整大小、裁剪和归一化进行预处理的 CUDA 内核
- 预处理 CUDA 内核包括两种插值方法：最近邻插值和双三次插值
- 使用 C++ 和 CUDA 内核实现对齐相邻帧 BEV 特征
{% endnote %}

{% note info flat %}
参考：[BEVDet-ROS-TensorRT工程实现](https://github.com/linClubs/BEVDet-ROS-TensorRT)
项目特色：
- 使用 CUDA、TensorRT、ROS1 和 C++ 进行 BEVDet 在线实时推理的源代码和模型。
{% endnote %}

<!-- endtimeline -->

<!-- timeline 2024/5/28 -->
阅读了bevdet-tensort-cpp-master文件中的bevdet.cpp文件
此文件中主要是各种引擎，参数的初始化以及部分的处理图像
图像处理主要是再两个函数：
    读取相机数据（从文件中读取？！（待定））以及对齐图片，处理矩阵都是函数InitParams
    预处理、特征提取、BEV特征池化、特征对齐（可选）、BEV阶段网络前向传播以及后处理是DoInfer函数
[步骤 1] : 预处理图像，包括调整大小、裁剪和归一化
```cpp
    CHECK_CUDA(cudaMemcpy(src_imgs_dev, cam_data.imgs_dev,
        N_img * src_img_h * src_img_w * 3 * sizeof(uchar), cudaMemcpyDeviceToDevice));

    preprocess(src_imgs_dev, (float*)imgstage_buffer[imgbuffer_map["images"]], N_img, src_img_h, src_img_w,
               input_img_h, input_img_w, resize_radio, resize_radio, crop_h, crop_w, mean, std, pre_sample);

    // 初始化深度信息
    InitDepth(cam_data.param.cams2ego_rot, cam_data.param.cams2ego_trans, cam_data.param.cams_intrin);

```
[步骤 2] : 图像阶段网络前向传递
```cpp
    cudaStream_t stream;
    CHECK_CUDA(cudaStreamCreate(&stream));
    if(!imgstage_context->enqueueV2(imgstage_buffer, stream, nullptr)){
        printf("Image stage forward failing!\n");
    }
```
 [步骤 3] : BEV池化
```cpp
    bev_pool_v2(bevpool_channel, unique_bev_num, bev_h * bev_w,
                (float*)imgstage_buffer[imgbuffer_map["depth"]],
                (float*)imgstage_buffer[imgbuffer_map["images_feat"]],
                ranks_depth_dev, ranks_feat_dev, ranks_bev_dev,
                interval_starts_dev, interval_lengths_dev,
                (float*)bevstage_buffer[bevbuffer_map["BEV_feat"]]
                );
```
[步骤 4] : 对齐 BEV 特征
```cpp
    if(use_adj){
        GetAdjFrameFeature(cam_data.param.scene_token, cam_data.param.ego2global_rot,
                           cam_data.param.ego2global_trans, (float*)bevstage_buffer[bevbuffer_map["BEV_feat"]]);
        // 同步 CUDA 设备
        CHECK_CUDA(cudaDeviceSynchronize());
    }
```
[步骤 5] : BEV阶段网络前向传递
```cpp
    if(!bevstage_context->enqueueV2(bevstage_buffer, stream, nullptr)){
        printf("BEV stage forward failing!\n");
    }
```
[步骤 6] : 后处理
```cpp
    postprocess_ptr->DoPostprocess(bevstage_buffer, out_detections);
```

<!-- endtimeline -->

<!-- timeline 2024/5/29 -->
阅读了`bevdet-tensort-cpp-master`文件中的`nvjpegdecoder.cpp`文件
此文件主要用于图像解码
对`Jpeg`图像进行解码

阅读了`bevdet-tensort-cpp-master`文件中的`cpu_jpegdecoder.cpp`文件
此文件主要用于图像输出
将`Jpeg`压缩图像转化为Reg可是图像

阅读了`bevdet-tensort-cpp-master`文件中的`data.cpp`文件
此文件主要用于获取数据和是数据处理
此文件获取数据是从{%span red,yaml%}文件中获取

完成了对`bevdet-tensort-cpp-master`文件主要代码的阅读

<!-- endtimeline -->

<!-- timeline 2024/5/30 -->
### 开始阅读BEVDet-Ros-TensorRT-main文件：
    此文件是基于bevdet-tensort-cpp-master上与ros的结合，所以此文件的src文件与cpp相同，作用也相同
    所以需要看./test部分

./test/01test.cpp文件中
    此文件主要为测试单个样例
    其中值得关注部分为Egobox2Lidarbox函数
    此函数的目的是
    1. 将盒子的中心从自车坐标系转换到激光雷达坐标系。
    2. 使用激光雷达到自车的旋转矩阵和平移向量对盒子进行旋转和平移变换。
    3. 将变换后的盒子添加到激光雷达坐标系的盒子集合中。

```cpp
void Egobox2Lidarbox(const std::vector<Box>& ego_boxes, 
                     std::vector<Box>& lidar_boxes,
                     const Eigen::Quaternion<float>& lidar2ego_rot,
                     const Eigen::Translation3f& lidar2ego_trans){
    // 遍历所有的自车框 ego_boxes
    for(size_t i = 0; i < ego_boxes.size(); i++){
        // 复制当前自车框
        Box b = ego_boxes[i];
        // 提取当前框的中心点坐标
        Eigen::Vector3f center(b.x, b.y, b.z);
        // 将中心点从自车坐标系转换到激光雷达坐标系
        center -= lidar2ego_trans.translation();
        center = lidar2ego_rot.inverse().matrix() * center;
        // 调整框的旋转角度以匹配激光雷达坐标系
        b.r -= lidar2ego_rot.matrix().eulerAngles(0, 1, 2).z();
        // 更新框的位置到激光雷达坐标系
        b.x = center.x();
        b.y = center.y();
        b.z = center.z();
        // 将转换后的框添加到输出容器 lidar_boxes 中
        lidar_boxes.push_back(b);
    }
}
```
`./test/demo_bevdet.cpp`文件
    此文件是01test的一个demo，可以运行测试一个关键帧

`./test/view.cpp`文件
    此文件是一个视图文件，和demo没有很大区别

重点：
    `./test/view.cpp`文件
        在这个文件中找到了发布订阅消息的部分
         处理来自多个传感器的数据，执行图像处理和目标检测，并将结果发布到 ROS 话题中。
             1. 将接收到的点云数据转换为 PCL 中的点云格式。
             2. 将接收到的图像数据转换为 OpenCV 的图像格式，并存储在一个向量中。
             3. 将图像数据从 CPU 拷贝到 GPU，并进行通道转换。
             4. 进行推理，使用深度学习模型对图像数据进行检测，得到一些边界框。
             5. 将检测到的边界框从车辆坐标系转换到雷达坐标系。
             6. 发布转换后的边界框消息和点云消息到 ROS。 
   
```cpp
void RosNode::callback(const sensor_msgs::PointCloud2ConstPtr& msg_cloud, 
    const sensor_msgs::ImageConstPtr& msg_fl_img,
    const sensor_msgs::ImageConstPtr& msg_f_img,
    const sensor_msgs::ImageConstPtr& msg_fr_img,
    const sensor_msgs::ImageConstPtr& msg_bl_img,
    const sensor_msgs::ImageConstPtr& msg_b_img,
    const sensor_msgs::ImageConstPtr& msg_br_img)
{   
    // 创建一个指向 PCL 点云的指针
    pcl::PointCloud<PointT>::Ptr cloud(new pcl::PointCloud<PointT>);
    
    // 将 ROS 中的点云消息转换为 PCL 点云格式
    pcl::fromROSMsg(*msg_cloud, *cloud);
   
    // 创建用于存储图像的 OpenCV Mat 对象
    cv::Mat img_fl, img_f, img_fr, img_bl, img_b, img_br;
    std::vector<cv::Mat> imgs;

    // 将 ROS 中的图像消息转换为 OpenCV 图像格式
    img_fl = cv_bridge::toCvShare(msg_fl_img, "bgr8")->image;
    img_f  = cv_bridge::toCvShare(msg_f_img, "bgr8")->image;
    img_fr = cv_bridge::toCvShare(msg_fr_img, "bgr8")->image;
    img_bl = cv_bridge::toCvShare(msg_bl_img, "bgr8")->image;
    img_b  = cv_bridge::toCvShare(msg_b_img, "bgr8")->image;
    img_br = cv_bridge::toCvShare(msg_br_img, "bgr8")->image;

    // 将图像存储在一个向量中
    imgs.emplace_back(img_fl);
    imgs.emplace_back(img_f);
    imgs.emplace_back(img_fr);
    imgs.emplace_back(img_bl);
    imgs.emplace_back(img_b);
    imgs.emplace_back(img_br);

    // 将图像数据转换为向量，并存储在 imgs_data 中
    std::vector<std::vector<char>> imgs_data;
    cvImgToArr(imgs, imgs_data);
    
    // 将图像数据从 CPU 拷贝到 GPU 上，并进行通道转换
    decode_cpu(imgs_data, imgs_dev_, img_w_, img_h_);

    // 将 GPU 上的图像数据存储在样本数据中
    sampleData_.imgs_dev = imgs_dev_;

    // 创建一个存储边界框的向量
    std::vector<Box> ego_boxes;
    ego_boxes.clear();
    float time = 0.f;
    
    // 使用深度学习模型对图像数据进行推理，得到一些边界框
    bevdet_->DoInfer(sampleData_, ego_boxes, time);
    
    // 创建一个 ROS BoundingBoxArray 消息指针
    jsk_recognition_msgs::BoundingBoxArrayPtr lidar_boxes(new jsk_recognition_msgs::BoundingBoxArray);
    
    // 清空雷达边界框数组
    lidar_boxes->boxes.clear();
    
    // 将车辆坐标系下的边界框转换到雷达坐标系下
    Egobox2Lidarbox(ego_boxes, lidar_boxes, sampleData_.param.lidar2ego_rot, 
                                            sampleData_.param.lidar2ego_trans);

    // 设置雷达边界框消息的帧 ID 和时间戳
    lidar_boxes->header.frame_id = "map";
    lidar_boxes->header.stamp = ros::Time::now();
    
    // 发布转换后的边界框消息
    pub_boxes_.publish(*lidar_boxes);
    
    // 创建一个新的 ROS 点云消息
    sensor_msgs::PointCloud2 msg_cloud_new;
    pcl::toROSMsg(*cloud, msg_cloud_new);

    // 设置点云消息的帧 ID 和时间戳
    msg_cloud_new.header.frame_id = "map";
    msg_cloud_new.header.stamp = ros::Time::now();
    
    // 发布点云消息
    pub_cloud_.publish(msg_cloud_new);
}

```

<!-- endtimeline -->

<!-- timeline 2024/5/31 -->
### 已经阅读完整体项目代码，已经标记好输入输出
1. 完成新开发板的环境迁移及项目迁移
2. 配合研究生学长调试摄像头
3. 目前实现了opencv读取摄像头
   

#### 还需要解决的问题
1. 读取的摄像头图片尺寸问题，可以调整摄像头焦距或opencv对图像大小进行处理
2. 摄像头结点的标定
   

<!-- endtimeline -->

<!-- timeline 2024/6/3 -->
### 发现周五摄像头处理的问题，经过改进发现解决不了，选择换一条思路进行
- 摄像头不是一般的usb摄像头，直接通过`opencv`读取效果不好，经查询得知拉流使用gstreamer进行拉流 
- 在orin板子上的`camera.sh`脚本也是通过`gstreamer`进行拉流调取的
- 经过网络搜索，成功找到了`gstreamer`的官方文档，并找到`opencv`相关，正在学习之中
- 也找到了一个同样使用`opencv+gstreamer`拉流，并能在`jetson`平台使用的项目
- 同时也找到如何在ROS结点中传输视频流

<!-- endtimeline -->

<!-- timeline 2024/6/4 -->
### 继续对周五遗留问题的处理，尝试利用多线程
- 自己写了一个多线程程序进行读取，但是摄像头出现warning报错，得不到图像数据
- github上找到了其他人写好的项目，还没有来得及尝试
- 尝试修改`cv::VideoCapture::VideoCapture`的参数，加入gst的指令

<!-- endtimeline -->

{% endtimeline %}

# 详细开发过程记录


## 参考文档
{% link BEVDet by TensorRT、C++, https://github.com/LCH1238/bevdet-tensorrt-cpp, https://s2.loli.net/2024/03/05/SyojJXzxWFNIYT8.jpg %} 
{% link BEVDet-ROS-TensorRT工程实现, https://github.com/linClubs/BEVDet-ROS-TensorRT, https://s2.loli.net/2024/03/05/SyojJXzxWFNIYT8.jpg %} 
{% link ROS学习与踩坑记录（持续更新版）, https://blog.csdn.net/weixin_43603658/article/details/130236856, https://s2.loli.net/2024/03/18/HQAVcrZC1NKT3pO.png %} 
{% link 全网最全Linux软件包管理apt-get命令, https://blog.csdn.net/qq_57737603/article/details/131117772, https://s2.loli.net/2024/03/18/HQAVcrZC1NKT3pO.png %} 

## 环境配置
### ROS_BEVDet搭建
#### 拉取源码
```bash
mkdir -p bev_ws/src
cd bev_ws/src
git clone https://github.com/linClubs/BEVDet-ROS-TensorRT.git
```

#### onnx引擎，TensorRT
```bash
# 1 创建一个python环境，并安装相关python库
pip install onnx ruamel.yaml==0.17.32

# 2 进入工程目录
cd BEVDet-ROS-TensorRT

# 3 onnx2engine
python tools/export_engine.py ./ckpts/lt_d.yaml ./ckpts/img_stage_lt_d.onnx ./ckpts/bev_stage_lt_d.onnx --postfix=_lt_d --fp16 true
```
#### ROS功能包
```bash
# 更新软件源列表
sudo apt-get update
# 根据ROS版本选择对应的指令
# ubuntu16.04:kinetic
sudo apt-get install ros-kinetic-jsk-recognition-msgs
sudo apt-get install ros-kinetic-jsk-rviz-plugins
# ubuntu18.04:melodic
sudo apt-get install ros-melodic-jsk-recognition-msgs
sudo apt-get install ros-melodic-jsk-rviz-plugins
# ubuntu20.04:noetic
sudo apt-get install ros-noetic-jsk-recognition-msgs
sudo apt-get install ros-noetic-jsk-rviz-plugins
```

#### 编译运行
```bash
# 1. 编译
cd bev_ws 
catkin make

# 2 工作空间生效
source devel/setup.bash

# 3 运行
roslaunch bevdet bevdet_node.launch

# 4 播放数据集
rosbag play nus.bag

```

## 摄像头opencv+gstreamer拉流

### 参考文档
{% link gstreamer opencv文档, https://gstreamer.freedesktop.org/documentation//opencv/index.html?gi-language=c, https://gstreamer.freedesktop.org/documentation//assets/images/gstreamer-logo.svg %} 
{% link opencv+gstreamer拉流, https://blog.csdn.net/hello_dear_you/article/details/129290712, https://s2.loli.net/2024/03/18/HQAVcrZC1NKT3pO.png %} 
{% link ros_rtsp订阅Image类型topic转换为rtsp视频流, https://blog.csdn.net/qq_42257666/article/details/130559000, https://s2.loli.net/2024/03/18/HQAVcrZC1NKT3pO.png %} 
{% link jetson_gmsl_camera_streamer, https://github.com/stanislavkuskov/jetson_gmsl_camera_streamer, https://s2.loli.net/2024/03/05/SyojJXzxWFNIYT8.jpg %} 
{% link gmsl相机的C++驱动，目前支持ros消息发布, https://github.com/CodeColdCook/gmsl_camera_driver, https://s2.loli.net/2024/03/05/SyojJXzxWFNIYT8.jpg %} 
{% link 读取gmsl接口数据，rosbag录制话题, https://blog.csdn.net/weixin_43492473/article/details/124992400, https://s2.loli.net/2024/03/18/HQAVcrZC1NKT3pO.png %} 


### gstreamer相关函数说明
|函数			|作用													|
| -------------- | ------------------------------ |
|cameracalibrate	|通过使用上游/下游相机将相机指向棋盘图案来执行相机校准undistort|
|cameraundistort	|该元件执行相机校准。校准程序完成后：包含相机校正参数的事件将发送到上游 和下游被相机消耗不失真元素。settings 属性设置为相机校正参数（如 序列化 OpenCV 对象的不透明字符串）。 此属性的值稍后可用于配置 cameraundistort 元素。该元素变为空闲状态，稍后可以重新启动 [TODO]。|
|cvdilate			|放大图像|
|cvequalizehist	|均衡灰度图像的直方图 功能|
|cverode			|侵蚀图像|
|cvlaplace		|将 cvLaplace OpenCV 函数应用于图像(??)|
|cvsmooth		|平滑图像|
|cvtracker			|对视频执行对象跟踪，并将其存储在视频缓冲区元数据中|
|dewarp			|畸变图像|
|disparity			|从两个（序列）校正和对齐的立体图像计算立体视差图|
|edgedetect		|对视频和图像执行精明的边缘检测|
|faceblur			|模糊图像和视频中的人脸|
|facedetect		|对视频和图像执行人脸检测|
|grabcut			|此元素是 OpenCV grabcut 实现的包装器。|
|handdetect		|对视频进行手势检测，通过总线消息和导航事件提供检测到的手部位置，并处理手势事件|
|motioncells		|对视频执行运动检测|
|opencvtextoverlay	|OpenCVtextOverlay 在视频帧顶部呈现文本|
|retinex			|用于彩色图像增强的基本和多尺度 retinex|
|segmentation		|此元素使用多种方法之一创建和更新 fg/bg 模型。|
|skindetect			|视频和图像上的人体皮肤检测|
|templatematch	|对视频和图像进行模板匹配，通过总线消息提供检测到的位置。|


- GrabCut:
 是一个 基于图切技术的图像分割方法。它可以看作是 从某些 FG 和 BG“种子”区域对图像进行细粒度分割的方法。这 OpenCV 的实现遵循文章
- Retinex:
	此元素使用多种方法之一创建和更新 fg/bg 模型。 称为“codebook”的是指遵循 opencv 的 codebook 方法 O'Reilly 书 [1] 在 K. Kim 中描述的算法的实现， T. H. Chalidabhongse、D. Harwood 和 L. Davis [2]。背景SubtractorMOG [3]， 或短片的 MOG，指基于高斯混合的背景/前景 分割算法。OpenCV MOG 实现了 [4] 中描述的算法。 背景SubtractorMOG2 [5]，指的是另一个基于高斯混合的 背景/前景分割算法。OpenCV MOG2 实现了 算法在 [6] 和 [7] 中描述。

[1] 学习 OpenCV：使用 OpenCV 库的计算机视觉，作者：Gary Bradski 和 Adrian Kaehler，由 O'Reilly Media 出版，2008 年 10 月 3 日 [2] “使用 Codebook 模型的实时前景-背景分割”， Real-time Imaging，第 11 卷，第 3 期，第 167-256 页，2005 年 6 月。 [3] http://opencv.itseez.com/modules/video/doc/motion_analysis_and_object_tracking.html#backgroundsubtractormog [4] P. KadewTraKuPong 和 R. Bowden，“改进的自适应背景 用于阴影检测实时跟踪的混合模型“，第 2 期 2001年欧洲先进视频监控系统研讨会 [5] http://opencv.itseez.com/modules/video/doc/motion_analysis_and_object_tracking.html#backgroundsubtractormog2 [6] Z.Zivkovic，“改进的背景自适应高斯混合模型 减法“，国际模式识别会议，英国，2004 年 8 月。 [7] Z.Zivkovic，F. van der Heijden，“高效的自适应密度估计 每个图像像素的背景减法任务“，模式识别 《信函》，第27卷，第7期，第773-780页，2006年。

### 尝试利用多线程进行图像数据传输
尝试多种办法来对摄像头数据进行读取，发现还是不能解决读取连续的问题，搜索后尝试使用多线程来处理这个问题
```cpp
#include <ros/ros.h>
#include <image_transport/image_transport.h>
#include <opencv2/highgui/highgui.hpp>
#include <cv_bridge/cv_bridge.h>
#include <thread> // 引入多线程支持

void imagePublisher(int video_source, image_transport::Publisher pub) {
    cv::VideoCapture cap(video_source);
    if (!cap.isOpened()) {
        ROS_ERROR("无法打开视频设备");
        return;
    }

    cv::Mat frame;
    sensor_msgs::ImagePtr msg;

    while (ros::ok()) {
        cap >> frame;
        if (!frame.empty()) {
            msg = cv_bridge::CvImage(std_msgs::Header(), "bgr8", frame).toImageMsg();
            pub.publish(msg);
            ROS_INFO("发布了一帧图像");
        } else {
            ROS_WARN("捕获的帧为空");
        }

        ros::spinOnce();
    }
}

int main(int argc, char** argv) {
    ros::init(argc, argv, "image_publisher");
    ros::NodeHandle nh;
    image_transport::ImageTransport it(nh);
    image_transport::Publisher pub = it.advertise("camera/image", 1);

    int video_source = 2;

    // 创建一个线程来发布图像
    std::thread image_publish_thread(imagePublisher, video_source, pub);

    ros::spin(); // 主线程继续进行ROS回调处理

    // 等待图像发布线程结束
    image_publish_thread.join();

    return 0;
}
```
将图像发布的功能封装到了一个名为`imagePublisher`的函数中，并通过`std::thread`在主函数中创建了一个新的线程来运行这个函数。这样，图像的发布和ROS回调处理就可以在不同的线程中并行执行，从而提高发送速度。但是由于摄像头报错，暂未实现得出。



### 总结：
gstreamer拉流+opencv大部分是一些关于人体识别的函数，但是一些图像处理的运用是可以使用的。

1. opencv+gstreamer拉流
来源：https://blog.csdn.net/hello_dear_you/article/details/129290712
- 此内容主要用在NVIDIA的dGPU和Jetson平台都可以使用。主要是论证硬件解码更好用，但是重点关注代码内容
- OpenCV的VideoCapture类支持视频文件、图像序列或摄像头作为输入，获取数据。在这小节中，重点介绍如何使用VideoCapture类来拉取RTSP流媒体数据。
- 通过调用如下的VideoCapure构造函数能实现对RTSP流数据的拉取
```C++
cv::VideoCapture::VideoCapture	(	const String & 	filename,
									int 	apiPreference 	
)	
```
其中filename的输入可以是：
1. 视频文件路径；
2. 图像序列，例如图像序列名为image_00.jpg, image_01.jpg, image_02.jpg, 则可以设置为image_%02d.jpg；
3. 视频流的URL，例如：rtsp://admin@password:192.168.170.XXX；
4. streamer的pipeline string，该pipeline可以使用gst-launch-1.0测试可用性；
5. apiPreference指的是Capture API使用何种后端，包括cv::CAP_FFMPEG or cv::CAP_IMAGES or cv::CAP_DSHOW等。


## 问题与解决
## 参考文档
{% link 科学的分析和解决ROS运行中产生的undefined symbol类报错的方法, https://blog.csdn.net/qq_44339029/article/details/137341737, https://s2.loli.net/2024/03/18/HQAVcrZC1NKT3pO.png %} 
{% link rosrun Couldn‘t find executable named below, https://zhuanlan.zhihu.com/p/377429776, https://k.sinaimg.cn/n/sinakd20220810s/261/w650h411/20220810/574c-ec748c90d9828e5c452aca3f611d0df6.png/w700d1q75cms.jpg%}



### 1. ROS功能包（3D边界框）版本依赖
#### 报错说明：
```bash
[ERROR] [1716102452.243813263]: PluginlibFactory: The plugin for class 'jsk_rviz_plugin/BoundingBoxArray' failed to load.  Error: According to the loaded plugin descriptions the cl                ass jsk_rviz_plugin/BoundingBoxArray with base class type rviz::Display does not exist. Declared types are  rviz/AccelStamped rviz/Axes rviz/Camera rviz/DepthCloud rviz/Effort rviz                /FluidPressure rviz/Grid rviz/GridCells rviz/Illuminance rviz/Image rviz/InteractiveMarkers rviz/LaserScan rviz/Map rviz/Marker rviz/MarkerArray rviz/Odometry rviz/Path rviz/PointC                loud rviz/PointCloud2 rviz/PointStamped rviz/Polygon rviz/Pose rviz/PoseArray rviz/PoseWithCovariance rviz/Range rviz/RelativeHumidity rviz/RobotModel rviz/TF rviz/Temperature rviz                /TwistStamped rviz/WrenchStamped rviz_plugin_tutorials/Imu
```
##### 解决方法：
卸载并重新安装`jsk_rviz_plugin/BoundingBoxArray`功能包，但后续遇到：undefined symbol类报错: undefined symbol: _ZN8GMapping 14 sampleGaussianEdm
该问题一般是缺失或者没有找到该功能包所需的某个.so 动态链接库文件。
对符号解析：
```bash
c++filt _ZN8GMapping14sampleGaussianEdm
```
结果是`GMapping::sampleGaussian(double, unsigned long)`
这表明运行报错的原因是系统在寻找 GMapping::sampleGaussian 函数的定义时，未能成功找到
估计问题出现在其他依赖版本低，但这个ROS包依赖版本较高，于是使用`sudo apt-get upgrade`更新所有已经安装的依赖包，问题得以解决 

### 2. [rosrun] Couldn‘t find executable named hello_world_cpp below
#### 报错说明：
catkin_make之后source ./devel/setup.bash
source之后运行节点的时候,ros找不到可执行文件

#### 解决方法
1. 检查Cmakelists.txt.
2. 查看catkin_package()位置.
3. 要注意的是 This function must be called before declaring any targets with add_library() or add_executable().，也就是说Cmakelists.txt.中，要确保配置是在`catkin_package()`之后，否则编译完运行就会出现这个问题



#### 摄像头运行报错
{% note danger flat %}摄像头运行
(my_publisher:59106): GStreamer-CRITICAL **: 20:44:32.821: 
Trying to dispose element pipeline0, but it is in PAUSED instead of the NULL state.
You need to explicitly set elements to the NULL state before
dropping the final reference, to allow them to clean up.
This problem may also be caused by a refcounting bug in the
application or some element.

[ WARN:0] global ../modules/videoio/src/cap_gstreamer.cpp (888) open OpenCV | GStreamer warning: unable to start pipeline

(my_publisher:59106): GStreamer-CRITICAL **: 20:44:32.821: 
Trying to dispose element videoconvert0, but it is in PLAYING instead of the NULL state.
You need to explicitly set elements to the NULL state before
dropping the final reference, to allow them to clean up.
This problem may also be caused by a refcounting bug in the
application or some element.

[ WARN:0] global ../modules/videoio/src/cap_gstreamer.cpp (480) isPipelinePlaying OpenCV | GStreamer warning: GStreamer: pipeline have not been created

(my_publisher:59106): GStreamer-CRITICAL **: 20:44:32.822: 
Trying to dispose element appsink0, but it is in READY instead of the NULL state.
You need to explicitly set elements to the NULL state before
dropping the final reference, to allow them to clean up.
This problem may also be caused by a refcounting bug in the
application or some element.

[ WARN:0] global ../modules/videoio/src/cap_v4l.cpp (887) open VIDEOIO(V4L2:/dev/video2): can't open camera by index
[ERROR] [1717505072.826801829]: ????????????????????????

(my_publisher:59106): GStreamer-CRITICAL **: 20:44:32.829: gst_element_post_message: assertion 'GST_IS_ELEMENT (element)' failed

{% endnote %}

#### 解决问题：
报错原因是因为摄像头打开之后没有关闭，在代码中虽然写了关闭命令，但是没有执行到那一步就直接关闭了，导致的问题，重新启动roscore，rosrun就行







