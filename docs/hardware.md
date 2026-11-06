# 硬件与系统布局

本文只写 Fedora 侧需要知道的部分。各硬件的驱动、调试过程和已知问题，见上游
[`l410-mainline/docs/hardware`](https://github.com/liuwang97/l410-mainline/tree/main/docs/hardware)
与 [`l410-mainline/docs/known-issues.md`](https://github.com/liuwang97/l410-mainline/blob/main/docs/known-issues.md)。

## 机型

| 项 | 值 |
|---|---|
| 型号 | HUAWEI L410 KLVU-WDU0（本项目实测）/ KLVU-WDU0B（上游） |
| SoC | Kirin 990 |
| 固件 | Byosoft，Secure Boot 必须关闭 |
| 启动键 | F2 进 BIOS，F12 选启动设备 |
| RAM | 8 / 16 GB，视配置 |
| 存储 | UFS 3.1，单颗，暴露为四个 LUN |

## 磁盘与分区

`sda`、`sdb`、`sdc` 是 UFS 的固件 LUN（引导、TEE、LPM3、设备树、recovery 等），
**任何时候都不要写它们**。

用户盘是 `sdd`，出厂布局：

| 分区 | 卷标 | 用途 |
|---|---|---|
| sdd1 | ESP | UEFI 系统分区，`EFI/BOOT/BOOTAA64.EFI` |
| sdd2 | SYSBOOT | 麒麟的 GRUB 配置和内核 |
| sdd3 | SYSROOT | 麒麟根分区 |
| sdd4 | KYLIN-BACKUP | 麒麟系统备份 |
| sdd5 | DATA | 麒麟的 `/data` |

出厂没有空闲空间。给 Fedora 腾地方只有两条路：缩小 DATA，或删除麒麟根分区。
装完之前不要删麒麟——固件和启动项都还要从它那里取。

## 启动链

```text
固件（无 EFI 变量）
  → ESP 的 EFI/BOOT/BOOTAA64.EFI                     ← 唯一的入口
  → 阶段二：麒麟 SYSBOOT 的 grub/custom.cfg
      → source (<root>)/boot/l410/boot.cfg
  → 阶段三：Fedora GRUB 的 EFI/fedora/grub.cfg
      → source (<root>)/boot/l410/boot.cfg
  → /boot/l410/{Image,initrd.img,l410.dtb}
  → root=UUID=<Fedora 根>
```

内核命令行放在 `/boot/l410/boot.cfg` 而不是 GRUB 配置里，换内核不用改 GRUB。

## 内核

`Image` 里 UFS 和 ext4 都是编译进内核的（`CONFIG_SCSI_UFS_KIRIN=y`、
`CONFIG_EXT4_FS=y`），所以内核包自带的 1 MB busybox initramfs 就够挂根分区，
不需要 dracut。

| 路径 | 内容 |
|---|---|
| `/boot/l410/Image` | 内核 |
| `/boot/l410/initrd.img` | busybox initramfs |
| `/boot/l410/l410.dtb` | 设备树 |
| `/boot/l410/boot.cfg` | GRUB 片段，`@ROOT_UUID@` 已替换 |
| `/boot/l410/config-6.18.54-l410` | 内核配置 |
| `/usr/lib/modules/6.18.54-l410/` | 模块 |

## 配置安装位置

| 本仓库文件 | 目标路径 |
|---|---|
| `configs/l410.modules` | `/etc/modules-load.d/l410.conf` |
| `configs/l410-cfg80211.conf` | `/etc/modprobe.d/l410-cfg80211.conf` |
| `third_party/.../60-l410-ufs-fw-ro.rules` | `/etc/udev/rules.d/` |
| `third_party/.../61-l410-keyboard.hwdb` | `/etc/udev/hwdb.d/` |
| `third_party/.../90-l410-net.conf` | `/etc/sysctl.d/` |
| `third_party/.../l410-watchdog-off.service` | `/etc/systemd/system/` |
| `third_party/.../ucm2/conf.d/hi6405/` | `/usr/share/alsa/ucm2/conf.d/hi6405/` |

## 固件

厂商固件不在本仓库，也不在 rootfs 里。部署时必须从运行中的麒麟复制：

```text
/vendor/firmware/hi1103/pilot/
/vendor/etc/cfg_udp_1103_pilot.ini
```

清单见 `third_party/l410-mainline/rootfs/firmware.list`。

## SELinux

Fedora 默认启用，本方案在命令行上关掉（`selinux=0`），不修改 `/etc/selinux/config`，
保持可逆。rootfs 构建时也不写 SELinux 标签。
