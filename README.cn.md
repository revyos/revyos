# RevyOS

## 简介

[RevyOS](https://github.com/orgs/revyos/repositories)是由RuyiSDK团队的RevyOS小队支持开发的一款针对T-Head芯片生态的Debian优化定制发行版。

__RevyOS__ 围绕 c906fdv/c910v/c908 等芯片提供了完整而全面的适配和优化支持，默认集成支持 RVV0.7.1和 XThead 的 GCC 工具链，并搭载使用 RVV0.7.1指令集优化过的 glibc 和 kernel。

目前，__RevyOS__ 在办公、网页浏览、观看视频等方面已经能满足用户的基本使用需求。

基于上述定制和优化的 __RevyOS__，在 Lichee RV，Lichee Pi 4A 等硬件平台上，能够提供优秀的性能和极佳的体验。

## 镜像下载及刷写

__RevyOS__ 的用户版镜像目前在 ISCAS（中国科学院软件研究所） 开源镜像站进行更新。如您想获取 __RevyOS__ 最新版镜像请访问[镜像下载](https://mirror.iscas.ac.cn/revyos/extra/images/)目录，根据所使用设备来获取对应镜像。

### Lichee Pi 4A

如您想获取 __RevyOS__ 最新版Lichee Pi 4A用户版镜像请访问[镜像下载](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4a/)目录进行获取。

__RevyOS__ 最新版镜像为 20240529 版本，镜像下载链接为：[RevyOS 0529版本镜像下载](https://github.com/revyos/mkimg-th1520/releases/tag/20240529)，Uboot下载链接为：[RevyOS 0529版本Uboot下载](https://github.com/revyos/thead-u-boot/actions/runs/9326868005)

刷写方式请参考[镜像刷写教程](https://wiki.sipeed.com/hardware/zh/lichee/th1520/lpi4a/4_burn_image.html#%E6%89%B9%E9%87%8F%E7%83%A7%E5%BD%95)。

在完成镜像刷写后用户在登录界面，输入用户名 `debian`，密码 `debian` 即可登录进入系统。

### Lichee Pi 4A&LicheePi Cluster 4A

__RevyOS__ 在稳定发布Lichee Pi 4A用户版镜像的同时还发布了适用于LicheePi Cluster 4A的 __主线内核__ 版本镜像，本镜像在Lichee Pi 4A与LicheePi Cluster 4A上均可进行刷写使用。

此镜像下载目录为：[镜像下载](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4amain/)

最新版本镜像为：[20240529版本下载](https://github.com/revyos/mkimg-th1520/releases/tag/20240529)

### Milk-V Pioneer

__RevyOS__ 目前还发布了适用于Milk-V Pioneer的用户镜像。

此镜像下载目录为：[镜像下载](https://mirror.iscas.ac.cn/revyos/extra/images/sg2042/)

最新版本镜像为：[20240327版本下载](https://mirror.iscas.ac.cn/revyos/extra/images/sg2042/20240327/)

## 更新内容

镜像版本更新后我们会公布当前版本镜像支持内容，如您想查看镜像支持内容请点击[RevyOS版本更新日志](./Change%20Log/)后选择您所需要的版本进行查看。

## issue相关

如果您在使用过程中遇到问题，可以进行[issue申报](https://github.com/revyos/revyos/issues)。

## 用户文档

在本仓库中，我们拥有相关的使用构建与适配文档以及测试文档方便让用户对部分内容进行参考，完善的文档支持加快了用户对于系统的上手时间。

编译与构建相关文档：[点击此处](./Build/)

适配相关文档：[点击此处](./Adaptation/)

测试相关文档：[点击此处](./Test/)

玄铁曳影1520芯片原型用户手册_CN：[点击此处](https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource//1697208997919/%E7%8E%84%E9%93%81%E6%9B%B3%E5%BD%B11520%E8%8A%AF%E7%89%87%E5%8E%9F%E5%9E%8B%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_CN.zip)

TH1520 User Manual_EN：[点击此处](https://occ-intl-prod.oss-ap-southeast-1.aliyuncs.com/resource//1698839996662/TH1520%20User%20Manual%20(1).zip)

曳影1520规格书V1.03: [点击此处](https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource/889768/1698042403122/%E6%9B%B3%E5%BD%B11520%E8%A7%84%E6%A0%BC%E4%B9%A6V1.03.pdf)
