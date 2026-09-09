---
title: 常见问题
description: 关于 AviQtl-Plus、其目标以及与 AviUtl 和 NeoUtl 关系的常见问题。
---

# 常见问题

> [!NOTE]
> 以下回答反映的是原作者（taisho-guy）的个人观点与开发经历。AviQtl-Plus 相关回答反映的是维护者（[GT610](https://github.com/GT-610)）的观点。

## 开发的契机是什么？

### 操作系统的壁垒
起因是 AviUtl 无法在 Linux 上运行。**仅为 AviUtl 而维护 Windows 环境**是难以接受的。

### 膨胀的生态系统
无论出于何种原因，不少人"不得不"继续使用 AviUtl。经过长年扩展而变得臃肿的生态系统如同"哈尔的移动城堡"，即使心怀不满也难以割舍。

### 项目目标与使命
在[鹿儿岛县立甲南高等学校](https://edunet002.synapse-blog.jp/konan/)的课题研究中，为解决这一问题，原作者决定独立开发 NeoUtl。

- **个人目标：** 无需在 Domino、VocalShifter、REAPER、AviUtl 之间来回切换，仅用 Linux 上的 NeoUtl 即可制作音 MAD。
- **NeoUtl 的使命：** 成为那些"不得不"使用 AviUtl 的人的最佳解决方案。

## 为什么要开发 AviUtl 的克隆？

AviQtl-Plus 并非"重新发明 AviUtl"。虽然深受 AviUtl 影响，但内部实现完全不同。

| 项目 | AviQtl-Plus | ExEdit0 | ExEdit2 |
| :--- | :--- | :--- | :--- |
| 核心技术 | Rust + Slint（Qt6 旧版） | Win32 API | Win32 API |
| 并行处理模型 | 数据驱动（ECS） | 单线程 | 多线程 |
| 内存空间 | 64位 | 32位（最大4GB） | 64位 |
| 预览渲染 | Vulkan / Metal / DX12 | GDI | DX11 |
| 音频引擎 | Carla（VST3/LV2等） | 仅内置功能 | 仅内置功能 |
| 插件系统 | LuaJIT / C++ / QML / GLSL | Lua / C++ | LuaJIT / C++ |
| 支持的操作系统 | Linux、Windows、macOS | Windows | Windows |

AviQtl-Plus 从根本上解决结构性弱点：

1. **基于 ECS（实体组件系统）的数据导向设计：** 极大提升 CPU 缓存效率，加速大量对象的处理。
2. **现代化内存管理：** 采用 C++23 智能指针，从结构上最小化原因不明的崩溃。
3. **UI 与渲染分离：** 即使在繁重的渲染过程中，时间线操作也不会受阻，高 DPI 环境下 UI 依然清晰。

## 名称和图标的由来？

名称是"AviUtl"和"Qt"的合成词。
图标是 Qt 和 AviUtl 标志的组合设计。

## 可以使用 AviUtl 的插件吗？

不可以。由于机制不同，不兼容。也没有计划实现兼容层。

## 为什么要继续开发 AviQtl-Plus？

原项目因 Qt Quick 的技术难题而暂停，但同时也证明了 **Qt + FFmpeg 可以快速实现高质量的视频编辑器原型**。项目架构设计优秀，基础扎实。尤其现在 QRhi 提供了可行的 Compute Shader 路径，Qt Quick 路线仍然值得继续探索。

作为早期的核心贡献者，维护者亲眼目睹了这个项目的潜力，并接过了原作者留下的开发位置——不仅是为了让项目继续存活，更是为了实现最初的愿景：打造一款继承 AviUtl 操作体验并超越其性能的视频编辑软件。

## AviQtl-Plus 与原项目有什么不同？

技术方向大致相同（Qt Quick + QRhi + ECS），但 AviQtl-Plus 更注重：

- **渐进式交付：** 先让基本编辑功能可用，而非一开始就追求架构完美。
- **务实解决问题：** 在 Qt Quick 的约束范围内寻找最优解，而非将其视为障碍。
- **社区透明度：** 清晰记录 fork 关系、计划和长期意图。

项目达到可用状态后，可能会进行更名和品牌重塑，以明确与原始项目的区分。路线图见[关于项目](../start-here/about)。
