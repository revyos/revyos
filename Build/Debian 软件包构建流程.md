## 宿主 debian

```bash
# qemu-user 下编译
sudo apt update
sudo apt install -y \
	sbuild buildd qemu-system-misc qemu-user-static binfmt-support \
	ca-certificates apt-transport-https devscripts mmdebstrap

# native
sudo apt install -y \
	sbuild buildd ca-certificates apt-transport-https devscripts mmdebstrap

# 修正宿主的debian-ports证书相关问题 现阶段可能不需要了
wget https://mirror.sjtu.edu.cn/debian/pool/main/d/debian-ports-archive-keyring/debian-ports-archive-keyring_2023.02.01_all.deb
sudo dpkg -i ./debian-ports-archive-keyring_2023.02.01_all.deb


# sbuild 增加当前用户免root
sudo sbuild-adduser $USER
```

## 创建打包环境

### revyos-c910v

```bash
export SUFFIX=revyos-c910v-sbuild
sudo sbuild-createchroot --debootstrap=debootstrap --arch=riscv64 \
	--chroot-suffix=-$SUFFIX \
	--keyring='' \
	--no-deb-src \
	--include=debian-ports-archive-keyring,ca-certificates,apt-transport-https,eatmydata \
	--extra-repository="deb [trusted=yes] https://mirror.iscas.ac.cn/revyos/revyos-c910v/ revyos-c910v main contrib non-free" \
	--extra-repository="deb [trusted=yes] https://mirror.iscas.ac.cn/revyos/revyos-addons/ revyos-addons main" \
	sid /srv/chroot/sid-riscv64-$SUFFIX \
	https://mirror.iscas.ac.cn/revyos/revyos-base/

# 修正环境相关问题
sudo sed -i 's/deb http/deb [trusted=yes] http/g' /srv/chroot/sid-riscv64-$SUFFIX/etc/apt/sources.list
sudo rm -rf /srv/chroot/sid-riscv64-$SUFFIX/var/lib/apt/lists/*
echo "command-prefix=eatmydata" | sudo tee -a /etc/schroot/chroot.d/sid-riscv64-$SUFFIX-*

# 调整source顺序 - 目的是同版本使用c910v仓库的
# 编辑sources.list 确保以下顺序
deb [trusted=yes] https://mirror.iscas.ac.cn/revyos/revyos-c910v/ revyos-c910v main contrib non-free
deb [trusted=yes] https://mirror.iscas.ac.cn/revyos/revyos-addons/ revyos-addons main
deb [trusted=yes] https://mirror.iscas.ac.cn/revyos/revyos-base/ sid main contrib non-free non-free-firmware
```

## 构建命令

### revyos-c910v

```bash
sbuild --arch=riscv64 -d sid -c sid-riscv64-revyos-c910v-sbuild xxx.dsc
```