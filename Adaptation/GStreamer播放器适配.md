
# 支持 PTG omxil 库的 GStreamer 播放器适配文档

适用SDK v1.1.2

## 概述

PTG 的 OpenMAX IL 库（下称 `vpu-omxil`）可使 LicheePi 4A 能够流畅硬解码 4k 60fps 的视频，那么具体应该如何使用该库呢？本文将主要介绍 LicheePi 4A 开发板上 Parole 播放器的集成与使用，用户可根据本文来了解在 LicheePi 4A 上的适配过程
以 h264 的硬解为例，视频硬解的工作流程如图所示

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

1. 视频流（video stream）由 GStreamer 读入后经过一系列预处理，送到 GStreamer 的解码器`omxh264dec` 中
2. omxh264dec 调用动态库，即 PTG 提供的 vpu-omxil 库，该库通过驱动访问硬件（kernel module）进行硬解
3. 解码后的流传输到 GStreamer 的 video-sink 中，并由播放器（player）呈现

## A. GStreamer omxh264dec 解码测试

将 omxh264 解码的部分单独拎出来，大体的结构如下

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

我们依照自底向上的顺序构建图示的链条。
本节的主要目的是使 omxh264dec 解码器能够运行，并不涉及到输出屏幕等内容。

### 1. 驱动编译、安装以及硬件访问权限的设置

硬解码需要访问硬件，而访问硬件又需要驱动，所以需要编译并安装驱动

#### 1.1 编译驱动

PTG 提供的驱动源：

