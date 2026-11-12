---
title: 常见问题
description: 关于 Freya NLE、其目标以及与 AviUtl 和 NeoUtl 关系的常见问题。
---

# 常见问题

> [!NOTE]
> 以下回答反映的是原作者（taisho-guy）的个人观点与开发经历。Freya NLE 相关回答反映的是维护者（[GT610](https://github.com/GT-610)）的观点。

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

Freya NLE 并非"重新发明 AviUtl"。虽然深受 AviUtl 影响，但内部实现完全不同。

| 项目 | Freya NLE | ExEdit0 | ExEdit2 |
| :--- | :--- | :--- | :--- |
| 核心技术 | Rust + Slint + wgpu | Win32 API | Win32 API |
| 并行处理模型 | GPU 合成，解码与导出多线程 | 单线程 | 多线程 |
| 内存空间 | 64位 | 32位（最大4GB） | 64位 |
| 预览渲染 | Vulkan / Metal / DX12 | GDI | DX11 |
| 音频引擎 | Carla（VST3/LV2等） | 仅内置功能 | 仅内置功能 |
| 插件系统 | Lua 5.1（沙箱）/ WGSL 效果包 | Lua / C++ | LuaJIT / C++ |
| 支持的操作系统 | Linux、Windows、macOS | Windows | Windows |

Freya NLE 从根本上解决结构性弱点：

1. **合成器跑在 GPU 上：** 特效链、混合和预览合成都是 wgpu 的 compute / render pass，开销随帧变化，而不是随 CPU 上的对象数量增长。
2. **用所有权代替手写内存管理：** 渲染器、解码器和导出器都是 Rust crate，除 FFmpeg 与 Carla 两处有文档说明的桥接外没有 `unsafe`，从结构上消除了促使移植的那类崩溃。
3. **UI 与渲染分离：** 即使在繁重的渲染过程中，时间线操作也不会受阻，高 DPI 环境下 UI 依然清晰。

## 名称和图标的由来？

项目在 0.8 之前叫 **AviQtl-Plus**：名称是"AviUtl"和"Qt"的合成词，图标是 Qt 和
AviUtl 标志的组合设计。

现在项目名为 **Freya NLE**，标志也随名称更新。胶片、切角底板和像素标记沿用原图标，
因此仍能看出是同一个项目；加号改为字母 **F**（"Plus" 已不再能描述这个项目），
配色由绿色改为浅蓝色。

## 可以使用 AviUtl 的插件吗？

不可以。由于机制不同，不兼容。也没有计划实现兼容层。

## 为什么要继续开发 Freya NLE？

原项目因 Qt Quick 的技术难题而暂停，但同时也证明了 **Qt + FFmpeg 可以快速实现高质量的视频编辑器原型**。作为早期的核心贡献者，维护者接过了原作者留下的位置，为实现最初的愿景继续开发：打造一款继承 AviUtl 操作体验并超越其性能的视频编辑软件。

真正值得保留的不是 Qt Quick，而是**编辑模型**——让 AviUtl 好用的"对象 + 特效"时间线。Freya NLE 把前端重建在 **Rust + Slint + wgpu** 上，0.8 之后这是唯一的构建。

## Freya NLE 与原项目有什么不同？

Freya NLE 与上游共享的是 AviUtl 的**编辑模型**，而非实现。它更注重：

- **渐进式交付：** 先让基本编辑功能可用，而非一开始就追求架构完美。
- **务实解决问题：** 改掉挡路的部分（前端与渲染器），而不是绕着它们走。
- **社区透明度：** 清晰记录 fork 关系、计划和长期意图。

路线图见[关于项目](../start-here/about)。
