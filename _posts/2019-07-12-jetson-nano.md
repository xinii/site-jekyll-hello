---
layout: post
title: "Install OpenCV on Jetson Nano"
subtitle:  "在 Jetson Nano 上安装 OpenCV"
date:   2019-04-02 14:47:00 +0900
background: '/img/posts/jekyll.png'
---

# 创建 swap 文件
{:.section-heading}

为防止编译途中由于内存不足等问题报错，
这里首先提前创建一个 swap 文件，存放路径任意。
重启后 swap 空间会消失，以下编译等全部工作完成后可以手动删除这个文件。

```
$ fallocate -l 4G swapfile
$ chmod 600 swapfile
$ mkswap swapfile
$ sudo swapon swapfile
$ swapon -s
```

# 创建安装脚本
{:.section-heading}

[安装 OpenCV 4.0.0 版本的脚本原始链接](https://github.com/AastaNV/JEP/blob/master/script/install_opencv4.0.0_Nano.sh)

以上是安装 OpenCV 4.0.0 的版本，在这里我们希望安装较新的 4.1.0 版本，将上面的代码稍作改动（版本号的部分）：

```
#!/bin/bash
#
# Copyright (c) 2018, NVIDIA CORPORATION.  All rights reserved.
#
# NVIDIA Corporation and its licensors retain all intellectual property
# and proprietary rights in and to this software, related documentation
# and any modifications thereto.  Any use, reproduction, disclosure or
# distribution of this software and related documentation without an express
# license agreement from NVIDIA Corporation is strictly prohibited.
#

if [ "$#" -ne 1 ]; then
    echo "Usage: $0 <Install Folder>"
    exit
fi
folder="$1"
user="nvidia"
passwd="nvidia"

echo "** Install requirement"
sudo apt-get update
sudo apt-get install -y build-essential cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install -y libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
sudo apt-get install -y python2.7-dev python3.6-dev python-dev python-numpy python3-numpy
sudo apt-get install -y libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
sudo apt-get install -y libv4l-dev v4l-utils qv4l2 v4l2ucp
sudo apt-get install -y curl
sudo apt-get update

echo "** Download opencv-4.1.0"
cd $folder
curl -L https://github.com/opencv/opencv/archive/4.1.0.zip -o opencv-4.1.0.zip
curl -L https://github.com/opencv/opencv_contrib/archive/4.1.0.zip -o opencv_contrib-4.1.0.zip
unzip opencv-4.1.0.zip 
unzip opencv_contrib-4.1.0.zip 
cd opencv-4.1.0/

echo "** Building..."
mkdir release
cd release/
cmake -D WITH_CUDA=ON -D CUDA_ARCH_BIN="5.3" -D CUDA_ARCH_PTX="" -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.1.0/modules -D WITH_GSTREAMER=ON -D WITH_LIBV4L=ON -D BUILD_opencv_python2=ON -D BUILD_opencv_python3=ON -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..

make -j3
sudo make install
sudo apt-get install -y python-opencv python3-opencv

echo "** Install opencv-4.1.0 successfully"
echo "** Bye :)"
```

将上面的脚本保存为 sh 文件，例如：install_opencv4.1.0_Nano.sh

# 执行脚本

```
mkdir opencv
sh install_opencv4.1.0_Nano.sh opencv
```
安装会比较花时间，耐心等待安装结束，建议使用 tmux 等挂在后台执行。


# 测试已安装的 OpenCV

这里应该在 python 的交互界面执行 `import cv2` 就不会报错了。

不过，考虑到使用 pyenv 等工具进行 python 版本管理的用户，
此时如果切换到 pyenv 下安装的 python 版本可能无法检测到已安装的 OpenCV，
作为解决版本是创建一个 OpenCV 库的软连接到 pyenv 中 python 版本的库目录下。

假设你的 pyenv 中的 python 版本的库目录为`/home/xin/.anyenv/envs/pyenv/versions/3.6.8/lib/python3.6/site-packages`，
安装好的 OpenCV 库路径为`/usr/local/lib/python3.6/dist-packages/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so`，
那么执行下面的语句：

```
cd /home/xin/.anyenv/envs/pyenv/versions/3.6.8/lib/python3.6/site-packages
ln -s /usr/local/lib/python3.6/dist-packages/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so ./
```

另外提前使用 pyenv 中的 python 版本的 pip 安装好 numpy（`pip install numpy`），
然后去交互界面执行 `import cv2` 应该就一切正常了。
