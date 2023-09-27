# GStreamer Player Integration with PTG omxil Library

For SDK v1.1.2

## Overview

The PTG OpenMAX IL library (hereinafter referred to as `vpu-omxil`) enables the LicheePi 4A to smoothly hardware-decode 4k 60fps videos. How should this library be used? This document mainly discusses the integration and use of the Parole player on the LicheePi 4A development board. By referring to this guide, users can understand the adaptation process on LicheePi 4A. Using h264 hardware decoding as an example, the workflow of video hardware decoding is shown in the following diagram:

```plain
                +-------------------------------------------+
                |    +------------+       +------------+    |   +--------+
video stream----+--->| omxh264dec +------>| video-sink +----+-->| player |
                |    +------+-----+       +------------+    |   +--------+
                |           |     GStreamer                 |
                +-----------+-------------------------------+
                            |
                      +-----v-----+
                      | vpu-omxil |
                      +-----+-----+
                            |
                            |
                    +-------v-------+
                    | kernel module |
                    |    (driver)   |
                    +-------+-------+
                            |
                            v
                        hardware
```

1. The video stream is read by GStreamer, pre-processed, and then sent to the GStreamer decoder `omxh264dec`.
2. `omxh264dec` invokes the dynamic library, i.e., the vpu-omxil library provided by PTG. This library accesses the hardware (kernel module) for hardware decoding.
3. The decoded stream is transmitted to the GStreamer's video-sink and is displayed by the player.

## A. GStreamer omxh264dec Decoding Test

By isolating the omxh264 decoding section, we get the following structure:

```plain
  +---+------------+----+
  |   +------------+    |
  |   | omxh264dec |    |
  |   +------------+    |
  |      GStreamer      |
  +----------+----------+
             |
  +----+-----v-----+----+
  |    +-----------+    |
  |    | vpu-omxil |    |
  |    +-----------+    |
  |  libomxil-bellagio  |
  +----------+----------+
             |
+------------v------------+
|  - memalloc   - vc8000  |
|  - hantrodec  - vidmem  |
|      kernel modules     |
+------------+------------+
             |
             v
          hardware
```

We construct the chain depicted in the diagram from the bottom up.
The main goal of this section is to ensure the `omxh264dec` decoder can run without dealing with output to the screen.

### 1. Driver Compilation, Installation, and Hardware Access Permission Configuration

Hardware decoding requires access to the hardware, which in turn requires drivers. Therefore, drivers need to be compiled and installed.

#### 1.1 Compile the driver

Sources for the PTG drivers:

