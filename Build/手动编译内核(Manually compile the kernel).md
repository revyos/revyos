### 中文

内核工具链下载地址：

https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource//1663142514282/Xuantie-900-gcc-linux-5.10.4-glibc-x86_64-V2.6.1-20220906.tar.gz

这里假设编译环境为 `Ubuntu` 或 `Debian`

安装依赖：
```
sudo apt install -y gdisk dosfstools g++-12-riscv64-linux-gnu build-essential libncurses-dev gawk flex bison openssl libssl-dev tree dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf device-tree-compiler
```

解压工具链（这里解压到/opt）：
```
tar -xvf Xuantie-900-gcc-linux-5.10.4-glibc-x86_64-V2.6.1-20220906.tar.gz -C /opt
```

设置环境变量，将工具链加入环境变量中（假设工具链放在/opt中）：
```
export PATH="/opt/Xuantie-900-gcc-linux-5.10.4-glibc-x86_64-V2.6.1/bin:$PATH"
export CROSS_COMPILE=riscv64-unknown-linux-gnu-
export ARCH=riscv
```

使用git下载内核代码：
```
# 内核仓库
git clone https://github.com/revyos/thead-kernel.git
```

编译内核：
```
# 创建安装目标目录
mkdir rootfs && mkdir rootfs/boot

# 目录创建完成后，目录结构应该看起来是这样:
# .. << 当前工作路径
# |-- rootfs
#     |-- boot
# |-- thead-kernel
#     |-- ...

# 进入内核代码目录，开始构建
cd thead-kernel
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv revyos_defconfig
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv -j$(nproc)
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv -j$(nproc) dtbs
sudo make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv INSTALL_MOD_PATH=../rootfs/ modules_install -j$(nproc)
sudo make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv INSTALL_PATH=../rootfs/boot zinstall -j$(nproc)
# 构建perf（如果需要的话）
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv LDFLAGS=-static NO_LIBELF=1 NO_JVMTI=1 VF=1 -C tools/perf/
sudo cp -v tools/perf/perf ../rootfs/sbin/perf-thead
# 安装内核到安装目标目录
sudo cp -v arch/riscv/boot/Image ../rootfs/boot/
# 安装设备树到安装目标目录
sudo cp -v arch/riscv/boot/dts/thead/light-lpi4a.dtb ../rootfs/boot/
sudo cp -v arch/riscv/boot/dts/thead/light-lpi4a-dsi0-hdmi.dtb ../rootfs/boot/
```

之后只需要把rootfs中内容拷贝或覆盖到对应目录即可，注意内核Image和内核module目录一定要对应，不然会因缺失内核模块导致外设功能失效。

### English

Build Toolchain Download link:

https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource//1663142514282/Xuantie-900-gcc-linux-5.10.4-glibc-x86_64-V2.6.1-20220906.tar.gz

Assuming Build environment is `Ubuntu` or `Debian`

Install dependency:
```
sudo apt install -y gdisk dosfstools g++-12-riscv64-linux-gnu build-essential libncurses-dev gawk flex bison openssl libssl-dev tree dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf device-tree-compiler
```

Uncompress Toolchain (Assuming install to /opt):
```
tar -xvf Xuantie-900-gcc-linux-5.10.4-glibc-x86_64-V2.6.1-20220906.tar.gz -C /opt
```

Setup environment variables (Assuming Toolchain is in /opt):
```
export PATH="/opt/Xuantie-900-gcc-linux-5.10.4-glibc-x86_64-V2.6.1/bin:$PATH"
export CROSS_COMPILE=riscv64-unknown-linux-gnu-
export ARCH=riscv
```

Download code using git:
```
# Kernel repo
git clone https://github.com/revyos/thead-kernel.git
```

Build Kernel:
```
# make install target directory
mkdir rootfs && mkdir rootfs/boot

# after mkdir, the directory tree should look like this:
# .. << current workdir
# |-- rootfs
#     |-- boot
# |-- thead-kernel
#     |-- ...

# enter kernel directory and start build
cd thead-kernel
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv revyos_defconfig
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv -j$(nproc)
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv -j$(nproc) dtbs
sudo make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv INSTALL_MOD_PATH=../rootfs/ modules_install -j$(nproc)
sudo make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv INSTALL_PATH=../rootfs/boot zinstall -j$(nproc)
# build perf (if needed)
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv LDFLAGS=-static NO_LIBELF=1 NO_JVMTI=1 VF=1 -C tools/perf/
sudo cp -v tools/perf/perf ../rootfs/sbin/perf-thead
# Install Kernel
sudo cp -v arch/riscv/boot/Image ../rootfs/boot/
# Install DTB
sudo cp -v arch/riscv/boot/dts/thead/light-lpi4a.dtb ../rootfs/boot/
sudo cp -v arch/riscv/boot/dts/thead/light-lpi4a-dsi0-hdmi.dtb ../rootfs/boot/
```

After all build steps, you can copy or override kernel and module files on your board using files in "rootfs", If you replace kernel with new one please make sure you also replace with corresponding kernel module folder.