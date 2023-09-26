# RevyOS

## Introduction

[RevyOS](https://github.com/revyos)  is a custom Debian distribution optimized for the T-Head chip ecosystem, supported and developed by the RevyOS team from the [Ruyi SDK](https://github.com/ruyisdk) team.

RevyOS provides comprehensive adaptation and optimization support for chips such as c906fdv, c910v, and c908. By default, it integrates support for the RVV0.7.1 and XThead GCC toolchains. Furthermore, it is equipped with glibc and kernel optimized for the RVV0.7.1 instruction set.

Currently, RevyOS meets basic user needs in areas like office tasks, web browsing, and video playback.

Thanks to its customizations and optimizations, RevyOS offers superior performance and an excellent experience on hardware platforms like Lichee RV and Lichee Pi 4A.

## Image Download and Installation

The user image of RevyOS for Lichee Pi 4A is currently being updated on the open-source mirror site of [ISCAS (Institute of Software, Chinese Academy of Sciences)](https://mirror.iscas.ac.cn/revyos). To obtain the latest version of the RevyOS image, please visit: [Image Download](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4a/).

For instructions on installing the image, please refer to: [Image Installation Guide](https://wiki.sipeed.com/hardware/zh/lichee/th1520/lpi4a/4_burn_image.html#%E6%89%B9%E9%87%8F%E7%83%A7%E5%BD%95).

The latest image version of RevyOS is the 20230916 edition. The download link for this version is: [RevyOS 0916 Version Download](https://mirror.iscas.ac.cn/revyos/extra/images/lpi4a/20230916/).

The provided link contains the U-boot, boot, and root files. For installation methods, please refer to the [Image Installation Guide](https://wiki.sipeed.com/hardware/zh/lichee/th1520/lpi4a/4_burn_image.html#%E6%89%B9%E9%87%8F%E7%83%A7%E5%BD%95).

Once the image installation is complete, users can log in using the username "debian" and the password "debian".

## Updates and Announcements

After an image version update, we will announce the features and changes supported by the current version. If you wish to view the contents supported by the image, click on [RevyOS Version ChangeLog](https://github.com/ruyisdk/revyos/tree/main/Change%20Log) and select the version you're interested in.

## Issues

If you encounter any issues while using the system, you can [submit an issue](https://github.com/revyos/revyos/issues).

User Documentation
In this repository, we offer various user guides, including documentation on system building, adaptation, and testing. This comprehensive documentation helps users get acquainted with the system more quickly.

For compile and build documentation: [Click here](https://github.com/ruyisdk/revyos/tree/main/Build)

For adaptation-related documentation: [Click here](https://github.com/ruyisdk/revyos/tree/main/Adaptation)

For testing-related documentation: [Click here](https://github.com/ruyisdk/revyos/tree/main/Test).
