---
title: Fragment files (.aviqtlfrag)
description: How AviQtl-Plus saves a block of clips as a fragment, and why that is the default export rather than an AviUtl object file.
---

# Fragment files (`.aviqtlfrag`)

A **fragment** is a block of clips saved on its own: a title card, an effect chain, a whole
scene's worth of layered objects. It is what a user reaches for when work has to leave the
project it was made in.

AviUtl solves the same problem with the [object file](./exo-object-files) (`.exo`), and
AviQtl-Plus reads those. Writing one, though, means translating every clip into ExEdit's
vocabulary, and that vocabulary is smaller: whatever AviQtl-Plus can express and ExEdit cannot
has to be dropped. Guessing at a mapping for something ExEdit does not have would be worse
than dropping it, because the file would look complete and render differently.

A fragment has neither problem. It stores the project's own clip representation, so nothing is
translated and nothing is lost.

- **File → Insert Fragment…** appends the file's clips after the current scene's last clip, as
  one undoable edit.
- **Dropping a fragment on the timeline** places it at the pointer, anchored at its own first
  frame and lowest layer.
- **File → Export Fragment…** writes the selected clips, or the whole scene when nothing is
  selected.

## Choosing between the two formats

| | Fragment (`.aviqtlfrag`) | AviUtl object file (`.exo`) |
| --- | --- | --- |
| Import | Lossless | Translated; anything unknown becomes a disabled placeholder |
| Export | Lossless, every clip | Only the objects ExEdit can describe; the rest is reported as skipped |
| Opens in AviUtl | No | Yes |
| Reads AviUtl material | No | Yes |
| Keeps AviQtl-Plus-only parameters | Yes | No |

Reach for a fragment to move work between AviQtl-Plus projects, and for an object file only
when the other end is AviUtl.

## File format

A fragment is UTF-8 JSON, with a marker that tells it apart from a project document:

```json
{
  "format": "aviqtl.fragment",
  "version": 1,
  "settings": { "width": 1920, "height": 1080, "fps": 60, "sampleRate": 48000 },
  "clips": [
    {
      "id": 2,
      "sceneId": 1,
      "type": "text",
      "start": 30,
      "duration": 20,
      "layer": 1,
      "clipByUpperObject": false,
      "params": {},
      "audioPlugins": [],
      "effects": [ ... ]
    }
  ]
}
```

- `format` is always `aviqtl.fragment`. A reader rejects a file whose marker is missing or
  different, which is how a project document opened by mistake is caught before it produces
  nonsense.
- `version` is the layout version. A reader refuses versions it does not know rather than
  guessing, because a half-understood fragment would import silently wrong.
- `settings` records the frame rate, frame size, and audio rate the clips were authored
  against. Nothing is scaled on import; an import into a scene that disagrees says so in its
  report instead of quietly restretching the timing.
- `clips` holds the clips exactly as the project stores them, including any field this build
  writes and does not yet read. `id` and `sceneId` are replaced on import.

The largest fragment this build will read is 64 MiB, and the largest clip count is 100,000.
A fragment references media by path; it does not contain the media itself.

## What an import does

1. Every clip gets a fresh id from the scene's own id space, so importing the same fragment
   twice never collides.
2. `start` and `layer` are shifted by the requested offset. A layer pushed past the end of the
   timeline is clamped to the last layer instead of failing the import.
3. The whole block moves forward if it would land on frames that already hold objects, keeping
   its internal layout the way a drag-move of a selection does. Laying it on top instead would
   leave two clips sharing a layer and a frame, which the renderer resolves by stacking order
   rather than by intent.
4. The scene is extended when the block reaches past its end, as part of the same undoable
   edit. An import never shortens a scene.

Notes appear in the report when the block had to move or when the destination's frame rate or
frame size differs from what the fragment was made for.

## Limits

- **One scene at a time.** A fragment carries clips from a single scene; it does not carry
  scene definitions, nested durations, or other scenes of the project.
- **Media is referenced, not bundled.** Moving a fragment to another machine needs the media
  to come along or to be relinked; **File → Manage Missing Media** reports what is absent.
- **No automatic migration.** Version 1 is the only layout, and a newer file is refused rather
  than partially read.

## Related

- [AviUtl object files](./exo-object-files) — the compatibility format, and what it translates.
- [AviUtl operability targets](./operability-targets) — the editing-model goals this serves.
