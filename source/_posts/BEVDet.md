---
title: BEVDet
description: 本文用于记录BEVDet学习过程中出现的问题和相关的解决过程
tags:
  - BEV
  - 目标检测
categories:
  - AI
abbrlink: 1976
cover: 'https://image.aruoshui.fun/i/2024/12/31/vuewo5-0.webp'
swiper_index: 3 #置顶轮播图顺序，非负整数，数字越大越靠前
date: 2024-02-26 13:44:30
---
# 1.环境配置
{% note info flat %}
参考：[官方文档](https://github.com/HuangJunJie2017/BEVDet)
参考：[BEVDet复现](https://blog.csdn.net/h904798869/article/details/133363474)
{% endnote %}

## 1. 基本配置
需求：ubuntu20.04，python-3.8, torch1.13.0, cuda-11.7, cudnn-8.6, conda 
[未安装可以参考](https://blog.csdn.net/h904798869/article/details/131719404)

## 2. 基本依赖
```bash 
apt-get install -y vim libsm6 libxext6 libxrender-dev libgl1-mesa-glx git wget libssl-dev libopencv-dev libspdlog-dev
```

## 3. 拉取源码
```bash
# 1 下载源码   目前默认是dev3.1
git clone https://github.com/HuangJunJie2017/BEVDet.git

# 2 查看代码版本号 显示dev3.1版本
git checkout

# 3 如果要切换版本号，如dev2.1，输入
git checkout dev2.1

```
## 4. 配置bevdet的python虚拟环境
```bash
# 1 创建虚拟环境
conda create -n bevdet python=3.8

# 2 激活python环境
conda activate bevdet

# 3 虚拟环境bevdet中安装torch
pip install torch==1.13.0+cu117 torchvision==0.14.0+cu117 torchaudio==0.13.0 --extra-index-url https://download.pytorch.org/whl/cu117

# 4 虚拟环境bevdet中安装openlib相关库
pip install mmcv-full==1.5.3 onnxruntime-gpu==1.8.1 mmdet==2.25.1 mmsegmentation==0.25.0

# 5 进入BEVDet工程目录,安装mmdet3d
pip install -e -v .

# 6 安装其他依赖 numpy==1.23.4 setuptools==58.2.0等
pip install pycuda lyft_dataset_sdk networkx==2.2 numba==0.53.0 numpy==1.23.4 nuscenes-devkit plyfile scikit-image tensorboard trimesh==2.35.39 setuptools==58.2.0 yapf==0.40.1
 
```


# 2. 解决问题
## 2.1 pip装包编译错误
{% note info flat %}
参考：[https://blog.csdn.net/h904798869/article/details/131719404](https://blog.csdn.net/h904798869/article/details/131719404) 
参考：[https://blog.csdn.net/weixin_62075168/article/details/129758163](https://blog.csdn.net/weixin_62075168/article/details/129758163)

{% endnote %}

#### 问题：
```bash
    src/cpp/cuda.hpp:14:10: fatal error: cuda.h: 没有那个文件或目录
       #include <cuda.h>
                ^~~~~~~~
      compilation terminated.
      /tmp/pip-build-env-lze7ntp3/overlay/lib/python3.8/site-packages/setuptools/command/build_py.py:207: _Warning: Package 'pycuda.cuda' is absent from the `pa                                                                              ckages` configuration.
      !!

              ********************************************************************************
              ############################
              # Package would be ignored #
              ############################
              Python recognizes 'pycuda.cuda' as an importable package[^1],
              but it is absent from setuptools' `packages` configuration.

              This leads to an ambiguous overall configuration. If you want to distribute this
              package, please make sure that 'pycuda.cuda' is explicitly added
              to the `packages` configuration field.

              Alternatively, you can also rely on setuptools' discovery methods
              (for example by using `find_namespace_packages(...)`/`find_namespace:`
              instead of `find_packages(...)`/`find:`).

              You can read more about "package discovery" on setuptools documentation page:

              - https://setuptools.pypa.io/en/latest/userguide/package_discovery.html

              If you don't want 'pycuda.cuda' to be distributed and are
              already explicitly excluding 'pycuda.cuda' via
              `find_namespace_packages(...)/find_namespace` or `find_packages(...)/find`,
              you can try to use `exclude_package_data`, or `include-package-data=False` in
              combination with a more fine grained `package-data` configuration.

              You can read more about "package data files" on setuptools documentation page:

              - https://setuptools.pypa.io/en/latest/userguide/datafiles.html


              [^1]: For Python, any directory (with suitable naming) can be imported,
                    even if it does not contain any `.py` files.
                    On the other hand, currently there is no concept of package data
                    directory, all directories are treated like packages.
              ********************************************************************************

      !!
        check.warn(importable)
      error: command '/usr/bin/gcc' failed with exit code 1
      [end of output]

  note: This error originates from a subprocess, and is likely not a problem with pip.
  ERROR: Failed building wheel for pycuda
Failed to build pycuda
ERROR: Could not build wheels for pycuda, which is required to install pyproject.toml-based projects
```

#### 解决思路：
`required to install pyproject.toml-based projects`这个问题主要出现在C++构建工具未安装 
{% tabs 分栏 %}

<!-- tab Windows平台解决思路 -->
在Windows平台安装比较繁琐，首先要下载[Vs](https://go.microsoft.com/fwlink/?linkid=836911),一般来说可以直接安装Visual Studio，但是占用空间大，只安装构建也能解决问题 

在其中标题栏选择下载一栏，并在下载的搜索框键入：build tools，并回车进入下载界面  

下载如图的工具，下载完成以后是一个ISO结尾的光盘映射文件，注意下载的时候将类型改为DVD![](https://s2.loli.net/2024/02/29/IjWZexPbcrX4ziv.png)  

双击运行该光盘映射文件，点击exe文件，一路进行安装 

![](https://s2.loli.net/2024/02/29/TiVSqzGMhrpuelQ.png)  

按图片勾选的工具进行安装，即可解决

<!-- endtab -->

<!-- tab Linux平台解决思路 -->
下载依赖
```bash
sudo apt install g++ make
```
<!-- endtab -->
{% endtabs %}

解决完问题接下来就继续`pip`安装即可。


## 2.2 MMCV工具安装
#### 问题：
```bash
(bevdet) root@wahaha-System-Product-Name:/media/wahaha/222/BEVDet/bev/BEVDet2/BEVDet# python tools/create_data_bevdet.py                                   /root/miniconda3/envs/bevdet/lib/python3.8/site-packages/mmcv/__init__.py:20: UserWarning: On January 1, 2023, MMCV will release v2.0.0, in which it will remove components related to the training process and add a data transformation module. In addition, it will rename the package names mmcv to mmcv-lite and mmcv-full to mmcv. See https://github.com/open-mmlab/mmcv/blob/master/docs/en/compatibility.md for more details.
  warnings.warn(
Traceback (most recent call last):
  File "tools/create_data_bevdet.py", line 9, in <module>
    from tools.data_converter import nuscenes_converter as nuscenes_converter
  File "/media/wahaha/222/BEVDet/bev/BEVDet2/BEVDet/tools/data_converter/nuscenes_converter.py", line 14, in <module>
    from mmdet3d.core.bbox import points_cam2img
  File "/media/wahaha/222/BEVDet/bev/BEVDet2/BEVDet/mmdet3d/__init__.py", line 4, in <module>
    import mmdet
  File "/root/miniconda3/envs/bevdet/lib/python3.8/site-packages/mmdet/__init__.py", line 24, in <module>
    assert (mmcv_version >= digit_version(mmcv_minimum_version)
AssertionError: MMCV==1.7.0 is used but incompatible. Please install mmcv>=1.3.17, <=1.6.0.
```
在运行脚本创建数据集时，按照官方配置安装的MMCV工具提示版本不兼容，需要安装其他版本的，但是安装完所推荐的版本，又会提示不兼容，安装其他的版本，反复横跳 

#### 解决思路
自己修改MMCV工具版本需求范围
打开MMCV工具目录，找到`__init__.py`文件，找到如下的[范围](https://s2.loli.net/2024/02/29/JMU9wfo3mdEgVHx.png) 
把这三个文件的`__init__.py`文件中的MMCV_MIN,MMCV_MAX改成包含你现在拥有的版本，忽略版本范围
![如图](https://s2.loli.net/2024/02/29/PTVHlDAIMZWr4aG.png)
![如图](https://s2.loli.net/2024/02/29/PBhF3rvkVtjAWlS.png)
再次运行脚本，发现脚本运行成功
![](https://s2.loli.net/2024/02/29/HfIUOBLZX6YmWAp.png)
自动生成了训练集与验证集

## 2.3 其他问题···



# 3.数据集的创建
{% note info flat %}
参考：[https://blog.csdn.net/h904798869/article/details/133363474](https://blog.csdn.net/h904798869/article/details/133363474) 
参考：[https://blog.csdn.net/h904798869/article/details/130317240?spm=1001.2014.3001.5502](https://blog.csdn.net/h904798869/article/details/130317240?spm=1001.2014.3001.5502)
{% endnote %}

## 3.1 下载数据集
[nuscenes下载地址](https://www.nuscenes.org/nuscenes#download)
下载细节参考 [FastBEV](https://blog.csdn.net/h904798869/article/details/130317240?spm=1001.2014.3001.5502)

由于我们使用的mini数据集进行测试，而源码中需要full数据集，将v1.0-mini复制一份并命名为v1.0-trainval``./data
文件结构如下：
```bash
data
  └──nuscenes
    ├── gts
    ├── maps
    ├── samples
    ├── sweeps
    ├── v1.0-mini
    └── v1.0-trainval  # 复制的文件夹

```
## 3.2 生成bevdet数据集
运行tools/create_data_bevdet.py生成数据集

```python
python tools/create_data_bevdet.py
```

结果如图：
![](https://s2.loli.net/2024/02/29/HfIUOBLZX6YmWAp.png)

# 4.测试
首先通过测试来实现论文效果，直接使用官方训练好的权重，[详情请见](https://github.com/HuangJunJie2017/BEVDet/blob/dev2.1/README.md)，[官方权重](https://pan.baidu.com/s/1237QyV18zvRJ1pU3YzRItw?pwd=npe1#list/path=%2F)
```python
python tools/test.py ./configs/bevdet/bevdet-r50.py ckpts/bevdet-r50.pth --eval mAP
``` 


# 5.可视化
## 5.1 生成json
```bash
# 运行test.py 必须--out", "--eval", "--format-only", "--show" or "--show-dir至少跟一个
# json文件生成需要增加 --eval-options参数 jsonfile_prefix=test_dirs
# 实在搞不清楚请看test.py的源码，看如何加载参数即可

# 1 直接测试
python tools/test.py ./configs/bevdet/bevdet-r50.py ckpts/bevdet-r50.pth --format-only

# 2 测试保存json文件
python tools/test.py ./configs/bevdet/bevdet-r50.py ckpts/bevdet-r50.pth --format-only --eval-options jsonfile_prefix=test_dirs
# 保留json位于目录test_dirs下

# 3 直接生成保存为pkl格式
python tools/test.py ./configs/bevdet/bevdet-r50.py ckpts/bevdet-r50.pth --out=./test_dirs/out.pkl

```

## 5.2 json文件转可视化
```python
python tools/analysis_tools/vis.py ./work_dir/results_nusc.json
```
生成视频./vis/vis.mp4文件, 如下：
<video
src="/img/WeChat_20240229225803.mp4" controls=""
height=400 
width=600> 
</video>

# 6.训练
![](https://cdnimg103.lizhi.fm/audio_cover/2017/08/17/2619367354329486855_580x580.jpg)


# 7. 鸣谢
{% note info flat %}
最感谢的是LHL小可爱的支持和努力，解决了无数的问题，完成了配置测试
{% endnote %}

