从 https://github.com/revyos/thead-kernel/actions/workflows/kernel.yml 获取 Github CI 的 Artifacts (kernel.tar.gz)

```bash
# /boot 文件系统只有 32M，而 该分区有 500M 的空间
# 需要将文件系统拓展到整个分区
sudo resize2fs /dev/mmcblk0p2

# 解压后出来一个 rootfs 文件夹，目录树结构为：
# .
# |-- boot
# |   |--...
# |-- lib
# |   |--...
# |-- sbin
#     |--...
unzip kernel.tar.gz.zip
tar xf kernel.tar.gz

cd rootfs
# 注意权限，注意复制的目标位置
sudo cp -r boot /
sudo cp -r lib /usr
sudo cp -r sbin /usr

sudo reboot
```

可以通过uname -a对比当前内核版本与rootfs/boot内文件的版本号来确定修改是否生效

## English

Retrieve the Github CI's Artifacts (`kernel.tar.gz`) from `https://github.com/revyos/thead-kernel/actions/workflows/kernel.yml`:

```bash
# The /boot filesystem only has 32M, but this partition has 500M of space.
# It's necessary to expand the filesystem to the entire partition.
sudo resize2fs /dev/mmcblk0p2

# After decompression, a folder named 'rootfs' will be produced, with a directory structure as:
# .
# |-- boot
# |   |--...
# |-- lib
# |   |--...
# |-- sbin
#     |--...
unzip kernel.tar.gz.zip
tar xf kernel.tar.gz

cd rootfs
# Be mindful of permissions and the destination where files are copied.
sudo cp -r boot /
sudo cp -r lib /usr
sudo cp -r sbin /usr

sudo reboot
```

You can compare the current kernel version with the version number in the `rootfs/boot` directory using `uname -a` to confirm if the changes were applied successfully.