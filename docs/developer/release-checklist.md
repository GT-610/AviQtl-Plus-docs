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

Required platforms:

- Linux runner or Arch container
- macOS runner
- Windows MSVC runner (the default Windows build)

MSYS2 is an explicit alternative and is not the Windows build the release is
verified on.

The `--ignored` tests need a GPU adapter, the FFmpeg CLI with libx264 and AAC, or
installed Carla libraries. Record the platform each one ran on, and record a test
that could not run as unverified rather than passed.

## Packaging and smoke test

- Package each platform with `python3 BUILD.py` (`--msvc` on Windows).
- Confirm the archive contains the executable, `effects`, `objects`, `plugins`,
  `effect-packages`, `repos`, and the translations. `BUILD.py` stages
  `catalog/effects`, `catalog/objects`, `plugins/`, `effect-packages/`, and
  `repos/`; the archive must contain no `Qt6*` runtime files.
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

- Confirm all required GitHub Actions jobs are green for the release commit.
- Create the tag `v<version>` only after the checks above pass. A tag containing
  `-` is published as a GitHub pre-release and does not replace the latest stable
  release.
- Download and smoke-test the artifacts produced by the tag workflow before
  announcing the release.
