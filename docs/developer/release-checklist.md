---
title: Release checklist
description: Requirements for creating a Freya NLE release.
---

# Release Checklist

Use this checklist before creating a Freya NLE tag. A release is valid only when
the tagged commit passes the checks below from a clean build. A skipped check or
a build left over from an earlier commit is not evidence.

## Version

- `[workspace.package] version` in `rust/Cargo.toml` is the single source of the
  project version. `BUILD.py` reads it, and the release workflow rejects a tag
  that is not exactly `v<version>`.
- A pre-release uses the dotted counter form: version `0.8.0-rc.1`, tag
  `v0.8.0-rc.1`. Do not use `-rc1` or `-rc-1`; see
  [Versions and releases](https://github.com/GT-610/Freya-NLE/blob/main/CONTRIBUTING.md#versions-and-releases).
- The macOS bundle metadata carries the numeric core of the same version
  (`0.8.0` for `0.8.0-rc.1`), because the bundle format cannot express a suffix.
  `BUILD.py` derives it from the workspace version, so there is no second place
  to edit.
- `CHANGELOG.md` has a `## [<version>] - <YYYY-MM-DD>` section for the version
  being tagged, and `Unreleased` is empty.
- `python -m unittest tests.test_build_version` passes; it fails when the version,
  the tag form, and the changelog disagree.
- Confirm the About dialog of the packaged build reports the workspace version.

## Rust frontend

Rust + Slint + wgpu is the only frontend. Run from a clean `rust/target`:

```text
cargo fmt --manifest-path rust/Cargo.toml --all -- --check
cargo clippy --manifest-path rust/Cargo.toml --locked --workspace --all-targets --all-features -- -D warnings
cargo test --manifest-path rust/Cargo.toml --locked --workspace --all-features
cargo test --manifest-path rust/Cargo.toml --locked -p freya-nle-media -p freya-nle-audio -- --ignored
python -m unittest tests.test_build_version
```

The pinned toolchain in `rust-toolchain.toml` must be the one on `PATH`. A
newer `cargo` installed by a system package manager takes precedence over the
`rustup` shim and turns deprecation lints in the pinned release into failures, so
check `cargo --version` before trusting a red or green result.

Required platforms, one release artifact each:

- Linux, portable: `Freya-NLE-<version>-linux-x86_64.AppImage`, built on the
  pinned Ubuntu 22.04 image and linked against the pinned FFmpeg
- Linux, Arch-compatible: `Freya-NLE-<version>-linux-x86_64-archlinux.tar.zst`,
  built on the pinned Arch image
- macOS, Apple Silicon and Intel: one `macos-15` runner produces both
- Windows MSVC: the default Windows build

MSYS2 is an explicit alternative and is not the Windows build the release is
verified on. Intel macOS is cross-compiled rather than built on an Intel runner.

The `--ignored` tests need a GPU adapter, the FFmpeg CLI with libx264 and AAC, or
installed Carla libraries. Record the platform each one ran on, and record a test
that could not run as unverified rather than passed.

## Packaging and smoke test

- Package each platform with `python3 BUILD.py` (`--msvc` on Windows). The CI
  package workflow does this for every target, so a tag build is evidence only
  when each of its jobs is green.
- Confirm the archive contains the executable, `effects`, `objects`, `plugins`,
  `effect-packages`, `repos`, `licenses`, and `build-info.json`. `BUILD.py`
  stages `catalog/effects`, `catalog/objects`, `plugins/`, `effect-packages/`,
  and `repos/`; the archive must contain no `Qt6*` runtime files.
- Confirm `build-info.json` names the commit being released, and that its
  declared requirements match the platform: the glibc version for the
  `-archlinux` archive, and the minimum macOS for the bundles.
- On macOS, confirm every binary in the bundle is covered by
  `LSMinimumSystemVersion`. Packaging measures this and `package.yml` fails the
  build when a library needs a newer system than the plist claims.
- On Windows, confirm `vcruntime140.dll` and `vcruntime140_1.dll` are inside the
  archive, and that Explorer reports the version from the executable's
  embedded metadata.
- Every bundled package under `effect-packages/` must declare a
  `min_app_version` no newer than the version being released, otherwise the
  package manager refuses to install it.
- Start the packaged build once with `--validate-gpu` and confirm the reported
  backend, the frame counts, and an empty `wgpu_errors` list.
- On a host whose only adapter is a software renderer, confirm startup reports
  the unsupported host instead of crashing.
- Launch the packaged application, create a project, save and reopen it, render a
  short MP4, and verify the output outside the editor.
- Verify installation and startup on a machine or clean virtual machine without
  the build toolchain.

## Publication

- Confirm all required GitHub Actions jobs are green for the release commit. A
  tag runs `package.yml` through `release.yml`; the release job refuses to
  publish if a target is missing or an unexpected file is in `dist/`.
- Create the tag `v<version>` only after the checks above pass. A tag containing
  `-` is published as a GitHub pre-release and does not replace the latest stable
  release.
- Confirm the release carries exactly these five files plus `SHA256SUMS`:
  `Freya-NLE-<version>-linux-x86_64.AppImage`,
  `Freya-NLE-<version>-linux-x86_64-archlinux.tar.zst`,
  `Freya-NLE-<version>-macos-arm64.zip`,
  `Freya-NLE-<version>-macos-x86_64.zip`,
  `Freya-NLE-<version>-windows-x86_64.zip`.
- Download and smoke-test the published artifacts before announcing the release.
  Testing the locally built copy is not evidence about what was uploaded.
