---
title: Running the Slint frontend without a GPU
description: What a GPU-less Windows host can and cannot build and run, and where the Slint frontend actually fails.
---

# Running the Slint frontend without a GPU

Verified on 2026-10-10 on a host whose only display adapter is a WDDM basic display
driver. Every statement below was executed on that host; nothing is inferred from
documentation. Companion probe sources live in `no-gpu-probes/` at the repository root.

## What such a host has

- A single display adapter: `ELINK DISPLAY WDDM DRIVER` (WDDM 1.2 basic display driver).
  It is a display output, not a 3D device.
- wgpu therefore resolves exactly one adapter: **Microsoft Basic Render Driver (Dx12,
  `DeviceType::Cpu`)** — the WARP software rasterizer.

## What works

Each layer, measured in isolation:

| Layer | Result |
| --- | --- |
| Slint 1.17 with `renderer-software` | Window created, event loop ran. No GPU needed. |
| wgpu 29 texture upload → read-back on WARP | 0 mismatched bytes |
| wgpu 29 + WGSL pipeline on WARP | Device, shader module, and pipeline all created |
| Real winit window + surface + render pass + present on WARP | 5 frames presented |
| femtoVG `Canvas` + `WGPURenderer` + glyph atlas on WARP | 5 frames presented |

The last two matter most: they include a configured wgpu surface, a presentation loop, and
real text rendering, which is what one would expect to fail first.

## What does not work

`aviqtl-slint` cannot complete startup on such a host. Two blockers appear in order:

1. `slint::BackendSelector::require_wgpu_29(...)` refuses to select any backend, because
   `any_wgpu29_adapters_with_gpu` only accepts an adapter whose `device_type != Cpu` unless
   `SLINT_WGPU_CPU` is set
   (`i-slint-core/src/graphics/wgpu_29.rs:152`).
2. With `SLINT_WGPU_CPU=1`, selection succeeds, but the process exits with `0xC0000005`
   inside `d3dcompiler_47.dll` before the first window appears.

Blocker 2 is **pre-existing and not specific to this checkout**: the packaged
`dist/AviQtl-MSVC-x86_64.zip` built before these probes fails identically, and the
September 2026 code audit already recorded the same crash while stating that neither
attempt counted as successful GUI validation.

Because every layer above works on its own, the cause of blocker 2 is currently
**unidentified**. It lies in how Slint's `Manual` wgpu configuration — used by the
application to share one wgpu device between the preview, the export surfaces, and the Slint
windows — combines these layers. No attribution beyond that is supported by evidence. Two
earlier hypotheses, that femtoVG or WARP itself was at fault, were both measured and
disproved.

## Reproducing a build

Two prerequisites that are easy to miss:

**FFmpeg for the build.** Cargo needs `FFMPEG_DIR`, `PKG_CONFIG` and `LIBCLANG_PATH`
pointing at a vcpkg tree. Without them the build fails inside the `ffmpeg-sys-the-third`
build script.

```powershell
$env:VCPKG_ROOT        = "D:\Software\vcpkg"
$env:VCPKGRS_TRIPLET   = "x64-windows"
$env:FFMPEG_DIR        = "<repo>\.build_tmp\msvc\Release\vcpkg_installed\x64-windows"
$env:PKG_CONFIG        = "$env:FFMPEG_DIR\tools\pkgconf\pkgconf.exe"
$env:PKG_CONFIG_PATH   = "$env:FFMPEG_DIR\lib\pkgconfig"
$env:LIBCLANG_PATH     = "D:\Software\vcpkg\downloads\tools\clang\clang-21.1.8\bin"
cargo build -p aviqtl-slint
```

**FFmpeg runtime DLLs.** The executable links `avutil-61`, `avcodec-63`, `avformat-63`,
`swscale-10`, `swresample-7`, and `libx264-164`. These are not on `PATH`, so the process
aborts with `0xC0000135` unless they sit beside the binary:

```powershell
Copy-Item "$env:FFMPEG_DIR\debug\bin\*.dll" rust\target\debug\ -Force
```

`BUILD.py` performs the equivalent step when packaging.

## Running

```powershell
rust\target\debug\aviqtl-slint.exe              # requires a GPU-backed adapter
rust\target\debug\aviqtl-slint.exe --validate-gpu   # render self-check, then exit
```

There is no software-rendering escape hatch in the application. One was prototyped and
removed: it cleared blocker 1 but not blocker 2, so it supplied no working path while adding
a maintained flag and an `unsafe_code` exception. The information it surfaced — adapter name,
backend, and `device_type` on the startup line — is worth preserving if a real fix is
attempted later.

## Getting the frontend up on such a host

Blocker 2 needs one of:

- A real 3D adapter, which is what every machine the project otherwise targets has.
- A change to how GPU resources are shared with Slint. Switching `WGPUConfiguration::Manual`
  to `Automatic` would let Slint negotiate its own surface, but the preview and export
  surfaces are built on the assumption that they share the application's wgpu device — an
  assumption `GpuValidation` asserts explicitly. That is a substantially larger change than a
  flag.

For hosts without a GPU, what remains verifiable is the Cargo checks and the crate's unit
tests. Native desktop acceptance stays with the machines that have a GPU, as the migration
status already records.
