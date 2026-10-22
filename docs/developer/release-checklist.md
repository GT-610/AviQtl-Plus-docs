---
title: Release checklist
description: Requirements for creating an AviQtl-Plus release.
---

# Release Checklist

Use this checklist before creating an AviQtl-Plus tag. A release is valid only
when the tagged commit passes the checks below from a clean build. A skipped
check or a build left over from an earlier commit is not evidence.

## Version

- `[workspace.package] version` in `rust/Cargo.toml` is the single source of the
  project version. `BUILD.py` reads it, and the release workflow rejects a tag
  that is not exactly `v<version>`.
- A pre-release uses the dotted counter form: version `0.7.0-rc.1`, tag
  `v0.7.0-rc.1`. Do not use `-rc1` or `-rc-1`; see
  [Versions and releases](https://github.com/GT-610/AviQtl-Plus/blob/main/CONTRIBUTING.md#versions-and-releases).
- `project(AviQtl VERSION ...)` in `CMakeLists.txt` carries the numeric core of
  the same version (`0.7.0` for `0.7.0-rc.1`), because CMake cannot express a
  suffix. The macOS bundle metadata is written from that core.
- `CHANGELOG.md` has a `## [<version>] - <YYYY-MM-DD>` section for the version
  being tagged, and `Unreleased` is empty.
- `python -m unittest tests.test_build_version` passes; it fails when the version,
  the tag form, `CMakeLists.txt`, and the changelog disagree.
- Confirm the About dialog of the packaged build reports the workspace version.

## Rust frontend

The default and shipped frontend is Rust + Slint + wgpu. Run from a clean
`rust/target`:

```text
cargo fmt --manifest-path rust/Cargo.toml --all -- --check
cargo clippy --manifest-path rust/Cargo.toml --locked --workspace --all-targets --all-features -- -D warnings
cargo test --manifest-path rust/Cargo.toml --locked --workspace --all-features
cargo test --manifest-path rust/Cargo.toml --locked -p aviqtl-media -p aviqtl-audio -- --ignored
python -m unittest tests.test_build_version
python tests/test_rust_abi_contract.py
python tests/check_rust_abi.py --rust-src rust/aviqtl-core/src --header core/include/rust_core_abi.hpp
```

Required platforms:

- Linux runner or Arch container
- macOS runner
- Windows MSVC runner (the default Windows build)

MSYS2 is an explicit alternative and is not the Windows build the release is
verified on.

The `--ignored` tests need a GPU adapter, the FFmpeg CLI with libx264 and AAC, or
installed Carla libraries. Record the platform each one ran on, and record a test
that could not run as unverified rather than passed.

## Qt reference frontend

Qt/C++ and QML are kept as a behavior reference and are not part of the release
archives. When a change touches shared Rust code or the C++ adapters, build it
and run CTest before tagging:

```text
cmake -S . -B .build_tmp/qt-release -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build .build_tmp/qt-release --parallel 2
ctest --test-dir .build_tmp/qt-release --output-on-failure
```

## Packaging and smoke test

- Package each platform with `python3 BUILD.py` (`--msvc` on Windows).
- Confirm the archive contains the executable, `effects`, `objects`, `plugins`,
  `effect-packages`, `repos`, the translations, and the required FFmpeg and Carla
  runtime libraries, and that it contains no `Qt6*` runtime files.
- Start the packaged build once with `--validate-gpu` and confirm the reported
  backend, the frame counts, and an empty `wgpu_errors` list.
- On a host whose only adapter is a software renderer, confirm startup reports
  the unsupported host instead of crashing.
- Launch the packaged application, create a project, save and reopen it, render a
  short MP4, and verify the output outside the editor.
- Verify installation and startup on a machine or clean virtual machine without
  the build toolchain.

## Publication

- Confirm all required GitHub Actions jobs are green for the release commit.
- Create the tag `v<version>` only after the checks above pass. A tag containing
  `-` is published as a GitHub pre-release and does not replace the latest stable
  release.
- Download and smoke-test the artifacts produced by the tag workflow before
  announcing the release.
