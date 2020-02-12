# Intel® RealSense™ Cross Platform API for ARM

[ ![License] [license-image] ] [license]

[release-image]: http://img.shields.io/badge/release-1.12.0-blue.svg?style=flat
[releases]: https://github.com/IntelRealSense/librealsense/releases

[license-image]: http://img.shields.io/badge/license-Apache--2-blue.svg?style=flat
[license]: LICENSE

Platform | Build Status |
-------- | ------------ |
Linux and OS X | [![Build Status](https://travis-ci.org/IntelRealSense/librealsense.svg?branch=master)](https://travis-ci.org/IntelRealSense/librealsense) |
Windows | [![Build status](https://ci.appveyor.com/api/projects/status/y9f8qcebnb9v41y4?svg=true)](https://ci.appveyor.com/project/ddiakopoulos/librealsense) |


This project is a forked library from librealsense-v1.12.1.
I fixed and recompiled it for my experiment on Raspberry Pi 4 and Realsense F200. All functions have been reserved such as native streams, synthetic streams, intrinsic/extrinsic calibration information, ...

For more information, you can check out the official repo from `https://github.com/IntelRealSense/librealsense/`


## Installation Guide

I only considered about Linux/Raspbian OS so the document for those is here: `./doc/installation.md`

### 1. Re-install uvc-video

To make F200 work on Raspbian, uvc-video should be re-compiled and installed. `https://eleccelerator.com/wiki/index.php?title=Raspbian_Buster_ROS_RealSense#Patching_uvcvideo`

```
Patching uvcvideo

Here's the story in a nutshell: uvcvideo is a kernel module that essentially makes webcams plug-and-play. RealSense cameras have some special video streams, such as depth and IR image that have special video formats, but some software we are using still want it to be plug-and-play. (if you run "lsusb -v" you can actually see those GUIDs)

Intel's repo has both patch files and scripts to apply the patch to your kernel, but these won't work on Raspbian. Raspbian Buster, already has half of these already in its copy of uvcvideo so the patch file won't work anyways.

Here is where we need to download the kernel source code and patch it ourselves by editing the source files directly.

Start by gathering the source code and getting some tools

sudo apt-get install bison flex bc
sudo apt-get install raspberrypi-kernel-headers
mkdir ~/uvc_patch
cd ~/uvc_patch
git clone https://github.com/raspberrypi/linux.git
cd linux
uname -r
git status
at this point, "uname -r" shows you what kernel version you are on, "git status" should show you what branch you are on. If the two doesn't match, use "git checkout" to fetch the appropriate branch.

Take a look at https://github.com/IntelRealSense/librealsense/blob/master/scripts/realsense-camera-formats.patch , there are 4 files you are editing: "drivers/media/usb/uvc/Makefile", "drivers/media/usb/uvc/uvc_driver.c", "drivers/media/usb/uvc/uvcvideo.h", "include/uapi/linux/videodev2.h" . Edit those files appropriately according to the additions in the patch file, but without adding duplicates.

After saving the source files, continue with the commands

cd ~/uvcv_patch
cp  /usr/src/linux-headers-`uname -r`/Module.symvers .
sudo modprobe configs
zcat /proc/config.gz > .config
make scripts oldconfig modules_prepare
At this point, you may be prompted to answer some questions, use the default answer if you don't know what to do

cd ~/uvc_patch/linux/drivers/media/usb/uvc
cp ~/uvc_patch/linux/Module.symvers .
make -C ~/uvc_patch/linux M=~/uvc_patch/linux/drivers/media/usb/uvc modules
If you've edited the source files wrong, then this will tell you where.

sudo modprobe -r uvcvideo
sudo rm /lib/modules/`uname -r`/kernel/drivers/media/usb/uvc/uvcvideo.ko
sudo cp ~/uvc_patch/linux/drivers/media/usb/uvc/uvcvideo.ko /lib/modules/`uname -r`/kernel/drivers/media/usb/uvc/uvcvideo.ko
Reboot!

```

### 2. Download and compile librealsense v1.12.1


```
git clone --branch v1.12.1 https://github.com/IntelRealSense/librealsense.git
cd librealsense
```

Legacy SDK (v1.xx) isn't compatible with lastest linux kernel (v4.19.xx). To make it work, according to `https://github.com/IntelRealSense/librealsense/pull/3929`,
We need to change a little bit which showed here `https://github.com/IntelRealSense/librealsense/pull/3929/files`

And add 2 lines below in `CMakeLists.txt`
```
# Add for ARM
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mfpu=neon -mfloat-abi=hard -ftree-vectorize -latomic")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpu=neon -mfloat-abi=hard -ftree-vectorize -latomic")
```

Then compile from source

```
mkdir build && cd build
cmake .. -DBUILD_EXAMPLES=true -DCMAKE_BUILD_TYPE=Release -DFORCE_LIBUVC=true
make -j3
sudo make install
sudo ldconfig
```

then `cd example/` and run some examples to check whether the SDK works or not.

### 3. Install pyrealsense

```
pip3 install numpy cython
pip3 install pyrealsense
```

Checkout the examples from `pyrealsense` repo to make sure everything works.


## Compatible Devices

1. RealSense R200 (Not tested)
2. RealSense F200 (Successful)
3. RealSense SR300 (Not tested)
4. RealSense LR200 (Not tested)
5. [RealSense ZR300] (Not tested)


Additional language bindings (experimental, community maintained):
  * [Python](https://github.com/toinsson/pyrealsense)

## License

Copyright 2016 Intel Corporation

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this project except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
