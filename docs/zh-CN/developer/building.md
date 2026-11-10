---
title: 从源码构建
description: 在 Linux、macOS 或 Windows 上配置并构建 Freya NLE。
---

# 从源码构建

克隆应用仓库。只有需要在本地阅读或修改本站时，才需要同时获取文档 submodule：

```bash
git clone --recurse-submodules https://github.com/GT-610/Freya-NLE.git
cd Freya-NLE
```

只需要应用源码时，可以省略 `--recurse-submodules`。

## 自动构建

`BUILD.py` 会检测主机平台、准备依赖、配置项目并进行构建。通常使用：

```bash
python3 BUILD.py
```

构建的是 **Rust + Slint 前端**；自 0.8 移除 Qt 前端以来，这是唯一的前端。不再有 `--frontend` 与 `--qt-dir` 参数，也没有需要配置的 CMake 构建。

常用平台模式包括：

```bash
# Linux 容器构建环境
python3 BUILD.py --arch

# macOS Xcode 生成器
python3 BUILD.py --xcode

# Windows 默认使用 MSVC 工具链
python3 BUILD.py

# 显式选择 MSYS2 作为替代方案
python3 BUILD.py --msys2
```

MSVC 构建仅支持 Visual Studio 2022（17.x）及更高版本。Visual Studio 2019
及更早版本不受支持。`BUILD.py` 会通过 `vswhere` 选择受支持的 Visual
Studio 安装，并拒绝旧工具集。构建还需要 vcpkg。当前参数请查看
`python3 BUILD.py --help`。

## 平台说明

### Linux

在 Linux 上，默认使用 distrobox/podman 容器隔离构建环境。

1. **安装依赖**
   - Pacman: `sudo pacman -S --needed distrobox podman python git`
   - APT: `sudo apt install distrobox podman python3 git`
   - DNF: `sudo dnf install distrobox podman python3 git`
2. **构建**
   - `python3 BUILD.py --arch`
3. **运行**
   - `./build/freya-nle`

容器还会安装 LuaJIT、Vulkan 实现（例如 Mesa）、FFmpeg、Carla 和 clang（提供 libc++）。

### macOS

在 macOS 上，`BUILD.py` 通过 Homebrew 检查并安装依赖（pkgconf、FFmpeg 等），然后生成并签名 `.app` 包。

1. **安装依赖**
   - `brew install python git`
2. **构建**
   - `python3 BUILD.py --xcode`
3. **运行**
   - `open "./build/Freya NLE.app"`

### Windows (MSVC，默认)

Rust 在 Windows 上需要 MSVC 主机工具链，因此不带目标参数的
`python3 BUILD.py` 会选择 MSVC 构建器。对于提供匹配 GNU Rust
目标和依赖的环境，仍可显式使用 MSYS2。

1. **额外准备**
   - Visual Studio 2022 或更高版本的 Build Tools、C++ x64/x86 工具集和 Windows SDK
   - vcpkg（可通过 `VCPKG_ROOT` 环境变量指定；如未找到，`BUILD.py` 将尝试自动获取）
2. **构建**
   - `python3 BUILD.py`
   - 如需显式选择 MSYS2，请在 MSYS2 UCRT64 shell 中运行 `python3 BUILD.py --msys2`。
3. **运行**
   - `.\build\freya-nle.exe`

### Windows (MSYS2，显式替代方案)

1. **安装依赖**
   - `pacman -S git mingw-w64-ucrt-x86_64-python`
2. **构建**
   - `python3 BUILD.py --msys2`
3. **运行**
   - `./build/freya-nle.exe`

## 工具链

Rust 版本锁定在 `rust-toolchain.toml` 中，CI 使用
`dtolnay/rust-toolchain@1.99.1`。请确认 `PATH` 上生效的是锁定版本的
工具链：系统包管理器安装的更新版 `cargo` 会优先于 `rustup` 的中间层，
从而把弃用告警变成构建失败。在采信结果之前先检查 `cargo --version`。

## 测试与质量检查

在仓库根目录运行检查：

```bash
cargo fmt --manifest-path rust/Cargo.toml --all -- --check
cargo clippy --manifest-path rust/Cargo.toml --locked --workspace --all-targets --all-features -- -D warnings
cargo test --manifest-path rust/Cargo.toml --locked --workspace --all-features
python -m unittest tests.test_build_version
```

需要 GPU 适配器、FFmpeg 命令行工具或已安装 Carla 库的测试默认标记为
`#[ignore]`；详见[添加测试](https://github.com/GT-610/Freya-NLE/blob/main/CONTRIBUTING.md#adding-tests)。
工具被跳过意味着缺少检查覆盖，而不是检查成功。
