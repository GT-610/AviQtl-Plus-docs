---
title: 从源码构建
description: 在 Linux、macOS 或 Windows 上配置并构建 AviQtl-Plus。
---

# 从源码构建

克隆应用仓库。只有需要在本地阅读或修改本站时，才需要同时获取文档 submodule：

```bash
git clone --recurse-submodules https://github.com/GT-610/AviQtl-Plus.git
cd AviQtl-Plus
```

只需要应用源码时，可以省略 `--recurse-submodules`。

## 自动构建

`BUILD.py` 会检测主机平台、准备依赖、配置项目并进行构建。通常使用：

```bash
python3 BUILD.py
```

### 前端选择

`BUILD.py` 默认构建 **Rust + Slint 前端（建设中）**。迁移期间，旧 C++/QML（Qt）前端仍然可用：

```bash
# 旧 Qt 前端（debug 示例）
python3 BUILD.py --xcode --frontend qt --debug

# 指定 Qt6 安装路径
python3 BUILD.py --xcode --frontend qt --qt-dir <Qt6 路径>
```

Qt 构建经 CMake（Ninja）输出到 `.build_tmp/<target>/<Config>/qt-build`，再把原始 Qt 产物（macOS 为 `AviQtl.app` 包，其余平台为 `AviQtl` 二进制）连同共享资源装配到 `build/`，归档名带 `-Qt` 后缀。这些是开发构建：有意跳过平台部署（`macdeployqt`/`windeployqt`），分发 Qt 构建前请先运行对应的部署工具。`--qt-dir` 仅对 Qt 构建有效；省略时通过 `PATH` 中的 `qmake6`/`qmake` 检测。

常用平台模式包括：

```bash
# Linux 容器构建环境
python3 BUILD.py --arch

# macOS Xcode 生成器
python3 BUILD.py --xcode

# Windows MSYS2
python3 BUILD.py --msys2
```

MSVC 构建仅支持 Visual Studio 2022（17.x）及更高版本。Visual Studio 2019
及更早版本不受支持。`BUILD.py` 会通过 `vswhere` 选择受支持的 Visual
Studio 安装，并拒绝旧工具集。使用旧 Qt 前端时还需要匹配的 Qt MSVC
安装和 vcpkg。当前参数请查看 `python3 BUILD.py --help`。

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
   - `./build/AviQtl`

容器还会安装 Qt 6、LuaJIT、Vulkan 实现（例如 Mesa）、FFmpeg、Carla 和 clang（提供 libc++）。

### macOS

在 macOS 上，`BUILD.py` 通过 Homebrew 检查并安装依赖（CMake、Ninja、Qt6 等），然后执行 `macdeployqt` 和 `codesign` 创建 `.app` 包。

1. **安装依赖**
   - `brew install python git`
2. **构建**
   - `python3 BUILD.py --xcode`
3. **运行**
   - `open ./build/AviQtl.app`

### Windows (MSYS2)

1. **安装依赖**
   - `pacman -S git mingw-w64-ucrt-x86_64-python`
2. **构建**
   - `python3 BUILD.py --msys2`
3. **运行**
   - `./build/AviQtl.exe`

### Windows (MSVC)

MSVC 构建仅支持 Visual Studio 2022（17.x）及更高版本。Visual Studio 2019
及更早版本不能用于构建本项目。FFmpeg 9.0.1 由仓库内的 vcpkg overlay
负责构建。

1. **额外准备**
   - Visual Studio 2022 或更高版本的 Build Tools、C++ x64/x86 工具集和 Windows SDK
   - 官方 Qt 的 MSVC x64 版本（例如 `msvc2022_64`）
   - vcpkg（可通过 `VCPKG_ROOT` 环境变量指定；如未找到，`BUILD.py` 将尝试自动获取）
   - vcpkg 必须能够获取其 Clang 工具；`BUILD.py` 会为 Rust 的 FFmpeg 绑定自动设置 `LIBCLANG_PATH`
2. **构建**
   - `python3 BUILD.py --msvc`
   - 使用旧 Qt 前端时，增加 `--frontend qt --qt-dir <Qt 安装目录>`；如省略
     `--qt-dir`，将通过 `PATH` 中的 `qmake6`/`qmake` 检测 Qt。
3. **运行**
   - `.\build\AviQtl.exe`

## Qt 版本兼容性

项目目前使用 Qt 私有模块处理 ZIP、QRhi 集成和着色器工具。构建时与运行时的 Qt 补丁版本必须一致；升级 Qt 包后需要重新构建。

## 测试与质量检查

从已配置的构建目录运行测试：

```bash
ctest --test-dir build --output-on-failure
```

仓库还提供 `check.py` 执行格式、静态分析和其他质量检查。工具被跳过意味着缺少检查覆盖，而不是检查成功。
