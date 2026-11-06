# Fedora 44 on Huawei L410

把 Fedora 44 跑在华为擎云 L410（KLVU-WDU0 / KLVU-WDU0B，Kirin 990）上的系统层适配。

> **状态：v0.1 —— 只有配置和文档。** 部署脚本和安装介质仍在制作中，
> 现在还不能照着这个仓库把系统装好。

## 这是什么

让这台机器能跑 Linux 6.18 的内核、驱动、设备树和硬件调试，是另一个项目做的事，
不是这个。本仓库只负责把那些东西接到 Fedora 上：

- Fedora 44 aarch64 的根文件系统构建（`dnf --installroot`，不依赖 U 盘启动）
- Fedora 侧的系统配置：watchdog、WiFi 国家码、模块加载、sysctl、UFS 固件分区只读
- 从机器自带的麒麟分区内部署，装完再接管 ESP，切换到 Fedora 自己的 GRUB

## 这是什么之外

- 不开发内核，不修改内核，不维护驱动
- 不分发厂商固件
- 不做 Fedora Remix，不发 ISO
- 与华为、麒麟软件、Fedora 项目均无隶属关系

## 仓库内容

```text
configs/        Fedora 侧配置（模块加载、无线区域）
docs/           硬件说明、故障排查（安装说明制作中）
third_party/    原样收录的上游文件，GPL-2.0
```

## 硬件状态

| 部件 | 状态 | 验证 | 说明 |
|---|---|---|---|
| 显示 eDP | 可用 | 本机 | `kirin990-dss`，2160×1440 60 Hz |
| 存储 UFS 3.1 | 可用 | 本机 | `ufshcd-kirin`；固件 LUN（sda/sdb/sdc）不要碰 |
| USB 3.1 | 可用 | 本机 | UAS 设备需 `uas` |
| WiFi Hi1103 | 可用 | 本机 | 需从机器自带麒麟分区复制固件 |
| 蓝牙 | 可用 | 本机 | 与 WiFi 共用固件 |
| 键盘 / 触控板 | 可用 | 本机 | |
| 有线网 RTL8168 | 可用 | 上游 | PCIe |
| 休眠 s2idle | 可用 | 上游 | 本机未验证 |
| GPU Panfrost | 可用 | 上游 | 本机未验证 |
| 音频 | 可用 | 上游 | 本机未验证 |
| DP/HDMI、指纹、硬件视频编码 | 不支持 | 上游 | |

「本机」= 在一台 KLVU-WDU0 上实测；「上游」= 上游项目报告可用，本项目未实测。

## 安装

分三处施工，进度：

| 阶段 | 在哪 | 做什么 | 状态 |
|---|---|---|---|
| 一 | PC / CI | 生成 Fedora rootfs（`tar.zst`，约 600 MB） | 制作中 |
| 二 | L410 的麒麟里 | 写分区、解包、复制固件、装内核、加启动项 | 制作中 |
| 三 | L410 的 Fedora 里 | 接管 ESP，切到 Fedora GRUB；可选删除麒麟 | 制作中 |
| 恢复 | U 盘 | 内部盘起不来时使用 | 制作中 |

**为什么不直接用 Fedora 官方安装器**：官方 6.19 内核没有 Kirin 990 的显示和 UFS
驱动，安装器点不亮屏幕，也看不见内部磁盘。

## 内核来源

内核来自上游项目 `linux-l410`，GPL-2.0。本项目**不修改内核**，只分发其构建产物：

- 仓库 <https://github.com/liuwang97/linux-l410>
- 分支 `l410-6.18`
- tag `v6.18.54-l410.2`
- commit `a7b37088ae4c02ccd242f8af986905e48cae6e0a`
- 源码 <https://github.com/liuwang97/linux-l410/archive/refs/tags/v6.18.54-l410.2.tar.gz>
- 二进制 `l410-kernel-6.18.54-l410.2.tar.gz`
  sha256 `00b8817c2acd96f0443fc8b643536cad17731db3299a59c4af328a8e33cb3ab5`

`drivers/staging/hi110x/hi110x.ko` 的许可证为 GPL，随内核一并分发。

## 相关工作与来源

| 仓库 | 作者 | 内容 | 本项目的用法 |
|---|---|---|---|
| [`linux-l410`](https://github.com/liuwang97/linux-l410) | liuwang97 | 内核、驱动、设备树 | 使用其 release 二进制，不修改 |
| [`l410-mainline`](https://github.com/liuwang97/l410-mainline) | liuwang97 | Debian 移植、安装脚本、系统配置、硬件调试记录 | 原样收录于 `third_party/`，由部署脚本调用 |

`third_party/` 下的文件逐字复制自 `l410-mainline` commit
`6d43b38010363dac457ee4fb7eb5824fd4684a61`，未作修改，保留原版权声明与 GPL-2.0
许可。来源与校验见 `third_party/ORIGIN.md`。

本项目与上述作者无隶属关系，未经其审阅或背书。

## 固件

WiFi / 蓝牙固件是华为的，不可再分发。必须从**该机器自己**的麒麟分区复制：

```text
/vendor/firmware/hi1103/pilot
/vendor/etc/cfg_udp_1103_pilot.ini
```

## 已知限制

- **只能用 ext4**：L410 内核没有 XFS，根分区不能是 XFS 或 btrfs
- **不能启用 LUKS**：启动使用的是内核包自带的通用 initramfs
- **必须关闭 Secure Boot**：内核未签名
- **只支持 KLVU-WDU0 / KLVU-WDU0B**：其他 L410 变体设备树不同，且只在 WDU0 上实测
- **SELinux 关闭**：通过命令行 `selinux=0`
- **没有 EFI 变量**：启动全靠 ESP 上的 `EFI/BOOT/BOOTAA64.EFI` 回退路径
- **安装阶段必须有麒麟**：固件要从它复制，启动项也写在它的 SYSBOOT 上

## License

本项目脚本与配置：GPL-2.0。文档：CC-BY-4.0。
`third_party/` 下的内容归原作者，GPL-2.0。内核 GPL-2.0。

仅供参考，刷机风险自负。