[https://github.com/revyos/vpu-vc8000e-kernel](https://github.com/revyos/vpu-vc8000e-kernel)

[https://github.com/revyos/vpu-vc8000d-kernel](https://github.com/revyos/vpu-vc8000d-kernel)

[https://github.com/revyos/video_memory](https://github.com/revyos/video_memory)

##### 1.1.1替代方案

[revyos/thead-kernel](https://github.com/revyos/thead-kernel) 已经合并了上述三个内核模块, 使用revyos_defconfig 可以无需编译上述内核模块

#### 1.2 安装驱动

```shell
# depmod 分析可载入模块的依赖关系，在 /lib/modules/<kernel-version>中添加modules.dep文件，以便后续 modprobe 使用
sudo depmod -a
sudo modprobe vidmem vc8000 hantrodec memalloc

## 如果 modprobe 安装有问题的话，可以尝试使用 insmod 安装
#cd /usr/lib/modules/$(uname -r)
#sudo insmod $(find . -name *vidmem.ko*)
#sudo insmod $(find . -name *vc8000.ko*)
#sudo insmod $(find . -name *hantrodec.ko*)
#sudo insmod $(find . -name *memalloc.ko*)

# 可选：设置开机加载模块
echo -e "\nvidmem\nhantrodec\nmemalloc\nvc8000\n" | sudo tee -a /etc/modules > /dev/null
```

#### 1.3 设置硬件访问权限

安装内核模块后，`/dev` 目录下会出现 `hantrodec` `vidmem` `vc8000` 三个设备文件。默认情况下，用户对其没有访问权限，如果不修改权限的话，非 root 用户在打开 omxil 库时会报错。

```shell
# 生效一次
cd /dev
sudo chmod 666 hantrodec vidmem vc8000

# 长期生效
cat << EOF | sudo tee /lib/udev/rules.d/70-hantro.rules > /dev/null
KERNEL=="vidmem", MODE="0666"
KERNEL=="hantrodec", MODE="0666"
KERNEL=="vc8000", MODE="0666"
EOF
```

#### RevyOS 适配记录

如果要获取 RevyOS 特定版本的内核模块，可进入 [revyos/thead-kernel](https://github.com/revyos/thead-kernel) ，并在 GitHub CI 中下载 artifacts



### 2. 安装 vpu-omxil 并调整配置

首先，请将 vpu-omxil 下载并解压到 /usr/lib/omxil/中
[vpu-omxil_1.2.1.tar.gz](https://drive.google.com/file/d/1pYgCVI7WltfpskltJ-RqzVUCEC21FS56)
如下图所示， 需要

1. 将 `vpu-omxil` 中的 OpenMax 组件注册到 `libomxil-bellagio` 中
2. `gst-omx`（该包提供了 omxh264dec 解码器） 调用 `libomxil-bellagio` 的时候也需要知道调用的组件名称

```plain
+---------+   +-------------------+   +-----------+
| gst-omx +-->| libomxil-bellagio +-->| vpu-omxil |
+---------+   +-------------------+   +-----------+
```

#### 2.1 将 `vpu-omxil` 中的组件注册到 `libomxil-bellagio` 中

```shell
sudo apt install libomxil-bellagio-bin libomxil-bellagio0
# 注册组件
omxregister-bellagio -v /usr/lib/omxil/
```

使用 omxregister-bellagio 生成注册文件，默认路径为 ~/.omxregister

##### 2.1.1 RevyOS/Debian 注册组件

[th1520-vpu](https://github.com/revyos/th1520-vpu) 利用了 debian 在 `usr/lib/riscv64-linux-gnu/libomxil-bellagio0` 安装之后
触发自动注册行为 结果如下

```bash
cat /var/lib/libomxil-bellagio0/registry
/usr/lib/riscv64-linux-gnu/libomxil-bellagio0/libOMX.hantro.H2.video.encoder.so
 ==> OMX.hantro.H2.video.encoder ==> OMX.hantro.H2.video.encoder.avc:OMX.hantro.H2.video.encoder.hevc:
/usr/lib/riscv64-linux-gnu/libomxil-bellagio0/libOMX.hantro.VC8000D.image.decoder.so
 ==> OMX.hantro.VC8000D.image.decoder ==> OMX.hantro.VC8000D.image.decoder.jpeg:
/usr/lib/riscv64-linux-gnu/libomxil-bellagio0/libOMX.hantro.H2.image.encoder.so
 ==> OMX.hantro.H2.image.encoder ==> OMX.hantro.H2.image.encoder.jpeg:
/usr/lib/riscv64-linux-gnu/libomxil-bellagio0/libOMX.hantro.VC8000D.video.decoder.so
 ==> OMX.hantro.VC8000D.video.decoder ==> OMX.hantro.VC8000D.video.decoder.mpeg4:OMX.hantro.VC8000D.video.decoder.avc:OMX.hantro.VC8000D.video.decoder.avs:OMX.hantro.VC8000D.video.decoder.h263:OMX.hantro.VC8000D.video.decoder.wmv:OMX.hantro.VC8000D.video.decoder.vp6:OMX.hantro.VC8000D.video.decoder.vp8:OMX.hantro.VC8000D.video.decoder.jpeg:OMX.hantro.VC8000D.video.decoder.hevc:OMX.hantro.VC8000D.video.decoder.vp9:OMX.hantro.VC8000D.video.decoder.avs2:
```



#### 2.2 调整 gstomx.conf 的设置

调整 gstomx.conf 的设置以使解码器 omxh264dec 调用正确的组件，具体请查看针对 gst-omx 的补丁

[gst-omx-01-add-libomxil-config.patch](https://gist.github.com/Sakura286/26777ea8204c1819885e093806a4f7ca#file-gst-omx-01-add-libomxil-config-patch)

### 3. 添加 dmabuf 补丁

请查看 PTG 提供的针对 gst-omx 的 dmabuf 补丁
[gst-omx-02-set-dec-out-port-dmabuf.patch](https://gist.github.com/Sakura286/26777ea8204c1819885e093806a4f7ca#file-gst-omx-02-set-dec-out-port-dmabuf-patch)

### 4. GStreamer 解码初步测试

```shell
sudo apt install gstreamer1.0-omx-generic gstreamer1.0-omx-bellagio-config gstreamer1.0-plugins-bad gstreamer1.0-plugins-base gstreamer1.0-gl gstreamer1.0-plugins-good gstreamer1.0-tools

# 1 基本解码
gst-launch-1.0 filesrc location=<test.mp4> ! qtdemux !  h264parse ! omxh264dec  ! videoconvert ! fakesink  sync=false
# 2 在终端中显示 fps
# 参考：https://stackoverflow.com/questions/73948308
gst-launch-1.0 filesrc location=<test.mp4> ! qtdemux !  h264parse ! omxh264dec  ! videoconvert ! fpsdisplaysink video-sink=fakesink  text-overlay=false sync=false -v 2>&1
```

`fakesink`会把前面的视频流全部吞掉，不输出画面（因而不会在 video-sink 这一环节损失性能），但是结合fpsdisplaysink可以读取到解码的速度。正常日志如下：

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

**【TIP】**如果有 `[omxh264dec-omxh264dec0: Could not initialize supporting library.](https://gist.github.com/Sakura286/015fae6792e160268db7ad8a697dd2df)` 等字样的报错，可以安装`gst-omx`、`libomxil-bellagio`与`libc6`相关的 debug-symbol 包，使用 `gdb` 启动上述命令进行调试。调试时，先断`DWLInit`，然后再断`open`，具体看是打开哪个地方的时候出错了。

#### RevyOS 适配记录

RevyOS 适配过程中对于初始化动态库失败找到了如下三种原因：

1. 编译 vpu-omxil 时使用的工具链与当前系统不兼容
2. 未使用`omxregister-bellagio`注册 vpu-omxil
3. 未调整 `/dev` 目录下 `hantrodec` `vc8000` `vidmem` 等设备的权限

## B. 选用合适的 GStreamer video-sink

`video-sink` 是视频流在整个 [GStreamer pipeline](https://gstreamer.freedesktop.org/documentation/tutorials/basic/concepts.html) 中的最后一步，其作用一般是将视频流输出到屏幕上。
前文中`fakesink`只是测试解码器是否正常工作的特殊 `video-sink`，[可选的 video-sink](https://gstreamer.freedesktop.org/documentation/tutorials/basic/platform-specific-elements.html?gi-language=c)非常多，常见的有 `autovideosink`，`ximagesink`，`xvimagesink`，`fbdevsink`，`waylandsink`，`glimagesink`，`gtkglsink`等，它们各在不同的插件包里，需要酌情安装：

| **video-sink** | **所属包名** |
| --- | --- |
| waylandsink | gstreamer1.0-plugins-bad |
| fbdevsink | gstreamer1.0-plugins-bad |
| autovideosink | gstreamer1.0-plugins-good |
| gtkglsink | gstreamer1.0-plugins-good |
| ximagesink &#124; xvimagesink | gstreamer1.0-plugins-base |
| glimagesink | gstreamer1.0-plugins-base &#124; gstreamer1.0-gl |

**【TIP】**使用 `gst-inspect-1.0 <video-sink-name>` 来查看对应 video-sink 可用的选项
**【TIP】**添加 `--gst-debug-level=<lv>` 来获得更多的[输出日志](https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html#the-debug-log)，其中 `<lv>` 代表了从 1 到 6，啰嗦程度从低到高，建议在等级 4 及以下，否则日志会非常长
请尝试不同的 video-sink ，并尝试不同的插件参数，以及给予不同的环境变量，直至找到可以流畅硬解 H264 的那一个。

### RevyOS 适配记录

- **`waylandsink`**：由于现在（20230720）RevyOS 采用了 Xfce 桌面，不可能支持 Wayland，故 `waylandsink`从原理上无法使用
- **`fbdevsink`**与**`ximagesink`**：无法使用
- **`xvimagesink`**：通过[流水线图](https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html#getting-pipeline-graphs)以及日志可以确定，playbin 或 autovideosink 会自动调用 xvimagesink，使用 perf 分析后可以发现，使用xvimagesink 不可避免地会进行大量的 memcpy 操作，严重降低解码性能；该问题在获得PTG的 dmabuf 补丁后依然存在，故无法使用
- **`gtkglsink`**：[GTK3 不支持 EGL on X11](https://gitlab.gnome.org/GNOME/gtk/-/issues/738)，而 RevyOS 目前基于 x11，且只支持 EGL，故无法使用

剩下的只有`glimagesink`，根据 [Running and debugging GStreamer Applications](https://gstreamer.freedesktop.org/documentation/gstreamer/running.html#environment-variables)，并观察其他使用到 glimagesink 的例子，可以猜测需要明确指定环境变量 `GST_GL_API`与 `GST_GL_PLATFORM`
由于 RevyOS 使用了 gles2+egl 的组合，使用如下的命令，成功硬解。

```shell
GST_GL_API=gles2 GST_GL_PLATFORM=egl gst-launch-1.0 filesrc location=<test.mp4> ! qtdemux !  h264parse ! omxh264dec  ! videoconvert ! fpsdisplaysink video-sink=glimagesink sync=false
```

然而 GStreamer 被播放器调用时是无法通过环境变量来传递参数的，所以构建 gst-plugins-base 时应当传递额外的 meson 编译参数：

```shell
-Dgl_api=[\'gles2\'] -Dgl_platform=[\'egl\']
```

## C. 播放器支持

GStreamer 的 pipeline 没有问题之后，就需要使播放器支持。不同播放器会使用到不同的 video-sink，同样对 gstreamer 有着不同程度的依赖。
适配播放器时，最重要的工作便是①使播放器适配已验证的 video-sink，或者②使 gstreamer pipeline 支持播放器指定的 video-sink，此次 RevyOS 适配过程采用了①方案。

```plain
                +-------------------------------------------+
                |    +------------+       +------------+    |   +--------+
video stream----+--->| omxh264dec +------>| video-sink +----+-->| player |
                |    +------------+       +------------+    |   +--------+
                |                GStreamer                  |
                +-------------------------------------------+
```

### RevyOS 适配记录

#### videosink 选型

根据 [https://gstreamer.freedesktop.org/apps/](https://gstreamer.freedesktop.org/apps/) 进行简单的排查

| 是否可用 | 是否更新 | 应用名 | 备注 |
| --- | --- | --- | --- |
| ❌ |  | Gnash | Flash 播放器 |
| ❌ |  | GEntrans | Debian 未收录 |
| ❓ | 20230226 | Kaffeine | ❌ 需要大量 KDE 相关组件
||||✔️ 存在于[riscv64 仓库](https://buildd.debian.org/status/package.php?p=kaffeine&suite=sid)中
||||❌ 在 Debian amd64 Gnome 上，播放窗口与控制窗口分离，且默认调用了 VLC 进行播放 |
| ❌ |  | Lcdgrilo | Debian 未收录 |
| ✔️ | 20230218 | Parole | ✔️ For XFCE
||||❓ 不支持 Wayland，仅支持 x11
||||✔️ Debian amd64 Gnome 验证通过
||||✔️ 存在于[riscv64 仓库](https://buildd.debian.org/status/package.php?p=parole&suite=sid) 中 |
| ❌ |  | Songbird | Debian 未收录 |
| ❌ |  | Snappy | Debian 未收录 |
| ❌ |  | Totem | 需要 GTK3，然而 GTK3 不支持 EGL on X11 |

最初选择的播放器是 Totem，但是发现 Totem 无法指定除了 gtkglsink 以外的 video-sink，且由于 RevyOS 不支持 gtkglsink，所以支持 Totem 播放器的难度较大。
对支持 GStreamer 的播放器进行排查后发现了 Parole ， Parole 由 GObject 编写，与常见的面向对象编程略有区别。寻找其构建 parole_gst 对象时的方法 parole_gst_constructed，将 video-sink 属性设置为前文已验证的 glimagesink，补丁：

[parole-01-add-glimagesink-support.patch](https://gist.github.com/Sakura286/26777ea8204c1819885e093806a4f7ca#file-parole-01-add-glimagesink-support-patch)

#### seek 卡死问题

##### 问题描述

如果以 4s 以上的间隔拉动进度条，seek 基本不会出现问题。我试着以1~2s左右的间隔拖动进度条，大约有20%的概率会出现画面停住，大约4s后音频停止的情况，并报audio-sink underflow 错误。向前拖与向后拖，高分辨率与低分辨率，拖动距离长与拖动距离短，与这些因素未见明显的关系。

##### 问题思考

parole 相对比较顶层，应当把问题缩小排查。先抛开播放器，使用`gst-play-1.0`工具进行测试，该工具在包`gstreamer1.0-plugins-base-apps`中，使用如下命令进行测试：

```log
GST_DEBUG_DUMP_DOT_DIR=gst-dump/ gst-play-1.0 --videosink=glimagesink  /media/debian/4A21-0000/video/video/test_animal_4k60fps.mp4  --gst-debug-level=3
```

可以使用左右方向键调整播放进度，发现仍然会报如下错误

```log
0:00:13.579340331 34278 0x3fa465ba90 WARN pulse pulsesink.c:716:gst_pulsering_stream_overflow_cb:<pulsesink0> Got overflow
```

pulsesink 是一种 audio-sink ，所以出问题的可能并不是视频而是音频。因此，指定 audio-sink 为 fakeaudiosink，然后进行测试：

```shell
GST_DEBUG_DUMP_DOT_DIR=gst-dump/ gst-play-1.0 --videosink=glimagesink --audiosink=fakeaudiosink /media/debian/4A21-0000/video/video/test_animal_4k60fps.mp4  --gst-debug-level=3
```

没有再次出现问题，可以判定可能是 audio-sink 的问题。

接下来要解决的问题便成了——需要用什么 audio-sink？

##### audio-sink 的选型

根据 [GStreamer 官方文档](https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html?#getting-pipeline-graphs)，使用如下命令获得解码时的 pipeline:

```log
GST_DEBUG_DUMP_DOT_DIR=gst-dump gst-play-1.0 --videosink=glimagesink  /media/debian/4A21-0000/video/video/test_animal_4k60fps.mp4
cd gst-dump
dot -Tpng 0.00.01.544966313-gst-play.async-done.dot > test.png
```

默认情况下 gstbin 应该会使用 autoaudiosink 来自动选择 audio-sink，从流水线图可以看到是 pulsesink，与前文中日志的输出可以对应。

查看[parole 的源码](https://github.com/xfce-mirror/parole/blob/f2e72acec03b701bb65a425a1ce97720babbbdc6/src/gst/parole-gst.c#L1978)，发现 parole 使用了 autoaudiosink，那么自然会自动选择 pulsesink。

解决方案有两个：

1. 直接修复 pulsesink 的问题
2. 寻找下一种可用的 audio-sink

播放时难以调试，可以先试着探索方案2，那么摆在面前的解决方案又分成了两条

1. 使 autoaudiosink 自动选择另外的 audio-sink
2. 使 parole 直接指定特定的 audio-sink

parole 并没有特别指定 audio-sink，所以方案 1 不合适

查看 gst-inspect-1.0 内的插件，音频 sink 有

| 插件名 | 优先级 | 备注 |
| --- | --- | --- |
| ✔️ alsasink | 256 | ✔️ 可用 |
|||✔️ seek 正常 |
|||❌ 音频有噪点 |
| ✔️ autoaudiosink | 0 | ✔️ 可用，默认选用优先级最高的 pulsesink |
| ~~interaudiosink~~ | ~~0~~ | ~~❌ 不可用~~ |
| ~~jackaudiosink~~ | ~~256~~ | ~~❌ 不可用~~ |
| ✔️ openalsink | 128 | ✔️ 可用
|||✔️ seek 正常|
|||❌ 音频有噪点 |
| ~~oss4sink~~ | ~~129~~ | ~~❌ 不可用~~ |
| ~~osssink~~ | ~~128~~ | ~~❌ 不可用~~ |
| ✔️ pulsesink | 266 | ✔️ 默认选用 |
|||✔️ 可用|
|||❌ seek 出错 |

看起来，除了 pulsesink 之外，还有 alsasink openalsink 可用，但这两者都有噪点问题
根据 [Example GStreamer Pipelines](https://labs.isee.biz/index.php/Example_GStreamer_Pipelines) 搭建 gst-launch-1.0 的 pipeline

```shell
gst-launch-1.0 filesrc location=/media/debian/4A21-0000/video/video/test_motor_4k60.mp4 ! qtdemux name=demux \
demux.audio_0 ! queue !  aacparse ! avdec_aac ! audioconvert ! audioresample ! alsasink \
demux.video_0 ! queue ! h264parse ! decodebin ! videoconvert ! videoscale ! glimagesink --gst-debug-level=3
```

##### 使用 alsasink

GStreamer 的 autovideosink 与 autoaudiosink 本身并不提供解码功能，它们**根据 rank 的高低来选择默认的 sink** ，也就是说，如果我们想用 alsasink 的话，可以直接提高其 rank；pulsesink 的优先级是 `primary + 10`，只要比这个高即可。 alsasink patch:

https://gist.github.com/Sakura286/26777ea8204c1819885e093806a4f7ca#file-parole-02-upgrade-alsasink-patch

## 总结：RevyOS 适配工作

1. 编译驱动模块至内核，设置启动加载，设置设备权限
2. 将 PTG 提供的 omxil 二进制动态库文件打包为 th1520-vpu
   1. 修改 th1520-vpu 的依赖，使其依赖 gst-omx 、libomxil-bellagio 等包
   2. 设置了一些 postinstall 操作，例如使用  omxregister-bellagio  注册组件等
3. 修改 gst-omx
   1. 增加 config 中对 vpu-omxil 组件的支持
   2. 应用 dmabuf 补丁
   3. 增加对 h265 vp9 的支持
4. 修改 gst-base 编译时的 gl 支持，限制为 gles2+egl
5. 修改 parole 以支持 glimagesink

## 本文所用资源

补丁集合：

https://gist.github.com/Sakura286/26777ea8204c1819885e093806a4f7ca

PTG omxil 库

https://drive.google.com/file/d/1pYgCVI7WltfpskltJ-RqzVUCEC21FS56
