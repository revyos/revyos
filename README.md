# RevyOS

## 简介

[RevyOS](https://github.com/orgs/revyos/repositories)是由如意SDK团队的RevyOS小队支持开发的一款针对T-Head芯片生态的Debian优化定制发行版。

__RevyOS__ 围绕 c906fdv/c910v/c908 等芯片提供了完整而全面的适配和优化支持，默认集成支持 RVV0.7.1和 XThead 的 GCC 工具链，并搭载使用 RVV0.7.1指令集优化过的 glibc 和 kernel。

目前，__RevyOS__ 在办公、网页浏览、观看视频等方面已经能满足用户的基本使用需求。

基于上述定制和优化的 __RevyOS__，在 Lichee RV，Lichee Pi 4A 等硬件平台上，能够提供优秀的性能和极佳的体验。

## 镜像下载及刷写

__RevyOS__ 的Lichee Pi 4A用户版镜像目前在 ISCAS（中国科学院软件研究所） 开源镜像站进行更新。如您想获取 __RevyOS__ 最新版镜像请访问：[镜像下载](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4a/)

镜像刷写请参考：[镜像刷写教程](https://wiki.sipeed.com/hardware/zh/lichee/th1520/lpi4a/4_burn_image.html#%E6%89%B9%E9%87%8F%E7%83%A7%E5%BD%95)

__RevyOS__ 最新版镜像为 20230614 版本，其中 __U-boot__ 文件下载链接为：[U-boot下载](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4a/20230511/u-boot-with-spl-lpi4a-20230510.bin)

boot文件以及root文件下载链接为：[镜像下载](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4a/20230614/)

在完成镜像刷写后用户在登录界面，输入用户名 `debian`，密码 `debian` 就可以登录进入系统了。

## 更新与预告

镜像版本更新后我们会公布当前版本镜像支持内容，如您想查看镜像支持内容请点击[RevyOS版本更新日志](./Change%20Log/)后选择您所需要的版本进行查看。

同时我们会在[此界面](./update%20notice.md)发布下版本预期更新内容，您可在该页面进行查看。

## issue相关

如果您在使用过程中遇到问题，可以进行[issue申报](https://github.com/revyos/revyos/issues)。

## 用户文档

在本仓库中，我们拥有相关的使用构建与适配文档以及测试文档方便让用户对部分内容进行参考，完善的文档支持加快了用户对于系统的上手时间。

编译与构建相关文档：[点击此处](./Build/)

适配相关文档：[点击此处](./Adaptation/)

测试相关文档：[点击此处](./Test/)
