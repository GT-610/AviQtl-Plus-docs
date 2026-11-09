---
title: 发布检查清单
description: 创建 Freya NLE 发布版本的要求。
---

# 发布检查清单

在创建 Freya NLE 发布标签之前，请使用此清单。只有当被标记的提交在干净的构建环境中通过以下检查时，发布才是有效的。被跳过的检查项，或用先前提交遗留的构建结果，都不能作为证据。

## 版本

- `rust/Cargo.toml` 中的 `[workspace.package] version` 是项目版本的唯一来源。`BUILD.py` 从该处读取版本，发布工作流会拒绝不等于 `v<version>` 的标签。
- 预发布版本使用带点的计数器形式：版本 `0.8.0-rc.1`，标签 `v0.8.0-rc.1`。不要使用 `-rc1` 或 `-rc-1`，详见[版本与发布](https://github.com/GT-610/Freya-NLE/blob/main/CONTRIBUTING.md#versions-and-releases)。
- macOS 应用包的元数据携带同一版本的数值主干（`0.8.0-rc.1` 对应 `0.8.0`），因为应用包格式无法表达后缀。该值由 `BUILD.py` 从工作区版本推导，没有第二处需要修改。
- `CHANGELOG.md` 中存在本次打标签版本对应的 `## [<version>] - <YYYY-MM-DD>` 章节，且 `Unreleased` 为空。
- `python -m unittest tests.test_build_version` 通过；当版本号、标签形式与变更日志互相不一致时，该测试会失败。
- 确认打包后的"关于"对话框显示工作区版本。

## Rust 前端

Rust + Slint + wgpu 是唯一的前端。请在干净的 `rust/target` 下运行：

```text
cargo fmt --manifest-path rust/Cargo.toml --all -- --check
cargo clippy --manifest-path rust/Cargo.toml --locked --workspace --all-targets --all-features -- -D warnings
cargo test --manifest-path rust/Cargo.toml --locked --workspace --all-features
cargo test --manifest-path rust/Cargo.toml --locked -p freya-nle-media -p freya-nle-audio -- --ignored
python -m unittest tests.test_build_version
```

`rust-toolchain.toml` 锁定的工具链必须是 `PATH` 上生效的那一个。系统包管理器安装的更新版 `cargo` 会优先于 `rustup` 的中间层，从而把锁定版本中的弃用告警变成失败；因此在采信红或绿的结果之前，先检查 `cargo --version`。

必需平台：

- Linux 运行器或 Arch 容器
- macOS 运行器
- Windows MSVC 运行器（Windows 默认构建环境）

MSYS2 是显式选择的替代方案，并不是发布版本验证所用的 Windows 构建环境。

带 `--ignored` 的测试需要 GPU 适配器、支持 libx264 与 AAC 的 FFmpeg 命令行工具，或已安装的 Carla 库。请记录每个测试实际运行的平台，并把未能运行的测试记为"未验证"而非"通过"。

## 打包与冒烟测试

- 用 `python3 BUILD.py` 为每个平台打包（Windows 使用 `--msvc`）。
- 确认归档中包含可执行文件、`effects`、`objects`、`plugins`、`effect-packages`、`repos` 与翻译文件。`BUILD.py` 会装配 `catalog/effects`、`catalog/objects`、`plugins/`、`effect-packages/` 与 `repos/`；归档中不得出现任何 `Qt6*` 运行时文件。
- `effect-packages/` 下每个随包分发的包，其 `min_app_version` 都不得高于本次发布的版本，否则包管理器会拒绝安装它。
- 用 `--validate-gpu` 启动一次打包后的程序，确认报告的图形后端、帧计数以及为空的 `wgpu_errors` 列表。
- 在仅有软件渲染适配器的主机上，确认启动时报告主机不受支持，而不是崩溃。
- 启动打包后的应用，创建项目，保存并重新打开，渲染一个短视频，并在编辑器外验证输出。
- 在没有构建工具链的机器或干净的虚拟机上验证安装和启动。

## 发布

- 确认发布提交的所有必需 GitHub Actions 任务均为绿色。
- 仅在以上检查通过后创建版本标签 `v<version>`。包含 `-` 的标签会发布为 GitHub 预发布版本，并且不会取代最新的稳定版本。
- 在宣布发布之前，下载并对标签工作流生成的构件进行冒烟测试。
