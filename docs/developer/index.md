---
title: Developer documentation
description: Build, understand, extend, and contribute to Freya NLE.
---

# Developer documentation

Freya NLE is a shared Rust workspace (timeline, effects, audio,
rendering, media) behind the **Rust + Slint + wgpu** UI, the only frontend since
the Qt Quick frontend was retired in 0.8. The workspace provides typed
timeline data, FFmpeg media processing, Lua plugins, native effect packages,
and wgpu rendering.

## Start contributing

1. Read [Build from source](./building).
2. Review the main repository's
   [contribution guide](https://github.com/GT-610/Freya-NLE/blob/main/CONTRIBUTING.md).
3. Run the relevant automated tests before and after a change.
4. Update user documentation when behavior or interface labels change.

## Extension paths

- [Effects and objects](./effects): native package JSON metadata and the
  `aviqtl-wgsl-v1` render contract.
- [Plugin development](./plugins): Lua automation, lifecycle hooks, permissions,
  and package distribution.
- Native application development: Rust, Slint, wgpu, timeline services, media
  decoding, rendering, audio, and tests in the main repository.

## Product targets and workflows

- [AviUtl operability targets](./operability-targets): the editing-model
  compatibility goals that guide development.
- [AviUtl2 function parity](./aviutl2-function-parity): what was adopted from
  AviUtl2, what was kept, and what is deliberately out of scope.
- [Timeline edit target rules](./timeline-edit-targets): where timeline commands
  act.
- [Fragment files](./fragment-files): the native format for saving a block of
  clips, and how it relates to the AviUtl object file.
- [AviUtl object files](./exo-object-files): what `.exo` import translates,
  preserves, and deliberately leaves alone.
- [Release checklist](./release-checklist): requirements for a valid release.

User workflows remain in the [user guide](/guide/start-here/) so API and architecture
details do not interrupt task-oriented instructions.
