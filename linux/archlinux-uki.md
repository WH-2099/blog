---
description: systemd-boot、UKI 与 DPS
---

# Arch Linux UKI 启动

## 前提

本文假设：

1. 系统使用 UEFI、GPT、`systemd-boot`、`mkinitcpio`。
2. 根文件系统是 Btrfs，系统子卷类似 `@`。
3. ESP 和 root 在同一块物理磁盘上。
4. `mkinitcpio` 使用 `systemd` initramfs hook。没有这个 hook，root 分区不能只靠 DPS 自动发现。
5. root 没有跨盘、RAID 或加密层。
6. CPU 微代码包按机器实际情况选择：AMD 使用 `amd-ucode`，Intel 使用 `intel-ucode`。

操作前先确认分区与挂载关系：

```bash
findmnt / /boot /efi
lsblk -f
sudo sgdisk -p /dev/nvme0n1
```

下面命令里的磁盘、分区号、UUID 都要替换成自己的实际值。

## 启用 systemd initramfs

省略 `root=` 依赖 initrd 阶段的 `systemd-gpt-auto-generator`。清空内核命令行之前，先确认 `/etc/mkinitcpio.conf` 里的 `HOOKS` 已经从传统 busybox 流程切到 `systemd` 流程：

```bash
grep '^HOOKS=' /etc/mkinitcpio.conf
```

普通未加密 Btrfs root 至少要包含 `systemd`、`block`、`filesystems`。如果原配置还在使用 `udev`，要先按自己的磁盘、键盘和文件系统情况迁移到 systemd hook 体系。加密 root、LVM、RAID 等场景需要对应的 systemd initramfs hook，不在本文范围内。

## 迁移 ESP 挂载点

把 ESP 从 `/boot` 迁移到 `/efi`，修改 `/etc/fstab` 中 ESP 对应行：

```
UUID=XXXX-XXXX  /efi  vfat  noauto,x-systemd.automount,x-systemd.idle-timeout=60s,umask=0077,errors=remount-ro  0 2
```

重新挂载，并重装内核与微代码。这样原本写进 ESP 的内核文件会释放到 Btrfs 管理的 `/boot` 目录下：

```bash
sudo umount /boot
sudo mkdir -p /efi
sudo mount /efi
sudo pacman -S linux amd-ucode
```

Intel CPU 把最后一行替换成：

```bash
sudo pacman -S linux intel-ucode
```

## 设置 Btrfs 默认子卷

先找出系统根子卷 ID，例如 `@` 对应的 ID：

```bash
sudo btrfs subvolume list /
```

假设目标 ID 是 `256`，把它设成文件系统默认挂载子卷：

```bash
sudo btrfs subvolume set-default 256 /
```

这一步的意义是把“默认挂哪个子卷”写进 Btrfs 文件系统本身，而不是继续依赖内核命令行里的 `rootflags=subvol=`。

## 设置 DPS 分区类型

DPS 依赖 GPT 分区类型。以 `/dev/nvme0n1` 为例，假设 ESP 是第 1 个分区，root 是第 3 个分区：

```bash
sudo sgdisk -t 1:ef00 /dev/nvme0n1
sudo sgdisk -t 3:8304 /dev/nvme0n1
```

其中：

1. `ef00` 是 EFI System。
2. `8304` 是 Linux x86-64 root。

ARM64、RISC-V 等架构要换成对应的 root 分区类型。修改后可以检查一次：

```bash
sudo sgdisk -p /dev/nvme0n1
```

## 清空内核命令行

保留一个空的 `/etc/kernel/cmdline`，避免生成 UKI 时回退读取 `/proc/cmdline` 里的旧参数：

```bash
sudo sh -c '> /etc/kernel/cmdline'
```

如果仍然需要 `quiet`、`loglevel=`、`lsm=` 等参数，再显式写入这个文件。要做纯自动发现启动，就保持为空。

## 修改 mkinitcpio 预设

编辑 `/etc/mkinitcpio.d/linux.preset`，关闭传统 initramfs `img` 输出，启用 UKI 输出：

```ini
ALL_kver="/boot/vmlinuz-linux"
PRESETS=('default' 'fallback')

default_uki="/efi/EFI/Linux/arch-linux.efi"
fallback_uki="/efi/EFI/Linux/arch-linux-fallback.efi"
fallback_options="-S autodetect"
```

重点是删除或注释掉这些传统输出项：

```ini
default_image="/boot/initramfs-linux.img"
fallback_image="/boot/initramfs-linux-fallback.img"
```

## 重建引导环境

确认 `/efi` 已经挂载到 ESP 后，再删除本文会重新生成的旧启动产物：

```bash
findmnt /efi
sudo rm -rf /efi/EFI/Linux
sudo rm -rf /efi/loader
sudo bootctl install
sudo mkdir -p /efi/EFI/Linux
sudo mkinitcpio -P
```

删除 `/efi/EFI/Linux` 和 `/efi/loader` 只应该在确认 `/efi` 正确挂载 ESP 后执行。否则删的就是根文件系统里的普通目录。

## 验证

检查 `systemd-boot` 是否识别到 UKI：

```bash
bootctl list
```

看到 `arch-linux.efi` 对应条目显示为 `Type #2 (UKI, .efi)` 后，即可重启验证。

## 参考

1. [ArchWiki: Unified kernel image](https://wiki.archlinux.org/title/Unified_kernel_image)
2. [sd-boot(7)](https://www.freedesktop.org/software/systemd/man/latest/sd-boot.html)
3. [systemd-gpt-auto-generator(8)](https://www.freedesktop.org/software/systemd/man/devel/systemd-gpt-auto-generator.html)
4. [Discoverable Partitions Specification](https://uapi-group.org/specifications/specs/discoverable_partitions_specification/)
