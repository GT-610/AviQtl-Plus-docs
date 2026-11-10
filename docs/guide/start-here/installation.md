---
title: Install
description: Download and start Freya NLE on Windows, Linux, or macOS.
---

# Install Freya NLE

## Download a release

Download a build for your platform from the
[Freya NLE releases page](https://github.com/GT-610/Freya-NLE/releases).
Extract the archive to a folder you can write to, then start the Freya NLE
executable.

Every file is named `Freya-NLE-<version>-<os>-<arch>`, with a suffix when a
build only runs on a narrower system than its OS and architecture imply. Pick
the file that matches your operating system and processor:

| Your system | Download |
| :--- | :--- |
| Windows, 64-bit | `Freya-NLE-<version>-windows-x86_64.zip` |
| macOS on Apple Silicon | `Freya-NLE-<version>-macos-arm64.zip` |
| macOS on Intel | `Freya-NLE-<version>-macos-x86_64.zip` |
| Linux, any distribution | `Freya-NLE-<version>-linux-x86_64.AppImage` |
| Linux, Arch current | `Freya-NLE-<version>-linux-x86_64-archlinux.tar.zst` |

`SHA256SUMS` in the same release lists the checksum of every file, so a download
can be verified before it is run.

Each archive also contains `build-info.json`, which records the commit it was
built from, the Rust and FFmpeg versions it links, and the minimum system it
actually requires.

## Platform notes

### Windows

Use a current 64-bit Windows system. No installer and no redistributable is
needed: extract the archive to a folder you can write to and run
`freya-nle.exe`. The Visual C++ runtime travels inside the archive.

If Windows displays a security prompt, confirm that the archive came from the
official release page before allowing it to run.

### Linux

The AppImage is the build to download on any distribution. It carries its own
FFmpeg and the rest of its libraries, so it does not care which distribution you
run:

```bash
chmod +x Freya-NLE-*.AppImage
./Freya-NLE-*.AppImage
```

It needs a Vulkan-capable graphics driver and its Vulkan runtime, ALSA, and the
X11 or Wayland client libraries — all of which a desktop system already has.
Carla is required only for VST3 and LV2 audio effects.

If your system does not have FUSE, or you are inside a container, run it with
`--appimage-extract-and-run`.

The `-archlinux` archive is the other Linux build: it is linked against Arch
Linux's current glibc and FFmpeg, so it runs on Arch and its derivatives but not
on an older distribution. Its `build-info.json` names the glibc version it
needs. Extract it and run the executable directly:

```bash
tar --zstd -xf Freya-NLE-*.tar.zst
./freya-nle
```

The executable bit survives extraction, so no `chmod` is required.

### macOS

Open the application bundle from Finder. The bundle is ad-hoc signed rather
than notarized, so the first launch is blocked until you review it in
**System Settings → Privacy & Security** and allow it — do that only when the
archive came from the official release page.

The Intel and Apple Silicon bundles are separate downloads because they contain
one architecture each; Rosetta is not needed for either.

## Requirements

Freya NLE renders on the GPU: the preview, the export pipeline, and hardware codec
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
