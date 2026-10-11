---
title: 在无 GPU 环境下运行 Slint 前端
description: 无 GPU 的 Windows 主机能构建和运行什么、不能运行什么，以及 Slint 前端实际失败的位置。
---

# 在无 GPU 环境下运行 Slint 前端

本文内容于 2026-10-10 在一台仅有 WDDM 基本显示驱动程序的主机上逐条验证。下文所有结论都
在该主机上实际执行过，没有一条是照文档推断出来的。配套探针源码位于仓库根目录的
`no-gpu-probes/`。

## 这类主机有什么

- 只有一个显示适配器：`ELINK DISPLAY WDDM DRIVER`（WDDM 1.2 基本显示驱动程序）。它只是
  显示输出，不是 3D 设备。
- 因此 wgpu 只能解析出一个适配器：**Microsoft Basic Render Driver（Dx12，
  `DeviceType::Cpu`）**，即 WARP 软件光栅化器。

## 能正常工作的部分

以下每一层都是单独测量得到的结论：

| 层级 | 结果 |
| --- | --- |
| Slint 1.17 配合 `renderer-software` | 窗口创建成功，事件循环正常运行。不需要 GPU。 |
| WARP 上 wgpu 29 纹理上传 → 回读 | 0 字节不匹配。 |
| WARP 上 wgpu 29 + WGSL 管线 | 设备、着色器模块、渲染管线全部创建成功。 |
| WARP 上真实 winit 窗口 + surface + 渲染 pass + present | 成功呈现 5 帧。 |
| WARP 上 femtoVG `Canvas` + `WGPURenderer` + 字形图集 | 成功呈现 5 帧。 |

最后两项尤其值得注意：它们包含已配置的 wgpu surface、呈现循环，以及真实的文字渲染——
这些通常是被认为最可能先出错的部分。

## 不能正常工作的部分

`aviqtl-slint` 无法在这类主机上完成启动。按出现顺序有两个阻塞点：

1. `slint::BackendSelector::require_wgpu_29(...)` 拒绝选择任何后端，因为除非设置了
   `SLINT_WGPU_CPU`，`any_wgpu29_adapters_with_gpu` 只接受 `device_type != Cpu` 的适配器
   （`i-slint-core/src/graphics/wgpu_29.rs:152`）。
2. 设置 `SLINT_WGPU_CPU=1` 后，选择成功了，但进程在第一个窗口出现之前就以 `0xC0000005`
   退出，崩溃位置在 `d3dcompiler_47.dll`。

阻塞点 2 **是既有问题，并非本次检出独有**：这些探针出现之前构建的
`dist/AviQtl-MSVC-x86_64.zip` 也以同样方式失败，并且 2026 年 9 月的代码审计已经记录过同一
次崩溃，同时说明这两次尝试都不能算作成功的 GUI 验证。

由于上面每一层单独都能工作，阻塞点 2 的成因目前**尚未查明**。它位于 Slint 的 `Manual` wgpu
配置如何把这些层组合起来——应用正是用它来让预览、导出 surface 和 Slint 窗口共享同一个
wgpu 设备的。除此之外，任何更具体的归因都没有证据支持。此前有两个假设，分别认为问题出在
femtoVG 或 WARP 本身，都经过实测并被推翻。

## 复现构建过程

有两个容易忽略的前提条件：

**构建需要 FFmpeg。** Cargo 需要把 `FFMPEG_DIR`、`PKG_CONFIG` 和 `LIBCLANG_PATH` 指向
一个 vcpkg 目录树。缺少其中任何一个，构建就会在 `ffmpeg-sys-the-third` 的构建脚本里失败。

```powershell
$env:VCPKG_ROOT        = "D:\Software\vcpkg"
$env:VCPKGRS_TRIPLET   = "x64-windows"
$env:FFMPEG_DIR        = "<repo>\.build_tmp\msvc\Release\vcpkg_installed\x64-windows"
$env:PKG_CONFIG        = "$env:FFMPEG_DIR\tools\pkgconf\pkgconf.exe"
$env:PKG_CONFIG_PATH   = "$env:FFMPEG_DIR\lib\pkgconfig"
$env:LIBCLANG_PATH     = "D:\Software\vcpkg\downloads\tools\clang\clang-21.1.8\bin"
cargo build -p aviqtl-slint
```

**FFmpeg 运行时 DLL。** 可执行文件链接了 `avutil-61`、`avcodec-63`、`avformat-63`、
`swscale-10`、`swresample-7` 和 `libx264-164`。它们不在 `PATH` 中，所以除非把它们放在
可执行文件旁边，进程会以 `0xC0000135` 中止：

```powershell
Copy-Item "$env:FFMPEG_DIR\debug\bin\*.dll" rust\target\debug\ -Force
```

`BUILD.py` 在打包时会执行等价的步骤。

## 运行

```powershell
rust\target\debug\aviqtl-slint.exe              # 需要 GPU 后端适配器
rust\target\debug\aviqtl-slint.exe --validate-gpu   # 渲染自检，然后退出
```

应用中并没有软件渲染的逃生开关。曾经做过一个原型，后来移除了：它只能清除阻塞点 1，
清不掉阻塞点 2，因此没有提供任何可用的运行路径，却额外引入了一个需要维护的参数和一处
`unsafe_code` 破例。它暴露出来的信息——启动行上的适配器名称、后端和 `device_type`——
在日后真要修复时值得保留。

## 让这类主机跑起前端

阻塞点 2 需要下面二者之一：

- 一块真正的 3D 适配器，也就是本项目其他目标机器都有条件。
- 改变与 Slint 共享 GPU 资源的方式。把 `WGPUConfiguration::Manual` 换成 `Automatic` 可以
  让 Slint 自行协商 surface，但预览和导出 surface 建立在"与应用共享同一个 wgpu 设备"
  这一前提上，而 `GpuValidation` 正在显式断言这件事。那是一个比加个参数大得多的改动。

对于没有 GPU 的主机，仍然可以验证的是 Cargo 各项检查和该 crate 的单元测试。原生桌面验收
留给有 GPU 的机器——迁移状态文档里也是这样记录的。
