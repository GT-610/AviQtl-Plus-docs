---
title: About the project
description: What AviQtl-Plus is, its history, and its relationship with the upstream NeoUtl project.
---

# About the project

## What is AviQtl-Plus?

AviQtl-Plus is a free, open-source video editor that inherits the operability
of **AviUtl 1.10** and **ExEdit 0.92** while delivering **performance that
surpasses AviUtl**. It is a cross-platform desktop application. The frontend
is migrating from Qt Quick + QRhi to **Rust + Slint + wgpu** — Slint is the
default build (under construction) — on top of an ECS-oriented core with a
shared Rust workspace for timeline, effects, audio, rendering, and media.

Its key strengths are:

- A user interface closely resembling AviUtl.
- Fast and powerful GPU-accelerated effects.
- Support for audio effects such as VST3 and LV2 through Carla.
- A LuaJIT plugin system with package management, declarative parameters, and
  permission control.
- Cross-platform support for **Linux**, **Windows**, and **macOS**.

The [AviUtl operability targets](../../developer/operability-targets) describe the
editing-model compatibility goals that guide development.

## Project history

This repository is a fork of [taisho-guy/NeoUtl](https://codeberg.org/taisho-guy/NeoUtl).
In late May 2026, the original author decided to **reset development** — the
`main` branch now hosts a new NeoUtl built on **Rust + Slint + wgpu**, while the
Qt Quick-based version was moved to the `aviqtl` branch (referred to as
**AviQtl**).

As a result, three NeoUtl-related projects now exist in parallel:

- **[NeoUtl](https://codeberg.org/taisho-guy/NeoUtl)** — the original project,
  rebuilt with Rust + Slint + wgpu.
- **[AviQtl](https://codeberg.org/taisho-guy/NeoUtl/src/branch/aviqtl)** — the
  original Qt Quick-based version, continuing as "AviQtl".
- **AviQtl-Plus (this project)** — a fork continuing the Qt Quick + QRhi + ECS
  approach, now migrating its frontend to Rust + Slint + wgpu.

## Frontend migration

The default `BUILD.py` build now targets the **Rust + Slint frontend**
(under construction). While the migration is in progress, the legacy Qt
Quick frontend remains available for development use:

```bash
python3 BUILD.py --xcode --frontend qt --debug
```

Qt builds are undeployed development binaries; see
[Build from source](../../developer/building#frontend-selection) for details.

### Why the original project paused

The original author identified fundamental issues with Qt Quick:

- Qt Quick's resource management is incompatible with Compute Shaders, making
  implementation difficult.
- It conflicts with ECS architecture, hindering optimization.
- Real-time processing raised performance concerns.

The original author has since moved to a new technology stack based on
**Rust + Slint + wgpu**, abandoning the Qt Widgets + SDL3 + bgfx approach that
was initially planned.

### How AviQtl-Plus addresses them

Building on the insights gained from the original project, AviQtl-Plus tackles
these challenges:

1. **Compute Shader integration** — circumventing Qt Quick's constraints to
   leverage GPU compute for high-performance effect processing.
2. **ECS architecture optimization** — maximizing the benefits of data-oriented
   design while resolving friction with Qt Quick.
3. **Real-time performance** — eliminating preview rendering bottlenecks to
   deliver a smooth editing experience.

The vision remains unchanged: **a video editor that inherits and surpasses
AviUtl**. We believe the Qt Quick + QRhi + ECS approach is a viable and valuable
path — one we are committed to maturing into a practical, everyday editing tool.

### Relationship with the new NeoUtl

The original author and the AviQtl-Plus maintainer keep a friendly relationship.
If any contribution from AviQtl-Plus proves applicable to the new NeoUtl
(Rust + Slint + wgpu), it will gladly be submitted upstream. Similarly, both
projects hope their innovations can cross-pollinate over time, ultimately
benefiting all users who seek a modern, powerful, and intuitive AviUtl
successor.

The old Qt Quick source code remains available on the
[`aviqtl` branch](https://codeberg.org/taisho-guy/NeoUtl/src/branch/aviqtl), and
the original author recommends AviQtl-Plus to users who prefer the Qt Quick
approach.

## Roadmap

The project is entirely driven by personal motivation — there are no deadlines
or commercial pressures. Progress is steady but paced.

**Completed (0.3.x–0.5.x):**

- LuaJIT plugin system with lifecycle hooks, declarative parameters, and
  fine-grained permission control, plus a package manager for remote
  repositories.
- Thread-safe audio processing, HDR and multi-pass Compute Shader effects, and
  transition effects.
- GPU compute optimizations: SRB pre-allocation, fine-grained dirty flags,
  separable blur, bitonic sort, and a BorderBlur O(n²) → O(2n) rewrite.
- End-to-end daily editing, missing-media recovery, atomic project saves,
  independent autosave snapshots, and startup crash recovery.
- A unified searchable object/effect catalog with package provenance.
- Large-timeline virtualization, continuous zoom, and repeatable performance
  workloads.
- Green release baselines across Linux, macOS, and Windows with version and tag
  consistency checks.

**Next (0.5.9+):**

- Run performance workloads against a maintained corpus of representative
  production media and third-party plugins.
- Continue workflow-level usability hardening with real editing projects.

**Long-term:**

- A full-featured video editor suitable as an AviUtl replacement.
- Potential rebranding (name and logo) to reflect the independent direction of
  the project.
- Continue monitoring the new NeoUtl and exploring cross-pollination
  opportunities between the projects.
