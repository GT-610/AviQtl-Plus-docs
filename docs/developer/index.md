---
title: Developer documentation
description: Build, understand, extend, and contribute to AviQtl-Plus.
---

# Developer documentation

AviQtl-Plus pairs a shared Rust workspace (timeline, effects, audio,
rendering, media) with two frontends: the default **Rust + Slint** UI (under
construction) and the legacy **C++23 / Qt 6 + Qt Quick** UI with QRhi
rendering, ECS-oriented timeline data, FFmpeg media processing, LuaJIT plugins,
and QML/GLSL effect packages. Select the frontend with
`BUILD.py --frontend {slint,qt}` (default: `slint`).

## Start contributing

1. Read [Build from source](./building).
2. Review the main repository's
   [contribution guide](https://github.com/GT-610/AviQtl-Plus/blob/main/CONTRIBUTING.md).
3. Run the relevant automated tests before and after a change.
4. Update user documentation when behavior or interface labels change.

## Extension paths

- [Effects and objects](./effects): native package JSON metadata and the
  `aviqtl-wgsl-v1` render contract.
- [Plugin development](./plugins): LuaJIT automation, lifecycle hooks,
  permissions, and package distribution.
- Native application development: C++, Qt Quick, timeline services, media
  decoding, rendering, audio, and tests in the main repository.

## Product targets and workflows

- [AviUtl operability targets](./operability-targets): the editing-model
  compatibility goals that guide development.
- [Timeline edit target rules](./timeline-edit-targets): where timeline commands
  act.
- [Release checklist](./release-checklist): requirements for a valid release.

User workflows remain in the [user guide](/guide/start-here/) so API and architecture
details do not interrupt task-oriented instructions.
