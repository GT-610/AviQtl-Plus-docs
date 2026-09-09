---
title: Frequently asked questions
description: Common questions about AviQtl-Plus, its goals, and its relationship with AviUtl and NeoUtl.
---

# Frequently asked questions

> [!NOTE]
> The following answers reflect the original author's (taisho-guy) personal
> views and development history. AviQtl-Plus-specific answers reflect the
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

AviQtl-Plus is not a "reinvention of AviUtl". While it is strongly inspired by
AviUtl, its internals are entirely different.

| Item | AviQtl-Plus | ExEdit0 | ExEdit2 |
| :--- | :--- | :--- | :--- |
| Core technology | Rust + Slint (Qt6 legacy) | Win32 API | Win32 API |
| Parallelism model | Data-driven (ECS) | Single-threaded | Multi-threaded |
| Memory space | 64-bit | 32-bit (max 4GB) | 64-bit |
| Preview rendering | Vulkan / Metal / DX12 | GDI | DX11 |
| Audio engine | Carla (VST3/LV2 etc.) | Built-in only | Built-in only |
| Plugin system | LuaJIT / C++ / QML / GLSL | Lua / C++ | LuaJIT / C++ |
| Supported OS | Linux, Windows, macOS | Windows | Windows |

AviQtl-Plus fundamentally resolves structural weaknesses:

1. **Data-oriented design with ECS (Entity Component System):** Maximizes CPU
   cache efficiency, accelerating processing of large numbers of objects.
2. **Modern memory management:** Adopts C++23 smart pointers, structurally
   minimizing unexplained crashes.
3. **Separation of UI and rendering:** Timeline operations are not blocked
   during heavy rendering, and the UI remains crisp in High-DPI environments.

## Origin of the name and icon?

The name is a portmanteau of "AviUtl" and "Qt".
The icon is a design combining the Qt and AviUtl logos.

## Can I use AviUtl plugins?

No. The mechanisms differ, so there is no compatibility. There are no plans to
implement a compatibility layer either.

## Why continue AviQtl-Plus development?

The original project was suspended due to genuine technical difficulties with Qt
Quick — but it also proved that **Qt + FFmpeg can rapidly produce a high-quality
video editor prototype**. The architecture is well-designed, the foundation is
solid, and the Qt Quick route is still worth pursuing, especially now that QRhi
provides a viable compute shader path that wasn't fully explored.

As an early core contributor, the maintainer has seen the project's potential
firsthand and picked up where the original author left off — not just to keep
the code alive, but to see the original vision through: a video editor that
inherits AviUtl's operability while surpassing its performance.

## How is AviQtl-Plus different from the original project?

The technical direction remains largely the same (Qt Quick + QRhi + ECS), but
AviQtl-Plus places greater emphasis on:

- **Incremental deliverability:** getting a basic but usable editing workflow
  out the door, rather than pursuing architectural perfection upfront.
- **Pragmatic problem-solving:** working within Qt Quick's constraints rather
  than treating them as blockers.
- **Community transparency:** clear documentation of the fork relationship,
  plans, and long-term intentions.

Once the project reaches a usable state, a rename and rebranding are likely to
clearly distinguish it from the original project. See
[About the project](../start-here/about) for the roadmap.
