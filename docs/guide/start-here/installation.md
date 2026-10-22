---
title: Install
description: Download and start AviQtl-Plus on Windows, Linux, or macOS.
---

# Install AviQtl-Plus

## Download a release

Download a build for your platform from the
[AviQtl-Plus releases page](https://github.com/GT-610/AviQtl-Plus/releases).
Extract the archive to a folder you can write to, then start the AviQtl-Plus
executable.

![AviQtl-Plus release assets on GitHub](/images/guide/installation/release-assets.png)

Choose the archive that matches your operating system and architecture.

## Platform notes

### Windows

Use a current 64-bit Windows system. If Windows displays a security prompt,
confirm that the archive came from the official release page before allowing it
to run.

### Linux

Linux builds require a Vulkan-capable graphics driver and its Vulkan runtime,
the FFmpeg shared libraries, libxkbcommon, and the X11 or Wayland client
libraries. Carla is required only for VST3 and LV2 audio effects. An Arch
Linux-equivalent environment is the primary target. On older distributions,
using an Arch container through Distrobox is recommended.

You may need to grant execute permission after extracting the archive:

```bash
chmod +x AviQtl
./AviQtl
```

We're also planning platform-specific packages and AppImage release. These will be shipped in the future.

### macOS

Open the application bundle from Finder. If macOS blocks the first launch,
review the application in **System Settings → Privacy & Security** and allow it
only when it was downloaded from the official release.

## Requirements

AviQtl renders on the GPU: the preview, the export pipeline, and hardware codec
acceleration all need a GPU-backed adapter. A host that exposes only a software
renderer, such as the Microsoft Basic Render Driver, is reported as unsupported
at startup instead of failing later.

## First launch checklist

- The preview window opens without a graphics initialization error.
- The timeline and settings windows can be shown from the application menus.
- Audio and video files can be selected from a local folder.

If startup fails, see [Troubleshooting](../help-and-reference/troubleshooting). 

## Install from source code
See [Build from source](/developer/building).
