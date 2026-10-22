---
title: 安装
description: 在 Windows、Linux 或 macOS 上下载并启动 AviQtl-Plus。
---

# 安装 AviQtl-Plus

## 下载发行版本

从 [AviQtl-Plus Releases 页面](https://github.com/GT-610/AviQtl-Plus/releases)下载适用于你的平台的构建。将压缩包解压到具有写入权限的目录，然后启动 AviQtl-Plus 可执行文件。

![GitHub 上的 AviQtl-Plus 发行文件列表](/images/guide/installation/release-assets.png)

选择与你的操作系统和处理器架构匹配的压缩包。

## 平台说明

### Windows

使用较新的 64 位 Windows 系统。如果 Windows 显示安全提示，先确认压缩包来自官方发行页面，再允许其运行。

### Linux

Linux 构建需要支持 Vulkan 的显卡驱动及其 Vulkan 运行环境、FFmpeg 共享库、libxkbcommon，以及 X11 或 Wayland 客户端库。Carla 仅在需要 VST3、LV2 音频效果时才需要。项目主要面向与 Arch Linux 等效的环境；在较旧的发行版上，建议通过 Distrobox 使用 Arch 容器。

解压后可能需要授予执行权限：

```bash
chmod +x AviQtl
./AviQtl
```

我们也计划提供平台专用的软件包和 AppImage 发行版本，未来会陆续发布。

### macOS

从 Finder 打开应用包。如果 macOS 阻止首次启动，请在 **系统设置 → 隐私与安全性** 中检查应用，并仅在确认其来自官方发行页面后允许启动。

## 运行要求

AviQtl 的渲染全部在 GPU 上完成：预览、导出管线与硬件编码加速都需要可用的 GPU 适配器。如果主机只提供软件渲染器（例如 Microsoft Basic Render Driver），程序会在启动时报告该主机不受支持，而不是稍后失败。

## 首次启动检查

- 预览窗口正常打开，没有图形初始化错误。
- 可以通过应用菜单显示时间线和设置窗口。
- 可以从本地目录选择音频和视频文件。

如果无法启动，请阅读[故障排除](../help-and-reference/troubleshooting)。

## 从源代码安装

参见[从源码构建](/zh-CN/developer/building)。