- [https://github.com/revyos/vpu-vc8000e-kernel](https://github.com/revyos/vpu-vc8000e-kernel)
- [https://github.com/revyos/vpu-vc8000d-kernel](https://github.com/revyos/vpu-vc8000d-kernel)
- [https://github.com/revyos/video_memory](https://github.com/revyos/video_memory)

##### 1.1.1 Alternative Approach

The [revyos/thead-kernel](https://github.com/revyos/thead-kernel) has already merged the above three kernel modules. By using the `revyos_defconfig`, one can bypass the compilation of the above kernel modules.

#### 1.2 Install the driver

```shell
# depmod analyzes loadable module dependencies and creates modules.dep files in /lib/modules/<kernel-version> for modprobe to use later
sudo depmod -a
sudo modprobe vidmem vc8000 hantrodec memalloc

## If there's an issue with modprobe, you can try using insmod
#cd /usr/lib/modules/$(uname -r)
#sudo insmod $(find . -name *vidmem.ko*)
#sudo insmod $(find . -name *vc8000.ko*)
#sudo insmod $(find . -name *hantrodec.ko*)
#sudo insmod $(find . -name *memalloc.ko*)

# Optionally: Set modules to load on startup
echo -e "\nvidmem\nhantrodec\nmemalloc\nvc8000\n" | sudo tee -a /etc/modules > /dev/null
```

#### 1.3 Setting Hardware Access Permissions

After installing the kernel module, three device files `hantrodec`, `vidmem`, and `vc8000` will appear in the `/dev` directory. By default, users do not have access to them. If the permissions are not modified, non-root users will receive an error when opening the omxil library.

```shell
# Apply once
cd /dev
sudo chmod 666 hantrodec vidmem vc8000

# Make it permanent
cat << EOF | sudo tee /lib/udev/rules.d/70-hantro.rules > /dev/null
KERNEL=="vidmem", MODE="0666"
KERNEL=="hantrodec", MODE="0666"
KERNEL=="vc8000", MODE="0666"
EOF
```

#### RevyOS Adaptation Notes

To acquire the kernel module for specific versions of RevyOS, visit [revyos/thead-kernel](https://github.com/revyos/thead-kernel) and download artifacts from GitHub CI.

### 2. Install vpu-omxil and Adjust Configuration

First, please download and extract vpu-omxil to /usr/lib/omxil/
[vpu-omxil_1.2.1.tar.gz](https://drive.google.com/file/d/1pYgCVI7WltfpskltJ-RqzVUCEC21FS56)
As illustrated below, you need to:

1. Register the OpenMax components from `vpu-omxil` to `libomxil-bellagio`.
2. When `gst-omx` (which provides the omxh264dec decoder) invokes `libomxil-bellagio`, it also needs to know the name of the component being called.

```plain
+---------+   +-------------------+   +-----------+
| gst-omx +-->| libomxil-bellagio +-->| vpu-omxil |
+---------+   +-------------------+   +-----------+
```

#### 2.1 Register components from `vpu-omxil` to `libomxil-bellagio`

```shell
sudo apt install libomxil-bellagio-bin libomxil-bellagio0
# Register components
omxregister-bellagio -v /usr/lib/omxil/
```

Using omxregister-bellagio will generate a registration file. The default path is ~/.omxregister.

##### 2.1.1 RevyOS/Debian Component Registration

[th1520-vpu](https://github.com/revyos/th1520-vpu) leverages the automatic registration behavior of debian at `usr/lib/riscv64-linux-gnu/libomxil-bellagio0`. The result is as follows:

[...output shortened for brevity...]

#### 2.2 Adjust settings of gstomx.conf

Modify the settings of gstomx.conf to make the omxh264dec decoder invoke the correct component. For details, refer to the patch for gst-omx.

[gst-omx-01-add-libomxil-config.patch](https://gist.github.com/Sakura286/26777ea8204c1819885e093806a4f7ca#file-gst-omx-01-add-libomxil-config-patch)

### 3. Add dmabuf Patch

Please refer to the dmabuf patch for gst-omx provided by PTG:
[gst-omx-02-set-dec-out-port-dmabuf.patch](https://gist.github.com/Sakura286/26777ea8204c1819885e093806a4f7ca#file-gst-omx-02-set-dec-out-port-dmabuf-patch)

### 4. GStreamer Preliminary Decoding Test

```shell
sudo apt install gstreamer1.0-omx-generic gstreamer1.0-omx-bellagio-config gstreamer1.0-plugins-bad gstreamer1.0-plugins-base gstreamer1.0-gl gstreamer1.0-plugins-good gstreamer1.0-tools

# 1 Basic decoding
gst-launch-1.0 filesrc location=<test.mp4> ! qtdemux !  h264parse ! omxh264dec  ! videoconvert ! fakesink  sync=false
# 2 Display fps in the terminal
# Reference: https://stackoverflow.com/questions/73948308
gst-launch-1.0 filesrc location=<test.mp4> ! qtdemux !  h264parse ! omxh264dec  ! videoconvert ! fpsdisplaysink video-sink=fakesink  text-overlay=false sync=false -v 2>&1
```

The `fakesink` will swallow up the previous video stream without outputting an image (thus not losing performance at the video-sink stage), but when combined with fpsdisplaysink, the decoding speed can be read. A normal log appears as:

```log
Setting pipeline to PAUSED ...[DBGT]
vc8kdec compiled without trace support (ENABLE_DBGT_TRACE switch not enabled)
Pipeline is PREROLLING ...
Redistribute latency...
OMX  ! decoder_get_parameter OMX_ErrorNoMore (2)
Pipeline is PREROLLED ...
Setting pipeline to PLAYING ...
New clock: GstSystemClockRedistribute latency...
0:01:39.5 / 0:01:49.4 (90.9 %)
```

**【TIP】** If there are error messages like `[omxh264dec-omxh264dec0: Could not initialize supporting library.](https://gist.github.com/Sakura286/015fae6792e160268db7ad8a697dd2df)`, install the debug-symbol packages related to `gst-omx`, `libomxil-bellagio`, and `libc6`, and use `gdb` to launch the above command for debugging. When debugging, first interrupt at `DWLInit`, and then interrupt at `open` to see where the error occurs.