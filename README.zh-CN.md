# GaomonTablet M7 Linux Driver

[English](README.md)

这个仓库提供 Gaomon 数位屏的 Linux 版驱动设置程序，目前主要针对 **Gaomon M7**，但包内也包含其他 Gaomon 设备的资源文件。

仓库内只有官方编译好的二进制包，不包含源码：

- `GaomonTablet_LinuxDriver_v16.0.0.12.x86_64.deb`
- `gaomontablet-16.0.0.12-1-x86_64.pkg.tar.zst`

两个包都是 **x86_64** 架构。请确认你的发行版和 CPU 架构后再安装。

## 支持系统

| 文件 | 适用系统 |
| --- | --- |
| `.deb` | Debian、Ubuntu 及其他基于 dpkg 的发行版 |
| `.pkg.tar.zst` | Arch Linux、Manjaro 等基于 pacman 的发行版 |

目前没有官方 AUR 包，也没有官方 RPM 包。Arch 包是从 `.deb` 转换而来的，使用前请阅读下面的“已知问题”。

## Ubuntu / Debian 安装

在仓库目录执行：

```bash
sudo apt install ./GaomonTablet_LinuxDriver_v16.0.0.12.x86_64.deb
```

如果使用旧版 `apt` 不支持本地 `.deb`，可以改为：

```bash
sudo dpkg -i GaomonTablet_LinuxDriver_v16.0.0.12.x86_64.deb
sudo apt -f install
```

安装完成后建议重启系统。安装脚本会尝试关闭 Wayland 并切换到 X11。

卸载：

```bash
sudo apt remove GaomonTablet
```

## Arch Linux 安装

在仓库目录执行：

```bash
sudo pacman -U ./gaomontablet-16.0.0.12-1-x86_64.pkg.tar.zst
```

这个包 `Depends On` 为空，因此系统不会自动补齐依赖。建议先安装常用运行库，减少缺库导致启动失败的概率：

```bash
sudo pacman -S --needed \
  libxkbcommon libxkbcommon-x11 libxtst libglvnd libusb \
  fontconfig freetype2 libxcb libxinerama dbus systemd-libs \
  qt5-base qt5-svg qt5-x11extras
```

卸载：

```bash
sudo pacman -Rns gaomontablet
```

## 安装后使用

安装完成后：

1. 重启系统。
2. 登录 X11 会话。该驱动依赖 X11，安装脚本可能会修改 GDM 配置以禁用 Wayland。
3. 连接 Gaomon 数位屏。
4. 在应用菜单中打开 **GaoMonTablet**，或在终端运行：

```bash
/usr/lib/gaomontablet/gaomontablet.sh
```

驱动会启动两个进程：

- `huionCore`：后台服务。
- `gaomontablet`：设置界面。

如果数位屏没有反应，检查 `/usr/lib/udev/rules.d/20-gaomon.rules` 是否存在，然后重新插拔设备或重启。

## 已知问题

- Arch 包不是规范的原生 PKGBUILD 产物，`Depends On: None`、`Validated By: None`、没有签名。
- `namcap` 检查显示 Arch 包缺少依赖声明，包括 `libxkbcommon`、`libxtst`、`libglvnd`、`libusb`、`fontconfig`、`freetype2`、`libxcb`、`libxinerama`、`dbus`、`systemd-libs`、Qt5 相关库等。
- 包内自带一套 Qt5 和 ICU 库，会通过 `LD_LIBRARY_PATH` 优先加载，因此不要同时依赖系统 Qt 的版本一致性。
- 安装脚本会尝试修改 `/etc/gdm/custom.conf` 或 `/etc/gdm3/custom.conf`，写入 `WaylandEnable=false` 和 `DefaultSession=x11`。
- 包内声明了 `/etc/xdg/autostart/gaomontablet.desktop`，但当前测试环境中该文件缺失，自动启动可能不生效。
- 二进制包未附带明确的开源许可证说明。仓库根目录没有 `LICENSE` 文件，安装前请自行评估来源可信度。

## 本次 Arch 包测试结果

测试环境为 Arch Linux，`gaomontablet 16.0.0.12-1` 已经安装，文件可以解包，pacman 数据库中也存在该包。但检查结果并不理想：

- `pacman -Qkk gaomontablet` 返回非零，报告大量文件校验/权限告警。
- `/etc/xdg/autostart/gaomontablet.desktop` 在数据库中，但实际文件不存在。
- 主程序 `gaomontablet --help` 在离屏模式下发生 core dump，后台 `huionCore` 可以启动。
- `namcap` 报告缺少依赖声明、无签名、无有效 license 字段，并提示部分 QML 模块未正确声明。

结论：这个 Arch 包“能被 pacman 解包并记录安装状态”，但在干净系统上不能保证正常启动，尤其是依赖和 Wayland/X11 配置问题需要手动处理。

## 免责声明

本项目不是 Gaomon 官方仓库，二进制包来自第三方转换。安装驱动会修改系统显示管理器配置，请在安装前备份相关配置，并对系统文件变更保持谨慎。
