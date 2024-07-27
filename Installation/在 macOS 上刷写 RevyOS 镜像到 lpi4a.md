# 在 macOS 上刷写 RevyOS 镜像到 lip4a
## 一、 前置准备

- 使用 `brew` 或其他方式安装 android-platform-tools
- 下载对应硬件的 uboot 以及镜像
- 确保您的开发版通过串口以及 usb 连接 Mac
- 确保您的开发版有通过 12V 供电

开始下载 uboot 以及镜像：

访问该[网站](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4a/)，并且确保您进入的是最新日期的文件夹，这里以 20240720 为例：

下载以下文件：

> 确保您下载的文件与您的硬件相符，荔枝派 4a 分为 8G 以及 16G 版本，对应的 uboot 也不相同。


接着，使用 `brew` 安装 android-platform-tools：

`brew intsall android-platform-tools`

确定 `fastboot` 指令能够正常使用：

```
yuiyuuhayashi@MacBook-Air ~ % fastboot usb 0
< waiting for any device >
```

## 二、开始安装

首先， 让你的荔枝派在启动阶段进入 xx 模式。
输入 `fastboot usb 0`，并且确定您已经通过 type C 接口正确连接到您的 Mac 以及接入 12V 供电
在 Mac 上输入如下指令：

```
fastboot flash uboot <filename>
```

正确刷写后，使用 `fastboot reboot` 进行重启

> 若没有开发版没有正确重启，请手动断电重启

同样的进入 xx 模式，在 macOS 上执行如下指令：

```
fastboot flash root <filename>
```

同样的，使用 `fastboot reboot` 进行重启

## 三、进入系统

在进入系统后，请输入如下账号密码：

```
debian
debian
```

成功进入系统后，可使用 `neofetch` 等方式查看系统版本等信息。