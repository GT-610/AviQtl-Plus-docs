---
title: Frequently asked questions
description: Common questions about Freya NLE, its goals, and its relationship with AviUtl and NeoUtl.
---

# Frequently asked questions

> [!NOTE]
> The following answers reflect the original author's (taisho-guy) personal
> views and development history. Freya NLE-specific answers reflect the
> maintainer's ([GT610](https://github.com/GT-610)) perspective.

## What motivated the development?

### The OS barrier
It started with AviUtl not running on Linux. **Maintaining a Windows environment
solely for AviUtl** was unacceptable.

### Bloated ecosystem
Regardless of the reason, many users continue using AviUtl reluctantly. The
ecosystem, bloated by years of extensions like a "Howl's Moving Castle", is hard
to let go of despite the dissatisfaction.

### Project goals and mission
During a research project at
[Kagoshima Prefectural Konan High School](https://edunet002.synapse-blog.jp/konan/),
the original author decided to develop NeoUtl independently to solve this
problem.

- **Personal goal:** Produce music videos using only NeoUtl on Linux, without
  juggling Domino, VocalShifter, REAPER, and AviUtl.
- **NeoUtl's mission:** To be the optimal solution for those who use AviUtl
  reluctantly.

## Why develop an AviUtl clone?

Freya NLE is not a "reinvention of AviUtl". While it is strongly inspired by
AviUtl, its internals are entirely different.

| Item | Freya NLE | ExEdit0 | ExEdit2 |
| :--- | :--- | :--- | :--- |
| Core technology | Rust + Slint + wgpu | Win32 API | Win32 API |
| Parallelism model | GPU compositing, threaded decode and export | Single-threaded | Multi-threaded |
| Memory space | 64-bit | 32-bit (max 4GB) | 64-bit |
| Preview rendering | Vulkan / Metal / DX12 | GDI | DX11 |
| Audio engine | Carla (VST3/LV2 etc.) | Built-in only | Built-in only |
| Plugin system | Lua 5.1 (sandboxed) / WGSL effect packages | Lua / C++ | LuaJIT / C++ |
| Supported OS | Linux, Windows, macOS | Windows | Windows |

Freya NLE fundamentally resolves structural weaknesses:

1. **The compositor runs on the GPU:** effect chains, blends, and preview compositing
   are wgpu compute and render passes, so the cost scales with the frame rather than with
   the number of objects on the CPU.
2. **Ownership instead of manual memory management:** the renderer, decoder, and exporter
   are Rust crates with no `unsafe` outside the documented FFmpeg and Carla bridges, which
   removes the class of crash that motivated the port.
3. **Separation of UI and rendering:** Timeline operations are not blocked
   during heavy rendering, and the UI remains crisp in High-DPI environments.

## Origin of the name and icon?

The project was called **AviQtl-Plus** until 0.8: a portmanteau of "AviUtl" and
"Qt", with an icon that combined the Qt and AviUtl logos.

It is now called **Freya NLE**, and the mark follows the name. The filmstrip,
the chamfered plate, and the pixel spark are kept from the earlier icon, so the
mark still reads as the same project. The plus sign became an **F**, because
"Plus" no longer described the project, and the palette moved from green to a
light blue.

## Can I use AviUtl plugins?

No. The mechanisms differ, so there is no compatibility. There are no plans to
implement a compatibility layer either.

## Why continue Freya NLE development?

The original project was suspended over genuine technical difficulties with Qt
Quick, but it also proved that **Qt + FFmpeg can produce a high-quality video
editor prototype quickly**. As an early core contributor, the maintainer picked
up where the original author left off, to see the original vision through: a
video editor that inherits AviUtl's operability while surpassing its performance.

Qt Quick was not the part worth keeping. Freya NLE keeps the editing model — the
object-and-effect timeline that makes AviUtl usable — and rebuilt the frontend on
**Rust + Slint + wgpu**, which is the only build as of 0.8.

## How is Freya NLE different from the original project?

Freya NLE now shares AviUtl's *editing model* with the upstream project rather
than its implementation. What it emphasizes:

- **Incremental deliverability:** getting a basic but usable editing workflow
  out the door, rather than pursuing architectural perfection upfront.
- **Pragmatic problem-solving:** changing the parts that block progress — the
  frontend and the renderer — instead of working around them.
- **Community transparency:** clear documentation of the fork relationship,
  plans, and long-term intentions.

See [About the project](../start-here/about) for the roadmap.
