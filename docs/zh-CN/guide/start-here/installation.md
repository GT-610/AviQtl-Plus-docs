---
title: 安装
description: 在 Windows、Linux 或 macOS 上下载并启动 Freya NLE。
---

# 安装 Freya NLE

## 下载发行版本

从 [Freya NLE Releases 页面](https://github.com/GT-610/Freya-NLE/releases)下载适用于你的平台的构建。将压缩包解压到具有写入权限的目录，然后启动 Freya NLE 可执行文件。

所有发行文件都命名为 `Freya-NLE-<版本>-<系统>-<架构>`；如果某个构建只能在比"系统 + 架构"更窄的环境下运行，会再追加一个后缀。请选择与你的操作系统和处理器匹配的文件：

| 你的系统 | 下载 |
| :--- | :--- |
| Windows 64 位 | `Freya-NLE-<版本>-windows-x86_64.zip` |
| Apple Silicon 上的 macOS | `Freya-NLE-<版本>-macos-arm64.zip` |
| Intel 上的 macOS | `Freya-NLE-<版本>-macos-x86_64.zip` |
| Linux（任意发行版） | `Freya-NLE-<版本>-linux-x86_64.AppImage` |
| Linux（Arch current） | `Freya-NLE-<版本>-linux-x86_64-archlinux.tar.zst` |

同一版本的发行页面还提供 `SHA256SUMS`，列出了每个文件的校验和，可以在运行前校验下载结果。

每个压缩包内都带有 `build-info.json`，记录了它由哪个提交构建、链接的 Rust 与 FFmpeg 版本，以及它实际要求的最低系统版本。

## 平台说明

### Windows

使用较新的 64 位 Windows 系统。既不需要安装程序，也不需要单独安装运行库：将压缩包解压到具有写入权限的目录，直接运行 `freya-nle.exe` 即可，Visual C++ 运行库随压缩包一起分发。

如果 Windows 显示安全提示，先确认压缩包来自官方发行页面，再允许其运行。

### Linux

在任何发行版上都应下载 AppImage。它自带 FFmpeg 及其余运行库，因此与你使用哪个发行版无关：

```bash
chmod +x Freya-NLE-*.AppImage
./Freya-NLE-*.AppImage
```

它需要支持 Vulkan 的显卡驱动及其 Vulkan 运行环境、ALSA，以及 X11 或 Wayland 客户端库——桌面系统通常都已具备。Carla 仅在需要 VST3、LV2 音频效果时才需要。

如果系统没有 FUSE，或你身处容器中，请用 `--appimage-extract-and-run` 运行。

`-archlinux` 压缩包是另一种 Linux 构建：它链接 Arch Linux 当前的 glibc 与 FFmpeg，因此可在 Arch 及其衍生版上运行，但在较旧的发行版上无法运行；它所需的 glibc 版本写在包内的 `build-info.json` 里。解压后直接运行可执行文件：

```bash
tar --zstd -xf Freya-NLE-*.tar.zst
./freya-nle
```

解压后会保留执行权限，无需再执行 `chmod`。

### macOS

从 Finder 打开应用包。应用包使用 ad-hoc 签名而非公证，因此首次启动会被阻止，需要在 **系统设置 → 隐私与安全性** 中放行——并且仅在确认压缩包来自官方发行页面后才这样做。

Intel 与 Apple Silicon 是各自独立的下载，因为每个包只包含一种架构；两者都不需要 Rosetta。

## 运行要求

Freya NLE 的渲染全部在 GPU 上完成：预览、导出管线与硬件编码加速都需要可用的 GPU 适配器。如果主机只提供软件渲染器（例如 Microsoft Basic Render Driver），程序会在启动时报告该主机不受支持，而不是稍后失败。

## 首次启动检查

- 预览窗口正常打开，没有图形初始化错误。
- 可以通过应用菜单显示时间线和设置窗口。
- 可以从本地目录选择音频和视频文件。

如果无法启动，请阅读[故障排除](../help-and-reference/troubleshooting)。

## 从源代码安装

参见[从源码构建](/zh-CN/developer/building)。
