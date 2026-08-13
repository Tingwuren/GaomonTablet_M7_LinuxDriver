# GaomonTablet M7 Linux Driver

[简体中文](README.zh-CN.md)

This repository provides a Linux driver and configuration utility for Gaomon pen displays. It is primarily intended for the **Gaomon M7**, although the package also contains resources for other Gaomon devices.

The repository contains prebuilt binary packages only; source code is not included:

- `GaomonTablet_LinuxDriver_v16.0.0.12.x86_64.deb`
- `gaomontablet-16.0.0.12-1-x86_64.pkg.tar.zst`

Both packages are built for **x86_64**. Confirm your distribution and CPU architecture before installing.

## Supported Systems

| Package | Target systems |
| --- | --- |
| `.deb` | Debian, Ubuntu, and other dpkg-based distributions |
| `.pkg.tar.zst` | Arch Linux, Manjaro, and other pacman-based distributions |

There is no official AUR package and no official RPM package. The Arch package was converted from the `.deb` package; read the Known Issues section before using it.

## Ubuntu / Debian Installation

Run the following command from the repository directory:

```bash
sudo apt install ./GaomonTablet_LinuxDriver_v16.0.0.12.x86_64.deb
```

If your `apt` version does not support local `.deb` files, use:

```bash
sudo dpkg -i GaomonTablet_LinuxDriver_v16.0.0.12.x86_64.deb
sudo apt -f install
```

Reboot after installation. The installation script attempts to disable Wayland and switch the system to X11.

To uninstall:

```bash
sudo apt remove GaomonTablet
```

## Arch Linux Installation

Run the following command from the repository directory:

```bash
sudo pacman -U ./gaomontablet-16.0.0.12-1-x86_64.pkg.tar.zst
```

This package has an empty `Depends On` field, so pacman will not install required runtime libraries automatically. Install the common runtime dependencies first to reduce the chance of startup failures:

```bash
sudo pacman -S --needed \
  libxkbcommon libxkbcommon-x11 libxtst libglvnd libusb \
  fontconfig freetype2 libxcb libxinerama dbus systemd-libs \
  qt5-base qt5-svg qt5-x11extras
```

To uninstall:

```bash
sudo pacman -Rns gaomontablet
```

## Usage After Installation

After installation:

1. Reboot the system.
2. Log in to an X11 session. The driver depends on X11; the installer may modify the GDM configuration to disable Wayland.
3. Connect the Gaomon pen display.
4. Open **GaoMonTablet** from the application menu, or run:

```bash
/usr/lib/gaomontablet/gaomontablet.sh
```

The driver starts two processes:

- `huionCore`: the background service.
- `gaomontablet`: the settings UI.

If the device is not detected, verify that `/usr/lib/udev/rules.d/20-gaomon.rules` exists, then reconnect the device or reboot.

## Known Issues

- The Arch package is not a proper native PKGBUILD artifact. It has `Depends On: None`, `Validated By: None`, and no signature.
- `namcap` reports missing dependency declarations, including `libxkbcommon`, `libxtst`, `libglvnd`, `libusb`, `fontconfig`, `freetype2`, `libxcb`, `libxinerama`, `dbus`, `systemd-libs`, and Qt5-related libraries.
- The package bundles its own Qt5 and ICU libraries and loads them first through `LD_LIBRARY_PATH`; do not assume the system Qt version is the one being used.
- The installation script attempts to modify `/etc/gdm/custom.conf` or `/etc/gdm3/custom.conf`, writing `WaylandEnable=false` and `DefaultSession=x11`.
- The package declares `/etc/xdg/autostart/gaomontablet.desktop`, but that file was missing in the test environment, so autostart may not work.
- The binary packages do not include a clear open-source license statement. The repository root has no `LICENSE` file; assess the trustworthiness of the source before installing.

## Arch Package Test Results

The test environment was Arch Linux. `gaomontablet 16.0.0.12-1` was already installed, the package files could be extracted, and the package existed in the pacman database. However, the checks were not clean:

- `pacman -Qkk gaomontablet` returned a non-zero status and reported many file verification/ownership warnings.
- `/etc/xdg/autostart/gaomontablet.desktop` existed in the database but not on disk.
- `gaomontablet --help` crashed with a core dump in offscreen mode; the `huionCore` background service could start.
- `namcap` reported missing dependency declarations, no signature, an invalid license field, and some undeclared QML modules.

Conclusion: the Arch package can be extracted and recorded by pacman, but it is not guaranteed to run correctly on a clean system. Dependency and Wayland/X11 configuration issues must be handled manually.

## Disclaimer

This project is not an official Gaomon repository, and the binary packages come from third-party conversion. Installing the driver may modify your display-manager configuration. Back up relevant configuration files before installation and review system changes carefully.
