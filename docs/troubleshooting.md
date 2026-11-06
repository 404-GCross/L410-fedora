# 故障排查

## 开机约一分钟后自动重启 / `FIQ taken without a root FIQ handler`

固件在交出控制权时留下 AP watchdog（WDT0，SP805 @ `0xfe026000`），约 60 秒到期，
中断被路由成 FIQ。6.18 内核如果没有接管它，就会 panic。

- 确认内核命令行**没有** `l410_deadman=0`——带上它反而保留旧行为
- 安装 `third_party/l410-mainline/system/hardware/l410-watchdog-off.service` 并 `systemctl enable`
- 验证：`cat /sys/kernel/l410_deadman/timeout` 应为 `0`

## 屏幕不亮，只有背光

内核不是 L410 版。`uname -r` 应为 `6.18.54-l410`。Fedora 官方的 `6.19.x` 没有
`kirin990-dss` 显示驱动，只能输出到串口。

## 看不到内部磁盘

同样是内核不对，缺 `ufshcd-kirin`。`lsblk` 里只会出现 `sda`/`sdb`/`sdc` 这三块
固件 LUN，看不到用户盘 `sdd`。

## GRUB 报 `invalid magic number`

GRUB 缺压缩模块。内核包自带的 initramfs 是 gzip 压缩，自建 GRUB 时至少要包含
`gzio`；如果换成了 dracut 生成的 initrd，还需要 `zstd` 和 `xzio`。

## 掉进 `sh-5.3#` 之类的 initramfs shell

根分区挂不上。按顺序排查：

1. `blkid` 看 `root=UUID=` 写的 UUID 是否真实存在
2. `/etc/fstab` 与 `boot.cfg` 里的 UUID 是否一致——克隆或迁移后最容易忘
3. `rd.shell rd.debug` 加在命令行上，进 shell 手动 `mount`

## WiFi 没有 `wlan0`

依次检查：

```bash
lsmod | grep hi110x                  # 模块没加载 → configs/l410.modules
ls /vendor/firmware/hi1103/pilot     # 固件缺失 → 从麒麟分区复制
dmesg | grep -i hi110x               # 固件加载失败会有明确报错
```

## 蓝牙没有 `hci0`

和 WiFi 是同一份固件。另外 `hi110x.bt=1` 才会注册 HCI 设备。

## 5 GHz 信道不可用 / 无线区域不对

`configs/l410-cfg80211.conf` 设置 `ieee80211_regdom=CN`，按所在地区改。

## 上传时网页卡顿

`third_party/l410-mainline/system/hardware/90-l410-net.conf` 把
`net.core.default_qdisc` 设为 `fq_codel`，避免 WiFi 上传把下行延迟顶起来。
装到 `/etc/sysctl.d/`。

## 改了启动配置后开不了机

L410 没有 EFI 变量，启动完全依赖 ESP 上的 `EFI/BOOT/BOOTAA64.EFI` 及其读取的配置。
改动前先备份，并确保手上有一个能独立启动的 U 盘——它是唯一的恢复手段。

## 装完没有桌面

如果 rootfs 是 Minimal Install，需要自行安装桌面环境：

```bash
sudo dnf groupinstall -y "Fedora Workstation"
```
