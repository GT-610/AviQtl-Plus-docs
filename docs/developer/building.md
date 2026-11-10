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
# Linux, portable AppImage (the recommended Linux build)
python3 BUILD.py --appimage

# Linux, Arch-compatible archive
python3 BUILD.py --arch

# macOS bundle
python3 BUILD.py --xcode

# macOS bundle for Intel, cross-compiled on an Apple Silicon host
python3 BUILD.py --xcode --target x86_64-apple-darwin

# Windows with the default MSVC toolchain
python3 BUILD.py

# Windows with MSYS2 (explicit alternative)
python3 BUILD.py --msys2
```

## FFmpeg

Releases link one pinned FFmpeg rather than whatever the host provides, because
the host's copy decides the runtime floor: a distribution's FFmpeg decides which
glibc the binary needs, and a Homebrew bottle decides which macOS it needs.
`scripts/build-ffmpeg.sh` builds that pinned version:

```bash
scripts/build-ffmpeg.sh --target x86_64-unknown-linux-gnu
export FFMPEG_DIR="$PWD/build/ffmpeg/x86_64-unknown-linux-gnu"
```

`--target` is a Rust triple; the script accepts `x86_64-unknown-linux-gnu`,
`aarch64-apple-darwin`, and `x86_64-apple-darwin`. It needs `nasm` for the
assembly kernels and fails without it rather than silently building a much
slower encoder. `BUILD.py` runs it automatically for the macOS and AppImage
targets when `brew` or the container is available; setting `FFMPEG_DIR`
yourself makes `cargo` use your build instead.

MSVC builds require Visual Studio 2022 (17.x) or newer. Visual Studio 2019
and earlier are not supported. `BUILD.py` selects a supported Visual Studio
installation through `vswhere` and rejects older toolsets. The build also
requires vcpkg. See `python3 BUILD.py --help` for the current options.

## Platform notes

### Linux

On Linux the build environment is isolated in a container by default: `distrobox`
when it is installed, and plain `podman` otherwise. The image is pinned to a
digest, because an unpinned image silently changes the glibc and FFmpeg
a rebuild links against.

1. **Install dependencies**
   - Pacman: `sudo pacman -S --needed distrobox podman python git`
   - APT: `sudo apt install distrobox podman python3 git`
   - DNF: `sudo dnf install distrobox podman python3 git`
   - macOS with podman: `brew install podman`, then `podman machine init`
     (distrobox is not needed, and is not packaged for macOS)
2. **Build**
   - `python3 BUILD.py --appimage` for the portable build
   - `python3 BUILD.py --arch` for the Arch-compatible archive
3. **Run**
   - `./build/freya-nle`

The AppImage target builds on a pinned Ubuntu 22.04 image so the binary asks for
the oldest glibc the toolchain supports; the Arch target builds on the pinned
Arch image. Building the AppImage on a newer distribution links that
distribution's glibc and defeats the purpose, which is why CI jobs run inside
the pinned images rather than on the runner's own Ubuntu.

### macOS

On macOS, `BUILD.py` installs its build dependencies via Homebrew, builds the
pinned FFmpeg, then produces and ad-hoc signs a `.app` bundle.

1. **Install dependencies**
   - `brew install python git pkgconf nasm`
2. **Build**
   - `python3 BUILD.py --xcode`
   - Add `--target x86_64-apple-darwin` for the Intel bundle; run
     `rustup target add x86_64-apple-darwin` first
3. **Run**
   - `open "./build/Freya NLE.app"`

`MACOSX_DEPLOYMENT_TARGET` is exported during the build so every dependency
compiled from source agrees on one floor. Packaging measures the result and
declares the honest `LSMinimumSystemVersion`; pass
`--strict-deployment-target` to turn a dependency that needs a newer system
into a build failure instead of a widened support range.

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
   - `.\build\freya-nle.exe`

### Windows (MSYS2, explicit alternative)

1. **Install dependencies**
   - `pacman -S git mingw-w64-ucrt-x86_64-python`
2. **Build**
   - `python3 BUILD.py --msys2`
3. **Run**
   - `./build/freya-nle.exe`

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
