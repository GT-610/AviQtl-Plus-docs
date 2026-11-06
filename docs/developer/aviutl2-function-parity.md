---
title: AviUtl2 function parity
description: What Freya NLE adopted from AviUtl2, what it kept, and what it deliberately left out.
---

# AviUtl2 Function Parity

AviUtl2 (ExEdit2 2.1.16) is not "classic AviUtl with more effects". It is an
internal rewrite: RGBA16F and PCM32F precision, GPU rendering, free window
grouping inside one window, and redefined relationships between scenes,
objects, layers and effects. This page records which of its capabilities were
adopted here, which were not, and why — so the same ground does not have to be
covered twice.

Interface-level alignment is documented separately; this page is about the
editing model and the feature set behind it. The measured baseline is
`8c1efccc` (0.7.0-rc.2).

## How the comparison was made

AviUtl2 was analysed from four kinds of evidence, and every conclusion below
rests on at least one of them:

| Evidence | Source | What it settled |
| --- | --- | --- |
| Operation guide | `aviutl2.txt`, the 簡易説明 section | Feature semantics, interaction detail, documented limits |
| Appearance config | `style.conf`, `system.conf` | Window metrics and colours |
| Binary wide strings | `.rdata`, file offset `0x33d000`, size `0x1fda00` | Menu entries, command names, settings, component names |
| Official documentation | <https://docs.aviutl2.jp/usage.md> and the Lua reference | Window responsibilities, `obj` fields, script semantics |

No function was decompiled to infer an algorithm. The strings and class names
are enough to establish what a feature is, where it is entered and what its
bounds are; questions of implementation only matter once a port is attempted.

## Verdicts

Four verdicts are used. **Keep** means this project's approach is already
better. **Adopt** means it is worth doing as AviUtl2 does it. **Adopt,
differently** means the capability is worth having but the implementation has
to change. **Not doing** means it does not fit this project.

