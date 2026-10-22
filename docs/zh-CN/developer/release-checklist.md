---
title: 发布检查清单
description: 创建 AviQtl-Plus 发布版本的要求。
---

# 发布检查清单

在创建 AviQtl-Plus 发布标签之前，请使用此清单。只有当被标记的提交在干净的构建环境中通过以下检查时，发布才是有效的。被跳过的检查项，或用先前提交遗留的构建结果，都不能作为证据。

## 版本

- `rust/Cargo.toml` 中的 `[workspace.package] version` 是项目版本的唯一来源。`BUILD.py` 从该处读取版本，发布工作流会拒绝不等于 `v<version>` 的标签。
- 预发布版本使用带点的计数器形式：版本 `0.7.0-rc.1`，标签 `v0.7.0-rc.1`。不要使用 `-rc1` 或 `-rc-1`，详见[版本与发布](https://github.com/GT-610/AviQtl-Plus/blob/main/CONTRIBUTING.md#versions-and-releases)。
- 由于 CMake 无法表达预发布后缀，`CMakeLists.txt` 中的 `project(AviQtl VERSION ...)` 携带同一版本的数值主干（`0.7.0-rc.1` 对应 `0.7.0`）。macOS 应用包的元数据正是由该数值主干写入的。
- `CHANGELOG.md` 中存在本次打标签版本对应的 `## [<version>] - <YYYY-MM-DD>` 章节，且 `Unreleased` 为空。
- `python -m unittest tests.test_build_version` 通过；当版本号、标签形式、`CMakeLists.txt` 与变更日志互相不一致时，该测试会失败。
- 确认打包后的"关于"对话框显示工作区版本。

## Rust 前端

默认发布的即是 Rust + Slint + wgpu 前端。请在干净的 `rust/target` 下运行：

```text
cargo fmt --manifest-path rust/Cargo.toml --all -- --check
cargo clippy --manifest-path rust/Cargo.toml --locked --workspace --all-targets --all-features -- -D warnings
cargo test --manifest-path rust/Cargo.toml --locked --workspace --all-features
cargo test --manifest-path rust/Cargo.toml --locked -p aviqtl-media -p aviqtl-audio -- --ignored
python -m unittest tests.test_build_version
python tests/test_rust_abi_contract.py
python tests/check_rust_abi.py --rust-src rust/aviqtl-core/src --header core/include/rust_core_abi.hpp
```

必需平台：

- Linux 运行器或 Arch 容器
- macOS 运行器
- Windows MSVC 运行器（Windows 默认构建环境）

MSYS2 是显式选择的替代方案，并不是发布版本验证所用的 Windows 构建环境。

带 `--ignored` 的测试需要 GPU 适配器、支持 libx264 与 AAC 的 FFmpeg 命令行工具，或已安装的 Carla 库。请记录每个测试实际运行的平台，并把未能运行的测试记为"未验证"而非"通过"。

## Qt 参照前端

Qt/C++ 与 QML 仅作为行为参照保留，不进入发布归档。当改动涉及共享的 Rust 代码或 C++ 适配层时，请在打标签前构建并运行 CTest：

```text
cmake -S . -B .build_tmp/qt-release -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build .build_tmp/qt-release --parallel 2
ctest --test-dir .build_tmp/qt-release --output-on-failure
```

## 打包与冒烟测试

- 用 `python3 BUILD.py` 为每个平台打包（Windows 使用 `--msvc`）。
- 确认归档中包含可执行文件、`effects`、`objects`、`plugins`、`effect-packages`、`repos`、翻译文件，以及所需的 FFmpeg 和 Carla 运行时库，并且不包含任何 `Qt6*` 运行时文件。
- 用 `--validate-gpu` 启动一次打包后的程序，确认报告的图形后端、帧计数以及为空的 `wgpu_errors` 列表。
- 在仅有软件渲染适配器的主机上，确认启动时报告主机不受支持，而不是崩溃。
- 启动打包后的应用，创建项目，保存并重新打开，渲染一个短视频，并在编辑器外验证输出。
- 在没有构建工具链的机器或干净的虚拟机上验证安装和启动。

## 发布

- 确认发布提交的所有必需 GitHub Actions 任务均为绿色。
- 仅在以上检查通过后创建版本标签 `v<version>`。包含 `-` 的标签会发布为 GitHub 预发布版本，并且不会取代最新的稳定版本。
- 在宣布发布之前，下载并对标签工作流生成的构件进行冒烟测试。
