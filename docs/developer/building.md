---
title: Build from source
description: Configure and build AviQtl-Plus on Linux, macOS, or Windows.
---

# Build from source

Clone the application repository, including the documentation submodule only if
you want a local copy of this site:

```bash
git clone --recurse-submodules https://github.com/GT-610/AviQtl-Plus.git
cd AviQtl-Plus
```

For a source-only checkout, omit `--recurse-submodules`.

## Automated build

`BUILD.py` detects the host platform, prepares dependencies, configures the
project, and builds it. The normal entry point is:

```bash
python3 BUILD.py
```

### Frontend selection

`BUILD.py` builds the **Rust + Slint frontend by default** (under
construction). While the migration is in progress, the legacy C++/QML (Qt)
frontend remains available:

```bash
# Legacy Qt frontend (debug example)
python3 BUILD.py --xcode --frontend qt --debug

# Qt with an explicit Qt6 install prefix
python3 BUILD.py --xcode --frontend qt --qt-dir <Qt6 prefix>
```

Qt builds run CMake (Ninja) into `.build_tmp/<target>/<Config>/qt-build`,
then stage the raw Qt product — the `AviQtl.app` bundle on macOS, otherwise
the `AviQtl` binary — plus the shared resources into `build/`, with a `-Qt`
suffix on the archive name. These are development builds: platform deployment
(`macdeployqt`/`windeployqt`) is intentionally skipped, so run the matching
deployment tool before distributing a Qt build. `--qt-dir` is only honored
for Qt builds; without it, `qmake6`/`qmake` on `PATH` is used for detection.

Common platform-specific modes include:

```bash
# Linux container-based environment
python3 BUILD.py --arch

# macOS Xcode generator
python3 BUILD.py --xcode

# Windows with MSYS2
python3 BUILD.py --msys2
```

MSVC is supported for development but requires Visual Studio Build Tools, a
matching Qt MSVC installation, and vcpkg. See `python3 BUILD.py --help` for the
current options.

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
   - `./build/AviQtl`

The container also installs Qt 6, LuaJIT, a Vulkan implementation (for example
Mesa), FFmpeg, Carla, and clang (which provides libc++).

### macOS

On macOS, `BUILD.py` checks and installs dependencies via Homebrew (CMake,
Ninja, Qt6, etc.), then runs `macdeployqt` and `codesign` to create a `.app`
bundle.

1. **Install dependencies**
   - `brew install python git`
2. **Build**
   - `python3 BUILD.py --xcode`
3. **Run**
   - `open ./build/AviQtl.app`

### Windows (MSYS2)

1. **Install dependencies**
   - `pacman -S git mingw-w64-ucrt-x86_64-python`
2. **Build**
   - `python3 BUILD.py --msys2`
3. **Run**
   - `./build/AviQtl.exe`

### Windows (MSVC - not recommended)

MSVC builds are not recommended due to the complexity of environment setup.

1. **Additional prerequisites**
   - Visual Studio 2022 Build Tools with C++ toolset
   - Official Qt MSVC x64 build (for example `msvc2022_64`)
   - vcpkg (can be specified via `VCPKG_ROOT`; `BUILD.py` will attempt to fetch
     it if not found)
2. **Build**
   - `python3 BUILD.py --msvc --qt-dir <Qt installation directory>`
   - If `--qt-dir` is omitted, `qmake6`/`qmake` on `PATH` is used for
     Qt detection.
3. **Run**
   - `.\build\AviQtl.exe`

## Qt version compatibility

The project currently uses Qt private modules for ZIP handling, QRhi
integration, and shader tooling. Build-time and runtime Qt patch versions must
match. Rebuild after upgrading Qt packages.

## Tests and quality checks

Run tests from the configured build directory:

```bash
ctest --test-dir build --output-on-failure
```

The repository also provides `check.py` for formatting, static analysis, and
other quality checks. Treat a skipped tool as missing coverage rather than a
successful check.
