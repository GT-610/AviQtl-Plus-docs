---
title: 关于项目
description: AviQtl-Plus 是什么、它的历史，以及与上游 NeoUtl 项目的关系。
---

# 关于项目

## 什么是 AviQtl-Plus？

AviQtl-Plus 是一款自由、开源的视频编辑软件，继承 **AviUtl 1.10** 与 **ExEdit 0.92** 的操作体验，同时拥有**超越 AviUtl 的性能**。它是一款跨平台桌面应用。前端正在从 Qt Quick + QRhi 向 **Rust + Slint + wgpu** 迁移——Slint 为默认构建（建设中）——底层是 ECS 架构核心，以及时间线、特效、音频、渲染、媒体共享的 Rust 工作区。

其核心优势包括：

- 与 AviUtl 极为相似的用户界面。
- 使用 GPU 实现的**高速强大特效**。
- 通过 Carla 支持 VST3、LV2 等**音频特效**。
- **LuaJIT 插件系统**，支持包管理、声明式参数和权限控制。
- 支持 **Linux**、**Windows**、**macOS** 三大平台。

[AviUtl 操作体验目标](../../developer/operability-targets) 描述了指导开发的编辑模型兼容目标。

## 项目历史

本仓库是 [taisho-guy/NeoUtl](https://codeberg.org/taisho-guy/NeoUtl) 的 fork。2026 年 5 月底，原作者决定**重置开发**——现在的 `main` 分支已更换为基于 **Rust + Slint + wgpu** 的新 NeoUtl，而原 Qt Quick 版本已移至 `aviqtl` 分支（称为 **AviQtl**）。

因此，目前有三个 NeoUtl 相关项目并行存在：

- **[NeoUtl](https://codeberg.org/taisho-guy/NeoUtl)** — 使用 Rust + Slint + wgpu 重建的原项目。
- **[AviQtl](https://codeberg.org/taisho-guy/NeoUtl/src/branch/aviqtl)** — 原 Qt Quick 版本，继续作为 "AviQtl"。
- **AviQtl-Plus（本项目）** — 继续发展 Qt Quick + QRhi + ECS 路线的 fork，
  目前前端正在向 Rust + Slint + wgpu 迁移。

## 前端迁移

默认的 `BUILD.py` 构建已切换为 **Rust + Slint 前端（建设中）**。迁移期间，旧 Qt
Quick 前端仍可用于开发：

```bash
python3 BUILD.py --xcode --frontend qt --debug
```

Qt 构建为未部署的开发二进制；详见[从源码构建](../../developer/building#前端选择)。

### 原项目暂停开发的原因

原作者发现了以下 Qt Quick 的根本性问题：

- Qt Quick 独有的资源管理与 Compute Shader 不兼容，导致实现困难。
- 与 ECS 架构不兼容，优化困难。
- 实时处理存在性能隐患。

原作者此后转向基于 **Rust + Slint + wgpu** 的新技术栈，放弃了最初规划的 Qt Widgets + SDL3 + bgfx 方案。

### AviQtl-Plus 的应对方案

基于原项目获得的经验，AviQtl-Plus 将着力解决这些挑战：

1. **Compute Shader 集成** — 绕过 Qt Quick 的限制，利用 GPU Compute 实现高性能特效处理。
2. **ECS 架构优化** — 最大化数据驱动设计的优势，同时消除与 Qt Quick 的摩擦。
3. **实时性能提升** — 消除预览渲染瓶颈，提供流畅的编辑体验。

愿景不变：**打造一款继承并超越 AviUtl 的视频编辑软件**。我们相信 Qt Quick + QRhi + ECS 是一条可行且有价值的路线——我们将致力于将其打磨为实用、日常可用的编辑工具。

### 与新版 NeoUtl 的关系

原作者与 AviQtl-Plus 维护者保持着友好关系。如果 AviQtl-Plus 的某些贡献对新的 NeoUtl（Rust + Slint + wgpu）适用，会很乐意向上游提交。同样，两个项目的成果有望随着时间的推移相互滋养，最终造福所有寻求现代化、强大且直观的 AviUtl 替代品的用户。

旧 Qt Quick 源代码仍在 [`aviqtl` 分支](https://codeberg.org/taisho-guy/NeoUtl/src/branch/aviqtl) 可用，原作者也向喜欢 Qt Quick 路线的用户推荐 AviQtl-Plus。

## 路线图

本项目完全由个人动力驱动，没有截止日期或商业压力。进展稳定但节奏适中。

**已完成（0.3.x–0.5.x）：**

- LuaJIT 插件系统：生命周期钩子、声明式参数、细粒度权限控制，以及面向远程仓库的包管理器。
- 线程安全的音频处理、HDR 与多 pass Compute Shader 特效、转场特效。
- GPU Compute 优化：SRB 预分配、细粒度脏标志、可分离模糊、Bitonic 排序，以及 BorderBlur O(n²) → O(2n) 重写。
- 端到端的日常编辑、缺失媒体恢复、原子化项目保存、独立自动保存快照与启动崩溃恢复。
- 带包来源的统一可搜索对象/效果目录。
- 大型时间线虚拟化、连续缩放与可重复的性能测试工作负载。
- Linux、macOS、Windows 三平台的绿色发布基线，以及版本与标签一致性检查。

**下一步（0.5.9+）：**

- 使用有代表性的生产媒体和第三方插件语料运行性能测试工作负载。
- 用真实编辑项目继续加强工作流层面的可用性。

**长期：**

- 成为足以替代 AviUtl 的全功能视频编辑器。
- 考虑更名和品牌重塑（名称与 Logo），以反映项目独立的发展方向。
- 持续关注新版 NeoUtl 的进展，探索两个项目之间相互借鉴的机会。
