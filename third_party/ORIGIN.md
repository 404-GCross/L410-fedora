# 第三方代码来源

本目录下的文件**逐字复制**自以下仓库，未作任何修改。

```text
仓库    https://github.com/liuwang97/l410-mainline
作者    liuwang97
commit  6d43b38010363dac457ee4fb7eb5824fd4684a61
日期    2026-10-08
许可    GPL-2.0（见 LICENSE）
```

## 文件清单

| 文件 | sha256 |
|---|---|
| `boot/grub-entry.sh` | `4608f5cf08cde59513bad722bb52d5fe567198e4324f0853972c9a60f911062b` |
| `boot/grubenv-set` | `d1e445e0426e50e25b2d4a6cd583fd7106d4a750d6c58d98a8118da3e2c4fd30` |
| `boot/install-kernel.sh` | `6297f460a391ab0d3ce7283e950a6ad82bdd1125522058c7dcfbd8b338ba9f68` |
| `rootfs/firmware.list` | `215a1d698b835dc15558e8e36b7ca29c7f52333e8a111dd0a9db7b1a6f371092` |
| `system/hardware/60-l410-ufs-fw-ro.rules` | `8c84e26fef88e65dbd5db4a87863e2514230f42880c8c8eb126125700ccba710` |
| `system/hardware/61-l410-keyboard.hwdb` | `0a4e41b9ba2b7073d28316fbe2c38200c1ade3a022d49dfb8d5acbcacea84f45` |
| `system/hardware/90-l410-net.conf` | `3601f18bd35e8fa3f86ce3c31477cbb77b680af19114ef539833edb9ad898c23` |
| `system/hardware/l410-watchdog-off.service` | `16ae3889fdd49bf25192f2084077d8ca87897b1a1ecc7bba3881be60117bc5f7` |
| `system/hardware/ucm2/conf.d/hi6405/hi6405.conf` | `f3a12b2c95d42cbfc4927d439a0561173fbcfe6d0162ff9fb2040c2380aa9ee8` |
| `system/hardware/ucm2/conf.d/hi6405/HiFi.conf` | `5d75cd6c780cc6afe0727f12b9e4d8647bdc849fad091b002c869f46a55fceea` |

## 用途

| 文件 | 由谁使用 |
|---|---|
| `boot/install-kernel.sh` | 部署脚本调用，把内核包装进目标系统的 `/boot/l410/` |
| `boot/grub-entry.sh` | 部署脚本调用，往麒麟 GRUB 的 `custom.cfg` 写启动项 |
| `boot/grubenv-set` | `grub-entry.sh` 在目标系统内设置 `saved_entry` / `next_entry` |
| `rootfs/firmware.list` | 部署脚本按此清单从运行中的麒麟复制固件 |
| `system/hardware/*` | 部署脚本安装到目标系统的 udev / systemd / sysctl / ALSA 目录 |

如果上游更新，替换本目录时请同步更新上表的 sha256。
