LicheePi4A镜像刷写教程

LicheePi4A支持从 SD card 启动和从 eMMC 启动。以下分别介绍这两种启动方式下该如何刷写RevyOS镜像。

演示环境：Ubuntu22.04

## 从SD card 启动
注意！从sd卡启动不需要改变拨码开关！按照eMMC拨码开关进行设置！
### 准备工作

#### 硬件准备
准备MicroSD 读卡器和一张MicroSD 卡

#### 烧录相关工具
从官网获取烧录工具 BalenaEtcher [https://etcher.balena.io/](https://etcher.balena.io/)
安装zstd用于解压镜像文件

```bash
apt install zstd
```

#### 获取镜像
从以下链接下载 LicheePi4A 的系统镜像：[RevyOS0720](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4a/20240720/)。
解压镜像压缩包得到sdcard-lpi4a-20240720_171951.img文件

```bash
unzstd sdcard-lpi4a-20240720_171951.img.zst
```

###  写入镜像到 MicroSD 卡
把 SD 卡插入读卡器，并将其插入电脑。
运行 BalenaEtcher用于写入镜像到SD 卡。
在 BalenaEtcher 窗口中, 首先点击 "Flash from file" 来选择镜像文件。
![](./image%20for%20flash/lpi4a1.png)
选择完镜像文件后在第二栏选择需要写入的设备。
![](./image%20for%20flash/lpi4a2.png)
两项都选择完以后点击"Flash"写入镜像。
![](./image%20for%20flash/lpi4a3.png)
在等待一段时间后会显示烧录已完成。
![](./image%20for%20flash/lpi4a4.png)

### 系统启动
在写入镜像完成后将SD 卡插入如图所示卡槽中。
![](./image%20for%20flash/lpi4a5.png)
连接hdmi线与电源线后可直接启动。

### 用户登录
登录账户：debian
账户密码：debian

## 从 eMMC 启动

安装 fastboot, zstd

```bash
apt-get install fastboot zstd
```

### 获取镜像
从以下链接下载 LicheePi4A 的系统镜像：[RevyOS0720](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4a/20240720/)。

需要下载的文件是:

- u-boot-with-spl-lpi4a.bin
- boot-lpi4a-20240720_171951.ext4.zst
- root-lpi4a-20240720_171951.ext4.zst

解压
```bash
zstd -d boot-lpi4a-20240720_171951.ext4.zst
zstd -d root-lpi4a-20240720_171951.ext4.zst
```

### 刷写镜像

将 USB 线的一端接到开发板的 type C 接口，按住开发板上的 BOOT 键，并把另一头接到电脑

确认已连接

```
$ lsusb
Bus 003 Device 034: ID 2345:7654 T-HEAD USB download gadget
```

```bash
sudo fastboot flash ram u-boot-with-spl-lpi4a.bin
sudo fastboot reboot
```

稍等 USB 再次连接

```
sudo fastboot flash uboot u-boot-with-spl-lpi4a.bin
sudo fastboot flash boot boot-lpi4a-20240720_171951.ext4
sudo fastboot flash root root-lpi4a-20240720_171951.ext4
```

刷写完毕