| Capability | AviUtl2 | Freya NLE before | Verdict |
| --- | --- | --- | --- |
| Internal precision | RGBA16F, PCM32F | wgpu pipeline, float compositing | Keep |
| Scene operations in undo | Not supported | Scene add/remove/edit all undoable | Keep |
| Single-window grouping | Yes, detachable, mergeable, tabs | Multiple windows + project tabs + merged workspace | Keep |
| Frame marks with notes | Yes, list and jump | No | Adopt |
| Layer settings (per-layer offset and volume) | Yes | No layer offset; only per-object transform | Adopt |
| Layer name and background colour | Yes | `Layer N` only, no colour | Adopt |
| Text control characters | A complete set | Plain text and object parameters | Adopt |
| Parameter references | Cross-effect, cross-layer, with functions | `=` Lua expressions only (AviUtl 1.x style) | Adopt, differently |
| Effect-level copy/cut/paste | Yes, self and self-and-after | Object clipboard and effect reordering only | Adopt |
| Per-item presets | Yes | Effect-level presets | Adopt, differently |
| Creation and initialisation defaults | Yes, can combine by media | No | Adopt |
| `.object` aliases | Yes | No | Not doing — fragments cover most uses |
| Object list | Yes | No | Adopt |
| Object grouping | Yes | No | Adopt |
| Scene tags with filtering | Yes, `\` hierarchy | No | Adopt |
| Scene reference duplication | Yes | Scene objects, copy semantics unverified | Adopt, differently |
| Batch export | Yes | Single export with progress | Adopt |
| Backup history | Interval, limit, recoverable | Crash recovery snapshot only | Adopt, differently |
| Menu definition (order, nesting, visibility) | Yes, one mechanism | Fixed menus | Not doing — low value, high test cost |
| Configurable shortcuts | Yes | Yes, with conflict checking | Keep |
| Input/output plugin priority | Yes | Goes through FFmpeg, no plugin table | Not doing |
| Cache size setting | Yes | Backend-managed | Not doing — exposing it only invites misconfiguration |
| High DPI / GPU selection | Yes | Decided by the platform and wgpu | Not doing |
| Language files `.aul2` | Yes | gettext `.po` (en, zh, ja) | Keep |
| Drop-in fonts and emoji | Yes | System fonts, no emoji registration | Adopt, differently |
| Image, video and audio objects | Yes | Yes | Keep |
| Group control, camera, frame buffer, scene objects | Yes | Yes | Keep |
| Image composition and some filters | Yes | Frame buffer objects compose lower layers, but layer count, position and control ownership differ | Keep — no separate object |
| Clip objects and upper-layer clipping | Yes | `clipping` effect plus `clipByUpperObject` | Keep |
| BPM grid and snapping | Yes | Yes, scene-level BPM, subdivision, magnet | Keep |
| Ripple, roll and slide | No | Yes | Keep |

## Design decisions

### Adopted

- **Frame marks are scene data, not interface state.** They save with the
  project, roll back with undo and are isolated per scene. A note is part of a
  mark. The mark list is a navigation entry point, not a second timeline.
- **Layer settings are layer properties.** The coordinate and volume offsets
  apply to every media object on that layer; a camera control object is
  unaffected, matching AviUtl2. The layer name and background colour are the
  same set of properties.
- **Text control characters are a parsing layer.** Colour, size, font, position
  and spacing that do not affect timing come first; `<w>`, `<c>` and `<r>`,
  which need per-frame evaluation, come after.
- **References reuse the expression evaluator.** A reference resolves to "the
  parameter value at this frame" and is then handed to the existing Lua
  expression path, so there is no second evaluation semantic.
- **Batch export is a queue.** Each item carries a project path, output settings
  and an overwrite policy. The queue can pause, cancel and reorder; rendering
  reuses the single-export pipeline.
- **Backup history is an explicit version list.** "Keep N versions per project,
  and allow a manual marker" sits on top of the existing recovery snapshot, and
  recovery goes through the existing open path rather than a second project
  format.

### Adopted, differently

- **Object grouping.** AviUtl2's group is a union for selection and movement.
  Here it is a group id plus the objects in it, affecting only the atomicity of
  selection, movement and deletion — not the render order.
- **Scene tags.** `\` hierarchy and filtering are kept, but tags are stored as
  an array rather than parsed out of a string, and the interface folds them by
  level.
- **Per-item presets.** Reuses the existing preset store and extends its scope
  from an effect to an individual effect item.

### Not doing

These are excluded because the alternative is worse, not because they are hard:

- Fully custom-drawn controls and fixed pixel metrics — accessibility and DPI
  handling would regress.
- One document per window — project tabs are better.
- Keeping scene operations out of the undo stack — a known AviUtl2 weakness.
- A Windows-only plugin ABI — this project extends through packages and Lua.
- A user-facing cache size — the backend manages it, and exposing it only
  creates bad configurations.

## What is still not implemented

The comparison above is closed: no row is left unclassified. Six capabilities
are known gaps rather than decisions, and each costs more than it is worth
today. They are listed here so a future attempt starts from the reason.

| Capability | Why it stays open |
| --- | --- |
| `obj.sx` / `obj.sy` / `obj.sz` | The documentation notes that `zoom` and `aspect` compute differently between old and new script formats, and the Qt frontend only ever had an output side (`sx = baseScale * aspectX`), never an input path. The renderer has independent X/Y/Z scales, but the rule for combining them with `zoom` and `aspect` is undefined; guessing it is how a unit error ships |
| `obj.effect()` / `obj.draw()` / `obj.setfont()` | These let a script drive drawing. Text scripts currently run before layout; these calls require them to run during drawing, which means redesigning the script sandbox boundary and the render timing — not adding fields |
| Camera-control lights | Needs layer-plane normals, ten light uniforms, diffuse and specular terms, and shadow ranges. This is a renderer-level change, and correctness cannot be checked without an AviUtl2 reference to compare against |
| `obj.w` / `obj.h` | The size is a layout result, but the script runs before layout |
| Dragging palette swatches to reorder | Pure polish, cost exceeds value |
| Cross-project export batches | Single export and the batch queue already cover the main use; cross-project needs a definition of what merging different projects into one queue means |

## Contributing to this area

If you pick one of these up, state the evidence you are relying on and record
it in the same manner as above. The risk in this area is not effort but
plausible guesses: an incorrect unit or a guessed stacking rule can pass
hundreds of tests while producing visibly wrong output. The `aspect` field is
the cautionary example — it reached the compositor as a percentage while the
compositor read a ratio, and the result was every object stretched 101 times
wide with the full test suite green. Prefer a test that drives the real
pipeline over one that asserts a constant.
