---
title: Build from source
description: Configure and build Freya NLE on Linux, macOS, or Windows.
---

# Build from source

Clone the application repository, including the documentation submodule only if
you want a local copy of this site:

```bash
git clone --recurse-submodules https://github.com/GT-610/Freya-NLE.git
cd Freya-NLE
```

For a source-only checkout, omit `--recurse-submodules`.

## Automated build

`BUILD.py` detects the host platform, prepares dependencies, configures the
project, and builds it. The normal entry point is:

```bash
python3 BUILD.py
```

This builds the **Rust + Slint frontend**, the only frontend since the Qt Quick
frontend was retired in 0.8. There is no `--frontend` flag, no `--qt-dir`, and no
CMake build to configure.

Common platform-specific modes include:

```bash
# Linux container-based environment
python3 BUILD.py --arch

# macOS Xcode generator
python3 BUILD.py --xcode

# Windows with the default MSVC toolchain
python3 BUILD.py

# Windows with MSYS2 (explicit alternative)
python3 BUILD.py --msys2
```

MSVC builds require Visual Studio 2022 (17.x) or newer. Visual Studio 2019
and earlier are not supported. `BUILD.py` selects a supported Visual Studio
installation through `vswhere` and rejects older toolsets. The build also
requires vcpkg. See `python3 BUILD.py --help` for the current options.

## Platform notes

### Linux

On Linux, the build environment is isolated using a distrobox/podman container
by default.

1. **Install dependencies**
   - Pacman: `sudo pacman -S --needed distrobox podman python git`
   - APT: `sudo apt install distrobox podman python3 git`
   - DNF: `sudo dnf install distrobox podman python3 git`
2. **Build**
   - `python3 BUILD.py --arch`
3. **Run**
   - `./build/Freya NLE`

The container also installs LuaJIT, a Vulkan implementation (for example
Mesa), FFmpeg, Carla, and clang (which provides libc++).

### macOS

On macOS, `BUILD.py` checks and installs dependencies via Homebrew (pkgconf,
FFmpeg, etc.), then produces and codesigns a `.app` bundle.

1. **Install dependencies**
   - `brew install python git`
2. **Build**
   - `python3 BUILD.py --xcode`
3. **Run**
   - `open "./build/Freya NLE.app"`

### Windows (MSVC, default)

Rust requires the MSVC host on Windows, so an unqualified `python3 BUILD.py`
selects the MSVC builder. MSYS2 remains available as an explicit alternative
for environments that provide the matching GNU Rust target and dependencies.

1. **Additional prerequisites**
   - Visual Studio 2022 or newer Build Tools with the C++ x64/x86 toolset and
     a Windows SDK
   - vcpkg (can be specified via `VCPKG_ROOT`; `BUILD.py` will attempt to fetch
     it if not found)
2. **Build**
   - `python3 BUILD.py`
   - To select MSYS2 explicitly, use `python3 BUILD.py --msys2` from an MSYS2 UCRT64 shell.
3. **Run**
   - `.\build\Freya NLE.exe`

### Windows (MSYS2, explicit alternative)

1. **Install dependencies**
   - `pacman -S git mingw-w64-ucrt-x86_64-python`
2. **Build**
   - `python3 BUILD.py --msys2`
3. **Run**
   - `./build/Freya NLE.exe`

## Toolchain

The Rust version is pinned in `rust-toolchain.toml`, and CI uses
`dtolnay/rust-toolchain@1.97.1`. Make sure the pinned toolchain is the one on
`PATH`: a newer `cargo` from a system package manager takes precedence over the
`rustup` shim and can turn deprecation lints into build failures. Check
`cargo --version` before trusting a result.

## Tests and quality checks

Run the checks from the repository root:

```bash
cargo fmt --manifest-path rust/Cargo.toml --all -- --check
cargo clippy --manifest-path rust/Cargo.toml --locked --workspace --all-targets --all-features -- -D warnings
cargo test --manifest-path rust/Cargo.toml --locked --workspace --all-features
python -m unittest tests.test_build_version
```

Tests that need a GPU adapter, the FFmpeg CLI, or installed Carla libraries are
`#[ignore]`d by default; see
[Adding Tests](https://github.com/GT-610/Freya-NLE/blob/main/CONTRIBUTING.md#adding-tests).
Treat a skipped tool as missing coverage rather than a successful check.
